
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Zumbi 3D - Sobrevivência</title>
<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;background:#080d0a;color:white;font-family:Arial}
#hud{position:fixed;top:15px;left:15px;z-index:3;font-size:18px;text-shadow:2px 2px #000}
#crosshair{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);font-size:28px;z-index:3}
#message{position:fixed;top:22%;width:100%;text-align:center;font-size:28px;font-weight:bold;color:#ffcc55;text-shadow:2px 2px #000;pointer-events:none}
#start{position:fixed;inset:0;z-index:5;background:#000c;display:flex;flex-direction:column;align-items:center;justify-content:center;text-align:center;padding:20px}
button{font-size:22px;padding:15px 30px;background:#b52121;color:white;border:0;border-radius:8px;cursor:pointer}
#flash{position:fixed;inset:0;z-index:2;pointer-events:none;background:radial-gradient(circle at 50% 50%,#fff8a0aa,#ff5c0088 12%,transparent 38%);opacity:0}
</style>
</head>
<body>
<div id="hud">❤️ Vida: <span id="life">100</span><br>
🔫 Munição: <span id="ammo">12</span> / 12<br>
💀 Pontos: <span id="score">0</span><br>
🧟 Zumbis: <span id="count">8</span></div>
<div id="crosshair">+</div>
<div id="message"></div>
<div id="flash"></div>
<div id="start">
<h1>🧟 ZUMBI 3D</h1>
<p>Elimine os zumbis! Cada um morre com 2 tiros.</p>
<p>WASD: andar | Mouse: mirar | Clique: atirar<br>R: recarregar | Shift: correr</p>
<button id="play">JOGAR</button>
</div>
<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87928a);
scene.fog = new THREE.Fog(0x87928a, 25, 100);

const camera = new THREE.PerspectiveCamera(75, innerWidth/innerHeight, 0.1, 200);
camera.position.set(0,1.7,8);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled=true;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xcbdcff,0x333322,2));
const sun = new THREE.DirectionalLight(0xffffff,2.5);
sun.position.set(10,20,8);
sun.castShadow=true;
scene.add(sun);

const ground = new THREE.Mesh(
 new THREE.PlaneGeometry(200,200),
 new THREE.MeshStandardMaterial({color:0x42483e})
);
ground.rotation.x=-Math.PI/2;
ground.receiveShadow=true;
scene.add(ground);

// Estrada e construções
const road = new THREE.Mesh(new THREE.PlaneGeometry(12,200),
 new THREE.MeshStandardMaterial({color:0x292d2c}));
road.rotation.x=-Math.PI/2;
road.position.y=.015;
scene.add(road);

for(let i=0;i<24;i++){
 for(const side of [-1,1]){
  const h=4+Math.random()*9;
  const building=new THREE.Mesh(
   new THREE.BoxGeometry(5,h,6+Math.random()*5),
   new THREE.MeshStandardMaterial({color:[0x55564c,0x393f3b,0x65584b][i%3]})
  );
  building.position.set(side*(10+Math.random()*7),h/2,-i*7+25);
  building.castShadow=true;
  scene.add(building);
 }
}

const gun = new THREE.Group();
const gunBody = new THREE.Mesh(new THREE.BoxGeometry(.24,.2,.65),
 new THREE.MeshStandardMaterial({color:0x20252a,metalness:.7,roughness:.3}));
gunBody.position.set(.35,-.27,-.65);
gun.add(gunBody);
const barrel = new THREE.Mesh(new THREE.CylinderGeometry(.045,.055,.35,12),
 new THREE.MeshStandardMaterial({color:0x111111,metalness:.8}));
barrel.rotation.x=Math.PI/2;
barrel.position.set(.35,-.23,-1.02);
gun.add(barrel);
const grip = new THREE.Mesh(new THREE.BoxGeometry(.12,.3,.16),
 new THREE.MeshStandardMaterial({color:0x161616}));
grip.position.set(.35,-.46,-.49);
gun.add(grip);
camera.add(gun);
scene.add(camera);

// Clarão da arma
const muzzle = new THREE.Group();
const flame = new THREE.Mesh(new THREE.ConeGeometry(.14,.48,12),
 new THREE.MeshBasicMaterial({color:0xffb52e}));
flame.rotation.x=-Math.PI/2;
flame.position.z=-.25;
muzzle.add(flame);
const innerFlame = new THREE.Mesh(new THREE.ConeGeometry(.075,.32,10),
 new THREE.MeshBasicMaterial({color:0xffffff}));
innerFlame.rotation.x=-Math.PI/2;
innerFlame.position.z=-.26;
muzzle.add(innerFlame);
muzzle.position.set(.35,-.23,-1.22);
muzzle.visible=false;
gun.add(muzzle);
const muzzleLight = new THREE.PointLight(0xff8b27,0,7);
muzzleLight.position.set(.35,-.23,-1.25);
gun.add(muzzleLight);

