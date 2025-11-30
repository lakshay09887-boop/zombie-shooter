<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Zombie Shooter — Single File (All Sounds)</title>
<style>
  :root{--bg:#07101a;--panel:rgba(255,255,255,0.03);--accent:#ff4d4f}
  html,body{height:100%;margin:0;font-family:Inter,system-ui,Arial;background:linear-gradient(180deg,#07101a,#0b0b12);color:#e6edf3}
  #wrap{display:flex;gap:12px;padding:14px;align-items:flex-start;flex-wrap:wrap;justify-content:center}
  canvas{background:#0c0c0f;border-radius:12px;box-shadow:0 12px 40px rgba(0,0,0,.6)}
  .panel{width:320px;padding:12px;background:var(--panel);border-radius:12px}
  h1{margin:0 0 8px;font-size:20px}
  .info{font-size:14px;opacity:0.95}
  button{background:var(--accent);border:none;color:white;padding:8px 12px;border-radius:8px;cursor:pointer}
  .small{font-size:13px;opacity:0.9}
  footer{position:fixed;left:14px;bottom:12px;color:#9aa6b2;font-size:12px}
  .hud-row{display:flex;justify-content:space-between;gap:8px;margin-top:10px}
  .slider{width:100%}
  #joystick{position:fixed;left:14px;bottom:14px;width:120px;height:120px;border-radius:50%;display:none;z-index:1000}
  #mobile-shoot{position:fixed;right:14px;bottom:14px;width:120px;height:120px;border-radius:50%;display:none;z-index:1000;background:rgba(255,255,255,0.03);display:flex;align-items:center;justify-content:center}
  #shopModal{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);background:var(--panel);padding:18px;border-radius:10px;display:none;z-index:2000;width:320px}
  @media (max-width:900px){ #joystick{display:block} #mobile-shoot{display:block} .panel{width:95%}}
</style>
</head>
<body>
<div id="wrap">
  <canvas id="game" width="1024" height="640"></canvas>

  <div class="panel">
    <h1>Zombie Shooter</h1>
    <div class="info">WASD / Arrow keys: Move • Mouse: Aim • Click: Shoot • R: Reload</div>

    <div class="hud-row">
      <div class="small">Health: <span id="health">100</span></div>
      <div class="small">Score: <span id="score">0</span></div>
      <div class="small">Wave: <span id="wave">1</span></div>
    </div>

    <div class="hud-row" style="margin-top:8px">
      <div class="small">Ammo: <span id="ammo">12</span>/<span id="maxAmmo">12</span></div>
      <div class="small">Coins: <span id="coins">0</span></div>
    </div>

    <div style="margin-top:12px;display:flex;gap:8px">
      <button id="restart">Restart</button>
      <button id="pause">Pause</button>
      <button id="shopBtn">Shop</button>
    </div>

    <hr style="margin:12px 0;border:none;border-top:1px solid rgba(255,255,255,0.04)">

    <div class="small">Volume</div>
    <div class="small">Music <input id="musicVol" class="slider" type="range" min="0" max="1" step="0.01" value="0.45"></div>
    <div class="small">SFX <input id="sfxVol" class="slider" type="range" min="0" max="1" step="0.01" value="0.8"></div>
    <div style="margin-top:8px;display:flex;gap:8px"><button id="muteBtn">Mute</button><button id="instructions" style="background:#334155">How to Play</button></div>

    <hr style="margin:12px 0;border:none;border-top:1px solid rgba(255,255,255,0.04)">

    <div class="small"><strong>Shop</strong> — Buy ammo or upgrades</div>
    <div style="margin-top:6px" class="small">
      Ammo pack (30) — 50 coins<br>
      Max ammo +4 — 100 coins
    </div>
  </div>
</div>

<div id="joystick"></div>
<div id="mobile-shoot">Shoot</div>

<div id="shopModal">
  <h3>Shop</h3>
  <div>Coins: <span id="modalCoins">0</span></div>
  <div style="margin-top:8px">
    <button id="buyAmmo">Buy Ammo Pack (50 coins)</button>
    <button id="buyMax">Increase Max Ammo (100 coins)</button>
  </div>
  <div style="margin-top:8px"><button id="closeShop">Close</button></div>
</div>

<footer>Save as <strong>index.html</strong> and host on GitHub Pages / Netlify for full audio behavior</footer>

<script>
/* ------------------------------
  ASSET URLs (public) - replace with your own if you want
  (these are free stock sounds; if any fail, replace with other hosted mp3)
------------------------------*/
const ASSETS = {
  audio: {
    // ambient horror loop (Pixabay example)
    bg: 'https://cdn.pixabay.com/download/audio/2023/06/27/audio_79fe9a306c.mp3?filename=horror-ambience-168887.mp3',
    // gunshot
    shoot: 'https://cdn.pixabay.com/download/audio/2022/03/17/audio_4e6ce8e7aa.mp3?filename=gunshot-1-6987.mp3',
    // hit / grunt
    hit: 'https://cdn.pixabay.com/download/audio/2021/08/04/audio_85f516d2d3.mp3?filename=zombie-growl-2-103369.mp3',
    // death
    death: 'https://cdn.pixabay.com/download/audio/2021/09/23/audio_6f0324f5d6.mp3?filename=creepy-scream-16131.mp3',
    // jumpscare sting
    jumpscare: 'https://cdn.pixabay.com/download/audio/2021/10/07/audio_ec0c7d8a9c.mp3?filename=jump-scare-167074.mp3'
  }
};

/* ------------------------------
  Game config
------------------------------*/
const CONFIG = {
  PLAYER_SPEED: 3.2,
  PLAYER_RADIUS: 16,
  BULLET_SPEED: 14,
  BULLET_LIFETIME: 80,
  ZOMBIE_SPAWN_INTERVAL: 120,
  ZOMBIE_SPEED: 0.9,
  ZOMBIE_DAMAGE: 8,
  ZOMBIE_HEALTH: 40,
  BOSS_HEALTH: 220,
  BOSS_SPEED: 0.6,
  RELOAD_TIME: 120, // frames for reload
  AMMO_ON_RELOAD: 12
};

/* ------------------------------
  DOM elements
------------------------------*/
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const healthEl = document.getElementById('health');
const scoreEl = document.getElementById('score');
const waveEl = document.getElementById('wave');
const restartBtn = document.getElementById('restart');
const pauseBtn = document.getElementById('pause');
const ammoEl = document.getElementById('ammo');
const maxAmmoEl = document.getElementById('maxAmmo');
const coinsEl = document.getElementById('coins');
const musicVolEl = document.getElementById('musicVol');
const sfxVolEl = document.getElementById('sfxVol');
const muteBtn = document.getElementById('muteBtn');
const shopBtn = document.getElementById('shopBtn');
const shopModal = document.getElementById('shopModal');
const modalCoins = document.getElementById('modalCoins');
const buyAmmoBtn = document.getElementById('buyAmmo');
const buyMaxBtn = document.getElementById('buyMax');
const closeShopBtn = document.getElementById('closeShop');
const joystick = document.getElementById('joystick');
const mobileShoot = document.getElementById('mobile-shoot');
const instructionsBtn = document.getElementById('instructions');

/* ------------------------------
  Game state
------------------------------*/
let keys = {};
let mouse = { x: canvas.width/2, y: canvas.height/2, down:false };
let touch = { left: null, right: null }; // for mobile
let frames = 0;
let paused = false;
let gameOver = false;

let player = { x: canvas.width/2, y: canvas.height/2, r: CONFIG.PLAYER_RADIUS, hp: 100, angle:0, ammo: CONFIG.AMMO_ON_RELOAD, maxAmmo: CONFIG.AMMO_ON_RELOAD, reloading:false, reloadTimer:0, coins:0 };
let bullets = [];
let zombies = [];
let effects = [];
let score = 0;
let wave = 1;
let spawnTimer = CONFIG.ZOMBIE_SPAWN_INTERVAL;

/* ------------------------------
  Audio
------------------------------*/
const audio = {
  bg: new Audio(ASSETS.audio.bg),
  shoot: new Audio(ASSETS.audio.shoot),
  hit: new Audio(ASSETS.audio.hit),
  death: new Audio(ASSETS.audio.death),
  jumpscare: new Audio(ASSETS.audio.jumpscare)
};
audio.bg.loop = true;
let audioStarted = false;
let muted = false;

function setVolumes() {
  audio.bg.volume = parseFloat(musicVolEl.value) * (muted?0:1);
  audio.shoot.volume = parseFloat(sfxVolEl.value) * (muted?0:1);
  audio.hit.volume = parseFloat(sfxVolEl.value) * (muted?0:1);
  audio.death.volume = parseFloat(sfxVolEl.value) * (muted?0:1);
  audio.jumpscare.volume = parseFloat(sfxVolEl.value) * (muted?0:1);
}
setVolumes();

/* ------------------------------
  Input
------------------------------*/
window.addEventListener('keydown', e => {
  keys[e.key.toLowerCase()] = true;
  if (e.key.toLowerCase() === 'r') tryReload();
});
window.addEventListener('keyup', e => {
  keys[e.key.toLowerCase()] = false;
});
canvas.addEventListener('mousemove', e => {
  const rect = canvas.getBoundingClientRect();
  mouse.x = (e.clientX - rect.left)*(canvas.width/rect.width);
  mouse.y = (e.clientY - rect.top)*(canvas.height/rect.height);
});
canvas.addEventListener('mousedown', e => {
  mouse.down = true; startAudioOnInteraction();
  shoot();
});
canvas.addEventListener('mouseup', e => mouse.down = false);

/* Mobile touch: left side move (virtual joystick), right side shoot */
canvas.addEventListener('touchstart', e => {
  startAudioOnInteraction();
  for (const t of e.changedTouches) {
    const rect = canvas.getBoundingClientRect();
    const x = (t.clientX - rect.left)*(canvas.width/rect.width);
    const y = (t.clientY - rect.top)*(canvas.height/rect.height);
    if (t.clientX < window.innerWidth/2 && !touch.left) {
      touch.left = { id: t.identifier, startX:x, startY:y, x, y, dx:0, dy:0 };
    } else if (!touch.right) {
      touch.right = { id: t.identifier, x, y };
      // right side: shoot once on touchstart
      shoot();
    }
  }
}, {passive:false});
canvas.addEventListener('touchmove', e => {
  for (const t of e.changedTouches) {
    if (touch.left && t.identifier === touch.left.id) {
      const rect = canvas.getBoundingClientRect();
      const x = (t.clientX - rect.left)*(canvas.width/rect.width);
      const y = (t.clientY - rect.top)*(canvas.height/rect.height);
      touch.left.x = x; touch.left.y = y; touch.left.dx = x - touch.left.startX; touch.left.dy = y - touch.left.startY;
    }
    if (touch.right && t.identifier === touch.right.id) {
      // aim toward touch
      const rect = canvas.getBoundingClientRect();
      const x = (t.clientX - rect.left)*(canvas.width/rect.width);
      const y = (t.clientY - rect.top)*(canvas.height/rect.height);
      mouse.x = x; mouse.y = y;
    }
  }
}, {passive:false});
canvas.addEventListener('touchend', e => {
  for (const t of e.changedTouches) {
    if (touch.left && t.identifier === touch.left.id) touch.left = null;
    if (touch.right && t.identifier === touch.right.id) touch.right = null;
  }
}, {passive:false});

/* mobile UI buttons */
mobileShoot.addEventListener('touchstart', e => { startAudioOnInteraction(); shoot(); }, {passive:false});
mobileShoot.addEventListener('mousedown', e => { startAudioOnInteraction(); shoot(); });

/* ------------------------------
  UI controls
------------------------------*/
restartBtn.onclick = () => resetGame();
pauseBtn.onclick = () => { paused = !paused; pauseBtn.textContent = paused? 'Resume' : 'Pause'; };
musicVolEl.oninput = setVolumes;
sfxVolEl.oninput = setVolumes;
muteBtn.onclick = () => { muted = !muted; muteBtn.textContent = muted? 'Unmute' : 'Mute'; setVolumes(); };
shopBtn.onclick = () => { shopModal.style.display = 'block'; modalCoins.textContent = player.coins; };
closeShopBtn.onclick = () => shopModal.style.display = 'none';
buyAmmoBtn.onclick = () => {
  if (player.coins >= 50) { player.coins -= 50; player.ammo += 30; coinsEl.textContent = player.coins; modalCoins.textContent = player.coins; }
  else alert('Not enough coins');
};
buyMaxBtn.onclick = () => {
  if (player.coins >= 100) { player.coins -= 100; player.maxAmmo += 4; maxAmmoEl.textContent = player.maxAmmo; coinsEl.textContent = player.coins; modalCoins.textContent = player.coins; }
  else alert('Not enough coins');
};
instructionsBtn.onclick = () => alert('Move: WASD or joystick (mobile left). Aim: Mouse. Shoot: Click / mobile right. Reload: R. Shop: Buy ammo or increase max ammo.');

/* ------------------------------
  Game functions
------------------------------*/
function startAudioOnInteraction(){
  if (audioStarted) return;
  audioStarted = true;
  audio.bg.play().catch(()=>{}); // some browsers need user interaction
  setVolumes();
}

function shoot(){
  if (gameOver) return;
  if (player.reloading) return;
  if (player.ammo <= 0) {
    // empty click sound? we skip
    return;
  }
  // start background audio first time
  startAudioOnInteraction();

  player.ammo--;
  const angle = Math.atan2(mouse.y - player.y, mouse.x - player.x);
  bullets.push({ x: player.x + Math.cos(angle)*player.r, y: player.y + Math.sin(angle)*player.r, vx: Math.cos(angle)*CONFIG.BULLET_SPEED, vy: Math.sin(angle)*CONFIG.BULLET_SPEED, life: CONFIG.BULLET_LIFETIME });
  audio.shoot.currentTime = 0; audio.shoot.play().catch(()=>{});
}

function tryReload(){
  if (player.reloading) return;
  if (player.ammo >= player.maxAmmo) return;
  player.reloading = true;
  player.reloadTimer = CONFIG.RELOAD_TIME;
}

function spawnZombie(isBoss=false){
  const edge = Math.floor(Math.random()*4);
  let x,y;
  if(edge===0){ x = -40; y = Math.random()*canvas.height; }
  else if(edge===1){ x = canvas.width + 40; y = Math.random()*canvas.height; }
  else if(edge===2){ x = Math.random()*canvas.width; y = -40; }
  else { x = Math.random()*canvas.width; y = canvas.height + 40; }
  zombies.push({
    x, y,
    r: isBoss ? 48 : 16,
    hp: isBoss ? CONFIG.BOSS_HEALTH : CONFIG.ZOMBIE_HEALTH,
    speed: isBoss ? CONFIG.BOSS_SPEED : CONFIG.ZOMBIE_SPEED,
    boss: !!isBoss
  });
}

/* ------------------------------
  Update & draw loop
------------------------------*/
function update(){
  if (paused || gameOver) return;

  frames++;

  // player movement
  let vx = 0, vy = 0;
  if (keys['w']||keys['arrowup']) vy -= 1;
  if (keys['s']||keys['arrowdown']) vy += 1;
  if (keys['a']||keys['arrowleft']) vx -= 1;
  if (keys['d']||keys['arrowright']) vx += 1;

  // mobile left joystick movement
  if (touch.left) {
    // use joystick delta to move
    const dx = touch.left.dx, dy = touch.left.dy;
    // small deadzone
    if (Math.hypot(dx,dy) > 4) {
      vx += dx * 0.05;
      vy += dy * 0.05;
      // update aim to right-ish area
      mouse.x = player.x + dx*4;
      mouse.y = player.y + dy*4;
    }
  }

  const len = Math.hypot(vx,vy) || 1;
  player.x += (vx/len) * CONFIG.PLAYER_SPEED;
  player.y += (vy/len) * CONFIG.PLAYER_SPEED;
  player.x = Math.max(player.r, Math.min(canvas.width - player.r, player.x));
  player.y = Math.max(player.r, Math.min(canvas.height - player.r, player.y));
  player.angle = Math.atan2(mouse.y - player.y, mouse.x - player.x);

  // auto-fire if mouse held
  if (mouse.down && frames % 8 === 0) shoot();

  // reload
  if (player.reloading) {
    player.reloadTimer--;
    if (player.reloadTimer <= 0) {
      player.reloading = false;
      const toLoad = Math.min(player.maxAmmo - player.ammo, CONFIG.AMMO_ON_RELOAD);
      player.ammo += toLoad;
    }
  }

  // bullets
  for (let i = bullets.length-1; i >= 0; i--) {
    const b = bullets[i];
    b.x += b.vx; b.y += b.vy; b.life--;
    if (b.life <= 0 || b.x < -50 || b.x > canvas.width+50 || b.y < -50 || b.y > canvas.height+50) bullets.splice(i,1);
  }

  // zombies
  for (let i = zombies.length-1; i >= 0; i--) {
    const z = zombies[i];
    const dx = player.x - z.x, dy = player.y - z.y;
    const dist = Math.hypot(dx,dy) || 1;
    z.x += (dx/dist) * z.speed;
    z.y += (dy/dist) * z.speed;

    // bullets collision
    for (let j = bullets.length-1; j >= 0; j--) {
      const b = bullets[j];
      const d = Math.hypot(b.x - z.x, b.y - z.y);
      if (d < z.r + 4) {
        z.hp -= 25;
        bullets.splice(j,1);
        audio.hit.currentTime = 0; audio.hit.play().catch(()=>{});
        if (z.hp <= 0) {
          score += z.boss ? 500 : 50;
          player.coins += z.boss ? 100 : 10;
          coinsEl.textContent = player.coins;
          audio.death.currentTime = 0; audio.death.play().catch(()=>{});
          // blood effect
          effects.push({ type:'blood', x:z.x, y:z.y, life:60 });
          zombies.splice(i,1);
          break;
        }
      }
    }

    // player collision
    const pd = Math.hypot(player.x - z.x, player.y - z.y);
    if (pd < player.r + z.r) {
      // push back
      const nx = (z.x - player.x) / (pd || 1);
      const ny = (z.y - player.y) / (pd || 1);
      z.x += nx * 6; z.y += ny * 6;
      if (frames % 30 === 0) {
        player.hp -= z.boss ? 40 : CONFIG.ZOMBIE_DAMAGE;
        if (player.hp <= 0) { player.hp = 0; endGame(); }
      }
    }
  }

  // spawn timer
  spawnTimer--;
  if (spawnTimer <= 0) {
    // occasionally spawn boss (every 6 waves chance)
    if (wave % 6 === 0 && Math.random() < 0.5) spawnZombie(true);
    else spawnZombie();
    spawnTimer = Math.max(40, Math.floor(CONFIG.ZOMBIE_SPAWN_INTERVAL * Math.pow(0.92, wave-1)));
  }

  if (score >= wave * 500) wave++;

  // effects update
  for (let i = effects.length-1; i >= 0; i--) {
    effects[i].life--;
    if (effects[i].life <= 0) effects.splice(i,1);
  }

  // UI updates
  healthEl.textContent = Math.max(0, Math.round(player.hp));
  scoreEl.textContent = score;
  waveEl.textContent = wave;
  ammoEl.textContent = player.ammo;
  maxAmmoEl.textContent = player.maxAmmo;
}

/* ------------------------------
  Draw
------------------------------*/
function draw(){
  ctx.clearRect(0,0,canvas.width,canvas.height);

  // subtle grid
  ctx.save();
  ctx.fillStyle = 'rgba(255,255,255,0.02)';
  for (let x=0;x<canvas.width;x+=40) ctx.fillRect(x,0,1,canvas.height);
  for (let y=0;y<canvas.height;y+=40) ctx.fillRect(0,y,canvas.width,1);
  ctx.restore();

  // bullets
  ctx.save();
  for (const b of bullets) { ctx.beginPath(); ctx.arc(b.x,b.y,3,0,Math.PI*2); ctx.fillStyle='white'; ctx.fill(); }
  ctx.restore();

  // zombies
  for (const z of zombies) {
    ctx.save();
    if (z.boss) {
      ctx.beginPath(); ctx.arc(z.x,z.y,z.r,0,Math.PI*2); ctx.fillStyle='rgba(90,20,20,0.96)'; ctx.fill();
      ctx.fillStyle='white'; ctx.fillRect(z.x-8, z.y-10, 6,6); ctx.fillRect(z.x+2, z.y-10, 6,6);
    } else {
      ctx.beginPath(); ctx.arc(z.x,z.y,z.r,0,Math.PI*2); ctx.fillStyle='rgba(150,40,40,0.95)'; ctx.fill();
      ctx.fillStyle='white'; ctx.fillRect(z.x-6, z.y-4,4,4); ctx.fillRect(z.x+2, z.y-4,4,4);
    }
    // health bar
    const hpFrac = Math.max(0, z.hp) / (z.boss ? CONFIG.BOSS_HEALTH : CONFIG.ZOMBIE_HEALTH);
    ctx.fillStyle = 'rgba(0,0,0,0.6)'; ctx.fillRect(z.x - z.r, z.y - z.r - 10, z.r*2, 6);
    ctx.fillStyle = 'lime'; ctx.fillRect(z.x - z.r, z.y - z.r - 10, (z.r*2) * hpFrac, 6);
    ctx.restore();
  }

  // player
  ctx.save();
  ctx.translate(player.x, player.y);
  ctx.rotate(player.angle);
  // body
  ctx.beginPath(); ctx.arc(0,0,player.r,0,Math.PI*2); ctx.fillStyle='rgba(60,140,200,0.98)'; ctx.fill();
  // gun
  ctx.fillStyle='rgba(40,40,40,0.95)'; ctx.fillRect(0,-6,player.r+14,12);
  ctx.restore();

  // blood effects
  for (const e of effects) {
    if (e.type === 'blood') {
      ctx.save();
      ctx.globalAlpha = e.life/60;
      ctx.beginPath(); ctx.arc(e.x, e.y, (60 - e.life) / 1.2 + 6, 0, Math.PI*2);
      ctx.fillStyle='rgba(120,0,0,0.95)'; ctx.fill();
      ctx.restore();
    }
  }

  // HUD coin text
  ctx.save(); ctx.fillStyle='rgba(255,255,255,0.9)'; ctx.font='14px sans-serif'; ctx.fillText('Coins: ' + player.coins, 12, 20); ctx.restore();

  if (gameOver) {
    ctx.save();
    ctx.fillStyle='rgba(0,0,0,0.6)'; ctx.fillRect(0,0,canvas.width,canvas.height);
    ctx.fillStyle='white'; ctx.font='36px sans-serif'; ctx.textAlign='center'; ctx.fillText('Game Over', canvas.width/2, canvas.height/2 - 20);
    ctx.font='20px sans-serif'; ctx.fillText('Score: ' + score, canvas.width/2, canvas.height/2 + 12);
    ctx.restore();
  }
}

/* ------------------------------
  Game loop
------------------------------*/
function loop(){
  update();
  draw();
  requestAnimationFrame(loop);
}

/* ------------------------------
  End game
------------------------------*/
function endGame(){
  gameOver = true;
  try { audio.jumpscare.currentTime = 0; audio.jumpscare.play().catch(()=>{}); } catch(e){}
  // stop bg after short delay
  setTimeout(()=>{ audio.bg.pause(); audio.bg.currentTime = 0; }, 1000);
}

/* ------------------------------
  Reset
------------------------------*/
function resetGame(){
  player.x = canvas.width/2; player.y = canvas.height/2; player.hp = 100; player.ammo = player.maxAmmo; player.coins = 0; bullets = []; zombies = []; effects = []; score = 0; wave = 1; spawnTimer = CONFIG.ZOMBIE_SPAWN_INTERVAL; frames = 0; paused = false; gameOver = false;
  audio.bg.currentTime = 0; audio.bg.play().catch(()=>{}); // may require user gesture
  setVolumes();
}

/* ------------------------------
  Start
------------------------------*/
resetGame();
loop();

/* ------------------------------
  Spawn booster: spawn a few initially
------------------------------*/
for (let i=0;i<4;i++) spawnZombie();

/* ------------------------------
  Periodic boss spawn (extra) and difficulty tune
------------------------------*/
setInterval(()=> {
  // small chance to spawn an extra zombie wave
  if (!gameOver && !paused) {
    if (Math.random() < 0.5) spawnZombie();
    if (Math.random() < 0.06) spawnZombie(true); // rare boss
  }
}, 3000);

/* ------------------------------
  Basic autoplay & attach UI values
------------------------------*/
coinsEl.textContent = player.coins;
modalCoins.textContent = player.coins;
setVolumes();

/* Ensure audio continues to reflect sliders */
musicVolEl.addEventListener('input', setVolumes);
sfxVolEl.addEventListener('input', setVolumes);

/* Prevent context menu on mobile long press */
canvas.addEventListener('contextmenu', e => e.preventDefault());

/* ------------------------------
  Resize handling — keep canvas fixed size visually but scale on small screens
------------------------------*/
function scaleCanvasToFit() {
  const maxWidth = Math.min(window.innerWidth - 40, 1024);
  const scale = Math.min(1, maxWidth / 1024);
  canvas.style.width = Math.round(1024 * scale) + 'px';
  canvas.style.height = Math.round(640 * scale) + 'px';
}
window.addEventListener('resize', scaleCanvasToFit);
scaleCanvasToFit();

/* ------------------------------
  Notes: Reload logic (R) and ammo cap
------------------------------*/
window.addEventListener('keydown', e => {
  if (e.key.toLowerCase() === 'r') tryReload();
});

/* ------------------------------
  Make sure audio volumes updated if user toggles mute via system
------------------------------*/
document.addEventListener('visibilitychange', () => {
  if (document.hidden) audio.bg.pause();
  else if (!gameOver && audioStarted) audio.bg.play().catch(()=>{});
});
</script>
</body>
</html>

