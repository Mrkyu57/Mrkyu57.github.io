---
layout: post
title: "물리학"
koreantitle: 1차원 등속직선운동
englishtitle: 1D uniform linear motion
info: 1차원 공간에서의 운동
color: "#8354ce"
permalink: /physics_classicaldynamics_1_1/
---

<!-- 모델링 공간 -->
<div id="physics-1d-container" style="
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
    ">
        <h3 style="margin: 5px 20px 0px 0px; color: #333; font-family: 'Times New Roman', serif; ">
            1D Uniform Linear Motion
        </h3>
        <button id="start" title="시작" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 14px;">&#9654;</span>
        </button>
        <button id="reset" title="초기화" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 18px;">&#8635;</span>
        </button>
        <div style="width: 1px; height: 24px; background: #ddd; margin: 6px 4px;"></div>
        <button id="Com" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">Axis</button>
        <button id="initial" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">I</button>
        <button id="function" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">f(t)</button>
    </div>
    <!-- 캔버스 영역: 1차원 운동에 맞춰 높이를 줄임 (200px) -->
    <canvas id="motionCanvas" width="900" height="600" style="
        display: block;
        background: #f9fafb;
        cursor: crosshair;
    "></canvas>
</div>

<div style="text-align: center; font: bold 18px 'Times New Roman', serif; margin-top: 20px;">
  <h3>$v = \text{const.}$</h3>
  <p>시간 $t$일 때 물체의 위치 $x(t)$는 다음과 같습니다.</p>
  <p style="font-size: 22px; color: #8354ce;">$$x(t) = x_0 + v t$$</p>
</div>

<script>
  const canvas = document.getElementById('motionCanvas');
  const ctx = canvas.getContext('2d');

  const start = document.getElementById('start');
  const reset = document.getElementById('reset');
  const comBtn = document.getElementById('Com');
  const iniBtn = document.getElementById('initial');
  const funBtn = document.getElementById('function');

  const groundY = 300; // 1차원 운동이 일어나는 y축 높이 고정
  const originX = 450; // 원점 위치

  let initial_x = 300; 
  let initial_v = 1.5; 
  let posX = 300;
  let velX = 1.5;

  let isMoving = false;
  let isReset = true;
  let isDraggingObject = false;
  let isDraggingVelocity = false;

  let showAxis = true;
  let showInitial = false;
  let vectorMode = 0; // 0: 기본, 1: 함수형 표시

  start.addEventListener('click', () => {
    isMoving = !isMoving;
    isReset = false;
    if (isMoving) {
        start.innerHTML = '<span>&#10074;&#10074;</span>';
        start.style.background = "#ffc107";
        start.style.color = "#000";
    } else {
        start.innerHTML = '<span>&#9654;</span>';
        start.style.background = "#475569";
        start.style.color = "#fff";
    }
  });

  reset.addEventListener('click', () => {
    isMoving = false;
    isReset = true;
    posX = initial_x;
    velX = initial_v;
    start.innerHTML = '<span>&#9654;</span>';
    start.style.background = "#475569";
    start.style.color = "#fff";
  });

  comBtn.addEventListener('click', () => {
    showAxis = !showAxis;
    comBtn.style.background = showAxis ? "#8354ce" : "#f1f5f9";
    comBtn.style.color = showAxis ? "#fff" : "#334155";
  });

  iniBtn.addEventListener('click', () => {
    showInitial = !showInitial;
    iniBtn.style.background = showInitial ? "#8354ce" : "#f1f5f9";
    iniBtn.style.color = showInitial ? "#fff" : "#334155";
  });

  funBtn.addEventListener('click', () => {
    vectorMode = (vectorMode === 0) ? 1 : 0;
    funBtn.style.background = vectorMode === 1 ? "#3b1e68" : "#f1f5f9";
    funBtn.style.color = vectorMode === 1 ? "#fff" : "#334155";
  });

  function drawAxis() {
    ctx.strokeStyle = "#ccc";
    ctx.lineWidth = 1;
    ctx.beginPath();
    ctx.moveTo(50, groundY);
    ctx.lineTo(850, groundY);
    ctx.stroke();

  }

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (showAxis) drawAxis();

    if (isMoving) {
      posX += velX;
    }

    // 초기 상태 표시 (I 버튼)
    if (showInitial && !isReset) {
      ctx.fillStyle = "rgba(211, 211, 211, 0.5)";
      ctx.beginPath();
      ctx.arc(initial_x, groundY, 10, 0, Math.PI * 2);
      ctx.fill();
      drawArrow(initial_x, groundY - 25, initial_v * 40, 0, "#ff9696", "v₀");
    }

    const dispX = posX - originX;
    let pColor = vectorMode === 1 ? "#8354ce" : "#666";
    let vColor = vectorMode === 1 ? "#666" : "red";
    let pLabel = vectorMode === 1 ? "x(t)" : "x";
    let vLabel = vectorMode === 1 ? "v" : "v";

    // 위치(변위) 벡터
    drawArrow(originX, groundY, dispX, 0, pColor, pLabel);
    // 속도 벡터
    drawArrow(posX, groundY, velX * 50, 0, vColor, vLabel);

    // 원점 표시
    ctx.fillStyle = "#333";
    ctx.beginPath();
    ctx.arc(originX, groundY, 4, 0, Math.PI * 2);
    ctx.fill();

    // 물체(질점) 그리기
    ctx.fillStyle = "#475569";
    ctx.beginPath();
    ctx.arc(posX, groundY, 12, 0, Math.PI * 2);
    ctx.fill();
    ctx.strokeStyle = "#000";
    ctx.lineWidth = 2;
    ctx.stroke();

    requestAnimationFrame(draw);
  }

  function drawArrow(x, y, vx, vy, color, label) {
    if (Math.abs(vx) < 1 && Math.abs(vy) < 1) return;
    
    ctx.save();
    const headLength = 10;
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
    ctx.lineTo(x + vx - headLength * Math.cos(angle - Math.PI/7), y + vy - headLength * Math.sin(angle - Math.PI/7));
    ctx.lineTo(x + vx - headLength * Math.cos(angle + Math.PI/7), y + vy - headLength * Math.sin(angle + Math.PI/7));
    ctx.fill();

    ctx.font = "bold 16px 'Times New Roman', serif";
    ctx.textAlign = "center";
    ctx.fillText(label, x + vx / 2, y - 10);
    ctx.restore();
  }

  // 상호작용 로직
  function getMousePos(e) {
    const rect = canvas.getBoundingClientRect();
    return { x: e.clientX - rect.left, y: e.clientY - rect.top };
  }

  canvas.addEventListener('mousedown', (e) => {
    if (!isReset) return;
    const mouse = getMousePos(e);
    const distToObject = Math.abs(mouse.x - posX);
    const vEnd = posX + velX * 50;
    const distToVel = Math.abs(mouse.x - vEnd);

    if (distToObject < 20) isDraggingObject = true;
    else if (distToVel < 20) isDraggingVelocity = true;
  });

  window.addEventListener('mousemove', (e) => {
    if (!isDraggingObject && !isDraggingVelocity) return;
    const mouse = getMousePos(e);
    
    if (isDraggingObject) {
      posX = Math.max(50, Math.min(850, mouse.x));
      initial_x = posX;
    } else if (isDraggingVelocity) {
      velX = (mouse.x - posX) / 50;
      initial_v = velX;
    }
  });

  window.addEventListener('mouseup', () => {
    isDraggingObject = false;
    isDraggingVelocity = false;
  });

  draw();
</script>