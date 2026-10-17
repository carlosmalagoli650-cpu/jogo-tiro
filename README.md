
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Jogo de Tiro</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#17251d;color:white;font-family:Arial;text-align:center}
header{padding:12px;background:#101812}
#stats{font-size:19px;line-height:1.8}
#arena{position:relative;height:65vh;min-height:350px;overflow:hidden;background:linear-gradient(#83b5cb 0%,#d1d7bd 65%,#657c40 65%,#3a5429 100%);border-top:4px solid #263c26;cursor:crosshair}
.target{position:absolute;width:65px;height:65px;border-radius:50%;background:radial-gradient(circle,#fff 0 15%,#e93636 16% 35%,#fff 36% 57%,#e93636 58%);border:3px solid #222;cursor:crosshair;box-shadow:0 4px 8px #0007}
#gun{font-size:75px;position:absolute;bottom:0;left:50%;transform:translateX(-50%);pointer-events:none}
#flash{position:absolute;bottom:80px;left:50%;font-size:50px;transform:translateX(-50%);display:none;pointer-events:none}
#msg{min-height:35px;font-size:22px;font-weight:bold;padding:5px}
button{padding:12px 20px;margin:5px;border:0;border-radius:8px;background:#ffd04a;color:#222;font-weight:bold;font-size:17px;cursor:pointer}
</style>
</head>
<body>
<header>
<h1>🎯 CAMPO DE TIRO</h1>
<div id="stats">
Pontos: <span id="score">0</span> |
Munição: <span id="ammo">12</span>/12 |
Alvos: <span id="left">10</span>
</div>
</header>

<div id="arena">
<div id="gun">🔫</div>
<div id="flash">💥</div>
</div>
<div id="msg">Clique em COMEÇAR!</div>
<button id="start">COMEÇAR</button>
<button id="reload">RECARREGAR (R)</button>
<button id="reset">REINICIAR</button>

<script>
const arena=document.getElementById('arena');
const scoreEl=document.getElementById('score');
const ammoEl=document.getElementById('ammo');
const leftEl=document.getElementById('left');
const msg=document.getElementById('msg');
const flash=document.getElementById('flash');

let score=0,ammo=12,left=10,playing=false,target=null,moveTimer=null;

function update(){
 scoreEl.textContent=score;
 ammoEl.textContent=ammo;
 leftEl.textContent=left;
}

function newTarget(){
 if(!playing)return;
 if(target)target.remove();

 if(left<=0){
  playing=false;
  clearInterval(moveTimer);
  msg.textContent='🏆 VOCÊ VENCEU! Pontuação: '+score;
  return;
 }

 target=document.createElement('button');
 target.className='target';
 target.setAttribute('aria-label','Alvo');
 arena.appendChild(target);
 moveTarget();

 target.addEventListener('click',e=>{
  e.stopPropagation();
  if(!playing)return;
  if(ammo<=0){
   msg.textContent='Sem munição! Aperte R.';
   return;
  }

  ammo--;
  score+=10;
  left--;
  update();
  msg.textContent='🎯 ACERTOU! +10 PONTOS';
  burst(target.offsetLeft+32,target.offsetTop+32);
  target.remove();
  target=null;
  setTimeout(newTarget,250);
 });
}

function moveTarget(){
 if(!target)return;
 const maxX=Math.max(0,arena.clientWidth-target.offsetWidth);
 const maxY=Math.max(0,arena.clientHeight-target.offsetHeight-90);
 target.style.left=Math.floor(Math.random()*maxX)+'px';
 target.style.top=Math.floor(Math.random()*maxY)+'px';
}

function burst(x,y){
 const effect=document.createElement('div');
 effect.textContent='✨💥';
 effect.style.cssText='position:absolute;left:'+x+'px;top:'+y+'px;font-size:32px;pointer-events:none;z-index:4';
 arena.appendChild(effect);
 setTimeout(()=>effect.remove(),350);
}

arena.addEventListener('click',e=>{
 if(!playing)return;
 if(e.target===arena){
  if(ammo<=0){
   msg.textContent='Sem munição! Aperte R.';
   return;
  }
  ammo--;
  update();
  msg.textContent='💨 Você errou!';
  flash.style.display='block';
  setTimeout(()=>flash.style.display='none',100);
 }
});

function startGame(){
 clearInterval(moveTimer);
 if(target)target.remove();
 target=null;
 score=0;ammo=12;left=10;playing=true;
 update();
 msg.textContent='Mire nos alvos e clique neles!';
 newTarget();
 moveTimer=setInterval(()=>{
  if(playing&&target)moveTarget();
 },1200);
}

function reload(){
 if(!playing)return;
 ammo=12;
 update();
 msg.textContent='🔄 Arma recarregada!';
}

document.getElementById('start').addEventListener('click',startGame);
document.getElementById('reload').addEventListener('click',reload);
document.getElementById('reset').addEventListener('click',startGame);

document.addEventListener('keydown',e=>{
 if(e.key.toLowerCase()==='r')reload();
});

window.addEventListener('resize',()=>{
 if(target)moveTarget();
});

update();
</script>
</body>
</html>
