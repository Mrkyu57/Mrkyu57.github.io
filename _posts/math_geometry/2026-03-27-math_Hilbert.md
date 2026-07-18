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

<div id="hilbert-container" style="margin: 20px 0; text-align: center; font-family: 'Malgun Gothic', sans-serif; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); overflow: hidden;">
    <div style="margin-bottom: 20px;">
        <h3 id="axiomTitle" style="margin: 0; color: #333; font-size: 1.4em; font-family: 'Times New Roman'">Incidence Axioms I</h3>
        <p id="axiomDesc" style="margin: 8px 0 15px 0; color: #555; font-size: 1em; line-height: 1.4;">서로 다른 두 점 A, B에 대하여 두 점을 지나는 직선이 존재한다</p>
        <p id="axiomSymbol" style="margin: 0 0 15px 0; color: #555; font-size: 1em; line-height: 1.4;font-family: 'Times New Roman';font-weight: bold;">∀A,B (A≠B→∃l(A∈l∧B∈l))</p>
    </div>
    <div style="margin-bottom: 20px; display: flex; flex-wrap: wrap; justify-content: center; gap: 10px;">
        <button class="ax-btn active" onclick="setAxiom(1)">공리 I.1</button>
        <button class="ax-btn" onclick="setAxiom(2)">공리 I.2</button>
        <button class="ax-btn" onclick="setAxiom(3)">공리 I.3</button>
        <button class="ax-btn" onclick="setAxiom(4)">공리 I.4</button>
        <button class="ax-btn" onclick="setAxiom(5)">공리 I.5</button>
    </div>
    <canvas id="hilbertCanvas" width="800" height="450" style="border:1px solid #eee; background: #fdfdfd; display: block; margin: 0 auto; touch-action: none; max-width: 100%; height: auto; border-radius: 8px; cursor: crosshair;"></canvas>
    <p style="color: #999; font-size: 0.85em; margin-top: 15px;">※ 캔버스 안의 점들을 자유롭게 드래그하여 결합 공리(점, 선, 면의 관계)를 확인해보세요.</p>
</div>

