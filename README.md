
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Tiro 3D</title>
<style>
body{margin:0;overflow:hidden;font-family:Arial;background:#111;color:white}
#hud{position:fixed;top:12px;left:12px;background:#000b;padding:12px;line-height:1.8;z-index:2}
#mirar{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);font-size:30px;z-index:2;pointer-events:none}
#msg{position:fixed;top:25%;width:100%;text-align:center;font-size:25px;z-index:2;pointer-events:none}
#inicio{position:fixed;inset:0;background:#000d;display:flex;align-items:center;justify-content:center;text-align:center;z-index:5}
#caixa{background:#202b3a;padding:28px;border-radius:15px;max-width:440px}
button{padding:14px 24px;font-size:18px;cursor:pointer;background:#36adff;border:0;border-radius:8px;font-weight:bold}
#instrucoes{position:fixed;bottom:10px;width:100%;text-align:center;background:#0009;padding:8px 0;font-size:13px}
</style>
</head>
<body>
<div id="hud">
❤️ Vida: <span id="vida">100</span><br>
🔫 Munição: <span id="municao">30</span><br>
⭐ Pontos: <span id="pontos">0</span><br>
🤖 Robôs: <span id="robos">8</span>
</div>
<div id="mirar">+</div>
<div id="msg"></div>
<div id="instrucoes">WASD andar | Shift correr | Espaço pular | Mouse mirar | Clique atirar | R recarregar</div>
<div id="inicio"><div id="caixa">
<h1>TIRO 3D</h1>
<p>Elimine os 8 robôs! Agora você pode acertar inimigos de muito longe.</p>
<p>Use o mouse para mirar e clique para atirar.</p>
<button id="jogar">JOGAR</button>
</div></div>

<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x87b9df);
scene.fog=new THREE.Fog(0x87b9df,100,260);

const camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,.1,1500);
camera.rotation.order='YXZ';
camera.position.set(0,1.7,15);

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x526044,2));
const sun=new THREE.DirectionalLight(0xffffff,2);
sun.position.set(20,40,20);
scene.add(sun);

const material=c=>new THREE.MeshStandardMaterial({color:c});
const ground=new THREE.Mesh(new THREE.PlaneGeometry(220,220),material(0x526b45));
ground.rotation.x=-Math.PI/2;
scene.add(ground);

function block(x,y,z,w,h,d,color){
 const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),material(color));
 m.position.set(x,y,z);scene.add(m);return m;
}

// Muros da arena
block(0,3,-55,112,6,2,0x69747e);
block(0,3,55,112,6,2,0x69747e);
block(-55,3,0,2,6,112,0x69747e);
block(55,3,0,2,6,112,0x69747e);

// Caixas para cobertura
[[-12,-10],[12,-15],[23,8],[-23,18],[0,-32],[-33,-7],[32,30]]
.forEach(([x,z])=>block(x,1.2,z,3,2.4,3,0x92704a));

// Arma simples visível
const gun=new THREE.Group();
const metal=material(0x252b32);
const cano=new THREE.Mesh(new THREE.BoxGeometry(.15,.15,.8),metal);
cano.position.set(.3,-.25,-.7);
gun.add(cano);
const corpo=new THREE.Mesh(new THREE.BoxGeometry(.28,.23,.4),metal);
corpo.position.set(.3,-.32,-.35);
gun.add(corpo);
camera.add(gun);
scene.add(camera);

const enemies=[];
function criarRobo(x,z){
 const root=new THREE.Group();
 root.position.set(x,0,z);
 const torso=new THREE.Mesh(new THREE.BoxGeometry(1,1,.55),material(0xc73333));
 torso.position.y=1.3;root.add(torso);
 const head=new THREE.Mesh(new THREE.BoxGeometry(.7,.65,.6),material(0x303945));
 head.position.y=2.15;root.add(head);

 const eyesMat=material(0xffff55);
 for(const xeye of [-.18,.18]){
  const eye=new THREE.Mesh(new THREE.BoxGeometry(.13,.1,.04),eyesMat);
  eye.position.set(xeye,2.2,-.32);root.add(eye);
 }
 for(const side of [-1,1]){
  const arm=new THREE.Mesh(new THREE.BoxGeometry(.23,.8,.25),material(0x303945));
  arm.position.set(side*.65,1.3,0);root.add(arm);
  const leg=new THREE.Mesh(new THREE.BoxGeometry(.27,.75,.3),material(0x303945));
  leg.position.set(side*.25,.38,0);root.add(leg);
 }

 // Cada peça aponta para o robô que a contém
 root.traverse(obj=>{
  if(obj.isMesh)obj.userData.robo=root;
 });
 root.userData.vida=3;
 root.userData.ataque=0;
 scene.add(root);
 enemies.push(root);
}

for(let i=0;i<8;i++){
 let x,z;
 do{
  x=(Math.random()-.5)*82;
  z=-15-Math.random()*65;
 }while(Math.hypot(x,z-15)<22);
 criarRobo(x,z);
}

let vida=100,municao=30,pontos=0;
let jogando=false,acabou=false,recarregando=false;
const keys={};
const $=id=>document.getElementById(id);
function atualizar(){
 $('vida').textContent=Math.max(0,Math.ceil(vida));
 $('municao').textContent=municao;
 $('pontos').textContent=pontos;
 $('robos').textContent=enemies.length;
}
let msgTimer;
function mensagem(text){
 $('msg').textContent=text;
 clearTimeout(msgTimer);
 msgTimer=setTimeout(()=>$('msg').textContent='',850);
}

