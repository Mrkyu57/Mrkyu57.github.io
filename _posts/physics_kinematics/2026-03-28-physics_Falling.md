---
layout: post
title: "물리학"
koreantitle: 자유낙하 운동 (지면 원점)
englishtitle: free fall with ground origin
info: 바닥을 원점으로 하는 자유낙하 모델링
color: "#8354ce"
permalink: /physics_classicaldynamics_1_2/
---

<!-- 모델링 공간 -->
<div id="origami-axiom1-container" style="
    width: 100%; 
    max-width: 900px; 
    margin: 20px auto; 
    background: #ffffff; 
    border-radius: 12px; 
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.08); 
    border: 1px solid #eee;
    overflow: hidden;
    font-family: -apple-system, sans-serif;
    box-sizing: border-box;
">
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
        <h3 style="margin: 0 20px 0 0; color: #333; font-family: 'Times New Roman', serif; ">
        Free Fall (Ground Origin)
        </h3>
        <button id="start" title="시작" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center;">
            <span style="font-size: 14px;">&#9654;</span>
        </button>
        <button id="reset" title="초기화" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center;">
            <span style="font-size: 18px;">&#8635;</span>
        </button>
        <div style="width: 1px; height: 24px; background: #ddd; margin: 0 4px;"></div>
        <button id="Com" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">Com</button>
        <button id="initial" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">I</button>
        <button id="function" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">f</button>
    </div>
    <canvas id="motionCanvas" width="900" height="450" style="
        display: block;
        width: 100%;
        height: auto;
        aspect-ratio: 9 / 4.5;
        background: #f9fafb;
        cursor: crosshair;
        touch-action: none;
    "></canvas>
</div>

