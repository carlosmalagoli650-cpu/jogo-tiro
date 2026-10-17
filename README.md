
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Zumbi 3D</title>
<style>
body{margin:0;overflow:hidden;font-family:Arial;background:#222;color:white}
#hud{position:fixed;top:10px;left:10px;background:#000c;padding:12px;line-height:1.8;z-index:2}
#cross{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);font-size:32px;color:white;z-index:3;text-shadow:0 0 5px red;pointer-events:none}
#msg{position:fixed;top:22%;left:0;width:100%;text-align:center;font-size:24px;text-shadow:2px 2px black;z-index:3;pointer-events:none}
#startscreen{position:fixed;inset:0;background:#101510eF;display:flex;justify-content:center;align-items:center;text-align:center;z-index:10}
#panel{padding:25px;background:#273528;border:2px solid #7a9b58;border-radius:14px;max-width:430px}
button{padding:14px 24px;background:#9be34d;border:0;border-radius:8px;font-size:18px;font-weight:bold;cursor:pointer}
#help{position:fixed;bottom:8px;left:0;width:100%;text-align:center;background:#0009;padding:8px 0;font-size:13px;z-index:2}
</style>
</head>
<body>
<div id="hud">
<b>🧟 SOBREVIVÊNCIA</b><br>
❤️ Vida: <span id="life">100</span><br>
🔫 Munição: <span id="ammo">12</span><br>
🏆 Pontos: <span id="score">0</span><br>
Zumbis: <span id="num">6</span>
</div>
<div id="cross">+</div>
<div id="msg"></div>
<div id="help">WASD andar | Mouse mirar | Clique atirar | R recarregar | Shift correr</div>
<div id="startscreen"><div id="panel">
<h1>🧟 ZUMBI 3D</h1>
<p>Sobreviva ao ataque de zumbis!</p>
<p>Mire pelo centro da tela e clique para atirar.</p>
<button id="play">COMEÇAR</button>
<p>Use o Chrome com internet ligada.</p>
</div></div>

<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x68766b);
scene.fog=new THREE.Fog(0x68766b,60,180);

const camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,.1,500);
camera.rotation.order='YXZ';
camera.position.set(0,1.7,12);

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x48513e,2));
const sun=new THREE.DirectionalLight(0xffffff,2);
sun.position.set(10,30,20);scene.add(sun);

function mat(c){return new THREE.MeshStandardMaterial({color:c});}
function cube(x,y,z,w,h,d,color){
 const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),mat(color));
 m.position.set(x,y,z);scene.add(m);return m;
}

const floor=new THREE.Mesh(new THREE.PlaneGeometry(200,200),mat(0x4b5940));
floor.rotation.x=-Math.PI/2;scene.add(floor);

// Árvores e casas para o cenário
for(let i=0;i<16;i++){
 const x=(Math.random()-.5)*100,z=(Math.random()-.5)*100;
 if(Math.abs(x)<12&&Math.abs(z)<20)continue;
 cube(x,1,z,2,2,2,0x574b3e);
 const leaves=new THREE.Mesh(new THREE.ConeGeometry(2.5,5,7),mat(0x24472a));
 leaves.position.set(x,4,z);scene.add(leaves);
}

// Arma desenhada em frente à câmera
const gun=new THREE.Group();
const gunMaterial=mat(0x252a30);
const barrel=new THREE.Mesh(new THREE.BoxGeometry(.16,.16,.85),gunMaterial);
barrel.position.set(.32,-.24,-.65);gun.add(barrel);
const gunbody=new THREE.Mesh(new THREE.BoxGeometry(.3,.24,.48),gunMaterial);
gunbody.position.set(.3,-.33,-.35);gun.add(gunbody);
const handle=new THREE.Mesh(new THREE.BoxGeometry(.13,.28,.17),mat(0x594334));
handle.position.set(.3,-.52,-.3);gun.add(handle);
camera.add(gun);scene.add(camera);

// Zumbis verdes, construídos com peças 3D
const zombies=[];
function createZombie(x,z){
 const zombie=new THREE.Group();
 zombie.position.set(x,0,z);
 const skin=mat(0x59b52f);
 const shirt=mat(0x4c342a);
 const pants=mat(0x303c43);
 const eyes=mat(0xff1818);

 const body=new THREE.Mesh(new THREE.BoxGeometry(.9,1.05,.5),shirt);
 body.position.y=1.25;zombie.add(body);

 const head=new THREE.Mesh(new THREE.BoxGeometry(.72,.7,.65),skin);
 head.position.y=2.15;zombie.add(head);

 for(const ex of [-.19,.19]){
  const eye=new THREE.Mesh(new THREE.BoxGeometry(.14,.13,.06),eyes);
  eye.position.set(ex,2.2,-.34);zombie.add(eye);
 }
 const mouth=new THREE.Mesh(new THREE.BoxGeometry(.3,.09,.05),mat(0x251515));
 mouth.position.set(0,1.98,-.35);zombie.add(mouth);

 for(const side of [-1,1]){
  const arm=new THREE.Mesh(new THREE.BoxGeometry(.23,.9,.25),skin);
  arm.position.set(side*.58,1.35,-.12);
  arm.rotation.z=side*-.18;zombie.add(arm);
  const leg=new THREE.Mesh(new THREE.BoxGeometry(.28,.75,.3),pants);
  leg.position.set(side*.24,.38,0);zombie.add(leg);
 }

 zombie.traverse(o=>{if(o.isMesh)o.userData.zombie=zombie;});
 zombie.userData.hp=2;
 zombie.userData.attack=0;
 scene.add(zombie);
 zombies.push(zombie);
}

