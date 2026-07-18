---
layout: post
title: "물리학"
koreantitle: 경사면 위의 운동
englishtitle: Motion on an Inclined Plane
info: 경사면
color: "#8354ce"
permalink: /physics_classicaldynamics_1_2/
---

<div id="origami-axiom1-container" style="
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
        gap: 12px;
    ">
        <h3 style="margin: 0px 15px 0px 0px; color: #333; font-family: 'Times New Roman', serif; ">
        Inclined Plane
        </h3> 
        <button id="start" title="시작" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 14px;">&#9654;</span>
        </button>
        <button id="reset" title="초기화" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 18px;">&#8635;</span>
        </button>
        <div style="width: 1px; height: 24px; background: #ddd; margin: 0 4px;"></div>
        <div style="display: flex; align-items: center; gap: 8px;">
            <label for="angleSlider" style="font-weight: 600; font-size: 0.9rem; color: #334155;">Angle (θ): <span id="angleValue">30</span>°</label>
            <input type="range" id="angleSlider" min="0" max="60" value="30" style="width: 100px; cursor: pointer;">
        </div>
        <div style="width: 1px; height: 24px; background: #ddd; margin: 0 4px;"></div>
        <button id="toggleForce" style="padding: 0 16px; height: 36px; cursor: pointer; background: #8354ce; color: white; border: none; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">Forces</button>
    </div>
    <canvas id="motionCanvas" width="900" height="400" style="
        display: block;
        background: #f9fafb;
    "></canvas>
</div>

<div style="text-align: center; font: bold 18px 'Times New Roman', serif;">
  <h3>a = g sin(θ)</h3>
  <p>물체는 중력의 빗면 성분에 의해 등가속도 운동을 합니다.</p>
</div>

