
[bunny_kiel_game.html](https://github.com/user-attachments/files/22949781/bunny_kiel_game.html)
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>I Love You Kiel Bunny Game</title>
  <style>
    :root{--bg:#f7f8fc;--ground:#cde6a7;--bunny:#fff;--accent:#ffb703}
    html,body{height:100%;margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial}
    body{background:linear-gradient(180deg,var(--bg),#eaf6ff);display:flex;align-items:center;justify-content:center}
    .wrap{width:min(920px,96vw);height:min(600px,86vh);background:linear-gradient(#ffffffcc, #f0fffdcc);border-radius:16px;box-shadow:0 8px 30px rgba(20,30,50,0.12);overflow:hidden;position:relative}
    canvas{display:block;background:linear-gradient(#dff3ff, #f8fff0);width:100%;height:100%}

    .hud{position:absolute;left:14px;top:14px;background:rgba(255,255,255,0.85);padding:8px 12px;border-radius:10px;backdrop-filter:blur(4px);font-weight:600}
    .hint{position:absolute;right:14px;top:14px;background:rgba(255,255,255,0.85);padding:8px 12px;border-radius:10px;backdrop-filter:blur(4px);font-weight:500;font-size:13px}

    .overlay{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;pointer-events:none}
    .modal{pointer-events:auto;width:80%;max-width:520px;background:white;padding:22px;border-radius:14px;box-shadow:0 18px 40px rgba(20,40,80,0.2);text-align:center}
    .modal h1{margin:0 0 6px;font-size:26px}
    .modal p{margin:6px 0 18px;color:#334155}
    .btn{display:inline-block;padding:10px 14px;border-radius:10px;background:var(--accent);color:#111;font-weight:700;text-decoration:none;cursor:pointer}

    .small{font-size:13px;color:#374151}
    /* confetti pieces */
    .confetti-piece{position:absolute;width:10px;height:16px;opacity:0.95;border-radius:2px}

    /* instruction footer */
    .footer{position:absolute;left:50%;transform:translateX(-50%);bottom:12px;background:rgba(255,255,255,0.75);padding:6px 12px;border-radius:999px;font-size:13px}

    @media (max-width:540px){.modal{width:92%;padding:16px}.hud,.hint{font-size:12px}}
  </style>
</head>
<body>
  <div class="wrap" id="gameWrap">
    <canvas id="gameCanvas" width="800" height="520"></canvas>
    <div class="hud" id="score">Sunflowers: 0 / 3</div>
    <div class="hint">Use arrow keys or tap to move</div>
    <div class="footer">Pro tip: Bunny eats flowers when center overlaps them ❤️</div>
    <div class="overlay" id="overlay" style="display:none">
      <div class="modal" id="modal">
        <h1>Yay! The bunny's full 🐰✨</h1>
        <p id="cuteMsg">I love you, Kiel. You make my heart hop. 💖</p>
        <div style="display:flex;gap:8px;justify-content:center">
          <button class="btn" id="playAgain">Play again</button>
        </div>
      </div>
    </div>
  </div>

  <script>
    // Simple game engine using canvas
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    let W = canvas.width, H = canvas.height;

    function resizeCanvas(){
      const rect = canvas.getBoundingClientRect();
      canvas.style.width = rect.width + 'px';
      canvas.style.height = rect.height + 'px';
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // Game state
    const bunny = {x:100,y:240,r:26, speed:160};
    const flowers = [];
    let eaten = 0;
    const totalFlowers = 3;
    let keys = {};
    let lastTime = 0;
    let finished = false;

    function placeFlowers(){
      flowers.length = 0;
      const padding = 70;
      for(let i=0;i<totalFlowers;i++){
        const fx = padding + Math.random()*(W - padding*2);
        const fy = padding + Math.random()*(H - padding*2);
        flowers.push({x:fx,y:fy,r:20, eaten:false});
      }
    }

    function resetGame(){
      bunny.x = 80; bunny.y = H/2; eaten = 0; finished = false; placeFlowers();
      document.getElementById('score').textContent = `Sunflowers: ${eaten} / ${totalFlowers}`;
      hideModal();
    }

    function drawBunny(x,y,r){
      ctx.save();
      ctx.translate(x,y);
      ctx.beginPath(); ctx.ellipse(0+4, r+8, r*0.9, r*0.4, 0,0,Math.PI*2); ctx.fillStyle='rgba(30,30,30,0.06)'; ctx.fill();

      ctx.fillStyle = '#fff';
      ctx.beginPath(); ctx.ellipse(-12,-r-10,8,18, -0.2,0,Math.PI*2); ctx.fill();
      ctx.beginPath(); ctx.ellipse(12,-r-10,8,18, 0.2,0,Math.PI*2); ctx.fill();
      ctx.fillStyle='#ffd9e6'; ctx.beginPath(); ctx.ellipse(-12,-r-8,4,10,-0.2,0,Math.PI*2); ctx.fill();
      ctx.beginPath(); ctx.ellipse(12,-r-8,4,10,0.2,0,Math.PI*2); ctx.fill();

      ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(0,0,r,0,Math.PI*2); ctx.fill();
      ctx.fillStyle='#333'; ctx.beginPath(); ctx.arc(-8,-4,3,0,Math.PI*2); ctx.fill(); ctx.beginPath(); ctx.arc(8,-4,3,0,Math.PI*2); ctx.fill();
      ctx.fillStyle='#ff7aa2'; ctx.beginPath(); ctx.arc(0,4,4,0,Math.PI*2); ctx.fill();
      ctx.beginPath(); ctx.arc(0,8,8,0,Math.PI); ctx.strokeStyle='#e3a0b6'; ctx.lineWidth=1.4; ctx.stroke();

      ctx.fillStyle='#ffe0ea'; ctx.beginPath(); ctx.ellipse(-14,2,4,2,0,0,Math.PI*2); ctx.fill(); ctx.beginPath(); ctx.ellipse(14,2,4,2,0,0,Math.PI*2); ctx.fill();
      ctx.restore();
    }

    function drawFlower(f){
      ctx.save(); ctx.translate(f.x,f.y);
      ctx.beginPath(); ctx.moveTo(0,10); ctx.lineTo(0,40); ctx.strokeStyle='#2a7a2a'; ctx.lineWidth=6; ctx.stroke();
      for(let i=0;i<12;i++){
        ctx.rotate(Math.PI*2/12);
        ctx.beginPath(); ctx.ellipse(0,-12,8,14,0,0,Math.PI*2); ctx.fillStyle='#ffd54a'; ctx.fill();
      }
      ctx.beginPath(); ctx.arc(0,0,9,0,Math.PI*2); ctx.fillStyle='#6b3b1b'; ctx.fill();
      ctx.restore();
    }

    function update(dt){
      if(finished) return;
      let vx = 0, vy = 0;
      if(keys['ArrowLeft']||keys['a']) vx -= 1;
      if(keys['ArrowRight']||keys['d']) vx += 1;
      if(keys['ArrowUp']||keys['w']) vy -= 1;
      if(keys['ArrowDown']||keys['s']) vy += 1;
      if(vx!==0 || vy!==0){
        const len = Math.hypot(vx,vy); vx/=len; vy/=len;
        bunny.x += vx * bunny.speed * dt;
        bunny.y += vy * bunny.speed * dt;
      }
      bunny.x = Math.max(bunny.r+8, Math.min(W-bunny.r-8, bunny.x));
      bunny.y = Math.max(bunny.r+8, Math.min(H-bunny.r-8, bunny.y));

      for(const f of flowers){
        if(f.eaten) continue;
        const dx = f.x - bunny.x; const dy = f.y - bunny.y; const dist = Math.hypot(dx,dy);
        if(dist < f.r + bunny.r*0.6){
          f.eaten = true; eaten++;
          document.getElementById('score').textContent = `Sunflowers: ${eaten} / ${totalFlowers}`;
          spawnPop(f.x,f.y);
          if(eaten >= totalFlowers){
            finished = true; showModal();
          }
        }
      }
    }

    const pops = [];
    function spawnPop(x,y){
      for(let i=0;i<14;i++){
        pops.push({x,y,vx:(Math.random()-0.5)*220, vy:(Math.random()-0.8)*220, life:0.6+Math.random()*0.6, color: ['#ffd54a','#ffb703','#ffecd1'][Math.floor(Math.random()*3)]});
      }
    }

    function draw(dt){
      ctx.clearRect(0,0,W,H);
      ctx.fillStyle = '#f6fff0'; ctx.fillRect(0,H-72,W,72);

      for(const f of flowers){ if(!f.eaten) drawFlower(f); }

      for(let i=pops.length-1;i>=0;i--){
        const p = pops[i]; p.life -= dt; if(p.life<=0) {pops.splice(i,1); continue}
        ctx.globalAlpha = Math.max(0, p.life/1.0);
        ctx.fillStyle = p.color; ctx.beginPath(); ctx.ellipse(p.x,p.y,4,6,0,0,Math.PI*2); ctx.fill();
        p.x += p.vx*dt; p.y += p.vy*dt; p.vy += 300*dt;
        ctx.globalAlpha =1;
      }

      drawBunny(bunny.x,bunny.y,bunny.r);
    }

    function loop(ts){
      if(!lastTime) lastTime = ts; const dt = Math.min(0.033, (ts-lastTime)/1000); lastTime = ts;
      update(dt); draw(dt);
      requestAnimationFrame(loop);
    }

    window.addEventListener('keydown', e=>{keys[e.key]=true;});
    window.addEventListener('keyup', e=>{keys[e.key]=false;});

    let pointerActive = false;
    canvas.addEventListener('pointerdown', e=>{pointerActive=true; moveToward(e);});
    window.addEventListener('pointerup', ()=>{pointerActive=false;});
    canvas.addEventListener('pointermove', e=>{ if(pointerActive) moveToward(e); });

    function moveToward(e){
      const rect = canvas.getBoundingClientRect();
      const cx = (e.clientX - rect.left) * (W / rect.width);
      const cy = (e.clientY - rect.top) * (H / rect.height);
      const dx = cx - bunny.x; const dy = cy - bunny.y; const dist = Math.hypot(dx,dy);
      if(dist>4){ const step = Math.min(dist, 24); bunny.x += dx/dist * step; bunny.y += dy/dist * step; }
    }

    const overlay = document.getElementById('overlay');
    const modal = document.getElementById('modal');
    const playAgainBtn = document.getElementById('playAgain');
    playAgainBtn.addEventListener('click', ()=>{ resetGame(); });

    function showModal(){
      overlay.style.display = 'flex'; overlay.style.pointerEvents='auto';
      const msgs = [
        "I love you, Kiel. You make my heart hop.",
        "The bunny's tummy is full — love you lots!"
      ];
      document.getElementById('cuteMsg').textContent = msgs[Math.floor(Math.random()*msgs.length)];
      spawnConfetti();
    }
    function hideModal(){ overlay.style.display='none'; overlay.style.pointerEvents='none'; removeConfetti(); }

    const confettiContainer = document.createElement('div'); confettiContainer.style.position='absolute'; confettiContainer.style.left=0; confettiContainer.style.top=0; confettiContainer.style.right=0; confettiContainer.style.bottom=0; confettiContainer.style.pointerEvents='none'; document.getElementById('gameWrap').appendChild(confettiContainer);
    let confettiPieces = [];
    function spawnConfetti(){ confettiPieces.length=0; for(let i=0;i<80;i++){ confettiPieces.push({x: Math.random()*W, y:-10-Math.random()*200, vx:(Math.random()-0.5)*120, vy:50+Math.random()*160, rot:Math.random()*360, size:6+Math.random()*8, color:['#ff6b6b','#ffd93d','#6be4b4','#8ab4ff'][Math.floor(Math.random()*4)], el:null}); }
      confettiContainer.innerHTML='';
      for(const c of confettiPieces){ const el = document.createElement('div'); el.className='confetti-piece'; el.style.background = c.color; el.style.width = c.size+'px'; el.style.height = (c.size+6)+'px'; el.style.left = c.x+'px'; el.style.top = c.y+'px'; el.style.transform = `rotate(${c.rot}deg)`; confettiContainer.appendChild(el); c.el = el; }
      let t0 = performance.now(); function run(){ if(confettiPieces.length==0) return; const t = performance.now(); const dt = Math.min(0.033,(t-t0)/1000); t0=t; for(let i=confettiPieces.length-1;i>=0;i--){ const p=confettiPieces[i]; p.x += p.vx*dt; p.y += p.vy*dt; p.vy += 240*dt; p.rot += 220*dt; if(p.y > H+40){ p.el.remove(); confettiPieces.splice(i,1); continue } p.el.style.left = p.x + 'px'; p.el.style.top = p.y + 'px'; p.el.style.transform = `rotate(${p.rot}deg)`; } if(confettiPieces.length>0) requestAnimationFrame(run); }
      requestAnimationFrame(run);
    }
    function removeConfetti(){ confettiPieces.forEach(c=>{ if(c.el) c.el.remove(); }); confettiPieces.length=0; confettiContainer.innerHTML=''; }

    window.addEventListener('load', ()=>{ resetGame(); requestAnimationFrame(loop); });
  </script>
</body>
</html>
