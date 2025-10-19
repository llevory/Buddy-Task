// Buddy Hunt â€” progressive quiz
(() => {
  const questions = [
    {
      id: 1,
      text: 'è”¡å¾å¤æˆåæ›²æ­Œè¯é‡Œï¼Œå”±ï¼Œè·³ï¼Œrap çš„ä¸‹ä¸€å¥æ˜¯ä»€ä¹ˆï¼Ÿ',
      options: ['ç¯®çƒ', 'è¶³çƒ', 'ä¹’ä¹“çƒ'],
      correctIndex: 0
    },
    {
      id: 2,
      text: 'ã€Šé¸¡ä½ å¤ªç¾Žã€‹çš„ç»ƒä¹ æ—¶é•¿æ˜¯å¤šä¹…ï¼Ÿ',
      options: ['ä¸¤åˆ†åŠ', 'ä¸¤å¹´åŠ', 'å‡‰æ‹Œ'],
      correctIndex: 1
    },
    {
      id: 3,
      text: 'ä½ æ˜¯è”¡å¾å¤çš„çœŸçˆ±ç²‰å—ï¼Ÿ',
      options: ['çœŸçˆ±ç²‰', 'å°é»‘å­'],
      correctIndex: 0
    }
  ];

  // DOM
  const questionArea = document.getElementById('questionArea');
  const submitBtn = document.getElementById('submitBtn');
  const nextBtn = document.getElementById('nextBtn');
  const feedback = document.getElementById('feedback');
  const progressBar = document.getElementById('progressBar');

  let current = 0;
  let selectedOption = null;

  function renderQuestion() {
    const q = questions[current];
    progressBar.style.width = `${(current / questions.length) * 100}%`;
    feedback.textContent = '';
    feedback.className = 'feedback';
    submitBtn.disabled = true;
    nextBtn.hidden = true;
    selectedOption = null;

    // build HTML
    questionArea.innerHTML = '';
    const title = document.createElement('div');
    title.className = 'q-title';
    title.textContent = `ç¬¬ ${current + 1} é¢˜ï¼š ${q.text}`;
    questionArea.appendChild(title);

    const opts = document.createElement('div');
    opts.className = 'options';
    q.options.forEach((opt, idx) => {
      const o = document.createElement('label');
      o.className = 'option';
      o.tabIndex = 0;
      o.setAttribute('data-idx', idx);

      const radio = document.createElement('input');
      radio.type = 'radio';
      radio.name = 'opt';
      radio.value = idx;
      radio.setAttribute('aria-label', opt);

      const span = document.createElement('span');
      span.className = 'label';
      span.textContent = opt;

      o.appendChild(radio);
      o.appendChild(span);
      opts.appendChild(o);

      // click/select handlers
      o.addEventListener('click', () => {
        selectOption(idx, o);
      });
      o.addEventListener('keydown', (e) => {
        if (e.key === 'Enter' || e.key === ' ') {
          e.preventDefault();
          selectOption(idx, o);
        }
      });
    });

    questionArea.appendChild(opts);
  }

  function selectOption(idx, node) {
    selectedOption = idx;
    // mark UI
    document.querySelectorAll('.option').forEach(el => el.classList.remove('selected'));
    node.classList.add('selected');
    // check radio
    const input = node.querySelector('input');
    if (input) input.checked = true;
    submitBtn.disabled = false;
    feedback.textContent = '';
  }

  function showFeedback(text, type) {
    feedback.textContent = text;
    feedback.className = 'feedback ' + type;
  }

  function handleSubmit() {
    if (selectedOption === null) return;
    const q = questions[current];
    if (selectedOption === q.correctIndex) {
      showFeedback('å›žç­”æ­£ç¡® âœ”', 'success');
      // mark progress to next
      nextBtn.hidden = false;
      submitBtn.disabled = true;
      runConfetti();
    } else {
      // wrong: allow retry
      animateShake();
      showFeedback('å›žç­”é”™è¯¯ï¼Œè¯·å†è¯•ä¸€æ¬¡ã€‚', 'error');
    }
  }

  function handleNext() {
    current++;
    if (current >= questions.length) {
      // finished
      progressBar.style.width = '100%';
      showFinalMessage();
    } else {
      renderQuestion();
    }
  }

  function showFinalMessage() {
    questionArea.innerHTML = `
      <div class="q-title">æ­å–œä½ ç­”å¯¹äº†æ‰€æœ‰é—®é¢˜ï¼</div>
      <div style="margin-top:10px;color:var(--muted)">
        çŽ°åœ¨è¯·åˆ° <strong>Dry Labå¤–çš„204 locker</strong> é‡ŒèŽ·å–ä½ çš„çº¿ç´¢ï¼
      </div>
    `;
    nextBtn.hidden = true;
    submitBtn.hidden = true;
    feedback.textContent = '';
    runConfetti(900);
  }

  function animateShake() {
    const card = document.querySelector('.card');
    card.animate(
      [{ transform: 'translateX(0)' }, { transform: 'translateX(-8px)' }, { transform: 'translateX(8px)' }, { transform: 'translateX(0)' }],
      { duration: 360, easing: 'cubic-bezier(.2,.8,.2,1)' }
    );
  }

  // Basic confetti implementation
  function runConfetti(duration = 1400) {
    const canvas = document.getElementById('confettiCanvas');
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    // resize
    canvas.width = innerWidth;
    canvas.height = innerHeight;

    const pieces = [];
    const colors = ['#ffb86b', '#7ce7c7', '#8be9fd', '#ff8bcb', '#caa2ff'];

    for (let i = 0; i < 80; i++) {
      pieces.push({
        x: Math.random() * canvas.width,
        y: Math.random() * -canvas.height,
        w: 6 + Math.random() * 8,
        h: 8 + Math.random() * 8,
        vx: -2 + Math.random() * 4,
        vy: 2 + Math.random() * 6,
        color: colors[Math.floor(Math.random() * colors.length)],
        rot: Math.random() * Math.PI * 2,
        vr: -0.1 + Math.random() * 0.2
      });
    }

    let start = performance.now();
    function frame(now) {
      const t = now - start;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      pieces.forEach(p => {
        p.x += p.vx;
        p.y += p.vy;
        p.vy += 0.02; // gravity
        p.rot += p.vr;
        ctx.save();
        ctx.translate(p.x, p.y);
        ctx.rotate(p.rot);
        ctx.fillStyle = p.color;
        ctx.fillRect(-p.w / 2, -p.h / 2, p.w, p.h);
        ctx.restore();
      });
      if (t < duration) {
        requestAnimationFrame(frame);
      } else {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
      }
    }
    requestAnimationFrame(frame);
    // auto-clear after duration
    setTimeout(() => {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    }, duration + 250);
  }

  // Event listeners
  submitBtn.addEventListener('click', handleSubmit);
  nextBtn.addEventListener('click', handleNext);

  // Initialize
  renderQuestion();

  // Accessibility: allow Enter on submit button when focused
  submitBtn.addEventListener('keydown', (e) => {
    if ((e.key === 'Enter' || e.key === ' ') && !submitBtn.disabled) {
      e.preventDefault();
      handleSubmit();
    }
  });

  // make progress reach 100% when final
  const observer = new MutationObserver(() => {
    if (current >= questions.length) progressBar.style.width = '100%';
  });
  observer.observe(document.getElementById('app'), { childList: true, subtree: true });
})();

