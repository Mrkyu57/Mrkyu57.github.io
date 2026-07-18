---
layout: post
title: "해석학"
koreantitle: 해석학
englishtitle: analysis
info: 흠
color: "#5489ce"
permalink: /math_mainpost_analysis/
---

<div id="euclid-container" style="margin: 20px 0; text-align: center; font-family: 'Times New Roman', serif; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); overflow: hidden;">
    
<div style="margin-bottom: 15px;">
        <h3 id="stepTitle" style="margin: 0; color: #333;">단계 0: 선분 AB</h3>
        <p id="stepDesc" style="margin: 5px 0; color: #666; font-size: 0.95em;">주어진 선분 AB를 기준으로 작도를 시작합니다.</p>
</div>

<div style="margin-bottom: 20px; display: flex; align-items: center; justify-content: center; gap: 15px;">
        <span style="font-weight: bold; font-size: 0.9em;">Step</span>
        <input type="range" id="stepSlider" min="0" max="5" value="0" style="width: 200px; cursor: pointer; accent-color: #8354ce;">
        <span id="stepValue" style="font-weight: bold; color: #8354ce; width: 20px;">0</span>
</div>

<canvas id="euclidCanvas" width="800" height="450" style="border:1px solid #eee; background: #fdfdfd; display: block; margin: 0 auto; touch-action: none; max-width: 100%; height: auto; border-radius: 8px; cursor: crosshair;"></canvas>
    
<p style="color: #999; font-size: 0.85em; margin-top: 15px;">※ 점 A와 B를 드래그하여 작도를 조작해보세요.</p>
</div>

