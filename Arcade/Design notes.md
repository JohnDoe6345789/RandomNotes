
https://chatgpt.com/share/692372d1-69d4-8011-bd7d-95d834bba6b7

10mm thick fine for buttons and good joints, thing will be tough as old boots

3d gloop

prusaslicer chop tool --> auto make joint

8" screen popular

type c to barrel cable if needed

could even have small buffer battery

example: The Countercade line (including the Street Fighter II Countercade) uses an **8" CF0380D-CPT-M6 LCD panel**.

1024x768

4:3



```



<?xml version="1.0" encoding="UTF-8"?>
<svg width="2000mm" height="1600mm" viewBox="0 0 2000 1600" xmlns="http://www.w3.org/2000/svg">
  <style>
    .panel { fill:none; stroke:black; stroke-width:1; }
    .label { font-family:Arial; font-size:16px; }
    .dim { font-family:Arial; font-size:14px; font-style:italic; }
    .title { font-family:Arial; font-size:22px; font-weight:bold; }
  </style>

  <!-- Title -->
  <text x="20" y="40" class="title">Arcade Cabinet Panel Footprints (All dimensions in millimeters)</text>

  <!-- ========================= -->
  <!--  Chungus Motherboard Tray -->
  <!-- ========================= -->
  <g id="motherboard_tray" transform="translate(20,100)">
    <rect class="panel" x="0" y="0" width="405" height="344" rx="5" ry="5" />
    <text x="10" y="20" class="label">Motherboard Tray (405 mm × 344 mm)</text>

    <!-- CPU SERVICE CUTOUT (CHUNGUS: 180 × 140 mm) -->
    <rect class="panel" x="112.5" y="102" width="180" height="140" rx="3" ry="3" />
    <text x="115" y="98" class="dim">CPU Backplate Cutout (180×140 mm)</text>

    <!-- ATX/mATX/ITX HOLE GRID (spec-accurate coordinates) -->
    <!-- Full ATX hole pattern approx 305×244 mm -->
    <g id="atx_holes">
      <!-- Spec-accurate ATX holes -->
      <circle class="panel" cx="9.525" cy="9.525" r="2.75" /> <!-- A -->
      <circle class="panel" cx="294.64" cy="9.525" r="2.75" /> <!-- C -->
      <circle class="panel" cx="294.64" cy="231.775" r="2.75" /> <!-- F -->
      <circle class="panel" cx="9.525" cy="231.775" r="2.75" /> <!-- H -->
      <circle class="panel" cx="175.26" cy="9.525" r="2.75" /> <!-- G -->
      <circle class="panel" cx="175.26" cy="231.775" r="2.75" /> <!-- L -->
      <circle class="panel" cx="175.26" cy="120.65" r="2.75" /> <!-- M -->
      <circle class="panel" cx="294.64" cy="120.65" r="2.75" /> <!-- J -->
      <circle class="panel" cx="9.525" cy="120.65" r="2.75" /> <!-- K -->
      <circle class="panel" cx="45.72" cy="9.525" r="2.75" /> <!-- B microATX -->
      <circle class="panel" cx="294.64" cy="190.5" r="2.75" /> <!-- R microATX -->
      <circle class="panel" cx="9.525" cy="190.5" r="2.75" /> <!-- S microATX -->
      <text x="10" y="330" class="dim">ATX + microATX + ITX hole pattern (spec-accurate)</text>
    </g>

    <!-- Raspberry Pi 5 mount holes (58 × 48 mm grid) -->
    <g id="pi5_holes" transform="translate(300,200)">
      <circle class="panel" cx="0" cy="0" r="1.5" />
      <circle class="panel" cx="58" cy="0" r="1.5" />
      <circle class="panel" cx="0" cy="48" r="1.5" />
      <circle class="panel" cx="58" cy="48" r="1.5" />
      <text x="0" y="70" class="dim">Raspberry Pi 5 (58×48 mm)</text>
    </g>

    <!-- Dowel Sockets (10.2 mm for tolerance) -->
    <g id="tray_dowel_sockets">
      <circle class="panel" cx="40" cy="320" r="5.1" />
      <circle class="panel" cx="365" cy="320" r="5.1" />
      <circle class="panel" cx="40" cy="20" r="5.1" />
      <circle class="panel" cx="365" cy="20" r="5.1" />
      <text x="10" y="360" class="dim">Dowel Sockets (10.2 mm)</text>
    </g>
  </g>

  <!-- =============== -->
  <!-- PSU Tray (ATX) -->
  <!-- =============== -->
  <g id="psu_tray" transform="translate(500,100)">
    <rect class="panel" x="0" y="0" width="180" height="150" rx="5" ry="5" />
    <text x="10" y="20" class="label">ATX PSU Tray (Standard)</text>

    <!-- ATX PSU hole pattern approx 81.5 × 71.5 mm -->
    <g id="psu_holes" transform="translate(40,40)">
      <circle class="panel" cx="0" cy="0" r="2" />
      <circle class="panel" cx="81.5" cy="0" r="2" />
      <circle class="panel" cx="0" cy="71.5" r="2" />
      <circle class="panel" cx="81.5" cy="71.5" r="2" />
      <text x="0" y="100" class="dim">ATX PSU holes (81.5 × 71.5 mm)</text>
    </g>

    <!-- PSU dowel sockets -->
    <circle class="panel" cx="20" cy="130" r="5.1" />
    <circle class="panel" cx="160" cy="130" r="5.1" />
    <text x="10" y="145" class="dim">PSU dowel sockets</text>

  </g>

  <!-- ==================== -->
  <!-- Bottom Panel Dowels -->
  <!-- ==================== -->
  <g id="bottom_panel_dowels" transform="translate(20,500)">
    <rect class="panel" x="0" y="0" width="620" height="540" rx="5" ry="5" />
    <text x="10" y="20" class="label">Bottom Panel with Chungus Dowels</text>

    <!-- Motherboard tray dowels -->
    <circle class="panel" cx="60" cy="500" r="5" />
    <circle class="panel" cx="540" cy="500" r="5" />
    <circle class="panel" cx="60" cy="40" r="5" />
    <circle class="panel" cx="540" cy="40" r="5" />

    <!-- PSU tray dowels -->
    <circle class="panel" cx="700" cy="100" r="5" />
    <circle class="panel" cx="700" cy="300" r="5" />

  </g>

  <!-- ==================== -->
  <!-- Assembly Instructions -->
  <!-- ==================== -->
  <g id="assembly_instructions" transform="translate(900,100)">
    <text x="0" y="0" class="title">Assembly Instructions</text>
    <text x="0" y="40" class="label">1. Insert 10 mm bottom-panel dowels into motherboard tray sockets.</text>
    <text x="0" y="70" class="label">2. Slide motherboard tray forward until dowels fully seat.</text>
    <text x="0" y="100" class="label">3. Install ATX standoffs in labeled ATX/mATX/ITX hole pattern.</text>
    <text x="0" y="130" class="label">4. Mount motherboard and tighten screws evenly.</text>
    <text x="0" y="160" class="label">5. Align PSU tray dowel sockets with bottom panel dowels.</text>
    <text x="0" y="190" class="label">6. Slide PSU tray downward until fully locked.</text>
    <text x="0" y="220" class="label">7. Install PSU screws using 81.5 × 71.5 mm pattern.</text>
    <text x="0" y="250" class="label">8. Connect motherboard 24-pin, CPU 8-pin, GPU power as needed.</text>
    <text x="0" y="280" class="label">9. Install Raspberry Pi 5 on its 58×48 mm mount if used.</text>
    <text x="0" y="310" class="label">10. Ensure CPU cooler fits within service cutout area.</text>
  </g>

  <!-- ==================== -->
  <!-- Isometric Drawing of Finished Cabinet -->
  <!-- ==================== -->
  <g id="isometric_cabinet" transform="translate(900,500)">
    <text x="0" y="0" class="title">Isometric View (Simplified Line Art)</text>

    <!-- Basic isometric cabinet outline -->
    <!-- Front face -->
    <polygon class="panel" points="0,50 200,0 350,80 150,130" />

    <!-- Side panel -->
    <polygon class="panel" points="350,80 350,260 150,310 150,130" />

    <!-- Top panel -->
    <polygon class="panel" points="200,0 350,80 350,100 200,20" />

    <!-- Control panel -->
    <polygon class="panel" points="0,50 150,130 150,150 0,70" />

    <!-- Marquee section -->
    <polygon class="panel" points="200,0 220,-60 370,20 350,80" />

    <text x="0" y="360" class="dim">Isometric cabinet outline (schematic)</text>
  </g>

  <!-- ==================== -->
  <!-- Additional Isometric Camera Angles -->
  <!-- ==================== -->
  <g id="iso_front_left" transform="translate(900,900)">
    <text x="0" y="0" class="title">Isometric Front-Left</text>
    <polygon class="panel" points="0,40 180,0 320,70 140,110" />
    <polygon class="panel" points="320,70 320,230 140,270 140,110" />
    <polygon class="panel" points="180,0 320,70 320,90 180,20" />
  </g>

  <g id="iso_front_right" transform="translate(1250,900)">
    <text x="0" y="0" class="title">Isometric Front-Right</text>
    <polygon class="panel" points="200,40 20,0 -120,70 60,110" />
    <polygon class="panel" points="-120,70 -120,230 60,270 60,110" />
    <polygon class="panel" points="20,0 -120,70 -120,90 20,20" />
  </g>

  <g id="iso_rear_left" transform="translate(900,1200)">
    <text x="0" y="0" class="title">Isometric Rear-Left</text>
    <polygon class="panel" points="0,0 200,40 200,200 0,160" />
    <polygon class="panel" points="200,40 350,10 350,170 200,200" />
    <polygon class="panel" points="0,0 200,40 350,10 150,-30" />
  </g>

  <g id="iso_rear_right" transform="translate(1250,1200)">
    <text x="0" y="0" class="title">Isometric Rear-Right</text>
    <polygon class="panel" points="0,40 -180,0 -320,70 -140,110" />
    <polygon class="panel" points="-320,70 -320,230 -140,270 -140,110" />
    <polygon class="panel" points="-180,0 -320,70 -320,90 -180,20" />
  </g>

  <!-- ==================== -->
  <!-- Top-Down Isometric -->
  <!-- ==================== -->
  <g id="iso_top_down" transform="translate(1600,500)">
    <text x="0" y="0" class="title">Isometric Top-Down</text>
    <!-- Simplified top-down isometric footprint -->
    <polygon class="panel" points="0,0 180,-60 360,0 180,60" />
    <polygon class="panel" points="0,0 0,200 180,260 180,60" />
    <polygon class="panel" points="360,0 360,200 180,260 180,60" />
    <text x="0" y="300" class="dim">Top-down isometric outline</text>
  </g>

  <!-- ===================================== -->
  <!-- Side-Cut See-Through Internal View -->
  <!-- ===================================== -->
  <g id="iso_side_cut" transform="translate(1600,900)">
    <text x="0" y="0" class="title">Side-Cut See-Through</text>

    <!-- Outer shell -->
    <polygon class="panel" points="0,0 220,-40 400,40 180,80" />
    <polygon class="panel" points="400,40 400,240 180,280 180,80" />

    <!-- Internal: Motherboard tray (ghosted) -->
    <polygon class="panel" stroke-dasharray="5,5"
      points="50,60 230,20 350,70 170,110" />
    <text x="60" y="140" class="dim">Motherboard Tray (ghosted)</text>

    <!-- Internal: PSU tray (ghosted) -->
    <polygon class="panel" stroke-dasharray="5,5"
      points="230,140 330,120 380,150 280,170" />
    <text x="240" y="200" class="dim">PSU Tray (ghosted)</text>

    <!-- CPU cutout highlight -->
    <polygon class="panel" stroke-dasharray="4,4"
      points="110,90 160,75 210,90 160,105" />
    <text x="110" y="130" class="dim">CPU Access Cutout</text>

    <!-- Rough Motherboard sketch -->
    <rect class="panel" x="90" y="40" width="160" height="100" stroke="black" stroke-width="1" fill="none" />
    <circle class="panel" cx="110" cy="60" r="8" /> <!-- CPU socket rough -->
    <rect class="panel" x="150" y="50" width="60" height="18" /> <!-- RAM stick -->
    <rect class="panel" x="150" y="75" width="60" height="18" /> <!-- RAM stick -->
    <rect class="panel" x="95" y="110" width="30" height="20" /> <!-- VRM block -->
    <text x="95" y="165" class="dim">Motherboard (rough)</text>

    <!-- Rough PSU sketch -->
    <rect class="panel" x="270" y="130" width="70" height="40" stroke="black" stroke-width="1" fill="none" />
    <circle class="panel" cx="305" cy="150" r="10" stroke-dasharray="3,3" /> <!-- Fan -->
    <text x="270" y="190" class="dim">PSU (rough)</text>

  </g>

</svg>


```


