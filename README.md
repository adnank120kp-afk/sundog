
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>SUNDOG — endless dusk flight</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Unbounded:wght@500;900&display=swap" rel="stylesheet">
<style>
  :root{
    --cream:#f4e7d0; --ink:#140b06; --ember:#ff8c42; --ember2:#ffb35c;
    --mono:'Space Mono',monospace; --disp:'Unbounded',sans-serif;
  }
  *{margin:0;padding:0;box-sizing:border-box}
  html,body{height:100%;overflow:hidden;background:var(--ink)}
  body{font-family:var(--mono);color:var(--cream);user-select:none;-webkit-user-select:none;touch-action:none}
  #c{position:fixed;inset:0;width:100%;height:100%;display:block;cursor:crosshair}
  body[data-state="PLAYING"]{cursor:none}
#score{font-size:clamp(30px,5vw,46px);font-weight:700;line-height:1.05;letter-spacing:.02em;text-shadow:0 2px 14px rgba(0,0,0,.55)}
  #hud-best{font-size:11px;letter-spacing:.2em;opacity:.55;margin-top:2px}
  #hud-right{position:absolute;top:24px;right:28px;text-align:right}
  .ro{display:flex;gap:10px;justify-content:flex-end;align-items:baseline;margin-bottom:3px}
  .ro .v{font-size:19px;font-weight:700;min-width:52px}
  .hud-btns{margin-top:12px;display:flex;gap:8px;justify-content:flex-end;pointer-events:auto}
  .icobtn{width:34px;height:34px;display:grid;place-items:center;background:rgba(20,11,6,.55);border:1px solid rgba(244,231,208,.25);cursor:pointer;color:var(--cream);transition:border-color .15s,color .15s}
  .icobtn:hover{border-color:var(--ember);color:var(--ember)}
  .icobtn svg{width:16px;height:16px}
  .icobtn .off{display:none}
  .icobtn.alt .on{display:none}
  .icobtn.alt .off{display:block}
  #hint{position:absolute;bottom:20px;left:50%;transform:translateX(-50%);font-size:11px;letter-spacing:.22em;opacity:.45;white-space:nowrap}

  /* ---------- overlays ---------- */
  .overlay{position:fixed;inset:0;z-index:10;display:flex;flex-direction:column;align-items:center;justify-content:center;text-align:center;padding:20px;background:rgba(12,6,3,.38);backdrop-filter:blur(2px);transition:opacity .45s,visibility .45s}
  .overlay.hidden{opacity:0;visibility:hidden;pointer-events:none}
  .overline{font-size:11px;letter-spacing:.5em;color:var(--ember2);margin-bottom:18px}
  h1{font-family:var(--disp);font-weight:900;font-size:clamp(44px,9vw,104px);letter-spacing:.03em;display:flex;align-items:center;gap:.26em;line-height:1;color:var(--cream)}
  h1 .dog{width:.16em;height:.16em;border-radius:50%;background:var(--ember);box-shadow:0 0 26px rgba(255,140,66,.7)}
  .rule{width:56px;height:2px;background:var(--ember);margin:26px 0 18px}
  .tagline{font-size:13px;letter-spacing:.14em;opacity:.88;max-width:560px;line-height:1.7}
  .controls{display:flex;gap:24px;margin-top:28px;font-size:11px;letter-spacing:.18em;opacity:.85;flex-wrap:wrap;justify-content:center}
  .controls b{border:1px solid rgba(244,231,208,.35);padding:3px 9px;margin-right:8px;font-weight:400}
  .btn{pointer-events:auto;margin-top:36px;font-family:var(--mono);font-weight:700;font-size:13px;letter-spacing:.3em;padding:16px 42px 16px 46px;background:var(--cream);color:var(--ink);border:0;cursor:pointer;transition:background .15s}
  .btn:hover{background:var(--ember)}
  .subhint{margin-top:14px;font-size:10px;letter-spacing:.25em;opacity:.45}
  .bestline{margin-top:26px;font-size:12px;letter-spacing:.25em;color:var(--ember2)}
  #cause{margin-top:4px;font-size:13px;letter-spacing:.24em;color:var(--ember2)}
  .stats{display:flex;margin-top:6px;flex-wrap:wrap;justify-content:center}
  .stat{padding:4px 26px;border-left:1px solid rgba(244,231,208,.18)}
  .stat:first-child{border-left:0}
  .stat .hud-label{margin-bottom:7px}
  .stat .v{font-size:clamp(22px,4vw,30px);font-weight:700}
  #newbest{display:none;margin-top:18px;font-size:11px;letter-spacing:.44em;color:var(--ember)}
  #newbest.show{display:block}

  /* ---------- floating score popups ---------- */
  .pop{position:fixed;z-index:5;transform:translate(-50%,-50%);font-weight:700;font-size:15px;letter-spacing:.12em;color:var(--ember2);text-shadow:0 2px 10px rgba(0,0,0,.7);pointer-events:none;white-space:nowrap;animation:rise .95s ease-out forwards}
  @keyframes rise{
    0%{opacity:0;transform:translate(-50%,-40%) scale(.6)}
    18%{opacity:1;transform:translate(-50%,-75%) scale(1.12)}
    100%{opacity:0;transform:translate(-50%,-200%) scale(1)}
  }
</style>
<script type="importmap">
{ "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js" } }
</script>
</head>
<body data-state="MENU">

<canvas id="c"></canvas>
<div id="vignette"></div>
<div id="pulse"></div>