:root{
  --bg1: #0f172a;
  --bg2: #0b1220;
  --card: rgba(255,255,255,0.04);
  --accent: #ffb86b;
  --accent-2: #7ce7c7;
  --glass: rgba(255,255,255,0.03);
  --text: #e6eef8;
  --muted: #98a2b3;
  --danger: #ff6b6b;
  --success: #6ee7b7;
  font-family: "Inter", system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
}

*{box-sizing:border-box}
html,body{height:100%}
body{
  margin:0;
  min-height:100%;
  background: radial-gradient(1200px 600px at 10% 10%, rgba(124,58,237,0.12), transparent),
              linear-gradient(180deg,var(--bg1),var(--bg2));
  color:var(--text);
  -webkit-font-smoothing:antialiased;
  -moz-osx-font-smoothing:grayscale;
  display:flex;
  align-items:center;
  justify-content:center;
  padding:24px;
}

.wrap{
  width:100%;
  max-width:720px;
  margin:24px;
}

.header{
  text-align:center;
  margin-bottom:18px;
}
h1{
  margin:0;
  font-weight:800;
  letter-spacing: -0.02em;
  font-size:1.6rem;
}
.subtitle{
  margin:6px 0 0;
  color:var(--muted);
  font-size:0.95rem;
}

/* Card */
.card{
  background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
  border:1px solid rgba(255,255,255,0.04);
  padding:20px;
  border-radius:14px;
  box-shadow: 0 8px 30px rgba(2,6,23,0.6);
  backdrop-filter: blur(6px);
}

