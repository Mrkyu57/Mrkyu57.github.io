<div id="euclid-container-2" style="margin: 20px 0; text-align: center; font-family: 'Times New Roman', serif; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); overflow: hidden;">
    <div style="margin-bottom: 15px;">
        <h3 id="stepTitle2" style="margin: 0; color: #333;">단계 0: 점 A와 선분 BC</h3>
        <p id="stepDesc2" style="margin: 5px 0; color: #666; font-size: 0.95em;">주어진 점 A와 기준이 되는 선분 BC를 확인합니다.</p>
    </div>
    <div style="margin-bottom: 20px; display: flex; align-items: center; justify-content: center; gap: 15px;">
        <span style="font-weight: bold; font-size: 0.9em;">Step</span>
        <input type="range" id="stepSlider2" min="0" max="6" value="0" style="width: 250px; cursor: pointer; accent-color: #8354ce;">
        <span id="stepValue2" style="font-weight: bold; color: #8354ce; width: 20px;">0</span>
    </div>
    <canvas id="euclidCanvas2" width="800" height="600" style="border:1px solid #eee; background: #fdfdfd; display: block; margin: 0 auto; touch-action: none; max-width: 100%; height: auto; border-radius: 8px; cursor: crosshair;"></canvas>
    <p style="color: #999; font-size: 0.85em; margin-top: 15px;">※ 점 A, B, C를 드래그하여 작도를 조작해보세요.</p>
</div>