<div id="hud" class="hidden">
  <div id="hud-left">
    <div class="hud-label">SCORE</div>
    <div id="score">0</div>
    <div id="hud-best"></div>
  </div>
  <div id="hud-right">
    <div class="ro"><span class="hud-label">ALT</span><span class="v" id="alt">000</span></div>
    <div class="ro"><span class="hud-label">VEL</span><span class="v" id="vel">000</span></div>
    <div class="hud-btns">
      <button class="icobtn" id="btn-mute" title="mute (M)">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M11 5 6 9H2v6h4l5 4V5z"/>
          <path class="on" d="M15.5 8.5a5 5 0 0 1 0 7"/><path class="on" d="M18.5 5.5a9 9 0 0 1 0 13"/>
          <g class="off"><path d="M16 9l6 6"/><path d="M22 9l-6 6"/></g>
        </svg>
      </button>
      <button class="icobtn" id="btn-pause" title="pause (P)">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <g class="on"><path d="M9 5v14"/><path d="M15 5v14"/></g>
          <g class="off"><path d="M7 4l13 8-13 8V4z"/></g>
        </svg>
      </button>
    </div>
  </div>
  <div id="hint">MOUSE — steer &nbsp;·&nbsp; P — pause &nbsp;·&nbsp; M — mute</div>
</div>

<div id="menu" class="overlay">
  <div class="overline">ENDLESS DUSK FLIGHT</div>
  <h1><span class="dog"></span>SUNDOG<span class="dog"></span></h1>
  <div class="rule"></div>
  <div class="tagline">Race the falling sun. Thread the stones. Stay off the sand.<br>Graze a monolith for a slow-motion close call — chain them for more.</div>
  <div class="controls">
    <span><b>MOUSE</b>steer</span><span><b>WASD</b>steer</span><span><b>SPACE</b>start</span><span><b>P</b>pause</span>
  </div>
  <button class="btn" id="btn-start">BEGIN FLIGHT</button>
  <div class="subhint">RINGS +150 &nbsp;·&nbsp; CLOSE CALLS +40 × CHAIN &nbsp;·&nbsp; SPEED ALWAYS RISES</div>
  <div class="bestline" id="menu-best"></div>
</div>

<div id="over" class="overlay hidden">
  <div class="overline">FLIGHT OVER</div>
  <div id="cause">THE STONE TOOK YOU</div>
  <div class="rule"></div>
  <div class="stats">
    <div class="stat"><div class="hud-label">SCORE</div><div class="v" id="f-score">0</div></div>
    <div class="stat"><div class="hud-label">DISTANCE</div><div class="v" id="f-dist">0</div></div>
    <div class="stat"><div class="hud-label">RINGS</div><div class="v" id="f-rings">0</div></div>
    <div class="stat"><div class="hud-label">CLOSE CALLS</div><div class="v" id="f-near">0</div></div>
  </div>
  <div id="newbest">NEW BEST FLIGHT</div>
  <button class="btn" id="btn-again">FLY AGAIN</button>
  <div class="subhint">or press SPACE</div>
</div>

<div id="paused" class="overlay hidden">
  <div class="overline">PAUSED</div>
  <div class="tagline">press P to resume</div>
</div>

<script type="module">
import * as THREE from 'three';

/* ================= utils ================= */
const rand=(a,b)=>a+Math.random()*(b-a);
const clamp=(v,a,b)=>v<a?a:v>b?b:v;
const damp=(k,dt)=>1-Math.exp(-k*dt);

function hash2(ix,iz){
  let n=(ix*374761393+iz*668265263)|0;
  n=(n^(n>>>13))|0; n=Math.imul(n,1274126177);
  return ((n^(n>>>16))>>>0)/4294967295;
}
function vnoise(x,z){
  const ix=Math.floor(x),iz=Math.floor(z),fx=x-ix,fz=z-iz;
  const u=fx*fx*(3-2*fx),v=fz*fz*(3-2*fz);
  const a=hash2(ix,iz),b=hash2(ix+1,iz),c=hash2(ix,iz+1),d=hash2(ix+1,iz+1);
  return a+(b-a)*u+(c-a)*v+(a-b-c+d)*u*v;
}
function fbm(x,z){
  let h=0,amp=.55,f=.011;
  for(let i=0;i<4;i++){h+=vnoise(x*f+i*7.3,z*f+i*3.1)*amp;f*=2.07;amp*=.5;}
  return h/1.03125;
}
function terrainH(x,z){
  const a=Math.abs(x);
  const corr=a<38?0.22+0.78*Math.pow(a/38,1.6):1+(a-38)*0.012;
  return -3+Math.pow(fbm(x,z),1.5)*34*corr;
}

