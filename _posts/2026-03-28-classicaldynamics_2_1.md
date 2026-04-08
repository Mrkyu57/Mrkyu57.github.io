---
layout: post
title: "물리학"
koreantitle: 등속직선운동
englishtitle: uniform linear motion
info: 흠
color: "#8354ce"
permalink: /classicaldynamics_2_1_/
---

<div style="text-align: center; font-family: sans-serif;">
  <h3>바닥 위에 서 있는 물체 모델링</h3>
  <p>바닥 Y 좌표와 물체의 크기를 정의하여 시각화합니다.</p>
  
  <canvas id="modelingCanvas" width="800" height="400" style="border:1px solid #ddd; background: #fdfdfd;"></canvas>
</div>

<script>
  const canvas = document.getElementById('modelingCanvas');
  const ctx = canvas.getContext('2d');

  // 1. 환경 및 물체 정의 (모델링 매개변수)
  const floor = {
    y: 350,       // 바닥의 Y 좌표 (화면 상단에서 350px 아래)
    color: '#666', // 바닥 선 색상
    lineWidth: 3
  };

  const object = {
    x: canvas.width / 2 - 50, // 화면 중앙
    y: 0,                   // 초기 Y (바닥 위에 서도록 계산 예정)
    width: 100,
    height: 120,
    color: '#ffc107',        // 주황색
    borderColor: '#e0a800',
    legs: {
      width: 15,
      height: 20
    }
  };

  // 물체의 Y 좌표를 계산하여 바닥 위에 '서 있게' 만듦
  // 물체의 하단(y + height)이 바닥(floor.y)에 닿아야 함
  object.y = floor.y - object.height;

  function draw() {
    // 1. 화면 지우기
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 2. 바닥 그리기
    ctx.beginPath();
    ctx.moveTo(0, floor.y);
    ctx.lineTo(canvas.width, floor.y);
    ctx.strokeStyle = floor.color;
    ctx.lineWidth = floor.lineWidth;
    ctx.stroke();
    ctx.closePath();

    // 3. 물체 그리기 (상자 본체)
    ctx.fillStyle = object.color;
    ctx.fillRect(object.x, object.y, object.width, object.height);
    ctx.strokeStyle = object.borderColor;
    ctx.lineWidth = 2;
    ctx.strokeRect(object.x, object.y, object.width, object.height);

    // 4. 물체 다리 그리기 ('서 있는' 느낌 강조)
    ctx.fillStyle = object.borderColor;
    // 왼쪽 다리
    ctx.fillRect(object.x + 15, floor.y - object.legs.height, object.legs.width, object.legs.height);
    // 오른쪽 다리
    ctx.fillRect(object.x + object.width - 15 - object.legs.width, floor.y - object.legs.height, object.legs.width, object.legs.height);


    // 5. 모델링 정보 텍스트 (시각적 피드백)
    ctx.fillStyle = '#333';
    ctx.font = '14px sans-serif';
    ctx.textAlign = 'left';
    ctx.fillText(`Floor Y: ${floor.y} px`, 10, 20);
    ctx.fillText(`Object Pos: (${object.x}, ${object.y}) px`, 10, 40);
    ctx.fillText(`Object Size: ${object.width} x ${object.height} px`, 10, 60);
    
    // 물체 하단 표시
    ctx.beginPath();
    ctx.arc(object.x + object.width / 2, floor.y, 5, 0, Math.PI * 2);
    ctx.fillStyle = 'red';
    ctx.fill();
    ctx.closePath();
    ctx.fillStyle = 'red';
    ctx.fillText('Ground Contact Point', object.x + object.width / 2 + 10, floor.y + 5);
  }

  // 초기 그리기 호출
  draw();
</script>