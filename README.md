
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>PLAYVERSE | Arcade</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#090b18;color:white;font-family:Arial,sans-serif}
header{padding:16px;background:#15172c;position:sticky;top:0;z-index:2;border-bottom:1px solid #343751}
.logo{font-size:25px;font-weight:900;color:#43e7ff}
input{width:100%;padding:12px;border:0;border-radius:10px;margin-top:12px;font-size:16px}
.hero{text-align:center;padding:25px 12px;background:linear-gradient(135deg,#39215e,#075065)}
.hero h1{font-size:35px;margin:5px}
button{border:0;border-radius:10px;padding:12px;background:#43e7ff;color:#101020;font-weight:bold;cursor:pointer;touch-action:manipulation}
main{max-width:1000px;margin:auto;padding:15px}
.filters,.controls{display:flex;gap:8px;flex-wrap:wrap;justify-content:center;margin:14px 0}
.filters button{background:#242641;color:white}
.grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:12px}
.card{background:#171a30;border:1px solid #333750;border-radius:14px;overflow:hidden;text-align:center;padding-bottom:14px}
.art{font-size:48px;padding:22px;background:linear-gradient(135deg,#27204b,#124253)}
.card h3{margin:12px 4px 5px}
.card p{color:#bbc0d6;font-size:13px;min-height:30px}
.card button{width:88%}
#playbox{display:none;position:fixed;inset:0;background:#070915;z-index:5;overflow:auto;padding:12px}
.topline{display:flex;justify-content:space-between;align-items:center;gap:10px;max-width:650px;margin:5px auto 12px}
#gameTitle{font-size:20px;font-weight:bold}
#game{display:block;width:100%;max-width:650px;height:auto;aspect-ratio:3/2;background:#10152a;border:2px solid #41486b;border-radius:12px;margin:auto;touch-action:none}
#status{text-align:center;min-height:25px;margin:10px}
.controls button{min-width:65px;font-size:18px}
#taparea{display:none;margin:12px auto;padding:25px;font-size:25px;max-width:300px;width:100%;background:#ff4fd8}
.small{text-align:center;color:#c3c8dc;font-size:13px}
@media(min-width:650px){.grid{grid-template-columns:repeat(3,minmax(0,1fr))}}
</style>
</head>
<body>
<header>
<div class="logo">🎮 PLAYVERSE</div>
<input id="search" placeholder="Search games..." oninput="render()">
</header>
<section class="hero">
<h1>PLAY. WIN. REPEAT.</h1>
<p>10 arcade games. One gaming world.</p>
<button onclick="document.getElementById('games').scrollIntoView()">EXPLORE GAMES</button>
</section>
<main id="games">
<h2>🔥 All Games</h2>
<div class="filters">
<button onclick="cat='All';render()">All</button>
<button onclick="cat='Arcade';render()">Arcade</button>
<button onclick="cat='Puzzle';render()">Puzzle</button>
<button onclick="cat='Action';render()">Action</button>
<button onclick="cat='Casual';render()">Casual</button>
</div>
<div class="grid" id="grid"></div>
</main>

<div id="playbox">
<div class="topline">
<div id="gameTitle">Game</div>
<button onclick="closeGame()">✕ Close</button>
</div>
<canvas id="game" width="600" height="400"></canvas>
<div id="status">Ready to play!</div>
<div class="controls" id="controls">
<button id="left">⬅️</button>
<button id="jump">⬆️ Jump</button>
<button id="right">➡️</button>
<button id="action">🎯 Action</button>
</div>
<button id="taparea" onclick="tapGame()">TAP!</button>
<div class="controls"><button onclick="startGame()">↻ Restart</button></div>
<p class="small">Mobile: use the buttons. Computer: use arrow keys or Space.</p>
</div>

<script>
const games=[
["Neon Jumper","🦘","Arcade","Jump and avoid obstacles."],
["Memory Match","🧠","Puzzle","Find matching pairs."],
["Reflex Rush","⚡","Arcade","Tap the target quickly."],
["Cookie Clicker","🍪","Casual","Collect cookies and buy upgrades."],
["Tic-Tac-Toe","⭕","Puzzle","Play X against computer O."],
["Star Collector","🌟","Casual","Collect stars and avoid danger."],
["Zombie Survival","🧟","Action","Dodge zombies and survive."],
["Cycle Rush","🚴","Arcade","Avoid obstacles on your ride."],
["Sky Pilot","✈️","Action","Fly through the sky."],
["Meteor Dodge","☄️","Action","Dodge falling meteors."]
];
let cat="All";
function render(){
let q=document.getElementById("search").value.toLowerCase();
document.getElementById("grid").innerHTML=games.filter(g=>(cat==="All"||g[2]===cat)&&g[0].toLowerCase().includes(q)).map(g=>
`<div class="card"><div class="art">${g[1]}</div><h3>${g[0]}</h3><p>${g[3]}</p><button onclick="openGame('${g[0]}')">▶ PLAY NOW</button></div>`).join("");
}
const c=document.getElementById("game"),ctx=c.getContext("2d");
let current="",score=0,over=false,loop=null,frame=0,player={x:300,y:330,vx:0,vy:0},items=[],cells=[],turn="X",board=[],matches=[],first=null,locked=false,cookies=0,upgrade=10,started=0,target={x:300,y:200},pressed={};
const W=600,H=400;
function status(s){document.getElementById("status").textContent=s}
function text(t,x,y,size=24,color="white"){ctx.fillStyle=color;ctx.font=`bold ${size}px Arial`;ctx.textAlign="center";ctx.fillText(t,x,y)}
function rect(x,y,w,h,color){ctx.fillStyle=color;ctx.fillRect(x,y,w,h)}
function circle(x,y,r,color){ctx.fillStyle=color;ctx.beginPath();ctx.arc(x,y,r,0,Math.PI*2);ctx.fill()}
function rand(n){return Math.random()*n}
function openGame(name){
current=name;document.getElementById("playbox").style.display="block";
document.getElementById("gameTitle").textContent=name;
document.getElementById("taparea").style.display=name==="Reflex Rush"||name==="Cookie Clicker"?"block":"none";
startGame();
}
function closeGame(){clearInterval(loop);document.getElementById("playbox").style.display="none"}
function startGame(){
clearInterval(loop);frame=0;score=0;over=false;items=[];cells=[];matches=[];first=null;locked=false;turn="X";board=Array(9).fill("");cookies=0;upgrade=10;pressed={};
player={x:300,y:320,vx:0,vy:0};started=Date.now();target={x:rand(500)+50,y:rand(300)+50};
if(current==="Memory Match"){
let a=["🍎","🍎","🍋","🍋","🍇","🍇","🍒","🍒","🥝","🥝","🍉","🍉"];
a.sort(()=>Math.random()-.5);cells=a.map((v,i)=>({v,i,show:false,done:false}));
}
if(current==="Star Collector"||current==="Neon Jumper")items.push({x:rand(500)+50,y:rand(180)+40,r:12,type:"⭐"});
if(current==="Reflex Rush")status("Wait for green, then tap TAP!");
else status("Score: 0");
draw();
loop=setInterval(tick,50);
}
function end(msg){over=true;status(msg+" | Score: "+score);clearInterval(loop)}
function draw(){
ctx.clearRect(0,0,W,H);rect(0,0,W,H,"#10152a");
if(current==="Memory Match"){drawMemory();return}
if(current==="Tic-Tac-Toe"){drawTTT();return}
if(current==="Cookie Clicker"){text("🍪 COOKIE FACTORY",300,75,28);text("Cookies: "+cookies,300,145,25);text("Upgrade cost: "+upgrade,300,190,18);text("Tap the big cookie below!",300,245,17);text("🍪",300,325,90);return}
if(current==="Reflex Rush"){text("REFLEX RUSH",300,65,30);circle(target.x,target.y,35,frame>40?"#35ed81":"#f64b68");text(frame>40?"TAP NOW!":"WAIT...",300,350,22);return}
rect(0,360,W,40,"#272d49");
if(current==="Neon Jumper"){
for(let i=0;i<6;i++)rect(i*110,355-(i%2)*25,75,10,"#43e7ff");
text("🦘",player.x,player.y,35);
items.forEach(o=>text("🟥",o.x,o.y,26));
text("Score: "+score,80,30,20);
}else if(current==="Sky Pilot"){
items.forEach(o=>text("☁️",o.x,o.y,35));text("✈️",player.x,player.y,38);text("Score: "+score,90,30,20);
}else if(current==="Cycle Rush"){
items.forEach(o=>text("🚧",o.x,o.y,30));text("🚴",player.x,player.y,35);text("Score: "+score,90,30,20);
}else if(current==="Meteor Dodge"){
items.forEach(o=>text("☄️",o.x,o.y,30));text("🚀",player.x,player.y,35);text("Score: "+score,90,30,20);
}else if(current==="Zombie Survival"){
items.forEach(o=>text("🧟",o.x,o.y,30));text("🧍",player.x,player.y,35);text("Score: "+score,90,30,20);
}else if(current==="Star Collector"){
items.forEach(o=>text(o.type||"⭐",o.x,o.y,25));text("🙂",player.x,player.y,32);text("Score: "+score,90,30,20);
}
}
function drawMemory(){
text("Find all 6 matching pairs",300,35,22);
cells.forEach(o=>{let x=60+(o.i%4)*130,y=75+Math.floor(o.i/4)*105;
rect(x,y,100,80,o.done?"#185b51":"#292f51");
text(o.done||o.show?o.v:"?",x+50,y+50,32);
});text("Pairs: "+matches.length+"/6",300,390,18);
}
function drawTTT(){
for(let i=1;i<3;i++){rect(i*200-3,65,6,300,"#69718f");rect(40,i*100+30,520,6,"#69718f")}
for(let i=0;i<9;i++){let x=100+(i%3)*200,y=115+Math.floor(i/3)*100;text(board[i],x,y,55,board[i]==="X"?"#43e7ff":"#ff4fd8")}
text("You are X",300,35,22);
}
function tick(){
if(over)return;frame++;
if(current==="Memory Match"||current==="Tic-Tac-Toe"||current==="Cookie Clicker"||current==="Reflex Rush"){draw();return}
if(current==="Neon Jumper"){
if(pressed.left)player.x-=7;if(pressed.right)player.x+=7;
if(pressed.jump&&player.y>=320)player.vy=-13;
player.vy+=0.65;player.y+=player.vy;
if(player.y>320){player.y=320;player.vy=0}
}else if(current==="Sky Pilot"){
if(pressed.left)player.x-=7;if(pressed.right)player.x+=7;
if(pressed.jump||pressed.up)player.y-=7;else player.y+=2;
}else{
if(pressed.left)player.x-=7;if(pressed.right)player.x+=7;
if(pressed.jump||pressed.up)player.y-=6;if(pressed.down)player.y+=6;
}
player.x=Math.max(20,Math.min(580,player.x));player.y=Math.max(35,Math.min(340,player.y));
if(frame%23===0){
let x=rand(550)+25;
items.push({x,y:-10,r:14,type:current==="Star Collector"?"⭐":"☄️"});
}
items.forEach(o=>{o.y+=current==="Neon Jumper"?5:current==="Sky Pilot"?4:5+(score/30);});
items=items.filter(o=>o.y<420);
for(let o of items){
let d=Math.hypot(o.x-player.x,o.y-player.y);
if(d<30){
if(current==="Star Collector"){score++;o.y=500;status("Stars collected: "+score)}
else if(current==="Neon Jumper"&&o.y>player.y-25){score++;o.y=500;status("Score: "+score)}
else {end("Game over!");break}
}
if(o.y>390&&current==="Neon Jumper"){score++;o.y=500;status("Score: "+score)}
}
if(frame%10===0&&current==="Sky Pilot"){score++;status("Distance: "+score)}
if(frame%10===0&&current==="Cycle Rush"){score++;status("Distance: "+score)}
if(frame%10===0&&current==="Zombie Survival"){score++;status("Survival: "+score)}
if(frame%10===0&&current==="Meteor Dodge"){score++;status("Score: "+score)}
draw();
}
function tapGame(){
if(over)return;
if(current==="Cookie Clicker"){cookies++;score=cookies;status("Cookies: "+cookies+" | Upgrade costs "+upgrade);draw()}
if(current==="Reflex Rush"){
if(frame<40){status("Too early! Restart to try again.");over=true;clearInterval(loop)}
else {score+=Math.max(1,100-Math.floor((Date.now()-started)/10));status("Great reaction! Score: "+score);over=true;clearInterval(loop);draw()}
}
}
function canvasClick(e){
const r=c.getBoundingClientRect(),x=(e.clientX-r.left)*W/r.width,y=(e.clientY-r.top)*H/r.height;
if(current==="Memory Match"){
if(locked)return;
let i=Math.floor((y-75)/105)*4+Math.floor((x-60)/130);
if(x<60||x>550||y<75||y>285||!cells[i]||cells[i].done||cells[i].show)return;
cells[i].show=true;
if(!first){first=cells[i]}else{
locked=true;
if(first.v===cells[i].v){first.done=true;cells[i].done=true;matches.push(first.v);first=null;locked=false;score+=10;status(matches.length===6?"You won! 🎉":"Pairs: "+matches.length+"/6")}
else {let a=first,b=cells[i];setTimeout(()=>{a.show=false;b.show=false;first=null;locked=false;draw()},650)}
}
draw();return;
}
if(current==="Tic-Tac-Toe"){
if(over||turn!=="X")return;
let col=Math.floor(x/200),row=Math.floor((y-65)/100),i=row*3+col;
if(row<0||row>2||col<0||col>2||board[i])return;
board[i]="X";turn="O";
if(checkWin("X")){status("You win! 🎉");over=true;draw();return}
if(board.every(Boolean)){status("Draw!");over=true;draw();return}
setTimeout(()=>{
let free=board.map((v,i)=>v?null:i).filter(v=>v!==null);
if(!free.length)return;
let best=free.find(i=>wouldWin(i,"O"))??free.find(i=>wouldWin(i,"X"))??free[Math.floor(rand(free.length))];
board[best]="O";turn="X";
if(checkWin("O")){status("Computer wins!");over=true}
else if(board.every(Boolean)){status("Draw!");over=true}
else status("Your turn!");
draw();
},300);
draw();return;
}
}
function wouldWin(i,p){board[i]=p;let w=checkWin(p);board[i]="";return w}
function checkWin(p){return [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]].some(a=>a.every(i=>board[i]===p))}
c.addEventListener("pointerdown",e=>{
if(current==="Cookie Clicker"&&e.offsetY>240){tapGame();return}
if(current==="Reflex Rush"){tapGame();return}
canvasClick(e);
});
function bind(id,key){
let b=document.getElementById(id);
b.addEventListener("pointerdown",e=>{e.preventDefault();pressed[key]=true});
["pointerup","pointerleave","pointercancel"].forEach(ev=>b.addEventListener(ev,()=>pressed[key]=false));
}
bind("left","left");bind("right","right");bind("jump","jump");bind("action","action");
document.addEventListener("keydown",e=>{
if(["ArrowLeft","a","A"].includes(e.key))pressed.left=true;
if(["ArrowRight","d","D"].includes(e.key))pressed.right=true;
if(["ArrowUp"," ","w","W"].includes(e.key))pressed.jump=true;
if(["ArrowDown","s","S"].includes(e.key))pressed.down=true;
});
document.addEventListener("keyup",e=>{
if(["ArrowLeft","a","A"].includes(e.key))pressed.left=false;
if(["ArrowRight","d","D"].includes(e.key))pressed.right=false;
if(["ArrowUp"," ","w","W"].includes(e.key))pressed.jump=false;
if(["ArrowDown","s","S"].includes(e.key))pressed.down=false;
});
render();
</script>
</body>
</html>

