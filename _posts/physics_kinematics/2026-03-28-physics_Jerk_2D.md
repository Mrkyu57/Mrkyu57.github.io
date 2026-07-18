---
layout: post
title: "물리학"
koreantitle: 등저크직선운동
englishtitle: constant jerk motion
info: 저크가 일정하면 가속도가 변합니다.
color: "#8354ce"
permalink: /physics_classicaldynamics_1_3/
---

<div style="margin-top: 10px;">
    <button id="start" style="padding: 8px 16px; cursor: pointer; background: #000000; color: white; border: none; border-radius: 4px;">&#9654;</button>
    <button id="reset" style="padding: 8px 16px; cursor: pointer; background: #000000; color: white; border: none; border-radius: 4px; font-size: 17px;">&#8635;</button>
    <button id="Com" style="padding: 8px 16px; cursor: pointer; background: #555; color: white; border: none; border-radius: 4px; font-weight: bold;">Com</button>
    <button id="initial" style="padding: 8px 16px; cursor: pointer; background: #555; color: white; border: none; border-radius: 4px; font-weight: bold;">I</button>
    <button id="function" style="padding: 8px 16px; cursor: pointer; background: #555; color: white; border: none; border-radius: 4px; font-weight: bold;">f</button>
</div>

<canvas id="motionCanvas" width="900" height="400" style="border:1px solid #ddd; background: #f9f9f9; margin: 0 auto; display: block;"></canvas>

