
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>FPS 3D - Arena de Tiro</title>
<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;background:#111;color:white;font-family:Arial}
#hud{position:fixed;inset:0;pointer-events:none;text-shadow:1px 2px 3px black}
#info{position:absolute;top:15px;left:15px;font-size:19px;line-height:1.7}
#cross{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);font-size:27px}
#hint{position:absolute;bottom:15px;width:100%;text-align:center;font-size:14px}
#start{position:fixed;inset:0;z-index:5;display:flex;align-items:center;justify-content:center;background:#000b;text-align:center}
.card{background:#202b31;padding:28px;border:2px solid #748e9b;max-width:500px}
button{pointer-events:auto;padding:13px 20px;background:#279b50;border:0;border-radius:7px;color:white;font-weight:bold;font-size:17px;cursor:pointer}
#fire{position:fixed;right:25px;bottom:55px;z-index:2}
</style>
</head>
<body>
<div id="hud">
 <div id="info">🔫 ARENA 3D<br>Vida: <span id="life">100</span><br>Munição: <span id="ammo">30</span><br>Pontos: <span id="points">0</span></div>
 <div id="cross">+</div>
 <div id="hint">WASD: andar • Mouse: mirar • Clique: atirar • Espaço: pular • Shift: correr • R: recarregar</div>
</div>
<button id="fire" onclick="shoot()">ATIRAR</button>
<div id="start"><div class="card">
<h1>🔫 ARENA DE TIRO 3D</h1>
<p>Derrote os robôs inimigos e explore o mapa.</p>
<p>WASD para andar, mouse para olhar e clique para atirar.</p>
<button id="play">JOGAR</button>
</div></div>
<div id="game"></div>
<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x9bc4df);
scene.fog=new THREE.Fog(0x9bc4df,25,75);

const camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,.1,120);
camera.rotation.order='YXZ';

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,1.5));
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;
document.getElementById('game').appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x596451,2));
const sun=new THREE.DirectionalLight(0xffffff,2.5);
sun.position.set(-10,20,10);sun.castShadow=true;scene.add(sun);

const mat=(c)=>new THREE.MeshStandardMaterial({color:c,roughness:.85});
function cube(x,y,z,w,h,d,color){
 const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),mat(color));
 m.position.set(x,y,z);m.castShadow=true;m.receiveShadow=true;scene.add(m);return m;
}

// Chão e paredes
cube(0,-.3,0,70,.5,70,0x62754d);
cube(0,2,-35,70,4,1,0x667078);
cube(0,2,35,70,4,1,0x667078);
cube(-35,2,0,1,4,70,0x667078);
cube(35,2,0,1,4,70,0x667078);

// Obstáculos e caixas para cobertura
for(const p of [
 [-8,0,-8],[7,0,-9],[0,0,-15],[-14,0,3],
 [13,0,4],[-4,0,10],[8,0,14],[18,0,-18],
 [-19,0,-15],[20,0,10],[-15,0,19],[2,0,23]
]){
 cube(p[0],1,p[2],2.5,2,2.5,0x74604b);
}
// Pilares coloridos do mapa
for(let i=0;i<8;i++){
 const x=(i%4)*12-18,z=Math.floor(i/4)*24-12;
 cube(x,1.5,z,1.2,3,1.2,0x8d9ba0);
}

// Arma decorativa na tela
const weapon=new THREE.Group();
const gunmat=mat(0x292d33);
const barrel=new THREE.Mesh(new THREE.BoxGeometry(.16,.16,.8),gunmat);
barrel.position.set(.36,-.3,-.65);weapon.add(barrel);
const grip=new THREE.Mesh(new THREE.BoxGeometry(.13,.28,.18),mat(0x111518));
grip.position.set(.35,-.48,-.38);weapon.add(grip);
camera.add(weapon);scene.add(camera);
weapon.position.set(0,0,0);

// Player
const player={x:0,z:16,y:1.7,vy:0,onGround:true};
let yaw=0,pitch=0,locked=false,life=100,ammo=30,points=0;
const keys={};
const lifeEl=document.getElementById('life');
const ammoEl=document.getElementById('ammo');
const pointsEl=document.getElementById('points');
let gameOver=false,started=false;

document.getElementById('play').onclick=()=>{
 started=true;renderer.domElement.requestPointerLock();
};
document.addEventListener('pointerlockchange',()=>{
 locked=document.pointerLockElement===renderer.domElement;
 if(locked)document.getElementById('start').style.display='none';
 else if(started&&!gameOver)document.getElementById('start').style.display='flex';
});
document.addEventListener('mousemove',e=>{
 if(!locked)return;
 yaw-=e.movementX*.002;
 pitch-=e.movementY*.002;
 pitch=Math.max(-1.45,Math.min(1.45,pitch));
 camera.rotation.set(pitch,yaw,0,'YXZ');
});
document.addEventListener('keydown',e=>{
 keys[e.code]=true;
 if(e.code==='Space')e.preventDefault();
 if(e.code==='KeyR')reload();
});
document.addEventListener('keyup',e=>keys[e.code]=false);
document.addEventListener('mousedown',e=>{if(e.button===0)shoot()});

