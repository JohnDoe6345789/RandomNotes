# 3D CAD File Formats

## 1. Neutral and Interchange Formats

### STEP (.step, .stp)

* ISO 10303 standard for product data.
* Highly interoperable across CAD systems.
* Supports assemblies, geometry, PMI, and metadata.
* Used for long-term archival and data exchange.

### IGES (.iges, .igs)

* Legacy neutral format for wireframes, surfaces, and solids.
* Wide support but often produces non-manifold or repair-prone geometry.

### Parasolid XT (.x_t, .x_b)

* Native kernel format for Siemens Parasolid.
* Accurate B-Rep solids with excellent interoperability for Parasolid-based CAD.

### ACIS SAT (.sat)

* Native kernel format for Spatial ACIS.
* B-Rep solids, feature-level data sometimes preserved.

### JT (.jt)

* Lightweight visualization and CAD-rich format.
* Widely used in PLM pipelines.

## 2. Native CAD Formats

### SolidWorks (.sldprt, .sldasm)

* Feature trees, parametrics, mates, configurations.
* Proprietary and version-sensitive.

### Autodesk Inventor (.ipt, .iam)

* Parametric modeling files with full feature history.

### PTC Creo (.prt, .asm)

* Parametric solids with regeneration logic.

### Siemens NX (.prt)

* Feature-based CAD with variations per version.

### Catia V5 (.catpart, .catproduct)

* Enterprise-grade; strong assembly, surfacing, and aerospace usage.

### Fusion 360 (.f3d, .f3z)

* Cloud-native archive containing full design history.

### FreeCAD (.FCStd)

* Zip container storing B-Rep, parametric tree, and metadata.

## 3. Polygon and Mesh Formats

### STL (.stl)

* Triangle mesh without units or color.
* Ubiquitous for 3D printing.

### OBJ (.obj)

* Vertices, faces, UVs, normals.
* Common in graphics and light CAD.

### PLY (.ply)

* Supports attributes like color, transparency, normals.
* Used for scanning and point cloud workflows.

### 3MF (.3mf)

* Modern 3D printing format; zipped XML with colors, materials, textures.
* More compact than STL.

### AMF (.amf)

* XML-based additive manufacturing format; superseded by 3MF.

## 4. Parametric and CSG Formats

### OpenSCAD (.scad)

* Script-based constructive solid geometry.

### SDF (.sdf)

* Signed distance field description.
* Powerful for implicit modeling and generative workflows.

### CSG (.csg)

* Boolean tree representation.

## 5. Scene and Assembly Exchange Formats

### GLTF / GLB (.gltf, .glb)

* Graphics-centric; can store meshes, materials, nodes.
* Increasing use for lightweight engineering review.

### USD / USDZ (.usd, .usda, .usdc, .usdz)

* Pixar’s scene graph format.
* Strong for assemblies, materials, and collaborative pipelines.

### FBX (.fbx)

* Autodesk-backed animation and mesh format.
* Generally not used for engineering solids.

## 6. Point Cloud and Scan Formats

### LAS / LAZ (.las, .laz)

* LIDAR point cloud standard.

### E57 (.e57)

* Vendor-neutral 3D scan format including imagery.

### PTS / XYZ (.pts, .xyz)

* Raw space-delimited point sets.

## 7. Manufacturing and CAM Formats

### DXF (.dxf)

* 2D and 3D geometry interchange.

### DWG (.dwg)

* Native AutoCAD format.

### G-code (.gcode)

* Toolpath commands for CNC and 3D printers.

## 8. Visualization and Lightweight Formats

### U3D (.u3d)

* 3D PDF embedding.

### PRC (.prc)

* Lightweight embedding format used in PDFs.

## 9. Simulation and Engineering Formats

### Nastran (.nas, .bdf)

* FE mesh and analysis definitions.

### Abaqus (.inp)

* FE models and materials.

### ANSYS (.cdb)

* FE geometry and mesh data.

## 10. Packaging and Container Formats

### ZIP-based containers

* FCStd, 3MF, USDZ.
* Enable structured metadata, thumbnails, materials, scene graphs.

## 11. Emerging and Experimental Formats

### STEP-NC (.p21 with AP238)

* Integrates geometric design with manufacturing features.

### MPF / GMP (Machine Process Files)

* Next-gen CNC process descriptors.
