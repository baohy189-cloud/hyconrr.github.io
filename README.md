<!doctype html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Chúc Mừng Năm Mới — FINAL CLEAN</title>
  <style>
    *{box-sizing:border-box}
    html,body{margin:0;width:100%;height:100%;overflow:hidden;font-family:system-ui,-apple-system,Segoe UI,Roboto,sans-serif;background:radial-gradient(circle at top,#1f1b4d,#020617 65%);color:#fff}

    .bg-glow{position:absolute;inset:-20%;background:conic-gradient(#ef4444,#f59e0b,#fde047,#22c55e,#38bdf8,#a855f7,#ef4444);filter:blur(120px);opacity:.32;animation:spin 16s linear infinite;pointer-events:none}
    @keyframes spin{to{transform:rotate(360deg)}}

    .page{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;flex-direction:column;transition:opacity .5s ease}
    .hidden{opacity:0;pointer-events:none}

    .card{width:min(620px,92vw);padding:28px;border-radius:20px;background:rgba(255,255,255,.1);border:1px solid rgba(255,255,255,.25);backdrop-filter:blur(10px);text-align:center;z-index:2;box-shadow:0 20px 60px rgba(0,0,0,.45)}
    h1{margin:0 0 10px;font-size:clamp(2rem,3vw,3rem);background:linear-gradient(90deg,#f59e0b,#ef4444,#a855f7);-webkit-background-clip:text;background-clip:text;color:transparent}
    p{margin:0 0 14px;opacity:.95}
    input{width:100%;padding:12px 14px;border:none;border-radius:12px;font-size:1rem;margin-bottom:10px}
    button{width:100%;padding:12px 14px;border:none;border-radius:12px;font-weight:700;background:linear-gradient(135deg,#f59e0b,#ef4444);color:#fff;cursor:pointer}
    .hint{font-size:.9rem;opacity:.75;margin-top:8px}

    #fireworksCanvas{position:absolute;inset:0;width:100%;height:100%;pointer-events:none}
    .greeting{z-index:3;font-size:clamp(2rem,5vw,4rem);font-weight:800;text-align:center;text-shadow:0 10px 30px rgba(0,0,0,.6);padding:0 20px}
    .sub{z-index:3;margin-top:8px;opacity:.95}

    .next-btn{margin-top:12px;padding:6px 12px;font-size:.78rem;border-radius:999px;border:1px solid rgba(255,255,255,.22);background:rgba(255,255,255,.08);color:#fff;cursor:pointer;z-index:4;opacity:0;transform:translateY(8px);transition:.4s ease;width:auto}
    .next-btn.show{opacity:1;transform:translateY(0)}

    .video-overlay{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;z-index:10}
    .video-overlay.hidden{display:none}
    .video-backdrop{position:absolute;inset:0;background:rgba(0,0,0,.75);backdrop-filter:blur(4px)}
    .video-container{position:relative;width:min(960px,90vw);aspect-ratio:16/9;background:#000;border-radius:14px;overflow:hidden;z-index:2;box-shadow:0 20px 60px rgba(0,0,0,.6)}
    .video-container iframe{width:100%;height:100%;border:none}
    .close-video{position:absolute;top:10px;right:10px;width:30px;height:30px;border:none;border-radius:999px;background:rgba(0,0,0,.6);color:#fff;cursor:pointer;z-index:3}
  </style>
</head>
<body>
  <div class="bg-glow"></div>

  <audio id="tetMusic" loop preload="auto">
    <source src="https://cdn.pixabay.com/download/audio/2022/01/20/audio_8a15c2f0ef.mp3" type="audio/mpeg">
  </audio>

  <div id="page1" class="page">
    <div class="card">
      <h1>🎉 CHÚC MỪNG NĂM MỚI 🎉</h1>
      <p>Nhập tên để bắt đầu lễ hội</p>
      <input id="nameInput" placeholder="Tên của bạn..." />
      <button id="startBtn">Bắt đầu 🎆</button>
      <div class="hint">Nhấn Enter để tiếp tục</div>
    </div>
  </div>

  <div id="page2" class="page hidden">
    <canvas id="fireworksCanvas"></canvas>
    <div id="greetingText" class="greeting"></div>
    <div class="sub">Click để nổ thêm pháo hoa 🎇</div>
    <a id="nextVideoBtn" class="next-btn" href="https://www.youtube.com/watch?v=Ha6G5jlxuOY&list=RDHa6G5jlxuOY&start_radio=1" target="_blank" rel="noopener noreferrer">▶ Xem tiếp</a>
  </div>



<script>
(() => {
  // DOM
  const page1 = document.getElementById('page1');
  const page2 = document.getElementById('page2');
  const nameInput = document.getElementById('nameInput');
  const startBtn = document.getElementById('startBtn');
  const greeting = document.getElementById('greetingText');
  const nextVideoBtn = document.getElementById('nextVideoBtn');
  const tetMusic = document.getElementById('tetMusic');
  

  // audio context safe init
  let audioCtx = null;
  function ensureAudio() {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if (audioCtx.state === 'suspended') audioCtx.resume();
  }
  function boomSound() {
    if (!audioCtx) return;
    const o = audioCtx.createOscillator();
    const g = audioCtx.createGain();
    o.type = 'triangle';
    o.frequency.setValueAtTime(220, audioCtx.currentTime);
    o.frequency.exponentialRampToValueAtTime(60, audioCtx.currentTime + 0.45);
    g.gain.setValueAtTime(0.001, audioCtx.currentTime);
    g.gain.exponentialRampToValueAtTime(0.3, audioCtx.currentTime + 0.02);
    g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.5);
    o.connect(g).connect(audioCtx.destination);
    o.start(); o.stop(audioCtx.currentTime + 0.55);
  }

  // fireworks
  const canvas = document.getElementById('fireworksCanvas');
  const ctx = canvas.getContext('2d');
  let w=0,h=0;
  function resize(){ w = canvas.width = innerWidth; h = canvas.height = innerHeight; }
  addEventListener('resize', resize); resize();

  class Particle {
    constructor(x,y,c){
      this.x=x; this.y=y;
      const a=Math.random()*Math.PI*2, s=Math.random()*6+2;
      this.vx=Math.cos(a)*s; this.vy=Math.sin(a)*s;
      this.a=1; this.d=Math.random()*0.015+0.008;
      this.s=Math.random()*2+1.5; this.c=c;
    }
    update(){ this.x+=this.vx; this.y+=this.vy; this.vy+=0.03; this.a-=this.d; }
    draw(){ ctx.globalAlpha=Math.max(this.a,0); ctx.beginPath(); ctx.arc(this.x,this.y,this.s,0,Math.PI*2); ctx.fillStyle=this.c; ctx.fill(); }
  }

  const ps=[];
  function explode(x,y,count=150){
    const colors=['#f59e0b','#ef4444','#22c55e','#3b82f6','#e879f9','#fde047'];
    const c = colors[Math.floor(Math.random()*colors.length)];
    for(let i=0;i<count;i++) ps.push(new Particle(x,y,c));
    boomSound();
  }

  function anim(){
    ctx.fillStyle='rgba(2,6,23,.2)';
    ctx.fillRect(0,0,w,h);
    for(let i=ps.length-1;i>=0;i--){
      ps[i].update(); ps[i].draw();
      if(ps[i].a<=0) ps.splice(i,1);
    }
    ctx.globalAlpha=1;
    requestAnimationFrame(anim);
  }
  anim();

  let fireInterval=null;
  function startFireworks(){
    if(fireInterval) return;
    fireInterval=setInterval(()=>{
      explode(Math.random()*w*0.8+w*0.1, Math.random()*h*0.5+h*0.1);
    },520);
  }

  addEventListener('click',(e)=>{
    if(!page2.classList.contains('hidden')) explode(e.clientX,e.clientY,170);
  });

  function startExperience(){
    const name = nameInput.value.trim() || 'bạn';
    greeting.textContent = `Chúc mừng ${name}, năm mới vui vẻ! 🎊`;

    ensureAudio();
    tetMusic.play().catch(()=>{});

    page1.classList.add('hidden');
    page2.classList.remove('hidden');

    startFireworks();
    for(let i=0;i<6;i++) setTimeout(()=>explode(Math.random()*w*.8+w*.1,Math.random()*h*.45+h*.1,180),i*220);

    setTimeout(()=>nextVideoBtn.classList.add('show'), 2500);
  }

  // ENTER FIX (stable)
  startBtn.addEventListener('click', startExperience);
  nameInput.addEventListener('keydown', (e)=>{
    if(e.key === 'Enter'){
      e.preventDefault();
      startExperience();
    }
  });

  // Link Xem tiếp mở thẳng YouTube
  nextVideoBtn.addEventListener('click', ()=>{
    tetMusic.volume = 0.3;
  });
})();
</script>
</body>
</html>
