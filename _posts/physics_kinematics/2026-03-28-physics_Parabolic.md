---
layout: post
title: "물리학"
koreantitle: 포물선운동
englishtitle: projectile motion
info: 포물선 운동 시뮬레이션
color: "#8354ce"
permalink: /physics_classicaldynamics_1_2/
---

<!-- 모델링 공간 -->

<div id="proj-container" style="
    width: 900px;
    max-width: 100%;
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
        <h3 style="margin: 0 20px 0 0; color: #333; font-family: 'Times New Roman', serif;">
            projectile motion
        </h3>
        <!-- 재생 버튼 -->
        <button id="projStart" title="시작" style="width:45px;height:36px;cursor:pointer;background:#475569;color:white;border:none;border-radius:6px;display:flex;align-items:center;justify-content:center;font-size:14px;transition:0.2s;">
            &#9654;
        </button>
        <!-- 리셋 버튼 -->
        <button id="projReset" title="초기화" style="width:45px;height:36px;cursor:pointer;background:#475569;color:white;border:none;border-radius:6px;display:flex;align-items:center;justify-content:center;font-size:18px;transition:0.2s;">
            &#8635;
        </button>
        <!-- 구분선 -->
        <div style="width:1px;height:24px;background:#ddd;margin:0 4px;"></div>
        <!-- 기능 버튼 -->
        <button id="projCom" style="padding:0 16px;height:36px;cursor:pointer;background:#f1f5f9;color:#334155;border:1px solid #e2e8f0;border-radius:6px;font-weight:600;font-size:0.85rem;">Com</button>
        <button id="projInit" style="padding:0 16px;height:36px;cursor:pointer;background:#f1f5f9;color:#334155;border:1px solid #e2e8f0;border-radius:6px;font-weight:600;font-size:0.85rem;">I</button>
        <button id="projPath" style="padding:0 16px;height:36px;cursor:pointer;background:#f1f5f9;color:#334155;border:1px solid #e2e8f0;border-radius:6px;font-weight:600;font-size:0.85rem;">f</button>
    </div>
    <!-- 캔버스 -->
    <canvas id="projCanvas" width="900" height="450" style="
        display: block;
        width: 100%;
        height: auto;
        background: #f9fafb;
        cursor: default;
        touch-action: none;
    "></canvas>
</div>

<div style="text-align:center;">
    <h3 style="font-family:'Times New Roman',serif;">
        x = x₀ + v₀cosθ·t &nbsp;&nbsp; y = y₀ + v₀sinθ·t − ½gt²
    </h3>
    <p>발사각 θ와 초기속력 v₀를 드래그로 조정하세요. 호를 드래그하면 각도만 바뀝니다.</p>
</div>

