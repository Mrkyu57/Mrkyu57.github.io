<div id="origami-axiom3-overlap" style="margin: 20px 0; text-align: center; font-family: 'Times New Roman', serif; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05);">
    
<div style="margin-bottom: 15px;">
        <h3 style="margin: 0; color: #333;">3D 종이접기: 공리 3 (선과 선 겹치기)</h3>
        <p style="margin: 5px 0; color: #666; font-size: 0.95em;">
            <strong>빨간색 선(L1)</strong>이 <strong>파란색 선(L2)</strong> 위로 겹치도록 접습니다.<br>
            접는 선 <strong>L</strong>은 두 직선 사이의 각을 이등분하는 선입니다.
        </p>
    </div>

<div style="margin-bottom: 15px; display: flex; justify-content: center; align-items: center; gap: 15px;">
        <span style="font-weight: bold; font-size: 0.9em;">접기 (Fold)</span>
        <input type="range" id="foldSlider" min="0" max="180" value="0" style="width: 200px; cursor: pointer; accent-color: #ef4444;">
        <span id="foldValue" style="font-weight: bold; color: #ef4444; width: 45px;">0°</span>
    </div>

<div id="canvas-container" style="width: 100%; height: 500px; border: 1px solid #eee; border-radius: 8px; background: #fdf2f2; cursor: grab; overflow: hidden; position: relative;"></div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<script>
window.addEventListener('load', function() {
    const container = document.getElementById('canvas-container');
    const slider = document.getElementById('foldSlider');
    const foldValueTxt = document.getElementById('foldValue');

    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0xfdf2f2);

    const camera = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 1000);
    camera.position.set(0, 0, 10);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    const ambientLight = new THREE.AmbientLight(0xffffff, 1.0);
    scene.add(ambientLight);

    // 종이 설정
    const frontGroup = new THREE.Group(); scene.add(frontGroup);
    const hingeBase = new THREE.Group(); scene.add(hingeBase);
    const hingeFold = new THREE.Group(); hingeBase.add(hingeFold);
    const backGroup = new THREE.Group(); hingeFold.add(backGroup);

    const paperMat = new THREE.MeshStandardMaterial({ color: 0xffffff, side: THREE.DoubleSide });
    
    let fMesh = new THREE.Mesh(new THREE.BufferGeometry(), paperMat);
    frontGroup.add(fMesh);

    let bMesh = new THREE.Mesh(new THREE.BufferGeometry(), paperMat);
    backGroup.add(bMesh);

    // 선 설정 (L1: 빨강, L2: 파랑, L: 점선)
    const lineMatRed = new THREE.LineBasicMaterial({ color: 0xef4444, linewidth: 3 });
    const lineMatBlue = new THREE.LineBasicMaterial({ color: 0x3b82f6, linewidth: 3 });
    const foldLineMat = new THREE.LineDashedMaterial({ color: 0x4b5563, dashSize: 0.2, gapSize: 0.1 });

    const l1Line = new THREE.Line(new THREE.BufferGeometry(), lineMatRed);
    const l2Line = new THREE.Line(new THREE.BufferGeometry(), lineMatBlue);
    const foldLine = new THREE.Line(new THREE.BufferGeometry(), foldLineMat);

    // L2는 바닥(고정)에, L1은 접히는 부분(이동)에 그립니다.
    frontGroup.add(l2Line);
    backGroup.add(l1Line);
    scene.add(foldLine);

    // 드래그용 핸들
    const handleGeo = new THREE.SphereGeometry(0.12, 16, 16);
    const h2_A = new THREE.Mesh(handleGeo, new THREE.MeshBasicMaterial({ color: 0x3b82f6 }));
    const h2_B = new THREE.Mesh(handleGeo, new THREE.MeshBasicMaterial({ color: 0x3b82f6 }));
    scene.add(h2_A, h2_B);

    // 초기 상태: L1과 L2 정의
    let p2_A = new THREE.Vector2(-2, -1), p2_B = new THREE.Vector2(2, 1); // 고정된 파란 선
    let p1_A = new THREE.Vector2(-2, 1), p1_B = new THREE.Vector2(2, -1); // 이동할 빨간 선

    function updatePaper() {
        const dir1 = new THREE.Vector2().subVectors(p1_B, p1_A).normalize();
        const dir2 = new THREE.Vector2().subVectors(p2_B, p2_A).normalize();

        // 1. 두 직선의 교점(I) 찾기
        const det = dir1.x * dir2.y - dir1.y * dir2.x;
        let intersect = new THREE.Vector2();
        if (Math.abs(det) < 0.001) {
            intersect.addVectors(p1_A, p2_A).multiplyScalar(0.5);
        } else {
            const t = ((p2_A.x - p1_A.x) * dir2.y - (p2_A.y - p1_A.y) * dir2.x) / det;
            intersect.set(p1_A.x + dir1.x * t, p1_A.y + dir1.y * t);
        }

        // 2. 접는 선(L)의 방향 계산 (각 이등분선)
        let foldDir = new THREE.Vector2().addVectors(dir1, dir2).normalize();
        if (foldDir.length() < 0.1) foldDir.set(-dir1.y, dir1.x); 
        const foldAngle = Math.atan2(foldDir.y, foldDir.x);
        const foldNormal = new THREE.Vector2(-foldDir.y, foldDir.x);

        // 3. 종이 다각형 분할 (고정부 vs 이동부)
        const W = 3.5;
        const corners = [new THREE.Vector2(-W, W), new THREE.Vector2(-W, -W), new THREE.Vector2(W, -W), new THREE.Vector2(W, W)];
        let polyFront = [], polyBack = [];
        corners.forEach((A, i) => {
            let B = corners[(i + 1) % 4];
            let sideA = foldNormal.dot(new THREE.Vector2().subVectors(A, intersect));
            let sideB = foldNormal.dot(new THREE.Vector2().subVectors(B, intersect));
            if (sideA >= 0) polyFront.push(A);
            if (sideA <= 0) polyBack.push(A);
            if ((sideA > 0 && sideB < 0) || (sideA < 0 && sideB > 0)) {
                let I = new THREE.Vector2().lerpVectors(A, B, sideA / (sideA - sideB));
                polyFront.push(I); polyBack.push(I);
            }
        });

        const sort = (p) => {
            let c = p.reduce((a,b)=>({x:a.x+b.x, y:a.y+b.y}), {x:0, y:0});
            c.x/=p.length; c.y/=p.length;
            return p.sort((a,b)=>Math.atan2(a.y-c.y, a.x-c.x) - Math.atan2(b.y-c.y, b.x-c.x));
        }

        if (fMesh.geometry) fMesh.geometry.dispose();
        if (bMesh.geometry) bMesh.geometry.dispose();
        if (polyFront.length > 2) fMesh.geometry = new THREE.ShapeGeometry(new THREE.Shape(sort(polyFront)));
        
        if (polyBack.length > 2) {
            let localPoly = polyBack.map(P => {
                let dx = P.x - intersect.x, dy = P.y - intersect.y;
                let c = Math.cos(-foldAngle), s = Math.sin(-foldAngle);
                return new THREE.Vector2(dx * c - dy * s, dx * s + dy * c);
            });
            bMesh.geometry = new THREE.ShapeGeometry(new THREE.Shape(sort(localPoly)));
        }

        // 4. 선 그리기
        // 파란 선(L2)은 고정된 면에 표시 (Front)
        l2Line.geometry.setFromPoints([
            new THREE.Vector3(p2_A.x, p2_A.y, 0.01),
            new THREE.Vector3(p2_B.x, p2_B.y, 0.01)
        ]);

        // 빨간 선(L1)은 접히는 면의 로컬 좌표로 변환 (Back)
        const toLocal = (P) => {
            let dx = P.x - intersect.x, dy = P.y - intersect.y;
            let c = Math.cos(-foldAngle), s = Math.sin(-foldAngle);
            return new THREE.Vector3(dx * c - dy * s, dx * s + dy * c, 0.01);
        };
        l1Line.geometry.setFromPoints([toLocal(p1_A), toLocal(p1_B)]);

        // 5. 힌지 업데이트
        hingeBase.position.set(intersect.x, intersect.y, 0);
        hingeBase.rotation.z = foldAngle;
        const rad = (slider.value * Math.PI) / 180;
        hingeFold.rotation.x = -rad;
        hingeFold.position.z = Math.sin(rad/2) * 0.1;

        // 드래그 핸들 (파란 선 제어용)
        h2_A.position.set(p2_A.x, p2_A.y, 0.05);
        h2_B.position.set(p2_B.x, p2_B.y, 0.05);

        // 접는 선 표시
        foldLine.geometry.setFromPoints([
            new THREE.Vector3(intersect.x - foldDir.x * 10, intersect.y - foldDir.y * 10, 0.02),
            new THREE.Vector3(intersect.x + foldDir.x * 10, intersect.y + foldDir.y * 10, 0.02)
        ]);
        foldLine.computeLineDistances();
    }

    // 마우스 드래그 (L1, L2 위치 변경)
    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();
    let dragging = null;

    container.addEventListener('pointerdown', (e) => {
        const rect = container.getBoundingClientRect();
        mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
        mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
        raycaster.setFromCamera(mouse, camera);
        const hits = raycaster.intersectObjects([h2_A, h2_B]);
        if (hits.length > 0) { dragging = hits[0].object; container.style.cursor = 'grabbing'; }
    });

    container.addEventListener('pointermove', (e) => {
        if (!dragging) return;
        const rect = container.getBoundingClientRect();
        mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
        mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
        raycaster.setFromCamera(mouse, camera);
        const intersectPt = new THREE.Vector3();
        raycaster.ray.intersectPlane(new THREE.Plane(new THREE.Vector3(0,0,1), 0), intersectPt);
        
        if (dragging === h2_A) p2_A.set(intersectPt.x, intersectPt.y);
        else if (dragging === h2_B) p2_B.set(intersectPt.x, intersectPt.y);
        
        // 공리 3의 핵심: L1을 L2에 겹치게 하는 접는 선이 유지되도록 L1은 L2의 반사된 위치에 고정
        // (사용자는 L2를 움직이고, 그에 대응하는 가상의 L1이 접힘을 통해 증명됨)
        updatePaper();
    });

    container.addEventListener('pointerup', () => { dragging = null; container.style.cursor = 'default'; });
    slider.addEventListener('input', () => { foldValueTxt.innerText = slider.value + "°"; updatePaper(); });

    function animate() { requestAnimationFrame(animate); renderer.render(scene, camera); }
    updatePaper();
    animate();
});
</script>