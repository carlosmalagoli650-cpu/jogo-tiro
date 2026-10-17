
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Apocalipse Zumbi 3D</title>
<style>
body{margin:0;overflow:hidden;font-family:Arial;background:#111;color:white}
#hud{position:fixed;top:12px;left:12px;background:#000b;padding:13px;line-height:1.8;z-index:2}
#aim{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);font-size:30px;z-index:2;pointer-events:none}
#msg{position:fixed;top:25%;width:100%;text-align:center;font-size:25px;text-shadow:2px 2px #000;z-index:3}
#overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;background:#000d;text-align:center;z-index:5}
#panel{background:#202820;padding:28px;border:1px solid #75836b;border-radius:14px;max-width:450px}
button{padding:14px 25px;font-size:18px;background:#a6d94c;border:0;border-radius:8px;cursor:pointer;font-weight:bold}
#help{position:fixed;bottom:10px;width:100%;text-align:center;background:#0009;padding:8px 0;font-size:13px}
</style>
</head>
<body>
<div id="hud">
🧟 APOCALIPSE ZUMBI<br>
❤️ Vida: <span id="hp">100</span><br>
🔫 Munição: <span id="ammo">30</span>/30<br>
🏆 Pontos: <span id="score">0</span><br>
🧟 Zumbis: <span id="count">10</span>
</div>
<div id="aim">+</div>
<div id="msg"></div>
<div id="help">WASD: andar | Mouse: mirar | Clique: atirar | R: recarregar | Shift: correr | Espaço: pular</div>
<div id="overlay"><div id="panel">
<h1>🧟 APOCALIPSE ZUMBI</h1>
<p>Sobreviva ao ataque! Elimine todos os zumbis antes que eles acabem com você.</p>
<p>Use WASD para andar e o mouse para mirar.</p>
<button id="start">COMEÇAR JOGO</button>
<p style="font-size:12px">Requer internet para carregar os gráficos 3D.</p>
</div></div>

<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x777d78);
scene.fog=new THREE.Fog(0x777d78,35,130);

const camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,.1,500);
camera.rotation.order='YXZ';
camera.position.set(0,1.7,12);

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xdde8dc,0x34392e,2));
const light=new THREE.DirectionalLight(0xffffff,2);
light.position.set(20,40,10);scene.add(light);

const mat=c=>new THREE.MeshStandardMaterial({color:c});
const ground=new THREE.Mesh(new THREE.PlaneGeometry(180,180),mat(0x424b3d));
ground.rotation.x=-Math.PI/2;scene.add(ground);

function box(x,y,z,w,h,d,c){
 const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),mat(c));
 m.position.set(x,y,z);scene.add(m);return m;
}

// Ruas e prédios abandonados
for(let i=0;i<16;i++){
 let x=(i%4)*30-45,z=Math.floor(i/4)*-30-15;
 box(x,3,z,9,6,8,0x535b58);
 for(let y=0;y<2;y++)for(let k=-1;k<=1;k++){
  box(x+k*2,2+y*2,z+4.05,0.8,1,0.1,0x303e43);
 }
}
for(let i=0;i<20;i++){
 const x=(Math.random()-.5)*95,z=(Math.random()-.5)*95;
 const car=box(x,.65,z,2.3,1.2,4,Math.random()>.5?0x6c3535:0x343b49);
}
box(0,.08,0,180,.1,180,0x424b3d);

const gun=new THREE.Group();
const gunmat=mat(0x20252a);
const barrel=new THREE.Mesh(new THREE.BoxGeometry(.15,.15,.8),gunmat);
barrel.position.set(.3,-.25,-.7);gun.add(barrel);
const body=new THREE.Mesh(new THREE.BoxGeometry(.28,.23,.45),gunmat);
body.position.set(.3,-.33,-.35);gun.add(body);
camera.add(gun);scene.add(camera);

const zombies=[];
function makeZombie(x,z){
 const root=new THREE.Group();root.position.set(x,0,z);
 const skin=mat(0x64874d),shirt=mat(0x563c39),eyes=mat(0xff2424);
 const torso=new THREE.Mesh(new THREE.BoxGeometry(.8,1,.45),shirt);
 torso.position.y=1.25;root.add(torso);
 const head=new THREE.Mesh(new THREE.BoxGeometry(.62,.62,.6),skin);
 head.position.y=2.08;root.add(head);
 for(const ex of [-.16,.16]){
  const eye=new THREE.Mesh(new THREE.BoxGeometry(.12,.1,.04),eyes);
  eye.position.set(ex,2.13,-.31);root.add(eye);
 }
 for(const side of [-1,1]){
  const arm=new THREE.Mesh(new THREE.BoxGeometry(.2,.85,.22),skin);
  arm.position.set(side*.52,1.3,-.12);arm.rotation.z=side*-.2;root.add(arm);
  const leg=new THREE.Mesh(new THREE.BoxGeometry(.24,.7,.25),mat(0x343b34));
  leg.position.set(side*.22,.36,0);root.add(leg);
 }
 root.traverse(o=>{if(o.isMesh)o.userData.owner=root;});
 root.userData.hp=3;root.userData.attack=0;
 scene.add(root);zombies.push(root);
}
for(let i=0;i<10;i++){
 let x,z;
 do{x=(Math.random()-.5)*75;z=(Math.random()-.5)*75;}
 while(Math.hypot(x,z-12)<20);
 makeZombie(x,z);
}

