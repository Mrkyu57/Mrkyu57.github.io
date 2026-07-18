---
layout: post
title: "물리학"
koreantitle: 치올콥스키 로켓 방정식
englishtitle: Tsiolkovsky Rocket Equation
info: 중력이 없는 우주 공간에서의 로켓 가속
color: "#8354ce"
permalink: /physics_classicaldynamics_rocket/
---

<!-- 모델링 공간 -->
<div id="rocket-sim-container" style="
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
    <!-- 상단 컨트롤 바 -->
    <div style="
        padding: 15px; 
        background: #fcfcfc; 
        border-bottom: 1px solid #f0f0f0; 
        display: flex; 
        justify-content: center; 
        align-items: center;
        gap: 10px;
    ">
        <h3 style="margin: 0 15px 0 0; color: #333; font-family: 'Times New Roman', serif;">Rocket Simulation</h3>
        <button id="start" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center;">
            <span id="start-icon" style="font-size: 14px;">&#9654;</span>
        </button>       
        <button id="reset" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center;">
            <span style="font-size: 18px;">&#8635;</span>
        </button>
        <div style="width: 1px; height: 24px; background: #ddd;"></div>
        <button id="vec-toggle" style="padding: 0 12px; height: 36px; cursor: pointer; background: #8354ce; color: white; border: none; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">Vectors: ON</button>
        <div id="massDisplay" style="margin-left: 10px; font-family: monospace; font-weight: bold; color: #334155; min-width: 120px;">
            Mass: 100.0 kg
        </div>
    </div>
    <!-- 캔버스 -->
    <canvas id="motionCanvas" width="900" height="400" style="display: block; background: #0f172a; cursor: crosshair;"></canvas>
</div>

<!-- 수식 설명 영역 -->
<div style="text-align: center; margin-top: 20px; font-family: 'Times New Roman', serif;">
    <div style="font-size: 24px; font-weight: bold; margin-bottom: 10px;">
        $$ \Delta v = v_e \ln \left( \frac{m_0}{m_f} \right) $$
    </div>
    <p style="color: #666; font-size: 0.95rem; line-height: 1.6;">
        <b>작동 방법:</b> 정지 상태(Reset)에서 <b>로켓</b>을 드래그하여 위치를 옮기고,<br>
        <b>파란색 벡터($v_e$)</b> 끝을 드래그하여 분사 방향과 속도를 조절하세요.
    </p>
</div>

