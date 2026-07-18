---
layout: post
title: "물리학: 2차원 비탄성 충돌"
koreantitle: 2차원 비탄성 충돌
englishtitle: 2D inelastic collision
info: 충돌 시 운동량은 보존되나, 운동 에너지는 손실됩니다.
color: "#8354ce"
permalink: /physics_classicaldynamics_collision/
---

<div id="collision-container" style="
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
    <div style="
        padding: 15px 0; 
        background: #fcfcfc; 
        border-bottom: 1px solid #f0f0f0; 
        display: flex; 
        justify-content: center; 
        align-items: center;
        gap: 8px;
    ">
        <h3 style="margin: 0px 20px 0px 0px; color: #333; font-family: 'Times New Roman', serif; ">
        2D inelastic collision
        </h3>
        <button id="start" title="시작" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 14px;">&#9654;</span>
        </button>
        <button id="reset" title="초기화" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 18px;">&#8635;</span>
        </button>
        <div style="width: 1px; height: 24px; background: #ddd; margin: 0 4px;"></div>
        <button id="Com" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;" title="속도 벡터 성분 표시">Com</button>
        <button id="initial" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;" title="초기 상태 표시">I</button>
        <div style="margin-left: 20px; font-size: 0.9rem; color: #666;">
            반발 계수 (e) = 0.6
        </div>
    </div>
    <canvas id="motionCanvas" width="900" height="400" style="
        display: block;
        background: #f9fafb;
        cursor: crosshair;
    "></canvas>
</div>

<div style="text-align: center;">
  <h3 style="font-family: 'Times New Roman', serif;">e < 1</h3>
  <p>운동량은 보존되지만, 충돌 시 형태 변형 등으로 인해 운동 에너지는 보존되지 않습니다.</p>
</div>

