---
layout: post
title: "물리학"
koreantitle: 단조화운동
englishtitle: Simple Harmonic Motion
info: 흠
color: "#8354ce"
permalink: /physics_classicaldynamics_harmonic/
---

<!-- 모델링 공간 -->
<div id="harmonic-container" style="
  width: 900px;
  margin: 20px auto;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0,0,0,0.08);
  border: 1px solid #eee;
  overflow: hidden;
  font-family: -apple-system, sans-serif;
  box-sizing: border-box;
">

  <!-- 버튼 제어 영역 -->
  <div style="
    padding: 15px 0;
    background: #fcfcfc;
    border-bottom: 1px solid #f0f0f0;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 8px;
    flex-wrap: wrap;
  ">
    <h3 style="margin: 5px 16px 0 0; color: #333; font-family: 'Times New Roman', serif; white-space: nowrap;">
      Simple Harmonic Motion
    </h3>
    <!-- 재생 버튼 -->
    <button id="ho-start" title="시작" style="width:45px; height:36px; cursor:pointer; background:#475569; color:white; border:none; border-radius:6px; display:flex; align-items:center; justify-content:center; transition:0.2s; flex-shrink:0;">
      <span style="font-size:14px;">&#9654;</span>
    </button>
    <!-- 리셋 버튼 -->
    <button id="ho-reset" title="초기화" style="width:45px; height:36px; cursor:pointer; background:#475569; color:white; border:none; border-radius:6px; display:flex; align-items:center; justify-content:center; transition:0.2s; flex-shrink:0;">
      <span style="font-size:18px;">&#8635;</span>
    </button>
    <div style="width:1px; height:24px; background:#ddd; margin:6px 4px;"></div>
    <!-- 기능 버튼 -->
    <button id="ho-force"   style="padding:0 14px; height:36px; cursor:pointer; background:#f1f5f9; color:#334155; border:1px solid #e2e8f0; border-radius:6px; font-weight:600; font-size:0.85rem; font-family:'Times New Roman',serif;">F</button>
    <button id="ho-com"     style="padding:0 14px; height:36px; cursor:pointer; background:#f1f5f9; color:#334155; border:1px solid #e2e8f0; border-radius:6px; font-weight:600; font-size:0.85rem;">Com</button>
    <button id="ho-initial" style="padding:0 14px; height:36px; cursor:pointer; background:#f1f5f9; color:#334155; border:1px solid #e2e8f0; border-radius:6px; font-weight:600; font-size:0.85rem;">I</button>
  </div>

  <!-- 슬라이더 영역 -->
  <div style="
    padding: 10px 20px;
    background: #f8fafc;
    border-bottom: 1px solid #f0f0f0;
    display: flex;
    justify-content: center;
    gap: 30px;
    flex-wrap: wrap;
    align-items: center;
  ">
    <label style="font-size:0.85rem; color:#334155; font-family:'Times New Roman',serif; display:flex; align-items:center; gap:8px;">
      <i>m</i> = <span id="mass-val">1.0</span> kg
      <input type="range" id="mass-slider" min="0.5" max="5" step="0.1" value="1.0" style="width:100px; accent-color:#8354ce;">
    </label>
    <label style="font-size:0.85rem; color:#334155; font-family:'Times New Roman',serif; display:flex; align-items:center; gap:8px;">
      <i>k</i> = <span id="spring-val">2.0</span> N/m
      <input type="range" id="spring-slider" min="0.5" max="10" step="0.1" value="2.0" style="width:100px; accent-color:#8354ce;">
    </label>
    <span id="omega-display" style="font-size:0.85rem; color:#8354ce; font-family:'Times New Roman',serif;">
      ω = 1.414 rad/s
    </span>
  </div>

  <!-- 캔버스 -->
  <canvas id="hoCanvas" width="900" height="420" style="
    display: block;
    background: #f9fafb;
    cursor: default;
    max-width: 100%;
    touch-action: none;
  "></canvas>
</div>

<div style="text-align:center; margin-top:16px;">
  <h3 style="font-family:'Times New Roman',serif;">
    F = &minus;kx, &nbsp;&nbsp; x(t) = A cos(ωt + φ)
  </h3>
  <p>ω = √(k/m) 이며, 물체의 위치와 속도를 드래그하여 초기 조건을 설정할 수 있습니다.</p>