// CORREÇÃO DO TIRO:
// O raio sai do centro da câmera e pode alcançar até 1000 unidades.
const raycaster=new THREE.Raycaster();
raycaster.near=0;
raycaster.far=1000;
const centro=new THREE.Vector2(0,0);

function atirar(){
 if(!jogando||acabou||recarregando)return;
 if(municao<=0){mensagem('Sem munição! Aperte R');return;}

 municao--;
 atualizar();

 raycaster.setFromCamera(centro,camera);
 raycaster.far=1000;

 // Reúne todas as partes de todos os robôs
 const pecas=[];
 for(const robo of enemies){
  robo.traverse(obj=>{
   if(obj.isMesh)pecas.push(obj);
  });
 }

 // Detecta o primeiro robô atingido, mesmo a grande distância
 const impactos=raycaster.intersectObjects(pecas,false);

 if(impactos.length>0){
  const robo=impactos[0].object.userData.robo;
  if(robo&&enemies.includes(robo)){
   robo.userData.vida--;
   if(robo.userData.vida<=0){
    scene.remove(robo);
    enemies.splice(enemies.indexOf(robo),1);
    pontos+=100;
    mensagem('ROBÔ ELIMINADO! +100');
   }else{
    mensagem('ACERTOU! Faltam '+robo.userData.vida+' tiros');
   }
   atualizar();
   if(enemies.length===0)fim(true);
  }
 }else{
  mensagem('TIROU — mire no robô!');
 }

 gun.position.z=.06;
 setTimeout(()=>gun.position.z=0,70);
}

function recarregar(){
 if(!jogando||recarregando||municao===30||acabou)return;
 recarregando=true;
 mensagem('Recarregando...');
 setTimeout(()=>{
  municao=30;
  recarregando=false;
  atualizar();
  mensagem('Arma recarregada!');
 },900);
}

function fim(venceu){
 acabou=true;
 document.exitPointerLock();
 $('inicio').style.display='flex';
 $('caixa').innerHTML=
 '<h1>'+(venceu?'VOCÊ VENCEU!':'FIM DE JOGO')+'</h1>'+
 '<p>Pontuação: '+pontos+'</p>'+
 '<button id="novamente">JOGAR NOVAMENTE</button>';
 $('novamente').onclick=()=>location.reload();
}

$('jogar').onclick=()=>renderer.domElement.requestPointerLock();

document.addEventListener('pointerlockchange',()=>{
 jogando=document.pointerLockElement===renderer.domElement&&!acabou;
 if(jogando)$('inicio').style.display='none';
 else if(!acabou)$('inicio').style.display='flex';
});

document.addEventListener('mousemove',e=>{
 if(!jogando)return;
 camera.rotation.y-=e.movementX*.0022;
 camera.rotation.x-=e.movementY*.0022;
 camera.rotation.x=THREE.MathUtils.clamp(camera.rotation.x,-1.45,1.45);
});

document.addEventListener('mousedown',e=>{
 if(e.button===0)atirar();
});
document.addEventListener('keydown',e=>{
 keys[e.code]=true;
 if(e.code==='KeyR')recarregar();
});
document.addEventListener('keyup',e=>keys[e.code]=false);
document.addEventListener('contextmenu',e=>e.preventDefault());

let anterior=performance.now();
function animar(agora){
 requestAnimationFrame(animar);
 const dt=Math.min((agora-anterior)/1000,.05);
 anterior=agora;

 if(jogando&&!acabou){
  const frente=new THREE.Vector3();
  camera.getWorldDirection(frente);
  frente.y=0;frente.normalize();
  const direita=new THREE.Vector3().crossVectors(frente,new THREE.Vector3(0,1,0)).negate().normalize();
  const movimento=new THREE.Vector3();

  if(keys.KeyW)movimento.add(frente);
  if(keys.KeyS)movimento.sub(frente);
  if(keys.KeyD)movimento.add(direita);
  if(keys.KeyA)movimento.sub(direita);

  if(movimento.lengthSq()>0){
   movimento.normalize().multiplyScalar((keys.ShiftLeft?12:7)*dt);
   camera.position.add(movimento);
  }

  camera.position.x=THREE.MathUtils.clamp(camera.position.x,-51,51);
  camera.position.z=THREE.MathUtils.clamp(camera.position.z,-51,51);

  camera.userData.velY=camera.userData.velY||0;
  if(keys.Space&&camera.position.y<=1.71)camera.userData.velY=6;
  camera.userData.velY-=18*dt;
  camera.position.y+=camera.userData.velY*dt;
  if(camera.position.y<1.7){
   camera.position.y=1.7;camera.userData.velY=0;
  }

  for(const robo of [...enemies]){
   const dx=camera.position.x-robo.position.x;
   const dz=camera.position.z-robo.position.z;
   const dist=Math.hypot(dx,dz);
   if(dist>2.2){
    robo.position.x+=dx/dist*1.5*dt;
    robo.position.z+=dz/dist*1.5*dt;
    robo.rotation.y=Math.atan2(dx,dz);
   }else{
    robo.userData.ataque+=dt;
    if(robo.userData.ataque>1){
     vida-=8;robo.userData.ataque=0;atualizar();
     if(vida<=0){vida=0;atualizar();fim(false);break;}
    }
   }
  }
 }
 renderer.render(scene,camera);
}
requestAnimationFrame(animar);

addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>
