<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<title>🌌 Jupiter & Saturn Mission</title>
<style>
html,body{margin:0;overflow:hidden;background:#00030a;font-family:system-ui}
canvas{display:block}
.hud{
 position:absolute;left:16px;top:16px;
 padding:14px 18px;border-radius:14px;
 background:rgba(255,255,255,0.06);
 backdrop-filter:blur(10px);
 color:#d9f3ff;font-size:13px;min-width:360px;
}
.good{color:#9ff0ff}
.mid{color:#ffd29f}
.bad{color:#ff9f9f}
.warn{color:#ffb366}
</style>
</head>
<body>

<canvas id="c"></canvas>
<div class="hud" id="hud"></div>

<script>
const c=document.getElementById("c");
const ctx=c.getContext("2d");
let w,h;
function resize(){w=c.width=innerWidth;h=c.height=innerHeight}
resize();addEventListener("resize",resize);

// Astronomical scale
const AU=110;

// Sun
const sun={x:w/2,y:h/2};

// Planets
const earth ={a:AU*1.0 ,angle:0  ,r:7 ,name:"Earth DSN"};
const jupiter={a:AU*5.2 ,angle:1.6,r:10,name:"Jupiter Relay"};
const saturn ={a:AU*9.6 ,angle:2.8,r:11,name:"Saturn Target"};

// Spacecraft
const ship={progress:0,angle:0.4,r:3};

// Solar storm
let storm={active:false,intensity:0};

// Radio model
const txPower=180;

// Utilities
function pos(a,ang){
 return {x:sun.x+Math.cos(ang)*a,y:sun.y+Math.sin(ang)*a};
}

// Solar storm
function updateStorm(){
 if(!storm.active && Math.random()<0.0006){
  storm.active=true;
  storm.intensity=1+Math.random()*4;
 }
 if(storm.active){
  storm.intensity*=0.995;
  if(storm.intensity<0.1) storm.active=false;
 }
}

// Deep-space link
function link(distAU){
 let signal=txPower/(distAU*distAU*90);
 let noise=0.06+Math.random()*0.05;
 if(storm.active) noise+=storm.intensity*0.4;
 let snr=signal/noise;
 let latency=distAU*8.3; // minutes
 return {snr,latency};
}

// Update
function update(){
 earth.angle+=0.0012;
 jupiter.angle+=0.0004;
 saturn.angle+=0.00025;

 ship.progress=Math.min(ship.progress+0.00018,1);
 ship.angle=
   earth.angle*(1-ship.progress)+
   saturn.angle*ship.progress;

 updateStorm();
}

// Draw
function draw(){
 ctx.fillStyle="rgba(0,3,10,0.35)";
 ctx.fillRect(0,0,w,h);

 update();

 const pe=pos(earth.a,earth.angle);
 const pj=pos(jupiter.a,jupiter.angle);
 const psat=pos(saturn.a,saturn.angle);
 const pship={
  x:sun.x+Math.cos(ship.angle)*
    (earth.a+(saturn.a-earth.a)*ship.progress),
  y:sun.y+Math.sin(ship.angle)*
    (earth.a+(saturn.a-earth.a)*ship.progress)
 };

 // Orbits
 ctx.strokeStyle="rgba(255,255,255,0.05)";
 [earth,jupiter,saturn].forEach(p=>{
  ctx.beginPath();
  ctx.arc(sun.x,sun.y,p.a,0,Math.PI*2);
  ctx.stroke();
 });

 // Sun
 ctx.beginPath();
 ctx.arc(sun.x,sun.y,12,0,Math.PI*2);
 ctx.fillStyle="#ffcc66";ctx.fill();

 // Planets
 ctx.fillStyle="#2a7fff";
 ctx.beginPath();ctx.arc(pe.x,pe.y,earth.r,0,Math.PI*2);ctx.fill();

 ctx.fillStyle="#ffaa66";
 ctx.beginPath();ctx.arc(pj.x,pj.y,jupiter.r,0,Math.PI*2);ctx.fill();

 ctx.fillStyle="#d6c38a";
 ctx.beginPath();ctx.arc(psat.x,psat.y,saturn.r,0,Math.PI*2);ctx.fill();

 // Ship
 ctx.fillStyle="#ffffff";
 ctx.beginPath();ctx.arc(pship.x,pship.y,ship.r,0,Math.PI*2);ctx.fill();

 // Distances
 const dES=Math.hypot(pship.x-pe.x,pship.y-pe.y)/AU;
 const dSJ=Math.hypot(pship.x-pj.x,pship.y-pj.y)/AU;
 const dEJ=Math.hypot(pe.x-pj.x,pe.y-pj.y)/AU;

 const linkEarth=link(dES);
 const linkRelay=link(dSJ);
 const relayPossible=linkRelay.snr>3 && link(dEJ).snr>3;

 // Choose path
 let mode="DIRECT EARTH";
 let snr=linkEarth.snr;
 let latency=linkEarth.latency;

 if(relayPossible && linkEarth.snr<5){
  mode="RELAY VIA JUPITER";
  snr=Math.min(linkRelay.snr,link(dEJ).snr);
  latency=linkRelay.latency+link(dEJ).latency;
 }

 let cls=snr>6?"good":snr>3?"mid":"bad";

 // Draw links
 ctx.strokeStyle=
   cls==="good"?"rgba(120,220,255,0.6)":
   cls==="mid" ?"rgba(255,200,120,0.6)":
                "rgba(255,120,120,0.6)";
 ctx.beginPath();
 if(mode==="DIRECT EARTH"){
  ctx.moveTo(pe.x,pe.y);ctx.lineTo(pship.x,pship.y);
 }else{
  ctx.moveTo(pe.x,pe.y);ctx.lineTo(pj.x,pj.y);
  ctx.lineTo(pship.x,pship.y);
 }
 ctx.stroke();

 // HUD
 document.getElementById("hud").innerHTML=`
🌌 Jupiter & Saturn Mission<br>
🚀 Progress: ${(ship.progress*100).toFixed(1)}%<br>
🔁 Mode: <b>${mode}</b><br>
📏 Distance (Earth): ${dES.toFixed(2)} AU<br>
📊 SNR: <span class="${cls}">${snr.toFixed(2)}</span><br>
🕒 Latency: ${latency.toFixed(1)} min<br>
🌞 Solar Storm: ${storm.active?'<span class="warn">ACTIVE</span>':'NO'}<br>
📡 Link: <span class="${cls}">${snr>2.5?'CONNECTED':'UNUSABLE'}</span>
`;

 requestAnimationFrame(draw);
}

draw();
</script>
</body>
</html>