/* ================= renderer / scene ================= */
const canvas=document.getElementById('c');
const renderer=new THREE.WebGLRenderer({canvas,antialias:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.setSize(innerWidth,innerHeight);
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;
renderer.toneMapping=THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure=1.12;
renderer.outputColorSpace=THREE.SRGBColorSpace;

const scene=new THREE.Scene();
scene.fog=new THREE.Fog(new THREE.Color('#2b1207'),70,470);
const camera=new THREE.PerspectiveCamera(66,innerWidth/innerHeight,.1,3000);
camera.position.set(0,14,20);

const sunDir=new THREE.Vector3(-.42,.16,-.89).normalize();

const dirLight=new THREE.DirectionalLight('#ffa04e',3.4);
dirLight.position.copy(sunDir).multiplyScalar(260);
dirLight.castShadow=true;
dirLight.shadow.mapSize.set(2048,2048);
Object.assign(dirLight.shadow.camera,{left:-150,right:150,top:150,bottom:-150,near:30,far:700});
dirLight.shadow.camera.updateProjectionMatrix();
dirLight.shadow.bias=-0.0005;
dirLight.shadow.normalBias=2;
scene.add(dirLight);
scene.add(new THREE.HemisphereLight('#7a4058','#241108',1.5));
const fill=new THREE.DirectionalLight('#ffb98a',.55);
fill.position.set(30,40,60);
scene.add(fill);

/* dusk sky dome */
const sky=new THREE.Mesh(
  new THREE.SphereGeometry(1500,24,14),
  new THREE.ShaderMaterial({
    side:THREE.BackSide,depthWrite:false,fog:false,
    uniforms:{
      cTop:{value:new THREE.Color('#160b09')},
      cMid:{value:new THREE.Color('#57200f')},
      cLow:{value:new THREE.Color('#c25f24')},
      sd:{value:sunDir}
    },
    vertexShader:`varying vec3 vD;void main(){vD=normalize(position);gl_Position=projectionMatrix*modelViewMatrix*vec4(position,1.);}`,
    fragmentShader:`
      varying vec3 vD;uniform vec3 cTop,cMid,cLow,sd;
      void main(){
        float h=vD.y;
        vec3 col=mix(cLow,cMid,smoothstep(0.,.18,h));
        col=mix(col,cTop,smoothstep(.14,.6,h));
        col=mix(col,cLow*.5,smoothstep(0.,-.25,h));
        float s=pow(max(dot(vD,sd),0.),5.);
        col+=vec3(1.,.55,.25)*s*.5;
        float s2=pow(max(dot(vD,sd),0.),60.);
        col+=vec3(1.,.75,.45)*s2*.6;
        gl_FragColor=vec4(col,1.);
      }`
  })
);
sky.frustumCulled=false;
scene.add(sky);

function makeDot(size){
  const c=document.createElement('canvas');c.width=c.height=size;
  const g=c.getContext('2d');
  const gr=g.createRadialGradient(size/2,size/2,0,size/2,size/2,size/2);
  gr.addColorStop(0,'rgba(255,255,255,1)');
  gr.addColorStop(.4,'rgba(255,255,255,.55)');
  gr.addColorStop(1,'rgba(255,255,255,0)');
  g.fillStyle=gr;g.fillRect(0,0,size,size);
  return new THREE.CanvasTexture(c);
}
const dotTex=makeDot(64),haloTex=makeDot(128);

const sunDisc=new THREE.Mesh(new THREE.CircleGeometry(90,48),new THREE.MeshBasicMaterial({color:'#ffd9a6',fog:false}));
sunDisc.position.copy(sunDir).multiplyScalar(1250);
sunDisc.lookAt(0,0,0);
scene.add(sunDisc);
const halo=new THREE.Sprite(new THREE.SpriteMaterial({map:haloTex,color:'#ff8a3c',transparent:true,opacity:.45,blending:THREE.AdditiveBlending,depthWrite:false,fog:false}));
halo.scale.set(620,620,1);
halo.position.copy(sunDir).multiplyScalar(1290);
scene.add(halo);

/* faint early stars */
{
  const n=260,p=new Float32Array(n*3);
  for(let i=0;i<n;i++){
    const t=rand(0,Math.PI*2),y=rand(.25,.95),r=Math.sqrt(1-y*y);
    p[i*3]=Math.cos(t)*r*1150;p[i*3+1]=y*1150;p[i*3+2]=Math.sin(t)*r*1150;
  }
  const g=new THREE.BufferGeometry();
  g.setAttribute('position',new THREE.BufferAttribute(p,3));
  scene.add(new THREE.Points(g,new THREE.PointsMaterial({color:'#ffdcb8',size:1.6,sizeAttenuation:false,transparent:true,opacity:.5,fog:false,depthWrite:false})));
}

/* ================= scrolling terrain ================= */
let scroll=0;
const CHUNK_W=170,CHUNK_D=220,SEG=34,ROWS=4;
const chunks=[];
const terraMat=new THREE.MeshStandardMaterial({vertexColors:true,flatShading:true,roughness:1});
const cLowC=new THREE.Color('#59351a'),cHighC=new THREE.Color('#d6a065'),tmpC=new THREE.Color();

function buildChunk(m){
  const g=m.geometry,p=g.attributes.position,col=g.attributes.color;
  const cx=m.userData.cx,cz=m.userData.cz,S=scroll;
  for(let i=0;i<p.count;i++){
    const wx=cx+p.getX(i),wz=cz+p.getZ(i)-S;
    const h=terrainH(wx,wz);
    p.setY(i,h);
    const t=Math.pow(clamp((h+3)/34,0,1),.85);
    tmpC.lerpColors(cLowC,cHighC,t);
    col.setXYZ(i,tmpC.r,tmpC.g,tmpC.b);
  }
  p.needsUpdate=true;col.needsUpdate=true;
  g.computeVertexNormals();
}
for(let r=0;r<ROWS;r++)for(let c=0;c<5;c++){
  const g=new THREE.PlaneGeometry(CHUNK_W,CHUNK_D,SEG,SEG);
  g.rotateX(-Math.PI/2);
  g.setAttribute('color',new THREE.BufferAttribute(new Float32Array(g.attributes.position.count*3),3));
  const m=new THREE.Mesh(g,terraMat);
  m.userData.cx=(c-2)*CHUNK_W;
  m.userData.cz=130-CHUNK_D/2-r*CHUNK_D;
  m.position.set(m.userData.cx,0,m.userData.cz);
  m.receiveShadow=true;
  buildChunk(m);
  scene.add(m);
  chunks.push(m);
}

/* ================= the glider ================= */
const ship=new THREE.Group();
const hullMat=new THREE.MeshStandardMaterial({color:'#221510',roughness:.5,metalness:.25,flatShading:true,side:THREE.DoubleSide});
function triGeo(tris){
  const g=new THREE.BufferGeometry();
  /* FIX: flat(Infinity) — plain flat() only unwraps one level, which produced NaN vertices */
  g.setAttribute('position',new THREE.BufferAttribute(new Float32Array(tris.flat(Infinity)),3));
  g.computeVertexNormals();
  return g;
}
const N=[0,0,-3.6],T=[0,.55,1.8],B=[0,-.3,1.9],L=[-.62,.05,1.85],R=[.62,.05,1.85];
const body=new THREE.Mesh(triGeo([
  [N,L,T],[N,T,R],
  [N,B,L],[N,R,B],
  [T,L,B],[T,B,R]
]),hullMat);
ship.add(body);
const wingR=new THREE.Mesh(triGeo([[.4,0,-.4],[3.3,.12,2],[.5,0,1.7]]),hullMat);
const wingL=new THREE.Mesh(triGeo([[-.4,0,-.4],[-3.3,.12,2],[-.5,0,1.7]]),hullMat);
const fin=new THREE.Mesh(triGeo([[0,.15,.6],[0,1.5,2],[0,.15,2]]),hullMat);
ship.add(wingR,wingL,fin);
const lampMat=new THREE.MeshStandardMaterial({color:'#241505',emissive:'#ff7b2f',emissiveIntensity:2.2});
const tipR=new THREE.Mesh(new THREE.BoxGeometry(.18,.18,.18),lampMat);tipR.position.set(3.3,.12,2);
const tipL=tipR.clone();tipL.position.x=-3.3;
const engine=new THREE.Mesh(new THREE.SphereGeometry(.17,8,8),new THREE.MeshStandardMaterial({color:'#33200e',emissive:'#ffc06a',emissiveIntensity:2.6}));
engine.position.set(0,.05,2.05);
ship.add(tipR,tipL,engine);
body.castShadow=wingR.castShadow=wingL.castShadow=fin.castShadow=true;
scene.add(ship);

/* wingtip / engine trails */
class Trail{
  constructor(n,hex){
    this.n=n;this.p=new Float32Array(n*3);this.c=new Float32Array(n*3);
    this.g=new THREE.BufferGeometry();
    this.g.setAttribute('position',new THREE.BufferAttribute(this.p,3));
    this.g.setAttribute('color',new THREE.BufferAttribute(this.c,3));
    this.base=new THREE.Color(hex);
    this.line=new THREE.Line(this.g,new THREE.LineBasicMaterial({vertexColors:true,transparent:true,opacity:.85,blending:THREE.AdditiveBlending,depthWrite:false}));
    this.line.frustumCulled=false;
    scene.add(this.line);
    this.v=new THREE.Vector3();
  }
  reset(w){for(let i=0;i<this.n;i++){this.p[i*3]=w.x;this.p[i*3+1]=w.y;this.p[i*3+2]=w.z;}}
  push(w,boost){
    this.p.copyWithin(3,0,this.p.length-3);
    this.p[0]=w.x;this.p[1]=w.y;this.p[2]=w.z;
    for(let i=0;i<this.n;i++){
      const f=Math.pow(1-i/this.n,2)*boost;
      this.c[i*3]=this.base.r*f;this.c[i*3+1]=this.base.g*f;this.c[i*3+2]=this.base.b*f;
    }
    this.g.attributes.position.needsUpdate=true;
    this.g.attributes.color.needsUpdate=true;
  }
}
const trails=[
  {t:new Trail(26,'#ff7b33'),l:new THREE.Vector3(-3.3,.12,2),m:.9},
  {t:new Trail(26,'#ff7b33'),l:new THREE.Vector3(3.3,.12,2),m:.9},
  {t:new Trail(30,'#ffb35c'),l:new THREE.Vector3(0,.05,2.05),m:1.4},
];

/* wind streak lines */
const SN=170,sPos=new Float32Array(SN*6),sDat=[];
for(let i=0;i<SN;i++)sDat.push({x:rand(-70,70),y:rand(1,52),z:rand(-380,60)});
const sGeo=new THREE.BufferGeometry();
sGeo.setAttribute('position',new THREE.BufferAttribute(sPos,3));
scene.add(new THREE.LineSegments(sGeo,new THREE.LineBasicMaterial({color:'#d8a874',transparent:true,opacity:.32,depthWrite:false})));
function updateStreaks(mv,spd){
  const len=clamp(spd*.07,1.6,7.5);
  for(let i=0;i<SN;i++){
    const d=sDat[i];
    d.z+=mv*1.25;
    if(d.z>60){d.z-=440;d.x=rand(-70,70);d.y=rand(1,52);}
    const j=i*6;
    sPos[j]=sPos[j+3]=d.x;sPos[j+1]=sPos[j+4]=d.y;
    sPos[j+2]=d.z;sPos[j+5]=d.z-len;
  }
  sGeo.attributes.position.needsUpdate=true;
}

/* ================= object pools ================= */
const spireGeos=[];
for(let v=0;v<4;v++){
  const g=new THREE.ConeGeometry(1,1,5,4),p=g.attributes.position;
  for(let i=0;i<p.count;i++){
    const x=p.getX(i),y=p.getY(i),z=p.getZ(i),l=Math.hypot(x,z)||1;
    const k=hash2(Math.round((x+2)*997)+v*13,Math.round((z+2)*997)+Math.round(y*131));
    const j=(k-.5)*.34;
    p.setX(i,x+x/l*j);p.setZ(i,z+z/l*j);
  }
  g.translate(0,.5,0);
  spireGeos.push(g);
}
const basaltMat=new THREE.MeshStandardMaterial({color:'#1a110b',roughness:.95,flatShading:true});
const spires=[];
for(let i=0;i<44;i++){
  const m=new THREE.Mesh(spireGeos[i%4],basaltMat);
  m.visible=false;m.castShadow=true;scene.add(m);
  spires.push({mesh:m,active:false,r:1,h:1,passed:false,minD:99});
}
const getSpire=()=>spires.find(o=>!o.active);

const floGeos=[];
for(let v=0;v<3;v++){
  const g=new THREE.OctahedronGeometry(1,0),p=g.attributes.position;
  for(let i=0;i<p.count;i++){
    const x=p.getX(i),y=p.getY(i),z=p.getZ(i);
    const k=hash2(Math.round((x+2)*991),Math.round((y+2)*631)+Math.round(z*313)+v*7);
    const s=1+(k-.5)*.5;
    p.setXYZ(i,x*s,y*s,z*s);
  }
  floGeos.push(g);
}
const floaters=[];
for(let i=0;i<16;i++){
  const m=new THREE.Mesh(floGeos[i%3],new THREE.MeshStandardMaterial({color:'#241812',roughness:.9,flatShading:true}));
  m.visible=false;m.castShadow=true;scene.add(m);
  floaters.push({mesh:m,active:false,r:1,passed:false,minD:99});
}
const getFlo=()=>floaters.find(o=>!o.active);

const ringGeo=new THREE.TorusGeometry(3.6,.34,10,40);
const ringMat=new THREE.MeshStandardMaterial({color:'#38200c',emissive:'#ff8c42',emissiveIntensity:1.7,roughness:.45});
const rings=[];
for(let i=0;i<18;i++){
  const m=new THREE.Mesh(ringGeo,ringMat);
  const glow=new THREE.Sprite(new THREE.SpriteMaterial({map:haloTex,color:'#ff9040',transparent:true,opacity:.4,blending:THREE.AdditiveBlending,depthWrite:false,fog:false}));
  glow.scale.set(22,22,1);
  m.add(glow);
  m.visible=false;scene.add(m);
  rings.push({mesh:m,active:false,collected:false,bob:rand(0,6)});
}
const getRing=()=>rings.find(o=>!o.active);

/* crash debris */
const shards=[];
const shardEmberMat=new THREE.MeshStandardMaterial({color:'#38200c',emissive:'#ff7b2f',emissiveIntensity:2,flatShading:true});
const shardHullMat=new THREE.MeshStandardMaterial({color:'#2a1a12',roughness:.7,flatShading:true});
for(let i=0;i<26;i++){
  const m=new THREE.Mesh(new THREE.TetrahedronGeometry(.42),i%4===0?shardEmberMat:shardHullMat);
  m.visible=false;scene.add(m);
  shards.push({mesh:m,active:false,v:new THREE.Vector3(),av:new THREE.Vector3()});
}
function updateShards(dt,mv){
  for(const s of shards){
    if(!s.active)continue;
    s.v.y-=26*dt;
    s.mesh.position.addScaledVector(s.v,dt);
    s.mesh.position.z+=mv*.6;
    s.mesh.rotation.x+=s.av.x*dt;s.mesh.rotation.y+=s.av.y*dt;s.mesh.rotation.z+=s.av.z*dt;
    if(s.mesh.position.z>60||s.mesh.position.y<-6){s.active=false;s.mesh.visible=false;}
  }
}

/* spark bursts */
const PN=240,spPos=new Float32Array(PN*3),spCol=new Float32Array(PN*3),spDat=[];
for(let i=0;i<PN;i++){spPos[i*3+1]=-999;spDat.push({life:0,vx:0,vy:0,vz:0,r:0,g:0,b:0});}
const spGeo=new THREE.BufferGeometry();
spGeo.setAttribute('position',new THREE.BufferAttribute(spPos,3));
spGeo.setAttribute('color',new THREE.BufferAttribute(spCol,3));
scene.add(new THREE.Points(spGeo,new THREE.PointsMaterial({size:.8,map:dotTex,vertexColors:true,transparent:true,blending:THREE.AdditiveBlending,depthWrite:false})));
const tmpBase=new THREE.Color();
function burst(x,y,z,n,spd,hex){
  tmpBase.set(hex);let placed=0;
  for(let i=0;i<PN&&placed<n;i++){
    const d=spDat[i];
    if(d.life>0)continue;
    d.life=rand(.5,.9);
    const th=rand(0,Math.PI*2),ph=Math.acos(rand(-1,1)),s=rand(.35,1)*spd;
    d.vx=Math.sin(ph)*Math.cos(th)*s;d.vy=Math.cos(ph)*s+3;d.vz=Math.sin(ph)*Math.sin(th)*s;
    d.r=tmpBase.r;d.g=tmpBase.g;d.b=tmpBase.b;
    spPos[i*3]=x;spPos[i*3+1]=y;spPos[i*3+2]=z;
    placed++;
  }
}
function updateSparks(dt,mv){
  for(let i=0;i<PN;i++){
    const d=spDat[i];
    if(d.life<=0)continue;
    d.life-=dt*1.5;
    if(d.life<=0){spPos[i*3+1]=-999;spCol[i*3]=spCol[i*3+1]=spCol[i*3+2]=0;continue;}
    d.vy-=18*dt;
    spPos[i*3]+=d.vx*dt;spPos[i*3+1]+=d.vy*dt;spPos[i*3+2]+=d.vz*dt+mv;
    const f=Math.min(1,d.life*1.6);
    spCol[i*3]=d.r*f;spCol[i*3+1]=d.g*f;spCol[i*3+2]=d.b*f;
  }
  spGeo.attributes.position.needsUpdate=true;
  spGeo.attributes.color.needsUpdate=true;
}

/* ================= audio ================= */
let AC=null,master,windGain,windFilt,noiseBuf=null,muted=false;
function audio(){
  if(AC)return;
  try{AC=new (window.AudioContext||window.webkitAudioContext)();}catch(e){return;}
  master=AC.createGain();master.gain.value=muted?0:.85;master.connect(AC.destination);
  const len=2*AC.sampleRate;
  noiseBuf=AC.createBuffer(1,len,AC.sampleRate);
  const d=noiseBuf.getChannelData(0);
  for(let i=0;i<len;i++)d[i]=Math.random()*2-1;
  const src=AC.createBufferSource();src.buffer=noiseBuf;src.loop=true;
  windFilt=AC.createBiquadFilter();windFilt.type='bandpass';windFilt.frequency.value=420;windFilt.Q.value=.55;
  windGain=AC.createGain();windGain.gain.value=0;
  src.connect(windFilt).connect(windGain).connect(master);
  src.start();
}
function setWind(n){
  if(!AC)return;
  windGain.gain.setTargetAtTime(.02+n*.13,AC.currentTime,.25);
  windFilt.frequency.setTargetAtTime(300+n*1100,AC.currentTime,.25);
}
function chime(chain){
  if(!AC||muted)return;
  const t=AC.currentTime,b=560*Math.pow(1.122,Math.min(chain,8));
  [[b,0],[b*1.5,.07]].forEach(([f,dl])=>{
    const o=AC.createOscillator(),g=AC.createGain();
    o.type='sine';o.frequency.value=f;
    g.gain.setValueAtTime(0,t+dl);
    g.gain.linearRampToValueAtTime(.24,t+dl+.015);
    g.gain.exponentialRampToValueAtTime(.001,t+dl+.4);
    o.connect(g).connect(master);o.start(t+dl);o.stop(t+dl+.45);
  });
}
function whoosh(){
  if(!AC||muted)return;
  const t=AC.currentTime,s=AC.createBufferSource();s.buffer=noiseBuf;
  const f=AC.createBiquadFilter();f.type='bandpass';f.Q.value=1.2;
  f.frequency.setValueAtTime(2600,t);f.frequency.exponentialRampToValueAtTime(240,t+.26);
  const g=AC.createGain();
  g.gain.setValueAtTime(0,t);g.gain.linearRampToValueAtTime(.5,t+.04);g.gain.exponentialRampToValueAtTime(.001,t+.3);
  s.connect(f).connect(g).connect(master);s.start(t);s.stop(t+.32);
}
function boom(){
  if(!AC||muted)return;
  const t=AC.currentTime;
  const o=AC.createOscillator();
  o.type='sine';
  o.frequency.setValueAtTime(140,t);o.frequency.exponentialRampToValueAtTime(34,t+.9);
  const g=AC.createGain();
  g.gain.setValueAtTime(.9,t);g.gain.exponentialRampToValueAtTime(.001,t+1);
  o.connect(g).connect(master);o.start(t);o.stop(t+1.05);
  const s=AC.createBufferSource();s.buffer=noiseBuf;
  const f=AC.createBiquadFilter();f.type='lowpass';
  f.frequency.setValueAtTime(900,t);f.frequency.exponentialRampToValueAtTime(90,t+.6);
  const g2=AC.createGain();
  g2.gain.setValueAtTime(.7,t);g2.gain.exponentialRampToValueAtTime(.001,t+.7);
  s.connect(f).connect(g2).connect(master);s.start(t);s.stop(t+.75);
}

/* ================= UI refs ================= */
const $=id=>document.getElementById(id);
const hud=$('hud'),scoreEl=$('score'),altEl=$('alt'),velEl=$('vel'),hudBest=$('hud-best'),
      menuEl=$('menu'),overEl=$('over'),pausedEl=$('paused'),pulseEl=$('pulse'),
      menuBest=$('menu-best'),hintEl=$('hint'),causeEl=$('cause'),newbestEl=$('newbest'),
      fScore=$('f-score'),fDist=$('f-dist'),fRings=$('f-rings'),fNear=$('f-near');
if('ontouchstart' in window)hintEl.textContent='DRAG — steer';

const proj=new THREE.Vector3();
function popupAt(txt,x,y){
  const el=document.createElement('div');
  el.className='pop';el.textContent=txt;
  el.style.left=x+'px';el.style.top=y+'px';
  document.body.appendChild(el);
  setTimeout(()=>el.remove(),960);
}
function popup(txt,wp){
  proj.copy(wp).project(camera);
  if(proj.z>1)return;
  popupAt(txt,(proj.x*.5+.5)*innerWidth,(-proj.y*.5+.5)*innerHeight);
}
function flashPulse(){
  pulseEl.classList.add('on');
  setTimeout(()=>pulseEl.classList.remove('on'),90);
}

/* ================= game state ================= */
let state='MENU',overShown=false,deathCause='';
let speed=26,timeScale=1,tsTarget=1,slowT=0,elapsed=0;
let bonus=0,chain=0,chainT=-9,ringsGot=0,nearCount=0,bestBeaten=false;
let shake=0,fovKick=0,deathT=0,nextSpawnZ=-700;
let px=0,py=11.5,vx=0,vy=0,bank=0,tx=0,ty=11.5;
let best=+(localStorage.getItem('sundog_best')||0);
let dist0Scroll=0;
const keys={};   /* FIX: this declaration was missing — every keypress threw ReferenceError */
const difficulty=()=>clamp((scroll-dist0Scroll)/2600,0,1);

function setState(s){state=s;document.body.dataset.state=s;}

function start(){
  audio();
  if(AC&&AC.state==='suspended')AC.resume();
  for(const o of spires){o.active=false;o.mesh.visible=false;}
  for(const o of floaters){o.active=false;o.mesh.visible=false;}
  for(const o of rings){o.active=false;o.mesh.visible=false;}
  for(const s of shards){s.active=false;s.mesh.visible=false;}
  dist0Scroll=scroll;bonus=0;chain=0;chainT=-9;ringsGot=0;nearCount=0;bestBeaten=false;
  nextSpawnZ=-680;slowT=0;deathT=0;overShown=false;shake=0;fovKick=0;
  tsTarget=1;timeScale=1;
  px=clamp(px,-30,30);py=clamp(py,3.5,42);
  py=Math.max(py,terrainH(px,-scroll)+3);
  vx=vy=0;tx=px;ty=py;
  ship.visible=true;
  ship.position.set(px,py,0);
  ship.updateMatrixWorld(true);
  for(const tr of trails){ship.localToWorld(tr.t.v.copy(tr.l));tr.t.reset(tr.t.v);}
  speed=32;   /* restart at a fair speed instead of keeping your old top speed */
  setState('PLAYING');
  menuEl.classList.add('hidden');overEl.classList.add('hidden');
  hud.classList.remove('hidden');
  hudBest.textContent='BEST '+best.toLocaleString('en-US');
}

function crash(cause){
  if(state!=='PLAYING')return;
  setState('DEAD');
  deathCause=cause;deathT=0;overShown=false;
  tsTarget=.16;shake=1.5;
  ship.visible=false;
  for(const s of shards){
    s.active=true;s.mesh.visible=true;
    s.mesh.position.set(px+rand(-.5,.5),py+rand(-.3,.3),rand(-.5,.5));
    s.v.set(rand(-1,1),rand(-.2,1),rand(-1,1)).normalize().multiplyScalar(rand(7,20));
    s.v.z+=speed*.35;
    s.av.set(rand(-6,6),rand(-6,6),rand(-6,6));
  }
  burst(px,py,0,34,16,'#ffab60');
  boom();
}

function showOver(){
  overShown=true;
  const dist=Math.floor(scroll-dist0Scroll);
  const sc=Math.floor(dist+bonus);
  fScore.textContent=sc.toLocaleString('en-US');
  fDist.textContent=dist.toLocaleString('en-US')+' m';
  fRings.textContent=ringsGot;
  fNear.textContent=nearCount;
  causeEl.textContent=deathCause;
  if(sc>best){best=sc;localStorage.setItem('sundog_best',best);newbestEl.classList.add('show');}
  else newbestEl.classList.remove('show');
  overEl.classList.remove('hidden');
}

function togglePause(){
  if(state==='PLAYING'){setState('PAUSED');pausedEl.classList.remove('hidden');if(AC)setWind(0);}
  else if(state==='PAUSED'){setState('PLAYING');pausedEl.classList.add('hidden');}
}
function toggleMute(){
  muted=!muted;
  if(master)master.gain.value=muted?0:.85;
  $('btn-mute').classList.toggle('alt',muted);
}

/* ================= spawning ================= */
function placeRing(o,x,zo){
  const y=Math.max(rand(7,26),terrainH(x,zo-scroll)+3.5);
  o.active=true;o.collected=false;o.mesh.visible=true;
  o.mesh.position.set(x,y,zo);
  o.mesh.scale.setScalar(1);
}
function spawnArc(z){
  const bx=rand(-18,18);let zz=z;
  for(let k=0;k<3;k++){
    const o=getRing();if(!o)break;
    placeRing(o,clamp(bx+Math.sin(k*1.15)*10,-28,28),zz);
    zz-=30;
  }
}
function spawnWave(z){
  const d=difficulty(),roll=Math.random();
  if(roll<.60){
    const lanes=[-26,-13,0,13,26].sort(()=>Math.random()-.5);
    const n=Math.min(5,2+Math.floor(Math.random()*1.7+d*1.4));
    for(let i=0;i<n;i++){
      const o=getSpire();if(!o)break;
      const x=lanes[i]+rand(-2.5,2.5),zo=z-rand(0,16);
      const h=rand(14,30)+d*10,r=rand(2.1,3.9);
      o.active=true;o.passed=false;o.minD=99;o.h=h;o.r=r;
      o.mesh.visible=true;
      o.mesh.position.set(x,terrainH(x,zo-scroll)-2,zo);
      o.mesh.scale.set(r,h,r);
      o.mesh.rotation.set(rand(-.07,.07),rand(0,6.28),rand(-.07,.07));
    }
  }else if(roll<.82){
    const n=1+(Math.random()<d*.8?1:0);
    for(let i=0;i<n;i++){
      const o=getFlo();if(!o)break;
      const x=rand(-26,26),zo=z-rand(0,20),s=rand(2.6,4.6);
      o.active=true;o.passed=false;o.minD=99;o.r=s*.95;
      o.mesh.visible=true;
      o.mesh.position.set(x,Math.max(rand(11,30),terrainH(x,zo-scroll)+5),zo);
      o.mesh.scale.set(s*rand(.8,1.3),s*rand(.8,1.3),s*rand(.8,1.3));
      o.mesh.rotation.set(rand(0,3),rand(0,3),rand(0,3));
    }
  }else spawnArc(z);
  if(Math.random()<.38){
    const o=getRing();
    if(o)placeRing(o,rand(-24,24),z-rand(20,60));
  }
}

/* ================= input ================= */
addEventListener('pointermove',e=>{
  tx=clamp((e.clientX/innerWidth*2-1)*34,-30,30);
  ty=clamp(3+(1-e.clientY/innerHeight)*40,3.2,42);
});
addEventListener('keydown',e=>{
  keys[e.code]=true;
  if(e.code==='Space'){
    e.preventDefault();
    if(state==='MENU')start();
    else if(state==='DEAD'&&overShown)start();
  }
  if(e.code==='KeyP'||e.code==='Escape')togglePause();
  if(e.code==='KeyM')toggleMute();
});
addEventListener('keyup',e=>keys[e.code]=false);
addEventListener('blur',()=>{for(const k in keys)keys[k]=false;});
document.addEventListener('visibilitychange',()=>{if(document.hidden&&state==='PLAYING')togglePause();});
 $('btn-start').addEventListener('click',e=>{e.currentTarget.blur();start();});
 $('btn-again').addEventListener('click',e=>{e.currentTarget.blur();start();});
 $('btn-mute').addEventListener('click',e=>{e.currentTarget.blur();audio();toggleMute();});
 $('btn-pause').addEventListener('click',e=>{e.currentTarget.blur();togglePause();});
addEventListener('resize',()=>{
  camera.aspect=innerWidth/innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth,innerHeight);
});

/* ================= main update ================= */
const camT=new THREE.Vector3();
function nearMiss(wp){
  chain=(elapsed-chainT<2.2)?chain+1:1;
  chainT=elapsed;
  const pts=40*chain;
  bonus+=pts;nearCount++;slowT=.3;fovKick=7;
  popup(`CLOSE ×${chain}  +${pts}`,wp);
  whoosh();flashPulse();
}

function update(dt,rdt){
  elapsed+=dt;
  const playing=state==='PLAYING',menu=state==='MENU';

  if(playing)speed+=(32+44*difficulty()-speed)*damp(.5,rdt);
  else if(menu)speed=26;
  const mv=speed*dt;
  scroll+=mv;

  if(playing){
    nextSpawnZ+=mv;
    if(nextSpawnZ>-640){
      spawnWave(nextSpawnZ);
      nextSpawnZ-=rand(105,155)-difficulty()*48;
    }
  }

  for(const m of chunks){
    m.position.z+=mv;
    if(m.position.z-CHUNK_D/2>170){m.position.z-=ROWS*CHUNK_D;buildChunk(m);}
  }

  if(playing){
    if(keys.KeyA||keys.ArrowLeft)tx-=60*rdt;
    if(keys.KeyD||keys.ArrowRight)tx+=60*rdt;
    if(keys.KeyW||keys.ArrowUp)ty+=48*rdt;
    if(keys.KeyS||keys.ArrowDown)ty-=48*rdt;
    tx=clamp(tx,-30,30);ty=clamp(ty,3.2,42);
  }else if(menu){
    tx=Math.sin(elapsed*.42)*17;
    ty=11.5+Math.sin(elapsed*.71)*3.5;
  }
  if(playing||menu){
    vx+=((tx-px)*26-vx*6.2)*dt;px+=vx*dt;
    vy+=((ty-py)*26-vy*6.2)*dt;py+=vy*dt;
    if(py>42){py=42;vy=Math.min(vy,0);}
    if(playing&&py<terrainH(px,-scroll)+.7)crash('THE SAND TOOK YOU');
    bank+=(clamp(-vx*.055,-1,1)-bank)*damp(8,rdt);
    ship.position.set(px,py,0);
    ship.rotation.z=bank*.8;
    ship.rotation.y=bank*.15;
    ship.rotation.x=clamp(vy*.05,-.5,.5);
  }

  if(playing){slowT-=rdt;tsTarget=slowT>0?.35:1;}
  if(state==='DEAD'){
    deathT+=rdt;
    tsTarget=deathT<.55?.16:.045;
    if(deathT>1.25&&!overShown)showOver();
  }

  for(const o of spires){
    if(!o.active)continue;
    o.mesh.position.z+=mv;
    const oz=o.mesh.position.z;
    if(oz>50){o.active=false;o.mesh.visible=false;continue;}
    if(!playing||oz<-40||oz>20)continue;
    const dx=px-o.mesh.position.x,dz=-oz;
    const t=clamp((py-o.mesh.position.y)/o.h,0,1);
    const rr=o.r*(1-t*.85)+1.05;
    const dd=Math.hypot(dx,dz)-rr;
    if(dd<o.minD)o.minD=dd;
    if(dd<0&&py<o.mesh.position.y+o.h){crash('THE STONE TOOK YOU');continue;}
    if(!o.passed&&oz>3){
      o.passed=true;
      if(o.minD<3.4){nearMiss(o.mesh.position);burst(px,py,oz,5,6,'#ff9040');}
    }
  }
  for(const o of floaters){
    if(!o.active)continue;
    o.mesh.position.z+=mv;
    o.mesh.rotation.y+=dt*.8;
    const oz=o.mesh.position.z;
    if(oz>50){o.active=false;o.mesh.visible=false;continue;}
    if(!playing||oz<-40||oz>20)continue;
    const dd=Math.hypot(px-o.mesh.position.x,py-o.mesh.position.y,-oz)-(o.r+1);
    if(dd<o.minD)o.minD=dd;
    if(dd<0){crash('THE STONE TOOK YOU');continue;}
    if(!o.passed&&oz>3){
      o.passed=true;
      if(o.minD<4){nearMiss(o.mesh.position);burst(px,py,oz,5,6,'#ff9040');}
    }
  }
  for(const o of rings){
    if(!o.active)continue;
    const m=o.mesh;
    m.position.z+=mv;
    m.rotation.z+=dt*1.2;
    const s=1+Math.sin(elapsed*4+o.bob)*.05;
    m.scale.setScalar(s);
    const oz=m.position.z;
    if(oz>40){o.active=false;m.visible=false;continue;}
    if(playing&&!o.collected&&Math.abs(oz)<3.4&&Math.hypot(px-m.position.x,py-m.position.y)<3.4){
      o.collected=true;o.active=false;m.visible=false;
      ringsGot++;bonus+=150;
      popup('+150',m.position);
      burst(m.position.x,m.position.y,oz,14,9,'#ffb35c');
      chime(ringsGot);
    }
  }

  ship.updateMatrixWorld(true);
  const boost=state==='DEAD'?0:(.7+speed/130);
  for(const tr of trails){
    tr.t.v.copy(tr.l);
    ship.localToWorld(tr.t.v);
    tr.t.push(tr.t.v,boost*tr.m);
  }

  updateStreaks(mv,speed);
  updateSparks(dt,mv);
  updateShards(dt,mv);

  camT.set(px*.5,6.1+py*.62,13.4+(speed-26)*.045);
  camera.position.lerp(camT,damp(4.2,rdt));
  if(shake>0){
    camera.position.x+=rand(-1,1)*shake*.45;
    camera.position.y+=rand(-1,1)*shake*.35;
    shake*=Math.exp(-rdt*2.4);
    if(shake<.01)shake=0;
  }
  camera.lookAt(px*.82,py*.82+1.1,-46);
  camera.rotateZ(-bank*.26);
  fovKick*=Math.exp(-rdt*3.5);
  const fv=66+fovKick;
  if(Math.abs(fv-camera.fov)>.02){camera.fov=fv;camera.updateProjectionMatrix();}

  if(AC&&state!=='PAUSED')setWind(clamp((speed-26)/48,0,1)*(playing?1:(menu?.45:0)));

  if(playing||state==='DEAD'){
    const dist=Math.floor(scroll-dist0Scroll);
    const sc=Math.floor(dist+bonus);
    scoreEl.textContent=sc.toLocaleString('en-US');
    if(!bestBeaten&&best>0&&sc>best){
      bestBeaten=true;
      popupAt('NEW BEST',innerWidth/2,innerHeight*.28);
      chime(6);
    }
    altEl.textContent=String(Math.max(0,Math.round(py-terrainH(px,-scroll)))).padStart(3,'0');
    velEl.textContent=String(Math.round(speed*2.6)).padStart(3,'0');
  }
}

/* ================= boot & loop ================= */
menuBest.textContent=best>0?'BEST FLIGHT — '+best.toLocaleString('en-US'):'';
ship.position.set(px,py,0);
ship.updateMatrixWorld(true);
for(const tr of trails){ship.localToWorld(tr.t.v.copy(tr.l));tr.t.reset(tr.t.v);}

let last=performance.now();
function frame(now){
  requestAnimationFrame(frame);
  const rdt=Math.min((now-last)/1000,.05);
  last=now;
  if(state!=='PAUSED'){
    timeScale+=(tsTarget-timeScale)*(1-Math.exp(-rdt*6));
    update(rdt*timeScale,rdt);
  }
  renderer.render(scene,camera);
}
requestAnimationFrame(frame);
</script>
</body>
</html>

