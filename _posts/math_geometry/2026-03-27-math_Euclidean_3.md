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


<div id="hilbert-order-container" style="margin: 20px 0; text-align: center; font-family: 'Malgun Gothic', sans-serif; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); overflow: hidden;">
    <div style="margin-bottom: 20px;">
        <h3 id="axiomTitle" style="margin: 0; color: #333; font-size: 1.4em;">공리 II.1 (사이의 관계)</h3>
        <p id="axiomDesc" style="margin: 8px 0 15px 0; color: #555; font-size: 1em; line-height: 1.4;">점 B가 A와 C 사이에 있다면, A, B, C는 일직선 상에 있고 B는 C와 A 사이에도 있다.</p>
    </div>
    <div style="margin-bottom: 20px; display: flex; flex-wrap: wrap; justify-content: center; gap: 10px;">
        <button class="ax-btn active" onclick="setAxiom(1)">공리 II.1</button>
        <button class="ax-btn" onclick="setAxiom(2)">공리 II.2</button>
        <button class="ax-btn" onclick="setAxiom(3)">공리 II.3</button>
        <button class="ax-btn" onclick="setAxiom(4)">공리 II.4</button>
    </div>
    <canvas id="hilbertOrderCanvas" width="800" height="450" style="border:1px solid #eee; background: #fdfdfd; display: block; margin: 0 auto; touch-action: none; max-width: 100%; height: auto; border-radius: 8px; cursor: crosshair;"></canvas>
    <p style="color: #999; font-size: 0.85em; margin-top: 15px;">※ 점 A와 C를 평면 상에서 자유롭게 움직여보세요. 점 B는 공리의 조건에 따라 직선 위를 움직입니다.</p>
</div>

