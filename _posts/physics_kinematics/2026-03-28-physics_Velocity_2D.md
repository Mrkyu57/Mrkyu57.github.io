---
layout: post
title: "물리학"
koreantitle: 등속직선운동
englishtitle: uniform linear motion
info: 흠
color: "#8354ce"
permalink: /physics_classicaldynamics_1_1/
---

<!-- 모델링 공간 -->
<div id="uniformlinearmotion2D-container" style="
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
        uniform linear motion
        </h3>
        <!-- 재생 버튼 -->
        <button id="start" title="시작" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 14px;">&#9654;</span>
        </button>
        <!-- 리셋 버튼 -->
        <button id="reset" title="초기화" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 18px;">&#8635;</span>
        </button>
        <!-- 구분선 -->
        <div style="width: 1px; height: 24px; background: #ddd; margin: 6px 4px;"></div>
        <!-- 기능 버튼들 -->
        <button id="Com" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">Com</button>
        <button id="initial" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem;">I</button>
    </div>
  <!-- 캔버스 영역 -->
  <canvas id="motionCanvas" width="900" height="600" style="
        display: block;
        background: #f9fafb;
        cursor: crosshair;
  "></canvas>
</div>

<div style="text-align: center; bold 18px 'Times New Roman', serif;">
  <h3>v = const.</h3>
  <p>따라서 시간이 t 일때, 물리량의 식은</p>
</div>