<script>
(function() {
    const cvs = document.getElementById('euclidCanvas');
    const ctx = cvs.getContext('2d');
    const slider = document.getElementById('stepSlider');
    const stepValTxt = document.getElementById('stepValue');
    const stepTitle = document.getElementById('stepTitle');
    const stepDesc = document.getElementById('stepDesc');

    let ptA = { x: 300, y: 280 };
    let ptB = { x: 500, y: 280 };
    let dragTarget = null;
    let currentStep = 0;

    const stepInfo = [
        { title: "단계 0: 선분 AB", desc: "기준이 되는 선분 AB를 설정합니다." },
        { title: "STEP 1: Postulates III", desc: "점 A,B에 대해 A를 중심으로 하고 반지름이 AB인 원을 그릴 수 있다." },
        { title: "STEP 2: Postulates III", desc: "점 A,B에 대해 B를 중심으로 하고 반지름이 BA인 원을 그릴 수 있다." },
        { title: "STEP 3: ???", desc: "두 원의 교점을 점 C라고 정의합니다." },
        { title: "STEP 4: Postulates I", desc: "각 점을 연결하여 정삼각형의 형태를 만듭니다." },
        { title: "STEP 5: 작도 완료", desc: "세 변의 길이가 같은 정삼각형 ABC가 완성되었습니다." }
    ];

    slider.oninput = function() {
        currentStep = parseInt(this.value);
        stepValTxt.innerText = currentStep;
        stepTitle.innerText = stepInfo[currentStep].title;
        stepDesc.innerText = stepInfo[currentStep].desc;
        draw();
    };

    function getDistance(p1, p2) {
        return Math.hypot(p2.x - p1.x, p2.y - p1.y);
    }

    function getPointC() {
        const d = getDistance(ptA, ptB);
        if (d === 0) return { ...ptA };
        const mx = (ptA.x + ptB.x) / 2;
        const my = (ptA.y + ptB.y) / 2;
        const h = d * Math.sqrt(3) / 2;
        const nx = -(ptB.y - ptA.y) / d;
        const ny = (ptB.x - ptA.x) / d;
        return { x: mx - nx * h, y: my - ny * h };
    }

    // --- 보조 함수 (작대기 그리기용) ---
    function markOffset(length, angle) {
        return { x: length * Math.cos(angle), y: length * Math.sin(angle) };
    }
    function gapOffset(gap, angle) {
        return { x: gap * Math.cos(angle), y: gap * Math.sin(angle) };
    }

    function drawEqualityMark(p1, p2) {
        const midX = (p1.x + p2.x) / 2;
        const midY = (p1.y + p2.y) / 2;
        const angle = Math.atan2(p2.y - p1.y, p2.x - p1.x);
        const markLength = 8; 
        const markGap = 4;    
        const markAngle = angle + Math.PI / 2;

        ctx.save();
        ctx.strokeStyle = "#8354ce";
        ctx.lineWidth = 2;
        ctx.setLineDash([]);

        for (let i = -1; i <= 1; i += 2) {
            const shift = gapOffset((markGap / 2) * i, angle);
            const edge = markOffset(markLength / 2, markAngle);
            ctx.beginPath();
            ctx.moveTo(midX + shift.x + edge.x, midY + shift.y + edge.y);
            ctx.lineTo(midX + shift.x - edge.x, midY + shift.y - edge.y);
            ctx.stroke();
        }
        ctx.restore();
    }

    function draw() {
        ctx.clearRect(0, 0, cvs.width, cvs.height);
        const radius = getDistance(ptA, ptB);
        const ptC = getPointC();

        // 1. 보조 원 (1~4단계)
        if (currentStep >= 1 && currentStep < 6) {
            ctx.setLineDash([5, 5]);
            ctx.lineWidth = 1;
            ctx.strokeStyle = "rgba(255, 99, 132, 0.4)";
            ctx.beginPath(); ctx.arc(ptA.x, ptA.y, radius, 0, Math.PI * 2); ctx.stroke();
            if (currentStep >= 2) {
                ctx.strokeStyle = "rgba(54, 162, 235, 0.4)";
                ctx.beginPath(); ctx.arc(ptB.x, ptB.y, radius, 0, Math.PI * 2); ctx.stroke();
            }
            ctx.setLineDash([]);
        }

        // 2. 삼각형 및 선분
        ctx.lineWidth = 2.5;
        ctx.strokeStyle = "#333";
        ctx.beginPath();
        ctx.moveTo(ptA.x, ptA.y);
        ctx.lineTo(ptB.x, ptB.y);
        ctx.stroke();

        if (currentStep >= 5) {
            ctx.beginPath();
            ctx.moveTo(ptA.x, ptA.y);
            ctx.lineTo(ptC.x, ptC.y);
            ctx.lineTo(ptB.x, ptB.y);
            ctx.closePath();
            if (currentStep === 5) {
                ctx.fillStyle = "rgba(131, 84, 206, 0.1)";
                ctx.fill();
                ctx.strokeStyle = "#8354ce";
                ctx.lineWidth = 3.5;
            } else {
                ctx.strokeStyle = "#8354ce";
            }
            ctx.stroke();

            // 같다 기호 표시
            drawEqualityMark(ptA, ptB);
            drawEqualityMark(ptA, ptC);
            drawEqualityMark(ptB, ptC);
        }

        // 3. 점 및 라벨
        const drawPt = (p, label, color) => {
            ctx.beginPath(); ctx.arc(p.x, p.y, 6, 0, Math.PI * 2);
            ctx.fillStyle = color; ctx.fill();
            ctx.strokeStyle = "#fff"; ctx.lineWidth = 2; ctx.stroke();
            ctx.fillStyle = "#333"; ctx.font = "bold 16px serif";
            ctx.fillText(label, p.x - 8, p.y - 15);
        };
        drawPt(ptA, "A", "#ff6384");
        drawPt(ptB, "B", "#36a2eb");
        if (currentStep >= 3) drawPt(ptC, "C", "#8354ce");
    }

    // 인터랙션 핸들러
    function getPos(e) {
        const rect = cvs.getBoundingClientRect();
        const clientX = e.touches ? e.touches[0].clientX : e.clientX;
        const clientY = e.touches ? e.touches[0].clientY : e.clientY;
        return {
            x: (clientX - rect.left) * (cvs.width / rect.width),
            y: (clientY - rect.top) * (cvs.height / rect.height)
        };
    }

    function moveStart(e) {
        const pos = getPos(e);
        if (getDistance(pos, ptA) < 30) dragTarget = 'A';
        else if (getDistance(pos, ptB) < 30) dragTarget = 'B';
        if (dragTarget && e.cancelable) e.preventDefault();
    }

    function moveAction(e) {
        if (!dragTarget) return;
        const pos = getPos(e);
        if (dragTarget === 'A') { ptA.x = pos.x; ptA.y = pos.y; }
        else { ptB.x = pos.x; ptB.y = pos.y; }
        draw();
        if (e.cancelable) e.preventDefault();
    }

    cvs.addEventListener('mousedown', moveStart);
    window.addEventListener('mousemove', moveAction);
    window.addEventListener('mouseup', () => dragTarget = null);
    cvs.addEventListener('touchstart', moveStart, { passive: false });
    window.addEventListener('touchmove', moveAction, { passive: false });
    window.addEventListener('touchend', () => dragTarget = null);

    draw();
})();
</script>