<script>
(function() {
    const canvas = document.getElementById('motionCanvas');
    const ctx = canvas.getContext('2d');
    const startBtn = document.getElementById('start');
    const resetBtn = document.getElementById('reset');
    const vecBtn = document.getElementById('vec-toggle');
    const massLabel = document.getElementById('massDisplay');

    // --- 물리 상수 및 상태 변수 ---
    const M0 = 100.0;     // 초기 질량
    const MF = 20.0;      // 최종 질량 (연료 소진 후)
    const DM = 0.25;      // 프레임당 연료 소모량
    const VE_SCALE = 40;  // 벡터 시각화 배율

    let pos = { x: 200, y: 200 };
    let vel = { x: 0, y: 0 };
    let ve = { x: -2.5, y: 0.5 }; // 로켓 기준 배기가스 속도
    let currentMass = M0;
    
    let isMoving = false;
    let showVectors = true;
    let isDraggingObj = false;
    let isDraggingVe = false;

    // --- 이벤트 리스너 ---
    startBtn.onclick = () => {
        isMoving = !isMoving;
        document.getElementById('start-icon').innerHTML = isMoving ? '&#10074;&#10074;' : '&#9654;';
        startBtn.style.background = isMoving ? "#ffc107" : "#475569";
    };

    resetBtn.onclick = () => {
        isMoving = false;
        currentMass = M0;
        vel = { x: 0, y: 0 };
        document.getElementById('start-icon').innerHTML = '&#9654;';
        startBtn.style.background = "#475569";
        massLabel.innerText = `Mass: ${M0.toFixed(1)} kg`;
    };

    vecBtn.onclick = () => {
        showVectors = !showVectors;
        vecBtn.innerText = showVectors ? "Vectors: ON" : "Vectors: OFF";
        vecBtn.style.background = showVectors ? "#8354ce" : "#64748b";
    };

    // --- 그리기 로직 ---
    function draw() {
        // 배경 청소
        ctx.fillStyle = "#ffffff";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        if (isMoving) {
            if (currentMass > MF) {
                // 가속도 계산: a = (v_e * dm/dt) / m
                let acc = {
                    x: (-ve.x * DM) / currentMass,
                    y: (-ve.y * DM) / currentMass
                };
                vel.x += acc.x;
                vel.y += acc.y;
                currentMass -= DM;
            }
            pos.x += vel.x;
            pos.y += vel.y;
            massLabel.innerText = `Mass: ${Math.max(currentMass, MF).toFixed(1)} kg`;
        }

        // 벡터 표시
        if (showVectors) {
            drawArrow(pos.x, pos.y, vel.x * 20, vel.y * 20, "#ef4444", "v"); // 속도
            if (!isMoving || currentMass > MF) {
                drawArrow(pos.x, pos.y, ve.x * VE_SCALE, ve.y * VE_SCALE, "#3b82f6", "ve"); // 배기속도
            }
        }

        // 로켓 본체 그리기
        ctx.save();
        ctx.translate(pos.x, pos.y);
        
        // 이동 방향 또는 추력 방향으로 로켓 회전
        let angle = Math.atan2(vel.y || -ve.y, vel.x || -ve.x);
        ctx.rotate(angle);
        
        // 1. 엔진 노즐
        ctx.fillStyle = "#ffffff";
        ctx.beginPath();
        ctx.moveTo(-15*1.2, 5*1.2);
        ctx.lineTo(-22*1.5, 7*1.5);
        ctx.lineTo(-22*1.5, -7*1.5);
        ctx.lineTo(-15*1.2, -5*1.2);
        ctx.fill();
        ctx.lineWidth = 2; 
        ctx.strokeStyle = "#000000"; 
        ctx.stroke();            
        ctx.closePath();

        

        // 3. 유선형 로켓 몸통
        ctx.fillStyle = "#f8fafc";
        ctx.beginPath();
        ctx.moveTo(22*1.5, 0); // 코끝
        ctx.quadraticCurveTo(15*1.5, 9*1.5, -15*1.5, 9*1.5);
        ctx.lineTo(-15*1.5, -9*1.5);
        ctx.quadraticCurveTo(15*1.5, -9*1.5, 22*1.5, 0);
        ctx.fill();
        ctx.lineWidth = 1.5;
        ctx.strokeStyle = "#94a3b8";
        ctx.stroke();
        ctx.lineWidth = 2; 
        ctx.strokeStyle = "#000000"; 
        ctx.stroke();            
        ctx.closePath();

        // 4. 조종석 창문
        
        
        // 5. 중앙 연료 게이지
        let fuelPercent = (currentMass - MF) / (M0 - MF);
        ctx.fillStyle = "#cbd5e1"; // 게이지 배경색 (회색)
        ctx.fillRect(-8, -2.5, 12, 5);
        ctx.fillStyle = fuelPercent > 0.25 ? "#22c55e" : "#ef4444"; // 연료 잔량색 (녹색/빨간색)
        ctx.fillRect(-8, -2.5, 12 * Math.max(0, fuelPercent), 5);
        
        ctx.restore();

        requestAnimationFrame(draw);
    }

    function drawArrow(x, y, vx, vy, color, label) {
        if (Math.abs(vx) < 1 && Math.abs(vy) < 1) return;
        const head = 10;
        const angle = Math.atan2(vy, vx);
        ctx.strokeStyle = color;
        ctx.fillStyle = color;
        ctx.lineWidth = 2;

        ctx.beginPath();
        ctx.moveTo(x, y);
        ctx.lineTo(x + vx, y + vy);
        ctx.stroke();

        ctx.beginPath();
        ctx.moveTo(x + vx, y + vy);
        ctx.lineTo(x + vx - head * Math.cos(angle - Math.PI/6), y + vy - head * Math.sin(angle - Math.PI/6));
        ctx.lineTo(x + vx - head * Math.cos(angle + Math.PI/6), y + vy - head * Math.sin(angle + Math.PI/6));
        ctx.fill();

        ctx.font = "bold 14px serif";
        ctx.fillText(label, x + vx + 5, y + vy + 5);
    }

    // --- 마우스 인터랙션 ---
    function getMousePos(e) {
        const rect = canvas.getBoundingClientRect();
        return { x: e.clientX - rect.left, y: e.clientY - rect.top };
    }

    canvas.onmousedown = (e) => {
        if (isMoving) return;
        const m = getMousePos(e);
        const distToRocket = Math.hypot(m.x - pos.x, m.y - pos.y);
        const distToVe = Math.hypot(m.x - (pos.x + ve.x * VE_SCALE), m.y - (pos.y + ve.y * VE_SCALE));

        if (distToVe < 20) isDraggingVe = true;
        else if (distToRocket < 30) isDraggingObj = true;
    };

    window.onmousemove = (e) => {
        if (!isDraggingObj && !isDraggingVe) return;
        const m = getMousePos(e);
        if (isDraggingObj) {
            pos.x = m.x; pos.y = m.y;
        } else if (isDraggingVe) {
            ve.x = (m.x - pos.x) / VE_SCALE;
            ve.y = (m.y - pos.y) / VE_SCALE;
        }
    };

    window.onmouseup = () => { isDraggingObj = isDraggingVe = false; };

    draw();
})();
</script>