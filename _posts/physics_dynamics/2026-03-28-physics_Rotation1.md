
<h2 class="sr-only">막대의 회전운동 시뮬레이션 — 접선 속도와 접선 가속도 화살표 표시, 드래그로 초기값 조정 가능</h2>

<div style="border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-lg);overflow:hidden;font-family:var(--font-sans);">

  <div style="padding:12px 16px;background:var(--color-background-secondary);border-bottom:0.5px solid var(--color-border-tertiary);display:flex;align-items:center;gap:8px;flex-wrap:wrap;">
    <span style="font-family:var(--font-serif);font-size:14px;font-weight:500;color:var(--color-text-primary);margin-right:8px;">회전운동 (Rotational Motion)</span>
    <button id="rod-start" style="width:40px;height:32px;cursor:pointer;background:#475569;color:white;border:none;border-radius:6px;font-size:13px;">&#9654;</button>
    <button id="rod-reset" style="width:40px;height:32px;cursor:pointer;background:#475569;color:white;border:none;border-radius:6px;font-size:15px;">&#8635;</button>
    <div style="width:1px;height:20px;background:var(--color-border-secondary);margin:0 4px;"></div>
    <div style="display:flex;align-items:center;gap:16px;font-family:var(--font-serif);font-size:13px;color:var(--color-text-secondary);">
      <span>ω = <b id="rod-omega" style="color:var(--color-text-primary);">0.0200</b> rad/f</span>
      <span>α = <b id="rod-alpha" style="color:var(--color-text-primary);">2.00×10⁻⁴</b> rad/f²</span>
      <span>θ = <b id="rod-theta" style="color:var(--color-text-primary);">30.0</b>°</span>
      <span>L = <b id="rod-len" style="color:var(--color-text-primary);">280</b> px</span>
    </div>
    <div style="margin-left:auto;display:flex;gap:12px;font-size:12px;color:var(--color-text-secondary);align-items:center;">
      <span style="display:flex;align-items:center;gap:5px;"><span style="display:inline-block;width:14px;height:3px;background:#e53e3e;border-radius:2px;"></span>v (접선속도)</span>
      <span style="display:flex;align-items:center;gap:5px;"><span style="display:inline-block;width:14px;height:3px;background:#dd6b20;border-radius:2px;"></span>a (접선가속도)</span>
    </div>
  </div>

  <canvas id="rod-canvas" width="680" height="440" style="display:block;background:var(--color-background-primary);cursor:crosshair;width:100%;"></canvas>

  <div style="padding:6px 16px;background:var(--color-background-secondary);border-top:0.5px solid var(--color-border-tertiary);font-size:11px;color:var(--color-text-secondary);text-align:center;">
    정지 상태: 화살표 끝 드래그 → ω/α 조정 &nbsp;|&nbsp; 막대 끝 드래그 → 길이 조정 &nbsp;|&nbsp; 재생 중 막대 클릭 → 해당 지점의 접선 속도 표시
  </div>
</div>

