---
layout: post
title: "물리학"
koreantitle: 수소 원자의 보어 모델
englishtitle: Bohr Model of Hydrogen Atom
info: 흠
color: "#8354ce"
permalink: /physics_quantummechanics_bohr/
---

<!-- 모델링 공간 -->

<div id="bohr-container" style="
    width: 900px;
    margin: 20px auto;
    background: #ffffff;
    border-radius: 12px;
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.08);
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
    ">
        <h3 style="margin: 0 20px 0 0; color: #333; font-family: 'Times New Roman', serif;">
            Bohr Model of Hydrogen
        </h3>
        <!-- 재생 버튼 -->
        <button id="bohr-start" title="시작" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 14px;">&#9654;</span>
        </button>
        <!-- 리셋 버튼 -->
        <button id="bohr-reset" title="초기화" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 18px;">&#8635;</span>
        </button>
        <!-- 구분선 -->
        <div style="width: 1px; height: 24px; background: #ddd; margin: 0 4px;"></div>
        <!-- 기능 버튼들 -->
        <button id="bohr-orbits" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">Orbits</button>
        <button id="bohr-velocity" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">v</button>
        <button id="bohr-energy" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">E</button>
    </div>
    <!-- 캔버스 영역 -->
    <canvas id="bohrCanvas" width="900" height="500" style="
        display: block;
        background: #f9fafb;
        cursor: default;
    "></canvas>
</div>

<div style="text-align: center; font-family: 'Times New Roman', serif;">
    <h3>E<sub>n</sub> = &minus;13.6 / n² &nbsp;[eV]</h3>
    <p>전자는 n번째 궤도에서 정상파 조건을 만족하며 원운동을 한다.</p>
</div>