<script>
(function() {
    const cvs = document.getElementById('euclidCanvas2');
    const ctx = cvs.getContext('2d');
    const slider = document.getElementById('stepSlider2');
    const stepValTxt = document.getElementById('stepValue2');
    const stepTitle = document.getElementById('stepTitle2');
    const stepDesc = document.getElementById('stepDesc2');

    // 초기 좌표 설정
    let ptA = { x: 300, y: 300 }; // 주어진 점
    let ptB = { x: 450, y: 300 }; // 주어진 선분의 한 끝점
    let ptC = { x: 500, y: 400 }; // 주어진 선분의 다른 끝점
    
    let dragTarget = null;
    let currentStep = 0;

    const stepInfo = [
        { title: "단계 0: 점 A와 선분 BC", desc: "주어진 점 A에서 선분 BC와 길이가 같은 선분을 작도할 것입니다." },
        { title: "STEP 1: 선분 AB (공리 1)", desc: "점 A와 점 B를 연결하여 선분 AB를 만듭니다." },
        { title: "STEP 2: 정삼각형 작도 (명제 1)", desc: "선분 AB를 한 변으로 하는 정삼각형 ABD를 작도합니다." },
        { title: "STEP 3: 선분 연장 (공리 2)", desc: "선분 DA와 DB를 A와 B 방향으로 충분히 길게 연장합니다." },
        { title: "STEP 4: 원 B 작도 (공리 3)", desc: "점 B를 중심으로 반지름이 BC인 원을 그려 연장선과의 교점 G를 찾습니다." },
        { title: "STEP 5: 원 D 작도 (공리 3)", desc: "점 D를 중심으로 반지름이 DG인 원을 그려 다른 연장선과의 교점 L을 찾습니다." },
        { title: "STEP 6: 작도 완료", desc: "DL=DG, DA=DB이므로 AL=BG가 됩니다. 즉, 선분 AL은 선분 BC와 길이가 같습니다!" }
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

    function getVec(p1, p2) {
        const d = getDistance(p1, p2);
        if (d === 0) return { x: 0, y: 0 };
        return { x: (p2.x - p1.x) / d, y: (p2.y - p1.y) / d };
    }

    // 정삼각형의 꼭짓점 D 구하기
    function getPointD() {
        const d = getDistance(ptA, ptB);
        if (d === 0) return { ...ptA };
        const mx = (ptA.x + ptB.x) / 2;
        const my = (ptA.y + ptB.y) / 2;
        const h = d * Math.sqrt(3) / 2;
        const nx = -(ptB.y - ptA.y) / d;
        const ny = (ptB.x - ptA.x) / d;
        // 작도 공간 확보를 위해 위쪽으로 뻗어나가도록 설정
        return { x: mx - nx * h, y: my - ny * h };
    }

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
        
        const ptD = getPointD();
        const rBC = getDistance(ptB, ptC);
        
        // 방향 벡터 계산
        const uDA = getVec(ptD, ptA);
        const uDB = getVec(ptD, ptB);
        
        // 교점 G와 L 계산
        const ptG = { x: ptB.x + uDB.x * rBC, y: ptB.y + uDB.y * rBC };
        const rDG = getDistance(ptD, ptG);
        const ptL = { x: ptD.x + uDA.x * rDG, y: ptD.y + uDA.y * rDG };

        // 0. 기본 제공되는 선분 BC (항상 표시)
        ctx.lineWidth = (currentStep === 6) ? 3.5 : 2.5;
        ctx.strokeStyle = (currentStep === 6) ? "#8354ce" : "#333";
        ctx.beginPath();
        ctx.moveTo(ptB.x, ptB.y);
        ctx.lineTo(ptC.x, ptC.y);
        ctx.stroke();
        if (currentStep === 6) drawEqualityMark(ptB, ptC);

        // 1. 선분 AB
        if (currentStep >= 1) {
            ctx.lineWidth = 2.5;
            ctx.strokeStyle = "#333";
            ctx.beginPath();
            ctx.moveTo(ptA.x, ptA.y);
            ctx.lineTo(ptB.x, ptB.y);
            ctx.stroke();
        }

        // 2. 정삼각형 ABD
        if (currentStep >= 2) {
            ctx.beginPath();
            ctx.moveTo(ptA.x, ptA.y);
            ctx.lineTo(ptD.x, ptD.y);
            ctx.lineTo(ptB.x, ptB.y);
            ctx.stroke();
        }

        // 3. 선분 연장선 (반직선)
        if (currentStep >= 3) {
            const extLength = Math.max(rDG + 50, 200);
            ctx.setLineDash([4, 4]);
            ctx.lineWidth = 1.5;
            ctx.strokeStyle = "#999";
            
            // DA 연장
            ctx.beginPath();
            ctx.moveTo(ptA.x, ptA.y);
            ctx.lineTo(ptD.x + uDA.x * extLength, ptD.y + uDA.y * extLength);
            ctx.stroke();
            
            // DB 연장
            ctx.beginPath();
            ctx.moveTo(ptB.x, ptB.y);
            ctx.lineTo(ptD.x + uDB.x * extLength, ptD.y + uDB.y * extLength);
            ctx.stroke();
            ctx.setLineDash([]);
        }

        // 4. 점 B 중심 원 (반지름 BC)
        if (currentStep >= 4) {
            ctx.setLineDash([5, 5]);
            ctx.lineWidth = 1;
            ctx.strokeStyle = "rgba(54, 162, 235, 0.6)";
            ctx.beginPath();
            ctx.arc(ptB.x, ptB.y, rBC, 0, Math.PI * 2);
            ctx.stroke();
            ctx.setLineDash([]);
        }

        // 5. 점 D 중심 원 (반지름 DG)
        if (currentStep >= 5) {
            ctx.setLineDash([5, 5]);
            ctx.lineWidth = 1;
            ctx.strokeStyle = "rgba(255, 99, 132, 0.6)";
            ctx.beginPath();
            ctx.arc(ptD.x, ptD.y, rDG, 0, Math.PI * 2);
            ctx.stroke();
            ctx.setLineDash([]);
        }

        // 6. 결과 선분 AL 강조
        if (currentStep >= 6) {
            ctx.lineWidth = 3.5;
            ctx.strokeStyle = "#8354ce";
            ctx.beginPath();
            ctx.moveTo(ptA.x, ptA.y);
            ctx.lineTo(ptL.x, ptL.y);
            ctx.stroke();
            drawEqualityMark(ptA, ptL);
        }

        // 7. 점 및 라벨 표시
        const drawPt = (p, label, color) => {
            ctx.beginPath(); ctx.arc(p.x, p.y, 6, 0, Math.PI * 2);
            ctx.fillStyle = color; ctx.fill();
            ctx.strokeStyle = "#fff"; ctx.lineWidth = 2; ctx.stroke();
            ctx.fillStyle = "#333"; ctx.font = "bold 16px serif";
            ctx.fillText(label, p.x - 8, p.y - 15);
        };

        drawPt(ptA, "A", "#ff6384");
        drawPt(ptB, "B", "#36a2eb");
        drawPt(ptC, "C", "#ffce56"); // 새로운 점 C
        
        if (currentStep >= 2) drawPt(ptD, "D", "#4bc0c0");
        if (currentStep >= 4) drawPt(ptG, "G", "#36a2eb");
        if (currentStep >= 5) drawPt(ptL, "L", "#8354ce");
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
        else if (getDistance(pos, ptC) < 30) dragTarget = 'C';
        if (dragTarget && e.cancelable) e.preventDefault();
    }

    function moveAction(e) {
        if (!dragTarget) return;
        const pos = getPos(e);
        if (dragTarget === 'A') { ptA.x = pos.x; ptA.y = pos.y; }
        else if (dragTarget === 'B') { ptB.x = pos.x; ptB.y = pos.y; }
        else if (dragTarget === 'C') { ptC.x = pos.x; ptC.y = pos.y; }
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

<div id="euclid-container-3" style="margin: 20px 0; text-align: center; font-family: 'Times New Roman', serif; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); overflow: hidden;">
    <div style="margin-bottom: 15px;">
        <h3 id="stepTitle3" style="margin: 0; color: #333;">단계 0: 두 선분 AB와 CD</h3>
        <p id="stepDesc3" style="margin: 5px 0; color: #666; font-size: 0.95em;">길이가 다른 두 선분 AB와 CD가 주어집니다. (AB > CD라고 가정합니다)</p>
    </div>
    <div style="margin-bottom: 20px; display: flex; align-items: center; justify-content: center; gap: 15px;">
        <span style="font-weight: bold; font-size: 0.9em;">Step</span>
        <input type="range" id="stepSlider3" min="0" max="4" value="0" style="width: 250px; cursor: pointer; accent-color: #8354ce;">
        <span id="stepValue3" style="font-weight: bold; color: #8354ce; width: 20px;">0</span>
    </div>
    <canvas id="euclidCanvas3" width="800" height="450" style="border:1px solid #eee; background: #fdfdfd; display: block; margin: 0 auto; touch-action: none; max-width: 100%; height: auto; border-radius: 8px; cursor: crosshair;"></canvas>
    <p style="color: #999; font-size: 0.85em; margin-top: 15px;">※ 점 A, B, C, D를 드래그하여 작도를 조작해보세요.</p>
</div>

<script>
(function() {
    const cvs = document.getElementById('euclidCanvas3');
    const ctx = cvs.getContext('2d');
    const slider = document.getElementById('stepSlider3');
    const stepValTxt = document.getElementById('stepValue3');
    const stepTitle = document.getElementById('stepTitle3');
    const stepDesc = document.getElementById('stepDesc3');

    // 초기 좌표 설정 (AB가 더 긴 선분, CD가 짧은 선분)
    let ptA = { x: 200, y: 250 }; 
    let ptB = { x: 650, y: 250 }; 
    let ptC = { x: 100, y: 100 }; 
    let ptD = { x: 300, y: 100 }; 
    
    let dragTarget = null;
    let currentStep = 0;

    const stepInfo = [
        { title: "단계 0: 두 선분 AB와 CD", desc: "긴 선분 AB에서 짧은 선분 CD와 길이가 같은 선분을 잘라낼 것입니다." },
        { title: "STEP 1: 선분 AF 작도 (명제 2)", desc: "명제 2를 이용해 점 A에서 시작하고 CD와 길이가 같은 선분 AF를 만듭니다." },
        { title: "STEP 2: 원 작도 (공리 3)", desc: "점 A를 중심으로 하고 반지름이 AF(즉, CD)인 원을 그립니다." },
        { title: "STEP 3: 교점 E", desc: "그려진 원과 긴 선분 AB가 만나는 교점을 점 E라고 정의합니다." },
        { title: "STEP 4: 작도 완료", desc: "AE = AF, AF = CD이므로 AE = CD입니다. 선분 AB에서 CD만큼을 잘라냈습니다!" }
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

    function getVec(p1, p2) {
        const d = getDistance(p1, p2);
        if (d === 0) return { x: 0, y: 0 };
        return { x: (p2.x - p1.x) / d, y: (p2.y - p1.y) / d };
    }

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
        
        const lenCD = getDistance(ptC, ptD);
        const vecAB = getVec(ptA, ptB);
        
        // 점 F 계산 (명제 2를 통해 옮겨온 선분 AF, 시각적으로 AB 위쪽 각도로 배치)
        const angleAB = Math.atan2(vecAB.y, vecAB.x);
        const angleAF = angleAB - Math.PI / 3; 
        const ptF = { x: ptA.x + Math.cos(angleAF) * lenCD, y: ptA.y + Math.sin(angleAF) * lenCD };
        
        // 점 E 계산 (원과 선분 AB의 교점)
        const ptE = { x: ptA.x + vecAB.x * lenCD, y: ptA.y + vecAB.y * lenCD };

        // 0. 기본 제공 선분 CD
        ctx.lineWidth = 2.5;
        ctx.strokeStyle = (currentStep === 4) ? "#8354ce" : "#333";
        ctx.beginPath();
        ctx.moveTo(ptC.x, ptC.y);
        ctx.lineTo(ptD.x, ptD.y);
        ctx.stroke();

        // 0. 기본 제공 선분 AB
        ctx.lineWidth = 2.5;
        ctx.strokeStyle = "#333";
        ctx.beginPath();
        ctx.moveTo(ptA.x, ptA.y);
        ctx.lineTo(ptB.x, ptB.y);
        ctx.stroke();

        // 1. 선분 AF (명제 2로 복사한 선분)
        if (currentStep >= 1) {
            ctx.setLineDash([4, 4]);
            ctx.lineWidth = 2;
            ctx.strokeStyle = "#4bc0c0";
            ctx.beginPath();
            ctx.moveTo(ptA.x, ptA.y);
            ctx.lineTo(ptF.x, ptF.y);
            ctx.stroke();
            ctx.setLineDash([]);
        }

        // 2. 점 A 중심 원 (반지름 AF = CD)
        if (currentStep >= 2) {
            ctx.setLineDash([5, 5]);
            ctx.lineWidth = 1;
            ctx.strokeStyle = "rgba(255, 99, 132, 0.6)";
            ctx.beginPath();
            ctx.arc(ptA.x, ptA.y, lenCD, 0, Math.PI * 2);
            ctx.stroke();
            ctx.setLineDash([]);
        }

        // 4. 결과 선분 AE 강조 및 같다 기호
        if (currentStep >= 4) {
            ctx.lineWidth = 4;
            ctx.strokeStyle = "#8354ce";
            ctx.beginPath();
            ctx.moveTo(ptA.x, ptA.y);
            ctx.lineTo(ptE.x, ptE.y);
            ctx.stroke();
            
            drawEqualityMark(ptC, ptD);
            drawEqualityMark(ptA, ptF);
            drawEqualityMark(ptA, ptE);
        }

        // 점 및 라벨 표시 함수
        const drawPt = (p, label, color) => {
            ctx.beginPath(); ctx.arc(p.x, p.y, 6, 0, Math.PI * 2);
            ctx.fillStyle = color; ctx.fill();
            ctx.strokeStyle = "#fff"; ctx.lineWidth = 2; ctx.stroke();
            ctx.fillStyle = "#333"; ctx.font = "bold 16px serif";
            ctx.fillText(label, p.x - 8, p.y - 15);
        };

        drawPt(ptA, "A", "#ff6384");
        drawPt(ptB, "B", "#36a2eb");
        drawPt(ptC, "C", "#ffce56");
        drawPt(ptD, "D", "#ffce56");
        
        if (currentStep >= 1) drawPt(ptF, "F", "#4bc0c0");
        if (currentStep >= 3) drawPt(ptE, "E", "#8354ce");
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
        else if (getDistance(pos, ptC) < 30) dragTarget = 'C';
        else if (getDistance(pos, ptD) < 30) dragTarget = 'D';
        if (dragTarget && e.cancelable) e.preventDefault();
    }

    function moveAction(e) {
        if (!dragTarget) return;
        const pos = getPos(e);
        if (dragTarget === 'A') { ptA.x = pos.x; ptA.y = pos.y; }
        else if (dragTarget === 'B') { ptB.x = pos.x; ptB.y = pos.y; }
        else if (dragTarget === 'C') { ptC.x = pos.x; ptC.y = pos.y; }
        else if (dragTarget === 'D') { ptD.x = pos.x; ptD.y = pos.y; }
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

<div id="euclid-container-4" style="margin: 20px 0; text-align: center; font-family: 'Times New Roman', serif; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); overflow: hidden;">
    <div style="margin-bottom: 15px;">
        <h3 id="stepTitle4" style="margin: 0; color: #333;">단계 0: 두 삼각형 ABC와 DEF</h3>
        <p id="stepDesc4" style="margin: 5px 0; color: #666; font-size: 0.95em;">두 변의 길이와 그 끼인각이 각각 같은 두 삼각형이 주어집니다. (AB=DE, AC=DF, ∠A=∠D)</p>
    </div>
    <div style="margin-bottom: 20px; display: flex; align-items: center; justify-content: center; gap: 15px;">
        <span style="font-weight: bold; font-size: 0.9em;">Step</span>
        <input type="range" id="stepSlider4" min="0" max="5" value="0" style="width: 250px; cursor: pointer; accent-color: #8354ce;">
        <span id="stepValue4" style="font-weight: bold; color: #8354ce; width: 20px;">0</span>
    </div>
    <canvas id="euclidCanvas4" width="800" height="450" style="border:1px solid #eee; background: #fdfdfd; display: block; margin: 0 auto; touch-action: none; max-width: 100%; height: auto; border-radius: 8px; cursor: crosshair;"></canvas> 
    <p style="color: #999; font-size: 0.85em; margin-top: 15px;">※ 점 A, B, C를 드래그하여 삼각형의 모양을 바꾸어보세요.</p>
</div>

<script>
(function() {
    const cvs = document.getElementById('euclidCanvas4');
    const ctx = cvs.getContext('2d');
    const slider = document.getElementById('stepSlider4');
    const stepValTxt = document.getElementById('stepValue4');
    const stepTitle = document.getElementById('stepTitle4');
    const stepDesc = document.getElementById('stepDesc4');

    // 변형 가능한 첫 번째 삼각형 ABC의 초기 좌표
    let ptA = { x: 180, y: 150 };
    let ptB = { x: 120, y: 320 };
    let ptC = { x: 300, y: 300 };
    
    // 두 번째 삼각형 DEF의 시작점 D (고정 정점)
    const ptD = { x: 550, y: 150 };
    const rotAngle = 0.5; // 두 번째 삼각형의 회전 각도 (라디안)

    let dragTarget = null;
    let currentStep = 0;

    const stepInfo = [
        { title: "단계 0: 두 삼각형 ABC와 DEF", desc: "두 변과 끼인각이 같은 두 삼각형이 있습니다. (AB=DE, AC=DF, ∠A=∠D)" },
        { title: "STEP 1: 점 A를 점 D에 포개기", desc: "삼각형 ABC를 평행이동하여 점 A를 점 D 위에 무겁게 겹쳐 놓습니다." },
        { title: "STEP 2: 변 AB와 변 DE의 일치", desc: "변 AB가 변 DE 위에 놓이도록 회전시킵니다. AB=DE이므로 점 B와 점 E가 일치합니다." },
        { title: "STEP 3: 끼인각과 변 AC, DF의 일치", desc: "∠A=∠D이므로 변 AC는 DF 방향과 겹칩니다. AC=DF이므로 점 C와 점 F도 일치합니다." },
        { title: "STEP 4: 밑변 BC와 EF의 일치", desc: "점 B가 E와 일치하고 C가 F와 일치하므로, 밑변 BC는 밑변 EF와 완전히 포개어집니다. (BC=EF)" },
        { title: "STEP 5: 증명 완료 (SAS 합동)", desc: "두 삼각형이 완전히 포개어지므로 넓이가 같고, 대응하는 나머지 각의 크기도 각각 같습니다." }
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

    function rotateVec(v, angle) {
        return {
            x: v.x * Math.cos(angle) - v.y * Math.sin(angle),
            y: v.x * Math.sin(angle) + v.y * Math.cos(angle)
        };
    }

    function markOffset(length, angle) {
        return { x: length * Math.cos(angle), y: length * Math.sin(angle) };
    }
    function gapOffset(gap, angle) {
        return { x: gap * Math.cos(angle), y: gap * Math.sin(angle) };
    }

    // 선분 같음 표시 기호 (ticks: 선 개수)
    function drawEqualityMark(p1, p2, ticks = 1) {
        const midX = (p1.x + p2.x) / 2;
        const midY = (p1.y + p2.y) / 2;
        const angle = Math.atan2(p2.y - p1.y, p2.x - p1.x);
        const markLength = 10; 
        const markGap = 4;    
        const markAngle = angle + Math.PI / 2;

        ctx.save();
        ctx.strokeStyle = "#8354ce";
        ctx.lineWidth = 1.5;

        if (ticks === 1) {
            const edge = markOffset(markLength / 2, markAngle);
            ctx.beginPath();
            ctx.moveTo(midX + edge.x, midY + edge.y);
            ctx.lineTo(midX - edge.x, midY - edge.y);
            ctx.stroke();
        } else if (ticks === 2) {
            for (let i = -1; i <= 1; i += 2) {
                const shift = gapOffset((markGap / 2) * i, angle);
                const edge = markOffset(markLength / 2, markAngle);
                ctx.beginPath();
                ctx.moveTo(midX + shift.x + edge.x, midY + shift.y + edge.y);
                ctx.lineTo(midX + shift.x - edge.x, midY + shift.y - edge.y);
                ctx.stroke();
            }
        } else if (ticks === 3) {
            for (let i = -1; i <= 1; i++) {
                const shift = gapOffset(markGap * i, angle);
                const edge = markOffset(markLength / 2, markAngle);
                ctx.beginPath();
                ctx.moveTo(midX + shift.x + edge.x, midY + shift.y + edge.y);
                ctx.lineTo(midX + shift.x - edge.x, midY + shift.y - edge.y);
                ctx.stroke();
            }
        }
        ctx.restore();
    }

    // 각도 표시 호(Arc) 그리기 함수
    function drawAngleArc(pCenter, p1, p2, radius, color) {
        const a1 = Math.atan2(p1.y - pCenter.y, p1.x - pCenter.x);
        const a2 = Math.atan2(p2.y - pCenter.y, p2.x - pCenter.x);
        let diff = a2 - a1;
        while (diff < -Math.PI) diff += Math.PI * 2;
        while (diff > Math.PI) diff -= Math.PI * 2;

        ctx.save();
        ctx.strokeStyle = color;
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(pCenter.x, pCenter.y, radius, a1, a1 + diff, diff < 0);
        ctx.stroke();
        ctx.restore();
    }

    function draw() {
        ctx.clearRect(0, 0, cvs.width, cvs.height);

        // 기본 벡터 계산
        const vecAB = { x: ptB.x - ptA.x, y: ptB.y - ptA.y };
        const vecAC = { x: ptC.x - ptA.x, y: ptC.y - ptA.y };

        // 합동인 삼각형 DEF의 좌표 동적 계산 (회전 변환 적용)
        const vecDE = rotateVec(vecAB, rotAngle);
        const vecDF = rotateVec(vecAC, rotAngle);
        const ptE = { x: ptD.x + vecDE.x, y: ptD.y + vecDE.y };
        const ptF = { x: ptD.x + vecDF.x, y: ptD.y + vecDF.y };

        // 1. 원본 삼각형 ABC 그리기
        ctx.lineWidth = 2.5;
        ctx.strokeStyle = "#333";
        ctx.beginPath();
        ctx.moveTo(ptA.x, ptA.y); ctx.lineTo(ptB.x, ptB.y); ctx.lineTo(ptC.x, ptC.y);
        ctx.closePath();
        ctx.stroke();

        // 2. 대상 삼각형 DEF 그리기
        ctx.strokeStyle = "#333";
        ctx.beginPath();
        ctx.moveTo(ptD.x, ptD.y); ctx.lineTo(ptE.x, ptE.y); ctx.lineTo(ptF.x, ptF.y);
        ctx.closePath();
        if (currentStep === 5) {
            ctx.fillStyle = "rgba(131, 84, 206, 0.15)";
            ctx.fill();
        }
        ctx.stroke();

        // 초기 조건 표시 (Step 0 ~ Step 1)
        if (currentStep <= 1) {
            drawEqualityMark(ptA, ptB, 1); // AB 일치 표시
            drawEqualityMark(ptD, ptE, 1); // DE 일치 표시
            drawEqualityMark(ptA, ptC, 2); // AC 일치 표시
            drawEqualityMark(ptD, ptF, 2); // DF 일치 표시
            drawAngleArc(ptA, ptB, ptC, 20, "#ff6384"); // 각 A
            drawAngleArc(ptD, ptE, ptF, 20, "#ff6384"); // 각 D
        }

        // 3. 중첩(포개어놓기) 애니메이션 상태 레이어 계산
        if (currentStep >= 1) {
            let ptA_prime, ptB_prime, ptC_prime;

            if (currentStep === 1) {
                // 평행이동 상태 (점 A만 D로 이동, 회전 안 됨)
                ptA_prime = { ...ptD };
                ptB_prime = { x: ptD.x + vecAB.x, y: ptD.y + vecAB.y };
                ptC_prime = { x: ptD.x + vecAC.x, y: ptD.y + vecAC.y };

                // 이동 경로 화살표
                ctx.save();
                ctx.strokeStyle = "rgba(131, 84, 206, 0.6)";
                ctx.lineWidth = 2;
                ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(ptA.x, ptA.y); ctx.lineTo(ptD.x, ptD.y); ctx.stroke();
                ctx.restore();
            } else {
                // 회전 완료 상태 (DEF와 기하학적으로 완전히 겹침)
                ptA_prime = { ...ptD };
                ptB_prime = { ...ptE };
                ptC_prime = { ...ptF };
            }

            // 가상의 움직이는 삼각형 투명하게 그리기
            ctx.save();
            ctx.strokeStyle = "rgba(131, 84, 206, 0.8)";
            ctx.lineWidth = 3;
            ctx.setLineDash([2, 2]);
            ctx.beginPath();
            ctx.moveTo(ptA_prime.x, ptA_prime.y);
            ctx.lineTo(ptB_prime.x, ptB_prime.y);
            ctx.lineTo(ptC_prime.x, ptC_prime.y);
            ctx.closePath();
            ctx.stroke();
            ctx.restore();

            // 단계별 하이라이트 효과
            if (currentStep === 2) {
                // B와 E의 일치 강조
                ctx.beginPath(); ctx.arc(ptE.x, ptE.y, 12, 0, Math.PI*2);
                ctx.strokeStyle = "#4bc0c0"; ctx.lineWidth = 2; ctx.stroke();
            }
            if (currentStep === 3) {
                // C와 F의 일치 강조
                ctx.beginPath(); ctx.arc(ptF.x, ptF.y, 12, 0, Math.PI*2);
                ctx.strokeStyle = "#4bc0c0"; ctx.lineWidth = 2; ctx.stroke();
            }
            if (currentStep >= 4) {
                // 밑변 BC와 EF의 겹침 강조 (보라색 두꺼운 선)
                ctx.save();
                ctx.strokeStyle = "#8354ce";
                ctx.lineWidth = 4;
                ctx.beginPath(); ctx.moveTo(ptE.x, ptE.y); ctx.lineTo(ptF.x, ptF.y); ctx.stroke();
                ctx.restore();
                drawEqualityMark(ptB, ptC, 3);
                drawEqualityMark(ptE, ptF, 3);
            }
        }

        // 4. 정점 및 라벨 그리기
        const drawPt = (p, label, color) => {
            ctx.beginPath(); ctx.arc(p.x, p.y, 6, 0, Math.PI * 2);
            ctx.fillStyle = color; ctx.fill();
            ctx.strokeStyle = "#fff"; ctx.lineWidth = 2; ctx.stroke();
            ctx.fillStyle = "#333"; ctx.font = "bold 16px serif";
            ctx.fillText(label, p.x - 8, p.y - 15);
        };

        drawPt(ptA, "A", "#ff6384");
        drawPt(ptB, "B", "#36a2eb");
        drawPt(ptC, "C", "#ffce56");
        
        // 겹쳐진 상태에 따라 라벨 유동적 변경
        if (currentStep >= 1) {
            drawPt(ptD, "D(A')", "#ff6384");
            if (currentStep >= 2) drawPt(ptE, "E(B')", "#36a2eb");
            else drawPt(ptE, "E", "#36a2eb");
            
            if (currentStep >= 3) drawPt(ptF, "F(C')", "#ffce56");
            else drawPt(ptF, "F", "#ffce56");
        } else {
            drawPt(ptD, "D", "#999");
            drawPt(ptE, "E", "#999");
            drawPt(ptF, "F", "#999");
        }
    }

    // 드래그 앤 드롭 인터랙션 핸들러
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
        if (getDistance(pos, ptA) < 25) dragTarget = 'A';
        else if (getDistance(pos, ptB) < 25) dragTarget = 'B';
        else if (getDistance(pos, ptC) < 25) dragTarget = 'C';
        if (dragTarget && e.cancelable) e.preventDefault();
    }

    function moveAction(e) {
        if (!dragTarget) return;
        const pos = getPos(e);
        if (dragTarget === 'A') { ptA.x = pos.x; ptA.y = pos.y; }
        else if (dragTarget === 'B') { ptB.x = pos.x; ptB.y = pos.y; }
        else if (dragTarget === 'C') { ptC.x = pos.x; ptC.y = pos.y; }
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