<script>
(function() {
  const canvas = document.getElementById('rod-canvas');
  const ctx = canvas.getContext('2d');
  const W = 680, H = 440;
  const CX = 340, CY = 220;
  const ROD_R = 10;
  const V_DISP = 38;
  const A_DISP = 3200;
  const A_OFF = 18;

  let angle = Math.PI / 6;
  let omega = 0.020;
  let alpha = 0.00020;
  let rodHL = 140;
  const init = () => ({ angle: Math.PI/6, omega: 0.020, alpha: 0.00020, rodHL: 140 });
  let state = init();

  let isMoving = false;
  let dragging = null;
  let popup = null;

  const startBtn = document.getElementById('rod-start');
  const resetBtn = document.getElementById('rod-reset');

  startBtn.onclick = () => {
    isMoving = !isMoving;
    startBtn.innerHTML = isMoving ? '&#10074;&#10074;' : '&#9654;';
    startBtn.style.background = isMoving ? '#ffc107' : '#475569';
    startBtn.style.color = isMoving ? '#000' : '#fff';
  };

  resetBtn.onclick = () => {
    isMoving = false;
    const s = init();
    angle = s.angle; omega = s.omega; alpha = s.alpha; rodHL = s.rodHL;
    popup = null;
    startBtn.innerHTML = '&#9654;';
    startBtn.style.background = '#475569';
    startBtn.style.color = '#fff';
  };

  const cos = () => Math.cos(angle);
  const sin = () => Math.sin(angle);
  const getTang = () => ({ x: -Math.sin(angle), y: Math.cos(angle) });
  const getEnd1 = () => ({ x: CX + rodHL * cos(), y: CY + rodHL * sin() });
  const getEnd2 = () => ({ x: CX - rodHL * cos(), y: CY - rodHL * sin() });
  const vArrowLen = () => omega * rodHL * V_DISP;
  const aArrowLen = () => alpha * rodHL * A_DISP;
  const getAStart = () => {
    const e1 = getEnd1();
    return { x: e1.x - A_OFF * cos(), y: e1.y - A_OFF * sin() };
  };

  function dist2(a, b) { return Math.hypot(a.x - b.x, a.y - b.y); }
  function dotV(a, b) { return a.x * b.x + a.y * b.y; }

  function getMousePos(e) {
    const r = canvas.getBoundingClientRect();
    return {
      x: (e.clientX - r.left) * W / r.width,
      y: (e.clientY - r.top) * H / r.height
    };
  }

  function hitRod(m) {
    const dx = m.x - CX, dy = m.y - CY;
    const ca = cos(), sa = sin();
    const along = dx * ca + dy * sa;
    const perp = Math.abs(-dx * sa + dy * ca);
    if (Math.abs(along) <= rodHL + 4 && perp <= ROD_R + 10) return along;
    return null;
  }

  function drawRod3D() {
    ctx.save();
    ctx.translate(CX, CY);
    ctx.rotate(angle);
    const L = rodHL, R = ROD_R;

    function makePath() {
      ctx.beginPath();
      ctx.moveTo(-L, -R);
      ctx.lineTo(L, -R);
      ctx.arc(L, 0, R, -Math.PI/2, Math.PI/2);
      ctx.lineTo(-L, R);
      ctx.arc(-L, 0, R, Math.PI/2, -Math.PI/2);
      ctx.closePath();
    }

    ctx.shadowColor = 'rgba(0,0,0,0.22)';
    ctx.shadowBlur = 10;
    ctx.shadowOffsetX = 3;
    ctx.shadowOffsetY = 5;
    makePath();
    ctx.fillStyle = '#2d4a60';
    ctx.fill();
    ctx.shadowColor = 'transparent';

    const g = ctx.createLinearGradient(0, -R, 0, R);
    g.addColorStop(0,    '#0d1b2a');
    g.addColorStop(0.15, '#2d4a60');
    g.addColorStop(0.42, '#7aa8c4');
    g.addColorStop(0.5,  '#c8dce8');
    g.addColorStop(0.58, '#7aa8c4');
    g.addColorStop(0.82, '#2d4a60');
    g.addColorStop(1,    '#0d1b2a');

    makePath();
    ctx.fillStyle = g;
    ctx.fill();

    makePath();
    ctx.save();
    ctx.clip();
    const sg = ctx.createLinearGradient(0, -R, 0, -R*0.1);
    sg.addColorStop(0, 'rgba(255,255,255,0.48)');
    sg.addColorStop(1, 'rgba(255,255,255,0)');
    ctx.fillStyle = sg;
    ctx.fillRect(-L - R, -R, (L + R) * 2, R);
    ctx.restore();

    ctx.strokeStyle = 'rgba(255,255,255,0.12)';
    ctx.lineWidth = 0.8;
    makePath();
    ctx.stroke();

    ctx.restore();
  }

  function drawPivot() {
    const isDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    ctx.beginPath();
    ctx.arc(CX, CY, 14, 0, Math.PI * 2);
    ctx.fillStyle = isDark ? '#2a3540' : '#d8e4ec';
    ctx.fill();
    ctx.strokeStyle = isDark ? '#4a6070' : '#8aa8c0';
    ctx.lineWidth = 1.5;
    ctx.stroke();

    ctx.beginPath();
    ctx.arc(CX, CY, 7, 0, Math.PI * 2);
    ctx.fillStyle = isDark ? '#4a6070' : '#5a7890';
    ctx.fill();

    ctx.beginPath();
    ctx.arc(CX, CY, 3, 0, Math.PI * 2);
    ctx.fillStyle = isDark ? '#a0c0d8' : '#c8dce8';
    ctx.fill();
  }

  function drawArrow(x, y, vx, vy, color, label) {
    if (Math.hypot(vx, vy) < 2) return;
    const a = Math.atan2(vy, vx);
    const hl = 13;

    ctx.save();
    ctx.strokeStyle = 'rgba(255,255,255,0.7)';
    ctx.lineWidth = 5;
    ctx.lineCap = 'round';
    ctx.beginPath();
    ctx.moveTo(x, y);
    ctx.lineTo(x + vx, y + vy);
    ctx.stroke();

    ctx.strokeStyle = color;
    ctx.lineWidth = 2.5;
    ctx.beginPath();
    ctx.moveTo(x, y);
    ctx.lineTo(x + vx, y + vy);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(x + vx, y + vy);
    ctx.lineTo(x + vx - hl * Math.cos(a - Math.PI/7), y + vy - hl * Math.sin(a - Math.PI/7));
    ctx.lineTo(x + vx - hl * Math.cos(a + Math.PI/7), y + vy - hl * Math.sin(a + Math.PI/7));
    ctx.closePath();
    ctx.fillStyle = color;
    ctx.fill();

    if (label) {
      const mx = x + vx * 0.55, my = y + vy * 0.55;
      const pa = a - Math.PI/2;
      const tx = mx + 16 * Math.cos(pa), ty = my + 16 * Math.sin(pa);
      ctx.font = "bold 15px 'Times New Roman', serif";
      ctx.strokeStyle = 'rgba(255,255,255,0.85)';
      ctx.lineWidth = 5;
      ctx.lineJoin = 'round';
      ctx.strokeText(label, tx, ty);
      ctx.fillStyle = color;
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(label, tx, ty);
    }
    ctx.restore();
  }

  function drawHandle(x, y, color) {
    ctx.save();
    ctx.beginPath();
    ctx.arc(x, y, 7, 0, Math.PI * 2);
    ctx.fillStyle = 'rgba(255,255,255,0.95)';
    ctx.fill();
    ctx.strokeStyle = color;
    ctx.lineWidth = 2.2;
    ctx.stroke();
    ctx.restore();
  }

  function drawCircularArrow(cx, cy, r, clockwise, color) {
    const startA = clockwise ? -Math.PI*0.6 : -Math.PI*0.4;
    const endA = clockwise ? Math.PI*0.2 : -Math.PI*1.2 + Math.PI;
    ctx.save();
    ctx.strokeStyle = color;
    ctx.lineWidth = 1.5;
    ctx.globalAlpha = 0.5;
    ctx.beginPath();
    ctx.arc(cx, cy, r, startA, endA, !clockwise);
    ctx.stroke();
    const ea = clockwise ? endA : startA;
    const ta = clockwise ? ea + Math.PI/2 : ea - Math.PI/2;
    const ax = cx + r * Math.cos(ea), ay = cy + r * Math.sin(ea);
    ctx.beginPath();
    ctx.moveTo(ax, ay);
    ctx.lineTo(ax - 8 * Math.cos(ta - 0.4), ay - 8 * Math.sin(ta - 0.4));
    ctx.lineTo(ax - 8 * Math.cos(ta + 0.4), ay - 8 * Math.sin(ta + 0.4));
    ctx.closePath();
    ctx.fillStyle = color;
    ctx.globalAlpha = 0.5;
    ctx.fill();
    ctx.restore();
  }

  function updateInfo() {
    const deg = ((angle * 180 / Math.PI) % 360 + 360) % 360;
    document.getElementById('rod-omega').textContent = omega.toFixed(4);
    const ae = alpha.toExponential(2);
    document.getElementById('rod-alpha').textContent = ae;
    document.getElementById('rod-theta').textContent = deg.toFixed(1);
    document.getElementById('rod-len').textContent = Math.round(rodHL * 2);
  }

  function draw() {
    ctx.clearRect(0, 0, W, H);

    const isDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    const gridColor = isDark ? 'rgba(255,255,255,0.04)' : 'rgba(0,0,0,0.04)';
    const axisColor = isDark ? 'rgba(255,255,255,0.08)' : 'rgba(0,0,0,0.08)';

    ctx.strokeStyle = gridColor;
    ctx.lineWidth = 1;
    for (let x = 0; x <= W; x += 40) { ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,H); ctx.stroke(); }
    for (let y = 0; y <= H; y += 40) { ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(W,y); ctx.stroke(); }
    ctx.strokeStyle = axisColor;
    ctx.lineWidth = 1.5;
    ctx.beginPath(); ctx.moveTo(CX,0); ctx.lineTo(CX,H); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(0,CY); ctx.lineTo(W,CY); ctx.stroke();

    if (isMoving) {
      omega += alpha;
      angle += omega;
    }

    const tang = getTang();
    const e1 = getEnd1();
    const e2 = getEnd2();
    const as = getAStart();
    const vl = vArrowLen();
    const al = aArrowLen();

    const isClockwise = omega >= 0;
    drawCircularArrow(CX, CY, 24, isClockwise, isDark ? '#60a0c0' : '#5588aa');

    drawRod3D();
    drawPivot();

    drawArrow(e1.x, e1.y, vl * tang.x, vl * tang.y, '#e53e3e', 'v');
    drawArrow(as.x, as.y, al * tang.x, al * tang.y, '#dd6b20', 'a');

    if (!isMoving) {
      const vTip = { x: e1.x + vl * tang.x, y: e1.y + vl * tang.y };
      const aTip = { x: as.x + al * tang.x, y: as.y + al * tang.y };
      drawHandle(vTip.x, vTip.y, '#e53e3e');
      drawHandle(aTip.x, aTip.y, '#dd6b20');
      drawHandle(e1.x, e1.y, '#4a9eff');
      drawHandle(e2.x, e2.y, '#4a9eff');
    }

    if (popup && popup.timer > 0) {
      popup.timer--;
      const fade = Math.min(1, popup.timer / 30);
      const px = CX + popup.dist * cos();
      const py = CY + popup.dist * sin();
      const cv = omega * popup.dist;
      const cvAbs = Math.abs(cv);
      const arrowScale = Math.min(80, cvAbs * 2000 + 20);

      ctx.save();
      ctx.globalAlpha = fade;
      ctx.setLineDash([5, 4]);
      ctx.strokeStyle = '#8354ce';
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.arc(px, py, 6, 0, Math.PI * 2);
      ctx.stroke();
      ctx.setLineDash([]);

      const dir = cv >= 0 ? 1 : -1;
      const avx = dir * arrowScale * tang.x;
      const avy = dir * arrowScale * tang.y;
      const aa = Math.atan2(avy, avx);
      ctx.strokeStyle = '#8354ce';
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.moveTo(px, py);
      ctx.lineTo(px + avx, py + avy);
      ctx.stroke();
      ctx.beginPath();
      ctx.moveTo(px + avx, py + avy);
      ctx.lineTo(px + avx - 10*Math.cos(aa-Math.PI/7), py + avy - 10*Math.sin(aa-Math.PI/7));
      ctx.lineTo(px + avx - 10*Math.cos(aa+Math.PI/7), py + avy - 10*Math.sin(aa+Math.PI/7));
      ctx.closePath();
      ctx.fillStyle = '#8354ce';
      ctx.fill();

      const distLabel = Math.abs(popup.dist).toFixed(0);
      const txt1 = `r = ${distLabel} px`;
      const txt2 = `v\u209c = \u03c9\u00b7r = ${cv.toFixed(3)} px/f`;

      ctx.font = 'bold 12px sans-serif';
      const tw = Math.max(ctx.measureText(txt1).width, ctx.measureText(txt2).width);
      let bx = px + 14, by = py - 36;
      if (bx + tw + 16 > W) bx = px - tw - 30;
      if (by < 10) by = py + 14;

      ctx.fillStyle = 'rgba(131,84,206,0.92)';
      ctx.beginPath();
      ctx.roundRect ? ctx.roundRect(bx - 6, by - 16, tw + 20, 40, 5) :
        (ctx.rect(bx - 6, by - 16, tw + 20, 40));
      ctx.fill();

      ctx.fillStyle = 'white';
      ctx.textAlign = 'left';
      ctx.textBaseline = 'middle';
      ctx.fillText(txt1, bx + 4, by - 4);
      ctx.fillText(txt2, bx + 4, by + 12);

      ctx.restore();
      if (popup.timer <= 0) popup = null;
    }

    updateInfo();
    requestAnimationFrame(draw);
  }

  canvas.addEventListener('mousedown', (e) => {
    const m = getMousePos(e);

    if (isMoving) {
      const along = hitRod(m);
      if (along !== null) {
        popup = { dist: along, timer: 150 };
      }
      return;
    }

    const tang = getTang();
    const e1 = getEnd1(), e2 = getEnd2(), as = getAStart();
    const vl = vArrowLen(), al = aArrowLen();
    const vTip = { x: e1.x + vl * tang.x, y: e1.y + vl * tang.y };
    const aTip = { x: as.x + al * tang.x, y: as.y + al * tang.y };

    if (dist2(m, vTip) < 16) dragging = 'v';
    else if (dist2(m, aTip) < 16) dragging = 'a';
    else if (dist2(m, e1) < 16) dragging = 'end1';
    else if (dist2(m, e2) < 16) dragging = 'end2';
  });

  window.addEventListener('mousemove', (e) => {
    if (!dragging) return;
    const m = getMousePos(e);
    const tang = getTang();
    const e1 = getEnd1(), as = getAStart();

    if (dragging === 'v') {
      const dx = m.x - e1.x, dy = m.y - e1.y;
      const newLen = dotV({ x: dx, y: dy }, tang);
      omega = newLen / (rodHL * V_DISP);
    } else if (dragging === 'a') {
      const dx = m.x - as.x, dy = m.y - as.y;
      const newLen = dotV({ x: dx, y: dy }, tang);
      alpha = newLen / (rodHL * A_DISP);
    } else if (dragging === 'end1' || dragging === 'end2') {
      const d = dist2(m, { x: CX, y: CY });
      if (d > 40 && d < 200) rodHL = d;
    }
  });

  window.addEventListener('mouseup', () => { dragging = null; });

  canvas.addEventListener('mousemove', (e) => {
    if (dragging || isMoving) return;
    const m = getMousePos(e);
    const tang = getTang();
    const e1 = getEnd1(), e2 = getEnd2(), as = getAStart();
    const vl = vArrowLen(), al = aArrowLen();
    const vTip = { x: e1.x + vl * tang.x, y: e1.y + vl * tang.y };
    const aTip = { x: as.x + al * tang.x, y: as.y + al * tang.y };

    const near = dist2(m, vTip) < 16 || dist2(m, aTip) < 16 || dist2(m, e1) < 16 || dist2(m, e2) < 16;
    canvas.style.cursor = near ? 'grab' : 'crosshair';
  });

  draw();
})();
</script>