// Efeitos de bala e impacto
const effects=[];
function lineEffect(start,end,color,life=.09){
 const geometry=new THREE.BufferGeometry().setFromPoints([start,end]);
 const material=new THREE.LineBasicMaterial({color,transparent:true,opacity:1});
 const line=new THREE.Line(geometry,material);
 scene.add(line);
 effects.push({obj:line,mat:material,life,max:life});
}
function sparks(pos){
 for(let i=0;i<8;i++){
  const mesh=new THREE.Mesh(new THREE.SphereGeometry(.035,5,5),
   new THREE.MeshBasicMaterial({color:0xffbb55}));
  mesh.position.copy(pos);
  scene.add(mesh);
  const velocity=new THREE.Vector3((Math.random()-.5)*3,
   Math.random()*3,(Math.random()-.5)*3);
  effects.push({obj:mesh,life:.35,max:.35,velocity});
 }
}

const zombies=[];
function makeZombie(x,z){
 const root=new THREE.Group();
 root.position.set(x,0,z);
 const green=new THREE.MeshStandardMaterial({color:0x51844a,roughness:1});
 const dark=new THREE.MeshStandardMaterial({color:0x30372b});
 const shirt=new THREE.MeshStandardMaterial({color:0x4b5144});
 function part(geo,mat,pos){
  const m=new THREE.Mesh(geo,mat);
  m.position.set(...pos);m.castShadow=true;root.add(m);return m;
 }
 part(new THREE.BoxGeometry(.65,.85,.38),shirt,[0,1.2,0]);
 part(new THREE.SphereGeometry(.28,12,10),green,[0,1.88,0]);
 part(new THREE.BoxGeometry(.12,.12,.06),
  new THREE.MeshBasicMaterial({color:0xff2020}),[-.1,1.92,-.25]);
 part(new THREE.BoxGeometry(.12,.12,.06),
  new THREE.MeshBasicMaterial({color:0xff2020}),[.1,1.92,-.25]);
 part(new THREE.BoxGeometry(.18,.48,.18),green,[-.48,1.25,0]);
 part(new THREE.BoxGeometry(.18,.48,.18),green,[.48,1.25,0]);
 part(new THREE.BoxGeometry(.22,.65,.23),dark,[-.19,.45,0]);
 part(new THREE.BoxGeometry(.22,.65,.23),dark,[.19,.45,0]);
 root.traverse(o=>{if(o.isMesh)o.userData.zombie=root});
 scene.add(root);
 zombies.push({root,hp:2,speed:0.45+Math.random()*.3,attack:0});
}
for(let i=0;i<8;i++)makeZombie((Math.random()-.5)*18,-12-Math.random()*35);

let playing=false,life=100,ammo=12,score=0,reloading=false,ended=false;
const keys={};
const raycaster=new THREE.Raycaster();
const center=new THREE.Vector2(0,0);
const flash=document.getElementById('flash');
const messageEl=document.getElementById('message');
let messageTimer;
function message(t){
 messageEl.textContent=t;
 clearTimeout(messageTimer);
 messageTimer=setTimeout(()=>messageEl.textContent='',1100);
}
function hud(){
 document.getElementById('life').textContent=life;
 document.getElementById('ammo').textContent=ammo;
 document.getElementById('score').textContent=score;
 document.getElementById('count').textContent=zombies.length;
}
function finish(win){
 ended=true;playing=false;
 document.exitPointerLock?.();
 document.getElementById('start').style.display='flex';
 document.querySelector('#start h1').textContent=win?'🏆 VOCÊ VENCEU!':'💀 VOCÊ FOI DERROTADO';
 document.querySelector('#start p').textContent='Pontos: '+score;
 document.getElementById('play').textContent='JOGAR NOVAMENTE';
}

document.getElementById('play').onclick=()=>{
 if(ended)location.reload();
 playing=true;
 document.getElementById('start').style.display='none';
 renderer.domElement.requestPointerLock();
};
document.addEventListener('pointerlockchange',()=>{
 if(document.pointerLockElement!==renderer.domElement&&playing){
  playing=false;
  document.getElementById('start').style.display='flex';
  document.querySelector('#start h1').textContent='PAUSADO';
  document.querySelector('#start p').textContent='Clique em JOGAR para continuar.';
 }
});
document.addEventListener('keydown',e=>{
 keys[e.code]=true;
 if(e.code==='KeyR')reload();
});
document.addEventListener('keyup',e=>keys[e.code]=false);
document.addEventListener('mousemove',e=>{
 if(!playing||document.pointerLockElement!==renderer.domElement)return;
 camera.rotation.order='YXZ';
 camera.rotation.y-=e.movementX*.002;
 camera.rotation.x-=e.movementY*.002;
 camera.rotation.x=Math.max(-1.4,Math.min(1.4,camera.rotation.x));
});
document.addEventListener('mousedown',e=>{
 if(e.button===0&&playing)shoot();
});