let hp=100,ammo=30,score=0,locked=false,over=false,reloading=false;
const $=id=>document.getElementById(id);
function hud(){
 $('hp').textContent=Math.max(0,Math.ceil(hp));
 $('ammo').textContent=ammo;
 $('score').textContent=score;
 $('count').textContent=zombies.length;
}
let timer;
function message(t){
 $('msg').textContent=t;clearTimeout(timer);
 timer=setTimeout(()=>$('msg').textContent='',900);
}
const ray=new THREE.Raycaster();
ray.near=0;ray.far=1000;
const center=new THREE.Vector2(0,0);

// Tiro de longo alcance: detecta a peça atingida e identifica seu zumbi.
function shoot(){
 if(!locked||over||reloading)return;
 if(ammo<=0){message('Sem munição! Aperte R');return;}
 ammo--;hud();
 ray.setFromCamera(center,camera);
 ray.far=1000;
 const parts=[];
 for(const z of zombies)z.traverse(o=>{if(o.isMesh)parts.push(o);});
 const hits=ray.intersectObjects(parts,false);
 if(hits.length){
  const z=hits[0].object.userData.owner;
  if(z&&zombies.includes(z)){
   z.userData.hp--;
   if(z.userData.hp<=0){
    scene.remove(z);zombies.splice(zombies.indexOf(z),1);
    score+=100;message('ZUMBI ELIMINADO +100');
   }else message('ACERTOU! Mais '+z.userData.hp+' tiros');
   hud();
   if(zombies.length===0)finish(true);
  }
 }else message('Errou! Mire no zumbi.');
 gun.position.z=.06;
 setTimeout(()=>gun.position.z=0,70);
}

function reload(){
 if(!locked||reloading||ammo===30||over)return;
 reloading=true;message('Recarregando...');
 setTimeout(()=>{ammo=30;reloading=false;hud();message('Arma recarregada!');},1000);
}
const keys={};
document.addEventListener('keydown',e=>{
 keys[e.code]=true;
 if(e.code==='KeyR')reload();
});
document.addEventListener('keyup',e=>keys[e.code]=false);
document.addEventListener('mousedown',e=>{if(e.button===0)shoot();});
$('start').onclick=()=>renderer.domElement.requestPointerLock();

document.addEventListener('pointerlockchange',()=>{
 locked=document.pointerLockElement===renderer.domElement&&!over;
 if(locked)$('overlay').style.display='none';
 else if(!over)$('overlay').style.display='flex';
});
document.addEventListener('mousemove',e=>{
 if(!locked)return;
 camera.rotation.y-=e.movementX*.0022;
 camera.rotation.x-=e.movementY*.0022;
 camera.rotation.x=THREE.MathUtils.clamp(camera.rotation.x,-1.45,1.45);
});

function finish(win){
 over=true;document.exitPointerLock();
 $('overlay').style.display='flex';
 $('panel').innerHTML='<h1>'+(win?'VOCÊ SOBREVIVEU!':'VOCÊ FOI DERROTADO!')+
 '</h1><p>Pontuação: '+score+'</p><button onclick="location.reload()">JOGAR NOVAMENTE</button>';
}

let last=performance.now();
function animate(now){
 requestAnimationFrame(animate);
 const dt=Math.min((now-last)/1000,.05);last=now;

 if(locked&&!over){
  const f=new THREE.Vector3();
  camera.getWorldDirection(f);f.y=0;f.normalize();
  const r=new THREE.Vector3().crossVectors(f,new THREE.Vector3(0,1,0)).negate().normalize();
  const move=new THREE.Vector3();
  if(keys.KeyW)move.add(f);
  if(keys.KeyS)move.sub(f);
  if(keys.KeyD)move.add(r);
  if(keys.KeyA)move.sub(r);
  if(move.lengthSq()){
   move.normalize().multiplyScalar((keys.ShiftLeft?11:6)*dt);
   camera.position.add(move);
  }
  camera.position.x=THREE.MathUtils.clamp(camera.position.x,-48,48);
  camera.position.z=THREE.MathUtils.clamp(camera.position.z,-48,48);

  camera.userData.vy=camera.userData.vy||0;
  if(keys.Space&&camera.position.y<=1.71)camera.userData.vy=6;
  camera.userData.vy-=18*dt;
  camera.position.y+=camera.userData.vy*dt;
  if(camera.position.y<1.7){camera.position.y=1.7;camera.userData.vy=0;}

  for(const z of [...zombies]){
   const dx=camera.position.x-z.position.x;
   const dz=camera.position.z-z.position.z;
   const d=Math.hypot(dx,dz);
   if(d>1.8){
    z.position.x+=dx/d*1.5*dt;
    z.position.z+=dz/d*1.5*dt;
    z.rotation.y=Math.atan2(dx,dz);
   }else{
    z.userData.attack+=dt;
    if(z.userData.attack>1){
     hp-=10;z.userData.attack=0;hud();
     if(hp<=0){hp=0;hud();finish(false);break;}
    }
   }
  }
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