// Coloca os zumbis à frente do jogador para que apareçam logo no início
createZombie(-4,-12);
createZombie(0,-17);
createZombie(4,-14);
createZombie(-8,-24);
createZombie(8,-27);
createZombie(2,-34);

let life=100,ammo=12,score=0,playing=false,ended=false,reloading=false;
const $=id=>document.getElementById(id);
const keys={};
function updateHUD(){
 $('life').textContent=Math.max(0,Math.ceil(life));
 $('ammo').textContent=ammo;
 $('score').textContent=score;
 $('num').textContent=zombies.length;
}
let messageTimer;
function message(t){
 $('msg').textContent=t;
 clearTimeout(messageTimer);
 messageTimer=setTimeout(()=>$('msg').textContent='',1000);
}

const raycaster=new THREE.Raycaster();
const center=new THREE.Vector2(0,0);

// O tiro vai pelo centro da tela e alcança até 1000 unidades.
function shoot(){
 if(!playing||ended||reloading)return;
 if(ammo<=0){message('SEM MUNIÇÃO! Aperte R');return;}
 ammo--;updateHUD();

 raycaster.setFromCamera(center,camera);
 raycaster.far=1000;

 const parts=[];
 for(const z of zombies){
  z.traverse(o=>{if(o.isMesh)parts.push(o);});
 }
 const hits=raycaster.intersectObjects(parts,false);

 if(hits.length){
  const z=hits[0].object.userData.zombie;
  if(z&&zombies.includes(z)){
   z.userData.hp--;
   if(z.userData.hp<=0){
    scene.remove(z);
    zombies.splice(zombies.indexOf(z),1);
    score+=100;
    message('ZUMBI ELIMINADO +100');
   }else{
    message('ACERTOU O ZUMBI!');
   }
   updateHUD();
   if(zombies.length===0)finish(true);
  }
 }else{
  message('Mire diretamente no zumbi');
 }
 gun.position.z=.08;
 setTimeout(()=>gun.position.z=0,80);
}

function reload(){
 if(!playing||reloading||ammo===12||ended)return;
 reloading=true;message('RECARREGANDO...');
 setTimeout(()=>{ammo=12;reloading=false;updateHUD();message('ARMA CARREGADA');},1000);
}

$('play').onclick=()=>renderer.domElement.requestPointerLock();

document.addEventListener('pointerlockchange',()=>{
 playing=document.pointerLockElement===renderer.domElement&&!ended;
 $('startscreen').style.display=playing?'none':(ended?'flex':'flex');
});

document.addEventListener('mousemove',e=>{
 if(!playing)return;
 camera.rotation.y-=e.movementX*.0022;
 camera.rotation.x-=e.movementY*.0022;
 camera.rotation.x=THREE.MathUtils.clamp(camera.rotation.x,-1.4,1.4);
});
document.addEventListener('mousedown',e=>{if(e.button===0)shoot();});
document.addEventListener('keydown',e=>{
 keys[e.code]=true;
 if(e.code==='KeyR')reload();
});
document.addEventListener('keyup',e=>keys[e.code]=false);
document.addEventListener('contextmenu',e=>e.preventDefault());

function finish(win){
 ended=true;playing=false;
 document.exitPointerLock();
 $('startscreen').style.display='flex';
 $('panel').innerHTML='<h1>'+(win?'VOCÊ VENCEU!':'FIM DE JOGO')+'</h1>'+
 '<p>Pontuação: '+score+'</p><button onclick="location.reload()">JOGAR NOVAMENTE</button>';
}

let last=performance.now();
function animate(now){
 requestAnimationFrame(animate);
 const dt=Math.min((now-last)/1000,.05);last=now;

 if(playing&&!ended){
  const f=new THREE.Vector3();
  camera.getWorldDirection(f);f.y=0;f.normalize();
  const r=new THREE.Vector3().crossVectors(f,new THREE.Vector3(0,1,0)).negate().normalize();
  const move=new THREE.Vector3();

  if(keys.KeyW)move.add(f);
  if(keys.KeyS)move.sub(f);
  if(keys.KeyA)move.sub(r);
  if(keys.KeyD)move.add(r);

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
   if(d>2){
    z.position.x+=dx/d*1.3*dt;
    z.position.z+=dz/d*1.3*dt;
    z.rotation.y=Math.atan2(dx,dz);
   }else{
    z.userData.attack+=dt;
    if(z.userData.attack>1){
     life-=8;z.userData.attack=0;updateHUD();
     if(life<=0){life=0;updateHUD();finish(false);break;}
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
