[Super Mario World Warp DX.html](https://github.com/user-attachments/files/32194985/Super.Mario.World.Warp.DX.html)
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SUPER MARIO WORLD WARP DELUXE</title>
<style>
body{margin:0;padding:0;background:#111;display:flex;flex-direction:column;align-items:center;justify-content:center;min-height:100vh;font-family:'Courier New',monospace;color:#fff;overflow:hidden;}
h1{margin:0 0 8px;font-size:18px;letter-spacing:2px;text-shadow:2px 2px #000;}
canvas{border:3px solid #fff;box-shadow:0 0 30px rgba(100,150,255,0.4);}
#ctrl{margin-top:10px;font-size:11px;color:#888;text-align:center;line-height:1.7;}
</style>
<script src="https://cdn.jsdelivr.net/npm/peerjs@1.5.4/dist/peerjs.min.js"></script>
</head>
<body>
<h1>SUPER MARIO BROS.</h1>
<canvas id="c" width="800" height="440"></canvas>
<div id="ctrl">
【P1】 ←→:移動 ／ Space/↑:ジャンプ ／ Shift/X:ダッシュ&ファイア(FF時) ／ Z:ファイア単独<br>
【P2】 A/D:移動 ／ W:ジャンプ ／ Q:ダッシュ&ファイア(FF時) ／ E:ファイア単独<br>
【バトル P1】Shift/X=甲羅発射 【バトル P2】Q=甲羅発射 ／ 【宇宙】矢印移動+Shift/X発射<br>
【MAP】S=セーブ L=ロード 1=1P 2=2P 3=バトル ／ DEBUG: 0913と入力
</div>
<script>
const CV=document.getElementById('c'),ctx=CV.getContext('2d');
ctx.imageSmoothingEnabled=false;
const W=800,H=440,GY=H-60,GR=0.6;

// ========== グローバル状態 ==========
// MODE: TITLE WORLD_MAP PLAYING TUTORIAL WARP_ANIMATION BATTLE SPACE DEBUG
let MODE='TITLE';
let world=1,stageId=1,cameraX=0,numP=1;
let theme='grassland',underwater=false,iceSlide=false;
let hardMode=false,gameCompleted=false,extraCleared=false,lives=10;
let gameTimer=0,stageActive=false;
let titleFrame=0,titleBlink=0,titleSel=1;
let demoTimer=0,demoMode=false;
let demoP={x:100,y:GY-48,vx:3,vy:0,fr:0,ft:0};
let warpTimer=0,warpYOff=0,warpSt=0;
let mapIdx=0;
// デバッグコード入力バッファ
let dbKeyBuf='';
let numCtrl=false;
let pKeyBuf='',tKeyBuf='',rKeyBuf='',showTeacherImg=false;
const TEACHER_IMG=new Image();TEACHER_IMG.src='data:image/png;base64,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';
let teacherImgLoaded=false;TEACHER_IMG.onload=()=>{teacherImgLoaded=true;};

// ========== カラー ==========
const R="#b83400",G="#746000",O="#fc9c00",_=null,LG="#1bb749",LB="#0d7030",LO="#ffccad";

// ========== スプライト ==========
const sIdle=[[_,_,_,_,_,R,R,R,R,R,_,_,_,_,_,_],[_,_,_,_,R,R,R,R,R,R,R,R,R,_,_,_],[_,_,_,_,G,G,G,O,O,G,O,_,_,_,_,_],[_,_,_,G,O,G,O,O,O,G,O,O,O,_,_,_],[_,_,_,G,O,G,G,O,O,O,G,O,O,_,_,_],[_,_,_,G,G,O,O,O,O,G,G,G,G,_,_,_],[_,_,_,_,_,O,O,O,O,O,O,O,_,_,_,_],[_,_,_,_,G,G,R,G,G,G,_,_,_,_,_,_],[_,_,_,G,G,G,R,G,G,R,G,G,G,_,_,_],[_,_,G,G,G,G,R,R,R,R,G,G,G,G,_,_],[_,_,O,O,G,R,O,R,R,O,R,G,O,O,_,_],[_,_,O,O,O,R,R,R,R,R,R,O,O,O,_,_],[_,_,O,O,R,R,R,R,R,R,R,R,O,O,_,_],[_,_,_,_,R,R,R,_,_,R,R,R,_,_,_,_],[_,_,_,G,G,G,_,_,_,_,G,G,G,_,_,_],[_,_,G,G,G,G,_,_,_,_,G,G,G,G,_,_]];
const sRun1=[[_,_,_,_,_,_,R,R,R,R,R,_,_,_,_,_],[_,_,_,_,_,R,R,R,R,R,R,R,R,R,_,_],[_,_,_,_,_,G,G,G,O,O,G,O,_,_,_,_],[_,_,_,_,G,O,G,O,O,O,G,O,O,O,_,_],[_,_,_,_,G,O,G,G,O,O,O,G,O,O,_,_],[_,_,_,_,G,G,O,O,O,O,G,G,G,G,_,_],[_,_,_,_,_,_,O,O,O,O,O,O,O,_,_,_],[_,_,_,G,G,G,G,G,R,G,G,G,_,_,_,_],[_,_,G,G,G,G,G,G,R,G,G,R,G,G,_,O],[_,O,O,G,G,G,G,G,R,R,R,R,G,G,_,O],[_,O,O,O,_,R,R,G,R,O,R,R,O,R,R,O],[_,_,O,_,G,R,R,R,R,R,R,R,R,R,R,_],[_,_,_,G,G,R,R,R,R,R,R,R,R,R,R,_],[_,_,G,G,G,R,R,R,R,R,R,R,_,_,_,_],[_,G,G,G,_,R,R,R,_,_,_,_,_,_,_,_],[_,G,G,_,_,R,R,R,R,_,_,_,_,_,_,_]];
const sRun2=[[_,_,_,_,_,R,R,R,R,R,_,_,_,_,_,_],[_,_,_,_,R,R,R,R,R,R,R,R,R,_,_,_],[_,_,_,_,G,G,G,O,O,G,O,_,_,_,_],[_,_,_,G,O,G,O,O,O,G,O,O,O,_,_,_],[_,_,_,G,O,G,G,O,O,O,G,O,O,_,_,_],[_,_,_,G,G,O,O,O,O,G,G,G,G,_,_,_],[_,_,_,_,_,O,O,O,O,O,O,O,_,_,_,_],[_,_,_,_,G,G,R,G,G,G,G,_,_,_,_,_],[_,_,_,G,G,G,R,R,G,G,G,G,_,_,_,_],[_,_,G,G,G,R,O,R,R,R,R,G,G,_,_,_],[_,_,G,G,_,R,R,R,R,R,R,R,G,G,_,_],[_,_,_,_,R,R,R,R,R,R,R,R,R,G,_,_],[_,_,_,R,R,R,R,R,R,R,R,R,R,_,_,_],[_,_,R,R,R,R,R,R,R,_,_,_,_,_,_],[_,G,G,G,R,R,R,_,_,_,_,_,_,_,_,_],[_,G,G,_,_,_,_,_,_,_,_,_,_,_,_,_]];
const sRun3=[[_,_,_,_,_,R,R,R,R,R,_,_,_,_,_,_],[_,_,_,_,R,R,R,R,R,R,R,R,R,_,_,_],[_,_,_,_,_,G,G,G,O,O,G,O,_,_,_,_],[_,_,_,G,O,G,O,O,O,G,O,O,O,_,_,_],[_,_,_,G,O,G,G,O,O,O,G,O,O,_,_,_],[_,_,_,G,G,O,O,O,O,G,G,G,G,_,_,_],[_,_,_,_,_,O,O,O,O,O,O,O,_,_,_,_],[_,_,_,_,_,G,G,R,G,G,G,_,_,_,_,_],[_,_,_,_,G,G,G,R,G,G,R,G,G,_,_,_],[_,_,_,G,G,G,G,R,R,R,R,G,G,G,_,_],[_,_,O,O,G,G,R,O,R,R,O,R,G,_,_,_],[_,_,O,O,O,_,R,R,R,R,R,R,R,_,_,_],[_,_,_,O,_,R,R,R,R,R,R,R,R,_,_,_],[_,_,_,_,R,R,R,R,R,R,R,R,R,_,_,_],[_,_,_,R,R,R,R,R,_,R,R,R,_,_,_,_],[_,_,_,G,G,G,_,_,_,_,_,_,_,_,_,_]];
const sJump=[[_,_,_,_,_,_,_,_,_,_,_,_,_,R,R,R],[_,_,_,_,_,_,R,R,R,R,R,R,_,R,R,R],[_,_,_,_,_,R,R,R,R,R,R,R,R,R,O,O],[_,_,_,_,_,G,G,G,O,O,G,O,_,G,G,G],[_,_,_,_,G,O,G,O,O,O,G,O,O,G,G,G],[_,_,_,_,G,O,G,G,O,O,O,G,O,O,O,G],[_,_,_,_,G,G,O,O,O,O,G,G,G,G,G,_],[_,_,_,_,_,O,O,O,O,O,O,O,G,_,_,_],[_,_,G,G,G,G,G,R,G,G,G,R,G,_,_,_],[_,G,G,G,G,G,G,G,R,G,G,G,R,_,_,G],[O,O,G,G,G,G,G,G,R,R,R,R,R,_,_,G],[O,O,O,_,_,R,R,G,R,O,R,R,O,R,R,G],[_,O,_,G,G,R,R,R,R,R,R,R,R,R,R,G],[_,_,G,G,G,R,R,R,R,R,R,R,R,R,R,G],[_,G,G,G,_,R,R,R,R,R,R,R,_,_,_,_],[_,G,G,_,_,R,R,R,R,R,_,_,_,_,_,_]];
const sDie=[[_,_,_,_,_,_,R,R,R,R,_,_,_,_,_,_],[_,_,_,_,_,R,R,R,R,R,R,_,_,_,_,_],[_,_,_,G,G,G,O,O,O,O,G,G,G,_,_,_],[_,_,G,O,O,G,O,O,O,O,G,O,O,G,_,_],[_,G,O,O,O,G,G,G,G,G,G,O,O,O,G,_],[_,G,O,O,G,O,O,O,O,O,O,G,O,O,G,_],[_,_,G,G,G,O,O,O,O,O,O,G,G,G,_,_],[_,_,_,_,G,G,G,G,G,G,G,G,_,_,_,_],[_,_,_,G,G,G,R,G,G,R,G,G,G,_,_,_],[_,_,G,G,G,G,R,G,G,R,G,G,G,G,_,_],[_,G,G,G,G,G,R,R,R,R,G,G,G,G,G,_],[_,O,O,G,G,R,R,O,O,R,R,G,G,O,O,_],[_,O,O,O,_,R,O,R,R,O,R,_,O,O,O,_],[_,_,O,_,_,R,R,R,R,R,R,_,_,O,_,_],[_,_,_,_,R,R,R,R,R,R,R,R,_,_,_,_],[_,_,_,_,R,R,R,_,_,R,R,R,_,_,_,_]];
function mkL(s){return s.map(r=>r.map(c=>c===R?LG:c===G?LB:c===O?LO:c));}
const lIdle=mkL(sIdle),lRun1=mkL(sRun1),lRun2=mkL(sRun2),lRun3=mkL(sRun3),lJump=mkL(sJump),lDie=mkL(sDie);
function drawSpr(mat,x,y,sz,dir){for(let r=0;r<mat.length;r++)for(let c=0;c<mat[r].length;c++){const tc=dir==='right'?c:mat[r].length-1-c;const col=mat[r][tc];if(col){ctx.fillStyle=col;ctx.fillRect(x+c*sz,y+r*sz,sz,sz);}}}

// ========== プレイヤーファクトリ ==========
function mkPl(x,y,p2){return{x,y,w:48,h:48,vx:0,vy:0,spd:5,dash:8,jmp:13.5,djmp:15.1,grnd:false,dir:'right',fr:0,ft:0,dead:false,clear:false,p2,star:false,starT:0,big:false,mini:false,fire:false,icd:0,score:0,fcd:0};}
let P1=mkPl(100,200,false),P2=mkPl(160,200,true);
const K1={r:false,l:false,j:false,d:false,f:false};
const K2={r:false,l:false,j:false,d:false,f:false};

// ========== ステージ設定 ==========
// ワールド1〜6: 通常ワールド
// ワールド7: ハードモードクリア後 → 夜（クリボーラッシュ）
// ワールド8: ハードモード → 月（低重力ジャンプ3倍）
// ワールド9: ハードモード → 宇宙シューティング
const STAGES=[
  {id:1, w:1,name:"TUTORIAL",type:"tutorial",  cleared:false,cx:120,cy:240},
  {id:2, w:1,name:"1-1",    type:"main",       cleared:false,cx:240,cy:240},
  {id:3, w:1,name:"1-2",    type:"main",       cleared:false,cx:360,cy:240},
  {id:4, w:1,name:"1-3",    type:"main",       cleared:false,cx:480,cy:240},
  {id:5, w:1,name:"1-4",    type:"castle",     cleared:false,cx:600,cy:240},
  {id:6, w:2,name:"2-1",    type:"main",       cleared:false,cx:160,cy:240},
  {id:7, w:2,name:"2-2",    type:"underwater", cleared:false,cx:290,cy:240},
  {id:8, w:2,name:"2-3",    type:"main",       cleared:false,cx:420,cy:240},
  {id:9, w:2,name:"2-4",    type:"castle",     cleared:false,cx:550,cy:240},
  {id:10,w:3,name:"3-1",    type:"main",       cleared:false,cx:160,cy:240},
  {id:11,w:3,name:"3-2",    type:"main",       cleared:false,cx:290,cy:240},
  {id:12,w:3,name:"3-3",    type:"main",       cleared:false,cx:420,cy:240},
  {id:13,w:3,name:"3-4",    type:"castle",     cleared:false,cx:550,cy:240},
  {id:14,w:4,name:"4-1",    type:"main",       cleared:false,cx:130,cy:240},
  {id:15,w:4,name:"4-2",    type:"main",       cleared:false,cx:250,cy:240},
  {id:16,w:4,name:"4-3",    type:"main",       cleared:false,cx:370,cy:240},
  {id:17,w:4,name:"4-4",    type:"castle",     cleared:false,cx:490,cy:240},
  {id:18,w:5,name:"5-1",    type:"sky",        cleared:false,cx:130,cy:240},
  {id:19,w:5,name:"5-2",    type:"sky",        cleared:false,cx:250,cy:240},
  {id:20,w:5,name:"5-3",    type:"sky",        cleared:false,cx:370,cy:240},
  {id:21,w:5,name:"5-4",    type:"castle",     cleared:false,cx:490,cy:240},
  {id:22,w:6,name:"6-1",    type:"lava",       cleared:false,cx:130,cy:240},
  {id:23,w:6,name:"6-2",    type:"lava",       cleared:false,cx:250,cy:240},
  {id:24,w:6,name:"6-3",    type:"lava",       cleared:false,cx:370,cy:240},
  {id:25,w:6,name:"6-4",    type:"castle",     cleared:false,cx:490,cy:240},
  // ---- ハードモード専用 ワールド7: 夜 ----
  {id:26,w:7,name:"7-1",    type:"night",        cleared:false,cx:130,cy:240},
  {id:27,w:7,name:"7-2",    type:"night",        cleared:false,cx:250,cy:240},
  {id:28,w:7,name:"7-3",    type:"night",        cleared:false,cx:370,cy:240},
  {id:29,w:7,name:"7-4",    type:"night_castle", cleared:false,cx:490,cy:240},
  // ---- ハードモード専用 ワールド8: 月 ----
  {id:30,w:8,name:"8-1",    type:"moon",         cleared:false,cx:130,cy:240},
  {id:31,w:8,name:"8-2",    type:"moon",         cleared:false,cx:250,cy:240},
  {id:32,w:8,name:"8-3",    type:"moon",         cleared:false,cx:370,cy:240},
  {id:33,w:8,name:"8-4",    type:"moon_castle",  cleared:false,cx:490,cy:240},
];
// ワールド9: 宇宙シューティング（ハードモード専用）
const SPACE_ST=[
  {id:101,name:"9-1",cleared:false,cx:80, cy:200},
  {id:102,name:"9-2",cleared:false,cx:180,cy:200},
  {id:103,name:"9-3",cleared:false,cx:280,cy:200},
  {id:104,name:"9-4",cleared:false,cx:380,cy:200},
  {id:105,name:"9-5",cleared:false,cx:480,cy:200},
  {id:106,name:"9-6",cleared:false,cx:580,cy:200},
  {id:107,name:"9-7",cleared:false,cx:680,cy:200},
  {id:108,name:"9-8",cleared:false,cx:500,cy:320},
];

// getCurStages: 現在worldのステージリストを返す
// world===9 → 宇宙シューティング用リスト
// world===1〜8 → STAGESから該当ワールドを取得
function getCurStages(){
  if(world===9) return SPACE_ST.map(s=>({...s,w:9,type:'space',canvasX:s.cx,canvasY:s.cy}));
  return STAGES.filter(s=>s.w===world).map(s=>({...s,canvasX:s.cx,canvasY:s.cy}));
}

// ========== ゲームデータ ==========
let platforms=[],coins=[],enemies=[],items=[],fireballs=[],gimmick={};
let goal={x:2000,y:GY-260,w:10,h:260};

// ========== セーブ・ロード ==========
function save(){try{localStorage.setItem('marioSave',JSON.stringify({stages:STAGES.map(s=>({id:s.id,cleared:s.cleared,name:s.name})),spaceStages:SPACE_ST.map(s=>({id:s.id,cleared:s.cleared})),world,mapIdx,numP,gameCompleted,extraCleared,hardMode,lives}));return true;}catch(e){return false;}}
function load(){try{const d=JSON.parse(localStorage.getItem('marioSave'));if(!d)return false;d.stages.forEach(sd=>{const s=STAGES.find(x=>x.id===sd.id);if(s){s.cleared=sd.cleared;s.name=sd.name;}});if(d.spaceStages)d.spaceStages.forEach(sd=>{const s=SPACE_ST.find(x=>x.id===sd.id);if(s)s.cleared=sd.cleared;});world=d.world||1;mapIdx=d.mapIdx||0;numP=d.numP||1;gameCompleted=d.gameCompleted||false;extraCleared=d.extraCleared||false;hardMode=d.hardMode||false;lives=d.lives||10;return true;}catch(e){return false;}}

// ========== テーマ・ギミック取得 ==========
function getTheme(sd){
  if(!sd)return'grassland';
  if(sd.type==='underwater')return'underwater';
  if(sd.type==='castle')return'castle';
  if(sd.type==='sky')return'sky';
  if(sd.type==='lava')return'lava';
  if(sd.type==='night'||sd.type==='night_castle')return'night';
  if(sd.type==='moon'||sd.type==='moon_castle')return'moon';
  if(sd.w===2)return'desert';
  if(sd.w===3)return'ice';
  if(sd.w===4)return'forest';
  return'grassland';
}
function getGimmick(w,type){
  if(type==='castle'||type==='underwater')return null;
  if(type==='night_castle')return'darkfog';
  if(type==='moon_castle')return'moonquake';
  if(type==='night')return'goombarush';
  if(type==='moon')return'lowgrav';
  return{1:'crumble',2:'current',3:'ice',4:'vine',5:'wind',6:'lavarise'}[w]||null;
}

// ========== 敵スポーン ==========
function spawnEnemies(plist,th){
  const res=[];const spd=(th==='castle'||th==='lava')?(hardMode?2.8:2.2):(hardMode?2.0:1.2);
  plist.forEach(p=>{if(p.type!=='ground'||p.width<=150||p.x<=200)return;
    res.push({type:'goomba',x:p.x+60,y:p.y-32,w:32,h:32,vx:-spd,vy:0,alive:true,sq:0,gr:p});
    if(p.width>250)res.push({type:'goomba',x:p.x+130,y:p.y-32,w:32,h:32,vx:spd*0.9,vy:0,alive:true,sq:0,gr:p});
    if(p.width>300)res.push({type:'koopa',x:p.x+p.width-80,y:p.y-40,w:32,h:40,vx:-(spd*0.8),vy:0,alive:true,shell:false,sq:0,svx:0,gr:p});
  });
  const lim=(th==='castle'||th==='lava')?4:3;
  return res.slice(0,lim);
}
function spawnUW(){
  const res=[];
  [300,500,700,950,1200,1500,1800,2100,2400,2700].forEach((x,i)=>res.push({type:'gesso',x,y:100+(i%3)*80,w:40,h:40,vx:0,vy:1.2,bt:i*30,alive:true,sq:0}));
  [180,240,300,200,260].forEach((y,i)=>res.push({type:'puku',x:400+i*500,y,w:36,h:28,vx:-2,vy:0,alive:true,sq:0}));
  return res;
}
function spawnItems(plist){
  return plist.filter(p=>p.type==='question'&&!p.hit).map(p=>{
    const r=Math.random();const t=r<0.2?'star':r<0.4?'fire':r<0.55?'mini':'mushroom';
    return{type:t,x:p.x+p.width/2-8,y:p.y-30,w:24,h:24,vx:1.5,vy:0,col:false,bt:0};
  });
}

// ========== チュートリアル ==========
const TPLAT=[
  {x:0,y:GY,width:400,height:60,type:'ground'},{x:200,y:260,width:96,height:32,type:'block'},
  {x:500,y:GY,width:500,height:60,type:'ground'},{x:640,y:240,width:64,height:140,type:'pipe'},
  {x:800,y:220,width:96,height:32,type:'block'},{x:1120,y:GY,width:300,height:60,type:'ground'},
  {x:1200,y:280,width:64,height:32,type:'block'},{x:1320,y:200,width:64,height:32,type:'block'},
  {x:1540,y:GY,width:600,height:60,type:'ground'},{x:1650,y:200,width:64,height:180,type:'pipe'},
  {x:1850,y:GY-32,width:32,height:32,type:'block'},{x:1882,y:GY-64,width:32,height:64,type:'block'},
  {x:1914,y:GY-96,width:32,height:96,type:'block'}
];
const TCOINS=[{x:216,y:210,w:16,h:24,col:false},{x:256,y:210,w:16,h:24,col:false},{x:816,y:170,w:16,h:24,col:false},{x:848,y:170,w:16,h:24,col:false},{x:1224,y:230,w:16,h:24,col:false},{x:1344,y:150,w:16,h:24,col:false}];
function loadTutorial(extra){
  platforms=TPLAT.map(p=>({...p}));
  platforms.push({x:300,y:220,width:32,height:32,type:'question',hit:false});
  platforms.push({x:850,y:200,width:32,height:32,type:'question',hit:false});
  coins=TCOINS.map(c=>({...c,col:false}));
  goal={x:2000,y:GY-260,w:10,h:260};
  underwater=false;iceSlide=false;
  theme=extra?'castle':'grassland';
  enemies=spawnEnemies(platforms,theme);
  items=[{type:'mushroom',x:210,y:220,w:24,h:24,vx:1.5,vy:0,col:false,bt:0},{type:'star',x:820,y:180,w:24,h:24,vx:1.5,vy:-2,col:false,bt:0},...spawnItems(platforms)];
  fireballs=[];gimmick={};
  if(extra){
    enemies.push({type:'goomba',x:700,y:GY-32,w:32,h:32,vx:-2.5,vy:0,alive:true,sq:0,gr:{x:540,y:GY,width:600}});
    enemies.push({type:'koopa',x:900,y:GY-40,w:32,h:40,vx:-2,vy:0,alive:true,shell:false,sq:0,svx:0,gr:{x:540,y:GY,width:600}});
  }
  bgmPlay(extra?'castle':'grassland');
}

// ========== ステージロード ==========
function loadStage(sd){
  platforms=[];coins=[];enemies=[];items=[];fireballs=[];gimmick={};
  theme=getTheme(sd);underwater=(theme==='underwater');iceSlide=false;
  const castle=(theme==='castle'),lava=(theme==='lava'),sky=(theme==='sky');
  const night=(theme==='night'),moon=(theme==='moon');
  const wg=getGimmick(sd.w,sd.type);
  if(wg==='ice')iceSlide=true;
  const isExtra=(sd.name==='EXTRA');
  const isCastleType=(theme==='castle'||sd.type==='night_castle'||sd.type==='moon_castle');
  const totalLen=isExtra?17600:isCastleType?8800:4300;
  const MAX_PIPE=isCastleType?50:sky?0:55;

  if(underwater){
    let ux=0;while(ux<totalLen+600){const gw=150+Math.floor(Math.random()*200);platforms.push({x:ux,y:GY,width:gw,height:60,type:'ground'});ux+=gw+60+Math.floor(Math.random()*80);}
    let bx=300;while(bx<totalLen){const by=140+Math.floor(Math.random()*160);platforms.push({x:bx,y:by,width:80,height:32,type:'question',hit:false});bx+=200+Math.floor(Math.random()*120);}
    enemies=spawnUW();items=spawnItems(platforms);
    goal={x:totalLen+200,y:GY-260,w:10,h:260};
    gimmick={currentForce:2.5};bgmPlay('underwater');return;
  }
  if(sky){
    let sx=50;while(sx<totalLen){const sw=70+Math.floor(Math.random()*100);const sy=180+Math.floor(Math.random()*140);platforms.push({x:sx,y:sy,width:sw,height:20,type:Math.random()<0.3?'question':'cloud',hit:false});sx+=sw+50+Math.floor(Math.random()*70);}
    platforms.push({x:totalLen,y:280,width:200,height:20,type:'cloud'});
    goal={x:totalLen+80,y:20,w:10,h:260};
    platforms.forEach(p=>{if(Math.random()<0.3&&p.x>200)enemies.push({type:'goomba',x:p.x+10,y:p.y-32,w:32,h:32,vx:-1.2,vy:0,alive:true,sq:0,gr:p});});
    items=spawnItems(platforms);gimmick={wind:0,wt:0,strong:true};bgmPlay('sky');return;
  }
  platforms.push({x:0,y:GY,width:400,height:60,type:'ground'});
  let cx=400;
  const gapMin=castle?90:80,gapMax=castle?130:140,gndMin=castle?180:200,gndMax=castle?300:280;
  while(cx<totalLen-400){
    const gap=Math.floor(Math.random()*gapMax)+gapMin;cx+=gap;
    const gw=Math.floor(Math.random()*gndMax)+gndMin;
    platforms.push({x:cx,y:GY,width:gw,height:60,type:'ground'});
    const sc=cx+gw/2;
    if(MAX_PIPE>0&&Math.random()<(castle?0.45:0.35)){
      const ph=Math.floor(Math.random()*30)+(lava?30:castle?40:60);
      platforms.push({x:sc-32,y:GY-Math.min(ph,MAX_PIPE),width:64,height:Math.min(ph,MAX_PIPE),type:'pipe'});
    }else if(Math.random()<0.45){
      const by=Math.random()<0.5?260:210;const bw=Math.random()<0.5?96:128;
      const bt=Math.random()<0.4?'question':'block';
      platforms.push({x:sc-bw/2,y:by,width:bw,height:32,type:bt,hit:false});
      if(Math.random()<0.7){coins.push({x:sc-8,y:by-40,w:16,h:24,col:false});coins.push({x:sc+16,y:by-40,w:16,h:24,col:false});}
    }
    if(castle&&Math.random()<0.25)platforms.push({x:sc+60,y:GY-80,width:32,height:80,type:'block'});
    if(wg==='vine'&&Math.random()<0.35){const vby=GY-120-Math.floor(Math.random()*60);platforms.push({x:sc-24,y:vby,width:48,height:16,type:'vine',vy:0.8,baseY:vby,dir:1});}
    if(wg==='crumble'){const last=platforms.filter(p=>p.type==='ground').slice(-1)[0];if(last&&cx>300){last.crumble=true;last.cTimer=0;last.cFall=false;}}
    cx+=gw;
  }
  const gax=cx;
  platforms.push({x:gax,y:GY,width:600,height:60,type:'ground'});
  [{y:32,h:32},{y:64,h:64},{y:96,h:96},{y:128,h:128}].forEach((o,i)=>platforms.push({x:gax+100+i*32,y:GY-o.y,width:32,height:o.h,type:'block'}));
  goal={x:gax+350,y:GY-260,w:10,h:260};
  enemies=spawnEnemies(platforms,theme);
  items=spawnItems(platforms);

  // ギミック設定
  if(wg==='wind'||sky)gimmick={wind:0,wt:0,strong:sky};
  // ワールド6溶岩: 上昇上限をy=340に緩和（ちょっとした段差で回避できる高さ）
  if(wg==='lavarise'||lava)gimmick={lava:GY+80,ldir:-1};
  if(wg==='current')gimmick={currentForce:2.5};
  if(wg==='crumble')gimmick={cplats:platforms.filter(p=>p.crumble).map(p=>({ref:p}))};
  if(wg==='vine')gimmick={vines:platforms.filter(p=>p.type==='vine')};

  // 夜ワールド(W7): クリボーラッシュなし
  if(wg==='goombarush'){gimmick={};}

  // 月ワールド(W8): 低重力ジャンプ3倍
  if(wg==='lowgrav'||moon){gimmick={...gimmick,lowgrav:true};}

  // 4面城: 専用ギミック
  if(sd.type==='night_castle'){gimmick={...gimmick,darkfog:true,fogTimer:0};}
  if(sd.type==='moon_castle'){gimmick={...gimmick,lowgrav:true,moonquake:0};}

  // BGM選択
  const bgmMap={night_castle:'night_castle',moon_castle:'moon_castle'};
  const bgmKey=bgmMap[sd.type]||(night?'night':moon?'moon':theme);
  bgmPlay(bgmKey);
}

function resetGame(){P1=mkPl(100,200,false);P2=mkPl(160,200,true);cameraX=0;fireballs=[];gameTimer=0;stageActive=false;K1.r=K1.l=K1.j=K1.d=K1.f=false;K2.r=K2.l=K2.j=K2.d=K2.f=false;}

// ========== 衝突判定 ==========
function hit(a,b){return a.x<b.x+(b.w||b.width)&&a.x+(a.w||a.width)>b.x&&a.y<b.y+(b.h||b.height)&&a.y+(a.h||a.height)>b.y;}

// ========== 足場衝突解決 ==========
function resolve(pl){
  pl.grnd=false;
  platforms.forEach(pf=>{
    if(pf.type==='vine'){const oT=(pl.y+pl.h)-pf.y;if(oT>0&&oT<20&&pl.vy>=0&&pl.x+pl.w>pf.x&&pl.x<pf.x+pf.width){pl.y=pf.y-pl.h;pl.vy=0;pl.grnd=true;}return;}
    if(!hit(pl,{x:pf.x,y:pf.y,w:pf.width,h:pf.height}))return;
    const oL=(pl.x+pl.w)-pf.x,oR=(pf.x+pf.width)-pl.x,oT=(pl.y+pl.h)-pf.y,oB=(pf.y+pf.height)-pl.y;
    const mn=Math.min(oL,oR,oT,oB);
    if(mn===oT&&pl.vy>=0){pl.y=pf.y-pl.h;pl.vy=0;pl.grnd=true;if(pf.crumble&&!pf.cFall){pf.cTimer=(pf.cTimer||0)+1;if(pf.cTimer>25)pf.cFall=true;}}
    else if(mn===oB&&pl.vy<0){pl.y=pf.y+pf.height;pl.vy=0;if(pf.type==='question'&&!pf.hit){pf.hit=true;const r=Math.random();const t=r<0.2?'star':r<0.4?'fire':r<0.55?'mini':'mushroom';items.push({type:t,x:pf.x+pf.width/2-8,y:pf.y-30,w:24,h:24,vx:1.5,vy:-4,col:false,bt:0});pl.score+=100;seFx('item');}}
    else if(mn===oL&&pl.vx>=0){pl.x=pf.x-pl.w;pl.vx=0;}
    else if(mn===oR&&pl.vx<=0){pl.x=pf.x+pf.width;pl.vx=0;}
  });
}

// ========== ギミック更新 ==========
function updateGimmick(){
  if(gimmick.cplats)gimmick.cplats.forEach(cp=>{if(cp.ref.cFall){cp.ref.y+=4;if(cp.ref.y>H+100)cp.ref.y=-9999;}});
  if(gimmick.wt!==undefined){gimmick.wt++;const str=gimmick.strong?5:2.5;gimmick.wind=Math.sin(gimmick.wt*0.015)*str;}
  // 溶岩: 上限y=340（低め）で緩やかに上下するだけ
  if(gimmick.lava!==undefined){const TH=340,TL=GY+80;gimmick.lava+=gimmick.ldir*0.5;if(gimmick.lava<=TH)gimmick.ldir=1;if(gimmick.lava>=TL)gimmick.ldir=-1;}
  if(gimmick.vines)gimmick.vines.forEach(v=>{v.y+=v.vy*(v.dir||1);if(v.y>v.baseY+80||v.y<v.baseY-100)v.dir=(v.dir||1)*-1;});
  // 夜(W7): クリボーラッシュなし
  // 月城(W8-4): 月震
  if(gimmick.moonquake!==undefined)gimmick.moonquake=(gimmick.moonquake+1)%360;
  // 夜城霧
  if(gimmick.fogTimer!==undefined)gimmick.fogTimer++;
}

// ========== 敵更新 ==========
function updateEnemies(){
  enemies.forEach(e=>{
    if(!e.alive){if(e.sq>0)e.sq--;return;}
    if(e.type==='gesso'){e.bt++;e.y+=Math.sin(e.bt*0.04)*1.5;e.x-=0.4;if(e.x<cameraX-100)e.alive=false;return;}
    if(e.type==='puku'){e.x+=e.vx;if(e.x<cameraX-100)e.x=cameraX+W+50;return;}
    if(e.type==='koopa'&&e.shell){
      e.x+=e.svx;e.vy+=GR;e.y+=e.vy;
      platforms.forEach(p=>{if(!hit(e,{x:p.x,y:p.y,w:p.width,h:p.height}))return;const oT=(e.y+e.h)-p.y,oB=(p.y+p.height)-e.y,oL=(e.x+e.w)-p.x,oR=(p.x+p.width)-e.x,mn=Math.min(oT,oB,oL,oR);if(mn===oT&&e.vy>=0){e.y=p.y-e.h;e.vy=0;}else if(mn===oL&&e.svx>=0){e.x=p.x-e.w;e.svx*=-1;}else if(mn===oR&&e.svx<=0){e.x=p.x+p.width;e.svx*=-1;}});
      if(e.y>H+100)e.alive=false;return;
    }
    e.vy+=GR;e.x+=e.vx;e.y+=e.vy;
    let onG=false;
    platforms.forEach(p=>{if(!hit(e,{x:p.x,y:p.y,w:p.width,h:p.height}))return;const oT=(e.y+e.h)-p.y,oB=(p.y+p.height)-e.y,oL=(e.x+e.w)-p.x,oR=(p.x+p.width)-e.x,mn=Math.min(oT,oB,oL,oR);if(mn===oT&&e.vy>=0){e.y=p.y-e.h;e.vy=0;onG=true;}else if(mn===oL&&e.vx>=0){e.x=p.x-e.w;e.vx*=-1;}else if(mn===oR&&e.vx<=0){e.x=p.x+p.width;e.vx*=-1;}});
    if(onG&&e.gr){if(e.vx>0&&e.x+e.w>e.gr.x+e.gr.width-4)e.vx*=-1;if(e.vx<0&&e.x<e.gr.x+4)e.vx*=-1;}
    if(e.y>H+100)e.alive=false;
  });
}

// ========== アイテム更新 ==========
function updateItems(){
  items.forEach(it=>{
    if(it.col)return;it.bt++;
    it.vy+=it.type==='star'?GR*0.5:GR*0.4;it.x+=it.vx;it.y+=it.vy;
    platforms.forEach(p=>{if(!hit(it,{x:p.x,y:p.y,w:p.width,h:p.height}))return;const oT=(it.y+it.h)-p.y,oB=(p.y+p.height)-it.y,oL=(it.x+it.w)-p.x,oR=(p.x+p.width)-it.x,mn=Math.min(oT,oB,oL,oR);if(mn===oT&&it.vy>=0){it.y=p.y-it.h;it.vy=it.type==='star'?-8:0;}else if(mn===oL&&it.vx>=0){it.x=p.x-it.w;it.vx*=-1;}else if(mn===oR&&it.vx<=0){it.x=p.x+p.width;it.vx*=-1;}});
    if(it.y>H)it.col=true;
  });
}

// ========== ファイアボール更新 ==========
function updateFBs(){
  fireballs=fireballs.filter(fb=>fb.life>0&&fb.x>cameraX-50&&fb.x<cameraX+W+50);
  fireballs.forEach(fb=>{fb.x+=fb.vx;fb.y+=fb.vy;fb.vy+=0.4;fb.life--;platforms.forEach(p=>{if(hit(fb,{x:p.x,y:p.y,w:p.width,h:p.height})&&fb.vy>0){fb.y=p.y-fb.h;fb.vy*=-0.6;}});enemies.forEach(e=>{if(e.alive&&hit(fb,e)){e.alive=false;e.sq=20;fb.life=0;(fb.own===1?P1:P2).score+=200;seFx('stomp');}});});
}

// ========== プレイヤー更新 ==========
function updatePlayer(pl,ks,other){
  if(pl.dead){pl.vy+=0.4;pl.y+=pl.vy;return;}
  if(pl.clear){if(pl.y<GY-pl.h)pl.y+=2;return;}
  if(pl.star){pl.starT--;if(pl.starT<=0)pl.star=false;}
  if(pl.icd>0)pl.icd--;if(pl.fcd>0)pl.fcd--;
  const dash=ks.d&&(ks.r||ks.l);const maxSpd=dash?pl.dash:pl.spd;
  const gs=underwater?0.35:1.0;const wind=gimmick.wind||0;
  const useIce=iceSlide&&pl.grnd;
  const ac=useIce?0.12:(pl.grnd?0.6:0.35);const dc=useIce?0.04:(pl.grnd?0.28:0.12);
  if(ks.r){pl.vx=Math.min(pl.vx+ac,maxSpd+wind*0.2);pl.dir='right';}
  else if(ks.l){pl.vx=Math.max(pl.vx-ac,-(maxSpd-wind*0.2));pl.dir='left';}
  else{if(pl.vx>0)pl.vx=Math.max(0,pl.vx-dc);else if(pl.vx<0)pl.vx=Math.min(0,pl.vx+dc);}
  pl.vx+=wind*0.02;if(underwater)pl.vx*=0.88;
  // 月: ふわふわ（1.2倍・超低重力・長押し滞空）
  const isLowGrav=!!gimmick.lowgrav;
  const jPow=underwater?(dash?-9:-7):(isLowGrav?(dash?-pl.djmp*1.2:-pl.jmp*1.2):(dash?-pl.djmp:-pl.jmp));
  if(ks.j&&pl.grnd){pl.vy=jPow;pl.grnd=false;seFx('jump');}
  if(isLowGrav&&ks.j&&pl.vy<0)pl.vy=Math.max(pl.vy-0.05,-pl.jmp*1.2);
  if(underwater&&ks.j)pl.vy=Math.max(pl.vy-0.5,-5);
  pl.vy+=GR*gs*(isLowGrav?0.15:1.0);if(underwater)pl.vy*=0.9;
  if(isLowGrav&&pl.vy>3.0)pl.vy=3.0;
  if(gimmick.currentForce)pl.vx-=gimmick.currentForce*0.04;
  pl.x+=pl.vx;pl.y+=pl.vy;
  if(pl.x<cameraX)pl.x=cameraX;
  const fireKey=(pl.fire&&pl.fcd===0&&(ks.f||(ks.d&&pl.fire)));
  if(fireKey){fireballs.push({x:pl.x+24,y:pl.y+16,w:10,h:10,vx:pl.dir==='right'?9:-9,vy:-3,life:80,own:pl.p2?2:1});pl.fcd=18;seFx('fire');}
  const lavaY=gimmick.lava||(H+50);const isLvStage=(theme==='lava'||gimmick.lava!==undefined);
  if(isLvStage&&pl.y+pl.h>lavaY){pl.dead=true;pl.vy=-11;ks.r=ks.l=ks.j=ks.d=false;seFx('die');return;}
  if(!isLvStage&&pl.y>H+50){pl.dead=true;pl.vy=-11;ks.r=ks.l=ks.j=ks.d=false;seFx('die');return;}
  if(theme==='sky'&&pl.y<-100){pl.dead=true;pl.vy=-11;return;}
  resolve(pl);
  if(numP===2&&other&&!other.dead){if(hit(pl,other)){const stomp=pl.vy>0&&(pl.y+pl.h-pl.vy)<=(other.y+10);if(stomp){pl.vy=dash?-18:-15;pl.grnd=false;other.vy=Math.min(other.vy,-2);}else{if(pl.x<other.x)pl.x=other.x-pl.w;else pl.x=other.x+other.w;}}}
  if(hit(pl,{x:goal.x,y:goal.y,w:goal.w,h:goal.h})){pl.clear=true;pl.vx=0;pl.vy=0;ks.r=ks.l=ks.j=ks.d=false;pl.score+=5000;seFx('clear');}
  coins.forEach(c=>{if(!c.col&&hit(pl,{x:c.x,y:c.y,w:c.w,h:c.h})){c.col=true;pl.score+=200;seFx('coin');}});
  items.forEach(it=>{
    if(it.col||!hit(pl,{x:it.x,y:it.y,w:it.w,h:it.h}))return;it.col=true;seFx('item');
    if(it.type==='mushroom'){if(!pl.big&&!pl.mini)pl.mini=true;else pl.big=true;pl.score+=1000;}
    else if(it.type==='mini'){pl.mini=true;pl.big=false;pl.score+=500;}
    else if(it.type==='star'){pl.star=true;pl.starT=300;pl.score+=1000;}
    else if(it.type==='fire'){pl.fire=true;pl.big=true;pl.mini=false;pl.score+=1000;}
  });
  enemies.forEach(e=>{
    if(!e.alive||!hit(pl,e))return;
    const iW=(e.type==='gesso'||e.type==='puku');
    const stomp=!iW&&pl.vy>0&&(pl.y+pl.h-pl.vy)<=(e.y+10);
    if(pl.star){e.alive=false;e.sq=20;pl.vy=-8;pl.score+=300;seFx('stomp');return;}
    if(stomp){
      if(e.type==='goomba'){e.alive=false;e.sq=20;pl.vy=-8;pl.score+=100;seFx('stomp');}
      else if(e.type==='koopa'){if(!e.shell){e.shell=true;e.vx=0;e.svx=0;pl.vy=-8;pl.score+=100;seFx('stomp');}else e.svx=(pl.x<e.x)?7:-7;}
      return;
    }
    const safeShell=e.type==='koopa'&&e.shell&&e.svx===0;
    if(safeShell||iW&&false)return;
    if(iW){if(pl.icd>0)return;damage(pl,ks);}
    else if(!safeShell){if(pl.icd>0)return;damage(pl,ks);}
  });
  if(Math.abs(pl.vx)>0.3&&pl.grnd){pl.ft++;const ad=dash?3:6;if(pl.ft>ad){pl.fr=(pl.fr+1)%3;pl.ft=0;}}else{pl.fr=0;pl.ft=0;}
}
function damage(pl,ks){
  if(pl.fire){pl.fire=false;pl.icd=120;seFx('hit');return;}
  if(pl.big){pl.big=false;pl.icd=120;seFx('hit');return;}
  if(pl.mini){pl.mini=false;pl.icd=120;seFx('hit');return;}
  pl.dead=true;pl.vy=-11;ks.r=ks.l=ks.j=ks.d=false;seFx('die');
  if(hardMode){lives--;if(lives<=0)lives=0;}
}

// ========== BGM / SE ==========
let audioCtx=null;function getAC(){if(!audioCtx)try{audioCtx=new(window.AudioContext||window.webkitAudioContext)();}catch(e){}return audioCtx;}
let bgmGain=null,bgmKey='',bgmLoop=null;
const BGM={
  grassland:[[523,.1],[659,.1],[784,.15],[659,.1],[523,.1],[440,.1],[523,.3],[392,.1],[440,.1],[523,.1],[587,.1],[523,.1],[440,.1],[392,.3]],
  desert:[[349,.15],[392,.1],[440,.2],[392,.1],[349,.1],[330,.1],[349,.3],[294,.1],[330,.1],[349,.15],[392,.2],[349,.3]],
  ice:[[523,.2],[587,.1],[659,.25],[698,.1],[659,.1],[587,.1],[523,.4],[494,.1],[523,.15],[587,.2],[659,.3],[587,.5]],
  forest:[[330,.2],[370,.15],[440,.2],[370,.1],[330,.1],[294,.1],[330,.4],[277,.1],[294,.15],[330,.2],[370,.15],[330,.4]],
  sky:[[784,.1],[880,.1],[1047,.2],[880,.1],[784,.1],[698,.15],[784,.4],[659,.1],[698,.15],[784,.2],[880,.3],[784,.5]],
  lava:[[220,.2],[246,.15],[261,.2],[246,.1],[233,.1],[220,.4],[196,.15],[220,.2],[246,.15],[261,.25],[246,.45]],
  castle:[[130,.3],[146,.15],[164,.3],[174,.15],[146,.1],[130,.5],[116,.15],[130,.3],[138,.15],[130,.5]],
  underwater:[[261,.2],[293,.15],[329,.2],[349,.15],[329,.1],[293,.1],[261,.4],[246,.1],[261,.2],[293,.15],[329,.25],[293,.5]],
  title:[[330,.12],[392,.12],[523,.12],[659,.25],[587,.12],[523,.12],[659,.5],[523,.12],[392,.12],[330,.12],[392,.25],[440,.12],[523,.5]],
  battle:[[659,.1],[587,.1],[523,.1],[587,.2],[659,.3],[784,.2],[698,.1],[659,.4],[523,.1],[587,.15],[659,.2],[587,.1],[523,.4]],
  space:[[164,.1],[184,.1],[220,.15],[246,.1],[220,.1],[184,.1],[164,.35],[138,.1],[164,.2],[184,.15],[220,.25],[184,.4]],
  // W7夜専用BGM（ミステリアス＆不気味）
  night:[[196,.2],[220,.15],[246,.2],[220,.1],[196,.15],[174,.1],[196,.4],[164,.1],[196,.2],[220,.15],[246,.3],[220,.5]],
  night_castle:[[130,.25],[146,.1],[164,.2],[146,.15],[138,.1],[130,.4],[116,.1],[130,.2],[123,.15],[116,.5],[110,.3],[116,.4]],
  // W8月専用BGM（浮遊感）
  moon:[[392,.15],[440,.1],[494,.2],[523,.1],[494,.1],[440,.1],[392,.4],[349,.1],[392,.15],[440,.2],[494,.25],[440,.5]],
  moon_castle:[[246,.2],[277,.15],[311,.2],[277,.1],[261,.1],[246,.4],[220,.1],[246,.2],[261,.15],[277,.25],[261,.4]],
};
const SE={
  jump:[[800,.04],[600,.04],[500,.06]],coin:[[1047,.04],[1318,.08]],stomp:[[200,.06],[100,.08]],
  die:[[400,.07],[300,.07],[200,.07],[150,.12]],fire:[[1200,.04],[900,.04],[700,.05]],
  item:[[523,.05],[659,.05],[784,.05],[1047,.1]],hit:[[300,.06],[200,.1]],
  clear:[[523,.1],[659,.1],[784,.1],[1047,.15],[784,.1],[1047,.3]],shell:[[400,.05],[600,.06]],
  shoot:[[1400,.03],[1000,.03],[700,.04]],
};
function seFx(k){const ac=getAC();if(!ac)return;const ns=SE[k];if(!ns)return;let t=ac.currentTime;ns.forEach(([f,d])=>{const o=ac.createOscillator(),g=ac.createGain();o.type='square';o.frequency.setValueAtTime(f,t);g.gain.setValueAtTime(0.1,t);g.gain.linearRampToValueAtTime(0,t+d);o.connect(g);g.connect(ac.destination);o.start(t);o.stop(t+d);t+=d;});}
function bgmPlay(k){const ac=getAC();if(!ac)return;if(bgmKey===k)return;bgmStop();bgmKey=k;const ns=BGM[k];if(!ns)return;bgmGain=ac.createGain();bgmGain.gain.value=0.06;bgmGain.connect(ac.destination);let alive=true;function loop(){if(!alive||bgmKey!==k)return;let t=ac.currentTime,tot=0;ns.forEach(([f,d])=>{const o=ac.createOscillator(),g=ac.createGain();o.type='triangle';o.frequency.setValueAtTime(f,t);g.gain.setValueAtTime(0.8,t);g.gain.linearRampToValueAtTime(0.35,t+d*0.7);g.gain.linearRampToValueAtTime(0,t+d);o.connect(g);g.connect(bgmGain);o.start(t);o.stop(t+d);t+=d;tot+=d;});setTimeout(loop,tot*1000-100);}loop();bgmLoop={stop:()=>{alive=false;}};}
function bgmStop(){bgmKey='';if(bgmLoop){bgmLoop.stop();bgmLoop=null;}if(bgmGain){try{bgmGain.disconnect();}catch(e){}bgmGain=null;}}

// ========== バトルモード ==========
let BS=null;
const BK1={r:false,l:false,j:false,s:false},BK2={r:false,l:false,j:false,s:false};
function initBattle(){
  BS={p1:{x:80,y:GY-54,vx:0,vy:0,dir:'right',hp:3,stun:0,scd:0,grnd:false,fr:0,ft:0},p2:{x:660,y:GY-54,vx:0,vy:0,dir:'left',hp:3,stun:0,scd:0,grnd:false,fr:0,ft:0},shells:[],result:'',cd:120};
  bgmPlay('battle');
}
const BPLATS=[{x:0,y:GY,w:800,h:60},{x:150,y:270,w:120,h:20},{x:340,y:230,w:120,h:20},{x:530,y:270,w:120,h:20},{x:240,y:175,w:100,h:20},{x:460,y:175,w:100,h:20}];
function resolveBP(bp){
  bp.vy=Math.min(bp.vy+GR,18);bp.x+=bp.vx;bp.y+=bp.vy;bp.grnd=false;
  BPLATS.forEach(p=>{const oL=(bp.x+48)-p.x,oR=(p.x+p.w)-bp.x,oT=(bp.y+54)-p.y,oB=(p.y+p.h)-bp.y;if(oL>0&&oR>0&&oT>0&&oB>0){const mn=Math.min(oL,oR,oT,oB);if(mn===oT&&bp.vy>=0){bp.y=p.y-54;bp.vy=0;bp.grnd=true;}else if(mn===oB&&bp.vy<0){bp.y=p.y+p.h;bp.vy=0;}else if(mn===oL)bp.x=p.x-48;else if(mn===oR)bp.x=p.x+p.w;}});
  bp.x=Math.max(0,Math.min(752,bp.x));if(bp.y>H){bp.y=GY-54;bp.vy=0;}
}
function updateBattle(){
  if(!BS)return;if(BS.cd>0){BS.cd--;return;}
  function mv(bp,bk){
    if(bp.stun>0){bp.stun--;bp.vx*=0.85;return;}
    const spd=4.5;
    if(bk.r){bp.vx=Math.min(bp.vx+0.5,spd);bp.dir='right';}else if(bk.l){bp.vx=Math.max(bp.vx-0.5,-spd);bp.dir='left';}else bp.vx*=0.8;
    if(bk.j&&bp.grnd){bp.vy=-13;bp.grnd=false;seFx('jump');}
    if(bp.scd>0)bp.scd--;
    if(bk.s&&bp.scd===0){BS.shells.push({x:bp.x+20,y:bp.y+18,vx:bp.dir==='right'?9:-9,vy:-2,w:22,h:22,owner:bp,life:220});bp.scd=35;seFx('shell');}
    bp.ft++;if(Math.abs(bp.vx)>0.5&&bp.grnd){if(bp.ft>6){bp.fr=(bp.fr+1)%3;bp.ft=0;}}else bp.fr=0;
  }
  mv(BS.p1,BK1);mv(BS.p2,BK2);resolveBP(BS.p1);resolveBP(BS.p2);
  BS.shells=BS.shells.filter(s=>s.life>0&&s.x>-50&&s.x<850);
  BS.shells.forEach(s=>{
    s.x+=s.vx;s.y+=s.vy;s.vy=Math.min(s.vy+0.3,8);s.life--;
    BPLATS.forEach(p=>{const oT=(s.y+s.h)-p.y,oB=(p.y+p.h)-s.y,oL=(s.x+s.w)-p.x,oR=(p.x+p.w)-s.x;if(oL>0&&oR>0&&oT>0&&oB>0){const mn=Math.min(oT,oB,oL,oR);if(mn===oT&&s.vy>=0){s.y=p.y-s.h;s.vy*=-0.45;}else if(mn===oL&&s.vx>=0){s.x=p.x-s.w;s.vx*=-1;}else if(mn===oR&&s.vx<=0){s.x=p.x+p.w;s.vx*=-1;}}});
    if(Math.abs(s.vx)<0.5)return;
    [BS.p1,BS.p2].forEach(bp=>{if(s.owner===bp)return;if(s.x<bp.x+48&&s.x+s.w>bp.x&&s.y<bp.y+54&&s.y+s.h>bp.y){bp.hp--;bp.stun=70;s.life=0;seFx('hit');if(bp.hp<=0)BS.result=bp===BS.p1?'p2win':'p1win';}});
  });
  if(!BS.result){if(BS.p1.hp<=0)BS.result='p2win';if(BS.p2.hp<=0)BS.result='p1win';}
}

// ========== 宇宙シューティング(W9) ==========
let SP=null,spShootCd=0;
const SK={l:false,r:false,u:false,d:false,shoot:false};
function initSpace(st){
  const spd=0.7+st*0.22;const rate=Math.max(38,85-st*7);const goal=12+st*4;
  SP={stage:st,ship:{x:380,y:340,w:40,h:30,hp:3,iframes:0},bullets:[],eBullets:[],enemies:[],stars:Array.from({length:60},()=>({x:Math.random()*800,y:Math.random()*440,spd:0.4+Math.random()*2,r:Math.random()<0.12?2:1})),spawnTimer:0,spawnRate:rate,eSpd:spd,timer:0,kills:0,goal,cleared:false,over:false};
  bgmPlay('space');
}
function updateSpace(){
  if(!SP||SP.cleared||SP.over)return;
  const sp=SP;sp.timer++;
  sp.stars.forEach(s=>{s.y+=s.spd;if(s.y>H){s.y=0;s.x=Math.random()*800;}});
  const ship=sp.ship;
  if(SK.l&&ship.x>5)ship.x-=4.5;if(SK.r&&ship.x<755)ship.x+=4.5;if(SK.u&&ship.y>5)ship.y-=4.5;if(SK.d&&ship.y<400)ship.y+=4.5;
  if(ship.iframes>0)ship.iframes--;
  if(spShootCd>0)spShootCd--;
  if(SK.shoot&&spShootCd===0){sp.bullets.push({x:ship.x+17,y:ship.y,w:6,h:14,vy:-11,alive:true});spShootCd=9;seFx('shoot');}
  sp.bullets=sp.bullets.filter(b=>b.alive&&b.y>-20);sp.bullets.forEach(b=>b.y+=b.vy);
  sp.eBullets=sp.eBullets.filter(b=>b.alive&&b.y<460);
  sp.eBullets.forEach(b=>{b.x+=b.vx;b.y+=b.vy;if(ship.iframes===0&&b.x<ship.x+ship.w&&b.x+8>ship.x&&b.y<ship.y+ship.h&&b.y+8>ship.y){ship.hp--;ship.iframes=90;b.alive=false;seFx('hit');if(ship.hp<=0)sp.over=true;}});
  if(++sp.spawnTimer>=sp.spawnRate){sp.spawnTimer=0;const t=['straight','zigzag','chase'][Math.floor(Math.random()*3)];sp.enemies.push({x:Math.random()*740+30,y:-44,w:36,h:30,type:t,ph:0,scd:50+Math.floor(Math.random()*60),alive:true});}
  sp.enemies=sp.enemies.filter(e=>e.alive&&e.y<490);
  sp.enemies.forEach(e=>{
    e.ph+=0.05;if(e.type==='straight')e.y+=sp.eSpd*1.3;else if(e.type==='zigzag'){e.y+=sp.eSpd;e.x+=Math.sin(e.ph*3)*2.5;}else{e.y+=sp.eSpd*0.7;e.x+=(ship.x-e.x)*0.012;}e.x=Math.max(0,Math.min(764,e.x));
    if(sp.stage>=3&&--e.scd<=0){e.scd=55+Math.floor(Math.random()*55);sp.eBullets.push({x:e.x+14,y:e.y+30,vx:(ship.x-e.x)*0.025,vy:2.8+sp.stage*0.25,alive:true});}
    if(ship.iframes===0&&e.x<ship.x+ship.w&&e.x+e.w>ship.x&&e.y<ship.y+ship.h&&e.y+e.h>ship.y){ship.hp--;ship.iframes=90;e.alive=false;seFx('hit');if(ship.hp<=0)sp.over=true;}
    sp.bullets.forEach(b=>{if(b.alive&&b.x<e.x+e.w&&b.x+b.w>e.x&&b.y<e.y+e.h&&b.y+b.h>e.y){e.alive=false;b.alive=false;sp.kills++;seFx('stomp');if(sp.kills>=sp.goal){sp.cleared=true;SPACE_ST[sp.stage-1].cleared=true;seFx('clear');}}});
  });
}

// ========== デバッグモード ==========
let dbMode=false,dbCursor=0;
const DB_ITEMS=[
  {lbl:'残機',g:()=>lives,s:v=>{lives=Math.max(0,v);}},
  {lbl:'ワールド',g:()=>world,s:v=>{world=Math.max(1,Math.min(9,v));mapIdx=0;}},
  {lbl:'ハードモード',g:()=>hardMode?1:0,s:v=>{hardMode=!!v;}},
  {lbl:'P1 BIG',g:()=>P1.big?1:0,s:v=>{P1.big=!!v;}},
  {lbl:'P1 FIRE',g:()=>P1.fire?1:0,s:v=>{P1.fire=!!v;if(v)P1.big=true;}},
  {lbl:'P1 STAR',g:()=>P1.star?1:0,s:v=>{P1.star=!!v;P1.starT=v?600:0;}},
  {lbl:'全ステージ完了',g:()=>'EXEC',s:()=>{STAGES.forEach(s=>s.cleared=true);}},
  {lbl:'全ステージリセット',g:()=>'EXEC',s:()=>{STAGES.forEach(s=>s.cleared=false);}},
  {lbl:'W7解放(夜)',g:()=>'EXEC',s:()=>{STAGES.filter(s=>s.w===7).forEach(s=>s.cleared=false);world=7;mapIdx=0;MODE='WORLD_MAP';bgmStop();dbMode=false;}},
  {lbl:'W8解放(月)',g:()=>'EXEC',s:()=>{STAGES.filter(s=>s.w===8).forEach(s=>s.cleared=false);world=8;mapIdx=0;MODE='WORLD_MAP';bgmStop();dbMode=false;}},
  {lbl:'W9解放(宇宙)',g:()=>'EXEC',s:()=>{SPACE_ST.forEach(s=>s.cleared=false);world=9;mapIdx=0;MODE='WORLD_MAP';bgmStop();dbMode=false;}},
  {lbl:'MAPへ戻る',g:()=>'[SPACE]',s:()=>{dbMode=false;MODE='WORLD_MAP';bgmStop();}},
];

// ========== キー入力 ==========
window.addEventListener('keydown',e=>{
  const k=e.key;
  // ★0913デバッグ
  if(k>='0'&&k<='9'&&k.length===1){dbKeyBuf+=k;if(dbKeyBuf.length>4)dbKeyBuf=dbKeyBuf.slice(-4);if(dbKeyBuf==='0913'){dbKeyBuf='';dbCursor=0;MODE='DEBUG';bgmStop();return;}}
  // ★p×2→L-Gate
  if(k==='p'||k==='P'){pKeyBuf+=k;if(pKeyBuf.length>2)pKeyBuf=pKeyBuf.slice(-2);if(pKeyBuf.toLowerCase()==='pp'){pKeyBuf='';window.open('https://kawaguchied.l-gate.net/','_blank');return;}}else{pKeyBuf='';}
  // ★t×2→先生画像
  if(k==='t'||k==='T'){tKeyBuf+=k;if(tKeyBuf.length>2)tKeyBuf=tKeyBuf.slice(-2);if(tKeyBuf.toLowerCase()==='tt'){tKeyBuf='';showTeacherImg=true;rKeyBuf='';return;}}else{tKeyBuf='';}
  // ★r×5→ゲームに戻る
  if(showTeacherImg){
    if(k==='r'||k==='R'){rKeyBuf+=k;if(rKeyBuf.length>5)rKeyBuf=rKeyBuf.slice(-5);if(rKeyBuf.toLowerCase()==='rrrrr'){rKeyBuf='';showTeacherImg=false;return;}}else{rKeyBuf='';}
    return;
  }
  // ★数字操作モード
  if(numCtrl&&(MODE==='PLAYING'||MODE==='TUTORIAL')){
    if(k==='8'){K1.l=true;P1.dir='left';}if(k==='9'){K1.r=true;P1.dir='right';}
    if(k==='4'){K1.j=true;}if(k==='3'){K1.d=true;}
    if('0123456789'.includes(k))return;
  }
  // バトルモード
  if(MODE==='BATTLE'){
    if(k==='ArrowRight')BK1.r=true;if(k==='ArrowLeft')BK1.l=true;
    if(k===' '||k==='ArrowUp')BK1.j=true;if(k==='Shift'||k==='x'||k==='X')BK1.s=true;
    if(k==='d'||k==='D')BK2.r=true;if(k==='a'||k==='A')BK2.l=true;
    if(k==='w'||k==='W')BK2.j=true;if(k==='q'||k==='Q')BK2.s=true;
    if(k===' '&&BS&&BS.result){bgmStop();MODE='TITLE';}
    return;
  }
  // 宇宙シューティング
  if(MODE==='SPACE'){
    if(k==='ArrowLeft')SK.l=true;if(k==='ArrowRight')SK.r=true;if(k==='ArrowUp')SK.u=true;if(k==='ArrowDown')SK.d=true;
    if(k==='Shift'||k==='x'||k==='X'||k===' ')SK.shoot=true;
    if(k===' '&&SP){
      if(SP.cleared){
        const ni=SP.stage;
        SPACE_ST[ni-1].cleared=true;
        if(ni<8){mapIdx=ni;initSpace(ni+1);}
        else{dbMode=true;MODE='DEBUG';bgmStop();}
      }else if(SP.over)initSpace(SP.stage);
    }
    return;
  }
  // デバッグモード操作
  if(MODE==='DEBUG'){
    if(k==='ArrowUp')dbCursor=(dbCursor-1+DB_ITEMS.length)%DB_ITEMS.length;
    if(k==='ArrowDown')dbCursor=(dbCursor+1)%DB_ITEMS.length;
    const it=DB_ITEMS[dbCursor];
    if(k==='ArrowRight'&&typeof it.g()==='number')it.s(it.g()+1);
    if(k==='ArrowLeft'&&typeof it.g()==='number')it.s(Math.max(0,it.g()-1));
    if(k===' '||k==='Enter')it.s(it.g());
    if(k==='n'||k==='N'){numCtrl=!numCtrl;if(numCtrl){MODE='WORLD_MAP';bgmStop();}}
    e.preventDefault();return;
  }
  // タイトル
  if(MODE==='TITLE'){
    if(k==='1')titleSel=1;if(k==='2')titleSel=2;if(k==='3')titleSel=3;
    demoTimer=0;demoMode=false;
    if(k===' '||k==='Enter'){
      if(titleSel===3){MODE='BATTLE';initBattle();return;}
      numP=titleSel===2?2:1;MODE='WORLD_MAP';bgmStop();
    }
    return;
  }
  if(MODE==='WARP_ANIMATION')return;
  // ワールドマップ
  if(MODE==='WORLD_MAP'){
    const ws=getCurStages();
    if(k==='ArrowRight'&&mapIdx<ws.length-1)mapIdx++;
    if(k==='ArrowLeft'&&mapIdx>0)mapIdx--;
    if(k==='1')numP=1;if(k==='2')numP=2;
    if(k==='s'||k==='S')save();if(k==='l'||k==='L')load();
    if(k===' '){
      const sel=ws[mapIdx];if(!sel)return;
      const si=ws.indexOf(sel);const canPlay=(si===0||ws[si-1].cleared);
      if(!canPlay)return;
      stageId=sel.id;
      // W9: 宇宙シューティング
      if(world===9){initSpace(mapIdx+1);MODE='SPACE';return;}
      const sd=STAGES.find(s=>s.id===stageId);
      if(!sd)return;
      if(sd.type==='tutorial'){loadTutorial(gameCompleted);MODE='TUTORIAL';}
      else{loadStage(sd);MODE='PLAYING';}
      resetGame();stageActive=true;
    }
    return;
  }
  // プレイ中 死亡/クリア後
  if(P1.dead||P1.clear){
    if(k===' '){
      if(P1.clear){
        const cs=STAGES.find(s=>s.id===stageId);if(cs)cs.cleared=true;
        // EXTRAクリア → ハードモード開始
        if(cs&&cs.name==='EXTRA'){
          extraCleared=true;hardMode=true;
          STAGES.forEach(s=>s.cleared=false);
          const tut=STAGES.find(s=>s.id===1);if(tut)tut.name='TUTORIAL';
          gameCompleted=false;world=1;mapIdx=0;lives=10;MODE='WORLD_MAP';bgmStop();return;
        }
        const ws=getCurStages();
        // W1〜6全クリア → EXTRA解放
        const allNT=STAGES.filter(s=>s.w<=6&&s.type!=='tutorial').every(s=>s.cleared);
        if(allNT&&!gameCompleted&&!hardMode){
          gameCompleted=true;world=1;mapIdx=0;
          const tut=STAGES.find(s=>s.id===1);if(tut){tut.name='EXTRA';tut.cleared=false;}
          MODE='WORLD_MAP';bgmStop();return;
        }
        // ワールド全ステージクリア → ワープアニメ
        // ハードモードのW6クリアでW7へ、W7でW8へ、W8でW9へ
        // 通常モードW6クリアでW1へ戻る
        if(ws.every(s=>s.cleared)){
          if(world<6||(world===6&&!hardMode)){
            // W1〜5は普通に次のワールドへ、W6通常→W1へ
            MODE='WARP_ANIMATION';warpTimer=0;warpSt=0;warpYOff=0;bgmStop();return;
          }
          if(world===6&&hardMode){
            // ハードW6クリア → W7(夜)へ
            MODE='WARP_ANIMATION';warpTimer=0;warpSt=0;warpYOff=0;bgmStop();return;
          }
          if(world===7&&hardMode){
            // ハードW7クリア → W8(月)へ
            MODE='WARP_ANIMATION';warpTimer=0;warpSt=0;warpYOff=0;bgmStop();return;
          }
          if(world===8&&hardMode){
            // ハードW8クリア → W9(宇宙)へ
            MODE='WARP_ANIMATION';warpTimer=0;warpSt=0;warpYOff=0;bgmStop();return;
          }
        }
        MODE='WORLD_MAP';const ni=ws.findIndex(s=>s.id===stageId)+1;if(ni<ws.length)mapIdx=ni;bgmStop();
      }else{
        if(hardMode&&lives<=0){MODE='WORLD_MAP';bgmStop();return;}
        const sd=STAGES.find(s=>s.id===stageId);
        if(sd&&sd.type==='tutorial')loadTutorial(gameCompleted);else if(sd)loadStage(sd);
        resetGame();stageActive=true;
      }
    }
    return;
  }
  // P1操作
  if(k==='ArrowRight'){K1.r=true;P1.dir='right';}if(k==='ArrowLeft'){K1.l=true;P1.dir='left';}
  if(k===' '||k==='ArrowUp')K1.j=true;
  if(k==='Shift'||k==='x'||k==='X'){K1.d=true;if(P1.fire)K1.f=true;}
  if(k==='z'||k==='Z')K1.f=true;
  // P2操作
  if(numP===2){
    if(k==='d'||k==='D'){K2.r=true;P2.dir='right';}if(k==='a'||k==='A'){K2.l=true;P2.dir='left';}
    if(k==='w'||k==='W')K2.j=true;if(k==='q'||k==='Q'){K2.d=true;if(P2.fire)K2.f=true;}
    if(k==='e'||k==='E')K2.f=true;
  }

});
window.addEventListener('keyup',e=>{
  const k2=e.key;
  if(numCtrl&&(MODE==='PLAYING'||MODE==='TUTORIAL')){
    if(k2==='8')K1.l=false;if(k2==='9')K1.r=false;
    if(k2==='4')K1.j=false;if(k2==='3'){K1.d=false;K1.f=false;}
  }
  const k=e.key;
  if(MODE==='BATTLE'){if(k==='ArrowRight')BK1.r=false;if(k==='ArrowLeft')BK1.l=false;if(k===' '||k==='ArrowUp')BK1.j=false;if(k==='Shift'||k==='x'||k==='X')BK1.s=false;if(k==='d'||k==='D')BK2.r=false;if(k==='a'||k==='A')BK2.l=false;if(k==='w'||k==='W')BK2.j=false;if(k==='q'||k==='Q')BK2.s=false;return;}
  if(MODE==='SPACE'){if(k==='ArrowLeft')SK.l=false;if(k==='ArrowRight')SK.r=false;if(k==='ArrowUp')SK.u=false;if(k==='ArrowDown')SK.d=false;if(k==='Shift'||k==='x'||k==='X'||k===' ')SK.shoot=false;return;}
  if(k==='ArrowRight')K1.r=false;if(k==='ArrowLeft')K1.l=false;if(k===' '||k==='ArrowUp')K1.j=false;if(k==='Shift'||k==='x'||k==='X'){K1.d=false;K1.f=false;}if(k==='z'||k==='Z')K1.f=false;
  if(k==='d'||k==='D')K2.r=false;if(k==='a'||k==='A')K2.l=false;if(k==='w'||k==='W')K2.j=false;if(k==='q'||k==='Q'){K2.d=false;K2.f=false;}if(k==='e'||k==='E')K2.f=false;
});

// ========== 描画ヘルパー ==========
function getSpr(pl){const L=pl.p2;return{idle:L?lIdle:sIdle,run:[L?lRun1:sRun1,L?lRun2:sRun2,L?lRun3:sRun3],jump:L?lJump:sJump,die:L?lDie:sDie};}
function drawPl(pl){
  const sp=getSpr(pl);let mat;
  if(pl.dead)mat=sp.die;else if(pl.clear)mat=sp.idle;else if(!pl.grnd)mat=sp.jump;else if(Math.abs(pl.vx)>0.3)mat=sp.run[pl.fr];else mat=sp.idle;
  let dy=pl.y;if(pl.big||pl.fire)dy-=16;
  if(pl.star&&Math.floor(pl.starT/4)%2===0)return;
  if(pl.icd>0&&Math.floor(pl.icd/6)%2===0)return;
  drawSpr(mat,pl.x,dy,3,pl.dir);
  if(pl.p2){ctx.fillStyle='#0f0';ctx.font='bold 11px monospace';ctx.fillText('P2',pl.x+16,pl.y-3);}
}

function drawEnemy(e){
  if(!e.alive&&e.sq<=0)return;
  if(e.type==='gesso'){
    const cx=e.x+20,cy=e.y+16;
    ctx.fillStyle='#cc44cc';ctx.beginPath();ctx.ellipse(cx,cy-4,18,14,0,0,Math.PI*2);ctx.fill();
    ctx.fillStyle='#ff88ff';ctx.beginPath();ctx.ellipse(cx-4,cy-8,5,4,0,0,Math.PI*2);ctx.fill();
    ctx.fillStyle='#fff';ctx.fillRect(cx-10,cy-8,7,7);ctx.fillRect(cx+3,cy-8,7,7);
    ctx.fillStyle='#000';ctx.fillRect(cx-8,cy-6,3,4);ctx.fillRect(cx+5,cy-6,3,4);
    ctx.fillStyle='#aa22aa';for(let t=0;t<5;t++){ctx.fillRect(e.x+4+t*8,e.y+26+Math.sin(e.bt*0.1+t)*4,5,12);}
    return;
  }
  if(e.type==='puku'){
    const fl=e.vx<0;ctx.fillStyle='#ff6644';ctx.beginPath();ctx.ellipse(e.x+18,e.y+14,18,12,0,0,Math.PI*2);ctx.fill();
    ctx.fillStyle='#ff4422';if(fl){ctx.beginPath();ctx.moveTo(e.x+36,e.y+4);ctx.lineTo(e.x+36,e.y+24);ctx.lineTo(e.x+46,e.y+14);ctx.fill();}else{ctx.beginPath();ctx.moveTo(e.x,e.y+4);ctx.lineTo(e.x,e.y+24);ctx.lineTo(e.x-10,e.y+14);ctx.fill();}
    const ex=fl?e.x+6:e.x+26;ctx.fillStyle='#fff';ctx.fillRect(ex,e.y+8,8,8);ctx.fillStyle='#000';ctx.fillRect(ex+(fl?1:2),e.y+10,4,4);
    ctx.fillStyle='#ff8866';ctx.fillRect(e.x+10,e.y+10,6,6);ctx.fillRect(e.x+20,e.y+14,5,5);return;
  }
  if(e.type==='goomba'){
    if(!e.alive){ctx.fillStyle='#8B4513';ctx.fillRect(e.x,e.y+e.h-8,e.w,8);return;}
    ctx.fillStyle='#8B4513';ctx.fillRect(e.x+2,e.y+12,e.w-4,e.h-12);ctx.fillStyle='#A0522D';ctx.fillRect(e.x,e.y,e.w,14);
    ctx.fillStyle='#fff';ctx.fillRect(e.x+5,e.y+3,8,7);ctx.fillRect(e.x+18,e.y+3,8,7);ctx.fillStyle='#000';ctx.fillRect(e.x+8,e.y+5,4,4);ctx.fillRect(e.x+21,e.y+5,4,4);ctx.fillRect(e.x+4,e.y+2,10,3);ctx.fillRect(e.x+18,e.y+2,10,3);
    ctx.fillStyle='#5C2E0A';ctx.fillRect(e.x,e.y+e.h-6,12,6);ctx.fillRect(e.x+20,e.y+e.h-6,12,6);
  }
  if(e.type==='koopa'){
    if(e.shell){ctx.fillStyle='#00a800';ctx.fillRect(e.x+2,e.y+8,e.w-4,e.h-8);ctx.fillStyle='#80d010';ctx.fillRect(e.x+8,e.y+12,e.w-16,e.h-16);ctx.strokeStyle='#003800';ctx.lineWidth=1;ctx.beginPath();ctx.moveTo(e.x+16,e.y+8);ctx.lineTo(e.x+16,e.y+e.h);ctx.stroke();ctx.beginPath();ctx.moveTo(e.x+2,e.y+18);ctx.lineTo(e.x+e.w-2,e.y+18);ctx.stroke();}
    else{ctx.fillStyle='#00a800';ctx.fillRect(e.x+4,e.y+14,e.w-8,e.h-14);ctx.fillStyle='#f4c542';ctx.fillRect(e.x+6,e.y,e.w-12,16);ctx.fillStyle='#000';ctx.fillRect(e.x+(e.vx>0?14:8),e.y+4,4,4);ctx.fillStyle='#00a800';ctx.fillRect(e.x+2,e.y+12,e.w-4,18);ctx.fillStyle='#80d010';ctx.fillRect(e.x+6,e.y+14,e.w-12,12);ctx.fillStyle='#f4c542';ctx.fillRect(e.x,e.y+e.h-8,10,8);ctx.fillRect(e.x+22,e.y+e.h-8,10,8);}
  }
}

function drawItem(it){
  if(it.col)return;const x=it.x,y=it.y;
  if(it.type==='mushroom'){ctx.fillStyle='#e00000';ctx.beginPath();ctx.arc(x+12,y+8,12,Math.PI,0);ctx.fill();ctx.fillStyle='#fff';ctx.fillRect(x+3,y+2,5,5);ctx.fillRect(x+15,y+3,5,5);ctx.fillStyle='#f5deb3';ctx.fillRect(x+6,y+10,12,10);ctx.fillStyle='#000';ctx.fillRect(x+7,y+12,3,3);ctx.fillRect(x+14,y+12,3,3);}
  else if(it.type==='mini'){ctx.fillStyle='#ffe040';ctx.beginPath();ctx.arc(x+12,y+12,8,Math.PI,0);ctx.fill();ctx.fillStyle='#fff';ctx.fillRect(x+7,y+7,3,3);ctx.fillRect(x+13,y+8,3,3);ctx.fillStyle='#daa010';ctx.fillRect(x+8,y+14,8,6);ctx.fillStyle='#000';ctx.fillRect(x+9,y+15,2,2);ctx.fillRect(x+13,y+15,2,2);}
  else if(it.type==='fire'){const fc=['#ff4400','#ff8800','#ffcc00','#ff0080'];for(let p=0;p<4;p++){const a=(p/4)*Math.PI*2+it.bt*0.05;ctx.fillStyle=fc[p];ctx.beginPath();ctx.arc(x+12+Math.cos(a)*7,y+10+Math.sin(a)*5,4,0,Math.PI*2);ctx.fill();}ctx.fillStyle='#00a800';ctx.fillRect(x+10,y+12,4,10);ctx.fillRect(x+7,y+16,10,3);}
  else if(it.type==='star'){const pulse=0.8+0.2*Math.sin(it.bt*0.2);ctx.save();ctx.translate(x+12,y+12);ctx.scale(pulse,pulse);ctx.fillStyle='#ffd700';ctx.beginPath();for(let i=0;i<5;i++){const oa=(i*4*Math.PI/5)-Math.PI/2,ia=oa+2*Math.PI/10;const ox=Math.cos(oa)*11,oy=Math.sin(oa)*11,ix=Math.cos(ia)*5,iy=Math.sin(ia)*5;if(i===0)ctx.moveTo(ox,oy);else ctx.lineTo(ox,oy);ctx.lineTo(ix,iy);}ctx.closePath();ctx.fill();ctx.fillStyle='rgba(255,255,255,0.5)';ctx.fillRect(-3,-8,6,4);ctx.restore();}
}

function drawPlatform(p){
  const t=theme;
  if(p.type==='vine'){ctx.fillStyle='#3a8020';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.fillStyle='#60c040';ctx.fillRect(p.x+4,p.y+2,p.width-8,6);return;}
  if(p.type==='cloud'){ctx.fillStyle='rgba(255,255,255,0.95)';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.strokeStyle='rgba(180,200,255,0.5)';ctx.lineWidth=2;ctx.strokeRect(p.x,p.y,p.width,p.height);return;}
  if(p.type==='question'){ctx.fillStyle=p.hit?'#888':'#fc9c00';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.strokeStyle='#000';ctx.lineWidth=2;ctx.strokeRect(p.x,p.y,p.width,p.height);ctx.fillStyle=p.hit?'#555':'#fff';ctx.font='bold 18px monospace';ctx.fillText(p.hit?'!':'?',p.x+p.width/2-6,p.y+p.height/2+6);if(!p.hit&&Math.floor(titleFrame/8)%2===0){ctx.fillStyle='rgba(255,255,200,0.5)';ctx.fillRect(p.x+2,p.y+2,4,4);}return;}
  if(p.type==='ground'){
    const al=p.crumble?(1-Math.min(p.cTimer||0,25)/30):1;ctx.globalAlpha=al;
    if(t==='desert'){ctx.fillStyle='#c8842a';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.fillStyle='#e8a040';ctx.fillRect(p.x,p.y,p.width,8);ctx.fillStyle='#b06018';ctx.fillRect(p.x,p.y+12,p.width,4);}
    else if(t==='ice'){ctx.fillStyle='#5080a8';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.fillStyle='#a8d0f0';ctx.fillRect(p.x,p.y,p.width,8);ctx.fillStyle='rgba(255,255,255,0.5)';ctx.fillRect(p.x+4,p.y+2,p.width-8,3);}
    else if(t==='castle'||t==='lava'){ctx.fillStyle='#4a2020';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.fillStyle='#6a3030';ctx.fillRect(p.x,p.y,p.width,6);ctx.fillStyle='#3a1010';for(let bx=p.x;bx<p.x+p.width;bx+=32)ctx.fillRect(bx,p.y+6,1,p.height-6);}
    else if(t==='underwater'){ctx.fillStyle='#204060';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.fillStyle='#306080';ctx.fillRect(p.x,p.y,p.width,6);}
    else if(t==='forest'){ctx.fillStyle='#3a2010';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.fillStyle='#5a4020';ctx.fillRect(p.x,p.y,p.width,6);ctx.fillStyle='#2a600a';ctx.fillRect(p.x,p.y,p.width,4);}
    // 夜ワールド(W7)
    else if(t==='night'){ctx.fillStyle='#1a1a3a';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.fillStyle='#2a2a5a';ctx.fillRect(p.x,p.y,p.width,6);ctx.fillStyle='rgba(100,100,200,0.3)';ctx.fillRect(p.x+2,p.y+2,p.width-4,2);}
    // 月ワールド(W8)
    else if(t==='moon'){ctx.fillStyle='#606060';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.fillStyle='#909090';ctx.fillRect(p.x,p.y,p.width,6);ctx.fillStyle='rgba(200,200,200,0.2)';for(let bx=p.x;bx<p.x+p.width;bx+=40){ctx.beginPath();ctx.arc(bx+10,p.y+3,5,Math.PI,0);ctx.fill();}}
    else{ctx.fillStyle='#d84000';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.fillStyle='#fc9c00';ctx.fillRect(p.x,p.y,p.width,6);}
    ctx.globalAlpha=1;
  }
  else if(p.type==='block'){
    if(t==='castle'||t==='lava'){ctx.fillStyle='#5a2828';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.strokeStyle='#8a4040';ctx.lineWidth=2;ctx.strokeRect(p.x,p.y,p.width,p.height);}
    else if(t==='ice'){ctx.fillStyle='#80b8e0';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.strokeStyle='#c0e0ff';ctx.lineWidth=2;ctx.strokeRect(p.x,p.y,p.width,p.height);}
    else if(t==='desert'){ctx.fillStyle='#d4a060';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.strokeStyle='#a87040';ctx.lineWidth=2;ctx.strokeRect(p.x,p.y,p.width,p.height);}
    else if(t==='forest'){ctx.fillStyle='#6a4020';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.strokeStyle='#8a6040';ctx.lineWidth=2;ctx.strokeRect(p.x,p.y,p.width,p.height);}
    else{ctx.fillStyle='#fc9c00';ctx.fillRect(p.x,p.y,p.width,p.height);ctx.strokeStyle='#000';ctx.lineWidth=2;ctx.strokeRect(p.x,p.y,p.width,p.height);}
  }
  else if(p.type==='pipe'){
    const c1=(t==='castle'||t==='lava')?'#6a0000':(t==='ice'?'#4080c0':'#00a800');
    const c2=(t==='castle'||t==='lava')?'#aa2020':(t==='ice'?'#80c0ff':'#80d010');
    ctx.fillStyle=c1;ctx.fillRect(p.x,p.y,p.width,p.height);ctx.fillStyle=c2;ctx.fillRect(p.x+4,p.y,12,p.height);ctx.strokeStyle='#000';ctx.strokeRect(p.x,p.y,p.width,p.height);
  }
}

function drawBG(){
  const t=theme;
  if(t==='underwater'){const g=ctx.createLinearGradient(0,0,0,H);g.addColorStop(0,'#001a40');g.addColorStop(1,'#003366');ctx.fillStyle=g;ctx.fillRect(0,0,W,H);ctx.fillStyle='rgba(100,200,255,0.12)';for(let i=0;i<12;i++){ctx.beginPath();ctx.arc((i*97+titleFrame*0.3)%W,(titleFrame*0.4+i*60)%H,4+i%4,0,Math.PI*2);ctx.fill();}}
  else if(t==='desert'){const g=ctx.createLinearGradient(0,0,0,H);g.addColorStop(0,'#e8a040');g.addColorStop(0.5,'#d4882a');g.addColorStop(1,'#c06010');ctx.fillStyle=g;ctx.fillRect(0,0,W,H);}
  else if(t==='ice'){const g=ctx.createLinearGradient(0,0,0,H);g.addColorStop(0,'#c8e8ff');g.addColorStop(1,'#8ab4d8');ctx.fillStyle=g;ctx.fillRect(0,0,W,H);ctx.fillStyle='rgba(255,255,255,0.7)';for(let i=0;i<15;i++){ctx.fillRect((i*131+titleFrame*0.5)%W,(titleFrame*0.8+i*50)%(H-60),3,3);}}
  else if(t==='castle'){const g=ctx.createLinearGradient(0,0,0,H);g.addColorStop(0,'#1a0020');g.addColorStop(1,'#3a0010');ctx.fillStyle=g;ctx.fillRect(0,0,W,H);for(let i=0;i<8;i++){const fx=((i*200+cameraX*0.05)%(W+200))-100,fy=GY-30+Math.sin(titleFrame*0.1+i)*10;ctx.fillStyle=`rgba(255,${80+Math.floor(Math.sin(titleFrame*0.1+i)*40)},0,0.4)`;ctx.beginPath();ctx.ellipse(fx,fy,10,20,0,0,Math.PI*2);ctx.fill();}}
  else if(t==='forest'){const g=ctx.createLinearGradient(0,0,0,H);g.addColorStop(0,'#0a2010');g.addColorStop(1,'#153020');ctx.fillStyle=g;ctx.fillRect(0,0,W,H);ctx.fillStyle='#0d2810';for(let i=0;i<8;i++){const tx=((i*180-cameraX*0.2+W*2)%(W+200));ctx.beginPath();ctx.moveTo(tx,GY);ctx.lineTo(tx+40,GY-120);ctx.lineTo(tx+80,GY);ctx.fill();ctx.beginPath();ctx.moveTo(tx+10,GY-70);ctx.lineTo(tx+40,GY-180);ctx.lineTo(tx+70,GY-70);ctx.fill();}}
  else if(t==='sky'){const g=ctx.createLinearGradient(0,0,0,H);g.addColorStop(0,'#4080ff');g.addColorStop(0.6,'#80c0ff');g.addColorStop(1,'#c0e8ff');ctx.fillStyle=g;ctx.fillRect(0,0,W,H);ctx.fillStyle='rgba(255,255,255,0.9)';for(let i=0;i<6;i++){const ccx=((i*240-cameraX*0.15+W*2)%(W+300)),cy=50+i*40,sc=2+i*0.3;ctx.save();ctx.translate(ccx,cy);ctx.scale(sc,sc);ctx.beginPath();ctx.arc(0,0,20,0,Math.PI*2);ctx.arc(25,-8,25,0,Math.PI*2);ctx.arc(50,0,20,0,Math.PI*2);ctx.fill();ctx.restore();}}
  else if(t==='lava'){const g=ctx.createLinearGradient(0,0,0,H);g.addColorStop(0,'#200000');g.addColorStop(0.7,'#500010');g.addColorStop(1,'#800000');ctx.fillStyle=g;ctx.fillRect(0,0,W,H);for(let i=0;i<6;i++){const sx=((i*180+cameraX*0.05)%(W+180)),sy=GY-40-Math.sin(titleFrame*0.05+i)*20;ctx.fillStyle=`rgba(80,20,0,${0.2+0.1*Math.sin(titleFrame*0.05+i)})`;ctx.beginPath();ctx.ellipse(sx,sy,20,30,0,0,Math.PI*2);ctx.fill();}}
  // W7夜: 暗い夜空＋月
  else if(t==='night'){
    const g=ctx.createLinearGradient(0,0,0,H);g.addColorStop(0,'#020210');g.addColorStop(0.6,'#0a0a30');g.addColorStop(1,'#0a1020');ctx.fillStyle=g;ctx.fillRect(0,0,W,H);
    ctx.fillStyle='#fff';for(let i=0;i<40;i++){const sx=((i*137+cameraX*0.05)%(W+100)),sy=(i*71)%200,br=0.5+0.5*Math.sin(titleFrame*0.06+i);ctx.globalAlpha=br;ctx.fillRect(sx,sy,i%3===0?2:1,i%3===0?2:1);}ctx.globalAlpha=1;
    const mx=((700-cameraX*0.02+W*2)%(W+200));ctx.fillStyle='#ffffc0';ctx.beginPath();ctx.arc(mx,60,28,0,Math.PI*2);ctx.fill();ctx.fillStyle='rgba(20,20,60,0.7)';ctx.beginPath();ctx.arc(mx+10,55,24,0,Math.PI*2);ctx.fill();
  }
  // W8月: 宇宙＋月面グレー地面
  else if(t==='moon'){
    const g=ctx.createLinearGradient(0,0,0,H);g.addColorStop(0,'#000008');g.addColorStop(0.65,'#050510');g.addColorStop(0.66,'#404040');g.addColorStop(1,'#505050');ctx.fillStyle=g;ctx.fillRect(0,0,W,H);
    ctx.fillStyle='#fff';for(let i=0;i<50;i++){const sx=((i*113+cameraX*0.02)%(W+100)),sy=(i*53)%280,br=0.3+0.7*Math.sin(titleFrame*0.04+i);ctx.globalAlpha=br;ctx.fillRect(sx,sy,1,1);}ctx.globalAlpha=1;
    ctx.fillStyle='rgba(0,100,200,0.6)';ctx.beginPath();ctx.arc(650,60,36,0,Math.PI*2);ctx.fill();ctx.fillStyle='rgba(0,180,80,0.5)';ctx.beginPath();ctx.arc(638,55,14,0,Math.PI*2);ctx.fill();ctx.strokeStyle='rgba(100,200,255,0.3)';ctx.lineWidth=3;ctx.beginPath();ctx.arc(650,60,36,0,Math.PI*2);ctx.stroke();
    [120,240,380,500,640].forEach((ccx,i)=>{const ccy=GY-15+(i%2)*8;ctx.strokeStyle='rgba(100,100,100,0.4)';ctx.lineWidth=2;ctx.beginPath();ctx.ellipse(ccx,ccy,16+i*4,6+i,0,0,Math.PI*2);ctx.stroke();});
  }
  else{ctx.fillStyle='#5c94fc';ctx.fillRect(0,0,W,H);ctx.fillStyle='rgba(255,255,255,0.8)';const co=-(cameraX*0.3)%(W+200);[[co+100,60,3],[co+300,80,2],[co+550,50,4],[co+750,70,2.5]].forEach(([ccx,cy,sc])=>{const cx2=((ccx%(W+200))+W+200)%(W+200);ctx.save();ctx.translate(cx2,cy);ctx.scale(sc,sc);ctx.beginPath();ctx.arc(0,0,14,0,Math.PI*2);ctx.arc(18,-5,18,0,Math.PI*2);ctx.arc(36,0,14,0,Math.PI*2);ctx.fill();ctx.restore();});}
}

function drawLava(){
  const lavaY=gimmick.lava||(GY+10);
  ctx.fillStyle='#ff4400';ctx.beginPath();ctx.moveTo(cameraX,H);for(let i=0;i<=20;i++)ctx.lineTo(cameraX+i*(W/20),lavaY+Math.sin(titleFrame*0.08+i*0.8)*6);ctx.lineTo(cameraX+W,H);ctx.closePath();ctx.fill();
  ctx.fillStyle='#ff8800';ctx.beginPath();ctx.moveTo(cameraX,H);for(let i=0;i<=20;i++)ctx.lineTo(cameraX+i*(W/20),lavaY+4+Math.sin(titleFrame*0.1+i*0.6+1)*4);ctx.lineTo(cameraX+W,H);ctx.closePath();ctx.fill();
  ctx.fillStyle='rgba(255,150,0,0.6)';for(let i=0;i<5;i++){const bx=((i*200+titleFrame*2+cameraX)%(W*2))+cameraX-200;ctx.beginPath();ctx.arc(bx,lavaY-5+Math.sin(titleFrame*0.1+i)*3,3+i%3,0,Math.PI*2);ctx.fill();}
}

// ========== タイトル画面 ==========
function drawTitle(){
  ctx.fillStyle='#050518';ctx.fillRect(0,0,W,H);
  ctx.fillStyle='#fff';
  [[60,30],[150,50],[300,20],[450,40],[600,25],[720,55],[80,90],[200,70],[380,85],[520,65],[680,80],[740,35],[120,110],[420,15]].forEach(([sx,sy])=>{ctx.globalAlpha=0.5+0.5*Math.sin(titleFrame*0.05+sx);ctx.fillRect(sx,sy,2,2);});ctx.globalAlpha=1;
  ctx.fillStyle='#1a1a4a';ctx.beginPath();ctx.moveTo(0,380);ctx.lineTo(80,280);ctx.lineTo(160,380);ctx.lineTo(200,300);ctx.lineTo(280,380);ctx.lineTo(400,240);ctx.lineTo(520,380);ctx.lineTo(600,290);ctx.lineTo(680,380);ctx.lineTo(720,310);ctx.lineTo(800,380);ctx.lineTo(800,H);ctx.lineTo(0,H);ctx.fill();
  ctx.fillStyle='#d84000';ctx.fillRect(0,GY,W,60);ctx.fillStyle='#fc9c00';ctx.fillRect(0,GY,W,6);
  ctx.fillStyle='#000c';ctx.fillRect(110,70,580,100);ctx.strokeStyle='#fc9c00';ctx.lineWidth=4;ctx.strokeRect(110,70,580,100);
  const pulse=1+0.05*Math.sin(titleFrame*0.08);ctx.save();ctx.translate(400,120);ctx.scale(pulse,pulse);ctx.fillStyle='#fc9c00';ctx.font='bold 50px monospace';ctx.textAlign='center';ctx.fillText('SUPER MARIO',0,0);ctx.restore();
  ctx.fillStyle='#fff';ctx.font='bold 21px monospace';ctx.textAlign='center';ctx.fillText('WORLD WARP DELUXE',400,160);
  ctx.fillStyle='#aaa';ctx.font='12px monospace';ctx.fillText('★ 1〜6+EX通常 / HARD: +W7夜 +W8月 +W9宇宙シューティング ★',400,178);
  if(hardMode){ctx.fillStyle='#ff4444';ctx.font='bold 14px monospace';ctx.fillText('🔥 HARD MODE ACTIVE 🔥',400,194);}
  ctx.textAlign='left';
  const btns=[{sel:titleSel===1,x:180,label:'1 PLAYER',key:'[1]',sp:[sIdle],lx:190,ly:248},{sel:titleSel===2,x:330,label:'2 PLAYERS',key:'[2]',sp:[sIdle,lIdle],lx:338,ly:248},{sel:titleSel===3,x:490,label:'⚔ BATTLE',key:'[3]',sp:[],lx:500,ly:248}];
  btns.forEach(b=>{ctx.fillStyle=b.sel?'#fc9c00':'#444';ctx.fillRect(b.x,228,130,55);ctx.strokeStyle=b.sel?'#fff':'#333';ctx.lineWidth=2;ctx.strokeRect(b.x,228,130,55);b.sp.forEach((s,i)=>drawSpr(s,b.x+5+i*28,228,2,'right'));ctx.fillStyle=b.sel?'#000':'#aaa';ctx.font='bold 13px monospace';ctx.fillText(b.label,b.lx,268);ctx.font='11px monospace';ctx.fillText(b.key,b.lx,283);});
  if(Math.floor(titleBlink/20)%2===0){ctx.fillStyle='#fff';ctx.font='bold 18px monospace';ctx.textAlign='center';ctx.fillText('- PRESS SPACE TO START -',400,325);}
  if(demoMode){ctx.fillStyle='rgba(0,0,0,0.4)';ctx.fillRect(0,0,W,H);ctx.fillStyle='#fff';ctx.font='bold 12px monospace';ctx.textAlign='center';ctx.fillText('DEMO - Press any key',400,16);drawSpr([sRun1,sRun2,sRun3][demoP.fr],demoP.x,demoP.y,3,'right');ctx.fillStyle='#fc9c00';for(let i=0;i<5;i++)ctx.fillRect((demoP.x+120+i*130)%W,GY-80,16,24);}
  ctx.fillStyle='#777';ctx.font='12px monospace';ctx.textAlign='center';
  ctx.fillText('P1: Arrow+Space+Shift/X(Dash/Fire)+Z  P2: WASD+Q+E  MAP: S=Save L=Load',400,370);
  ctx.fillText('W1〜6全クリ→EXTRA  EXTRAクリア→HARD  HARD+W6クリア→W7夜→W8月→W9宇宙',400,387);
  ctx.fillText('DEBUG: 0913と入力  BATTLE: [3]キー',400,404);
  ctx.textAlign='left';
}

// ========== ワールドマップ画面 ==========
function drawMap(){
  // W9: 宇宙シューティングMAP
  if(world===9){
    ctx.fillStyle='#00000c';ctx.fillRect(0,0,W,H);
    ctx.fillStyle='#fff';for(let i=0;i<50;i++){ctx.globalAlpha=0.5;ctx.fillRect((i*97+titleFrame*0.2)%W,(i*63+titleFrame*0.3)%H,1.5,1.5);}ctx.globalAlpha=1;
    ctx.fillStyle='#8888ff';ctx.font='bold 26px monospace';ctx.fillText('WORLD 9 - SPACE SHOOTING',100,55);
    ctx.fillStyle='#aaa';ctx.font='14px monospace';ctx.fillText('Arrow:Select  SPACE:Enter  (Hard Mode専用)',160,85);
    if(!hardMode){ctx.fillStyle='#ff4444';ctx.font='bold 16px monospace';ctx.textAlign='center';ctx.fillText('⚠ HARD MODE でW8クリア後に解放',400,250);ctx.textAlign='left';return;}
    SPACE_ST.forEach((s,i)=>{
      ctx.fillStyle=s.cleared?'#5c94fc':'#2a2a80';ctx.fillRect(s.cx,s.cy,64,50);
      ctx.strokeStyle=i===mapIdx?'#fff':'#8888ff';ctx.lineWidth=i===mapIdx?3:1;ctx.strokeRect(s.cx,s.cy,64,50);
      ctx.fillStyle='#fff';ctx.font='bold 12px monospace';ctx.fillText(s.name,s.cx+6,s.cy+30);
      if(s.cleared){ctx.fillStyle='#80d010';ctx.font='bold 11px monospace';ctx.fillText('★OK',s.cx+14,s.cy-7);}
    });
    const cs=SPACE_ST[mapIdx];
    ctx.fillStyle='#4488ff';ctx.fillRect(cs.cx+12,cs.cy-30,16,12);ctx.fillStyle='#ff8800';ctx.fillRect(cs.cx+14,cs.cy-20,12,6);
    const prev=mapIdx===0||SPACE_ST[mapIdx-1].cleared;
    ctx.fillStyle='#fff';ctx.font='bold 14px monospace';ctx.fillText(prev?`👉 [${cs.name}]: SPACE`:'🔒 Clear previous',160,370);
    return;
  }

  // 通常ワールドMAP
  const wc={1:'#111',2:'#1a0e00',3:'#001420',4:'#081408',5:'#000820',6:'#200000',7:'#000010',8:'#202030'};
  ctx.fillStyle=wc[world]||'#111';ctx.fillRect(0,0,W,H);

  // W7/W8はハードモード専用表示
  if((world===7||world===8)&&!hardMode){
    ctx.fillStyle='#ff4444';ctx.font='bold 18px monospace';ctx.textAlign='center';
    ctx.fillText('⚠ HARD MODE専用ワールド',400,220);ctx.textAlign='left';return;
  }

  const wn={1:'WORLD 1 - GRASSLAND',2:'WORLD 2 - DESERT',3:'WORLD 3 - ICE LAND',4:'WORLD 4 - FOREST',5:'WORLD 5 - SKY',6:'WORLD 6 - LAVA',7:'WORLD 7 - NIGHT 🌙',8:'WORLD 8 - MOON 🌕'};
  const wnc={1:'#80d010',2:'#e8a040',3:'#80c8ff',4:'#40c040',5:'#80c0ff',6:'#ff6040',7:'#8888ff',8:'#c0c0c0'};
  ctx.fillStyle=wnc[world]||'#fff';ctx.font='bold 24px monospace';ctx.fillText(wn[world]||`WORLD ${world}`,150,52);
  ctx.fillStyle='#aaa';ctx.font='13px monospace';ctx.fillText('Arrow:Select  SPACE:Enter  1/2:Mode  S:Save  L:Load',150,78);
  ctx.fillStyle=numP===2?'#0f0':'#fc9c00';ctx.font='bold 13px monospace';ctx.fillText(`[${numP}P${hardMode?' HARD':''}]`,660,78);
  if(gameCompleted){ctx.fillStyle='#ffd700';ctx.font='bold 12px monospace';ctx.fillText('★ ALL CLEAR! EXTRA UNLOCKED!',150,97);}
  if(hardMode){ctx.fillStyle='#ff4444';ctx.font='bold 12px monospace';ctx.fillText('🔥HARD 残機:'+lives,540,97);}
  ctx.fillStyle='#fff';ctx.font='bold 12px monospace';ctx.fillText('❤×'+lives,150,97+(gameCompleted?18:0));

  const ws=getCurStages();
  // 道
  ctx.strokeStyle='#555';ctx.lineWidth=4;ctx.setLineDash([6,6]);
  ctx.beginPath();if(ws.length>0){ctx.moveTo(ws[0].canvasX+30,ws[0].canvasY+25);ws.slice(1).forEach(s=>ctx.lineTo(s.canvasX+30,s.canvasY+25));}
  ctx.stroke();ctx.setLineDash([]);

  ws.forEach((s,i)=>{
    const acc=(i===0||ws[i-1].cleared);
    const ic=(s.type==='castle'||s.type==='night_castle'||s.type==='moon_castle');
    const iw=s.type==='underwater',ie=s.name==='EXTRA',isk=s.type==='sky',il=s.type==='lava';
    const inight=(s.type==='night'||s.type==='night_castle');
    const imoon=(s.type==='moon'||s.type==='moon_castle');
    if(s.cleared)ctx.fillStyle='#5c94fc';
    else if(acc){
      if(ic&&inight)ctx.fillStyle='#1a1a60';
      else if(ic&&imoon)ctx.fillStyle='#404060';
      else if(ic)ctx.fillStyle='#8a2020';
      else if(iw)ctx.fillStyle='#204080';
      else if(ie)ctx.fillStyle='#8800aa';
      else if(isk)ctx.fillStyle='#204888';
      else if(il)ctx.fillStyle='#882000';
      else if(inight)ctx.fillStyle='#1a1a50';
      else if(imoon)ctx.fillStyle='#505060';
      else ctx.fillStyle='#fc9c00';
    }else ctx.fillStyle='#444';
    ctx.fillRect(s.canvasX,s.canvasY,64,50);
    ctx.strokeStyle=i===mapIdx?'#fff':'#888';ctx.lineWidth=i===mapIdx?3:1;ctx.strokeRect(s.canvasX,s.canvasY,64,50);
    ctx.fillStyle=acc?'#fff':'#777';ctx.font='bold 11px monospace';
    const lbl=s.name==='TUTORIAL'?'TUTR':s.name==='EXTRA'?'EX!':s.name;
    ctx.fillText(lbl,s.canvasX+4,s.canvasY+30);
    const sub=ic?'CASTLE':iw?'WATER':isk?'SKY':il?'LAVA':ie?'EXTRA':inight?'NIGHT':imoon?'MOON':'';
    if(sub){ctx.fillStyle='#ffaaaa';ctx.font='9px monospace';ctx.fillText(sub,s.canvasX+4,s.canvasY+44);}
    if(s.cleared){ctx.fillStyle='#80d010';ctx.font='bold 11px monospace';ctx.fillText('★OK',s.canvasX+14,s.canvasY-7);}
  });
  // ゴールの旗
  if(ws.length>0){const ls=ws[ws.length-1];ctx.fillStyle='#00a800';ctx.fillRect(ls.canvasX+120,ls.canvasY-10,50,60);ctx.fillStyle='#80d010';ctx.fillRect(ls.canvasX+124,ls.canvasY-10,10,60);ctx.strokeStyle='#fff';ctx.strokeRect(ls.canvasX+120,ls.canvasY-10,50,60);}
  // カーソル
  const as=ws[mapIdx];if(!as)return;
  ctx.strokeStyle='#fff';ctx.lineWidth=3;ctx.strokeRect(as.canvasX-4,as.canvasY-4,72,58);
  drawSpr(sIdle,as.canvasX+8,as.canvasY-44,3,'right');
  if(numP===2)drawSpr(lIdle,as.canvasX+26,as.canvasY-40,2,'right');
  ctx.fillStyle='#fff';ctx.font='bold 15px monospace';
  const ce=(mapIdx===0||ws[mapIdx-1].cleared);
  ctx.fillText(ce?`👉 [${as.name}]: SPACE`:'🔒 LOCKED',150,365);
  const wg=getGimmick(world,as.type);
  const gn={crumble:'⚠崩れる足場',current:'🌊水流',ice:'❄氷スライド',vine:'🌿つる足場',wind:'💨風',lavarise:'🌋溶岩上昇(緩)',goombarush:'👾クリボーラッシュ！',lowgrav:'🌙低重力ジャンプ3倍',darkfog:'🌑暗視エフェクト',moonquake:'🌕月震'};
  if(wg){ctx.fillStyle='#ffd700';ctx.font='12px monospace';ctx.fillText('GIMMICK: '+(gn[wg]||wg),150,385);}
}

// ========== ワープアニメ ==========
// warpSt: 0=走る 1=ジャンプ(放物線) 2=土管上で待機 3=土管に沈む 4=暗転して遷移
function drawWarp(){
  ctx.fillStyle='#111';ctx.fillRect(0,0,W,H);
  // 地面
  ctx.fillStyle='#d84000';ctx.fillRect(0,GY,W,60);ctx.fillStyle='#fc9c00';ctx.fillRect(0,GY,W,6);
  // 次のワールド計算
  let nextW=world+1;
  if(world===6&&!hardMode)nextW=1;  // 通常W6→W1
  if(world===9)nextW=1;             // W9→最初に戻る
  ctx.fillStyle='#fff';ctx.font='bold 26px monospace';ctx.textAlign='center';
  if(warpSt<=2)ctx.fillText(`WORLD ${world} CLEAR!  WARPING...`,400,90);
  else ctx.fillText(`NEXT → WORLD ${nextW}`,400,90);
  ctx.textAlign='left';
  // 土管
  const tX=530,tPTop=GY-120,tPH=120,tPW=72;
  ctx.fillStyle='#00a800';ctx.fillRect(tX,tPTop+18,tPW,tPH-18);
  ctx.fillStyle='#80d010';ctx.fillRect(tX+4,tPTop+18,14,tPH-18);
  ctx.fillStyle='#006000';ctx.fillRect(tX+tPW-8,tPTop+18,8,tPH-18);
  ctx.strokeStyle='#003800';ctx.lineWidth=2;ctx.strokeRect(tX,tPTop+18,tPW,tPH-18);
  // 土管口
  ctx.fillStyle='#00a800';ctx.fillRect(tX-4,tPTop,tPW+8,22);
  ctx.fillStyle='#80d010';ctx.fillRect(tX,tPTop+2,14,18);
  ctx.fillStyle='#006000';ctx.fillRect(tX+tPW-4,tPTop,8,22);
  ctx.strokeStyle='#003800';ctx.lineWidth=2;ctx.strokeRect(tX-4,tPTop,tPW+8,22);
  // マリオ位置
  const mW=48,mSz=3;
  const runEnd=tX+tPW/2-mW/2;
  let mX,mY,mSpr=sIdle,mDir='right';
  if(warpSt===0){
    mX=80+warpTimer*(runEnd-80)/85;mY=GY-mW;mSpr=[sRun1,sRun2,sRun3][P1.fr];
  }else if(warpSt===1){
    mX=runEnd;mY=(GY-mW)+warpYOff;mSpr=sJump;
  }else if(warpSt===2){
    mX=runEnd;mY=tPTop-mW;mSpr=sIdle;
    if(warpTimer>10&&warpTimer%30<15)mY-=4;
  }else if(warpSt===3){
    mX=runEnd;mY=tPTop-mW+warpYOff;
  }else{mX=-999;}
  // マリオ描画（沈む時はクリップして土管背面に）
  if(warpSt===3&&mX>-900){
    ctx.save();ctx.beginPath();ctx.rect(tX+2,tPTop+20,tPW-4,tPH);ctx.clip();
    drawSpr(mSpr,mX,mY,mSz,mDir);ctx.restore();
    // 土管口を最前面に再描画
    ctx.fillStyle='#00a800';ctx.fillRect(tX-4,tPTop,tPW+8,24);
    ctx.fillStyle='#80d010';ctx.fillRect(tX,tPTop+2,14,20);
    ctx.fillStyle='#006000';ctx.fillRect(tX+tPW-4,tPTop,8,24);
    ctx.strokeStyle='#003800';ctx.lineWidth=2;ctx.strokeRect(tX-4,tPTop,tPW+8,24);
  }else if(mX>-900){
    drawSpr(mSpr,mX,mY,mSz,mDir);
  }
  // 待機中↓矢印
  if(warpSt===2){
    ctx.fillStyle='#ffff00';ctx.font='bold 28px monospace';ctx.textAlign='center';
    if(Math.floor(warpTimer/12)%2===0)ctx.fillText('↓',tX+tPW/2,tPTop-50);
    ctx.textAlign='left';
  }
  // 暗転
  if(warpSt===4){ctx.fillStyle=`rgba(0,0,0,${Math.min(1,warpTimer/40)})`;ctx.fillRect(0,0,W,H);}
}

// ========== バトル画面 ==========
function drawBattle(){
  if(!BS)return;
  ctx.fillStyle='#0a0a2a';ctx.fillRect(0,0,W,H);
  for(let i=0;i<30;i++){ctx.fillStyle='rgba(255,255,255,0.4)';ctx.fillRect((i*97+titleFrame*0.1)%W,(i*63)%H,1.5,1.5);}
  BPLATS.forEach(p=>{if(p.h>30){ctx.fillStyle='#d84000';ctx.fillRect(p.x,p.y,p.w,p.h);ctx.fillStyle='#fc9c00';ctx.fillRect(p.x,p.y,p.w,6);}else{ctx.fillStyle='#fc9c00';ctx.fillRect(p.x,p.y,p.w,p.h);ctx.strokeStyle='#000';ctx.lineWidth=2;ctx.strokeRect(p.x,p.y,p.w,p.h);}});
  BS.shells.forEach(s=>{ctx.fillStyle='#00a800';ctx.fillRect(s.x,s.y,s.w,s.h);ctx.fillStyle='#80d010';ctx.fillRect(s.x+4,s.y+4,s.w-8,s.h-8);ctx.strokeStyle='#003800';ctx.lineWidth=1;ctx.beginPath();ctx.moveTo(s.x+s.w/2,s.y);ctx.lineTo(s.x+s.w/2,s.y+s.h);ctx.stroke();ctx.beginPath();ctx.moveTo(s.x,s.y+s.h/2);ctx.lineTo(s.x+s.w,s.y+s.h/2);ctx.stroke();});
  function drawBP(bp,isL){if(bp.stun>0&&Math.floor(bp.stun/5)%2===0)return;const runs=[isL?lRun1:sRun1,isL?lRun2:sRun2,isL?lRun3:sRun3];const sp=bp.grnd&&Math.abs(bp.vx)>0.5?runs[bp.fr]:(isL?lIdle:sIdle);drawSpr(sp,bp.x,bp.y,3,bp.dir);}
  drawBP(BS.p1,false);drawBP(BS.p2,true);
  ctx.font='bold 17px monospace';ctx.fillStyle='#ff4444';ctx.fillText('P1 '+'❤'.repeat(BS.p1.hp)+'♡'.repeat(3-BS.p1.hp),10,26);ctx.fillStyle='#44ff44';ctx.fillText('P2 '+'❤'.repeat(BS.p2.hp)+'♡'.repeat(3-BS.p2.hp),560,26);
  ctx.fillStyle='#ffd700';ctx.font='bold 14px monospace';ctx.textAlign='center';ctx.fillText('⚔ BATTLE MODE',400,26);
  ctx.fillStyle='#888';ctx.font='12px monospace';ctx.fillText('P1:Arrow+Shift/X=Shell  P2:WASD+Q=Shell',400,430);ctx.textAlign='left';
  if(BS.cd>0){ctx.fillStyle='rgba(0,0,0,0.5)';ctx.fillRect(0,0,W,H);ctx.fillStyle='#fff';ctx.font='bold 70px monospace';ctx.textAlign='center';const c=Math.ceil(BS.cd/40);ctx.fillText(c>0?String(c):'FIGHT!',400,250);ctx.textAlign='left';}
  if(BS.result){ctx.fillStyle='rgba(0,0,0,0.6)';ctx.fillRect(0,0,W,H);ctx.fillStyle='#ffd700';ctx.font='bold 44px monospace';ctx.textAlign='center';ctx.fillText({p1win:'MARIO WINS!',p2win:'LUIGI WINS!',draw:'DRAW!'}[BS.result],400,200);ctx.fillStyle='#fff';ctx.font='bold 16px monospace';ctx.fillText('SPACE = Title',400,260);ctx.textAlign='left';}
}

// ========== 宇宙シューティング画面 ==========
function drawSpace(){
  if(!SP)return;
  ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
  SP.stars.forEach(s=>{ctx.fillStyle='#fff';ctx.globalAlpha=0.6+0.4*Math.random();ctx.fillRect(s.x,s.y,s.r,s.r);});ctx.globalAlpha=1;
  const ship=SP.ship;
  ctx.fillStyle='#4488ff';ctx.beginPath();ctx.moveTo(ship.x+20,ship.y);ctx.lineTo(ship.x,ship.y+ship.h);ctx.lineTo(ship.x+ship.w,ship.y+ship.h);ctx.closePath();ctx.fill();
  ctx.fillStyle='#88aaff';ctx.fillRect(ship.x+12,ship.y+4,16,14);
  if(ship.iframes>0){ctx.fillStyle='rgba(255,255,255,0.3)';ctx.beginPath();ctx.arc(ship.x+20,ship.y+15,28,0,Math.PI*2);ctx.fill();}
  SP.bullets.forEach(b=>{ctx.fillStyle='#ffff44';ctx.fillRect(b.x,b.y,b.w,b.h);ctx.fillStyle='#ffffff';ctx.fillRect(b.x+1,b.y,b.w-2,4);});
  SP.eBullets.forEach(b=>{ctx.fillStyle='#ff6644';ctx.beginPath();ctx.arc(b.x+4,b.y+4,5,0,Math.PI*2);ctx.fill();});
  SP.enemies.forEach(e=>{
    if(!e.alive)return;
    const col=e.type==='straight'?'#ff4444':e.type==='zigzag'?'#ff8800':'#aa00ff';
    ctx.fillStyle=col;ctx.beginPath();ctx.moveTo(e.x+18,e.y+e.h);ctx.lineTo(e.x,e.y);ctx.lineTo(e.x+e.w,e.y);ctx.closePath();ctx.fill();
    ctx.fillStyle='rgba(255,255,0,0.8)';ctx.fillRect(e.x+12,e.y+6,12,8);
    ctx.fillStyle='#ff8800';ctx.fillRect(e.x+10,e.y+ship.h,8,8);ctx.fillRect(e.x+22,e.y+ship.h,8,8);
    ctx.fillStyle='#ffff00';ctx.fillRect(e.x+12,e.y+ship.h+4,4,6);ctx.fillRect(e.x+24,e.y+ship.h+4,4,6);
  });
  ctx.fillStyle='#fff';ctx.font='bold 15px monospace';ctx.fillText('WORLD 9-'+SP.stage,8,24);
  ctx.fillStyle='#ff6666';ctx.fillText('HP '+'★'.repeat(ship.hp)+'☆'.repeat(3-ship.hp),8,46);
  ctx.fillStyle='#ffff00';ctx.fillText('KILLS: '+SP.kills+'/'+SP.goal,270,24);
  ctx.fillStyle='#aaa';ctx.font='12px monospace';ctx.textAlign='center';ctx.fillText('Arrow=Move  Shift/X/Space=Shoot',400,428);ctx.textAlign='left';
  if(SP.cleared){ctx.fillStyle='rgba(0,0,0,0.6)';ctx.fillRect(0,0,W,H);ctx.fillStyle='#ffd700';ctx.font='bold 40px monospace';ctx.textAlign='center';ctx.fillText('STAGE CLEAR!',400,190);ctx.fillStyle='#fff';ctx.font='bold 17px monospace';ctx.fillText(SP.stage===8?'🏆 W9 ALL CLEAR! SPACE→DEBUG':'SPACE = Next',400,260);ctx.textAlign='left';}
  if(SP.over){ctx.fillStyle='rgba(0,0,0,0.65)';ctx.fillRect(0,0,W,H);ctx.fillStyle='#ff4444';ctx.font='bold 46px monospace';ctx.textAlign='center';ctx.fillText('GAME OVER',400,210);ctx.fillStyle='#fff';ctx.font='bold 17px monospace';ctx.fillText('SPACE = Retry',400,268);ctx.textAlign='left';}
}

// ========== デバッグ画面 ==========
function drawDebug(){
  ctx.fillStyle='#001400';ctx.fillRect(0,0,W,H);ctx.strokeStyle='rgba(0,80,0,0.3)';ctx.lineWidth=1;for(let i=0;i<W;i+=40)ctx.strokeRect(i,0,40,H);
  ctx.fillStyle='#00ff00';ctx.font='bold 20px monospace';ctx.textAlign='center';ctx.fillText('★ DEBUG MODE ★  (0913でいつでも起動)',400,34);ctx.textAlign='left';
  ctx.fillStyle='#0f0';ctx.font='13px monospace';ctx.fillText('↑↓:Select ←→:Change SPACE:Execute [N]:数字操作ON/OFF',60,60);
  ctx.fillStyle=numCtrl?'#ffff00':'#226622';ctx.fillRect(50,64,700,20);ctx.fillStyle=numCtrl?'#000':'#88ff88';ctx.font='bold 12px monospace';ctx.fillText(numCtrl?'🎮 数字操作ON→MAP→ゲーム開始で有効  8=左 9=右 4=ジャンプ 3=ダッシュ':'[N]で数字操作モードON (8←  9→  4↑  3=ダッシュ)',55,79);
  DB_ITEMS.forEach((it,i)=>{const y=88+i*26;const sel=i===dbCursor;ctx.fillStyle=sel?'#ffff00':'#00bb00';ctx.font=(sel?'bold ':'')+'14px monospace';ctx.fillText((sel?'▶ ':'  ')+it.lbl,50,y);ctx.fillStyle=sel?'#fff':'#88ff88';ctx.fillText(String(it.g()),340,y);});
  ctx.fillStyle='#004400';ctx.font='11px monospace';ctx.textAlign='center';ctx.fillText('現在: world='+world+' hard='+hardMode+' lives='+lives,400,433);ctx.textAlign='left';
}

// ========== アクションゲーム画面 ==========
function drawGame(){
  drawBG();
  ctx.save();ctx.translate(-cameraX,0);
  if(underwater){ctx.fillStyle='rgba(0,80,160,0.2)';ctx.fillRect(cameraX,0,W,GY);}
  platforms.forEach(p=>drawPlatform(p));
  ctx.fillStyle='#fff';ctx.fillRect(goal.x+4,goal.y,4,goal.h);ctx.fillStyle='#00a800';ctx.beginPath();ctx.arc(goal.x+6,goal.y,8,0,Math.PI*2);ctx.fill();ctx.fillStyle='#000';ctx.fillRect(goal.x-24,goal.y+20,24,18);
  coins.forEach(c=>{if(!c.col){ctx.fillStyle='#fc9c00';ctx.fillRect(c.x,c.y,c.w,c.h);ctx.strokeStyle='#000';ctx.lineWidth=1;ctx.strokeRect(c.x,c.y,c.w,c.h);}});
  items.forEach(drawItem);enemies.forEach(drawEnemy);
  fireballs.forEach(fb=>{ctx.fillStyle='#ff8800';ctx.beginPath();ctx.arc(fb.x+5,fb.y+5,5,0,Math.PI*2);ctx.fill();ctx.fillStyle='#ffff00';ctx.beginPath();ctx.arc(fb.x+5,fb.y+5,2,0,Math.PI*2);ctx.fill();});
  drawPl(P1);if(numP===2)drawPl(P2);
  ctx.restore();
  if(theme==='lava'||gimmick.lava!==undefined){ctx.save();ctx.translate(-cameraX,0);drawLava();ctx.restore();}
  if(underwater){ctx.fillStyle='rgba(0,120,200,0.1)';ctx.fillRect(0,0,W,H);}
  if(gimmick.wind){const wf=gimmick.wind;ctx.strokeStyle=`rgba(200,230,255,${Math.abs(wf)*0.04})`;ctx.lineWidth=2;for(let i=0;i<8;i++){const wy=40+i*45,wx=(titleFrame*Math.abs(wf)*2+i*120)%(W+100);ctx.beginPath();ctx.moveTo(wf>0?wx:W-wx,wy);ctx.lineTo(wf>0?wx+40:W-wx-40,wy+5);ctx.stroke();}ctx.fillStyle='#80c8ff';ctx.font='bold 12px monospace';ctx.fillText(`💨 ${wf>0?'→':'←'} WIND`,8,H-8);}
  if(hardMode){const vg=ctx.createRadialGradient(400,220,200,400,220,420);vg.addColorStop(0,'rgba(0,0,0,0)');vg.addColorStop(1,'rgba(0,0,0,0.4)');ctx.fillStyle=vg;ctx.fillRect(0,0,W,H);}
  // 夜W7: 懐中電灯エフェクト
  if(theme==='night'){
    const vg=ctx.createRadialGradient(P1.x-cameraX+24,P1.y+24,70,P1.x-cameraX+24,P1.y+24,260);
    vg.addColorStop(0,'rgba(0,0,20,0)');vg.addColorStop(0.5,'rgba(0,0,20,0.6)');vg.addColorStop(1,'rgba(0,0,20,0.93)');
    ctx.fillStyle=vg;ctx.fillRect(0,0,W,H);
  }
  // 月W8: 月震エフェクト
  if(gimmick.moonquake!==undefined&&Math.floor(gimmick.moonquake/60)%8===0){
    const shk=Math.sin(gimmick.moonquake*0.3)*3;
    ctx.fillStyle='rgba(200,200,255,0.07)';ctx.fillRect(0,shk,W,H);
  }
  // クリボーラッシュ警告
  if(gimmick.goombarush&&enemies.filter(e=>e.alive).length>=8){
    ctx.fillStyle='rgba(255,0,0,0.07)';ctx.fillRect(0,0,W,H);
    if(Math.floor(titleFrame/20)%2===0){ctx.fillStyle='#ff4444';ctx.font='bold 13px monospace';ctx.textAlign='center';ctx.fillText('⚠ クリボーラッシュ！',400,H-8);ctx.textAlign='left';}
  }
  // オーバーレイ
  if(P1.dead){ctx.fillStyle='#fff';ctx.font='bold 36px monospace';ctx.textAlign='center';ctx.fillText('GAME OVER',400,180);ctx.font='bold 16px monospace';ctx.fillText((hardMode&&lives<=0)?'GAME OVER - WORLD MAPへ':'PRESS SPACE TO RETRY  残機:'+lives,400,228);ctx.textAlign='left';}
  if(P1.clear){ctx.fillStyle='#fff';ctx.font='bold 36px monospace';ctx.textAlign='center';ctx.fillText('STAGE CLEAR!',400,180);ctx.font='bold 16px monospace';ctx.fillText('PRESS SPACE TO CONTINUE',400,228);ctx.textAlign='left';}
  // HUD
  ctx.fillStyle='#fff';ctx.font='bold 18px monospace';ctx.fillText('MARIO',20,34);ctx.fillText(String(P1.score).padStart(6,'0'),20,56);
  if(numP===2){ctx.fillStyle='#80d010';ctx.font='bold 18px monospace';ctx.fillText('LUIGI',140,34);ctx.fillText(String(P2.score).padStart(6,'0'),140,56);}
  ctx.fillStyle='#fff';ctx.font='bold 18px monospace';ctx.fillText('WORLD',340,34);
  const sd=STAGES.find(s=>s.id===stageId);ctx.fillText((MODE==='TUTORIAL'?(gameCompleted?'EXTRA':'TUTR'):`${world}-${sd?sd.name:'?'}`),340,56);
  const themeLabel={grassland:'GRASS',desert:'DESERT',ice:'ICE',castle:'🏰CASTLE',underwater:'🌊WATER',forest:'🌲FOREST',sky:'☁SKY',lava:'🌋LAVA',night:'🌙NIGHT',moon:'🌕MOON'};
  ctx.fillStyle='#ffd700';ctx.font='bold 12px monospace';ctx.fillText(themeLabel[theme]||'',500,34);
  ctx.fillStyle='#ff8888';ctx.font='bold 13px monospace';ctx.fillText('❤×'+(hardMode?lives:'∞'),660,34);
  if(hardMode&&stageActive){const sl=Math.max(0,70-Math.floor(gameTimer/60));ctx.fillStyle=sl<=15?'#ff4444':'#fff';ctx.font='bold 18px monospace';ctx.textAlign='center';ctx.fillText('TIME '+String(sl).padStart(3,'0'),400,34);ctx.textAlign='left';}
  function pStatus(pl,bx){if(pl.fire){ctx.fillStyle='#ff8800';ctx.font='bold 11px monospace';ctx.fillText('🔥FIRE',bx,70);}else if(pl.star){ctx.fillStyle='#ffd700';ctx.font='bold 11px monospace';ctx.fillText('★STAR',bx,70);}else if(pl.big){ctx.fillStyle='#e00000';ctx.font='bold 11px monospace';ctx.fillText('BIG',bx,70);}else if(pl.mini){ctx.fillStyle='#ffe040';ctx.font='bold 11px monospace';ctx.fillText('MINI',bx,70);}}
  pStatus(P1,20);if(numP===2)pStatus(P2,140);
}

// ========== メインループ ==========
function update(){
  titleFrame++;titleBlink++;
  const bk=MODE==='TITLE'?'title':MODE==='BATTLE'?'battle':MODE==='SPACE'?'space':(MODE==='WORLD_MAP'?'grassland':null);
  if(bk&&bk!==bgmKey)bgmPlay(bk);

  if(MODE==='TITLE'){
    demoTimer++;if(demoTimer>300)demoMode=true;
    if(demoMode){demoP.vy+=GR;demoP.x+=demoP.vx;demoP.y+=demoP.vy;if(demoP.y>=GY-48){demoP.y=GY-48;demoP.vy=-12;}if(demoP.x>W+100)demoP.x=-100;demoP.ft++;if(demoP.ft>6){demoP.fr=(demoP.fr+1)%3;demoP.ft=0;}}
    return;
  }
  if(MODE==='BATTLE'){updateBattle();return;}
  if(MODE==='SPACE'){updateSpace();return;}
  if(MODE==='DEBUG')return;
  if(MODE==='WARP_ANIMATION'){
    warpTimer++;
    // 走る
    if(warpSt===0){
      if(warpTimer%4===0)P1.fr=(P1.fr+1)%3;
      if(warpTimer>85){warpSt=1;warpTimer=0;}
    }
    // ジャンプ放物線
    else if(warpSt===1){
      warpYOff=-(warpTimer*5.2-warpTimer*warpTimer*0.18);
      if(warpTimer>28){warpYOff=0;warpSt=2;warpTimer=0;}
    }
    // 土管上で待機
    else if(warpSt===2){
      if(warpTimer>40){warpSt=3;warpTimer=0;}
    }
    // 土管に沈む
    else if(warpSt===3){
      warpYOff=warpTimer*2.4;
      if(warpTimer>50){warpSt=4;warpTimer=0;}
    }
    // 暗転→ワールド遷移
    else if(warpSt===4){
      if(warpTimer>40){
        world++;
        if(world===7&&!hardMode)world=1; // 通常モードはW6→W1に戻る
        if(world===10)world=1;           // W9終了後は最初から
        mapIdx=0;MODE='WORLD_MAP';bgmStop();
      }
    }
    return;
  }
  if(MODE==='WORLD_MAP')return;
  if(stageActive&&hardMode&&!P1.dead&&!P1.clear){gameTimer++;if(gameTimer>=70*60){P1.dead=true;P1.vy=-11;K1.r=K1.l=K1.j=K1.d=false;}}
  updateGimmick();updateEnemies();updateItems();updateFBs();
  updatePlayer(P1,K1,numP===2?P2:null);
  if(numP===2)updatePlayer(P2,K2,P1);
  if(!P1.dead){const tc=P1.x-350;if(tc>cameraX)cameraX=tc;}
}

function draw(){
  if(MODE==='TITLE')drawTitle();
  else if(MODE==='WORLD_MAP')drawMap();
  else if(MODE==='WARP_ANIMATION')drawWarp();
  else if(MODE==='BATTLE')drawBattle();
  else if(MODE==='SPACE')drawSpace();
  else if(MODE==='DEBUG')drawDebug();
  else drawGame();
  if(showTeacherImg){
    ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
    if(teacherImgLoaded){const iw=TEACHER_IMG.naturalWidth,ih=TEACHER_IMG.naturalHeight;const sc2=Math.min(W/iw,H/ih);const dw=iw*sc2,dh=ih*sc2;ctx.imageSmoothingEnabled=false;ctx.drawImage(TEACHER_IMG,(W-dw)/2,(H-dh)/2,dw,dh);}
    ctx.fillStyle='rgba(0,0,0,0.6)';ctx.fillRect(0,H-30,W,30);
    ctx.fillStyle='#fff';ctx.font='bold 14px monospace';ctx.textAlign='center';
    ctx.fillText('rキーを5回押すとゲームに戻る  r:'+rKeyBuf.length+'/5',W/2,H-10);ctx.textAlign='left';
  }
  if(numCtrl&&(MODE==='PLAYING'||MODE==='TUTORIAL')){
    ctx.fillStyle='rgba(0,0,0,0.55)';ctx.fillRect(0,0,220,20);
    ctx.fillStyle='#ffff00';ctx.font='bold 11px monospace';ctx.fillText('🎮 8←  9→  4↑  3dash',4,14);
  }
}

function loop(){update();draw();requestAnimationFrame(loop);}
loop();

// ============================================================
// ★ 追加コード BEGIN ★
// ============================================================

// ========== ワールド10: 嵐 (ハードW9クリア後) ==========
STAGES.push(
  {id:34,w:10,name:"10-1",type:"storm",      cleared:false,cx:130,cy:240},
  {id:35,w:10,name:"10-2",type:"storm",      cleared:false,cx:250,cy:240},
  {id:36,w:10,name:"10-3",type:"storm",      cleared:false,cx:370,cy:240},
  {id:37,w:10,name:"10-4",type:"storm_castle",cleared:false,cx:490,cy:240}
);
// getThemeをラップしてstormを追加
const _getThemeOrig=getTheme;
window.getThemeFull=function(sd){
  if(!sd)return _getThemeOrig(sd);
  if(sd.type==='storm'||sd.type==='storm_castle')return'storm';
  return _getThemeOrig(sd);
};
// getGimmickをラップ
const _getGimmickOrig=getGimmick;
window.getGimmickFull=function(w,type){
  if(type==='storm')return'lightning';
  if(type==='storm_castle')return'lightning';
  return _getGimmickOrig(w,type);
};

// getCurStagesをラップ（W10対応）
const _getCurOrig=getCurStages;
window.getCurStagesFull=function(){
  if(world===10)return STAGES.filter(s=>s.w===10).map(s=>({...s,canvasX:s.cx,canvasY:s.cy}));
  return _getCurOrig();
};

// BGMにstormを追加
BGM.storm=[[164,.1],[138,.15],[110,.2],[130,.1],[164,.1],[138,.1],[110,.3],[98,.1],[110,.15],[130,.2],[164,.25],[138,.4]];
BGM.storm_castle=[[98,.2],[110,.15],[130,.2],[110,.1],[98,.15],[87,.1],[98,.4],[82,.1],[98,.2],[110,.15],[130,.25],[110,.4]];

// wc/wn/wncにW10追加（drawMapから参照）
const WC_EXT={10:'#001020'};
const WN_EXT={10:'WORLD 10 - STORM ⛈'};
const WNC_EXT={10:'#88aaff'};

// ========== 2P分割画面・復活・レースシステム ==========
// P2復活タイマー（-1=生存中, 0以上=復活カウントダウン）
let p2RespawnTimer=-1;
const P2_RESPAWN_FRAMES=120; // 2秒(60fps×2)
// レースシステム
let raceMode=false; // 2P時のみ有効
let raceP1Goal=false,raceP2Goal=false,raceP1Time=0,raceP2Time=0,raceTimer=0;
let raceResult='';
// 分割画面カメラ
let cameraX2=0; // P2専用カメラ

// 元のresetGameをラップ
const _resetGameOrig=resetGame;
function resetGameFull(){
  _resetGameOrig();
  p2RespawnTimer=-1;
  raceMode=(numP===2);
  raceP1Goal=false;raceP2Goal=false;raceP1Time=0;raceP2Time=0;raceTimer=0;
  raceResult='';
  cameraX2=0;
}

// P2復活処理（毎フレーム呼ぶ）
function updateP2Respawn(){
  if(numP!==2)return;
  if(P2.dead&&p2RespawnTimer<0){
    // 死亡した瞬間から復活カウント開始
    p2RespawnTimer=P2_RESPAWN_FRAMES;
  }
  if(p2RespawnTimer>0){
    p2RespawnTimer--;
    if(p2RespawnTimer===0){
      // スタート地点から復活
      p2RespawnTimer=-1;
      P2=mkPl(160,200,true);
      cameraX2=0;
    }
  }
}

// レースブロック描画（スタート地点に固定）
function drawRaceBlock(){
  if(!raceMode||numP!==2)return;
  // スタート地点（x=0〜200付近）にレースブロック
  const bx=-cameraX+10,by=GY-64;
  // 市松模様ブロック
  ctx.fillStyle='#fff';ctx.fillRect(bx,by,80,20);
  for(let i=0;i<4;i++)for(let j=0;j<2;j++){
    if((i+j)%2===0){ctx.fillStyle='#222';ctx.fillRect(bx+i*20,by+j*10,20,10);}
  }
  ctx.strokeStyle='#000';ctx.lineWidth=2;ctx.strokeRect(bx,by,80,20);
  ctx.fillStyle='#fff';ctx.font='bold 9px monospace';ctx.fillText('START',bx+8,by+14);
  // ゴール地点表示
  const gx=goal.x-cameraX,gy=goal.y;
  if(gx>-100&&gx<W+100){
    ctx.fillStyle='rgba(255,255,0,0.2)';ctx.fillRect(gx-10,gy,30,goal.h);
    ctx.fillStyle='#ff0';ctx.font='bold 11px monospace';ctx.fillText('GOAL',gx-10,gy-5);
  }
  // 現在の進行状況
  const p1pct=Math.min(100,Math.floor((P1.x/goal.x)*100));
  const p2pct=Math.min(100,Math.floor((P2.x/goal.x)*100));
  ctx.fillStyle='rgba(0,0,0,0.6)';ctx.fillRect(W-160,4,155,38);
  ctx.fillStyle='#fc9c00';ctx.font='bold 11px monospace';ctx.fillText('🏁P1:'+p1pct+'%',W-150,18);
  ctx.fillStyle='#1bb749';ctx.fillText('🏁P2:'+p2pct+'%',W-150,32);
  if(raceTimer>0){ctx.fillStyle='#fff';ctx.fillText('⏱'+Math.floor(raceTimer/60)+'s',W-60,25);}
}

// レース完了チェック（updateで毎フレーム呼ぶ）
function updateRace(){
  if(!raceMode||numP!==2)return;
  if(raceResult)return;
  raceTimer++;
  if(P1.clear&&!raceP1Goal){raceP1Goal=true;raceP1Time=raceTimer;}
  if(!P2.dead&&P2.clear&&!raceP2Goal){raceP2Goal=true;raceP2Time=raceTimer;}
  if(raceP1Goal&&raceP2Goal){
    raceResult=raceP1Time<=raceP2Time?'p1':'p2';
  }else if(raceP1Goal&&p2RespawnTimer<0&&P2.dead){
    raceResult='p1'; // P2が死亡状態でP1がゴール
  }
}

// レース結果表示
function drawRaceResult(){
  if(!raceMode||!raceResult)return;
  ctx.fillStyle='rgba(0,0,0,0.7)';ctx.fillRect(150,150,500,140);
  ctx.strokeStyle='#ffd700';ctx.lineWidth=4;ctx.strokeRect(150,150,500,140);
  ctx.fillStyle='#ffd700';ctx.font='bold 36px monospace';ctx.textAlign='center';
  ctx.fillText(raceResult==='p1'?'🏆 MARIO WINS!':'🏆 LUIGI WINS!',400,210);
  ctx.fillStyle='#fff';ctx.font='bold 14px monospace';
  if(raceP1Time>0)ctx.fillText('Mario: '+Math.floor(raceP1Time/60)+'.'+String(raceP1Time%60).padStart(2,'0')+'s',400,245);
  if(raceP2Time>0)ctx.fillText('Luigi: '+Math.floor(raceP2Time/60)+'.'+String(raceP2Time%60).padStart(2,'0')+'s',400,265);
  ctx.textAlign='left';
}

// P2復活カウントダウン表示
function drawP2Respawn(){
  if(numP!==2||p2RespawnTimer<=0)return;
  const sec=Math.ceil(p2RespawnTimer/60);
  ctx.fillStyle='rgba(0,80,0,0.7)';ctx.fillRect(W/2-100,H-55,200,40);
  ctx.fillStyle='#1bb749';ctx.font='bold 14px monospace';ctx.textAlign='center';
  ctx.fillText('LUIGI 復活まで '+sec+'秒...',W/2,H-30);ctx.textAlign='left';
}

// ========== 分割画面描画 ==========
// 2Pモード時：上半分P1視点、下半分P2視点
function drawSplitScreen(){
  const HH=H/2; // 各画面の高さ

  // --- P1画面（上半分）---
  ctx.save();
  ctx.beginPath();ctx.rect(0,0,W,HH);ctx.clip();
  ctx.scale(1,0.5);
  // カメラをP1に合わせる
  const cam1=cameraX;
  drawBGSplit(cam1,0,W,H); // 背景をフルで描いてからクリップ
  ctx.save();ctx.translate(-cam1,0);
  platforms.forEach(p=>drawPlatform(p));
  drawGoalFlag();
  coins.forEach(c=>{if(!c.col){ctx.fillStyle='#fc9c00';ctx.fillRect(c.x,c.y,c.w,c.h);}});
  items.forEach(drawItem);enemies.forEach(drawEnemy);
  fireballs.forEach(fb=>{ctx.fillStyle='#ff8800';ctx.beginPath();ctx.arc(fb.x+5,fb.y+5,5,0,Math.PI*2);ctx.fill();});
  drawPl(P1);
  if(!P2.dead)drawPl(P2);
  ctx.restore();
  if(theme==='lava'||gimmick.lava!==undefined){ctx.save();ctx.translate(-cam1,0);drawLava();ctx.restore();}
  ctx.restore();

  // --- P2画面（下半分）---
  ctx.save();
  ctx.beginPath();ctx.rect(0,HH,W,HH);ctx.clip();
  ctx.translate(0,HH);ctx.scale(1,0.5);
  const cam2=cameraX2;
  drawBGSplit(cam2,0,W,H);
  ctx.save();ctx.translate(-cam2,0);
  platforms.forEach(p=>drawPlatform(p));
  drawGoalFlag();
  coins.forEach(c=>{if(!c.col){ctx.fillStyle='#fc9c00';ctx.fillRect(c.x,c.y,c.w,c.h);}});
  items.forEach(drawItem);enemies.forEach(drawEnemy);
  fireballs.forEach(fb=>{ctx.fillStyle='#ff8800';ctx.beginPath();ctx.arc(fb.x+5,fb.y+5,5,0,Math.PI*2);ctx.fill();});
  drawPl(P1);
  if(!P2.dead)drawPl(P2);
  ctx.restore();
  if(theme==='lava'||gimmick.lava!==undefined){ctx.save();ctx.translate(-cam2,0);drawLava();ctx.restore();}
  ctx.restore();

  // --- 区切り線 ---
  ctx.fillStyle='#000';ctx.fillRect(0,HH-2,W,4);
  ctx.fillStyle='#fff';ctx.fillRect(0,HH-1,W,2);

  // --- 各画面HUD ---
  ctx.fillStyle='rgba(0,0,0,0.5)';ctx.fillRect(0,0,W,22);
  ctx.fillStyle='#fc9c00';ctx.font='bold 12px monospace';ctx.fillText('P1:MARIO  '+String(P1.score).padStart(6,'0'),4,15);
  if(P1.dead)ctx.fillStyle='#ff4444';else ctx.fillStyle='#80d010';
  ctx.fillText(P1.dead?'DEAD':'LIVE',220,15);

  ctx.fillStyle='rgba(0,0,0,0.5)';ctx.fillRect(0,HH+2,W,22);
  ctx.fillStyle='#1bb749';ctx.font='bold 12px monospace';ctx.fillText('P2:LUIGI  '+String(P2.score).padStart(6,'0'),4,HH+17);
  if(P2.dead)ctx.fillStyle='#ff4444';else ctx.fillStyle='#80d010';
  ctx.fillText(P2.dead?'DEAD':'LIVE',220,HH+17);

  // レースブロック・結果・復活表示
  drawRaceBlock();
  drawRaceResult();
  drawP2Respawn();

  // P1 CLEAR/DEAD表示
  if(P1.clear&&!P1.dead){
    ctx.fillStyle='rgba(0,0,0,0.6)';ctx.fillRect(200,HH/2-30,400,50);
    ctx.fillStyle='#ffd700';ctx.font='bold 22px monospace';ctx.textAlign='center';
    ctx.fillText('STAGE CLEAR! SPACE→次へ',W/2,HH/2+6);ctx.textAlign='left';
  }
  if(P1.dead){
    ctx.fillStyle='rgba(0,0,0,0.6)';ctx.fillRect(200,HH/2-30,400,50);
    ctx.fillStyle='#ff4444';ctx.font='bold 22px monospace';ctx.textAlign='center';
    ctx.fillText('GAME OVER  SPACE→RETRY',W/2,HH/2+6);ctx.textAlign='left';
  }
}

function drawBGSplit(cam,ox,ow,oh){
  const savedCameraX=cameraX;
  cameraX=cam;
  drawBG();
  cameraX=savedCameraX;
}

function drawGoalFlag(){
  ctx.fillStyle='#fff';ctx.fillRect(goal.x+4,goal.y,4,goal.h);
  ctx.fillStyle='#00a800';ctx.beginPath();ctx.arc(goal.x+6,goal.y,8,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='#000';ctx.fillRect(goal.x-24,goal.y+20,24,18);
}

// ========== ワールド10 嵐の背景・足場描画 ==========
const _drawBGOrig=drawBG;
window.drawBG = function(){
  if(theme==='storm'){
    const g=ctx.createLinearGradient(0,0,0,H);
    g.addColorStop(0,'#0a0a1a');g.addColorStop(0.5,'#101030');g.addColorStop(1,'#202040');
    ctx.fillStyle=g;ctx.fillRect(0,0,W,H);
    // 雨
    ctx.strokeStyle='rgba(150,180,255,0.4)';ctx.lineWidth=1;
    for(let i=0;i<30;i++){
      const rx=((i*137+titleFrame*8+cameraX*0.1)%(W+80))-40;
      const ry=(i*53+titleFrame*6)%H;
      ctx.beginPath();ctx.moveTo(rx,ry);ctx.lineTo(rx-4,ry+16);ctx.stroke();
    }
    // 稲妻（周期的）
    if(Math.floor(titleFrame/90)%5===0||(gimmick.ltTimer!==undefined&&gimmick.ltTimer<8)){
      ctx.strokeStyle=`rgba(200,220,255,${0.8-((gimmick.ltTimer||0)*0.1)})`;ctx.lineWidth=2;
      const lx=180+((Math.floor(titleFrame/90)*197)%(W-360));
      ctx.beginPath();ctx.moveTo(lx,0);ctx.lineTo(lx+20,80);ctx.lineTo(lx+5,80);ctx.lineTo(lx+30,180);ctx.stroke();
    }
    // 雲
    ctx.fillStyle='rgba(40,40,80,0.8)';
    for(let i=0;i<4;i++){
      const ccx=((i*250-cameraX*0.15+W*2)%(W+300));
      ctx.beginPath();ctx.arc(ccx,30+i*8,40,0,Math.PI*2);ctx.arc(ccx+40,25+i*8,50,0,Math.PI*2);ctx.arc(ccx+80,30+i*8,40,0,Math.PI*2);ctx.fill();
    }
    return;
  }
  _drawBGOrig();
}

// ========== ギミック: 稲妻ダメージ ==========
const _updateGimmickOrig=updateGimmick;
window.updateGimmick = function(){
  _updateGimmickOrig();
  // 嵐ワールド: 稲妻ギミック
  if(gimmick.lightning){
    gimmick.ltTimer=(gimmick.ltTimer||0)+1;
    gimmick.ltNext=(gimmick.ltNext||200);
    if(gimmick.ltTimer>=gimmick.ltNext){
      // 稲妻発生: プレイヤー付近にランダムで落下
      gimmick.ltX=P1.x+(-100+Math.random()*200);
      gimmick.ltFlash=12;
      gimmick.ltTimer=0;gimmick.ltNext=180+Math.floor(Math.random()*120);
      // 命中判定
      if(Math.abs(P1.x-gimmick.ltX)<48&&!P1.dead&&!P1.star){seFx('hit');if(P1.fire)P1.fire=false;else if(P1.big)P1.big=false;else{P1.dead=true;P1.vy=-11;}}
      if(numP===2&&Math.abs(P2.x-gimmick.ltX)<48&&!P2.dead&&!P2.star){seFx('hit');if(P2.fire)P2.fire=false;else if(P2.big)P2.big=false;else{P2.dead=true;P2.vy=-11;}}
    }
    if(gimmick.ltFlash>0)gimmick.ltFlash--;
  }
}

// ========== W10をloadStageで処理 ==========
const _loadStageOrig=loadStage;
window.loadStage = function(sd){
  if(sd.type==='storm'||sd.type==='storm_castle'){
    platforms=[];coins=[];enemies=[];items=[];fireballs=[];gimmick={};
    theme='storm';underwater=false;iceSlide=false;
    const castle=(sd.type==='storm_castle');
    const totalLen=castle?8800:4300;
    platforms.push({x:0,y:GY,width:400,height:60,type:'ground'});
    let cx2=400;
    while(cx2<totalLen-400){
      const gap=100+Math.floor(Math.random()*120);cx2+=gap;
      const gw=200+Math.floor(Math.random()*250);
      platforms.push({x:cx2,y:GY,width:gw,height:60,type:'ground'});
      const sc2=cx2+gw/2;
      if(Math.random()<0.4){const ph=40+Math.floor(Math.random()*40);platforms.push({x:sc2-32,y:GY-ph,width:64,height:ph,type:'pipe'});}
      else if(Math.random()<0.5){const by=Math.random()<0.5?260:210;platforms.push({x:sc2-48,y:by,width:96,height:32,type:Math.random()<0.4?'question':'block',hit:false});coins.push({x:sc2-8,y:by-40,w:16,h:24,col:false});coins.push({x:sc2+16,y:by-40,w:16,h:24,col:false});}
      cx2+=gw;
    }
    platforms.push({x:cx2,y:GY,width:600,height:60,type:'ground'});
    [{y:32,h:32},{y:64,h:64},{y:96,h:96},{y:128,h:128}].forEach((o,i2)=>platforms.push({x:cx2+100+i2*32,y:GY-o.y,width:32,height:o.h,type:'block'}));
    goal={x:cx2+350,y:GY-260,w:10,h:260};
    enemies=spawnEnemies(platforms,'storm');
    items=spawnItems(platforms);
    gimmick={lightning:true,ltTimer:0,ltNext:200,ltX:0,ltFlash:0};
    bgmPlay(castle?'storm_castle':'storm');
    return;
  }
  _loadStageOrig(sd);
}

// ========== W10をワールドマップ・ナビに追加 ==========
const _getCurStagesOrig=getCurStages;
window.getCurStages = function(){
  if(world===10)return STAGES.filter(s=>s.w===10).map(s=>({...s,canvasX:s.cx,canvasY:s.cy}));
  return _getCurStagesOrig();
}
const _getThemeO=getTheme;
window.getTheme = function(sd){
  if(!sd)return _getThemeO(sd);
  if(sd.type==='storm'||sd.type==='storm_castle')return'storm';
  return _getThemeO(sd);
}
const _getGimmickO=getGimmick;
window.getGimmick = function(w,type){
  if(type==='storm'||type==='storm_castle')return'lightning';
  return _getGimmickO(w,type);
}

// ========== drawMapをラップ（W10対応） ==========
const _drawMapOrig=drawMap;
window.drawMap = function(){
  if(world===10){
    ctx.fillStyle='#001020';ctx.fillRect(0,0,W,H);
    ctx.fillStyle='#88aaff';ctx.font='bold 24px monospace';ctx.fillText('WORLD 10 - STORM ⛈',150,52);
    ctx.fillStyle='#aaa';ctx.font='13px monospace';ctx.fillText('Arrow:Select  SPACE:Enter',150,78);
    if(!hardMode){ctx.fillStyle='#ff4444';ctx.font='bold 16px monospace';ctx.textAlign='center';ctx.fillText('⚠ HARD MODE W9クリア後に解放',400,250);ctx.textAlign='left';return;}
    const ws=getCurStages();
    ctx.strokeStyle='#334';ctx.lineWidth=4;ctx.setLineDash([6,6]);
    if(ws.length>0){ctx.beginPath();ctx.moveTo(ws[0].canvasX+30,ws[0].canvasY+25);ws.slice(1).forEach(s=>ctx.lineTo(s.canvasX+30,s.canvasY+25));ctx.stroke();}ctx.setLineDash([]);
    ws.forEach((s,i)=>{
      const acc=(i===0||ws[i-1].cleared);
      ctx.fillStyle=s.cleared?'#5c94fc':acc?'#334488':'#333';
      ctx.fillRect(s.canvasX,s.canvasY,64,50);
      ctx.strokeStyle=i===mapIdx?'#fff':'#667';ctx.lineWidth=i===mapIdx?3:1;ctx.strokeRect(s.canvasX,s.canvasY,64,50);
      ctx.fillStyle=acc?'#fff':'#666';ctx.font='bold 11px monospace';ctx.fillText(s.name,s.canvasX+4,s.canvasY+30);
      if(s.type==='storm_castle'){ctx.fillStyle='#aaf';ctx.font='9px monospace';ctx.fillText('CASTLE',s.canvasX+4,s.canvasY+44);}
      if(s.cleared){ctx.fillStyle='#80d010';ctx.font='bold 11px monospace';ctx.fillText('★OK',s.canvasX+14,s.canvasY-7);}
    });
    const as=ws[mapIdx];if(!as)return;
    ctx.strokeStyle='#fff';ctx.lineWidth=3;ctx.strokeRect(as.canvasX-4,as.canvasY-4,72,58);
    drawSpr(sIdle,as.canvasX+8,as.canvasY-44,3,'right');
    if(numP===2)drawSpr(lIdle,as.canvasX+26,as.canvasY-40,2,'right');
    ctx.fillStyle='#fff';ctx.font='bold 15px monospace';
    const ce=(mapIdx===0||ws[mapIdx-1].cleared);
    ctx.fillText(ce?`👉 [${as.name}]: SPACE`:'🔒 LOCKED',150,365);
    ctx.fillStyle='#ffd700';ctx.font='12px monospace';ctx.fillText('GIMMICK: ⚡稲妻ダメージ',150,385);
    return;
  }
  _drawMapOrig();
}

// ========== drawGameをラップ（2P分割画面対応） ==========
const _drawGameOrig=drawGame;
window.drawGame = function(){
  if(numP===2&&(MODE==='PLAYING'||MODE==='TUTORIAL')){
    drawSplitScreen();
    return;
  }
  _drawGameOrig();
  // 嵐の稲妻フラッシュエフェクト
  if(theme==='storm'&&gimmick.ltFlash>0){
    ctx.fillStyle=`rgba(220,230,255,${gimmick.ltFlash*0.05})`;ctx.fillRect(0,0,W,H);
    const lx=(gimmick.ltX||400)-cameraX;
    ctx.strokeStyle=`rgba(220,230,255,${gimmick.ltFlash*0.12})`;ctx.lineWidth=3;
    ctx.beginPath();ctx.moveTo(lx,0);ctx.lineTo(lx+20,80);ctx.lineTo(lx+5,80);ctx.lineTo(lx+30,180);ctx.stroke();
  }
}

// ========== updateをラップ（2P復活・レース・W10ワープ対応） ==========
const _updateOrig=update;
window.update = function(){
  _updateOrig();
  // 2P復活システム
  if(MODE==='PLAYING'||MODE==='TUTORIAL'){
    updateP2Respawn();
    updateRace();
    // P2カメラをP2に追従
    if(numP===2&&!P2.dead){const tc2=P2.x-350;if(tc2>cameraX2)cameraX2=tc2;}
  }
}

// ========== ワープアニメでW10対応 ==========
// warpSt===4でworld++後にW10チェック追加
const _loopOrig=loop;
// ワープ遷移でW10への進行チェック（updateの中でwarpSt===4の後world++されているため）
// → drawWarpの nextW計算を上書きするためwrapNextWorld関数を用意
window.getNextWorld=function(w){
  if(w===9&&hardMode)return 10;
  if(w===10)return 1;
  return null; // 既存ロジックに任せる
};

// ========== ステージクリア処理にW9→W10を追加 ==========
// keydownのP1.clear処理で、W9の後W10へ進むよう既存のif(world===8&&hardMode)の後に追加
// → 既存コードを変えず、updateのwrapアニメ内でworld++後に補正
// updateのwarpSt===4部分は既存のupdateOrig内なので、以下で補間：
const _origLoopFn=loop;
window.loop = function(){
  // warpアニメのworld遷移補正（W9→W10, W10→W1）
  if(MODE==='WORLD_MAP'&&world===10&&!hardMode)world=1;
  _origLoopFn();
}
// W9クリア後W10への誘導: keydown clear処理を補完
// P1.clearのspaceキー処理でwarpアニメ開始条件にworld===9を追加
const _origOnKeydown=window.onkeydown; // 参照なし
// keydownのW9処理を直接addEventListenerで追補
window.addEventListener('keydown',function(e){
  if(e.key===' '&&(MODE==='PLAYING'||MODE==='TUTORIAL')&&P1.clear&&world===9&&hardMode){
    const ws=getCurStages();
    if(ws.every(s=>s.cleared)){
      MODE='WARP_ANIMATION';warpTimer=0;warpSt=0;warpYOff=0;bgmStop();
    }
  }
  // W10クリア後の処理
  if(e.key===' '&&(MODE==='PLAYING'||MODE==='TUTORIAL')&&P1.clear&&world===10){
    const cs=STAGES.find(s=>s.id===stageId);if(cs)cs.cleared=true;
    const ws=getCurStages();
    if(ws.every(s=>s.cleared)){MODE='WARP_ANIMATION';warpTimer=0;warpSt=0;warpYOff=0;bgmStop();}
    else{MODE='WORLD_MAP';const ni=ws.findIndex(s=>s.id===stageId)+1;if(ni<ws.length)mapIdx=ni;bgmStop();}
  }
  // 2P分割画面中のresetGame呼び出しをresetGameFullに差し替え
  if(e.key===' '&&P1.clear&&numP===2&&raceResult){
    const cs2=STAGES.find(s=>s.id===stageId);if(cs2)cs2.cleared=true;
    MODE='WORLD_MAP';bgmStop();
  }
});

// warpSt===4でworld++後にW9→W10補正
// updateOrig内のwarpSt===4処理後にworld===10への補正が必要
// → 既存のloop()にフック済み（上のloop override参照）
// ただしwarpTimerリセット後のworld値修正が必要なため追加フック
const __origUpdate=update;
window._warpW10Fixed=false;

// ========== resetGame差し替え ==========
// resetGameをresetGameFullで上書き（既存resetGameは_resetGameOrigに保存済み）
window.resetGame=resetGameFull;

// DB_ITEMSにW10追加
DB_ITEMS.push(
  {lbl:'W10解放(嵐)',g:()=>'EXEC',s:()=>{STAGES.filter(s=>s.w===10).forEach(s=>s.cleared=false);world=10;mapIdx=0;MODE='WORLD_MAP';bgmStop();}}
);

// world===10の時にgetCurStagesが正しく動くのでOK（二重ラップ不要）

// ============================================================
// ★ 追加コード END ★
// ============================================================


// ============================================================
// ★ 攻略本(デバッグモード 2ページ目) 追加コード START ★
// 既存コードは一切変更せず、新規の変数・関数・イベントリスナー・
// 関数上書き(既存パターンと同様の手法)のみで追加しています。
// ============================================================

// ---- 攻略本の状態管理 ----
let dbPage=0;        // 0=デバッグメニュー 1=攻略本
let dbGuideIdx=0;    // 攻略本で表示中のワールド番号(GUIDE配列のindex)
let __dbGuideKeyBuf=''; // 0913入力を独自に監視してページをリセットするための独立バッファ

// ---- 攻略情報データ ----
const GUIDE=[
  {w:0,title:'チュートリアル',gimmick:'ギミックなし',
   desc:'操作に慣れるための練習ステージ。ブロックからキノコとスターが出てくるので取ってみよう。ダッシュジャンプ(Shift/X + ジャンプ)の感覚を掴んでおくと後のワールドで役立つ。',
   stages:[{name:'TUTORIAL',tip:'左右移動とジャンプの基本練習。危険は少ないので落ち着いて操作を確認しよう。EXTRA解放後は敵が追加され少し難化する。'}]},
  {w:1,title:'ワールド1: 草原(グラスランド)',gimmick:'⚠崩れる床(crumble)',
   desc:'一部の地面ブロックは乗ってからしばらくすると崩れ落ちる。同じ足場に長居せずテンポよく進もう。基本操作を試す入門ワールド。',
   stages:[
     {name:'1-1',tip:'通常ステージ。崩れる床の位置を意識しつつクリボー・ノコノコを踏んで進もう。'},
     {name:'1-2',tip:'1-1よりやや距離が長い。パイプの高さやブロック配置を確認しながら進もう。'},
     {name:'1-3',tip:'同系統だが敵の数がやや増加。ダッシュで一気に駆け抜けるのも有効。'},
     {name:'1-4 (城)',tip:'城ステージは総距離8800と長く、パイプやブロックの密度も高い。ゴールの旗まで気を抜かずに進もう。'},
   ]},
  {w:2,title:'ワールド2: 砂漠と水中',gimmick:'🌊水流(current) / 2-2のみ水中',
   desc:'常に左向きの力(current)がかかり続けるため、立ち止まると押し戻される。ダッシュ気味に進むこと。2-2のみ水中ステージになり、浮力のある泳ぎ操作に変化。ゲッソーの落下とプクプクの突進に注意。',
   stages:[
     {name:'2-1',tip:'currentによる押し戻しに注意。ジャンプ中も流されるので着地位置を先読みしよう。'},
     {name:'2-2 (水中)',tip:'水中ステージ。ジャンプでゆっくり浮上しゆっくり沈む。プクプクの突進とゲッソーの落下タイミングを見極めよう。'},
     {name:'2-3',tip:'2-1と同系統。current+穴(足場の切れ目)の組み合わせに注意。'},
     {name:'2-4 (城)',tip:'水流はなくなるが、城特有の長い距離とブロック密度に注意。'},
   ]},
  {w:3,title:'ワールド3: 氷',gimmick:'❄氷スライド(ice)',
   desc:'地面が滑りやすくなり、キーを離しても慣性で滑り続ける。止まりたい場所より手前で早めにキーを離すのがコツ。着地後も滑るので穴の近くでは特に注意。',
   stages:[
     {name:'3-1',tip:'滑る床に慣れよう。穴の手前では早めに減速を始める。'},
     {name:'3-2',tip:'同系統で敵が増加。滑りながらの敵回避を練習しよう。'},
     {name:'3-3',tip:'同系統でブロック配置がやや複雑。着地点を落ち着いて見極めよう。'},
     {name:'3-4 (城)',tip:'氷スライドはなくなる。長距離の通常城ステージ。'},
   ]},
  {w:4,title:'ワールド4: 森',gimmick:'🌿つる足場(vine)',
   desc:'上下に揺れる「つる」ブロックが足場として出現する。動きを見てからタイミングよく飛び乗ろう。掴み直しはできないので焦らず待つのがコツ。',
   stages:[
     {name:'4-1',tip:'つるの上下動をよく見てから飛び乗ろう。焦って飛ぶと落下しやすい。'},
     {name:'4-2',tip:'同系統でつるの数が増加。複数のつるを連続して渡る場面がある。'},
     {name:'4-3',tip:'同系統。つると敵の組み合わせに注意し、着地後すぐ敵に踏まれないようにしよう。'},
     {name:'4-4 (城)',tip:'つるギミックはなくなる。長距離の通常城ステージ。'},
   ]},
  {w:5,title:'ワールド5: 空',gimmick:'💨強風(wind, strong)',
   desc:'足場は雲とブロックのみで細く、画面上方(Y<-100)まで飛ばされる・落下すると即ミスになる。強い横風が常に吹いているため、風上側へ踏ん張りながら進もう。5-4のみ通常の城ステージ(空ではない)。',
   stages:[
     {name:'5-1',tip:'雲の足場は細く不安定。風で流されて隙間に落ちないよう慎重に。'},
     {name:'5-2',tip:'同系統で足場の間隔が広がる。ジャンプ前に風向きを意識しよう。'},
     {name:'5-3',tip:'同系統で敵の配置が増加。着地と同時に敵と接触しないよう注意。'},
     {name:'5-4 (城)',tip:'地上の通常城ステージに戻る。空の緊張感からは解放されるが距離は長い。'},
   ]},
  {w:6,title:'ワールド6: 溶岩',gimmick:'🌋溶岩上昇(lavarise, 緩やか)',
   desc:'画面下から溶岩がゆっくり上昇してくる。低い場所に長く留まらず、常に高い足場を目指して進もう。溶岩に触れると即ミス。',
   stages:[
     {name:'6-1',tip:'溶岩の上昇速度を意識し、先読みでジャンプして高い場所を確保しよう。'},
     {name:'6-2',tip:'同系統で足場がやや不安定。慎重に足場を選ぼう。'},
     {name:'6-3',tip:'同系統で敵が増加。溶岩と敵の両方から逃げながら進む必要がある。'},
     {name:'6-4 (城)',tip:'溶岩ギミックはなくなる。W1〜6最後のステージ、慎重にゴールを目指そう。'},
   ]},
  {w:'EX',title:'EXTRA (ハードモード開放条件)',gimmick:'なし',
   desc:'ワールド1〜6を全てクリアすると、TUTORIAL枠が「EXTRA」に変化して挑戦できるようになる。EXTRAをクリアするとハードモードが開始し、全ステージの クリア状況がリセットされてワールド7(夜)以降が解放される。',
   stages:[{name:'EXTRA',tip:'クリボーとノコノコが追加された特別なチュートリアル面。クリアするとハードモード開始、残機は10にリセットされる。'}]},
  {w:7,title:'ワールド7: 夜(ハードモード)',gimmick:'👾クリボーラッシュ(goombarush)',
   desc:'画面内に生存している敵が8体以上になると警告表示が出る、敵の絶えないステージ。囲まれないよう位置取りに注意し、無理せず踏みつけて数を減らそう。7-4は夜の城ステージで暗視エフェクト(darkfog)がかかり視界が制限される。',
   stages:[
     {name:'7-1',tip:'クリボーラッシュ状態(敵8体以上)になったら無理に戦わず、隙間を縫って前進しよう。'},
     {name:'7-2',tip:'同系統。踏みつけ連打で数を減らしつつ安全な足場を確保しよう。'},
     {name:'7-3',tip:'同系統。敵の密集地帯ではダッシュジャンプで頭上を飛び越えるのも有効。'},
     {name:'7-4 (夜の城)',tip:'darkfogにより視界が暗くなる。先の地形が見えにくいので慎重に、小刻みに進もう。'},
   ]},
  {w:8,title:'ワールド8: 月(ハードモード)',gimmick:'🌙低重力ジャンプ3倍(lowgrav)',
   desc:'重力とジャンプの慣性が弱くなり、大きく浮遊するようになる。滞空時間が長くなる分、着地点のコントロールが難しくなるので慎重に。8-4は月震(moonquake)により画面が周期的に揺れる。',
   stages:[
     {name:'8-1',tip:'低重力に慣れよう。ジャンプの押しっぱなし時間で高度をコントロールする感覚を掴む。'},
     {name:'8-2',tip:'同系統。浮遊時間が長いため、敵の頭上をゆっくり飛び越えられる。'},
     {name:'8-3',tip:'同系統。着地が乱れやすいので、穴の手前では早めに減速の意識を。'},
     {name:'8-4 (月の城)',tip:'moonquakeで画面が揺れる中でのジャンプになる。揺れに惑わされず足場をよく見よう。'},
   ]},
  {w:9,title:'ワールド9: 宇宙シューティング(ハードモード)',gimmick:'🚀縦スクロールシューティング',
   desc:'これまでのアクションとは異なり、自機を操作して敵を撃墜するミニゲーム。各ステージのキル数目標は「12+ステージ番号×4」体（9-1:16体 〜 9-8:44体）。ステージ3以降は敵も弾を撃ってくるようになり、ステージが進むほど敵の出現速度・移動速度も上昇する。HPは3。',
   stages:[
     {name:'9-1〜9-2',tip:'目標16〜20体。敵弾はまだ来ないので撃墜に専念しよう。'},
     {name:'9-3〜9-4',tip:'目標24〜28体。この辺りから敵が反撃してくる。被弾を避けつつ倒そう。'},
     {name:'9-5〜9-6',tip:'目標32〜36体。敵の出現ペースが上がるため、左右に細かく動いて弾を回避しよう。'},
     {name:'9-7〜9-8',tip:'目標40〜44体。最終盤。HP3を大切にしながら確実に敵を減らしていこう。9-8クリアでW10へ進出。'},
   ]},
  {w:10,title:'ワールド10: 嵐',gimmick:'⚡稲妻(lightning)',
   desc:'自機(プレイヤー)付近にランダムなタイミングで稲妻が落ちてくる。画面が光る(フラッシュ)瞬間が予兆で、命中するとFIRE→BIG→小さい状態の順にダウングレードし、小さい状態で被弾すると即ミスになる。同じ位置に留まりすぎず、こまめに移動しよう。10-4は嵐の城(storm_castle)。',
   stages:[
     {name:'10-1',tip:'画面のフラッシュに注意し、頻繁に位置を変えながら進もう。'},
     {name:'10-2',tip:'同系統。稲妻を警戒しつつ足場の見極めも忘れずに。'},
     {name:'10-3',tip:'同系統。終盤に向けて残機とパワーアップ状態を大事にしよう。'},
     {name:'10-4 (嵐の城)',tip:'稲妻に加えて城特有の長い距離。最終ステージ、落ち着いて攻略しよう。'},
   ]},
];

// ---- 攻略本用: 日本語対応テキスト折り返し ----
function __gdWrap(context,text,maxW){
  const lines=[];let cur='';
  for(const ch of text){
    const test=cur+ch;
    if(context.measureText(test).width>maxW && cur.length>0){lines.push(cur);cur=ch;}
    else cur=test;
  }
  if(cur)lines.push(cur);
  return lines;
}

// ---- 攻略本 描画 ----
function drawStrategyGuide(){
  ctx.fillStyle='#001400';ctx.fillRect(0,0,W,H);
  ctx.strokeStyle='rgba(0,80,0,0.3)';ctx.lineWidth=1;
  for(let i=0;i<W;i+=40)ctx.strokeRect(i,0,40,H);

  const g=GUIDE[dbGuideIdx];

  ctx.fillStyle='#00ff00';ctx.font='bold 18px monospace';ctx.textAlign='center';
  ctx.fillText('📖 攻略本 ('+(dbGuideIdx+1)+'/'+GUIDE.length+')',400,24);
  ctx.textAlign='left';

  ctx.fillStyle='#0f0';ctx.font='11px monospace';
  ctx.fillText('[Z]前のワールド  [X]次のワールド  [ESC]または[BackSpace]でデバッグメニューに戻る',20,40);

  ctx.fillStyle='#ffff00';ctx.font='bold 15px monospace';
  ctx.fillText(g.title,20,62);

  ctx.fillStyle='#88ff88';ctx.font='12px monospace';
  ctx.fillText('ギミック: '+g.gimmick,20,80);

  ctx.fillStyle='#cfc';ctx.font='11px monospace';
  let y=98;
  __gdWrap(ctx,g.desc,755).forEach(line=>{ctx.fillText(line,20,y);y+=15;});

  y+=6;
  ctx.strokeStyle='#0a0';ctx.beginPath();ctx.moveTo(20,y);ctx.lineTo(780,y);ctx.stroke();
  y+=16;

  g.stages.forEach(st=>{
    if(y>418)return; // 画面はみ出し防止
    ctx.fillStyle='#ffff88';ctx.font='bold 12px monospace';
    ctx.fillText('■ '+st.name,20,y);y+=14;
    ctx.fillStyle='#cfc';ctx.font='11px monospace';
    __gdWrap(ctx,st.tip,730).forEach(line=>{if(y<=418){ctx.fillText('　'+line,20,y);y+=14;}});
    y+=6;
  });

  ctx.fillStyle='#004400';ctx.font='11px monospace';ctx.textAlign='center';
  ctx.fillText('DEBUGメニューの「📖 攻略本を開く」からいつでもここに来られます',400,433);
  ctx.textAlign='left';
}

// ---- デバッグメニューに攻略本を開く項目を追加 ----
DB_ITEMS.push(
  {lbl:'📖 攻略本を開く(2ページ目)',g:()=>'[SPACE]',s:()=>{dbGuideIdx=0;dbPage=1;}}
);

// ---- drawDebugを上書きし、dbPage===1のときは攻略本を描画 ----
// (既存コードと同じ「関数の上書き」パターンを使用。既存のdrawDebug自体は変更しない)
const __drawDebugOrig=drawDebug;
window.drawDebug=function(){
  if(dbPage===1){drawStrategyGuide();return;}
  __drawDebugOrig();
};

// ---- 攻略本ページ用のキー操作を独自リスナーで追加 ----
window.addEventListener('keydown',e=>{
  const k=e.key;
  // 0913入力を独自バッファでも監視し、デバッグモードに新規で入る際は
  // 常にメニューページ(dbPage=0)から始まるようにする
  if(k>='0'&&k<='9'&&k.length===1){
    __dbGuideKeyBuf+=k;
    if(__dbGuideKeyBuf.length>4)__dbGuideKeyBuf=__dbGuideKeyBuf.slice(-4);
    if(__dbGuideKeyBuf==='0913'){__dbGuideKeyBuf='';dbPage=0;}
  }
  if(MODE!=='DEBUG')return;
  if(dbPage===1){
    if(k==='z'||k==='Z')dbGuideIdx=(dbGuideIdx-1+GUIDE.length)%GUIDE.length;
    if(k==='x'||k==='X')dbGuideIdx=(dbGuideIdx+1)%GUIDE.length;
    if(k==='Escape'||k==='Backspace'){dbPage=0;e.preventDefault();}
  }
});

// ============================================================
// ★ 攻略本(デバッグモード 2ページ目) 追加コード END ★
// ============================================================


// ============================================================
// ★ 大型追加パック: バトル10ステージ / 宇宙2P / エディットモード /
//    倉庫・スペシャル1 / メガマリオ / 隠し土管ワープ  START ★
// 既存コードは一切変更せず、新規変数・新規関数・関数の上書き
// （既存と同じ手法）・独自イベントリスナーの追加のみで実装しています。
// ============================================================

// ---------------------------------------------------------------
// 【1】バトルモード: ワールドの雰囲気に合わせた10ステージ
// ---------------------------------------------------------------
let battleSelIdx = 0;
let battleActiveTheme = 'grassland';
const BATTLE_STAGES = [
  {short:'草原', name:'グラスランド闘技場', theme:'grassland', plats:[
    {x:150,y:270,w:120,h:20},{x:340,y:230,w:120,h:20},{x:530,y:270,w:120,h:20},{x:240,y:175,w:100,h:20},{x:460,y:175,w:100,h:20}]},
  {short:'砂漠', name:'デザート闘技場', theme:'desert', plats:[
    {x:100,y:260,w:140,h:20},{x:330,y:210,w:140,h:20},{x:560,y:260,w:140,h:20},{x:280,y:150,w:90,h:20},{x:430,y:150,w:90,h:20}]},
  {short:'氷', name:'アイス闘技場', theme:'ice', plats:[
    {x:120,y:250,w:100,h:20},{x:260,y:290,w:100,h:20},{x:400,y:220,w:100,h:20},{x:540,y:290,w:100,h:20},{x:640,y:250,w:100,h:20}]},
  {short:'森', name:'フォレスト闘技場', theme:'forest', plats:[
    {x:80,y:280,w:110,h:20},{x:230,y:230,w:110,h:20},{x:380,y:180,w:110,h:20},{x:530,y:230,w:110,h:20},{x:640,y:280,w:110,h:20}]},
  {short:'空', name:'スカイ闘技場', theme:'sky', plats:[
    {x:100,y:300,w:90,h:18},{x:240,y:250,w:90,h:18},{x:380,y:200,w:90,h:18},{x:520,y:250,w:90,h:18},{x:640,y:300,w:90,h:18},{x:340,y:140,w:120,h:18}]},
  {short:'溶岩', name:'ラバ闘技場', theme:'lava', plats:[
    {x:60,y:260,w:130,h:20},{x:280,y:210,w:100,h:20},{x:420,y:260,w:100,h:20},{x:580,y:210,w:130,h:20},{x:330,y:150,w:120,h:20}]},
  {short:'夜', name:'ナイト闘技場', theme:'night', plats:[
    {x:110,y:270,w:110,h:20},{x:280,y:220,w:110,h:20},{x:450,y:270,w:110,h:20},{x:600,y:220,w:110,h:20},{x:350,y:150,w:100,h:20}]},
  {short:'月', name:'ムーン闘技場', theme:'moon', plats:[
    {x:100,y:290,w:100,h:20},{x:230,y:240,w:100,h:20},{x:360,y:190,w:100,h:20},{x:490,y:240,w:100,h:20},{x:620,y:290,w:100,h:20}]},
  {short:'宇宙', name:'スペース闘技場', theme:'spacebg', plats:[
    {x:150,y:260,w:90,h:18},{x:300,y:200,w:90,h:18},{x:450,y:260,w:90,h:18},{x:600,y:200,w:90,h:18},{x:370,y:320,w:60,h:18}]},
  {short:'嵐', name:'ストーム闘技場', theme:'storm', plats:[
    {x:80,y:270,w:120,h:20},{x:260,y:220,w:120,h:20},{x:440,y:270,w:120,h:20},{x:600,y:220,w:120,h:20},{x:340,y:150,w:130,h:20}]},
];

function applyBattleStage(idx){
  battleSelIdx = Math.max(0, Math.min(BATTLE_STAGES.length-1, idx));
  const st = BATTLE_STAGES[battleSelIdx];
  battleActiveTheme = st.theme;
  BPLATS.length = 0;
  BPLATS.push({x:0,y:GY,w:800,h:60});
  st.plats.forEach(p=>BPLATS.push({x:p.x,y:p.y,w:p.w,h:p.h}));
}

const _initBattleOrig = initBattle;
window.initBattle = function(){
  applyBattleStage(battleSelIdx);
  _initBattleOrig();
};

window.drawBattle = function(){
  if(!BS)return;
  if(battleActiveTheme==='spacebg'){
    ctx.fillStyle='#00000c';ctx.fillRect(0,0,W,H);
    ctx.fillStyle='#fff';for(let i=0;i<50;i++){ctx.globalAlpha=0.5+0.5*Math.sin(titleFrame*0.05+i);ctx.fillRect((i*97+titleFrame*0.15)%W,(i*63)%H,1.5,1.5);}ctx.globalAlpha=1;
  } else {
    const savedTheme=theme, savedCam=cameraX;
    theme=battleActiveTheme; cameraX=0;
    drawBG();
    theme=savedTheme; cameraX=savedCam;
  }
  ctx.fillStyle='rgba(0,0,0,0.30)';ctx.fillRect(0,0,W,H);
  BPLATS.forEach(p=>{
    if(p.h>30){ctx.fillStyle='#d84000';ctx.fillRect(p.x,p.y,p.w,p.h);ctx.fillStyle='#fc9c00';ctx.fillRect(p.x,p.y,p.w,6);}
    else{ctx.fillStyle='#fc9c00';ctx.fillRect(p.x,p.y,p.w,p.h);ctx.strokeStyle='#000';ctx.lineWidth=2;ctx.strokeRect(p.x,p.y,p.w,p.h);}
  });
  BS.shells.forEach(s=>{ctx.fillStyle='#00a800';ctx.fillRect(s.x,s.y,s.w,s.h);ctx.fillStyle='#80d010';ctx.fillRect(s.x+4,s.y+4,s.w-8,s.h-8);ctx.strokeStyle='#003800';ctx.lineWidth=1;ctx.beginPath();ctx.moveTo(s.x+s.w/2,s.y);ctx.lineTo(s.x+s.w/2,s.y+s.h);ctx.stroke();ctx.beginPath();ctx.moveTo(s.x,s.y+s.h/2);ctx.lineTo(s.x+s.w,s.y+s.h/2);ctx.stroke();});
  function drawBP(bp,isL){if(bp.stun>0&&Math.floor(bp.stun/5)%2===0)return;const runs=[isL?lRun1:sRun1,isL?lRun2:sRun2,isL?lRun3:sRun3];const sp=bp.grnd&&Math.abs(bp.vx)>0.5?runs[bp.fr]:(isL?lIdle:sIdle);drawSpr(sp,bp.x,bp.y,3,bp.dir);}
  drawBP(BS.p1,false);drawBP(BS.p2,true);
  ctx.font='bold 17px monospace';ctx.fillStyle='#ff4444';ctx.fillText('P1 '+'❤'.repeat(BS.p1.hp)+'♡'.repeat(3-BS.p1.hp),10,26);
  ctx.fillStyle='#44ff44';ctx.fillText('P2 '+'❤'.repeat(BS.p2.hp)+'♡'.repeat(3-BS.p2.hp),560,26);
  ctx.fillStyle='#ffd700';ctx.font='bold 14px monospace';ctx.textAlign='center';
  ctx.fillText('⚔ '+(BATTLE_STAGES[battleSelIdx]?BATTLE_STAGES[battleSelIdx].name:'BATTLE MODE'),400,26);
  ctx.fillStyle='#888';ctx.font='12px monospace';ctx.fillText('P1:Arrow+Shift/X=甲羅  P2:WASD+Q=甲羅',400,430);ctx.textAlign='left';
  if(BS.cd>0){ctx.fillStyle='rgba(0,0,0,0.5)';ctx.fillRect(0,0,W,H);ctx.fillStyle='#fff';ctx.font='bold 70px monospace';ctx.textAlign='center';const c=Math.ceil(BS.cd/40);ctx.fillText(c>0?String(c):'FIGHT!',400,250);ctx.textAlign='left';}
  if(BS.result){ctx.fillStyle='rgba(0,0,0,0.6)';ctx.fillRect(0,0,W,H);ctx.fillStyle='#ffd700';ctx.font='bold 44px monospace';ctx.textAlign='center';ctx.fillText({p1win:'MARIO WINS!',p2win:'LUIGI WINS!',draw:'DRAW!'}[BS.result],400,200);ctx.fillStyle='#fff';ctx.font='bold 16px monospace';ctx.fillText('SPACE = Title',400,260);ctx.textAlign='left';}
};

function drawBattleSelect(){
  ctx.fillStyle='#101018';ctx.fillRect(0,0,W,H);
  ctx.fillStyle='#ffd700';ctx.font='bold 24px monospace';ctx.textAlign='center';
  ctx.fillText('⚔ BATTLE STAGE SELECT',400,42);
  ctx.fillStyle='#aaa';ctx.font='12px monospace';
  ctx.fillText('←→↑↓:選択  SPACE:決定  ESC:タイトルへ',400,64);
  ctx.textAlign='left';
  const cols=5,cellW=140,cellH=130,startX=(W-cols*cellW)/2,startY=90;
  BATTLE_STAGES.forEach((st,i)=>{
    const cx=startX+(i%cols)*cellW, cy=startY+Math.floor(i/cols)*cellH;
    const sel=(i===battleSelIdx);
    ctx.fillStyle=sel?'#334488':'#222';
    ctx.fillRect(cx+10,cy,cellW-20,cellH-20);
    ctx.strokeStyle=sel?'#fff':'#555';ctx.lineWidth=sel?3:1;ctx.strokeRect(cx+10,cy,cellW-20,cellH-20);
    ctx.fillStyle='#8899ff';ctx.font='11px monospace';ctx.textAlign='center';
    ctx.fillText('WORLD '+(i+1),cx+cellW/2,cy+22);
    ctx.fillStyle='#fff';ctx.font='bold 16px monospace';
    ctx.fillText(st.short,cx+cellW/2,cy+cellH/2+4);
    ctx.textAlign='left';
  });
}

// ---------------------------------------------------------------
// 【2】宇宙シューティング 2人プレイ対応
// ---------------------------------------------------------------
const SK2={l:false,r:false,u:false,d:false,shoot:false};
let spShootCd2=0;

const _initSpaceOrig = initSpace;
window.initSpace = function(st){
  _initSpaceOrig(st);
  spShootCd2=0;
  if(numP===2){
    SP.ship2={x:340,y:380,w:40,h:30,hp:3,iframes:0};
    SP.bullets2=[];
  } else {
    SP.ship2=null;
    SP.bullets2=[];
  }
};

const _updateSpaceOrig = updateSpace;
window.updateSpace = function(){
  _updateSpaceOrig();
  if(!SP||SP.cleared||SP.over)return;
  if(numP!==2||!SP.ship2)return;
  const sp=SP, ship2=sp.ship2;
  if(ship2.hp<=0)return;
  if(SK2.l&&ship2.x>5)ship2.x-=4.5; if(SK2.r&&ship2.x<755)ship2.x+=4.5;
  if(SK2.u&&ship2.y>5)ship2.y-=4.5; if(SK2.d&&ship2.y<400)ship2.y+=4.5;
  if(ship2.iframes>0)ship2.iframes--;
  if(spShootCd2>0)spShootCd2--;
  if(SK2.shoot&&spShootCd2===0){sp.bullets2.push({x:ship2.x+17,y:ship2.y,w:6,h:14,vy:-11,alive:true});spShootCd2=9;seFx('shoot');}
  sp.bullets2=sp.bullets2.filter(b=>b.alive&&b.y>-20);
  sp.bullets2.forEach(b=>b.y+=b.vy);
  sp.eBullets.forEach(b=>{
    if(ship2.iframes===0&&b.alive&&b.x<ship2.x+ship2.w&&b.x+8>ship2.x&&b.y<ship2.y+ship2.h&&b.y+8>ship2.y){
      ship2.hp--;ship2.iframes=90;b.alive=false;seFx('hit');if(ship2.hp<=0)sp.over=true;
    }
  });
  sp.enemies.forEach(e=>{
    if(!e.alive)return;
    if(ship2.iframes===0&&e.x<ship2.x+ship2.w&&e.x+e.w>ship2.x&&e.y<ship2.y+ship2.h&&e.y+e.h>ship2.y){
      ship2.hp--;ship2.iframes=90;e.alive=false;seFx('hit');if(ship2.hp<=0)sp.over=true;
    }
    sp.bullets2.forEach(b=>{
      if(b.alive&&e.alive&&b.x<e.x+e.w&&b.x+b.w>e.x&&b.y<e.y+e.h&&b.y+b.h>e.y){
        e.alive=false;b.alive=false;sp.kills++;seFx('stomp');
        if(sp.kills>=sp.goal){sp.cleared=true;SPACE_ST[sp.stage-1].cleared=true;seFx('clear');}
      }
    });
  });
};

const _drawSpaceOrig = drawSpace;
window.drawSpace = function(){
  _drawSpaceOrig();
  if(!SP||!SP.ship2)return;
  const ship2=SP.ship2;
  if(ship2.hp>0){
    ctx.fillStyle='#44ff88';ctx.beginPath();ctx.moveTo(ship2.x+20,ship2.y);ctx.lineTo(ship2.x,ship2.y+ship2.h);ctx.lineTo(ship2.x+ship2.w,ship2.y+ship2.h);ctx.closePath();ctx.fill();
    ctx.fillStyle='#aaffcc';ctx.fillRect(ship2.x+12,ship2.y+4,16,14);
    if(ship2.iframes>0){ctx.fillStyle='rgba(255,255,255,0.3)';ctx.beginPath();ctx.arc(ship2.x+20,ship2.y+15,28,0,Math.PI*2);ctx.fill();}
  }
  SP.bullets2.forEach(b=>{ctx.fillStyle='#44ffaa';ctx.fillRect(b.x,b.y,b.w,b.h);ctx.fillStyle='#ffffff';ctx.fillRect(b.x+1,b.y,b.w-2,4);});
  ctx.fillStyle='#66ff99';ctx.font='bold 15px monospace';ctx.textAlign='right';
  ctx.fillText('P2 HP '+'★'.repeat(Math.max(0,ship2.hp))+'☆'.repeat(3-Math.max(0,ship2.hp)),790,46);
  ctx.textAlign='left';
};


// ---------------------------------------------------------------
// 【3】メガマリオ (新パワーアップ)
// ---------------------------------------------------------------
// pl.mega / pl.megaT はmkPl側で未定義のままでも動作します(undefinedはfalsy)

function bulldozeMega(pl){
  const mx=pl.x-10, my=pl.y-10, mw=pl.w+20, mh=pl.h+20;
  for(let i=platforms.length-1;i>=0;i--){
    const p=platforms[i];
    if(p.type==='ground'||p.type==='vine')continue;
    if(mx<p.x+p.width && mx+mw>p.x && my<p.y+p.height && my+mh>p.y){
      platforms.splice(i,1);
    }
  }
}

const _updatePlayerOrig = updatePlayer;
window.updatePlayer = function(pl, ks, other){
  if(pl.dead||pl.clear){_updatePlayerOrig(pl,ks,other);return;}
  if(pl.mega){
    pl.megaT--;
    if(pl.megaT<=0){
      pl.mega=false; pl.w=48; pl.h=48; pl.big=true;
    } else {
      bulldozeMega(pl);
    }
  }
  if(pl.mega){
    const realStar=pl.star, realStarT=pl.starT;
    pl.star=true; pl.starT=5;
    _updatePlayerOrig(pl,ks,other);
    pl.star=realStar; pl.starT=realStarT;
  } else {
    _updatePlayerOrig(pl,ks,other);
  }
  // メガキノコ(？ブロック)判定: mega印つきブロックが叩かれたらアイテムをmegaに差し替え
  platforms.forEach(p=>{
    if(p.megaBlock && p.hit && !p._megaDone){
      p._megaDone=true;
      const cx=p.x+(p.width||40)/2;
      for(let i=items.length-1;i>=0;i--){
        const it=items[i];
        if(!it.col && Math.abs((it.x+it.w/2)-cx)<40 && Math.abs(it.y-(p.y-30))<60){ it.type='mega'; break; }
      }
    }
  });
  // メガキノコ取得判定
  items.forEach(it=>{
    if(it.col)return;
    if(it.type==='mega' && hit(pl,{x:it.x,y:it.y,w:it.w,h:it.h})){
      it.col=true; pl.mega=true; pl.megaT=600; pl.big=true; pl.w=96; pl.h=96; pl.score+=2000; seFx('item');
    }
  });
};

window.drawPl = function(pl){
  const sp=getSpr(pl);let mat;
  if(pl.dead)mat=sp.die;else if(pl.clear)mat=sp.idle;else if(!pl.grnd)mat=sp.jump;else if(Math.abs(pl.vx)>0.3)mat=sp.run[pl.fr];else mat=sp.idle;
  const scale = pl.mega?6:3;
  let dy=pl.y; if(pl.big||pl.fire)dy-=16*(pl.mega?2:1);
  if(pl.star&&Math.floor(pl.starT/4)%2===0)return;
  if(pl.icd>0&&Math.floor(pl.icd/6)%2===0)return;
  if(pl.mega){ctx.save();ctx.shadowColor='rgba(255,140,0,0.9)';ctx.shadowBlur=18;}
  drawSpr(mat,pl.x-(pl.mega?24:0),dy,scale,pl.dir);
  if(pl.mega)ctx.restore();
  if(pl.p2){ctx.fillStyle='#0f0';ctx.font='bold 11px monospace';ctx.fillText('P2',pl.x+16,pl.y-3);}
};

const _drawGameOrigMega = drawGame;
window.drawGame = function(){
  _drawGameOrigMega();
  if((MODE==='PLAYING'||MODE==='TUTORIAL')&&((P1&&P1.mega)||(numP===2&&P2&&P2.mega))){
    ctx.fillStyle='rgba(0,0,0,0.35)';ctx.fillRect(150,4,500,26);
    ctx.fillStyle='#ffcc00';ctx.font='bold 16px monospace';ctx.textAlign='center';
    const t1=P1&&P1.mega?Math.ceil(P1.megaT/60):0, t2=numP===2&&P2&&P2.mega?Math.ceil(P2.megaT/60):0;
    ctx.fillText('🍄 MEGA MARIO! 残り'+Math.max(t1,t2)+'秒 — 障害物や敵を弾き飛ばせ！',400,22);
    ctx.textAlign='left';
  }
};

// ---------------------------------------------------------------
// 【4】隠し土管ワープ: 1-2→W4, 4-2→W6
// ---------------------------------------------------------------
const _loadStageOrigSecret = loadStage;
window.loadStage = function(sd){
  _loadStageOrigSecret(sd);
  if(sd && sd.name==='1-2'){
    platforms.push({x:150,y:GY-90,width:64,height:90,type:'pipe',secretWarp:4});
  } else if(sd && sd.name==='4-2'){
    platforms.push({x:150,y:GY-90,width:64,height:90,type:'pipe',secretWarp:6});
  }
};

window.addEventListener('keydown', function(e){
  if(e.key!=='ArrowDown')return;
  if(MODE!=='PLAYING')return;
  const checkWarp=(pl)=>{
    if(!pl||pl.dead||pl.clear||!pl.grnd)return false;
    const sp = platforms.find(p=>p.secretWarp && pl.x+pl.w>p.x+8 && pl.x<p.x+p.width-8 && Math.abs((pl.y+pl.h)-p.y)<8);
    if(sp){ world=sp.secretWarp; mapIdx=0; MODE='WORLD_MAP'; bgmStop(); return true; }
    return false;
  };
  if(checkWarp(P1))return;
  if(numP===2)checkWarp(P2);
});

// ---------------------------------------------------------------
// 【5】エディットモード / 倉庫 / スペシャル1ワールド
// ---------------------------------------------------------------
const EDIT_PALETTE=[
  {key:'ground',label:'地面',color:'#5c3a1a'},
  {key:'pipe',label:'土管',color:'#00a800'},
  {key:'block',label:'ブロック',color:'#c87830'},
  {key:'question',label:'？',color:'#fc9c00'},
  {key:'megablock',label:'メガ？',color:'#ff00ff'},
  {key:'goomba',label:'クリボー',color:'#8B4513'},
  {key:'koopa',label:'ノコノコ',color:'#00a800'},
  {key:'coin',label:'コイン',color:'#ffd700'},
  {key:'vine',label:'つる',color:'#33aa33'},
  {key:'goal',label:'ゴール',color:'#ffffff'},
];
let EDIT=null;
let WAREHOUSE=[];
let SPECIAL1=[];
let __warehouseIdSeq=0;
let warehouseSelIdx=0, warehouseFocus='all';

function initEdit(){
  EDIT={camX:0,cursorTx:2,cursorTy:8,selIdx:0,cols:50,rows:10,cells:{},goalTx:null};
}

function drawEditCell(val,sx,sy,TILE){
  const p=EDIT_PALETTE.find(pp=>pp.key===val);
  if(!p)return;
  if(val==='goomba'||val==='koopa'){
    ctx.fillStyle=p.color;ctx.beginPath();ctx.arc(sx+TILE/2,sy+TILE/2,TILE/2-4,0,Math.PI*2);ctx.fill();
  } else if(val==='coin'){
    ctx.fillStyle='#ffd700';ctx.beginPath();ctx.arc(sx+TILE/2,sy+TILE/2,10,0,Math.PI*2);ctx.fill();
  } else {
    ctx.fillStyle=p.color;ctx.fillRect(sx+1,sy+1,TILE-2,TILE-2);
    ctx.strokeStyle='#000';ctx.strokeRect(sx+1,sy+1,TILE-2,TILE-2);
  }
  ctx.fillStyle='#fff';ctx.font='9px monospace';ctx.fillText(p.label[0],sx+4,sy+14);
}

function drawEdit(){
  if(!EDIT)initEdit();
  ctx.fillStyle='#223';ctx.fillRect(0,0,W,H);
  const TILE=40;
  ctx.strokeStyle='rgba(255,255,255,0.08)';ctx.lineWidth=1;
  for(let x=0;x<=W;x+=TILE){ctx.beginPath();ctx.moveTo(x,0);ctx.lineTo(x,H-20);ctx.stroke();}
  for(let y=0;y<=H-20;y+=TILE){ctx.beginPath();ctx.moveTo(0,y);ctx.lineTo(W,y);ctx.stroke();}
  const camTx=Math.floor(EDIT.camX/TILE);
  for(let tx=camTx;tx<camTx+21;tx++){
    for(let ty=0;ty<EDIT.rows;ty++){
      const key=tx+'_'+ty;const val=EDIT.cells[key];
      if(!val)continue;
      drawEditCell(val,tx*TILE-EDIT.camX,ty*TILE,TILE);
    }
  }
  if(EDIT.goalTx!=null){
    const sx=EDIT.goalTx*TILE-EDIT.camX;
    if(sx>-TILE&&sx<W){ctx.fillStyle='#00ff00';ctx.fillRect(sx+14,0,4,H-20);ctx.fillStyle='#0f0';ctx.font='bold 10px monospace';ctx.fillText('GOAL',sx-4,H-24);}
  }
  const csx=EDIT.cursorTx*TILE-EDIT.camX, csy=EDIT.cursorTy*TILE;
  ctx.strokeStyle='#ffff00';ctx.lineWidth=3;ctx.strokeRect(csx,csy,TILE,TILE);
  ctx.fillStyle='rgba(0,0,0,0.75)';ctx.fillRect(0,H-20,W,20);
  EDIT_PALETTE.forEach((p,i)=>{
    const px=6+i*79;
    ctx.fillStyle=i===EDIT.selIdx?'#ffff00':'#fff';
    ctx.font='bold 10px monospace';
    ctx.fillText(((i+1)%10)+':'+p.label,px,H-6);
  });
  ctx.fillStyle='#fff';ctx.font='11px monospace';
  ctx.fillText('矢印:移動  1-0:材料選択  SPACE:設置  BackSpace:削除  S:保存してタイトルへ  ESC:破棄してタイトルへ',10,14);
}

function saveEditCourse(){
  if(!EDIT)return;
  if(EDIT.goalTx==null){
    let maxTx=10;
    Object.keys(EDIT.cells).forEach(k=>{const tx=+k.split('_')[0];if(tx>maxTx)maxTx=tx;});
    EDIT.goalTx=maxTx+3;
  }
  const id=++__warehouseIdSeq;
  const name='MYCOURSE-'+id;
  WAREHOUSE.push({id,name,cleared:false,data:{cells:Object.assign({},EDIT.cells),goalTx:EDIT.goalTx,cols:EDIT.cols,rows:EDIT.rows}});
  if(SPECIAL1.length<10)SPECIAL1.push(id);
  syncSpecial1Stages();
  EDIT=null;
  MODE='TITLE';bgmStop();
}

function syncSpecial1Stages(){
  for(let i=STAGES.length-1;i>=0;i--) if(STAGES[i].w===12) STAGES.splice(i,1);
  SPECIAL1.forEach((wid,i)=>{
    const course=WAREHOUSE.find(c=>c.id===wid);
    if(!course)return;
    STAGES.push({id:20000+wid,w:12,name:'S1-'+(i+1),type:'custom',cleared:!!course.cleared,cx:100+i*70,cy:(i%2===0?200:280),_wid:wid});
  });
}

function loadCustomCourse(sd){
  platforms=[];coins=[];enemies=[];items=[];fireballs=[];gimmick={};
  theme='grassland';underwater=false;iceSlide=false;
  const course=WAREHOUSE.find(c=>c.id===sd._wid);
  const TILE=40;
  if(!course){
    platforms.push({x:0,y:GY,width:2000,height:60,type:'ground'});
    goal={x:1900,y:GY-260,w:10,h:260};
    bgmPlay('grassland');return;
  }
  Object.keys(course.data.cells).forEach(key=>{
    const parts=key.split('_');const tx=+parts[0],ty=+parts[1];
    const val=course.data.cells[key];
    const x=tx*TILE,y=ty*TILE;
    if(val==='ground')platforms.push({x,y,width:TILE,height:H-y,type:'ground'});
    else if(val==='pipe')platforms.push({x,y,width:TILE,height:H-y,type:'pipe'});
    else if(val==='block')platforms.push({x,y,width:TILE,height:TILE,type:'block'});
    else if(val==='question')platforms.push({x,y,width:TILE,height:TILE,type:'question',hit:false});
    else if(val==='megablock')platforms.push({x,y,width:TILE,height:TILE,type:'question',hit:false,megaBlock:true});
    else if(val==='coin')coins.push({x:x+8,y,w:16,h:24,col:false});
    else if(val==='vine')platforms.push({x,y,width:TILE,height:16,type:'vine',vy:0.8,baseY:y,dir:1});
    else if(val==='goomba')enemies.push({type:'goomba',x,y:y+TILE-32,w:32,h:32,vx:-1.2,vy:0,alive:true,sq:0});
    else if(val==='koopa')enemies.push({type:'koopa',x,y:y+TILE-40,w:32,h:40,vx:-1.2,vy:0,alive:true,shell:false,svx:0,sq:0});
  });
  platforms.push({x:0,y:GY+400,width:1,height:1,type:'ground'}); // ダミー(空配列対策、実質見えない)
  const gtx=(course.data.goalTx!=null)?course.data.goalTx:(course.data.cols-2);
  goal={x:gtx*TILE,y:GY-260,w:10,h:260};
  bgmPlay('grassland');
}

const _getCurStagesOrigEdit = getCurStages;
window.getCurStages = function(){
  if(world===12){
    if(!STAGES.some(s=>s.w===12)) syncSpecial1Stages();
    return STAGES.filter(s=>s.w===12).map(s=>({...s,canvasX:s.cx,canvasY:s.cy}));
  }
  return _getCurStagesOrigEdit();
};

const _getThemeOrigEdit = getTheme;
window.getTheme = function(sd){
  if(sd&&sd.type==='custom')return 'grassland';
  return _getThemeOrigEdit(sd);
};

const _loadStageOrigEdit = window.loadStage;
window.loadStage = function(sd){
  if(sd&&sd.type==='custom'){loadCustomCourse(sd);return;}
  _loadStageOrigEdit(sd);
};

// クリア済みフラグをWAREHOUSEへ反映
window.addEventListener('keydown', function(e){
  if(e.key!==' ')return;
  if(!(MODE==='PLAYING'||MODE==='TUTORIAL'))return;
  if(world!==12)return;
  if(!P1||!P1.clear)return;
  const s=STAGES.find(x=>x.id===stageId);
  if(s&&s._wid!=null){
    const c=WAREHOUSE.find(cc=>cc.id===s._wid);
    if(c)c.cleared=true;
    s.cleared=true;
  }
});

DB_ITEMS.push(
  {lbl:'🎨 スペシャル1へ',g:()=>'EXEC',s:()=>{syncSpecial1Stages();world=12;mapIdx=0;MODE='WORLD_MAP';bgmStop();}}
);

// --- 倉庫画面 ---
function drawWarehouse(){
  ctx.fillStyle='#151022';ctx.fillRect(0,0,W,H);
  ctx.fillStyle='#ffd700';ctx.font='bold 22px monospace';ctx.textAlign='center';
  ctx.fillText('🏭 倉庫 (コース管理)',400,32);
  ctx.fillStyle='#aaa';ctx.font='11px monospace';
  ctx.fillText('TAB:列切替  ↑↓:選択  SPACE:Special1へ追加/削除  [ ]:並び替え  ESC:戻る',400,50);
  ctx.textAlign='left';
  ctx.fillStyle='#fff';ctx.font='bold 13px monospace';ctx.fillText('全コース ('+WAREHOUSE.length+')',40,78);
  WAREHOUSE.forEach((c,i)=>{
    const y=98+i*22;
    const inSp=SPECIAL1.includes(c.id);
    if(warehouseFocus==='all'&&i===warehouseSelIdx){ctx.fillStyle='#334477';ctx.fillRect(30,y-14,340,20);}
    ctx.fillStyle=inSp?'#80d010':'#fff';ctx.font='12px monospace';
    ctx.fillText((inSp?'★':'　')+c.name+(c.cleared?' [CLEAR]':''),40,y);
  });
  if(WAREHOUSE.length===0){ctx.fillStyle='#666';ctx.font='12px monospace';ctx.fillText('(まだコースがありません。タイトルの[4]EDITから作成してください)',40,98);}
  ctx.fillStyle='#fff';ctx.font='bold 13px monospace';ctx.fillText('SPECIAL1 (最大10)',460,78);
  SPECIAL1.forEach((wid,i)=>{
    const c=WAREHOUSE.find(w=>w.id===wid);
    const y=98+i*22;
    if(warehouseFocus==='sp1'&&i===warehouseSelIdx){ctx.fillStyle='#334477';ctx.fillRect(450,y-14,300,20);}
    ctx.fillStyle='#fff';ctx.font='12px monospace';
    ctx.fillText((i+1)+'. '+(c?c.name:'???'),460,y);
  });
}

let __whKeyBuf='';
window.addEventListener('keydown', function(e){
  const k=e.key;
  if(k>='0'&&k<='9'&&k.length===1){
    __whKeyBuf+=k;if(__whKeyBuf.length>4)__whKeyBuf=__whKeyBuf.slice(-4);
    if(__whKeyBuf==='1985'){__whKeyBuf='';MODE='WAREHOUSE';warehouseFocus='all';warehouseSelIdx=0;bgmStop();}
  }
  if(MODE==='WAREHOUSE'){
    if(k==='Tab'){warehouseFocus=(warehouseFocus==='all')?'sp1':'all';warehouseSelIdx=0;e.preventDefault();}
    if(k==='ArrowUp'||k==='ArrowDown'){
      const len=warehouseFocus==='all'?WAREHOUSE.length:SPECIAL1.length;
      if(len>0){
        if(k==='ArrowUp')warehouseSelIdx=(warehouseSelIdx-1+len)%len;
        else warehouseSelIdx=(warehouseSelIdx+1)%len;
      }
    }
    if(k===' '&&warehouseFocus==='all'&&WAREHOUSE[warehouseSelIdx]){
      const c=WAREHOUSE[warehouseSelIdx];
      const idx=SPECIAL1.indexOf(c.id);
      if(idx>=0)SPECIAL1.splice(idx,1);
      else if(SPECIAL1.length<10)SPECIAL1.push(c.id);
      syncSpecial1Stages();
    }
    if((k==='['||k===']')&&warehouseFocus==='sp1'&&SPECIAL1.length>1){
      const i=warehouseSelIdx;const j=(k==='[')?i-1:i+1;
      if(j>=0&&j<SPECIAL1.length){const tmp=SPECIAL1[i];SPECIAL1[i]=SPECIAL1[j];SPECIAL1[j]=tmp;warehouseSelIdx=j;syncSpecial1Stages();}
    }
    if(k==='Escape'){MODE='TITLE';bgmStop();}
    e.preventDefault();
  }
});

// --- タイトル画面: EDITボタン(4番目)追加描画 & 分岐 ---
window.addEventListener('keydown', function(e){
  if(MODE==='TITLE'&&e.key==='4'){titleSel=4;}
}, true);

window.addEventListener('keydown', function(e){
  if(MODE==='TITLE'&&(e.key===' '||e.key==='Enter')){
    if(titleSel===3){e.stopImmediatePropagation();battleSelIdx=battleSelIdx||0;MODE='BATTLE_SELECT';return;}
    if(titleSel===4){e.stopImmediatePropagation();initEdit();MODE='EDIT';return;}
  }
}, true);

window.addEventListener('keydown', function(e){
  const k=e.key;
  if(MODE==='BATTLE_SELECT'){
    if(k==='ArrowRight')battleSelIdx=(battleSelIdx+1)%BATTLE_STAGES.length;
    if(k==='ArrowLeft')battleSelIdx=(battleSelIdx-1+BATTLE_STAGES.length)%BATTLE_STAGES.length;
    if(k==='ArrowDown')battleSelIdx=Math.min(BATTLE_STAGES.length-1,battleSelIdx+5);
    if(k==='ArrowUp')battleSelIdx=Math.max(0,battleSelIdx-5);
    if(k===' '||k==='Enter'){MODE='BATTLE';initBattle();}
    if(k==='Escape'){MODE='TITLE';}
    return;
  }
  if(MODE==='EDIT'){
    if(!EDIT)initEdit();
    if(k==='ArrowRight'){EDIT.cursorTx=Math.min(EDIT.cols-1,EDIT.cursorTx+1);if(EDIT.cursorTx*40-EDIT.camX>680)EDIT.camX+=40;}
    if(k==='ArrowLeft'){EDIT.cursorTx=Math.max(0,EDIT.cursorTx-1);if(EDIT.cursorTx*40<EDIT.camX)EDIT.camX=Math.max(0,EDIT.camX-40);}
    if(k==='ArrowDown')EDIT.cursorTy=Math.min(EDIT.rows-1,EDIT.cursorTy+1);
    if(k==='ArrowUp')EDIT.cursorTy=Math.max(0,EDIT.cursorTy-1);
    if(k>='1'&&k<='9')EDIT.selIdx=(+k)-1;
    if(k==='0')EDIT.selIdx=9;
    if(k===' '){
      const item=EDIT_PALETTE[EDIT.selIdx];
      if(item.key==='goal'){EDIT.goalTx=EDIT.cursorTx;}
      else{EDIT.cells[EDIT.cursorTx+'_'+EDIT.cursorTy]=item.key;}
    }
    if(k==='Backspace'||k==='Delete'){
      delete EDIT.cells[EDIT.cursorTx+'_'+EDIT.cursorTy];
      if(EDIT.goalTx===EDIT.cursorTx)EDIT.goalTx=null;
    }
    if(k==='s'||k==='S')saveEditCourse();
    if(k==='Escape'){EDIT=null;MODE='TITLE';bgmStop();}
    e.preventDefault();
    return;
  }
});

const _updateOrigEdit = window.update || update;
window.update = function(){
  if(MODE==='EDIT'||MODE==='BATTLE_SELECT'||MODE==='WAREHOUSE'){titleFrame++;return;}
  _updateOrigEdit();
};

const _drawOrigEdit = window.draw || draw;
window.draw = function(){
  if(MODE==='EDIT'){drawEdit();return;}
  if(MODE==='BATTLE_SELECT'){drawBattleSelect();return;}
  if(MODE==='WAREHOUSE'){drawWarehouse();return;}
  _drawOrigEdit();
  if(MODE==='TITLE'){
    const sel=(titleSel===4);
    ctx.fillStyle=sel?'#fc9c00':'#444';ctx.fillRect(640,228,130,55);
    ctx.strokeStyle=sel?'#fff':'#333';ctx.lineWidth=2;ctx.strokeRect(640,228,130,55);
    ctx.fillStyle=sel?'#000':'#aaa';ctx.font='bold 13px monospace';ctx.fillText('🎨 EDIT',650,268);
    ctx.font='11px monospace';ctx.fillText('[4]',650,283);
  }
};

// --- 保存/ロードにも倉庫・スペシャル1情報を含める ---
const _saveOrigEdit = save;
window.save = function(){
  try{
    const ok=_saveOrigEdit();
    const extra=JSON.parse(localStorage.getItem('marioSaveExtra')||'{}');
    extra.WAREHOUSE=WAREHOUSE;extra.SPECIAL1=SPECIAL1;extra.__warehouseIdSeq=__warehouseIdSeq;
    localStorage.setItem('marioSaveExtra',JSON.stringify(extra));
    return ok;
  }catch(e){return false;}
};
const _loadOrigEdit = load;
window.load = function(){
  const ok=_loadOrigEdit();
  try{
    const extra=JSON.parse(localStorage.getItem('marioSaveExtra')||'{}');
    if(extra.WAREHOUSE){WAREHOUSE=extra.WAREHOUSE;SPECIAL1=extra.SPECIAL1||[];__warehouseIdSeq=extra.__warehouseIdSeq||0;syncSpecial1Stages();}
  }catch(e){}
  return ok;
};

// ============================================================
// ★ 大型追加パック END ★
// ============================================================

// ============================================================
// ★ オンラインマルチプレイパック BEGIN ★
// PeerJS(WebRTC)のパブリック無料シグナリングサーバーを利用。
// ホストが部屋を作り、参加者はホストにのみ接続する「星形」構成。
// 最大16人まで、ワールド中どこからでもルームコードで合流可能。
// ============================================================

// ---- カラーチェンジ用パレット ----
const MP_COLORS=['#e52521','#049cd8','#43b047','#fbd000','#ff8c00','#a349a3','#00cccc','#ff69b4','#8b4513','#ffffff'];
const MP_COLOR_NAMES=['レッド(標準)','ブルー','グリーン','イエロー','オレンジ','パープル','シアン','ピンク','ブラウン','ホワイト'];
// パフォーマンス対策: 毎フレームのCSSフィルター(ctx.filter)は非常に重いため使わず、
// 色ごとにスプライトの'R'(赤)セルだけを差し替えた行列を一度だけ作ってキャッシュする。
const MP_SPR_CACHE={};
function mpRecolorMat(mat,color){return mat.map(row=>row.map(cell=>cell===R?color:cell));}
function mpGetSprSet(colorIdx){
  colorIdx=colorIdx||0;
  if(colorIdx===0)return{idle:sIdle,run:[sRun1,sRun2,sRun3],jump:sJump,die:sDie};
  if(MP_SPR_CACHE[colorIdx])return MP_SPR_CACHE[colorIdx];
  const c=MP_COLORS[colorIdx]||MP_COLORS[0];
  const set={idle:mpRecolorMat(sIdle,c),run:[mpRecolorMat(sRun1,c),mpRecolorMat(sRun2,c),mpRecolorMat(sRun3,c)],jump:mpRecolorMat(sJump,c),die:mpRecolorMat(sDie,c)};
  MP_SPR_CACHE[colorIdx]=set;
  return set;
}

// ---- マルチプレイ状態 ----
const MP={
  peer:null, isHost:false, roomCode:null, conns:{}, hostConn:null,
  myName:'PLAYER', myColorIdx:0, myId:null,
  players:{}, active:false, inPlay:false, sendCd:0, status:''
};
let onlineSel=1; // 1=ホスト 2=ジョイン

function mpMakeCode(){const cs='ABCDEFGHJKLMNPQRSTUVWXYZ23456789';let s='';for(let i=0;i<4;i++)s+=cs[Math.floor(Math.random()*cs.length)];return s;}

function mpResetNet(){
  try{if(MP.peer)MP.peer.destroy();}catch(err){}
  MP.peer=null;MP.isHost=false;MP.roomCode=null;MP.conns={};MP.hostConn=null;
  MP.players={};MP.active=false;MP.inPlay=false;
}
function mpLeave(){mpResetNet();MODE='TITLE';bgmStop();}

// ---- ホスト側 ----
function mpStartHost(attempt){
  attempt=attempt||0;
  if(typeof Peer==='undefined'){MP.status='オンライン機能の読み込みに失敗しました(通信環境を確認してください)';return;}
  if(attempt>6){MP.status='部屋を作成できませんでした。もう一度お試しください';return;}
  const code=mpMakeCode();
  MP.status='ルーム作成中...';
  const p=new Peer('mswd-'+code,{debug:0});
  MP.peer=p;
  p.on('open',id=>{
    MP.isHost=true;MP.roomCode=code;MP.myId=id;MP.active=true;MP.status='';
    MP.players[id]={name:MP.myName,colorIdx:MP.myColorIdx,x:100,y:200,dir:'right',fr:0,anim:'idle',big:false,fire:false,mini:false,dead:false,host:true};
  });
  p.on('connection',conn=>{
    if(Object.keys(MP.conns).length+1>=16){conn.on('open',()=>{try{conn.send({t:'full'});}catch(err){}conn.close();});return;}
    MP.conns[conn.peer]=conn;
    conn.on('data',data=>mpHostOnData(conn,data));
    conn.on('close',()=>{delete MP.conns[conn.peer];delete MP.players[conn.peer];mpBroadcastLobby();});
    conn.on('error',()=>{delete MP.conns[conn.peer];delete MP.players[conn.peer];});
  });
  p.on('error',err=>{
    if(err&&err.type==='unavailable-id'){try{p.destroy();}catch(e2){}mpStartHost(attempt+1);return;}
    MP.status='通信エラー: '+(err&&err.type?err.type:'unknown');
  });
}
function mpHostOnData(conn,data){
  if(!data||!data.t)return;
  if(data.t==='join'){
    MP.players[conn.peer]={name:(data.name||'PLAYER').slice(0,10),colorIdx:data.colorIdx||0,x:100,y:200,dir:'right',fr:0,anim:'idle',big:false,fire:false,mini:false,dead:false,host:false};
    mpBroadcastLobby();
  }else if(data.t==='state'){
    const pl=MP.players[conn.peer];if(!pl)return;
    Object.assign(pl,data.s);
  }
}
function mpBroadcastLobby(){
  if(!MP.isHost)return;
  const msg={t:'lobby',players:mpPlayerSummary()};
  Object.values(MP.conns).forEach(c=>{try{c.send(msg);}catch(err){}});
}
function mpPlayerSummary(){
  const out={};
  Object.keys(MP.players).forEach(id=>{const p=MP.players[id];out[id]={name:p.name,colorIdx:p.colorIdx,host:!!p.host};});
  return out;
}
function mpHostStartGame(){
  if(!MP.isHost)return;
  const seed=Math.floor(Math.random()*2147483647);
  const extra=!!gameCompleted; // ホストの進行状況に合わせて全員同じバリエーションにする
  const msg={t:'start',seed,extra};
  Object.values(MP.conns).forEach(c=>{try{c.send(msg);}catch(err){}});
  MP.inPlay=true;mpEnterTutorial(seed,extra);
}

// ---- 参加側 ----
function mpJoin(code){
  if(typeof Peer==='undefined'){MP.status='オンライン機能の読み込みに失敗しました(通信環境を確認してください)';return;}
  MP.status='接続中...';
  const p=new Peer({debug:0});
  MP.peer=p;
  p.on('open',id=>{
    MP.myId=id;
    const conn=p.connect('mswd-'+code.toUpperCase(),{serialization:'json',reliable:false});
    MP.hostConn=conn;
    conn.on('open',()=>{
      MP.roomCode=code.toUpperCase();MP.active=true;MP.isHost=false;MP.status='接続しました。ホストの開始を待っています...';
      conn.send({t:'join',name:MP.myName,colorIdx:MP.myColorIdx});
    });
    conn.on('data',data=>mpClientOnData(data));
    conn.on('close',()=>{MP.status='ホストとの接続が切れました';mpResetNet();MODE='ONLINE_MENU';});
    conn.on('error',()=>{MP.status='接続エラー';});
  });
  p.on('error',err=>{
    if(err&&err.type==='peer-unavailable'){MP.status='部屋が見つかりません: '+code;try{p.destroy();}catch(e2){}return;}
    MP.status='通信エラー: '+(err&&err.type?err.type:'unknown');
  });
}
function mpClientOnData(data){
  if(!data||!data.t)return;
  if(data.t==='full'){MP.status='この部屋は満員です(最大16人)';mpResetNet();MODE='ONLINE_MENU';return;}
  if(data.t==='lobby'){MP.players=data.players;return;}
  if(data.t==='start'){MP.inPlay=true;mpEnterTutorial(data.seed,data.extra);return;}
  if(data.t==='snapshot'){
    Object.keys(data.players).forEach(id=>{if(id!==MP.myId)MP.players[id]=Object.assign(MP.players[id]||{},data.players[id]);});
    Object.keys(MP.players).forEach(id=>{if(!data.players[id]&&id!==MP.myId)delete MP.players[id];});
  }
}

// ---- 共有アリーナステージ ----
// シード付き擬似乱数(mulberry32)。ホストが決めた同じ種を全員が使うことで、
// ステージ生成時にMath.random()で決まる部分(アイテムの中身など)も全員同じ結果になる。
function mpSeededRandom(seed){
  let s=seed>>>0;
  return function(){
    s|=0;s=(s+0x6D2B79F5)|0;
    let t=Math.imul(s^(s>>>15),1|s);
    t=(t+Math.imul(t^(t>>>7),61|t))^t;
    return ((t^(t>>>14))>>>0)/4294967296;
  };
}
function mpEnterTutorial(seed,extra){
  const origRandom=Math.random;
  Math.random=mpSeededRandom(seed>>>0);
  try{
    stageId=1;
    loadTutorial(!!extra);
  }finally{
    Math.random=origRandom;
  }
  resetGame();stageActive=true;numP=1;
  MODE='TUTORIAL';
}

function mpNetTick(){
  if(!MP.active)return;
  if(MODE==='WARP_ANIMATION'||MODE==='BATTLE'||MODE==='SPACE'||MODE==='DEBUG'||MODE==='EDIT'||MODE==='BATTLE_SELECT'||MODE==='WAREHOUSE')return;
  MP.sendCd++;if(MP.sendCd<6)return;MP.sendCd=0; // 約10回/秒に抑えて負荷軽減
  let state;
  if(MODE==='WORLD_MAP'){
    state={mode:'map',world:world,mapIdx:mapIdx};
  }else{
    const anim=P1.dead?'die':P1.clear?'idle':!P1.grnd?'jump':(Math.abs(P1.vx)>0.3?'run':'idle');
    state={mode:'play',x:Math.round(P1.x),y:Math.round(P1.y),dir:P1.dir,fr:P1.fr,anim,big:P1.big,fire:P1.fire,mini:P1.mini,dead:P1.dead};
  }
  if(MP.isHost){
    Object.assign(MP.players[MP.myId]||(MP.players[MP.myId]={}),state,{host:true});
    const msg={t:'snapshot',players:MP.players};
    Object.values(MP.conns).forEach(c=>{try{c.send(msg);}catch(err){}});
  }else if(MP.hostConn){
    try{MP.hostConn.send({t:'state',s:state});}catch(err){}
  }
}
function mpSprFor(anim,fr,colorIdx){
  const sp=mpGetSprSet(colorIdx);
  if(anim==='die')return sp.die;
  if(anim==='jump')return sp.jump;
  if(anim==='run')return sp.run[fr%3];
  return sp.idle;
}
function drawMpGhosts(){
  if(!MP.active)return;
  Object.keys(MP.players).forEach(id=>{
    if(id===MP.myId)return;
    const p=MP.players[id];if(!p||p.mode!=='play'||p.x==null)return;
    const mat=mpSprFor(p.anim,p.fr||0,p.colorIdx);
    let dy=p.y;if(p.big||p.fire)dy-=16;
    drawSpr(mat,p.x,dy,3,p.dir||'right');
    ctx.fillStyle='#fff';ctx.font='bold 10px monospace';ctx.textAlign='center';
    ctx.fillText((p.name||'PLAYER').slice(0,10),p.x+24,p.y-6);
    ctx.textAlign='left';
  });
}
function drawMpMapMarkers(){
  if(!MP.active)return;
  const ws=getCurStages();
  let n=0;
  Object.keys(MP.players).forEach(id=>{
    if(id===MP.myId)return;
    const p=MP.players[id];if(!p||p.mode!=='map'||p.world!==world)return;
    const node=ws[p.mapIdx];if(!node)return;
    const ox=(n%4)*10,oy=Math.floor(n/4)*10;n++;
    ctx.beginPath();ctx.fillStyle=MP_COLORS[p.colorIdx]||'#fff';
    ctx.arc(node.canvasX+56+ox,node.canvasY-10+oy,6,0,Math.PI*2);ctx.fill();
    ctx.lineWidth=1;ctx.strokeStyle='#000';ctx.stroke();
  });
}
function drawMpHud(){
  const n=Object.keys(MP.players).length||1;
  ctx.fillStyle='rgba(0,0,0,0.5)';ctx.fillRect(0,0,230,20);
  ctx.fillStyle='#ffd700';ctx.font='bold 11px monospace';
  ctx.fillText('🌐 '+(MP.roomCode||'-')+'  '+n+'/16人  ESC:退出',4,14);
}

// ---- タイトル画面のキー入力: [5]で ONLINE を選択 ----
window.addEventListener('keydown',function(e){
  if(MODE==='TITLE'&&e.key==='5')titleSel=5;
},true);
window.addEventListener('keydown',function(e){
  if(MODE==='TITLE'&&(e.key===' '||e.key==='Enter')&&titleSel===5){
    e.stopImmediatePropagation();MODE='ONLINE_MENU';onlineSel=1;MP.status='';bgmStop();
  }
},true);

// ---- オンラインメニュー/ロビーの入力 ----
window.addEventListener('keydown',function(e){
  const k=e.key;
  if(MODE==='ONLINE_MENU'){
    e.stopImmediatePropagation();e.preventDefault();
    if(k==='ArrowUp'||k==='ArrowDown')onlineSel=onlineSel===1?2:1;
    if(k==='ArrowLeft')MP.myColorIdx=(MP.myColorIdx-1+MP_COLORS.length)%MP_COLORS.length;
    if(k==='ArrowRight')MP.myColorIdx=(MP.myColorIdx+1)%MP_COLORS.length;
    if(k==='1')onlineSel=1;
    if(k==='2')onlineSel=2;
    if(k===' '||k==='Enter'){
      const nm=(prompt('プレイヤー名を入力してください(最大10文字)','PLAYER')||'PLAYER').trim().slice(0,10);
      MP.myName=nm||'PLAYER';
      if(onlineSel===1){MODE='ONLINE_HOST_WAIT';mpStartHost();}
      else{
        const code=prompt('参加するルームコード(4文字)を入力してください');
        if(code&&code.trim()){MODE='ONLINE_JOIN_WAIT';mpJoin(code.trim());}
      }
    }
    if(k==='Escape'){MODE='TITLE';bgmStop();}
    return;
  }
  if(MODE==='ONLINE_HOST_WAIT'){
    e.stopImmediatePropagation();e.preventDefault();
    if((k===' '||k==='Enter')&&MP.isHost&&Object.keys(MP.players).length>0)mpHostStartGame();
    if(k==='Escape')mpLeave();
    return;
  }
  if(MODE==='ONLINE_JOIN_WAIT'){
    e.stopImmediatePropagation();e.preventDefault();
    if(k==='Escape')mpLeave();
    return;
  }
  if(MP.active&&MODE!=='TITLE'&&k==='Escape'){
    e.stopImmediatePropagation();e.preventDefault();mpLeave();return;
  }
},true);

// ---- 描画: オンラインメニュー ----
function drawOnlineMenu(){
  ctx.fillStyle='#0a0a2a';ctx.fillRect(0,0,W,H);
  ctx.fillStyle='#fff';ctx.font='bold 26px monospace';ctx.textAlign='center';ctx.fillText('🌐 オンラインプレイ',400,55);
  ctx.font='12px monospace';ctx.fillStyle='#aaa';ctx.fillText('最大16人まで同時プレイ・世界中の誰とでも遊べます',400,78);
  ctx.textAlign='left';
  const bx1=220,bx2=440,by=110,bw=140,bh=58;
  ctx.fillStyle=onlineSel===1?'#fc9c00':'#333';ctx.fillRect(bx1,by,bw,bh);ctx.strokeStyle=onlineSel===1?'#fff':'#555';ctx.lineWidth=2;ctx.strokeRect(bx1,by,bw,bh);
  ctx.fillStyle=onlineSel===1?'#000':'#ccc';ctx.font='bold 14px monospace';ctx.fillText('[1] ホスト',bx1+18,by+26);ctx.fillText('ゲーム',bx1+18,by+44);
  ctx.fillStyle=onlineSel===2?'#fc9c00':'#333';ctx.fillRect(bx2,by,bw,bh);ctx.strokeStyle=onlineSel===2?'#fff':'#555';ctx.strokeRect(bx2,by,bw,bh);
  ctx.fillStyle=onlineSel===2?'#000':'#ccc';ctx.font='bold 14px monospace';ctx.fillText('[2] ジョイン',bx2+14,by+26);ctx.fillText('ゲーム',bx2+18,by+44);
  ctx.fillStyle='#fff';ctx.font='12px monospace';ctx.fillText('↑↓/1/2:選択  SPACE/ENTER:決定(名前入力あり)  ESC:戻る',180,200);
  ctx.fillStyle='#fff';ctx.font='13px monospace';ctx.fillText('カラーチェンジ (←→キーで変更):',180,235);
  MP_COLORS.forEach((c,i)=>{
    const x=180+i*44,y=248;
    ctx.fillStyle=c;ctx.fillRect(x,y,34,34);
    ctx.strokeStyle=i===MP.myColorIdx?'#fff':'#000';ctx.lineWidth=i===MP.myColorIdx?3:1;ctx.strokeRect(x,y,34,34);
  });
  ctx.fillStyle='#ffd700';ctx.font='12px monospace';ctx.fillText(MP_COLOR_NAMES[MP.myColorIdx],180,302);
  if(MP.status){ctx.fillStyle='#ff8080';ctx.font='12px monospace';ctx.fillText(MP.status,180,330);}
  ctx.fillStyle='#888';ctx.font='11px monospace';ctx.fillText('※ 相手のブラウザとインターネット経由でP2P接続します(公開シグナリングサーバー使用)',180,410);
}
function drawOnlineHostWait(){
  ctx.fillStyle='#0a0a2a';ctx.fillRect(0,0,W,H);
  ctx.fillStyle='#fff';ctx.font='bold 22px monospace';ctx.textAlign='center';ctx.fillText('ホストゲーム - ロビー',400,45);ctx.textAlign='left';
  if(!MP.roomCode){
    ctx.fillStyle='#ffd700';ctx.font='14px monospace';ctx.fillText(MP.status||'ルーム作成中...',260,150);return;
  }
  ctx.fillStyle='#000';ctx.fillRect(220,70,360,68);ctx.strokeStyle='#fc9c00';ctx.lineWidth=3;ctx.strokeRect(220,70,360,68);
  ctx.fillStyle='#aaa';ctx.font='12px monospace';ctx.fillText('このルームコードを友達に伝えてください',240,88);
  ctx.fillStyle='#fc9c00';ctx.font='bold 38px monospace';ctx.textAlign='center';ctx.fillText(MP.roomCode,400,126);ctx.textAlign='left';
  ctx.fillStyle='#fff';ctx.font='13px monospace';ctx.fillText('参加者 ('+Object.keys(MP.players).length+'/16):',220,163);
  Object.values(MP.players).forEach((p,i)=>{
    const y=182+i*20;
    ctx.fillStyle=MP_COLORS[p.colorIdx]||'#fff';ctx.fillRect(220,y-12,14,14);
    ctx.fillStyle='#fff';ctx.font='12px monospace';ctx.fillText((p.host?'★ ':'')+p.name,242,y);
  });
  ctx.fillStyle='#80ff80';ctx.font='bold 13px monospace';ctx.fillText('SPACE/ENTER: ゲーム開始    ESC: 部屋を閉じて戻る',220,415);
}
function drawOnlineJoinWait(){
  ctx.fillStyle='#0a0a2a';ctx.fillRect(0,0,W,H);
  ctx.fillStyle='#fff';ctx.font='bold 22px monospace';ctx.textAlign='center';ctx.fillText('ジョインゲーム',400,45);
  ctx.font='14px monospace';ctx.fillStyle='#ffd700';ctx.fillText(MP.status||'接続中...',400,90);ctx.textAlign='left';
  if(MP.roomCode){
    ctx.fillStyle='#fff';ctx.font='13px monospace';ctx.fillText('部屋: '+MP.roomCode,260,130);
    ctx.fillText('参加者 ('+Object.keys(MP.players).length+'/16):',260,155);
    Object.values(MP.players).forEach((p,i)=>{
      const y=175+i*20;
      ctx.fillStyle=MP_COLORS[p.colorIdx]||'#fff';ctx.fillRect(260,y-12,14,14);
      ctx.fillStyle='#fff';ctx.font='12px monospace';ctx.fillText((p.host?'★ ':'')+p.name,282,y);
    });
  }
  ctx.fillStyle='#aaa';ctx.font='13px monospace';ctx.fillText('ESC: キャンセルして戻る',260,400);
}

// ---- update / draw を再ラップ ----
const _mpUpdatePrev=window.update;
window.update=function(){
  if(MODE==='ONLINE_MENU'||MODE==='ONLINE_HOST_WAIT'||MODE==='ONLINE_JOIN_WAIT'){titleFrame++;return;}
  _mpUpdatePrev();
  if(MP.active)mpNetTick();
};
const _mpDrawPrev=window.draw;
window.draw=function(){
  if(MODE==='ONLINE_MENU'){drawOnlineMenu();return;}
  if(MODE==='ONLINE_HOST_WAIT'){drawOnlineHostWait();return;}
  if(MODE==='ONLINE_JOIN_WAIT'){drawOnlineJoinWait();return;}
  _mpDrawPrev();
  if(MP.active){
    if(MODE==='WORLD_MAP'){
      drawMpMapMarkers();
      const n=Object.keys(MP.players).length||1;
      ctx.fillStyle='rgba(0,0,0,0.5)';ctx.fillRect(W-150,0,150,20);
      ctx.fillStyle='#ffd700';ctx.font='bold 11px monospace';ctx.fillText('🌐 '+n+'/16人  ESC:退出',W-146,14);
    }else if(MODE==='PLAYING'||MODE==='TUTORIAL'){
      ctx.save();ctx.translate(-cameraX,0);drawMpGhosts();ctx.restore();
      drawMpHud();
    }
  }
  if(MODE==='TITLE'){
    const sel=(titleSel===5);
    ctx.fillStyle=sel?'#fc9c00':'#333';ctx.fillRect(640,300,130,40);
    ctx.strokeStyle=sel?'#fff':'#555';ctx.lineWidth=2;ctx.strokeRect(640,300,130,40);
    ctx.fillStyle=sel?'#000':'#aaa';ctx.font='bold 12px monospace';ctx.fillText('🌐 ONLINE',648,323);
    ctx.font='10px monospace';ctx.fillText('[5]',648,335);
  }
};

// ---- 自キャラの見た目にカラーチェンジを反映 ----
const _drawPlPrev=drawPl;
window.drawPl=function(pl){
  if(MP.active&&pl===P1&&MP.myColorIdx){
    // 自分の色を反映(キャッシュ済みスプライトを直接描画。ctx.filterは使わない=軽い)
    const sp=mpGetSprSet(MP.myColorIdx);
    let mat;
    if(pl.dead)mat=sp.die;else if(pl.clear)mat=sp.idle;else if(!pl.grnd)mat=sp.jump;else if(Math.abs(pl.vx)>0.3)mat=sp.run[pl.fr];else mat=sp.idle;
    let dy=pl.y;if(pl.big||pl.fire)dy-=16;
    if(pl.star&&Math.floor(pl.starT/4)%2===0)return;
    if(pl.icd>0&&Math.floor(pl.icd/6)%2===0)return;
    drawSpr(mat,pl.x,dy,3,pl.dir);
  }else{
    _drawPlPrev(pl);
  }
};

// ============================================================
// ★ オンラインマルチプレイパック END ★
// ============================================================

</script>
</body>
</html>