<script>
(function () {
    const canvas = document.getElementById('projCanvas');
    const ctx    = canvas.getContext('2d');

    /* ── 상수 ── */
    const G       = 0.12;   // 중력 가속도 (px/frame²)
    const SCALE   = 22;     // 속도 → 화살표 길이 배율
    const ARC_R   = 55;     // 각도 호 반지름
    const HANDLE_R = 8;     // 호 위의 핸들 원 반지름

    /* ── 초기 조건 (사용자 조정 가능) ── */
    let launch = { x: 180, y: 0 };   // y는 draw() 첫 호출 시 GROUND_Y로 설정
    let speed    = 8;                 // px/frame
    let angleDeg = -58;              // 음수 = 위쪽

    /* ── 시뮬레이션 상태 ── */
    let t       = 0;
    let pos     = { x: 0, y: 0 };
    let vel     = { x: 0, y: 0 };
    let trail   = [];
    let isMoving = false;
    let isReset  = true;
    let landed   = false;
    let GROUND_Y = 0;   // 첫 draw에서 계산

    /* ── 드래그 상태 ── */
    let drag = null; // null | 'object' | 'velTip' | 'arc'

    /* ── 표시 옵션 ── */
    let showComp = false;
    let showInit = false;
    let showPath = false;

    /* ── 파생 값 계산 ── */
    const rad    = () => angleDeg * Math.PI / 180;
    const vx0    = () => speed * Math.cos(rad());
    const vy0    = () => speed * Math.sin(rad());
    const velTip = () => ({ x: launch.x + vx0() * SCALE, y: launch.y + vy0() * SCALE });

    /* ── 버튼 참조 ── */
    const btnStart = document.getElementById('projStart');
    const btnReset = document.getElementById('projReset');
    const btnCom   = document.getElementById('projCom');
    const btnInit  = document.getElementById('projInit');
    const btnPath  = document.getElementById('projPath');

    /* ── 버튼 이벤트 ── */
    btnStart.addEventListener('click', () => {
        if (landed) return;
        isMoving = !isMoving;
        isReset  = false;
        if (isMoving) {
            btnStart.innerHTML      = '&#10074;&#10074;';
            btnStart.style.background = '#ffc107';
            btnStart.style.color      = '#000';
        } else {
            btnStart.innerHTML      = '&#9654;';
            btnStart.style.background = '#475569';
            btnStart.style.color      = '#fff';
        }
    });

    btnReset.addEventListener('click', () => {
        isMoving = false;
        isReset  = true;
        landed   = false;
        t        = 0;
        trail    = [];
        pos      = { ...launch };
        vel      = { x: vx0(), y: vy0() };
        btnStart.innerHTML      = '&#9654;';
        btnStart.style.background = '#475569';
        btnStart.style.color      = '#fff';
    });

    function toggleBtn(btn, flag) {
        btn.style.background = flag ? '#8354ce' : '#f1f5f9';
        btn.style.color      = flag ? '#fff'    : '#334155';
    }

    btnCom.addEventListener('click',  () => { showComp = !showComp; toggleBtn(btnCom,  showComp); });
    btnInit.addEventListener('click', () => { showInit = !showInit; toggleBtn(btnInit, showInit); });
    btnPath.addEventListener('click', () => { showPath = !showPath; toggleBtn(btnPath, showPath); });

    /* ─────────────────── 그리기 함수 ─────────────────── */

    function drawArrow(x, y, dx, dy, color, label, italic) {
        if (Math.abs(dx) < 0.1 && Math.abs(dy) < 0.1) return;
        const ang = Math.atan2(dy, dx);
        const hl  = 10;

        ctx.save();
        ctx.strokeStyle = color;
        ctx.fillStyle   = color;
        ctx.lineWidth   = 2.2;

        ctx.beginPath();
        ctx.moveTo(x, y);
        ctx.lineTo(x + dx, y + dy);
        ctx.stroke();

        ctx.beginPath();
        ctx.moveTo(x + dx, y + dy);
        ctx.lineTo(x + dx - hl * Math.cos(ang - Math.PI / 7), y + dy - hl * Math.sin(ang - Math.PI / 7));
        ctx.lineTo(x + dx - hl * Math.cos(ang + Math.PI / 7), y + dy - hl * Math.sin(ang + Math.PI / 7));
        ctx.closePath();
        ctx.fill();

        if (label) {
            const mx = x + dx / 2, my = y + dy / 2;
            const off = 14;
            const tx = mx + off * Math.cos(ang - Math.PI / 2);
            const ty = my + off * Math.sin(ang - Math.PI / 2);
            const fStyle = italic ? 'italic ' : 'bold ';
            ctx.font      = fStyle + "16px 'Times New Roman', serif";
            ctx.textAlign    = 'center';
            ctx.textBaseline = 'middle';
            ctx.strokeStyle = 'rgba(249,250,251,0.95)';
            ctx.lineWidth   = 5;
            ctx.lineJoin    = 'round';
            ctx.strokeText(label, tx, ty);
            ctx.fillStyle = color;
            ctx.fillText(label, tx, ty);
        }
        ctx.restore();
    }

    function drawAngleArc() {
        const a0 = 0;
        const a1 = rad();
        const aMin = Math.min(a0, a1);
        const aMax = Math.max(a0, a1);

        ctx.save();

        /* 호 채우기 */
        ctx.fillStyle = 'rgba(243,156,18,0.12)';
        ctx.beginPath();
        ctx.moveTo(launch.x, launch.y);
        ctx.arc(launch.x, launch.y, ARC_R, aMin, aMax);
        ctx.closePath();
        ctx.fill();

        /* 호 선 */
        ctx.strokeStyle = '#f39c12';
        ctx.lineWidth   = 2.5;
        ctx.beginPath();
        ctx.arc(launch.x, launch.y, ARC_R, aMin, aMax);
        ctx.stroke();

        /* 핸들 (호 중간 지점에 작은 원) */
        const midAng = (a0 + a1) / 2;
        const hx = launch.x + ARC_R * Math.cos(midAng);
        const hy = launch.y + ARC_R * Math.sin(midAng);

        ctx.fillStyle   = drag === 'arc' ? '#e67e22' : '#f39c12';
        ctx.strokeStyle = '#fff';
        ctx.lineWidth   = 2;
        ctx.beginPath();
        ctx.arc(hx, hy, HANDLE_R, 0, Math.PI * 2);
        ctx.fill();
        ctx.stroke();

        /* 각도 표시 텍스트 */
        const labelR = ARC_R + 20;
        const lx = launch.x + labelR * Math.cos(midAng);
        const ly = launch.y + labelR * Math.sin(midAng);
        ctx.font      = "bold 13px 'Times New Roman', serif";
        ctx.fillStyle = '#e67e22';
        ctx.textAlign    = 'center';
        ctx.textBaseline = 'middle';
        // halo
        ctx.strokeStyle = 'rgba(249,250,251,0.9)';
        ctx.lineWidth   = 4;
        ctx.lineJoin    = 'round';
        const labelTxt = Math.round(Math.abs(angleDeg)) + '\u00B0';
        ctx.strokeText(labelTxt, lx, ly);
        ctx.fillText(labelTxt, lx, ly);

        ctx.restore();
    }

    function drawGround() {
        ctx.save();
        /* 지면 선 */
        ctx.strokeStyle = '#94a3b8';
        ctx.lineWidth   = 2;
        ctx.beginPath();
        ctx.moveTo(0, GROUND_Y);
        ctx.lineTo(canvas.width, GROUND_Y);
        ctx.stroke();
        /* 지면 해치 */
        ctx.strokeStyle = 'rgba(148,163,184,0.3)';
        ctx.lineWidth   = 1;
        const step = 18;
        for (let gx = 0; gx < canvas.width + 30; gx += step) {
            ctx.beginPath();
            ctx.moveTo(gx, GROUND_Y);
            ctx.lineTo(gx - 14, GROUND_Y + 10);
            ctx.stroke();
        }
        ctx.restore();
    }

    function drawTrail() {
        if (trail.length < 2) return;
        ctx.save();
        ctx.strokeStyle = 'rgba(131,84,206,0.55)';
        ctx.lineWidth   = 2;
        ctx.setLineDash([4, 4]);
        ctx.beginPath();
        ctx.moveTo(trail[0].x, trail[0].y);
        for (const p of trail) ctx.lineTo(p.x, p.y);
        ctx.stroke();
        ctx.setLineDash([]);
        ctx.restore();
    }

    function drawPredictedPath() {
        const vx = vx0(), vy = vy0();
        ctx.save();
        ctx.strokeStyle = 'rgba(131,84,206,0.22)';
        ctx.lineWidth   = 1.5;
        ctx.setLineDash([5, 5]);
        ctx.beginPath();
        ctx.moveTo(launch.x, launch.y);
        for (let t2 = 0; t2 <= 2000; t2 += 1) {
            const px = launch.x + vx * t2;
            const py = launch.y + vy * t2 + 0.5 * G * t2 * t2;
            if (py >= GROUND_Y) {
                /* 정확한 착지 지점 보간 */
                const prevPy = launch.y + vy * (t2-1) + 0.5 * G * (t2-1) * (t2-1);
                const prevPx = launch.x + vx * (t2-1);
                const frac   = (GROUND_Y - prevPy) / (py - prevPy);
                ctx.lineTo(prevPx + frac * (px - prevPx), GROUND_Y);
                break;
            }
            ctx.lineTo(px, py);
        }
        ctx.stroke();
        ctx.setLineDash([]);
        ctx.restore();
    }

    function drawRangeMarker() {
        const lx = launch.x, rx = pos.x;
        const midX = (lx + rx) / 2;
        const labelY = GROUND_Y + 28;
        ctx.save();
        ctx.strokeStyle = '#27ae60';
        ctx.fillStyle   = '#27ae60';
        ctx.lineWidth   = 1.5;
        /* 수평선 */
        ctx.setLineDash([4, 3]);
        ctx.beginPath();
        ctx.moveTo(lx, GROUND_Y + 14);
        ctx.lineTo(rx, GROUND_Y + 14);
        ctx.stroke();
        ctx.setLineDash([]);
        /* 끝 수직 막대 */
        [[lx], [rx]].forEach(([bx]) => {
            ctx.beginPath();
            ctx.moveTo(bx, GROUND_Y + 9);
            ctx.lineTo(bx, GROUND_Y + 19);
            ctx.stroke();
        });
        /* 텍스트 */
        ctx.font      = "bold 13px 'Times New Roman', serif";
        ctx.textAlign    = 'center';
        ctx.textBaseline = 'middle';
        ctx.strokeStyle = 'rgba(249,250,251,0.9)';
        ctx.lineWidth   = 4;
        ctx.lineJoin    = 'round';
        const txt = 'R \u2248 ' + Math.round(Math.abs(rx - lx)) + ' px';
        ctx.strokeText(txt, midX, labelY);
        ctx.fillText(txt, midX, labelY);
        ctx.restore();
    }

    function drawObject(cx, cy, alpha) {
        ctx.save();
        ctx.globalAlpha = alpha || 1;
        ctx.fillStyle   = '#64748b';
        ctx.beginPath();
        ctx.arc(cx, cy, 12, 0, Math.PI * 2);
        ctx.fill();
        /* 광택 */
        const grad = ctx.createRadialGradient(cx - 4, cy - 4, 1, cx, cy, 12);
        grad.addColorStop(0, 'rgba(255,255,255,0.45)');
        grad.addColorStop(1, 'rgba(255,255,255,0)');
        ctx.fillStyle = grad;
        ctx.beginPath();
        ctx.arc(cx, cy, 12, 0, Math.PI * 2);
        ctx.fill();
        ctx.strokeStyle = '#1e293b';
        ctx.lineWidth   = 1.8;
        ctx.beginPath();
        ctx.arc(cx, cy, 12, 0, Math.PI * 2);
        ctx.stroke();
        ctx.restore();
    }

    /* ─────────────────── 메인 draw 루프 ─────────────────── */
    function draw() {
        /* 지면 Y 초기 설정 */
        if (GROUND_Y === 0) {
            GROUND_Y     = canvas.height - 30;
            launch.y     = GROUND_Y;
            pos          = { ...launch };
        }

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        /* 물리 업데이트 */
        if (isMoving && !landed) {
            t++;
            pos.x = launch.x + vx0() * t;
            pos.y = launch.y + vy0() * t + 0.5 * G * t * t;
            vel.x = vx0();
            vel.y = vy0() + G * t;
            trail.push({ x: pos.x, y: pos.y });

            if (pos.y >= GROUND_Y) {
                pos.y = GROUND_Y;
                if (trail.length) trail[trail.length - 1] = { x: pos.x, y: GROUND_Y };
                isMoving = false;
                landed   = true;
                btnStart.innerHTML      = '&#9654;';
                btnStart.style.background = '#475569';
                btnStart.style.color      = '#fff';
            }
        }

        drawGround();

        /* 예상 궤적 */
        if (isReset || showPath) drawPredictedPath();

        drawTrail();

        /* 초기 상태 오버레이 */
        if (showInit && !isReset) {
            ctx.save();
            ctx.globalAlpha = 0.45;
            drawArrow(launch.x, launch.y, vx0() * SCALE, vy0() * SCALE, '#f87171', 'v₀');
            drawObject(launch.x, launch.y, 0.4);
            ctx.restore();
        }

        /* ── 리셋 상태: 초기 조건 편집 UI ── */
        if (isReset) {
            drawAngleArc();

            /* 속도 벡터 */
            const vdx = vx0() * SCALE, vdy = vy0() * SCALE;
            drawArrow(launch.x, launch.y, vdx, vdy, '#e74c3c', 'v₀');

            /* 성분 */
            if (showComp) {
                ctx.save();
                ctx.setLineDash([5, 5]);
                drawArrow(launch.x, launch.y, vdx, 0, '#c0392b', 'v₀ₓ');
                drawArrow(launch.x, launch.y, 0, vdy, '#c0392b', 'v₀ᵧ');
                ctx.setLineDash([]);
                ctx.restore();
            }

            /* 속도 벡터 끝 핸들 */
            const tip = velTip();
            ctx.save();
            ctx.fillStyle   = '#e74c3c';
            ctx.strokeStyle = '#fff';
            ctx.lineWidth   = 2;
            ctx.beginPath();
            ctx.arc(tip.x, tip.y, 7, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();
            ctx.restore();

            drawObject(launch.x, launch.y, 1);

        } else {
            /* 시뮬레이션 중/후 */
            const curPos = landed ? pos : pos;
            if (!landed) {
                /* 현재 속도벡터 (작게) */
                const vscale = SCALE * 0.45;
                drawArrow(curPos.x, curPos.y, vel.x * vscale, vel.y * vscale, '#3b82f6', 'v', false);
            }
            drawObject(curPos.x, curPos.y, 1);
        }

        /* 사거리 표시 */
        if (landed) drawRangeMarker();

        requestAnimationFrame(draw);
    }

    /* ─────────────────── 입력 처리 ─────────────────── */

    function getCanvasPos(e) {
        const rect = canvas.getBoundingClientRect();
        const sx   = canvas.width  / rect.width;
        const sy   = canvas.height / rect.height;
        const src  = (e.touches && e.touches.length) ? e.touches[0] : e;
        return {
            x: (src.clientX - rect.left) * sx,
            y: (src.clientY - rect.top)  * sy
        };
    }

    function arcHandlePos() {
        const midAng = (0 + rad()) / 2;
        return {
            x: launch.x + ARC_R * Math.cos(midAng),
            y: launch.y + ARC_R * Math.sin(midAng)
        };
    }

    function hitTest(p) {
        if (!isReset) return null;

        /* 속도 끝 핸들 우선 */
        const tip  = velTip();
        if (Math.hypot(p.x - tip.x, p.y - tip.y) < 18) return 'velTip';

        /* 호 핸들 */
        const h = arcHandlePos();
        if (Math.hypot(p.x - h.x, p.y - h.y) < HANDLE_R + 8) return 'arc';

        /* 호 선 근처 (핸들 없이도 선 위에서 드래그 가능) */
        const distCenter = Math.hypot(p.x - launch.x, p.y - launch.y);
        if (Math.abs(distCenter - ARC_R) < 14) return 'arc';

        /* 물체 */
        if (Math.hypot(p.x - launch.x, p.y - launch.y) < 20) return 'object';

        return null;
    }

    function handleDrag(p) {
        if (drag === 'object') {
            launch.x = Math.max(12, Math.min(canvas.width - 12, p.x));
            launch.y = Math.max(12, Math.min(GROUND_Y, p.y));
            pos      = { ...launch };
        } else if (drag === 'velTip') {
            const dx  = p.x - launch.x;
            const dy  = p.y - launch.y;
            const spd = Math.hypot(dx, dy) / SCALE;
            if (spd > 0.5) {
                speed    = Math.min(spd, 18);
                angleDeg = Math.atan2(dy, dx) * 180 / Math.PI;
            }
        } else if (drag === 'arc') {
            const dx = p.x - launch.x;
            const dy = p.y - launch.y;
            if (Math.hypot(dx, dy) > 5) {
                let ang  = Math.atan2(dy, dx) * 180 / Math.PI;
                /* -90 ~ 90 사이로 제한 */
                if (ang >  90) ang =  90;
                if (ang < -90) ang = -90;
                angleDeg = ang;
            }
        }
    }

    /* 마우스 이벤트 */
    canvas.addEventListener('mousedown', (e) => {
        const p = getCanvasPos(e);
        drag = hitTest(p);
    });
    window.addEventListener('mousemove', (e) => {
        if (!drag) return;
        handleDrag(getCanvasPos(e));
    });
    window.addEventListener('mouseup', () => { drag = null; });
    canvas.addEventListener('mousemove', (e) => {
        const p   = getCanvasPos(e);
        const hit = hitTest(p);
        canvas.style.cursor = hit ? (drag ? 'grabbing' : 'grab') : 'default';
    });

    /* 터치 이벤트 */
    canvas.addEventListener('touchstart', (e) => {
        const p = getCanvasPos(e);
        drag = hitTest(p);
        if (drag) e.preventDefault();
    }, { passive: false });

    window.addEventListener('touchmove', (e) => {
        if (!drag) return;
        e.preventDefault();
        handleDrag(getCanvasPos(e));
    }, { passive: false });

    window.addEventListener('touchend', () => { drag = null; });

    /* 시작 */
    draw();
})();
</script>