<script>
  const canvas = document.getElementById('motionCanvas');
  const ctx = canvas.getContext('2d');

  const start = document.getElementById('start');
  const reset = document.getElementById('reset');
  const comBtn = document.getElementById('Com');
  const iniBtn = document.getElementById('initial');
  const radius = 15;                                              // 물체의 반지름
  const origin_position = { x: 450, y: 300 };                     // 원점의 위치
  
  let initial_position = { x: 450, y: 300 };                      // 처음 물체의 위치
  let initial_velocity = { x: 0.5, y: -0.1 };                     // 처음 물체의 속도
  let position = { x: 450, y: 300 };                              // 위치 벡터
  let velocity = { x: 0.5, y: -0.1 };                             // 속도 벡터
  let isMoving = false;                                           // 정지 여부 변수
  let isReset = true;                                             // 리셋 여부 변수
  let isDraggingObject = false;
  let isDraggingVelocity = false;
  let showInitial = false;
  let showComponents = false;                                     // 벡터 성분 표시 (직각좌표)
  
  start.addEventListener('click', () => {                         // 정지 버튼을 클릭한 경우 >
    isMoving = !isMoving;                                         // isMoving의 bool값을 반대로 변환
    isReset = false;                                              // 리셋 미실행

    if (isMoving) {                                               // 움직이는 경우 >
        start.innerHTML = '<span>&#10074;&#10074;</span>';        // ▮▮로 버튼의 모양 변환
        start.style.background = "#ffc107";                     // 버튼 배경 색상은 노란색
        start.style.color = "#000";                             // 버튼 글씨 색상은 검정색  
    } 
    else {                                                        // 정지한 경우 >
        start.innerHTML = '<span>&#9654;</span>';                 // ▶로 버튼의 모양 변환
        start.style.background = "#475569";                     // 버튼 배경 색상은 검정색
        start.style.color = "#fff";                             // 버튼 글씨 색상은 하양색 
    }
  });

  // 리셋 버튼을 클릭한 경우 >
  reset.addEventListener('click', () => {                         
    isMoving = false;                                             // 정지
    isReset = true;                                               // 리셋 실행함
    position = { ...initial_position };                           // 위치는 초기 위치로
    velocity = { ...initial_velocity };                           // 속도는 초기 속도로
    start.innerHTML = '<span>&#9654;</span>';                     // ▶로 버튼의 모양 변환
    start.style.background = "#475569";                         // 버튼 배경 색상은 검정색
    start.style.color = "#fff";                                 // 버튼 글씨 색상은 하양색
  });

   // 데카르트 벡터 분해 버튼을 클릭한 경우 >
  comBtn.addEventListener('click', () => {                        
    showComponents = !showComponents;                             // showComponents의 bool값을 반대로 변환
    comBtn.style.background =                                     // 지정된 두 색상을 반대로 변환
    showComponents ? "#8354ce" : "#555";                     
  });

  // 초기 상태 버튼을 클릭한 경우 >
  iniBtn.addEventListener('click', () => {                        
    showInitial = !showInitial;                                   // showInitial의 bool값을 반대로 변환
    iniBtn.style.background =                                     // 지정된 두 색상을 반대로 변환
    showInitial ? "#8354ce" : "#555";                        
  });




  function draw() {                                               // 그리기 함수 >>>
    ctx.clearRect(0, 0, canvas.width, canvas.height);             // 캔버스 초기화

    if (isMoving) {                                               // 움직이는 경우 >
      position.x += velocity.x;                                   // x 위치 계산
      position.y += velocity.y;                                   // y 위치 계산      
    }

    if (showInitial === true && isReset === false) {              // 초기상태 버튼을 누르고 리셋미실행일 경우 >

      drawMoveVector(                                             // [초기 속도 벡터] :
      initial_position.x,                                         // 초기 위치의 X값
      initial_position.y,                                         // 초기 위치의 y값
      initial_velocity.x * 80,                                    // 초기 속도의 X좌표값 (보정)
      initial_velocity.y * 80,                                    // 초기 속도의 Y좌표값 (보정)
      "#ff9696","v₀");
      
      drawMoveVector(
      origin_position.x, 
      origin_position.y, 
      initial_position.x - origin_position.x, 
      initial_position.y - origin_position.y, 
      "#d3d3d396","r₀");

      ctx.fillStyle = "#d3d3d3";
      ctx.beginPath();
      ctx.arc(initial_position.x, initial_position.y, 10, 0, Math.PI * 2);
      ctx.fill();
    }

    const dispX = position.x - origin_position.x;                                                         // 원점으로 부터의 x방향 위치 차이
    const dispY = position.y - origin_position.y;                                                         // 원점으로 부터의 y방향 위치 차이

    let positionColor = "gray"; 
    let positionColor_X = "gray"; 
    let positionColor_Y = "gray"; 
    let vectorColor = "red"; 
    let vectorColor_X = "red";
    let vectorColor_Y = "red";

    let positionName = "r"; 
    let positionName_X = "rx"; 
    let positionName_Y = "ry"; 
    let vectorName = "v"; 
    let vectorName_X = "vx";
    let vectorName_Y = "vy";
    
    

    drawMoveVector(origin_position.x, origin_position.y, dispX, dispY, positionColor, positionName);                      // 변위벡터 그리기 :
    drawMoveVector(position.x, position.y, velocity.x * 80, velocity.y * 80, vectorColor, vectorName);                  // 속도벡터 그리기 :
    
    if (showComponents) {                                                                                 // 벡터 성분을 표시할때 >
      ctx.setLineDash([5, 5]);                                                                            // 선 스타일 - 점선
      drawMoveVector(origin_position.x, origin_position.y, dispX, 0, positionColor_X, positionName_X);                     // 변위벡터 x축 분해 그리기 :
      drawMoveVector(origin_position.x + dispX, origin_position.y, 0, dispY, positionColor_Y, positionName_Y);             // 변위벡터 y축 분해 그리기 :
      drawMoveVector(position.x, position.y, velocity.x * 80, 0, vectorColor_X, vectorName_X);                            // 속도벡터 x축 분해 그리기 :
      drawMoveVector(position.x + velocity.x * 80, position.y, 0, velocity.y * 80, vectorColor_Y, vectorName_Y);          // 속도벡터 y축 분해 그리기 :
      ctx.setLineDash([]);                                                                                // 선 스타일 - 실선
    }

    ctx.fillStyle = "gray";                                                                   // 원점 그리기 >
    ctx.beginPath();
    ctx.arc(origin_position.x, origin_position.y, 3, 0, Math.PI * 2);
    ctx.fill();

    ctx.fillStyle = "#a2a2a2";                                                              // 물체 그리기 >
    ctx.beginPath();
    ctx.arc(position.x, position.y, 10, 0, Math.PI * 2);
    ctx.fill();
    ctx.lineWidth = 2; 
    ctx.strokeStyle = "#000000"; 
    ctx.stroke();            
    ctx.closePath();

    requestAnimationFrame(draw);
  }





