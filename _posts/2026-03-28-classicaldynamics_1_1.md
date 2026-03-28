---
layout: post
title: "물리학"
koreantitle: 등속직선운동
englishtitle: uniform linear motion
info: 흠
color: "#8354ce"
permalink: /classicaldynamics_1_1_/
---

<canvas id="motionCanvas" width="600" height="200" style="border:1px solid #ddd; background: #f9f9f9;"></canvas>
<script>
  const canvas = document.getElementById('motionCanvas');
  const ctx = canvas.getContext('2d');
  let posX = 0;       // 시작 X 좌표
  const posY = 100;    // Y 좌표 (고정)
  const velocity = 2;  // 속도 (프레임당 이동 픽셀)
  const radius = 15;   // 물체(원)의 반지름
  function draw() {
    // 1. 화면 지우기 (이전 프레임 잔상 제거)
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    // 2. 물체 그리기
    ctx.beginPath();
    ctx.arc(posX, posY, radius, 0, Math.PI * 2);
    ctx.fillStyle = "#007bff";
    ctx.fill();
    ctx.closePath();
    // 3. 위치 업데이트 (등속 운동)
    posX += velocity;
    // 4. 화면 끝에 도달하면 처음으로 리셋 (무한 반복용)
    if (posX > canvas.width + radius) {
      posX = -radius;
    }
    // 다음 프레임 요청
    requestAnimationFrame(draw);
  }

  draw();
</script>