function reload(){
 if(ammo<30){
  ammo=30;ammoEl.textContent=ammo;
 }
}
window.shoot=shoot;
function shoot(){
 if(!locked||gameOver||!started)return;
 if(ammo<=0){reload();return}
 ammo--;ammoEl.textContent=ammo;
 weapon.position.z=.1;
 setTimeout(()=>weapon.position.z=0,70);

 const ray=new THREE.Raycaster();
 ray.setFromCamera(new THREE.Vector2(0,0),camera);
 ray.far=45;
 const hits=ray.intersectObjects(enemies.map(e=>e.group),true);
 if(hits.length){
  let obj=hits[0].object;
  while(obj.parent&&!enemies.some(e=>e.group===obj))obj=obj.parent;
  const enemy=enemies.find(e=>e.group===obj);
  if(enemy){
   enemy.hp--;
   enemy.group.children.forEach(m=>{if(m.material&&m.material.color)m.material.color.setHex(0xff3333)});
   setTimeout(()=>{if(enemy.group.parent)enemy.group.children.forEach(m=>{if(m.material&&m.material.color)m.material.color.setHex(0xb63838)})},100);
   if(enemy.hp<=0){
    scene.remove(enemy.group);
    enemies.splice(enemies.indexOf(enemy),1);
    points+=100;pointsEl.textContent=points;
    spawnEnemy();
   }
  }
 }
}
function makeEnemy(x,z){
 const g=new THREE.Group();
 const red=mat(0xb63838),dark=mat(0x282b30),eye=mat(0xffff55);
 function part(w,h,d,material,px,py,pz){
  const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),material);
  m.position.set(px,py,pz);m.castShadow=true;g.add(m);return m;
 }
 part(.8,1,.45,red,0,1,0);
 part(.65,.6,.6,dark,0,1.8,0);
 part(.13,.13,.08,eye,-.16,1.85,.32);
 part(.13,.13,.08,eye,.16,1.85,.32);
 part(.2,.7,.25,dark,-.55,1,0);
 part(.2,.7,.25,dark,.55,1,0);
 part(.24,.65,.25,dark,-.22,.35,0);
 part(.24,.65,.25,dark,.22,.35,0);
 g.position.set(x,0,z);scene.add(g);
 return {group:g,hp:3,attack:0};
}
const enemies=[];
function spawnEnemy(){
 if(gameOver)return;
 let x,z;
 do{x=Math.random()*46-23;z=Math.random()*46-23}
 while(Math.hypot(x-player.x,z-player.z)<12);
 enemies.push(makeEnemy(x,z));
}
for(let i=0;i<7;i++)spawnEnemy();

function updateEnemies(dt){
 for(const e of enemies){
  const dx=player.x-e.group.position.x,dz=player.z-e.group.position.z;
  const d=Math.hypot(dx,dz);
  if(d>1.8){
   e.group.position.x+=dx/d*dt*.8;
   e.group.position.z+=dz/d*dt*.8;
   e.group.rotation.y=Math.atan2(dx,dz);
  }else{
   e.attack+=dt;
   if(e.attack>1.2){
    e.attack=0;life=Math.max(0,life-8);lifeEl.textContent=life;
    if(life<=0)endGame();
   }
  }
 }
}
function endGame(){
 gameOver=true;document.exitPointerLock();
 document.getElementById('start').style.display='flex';
 document.querySelector('#start h1').textContent='VOCÊ FOI DERROTADO!';
 document.querySelector('#start p').textContent='Pontos: '+points;
 document.getElementById('play').textContent='JOGAR NOVAMENTE';
 document.getElementById('play').onclick=()=>location.reload();
}
let last=performance.now();
function animate(now){
 requestAnimationFrame(animate);
 const dt=Math.min((now-last)/1000,.04);last=now;
 if(locked&&!gameOver){
  const forward=new THREE.Vector3(-Math.sin(yaw),0,-Math.cos(yaw));
  const right=new THREE.Vector3(Math.cos(yaw),0,-Math.sin(yaw));
  const dir=new THREE.Vector3();
  if(keys.KeyW)dir.add(forward);
  if(keys.KeyS)dir.sub(forward);
  if(keys.KeyD)dir.add(right);
  if(keys.KeyA)dir.sub(right);
  if(dir.lengthSq()){
   dir.normalize();
   const speed=keys.ShiftLeft?9:5;
   const nx=player.x+dir.x*speed*dt;
   const nz=player.z+dir.z*speed*dt;
   // Limites do mapa
   if(Math.abs(nx)<32)player.x=nx;
   if(Math.abs(nz)<32)player.z=nz;
  }
  if(keys.Space&&player.onGround){player.vy=6;player.onGround=false}
  player.vy-=16*dt;player.y+=player.vy*dt;
  if(player.y<1.7){player.y=1.7;player.vy=0;player.onGround=true}
  camera.position.set(player.x,player.y,player.z);
  updateEnemies(dt);
 }
 renderer.render(scene,camera);
}
requestAnimationFrame(animate);
addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>