<script>
(function() {
    const cvs = document.getElementById('hilbertOrderCanvas');
    const ctx = cvs.getContext('2d');

    const axiomData = [
        { title: "공리 II.1 (사이의 관계)", desc: "점 B가 A와 C 사이에 있다면, 세 점은 일직선 상에 있으며 순서 'A-B-C'는 'C-B-A'와 동일하다." },
        { title: "공리 II.2 (연장선 상의 점)", desc: "임의의 두 점 A, C에 대하여, C가 A와 B 사이에 오도록 하는 점 B가 선분 AC의 연장선 위에 적어도 하나 존재한다." },
        { title: "공리 II.3 (유일한 중간점)", desc: "일직선 위에 있는 임의의 세 점에 대하여, 오직 한 점만이 다른 두 점 사이에 위치한다." },
        { title: "공리 II.4 (파슈의 공리, Pasch's Axiom)", desc: "일직선 위에 없는 세 점 A, B, C에 대해 한 직선이 변 AB와 교차하면, 반드시 변 AC 또는 변 BC 중 하나와 교차한다." }
    ];

    let currentAx = 1;
    let dragTarget = null;

    // 공용 점 객체
    let pts = {
        A: { x: 200, y: 300, show: true, color: "#ff6384", name: "A" },
        B: { x: 400, y: 225, show: true, color: "#36a2eb", name: "B" },
        C: { x: 600, y: 150, show: true, color: "#4bc0c0", name: "C" },
        P1: { x: 100, y: 250, show: false, color: "#9b59b6", name: "P1" },
        P2: { x: 700, y: 250, show: false, color: "#9b59b6", name: "P2" }
    };

    window.setAxiom = function(n) {
        currentAx = n;
        document.getElementById("axiomTitle").innerText = axiomData[n-1].title;
        document.getElementById("axiomDesc").innerText = axiomData[n-1].desc;

        const btns = document.querySelectorAll(".ax-btn");
        btns.forEach((btn, idx) => {
            if (idx === n - 1) btn.classList.add("active");
            else btn.classList.remove("active");
        });

        // 공리별 초기 상태 세팅 (대각선으로 배치하여 2차원임을 시각화)
        if (n === 1 || n === 2 || n === 3) {
            pts.A.x = 200; pts.A.y = 300;
            pts.C.x = 600; pts.C.y = 150;
            
            if (n === 2) {
                // B는 C 너머 연장선에 자동 배치됨 (draw에서 처리)
            } else {
                // B를 A와 C의 중간에 배치
                pts.B.x = (pts.A.x + pts.C.x) / 2;
                pts.B.y = (pts.A.y + pts.C.y) / 2;
            }
            pts.P1.show = false; pts.P2.show = false;
        } else if (n === 4) {
            // 파슈의 공리 초기 셋업 (삼각형 및 횡단 직선)
            pts.A.x = 400; pts.A.y = 100;
            pts.B.x = 200; pts.B.y = 350;
            pts.C.x = 600; pts.C.y = 350;
            pts.P1.x = 100; pts.P1.y = 200;
            pts.P2.x = 700; pts.P2.y = 280;
            pts.P1.show = true; pts.P2.show = true;
        }
        draw();
    };

    function getDistance(p1, p2) {
        return Math.hypot(p2.x - p1.x, p2.y - p1.y);
    }

    // 직선 위로 점을 투영(Projection)하는 함수
    function projectOnLine(p, a, b) {
        const ab = { x: b.x - a.x, y: b.y - a.y };
        const ap = { x: p.x - a.x, y: p.y - a.y };
        const abLenSq = ab.x * ab.x + ab.y * ab.y;
        if (abLenSq === 0) return { x: a.x, y: a.y, t: 0 };
        const t = (ap.x * ab.x + ap.y * ab.y) / abLenSq;
        return { 
            x: a.x + t * ab.x, 
            y: a.y + t * ab.y, 
            t: t // A=0, B=1 비율 반환
        };
    }

    function drawInfiniteLine(p1, p2, color="#333", width=2, isDashed=false) {
        const dx = p2.x - p1.x; const dy = p2.y - p1.y;
        const len = Math.hypot(dx, dy);
        if (len === 0) return;
        const L = 2000; 
        ctx.beginPath();
        ctx.moveTo(p1.x - (dx/len)*L, p1.y - (dy/len)*L);
        ctx.lineTo(p1.x + (dx/len)*L, p1.y + (dy/len)*L);
        ctx.strokeStyle = color;
        ctx.lineWidth = width;
        if(isDashed) ctx.setLineDash([5, 5]);
        ctx.stroke();
        ctx.setLineDash([]);
    }

    function getLineSegIntersection(l1, l2, s1, s2) {
        const denom = (l1.x - l2.x)*(s1.y - s2.y) - (l1.y - l2.y)*(s1.x - s2.x);
        if (Math.abs(denom) < 0.001) return null;
        const t = ((l1.x - s1.x)*(s1.y - s2.y) - (l1.y - s1.y)*(s1.x - s2.x)) / denom;
        const u = -((l1.x - l2.x)*(l1.y - s1.y) - (l1.y - l2.y)*(l1.x - s1.x)) / denom;
        if (u >= 0 && u <= 1) { 
            return {
                x: l1.x + t * (l2.x - l1.x),
                y: l1.y + t * (l2.y - l1.y)
            };
        }
        return null;
    }

    function draw() {
        ctx.clearRect(0, 0, cvs.width, cvs.height);

        if (currentAx === 1) {
            // 공리 1: A-B-C 가 일직선, B는 항상 가운데
            drawInfiniteLine(pts.A, pts.C, "#eee", 2);
            
            ctx.beginPath(); ctx.moveTo(pts.A.x, pts.A.y); ctx.lineTo(pts.C.x, pts.C.y);
            ctx.strokeStyle = "#8354ce"; ctx.lineWidth = 6; ctx.stroke();
            
            ctx.fillStyle = "#333"; ctx.font = "bold 16px sans-serif";
            ctx.fillText("A - B - C 형태 (또는 C - B - A)", 20, 30);
            ctx.fillText("※ B를 드래그해보세요 (A와 C 사이만 이동 가능)", 20, 55);
        } 
        else if (currentAx === 2) {
            // 공리 2: C 바깥쪽으로 연장
            // B의 위치를 2D 벡터를 이용해 C의 연장선에 동적 계산
            pts.B.x = pts.C.x + (pts.C.x - pts.A.x) * 0.4;
            pts.B.y = pts.C.y + (pts.C.y - pts.A.y) * 0.4;
            
            drawInfiniteLine(pts.A, pts.C, "#eee", 2);
            
            // 선분 AC (실선)
            ctx.beginPath(); ctx.moveTo(pts.A.x, pts.A.y); ctx.lineTo(pts.C.x, pts.C.y);
            ctx.strokeStyle = "#333"; ctx.lineWidth = 4; ctx.stroke();
            
            // 연장선 CB (점선)
            ctx.beginPath(); ctx.moveTo(pts.C.x, pts.C.y); ctx.lineTo(pts.B.x, pts.B.y);
            ctx.strokeStyle = "#8354ce"; ctx.setLineDash([6, 6]); ctx.lineWidth = 4; ctx.stroke();
            ctx.setLineDash([]);

            ctx.fillStyle = "#8354ce"; ctx.font = "bold 16px sans-serif";
            ctx.fillText("B는 항상 연장선에 존재 (A - C - B)", 20, 30);
        } 
        else if (currentAx === 3) {
            // 공리 3: 세 점 중 하나만 사이에 있다.
            drawInfiniteLine(pts.A, pts.C, "#eee", 2);
            
            // A, B, C를 일직선에 투영했을 때의 비율(t) 계산
            let tA = 0; // 기준점
            let tC = 1; // 끝점
            let projB = projectOnLine(pts.B, pts.A, pts.C);
            let tB = projB.t;

            // 순서 정렬을 위해 배열에 담기
            let arr = [
                { pt: pts.A, t: tA },
                { pt: pts.B, t: tB },
                { pt: pts.C, t: tC }
            ].sort((a, b) => a.t - b.t);

            // 선분 그리기 (가장 끝 점부터 끝 점까지)
            ctx.beginPath(); ctx.moveTo(arr[0].pt.x, arr[0].pt.y); ctx.lineTo(arr[2].pt.x, arr[2].pt.y);
            ctx.strokeStyle = "#333"; ctx.lineWidth = 3; ctx.stroke();

            // 가운데 있는 점(배열의 1번 인덱스)
            let midPt = arr[1].pt;

            // 중간점 강조 원
            ctx.beginPath(); ctx.arc(midPt.x, midPt.y, 16, 0, Math.PI*2);
            ctx.fillStyle = "rgba(241, 196, 15, 0.4)"; ctx.fill();
            ctx.strokeStyle = "#f39c12"; ctx.lineWidth = 2; ctx.stroke();

            ctx.fillStyle = "#333"; ctx.font = "bold 16px sans-serif";
            ctx.fillText(`가운데 위치한 유일한 점: ${midPt.name}`, 20, 30);
            ctx.fillText("※ B를 직선을 따라 A나 C 바깥으로 드래그해보세요.", 20, 55);
        } 
        else if (currentAx === 4) {
            // 파슈의 공리
            ctx.beginPath(); ctx.moveTo(pts.A.x, pts.A.y);
            ctx.lineTo(pts.B.x, pts.B.y); ctx.lineTo(pts.C.x, pts.C.y);
            ctx.closePath();
            ctx.fillStyle = "rgba(54, 162, 235, 0.05)"; ctx.fill();
            ctx.strokeStyle = "#333"; ctx.lineWidth = 2; ctx.stroke();

            drawInfiniteLine(pts.P1, pts.P2, "#9b59b6", 2);

            let intersects = [];
            let i1 = getLineSegIntersection(pts.P1, pts.P2, pts.A, pts.B);
            let i2 = getLineSegIntersection(pts.P1, pts.P2, pts.A, pts.C);
            let i3 = getLineSegIntersection(pts.P1, pts.P2, pts.B, pts.C);
            if (i1) intersects.push({ p: i1, name: "AB" });
            if (i2) intersects.push({ p: i2, name: "AC" });
            if (i3) intersects.push({ p: i3, name: "BC" });

            intersects.forEach(int => {
                ctx.beginPath(); ctx.arc(int.p.x, int.p.y, 6, 0, Math.PI*2);
                ctx.fillStyle = "#e74c3c"; ctx.fill();
                ctx.strokeStyle = "#fff"; ctx.lineWidth = 2; ctx.stroke();
                ctx.fillStyle = "#e74c3c"; ctx.font = "bold 14px sans-serif";
                ctx.fillText(`변 ${int.name} 와 교차`, int.p.x + 10, int.p.y - 10);
            });
        }

        // --- 점 렌더링 ---
        const drawPt = (p, label) => {
            if (!p.show) return;
            if (currentAx === 2 && label === 'B') ctx.globalAlpha = 0.6; // 공리2의 B는 반투명 (자동)
            
            ctx.beginPath(); ctx.arc(p.x, p.y, 7, 0, Math.PI * 2);
            ctx.fillStyle = p.color; ctx.fill();
            ctx.strokeStyle = "#fff"; ctx.lineWidth = 2; ctx.stroke();
            ctx.fillStyle = "#333"; ctx.font = "bold 16px sans-serif";
            
            if (label !== 'P1' && label !== 'P2') {
                ctx.fillText(label, p.x - 8, p.y - 15);
            }
            ctx.globalAlpha = 1.0;
        };
        for (const key in pts) { drawPt(pts[key], key); }
    }

    // --- 인터랙션 로직 ---
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
            if (currentAx === 2 && key === 'B') continue; // 공리2의 B는 드래그 불가
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
        
        if (currentAx === 1) {
            if (dragTarget === 'B') {
                // B는 A와 C를 이은 선분 안에만 존재해야 함 (t값을 0.05 ~ 0.95로 제한)
                const proj = projectOnLine(pos, pts.A, pts.C);
                let t = Math.max(0.05, Math.min(0.95, proj.t));
                pts.B.x = pts.A.x + t * (pts.C.x - pts.A.x);
                pts.B.y = pts.A.y + t * (pts.C.y - pts.A.y);
            } else {
                // A와 C는 자유롭게 2D 이동
                pts[dragTarget].x = pos.x;
                pts[dragTarget].y = pos.y;
                // A나 C가 이동할 때 B의 위치도 선분을 따라 자동으로 보정됨
                const proj = projectOnLine(pts.B, pts.A, pts.C);
                let t = Math.max(0.05, Math.min(0.95, proj.t));
                pts.B.x = pts.A.x + t * (pts.C.x - pts.A.x);
                pts.B.y = pts.A.y + t * (pts.C.y - pts.A.y);
            }
        } 
        else if (currentAx === 2) {
            // A, C 자유롭게 2D 이동 (B는 드래그되지 않으며 draw()에서 자동 계산)
            pts[dragTarget].x = pos.x;
            pts[dragTarget].y = pos.y;
        }
        else if (currentAx === 3) {
            if (dragTarget === 'B') {
                // B는 A와 C가 만드는 무한 직선 위를 이동
                const proj = projectOnLine(pos, pts.A, pts.C);
                pts.B.x = proj.x;
                pts.B.y = proj.y;
            } else {
                // A, C 자유 이동 (직선의 기울기가 변하면 B도 그 직선에 투영됨)
                pts[dragTarget].x = pos.x;
                pts[dragTarget].y = pos.y;
                const proj = projectOnLine(pts.B, pts.A, pts.C);
                pts.B.x = proj.x;
                pts.B.y = proj.y;
            }
        } 
        else if (currentAx === 4) {
            // 파슈의 공리 (모든 점 2D 자유 이동)
            pts[dragTarget].x = pos.x;
            pts[dragTarget].y = pos.y;
        }
        
        draw();
        if (e.cancelable) e.preventDefault();
    }

    cvs.addEventListener('mousedown', moveStart);
    window.addEventListener('mousemove', moveAction);
    window.addEventListener('mouseup', () => dragTarget = null);
    cvs.addEventListener('touchstart', moveStart, { passive: false });
    window.addEventListener('touchmove', moveAction, { passive: false });
    window.addEventListener('touchend', () => dragTarget = null);

    setAxiom(1);
})();
</script>