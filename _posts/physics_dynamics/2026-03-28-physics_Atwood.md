---
layout: post
title: "물리학"
koreantitle: 애트우드 기계 (상호작용형)
englishtitle: Atwood Machine
info: 줄의 길이와 힘의 평형을 실시간으로 관찰할 수 있는 애트우드 기계 모델입니다.
color: "#8354ce"
permalink: /physics_classicaldynamics_atwood_v3/
---

<!-- 모델링 공간 -->
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
    <!-- 제어 패널 -->
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
        Atwood Machine
        </h3>
        <button id="start" title="시작" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 14px;">&#9654;</span>
        </button>
        <button id="reset" title="초기화" style="width: 45px; height: 36px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px; display: flex; align-items: center; justify-content: center; transition: 0.2s;">
            <span style="font-size: 18px;">&#8635;</span>
        </button>
        <div style="width: 1px; height: 24px; background: #ddd; margin: 0 4px;"></div>
        <div style="display: flex; align-items: center; gap: 5px; font-weight: 600; color: #334155; font-size: 0.9rem;">
            <label style="font-family: 'Times New Roman', serif; font-style: italic;">m₁</label>
            <input type="range" id="m1_slider" min="0.5" max="10" step="0.5" value="3" style="width: 60px;">
            <span id="m1_val" style="width: 25px; text-align: right;">3.0</span>
        </div>
        <div style="display: flex; align-items: center; gap: 5px; font-weight: 600; color: #334155; font-size: 0.9rem;">
            <label style="font-family: 'Times New Roman', serif; font-style: italic;">m₂</label>
            <input type="range" id="m2_slider" min="0.5" max="10" step="0.5" value="5" style="width: 60px;">
            <span id="m2_val" style="width: 25px; text-align: right;">5.0</span>
        </div>
        <div style="width: 1px; height: 24px; background: #ddd; margin: 0 4px;"></div>
        <button id="forceBtn" style="padding: 0 16px; height: 36px; cursor: pointer; background: #f1f5f9; color: #334155; border: 1px solid #e2e8f0; border-radius: 6px; font-weight: 600; font-size: 0.85rem; transition: 0.2s;">Force Vector</button>
    </div>
    <canvas id="motionCanvas" width="900" height="420" style="display: block; background: #f9fafb; cursor: grab;"></canvas>
</div>

<div style="text-align: center; font: bold 18px 'Times New Roman', serif; margin-bottom: 30px;">
  <p>시스템 방정식</p>
  <h3>$$a = \frac{m_2 - m_1}{m_1 + m_2} g, \quad L_{total} = L_1 + L_2 = \text{const.}$$</h3>
</div>

