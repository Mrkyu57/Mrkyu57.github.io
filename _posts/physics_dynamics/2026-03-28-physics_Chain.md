---
layout: post
title: "물리학"
koreantitle: 체인이 연결된 공의 운동 (바닥 뚫림 수정)
englishtitle: Motion of a Chained Ball
info: 질량이 위치에 의존하는 가변 질량 시스템 분석 (지면 체인 묘사 수정)
color: "#8354ce"
permalink: /physics_chained_ball_v4/
---

<!-- 모델링 공간 -->

<div id="origami-axiom1-container" style="
    width: 900px;
    height: 600px;
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
        gap: 8px;
        align-items: center;
    ">
        <h3 style="margin: 0px 20px 0px 0px; color: #333; font-family: 'Times New Roman', serif; ">
        Chained Ball Motion
        </h3>
        <button id="start" title="시작" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 14px;">&#9654;</span>
        </button>
        <button id="reset" title="초기화" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 18px;">&#8635;</span>
        </button>
        <div style="width: 1px; height: 24px; background: #ddd; margin: 0 4px;"></div>
        <button id="toggleInfo" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">데이터 보기</button>
    </div>
    <!-- 캔버스 영역 -->
    <canvas id="motionCanvas" width="900" height="600" style="
        display: block;
        background: #f9fafb;
        cursor: crosshair;
    "></canvas>
</div>