```


<?xml version="1.0" encoding="UTF-8"?>
<svg width="2200mm" height="2000mm" viewBox="0 0 2200 2000" xmlns="http://www.w3.org/2000/svg">
  <style>
    .panel { fill:none; stroke:black; stroke-width:1.5; }
    .panel-light { fill:none; stroke:black; stroke-width:0.8; }
    .dimension-line { fill:none; stroke:#666; stroke-width:0.5; stroke-dasharray:3,3; }
    .label { font-family:Arial, sans-serif; font-size:14px; fill:black; }
    .dim { font-family:Arial, sans-serif; font-size:11px; fill:#444; font-style:italic; }
    .title { font-family:Arial, sans-serif; font-size:20px; font-weight:bold; fill:black; }
    .subtitle { font-family:Arial, sans-serif; font-size:16px; font-weight:bold; fill:#333; }
    .note { font-family:Arial, sans-serif; font-size:10px; fill:#666; }
    .hole { fill:none; stroke:black; stroke-width:0.8; }
    .dowel-pin { fill:#ddd; stroke:black; stroke-width:0.8; }
    .dowel-socket { fill:#fff; stroke:black; stroke-width:0.8; }
    .cutout { fill:none; stroke:red; stroke-width:1; stroke-dasharray:2,2; }
  </style>

  <!-- Title Block -->
  <rect x="10" y="10" width="2180" height="120" fill="#f0f0f0" stroke="black" stroke-width="2"/>
  <text x="30" y="45" class="title">ARCADE CABINET PC HARDWARE MOUNTING SYSTEM</text>
  <text x="30" y="70" class="label">Technical Drawing - All dimensions in millimeters unless noted</text>
  <text x="30" y="90" class="note">Material: 3mm Aluminum Sheet (6061-T6) | Finish: Anodized Black</text>
  <text x="30" y="110" class="note">Revision: 2.0 | Date: 2024-11-24 | Drawing Scale: 1:2 (Reference)</text>
  
  <text x="1800" y="45" class="label">TOLERANCE TABLE</text>
  <text x="1800" y="65" class="note">Linear: ±0.2mm</text>
  <text x="1800" y="80" class="note">Holes: ±0.1mm</text>
  <text x="1800" y="95" class="note">Dowels: +0.0/-0.05mm</text>
  <text x="1800" y="110" class="note">Cutouts: ±0.5mm</text>

  <!-- ========================= -->
  <!--  1. MOTHERBOARD TRAY -->
  <!-- ========================= -->
  <g id="motherboard_tray" transform="translate(50,180)">
    <text x="0" y="-10" class="subtitle">1. MOTHERBOARD TRAY</text>
    
    <!-- Main tray outline -->
    <rect class="panel" x="0" y="0" width="350" height="290" rx="5" ry="5"/>
    <text x="10" y="20" class="label">Overall: 350mm × 290mm × 3mm</text>
    <text x="10" y="35" class="dim">Z-Height above bottom panel: 15mm (clearance)</text>
    
    <!-- CPU SERVICE CUTOUT (repositioned to avoid holes) -->
    <rect class="cutout" x="85" y="75" width="180" height="140" rx="3" ry="3"/>
    <text x="90" y="70" class="dim">CPU Backplate Access: 180×140mm</text>
    <text x="90" y="225" class="note">Clearance for cooler backplate installation</text>
    
    <!-- ATX HOLE PATTERN (adjusted to fit within 350×290mm tray) -->
    <!-- Origin offset to center pattern: (22.5, 22.5) for margins -->
    <g id="atx_holes" transform="translate(22.5, 22.5)">
      <!-- ATX Corners -->
      <circle class="hole" cx="9.525" cy="9.525" r="2"/>
      <circle class="hole" cx="294.64" cy="9.525" r="2"/>
      <circle class="hole" cx="294.64" cy="231.775" r="2"/>
      <circle class="hole" cx="9.525" cy="231.775" r="2"/>
      
      <!-- ATX Middle holes -->
      <circle class="hole" cx="175.26" cy="9.525" r="2"/>
      <circle class="hole" cx="175.26" cy="231.775" r="2"/>
      <circle class="hole" cx="175.26" cy="120.65" r="2"/>
      <circle class="hole" cx="294.64" cy="120.65" r="2"/>
      <circle class="hole" cx="9.525" cy="120.65" r="2"/>
      
      <!-- microATX additional holes -->
      <circle class="hole" cx="45.72" cy="9.525" r="2"/>
      <circle class="hole" cx="294.64" cy="190.5" r="2"/>
      <circle class="hole" cx="9.525" cy="190.5" r="2"/>
      
      <!-- Dimension annotations -->
      <text x="5" y="275" class="dim">ATX/mATX Pattern: 305×244mm grid</text>
      <text x="5" y="287" class="note">Holes: Ø4mm for M3 standoffs (6.35mm height)</text>
    </g>
    
    <!-- DOWEL SOCKETS (for bottom panel pins) -->
    <g id="tray_dowel_sockets">
      <circle class="dowel-socket" cx="30" cy="270" r="5.1"/>
      <circle class="dowel-socket" cx="320" cy="270" r="5.1"/>
      <circle class="dowel-socket" cx="30" cy="20" r="5.1"/>
      <circle class="dowel-socket" cx="320" cy="20" r="5.1"/>
      <text x="100" y="282" class="dim">Dowel Sockets: Ø10.2mm depth 12mm</text>
    </g>
    
    <!-- Cable routing notches -->
    <rect class="panel-light" x="340" y="130" width="10" height="30"/>
    <text x="270" y="125" class="note">Cable routing notch</text>
  </g>

  <!-- ========================= -->
  <!--  2. PSU TRAY (ATX) -->
  <!-- ========================= -->
  <g id="psu_tray" transform="translate(450,180)">
    <text x="0" y="-10" class="subtitle">2. ATX PSU TRAY</text>
    
    <!-- Main tray outline -->
    <rect class="panel" x="0" y="0" width="200" height="180" rx="5" ry="5"/>
    <text x="10" y="20" class="label">Overall: 200mm × 180mm × 3mm</text>
    <text x="10" y="35" class="dim">Z-Height above bottom panel: 15mm</text>
    
    <!-- ATX PSU hole pattern (centered) -->
    <g id="psu_holes" transform="translate(59.25, 54.25)">
      <circle class="hole" cx="0" cy="0" r="2"/>
      <circle class="hole" cx="81.5" cy="0" r="2"/>
      <circle class="hole" cx="0" cy="71.5" r="2"/>
      <circle class="hole" cx="81.5" cy="71.5" r="2"/>
      <text x="-10" y="85" class="dim">ATX PSU: 81.5×71.5mm</text>
      <text x="-10" y="97" class="note">Holes: Ø4mm for M4 screws</text>
    </g>
    
    <!-- Ventilation cutout -->
    <rect class="panel-light" x="20" y="140" width="160" height="25" rx="2" ry="2"/>
    <text x="25" y="155" class="note">Ventilation slot</text>
    
    <!-- PSU dowel sockets (4 corners for stability) -->
    <circle class="dowel-socket" cx="20" cy="160" r="5.1"/>
    <circle class="dowel-socket" cx="180" cy="160" r="5.1"/>
    <circle class="dowel-socket" cx="20" cy="20" r="5.1"/>
    <circle class="dowel-socket" cx="180" cy="20" r="5.1"/>
    <text x="40" y="172" class="dim">Dowel Sockets: Ø10.2mm × 12mm deep</text>
  </g>

  <!-- ========================= -->
  <!--  3. BOTTOM PANEL WITH DOWEL PINS -->
  <!-- ========================= -->
  <g id="bottom_panel" transform="translate(50,520)">
    <text x="0" y="-10" class="subtitle">3. BOTTOM PANEL (BASE)</text>
    
    <!-- Main panel outline -->
    <rect class="panel" x="0" y="0" width="600" height="500" rx="5" ry="5"/>
    <text x="10" y="20" class="label">Overall: 600mm × 500mm × 3mm</text>
    <text x="10" y="35" class="dim">Base panel with raised dowel pins</text>
    
    <!-- Motherboard tray dowel pins -->
    <g id="mobo_tray_dowels" transform="translate(0,0)">
      <circle class="dowel-pin" cx="80" cy="490" r="5"/>
      <circle class="dowel-pin" cx="370" cy="490" r="5"/>
      <circle class="dowel-pin" cx="80" cy="40" r="5"/>
      <circle class="dowel-pin" cx="370" cy="40" r="5"/>
      <text x="140" y="505" class="dim">Mobo Tray Pins: Ø10mm × 15mm height</text>
      
      <!-- Ghost outline of motherboard tray position -->
      <rect class="panel-light" stroke-dasharray="5,5" x="30" y="20" width="350" height="290"/>
      <text x="150" y="325" class="note">Motherboard Tray Position</text>
    </g>
    
    <!-- PSU tray dowel pins -->
    <g id="psu_tray_dowels" transform="translate(400,0)">
      <circle class="dowel-pin" cx="20" cy="180" r="5"/>
      <circle class="dowel-pin" cx="180" cy="180" r="5"/>
      <circle class="dowel-pin" cx="20" cy="20" r="5"/>
      <circle class="dowel-pin" cx="180" cy="20" r="5"/>
      <text x="40" y="195" class="dim">PSU Pins: Ø10mm × 15mm</text>
      
      <!-- Ghost outline of PSU tray position -->
      <rect class="panel-light" stroke-dasharray="5,5" x="0" y="0" width="200" height="180"/>
      <text x="50" y="100" class="note">PSU Tray Position</text>
    </g>
    
    <!-- Ventilation holes grid -->
    <g id="ventilation" transform="translate(150, 350)">
      <circle class="hole" cx="0" cy="0" r="3"/>
      <circle class="hole" cx="30" cy="0" r="3"/>
      <circle class="hole" cx="60" cy="0" r="3"/>
      <circle class="hole" cx="90" cy="0" r="3"/>
      <circle class="hole" cx="0" cy="30" r="3"/>
      <circle class="hole" cx="30" cy="30" r="3"/>
      <circle class="hole" cx="60" cy="30" r="3"/>
      <circle class="hole" cx="90" cy="30" r="3"/>
      <text x="0" y="50" class="note">Ventilation: Ø6mm holes (pattern repeats)</text>
    </g>
  </g>

  <!-- ========================= -->
  <!--  4. EXPLODED ASSEMBLY VIEW -->
  <!-- ========================= -->
  <g id="exploded_view" transform="translate(750,520)">
    <text x="0" y="-10" class="subtitle">4. EXPLODED ASSEMBLY VIEW</text>
    
    <!-- Bottom panel (base layer) -->
    <rect class="panel" x="0" y="200" width="400" height="300" rx="3" ry="3"/>
    <text x="150" y="520" class="label">Bottom Panel (Base)</text>
    <circle class="dowel-pin" cx="50" cy="250" r="4"/>
    <circle class="dowel-pin" cx="250" cy="250" r="4"/>
    
    <!-- Motherboard tray (middle layer - offset up) -->
    <rect class="panel" x="20" y="80" width="260" height="200" rx="3" ry="3"/>
    <text x="80" y="295" class="label">Motherboard Tray</text>
    <circle class="dowel-socket" cx="70" cy="260" r="4"/>
    <circle class="dowel-socket" cx="270" cy="260" r="4"/>
    
    <!-- Assembly arrows -->
    <path class="dimension-line" d="M 70,260 L 50,250" stroke-width="1.5" marker-end="url(#arrowhead)"/>
    <path class="dimension-line" d="M 270,260 L 250,250" stroke-width="1.5" marker-end="url(#arrowhead)"/>
    <text x="10" y="270" class="note">↓ 15mm clearance</text>
    
    <!-- Motherboard (top layer - offset up more) -->
    <rect class="panel-light" x="40" y="20" width="220" height="140" fill="#e8f4f8"/>
    <text x="100" y="10" class="label">Motherboard</text>
    <circle class="hole" cx="60" cy="40" r="1.5"/>
    <circle class="hole" cx="240" cy="40" r="1.5"/>
    
    <!-- PSU (side) -->
    <rect class="panel" x="300" y="240" width="80" height="120" rx="2" ry="2" fill="#f0f0f0"/>
    <text x="310" y="385" class="label">ATX PSU</text>
    <circle class="dowel-socket" cx="315" cy="345" r="3"/>
    <circle class="dowel-socket" cx="365" cy="345" r="3"/>
  </g>

  <!-- ========================= -->
  <!--  5. SIDE CUTAWAY VIEW -->
  <!-- ========================= -->
  <g id="side_cutaway" transform="translate(1250,180)">
    <text x="0" y="-10" class="subtitle">5. SIDE CUTAWAY (Section A-A)</text>
    
    <!-- Ground reference -->
    <line x1="0" y1="280" x2="500" y2="280" class="dimension-line" stroke-width="1"/>
    <text x="5" y="295" class="note">Ground Reference (Cabinet Floor)</text>
    
    <!-- Bottom panel -->
    <rect class="panel" x="50" y="277" width="400" height="3" fill="#888"/>
    <text x="230" y="295" class="dim">Bottom Panel (3mm)</text>
    
    <!-- Dowel pins -->
    <rect class="dowel-pin" x="98" y="265" width="4" height="12"/>
    <rect class="dowel-pin" x="298" y="265" width="4" height="12"/>
    <text x="105" y="263" class="dim">Ø10mm pins</text>
    
    <!-- Air gap -->
    <line x1="50" y1="265" x2="450" y2="265" class="dimension-line"/>
    <text x="455" y="268" class="dim">15mm clearance</text>
    
    <!-- Motherboard tray -->
    <rect class="panel" x="50" y="262" width="300" height="3" fill="#666"/>
    <text x="160" y="258" class="dim">Motherboard Tray</text>
    
    <!-- Standoffs -->
    <rect class="panel-light" x="98" y="255.65" width="4" height="6.35" fill="#bbb"/>
    <rect class="panel-light" x="148" y="255.65" width="4" height="6.35" fill="#bbb"/>
    <rect class="panel-light" x="198" y="255.65" width="4" height="6.35" fill="#bbb"/>
    <text x="105" y="253" class="dim">M3 Standoffs (6.35mm)</text>
    
    <!-- Motherboard -->
    <rect class="panel" x="80" y="254" width="240" height="1.6" fill="#4a90e2"/>
    <text x="160" y="250" class="label">Motherboard PCB</text>
    
    <!-- CPU & Cooler -->
    <rect class="panel-light" x="140" y="249" width="40" height="5" fill="#333"/>
    <text x="145" y="247" class="dim">CPU</text>
    <rect class="panel-light" x="125" y="220" width="70" height="29" fill="#999"/>
    <text x="135" y="238" class="label">CPU Cooler</text>
    
    <!-- CPU cutout indication -->
    <rect class="cutout" x="135" y="262" width="50" height="15"/>
    <text x="140" y="290" class="note">CPU cutout allows backplate access</text>
    
    <!-- RAM -->
    <rect class="panel-light" x="200" y="235" width="15" height="19" fill="#2d5"/>
    <rect class="panel-light" x="220" y="235" width="15" height="19" fill="#2d5"/>
    <text x="200" y="232" class="dim">RAM</text>
    
    <!-- PSU (side view) -->
    <rect class="panel" x="360" y="262" width="3" height="120" fill="#666"/>
    <rect class="panel" x="363" y="262" width="70" height="120" fill="#444"/>
    <circle class="panel-light" cx="398" cy="322" r="20" stroke-dasharray="2,2"/>
    <text x="375" y="395" class="label">ATX PSU</text>
    
    <!-- Dimensions -->
    <path class="dimension-line" d="M 45,280 L 45,265" stroke-width="0.8"/>
    <path class="dimension-line" d="M 40,280 L 40,220" stroke-width="0.8"/>
    <text x="10" y="250" class="dim">~60mm total height</text>
  </g>

  <!-- ========================= -->
  <!--  6. BILL OF MATERIALS -->
  <!-- ========================= -->
  <g id="bom" transform="translate(1250,600)">
    <text x="0" y="0" class="subtitle">6. BILL OF MATERIALS</text>
    
    <rect x="0" y="10" width="500" height="280" fill="white" stroke="black" stroke-width="1"/>
    
    <!-- Header -->
    <rect x="0" y="10" width="500" height="25" fill="#e0e0e0" stroke="black" stroke-width="1"/>
    <text x="10" y="28" class="label">QTY | PART | SPECIFICATION</text>
    
    <!-- Items -->
    <text x="10" y="53" class="dim">1 | Bottom Panel | 600×500×3mm aluminum, anodized</text>
    <text x="10" y="71" class="dim">1 | Motherboard Tray | 350×290×3mm aluminum, anodized</text>
    <text x="10" y="89" class="dim">1 | PSU Tray | 200×180×3mm aluminum, anodized</text>
    <text x="10" y="107" class="dim">8 | Dowel Pins | Ø10mm × 15mm, stainless steel, press-fit</text>
    <text x="10" y="125" class="dim">12 | ATX Standoffs | M3 × 6.35mm brass hex standoffs</text>
    <text x="10" y="143" class="dim">12 | M3 Screws | M3 × 6mm pan head, Phillips drive</text>
    <text x="10" y="161" class="dim">4 | PSU Screws | M4 × 8mm pan head, Phillips drive</text>
    <text x="10" y="179" class="dim">4 | Rubber Feet | 10mm diameter adhesive bumpers</text>
    <text x="10" y="197" class="dim">1 | Cable Ties | Assorted sizes for cable management</text>
    <text x="10" y="215" class="dim">1 | Thermal Paste | High-performance CPU thermal compound</text>
    
    <line x1="0" y1="225" x2="500" y2="225" stroke="black" stroke-width="0.5"/>
    
    <text x="10" y="245" class="note">NOTES:</text>
    <text x="10" y="260" class="note">• All fasteners: torque to 0.8-1.0 N⋅m (M3), 1.2-1.5 N⋅m (M4)</text>
    <text x="10" y="275" class="note">• Apply thread-locking compound to prevent loosening from vibration</text>
  </g>

  <!-- ========================= -->
  <!--  7. ASSEMBLY INSTRUCTIONS -->
  <!-- ========================= -->
  <g id="assembly_instructions" transform="translate(50,1100)">
    <text x="0" y="0" class="subtitle">7. ASSEMBLY INSTRUCTIONS</text>
    
    <rect x="0" y="10" width="700" height="380" fill="#fffef8" stroke="black" stroke-width="1.5"/>
    
    <text x="15" y="35" class="label">PREPARATION:</text>
    <text x="15" y="55" class="dim">• Verify all panels are deburred and clean</text>
    <text x="15" y="70" class="dim">• Press-fit 8× dowel pins into bottom panel (4 for mobo tray, 4 for PSU tray)</text>
    <text x="15" y="85" class="dim">• Install 12× M3 brass standoffs into motherboard tray per ATX/mATX pattern</text>
    
    <text x="15" y="115" class="label">STEP 1 - MOTHERBOARD INSTALLATION:</text>
    <text x="15" y="135" class="dim">1. Align motherboard tray dowel sockets with bottom panel pins</text>
    <text x="15" y="150" class="dim">2. Lower tray onto pins until fully seated (should drop 15mm)</text>
    <text x="15" y="165" class="dim">3. Place motherboard on standoffs, align I/O shield with rear cutout</text>
    <text x="15" y="180" class="dim">4. Install M3 screws in star pattern, torque to 0.8-1.0 N⋅m</text>
    <text x="15" y="195" class="dim">5. Verify no PCB flex when pressing gently on center</text>
    
    <text x="15" y="225" class="label">STEP 2 - CPU & COOLER:</text>
    <text x="15" y="245" class="dim">1. Access CPU socket from bottom via 180×140mm service cutout</text>
    <text x="15" y="260" class="dim">2. Install CPU per manufacturer instructions</text>
    <text x="15" y="275" class="dim">3. Apply thermal paste (rice grain size, center of IHS)</text>
    <text x="15" y="290" class="dim">4. Mount cooler backplate through cutout, install cooler from top</text>
    <text x="15" y="305" class="dim">5. Connect CPU fan to motherboard header</text>
    
    <text x="15" y="335" class="label">STEP 3 - PSU INSTALLATION:</text>
    <text x="15" y="355" class="dim">1. Align PSU tray dowel sockets with bottom panel pins</text>
    <text x="15" y="370" class="dim">2. Lower PSU tray onto pins until seated</text>
    
    <text x="400" y="35" class="label">STEP 3 (continued):</text>
    <text x="400" y="55" class="dim">3. Place PSU on tray, fan side toward ventilation slot</text>
    <text x="400" y="70" class="dim">4. Install 4× M4 screws, torque to 1.2-1.5 N⋅m</text>
    <text x="400" y="85" class="dim">5. Route cables through tray notch to motherboard area</text>
    
    <text x="400" y="115" class="label">STEP 4 - CABLE MANAGEMENT:</text>
    <text x="400" y="135" class="dim">1. Connect 24-pin ATX power to motherboard</text>
    <text x="400" y="150" class="dim">2. Connect 8-pin CPU power (top-left of motherboard)</text>
    <text x="400" y="165" class="dim">3. Connect GPU PCIe power if graphics card installed</text>
    <text x="400" y="180" class="dim">4. Bundle excess cables with cable ties, avoid airflow paths</text>
    <text x="400" y="195" class="dim">5. Ensure no cables contact CPU cooler fan blades</text>
    
    <text x="400" y="225" class="label">STEP 5 - FINAL CHECKS:</text>
    <text x="400" y="245" class="dim">1. Verify all power connections seated fully</text>
    <text x="400" y="260" class="dim">2. Check no loose screws or tools left in assembly</text>
    <text x="400" y="275" class="dim">3. Confirm CPU cooler makes firm contact (no wobble)</text>
    <text x="400" y="290" class="dim">4. Apply 4× rubber feet to bottom panel corners</text>
    <text x="400" y="305" class="dim">5. Test POST before installing in cabinet</text>
    
    <text x="15" y="350" class="note">⚠ WARNING: Ensure proper grounding. Connect chassis ground to PS


```

