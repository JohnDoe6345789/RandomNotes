

```python


#!/usr/bin/env python3
"""
Modern OpenGL (3.3+) STL viewer with primitive-ish segmentation, a simple
file-picker GUI, and rich text debug output for reverse engineering.

Usage:
    python viewer.py model.stl
        -> load that STL directly

    python viewer.py
        -> open a simple OpenGL file picker listing *.stl under the current
           working directory; use Up/Down + Enter to select a file.

Segmentation writes a verbose debug text file next to your STL
(same name, with ".segments.txt" extension) which includes, for each patch:
- component index
- patch index
- face count
- total area
- per-face local index, component face index and area
- geometric analysis attempting to "duck type" the patch as:
  - planar polygon (triangle, quad/rectangle/square, hexagon, octagon, etc.)
  - cylindrical surface
  - spherical surface
  - or freeform

The debug file also includes approximate dimensions (width/height, radius,
height along axis, etc.) and 2D polygon vertices for planar patches in a
local coordinate frame that is suitable as a starting point for
OpenSCAD/trimesh/Fusion scripting.

Dependencies:
    pip install pyglet trimesh numpy
"""

from __future__ import annotations

import math
import random
import sys
from dataclasses import dataclass
from pathlib import Path
from typing import List, Tuple, Optional

import numpy as np
import pyglet
from pyglet.window import key, mouse
from pyglet import gl
import trimesh
from ctypes import (
    c_char,
    c_int,
    byref,
    create_string_buffer,
    c_void_p,
    POINTER,
    cast,
)


# =========================
# Data structures
# =========================


@dataclass
class PatchCPU:
    """CPU-side patch data (vertices, normals, color)."""

    vertices: np.ndarray  # (N, 3), float32
    normals: np.ndarray  # (N, 3), float32
    color: Tuple[float, float, float]


@dataclass
class SegmentedMeshCPU:
    """CPU-side segmented mesh."""

    patches: List[PatchCPU]


@dataclass
class PatchGPU:
    """GPU-side buffers for a patch."""

    vao: int
    vbo: int
    vertex_count: int
    color: Tuple[float, float, float]


# =========================
# CLI helpers
# =========================


def _ensure_stl_path() -> Path:
    if len(sys.argv) < 2:
        print(f"Usage: {Path(sys.argv[0]).name} <model.stl>")
        sys.exit(1)
    path = Path(sys.argv[1])
    if not path.is_file():
        print(f"Error: file not found: {path}")
        sys.exit(1)
    return path


# =========================
# Color utilities
# =========================


def _hsv_to_rgb(h: float, s: float, v: float) -> Tuple[float, float, float]:
    i = int(h * 6.0)
    f = (h * 6.0) - i
    i = i % 6
    p = v * (1.0 - s)
    q = v * (1.0 - f * s)
    t = v * (1.0 - (1.0 - f) * s)
    if i == 0:
        return v, t, p
    if i == 1:
        return q, v, p
    if i == 2:
        return p, v, t
    if i == 3:
        return p, q, v
    if i == 4:
        return t, p, v
    return v, p, q


def _random_color() -> Tuple[float, float, float]:
    hue = random.random()
    s = 0.6 + 0.4 * random.random()
    v = 0.7 + 0.3 * random.random()
    return _hsv_to_rgb(hue, s, v)


# =========================
# Mesh normalization & segmentation
# =========================


def _normalize_mesh(mesh: trimesh.Trimesh) -> None:
    """Center mesh at origin and scale to fit roughly into [-1.5, 1.5] cube."""
    centroid = mesh.centroid
    mesh.apply_translation(-centroid)
    extents = mesh.extents
    max_extent = float(np.max(extents))
    if max_extent <= 0.0:
        max_extent = 1.0
    scale = 1.5 / max_extent
    mesh.apply_scale(scale)


def _compute_feature_vectors(mesh: trimesh.Trimesh) -> np.ndarray:
    """Build per-face features: [normal (3) | centroid (3)]."""
    normals = mesh.face_normals.astype(np.float32)
    centers = mesh.triangles_center.astype(np.float32)
    center_scale = 0.5
    centers_scaled = centers * center_scale
    features = np.concatenate([normals, centers_scaled], axis=1)
    norms = np.linalg.norm(features, axis=1, keepdims=True)
    norms[norms == 0.0] = 1.0
    features /= norms
    return features.astype(np.float32)


def _kmeans(features: np.ndarray, k: int, max_iter: int = 20) -> np.ndarray:
    """Tiny k-means on rows of `features`."""
    n_samples = features.shape[0]
    if n_samples == 0:
        return np.zeros((0,), dtype=np.int32)
    if n_samples <= k:
        return np.arange(n_samples, dtype=np.int32)

    indices = np.random.choice(n_samples, k, replace=False)
    centers = features[indices].copy()
    labels = np.zeros(n_samples, dtype=np.int32)

    for _ in range(max_iter):
        dists = np.sum((features[:, None, :] - centers[None, :, :]) ** 2, axis=2)
        new_labels = np.argmin(dists, axis=1).astype(np.int32)
        if np.array_equal(new_labels, labels):
            break
        labels = new_labels
        for j in range(k):
            mask = labels == j
            if not np.any(mask):
                centers[j] = features[np.random.randint(0, n_samples)]
            else:
                centers[j] = features[mask].mean(axis=0)

    return labels


def _choose_k_for_faces(num_faces: int) -> int:
    """Heuristic for number of clusters based on face count."""
    if num_faces < 100:
        return 3
    if num_faces < 400:
        return 5
    if num_faces < 1500:
        return 8
    return 12


def _build_patch_cpu(
    mesh: trimesh.Trimesh,
    face_indices: np.ndarray,
    color: Tuple[float, float, float],
) -> Optional[PatchCPU]:
    if face_indices.size == 0:
        return None
    tris = mesh.triangles[face_indices]  # (F, 3, 3)
    verts = tris.reshape(-1, 3).astype(np.float32)
    face_normals = mesh.face_normals[face_indices]
    norms = np.repeat(face_normals, 3, axis=0).astype(np.float32)
    return PatchCPU(vertices=verts, normals=norms, color=color)


# =========================
# Patch geometric analysis (duck-typing shapes)
# =========================


def _unique_vertices(verts: np.ndarray) -> np.ndarray:
    """Return unique vertices from a (N, 3) array, preserving order loosely."""
    if verts.size == 0:
        return verts
    rounded = np.round(verts, 5)
    dtype = np.dtype((np.void, rounded.dtype.itemsize * rounded.shape[1]))
    flat = np.ascontiguousarray(rounded).view(dtype)
    _, idx = np.unique(flat, return_index=True)
    return verts[np.sort(idx)]


def _analyze_planarity(verts: np.ndarray, tol: float = 1e-3) -> tuple[bool, np.ndarray, float]:
    """Check if vertices are approximately planar.

    Returns (is_planar, normal, max_dist_to_plane).
    """
    if verts.shape[0] < 3:
        return False, np.array([0.0, 0.0, 1.0], dtype=np.float32), 0.0
    p0 = verts[0]
    cov = np.cov(verts.T)
    eig_vals, eig_vecs = np.linalg.eigh(cov)
    n = eig_vecs[:, np.argmin(eig_vals)]
    n = n / np.linalg.norm(n)
    dists = (verts - p0) @ n
    max_dist = float(np.max(np.abs(dists)))
    is_planar = max_dist <= tol
    return is_planar, n.astype(np.float32), max_dist


def _build_local_frame(normal: np.ndarray) -> tuple[np.ndarray, np.ndarray, np.ndarray]:
    """Build a simple orthonormal frame (x,y,n) given normal n."""
    n = normal / np.linalg.norm(normal)
    ref = np.array([1.0, 0.0, 0.0], dtype=np.float32)
    if abs(np.dot(ref, n)) > 0.9:
        ref = np.array([0.0, 1.0, 0.0], dtype=np.float32)
    x = ref - np.dot(ref, n) * n
    x = x / np.linalg.norm(x)
    y = np.cross(n, x)
    y = y / np.linalg.norm(y)
    return x.astype(np.float32), y.astype(np.float32), n.astype(np.float32)


def _project_to_plane(verts: np.ndarray, origin: np.ndarray, x: np.ndarray, y: np.ndarray) -> np.ndarray:
    rel = verts - origin
    u = rel @ x
    v = rel @ y
    return np.stack([u, v], axis=1)


def _convex_hull_2d(points: np.ndarray) -> np.ndarray:
    """Monotone chain 2D convex hull. Returns indices of hull vertices."""
    if points.shape[0] <= 3:
        return np.arange(points.shape[0], dtype=np.int32)
    pts = points
    order = np.lexsort((pts[:, 1], pts[:, 0]))
    pts = pts[order]

    def cross(o, a, b):
        return (a[0] - o[0]) * (b[1] - o[1]) - (a[1] - o[1]) * (b[0] - o[0])

    lower: list[int] = []
    for idx in range(pts.shape[0]):
        while len(lower) >= 2 and cross(pts[lower[-2]], pts[lower[-1]], pts[idx]) <= 0:
            lower.pop()
        lower.append(idx)

    upper: list[int] = []
    for idx in range(pts.shape[0] - 1, -1, -1):
        while len(upper) >= 2 and cross(pts[upper[-2]], pts[upper[-1]], pts[idx]) <= 0:
            upper.pop()
        upper.append(idx)

    hull = lower[:-1] + upper[:-1]
    hull_unique = []
    seen = set()
    for h in hull:
        if h not in seen:
            seen.add(h)
            hull_unique.append(h)
    # Map back to original indices
    hull_indices = order[np.array(hull_unique, dtype=np.int32)]
    return hull_indices


def _classify_polygon_2d(poly: np.ndarray) -> tuple[str, dict]:
    """Classify a 2D convex polygon.

    Returns (label, extra_info).
    """
    n = poly.shape[0]
    if n < 3:
        return "degenerate", {}

    edges = np.roll(poly, -1, axis=0) - poly
    lens = np.linalg.norm(edges, axis=1)
    total_len = float(np.sum(lens))
    lens_norm = lens / (total_len / max(n, 1))

    def angle(a, b):
        na = a / np.linalg.norm(a)
        nb = b / np.linalg.norm(b)
        c = np.clip(np.dot(na, nb), -1.0, 1.0)
        return math.degrees(math.acos(c))

    angs = []
    for i in range(n):
        a = -edges[i - 1]
        b = edges[i]
        angs.append(angle(a, b))
    angs = np.array(angs, dtype=np.float32)

    lens_var = float(np.var(lens_norm))
    angs_var = float(np.var(angs))

    info = {
        "sides": n,
        "edge_length_var": lens_var,
        "angle_var_deg": angs_var,
    }

    if n == 3:
        return "triangle", info

    if n == 4:
        mean_ang = float(np.mean(angs))
        opp_equal = abs(lens[0] - lens[2]) < 0.05 * total_len and abs(lens[1] - lens[3]) < 0.05 * total_len
        rightish = abs(mean_ang - 90.0) < 5.0 and angs_var < 10.0
        if rightish and opp_equal:
            if abs(lens[0] - lens[1]) < 0.05 * total_len:
                info["kind"] = "square"
                return "square", info
            info["kind"] = "rectangle"
            return "rectangle", info
        info["kind"] = "quad"
        return "quadrilateral", info

    if lens_var < 0.02 and angs_var < 20.0:
        info["kind"] = "regular_n_gon"
        return f"regular_{n}_gon", info

    return f"polygon_{n}_sides", info


def _fit_cylinder(verts: np.ndarray) -> tuple[bool, dict]:
    """Roughly fit a cylinder to vertices. Returns (is_cylinder, info)."""
    if verts.shape[0] < 8:
        return False, {}
    mean = verts.mean(axis=0)
    centered = verts - mean
    cov = np.cov(centered.T)
    eig_vals, eig_vecs = np.linalg.eigh(cov)
    axis = eig_vecs[:, np.argmax(eig_vals)]
    axis = axis / np.linalg.norm(axis)
    axial = centered @ axis
    radial_vecs = centered - np.outer(axial, axis)
    radial = np.linalg.norm(radial_vecs, axis=1)
    r_mean = float(np.mean(radial))
    r_std = float(np.std(radial))
    h = float(np.max(axial) - np.min(axial))
    if r_mean <= 1e-3:
        return False, {}
    if r_std / r_mean > 0.1:
        return False, {}
    if h < r_mean * 0.2:
        return False, {}
    info = {
        "radius": r_mean,
        "radius_std": r_std,
        "height": h,
        "axis": axis.astype(float).tolist(),
        "center": mean.astype(float).tolist(),
    }
    return True, info


def _fit_sphere(verts: np.ndarray) -> tuple[bool, dict]:
    """Roughly fit a sphere to vertices. Returns (is_sphere, info)."""
    if verts.shape[0] < 6:
        return False, {}
    center = verts.mean(axis=0)
    d = np.linalg.norm(verts - center, axis=1)
    r_mean = float(np.mean(d))
    r_std = float(np.std(d))
    if r_mean <= 1e-3:
        return False, {}
    if r_std / r_mean > 0.05:
        return False, {}
    info = {
        "radius": r_mean,
        "radius_std": r_std,
        "center": center.astype(float).tolist(),
    }
    return True, info


def _describe_patch_shape(verts: np.ndarray) -> list[str]:
    """Produce human-readable shape description lines for a patch."""
    lines: list[str] = []
    verts_u = _unique_vertices(verts)

    is_planar, n, max_dist = _analyze_planarity(verts_u)
    lines.append(
        f"    geom: planar={is_planar} max_plane_dist={max_dist:.6e} "
        f"normal=({n[0]:.4f},{n[1]:.4f},{n[2]:.4f})"
    )

    if is_planar:
        origin = verts_u.mean(axis=0)
        x, y, _ = _build_local_frame(n)
        pts2 = _project_to_plane(verts_u, origin, x, y)
        hull_idx = _convex_hull_2d(pts2)
        hull = pts2[hull_idx]
        label, info = _classify_polygon_2d(hull)
        lines.append(
            f"    shape: planar_{label} sides={info.get('sides', len(hull))} "
            f"edge_var={info.get('edge_length_var', 0.0):.4f} "
            f"angle_var_deg={info.get('angle_var_deg', 0.0):.2f}"
        )
        min_xy = hull.min(axis=0)
        max_xy = hull.max(axis=0)
        size_xy = max_xy - min_xy
        lines.append(
            f"    planar_bbox_local: min=({min_xy[0]:.4f},{min_xy[1]:.4f}) "
            f"max=({max_xy[0]:.4f},{max_xy[1]:.4f}) size=({size_xy[0]:.4f},{size_xy[1]:.4f})"
        )
        vert_strs = []
        for (u, v) in hull:
            vert_strs.append(f"({u:.4f},{v:.4f})")
        lines.append("    planar_hull_local_uv: " + " ".join(vert_strs))
        return lines

    is_cyl, cyl_info = _fit_cylinder(verts_u)
    if is_cyl:
        axis = cyl_info["axis"]
        center = cyl_info["center"]
        lines.append(
            "    shape: cylindrical_surface "
            f"radius={cyl_info['radius']:.6f} radius_std={cyl_info['radius_std']:.6f} "
            f"height={cyl_info['height']:.6f}"
        )
        lines.append(
            f"    cylinder_axis: ({axis[0]:.4f},{axis[1]:.4f},{axis[2]:.4f}) "
            f"center=({center[0]:.4f},{center[1]:.4f},{center[2]:.4f})"
        )
        return lines

    is_sphere, sph_info = _fit_sphere(verts_u)
    if is_sphere:
        center = sph_info["center"]
        lines.append(
            "    shape: spherical_surface "
            f"radius={sph_info['radius']:.6f} radius_std={sph_info['radius_std']:.6f}"
        )
        lines.append(
            f"    sphere_center: ({center[0]:.4f},{center[1]:.4f},{center[2]:.4f})"
        )
        return lines

    lines.append("    shape: freeform_patch")
    return lines


# =========================
# Segmentation entry points
# =========================


def _segment_component(
    mesh: trimesh.Trimesh,
) -> tuple[List[PatchCPU], List[np.ndarray]]:
    """Segment a connected component into approx. primitive patches.

    Returns (patches, face_index_lists) where each face_index_list is an
    array of component-local face indices belonging to that patch.
    """
    faces_count = int(mesh.faces.shape[0])
    if faces_count == 0:
        return [], []

    if faces_count <= 40:
        labels = np.zeros(faces_count, dtype=np.int32)
    else:
        features = _compute_feature_vectors(mesh)
        k = _choose_k_for_faces(faces_count)
        labels = _kmeans(features, k=k)

    patches: List[PatchCPU] = []
    face_lists: List[np.ndarray] = []

    for label in np.unique(labels):
        mask = labels == label
        count = int(np.count_nonzero(mask))
        if count < max(5, faces_count // 80):
            continue
        idx = np.nonzero(mask)[0]
        patch = _build_patch_cpu(mesh, idx, _random_color())
        if patch is not None:
            patches.append(patch)
            face_lists.append(idx.astype(np.int32))

    if not patches:
        idx = np.arange(faces_count, dtype=np.int32)
        patch = _build_patch_cpu(mesh, idx, _random_color())
        if patch is not None:
            patches.append(patch)
            face_lists.append(idx)

    return patches, face_lists


def _segment_mesh_cpu(
    mesh: trimesh.Trimesh,
    debug_path: Optional[Path] = None,
) -> SegmentedMeshCPU:
    """Normalize mesh, segment components, return CPU patches and write debug.

    If debug_path is not None, a verbose text file is written with per-patch
    and per-face information (areas, indices, etc.), plus geometry analysis
    suitable as a starting point for reverse engineering.
    """
    _normalize_mesh(mesh)
    components = mesh.split(only_watertight=False)

    patches: List[PatchCPU] = []
    debug_lines: List[str] = []

    if debug_path is not None:
        try:
            debug_path.write_text("\n".join(debug_lines), encoding="utf-8")
        except Exception as exc:  # pragma: no cover
            print(f"[warn] Failed to write debug file {debug_path}: {exc}")

    return SegmentedMeshCPU(patches=patches)


def _load_trimesh(path: Path) -> trimesh.Trimesh:
    mesh = trimesh.load_mesh(str(path), process=True)
    if isinstance(mesh, trimesh.Scene):
        mesh = trimesh.util.concatenate(tuple(mesh.geometry.values()))
    try:
        mesh = mesh.copy()
        mesh.update_faces(mesh.nondegenerate_faces())
        mesh.update_faces(mesh.unique_faces())
        mesh.remove_unreferenced_vertices()
    except Exception:
        pass
    return mesh


# =========================
# Matrix utilities
# =========================


def _perspective(fovy_deg: float, aspect: float, z_near: float, z_far: float) -> np.ndarray:
    fovy = math.radians(fovy_deg)
    f = 1.0 / math.tan(fovy / 2.0)
    m = np.zeros((4, 4), dtype=np.float32)
    m[0, 0] = f / aspect
    m[1, 1] = f
    m[2, 2] = (z_far + z_near) / (z_near - z_far)
    m[2, 3] = (2.0 * z_far * z_near) / (z_near - z_far)
    m[3, 2] = -1.0
    return m


def _look_at(eye: np.ndarray, center: np.ndarray, up: np.ndarray) -> np.ndarray:
    f = center - eye
    f = f / np.linalg.norm(f)
    u = up / np.linalg.norm(up)
    s = np.cross(f, u)
    s = s / np.linalg.norm(s)
    u = np.cross(s, f)

    m = np.identity(4, dtype=np.float32)
    m[0, 0:3] = s
    m[1, 0:3] = u
    m[2, 0:3] = -f
    translate = np.identity(4, dtype=np.float32)
    translate[0:3, 3] = -eye[0:3]
    return m @ translate


def _rotation_matrix_x(angle_deg: float) -> np.ndarray:
    a = math.radians(angle_deg)
    c = math.cos(a)
    s = math.sin(a)
    m = np.identity(4, dtype=np.float32)
    m[1, 1] = c
    m[1, 2] = -s
    m[2, 1] = s
    m[2, 2] = c
    return m


def _rotation_matrix_y(angle_deg: float) -> np.ndarray:
    a = math.radians(angle_deg)
    c = math.cos(a)
    s = math.sin(a)
    m = np.identity(4, dtype=np.float32)
    m[0, 0] = c
    m[0, 2] = s
    m[2, 0] = -s
    m[2, 2] = c
    return m


# =========================
# Shader compilation
# =========================


VERT_SHADER_SRC = """
#version 330 core

layout(location = 0) in vec3 in_position;
layout(location = 1) in vec3 in_normal;

uniform mat4 u_mvp;
uniform mat4 u_model;

out vec3 v_normal;
out vec3 v_world_pos;

void main()
{
    vec4 world_pos = u_model * vec4(in_position, 1.0);
    v_world_pos = world_pos.xyz;

    mat3 normal_matrix = mat3(transpose(inverse(u_model)));
    v_normal = normalize(normal_matrix * in_normal);

    gl_Position = u_mvp * vec4(in_position, 1.0);
}
"""


FRAG_SHADER_SRC = """
#version 330 core

in vec3 v_normal;
in vec3 v_world_pos;

uniform vec3 u_color;
uniform vec3 u_light_dir;
uniform vec3 u_view_pos;

out vec4 frag_color;

void main()
{
    vec3 N = normalize(v_normal);
    vec3 L = normalize(-u_light_dir);

    float diff = max(dot(N, L), 0.0);

    vec3 V = normalize(u_view_pos - v_world_pos);
    vec3 R = reflect(-L, N);
    float spec = pow(max(dot(V, R), 0.0), 32.0);

    vec3 ambient = 0.15 * u_color;
    vec3 diffuse = 0.8 * diff * u_color;
    vec3 specular = 0.7 * spec * vec3(1.0);

    vec3 color = ambient + diffuse + specular;
    frag_color = vec4(color, 1.0);
}
"""


def _compile_shader(source: str, shader_type: int) -> int:
    shader = gl.glCreateShader(shader_type)

    src_bytes = source.encode("utf-8")
    buf = create_string_buffer(src_bytes)
    char_ptr = cast(buf, POINTER(c_char))

    string_array = (POINTER(c_char) * 1)(char_ptr)
    length_array = (c_int * 1)(len(src_bytes))

    gl.glShaderSource(shader, 1, string_array, length_array)
    gl.glCompileShader(shader)

    status = c_int()
    gl.glGetShaderiv(shader, gl.GL_COMPILE_STATUS, byref(status))
    if not status.value:
        log_length = c_int()
        gl.glGetShaderiv(shader, gl.GL_INFO_LOG_LENGTH, byref(log_length))
        log = create_string_buffer(log_length.value)
        gl.glGetShaderInfoLog(shader, log_length, None, log)
        raise RuntimeError(
            f"Shader compile error ({shader_type}): {log.value.decode('utf-8')}"
        )

    return shader


def _link_program(vs: int, fs: int) -> int:
    program = gl.glCreateProgram()
    gl.glAttachShader(program, vs)
    gl.glAttachShader(program, fs)
    gl.glLinkProgram(program)

    status = c_int()
    gl.glGetProgramiv(program, gl.GL_LINK_STATUS, byref(status))
    if not status.value:
        log_length = c_int()
        gl.glGetProgramiv(program, gl.GL_INFO_LOG_LENGTH, byref(log_length))
        log = create_string_buffer(log_length.value)
        gl.glGetProgramInfoLog(program, log_length, None, log)
        raise RuntimeError(f"Program link error: {log.value.decode('utf-8')}")

    gl.glDetachShader(program, vs)
    gl.glDetachShader(program, fs)
    gl.glDeleteShader(vs)
    gl.glDeleteShader(fs)
    return program


# =========================
# GPU upload
# =========================


def _create_patch_gpu(patch: PatchCPU) -> PatchGPU:
    """Upload a CPU patch (vertices + normals) into a single interleaved VBO."""
    vertex_count = patch.vertices.shape[0]
    interleaved = np.hstack([patch.vertices, patch.normals]).astype(np.float32)
    interleaved_flat = interleaved.ravel()

    vao_id = gl.GLuint()
    vbo_id = gl.GLuint()

    gl.glGenVertexArrays(1, byref(vao_id))
    gl.glGenBuffers(1, byref(vbo_id))

    gl.glBindVertexArray(vao_id)
    gl.glBindBuffer(gl.GL_ARRAY_BUFFER, vbo_id)

    array_type = gl.GLfloat * interleaved_flat.size
    gl.glBufferData(
        gl.GL_ARRAY_BUFFER,
        interleaved_flat.nbytes,
        array_type(*interleaved_flat),
        gl.GL_STATIC_DRAW,
    )

    stride = 6 * 4

    gl.glEnableVertexAttribArray(0)
    gl.glVertexAttribPointer(
        0,
        3,
        gl.GL_FLOAT,
        gl.GL_FALSE,
        stride,
        c_void_p(0),
    )

    gl.glEnableVertexAttribArray(1)
    gl.glVertexAttribPointer(
        1,
        3,
        gl.GL_FLOAT,
        gl.GL_FALSE,
        stride,
        c_void_p(3 * 4),
    )

    gl.glBindBuffer(gl.GL_ARRAY_BUFFER, 0)
    gl.glBindVertexArray(0)

    return PatchGPU(
        vao=int(vao_id.value),
        vbo=int(vbo_id.value),
        vertex_count=vertex_count,
        color=patch.color,
    )


def _create_patches_gpu(segmented_cpu: SegmentedMeshCPU) -> List[PatchGPU]:
    return [_create_patch_gpu(p) for p in segmented_cpu.patches]


# =========================
# Viewer window
# =========================


class MeshViewerWindow(pyglet.window.Window):
    """Modern OpenGL viewer window."""

    def __init__(
        self,
        patches_cpu: SegmentedMeshCPU,
        width: int = 1280,
        height: int = 720,
        title: str = "STL Primitive Segment Viewer (Phong)",
    ) -> None:
        config = pyglet.gl.Config(
            double_buffer=True,
            depth_size=24,
            major_version=3,
            minor_version=3,
        )
        super().__init__(
            width=width,
            height=height,
            caption=title,
            config=config,
            resizable=True,
        )

        self.rot_x = 25.0
        self.rot_y = -35.0
        self.distance = 4.0
        self.aspect = width / float(max(height, 1))

        self.program = self._create_program()
        self.u_mvp_loc = gl.glGetUniformLocation(self.program, b"u_mvp")
        self.u_model_loc = gl.glGetUniformLocation(self.program, b"u_model")
        self.u_color_loc = gl.glGetUniformLocation(self.program, b"u_color")
        self.u_light_dir_loc = gl.glGetUniformLocation(self.program, b"u_light_dir")
        self.u_view_pos_loc = gl.glGetUniformLocation(self.program, b"u_view_pos")

        self.patches_gpu: List[PatchGPU] = _create_patches_gpu(patches_cpu)

        self._init_gl()
        pyglet.clock.schedule_interval(self._update, 1.0 / 60.0)

    def _create_program(self) -> int:
        vs = _compile_shader(VERT_SHADER_SRC, gl.GL_VERTEX_SHADER)
        fs = _compile_shader(FRAG_SHADER_SRC, gl.GL_FRAGMENT_SHADER)
        return _link_program(vs, fs)

    def _init_gl(self) -> None:
        gl.glEnable(gl.GL_DEPTH_TEST)
        gl.glEnable(gl.GL_CULL_FACE)
        gl.glCullFace(gl.GL_BACK)
        gl.glClearColor(0.05, 0.05, 0.08, 1.0)

    def _update(self, dt: float) -> None:
        pass

    def on_resize(self, width: int, height: int) -> None:
        super().on_resize(width, height)
        self.aspect = width / float(max(height, 1))
        gl.glViewport(0, 0, width, height)

    def on_draw(self) -> None:
        gl.glClear(gl.GL_COLOR_BUFFER_BIT | gl.GL_DEPTH_BUFFER_BIT)

        gl.glUseProgram(self.program)

        eye = np.array([0.0, 0.0, self.distance], dtype=np.float32)
        center = np.array([0.0, 0.0, 0.0], dtype=np.float32)
        up = np.array([0.0, 1.0, 0.0], dtype=np.float32)

        proj = _perspective(45.0, self.aspect, 0.1, 100.0)
        view = _look_at(eye, center, up)
        model = _rotation_matrix_x(self.rot_x) @ _rotation_matrix_y(self.rot_y)
        mvp = proj @ view @ model

        gl.glUniformMatrix4fv(
            self.u_mvp_loc,
            1,
            gl.GL_TRUE,
            (gl.GLfloat * 16)(*mvp.astype(np.float32).ravel()),
        )
        gl.glUniformMatrix4fv(
            self.u_model_loc,
            1,
            gl.GL_TRUE,
            (gl.GLfloat * 16)(*model.astype(np.float32).ravel()),
        )

        light_dir = np.array([0.4, 0.8, 1.0], dtype=np.float32)
        light_dir = light_dir / np.linalg.norm(light_dir)
        gl.glUniform3f(
            self.u_light_dir_loc,
            float(light_dir[0]),
            float(light_dir[1]),
            float(light_dir[2]),
        )
        gl.glUniform3f(
            self.u_view_pos_loc,
            float(eye[0]),
            float(eye[1]),
            float(eye[2]),
        )

        for patch in self.patches_gpu:
            r, g, b = patch.color
            gl.glUniform3f(self.u_color_loc, float(r), float(g), float(b))

            gl.glBindVertexArray(patch.vao)
            gl.glDrawArrays(gl.GL_TRIANGLES, 0, patch.vertex_count)
            gl.glBindVertexArray(0)

        gl.glUseProgram(0)

    def on_mouse_drag(
        self,
        x: int,
        y: int,
        dx: int,
        dy: int,
        buttons: int,
        modifiers: int,
    ) -> None:
        if buttons & mouse.LEFT:
            self.rot_y += dx * 0.5
            self.rot_x -= dy * 0.5
            self.rot_x = max(-89.0, min(89.0, self.rot_x))

    def on_mouse_scroll(self, x: int, y: int, scroll_x: int, scroll_y: int) -> None:
        factor = math.pow(0.9, scroll_y)
        self.distance *= factor
        self.distance = max(1.0, min(20.0, self.distance))

    def on_key_press(self, symbol, modifiers: int) -> None:
        if symbol in (key.ESCAPE, key.Q):
            pyglet.app.exit()

    def on_close(self) -> None:
        for patch in self.patches_gpu:
            vao_id = gl.GLuint(patch.vao)
            vbo_id = gl.GLuint(patch.vbo)
            gl.glDeleteVertexArrays(1, byref(vao_id))
            gl.glDeleteBuffers(1, byref(vbo_id))
        super().on_close()


# =========================
# Simple file picker window
# =========================

PICKED_STL: Optional[Path] = None


def _find_stl_files(base: Path, max_files: int = 512) -> List[Path]:
    results: List[Path] = []
    for path in base.rglob("*.stl"):
        results.append(path)
        if len(results) >= max_files:
            break
    if len(results) < max_files:
        for path in base.rglob("*.STL"):
            if path not in results:
                results.append(path)
                if len(results) >= max_files:
                    break
    results.sort()
    return results


class FilePickerWindow(pyglet.window.Window):
    """Very simple in-engine file picker for STL files.

    - Up / Down arrows: move selection
    - Enter: accept current file
    - Esc: cancel and exit
    """

    def __init__(self, files: List[Path]) -> None:
        super().__init__(
            width=900,
            height=600,
            caption="Pick STL file",
            resizable=True,
        )
        self.files = files
        self.selected = 0
        self.line_height = 24
        self.margin_x = 40
        self.margin_top = 80
        self.bg_color = (0.05, 0.05, 0.08, 1.0)
        self.title_label = pyglet.text.Label(
            "Select an STL (Up/Down, Enter, Esc)",
            x=self.width // 2,
            y=self.height - 40,
            anchor_x="center",
            anchor_y="center",
            color=(255, 255, 255, 255),
        )

    def on_resize(self, width: int, height: int) -> None:
        super().on_resize(width, height)
        self.title_label.x = width // 2
        self.title_label.y = height - 40

    def on_key_press(self, symbol, modifiers: int) -> None:
        global PICKED_STL
        if symbol == key.DOWN:
            if self.files:
                self.selected = min(self.selected + 1, len(self.files) - 1)
        elif symbol == key.UP:
            if self.files:
                self.selected = max(self.selected - 1, 0)
        elif symbol == key.ENTER:
            if self.files:
                PICKED_STL = self.files[self.selected]
                pyglet.app.exit()
        elif symbol == key.ESCAPE:
            PICKED_STL = None
            pyglet.app.exit()

    def on_draw(self) -> None:
        self.clear()
        gl.glClearColor(*self.bg_color)
        gl.glClear(gl.GL_COLOR_BUFFER_BIT)

        self.title_label.draw()

        if not self.files:
            msg = pyglet.text.Label(
                "No STL files found.",
                x=self.width // 2,
                y=self.height // 2,
                anchor_x="center",
                anchor_y="center",
                color=(255, 120, 120, 255),
            )
            msg.draw()
            return

        available_height = self.height - self.margin_top - 40
        rows = max(1, available_height // self.line_height)
        start = max(0, self.selected - rows // 2)
        end = min(len(self.files), start + rows)

        y = self.height - self.margin_top
        batch = pyglet.graphics.Batch()
        for idx in range(start, end):
            file_path = self.files[idx]
            rel = file_path.relative_to(Path.cwd())
            text = str(rel)
            if len(text) > 80:
                text = "..." + text[-77:]
            if idx == self.selected:
                color = (255, 255, 0, 255)
            else:
                color = (200, 200, 200, 255)
            pyglet.text.Label(
                text,
                x=self.margin_x,
                y=y,
                anchor_x="left",
                anchor_y="baseline",
                color=color,
                batch=batch,
            )
            y -= self.line_height

        batch.draw()


def _pick_stl_via_gui(base: Path) -> Optional[Path]:
    global PICKED_STL
    PICKED_STL = None
    files = _find_stl_files(base)
    if not files:
        print(f"[info] No STL files found under {base}")
        return None
    _ = FilePickerWindow(files)
    pyglet.app.run()
    return PICKED_STL


# =========================
# Entry point
# =========================


def main() -> None:
    if len(sys.argv) >= 2:
        stl_path = _ensure_stl_path()
    else:
        base = Path.cwd()
        stl_path = _pick_stl_via_gui(base)
        if stl_path is None:
            return

    mesh = _load_trimesh(stl_path)
    debug_path = stl_path.with_suffix(".segments.txt")
    segmented_cpu = _segment_mesh_cpu(mesh, debug_path=debug_path)
    _ = MeshViewerWindow(segmented_cpu)
    pyglet.app.run()


if __name__ == "__main__":
    main()




```


end