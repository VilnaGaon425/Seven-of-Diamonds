<!DOCTYPE html>
<html lang="he">
<head>
<meta charset="UTF-8">
<title>Gematria Calculator</title>

<link href="https://fonts.googleapis.com/css2?family=Great+Vibes&display=swap" rel="stylesheet">

<style>
body{margin:0;overflow:hidden;background:#1e1e1e;display:flex;justify-content:center;align-items:center;height:100vh;direction:rtl;font-family:Arial}
canvas{position:fixed;top:0;left:0;z-index:0}

.calculator{position:relative;z-index:2;background:#2b2b2b;border-radius:20px;padding:30px;width:420px}

/* INTRO */
#introScreen{
position:fixed;top:0;left:0;width:100%;height:100%;
background:#1e1e1e;display:flex;justify-content:center;align-items:center;z-index:9999;
}

#introText{
color:white;font-size:56px;font-weight:bold;font-family:Arial;
white-space:pre;border-right:3px solid white;padding-right:8px;
animation:blink 1s infinite;letter-spacing:2px;
}

@keyframes blink{
0%{border-color:white}
50%{border-color:transparent}
100%{border-color:white}
}

.star{position:absolute;top:8px;left:10px;color:white;cursor:pointer}
.star.active{color:gold}
.settingsBtn{position:absolute;top:8px;right:10px;cursor:pointer}