<script>
  const canvas = document.getElementById('motionCanvas');
  const ctx = canvas.getContext('2d');

  const start = document.getElementById('start');
  const reset = document.getElementById('reset');
  const forceBtn = document.getElementById('forceBtn');
  const m1Slider = document.getElementById('m1_slider');
  const m2Slider = document.getElementById('m2_slider');
  const m1Val = document.getElementById('m1_val');
  const m2Val = document.getElementById('m2_val');

  // 물리 및 환경 변수
  const g = 0.5;
  const pulley = { x: 450, y: 80, r: 40 };
  const stringTotalLen = 380; // 줄의 총 길이 (상단 기준)
  
  let m1 = parseFloat(m1Slider.value);
  let m2 = parseFloat(m2Slider.value);
  let L1 = 130; // 왼쪽 줄 길이
  let L2 = stringTotalLen - L1; // 오른쪽 줄 길이
  let init_L1 = L1, init_L2 = L2;
  let v = 0;
  
  let isMoving = false, isReset = true;
  let isDrag1 = false, isDrag2 = false;
  let showForces = false;

  // 슬라이더 제어
  m1Slider.oninput = (e) => { m1 = parseFloat(e.target.value); m1Val.innerText = m1.toFixed(1); if(isReset) resetSim(); };
  m2Slider.oninput = (e) => { m2 = parseFloat(e.target.value); m2Val.innerText = m2.toFixed(1); if(isReset) resetSim(); };

  // 재생/정지
  start.onclick = () => {
    isMoving = !isMoving;
    isReset = false;
    updateUI();
  };

  function updateUI() {
    start.innerHTML = isMoving ? '<span>&#10074;&#10074;</span>' : '<span>&#9654;</span>';
    start.style.background = isMoving ? "#ffc107" : "#475569";
    start.style.color = isMoving ? "#000" : "#fff";
  }

  function resetSim() {
    isMoving = false; isReset = true;
    L1 = init_L1; L2 = init_L2; v = 0;
    updateUI();
  }
  reset.onclick = resetSim;

  forceBtn.onclick = () => {
    showForces = !showForces;
    forceBtn.style.background = showForces ? "#8354ce" : "#f1f5f9";
    forceBtn.style.color = showForces ? "#fff" : "#334155";
  };

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (isMoving) {
        let a = ((m2 - m1) / (m1 + m2)) * g;
        v += a;
        L1 -= v; L2 += v; 

        // 충돌 경계값 계산
        const minL = 0; 
        const maxL = stringTotalLen - 0;

        if (L1 <= minL || L2 <= minL || L1 >= maxL || L2 >= maxL) {
            v = 0; isMoving = false;
            if(L1 <= minL) { L1 = minL; L2 = stringTotalLen - L1; }
            if(L2 <= minL) { L2 = minL; L1 = stringTotalLen - L2; }
            updateUI();
        }
    }

    // 도르래 지지 구조
    ctx.strokeStyle = "#000000"; ctx.lineWidth = 2;
    ctx.beginPath(); ctx.moveTo(pulley.x, 0); ctx.lineTo(pulley.x, pulley.y); ctx.stroke();

    // 줄 (String)
    const leftX = pulley.x - pulley.r;
    const rightX = pulley.x + pulley.r;
    ctx.strokeStyle = "#475569"; ctx.lineWidth = 2;
    ctx.beginPath(); 
    ctx.moveTo(leftX, pulley.y); ctx.lineTo(leftX, pulley.y + L1);
    ctx.moveTo(rightX, pulley.y); ctx.lineTo(rightX, pulley.y + L2);
    ctx.stroke();
    ctx.beginPath(); ctx.arc(pulley.x, pulley.y, pulley.r, Math.PI, 0); ctx.stroke();

    // 줄 길이 텍스트 표기
    ctx.font = "bold 13px 'Times New Roman'";
    ctx.fillStyle = "#000000";
    ctx.textAlign = "right";
    ctx.fillText(`L1`, leftX - 5, pulley.y + L1/2 - 4);
    ctx.textAlign = "left";
    ctx.fillText(`L2`, rightX + 5, pulley.y + L2/2 - 4);

    // 도르래
    

    // 물체 (사각형) 크기 설정
    const w1 = 30 + m1 * 2.5, h1 = 30 + m1 * 2.5;
    const w2 = 30 + m2 * 2.5, h2 = 30 + m2 * 2.5;

    // m1 물체 (사각형)
    ctx.fillStyle = "#8354ce";
    ctx.fillRect(leftX - w1/2, pulley.y + L1, w1, h1);
    ctx.strokeStyle = "#000"; ctx.lineWidth = 2;
    ctx.strokeRect(leftX - w1/2, pulley.y + L1, w1, h1);
    ctx.fillStyle = "#fff"; ctx.font = "bold 15px 'Times New Roman'"; ctx.textAlign = "center";
    ctx.fillText("m₁", leftX, pulley.y + L1 + h1/2 + 5);

    // m2 물체 (사각형)
    ctx.fillStyle = "#3b82f6";
    ctx.fillRect(rightX - w2/2, pulley.y + L2, w2, h2);
    ctx.strokeRect(rightX - w2/2, pulley.y + L2, w2, h2);
    ctx.fillStyle = "#fff";
    ctx.fillText("m₂", rightX, pulley.y + L2 + h2/2 + 5);

    ctx.fillStyle = "#ffffff"; ctx.beginPath(); ctx.arc(pulley.x, pulley.y, pulley.r, 0, Math.PI * 2); ctx.fill();
    ctx.strokeStyle = "#000000"; ctx.lineWidth = 2; ctx.stroke();

    // 벡터 그리기
    if (showForces) {
        const tScale = 25, gScale = 15; 
        const tension = (2 * m1 * m2 / (m1 + m2)) * g;
        
        // 힘 벡터: 최소길이 35px, 머리크기 10px 고정
        drawMoveVector(leftX, pulley.y + L1, 0, -tension * tScale, "#10b981", "T", 35); // m1 장력
        drawMoveVector(leftX, pulley.y + L1 + h1, 0, m1 * g * gScale, "#ef4444", "m₁g", 35); // m1 중력
        
        drawMoveVector(rightX, pulley.y + L2, 0, -tension * tScale, "#10b981", "T", 35); // m2 장력
        drawMoveVector(rightX, pulley.y + L2 + h2, 0, m2 * g * gScale, "#ef4444", "m₂g", 35); // m2 중력
    }

    // 속도 벡터
    if (Math.abs(v) > 0.01) {
        drawMoveVector(leftX - w1/2 - 15, pulley.y + L1 + h1/2, 0, -v * 15, "gray", "v₁", 15);
        drawMoveVector(rightX + w2/2 + 15, pulley.y + L2 + h2/2, 0, v * 15, "gray", "v₂", 15);
    }

    requestAnimationFrame(draw);
  }

  // 벡터 드로잉 함수 (머리 크기 10px 고정)
  function drawMoveVector(x, y, v_x, v_y, color, label, minLen = 0) {
    if (Math.abs(v_x) < 0.001 && Math.abs(v_y) < 0.001) return;
    ctx.save();
    
    let realLen = Math.hypot(v_x, v_y);
    let drawLen = Math.max(realLen, minLen);
    const angle = Math.atan2(v_y, v_x);
    
    const drawX = x + drawLen * Math.cos(angle);
    const drawY = y + drawLen * Math.sin(angle);
    const headLen = 10; // 화살표 머리 크기 고정

    ctx.strokeStyle = color;
    ctx.lineWidth = label.includes("v") ? 2 : 2.5;
    ctx.beginPath(); ctx.moveTo(x, y); ctx.lineTo(drawX, drawY); ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(drawX, drawY);
    ctx.lineTo(drawX - headLen * Math.cos(angle - Math.PI / 7), drawY - headLen * Math.sin(angle - Math.PI / 7));
    ctx.lineTo(drawX - headLen * Math.cos(angle + Math.PI / 7), drawY - headLen * Math.sin(angle + Math.PI / 7));
    ctx.closePath();
    ctx.fillStyle = color; ctx.fill();

    // 라벨 출력
    const txtOff = 18;
    const tx = drawX + txtOff * Math.cos(angle), ty = drawY + txtOff * Math.sin(angle);
    ctx.textAlign = "center"; ctx.textBaseline = "middle";
    ctx.font = "bold 14px 'Times New Roman', serif";
    ctx.strokeStyle = "#ffffff"; ctx.lineWidth = 3;
    ctx.strokeText(label, tx, ty);
    ctx.fillStyle = color; ctx.fillText(label, tx, ty);

    ctx.restore();
  }

  // 마우스 인터랙션 (사각형 기반 판정)
  function getPos(e) { const r = canvas.getBoundingClientRect(); return { x: e.clientX - r.left, y: e.clientY - r.top }; }

  canvas.onmousedown = (e) => {
    if (!isReset || isMoving) return;
    const m = getPos(e);
    const w1 = 30 + m1 * 2.5, h1 = 30 + m1 * 2.5;
    const w2 = 30 + m2 * 2.5, h2 = 30 + m2 * 2.5;
    
    // m1 사각형 범위 판정
    if (m.x > (pulley.x - pulley.r - w1/2) && m.x < (pulley.x - pulley.r + w1/2) &&
        m.y > (pulley.y + L1) && m.y < (pulley.y + L1 + h1)) {
        isDrag1 = true;
    } 
    // m2 사각형 범위 판정
    else if (m.x > (pulley.x + pulley.r - w2/2) && m.x < (pulley.x + pulley.r + w2/2) &&
             m.y > (pulley.y + L2) && m.y < (pulley.y + L2 + h2)) {
        isDrag2 = true;
    }
  };

  window.onmousemove = (e) => {
    if (!isDrag1 && !isDrag2) return;
    const m = getPos(e);
    const minL = 40, maxL = stringTotalLen - 40;

    if (isDrag1) { 
        L1 = Math.max(minL, Math.min(maxL, m.y - pulley.y)); 
        L2 = stringTotalLen - L1; 
    }
    else if (isDrag2) { 
        L2 = Math.max(minL, Math.min(maxL, m.y - pulley.y)); 
        L1 = stringTotalLen - L2; 
    }
    init_L1 = L1; init_L2 = L2;
  };

  window.onmouseup = () => { isDrag1 = isDrag2 = false; };
  
  canvas.onmousemove = (e) => {
    if(isMoving) return;
    const m = getPos(e);
    const w1 = 30 + m1 * 2.5, h1 = 30 + m1 * 2.5;
    const w2 = 30 + m2 * 2.5, h2 = 30 + m2 * 2.5;
    
    const over1 = (m.x > (pulley.x - pulley.r - w1/2) && m.x < (pulley.x - pulley.r + w1/2) &&
                   m.y > (pulley.y + L1) && m.y < (pulley.y + L1 + h1));
    const over2 = (m.x > (pulley.x + pulley.r - w2/2) && m.x < (pulley.x + pulley.r + w2/2) &&
                   m.y > (pulley.y + L2) && m.y < (pulley.y + L2 + h2));
    
    canvas.style.cursor = (over1 || over2) ? (isDrag1 || isDrag2 ? 'grabbing' : 'grab') : 'default';
  };

  draw();
</script>