<script>
  const canvas = document.getElementById('motionCanvas');
  const ctx = canvas.getContext('2d');

  const startBtn = document.getElementById('start');
  const resetBtn = document.getElementById('reset');
  const angleSlider = document.getElementById('angleSlider');
  const angleValueDisplay = document.getElementById('angleValue');
  const toggleForceBtn = document.getElementById('toggleForce');

  // 물리 상수 및 환경 설정
  const g = 0.2; // 중력 가속도 (화면 픽셀 스케일에 맞춤)
  const origin = { x: 150, y: 350 }; // 빗면의 가장 아래쪽(왼쪽) 좌표
  const slopeLength = 600; // 빗면의 길이
  const boxSize = 40; // 상자 크기

  // 상태 변수
  let angleDeg = parseInt(angleSlider.value);
  let angleRad = angleDeg * (Math.PI / 180);
  
  // 물체의 위치는 (x, y)가 아닌 빗면을 따라 이동한 거리(s)로 1차원 취급합니다.
  let initial_s = 500; // 초기 빗면 위 위치 (위쪽에서 시작)
  let s = initial_s; // 현재 빗면 위 위치
  let v = 0; // 빗면 방향 속도

  let isMoving = false;
  let isDraggingObject = false;
  let showForces = true;

  // 이벤트 리스너: 재생/일시정지
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

  // 이벤트 리스너: 초기화
  resetBtn.addEventListener('click', () => {
    isMoving = false;
    s = initial_s;
    v = 0;
    startBtn.innerHTML = '<span>&#9654;</span>';
    startBtn.style.background = "#475569";
    startBtn.style.color = "#fff";
  });

  // 이벤트 리스너: 각도 슬라이더 조절
  angleSlider.addEventListener('input', (e) => {
    angleDeg = parseInt(e.target.value);
    angleValueDisplay.innerText = angleDeg;
    angleRad = angleDeg * (Math.PI / 180);
    
    // 각도 변경 시 물체 정지 및 초기 위치 세팅
    isMoving = false;
    v = 0;
    startBtn.innerHTML = '<span>&#9654;</span>';
    startBtn.style.background = "#475569";
    startBtn.style.color = "#fff";
  });

  // 이벤트 리스너: 힘(벡터) 표시 토글
  toggleForceBtn.addEventListener('click', () => {
    showForces = !showForces;
    toggleForceBtn.style.background = showForces ? "#8354ce" : "#f1f5f9";
    toggleForceBtn.style.color = showForces ? "white" : "#334155";
    toggleForceBtn.style.border = showForces ? "none" : "1px solid #e2e8f0";
  });

  // 좌표 변환 함수: 빗면 상의 거리 s를 캔버스의 (x, y)로 변환
  function getPositionFromS(distance_s) {
    return {
      x: origin.x + distance_s * Math.cos(angleRad),
      y: origin.y - distance_s * Math.sin(angleRad)
    };
  }

  // 그리기 루프
  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 1. 물리 로직 업데이트 (움직임 처리)
    if (isMoving && !isDraggingObject) {
      const a = -g * Math.sin(angleRad); // 가속도: 아래 방향이 음수(-), origin 방향
      v += a;
      s += v;

      // 바닥에 닿거나 빗면 끝을 넘어가면 정지 및 제한
      if (s <= boxSize / 2) {
        s = boxSize / 2;
        v = 0;
      }
    }

    // 2. 배경 및 빗면(삼각형) 그리기
    const topX = origin.x + slopeLength * Math.cos(angleRad);
    const topY = origin.y - slopeLength * Math.sin(angleRad);
    
    ctx.fillStyle = "#e2e8f0";
    ctx.beginPath();
    ctx.moveTo(origin.x, origin.y);
    ctx.lineTo(topX, topY);
    ctx.lineTo(topX, origin.y); // 직각삼각형 밑변
    ctx.closePath();
    ctx.fill();
    ctx.strokeStyle = "#cbd5e1";
    ctx.lineWidth = 2;
    ctx.stroke();

    // 3. 각도 표시 (호 그리기)
    if (angleDeg > 0) {
      ctx.beginPath();
      ctx.arc(topX, origin.y, 40, Math.PI, Math.PI + angleRad);
      ctx.strokeStyle = "#94a3b8";
      ctx.stroke();
      ctx.fillStyle = "#475569";
      ctx.font = "italic 16px 'Times New Roman'";
      ctx.fillText("θ", topX - 55, origin.y - 15);
    }

    // 현재 물체의 캔버스 좌표 계산
    const pos = getPositionFromS(s);

    // 4. 물체(상자) 그리기
    ctx.save();
    ctx.translate(pos.x, pos.y);
    ctx.rotate(-angleRad); // 빗면 각도에 맞춰 캔버스 회전
    
    // 상자 그리기 (밑면이 빗면에 닿도록 중심점 보정)
    ctx.fillStyle = "#a2a2a2";
    ctx.strokeStyle = "#000000";
    ctx.lineWidth = 2;
    ctx.fillRect(-boxSize/2, -boxSize, boxSize, boxSize);
    ctx.strokeRect(-boxSize/2, -boxSize, boxSize, boxSize);
    
    ctx.restore();

    // 5. 벡터 표시 (선택사항)
    if (showForces) {
      const centerX = pos.x - (boxSize/2) * Math.sin(angleRad);
      const centerY = pos.y - (boxSize/2) * Math.cos(angleRad);

      // 중력 벡터 (mg) - 아래로
      drawVector(centerX, centerY, 0, 80, "#ef4444", "mg");
      
      // 속도 벡터 (v) - 빗면 방향
      if (Math.abs(v) > 0.1) {
        const vx = v * 5 * Math.cos(angleRad); // 화면 스케일링
        const vy = -v * 5 * Math.sin(angleRad);
        drawVector(centerX, centerY, vx, vy, "#3b82f6", "v");
      }
    }

    requestAnimationFrame(draw);
  }

  // 간단한 벡터 그리기 함수 (기존 로직 축소판)
  function drawVector(x, y, vx, vy, color, label) {
    const headLength = 10;
    const angle = Math.atan2(vy, vx);

    ctx.save();
    ctx.strokeStyle = color;
    ctx.fillStyle = color;
    ctx.lineWidth = 2;

    ctx.beginPath();
    ctx.moveTo(x, y);
    ctx.lineTo(x + vx, y + vy);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(x + vx, y + vy);
    ctx.lineTo(x + vx - headLength * Math.cos(angle - Math.PI / 7), y + vy - headLength * Math.sin(angle - Math.PI / 7));
    ctx.lineTo(x + vx - headLength * Math.cos(angle + Math.PI / 7), y + vy - headLength * Math.sin(angle + Math.PI / 7));
    ctx.closePath();
    ctx.fill();

    ctx.font = "bold 16px 'Times New Roman', serif";
    ctx.fillText(label, x + vx + 5, y + vy + 5);
    ctx.restore();
  }

  // --- 마우스 인터랙션 로직 ---
  function getMousePos(e) {
    const rect = canvas.getBoundingClientRect();
    return { x: e.clientX - rect.left, y: e.clientY - rect.top };
  }

  canvas.addEventListener('mousedown', (e) => {
    const mouse = getMousePos(e);
    const pos = getPositionFromS(s);
    // 물체 중심(대략)과의 거리 확인
    const dist = Math.hypot(mouse.x - pos.x, mouse.y - pos.y);
    
    if (dist < boxSize) {
      isDraggingObject = true;
      isMoving = false; // 드래그 중에는 물리엔진 일시정지
      v = 0; // 속도 초기화
      startBtn.innerHTML = '<span>&#9654;</span>';
      startBtn.style.background = "#475569";
    }
  });

  window.addEventListener('mousemove', (e) => {
    if (!isDraggingObject) {
      const mouse = getMousePos(e);
      const pos = getPositionFromS(s);
      if (Math.hypot(mouse.x - pos.x, mouse.y - pos.y) < boxSize) {
        canvas.style.cursor = 'grab';
      } else {
        canvas.style.cursor = 'default';
      }
      return;
    }
    
    canvas.style.cursor = 'grabbing';
    const mouse = getMousePos(e);
    
    // 마우스 좌표를 빗면 방향의 1차원 위치(s)로 투영(Projection)
    // 벡터 내적 활용: u = (cos(theta), -sin(theta))
    const dx = mouse.x - origin.x;
    const dy = origin.y - mouse.y; // 캔버스는 y가 아래로 갈수록 커지므로 반전
    
    // 점과 선의 투영 공식
    let new_s = dx * Math.cos(angleRad) + dy * Math.sin(angleRad);
    
    // 빗면을 벗어나지 않도록 범위 제한
    if (new_s < boxSize / 2) new_s = boxSize / 2;
    if (new_s > slopeLength) new_s = slopeLength;
    
    s = new_s;
    initial_s = s; // 마우스를 놓았을 때 리셋될 초기 위치 업데이트
  });

  window.addEventListener('mouseup', () => {
    if (isDraggingObject) {
      isDraggingObject = false;
      canvas.style.cursor = 'default';
    }
  });

  // 애니메이션 시작
  draw();

</script>