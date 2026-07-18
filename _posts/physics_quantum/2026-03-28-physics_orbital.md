---
layout: post
title: "물리학"
koreantitle: 수소 원자 양자 모델링
englishtitle: Hydrogen Atom Quantum Model
info: 흠
color: "#8354ce"
permalink: /physics_quantum_hydrogen_1_1/
---

<!-- 모델링 공간 -->
<div id="quantum-model-container" style="
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
    <!-- 제어 영역 -->
    <div style="
        padding: 15px 20px; 
        background: #fcfcfc; 
        border-bottom: 1px solid #f0f0f0; 
        display: flex; 
        flex-wrap: wrap;
        justify-content: center; 
        align-items: center;
        gap: 15px;
    ">
        <h3 style="margin: 0; color: #333; font-family: 'Times New Roman', serif; font-size: 1.1rem; min-width: 120px; text-align: center;">
            Hydrogen Atom
        </h3>
        <!-- 양자수 선택기 -->
        <div style="display: flex; gap: 10px; align-items: center; font-size: 0.9rem; font-weight: bold; color: #475569;">
            <label>n: 
                <select id="quantum-n" style="padding: 4px; border-radius: 4px; border: 1px solid #ccc;">
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>
            </label>
            <label>l: 
                <select id="quantum-l" style="padding: 4px; border-radius: 4px; border: 1px solid #ccc;">
                    <option value="0">0 (s)</option>
                </select>
            </label>
            <label>m: 
                <select id="quantum-m" style="padding: 4px; border-radius: 4px; border: 1px solid #ccc;">
                    <option value="0">0</option>
                </select>
            </label>
        </div>
        <button id="generate-btn" style="padding: 0 16px; height: 36px; cursor: pointer; background: #8354ce; color: white; border: none; border-radius: 6px; font-weight: 600; font-size: 0.85rem; transition: 0.2s;">
            궤도 렌더링
        </button>
    </div>
    <!-- 캔버스 영역 (반응형 지원 및 터치 제어) -->
    <div style="position: relative; width: 100%; aspect-ratio: 16/9; background: #ffffff; touch-action: none;">
        <canvas id="quantumCanvas" style="
            display: block;
            width: 100%;
            height: 100%;
            cursor: grab;
        "></canvas>
        <div style="position: absolute; bottom: 10px; right: 10px; color: rgba(255,255,255,0.5); font-size: 12px; pointer-events: none;">
            드래그/터치하여 회전
        </div>
    </div>
</div>

<div style="text-align: center; font: bold 18px 'Times New Roman', serif; margin-top: 20px;">
  <h3>Schrödinger Equation</h3>
  <p>주양자수(n), 방위양자수(l), 자기양자수(m)에 따른 확률 밀도 시각화</p>
</div>