</div>

<script>
(function () {
  /* ──────────────────── 캔버스 / 반응형 ──────────────────── */
  const canvas    = document.getElementById('hoCanvas');
  const ctx       = canvas.getContext('2d');
  const container = document.getElementById('harmonic-container');

  const W0 = 900, H0 = 420;          // 논리(내부) 해상도

  function updateScale() {
    const cw = container.clientWidth;
    if (cw < W0) {
      canvas.style.width  = cw + 'px';
      canvas.style.height = (H0 * cw / W0) + 'px';
    } else {
      canvas.style.width  = W0 + 'px';
      canvas.style.height = H0 + 'px';
    }
  }
  updateScale();
  window.addEventListener('resize', updateScale);

  /* ──────────────────── UI 요소 ──────────────────── */
  const startBtn   = document.getElementById('ho-start');
  const resetBtn   = document.getElementById('ho-reset');
  const forceBtn   = document.getElementById('ho-force');
  const comBtn     = document.getElementById('ho-com');
  const iniBtn     = document.getElementById('ho-initial');
  const massSlider = document.getElementById('mass-slider');
  const springSlider = document.getElementById('spring-slider');
  const massVal    = document.getElementById('mass-val');
  const springVal  = document.getElementById('spring-val');
  const omegaDisp  = document.getElementById('omega-display');

  /* ──────────────────── 물리 상수 / 좌표계 ──────────────────── */
  const SCALE   = 85;          // 픽셀/미터
  const originX = 500;         // 평형점 캔버스 X
  const originY = 210;         // 평형점 캔버스 Y
  const wallX   = 80;          // 벽 X

  let mass    = 1.0;
  let springK = 2.0;

  /* ──────────────────── 상태 변수 ──────────────────── */
  let initPosX = 2.0;          // 초기 위치 (m)
  let initVelX = 0.0;          // 초기 속도 (m/s)
  let posX     = initPosX;
  let velX     = initVelX;

  let isMoving  = false;
  let isReset   = true;
  let isDragObj = false;
  let isDragVel = false;

  let showForce   = false;
  let showCom     = false;
  let showInitial = false;

  const DT = 1 / 60;

  /* ──────────────────── 좌표 변환 ──────────────────── */
  const physToCanX = x  => originX + x * SCALE;
  const canToPhysX = cx => (cx - originX) / SCALE;

  function getEventPos(e, isTouch) {
    const rect   = canvas.getBoundingClientRect();
    const scaleX = W0 / rect.width;
    const scaleY = H0 / rect.height;
    const src    = isTouch ? e.touches[0] : e;
    return {
      x: (src.clientX - rect.left) * scaleX,
      y: (src.clientY - rect.top)  * scaleY
    };
  }

  /* ──────────────────── 버튼 이벤트 ──────────────────── */
  startBtn.addEventListener('click', () => {
    isMoving = !isMoving;
    isReset  = false;
    if (isMoving) {
      startBtn.innerHTML        = '<span>&#10074;&#10074;</span>';
      startBtn.style.background = '#ffc107';
      startBtn.style.color      = '#000';
    } else {
      startBtn.innerHTML        = '<span>&#9654;</span>';
      startBtn.style.background = '#475569';
      startBtn.style.color      = '#fff';
    }
  });

  resetBtn.addEventListener('click', () => {
    isMoving = false;
    isReset  = true;
    posX     = initPosX;
    velX     = initVelX;
    startBtn.innerHTML        = '<span>&#9654;</span>';
    startBtn.style.background = '#475569';
    startBtn.style.color      = '#fff';
  });

  function toggleBtn(btn, state, onColor = '#8354ce') {
    btn.style.background = state ? onColor  : '#f1f5f9';
    btn.style.color      = state ? '#ffffff' : '#334155';
  }

  forceBtn.addEventListener('click', () => {
    showForce = !showForce;
    toggleBtn(forceBtn, showForce, '#e05252');
  });
  comBtn.addEventListener('click', () => {
    showCom = !showCom;
    toggleBtn(comBtn, showCom);
  });
  iniBtn.addEventListener('click', () => {
    showInitial = !showInitial;
    toggleBtn(iniBtn, showInitial);
  });

  massSlider.addEventListener('input', () => {
    mass = parseFloat(massSlider.value);
    massVal.textContent  = mass.toFixed(1);
    omegaDisp.textContent = 'ω = ' + Math.sqrt(springK / mass).toFixed(3) + ' rad/s';
  });
  springSlider.addEventListener('input', () => {
    springK = parseFloat(springSlider.value);
    springVal.textContent = springK.toFixed(1);
    omegaDisp.textContent = 'ω = ' + Math.sqrt(springK / mass).toFixed(3) + ' rad/s';
  });

  /* ──────────────────── 드래그 히트 테스트 ──────────────────── */
  function objCanX() { return physToCanX(posX); }

  function hitObj(p) {
    return Math.hypot(p.x - objCanX(), p.y - originY) < 28;
  }
  function hitVel(p) {
    const vx = objCanX() + velX * SCALE * 0.7;
    return Math.hypot(p.x - vx, p.y - (originY - 38)) < 16;
  }

  function startDrag(e, isTouch) {
    if (!isReset) return;
    const p = getEventPos(e, isTouch);
    if (hitObj(p))      { isDragObj = true; if (isTouch) e.preventDefault(); }
    else if (hitVel(p)) { isDragVel = true; if (isTouch) e.preventDefault(); }
  }
  function moveDrag(e, isTouch) {
    if (!isDragObj && !isDragVel) return;
    if (isTouch) e.preventDefault();
    const p = getEventPos(e, isTouch);
    if (isDragObj) {
      posX     = Math.max(-3.8, Math.min(3.8, canToPhysX(p.x)));
      initPosX = posX;
    } else {
      const dx = p.x - objCanX();
      velX     = Math.max(-6, Math.min(6, dx / (SCALE * 0.7)));
      initVelX = velX;
    }
  }
  function endDrag() { isDragObj = false; isDragVel = false; }

  canvas.addEventListener('mousedown',  e => startDrag(e, false));
  window.addEventListener('mousemove',  e => moveDrag(e, false));
  window.addEventListener('mouseup',    endDrag);

  canvas.addEventListener('touchstart', e => startDrag(e, true),  { passive: false });
  canvas.addEventListener('touchmove',  e => moveDrag(e, true),   { passive: false });
  canvas.addEventListener('touchend',   endDrag);

  canvas.addEventListener('mousemove', e => {
    if (!isReset) { canvas.style.cursor = 'default'; return; }
    const p = getEventPos(e, false);
    canvas.style.cursor = (hitObj(p) || hitVel(p)) ? 'grab' : 'default';
    if (isDragObj || isDragVel) canvas.style.cursor = 'grabbing';
  });

  /* ──────────────────── 그리기 헬퍼 ──────────────────── */
  function drawArrow(x, y, vx, vy, color, label, labelOffset) {
    if (Math.abs(vx) < 0.5 && Math.abs(vy) < 0.5) return;
    ctx.save();
    const headLen = 11;
    const angle   = Math.atan2(vy, vx);

    ctx.strokeStyle = color;
    ctx.fillStyle   = color;
    ctx.lineWidth   = 2.5;
    ctx.beginPath();
    ctx.moveTo(x, y);
    ctx.lineTo(x + vx, y + vy);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(x + vx, y + vy);
    ctx.lineTo(x + vx - headLen * Math.cos(angle - Math.PI / 7),
               y + vy - headLen * Math.sin(angle - Math.PI / 7));
    ctx.lineTo(x + vx - headLen * Math.cos(angle + Math.PI / 7),
               y + vy - headLen * Math.sin(angle + Math.PI / 7));
    ctx.closePath();
    ctx.fill();

    if (label) {
      const off   = labelOffset || { x: 0, y: -14 };
      const lx    = x + vx * 0.55 + off.x;
      const ly    = y + vy * 0.55 + off.y;

      ctx.font      = "bold 16px 'Times New Roman', serif";
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';

      ctx.strokeStyle = 'rgba(249,250,251,0.9)';
      ctx.lineWidth   = 5;
      ctx.lineJoin    = 'round';
      ctx.strokeText(label, lx, ly);

      ctx.fillStyle = color;
      ctx.fillText(label, lx, ly);
    }
    ctx.restore();
  }

  function drawSpring(x1, y, x2) {
    const leadLen  = 14;
    const tailLen  = 14;
    const coilSpan = (x2 - x1) - leadLen - tailLen;
    if (coilSpan < 2) return;

    const NUM_COILS = 9;
    const STEPS     = NUM_COILS * 64;
    const ry        = 15;   // 고정 진폭 — 압축/인장해도 코일 크기 불변
    const BG        = '#f9fafb';   // 캔버스 배경색과 동일
    const SC        = '#333';      // 스프링 색

    // 포인트 생성: y + ry*sin(θ)
    // sin > 0 → 아래쪽(앞면), sin < 0 → 위쪽(뒷면)
    const pts = [];
    for (let s = 0; s <= STEPS; s++) {
      const t     = s / STEPS;
      const theta = t * Math.PI * 2 * NUM_COILS;
      pts.push({
        x:     x1 + leadLen + t * coilSpan,
        y:     y + ry * Math.sin(theta),
        front: Math.sin(theta) >= 0
      });
    }

    ctx.save();
    ctx.lineCap  = 'round';
    ctx.lineJoin = 'round';

    // ── 연결선 ─────────────────────────────────────────────────
    ctx.strokeStyle = SC;
    ctx.lineWidth   = 2;
    ctx.beginPath(); ctx.moveTo(x1, y); ctx.lineTo(x1 + leadLen, y); ctx.stroke();

    // ── 1패스: 전체 사인 곡선 (뒷면 포함) ──────────────────────
    ctx.strokeStyle = SC;
    ctx.lineWidth   = 2;
    ctx.beginPath();
    ctx.moveTo(pts[0].x, pts[0].y);
    for (let i = 1; i < pts.length; i++) ctx.lineTo(pts[i].x, pts[i].y);
    ctx.stroke();

    // ── 2패스: 앞면(아래쪽 호)을 배경색 굵은 선으로 덮어 뒷면 지우기 ──
    ctx.strokeStyle = BG;
    ctx.lineWidth   = 5;
    let inFront = false;
    ctx.beginPath();
    for (let i = 0; i < pts.length; i++) {
      const p = pts[i];
      if (p.front) { if (!inFront) { ctx.moveTo(p.x, p.y); inFront = true; } else ctx.lineTo(p.x, p.y); }
      else { inFront = false; }
    }
    ctx.stroke();

    // ── 3패스: 앞면을 스프링 색으로 그리기 ────────────────────
    ctx.strokeStyle = SC;
    ctx.lineWidth   = 2;
    inFront = false;
    ctx.beginPath();
    for (let i = 0; i < pts.length; i++) {
      const p = pts[i];
      if (p.front) { if (!inFront) { ctx.moveTo(p.x, p.y); inFront = true; } else ctx.lineTo(p.x, p.y); }
      else { inFront = false; }
    }
    ctx.stroke();

    // ── 연결선 끝 ──────────────────────────────────────────────
    ctx.strokeStyle = SC;
    ctx.lineWidth   = 2;
    ctx.beginPath(); ctx.moveTo(x2 - tailLen, y); ctx.lineTo(x2, y); ctx.stroke();

    ctx.restore();
  }

  function drawWall(x, y) {
    ctx.save();
    const h = 90;
    ctx.fillStyle = '#9aa5b4';
    ctx.fillRect(x - 12, y - h / 2, 12, h);

    ctx.strokeStyle = '#6b7889';
    ctx.lineWidth   = 1.2;
    const hatches = 7;
    for (let i = 0; i <= hatches; i++) {
      const hy = y - h / 2 + (h / hatches) * i;
      ctx.beginPath();
      ctx.moveTo(x - 12, hy);
      ctx.lineTo(x - 22, hy + 10);
      ctx.stroke();
    }
    ctx.strokeStyle = '#5a6575';
    ctx.lineWidth   = 2.5;
    ctx.beginPath();
    ctx.moveTo(x - 12, y - h / 2);
    ctx.lineTo(x - 12, y + h / 2);
    ctx.stroke();
    ctx.restore();
  }

  function drawAxis() {
    ctx.save();
    ctx.strokeStyle = '#d0d4da';
    ctx.lineWidth   = 1;
    ctx.setLineDash([5, 5]);
    ctx.beginPath();
    ctx.moveTo(wallX, originY);
    ctx.lineTo(W0 - 20, originY);
    ctx.stroke();
    ctx.setLineDash([]);

    // 눈금
    for (let m = -4; m <= 4; m++) {
      const cx = originX + m * SCALE;
      if (cx < wallX || cx > W0 - 10) continue;
      ctx.strokeStyle = m === 0 ? '#999' : '#d0d4da';
      ctx.lineWidth   = m === 0 ? 1.5 : 1;
      ctx.beginPath();
      ctx.moveTo(cx, originY - (m === 0 ? 16 : 6));
      ctx.lineTo(cx, originY + (m === 0 ? 16 : 6));
      ctx.stroke();
      if (m !== 0) {
        ctx.fillStyle    = '#bbb';
        ctx.font         = "11px 'Times New Roman', serif";
        ctx.textAlign    = 'center';
        ctx.textBaseline = 'top';
        ctx.fillText(m + 'm', cx, originY + 8);
      }
    }
    ctx.fillStyle    = '#888';
    ctx.font         = "12px 'Times New Roman', serif";
    ctx.textAlign    = 'center';
    ctx.textBaseline = 'top';
    ctx.fillText('x=0', originX, originY + 8);
    ctx.restore();
  }

  function drawInfoPanel() {
    const force = -springK * posX;
    const omega = Math.sqrt(springK / mass);
    const KE    = 0.5 * mass * velX * velX;
    const PE    = 0.5 * springK * posX * posX;
    const E     = KE + PE;

    const lines = [
      { label: 'x',  val: posX.toFixed(3) + ' m',    color: '#666' },
      { label: 'v',  val: velX.toFixed(3) + ' m/s',  color: '#e05252' },
      showForce
        ? { label: 'F', val: force.toFixed(3) + ' N',   color: '#f97316' }
        : null,
      { label: 'KE', val: KE.toFixed(3) + ' J',      color: '#3b82f6' },
      { label: 'PE', val: PE.toFixed(3) + ' J',      color: '#10b981' },
      { label: 'E',  val: E.toFixed(3) + ' J',       color: '#8354ce' },
    ].filter(Boolean);

    ctx.save();
    const px = 16, py = 14, lh = 18;
    ctx.fillStyle   = 'rgba(255,255,255,0.82)';
    ctx.strokeStyle = '#e2e8f0';
    ctx.lineWidth   = 1;
    const bw = 160, bh = lines.length * lh + 14;
    const rx = 8;
    ctx.beginPath();
    ctx.moveTo(px + rx, py);
    ctx.lineTo(px + bw - rx, py);
    ctx.quadraticCurveTo(px + bw, py, px + bw, py + rx);
    ctx.lineTo(px + bw, py + bh - rx);
    ctx.quadraticCurveTo(px + bw, py + bh, px + bw - rx, py + bh);
    ctx.lineTo(px + rx, py + bh);
    ctx.quadraticCurveTo(px, py + bh, px, py + bh - rx);
    ctx.lineTo(px, py + rx);
    ctx.quadraticCurveTo(px, py, px + rx, py);
    ctx.closePath();
    ctx.fill();
    ctx.stroke();

    ctx.font         = "13px 'Times New Roman', serif";
    ctx.textBaseline = 'top';
    ctx.textAlign    = 'left';
    lines.forEach((ln, i) => {
      ctx.fillStyle = ln.color;
      ctx.fillText(ln.label + ' = ' + ln.val, px + 12, py + 8 + i * lh);
    });
    ctx.restore();
  }

  /* ──────────────────── 메인 드로우 루프 ──────────────────── */
  function draw() {
    ctx.clearRect(0, 0, W0, H0);

    /* 물리 업데이트 */
    if (isMoving) {
      const acc = -springK * posX / mass;
      velX += acc * DT;
      posX += velX * DT;
    }

    const cx    = physToCanX(posX);   // 물체 캔버스 X
    const cy    = originY;
    const R     = 20;                 // 물체 반지름

    drawAxis();
    drawWall(wallX, originY);
    drawSpring(wallX, originY, cx - R);

    /* 초기 조건 표시 */
    if (showInitial && !isReset) {
      const icx = physToCanX(initPosX);
      ctx.save();
      ctx.strokeStyle = '#c8c8c8';
      ctx.lineWidth   = 1.5;
      ctx.setLineDash([5, 4]);
      ctx.beginPath();
      ctx.moveTo(icx, originY - 36);
      ctx.lineTo(icx, originY + 36);
      ctx.stroke();
      ctx.setLineDash([]);
      ctx.fillStyle = '#d0d0d0';
      ctx.beginPath();
      ctx.arc(icx, originY, 11, 0, Math.PI * 2);
      ctx.fill();
      ctx.restore();
      if (Math.abs(initVelX) > 0.02) {
        drawArrow(icx, originY, initVelX * SCALE * 0.7, 0, '#ffaaaa', 'v₀');
      }
    }

    /* 변위 벡터 */
    const disp = cx - originX;
    drawArrow(originX, originY, disp, 0, '#9aa5b4', 'x', { x: 0, y: 12 });

    /* 속도 벡터 */
    const vPx = velX * SCALE * 0.7;
    drawArrow(cx, cy - R - 5, vPx, 0, '#e05252', 'v', { x: 0, y: -14 });

    /* 성분 표시 (1D 이므로 x, v의 크기 숫자 강조) */
    if (showCom) {
      ctx.save();
      ctx.strokeStyle = '#e2e8f0';
      ctx.lineWidth   = 1;
      ctx.setLineDash([4, 4]);
      ctx.beginPath(); ctx.moveTo(cx, originY - 55); ctx.lineTo(cx, originY); ctx.stroke();
      ctx.beginPath(); ctx.moveTo(originX, originY - 55); ctx.lineTo(originX, originY - 50); ctx.stroke();
      ctx.setLineDash([]);

      ctx.fillStyle    = '#9aa5b4';
      ctx.font         = "bold 13px 'Times New Roman', serif";
      ctx.textAlign    = 'center';
      ctx.textBaseline = 'bottom';
      ctx.fillText(posX.toFixed(2) + ' m', originX + disp / 2, originY - 2);

      ctx.fillStyle = '#e05252';
      ctx.fillText(velX.toFixed(2) + ' m/s', cx + vPx / 2, cy - R - 22);
      ctx.restore();
    }

    /* 힘 벡터 */
    if (showForce) {
      const force = -springK * posX;
      const fPx   = force * SCALE * 0.28;
      drawArrow(cx, cy + R + 6, fPx, 0, '#f97316', 'F', { x: 0, y: 14 });
    }

    /* 원점 점 */
    ctx.fillStyle = '#aaa';
    ctx.beginPath();
    ctx.arc(originX, originY, 3.5, 0, Math.PI * 2);
    ctx.fill();

    /* 물체 (공) */
    const grad = ctx.createRadialGradient(cx - 5, cy - 6, 2, cx, cy, R);
    grad.addColorStop(0, '#d6dce8');
    grad.addColorStop(1, '#5a6a8a');
    ctx.fillStyle = grad;
    ctx.beginPath();
    ctx.arc(cx, cy, R, 0, Math.PI * 2);
    ctx.fill();
    ctx.strokeStyle = '#3a4a6b';
    ctx.lineWidth   = 2;
    ctx.stroke();
    ctx.closePath();

    /* 속도 핸들 (리셋 상태에서만) */
    if (isReset) {
      const vEndX = cx + velX * SCALE * 0.7;
      ctx.fillStyle   = '#e05252';
      ctx.strokeStyle = '#fff';
      ctx.lineWidth   = 2;
      ctx.beginPath();
      ctx.arc(vEndX, cy - R - 5, 8, 0, Math.PI * 2);
      ctx.fill();
      ctx.stroke();
    }

    drawInfoPanel();
    requestAnimationFrame(draw);
  }

  draw();
})();
</script>