```


<?xml version="1.0" encoding="UTF-8"?>
<svg width="2000mm" height="1600mm" viewBox="0 0 2000 1600" xmlns="http://www.w3.org/2000/svg">
  <style>
    .panel { fill:none; stroke:black; stroke-width:1; }
    .label { font-family:Arial; font-size:16px; }
    .dim { font-family:Arial; font-size:14px; font-style:italic; }
    .title { font-family:Arial; font-size:22px; font-weight:bold; }
  </style>

  <!-- Title -->
  <text x="20" y="40" class="title">Arcade Cabinet Panel Footprints (All dimensions in millimeters)</text>

  <!-- ========================= -->
  <!--  Chungus Motherboard Tray -->
  <!-- ========================= -->
  <g id="motherboard_tray" transform="translate(20,100)">
    <rect class="panel" x="0" y="0" width="405" height="344" rx="5" ry="5" />
    <text x="10" y="20" class="label">Motherboard Tray (405 mm × 344 mm)</text>

    <!-- CPU SERVICE CUTOUT (CHUNGUS: 180 × 140 mm) -->
    <rect class="panel" x="112.5" y="102" width="180" height="140" rx="3" ry="3" />
    <text x="115" y="98" class="dim">CPU Backplate Cutout (180×140 mm)</text>

    <!-- ATX/mATX/ITX HOLE GRID (spec-accurate coordinates) -->
    <!-- Full ATX hole pattern approx 305×244 mm -->
    <g id="atx_holes">
      <!-- Spec-accurate ATX holes -->
      <circle class="panel" cx="9.525" cy="9.525" r="2.75" /> <!-- A -->
      <circle class="panel" cx="294.64" cy="9.525" r="2.75" /> <!-- C -->
      <circle class="panel" cx="294.64" cy="231.775" r="2.75" /> <!-- F -->
      <circle class="panel" cx="9.525" cy="231.775" r="2.75" /> <!-- H -->
      <circle class="panel" cx="175.26" cy="9.525" r="2.75" /> <!-- G -->
      <circle class="panel" cx="175.26" cy="231.775" r="2.75" /> <!-- L -->
      <circle class="panel" cx="175.26" cy="120.65" r="2.75" /> <!-- M -->
      <circle class="panel" cx="294.64" cy="120.65" r="2.75" /> <!-- J -->
      <circle class="panel" cx="9.525" cy="120.65" r="2.75" /> <!-- K -->
      <circle class="panel" cx="45.72" cy="9.525" r="2.75" /> <!-- B microATX -->
      <circle class="panel" cx="294.64" cy="190.5" r="2.75" /> <!-- R microATX -->
      <circle class="panel" cx="9.525" cy="190.5" r="2.75" /> <!-- S microATX -->
      <text x="10" y="330" class="dim">ATX + microATX hole pattern (spec-accurate)</text>
    </g>

    <!-- Raspberry Pi 5 mount holes (58 × 48 mm grid) -->
    

    <!-- Dowel Sockets (10.2 mm for tolerance) -->
    <g id="tray_dowel_sockets">
      <circle class="panel" cx="40" cy="320" r="5.1" />
      <circle class="panel" cx="365" cy="320" r="5.1" />
      <circle class="panel" cx="40" cy="20" r="5.1" />
      <circle class="panel" cx="365" cy="20" r="5.1" />
      <text x="10" y="360" class="dim">Dowel Sockets (10.2 mm)</text>
    </g>
  </g>

  <!-- =============== -->
  <!-- PSU Tray (ATX) -->
  <!-- =============== -->
  <g id="psu_tray" transform="translate(500,100)">
    <rect class="panel" x="0" y="0" width="180" height="150" rx="5" ry="5" />
    <text x="10" y="20" class="label">ATX PSU Tray (Standard)</text>

    <!-- ATX PSU hole pattern approx 81.5 × 71.5 mm -->
    <g id="psu_holes" transform="translate(40,40)">
      <circle class="panel" cx="0" cy="0" r="2" />
      <circle class="panel" cx="81.5" cy="0" r="2" />
      <circle class="panel" cx="0" cy="71.5" r="2" />
      <circle class="panel" cx="81.5" cy="71.5" r="2" />
      <text x="0" y="100" class="dim">ATX PSU holes (81.5 × 71.5 mm)</text>
    </g>

    <!-- PSU dowel sockets -->
    <circle class="panel" cx="20" cy="130" r="5.1" />
    <circle class="panel" cx="160" cy="130" r="5.1" />
    <text x="10" y="145" class="dim">PSU dowel sockets</text>

  </g>

  <!-- ==================== -->
  <!-- Bottom Panel Dowels -->
  <!-- ==================== -->
  <g id="bottom_panel_dowels" transform="translate(20,500)">
    <rect class="panel" x="0" y="0" width="620" height="540" rx="5" ry="5" />
    <text x="10" y="20" class="label">Bottom Panel with Chungus Dowels</text>

    <!-- Motherboard tray dowels -->
    <circle class="panel" cx="60" cy="500" r="5" />
    <circle class="panel" cx="540" cy="500" r="5" />
    <circle class="panel" cx="60" cy="40" r="5" />
    <circle class="panel" cx="540" cy="40" r="5" />

    <!-- PSU tray dowels -->
    <circle class="panel" cx="700" cy="100" r="5" />
    <circle class="panel" cx="700" cy="300" r="5" />

  </g>

  <!-- ==================== -->
  <!-- Assembly Instructions -->
  <!-- ==================== -->
  <g id="assembly_instructions" transform="translate(900,100)">
    <text x="0" y="0" class="title">Assembly Instructions</text>
    <text x="0" y="40" class="label">1. Insert 10 mm bottom-panel dowels into motherboard tray sockets.</text>
    <text x="0" y="70" class="label">2. Slide motherboard tray forward until dowels fully seat.</text>
    <text x="0" y="100" class="label">3. Install ATX standoffs in labeled ATX/mATX/ITX hole pattern.</text>
    <text x="0" y="130" class="label">4. Mount motherboard and tighten screws evenly.</text>
    <text x="0" y="160" class="label">5. Align PSU tray dowel sockets with bottom panel dowels.</text>
    <text x="0" y="190" class="label">6. Slide PSU tray downward until fully locked.</text>
    <text x="0" y="220" class="label">7. Install PSU screws using 81.5 × 71.5 mm pattern.</text>
    <text x="0" y="250" class="label">8. Connect motherboard 24-pin, CPU 8-pin, GPU power as needed.</text>
    <text x="0" y="280" class="label">9. Install Raspberry Pi 5 on its 58×48 mm mount if used.</text>
    <text x="0" y="310" class="label">10. Ensure CPU cooler fits within service cutout area.</text>
  </g>

  <!-- ==================== -->
  <!-- Isometric Drawing of Finished Cabinet -->
  <!-- ==================== -->
  <g id="isometric_cabinet" transform="translate(900,500)">
    <text x="0" y="0" class="title">Isometric View (Simplified Line Art)</text>

    <!-- Basic isometric cabinet outline -->
    <!-- Front face -->
    <polygon class="panel" points="0,50 200,0 350,80 150,130" />

    <!-- Side panel -->
    <polygon class="panel" points="350,80 350,260 150,310 150,130" />

    <!-- Top panel -->
    <polygon class="panel" points="200,0 350,80 350,100 200,20" />

    <!-- Control panel -->
    <polygon class="panel" points="0,50 150,130 150,150 0,70" />

    <!-- Marquee section -->
    <polygon class="panel" points="200,0 220,-60 370,20 350,80" />

    <text x="0" y="360" class="dim">Isometric cabinet outline (schematic)</text>
  </g>

  <!-- ==================== -->
  <!-- Additional Isometric Camera Angles -->
  <!-- ==================== -->
  <g id="iso_front_left" transform="translate(900,900)">
    <text x="0" y="0" class="title">Isometric Front-Left</text>
    <polygon class="panel" points="0,40 180,0 320,70 140,110" />
    <polygon class="panel" points="320,70 320,230 140,270 140,110" />
    <polygon class="panel" points="180,0 320,70 320,90 180,20" />
  </g>

  <g id="iso_front_right" transform="translate(1250,900)">
    <text x="0" y="0" class="title">Isometric Front-Right</text>
    <polygon class="panel" points="200,40 20,0 -120,70 60,110" />
    <polygon class="panel" points="-120,70 -120,230 60,270 60,110" />
    <polygon class="panel" points="20,0 -120,70 -120,90 20,20" />
  </g>

  <g id="iso_rear_left" transform="translate(900,1200)">
    <text x="0" y="0" class="title">Isometric Rear-Left</text>
    <polygon class="panel" points="0,0 200,40 200,200 0,160" />
    <polygon class="panel" points="200,40 350,10 350,170 200,200" />
    <polygon class="panel" points="0,0 200,40 350,10 150,-30" />
  </g>

  <g id="iso_rear_right" transform="translate(1250,1200)">
    <text x="0" y="0" class="title">Isometric Rear-Right</text>
    <polygon class="panel" points="0,40 -180,0 -320,70 -140,110" />
    <polygon class="panel" points="-320,70 -320,230 -140,270 -140,110" />
    <polygon class="panel" points="-180,0 -320,70 -320,90 -180,20" />
  </g>

  <!-- ==================== -->
  <!-- Top-Down Isometric -->
  <!-- ==================== -->
  <g id="iso_top_down" transform="translate(1600,500)">
    <text x="0" y="0" class="title">Isometric Top-Down</text>
    <!-- Simplified top-down isometric footprint -->
    <polygon class="panel" points="0,0 180,-60 360,0 180,60" />
    <polygon class="panel" points="0,0 0,200 180,260 180,60" />
    <polygon class="panel" points="360,0 360,200 180,260 180,60" />
    <text x="0" y="300" class="dim">Top-down isometric outline</text>
  </g>

  <!-- ===================================== -->
  <!-- Side-Cut See-Through Internal View -->
  <!-- ===================================== -->
  <g id="iso_side_cut" transform="translate(1600,900)">
    <text x="0" y="0" class="title">Side-Cut See-Through</text>

    <!-- Outer shell -->
    <polygon class="panel" points="0,0 220,-40 400,40 180,80" />
    <polygon class="panel" points="400,40 400,240 180,280 180,80" />

    <!-- Internal: Motherboard tray (ghosted) -->
    <polygon class="panel" stroke-dasharray="5,5"
      points="50,60 230,20 350,70 170,110" />
    <text x="60" y="140" class="dim">Motherboard Tray (ghosted)</text>

    <!-- Internal: PSU tray (ghosted) -->
    <polygon class="panel" stroke-dasharray="5,5"
      points="230,140 330,120 380,150 280,170" />
    <text x="240" y="200" class="dim">PSU Tray (ghosted)</text>

    <!-- CPU cutout highlight -->
    <polygon class="panel" stroke-dasharray="4,4"
      points="110,90 160,75 210,90 160,105" />
    <text x="110" y="130" class="dim">CPU Access Cutout</text>

    <!-- Rough Motherboard sketch -->
    <rect class="panel" x="90" y="40" width="160" height="100" stroke="black" stroke-width="1" fill="none" />
    <circle class="panel" cx="110" cy="60" r="8" /> <!-- CPU socket rough -->
    <rect class="panel" x="150" y="50" width="60" height="18" /> <!-- RAM stick -->
    <rect class="panel" x="150" y="75" width="60" height="18" /> <!-- RAM stick -->
    <rect class="panel" x="95" y="110" width="30" height="20" /> <!-- VRM block -->
    <text x="95" y="165" class="dim">Motherboard (rough)</text>

    <!-- Rough PSU sketch -->
    <rect class="panel" x="270" y="130" width="70" height="40" stroke="black" stroke-width="1" fill="none" />
    <circle class="panel" cx="305" cy="150" r="10" stroke-dasharray="3,3" /> <!-- Fan -->
    <text x="270" y="190" class="dim">PSU (rough)</text>

  </g>

  <!-- ==================== -->
  <!-- Two-Player Control Panel (Classic Layout, 30 mm SANWA) -->
  <!-- ==================== -->
  <g id="control_panel" transform="translate(20,1100)">
    <rect class="panel" x="0" y="0" width="620" height="200" rx="5" ry="5" />
    <text x="10" y="20" class="label">Two-Player Control Panel (Classic 2P, 30 mm)</text>
    <text x="10" y="40" class="dim">Nominal width 620 mm, supports two players with 8 buttons each</text>

    <!-- Player 1 joystick (left) -->
    <circle class="panel" cx="170" cy="110" r="12" />
    <text x="150" y="145" class="dim">P1 Stick</text>

    <!-- Player 1 buttons (8 × 30 mm, classic stagger) -->
    <!-- Top row -->
    <circle class="panel" cx="230" cy="80" r="15" />
    <circle class="panel" cx="265" cy="80" r="15" />
    <circle class="panel" cx="300" cy="80" r="15" />
    <circle class="panel" cx="335" cy="80" r="15" />
    <!-- Bottom row (staggered) -->
    <circle class="panel" cx="240" cy="120" r="15" />
    <circle class="panel" cx="275" cy="120" r="15" />
    <circle class="panel" cx="310" cy="120" r="15" />
    <circle class="panel" cx="345" cy="120" r="15" />

    <!-- Player 2 joystick (right) -->
    <circle class="panel" cx="450" cy="110" r="12" />
    <text x="430" y="145" class="dim">P2 Stick</text>

    <!-- Player 2 buttons (mirrored) -->
    <!-- Top row -->
    <circle class="panel" cx="390" cy="80" r="15" />
    <circle class="panel" cx="425" cy="80" r="15" />
    <circle class="panel" cx="460" cy="80" r="15" />
    <circle class="panel" cx="495" cy="80" r="15" />
    <!-- Bottom row (staggered) -->
    <circle class="panel" cx="400" cy="120" r="15" />
    <circle class="panel" cx="435" cy="120" r="15" />
    <circle class="panel" cx="470" cy="120" r="15" />
    <circle class="panel" cx="505" cy="120" r="15" />

    <!-- Admin buttons (24–30 mm) centered above -->
    <circle class="panel" cx="310" cy="40" r="12" />
    <circle class="panel" cx="340" cy="40" r="12" />
    <circle class="panel" cx="370" cy="40" r="12" />
    <text x="260" y="30" class="dim">Start / Select / Menu</text>
  </g>

  <!-- ==================== -->
  <!-- Raspberry Pi 5 Side Module -->
  <!-- ==================== -->
  <g id="pi5_side_module" transform="translate(700,1100)">
    <rect class="panel" x="0" y="0" width="140" height="100" rx="5" ry="5" />
    <text x="10" y="20" class="label">Raspberry Pi 5 Side Module</text>
    <text x="10" y="40" class="dim">58 × 48 mm mount, 2.9 mm holes</text>
    <g transform="translate(20,50)">
      <circle class="panel" cx="0" cy="0" r="1.45" />
      <circle class="panel" cx="58" cy="0" r="1.45" />
      <circle class="panel" cx="0" cy="48" r="1.45" />
      <circle class="panel" cx="58" cy="48" r="1.45" />
    </g>
  </g>

  <!-- ==================== -->
  <!-- Monitor Bezel Panel (22–24 inch, Landscape) -->
  <!-- ==================== -->
  <g id="monitor_bezel_panel" transform="translate(20,1350)">
    <!-- Front bezel panel for monitor -->
    <rect class="panel" x="0" y="0" width="620" height="420" rx="5" ry="5" />
    <text x="10" y="20" class="label">Monitor Bezel Panel (22–24" LCD, landscape)</text>
    <text x="10" y="40" class="dim">Outer panel: 620 mm (W) × 420 mm (H)</text>

    <!-- Inner viewing window (for 24" class, with bezel margin) -->
    <!-- Window sized to ~560 × 320 mm, centered -->
    <rect class="panel" x="30" y="50" width="560" height="320" rx="3" ry="3" />
    <text x="30" y="390" class="dim">Viewing window: 560 mm (W) × 320 mm (H)</text>
    <text x="30" y="410" class="dim">Suitable for most 22–24" 16:9 monitors (hidden bezel)</text>
  </g>

  <!-- ==================== -->
  <!-- VESA Mount Plate (75 × 75 and 100 × 100) -->
  <!-- ==================== -->
  <g id="vesa_mount_plate" transform="translate(700,1350)">
    <rect class="panel" x="0" y="0" width="300" height="300" rx="5" ry="5" />
    <text x="10" y="20" class="label">VESA Mount Plate</text>
    <text x="10" y="40" class="dim">Supports 75×75 mm and 100×100 mm patterns</text>

    <!-- Center reference cross -->
    <line class="panel" x1="150" y1="20" x2="150" y2="280" />
    <line class="panel" x1="20" y1="150" x2="280" y2="150" />
    <text x="155" y="150" class="dim">Panel center</text>

    <!-- 100 × 100 mm VESA holes (M4 clearance) -->
    <!-- Centered on plate: offsets ±50 mm from center -->
    <circle class="panel" cx="100" cy="100" r="3" />
    <circle class="panel" cx="200" cy="100" r="3" />
    <circle class="panel" cx="100" cy="200" r="3" />
    <circle class="panel" cx="200" cy="200" r="3" />
    <text x="90" y="95" class="dim">100 × 100 mm</text>

    <!-- 75 × 75 mm VESA holes (M4 clearance) -->
    <!-- Centered on same origin: offsets ±37.5 mm -->
    <circle class="panel" cx="112.5" cy="112.5" r="3" />
    <circle class="panel" cx="187.5" cy="112.5" r="3" />
    <circle class="panel" cx="112.5" cy="187.5" r="3" />
    <circle class="panel" cx="187.5" cy="187.5" r="3" />
    <text x="105" y="210" class="dim">75 × 75 mm</text>
  </g>

</svg>



```

![[arcade_semantic_accessible.zip]]


![[arcade_semantic_descriptive.zip]]



end