function reload(){
 if(reloading||ammo===12||ended)return;
 reloading=true;message('RECARREGANDO...');
 setTimeout(()=>{ammo=12;reloading=false;hud();message('ARMA RECARREGADA');},1000);
}

function shoot(){
 if(!playing||ended||reloading)return;
 if(ammo<=0){message('SEM MUNIÇÃO! Aperte R');return}
 ammo--;hud();

 // Clarão rápido e recuo da arma
 muzzle.visible=true;muzzleLight.intensity=8;
 flash.style.opacity='1';
 gun.position.z=.12;
 setTimeout(()=>{muzzle.visible=false;muzzleLight.intensity=0;flash.style.opacity='0';gun.position.z=0;},75);

 // Raycast parte do centro da mira
 raycaster.setFromCamera(center,camera);
 raycaster.far=100;
 const parts=[];
 zombies.forEach(z=>{
  z.root.traverse(o=>{
   if(o.isMesh){o.userData.zombie=z;parts.push(o)}
  });
 });
 const hits=raycaster.intersectObjects(parts,false);
 const start=new THREE.Vector3();
 const end=new THREE.Vector3();
 camera.getWorldPosition(start);
 camera.getWorldDirection(end);
 end.multiplyScalar(80).add(start);
 if(hits.length){
  const hit=hits[0];
  const target=hit.point.clone();
  lineEffect(start,target,0xffdf8a,.1);
  sparks(target);
  const z=hit.object.userData.zombie;
  if(z&&zombies.includes(z)){
   z.hp--;
   if(z.hp===1){
    message('ACERTOU! FALTA 1 TIRO');
    hit.object.material=hit.object.material.clone();
    hit.object.material.color.set(0xb7a64a);
   }else{
    scene.remove(z.root);
    zombies.splice(zombies.indexOf(z),1);
    score+=100;message('💥 ZUMBI ELIMINADO +100');
    if(zombies.length===0)finish(true);
   }
  }
 }else{
  lineEffect(start,end,0xffd36b,.08);
 }
 hud();
}

let last=performance.now();
function animate(now){
 requestAnimationFrame(animate);
 const dt=Math.min((now-last)/1000,.05);last=now;
 if(playing&&!ended){
  const forward=new THREE.Vector3();
  camera.getWorldDirection(forward);forward.y=0;forward.normalize();
  const right=new THREE.Vector3().crossVectors(forward,new THREE.Vector3(0,1,0)).normalize();
  const speed=keys.ShiftLeft?9:5;
  if(keys.KeyW||keys.ArrowUp)camera.position.addScaledVector(forward,speed*dt);
  if(keys.KeyS||keys.ArrowDown)camera.position.addScaledVector(forward,-speed*dt);
  if(keys.KeyA||keys.ArrowLeft)camera.position.addScaledVector(right,-speed*dt);
  if(keys.KeyD||keys.ArrowRight)camera.position.addScaledVector(right,speed*dt);
  camera.position.y=1.7;
  camera.position.x=Math.max(-45,Math.min(45,camera.position.x));
  camera.position.z=Math.max(-90,Math.min(12,camera.position.z));

  zombies.forEach(z=>{
   const dir=new THREE.Vector3().subVectors(camera.position,z.root.position);
   dir.y=0;
   const dist=dir.length();
   if(dist>1.4){
    dir.normalize();
    z.root.position.addScaledVector(dir,z.speed*dt);
    z.root.rotation.y=Math.atan2(dir.x,dir.z);
   }else{
    z.attack+=dt;
    if(z.attack>1.2){
     life=Math.max(0,life-10);z.attack=0;hud();
     message('O ZUMBI TE ATACOU!');
     if(life<=0)finish(false);
    }
   }
  });
 }
 for(let i=effects.length-1;i>=0;i--){
  const e=effects[i];e.life-=dt;
  if(e.velocity)e.obj.position.addScaledVector(e.velocity,dt);
  if(e.mat)e.mat.opacity=Math.max(0,e.life/e.max);
  if(e.life<=0){
   scene.remove(e.obj);e.obj.geometry?.dispose();e.mat?.dispose();
   if(e.obj.material&&!e.mat)e.obj.material.dispose();
   effects.splice(i,1);
  }
 }
 renderer.render(scene,camera);
}
animate(performance.now());
addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
hud();
</script>
</body>
</html>