<script>
  const canvas = document.getElementById('motionCanvas');
  const ctx = canvas.getContext('2d');

  const start = document.getElementById('start');
  const reset = document.getElementById('reset');
  const comBtn = document.getElementById('Com');
  const iniBtn = document.getElementById('initial');
  const funBtn = document.getElementById('function');

  const origin_position = { x: 450, y: 200 };
  
  let initial_position = { x: 100, y: 200 };
  let initial_velocity = { x: 1.0, y: 0 };
  let initial_acceleration = { x: 0, y: 0 };
  let initial_jerk = { x: 0.0002, y: 0 }; // 저크는 값이 매우 작아야 제어가 가능합니다.

  let position = { ...initial_position };
  let velocity = { ...initial_velocity };
  let acceleration = { ...initial_acceleration };
  let jerk = { ...initial_jerk };

  let isMoving = false;
  let isReset = true;
  let isDraggingObject = false;
  let isDraggingVelocity = false;
  let isDraggingAcceleration = false;
  let isDraggingJerk = false;

  let showInitial = false;
  let showComponents = false;
  let vectorMode = 0;

  // 벡터 표시 배율
  const V_SCALE = 80;
  const A_SCALE = 3000;
  const J_SCALE = 200000; // 저크는 매우 작으므로 큰 배율 필요

  start.addEventListener('click', () => {
    isMoving = !isMoving;
    isReset = false;
    if (isMoving) {
        start.innerHTML = '<span>&#10074;&#10074;</span>';
        start.style.background = "#ffc107";
        start.style.color = "#000";
    } else {
        start.innerHTML = '<span>&#9654;</span>';
        start.style.background = "#000000";
        start.style.color = "#fff";
    }
  });

  reset.addEventListener('click', () => {
    isMoving = false;
    isReset = true;
    position = { ...initial_position };
    velocity = { ...initial_velocity };
    acceleration = { ...initial_acceleration };
    jerk = { ...initial_jerk };
    start.innerHTML = '<span>&#9654;</span>';
    start.style.background = "#000000";
  });

  comBtn.addEventListener('click', () => {
    showComponents = !showComponents;
    comBtn.style.background = showComponents ? "#8354ce" : "#555";
  });

  iniBtn.addEventListener('click', () => {
    showInitial = !showInitial;
    iniBtn.style.background = showInitial ? "#8354ce" : "#555";
  });

  funBtn.addEventListener('click', () => {
    vectorMode = (vectorMode + 1) % 2;
    funBtn.style.background = vectorMode === 1 ? "#8354ce" : "#555";
  });

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (isMoving) {
      // 등저크 운동의 핵심 로직
      acceleration.x += jerk.x;
      acceleration.y += jerk.y;
      velocity.x += acceleration.x;
      velocity.y += acceleration.y;
      position.x += velocity.x;
      position.y += velocity.y;
    }

    const dispX = position.x - origin_position.x;
    const dispY = position.y - origin_position.y;

    let posCol = vectorMode === 1 ? "#8354ce" : "gray";
    let velCol = vectorMode === 1 ? "gray" : "red";
    let posName = vectorMode === 1 ? "r(t)" : "r";
    let velName = vectorMode === 1 ? "v(t)" : "v";

    // 원점 표시
    ctx.fillStyle = "gray";
    ctx.beginPath(); ctx.arc(origin_position.x, origin_position.y, 3, 0, Math.PI * 2); ctx.fill();

    // 벡터 그리기
    drawMoveVector(origin_position.x, origin_position.y, dispX, dispY, posCol, posName);
    drawMoveVector(position.x, position.y, velocity.x * V_SCALE, velocity.y * V_SCALE, velCol, velName);
    drawMoveVector(position.x, position.y, acceleration.x * A_SCALE, acceleration.y * A_SCALE, "blue", "a");
    drawMoveVector(position.x, position.y, jerk.x * J_SCALE, jerk.y * J_SCALE, "green", "j");

    // 물체 그리기
    ctx.fillStyle = "#a2a2a2";
    ctx.beginPath(); ctx.arc(position.x, position.y, 12, 0, Math.PI * 2); ctx.fill();
    ctx.strokeStyle = "#000"; ctx.lineWidth = 2; ctx.stroke();

    requestAnimationFrame(draw);
  }

  function drawMoveVector(x, y, vx, vy, color, label) {
    if (Math.abs(vx) < 0.1 && Math.abs(vy) < 0.1) return;
    ctx.save();
    const angle = Math.atan2(vy, vx);
    ctx.strokeStyle = color; ctx.fillStyle = color; ctx.lineWidth = 2;
    ctx.beginPath(); ctx.moveTo(x, y); ctx.lineTo(x + vx, y + vy); ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(x + vx, y + vy);
    ctx.lineTo(x + vx - 10 * Math.cos(angle - 0.4), y + vy - 10 * Math.sin(angle - 0.4));
    ctx.lineTo(x + vx - 10 * Math.cos(angle + 0.4), y + vy - 10 * Math.sin(angle + 0.4));
    ctx.fill();
    ctx.font = "bold 16px 'Times New Roman', serif";
    ctx.fillText(label, x + vx + 10 * Math.cos(angle), y + vy + 10 * Math.sin(angle));
    ctx.restore();
  }

  function getMousePos(e) {
    const rect = canvas.getBoundingClientRect();
    return { x: e.clientX - rect.left, y: e.clientY - rect.top };
  }

  canvas.addEventListener('mousedown', (e) => {
    if (!isReset) return;
    const mouse = getMousePos(e);
    if (Math.hypot(mouse.x - position.x, mouse.y - position.y) < 20) isDraggingObject = true;
    else if (Math.hypot(mouse.x - (position.x + velocity.x * V_SCALE), mouse.y - (position.y + velocity.y * V_SCALE)) < 20) isDraggingVelocity = true;
    else if (Math.hypot(mouse.x - (position.x + acceleration.x * A_SCALE), mouse.y - (position.y + acceleration.y * A_SCALE)) < 20) isDraggingAcceleration = true;
    else if (Math.hypot(mouse.x - (position.x + jerk.x * J_SCALE), mouse.y - (position.y + jerk.y * J_SCALE)) < 20) isDraggingJerk = true;
  });

  window.addEventListener('mousemove', (e) => {
    if (!isDraggingObject && !isDraggingVelocity && !isDraggingAcceleration && !isDraggingJerk) return;
    const mouse = getMousePos(e);
    if (isDraggingObject) {
        position.x = initial_position.x = mouse.x;
        position.y = initial_position.y = mouse.y;
    } else if (isDraggingVelocity) {
        velocity.x = initial_velocity.x = (mouse.x - position.x) / V_SCALE;
        velocity.y = initial_velocity.y = (mouse.y - position.y) / V_SCALE;
    } else if (isDraggingAcceleration) {
        acceleration.x = initial_acceleration.x = (mouse.x - position.x) / A_SCALE;
        acceleration.y = initial_acceleration.y = (mouse.y - position.y) / A_SCALE;
    } else if (isDraggingJerk) {
        jerk.x = initial_jerk.x = (mouse.x - position.x) / J_SCALE;
        jerk.y = initial_jerk.y = (mouse.y - position.y) / J_SCALE;
    }
  });

  window.addEventListener('mouseup', () => {
    isDraggingObject = isDraggingVelocity = isDraggingAcceleration = isDraggingJerk = false;
  });

  draw();
</script>

<div style="text-align: center; margin-top: 20px; font-family: 'Times New Roman', serif;">
  <h3>$j = \text{const.}$</h3>
  <p>따라서 시간이 $t$일 때, 물리량의 식은:</p>
  <p>$a(t) = a_0 + jt$</p>
  <p>$v(t) = v_0 + a_0t + \frac{1}{2}jt^2$</p>
  <p>$r(t) = r_0 + v_0t + \frac{1}{2}a_0t^2 + \frac{1}{6}jt^3$</p>
</div>