/* Progress */
.progress{
  height:10px;
  background:var(--glass);
  border-radius:999px;
  overflow:hidden;
  margin-bottom:16px;
}
.progress__fill{
  height:100%;
  background: linear-gradient(90deg,var(--accent), var(--accent-2));
  width:0%;
  transition:width 450ms cubic-bezier(.2,.9,.3,1);
}

/* Question */
.question-area{
  min-height:180px;
  display:flex;
  flex-direction:column;
  gap:14px;
}
.q-title{
  font-weight:700;
  font-size:1.1rem;
}
.options{
  display:flex;
  flex-direction:column;
  gap:10px;
  margin-top:6px;
}
.option{
  display:flex;
  align-items:center;
  gap:12px;
  padding:12px 14px;
  border-radius:10px;
  background:rgba(255,255,255,0.02);
  border:1px solid rgba(255,255,255,0.02);
  cursor:pointer;
  transition: transform 120ms ease, box-shadow 120ms;
  user-select:none;
}
.option:hover{ transform:translateY(-3px); box-shadow: 0 6px 18px rgba(2,6,23,0.6);}
.option input{display:none}
.option .label{
  font-size:0.98rem;
}
.option.selected{
  outline: 2px solid rgba(124,58,237,0.14);
  background: linear-gradient(90deg, rgba(124,58,237,0.06), rgba(124,58,237,0.02));
}

/* Controls */
.controls{
  display:flex;
  gap:12px;
  margin-top:18px;
  align-items:center;
}
.btn{
  padding:10px 16px;
  border-radius:10px;
  border:1px solid rgba(255,255,255,0.04);
  background:transparent;
  color:var(--text);
  cursor:pointer;
  font-weight:600;
}
.btn[disabled]{opacity:0.45; cursor:not-allowed}
.btn.primary{
  background:linear-gradient(90deg,var(--accent),var(--accent-2));
  color:#081025;
  box-shadow: 0 8px 24px rgba(124,58,237,0.12);
  border:none;
}

.feedback{
  margin-top:12px;
  min-height:26px;
  font-weight:600;
}

/* Footer */
.footer{ text-align:center; margin-top:14px; color:var(--muted); font-size:0.9rem; }

/* Success / Error styles */
.feedback.success{ color:var(--success) }
.feedback.error{ color:var(--danger) }

/* Confetti canvas */
.confetti-canvas{
  position:fixed;
  pointer-events:none;
  width:100%;
  height:100%;
  left:0;
  top:0;
}

/* Responsive */
@media (max-width:520px){
  .wrap{ padding:0 6px }
  h1{ font-size:1.25rem }
}

