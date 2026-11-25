<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Hémostase en 3D</title>
<style>
:root{--bg:#f6f3ef;--ink:#2a1a1c;--mute:#6f5d5f;--line:#ddd2cc;--card:#fffdfb;--acc:#a8322d;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
:root:not([data-theme="light"]){}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#1b1213;--ink:#f3e7e4;--mute:#b79f9b;--line:#3d2a2c;--card:#251819;--acc:#e0645e}}
:root[data-theme="dark"]{--bg:#1b1213;--ink:#f3e7e4;--mute:#b79f9b;--line:#3d2a2c;--card:#251819;--acc:#e0645e}
html{scroll-padding-top:env(safe-area-inset-top,0px);height:100%}
*,*::before,*::after{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);font-family:Georgia,"Times New Roman",serif;line-height:1.6;min-height:100%}
body::before{content:"";position:fixed;inset:0 0 auto;height:env(safe-area-inset-top,0px);background:var(--bg);z-index:10}
main{max-width:940px;margin:0 auto;padding:20px 16px 32px}
h1{font-size:clamp(26px,5vw,40px);line-height:1.15;margin:8px 0 6px;font-weight:700;letter-spacing:-.01em}
.sub{margin:0 0 18px;color:var(--mute);font-size:16px;max-width:60ch}
.stage{position:relative;border-radius:14px;overflow:hidden;border:1px solid var(--line);background:#1d0a0d}
canvas{display:block;width:100%;touch-action:pan-y;cursor:grab}
.lg{display:flex;flex-wrap:wrap;gap:6px 16px;margin:14px 0 4px;font-family:system-ui,sans-serif;font-size:13px;color:var(--mute)}
.lg span{display:flex;align-items:center;gap:6px}
.lg i{width:11px;height:11px;border-radius:50%;display:inline-block}
.hint{font-family:system-ui,sans-serif;font-size:12.5px;color:var(--mute);margin:0 0 12px}
.steps{display:flex;gap:8px;margin:14px 0 10px;flex-wrap:wrap}
.steps button{font:500 13px system-ui,sans-serif;padding:7px 12px;border-radius:999px;border:1px solid var(--line);background:var(--card);color:var(--ink);cursor:pointer}
.steps button[aria-current="true"]{background:var(--ink);color:var(--bg);border-color:var(--ink)}
.cap{min-height:104px;border-left:3px solid var(--acc);padding:2px 0 2px 14px;margin:6px 0 14px}
.cap h2{font-size:19px;margin:0 0 4px}
.cap p{margin:0;color:var(--mute);font-size:16px;max-width:62ch}
.ctl{display:flex;gap:8px;max-width:440px}
.ctl button{font:500 15px system-ui,sans-serif;padding:11px 14px;border-radius:10px;border:1px solid var(--line);background:var(--card);color:var(--ink);cursor:pointer;flex:1}
.ctl button.main{flex:2;background:var(--ink);color:var(--bg);border-color:var(--ink)}
button:focus-visible{outline:2px solid var(--acc);outline-offset:2px}
</style>
</head>
<body>
<main>
<h1>Comment le corps arrête un saignement</h1>
<p class="sub">Une brèche dans un vaisseau sanguin, en 3D : les six étapes de l’hémostase, de la lésion à la dissolution du caillot.</p>
<div class="stage"><canvas id="cv" role="img" aria-label="Vaisseau sanguin en 3D avec une brèche, rotation possible au doigt ou à la souris"></canvas></div>
<div class="lg">
<span><i style="background:#D5403A"></i>Globule rouge</span>
<span><i style="background:#8F86E8"></i>Plaquette</span>
<span><i style="background:#F3E0B8"></i>Fibrine</span>
<span><i style="background:#EF9F27"></i>Inhibiteur</span>
<span><i style="background:#D85A30"></i>Plasmine</span>
</div>
<p class="hint">Fais glisser la scène pour tourner autour du vaisseau.</p>
<div class="steps" id="dots" role="group" aria-label="Étapes"></div>
<div class="cap" aria-live="polite"><h2 id="ttl"></h2><p id="txt"></p></div>
<div class="ctl">
<button id="prev" aria-label="Étape précédente">‹</button>
<button id="play" class="main">Lecture</button>
<button id="next" aria-label="Étape suivante">›</button>
</div>
</main>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const rnd=(a,b)=>a+Math.random()*(b-a),cl=(v,a,b)=>Math.max(a,Math.min(b,v)),ease=p=>p*p*(3-2*p),rm=matchMedia('(prefers-reduced-motion:reduce)').matches?.4:1;
const cv=document.getElementById('cv');let W=380,H=300;
let rd;try{rd=new THREE.WebGLRenderer({canvas:cv,antialias:true});}catch(e){cv.parentNode.innerHTML='<p style="padding:20px;color:#f3e7e4;font-family:system-ui">La 3D n’est pas disponible sur cet appareil.</p>';throw e}
rd.setPixelRatio(Math.min(2,devicePixelRatio||1));
const scene=new THREE.Scene();scene.background=new THREE.Color(0x1d0a0d);scene.fog=new THREE.Fog(0x1d0a0d,11,28);
const camera=new THREE.PerspectiveCamera(42,W/H,.1,60);
function resize(){const w=cv.parentNode.clientWidth;W=w;H=Math.round(Math.min(560,Math.max(300,w*(w>600?.56:.8))));cv.style.height=H+'px';rd.setSize(W,H,false);camera.aspect=W/H;camera.updateProjectionMatrix()}
addEventListener('resize',resize);resize();
scene.add(new THREE.HemisphereLight(0xffd9c8,0x3a1519,1));
const dl=new THREE.DirectionalLight(0xffeedd,1.1);dl.position.set(4,8,10);scene.add(dl);
const pl=new THREE.PointLight(0xff8a5a,1.3,30);pl.position.set(-6,2,5);scene.add(pl);
const R0=3,LEN=26,NX=96,NT=64;let Rc=R0;
const wd=y=>{const q=Math.min(.9,Math.abs(y)/Rc);return Rc*(Math.sqrt(1-q*q)-1)};
function wallTex(){const c=document.createElement('canvas');c.width=c.height=512;const g=c.getContext('2d');g.fillStyle='#e9a3a3';g.fillRect(0,0,512,512);
const cols=5,rows=24,cw=512/cols,ch=512/rows;
for(let r=0;r<rows;r++)for(let q=-1;q<=cols;q++){const x=q*cw+(r%2)*cw/2,y=r*ch;g.fillStyle=`rgb(${230+rnd(-12,12)|0},${150+rnd(-15,15)|0},${155+rnd(-15,15)|0})`;g.strokeStyle='rgba(120,50,60,.55)';g.lineWidth=1.5;g.beginPath();g.ellipse(x+cw/2,y+ch/2,cw*.48,ch*.46,0,0,7);g.fill();g.stroke();g.fillStyle='rgba(150,70,90,.6)';g.beginPath();g.ellipse(x+cw/2,y+ch/2,cw*.16,ch*.28,0,0,7);g.fill()}
const t=new THREE.CanvasTexture(c);t.wrapS=t.wrapT=THREE.RepeatWrapping;return t}
function wallGeo(){const pos=[],uv=[],idx=[];for(let i=0;i<=NX;i++)for(let j=0;j<=NT;j++){const x=-LEN/2+LEN*i/NX,th=Math.PI*2*j/NT;pos.push(x,Math.cos(th),Math.sin(th));uv.push(i/NX*6,j/NT)}
for(let i=0;i<NX;i++)for(let j=0;j<NT;j++){const xc=-LEN/2+LEN*(i+.5)/NX,dth=Math.PI*2*(j+.5)/NT-Math.PI/2;if((xc/1.7)**2+(dth/.55)**2<1)continue;const a=i*(NT+1)+j,b=a+NT+1;idx.push(a,b,a+1,b,b+1,a+1)}
const g=new THREE.BufferGeometry();g.setAttribute('position',new THREE.Float32BufferAttribute(pos,3));g.setAttribute('uv',new THREE.Float32BufferAttribute(uv,2));g.setIndex(idx);g.computeVertexNormals();return g}
const wall=new THREE.Mesh(wallGeo(),new THREE.MeshStandardMaterial({map:wallTex(),color:0xffc2bc,transparent:true,opacity:.4,side:THREE.DoubleSide,depthWrite:false,roughness:.55}));scene.add(wall);
const cg=new THREE.Group();scene.add(cg);
function rbcGeo(){const pts=[],N=14,f=r=>.5*Math.sqrt(Math.max(0,1-r*r))*(.2072+2.0026*r*r-1.1228*r**4);for(let i=0;i<=N;i++)pts.push(new THREE.Vector2(i/N,f(i/N)));for(let i=N;i>=0;i--)pts.push(new THREE.Vector2(i/N,-f(i/N)));return new THREE.LatheGeometry(pts,18)}
const rg=rbcGeo(),rmat=new THREE.MeshStandardMaterial({color:0xd5403a,roughness:.38,metalness:.05,emissive:0x2e0605});
function inst(geo,mat,n,parent){const m=new THREE.InstancedMesh(geo,mat,n);m.frustumCulled=false;m.instanceMatrix.setUsage(THREE.DynamicDrawUsage);(parent||scene).add(m);return m}
const d=new THREE.Object3D();
function put(m,i,x,y,z,rx,ry,rz,sx,sy,sz){d.position.set(x,y,z);d.rotation.set(rx,ry,rz);d.scale.set(Math.max(sx,.0001),Math.max(sy,.0001),Math.max(sz,.0001));d.updateMatrix();m.setMatrixAt(i,d.matrix)}
const NR=64,rbcM=inst(rg,rmat,NR),cells=[];for(let i=0;i<NR;i++)cells.push({x:rnd(-13,13),rho:Math.sqrt(Math.random()),phi:rnd(0,6.28),ax:rnd(0,6),ay:rnd(0,6),az:rnd(0,6),w:rnd(.6,1.6)});
const NB=36,bpM=inst(new THREE.SphereGeometry(1,10,8),new THREE.MeshStandardMaterial({color:0xb9b2ff,roughness:.5}),NB),bps=[];for(let i=0;i<NB;i++)bps.push({x:rnd(-13,13),rho:Math.sqrt(Math.random()),phi:rnd(0,6.28),r:rnd(0,6)});
const ag=new THREE.IcosahedronGeometry(1,2),ap=ag.attributes.position;for(let i=0;i<ap.count;i++){const x=ap.getX(i),y=ap.getY(i),z=ap.getZ(i),l=1+Math.max(0,Math.sin(x*7+1)*Math.sin(y*6+2)*Math.sin(z*7+3))*1.0;ap.setXYZ(i,x*l,y*l,z*l)}ag.computeVertexNormals();
const NP=28,plM=inst(ag,new THREE.MeshStandardMaterial({color:0x8f86e8,roughness:.45,emissive:0x1b1740}),NP,cg),plug=[];
for(let i=0;i<NP;i++){const a=rnd(0,6.28),r=Math.sqrt(Math.random()),x=Math.cos(a)*r*2,y=Math.sin(a)*r*1.7;plug.push({tx:x,ty:y,tz:-(.12+1.3*(1-r*r)*rnd(.55,1)),sx:-9+rnd(-1,1),sy:rnd(-1.4,1.4),sz:rnd(-2.2,-1),p:0,rx:rnd(0,6),ry:rnd(0,6),rz:rnd(0,6),sp:rnd(.3,1)})}
const NT2=9,trM=inst(rg,rmat,NT2,cg),trap=[];for(let i=0;i<NT2;i++)trap.push({x:rnd(-1.5,1.5),y:rnd(-1.2,1.2),z:-rnd(.3,1.1),a:rnd(0,6),b:rnd(0,6),c:rnd(0,6)});
const NF=150,fb=[];for(let i=0;i<NF;i++){const x=rnd(-2,2),y=rnd(-1.7,1.7),h=1.5*Math.sqrt(Math.max(0,1-(x/2.2)**2-(y/1.9)**2)),z=-rnd(.1,h+.2),dir=new THREE.Vector3(rnd(-1,1),rnd(-1,1),rnd(-.7,.7)).normalize().multiplyScalar(rnd(.8,1.8));fb.push({a:[x,y,z],b:[x+dir.x,y+dir.y,Math.min(-.05,z+dir.z)],m:[rnd(-.15,.15),rnd(-.15,.15),rnd(-.15,.15)],d:i/NF})}
const fGeo=new THREE.BufferGeometry();fGeo.setAttribute('position',new THREE.BufferAttribute(new Float32Array(NF*12),3));
const fl=new THREE.LineSegments(fGeo,new THREE.LineBasicMaterial({color:0xf3e0b8,transparent:true,opacity:.9}));fl.frustumCulled=false;cg.add(fl);
function ringPts(rx,th,j){const p=[];for(let i=0;i<48;i++){const a=i/48*6.283,dt=th*Math.sin(a)+(j?rnd(-.04,.04):0);p.push(new THREE.Vector3(rx*Math.cos(a)+(j?rnd(-.05,.05):0),R0*Math.sin(dt),R0*(Math.cos(dt)-1)))}return p}
const colM=new THREE.Mesh(new THREE.TubeGeometry(new THREE.CatmullRomCurve3(ringPts(1.75,.55,1),true),64,.07,6,true),new THREE.MeshStandardMaterial({color:0xd9b382,roughness:.8,emissive:0x3a2a12,transparent:true}));cg.add(colM);
const barM=new THREE.Mesh(new THREE.TubeGeometry(new THREE.CatmullRomCurve3(ringPts(3.4,1.05,0),true),96,.05,6,true),new THREE.MeshBasicMaterial({color:0xef9f27,transparent:true}));cg.add(barM);
const pg=new THREE.PlaneGeometry(2,2,14,10),pp=pg.attributes.position;for(let i=0;i<pp.count;i++){const u=pp.getX(i),v=pp.getY(i),ux=u*Math.sqrt(1-v*v/2),vy=v*Math.sqrt(1-u*u/2),th=vy*.55;pp.setXYZ(i,ux*1.75,R0*Math.sin(th),R0*(Math.cos(th)-1)+.02)}pg.computeVertexNormals();
const patch=new THREE.Mesh(pg,new THREE.MeshStandardMaterial({map:wall.material.map,color:0xffb0b0,transparent:true,opacity:.6,side:THREE.DoubleSide,roughness:.55}));cg.add(patch);
const NA=10,amM=inst(new THREE.DodecahedronGeometry(.14),new THREE.MeshStandardMaterial({color:0xef9f27,emissive:0x5a3a08,roughness:.4}),NA,cg);
const NPm=8,pmM=inst(new THREE.IcosahedronGeometry(.22,1),new THREE.MeshStandardMaterial({color:0xd85a30,emissive:0x401006,roughness:.5}),NPm,cg);
const NLK=14,lkM=inst(rg,rmat,NLK,cg),lk=[...Array(NLK)].map(()=>({on:0,life:0,x:0,y:0,z:0,vx:0,vy:0,vz:0,r:0}));
const NPo=70,fcM=inst(new THREE.SphereGeometry(.06,8,6),new THREE.MeshBasicMaterial({color:0xffffff}),NPo,cg),pool=[...Array(NPo)].map(()=>({on:0,kind:0,life:0,max:4,x:0,y:0,z:0,vx:0,vy:0,vz:0}));
for(let i=0;i<NPo;i++)fcM.setColorAt(i,new THREE.Color(0xffffff));
const C0=new THREE.Color(0x2fb594),C1=new THREE.Color(0x9be7cf);
function emit(kind){const i=pool.findIndex(q=>!q.on);if(i<0)return;const p=pool[i];p.on=1;p.life=0;p.kind=kind;
if(kind==0){p.x=rnd(-1,1);p.y=rnd(-1,1);p.z=rnd(-1.2,-.3);const a=rnd(0,6.28),s=rnd(.8,1.5);p.vx=Math.cos(a)*s*1.1;p.vy=Math.sin(a)*s;p.vz=0;p.max=9;fcM.setColorAt(i,C0)}
else{p.x=rnd(-1.5,1.5);p.y=rnd(-1.2,1.2);p.z=rnd(-1,-.3);p.vx=rnd(1,1.8);p.vy=rnd(-.2,.2);p.vz=rnd(-.9,-.4);p.max=4;fcM.setColorAt(i,C1)}fcM.instanceColor.needsUpdate=true}
const S=[
["1. Lésion du vaisseau","La paroi est déchirée : le sang s’échappe vers les tissus et le collagène sous l’endothélium est exposé."],
["2. Vasoconstriction","Le vaisseau se contracte : le flux ralentit et la perte de sang diminue."],
["3. Hémostase primaire","Les plaquettes adhèrent au collagène (facteur de von Willebrand), s’activent en émettant des pseudopodes et s’agrègent en clou plaquettaire."],
["4. Coagulation","La cascade produit de la thrombine, qui transforme le fibrinogène en fibres de fibrine. Le réseau piège des globules rouges et stabilise le caillot."],
["5. Régulation","L’antithrombine, les protéines C et S et le TFPI bloquent les facteurs activés : le caillot ne s’étend pas aux zones saines."],
["6. Fibrinolyse et réparation","La plasmine dégrade la fibrine (libérant des D-dimères), le caillot se dissout et l’endothélium referme la brèche."]];
let stage=0,t=0,time=0,tm=null,azU=0,el=.28,drag=0,idle=0,sway=1;
const V={c:0,flow:1,fib:0,keep:0,rep:0,leak:1,pm:0,inh:0,coll:1},K={c:1.5,flow:1.5,fib:.6,keep:.5,rep:.45,leak:1.2,pm:1.5,inh:1.5,coll:1.5};
function update(dt){
const T={c:stage>=1&&stage<=4?1:0,flow:[1,.7,.55,.4,.4,.9][stage],fib:stage==3||stage==4?1:0,keep:stage>=2&&stage<=4?1:0,rep:stage==5?1:0,leak:[1,.6,.15,0,0,0][stage],pm:stage==5?1:0,inh:stage==4?1:0,coll:stage<=2?1:0};
for(const k in K)V[k]+=(T[k]-V[k])*Math.min(1,dt*K[k]);
Rc=R0*(1-.24*V.c);wall.scale.set(1,Rc,Rc);cg.position.z=Rc;
const mx=(Rc-.5)/Rc;
cells.forEach((c,i)=>{c.x+=(1.1+2.7*(1-c.rho*c.rho))*V.flow*rm*dt;if(c.x>13){c.x=-13;c.rho=Math.sqrt(Math.random());c.phi=rnd(0,6.28)}
const r=Rc*mx*c.rho,y=r*Math.cos(c.phi);let z=r*Math.sin(c.phi);const zm=Rc-.55-1.6*V.keep*Math.max(0,1-Math.abs(c.x)/3.4);if(z>zm)z=zm;
put(rbcM,i,c.x,y,z,c.ax+time*c.w,c.ay+time*c.w*.6,c.az,.36,.3,.36)});rbcM.instanceMatrix.needsUpdate=true;
bps.forEach((b,i)=>{b.x+=(1.1+2.7*(1-b.rho*b.rho))*V.flow*rm*dt;if(b.x>13){b.x=-13;b.rho=Math.sqrt(Math.random())}const r=Rc*mx*b.rho;let z=r*Math.sin(b.phi);const zm=Rc-.4-1.6*V.keep*Math.max(0,1-Math.abs(b.x)/3.4);if(z>zm)z=zm;put(bpM,i,b.x,r*Math.cos(b.phi),z,b.r+time,b.r,b.r*2,.15,.055,.15)});bpM.instanceMatrix.needsUpdate=true;
plug.forEach((q,i)=>{if(stage<2)q.p=0;else if(stage>2||t>i*.22)q.p=Math.min(1,q.p+dt*.55);
const e=ease(q.p),ar=Math.sin(q.p*3.14)*.3,y=q.sy+(q.ty-q.sy)*e+ar,z=q.sz+(q.tz-q.sz)*e,sc=(q.p>0?.1+.14*cl((q.p-.6)/.4,0,1):0)*(stage==5?V.keep:1);
put(plM,i,q.sx+(q.tx-q.sx)*e,y,z+wd(y)*e,q.rx+time*q.sp,q.ry,q.rz,sc,sc,sc)});plM.instanceMatrix.needsUpdate=true;
const ta=cl(V.fib*1.5,0,1);trap.forEach((q,i)=>put(trM,i,q.x,q.y,q.z+wd(q.y),q.a,q.b,q.c+time*.1,.34*ta,.27*ta,.34*ta));trM.instanceMatrix.needsUpdate=true;
const fp=fGeo.attributes.position.array;fb.forEach((f,i)=>{const g=cl(V.fib*1.6-f.d*.6,0,1),px=f.a[0]+(f.b[0]-f.a[0])*g,py=f.a[1]+(f.b[1]-f.a[1])*g,pz=f.a[2]+(f.b[2]-f.a[2])*g,mxx=(f.a[0]+px)/2+f.m[0]*g,myy=(f.a[1]+py)/2+f.m[1]*g,mzz=(f.a[2]+pz)/2+f.m[2]*g,o=i*12;
fp[o]=f.a[0];fp[o+1]=f.a[1];fp[o+2]=f.a[2]+wd(f.a[1]);fp[o+3]=mxx;fp[o+4]=myy;fp[o+5]=mzz+wd(myy);fp[o+6]=mxx;fp[o+7]=myy;fp[o+8]=mzz+wd(myy);fp[o+9]=px;fp[o+10]=py;fp[o+11]=pz+wd(py)});fGeo.attributes.position.needsUpdate=true;
if(V.leak>.05&&V.rep<.5&&Math.random()<dt*V.leak*5){const p=lk.find(q=>!q.on);if(p){p.on=1;p.life=0;p.x=rnd(-1.1,1.1);p.y=rnd(-1,1);p.z=-.2;p.vx=rnd(-.3,.3);p.vy=rnd(-.3,.3);p.vz=rnd(1.2,2.2);p.r=rnd(0,6)}}
lk.forEach((p,i)=>{if(p.on){p.life+=dt;p.x+=p.vx*dt;p.y+=p.vy*dt;p.z+=p.vz*dt;if(p.life>3)p.on=0}const s=p.on?.34*(1-p.life/3)*cl(p.life*5,0,1):0;put(lkM,i,p.x,p.y,p.z+wd(p.y),p.r+time,p.r*2,p.r+time*.7,s,s*.85,s)});lkM.instanceMatrix.needsUpdate=true;
if(stage==3||stage==4){if(Math.random()<dt*14)emit(0)}if(stage==5&&V.fib>.05&&Math.random()<dt*8)emit(1);
pool.forEach((p,i)=>{if(p.on){p.life+=dt;p.x+=p.vx*dt;p.y+=p.vy*dt;p.z+=p.vz*dt;if(p.life>p.max||(p.kind==0&&Math.hypot(p.x/3.4,p.y/2.6)>(stage==4?1:2)))p.on=0}
const s=p.on?(p.kind?1.4*(1-p.life/p.max):cl(p.life*5,0,1)):0;put(fcM,i,p.x,p.y,p.z+wd(p.y),0,0,0,s,s,s)});fcM.instanceMatrix.needsUpdate=true;
for(let i=0;i<NA;i++){const a=time*.35+i*.628,y=R0*Math.sin(1.05*Math.sin(a)),s=V.inh*(1+.25*Math.sin(time*3+i));put(amM,i,3.4*Math.cos(a),y,R0*(Math.cos(1.05*Math.sin(a))-1)-.15+.1*Math.sin(time*2+i),time+i,time*.7,i,s,s,s)}amM.instanceMatrix.needsUpdate=true;
for(let i=0;i<NPm;i++){const y=1.5*Math.cos(time*1.1+i*.9);put(pmM,i,1.9*Math.sin(time*.9+i*1.3),y,-.7+.4*Math.sin(time*1.7+i)+wd(y),time,i,0,V.pm,V.pm,V.pm)}pmM.instanceMatrix.needsUpdate=true;
colM.material.opacity=V.coll;colM.visible=V.coll>.02;barM.material.opacity=V.inh*(.65+.3*Math.sin(time*3));barM.visible=V.inh>.02;
patch.visible=V.rep>.02;patch.scale.set(V.rep,V.rep,1);
}
function cam(){const az=azU+sway*.35*Math.sin(time*.22),r=12,tz=1.2;camera.position.set(r*Math.sin(az)*Math.cos(el),r*Math.sin(el),tz+r*Math.cos(az)*Math.cos(el));camera.lookAt(0,0,tz)}
let lx=0,ly=0;
cv.addEventListener('pointerdown',e=>{drag=1;lx=e.clientX;ly=e.clientY;cv.style.cursor='grabbing'});
addEventListener('pointerup',()=>{drag=0;idle=0;cv.style.cursor='grab'});
cv.addEventListener('pointermove',e=>{if(!drag)return;azU=cl(azU+(e.clientX-lx)*.01,-1.4,1.4);el=cl(el+(e.clientY-ly)*.005,-.2,.9);lx=e.clientX;ly=e.clientY;idle=0});
let last=performance.now();
function frame(now){const dt=Math.min(.05,(now-last)/1000);last=now;time+=dt*rm;t+=dt;if(!drag)idle+=dt;sway+=((drag||idle<3?0:1)-sway)*Math.min(1,dt);update(dt);cam();rd.render(scene,camera);requestAnimationFrame(frame)}
const dots=document.getElementById('dots'),playB=document.getElementById('play');
S.forEach((s,i)=>{const b=document.createElement('button');b.textContent=s[0].split('. ')[0];b.setAttribute('aria-label',s[0]);b.onclick=()=>{stop();go(i)};dots.appendChild(b)});
function go(n){stage=n;t=0;if(n<3)V.fib=0;if(n<2){V.keep=0;plug.forEach(q=>q.p=0)}if(n<5)V.rep=0;pool.forEach(p=>{if(p.kind==1)p.on=0});
document.getElementById('ttl').textContent=S[n][0];document.getElementById('txt').textContent=S[n][1];
[...dots.children].forEach((x,i)=>{if(i===n)x.setAttribute('aria-current','true');else x.removeAttribute('aria-current')})}
function stop(){clearInterval(tm);tm=null;playB.textContent='Lecture'}
function start(){if(stage===S.length-1)go(0);playB.textContent='Pause';tm=setInterval(()=>{if(stage>=S.length-1){stop();return}go(stage+1)},8000)}
playB.onclick=()=>tm?stop():start();
document.getElementById('prev').onclick=()=>{stop();go(Math.max(0,stage-1))};
document.getElementById('next').onclick=()=>{stop();go(Math.min(S.length-1,stage+1))};
go(0);requestAnimationFrame(frame);
</script>
</body>
</html>

