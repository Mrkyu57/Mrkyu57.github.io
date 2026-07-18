<style>
    .ax-btn {
        padding: 8px 16px; 
        border: 2px solid #8354ce; 
        border-radius: 20px;
        background: #fff; 
        color: #8354ce; 
        cursor: pointer; 
        transition: all 0.2s ease;
        font-weight: bold; 
        font-size: 0.95em; 
        outline: none;
    }
    .ax-btn:hover { background: #f4ebff; }
    .ax-btn.active { background: #8354ce; color: #fff; }
</style>

<div id="euclid-container" style="margin: 20px 0; text-align: center; font-family: 'Times New Roman', sans-serif; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); overflow: hidden;">
    <div style="margin-bottom: 20px;">
        <h3 id="axiomTitle" style="margin: 0; color: #333; font-size: 1.4em;">공리 1 (Postulate I)</h3>
        <p id="axiomDesc" style="margin: 8px 0 15px 0; color: #555; font-size: 1em; line-height: 1.4;">임의의 점에서 임의의 다른 점으로 직선을 그릴 수 있다.</p>
    </div>
    <div style="margin-bottom: 20px; display: flex; flex-wrap: wrap; justify-content: center; gap: 10px;">
        <button class="ax-btn active" onclick="setAxiom(1)">공리 1</button>
        <button class="ax-btn" onclick="setAxiom(2)">공리 2</button>
        <button class="ax-btn" onclick="setAxiom(3)">공리 3</button>
        <button class="ax-btn" onclick="setAxiom(4)">공리 4</button>
        <button class="ax-btn" onclick="setAxiom(5)">공리 5</button>
    </div>
    <canvas id="euclidCanvas" width="800" height="450" style="border:1px solid #eee; background: #fdfdfd; display: block; margin: 0 auto; touch-action: none; max-width: 100%; height: auto; border-radius: 8px; cursor: crosshair;"></canvas>
    <p style="color: #999; font-size: 0.85em; margin-top: 15px;">※ 캔버스 안의 점들을 자유롭게 드래그하여 공리를 확인해보세요.</p>
</div>

<script>
(function() {
    const cvs = document.getElementById('euclidCanvas');
    const ctx = cvs.getContext('2d');

    const axiomData = [
        { title: "Postulate I", desc: "임의의 점에서 임의의 다른 점으로 선분을 그릴 수 있다." },
        { title: "Postulate II", desc: "유한한 선분을 양방향으로 무한히 곧게 연장할 수 있다." },
        { title: "Postulate III", desc: "임의의 중심과 반지름을 가진 원을 그릴 수 있다." },
        { title: "Postulate IV", desc: "모든 직각은 (어디에 있든, 어떻게 회전되어 있든) 서로 같다." },
        { title: "Postulate V", desc: "한 직선(AB)이 두 직선과 만날 때, 내각의 합이 180도보다 작은 쪽으로 두 직선을 무한히 연장하면 결국 교차한다." }
    ];

    let currentAx = 1;
    let dragTarget = null;

    // 점 데이터 (A, B는 기본, C, D는 공리 5를 위해 존재)
    let pts = {
        A: { x: 300, y: 225, show: true, color: "#ff6384" },
        B: { x: 500, y: 225, show: true, color: "#36a2eb" },
        C: { x: 550, y: 100, show: false, color: "#4bc0c0" },
        D: { x: 500, y: 350, show: false, color: "#ff9f40" }
    };

    window.setAxiom = function(n) {
        currentAx = n;
        document.getElementById("axiomTitle").innerText = axiomData[n-1].title;
        document.getElementById("axiomDesc").innerText = axiomData[n-1].desc;

        // 버튼 스타일 업데이트
        const btns = document.querySelectorAll(".ax-btn");
        btns.forEach((btn, idx) => {
            if (idx === n - 1) btn.classList.add("active");
            else btn.classList.remove("active");
        });

        // 공리에 따른 점 초기화 및 노출
        if (n === 5) {
            if (!pts.C.show) {
                pts.A.x = 350; pts.A.y = 150;
                pts.B.x = 350; pts.B.y = 350;
                pts.C.x = 550; pts.C.y = 100;
                pts.D.x = 500; pts.D.y = 300;
            }
            pts.C.show = true; pts.D.show = true;
        } else {
            pts.C.show = false; pts.D.show = false;
            if (n === 4 && (pts.A.x === 350 && pts.B.x === 350)) {
                pts.A.x = 250; pts.A.y = 225;
                pts.B.x = 550; pts.B.y = 225;
            }
        }
        draw();
    };

    function getDistance(p1, p2) {
        return Math.hypot(p2.x - p1.x, p2.y - p1.y);
    }

    // 무한 직선 그리기 (화면 밖까지)
    function drawInfiniteLine(p1, p2) {
        const dx = p2.x - p1.x; const dy = p2.y - p1.y;
        const len = Math.hypot(dx, dy);
        if (len === 0) return;
        const L = 2000; 
        ctx.beginPath();
        ctx.moveTo(p1.x - (dx/len)*L, p1.y - (dy/len)*L);
        ctx.lineTo(p1.x + (dx/len)*L, p1.y + (dy/len)*L);
        ctx.stroke();
    }

    // 교점 계산 함수 (공리 5)
    function getIntersection(p1, p2, p3, p4) {
        const denom = (p1.x - p2.x)*(p3.y - p4.y) - (p1.y - p2.y)*(p3.x - p4.x);
        if (Math.abs(denom) < 0.001) return null; // 평행할 때
        const t = ((p1.x - p3.x)*(p3.y - p4.y) - (p1.y - p3.y)*(p3.x - p4.x)) / denom;
        return {
            x: p1.x + t * (p2.x - p1.x),
            y: p1.y + t * (p2.y - p1.y)
        };
    }

    // 직각 기호 그리기 (공리 4)
    function drawRightAngleIcon(p, angleOff) {
        ctx.save();
        ctx.translate(p.x, p.y);
        ctx.rotate(angleOff);

        ctx.lineWidth = 2;
        ctx.strokeStyle = "#8354ce";
        ctx.beginPath();
        ctx.moveTo(-60, 0); ctx.lineTo(60, 0);
        ctx.moveTo(0, -60); ctx.lineTo(0, 60);
        ctx.stroke();

        ctx.strokeStyle = "#e74c3c";
        ctx.beginPath();
        ctx.moveTo(15, 0); ctx.lineTo(15, -15); ctx.lineTo(0, -15);
        ctx.stroke();
        ctx.restore();
    }

    function draw() {
        ctx.clearRect(0, 0, cvs.width, cvs.height);

        // --- 공리별 시각화 로직 ---
        if (currentAx === 1) {
            // 공리 1: 단순 선분
            ctx.beginPath(); ctx.moveTo(pts.A.x, pts.A.y); ctx.lineTo(pts.B.x, pts.B.y);
            ctx.strokeStyle = "#333"; ctx.lineWidth = 3; ctx.stroke();
        } 
        else if (currentAx === 2) {
            // 공리 2: 선분 연장 (점선)
            ctx.strokeStyle = "#8354ce"; ctx.lineWidth = 2; ctx.setLineDash([5, 5]);
            drawInfiniteLine(pts.A, pts.B);
            ctx.setLineDash([]);
            // 실선 부분 (유한 선분)
            ctx.beginPath(); ctx.moveTo(pts.A.x, pts.A.y); ctx.lineTo(pts.B.x, pts.B.y);
            ctx.strokeStyle = "#333"; ctx.lineWidth = 3; ctx.stroke();
        } 
        else if (currentAx === 3) {
            // 공리 3: 원 그리기
            const r = getDistance(pts.A, pts.B);
            ctx.beginPath(); ctx.moveTo(pts.A.x, pts.A.y); ctx.lineTo(pts.B.x, pts.B.y);
            ctx.strokeStyle = "#888"; ctx.setLineDash([5, 5]); ctx.lineWidth = 2; ctx.stroke();
            ctx.setLineDash([]);
            
            ctx.beginPath(); ctx.arc(pts.A.x, pts.A.y, r, 0, Math.PI*2);
            ctx.strokeStyle = "#8354ce"; ctx.lineWidth = 3; ctx.stroke();
            ctx.fillStyle = "rgba(131, 84, 206, 0.05)"; ctx.fill();
        } 
        else if (currentAx === 4) {
            // 공리 4: 모든 직각은 같다
            drawRightAngleIcon(pts.A, 0);
            drawRightAngleIcon(pts.B, Math.PI / 5); // B는 임의로 회전되어 있음
        } 
        else if (currentAx === 5) {
            // 공리 5: 평행선
            // 횡단선(Transversal) AB
            ctx.beginPath(); ctx.moveTo(pts.A.x, pts.A.y); ctx.lineTo(pts.B.x, pts.B.y);
            ctx.strokeStyle = "#333"; ctx.lineWidth = 3; ctx.stroke();

            // 직선 1 (A-C를 지남)
            ctx.strokeStyle = pts.C.color; ctx.lineWidth = 2; 
            drawInfiniteLine(pts.A, pts.C);
            
            // 직선 2 (B-D를 지남)
            ctx.strokeStyle = pts.D.color; ctx.lineWidth = 2; 
            drawInfiniteLine(pts.B, pts.D);

            // 교점 확인 및 표시
            const intersect = getIntersection(pts.A, pts.C, pts.B, pts.D);
            if (intersect) {
                ctx.beginPath(); ctx.arc(intersect.x, intersect.y, 6, 0, Math.PI*2);
                ctx.fillStyle = "#e74c3c"; ctx.fill();
                ctx.strokeStyle = "#fff"; ctx.lineWidth = 2; ctx.stroke();
                
                ctx.fillStyle = "#e74c3c"; ctx.font = "bold 14px sans-serif";
                ctx.fillText("교차점", intersect.x + 12, intersect.y - 12);
            }
        }

        // --- 점 그리기 ---
        const drawPt = (p, label) => {
            if (!p.show) return;
            ctx.beginPath(); ctx.arc(p.x, p.y, 7, 0, Math.PI * 2);
            ctx.fillStyle = p.color; ctx.fill();
            ctx.strokeStyle = "#fff"; ctx.lineWidth = 2; ctx.stroke();
            ctx.fillStyle = "#333"; ctx.font = "bold 16px sans-serif";
            ctx.fillText(label, p.x - 8, p.y - 15);
        };
        for (const key in pts) { drawPt(pts[key], key); }
    }

    // --- 인터랙션(드래그) 이벤트 ---
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
        dragTarget = null;
        for (const key in pts) {
            if (pts[key].show && getDistance(pos, pts[key]) < 25) {
                dragTarget = key;
                break;
            }
        }
        if (dragTarget && e.cancelable) e.preventDefault();
    }

    function moveAction(e) {
        if (!dragTarget) return;
        const pos = getPos(e);
        pts[dragTarget].x = pos.x; 
        pts[dragTarget].y = pos.y;
        draw();
        if (e.cancelable) e.preventDefault();
    }

    cvs.addEventListener('mousedown', moveStart);
    window.addEventListener('mousemove', moveAction);
    window.addEventListener('mouseup', () => dragTarget = null);
    cvs.addEventListener('touchstart', moveStart, { passive: false });
    window.addEventListener('touchmove', moveAction, { passive: false });
    window.addEventListener('touchend', () => dragTarget = null);

    // 초기 화면 렌더링
    setAxiom(1);
})();
</script>

