<!DOCTYPE html>  
  
<html lang="en">  
<head>  
<meta charset="UTF-8">  
<meta name="viewport" content="width=device-width, initial-scale=1.0">  
<title>AgriVision — 3D Assembly</title>  
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@700;800&display=swap" rel="stylesheet">  
<style>  
*{margin:0;padding:0;box-sizing:border-box;}  
body{background:#f5f7f5;font-family:'Space Mono',monospace;color:#1a2e1a;overflow:hidden;width:100vw;height:100vh;}  
  
#canvas-container{position:fixed;inset:0;}  
  
/* HUD overlay */  
#hud{position:fixed;inset:0;pointer-events:none;z-index:10;}  
  
.hud-corner{position:absolute;width:40px;height:40px;opacity:0.4;}  
.hud-tl{top:16px;left:16px;border-top:1px solid #00a855;border-left:1px solid #00a855;}  
.hud-tr{top:16px;right:16px;border-top:1px solid #00a855;border-right:1px solid #00a855;}  
.hud-bl{bottom:16px;left:16px;border-bottom:1px solid #00a855;border-left:1px solid #00a855;}  
.hud-br{bottom:16px;right:16px;border-bottom:1px solid #00a855;border-right:1px solid #00a855;}  
  
#title-area{position:absolute;top:28px;left:50%;transform:translateX(-50%);text-align:center;}  
.main-title{font-family:‘Syne’,sans-serif;font-size:28px;font-weight:800;color:#111;letter-spacing:4px;}  
.main-title span{color:#00a855;}  
.sub-title{font-size:9px;letter-spacing:4px;color:#7a9a7a;margin-top:4px;}  
  
#component-label{position:absolute;bottom:80px;left:50%;transform:translateX(-50%);text-align:center;transition:all 0.4s;}  
.comp-name{font-family:‘Syne’,sans-serif;font-size:22px;font-weight:800;color:#111;letter-spacing:2px;}  
.comp-desc{font-size:10px;letter-spacing:2px;color:#5a7a5a;margin-top:6px;max-width:500px;}  
.comp-tag{display:inline-block;font-size:9px;letter-spacing:3px;border:1px solid #00a855;color:#00a855;padding:3px 12px;margin-top:8px;}  
  
#progress-bar{position:absolute;bottom:52px;left:50%;transform:translateX(-50%);width:400px;}  
.prog-track{height:2px;background:#ccdacc;position:relative;}  
.prog-fill{height:100%;background:#00a855;transition:width 0.6s ease;width:0%;}  
.prog-dots{display:flex;justify-content:space-between;margin-top:8px;}  
.prog-dot{width:6px;height:6px;border-radius:50%;background:#dde8dd;border:1px solid #aaccaa;transition:all 0.3s;}  
.prog-dot.active{background:#00e87a;border-color:#00a855;box-shadow:0 0 6px rgba(0,168,85,0.4);}  
.prog-dot.done{background:#00a855;border-color:#007a3d;}  
  
#controls{position:absolute;bottom:16px;left:50%;transform:translateX(-50%);display:flex;gap:10px;pointer-events:all;}  
.btn{font-family:‘Space Mono’,monospace;font-size:9px;letter-spacing:2px;padding:8px 18px;border:1px solid;cursor:pointer;text-transform:uppercase;background:transparent;transition:all 0.2s;}  
.btn-g{border-color:#00a855;color:#00a855;}  
.btn-g:hover{background:rgba(0,168,85,0.08);}  
.btn-m{border-color:#ccdacc;color:#5a7a5a;}  
.btn-m:hover{border-color:#7a9a7a;color:#1a2e1a;}  
  
#stats-left{position:absolute;left:24px;top:50%;transform:translateY(-50%);display:flex;flex-direction:column;gap:12px;}  
#stats-right{position:absolute;right:24px;top:50%;transform:translateY(-50%);display:flex;flex-direction:column;gap:12px;text-align:right;}  
.stat-block{font-size:9px;letter-spacing:2px;}  
.stat-val{font-family:‘Syne’,sans-serif;font-size:18px;font-weight:800;color:#00a855;}  
.stat-key{color:#7a9a7a;margin-top:2px;}  
  
#status-chip{position:absolute;top:28px;right:80px;font-size:9px;letter-spacing:2px;color:#00a855;border:1px solid #00a855;padding:4px 12px;display:flex;align-items:center;gap:6px;background:rgba(255,255,255,0.7);}  
.blink-dot{width:5px;height:5px;border-radius:50%;background:#00a855;animation:blink 1.2s infinite;}  
@keyframes blink{0%,100%{opacity:1;}50%{opacity:0.2;}}  
  
/* Scan line */  
#scanline{position:absolute;left:0;right:0;height:1px;background:linear-gradient(90deg,transparent,rgba(0,168,85,0.2),transparent);animation:scanMove 4s linear infinite;pointer-events:none;}  
@keyframes scanMove{0%{top:-2px;}100%{top:100vh;}}  
  
/* Assembly complete flash */  
#complete-flash{position:fixed;inset:0;background:rgba(0,168,85,0.06);opacity:0;pointer-events:none;z-index:20;transition:opacity 0.5s;}  
  
/* Grid bg */  
#grid-bg{position:fixed;inset:0;background-image:linear-gradient(rgba(0,100,50,0.05) 1px,transparent 1px),linear-gradient(90deg,rgba(0,100,50,0.05) 1px,transparent 1px);background-size:40px 40px;pointer-events:none;}  
  
/* Explode labels */  
.explode-label{position:absolute;pointer-events:none;transform:translate(-50%,-50%);text-align:center;opacity:0;transition:opacity 0.4s;}  
.explode-label.visible{opacity:1;}  
.el-name{font-family:‘Syne’,sans-serif;font-size:11px;font-weight:700;color:#111;letter-spacing:1px;background:rgba(255,255,255,0.92);border:1px solid #ccdacc;padding:4px 10px;white-space:nowrap;box-shadow:0 2px 8px rgba(0,0,0,0.1);}  
.el-tag{font-size:8px;letter-spacing:1px;color:#00a855;margin-top:3px;background:rgba(255,255,255,0.85);padding:2px 8px;white-space:nowrap;}  
</style>  
  
</head>  
<body>  
  
<div id="grid-bg"></div>  
<div id="canvas-container"><canvas id="c"></canvas></div>  
<div id="complete-flash"></div>  
  
<div id="explode-labels" style="position:fixed;inset:0;pointer-events:none;z-index:15;"></div>  
  
<div id="hud">  
  <div class="hud-corner hud-tl"></div>  
  <div class="hud-corner hud-tr"></div>  
  <div class="hud-corner hud-bl"></div>  
  <div class="hud-corner hud-br"></div>  
  <div id="scanline"></div>  
  
  <div id="title-area">  
    <div class="main-title">Agri<span>Vision</span></div>  
    <div class="sub-title">3D HARDWARE ASSEMBLY · PCIT AI SUMMIT</div>  
  </div>  
  
  <div id="status-chip"><div class="blink-dot"></div><span id="status-text">INITIALIZING</span></div>  
  
  <div id="stats-left">  
    <div class="stat-block"><div class="stat-val">8</div><div class="stat-key">COMPONENTS</div></div>  
    <div class="stat-block"><div class="stat-val" style="color:#42c8f5;">8–10h</div><div class="stat-key">BATTERY LIFE</div></div>  
  </div>  
  
  <div id="stats-right">  
    <div class="stat-block"><div class="stat-val" style="color:#ff8c42;">2.4</div><div class="stat-key">ACRES MAPPED</div></div>  
    <div class="stat-block"><div class="stat-val">±1.2m</div><div class="stat-key">GPS ACCURACY</div></div>  
    <div class="stat-block"><div class="stat-val" style="color:#c084fc;">94%</div><div class="stat-key">AI CONFIDENCE</div></div>  
  </div>  
  
  <div id="component-label">  
    <div class="comp-name" id="lbl-name">AGRIVISION DEVICE</div>  
    <div class="comp-desc" id="lbl-desc">Watch all 8 hardware components assemble into one smart portable farm intelligence unit</div>  
    <div class="comp-tag" id="lbl-tag">SMART FARM DEVICE</div>  
  </div>  
  
  <div id="progress-bar">  
    <div class="prog-track"><div class="prog-fill" id="prog-fill"></div></div>  
    <div class="prog-dots" id="prog-dots"></div>  
  </div>  
  
  <div id="controls">  
    <button class="btn btn-g" onclick="playAssembly()">▶ PLAY ASSEMBLY</button>  
    <button class="btn btn-m" onclick="toggleAutoRotate()">⟳ AUTO ROTATE</button>  
    <button class="btn btn-m" onclick="explodeView()">⊞ EXPLODE VIEW</button>  
    <button class="btn btn-m" onclick="resetView()">↺ RESET</button>  
  </div>  
</div>  
  
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>  
  
<script>  
// ── SCENE SETUP ──────────────────────────────────────────────  
const canvas = document.getElementById('c');  
const renderer = new THREE.WebGLRenderer({canvas, antialias:true, alpha:true});  
renderer.setPixelRatio(Math.min(devicePixelRatio,2));  
renderer.setSize(innerWidth, innerHeight);  
renderer.shadowMap.enabled = true;  
renderer.shadowMap.type = THREE.PCFSoftShadowMap;  
renderer.toneMapping = THREE.ACESFilmicToneMapping;  
renderer.toneMappingExposure = 1.1;  
  
const scene = new THREE.Scene();  
scene.background = new THREE.Color(0xf0f5f0);  
scene.fog = new THREE.Fog(0xf0f5f0, 18, 40);  
  
const camera = new THREE.PerspectiveCamera(45, innerWidth/innerHeight, 0.1, 100);  
camera.position.set(0, 4, 12);  
camera.lookAt(0, 0, 0);  
  
window.addEventListener('resize',()=>{  
  camera.aspect = innerWidth/innerHeight;  
  camera.updateProjectionMatrix();  
  renderer.setSize(innerWidth,innerHeight);  
});  
  
// ── LIGHTING ──────────────────────────────────────────────────  
const ambient = new THREE.AmbientLight(0xffffff, 1.8);  
scene.add(ambient);  
  
const keyLight = new THREE.DirectionalLight(0x88ffcc, 1.0);  
keyLight.position.set(5, 10, 7);  
keyLight.castShadow = true;  
scene.add(keyLight);  
  
const fillLight = new THREE.DirectionalLight(0xaaddff, 0.5);  
fillLight.position.set(-6, 4, -3);  
scene.add(fillLight);  
  
const rimLight = new THREE.DirectionalLight(0xffffff, 0.6);  
rimLight.position.set(0, -5, -8);  
scene.add(rimLight);  
  
// Point lights for glow  
const glow1 = new THREE.PointLight(0x00e87a, 1.5, 8);  
glow1.position.set(0, 2, 2);  
scene.add(glow1);  
  
const glow2 = new THREE.PointLight(0x42c8f5, 0.8, 6);  
glow2.position.set(-3, 1, 0);  
scene.add(glow2);  
  
// ── MATERIALS ─────────────────────────────────────────────────  
function mat(color, opts={}) {  
  return new THREE.MeshStandardMaterial({  
    color: new THREE.Color(color),  
    metalness: opts.metalness ?? 0.3,  
    roughness: opts.roughness ?? 0.5,  
    emissive: new THREE.Color(opts.emissive ?? color),  
    emissiveIntensity: opts.emissiveIntensity ?? 0.08,  
    ...opts  
  });  
}  
  
const mats = {  
  pcb:      mat(0x1a3d1a, {metalness:0.2, roughness:0.8, emissiveIntensity:0.05}),  
  pcbGreen: mat(0x0d2b0d, {metalness:0.1, roughness:0.9}),  
  chip:     mat(0x111111, {metalness:0.8, roughness:0.2, emissiveIntensity:0.0}),  
  green:    mat(0x00e87a, {metalness:0.1, roughness:0.4, emissiveIntensity:0.3}),  
  yellow:   mat(0xf5e642, {metalness:0.1, roughness:0.5, emissiveIntensity:0.25}),  
  blue:     mat(0x42c8f5, {metalness:0.1, roughness:0.4, emissiveIntensity:0.3}),  
  orange:   mat(0xff8c42, {metalness:0.1, roughness:0.5, emissiveIntensity:0.2}),  
  purple:   mat(0xc084fc, {metalness:0.1, roughness:0.4, emissiveIntensity:0.25}),  
  silver:   mat(0xaaaaaa, {metalness:0.9, roughness:0.15, emissiveIntensity:0.0}),  
  black:    mat(0x111111, {metalness:0.5, roughness:0.3}),  
  white:    mat(0xdddddd, {metalness:0.1, roughness:0.6}),  
  case_:    mat(0x1a2a1a, {metalness:0.4, roughness:0.6, emissiveIntensity:0.04}),  
  lens:     mat(0x001122, {metalness:0.1, roughness:0.05, emissiveIntensity:0.0}),  
  screen:   mat(0x001a10, {metalness:0.0, roughness:0.1, emissiveIntensity:0.5, emissive:0x00e87a}),  
};  
  
// ── HELPERS ───────────────────────────────────────────────────  
function box(w,h,d,m){  
  const g = new THREE.BoxGeometry(w,h,d);  
  const mesh = new THREE.Mesh(g,m);  
  mesh.castShadow=true; mesh.receiveShadow=true;  
  return mesh;  
}  
function cyl(rt,rb,h,seg,m){  
  const g = new THREE.CylinderGeometry(rt,rb,h,seg);  
  const mesh = new THREE.Mesh(g,m);  
  mesh.castShadow=true;  
  return mesh;  
}  
function sphere(r,m){  
  const mesh = new THREE.Mesh(new THREE.SphereGeometry(r,16,16),m);  
  mesh.castShadow=true;  
  return mesh;  
}  
function addPin(parent, x, z, color='silver'){  
  const pin = cyl(0.015,0.015,0.18,6, mats[color]||mats.silver);  
  pin.position.set(x, -0.09, z);  
  parent.add(pin);  
}  
function group(...children){  
  const g = new THREE.Group();  
  children.forEach(c=>g.add(c));  
  return g;  
}  
  
// ── BUILD COMPONENTS ──────────────────────────────────────────  
  
// 1. RASPBERRY PI 4B — main board  
function buildRaspberryPi() {  
  const g = new THREE.Group();  
  // Main PCB  
  const pcb = box(2.2, 0.08, 1.45, mats.pcb);  
  g.add(pcb);  
  // CPU chip  
  const cpu = box(0.4,0.06,0.4, mats.chip); cpu.position.set(-0.2,0.07,0.1); g.add(cpu);  
  // RAM  
  const ram = box(0.35,0.05,0.15, mats.chip); ram.position.set(0.3,0.065,0.1); g.add(ram);  
  // USB ports  
  for(let i=0;i<2;i++){  
    const usb = box(0.18,0.1,0.1,mats.silver);  
    usb.position.set(0.95,0.05,-0.35+i*0.22); g.add(usb);  
  }  
  // Ethernet  
  const eth = box(0.22,0.12,0.18,mats.silver); eth.position.set(0.95,0.06,0.25); g.add(eth);  
  // GPIO pins row  
  for(let i=0;i<10;i++){  
    const pin = box(0.04,0.12,0.04,mats.yellow); pin.position.set(-0.85+i*0.14,0.1,-0.62); g.add(pin);  
  }  
  for(let i=0;i<10;i++){  
    const pin = box(0.04,0.12,0.04,mats.yellow); pin.position.set(-0.85+i*0.14,0.1,-0.5); g.add(pin);  
  }  
  // Green LED  
  const led = sphere(0.03, mats.green); led.position.set(-0.9,0.08,0.55); g.add(led);  
  // Screen connector  
  const conn = box(0.3,0.04,0.1,mats.silver); conn.position.set(-0.7,0.06,-0.5); g.add(conn);  
  // Power LED  
  const pled = sphere(0.03, mats.yellow); pled.position.set(-0.8,0.08,0.55); g.add(pled);  
  // Wifi chip  
  const wifi = box(0.2,0.04,0.15,mats.silver); wifi.position.set(0.0,0.065,-0.4); g.add(wifi);  
  return g;  
}  
  
// 2. CAMERA MODULE 3  
function buildCamera() {  
  const g = new THREE.Group();  
  // PCB  
  const pcb = box(0.6,0.05,0.6, mats.pcb); g.add(pcb);  
  // Camera housing  
  const housing = box(0.35,0.12,0.35, mats.black); housing.position.set(0,0.085,0); g.add(housing);  
  // Lens outer  
  const lensOut = cyl(0.14,0.14,0.06,24, mats.black); lensOut.position.set(0,0.15,0); g.add(lensOut);  
  // Lens inner  
  const lensIn = cyl(0.1,0.1,0.04,24, mats.lens); lensIn.position.set(0,0.17,0); g.add(lensIn);  
  // Lens glass highlight  
  const lensGlass = cyl(0.06,0.06,0.01,24, new THREE.MeshStandardMaterial({color:0x0033ff,metalness:0,roughness:0,emissive:0x0033ff,emissiveIntensity:0.4,transparent:true,opacity:0.7}));  
  lensGlass.position.set(0,0.18,0); g.add(lensGlass);  
  // FPC ribbon connector  
  const ribbon = box(0.08,0.02,0.3, mats.orange); ribbon.position.set(-0.3,0.01,0); g.add(ribbon);  
  // Corner screws  
  [[.22,.22],[-.22,.22],[.22,-.22],[-.22,-.22]].forEach(([x,z])=>{  
    const sc = cyl(0.025,0.025,0.04,8,mats.silver); sc.position.set(x,0.045,z); g.add(sc);  
  });  
  return g;  
}  
  
// 3. NEO-6M GPS  
function buildGPS() {  
  const g = new THREE.Group();  
  const pcb = box(0.8,0.05,0.8, mats.pcb); g.add(pcb);  
  // Main chip  
  const chip = box(0.3,0.05,0.3, mats.chip); chip.position.set(0,0.05,0); g.add(chip);  
  // Patch antenna  
  const ant = box(0.4,0.06,0.4, mat(0xffffff,{metalness:0.4,roughness:0.3}));  
  ant.position.set(0,0.08,0); g.add(ant);  
  // Antenna trace cross  
  const h = box(0.35,0.005,0.04, mat(0xcccc00,{emissiveIntensity:0.4})); h.position.set(0,0.115,0); g.add(h);  
  const v = box(0.04,0.005,0.35, mat(0xcccc00,{emissiveIntensity:0.4})); v.position.set(0,0.115,0); g.add(v);  
  // pins  
  for(let i=0;i<6;i++) addPin(g, -0.4+i*0.12, 0.35);  
  // LED  
  const led = sphere(0.025, mats.yellow); led.position.set(0.32,0.06,0.32); g.add(led);  
  return g;  
}  
  
// 4. DHT22 TEMP/HUMIDITY  
function buildDHT22() {  
  const g = new THREE.Group();  
  // Body  
  const body = box(0.24,0.5,0.18, mats.white); g.add(body);  
  // Sensor mesh face  
  const face = box(0.16,0.18,0.02, mat(0x888888,{metalness:0.4,roughness:0.7}));  
  face.position.set(0,0.1,0.1); g.add(face);  
  // Three pins down  
  for(let i=0;i<3;i++){  
    const pin = cyl(0.018,0.018,0.35,6,mats.silver);  
    pin.position.set(-0.06+i*0.06,-0.42,0); g.add(pin);  
  }  
  // Orange accent stripe  
  const stripe = box(0.25,0.04,0.19, mats.orange); stripe.position.set(0,0.25,0); g.add(stripe);  
  return g;  
}  
  
// 5. CAPACITIVE MOISTURE SENSOR  
function buildMoisture() {  
  const g = new THREE.Group();  
  // Long probe PCB  
  const probe = box(0.18,1.0,0.04, mats.pcb); g.add(probe);  
  // Copper pads (capacitive)  
  for(let i=0;i<8;i++){  
    const pad = box(0.06,0.06,0.02, mat(0xb87333,{metalness:0.8,roughness:0.2}));  
    pad.position.set(-0.04,-0.2+i*0.1,0.03); g.add(pad);  
  }  
  for(let i=0;i<8;i++){  
    const pad = box(0.06,0.06,0.02, mat(0xb87333,{metalness:0.8,roughness:0.2}));  
    pad.position.set(0.04,-0.2+i*0.1,0.03); g.add(pad);  
  }  
  // Head connector  
  const head = box(0.25,0.2,0.1, mats.pcb); head.position.set(0,0.6,0); g.add(head);  
  // Connector pins  
  for(let i=0;i<3;i++) addPin(g,-0.06+i*0.06, -0.505);  
  // LED  
  const led = sphere(0.025, mats.blue); led.position.set(0.07,0.55,0.06); g.add(led);  
  // Tip point  
  const tip = new THREE.Mesh(new THREE.ConeGeometry(0.04,0.1,8),mats.pcb);  
  tip.position.set(0,-0.55,0); tip.rotation.z=Math.PI; g.add(tip);  
  return g;  
}  
  
// 6. pH SENSOR  
function buildPH() {  
  const g = new THREE.Group();  
  // Glass body  
  const body = cyl(0.07,0.07,0.8,16, new THREE.MeshStandardMaterial({color:0x88aaff,metalness:0,roughness:0.05,transparent:true,opacity:0.5,emissive:0x1122ff,emissiveIntensity:0.1}));  
  g.add(body);  
  // Metal body upper  
  const metal = cyl(0.08,0.08,0.3,16,mats.silver); metal.position.set(0,0.55,0); g.add(metal);  
  // Reference junction ring  
  const ring = cyl(0.09,0.09,0.04,16, mat(0xffff00,{emissiveIntensity:0.3})); ring.position.set(0,0.3,0); g.add(ring);  
  // Glass tip  
  const tip = new THREE.Mesh(new THREE.SphereGeometry(0.07,12,12,0,Math.PI*2,0,Math.PI/2),new THREE.MeshStandardMaterial({color:0x88aaff,transparent:true,opacity:0.4}));  
  tip.position.set(0,-0.4,0); tip.rotation.x=Math.PI; g.add(tip);  
  // Cable  
  const cable = cyl(0.025,0.025,0.2,8,mats.black); cable.position.set(0,0.8,0); g.add(cable);  
  return g;  
}  
  
// 7. ADS1115 ADC MODULE  
function buildADS1115() {  
  const g = new THREE.Group();  
  const pcb = box(0.5,0.05,0.5, mats.pcb); g.add(pcb);  
  // Main ADS chip  
  const chip = box(0.2,0.04,0.2, mats.chip); chip.position.set(0,0.045,0); g.add(chip);  
  // Decoupling caps  
  [[.15,.15],[-.15,.15],[.15,-.15]].forEach(([x,z])=>{  
    const cap = cyl(0.04,0.04,0.06,8, mats.silver); cap.position.set(x,0.06,z); g.add(cap);  
  });  
  // I2C pullup resistors (small blobs)  
  for(let i=0;i<4;i++){  
    const r = box(0.04,0.03,0.08, mat(0x111111,{metalness:0.2})); r.position.set(-0.2+i*0.12,0.04,-0.18); g.add(r);  
  }  
  // Header pins  
  for(let i=0;i<4;i++) addPin(g,-0.18+i*0.12, 0.22);  
  for(let i=0;i<4;i++) addPin(g,-0.18+i*0.12, -0.22);  
  // Purple accent LED  
  const led = sphere(0.025, mats.purple); led.position.set(0.2,0.06,0.18); g.add(led);  
  return g;  
}  
  
// 8. POWER BANK 20000mAh  
function buildPowerBank() {  
  const g = new THREE.Group();  
  // Main body  
  const body = box(2.0,0.45,0.9, mats.case_);  
  body.castShadow=true; g.add(body);  
  // Rounded edges via extra boxes  
  const top = box(1.98,0.06,0.88, mat(0x111111,{metalness:0.6,roughness:0.3}));  
  top.position.set(0,0.255,0); g.add(top);  
  // LED indicators  
  const ledColors = [mats.green, mats.green, mats.green, mat(0x333333,{})];  
  ledColors.forEach((m,i)=>{  
    const led = cyl(0.03,0.03,0.02,8,m); led.rotation.x=Math.PI/2;  
    led.position.set(-0.5+i*0.2,0.23,-0.46); g.add(led);  
  });  
  // USB-A port  
  const usb = box(0.22,0.1,0.05, mats.silver); usb.position.set(0.7,0,0.47); g.add(usb);  
  // USB-C port  
  const usbc = box(0.18,0.07,0.05, mats.silver); usbc.position.set(0.4,0,0.47); g.add(usbc);  
  // Button  
  const btn = cyl(0.06,0.06,0.02,12, mats.silver); btn.rotation.x=Math.PI/2;  
  btn.position.set(-0.7,0,0.47); g.add(btn);  
  // Lightning bolt logo  
  const bolt = box(0.02,0.25,0.02, mat(0xf5e642,{emissiveIntensity:0.6}));  
  bolt.position.set(-0.85,0.1,-0.1); g.add(bolt);  
  // Capacity label  
  const cap = box(0.5,0.04,0.04, mat(0xf5e642,{emissiveIntensity:0.4}));  
  cap.position.set(0,0.0,-0.1); g.add(cap);  
  return g;  
}  
  
// ── ENCLOSURE BOX (assembled) ─────────────────────────────────  
function buildEnclosure() {  
  const g = new THREE.Group();  
  // Base  
  const base = box(3.2,0.1,2.2, mats.case_); base.position.set(0,-0.8,0); g.add(base);  
  // Walls  
  const wallF = box(3.2,1.6,0.08, mats.case_); wallF.position.set(0,0,1.1); g.add(wallF);  
  const wallB = box(3.2,1.6,0.08, mats.case_); wallB.position.set(0,0,-1.1); g.add(wallB);  
  const wallL = box(0.08,1.6,2.2, mats.case_); wallL.position.set(-1.6,0,0); g.add(wallL);  
  const wallR = box(0.08,1.6,2.2, mats.case_); wallR.position.set(1.6,0,0); g.add(wallR);  
  // Green accent trim  
  const trim = box(3.2,0.04,2.2, mat(0x00e87a,{emissiveIntensity:0.6})); trim.position.set(0,0.82,0); g.add(trim);  
  // Weatherproof seal line  
  const seal = box(3.18,0.02,2.18, mat(0xff8c42,{emissiveIntensity:0.2})); seal.position.set(0,0.5,0); g.add(seal);  
  // Clip/mount on side  
  const clip = box(0.15,0.4,0.3, mats.silver); clip.position.set(-1.72,0.2,0.3); g.add(clip);  
  const clipHole = box(0.16,0.25,0.12, mats.black); clipHole.position.set(-1.72,0.2,0.3); g.add(clipHole);  
  return g;  
}  
  
// ── COMPONENT DEFINITIONS ─────────────────────────────────────  
const componentDefs = [  
  {  
    name:'RASPBERRY PI 4B',  
    desc:'The brain — runs all Python, AI models, sensor reading, GPS & WiFi',  
    tag:'PROCESSOR · PYTHON · AI RUNTIME',  
    color:'#00e87a',  
    build: buildRaspberryPi,  
    finalPos: new THREE.Vector3(0, 0.1, 0),  
    finalRot: new THREE.Euler(0, 0, 0),  
    startPos: new THREE.Vector3(0, 6, 0),  
  },  
  {  
    name:'CAMERA MODULE 3',  
    desc:'Photographs soil for MobileNetV2 AI — classifies type in 2 seconds',  
    tag:'VISION AI · MOBILENETV2 · SOIL DETECTION',  
    color:'#42c8f5',  
    build: buildCamera,  
    finalPos: new THREE.Vector3(-0.9, 0.35, -0.5),  
    finalRot: new THREE.Euler(0, 0, 0),  
    startPos: new THREE.Vector3(-6, 4, 0),  
  },  
  {  
    name:'NEO-6M GPS MODULE',  
    desc:'Locks 5 satellites · maps farm boundary · tracks farmer live on 5m grid',  
    tag:'GPS · GEOJSON · ±1.2m ACCURACY',  
    color:'#f5e642',  
    build: buildGPS,  
    finalPos: new THREE.Vector3(0.8, 0.35, -0.5),  
    finalRot: new THREE.Euler(0, 0, 0),  
    startPos: new THREE.Vector3(6, 4, 0),  
  },  
  {  
    name:'DHT22 SENSOR',  
    desc:'Measures air temp & humidity — feeds crop recommendation engine',  
    tag:'CLIMATE · TEMPERATURE · HUMIDITY',  
    color:'#ff8c42',  
    build: buildDHT22,  
    finalPos: new THREE.Vector3(-1.1, 0.2, 0.3),  
    finalRot: new THREE.Euler(0, 0, 0),  
    startPos: new THREE.Vector3(-4, 6, 3),  
  },  
  {  
    name:'CAPACITIVE MOISTURE SENSOR',  
    desc:'Inserted in soil — measures water content % per zone in real time',  
    tag:'HYDRATION · SOIL WATER · IRRIGATION ALERTS',  
    color:'#42c8f5',  
    build: buildMoisture,  
    finalPos: new THREE.Vector3(1.1, 0.15, 0.3),  
    finalRot: new THREE.Euler(0, 0, 0),  
    startPos: new THREE.Vector3(4, 6, 3),  
  },  
  {  
    name:'ANALOG pH SENSOR',  
    desc:'Reads soil acidity 0–14 · flags lime or sulfur needed per zone',  
    tag:'pH SCALE · SOIL CHEMISTRY · CROP OPTIMIZATION',  
    color:'#f5e642',  
    build: buildPH,  
    finalPos: new THREE.Vector3(0.0, 0.15, 0.7),  
    finalRot: new THREE.Euler(Math.PI/2, 0, 0),  
    startPos: new THREE.Vector3(0, 8, 6),  
  },  
  {  
    name:'ADS1115 ADC MODULE',  
    desc:'Converts analog pH & moisture signals to digital for Raspberry Pi',  
    tag:'16-BIT ADC · I2C · ANALOG→DIGITAL',  
    color:'#c084fc',  
    build: buildADS1115,  
    finalPos: new THREE.Vector3(-0.5, 0.35, 0.5),  
    finalRot: new THREE.Euler(0, 0, 0),  
    startPos: new THREE.Vector3(-5, 5, 5),  
  },  
  {  
    name:'20,000mAh POWER BANK',  
    desc:'8–10 hours field operation · fully portable · no socket needed',  
    tag:'POWER · 8–10HR LIFE · PORTABLE',  
    color:'#f5e642',  
    build: buildPowerBank,  
    finalPos: new THREE.Vector3(0, -0.45, 0),  
    finalRot: new THREE.Euler(0, Math.PI/2, 0),  
    startPos: new THREE.Vector3(0, -8, 0),  
  },  
];  
  
// ── ASSEMBLE SCENE ────────────────────────────────────────────  
const componentMeshes = [];  
const enclosure = buildEnclosure();  
enclosure.visible = false;  
scene.add(enclosure);  
  
componentDefs.forEach((def, i) => {  
  const mesh = def.build();  
  mesh.position.copy(def.startPos);  
  mesh.rotation.copy(def.finalRot);  
  mesh.userData = { def, idx: i, assembled: false };  
  scene.add(mesh);  
  componentMeshes.push(mesh);  
});  
  
// ── PROGRESS DOTS ─────────────────────────────────────────────  
const dotsEl = document.getElementById('prog-dots');  
componentDefs.forEach((_,i)=>{  
  const d = document.createElement('div');  
  d.className='prog-dot';  
  d.id='dot-'+i;  
  dotsEl.appendChild(d);  
});  
  
// ── FLOATING PARTICLES ────────────────────────────────────────  
const particleGeo = new THREE.BufferGeometry();  
const pCount = 120;  
const pPos = new Float32Array(pCount*3);  
for(let i=0;i<pCount;i++){  
  pPos[i*3]=(Math.random()-0.5)*20;  
  pPos[i*3+1]=(Math.random()-0.5)*12;  
  pPos[i*3+2]=(Math.random()-0.5)*20;  
}  
particleGeo.setAttribute('position',new THREE.BufferAttribute(pPos,3));  
const particleMat = new THREE.PointsMaterial({color:0x00a855,size:0.05,transparent:true,opacity:0.25});  
const particles = new THREE.Points(particleGeo,particleMat);  
scene.add(particles);  
  
// ── CIRCUIT LINES (connection wires) ──────────────────────────  
const wireGroup = new THREE.Group();  
scene.add(wireGroup);  
function drawWire(from, to, color=0x00e87a){  
  const points=[from,to];  
  const geo = new THREE.BufferGeometry().setFromPoints(points);  
  const mat2 = new THREE.LineBasicMaterial({color,transparent:true,opacity:0});  
  const line = new THREE.Line(geo,mat2);  
  wireGroup.add(line);  
  return mat2;  
}  
const wireMats = [];  
componentDefs.forEach((def,i)=>{  
  if(i===0) return;  
  const wm = drawWire(def.finalPos, componentDefs[0].finalPos,  
    [0x00e87a,0x42c8f5,0xf5e642,0xff8c42,0x42c8f5,0xf5e642,0xc084fc,0xf5e642][i]);  
  wireMats.push(wm);  
});  
  
// ── GROUND REFLECTION ─────────────────────────────────────────  
const groundGeo = new THREE.PlaneGeometry(30,30);  
const groundMat = new THREE.MeshStandardMaterial({color:0xe8f0e8,roughness:1,metalness:0});  
const ground = new THREE.Mesh(groundGeo,groundMat);  
ground.rotation.x=-Math.PI/2; ground.position.y=-1.5;  
ground.receiveShadow=true; scene.add(ground);  
  
// ── GRID HELPER ───────────────────────────────────────────────  
const grid = new THREE.GridHelper(20,40,0xccddcc,0xccddcc);  
grid.position.y=-1.49; scene.add(grid);  
  
// ── CAMERA MOUSE DRAG ─────────────────────────────────────────  
let isDragging=false, lastMX=0, lastMY=0;  
let camTheta=0.3, camPhi=0.4, camRadius=12;  
let targetTheta=0.3, targetPhi=0.4;  
  
canvas.addEventListener('mousedown',e=>{isDragging=true;lastMX=e.clientX;lastMY=e.clientY;});  
canvas.addEventListener('mouseup',()=>isDragging=false);  
canvas.addEventListener('mousemove',e=>{  
  if(!isDragging)return;  
  targetTheta-=(e.clientX-lastMX)*0.005;  
  targetPhi=Math.max(0.1,Math.min(1.4,targetPhi-(e.clientY-lastMY)*0.005));  
  lastMX=e.clientX;lastMY=e.clientY;  
});  
canvas.addEventListener('wheel',e=>{  
  camRadius=Math.max(4,Math.min(20,camRadius+e.deltaY*0.01));  
});  
// Touch  
canvas.addEventListener('touchstart',e=>{isDragging=true;lastMX=e.touches[0].clientX;lastMY=e.touches[0].clientY;});  
canvas.addEventListener('touchend',()=>isDragging=false);  
canvas.addEventListener('touchmove',e=>{  
  if(!isDragging)return;  
  targetTheta-=(e.touches[0].clientX-lastMX)*0.005;  
  targetPhi=Math.max(0.1,Math.min(1.4,targetPhi-(e.touches[0].clientY-lastMY)*0.005));  
  lastMX=e.touches[0].clientX;lastMY=e.touches[0].clientY;  
  e.preventDefault();  
},{passive:false});  
  
// ── ANIMATION STATE ───────────────────────────────────────────  
let autoRotate = true;  
let currentAssembling = -1;  
let assemblyPlaying = false;  
let isExploded = false;  
const explodedPositions = componentDefs.map((def,i)=>{  
  const angle = (i/componentDefs.length)*Math.PI*2;  
  const r = 3.5;  
  return new THREE.Vector3(Math.cos(angle)*r, 1+Math.sin(i)*0.5, Math.sin(angle)*r);  
});  
  
// ── LERP HELPERS ──────────────────────────────────────────────  
function lerpV3(a,b,t){return a.clone().lerp(b,t);}  
  
// ── ASSEMBLY SEQUENCE ─────────────────────────────────────────  
function playAssembly(){  
  if(assemblyPlaying)return;  
  assemblyPlaying=true;  
  currentAssembling=0;  
  isExploded=false;  
  enclosure.visible=false;  
  wireMats.forEach(m=>m.opacity=0);  
  // Reset all  
  componentMeshes.forEach((m,i)=>{  
    m.position.copy(componentDefs[i].startPos);  
    m.userData.assembled=false;  
    document.getElementById('dot-'+i)?.classList.remove('active','done');  
  });  
  document.getElementById('prog-fill').style.width='0%';  
  setLabel(0);  
  assembleNext();  
}  
  
function assembleNext(){  
  if(currentAssembling>=componentDefs.length){  
    // All assembled — show enclosure  
    setTimeout(()=>{  
      enclosure.visible=true;  
      enclosure.scale.set(0,0,0);  
      wireMats.forEach(m=>m.opacity=0.5);  
      setStatusText('ASSEMBLY COMPLETE ✓');  
      setLabel(-1);  
      document.getElementById('complete-flash').style.opacity='1';  
      setTimeout(()=>document.getElementById('complete-flash').style.opacity='0',600);  
    },400);  
    return;  
  }  
  const mesh = componentMeshes[currentAssembling];  
  const def = componentDefs[currentAssembling];  
  setLabel(currentAssembling);  
  setStatusText('INSTALLING: '+def.name);  
  document.getElementById('dot-'+currentAssembling)?.classList.add('active');  
  
  let t=0;  
  const startPos = mesh.position.clone();  
  const endPos = def.finalPos.clone();  
  
  const animInterval = setInterval(()=>{  
    t=Math.min(1,t+0.025);  
    const ease = 1-Math.pow(1-t,3); // ease out cubic  
    mesh.position.lerpVectors(startPos,endPos,ease);  
    if(t>=1){  
      clearInterval(animInterval);  
      mesh.userData.assembled=true;  
      document.getElementById('dot-'+currentAssembling)?.classList.remove('active');  
      document.getElementById('dot-'+currentAssembling)?.classList.add('done');  
      document.getElementById('prog-fill').style.width=((currentAssembling+1)/componentDefs.length*100)+'%';  
      currentAssembling++;  
      setTimeout(assembleNext, 500);  
    }  
  },16);  
}  
  
function setLabel(idx){  
  const name = document.getElementById('lbl-name');  
  const desc = document.getElementById('lbl-desc');  
  const tag = document.getElementById('lbl-tag');  
  if(idx===-1){  
    name.textContent='AGRIVISION DEVICE — ASSEMBLED';  
    desc.textContent='All 8 components installed. Smart farm intelligence unit ready for field deployment.';  
    tag.textContent='ASSEMBLY COMPLETE · READY TO DEPLOY';  
    tag.style.color='#00e87a';  
  } else if(idx<componentDefs.length){  
    const d=componentDefs[idx];  
    name.textContent=d.name;  
    desc.textContent=d.desc;  
    tag.textContent=d.tag;  
    tag.style.color=d.color;  
    tag.style.borderColor=d.color;  
  }  
}  
  
function setStatusText(t){document.getElementById('status-text').textContent=t;}  
  
function toggleAutoRotate(){autoRotate=!autoRotate;}  
  
// ── EXPLODE LABELS ────────────────────────────────────────────  
const labelsContainer = document.getElementById('explode-labels');  
const explodeLabelEls = [];  
  
componentDefs.forEach((def,i)=>{  
  const wrapper = document.createElement('div');  
  wrapper.className = 'explode-label';  
  wrapper.id = 'elabel-'+i;  
  wrapper.innerHTML = `<div class="el-name">${def.name}</div><div class="el-tag">${def.tag.split('·')[0].trim()}</div>`;  
  labelsContainer.appendChild(wrapper);  
  explodeLabelEls.push(wrapper);  
});  
  
function updateExplodeLabels(){  
  if(!isExploded){ explodeLabelEls.forEach(el=>el.classList.remove('visible')); return; }  
  const w=innerWidth, h=innerHeight;  
  componentMeshes.forEach((mesh,i)=>{  
    const pos = mesh.position.clone();  
    pos.project(camera);  
    const x = (pos.x*0.5+0.5)*w;  
    const y = (-pos.y*0.5+0.5)*h - 60;  
    explodeLabelEls[i].style.left = x+'px';  
    explodeLabelEls[i].style.top = y+'px';  
    explodeLabelEls[i].classList.add('visible');  
    // Set tag color to match component  
    explodeLabelEls[i].querySelector('.el-tag').style.color = componentDefs[i].color;  
    explodeLabelEls[i].querySelector('.el-name').style.borderColor = componentDefs[i].color;  
  });  
}  
  
function explodeView(){  
  isExploded=!isExploded;  
  if(isExploded){  
    componentMeshes.forEach((m,i)=>{ m.userData.targetPos=explodedPositions[i].clone(); });  
    enclosure.visible=false;  
    wireMats.forEach(wm=>wm.opacity=0);  
    setStatusText('EXPLODED VIEW');  
    setLabel(-1);  
    document.getElementById('lbl-name').textContent='EXPLODED VIEW — ALL COMPONENTS';  
    document.getElementById('lbl-desc').textContent='All 8 modules separated for inspection. Drag to rotate, scroll to zoom.';  
  } else {  
    explodeLabelEls.forEach(el=>el.classList.remove('visible'));  
    componentMeshes.forEach((m,i)=>{ m.userData.targetPos=componentDefs[i].finalPos.clone(); });  
    if(currentAssembling>=componentDefs.length) {enclosure.visible=true; wireMats.forEach(wm=>wm.opacity=0.5);}  
    setStatusText('ASSEMBLED VIEW');  
  }  
}  
  
function resetView(){  
  assemblyPlaying=false;  
  isExploded=false;  
  currentAssembling=-1;  
  enclosure.visible=false;  
  wireMats.forEach(m=>m.opacity=0);  
  explodeLabelEls.forEach(el=>el.classList.remove('visible'));  
  componentMeshes.forEach((m,i)=>{  
    m.position.copy(componentDefs[i].startPos);  
    m.userData.assembled=false;  
    m.userData.targetPos=null;  
    document.getElementById('dot-'+i)?.classList.remove('active','done');  
  });  
  document.getElementById('prog-fill').style.width='0%';  
  setLabel(-1);  
  document.getElementById('lbl-name').textContent='AGRIVISION DEVICE';  
  document.getElementById('lbl-desc').textContent='Watch all 8 hardware components assemble into one smart portable farm intelligence unit';  
  document.getElementById('lbl-tag').textContent='SMART FARM DEVICE';  
  setStatusText('READY');  
}  
  
// ── RENDER LOOP ───────────────────────────────────────────────  
let frameCount=0;  
function animate(){  
  requestAnimationFrame(animate);  
  frameCount++;  
  
  // Smooth camera  
  camTheta+=(targetTheta-camTheta)*0.05;  
  camPhi+=(targetPhi-camPhi)*0.05;  
  if(autoRotate) targetTheta+=0.003;  
  
  camera.position.x=Math.sin(camTheta)*Math.cos(camPhi)*camRadius;  
  camera.position.y=Math.sin(camPhi)*camRadius;  
  camera.position.z=Math.cos(camTheta)*Math.cos(camPhi)*camRadius;  
  camera.lookAt(0,0,0);  
  
  // Smooth component lerp for explode/assemble  
  componentMeshes.forEach((m,i)=>{  
    if(m.userData.targetPos){  
      m.position.lerp(m.userData.targetPos,0.06);  
    }  
  });  
  
  // Enclosure scale in  
  if(enclosure.visible){  
    enclosure.scale.lerp(new THREE.Vector3(1,1,1),0.08);  
  }  
  
  // Particles drift  
  const pos=particles.geometry.attributes.position;  
  for(let i=0;i<pCount;i++){  
    pos.array[i*3+1]+=0.005;  
    if(pos.array[i*3+1]>6) pos.array[i*3+1]=-6;  
  }  
  pos.needsUpdate=true;  
  
  // Glow pulse  
  glow1.intensity=1.2+Math.sin(frameCount*0.04)*0.4;  
  glow2.intensity=0.7+Math.cos(frameCount*0.03)*0.3;  
  
  // Assembled component gentle float  
  componentMeshes.forEach((m,i)=>{  
    if(m.userData.assembled && !isExploded && !m.userData.targetPos){  
      m.position.y=componentDefs[i].finalPos.y+Math.sin(frameCount*0.02+i)*0.015;  
    }  
  });  
  
  // Update explode labels  
  updateExplodeLabels();  
  
  renderer.render(scene,camera);  
}  
animate();  
  
// Auto-start after short delay  
setTimeout(()=>{  
  setStatusText('READY — PRESS PLAY');  
}, 500);  
</script>  
  
</body>  
</html>  