function drawMoveVector(x, y, delta_x, delta_y, color, label) {
    if (delta_x === 0 && delta_y === 0) return;
    
    ctx.save(); 
    
    const headLength = 10;                
    const angle = Math.atan2(delta_y, delta_x);

    // 벡터 그리기
    ctx.strokeStyle = color;
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(x, y);
    ctx.lineTo(x + delta_x, y + delta_y);
    ctx.stroke();

    // 화살표 머리 그리기
    ctx.beginPath();
    ctx.moveTo(x + delta_x, y + delta_y);
    ctx.lineTo(
        x + delta_x - headLength * Math.cos(angle - Math.PI / 7), 
        y + delta_y - headLength * Math.sin(angle - Math.PI / 7)
    );
    ctx.lineTo(
        x + delta_x - headLength * Math.cos(angle + Math.PI / 7), 
        y + delta_y - headLength * Math.sin(angle + Math.PI / 7)
    );
    ctx.closePath();
    ctx.fillStyle = color;
    ctx.fill();

    // 3. 텍스트 분리 로직 (핵심 수정 부분!)
    const labelStr = String(label);

    // 첫 글자는 기본, 나머지는 아래첨자로 분리
    let mainText = labelStr.charAt(0); 
    let subText = labelStr.slice(1); 

    // 텍스트 좌표 계산
    const midX = x + delta_x / 2;
    const midY = y + delta_y / 2;
    const offset = 15; 
    const textX = midX + offset * Math.cos(angle - Math.PI / 2);
    const textY = midY + offset * Math.sin(angle - Math.PI / 2);

    ctx.textAlign = "left"; 
    ctx.textBaseline = "middle";

    // 외곽선 효과 (가독성용)
    ctx.strokeStyle = "#f9f9f9";
    ctx.lineWidth = 4;
    ctx.lineJoin = "round";

    // 메인 글자 출력
    ctx.font = "bold 18px 'Times New Roman', serif";
    ctx.strokeText(mainText, textX, textY);
    ctx.fillStyle = color; // 라벨 색상을 벡터 색상과 맞춤
    ctx.fillText(mainText, textX, textY);

    // 아래첨자 출력
    if (subText) {
        const mainWidth = ctx.measureText(mainText).width; 
        ctx.font = "bold 12px 'Times New Roman', serif";  
        
        const subX = textX + mainWidth;
        const subY = textY + 5; 
        
        ctx.strokeText(subText, subX, subY);
        ctx.fillText(subText, subX, subY);
    }
    ctx.restore(); 
}




  function getMousePos(e) {
    const rect = canvas.getBoundingClientRect();
    return { x: e.clientX - rect.left, y: e.clientY - rect.top };
  }

  // [마우스 드래그 로직 핵심 수정]
  window.addEventListener('mousemove', (e) => {
    if (!isDraggingObject && !isDraggingVelocity) return;
    const mouse = getMousePos(e);

    if (isDraggingObject) {
        // 1. 물체의 현재 위치 업데이트
        position.x = mouse.x;
        position.y = mouse.y;
        // 2. 초기 위치(Reset 시 돌아갈 곳)를 현재 마우스 위치로 저장
        initial_position.x = mouse.x;
        initial_position.y = mouse.y;
        // ※ origin_position(원점)은 건드리지 않으므로 고정됩니다.
    } else if (isDraggingVelocity) {
        velocity.x = (mouse.x - position.x) / 80;
        velocity.y = (mouse.y - position.y) / 80;
        initial_velocity.x = velocity.x;
        initial_velocity.y = velocity.y;
    }
  });

  // (나머지 mousedown, mouseup, cursor 처리 리스너는 동일)
  canvas.addEventListener('mousedown', (e) => {
    if (!isReset) return; 
    const mouse = getMousePos(e);
    const distToObject = Math.hypot(mouse.x - position.x, mouse.y - position.y);
    const vEnd = { x: position.x + velocity.x * 80, y: position.y + velocity.y * 80 };
    const distToVelocity = Math.hypot(mouse.x - vEnd.x, mouse.y - vEnd.y);
    if (distToObject < 20) isDraggingObject = true;
    else if (distToVelocity < 20) isDraggingVelocity = true;
  });

  window.addEventListener('mouseup', () => {
    isDraggingObject = false;
    isDraggingVelocity = false;
  });

  canvas.addEventListener('mousemove', (e) => {
    const mouse = getMousePos(e);
    const distToObject = Math.hypot(mouse.x - position.x, mouse.y - position.y);
    const vEnd = { x: position.x + velocity.x * 80, y: position.y + velocity.y * 80 };
    const distToVelocity = Math.hypot(mouse.x - vEnd.x, mouse.y - vEnd.y);
    if (distToObject < 20 || distToVelocity < 20) canvas.style.cursor = 'grab';
    else canvas.style.cursor = 'default';
    if (isDraggingObject || isDraggingVelocity) canvas.style.cursor = 'grabbing';
  });

  draw();

</script>