<script>
    const canvas = document.getElementById('motionCanvas');
    const ctx = canvas.getContext('2d');
    const startBtn = document.getElementById('start');
    const resetBtn = document.getElementById('reset');
    const infoBtn = document.getElementById('toggleInfo');

    // 물리 상수 및 변수
    const g = 0.5;            // 중력 가속도
    const m = 10;             // 공의 질량
    const rho = 0.8;          // 체인의 선밀도
    const groundY = 450;      // 지면 위치
    
    // 시각적 설정
    const ballRadius = 12;    
    const visualLinkLength = 15; // 공중 체인 마디 간격
    const nodeRadius = 4;     // 체인 마디 반지름
    const totalChainLength = 250; // 시각적 묘사를 위한 전체 체인 길이 (화면 내에서 공중에 뜰 수 있도록 수정)

    let x = 0;                // 지면으로부터의 높이
    let v = 0;                // 속도
    let isMoving = false;
    let isDragging = false;
    let showInfo = false;

    // 초기 상태
    const initialX = 150;

    function resetSimulation() {
        x = initialX;
        v = 0;
        isMoving = false;
        startBtn.innerHTML = '<span>&#9654;</span>';
        startBtn.style.background = "#475569";
        startBtn.style.color = "#fff";
    }

    resetSimulation();

    // 버튼 이벤트
    startBtn.addEventListener('click', () => {
        isMoving = !isMoving;
        if (isMoving) {
            startBtn.innerHTML = '<span>&#10074;&#10074;</span>';
            startBtn.style.background = "#ffc107";
            startBtn.style.color = "#000";
        } else {
            startBtn.innerHTML = '<span>&#9654;</span>';
            startBtn.style.background = "#475569";
            startBtn.style.color = "#fff";
        }
    });

    resetBtn.addEventListener('click', resetSimulation);
    infoBtn.addEventListener('click', () => {
        showInfo = !showInfo;
        infoBtn.style.background = showInfo ? "#8354ce" : "#f1f5f9";
        infoBtn.style.color = showInfo ? "#fff" : "#334155";
    });

    function update() {
        if (isMoving && !isDragging) {
            // 공중에 떠 있을 때 체인 질량이 총길이를 초과하지 않도록 수정
            let activeX = Math.min(x, totalChainLength); 
            let mass_total = m + rho * activeX;
            let acceleration = (-m * g - rho * v * Math.abs(v)) / mass_total; 

            v += acceleration;
            x += v;

            // 지면 충돌
            if (x <= 0) {
                x = 0;
                v = 0;
                isMoving = false;
                startBtn.innerHTML = '<span>&#9654;</span>';
                startBtn.style.background = "#475569";
                startBtn.style.color = "#fff";
            }
        }
    }

    // 마디(Link) 하나를 그리는 공통 함수
    function drawNode(ctx, x, y, r) {
        ctx.beginPath();
        ctx.arc(x, y, r, 0, Math.PI * 2);
        ctx.fillStyle = "#ffffff";
        ctx.fill();
        ctx.lineWidth = 1;
        ctx.strokeStyle = "#666";
        ctx.stroke();
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        const ballX = canvas.width / 2;
        const ballBottomY = groundY - x; 
        const ballCenterY = ballBottomY - ballRadius; 

        // 1. 지면 그리기 (먼저 그려서 체인이 위에 덮이도록 함)
        ctx.strokeStyle = "#333";
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(100, groundY);
        ctx.lineTo(800, groundY);
        ctx.stroke();

        // --- 체인 시각화 ---
        ctx.strokeStyle = "#a2a2a2"; 
        ctx.lineWidth = 3;

        // 2. 공중에 떠 있는 체인 그리기
        if (x > 0) {
            // 체인의 맨 끝단 Y좌표 (체인이 온전히 공중에 떠 있으면 지면보다 높음)
            let chainBottomY = groundY;
            if (x > totalChainLength) {
                chainBottomY = ballBottomY + totalChainLength;
            }

            // 선
            ctx.beginPath();
            ctx.moveTo(ballX, ballBottomY);
            ctx.lineTo(ballX, chainBottomY);
            ctx.stroke();

            // 마디들
            let currentNodeY = ballBottomY;
            while (currentNodeY <= chainBottomY + 0.1) {
                // 바닥에 닿아있을 때 지면 마디와 겹치지 않게 조절
                if (x <= totalChainLength && currentNodeY > groundY - nodeRadius) {
                    break;
                }
                drawNode(ctx, ballX, currentNodeY, nodeRadius);
                currentNodeY += visualLinkLength;
            }
        }

        // 3. 지면에 쌓여 있는 체인 묘사 (체인이 바닥에 닿아있을 때만)
        let piledChainLength = totalChainLength - x;
        if (piledChainLength > 0) {
            const pileCenterX = ballX;
            const pileBaseY = groundY - nodeRadius; 
            
            const piledNodeSpacing = 8; 
            let numPiledNodes = Math.floor(piledChainLength / piledNodeSpacing);
            
            for (let i = 0; i < numPiledNodes; i++) {
                let angle = i * 0.5; 
                let radius = i * 0.8; 
                
                let nodeX = pileCenterX + radius * Math.cos(angle);
                let calculatedY = pileBaseY; 
                let finalNodeY = Math.min(calculatedY, pileBaseY);
                drawNode(ctx, nodeX, finalNodeY, nodeRadius);
            }
            
            // 지면과 만나는 수직 체인의 마지막 마디 고정 그리기
            drawNode(ctx, ballX, groundY - nodeRadius, nodeRadius);
        }


        // 4. 공 그리기
        ctx.beginPath();
        ctx.arc(ballX, ballCenterY, ballRadius, 0, Math.PI * 2);
        ctx.fillStyle = "#8354ce"; 
        ctx.fill();
        ctx.strokeStyle = "#000";
        ctx.lineWidth = 2;
        ctx.stroke();

        // 5. 데이터 표시
        if (showInfo) {
            ctx.fillStyle = "#333";
            ctx.font = "14px Arial";
            ctx.fillText(`현재 높이 (x): ${x.toFixed(2)}`, 50, 50);
            
            let displayMass = rho * Math.min(x, totalChainLength);
            ctx.fillText(`떠있는 체인 질량 (ρx): ${(displayMass).toFixed(2)}`, 50, 70);
            
            let displayPiled = Math.max(0, piledChainLength);
            ctx.fillText(`지면 체인 길이 묘사: ${(displayPiled).toFixed(0)}`, 50, 90);
        }

        update();
        requestAnimationFrame(draw);
    }

    // 마우스 상호작용
    let lastMouseY = 0;
    canvas.addEventListener('mousedown', (e) => {
        const rect = canvas.getBoundingClientRect();
        const mouseY = e.clientY - rect.top;
        const ballBottomY = groundY - x;
        const ballCenterY = ballBottomY - ballRadius;
        
        if (Math.hypot(canvas.width/2 - (e.clientX - rect.left), ballCenterY - mouseY) < 30) {
            isDragging = true;
            lastMouseY = mouseY;
        }
    });

    window.addEventListener('mousemove', (e) => {
        const rect = canvas.getBoundingClientRect();
        const mouseX = e.clientX - rect.left;
        const mouseY = e.clientY - rect.top;
        const ballBottomY = groundY - x;
        const ballCenterY = ballBottomY - ballRadius;

        if (Math.hypot(canvas.width/2 - mouseX, ballCenterY - mouseY) < 30) {
            canvas.style.cursor = 'grab';
        } else {
            canvas.style.cursor = 'crosshair';
        }

        if (isDragging) {
            canvas.style.cursor = 'grabbing';
            let deltaY = lastMouseY - mouseY;
            x += deltaY;
            v = deltaY * 0.8; 
            
            if (x > 400) x = 400; // 드래그 최대 높이 수정 (온전히 공중에 뜨도록 허용)
            if (x < 0) x = 0;
            lastMouseY = mouseY;
        }
    });

    window.addEventListener('mouseup', () => {
        if (isDragging) {
            isDragging = false;
            isMoving = true; 
            startBtn.innerHTML = '<span>&#10074;&#10074;</span>';
            startBtn.style.background = "#ffc107";
            startBtn.style.color = "#000";
        }
    });

    draw();
</script>