<script>
(function() {
    const cvs = document.getElementById('hilbertCanvas');
    const ctx = cvs.getContext('2d');

    const axiomData = [
        { title: "Incidence Axioms I", desc: "서로 다른 두 점 A, B를 모두 지나는 직선이 적어도 하나 존재한다." },
        { title: "Incidence Axioms II", desc: "서로 다른 두 점 A, B를 모두 지나는 직선은 유일하게 존재한다" },
        { title: "Incidence Axioms III", desc: "일직선 위에 있지 않은 세 점이 적어도 하나 존재한다." },
        { title: "Incidence Axioms I", desc: "일직선 위에 있지 않은 세 점 A, B, C에 대하여 이들을 모두 포함하는 평면이 적어도 하나 존재한다." },
        { title: "Incidence Axioms", desc: "일직선 위에 있지 않은 세 점 A, B, C를 포함하는 평면은 많아야 하나 존재한다. (즉, 평면 α는 유일하다.)" }
    ];

    let currentAx = 1;
    let dragTarget = null;

    // 점 데이터 (A, B는 기본, C는 공리 3~5에서 사용)
    let pts = {
        A: { x: 250, y: 225, show: true, color: "#ff6384" },
        B: { x: 550, y: 225, show: true, color: "#36a2eb" },
        C: { x: 400, y: 100, show: false, color: "#4bc0c0" }
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

        // 공리에 따른 점 노출 상태
        if (n >= 3) {
            pts.C.show = true;
            if (n === 3 && pts.C.y === 225) pts.C.y = 100; // C가 우연히 일직선에 겹쳤다면 튕겨냄
        } else {
            pts.C.show = false;
        }
        
        draw();
    };

    function getDistance(p1, p2) {
        return Math.hypot(p2.x - p1.x, p2.y - p1.y);
    }

    // 직선 그리기
    function drawInfiniteLine(p1, p2, color = "#333", width = 3) {
        const dx = p2.x - p1.x; const dy = p2.y - p1.y;
        const len = Math.hypot(dx, dy);
        if (len === 0) return;
        const L = 2000; 
        ctx.beginPath();
        ctx.moveTo(p1.x - (dx/len)*L, p1.y - (dy/len)*L);
        ctx.lineTo(p1.x + (dx/len)*L, p1.y + (dy/len)*L);
        ctx.strokeStyle = color;
        ctx.lineWidth = width;
        ctx.stroke();
    }

    // 점과 직선 사이의 거리 계산 (공리 3 시각화 용)
    function distToSegment(p, v, w) {
        const l2 = Math.pow(w.x - v.x, 2) + Math.pow(w.y - v.y, 2);
        if (l2 === 0) return getDistance(p, v);
        let t = ((p.x - v.x) * (w.x - v.x) + (p.y - v.y) * (w.y - v.y)) / l2;
        return getDistance(p, { x: v.x + t * (w.x - v.x), y: v.y + t * (w.y - v.y) });
    }

    function draw() {
        ctx.clearRect(0, 0, cvs.width, cvs.height);

        // --- 공리별 시각화 로직 ---
        if (currentAx === 1) {
            // 공리 I.1: 두 점을 지나는 직선의 존재
            drawInfiniteLine(pts.A, pts.B);
            ctx.fillStyle = "#333"; ctx.font = "bold 16px sans-serif";
            ctx.fillText("Exist l", pts.B.x + 10, pts.B.y - 10);
        } 
        else if (currentAx === 2) {
            // 공리 I.2: 직선의 유일성
            drawInfiniteLine(pts.A, pts.B, "#8354ce", 4);
            
            // 직선이 아닌 다른 임의의 경로(곡선)를 그려서, 곧은 '직선'은 하나뿐임을 시각적으로 강조
            ctx.beginPath();
            ctx.moveTo(pts.A.x, pts.A.y);
            ctx.quadraticCurveTo(400, pts.A.y - 150, pts.B.x, pts.B.y);
            ctx.strokeStyle = "rgba(231, 76, 60, 0.4)";
            ctx.lineWidth = 2;
            ctx.setLineDash([5, 5]);
            ctx.stroke();
            ctx.setLineDash([]);

            ctx.fillStyle = "#8354ce"; ctx.font = "bold 16px sans-serif";
            ctx.fillText("유일한 직선 a", pts.B.x + 20, pts.B.y - 20);
        } 
        else if (currentAx === 3) {
            // 공리 I.3: 직선 위의 두 점과 직선 밖의 점
            drawInfiniteLine(pts.A, pts.B);
            
            // 점 C가 일직선에 있는지 검사
            const dist = distToSegment(pts.C, pts.A, pts.B);
            if (dist < 10) {
                ctx.fillStyle = "#e74c3c"; ctx.font = "bold 16px sans-serif";
                ctx.fillText("경고: C가 일직선 위에 있습니다!", pts.C.x + 15, pts.C.y - 20);
            } else {
                // C에서 직선으로의 보조선 연결
                ctx.beginPath();
                ctx.moveTo(pts.C.x, pts.C.y);
                ctx.lineTo(pts.A.x, pts.A.y);
                ctx.moveTo(pts.C.x, pts.C.y);
                ctx.lineTo(pts.B.x, pts.B.y);
                ctx.strokeStyle = "rgba(75, 192, 192, 0.5)";
                ctx.setLineDash([4, 4]);
                ctx.stroke();
                ctx.setLineDash([]);
                
                ctx.fillStyle = "#4bc0c0"; ctx.font = "bold 14px sans-serif";
                ctx.fillText("일직선 밖에 있는 세 번째 점 존재", pts.C.x + 15, pts.C.y - 20);
            }
        } 
        else if (currentAx === 4 || currentAx === 5) {
            // 공리 I.4 & I.5: 평면의 존재와 유일성
            // A, B, C를 포함하는 거대한 평면(다각형) 렌더링
            const cx = (pts.A.x + pts.B.x + pts.C.x) / 3;
            const cy = (pts.A.y + pts.B.y + pts.C.y) / 3;
            const scale = 5; // 평면을 시각적으로 크게 확장

            ctx.beginPath();
            ctx.moveTo(cx + scale * (pts.A.x - cx), cy + scale * (pts.A.y - cy));
            ctx.lineTo(cx + scale * (pts.B.x - cx), cy + scale * (pts.B.y - cy));
            ctx.lineTo(cx + scale * (pts.C.x - cx), cy + scale * (pts.C.y - cy));
            ctx.closePath();

            // 공리 5일 때는 평면의 색상을 진하게 하고 강조 선을 넣음
            if (currentAx === 4) {
                ctx.fillStyle = "rgba(131, 84, 206, 0.1)";
                ctx.fill();
                ctx.strokeStyle = "rgba(131, 84, 206, 0.3)";
                ctx.stroke();
                ctx.fillStyle = "#8354ce";
                ctx.fillText("평면 α 생성", cx, cy - 50);
            } else {
                ctx.fillStyle = "rgba(255, 159, 64, 0.15)";
                ctx.fill();
                ctx.strokeStyle = "rgba(255, 159, 64, 0.8)";
                ctx.lineWidth = 3;
                ctx.stroke();
                ctx.fillStyle = "#d35400";
                ctx.fillText("오직 하나뿐인 평면 α (유일성)", cx, cy - 50);
            }

            // 삼각형 형태의 결합망
            ctx.beginPath();
            ctx.moveTo(pts.A.x, pts.A.y);
            ctx.lineTo(pts.B.x, pts.B.y);
            ctx.lineTo(pts.C.x, pts.C.y);
            ctx.closePath();
            ctx.strokeStyle = "#333";
            ctx.lineWidth = 2;
            ctx.setLineDash([5, 5]);
            ctx.stroke();
            ctx.setLineDash([]);
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





