<script>
(function () {
    const canvas = document.getElementById('bohrCanvas');
    const ctx = canvas.getContext('2d');

    const startBtn   = document.getElementById('bohr-start');
    const resetBtn   = document.getElementById('bohr-reset');
    const orbitsBtn  = document.getElementById('bohr-orbits');
    const velBtn     = document.getElementById('bohr-velocity');
    const energyBtn  = document.getElementById('bohr-energy');

    // ── 물리 상수 (스케일) ───────────────────────────────────────
    const CENTER    = { x: 390, y: 250 };   // 핵 위치 (캔버스 중앙 약간 왼쪽)
    const BASE_R    = 32;                    // n=1 궤도 반지름 (px)
    const MAX_N     = 5;                     // 최대 주양자수
    const BASE_OMEGA = 0.018;               // n=1 각속도 (rad/frame)

    // n번째 궤도 반지름: r_n = n² * BASE_R
    const orbitRadius = n => n * n * BASE_R;
    // n번째 궤도 각속도: ω_n = BASE_OMEGA / n³
    const orbitOmega  = n => BASE_OMEGA / (n * n * n);
    // n번째 궤도 에너지: E_n = -13.6 / n²  [eV]
    const orbitEnergy = n => (-13.6 / (n * n)).toFixed(2);

    // ── 상태 변수 ─────────────────────────────────────────────
    let n          = 1;          // 현재 주양자수
    let angle      = 0;          // 전자의 현재 각도 (rad)
    let isMoving   = false;
    let isDragging = false;

    let showOrbits    = true;
    let showVelocity  = false;
    let showEnergy    = false;

    // 정상 상태에서 버튼 초기화
    orbitsBtn.style.background = '#8354ce';
    orbitsBtn.style.color = '#fff';

    // ── 버튼 이벤트 ───────────────────────────────────────────
    startBtn.addEventListener('click', () => {
        isMoving = !isMoving;
        if (isMoving) {
            startBtn.innerHTML = '<span>&#10074;&#10074;</span>';
            startBtn.style.background = '#ffc107';
            startBtn.style.color = '#000';
        } else {
            startBtn.innerHTML = '<span>&#9654;</span>';
            startBtn.style.background = '#475569';
            startBtn.style.color = '#fff';
        }
    });

    resetBtn.addEventListener('click', () => {
        isMoving = false;
        n = 1;
        angle = 0;
        startBtn.innerHTML = '<span>&#9654;</span>';
        startBtn.style.background = '#475569';
        startBtn.style.color = '#fff';
    });

    orbitsBtn.addEventListener('click', () => {
        showOrbits = !showOrbits;
        orbitsBtn.style.background = showOrbits ? '#8354ce' : '#f1f5f9';
        orbitsBtn.style.color      = showOrbits ? '#fff'    : '#334155';
    });

    velBtn.addEventListener('click', () => {
        showVelocity = !showVelocity;
        velBtn.style.background = showVelocity ? '#8354ce' : '#f1f5f9';
        velBtn.style.color      = showVelocity ? '#fff'    : '#334155';
    });

    energyBtn.addEventListener('click', () => {
        showEnergy = !showEnergy;
        energyBtn.style.background = showEnergy ? '#8354ce' : '#f1f5f9';
        energyBtn.style.color      = showEnergy ? '#fff'    : '#334155';
    });

    // ── 드래그: 전자 클릭 → 궤도 변경 ─────────────────────────
    function getMousePos(e) {
        const rect = canvas.getBoundingClientRect();
        return { x: e.clientX - rect.left, y: e.clientY - rect.top };
    }

    function electronPos() {
        const r = orbitRadius(n);
        return { x: CENTER.x + r * Math.cos(angle), y: CENTER.y + r * Math.sin(angle) };
    }

    canvas.addEventListener('mousedown', (e) => {
        const mouse = getMousePos(e);
        const ep = electronPos();
        if (Math.hypot(mouse.x - ep.x, mouse.y - ep.y) < 18) {
            isDragging = true;
        }
    });

    window.addEventListener('mousemove', (e) => {
        if (!isDragging) return;
        const mouse = getMousePos(e);
        const dx = mouse.x - CENTER.x;
        const dy = mouse.y - CENTER.y;
        const dist = Math.hypot(dx, dy);

        // 마우스 거리에 따라 가장 가까운 궤도로 스냅
        let closestN = 1;
        let minDiff = Infinity;
        for (let i = 1; i <= MAX_N; i++) {
            const diff = Math.abs(dist - orbitRadius(i));
            if (diff < minDiff) { minDiff = diff; closestN = i; }
        }
        n = closestN;
        angle = Math.atan2(dy, dx);
        canvas.style.cursor = 'grabbing';
    });

    window.addEventListener('mouseup', () => {
        isDragging = false;
        canvas.style.cursor = 'default';
    });

    canvas.addEventListener('mousemove', (e) => {
        if (isDragging) return;
        const mouse = getMousePos(e);
        const ep = electronPos();
        canvas.style.cursor = Math.hypot(mouse.x - ep.x, mouse.y - ep.y) < 18 ? 'grab' : 'default';
    });

    // ── 그리기 보조 함수 ─────────────────────────────────────

    // 화살표 그리기
    function drawArrow(x, y, vx, vy, color, label) {
        if (vx === 0 && vy === 0) return;
        ctx.save();

        const headLen = 10;
        const angle_a = Math.atan2(vy, vx);

        ctx.strokeStyle = color;
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(x, y);
        ctx.lineTo(x + vx, y + vy);
        ctx.stroke();

        ctx.beginPath();
        ctx.moveTo(x + vx, y + vy);
        ctx.lineTo(x + vx - headLen * Math.cos(angle_a - Math.PI / 7), y + vy - headLen * Math.sin(angle_a - Math.PI / 7));
        ctx.lineTo(x + vx - headLen * Math.cos(angle_a + Math.PI / 7), y + vy - headLen * Math.sin(angle_a + Math.PI / 7));
        ctx.closePath();
        ctx.fillStyle = color;
        ctx.fill();

        if (label) {
            const midX = x + vx * 0.6;
            const midY = y + vy * 0.6;
            const offset = 14;
            const tx = midX + offset * Math.cos(angle_a - Math.PI / 2);
            const ty = midY + offset * Math.sin(angle_a - Math.PI / 2);

            ctx.fillStyle = color;
            ctx.font = "bold 16px 'Times New Roman', serif";
            ctx.strokeStyle = '#f9fafb';
            ctx.lineWidth = 5;
            ctx.lineJoin = 'round';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.strokeText(label, tx, ty);
            ctx.fillText(label, tx, ty);
        }

        ctx.restore();
    }

    // 궤도 원 그리기
    function drawOrbit(ni, highlight) {
        ctx.save();
        const r = orbitRadius(ni);
        ctx.beginPath();
        ctx.arc(CENTER.x, CENTER.y, r, 0, Math.PI * 2);
        ctx.strokeStyle = highlight ? 'rgba(131,84,206,0.5)' : 'rgba(180,180,200,0.35)';
        ctx.lineWidth   = highlight ? 1.5 : 1;
        ctx.setLineDash(highlight ? [] : [4, 4]);
        ctx.stroke();
        ctx.setLineDash([]);

        // n 레이블
        ctx.fillStyle = highlight ? '#8354ce' : '#aaa';
        ctx.font = "13px 'Times New Roman', serif";
        ctx.textAlign = 'left';
        ctx.textBaseline = 'middle';
        ctx.fillText('n=' + ni, CENTER.x + r + 4, CENTER.y - 6);
        ctx.restore();
    }

    // 핵 그리기 (양성자)
    function drawNucleus() {
        ctx.save();
        // 광채 효과
        const grd = ctx.createRadialGradient(CENTER.x, CENTER.y, 2, CENTER.x, CENTER.y, 20);
        grd.addColorStop(0, 'rgba(255,100,80,0.3)');
        grd.addColorStop(1, 'rgba(255,100,80,0)');
        ctx.beginPath();
        ctx.arc(CENTER.x, CENTER.y, 20, 0, Math.PI * 2);
        ctx.fillStyle = grd;
        ctx.fill();

        // 핵 원
        ctx.beginPath();
        ctx.arc(CENTER.x, CENTER.y, 10, 0, Math.PI * 2);
        ctx.fillStyle = '#e05a40';
        ctx.fill();
        ctx.lineWidth = 2;
        ctx.strokeStyle = '#c0392b';
        ctx.stroke();

        // p+ 레이블
        ctx.fillStyle = '#fff';
        ctx.font = "bold 9px Arial";
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText('p⁺', CENTER.x, CENTER.y);
        ctx.restore();
    }

    // 전자 그리기
    function drawElectron(ex, ey) {
        ctx.save();
        // 광채
        const grd = ctx.createRadialGradient(ex, ey, 2, ex, ey, 16);
        grd.addColorStop(0, 'rgba(90,140,255,0.4)');
        grd.addColorStop(1, 'rgba(90,140,255,0)');
        ctx.beginPath();
        ctx.arc(ex, ey, 16, 0, Math.PI * 2);
        ctx.fillStyle = grd;
        ctx.fill();

        ctx.beginPath();
        ctx.arc(ex, ey, 8, 0, Math.PI * 2);
        ctx.fillStyle = '#4a8cff';
        ctx.fill();
        ctx.lineWidth = 2;
        ctx.strokeStyle = '#2266dd';
        ctx.stroke();

        ctx.fillStyle = '#fff';
        ctx.font = "bold 9px Arial";
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText('e⁻', ex, ey);
        ctx.restore();
    }

    // 정보 패널 그리기
    function drawInfoPanel() {
        ctx.save();
        const px = 660, py = 40, pw = 220, ph = showEnergy ? 130 : 80;

        ctx.fillStyle = 'rgba(255,255,255,0.92)';
        ctx.strokeStyle = '#dde';
        ctx.lineWidth = 1;
        ctx.beginPath();
        ctx.roundRect(px, py, pw, ph, 8);
        ctx.fill();
        ctx.stroke();

        ctx.fillStyle = '#333';
        ctx.font = "bold 14px 'Times New Roman', serif";
        ctx.textAlign = 'left';
        ctx.textBaseline = 'top';
        ctx.fillText('Quantum State', px + 14, py + 12);

        ctx.font = "15px 'Times New Roman', serif";
        ctx.fillStyle = '#8354ce';
        ctx.fillText('n  =  ' + n, px + 14, py + 36);

        const r_au = (n * n * 0.529).toFixed(3);
        ctx.fillStyle = '#444';
        ctx.fillText('r_n  =  ' + r_au + ' Å', px + 14, py + 58);

        if (showEnergy) {
            ctx.fillStyle = '#e05a40';
            ctx.fillText('E_n  =  ' + orbitEnergy(n) + ' eV', px + 14, py + 80);

            // 에너지 준위 간단 표시
            ctx.fillStyle = '#aaa';
            ctx.font = "11px 'Times New Roman', serif";
            ctx.fillText('Ground state E₁ = -13.6 eV', px + 14, py + 106);
        }

        ctx.restore();
    }

    // ── 광자 파티클 시스템 ───────────────────────────────────
    // 각 광자: { x, y, vx, vy, energy, life, maxLife }
    const photons = [];
    const PHOTON_SPEED  = 2.8;   // px/frame
    const PHOTON_LIFE   = 140;   // frames

    function emitPhoton(ex, ey, tangentAngle, deltaE) {
        photons.push({
            x: ex,
            y: ey,
            vx: PHOTON_SPEED * Math.cos(tangentAngle),
            vy: PHOTON_SPEED * Math.sin(tangentAngle),
            energy: Math.abs(deltaE).toFixed(2),
            life: PHOTON_LIFE,
            maxLife: PHOTON_LIFE
        });
    }

    // 진행 중인 광자 업데이트 & 그리기
    function updatePhotons() {
        for (let i = photons.length - 1; i >= 0; i--) {
            const p = photons[i];
            p.x += p.vx;
            p.y += p.vy;
            p.life--;
            if (p.life <= 0) { photons.splice(i, 1); continue; }

            const alpha = p.life / p.maxLife;
            ctx.save();

            // 물결 파장 효과: 광자 궤적에 물결 그리기
            const age     = p.maxLife - p.life;          // 경과 프레임
            const waveLen = 18;                           // 파장 (px)
            const amp     = 5;                            // 진폭
            const dir     = Math.atan2(p.vy, p.vx);
            const perpX   = -Math.sin(dir);
            const perpY   =  Math.cos(dir);

            

            // 광자 본체: 노란 점
            const grd = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, 8);
            grd.addColorStop(0, `rgba(255,230,80)`);
            grd.addColorStop(1, `rgba(255,180,0,0)`);
            ctx.beginPath();
            ctx.arc(p.x, p.y, 8, 0, Math.PI * 2);
            ctx.fillStyle = grd;
            ctx.fill();

            // 에너지 라벨 (γ, ΔE = ... eV)
            
   
            ctx.font = "bold 12px 'Times New Roman', serif";
            ctx.fillStyle = '#cc8800';
            ctx.strokeStyle = 'rgba(249,250,251)';
            ctx.lineWidth = 4;
            ctx.lineJoin = 'round';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'bottom';
            const lx = p.x + perpX * 16;
            const ly = p.y + perpY * 16;
            ctx.strokeText(`ΔE`, lx, ly);
            ctx.fillText(`ΔE`, lx, ly);
            

            ctx.restore();
        }
    }

    // ── 상태 추적 ────────────────────────────────────────────
    let lastN = n;

    // ── 메인 루프 ────────────────────────────────────────────
    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // 궤도 그리기
        if (showOrbits) {
            for (let i = 1; i <= MAX_N; i++) {
                drawOrbit(i, i === n);
            }
        } else {
            drawOrbit(n, true);
        }

        // 각도 업데이트
        if (isMoving && !isDragging) {
            angle += orbitOmega(n);
            if (angle > Math.PI * 2) angle -= Math.PI * 2;
        }

        const r  = orbitRadius(n);
        const ex = CENTER.x + r * Math.cos(angle);
        const ey = CENTER.y + r * Math.sin(angle);

        // n이 낮아지면 → 광자 방출 (접선 방향, ΔE 계산)
        if (n < lastN) {
            const deltaE = (-13.6 / (n * n)) - (-13.6 / (lastN * lastN)); // 음수 (방출)
            const tangentAngle = angle + Math.PI / 2;
            emitPhoton(ex, ey, tangentAngle, deltaE);
        }
        lastN = n;

        // 광자 파티클 업데이트 & 그리기
        updatePhotons();

        // 속도벡터: 접선 방향
        if (showVelocity) {
            const speed  = orbitOmega(n) * r * 40;
            const vAngle = angle + Math.PI / 2;
            const vx = speed * Math.cos(vAngle);
            const vy = speed * Math.sin(vAngle);
            drawArrow(ex, ey, vx, vy, '#4a8cff', 'v');
        }

        // 위치벡터 (핵→전자)
        drawArrow(CENTER.x, CENTER.y, ex - CENTER.x, ey - CENTER.y, 'rgba(131,84,206,0.6)', 'r');

        drawNucleus();
        drawElectron(ex, ey);
        drawInfoPanel();

        // 드래그 힌트 (정지 상태에서)
        if (!isMoving) {
            ctx.save();
            ctx.fillStyle = 'rgba(150,150,180,0.7)';
            ctx.font = "12px -apple-system, sans-serif";
            ctx.textAlign = 'center';
            ctx.fillText('▶ 재생하거나, 전자(파란 원)를 드래그하여 궤도를 바꿔보세요', canvas.width / 2 - 60, canvas.height - 18);
            ctx.restore();
        }

        requestAnimationFrame(draw);
    }

    draw();
})();
</script>