<script>
    const canvas = document.getElementById('quantumCanvas');
    const ctx = canvas.getContext('2d');
    
    // UI Elements
    const selectN = document.getElementById('quantum-n');
    const selectL = document.getElementById('quantum-l');
    const selectM = document.getElementById('quantum-m');
    const generateBtn = document.getElementById('generate-btn');

    let width, height;
    let particles = [];
    
    // 회전 각도 (마우스 및 터치 제어용)
    let angleX = 0;
    let angleY = 0;
    
    // 크기 조절 스케일
    let scale = 30; 

    // 반응형 캔버스 리사이징
    function resizeCanvas() {
        width = canvas.clientWidth;
        height = canvas.clientHeight;
        canvas.width = width;
        canvas.height = height;
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // 양자수 제어 로직 (n에 따라 l과 m의 범위 제한)
    function updateSelectors() {
        const n = parseInt(selectN.value);
        
        // l 업데이트 (0 부터 n-1)
        selectL.innerHTML = '';
        const lLetters = ['s', 'p', 'd'];
        for (let l = 0; l < n; l++) {
            const opt = document.createElement('option');
            opt.value = l;
            opt.text = `${l} (${lLetters[l]})`;
            selectL.appendChild(opt);
        }
        
        updateMSelector();
    }

    function updateMSelector() {
        const l = parseInt(selectL.value);
        selectM.innerHTML = '';
        for (let m = -l; m <= l; m++) {
            const opt = document.createElement('option');
            opt.value = m;
            opt.text = m;
            selectM.appendChild(opt);
        }
    }

    selectN.addEventListener('change', updateSelectors);
    selectL.addEventListener('change', updateMSelector);

    // 파동함수 확률 밀도 계산 (Simplified Hydrogen Atom Orbitals up to n=3)
    function getProbability(x, y, z, n, l, m) {
        const r = Math.sqrt(x*x + y*y + z*z);
        if (r === 0 && l === 0) return 1;
        if (r === 0) return 0;

        let R = 0;
        let Y = 0;

        // 방사 방향 부분 R(r)
        if (n === 1 && l === 0) R = Math.exp(-r);
        else if (n === 2 && l === 0) R = (2 - r) * Math.exp(-r/2);
        else if (n === 2 && l === 1) R = r * Math.exp(-r/2);
        else if (n === 3 && l === 0) R = (27 - 18*r + 2*r*r) * Math.exp(-r/3);
        else if (n === 3 && l === 1) R = (6 - r) * r * Math.exp(-r/3);
        else if (n === 3 && l === 2) R = r * r * Math.exp(-r/3);

        // 각도 방향 부분 Y (Real spherical harmonics)
        const theta = Math.acos(z/r);
        const phi = Math.atan2(y, x);

        if (l === 0) {
            Y = 1;
        } else if (l === 1) {
            if (m === 0) Y = Math.cos(theta); // pz
            else if (m === 1) Y = Math.sin(theta) * Math.cos(phi); // px
            else if (m === -1) Y = Math.sin(theta) * Math.sin(phi); // py
        } else if (l === 2) {
            if (m === 0) Y = 3 * Math.pow(Math.cos(theta), 2) - 1; // dz2
            else if (m === 1) Y = Math.sin(theta) * Math.cos(theta) * Math.cos(phi); // dxz
            else if (m === -1) Y = Math.sin(theta) * Math.cos(theta) * Math.sin(phi); // dyz
            else if (m === 2) Y = Math.pow(Math.sin(theta), 2) * Math.cos(2*phi); // dx2-y2
            else if (m === -2) Y = Math.pow(Math.sin(theta), 2) * Math.sin(2*phi); // dxy
        }

        return (R * R) * (Y * Y);
    }

    // 전자 구름(포인트 클라우드) 생성
    function generateCloud() {
        const n = parseInt(selectN.value);
        const l = parseInt(selectL.value);
        const m = parseInt(selectM.value);
        
        particles = [];
        const maxR = n * n * 4; // 구름이 퍼지는 최대 반경 (휴리스틱)
        scale = (width / 3) / maxR; // 줌 스케일 자동 조절

        const numParticles = 8000;
        let attempts = 0;

        while (particles.length < numParticles && attempts < 100000) {
            const x = (Math.random() * 2 - 1) * maxR;
            const y = (Math.random() * 2 - 1) * maxR;
            const z = (Math.random() * 2 - 1) * maxR;
            
            const prob = getProbability(x, y, z, n, l, m);
            // Rejection Sampling
            if (Math.random() < prob * 0.5) { 
                // 확률이 높은 곳일수록 밝게 처리
                const brightness = Math.min(1, prob * 2);
                particles.push({
                    x: x, y: y, z: z,
                    alpha: 0.1 + brightness * 0.9
                });
            }
            attempts++;
        }
    }

    generateBtn.addEventListener('click', generateCloud);

    // 3D 회전 행렬 적용
    function rotate3D(x, y, z) {
        // Y축 회전
        let x1 = x * Math.cos(angleY) - z * Math.sin(angleY);
        let z1 = x * Math.sin(angleY) + z * Math.cos(angleY);
        // X축 회전
        let y2 = y * Math.cos(angleX) - z1 * Math.sin(angleX);
        let z2 = y * Math.sin(angleX) + z1 * Math.cos(angleX);
        return { x: x1, y: y2, z: z2 };
    }

    // 렌더링 루프
    function draw() {
        ctx.clearRect(0, 0, width, height);

        const centerX = width / 2;
        const centerY = height / 2;

        // 원점과 축 그리기
        ctx.strokeStyle = "rgba(255, 255, 255, 0.2)";
        ctx.beginPath();
        ctx.moveTo(centerX - 50, centerY); ctx.lineTo(centerX + 50, centerY);
        ctx.moveTo(centerX, centerY - 50); ctx.lineTo(centerX, centerY + 50);
        ctx.stroke();

        ctx.fillStyle = "#a855f7"; // 전자 점 색상 설정

        // 깊이(Z축)를 기준으로 정렬하여 렌더링 (원근감 부여)
        const projectedParticles = particles.map(p => {
            const rot = rotate3D(p.x, p.y, p.z);
            return {
                drawX: centerX + rot.x * scale,
                drawY: centerY + rot.y * scale,
                z: rot.z,
                alpha: p.alpha
            };
        });

        projectedParticles.sort((a, b) => a.z - b.z);

        projectedParticles.forEach(p => {
            ctx.globalAlpha = p.alpha;
            // Z축 거리에 따른 크기 변화 (약간의 원근감)
            const size = Math.max(0.5, 2 + (p.z / (scale * 2))); 
            
            ctx.beginPath();
            ctx.arc(p.drawX, p.drawY, size, 0, Math.PI * 2);
            ctx.fill();
        });

        ctx.globalAlpha = 1.0;
        requestAnimationFrame(draw);
    }

    // 드래그 및 터치 이벤트 처리 (회전 로직)
    let isDragging = false;
    let lastMouseX = 0;
    let lastMouseY = 0;

    function startInteraction(x, y) {
        isDragging = true;
        lastMouseX = x;
        lastMouseY = y;
        canvas.style.cursor = 'grabbing';
    }

    function moveInteraction(x, y) {
        if (!isDragging) return;
        const deltaX = x - lastMouseX;
        const deltaY = y - lastMouseY;

        angleY += deltaX * 0.01;
        angleX += deltaY * 0.01;

        lastMouseX = x;
        lastMouseY = y;
    }

    function endInteraction() {
        isDragging = false;
        canvas.style.cursor = 'grab';
    }

    // 마우스 이벤트
    canvas.addEventListener('mousedown', (e) => startInteraction(e.clientX, e.clientY));
    window.addEventListener('mousemove', (e) => moveInteraction(e.clientX, e.clientY));
    window.addEventListener('mouseup', endInteraction);

    // 터치 이벤트 (모바일 대응)
    canvas.addEventListener('touchstart', (e) => {
        e.preventDefault(); // 스크롤 방지
        const touch = e.touches[0];
        startInteraction(touch.clientX, touch.clientY);
    });
    canvas.addEventListener('touchmove', (e) => {
        e.preventDefault();
        const touch = e.touches[0];
        moveInteraction(touch.clientX, touch.clientY);
    });
    window.addEventListener('touchend', endInteraction);

    // 초기 실행
    updateSelectors();
    generateCloud();
    draw();

</script>