.topnote{text-align:center;font-size:12px;color:#ccc}

.display{background:#000;color:#0f0;padding:20px;border-radius:10px;margin-bottom:10px;min-height:100px;display:flex;flex-direction:column;justify-content:space-between}
#text{font-size:32px;text-align:right;cursor:text}
#result{font-size:30px;text-align:left;margin-top:10px}

.buttons{display:grid;grid-template-columns:repeat(4,1fr);gap:10px}

button{
height:70px;border:none;border-radius:12px;background:#444;color:white;
font-size:20px;display:flex;align-items:center;justify-content:center;
}

.equal{background:#34c759}
.clear{background:#ff3b30}
.space{background:orange;color:transparent}
.toggle{background:#0a84ff;grid-column:3/span 2}
.return{background:#0a84ff;grid-column:span 3}

.full{grid-column:span 4;display:flex;justify-content:center;align-items:center}
.colorBox{height:40px}
</style>
</head>

<body>

<div id="introScreen"><div id="introText"></div></div>

<canvas id="bg"></canvas>

<div class="calculator">
<div class="star" onclick="toggleKatan()">★</div>
<div class="settingsBtn" onclick="toggleSettings()">⚙️</div>
<div class="topnote">בס״ד</div>

<div class="display">
<div id="text"></div>
<div id="result">0</div>
</div>

<div class="buttons" id="buttons"></div>
</div>

<script>
/* INTRO */
const introStr="A Kohn Enterprises production";
const introEl=document.getElementById("introText");
const introScreen=document.getElementById("introScreen");

let i=0,deleting=false;

function animateIntro(){
if(!deleting){
if(i<introStr.length){
introEl.innerText+=introStr[i++];
setTimeout(animateIntro,80);
}else{deleting=true;setTimeout(animateIntro,1400);}
}else{
if(i>0){
introEl.innerText=introStr.substring(0,--i);
setTimeout(animateIntro,40);
}else{
introScreen.style.opacity=0;
introScreen.style.transition="opacity 1s";
setTimeout(()=>introScreen.remove(),1000);
}
}}
animateIntro();

/* DOM */
const textEl=document.getElementById('text');
const resultEl=document.getElementById('result');
const buttonsEl=document.getElementById('buttons');

/* KEYBOARD */
document.addEventListener("keydown",e=>{
if(document.getElementById("introScreen")) return;

if(e.key==="Backspace"){currentText=currentText.slice(0,-1);updateDisplay();return;}
if(e.key===" "){add(" ");return;}
if(e.key==="Enter"){pressEquals();return;}
if(/[\u0590-\u05FF]/.test(e.key)){add(e.key);}
});

/* PASTE */
document.addEventListener("paste",e=>{
if(document.getElementById("introScreen")) return;

let paste=(e.clipboardData||window.clipboardData).getData("text");
paste=paste.replace(/[^\u0590-\u05FF\s]/g,"");
currentText+=paste;
updateDisplay();
});

/* CANVAS */
const canvas=document.getElementById('bg');
const ctx=canvas.getContext('2d');
canvas.width=innerWidth;
canvas.height=innerHeight;

let density=80,particlesOn=true;

const chars=['א','ב','ג','ד','ה','ו','ז','ח','ט','י','1','2','3','4','5','6','7','8','9'];
let particles=[];

function createParticles(){
particles=[];
for(let i=0;i<density;i++){
particles.push({
x:Math.random()*canvas.width,
y:Math.random()*canvas.height,
s:Math.random()*20+10,
v:Math.random()+0.5,
c:chars[Math.floor(Math.random()*chars.length)]
});
}}
createParticles();

function draw(){
ctx.clearRect(0,0,canvas.width,canvas.height);
if(particlesOn){
ctx.fillStyle='rgba(255,255,255,0.5)';
particles.forEach(p=>{
ctx.font=p.s+'px Arial';
ctx.fillText(p.c,p.x,p.y);
p.y+=p.v;
if(p.y>canvas.height)p.y=0;
});
}
requestAnimationFrame(draw);
}
draw();

/* LOGIC */
function changeDensity(v){density=v;createParticles()}
function changeBg(c){document.body.style.background=c}

const gematria={'א':1,'ב':2,'ג':3,'ד':4,'ה':5,'ו':6,'ז':7,'ח':8,'ט':9,'י':10,'כ':20,'ך':20,'ל':30,'מ':40,'ם':40,'נ':50,'ן':50,'ס':60,'ע':70,'פ':80,'ף':80,'צ':90,'ץ':90,'ק':100,'ר':200,'ש':300,'ת':400};

let currentText='',useKatan=false,showFinal=false,settingsMode=false;
let lastCalculation={text:'',value:0};

function calculateValue(t){
let s=0;
for(let c of t){
if(gematria[c]){
let v=gematria[c];
if(useKatan){v=v%9||9}
s+=v;
}}
return s;
}

function updateDisplay(){
textEl.innerText=currentText||' ';
resultEl.innerText=calculateValue(currentText);
}

function add(l){currentText+=l;updateDisplay()}
function clearAll(){currentText='';updateDisplay()}

function toggleKatan(){
useKatan=!useKatan;
document.querySelector('.star').classList.toggle('active');
updateDisplay();
}

function toggleKeyboard(){showFinal=!showFinal;renderButtons()}
function toggleSettings(){settingsMode=!settingsMode;renderButtons()}

function pressEquals(){
lastCalculation.text=currentText;
lastCalculation.value=calculateValue(currentText);
updateDisplay();
}

function loadLast(){
if(lastCalculation.text){currentText=lastCalculation.text;updateDisplay();}
}

/* BUTTONS */
const normal=['א','ב','ג','ד','ה','ו','ז','ח','ט','י','כ','ל','מ','נ','ס','ע','פ','צ','ק','ר','ש','ת'];
const finalK=['ך','ם','ן','ף','ץ'];

function renderButtons(){
buttonsEl.innerHTML='';

if(settingsMode){

const colors=['#1e1e1e','#000','#fff','#f00','#0f0','#00f','#ff0','#f0f','#0ff','#ffa500','#800080','#008000','#808000','#800000','#008080','#c0c0c0','#808080','#9999ff','#ff9999','#99ff99','#ffff99','#ffcc99','#cc99ff','#66cccc','#ff6666','#66ff66','#6666ff','#ff66cc','#66ffcc','#cccccc','#222244','#444422'];

colors.forEach(col=>{
let b=document.createElement('button');
b.className='colorBox';
b.style.background=col;
b.onclick=()=>changeBg(col);
buttonsEl.appendChild(b);
});

let shade=document.createElement('input');
shade.type='range';shade.min=50;shade.max=150;shade.value=100;
shade.oninput=e=>document.body.style.filter=`brightness(${e.target.value}%)`;

let wrap1=document.createElement('div');
wrap1.className='full';
wrap1.appendChild(shade);
buttonsEl.appendChild(wrap1);

let toggle=document.createElement('button');
toggle.className='full';
toggle.innerText='Toggle Letters';
toggle.onclick=()=>particlesOn=!particlesOn;
buttonsEl.appendChild(toggle);

let dens=document.createElement('input');
dens.type='range';dens.min=40;dens.max=200;dens.value=density;
dens.oninput=e=>changeDensity(e.target.value);

let wrap2=document.createElement('div');
wrap2.className='full';
wrap2.appendChild(dens);
buttonsEl.appendChild(wrap2);

return;
}

if(showFinal){
finalK.forEach(k=>{
let b=document.createElement('button');
b.innerText=k;
b.onclick=()=>add(k);
buttonsEl.appendChild(b);
});

let r=document.createElement('button');
r.className='return';
r.innerText='חזור';
r.onclick=toggleKeyboard;
buttonsEl.appendChild(r);
return;
}

normal.forEach(k=>{
let b=document.createElement('button');
b.innerText=k;
b.onclick=()=>add(k);
buttonsEl.appendChild(b);
});

let t=document.createElement('button');
t.className='toggle';
t.innerText='סופיות';
t.onclick=toggleKeyboard;
buttonsEl.appendChild(t);

let eq=document.createElement('button');
eq.className='equal';
eq.innerText='=';
eq.onclick=pressEquals;
buttonsEl.appendChild(eq);

let sp=document.createElement('button');
sp.className='space';
sp.onclick=()=>add(' ');
buttonsEl.appendChild(sp);

let clr=document.createElement('button');
clr.className='clear';
clr.innerText='C';
clr.onclick=clearAll;
buttonsEl.appendChild(clr);

let title=document.createElement('button');
title.style.background='white';
title.style.color='black';
title.style.fontFamily="'Great Vibes', cursive";
title.innerText='Gematria Calculator';
title.onclick=loadLast;
buttonsEl.appendChild(title);
}

renderButtons();
</script>

</body>
</html>