<script>
  const canvas = document.getElementById('motionCanvas');
  const ctx = canvas.getContext('2d');

  const start = document.getElementById('start');
  const reset = document.getElementById('reset');
  const comBtn = document.getElementById('Com');
  const iniBtn = document.getElementById('initial');

  // 물리 상수 및 상태
  const e = 0; // 반발 계수 (0: 완전 비탄성, 1: 탄성)
  const vScale = 40; // 화면에 표시될 속도 벡터 화살표 배율

  // 물체 객체 정의 (질량, 위치, 속도, 색상 등)
  let objects = [
      { id: 1, radius: 20, mass: 1, pos: { x: 300, y: 200 }, vel: { x: 3, y: 1 }, color: "#e74c3c", name: "v1" },
      { id: 2, radius: 20, mass: 1, pos: { x: 600, y: 250 }, vel: { x: -3, y: -0.5 }, color: "#3498db", name: "v2" }
  ];

  // 초기 상태 저장을 위한 깊은 복사
  let initialObjects = JSON.parse(JSON.stringify(objects));

  let isMoving = false;
  let isReset = true;
  let showInitial = false;
  let showComponents = false;

  // 상호작용 상태
  let draggingTarget = null; // { obj: objectReference, type: 'body' | 'velocity' }

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
    objects = JSON.parse(JSON.stringify(initialObjects)); // 초기 상태로 되돌리기
    
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

  // 충돌 처리 함수
  function handleCollisions() {
      // 1. 벽면 충돌 (캔버스 경계)
      objects.forEach(obj => {
          if (obj.pos.x - obj.radius < 0) { obj.pos.x = obj.radius; obj.vel.x *= -1; }
          if (obj.pos.x + obj.radius > canvas.width) { obj.pos.x = canvas.width - obj.radius; obj.vel.x *= -1; }
          if (obj.pos.y - obj.radius < 0) { obj.pos.y = obj.radius; obj.vel.y *= -1; }
          if (obj.pos.y + obj.radius > canvas.height) { obj.pos.y = canvas.height - obj.radius; obj.vel.y *= -1; }
      });

      // 2. 두 물체 간의 충돌
      let o1 = objects[0];
      let o2 = objects[1];
      
      let dx = o2.pos.x - o1.pos.x;
      let dy = o2.pos.y - o1.pos.y;
      let dist = Math.hypot(dx, dy);
      let minDist = o1.radius + o2.radius;

      if (dist < minDist) {
          // 물체가 겹쳤을 때 떼어놓기 (Sticky 방지)
          let overlap = minDist - dist;
          let nx = dx / dist;
          let ny = dy / dist;
          
          o1.pos.x -= nx * (overlap / 2);
          o1.pos.y -= ny * (overlap / 2);
          o2.pos.x += nx * (overlap / 2);
          o2.pos.y += ny * (overlap / 2);

          // 1차원 충돌로 변환하여 속도 계산 (Normal & Tangent 방향)
          let un = { x: nx, y: ny }; // Normal vector
          let ut = { x: -ny, y: nx }; // Tangent vector

          // 충돌 전 방향별 속도
          let v1n = un.x * o1.vel.x + un.y * o1.vel.y;
          let v1t = ut.x * o1.vel.x + ut.y * o1.vel.y;
          let v2n = un.x * o2.vel.x + un.y * o2.vel.y;
          let v2t = ut.x * o2.vel.x + ut.y * o2.vel.y;

          // 반발 계수 e를 적용한 충돌 후 Normal 방향 속도
          let m1 = o1.mass;
          let m2 = o2.mass;
          
          let v1n_after = (m1 * v1n + m2 * v2n + m2 * e * (v2n - v1n)) / (m1 + m2);
          let v2n_after = (m1 * v1n + m2 * v2n + m1 * e * (v1n - v2n)) / (m1 + m2);

          // 충돌 후 속도를 원래 x, y 좌표계로 복원
          o1.vel.x = v1n_after * un.x + v1t * ut.x;
          o1.vel.y = v1n_after * un.y + v1t * ut.y;
          o2.vel.x = v2n_after * un.x + v2t * ut.x;
          o2.vel.y = v2n_after * un.y + v2t * ut.y;
      }
  }

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (isMoving) {
        objects.forEach(obj => {
            obj.pos.x += obj.vel.x;
            obj.pos.y += obj.vel.y;
        });
        handleCollisions();
    }

    // 초기 상태 표시 (고스트)
    if (showInitial && !isReset) {
        initialObjects.forEach((initObj, index) => {
            // 초기 위치 반투명 구
            ctx.fillStyle = initObj.color + "40"; // 40은 투명도(hex)
            ctx.beginPath();
            ctx.arc(initObj.pos.x, initObj.pos.y, initObj.radius, 0, Math.PI * 2);
            ctx.fill();
            
            // 초기 속도 벡터
            drawMoveVector(initObj.pos.x, initObj.pos.y, initObj.vel.x * vScale, initObj.vel.y * vScale, "#a2a2a2", initObj.name + "₀");
        });
    }

    // 현재 상태 그리기
    objects.forEach(obj => {
        // 물체 그리기
        ctx.fillStyle = obj.color;
        ctx.beginPath();
        ctx.arc(obj.pos.x, obj.pos.y, obj.radius, 0, Math.PI * 2);
        ctx.fill();
        ctx.lineWidth = 2;
        ctx.strokeStyle = "#222";
        ctx.stroke();
        ctx.closePath();

        // 속도 벡터 그리기
        drawMoveVector(obj.pos.x, obj.pos.y, obj.vel.x * vScale, obj.vel.y * vScale, "#ff3333", obj.name);

        // 성분 벡터 그리기
        if (showComponents) {
            ctx.setLineDash([5, 5]);
            drawMoveVector(obj.pos.x, obj.pos.y, obj.vel.x * vScale, 0, "gray", obj.name + "x");
            drawMoveVector(obj.pos.x + obj.vel.x * vScale, obj.pos.y, 0, obj.vel.y * vScale, "gray", obj.name + "y");
            ctx.setLineDash([]);
        }
    });

    requestAnimationFrame(draw);
  }

  // 벡터 그리기 함수 (기존 코드 개선)
  function drawMoveVector(x, y, v_x, v_y, color, label) {
    if (Math.abs(v_x) < 0.1 && Math.abs(v_y) < 0.1) return; // 속도가 거의 0이면 그리지 않음
    
    ctx.save();
    
    const headLength = 10;                
    const angle = Math.atan2(v_y, v_x);

    ctx.strokeStyle = color;
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(x, y);
    ctx.lineTo(x + v_x, y + v_y);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(x + v_x, y + v_y);
    ctx.lineTo(x + v_x - headLength * Math.cos(angle - Math.PI / 7), y + v_y - headLength * Math.sin(angle - Math.PI / 7));
    ctx.lineTo(x + v_x - headLength * Math.cos(angle + Math.PI / 7), y + v_y - headLength * Math.sin(angle + Math.PI / 7));
    ctx.closePath();
    ctx.fillStyle = color;
    ctx.fill();

    const midX = x + v_x / 2;
    const midY = y + v_y / 2;
    const offset = 15;
    const textX = midX + offset * Math.cos(angle - Math.PI / 2);
    const textY = midY + offset * Math.sin(angle - Math.PI / 2);

    const labelStr = String(label);
    const mainText = labelStr.charAt(0);
    const subText = labelStr.substring(1);

    ctx.textAlign = "left";
    ctx.textBaseline = "middle";
    ctx.strokeStyle = "#fcfcfc";
    ctx.lineWidth = 4;
    ctx.lineJoin = "round";

    ctx.font = "italic bold 18px 'Times New Roman', serif";
    ctx.fillStyle = "#333";
    
    ctx.strokeText(mainText, textX, textY);
    ctx.fillText(mainText, textX, textY);

    if (subText) {
        const mainWidth = ctx.measureText(mainText).width;
        ctx.font = "bold 12px 'Times New Roman', serif"; 
        const subX = textX + mainWidth + 1;
        const subY = textY + 5; 
        
        ctx.strokeText(subText, subX, subY);
        ctx.fillText(subText, subX, subY);
    }
    ctx.restore();
  }

  // --- 마우스 상호작용 로직 ---
  function getMousePos(e) {
    const rect = canvas.getBoundingClientRect();
    return { x: e.clientX - rect.left, y: e.clientY - rect.top };
  }

  canvas.addEventListener('mousedown', (e) => {
    if (!isReset) return; 
    const mouse = getMousePos(e);

    for (let i = 0; i < objects.length; i++) {
        let obj = objects[i];
        const distToObject = Math.hypot(mouse.x - obj.pos.x, mouse.y - obj.pos.y);
        const vEnd = { x: obj.pos.x + obj.vel.x * vScale, y: obj.pos.y + obj.vel.y * vScale };
        const distToVelocity = Math.hypot(mouse.x - vEnd.x, mouse.y - vEnd.y);

        if (distToObject < obj.radius) {
            draggingTarget = { obj: obj, initObj: initialObjects[i], type: 'body' };
            break;
        } else if (distToVelocity < 20) {
            draggingTarget = { obj: obj, initObj: initialObjects[i], type: 'velocity' };
            break;
        }
    }
  });

  window.addEventListener('mousemove', (e) => {
    if (!draggingTarget) return;
    const mouse = getMousePos(e);

    if (draggingTarget.type === 'body') {
        draggingTarget.obj.pos.x = mouse.x;
        draggingTarget.obj.pos.y = mouse.y;
        draggingTarget.initObj.pos.x = mouse.x;
        draggingTarget.initObj.pos.y = mouse.y;
    } else if (draggingTarget.type === 'velocity') {
        draggingTarget.obj.vel.x = (mouse.x - draggingTarget.obj.pos.x) / vScale;
        draggingTarget.obj.vel.y = (mouse.y - draggingTarget.obj.pos.y) / vScale;
        draggingTarget.initObj.vel.x = draggingTarget.obj.vel.x;
        draggingTarget.initObj.vel.y = draggingTarget.obj.vel.y;
    }
  });

  window.addEventListener('mouseup', () => {
    draggingTarget = null;
  });

  canvas.addEventListener('mousemove', (e) => {
    const mouse = getMousePos(e);
    let isHovering = false;

    for (let obj of objects) {
        const distToObject = Math.hypot(mouse.x - obj.pos.x, mouse.y - obj.pos.y);
        const vEnd = { x: obj.pos.x + obj.vel.x * vScale, y: obj.pos.y + obj.vel.y * vScale };
        const distToVelocity = Math.hypot(mouse.x - vEnd.x, mouse.y - vEnd.y);
        if (distToObject < obj.radius || distToVelocity < 20) {
            isHovering = true;
            break;
        }
    }

    if (isHovering && isReset) canvas.style.cursor = 'grab';
    else canvas.style.cursor = 'crosshair';
    
    if (draggingTarget) canvas.style.cursor = 'grabbing';
  });

  draw();
</script>