<script>
  const canvas = document.getElementById('motionCanvas');
  const ctx = canvas.getContext('2d');

  const start = document.getElementById('start');
  const reset = document.getElementById('reset');
  const comBtn = document.getElementById('Com');
  const iniBtn = document.getElementById('initial');
  const funBtn = document.getElementById('function');

  // 물리 상수 및 설정
  const groundY = 400; // 바닥의 Y 좌표
  const origin_position = { x: 450, y: groundY }; // 바닥 중앙을 원점으로 설정
  const gravity = 0.15; 
  const vectorScale = 40; 

  let initial_position = { x: 450, y: 100 };                       
  let initial_velocity = { x: 0, y: 0 };
  let position = { x: 450, y: 100 };                               
  let velocity = { x: 0, y: 0 };                              

  let isMoving = false;                                           
  let isReset = true;
  let isDraggingObject = false;
  let isDraggingVelocity = false;
  let showInitial = false;
  let showComponents = false;                                     
  let vectorMode = 0;

  function getPos(e) {
    const rect = canvas.getBoundingClientRect();
    const clientX = e.touches ? e.touches[0].clientX : e.clientX;
    const clientY = e.touches ? e.touches[0].clientY : e.clientY;
    const scaleX = canvas.width / rect.width;
    const scaleY = canvas.height / rect.height;
    return { x: (clientX - rect.left) * scaleX, y: (clientY - rect.top) * scaleY };
  }

  start.addEventListener('click', () => {                         
    isMoving = !isMoving;                                         
    isReset = false;
    start.innerHTML = isMoving ? '<span>&#10074;&#10074;</span>' : '<span>&#9654;</span>';
    start.style.background = isMoving ? "#ffc107" : "#475569";
    start.style.color = isMoving ? "#000" : "#fff";
  });

  reset.addEventListener('click', () => {                         
    isMoving = false;
    isReset = true;
    position = { ...initial_position };
    velocity = { ...initial_velocity };
    start.innerHTML = '<span>&#9654;</span>';
    start.style.background = "#475569";
    start.style.color = "#fff";
  });

  comBtn.addEventListener('click', () => {
    showComponents = !showComponents;
    comBtn.style.background = showComponents ? "#8354ce" : "#f1f5f9";
    comBtn.style.color = showComponents ? "#fff" : "#334155";
  });

  iniBtn.addEventListener('click', () => {
    showInitial = !showInitial;
    iniBtn.style.background = showInitial ? "#8354ce" : "#f1f5f9";
    iniBtn.style.color = showInitial ? "#fff" : "#334155";
  });

  funBtn.addEventListener('click', () => {
    vectorMode = (vectorMode + 1) % 3;
    funBtn.style.background = vectorMode > 0 ? (vectorMode === 2 ? "#3b1e68" : "#8354ce") : "#f1f5f9";
    funBtn.style.color = vectorMode > 0 ? "#fff" : "#334155";
  });

  function draw() {                                                                                 
    ctx.clearRect(0, 0, canvas.width, canvas.height);                                             

    // 바닥 그리기
    ctx.strokeStyle = "#475569";
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.moveTo(50, groundY);
    ctx.lineTo(850, groundY);
    ctx.stroke();
    
    // 바닥 격자 표시 (원점 강조)
    ctx.fillStyle = "#475569";
    ctx.font = "12px Arial";
    ctx.fillText("Ground (Origin)", origin_position.x + 10, groundY + 20);

    if (isMoving) {                                                                               
      velocity.y += gravity; 
      position.x += velocity.x;                                                                 
      position.y += velocity.y;
      
      // 바닥 충돌 처리
      if (position.y > groundY - 10) {
          position.y = groundY - 10;
          velocity.y = 0;
          velocity.x = 0;
          isMoving = false;
          start.innerHTML = '<span>&#9654;</span>';
          start.style.background = "#475569";
          start.style.color = "#fff";
      }
    }

    // 초기 상태 표시 (I 버튼)
    if (showInitial && !isReset) { 
      drawMoveVector(initial_position.x, initial_position.y, initial_velocity.x * vectorScale, initial_velocity.y * vectorScale, "#ff9696", "v₀");
      drawMoveVector(origin_position.x, origin_position.y, initial_position.x - origin_position.x, initial_position.y - origin_position.y, "#d3d3d396", "r₀");
    }

    // 벡터 라벨 설정
    let posColor = vectorMode === 1 ? "#8354ce" : "gray";
    let velColor = vectorMode === 1 ? "gray" : "red";
    let posLab = vectorMode === 1 ? "r(t)" : "r";
    let velLab = "v";

    // 현재 벡터 그리기
    drawMoveVector(origin_position.x, origin_position.y, position.x - origin_position.x, position.y - origin_position.y, posColor, posLab);                      
    drawMoveVector(position.x, position.y, velocity.x * vectorScale, velocity.y * vectorScale, velColor, velLab);                  
     
    if (showComponents) {                                                                         
      ctx.setLineDash([5, 5]);                                                                    
      drawMoveVector(origin_position.x, origin_position.y, position.x - origin_position.x, 0, posColor, posLab + "x");                    
      drawMoveVector(position.x, origin_position.y, 0, position.y - origin_position.y, posColor, posLab + "y");            
      ctx.setLineDash([]);                                                                        
    }

    // 원점 표시
    ctx.fillStyle = "#334155";                                                                       
    ctx.beginPath();
    ctx.arc(origin_position.x, origin_position.y, 5, 0, Math.PI * 2);
    ctx.fill();

    // 물체 그리기
    ctx.fillStyle = "#a2a2a2";                                                                    
    ctx.beginPath();
    ctx.arc(position.x, position.y, 10, 0, Math.PI * 2);
    ctx.fill();
    ctx.lineWidth = 2; 
    ctx.strokeStyle = "#000"; 
    ctx.stroke();             

    requestAnimationFrame(draw);
  }

 function drawMoveVector(x, y, v_x, v_y, color, label) {
    if (Math.abs(v_x) < 0.1 && Math.abs(v_y) < 0.1) return;
    ctx.save();
    const headLength = 10;                 
    const angle = Math.atan2(v_y, v_x);
    ctx.strokeStyle = color;
    ctx.lineWidth = 2;
    ctx.beginPath(); ctx.moveTo(x, y); ctx.lineTo(x + v_x, y + v_y); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(x + v_x, y + v_y);
    ctx.lineTo(x + v_x - headLength * Math.cos(angle - Math.PI / 7), y + v_y - headLength * Math.sin(angle - Math.PI / 7));
    ctx.lineTo(x + v_x - headLength * Math.cos(angle + Math.PI / 7), y + v_y - headLength * Math.sin(angle + Math.PI / 7));
    ctx.closePath(); ctx.fillStyle = color; ctx.fill();

    const midX = x + v_x / 2; const midY = y + v_y / 2;
    const textX = midX + 15 * Math.cos(angle - Math.PI / 2);
    const textY = midY + 15 * Math.sin(angle - Math.PI / 2);
    ctx.font = "bold 16px 'Times New Roman'";
    ctx.fillStyle = color;
    ctx.fillText(label, textX, textY);
    ctx.restore(); 
 }

 function handlePointerDown(e) {
    if (!isReset) return; 
    const pos = getPos(e);
    const distToObject = Math.hypot(pos.x - position.x, pos.y - position.y);
    const vEnd = { x: position.x + velocity.x * vectorScale, y: position.y + velocity.y * vectorScale };
    const distToVelocity = Math.hypot(pos.x - vEnd.x, pos.y - vEnd.y);
    
    if (distToObject < 25) isDraggingObject = true;
    else if (distToVelocity < 25) isDraggingVelocity = true;
    if (isDraggingObject || isDraggingVelocity) e.preventDefault();
 }

 function handlePointerMove(e) {
    const pos = getPos(e);
    if (isDraggingObject) {
        position.x = pos.x;
        position.y = Math.min(pos.y, groundY - 10); // 바닥 아래로 드래그 방지
        initial_position = { ...position };
    } else if (isDraggingVelocity) {
        velocity.x = (pos.x - position.x) / vectorScale;
        velocity.y = (pos.y - position.y) / vectorScale;
        initial_velocity = { ...velocity };
    }
    if (e.type === 'touchmove' && (isDraggingObject || isDraggingVelocity)) e.preventDefault();
 }

 function handlePointerUp() { isDraggingObject = false; isDraggingVelocity = false; }

 canvas.addEventListener('mousedown', handlePointerDown);
 window.addEventListener('mousemove', handlePointerMove);
 window.addEventListener('mouseup', handlePointerUp);
 canvas.addEventListener('touchstart', handlePointerDown, { passive: false });
 window.addEventListener('touchmove', handlePointerMove, { passive: false });
 window.addEventListener('touchend', handlePointerUp);

 draw();
</script>