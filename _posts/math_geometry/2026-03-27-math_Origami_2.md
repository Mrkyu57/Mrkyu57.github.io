---
layout: post
title: "물리학"
koreantitle: 등속직선운동
englishtitle: uniform linear motion
info: 흠
color: "#8354ce"
permalink: /physics_classicaldynamics_1_1/
---

<!-- 모델링 공간 -->
<div id="origami-axiom1-container" 
    style="
    width: 900px; 
    margin: 20px auto; 
    background: #ffffff; 
    border-radius: 12px; 
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.08); 
    border: 1px solid #eee;
    overflow: hidden;
    box-sizing: border-box; 
    position: relative;
">
    <!-- 제목/설명 -->
    <div style="margin-bottom: 15px; margin-top: 15px; display: flex; justify-content: center; align-items: center; gap: 15px; font-weight: bold; font-family: 'Times New Roman', serif;">
        <h3 style="margin: 10; color: #333; ">
        Huzita–Hatori axioms I :
        </h3>
        <p style="margin: 5px 0; color: #666; font-size: 0.95em;">
            서로 다른 두 점이 있을 때, 두 점을 통과하며 접는 방법은 유일하게 존재한다.
        </p>
    </div>
    <!-- 폴드 슬라이더 -->
    <div style="margin-bottom: 15px; display: flex; justify-content: center; align-items: center; gap: 15px;">
        <span style="font-weight: bold; font-size: 0.9em;">접기  Fold</span>
        <input type="range" id="foldSlider" min="0" max="180" value="0" style="width: 200px; cursor: pointer; accent-color: #2563eb;">
        <span id="foldValue" style="font-weight: bold; color: #2563eb; width: 45px;">0°</span>
    </div>
    <!-- 상호작용 영역 -->
    <div id="canvas-container" style="width: 100%; height: 450px; border: 0px solid #eee; border-radius: 8px; background: #f8fafc; cursor: grab; overflow: hidden; position: relative;"></div>
</div>

<!-- 3D 라이브러리 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<!--상호작용 -->
<script>
window.addEventListener('load', function() {
    const container = document.getElementById('canvas-container');
    const slider = document.getElementById('foldSlider');
    const foldValueTxt = document.getElementById('foldValue');

    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0xf8fafc);

    const camera = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 1000);
    camera.position.set(0, 0, 10);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    const ambientLight = new THREE.AmbientLight(0xffffff, 1.0);
    scene.add(ambientLight);
    
    // 그룹 및 메시 설정 (이전과 동일)
    const frontGroup = new THREE.Group(); scene.add(frontGroup);
    const hingeBase = new THREE.Group(); scene.add(hingeBase);
    const hingeFold = new THREE.Group(); hingeBase.add(hingeFold);
    const backGroup = new THREE.Group(); hingeFold.add(backGroup);

    const frontMat = new THREE.MeshStandardMaterial({ color: '#ffffff', side: THREE.FrontSide });
    const backMat = new THREE.MeshStandardMaterial({ color: '#eeeeee', side: THREE.BackSide });

    let fMeshF = new THREE.Mesh(new THREE.BufferGeometry(), frontMat);
    let fMeshB = new THREE.Mesh(new THREE.BufferGeometry(), backMat);
    fMeshB.position.z = -0.01;
    frontGroup.add(fMeshF, fMeshB);

    let bMeshF = new THREE.Mesh(new THREE.BufferGeometry(), frontMat);
    let bMeshB = new THREE.Mesh(new THREE.BufferGeometry(), backMat);
    bMeshB.position.z = -0.01;
    backGroup.add(bMeshF, bMeshB);

    const foldLine = new THREE.Line(new THREE.BufferGeometry(), new THREE.LineDashedMaterial({ color: 0x475569, dashSize: 0.2, gapSize: 0.1 }));
    scene.add(foldLine);

    // 라벨 생성 함수 (사이즈 최적화)
    function createLabel(text, color, fontSize = 40) {
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        canvas.width = 128; canvas.height = 64;
        ctx.font = `italic bold  ${fontSize}px 'Times New Roman', "serif"`; ctx.fillStyle = color;
        ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
        ctx.fillText(text, 64, 32);
        const sprite = new THREE.Sprite(new THREE.SpriteMaterial({ map: new THREE.CanvasTexture(canvas) }));
        sprite.scale.set(1, 0.5, 1); sprite.renderOrder = 1000;
        return sprite;
    }

    const p1Mesh = new THREE.Mesh(new THREE.SphereGeometry(0.12, 32, 32), new THREE.MeshBasicMaterial({ color: 0x2563eb }));
    const p2Mesh = new THREE.Mesh(new THREE.SphereGeometry(0.12, 32, 32), new THREE.MeshBasicMaterial({ color: 0x2563eb }));
    
    // 라벨들 생성
    const p1Label = createLabel('P₁', '#1e40af');
    const p2Label = createLabel('P₂', '#1e40af');
    const lineLabel = createLabel('l', '#475569', 50); // 선 라벨 'L' 추가

    scene.add(p1Mesh, p2Mesh, p1Label, p2Label, lineLabel);

    let p1_pos = new THREE.Vector2(-1.5, 0.5);
    let p2_pos = new THREE.Vector2(1.5, -0.5);

    function updatePaper() {
        const vLine = new THREE.Vector2().subVectors(p2_pos, p1_pos);
        const dist = vLine.length();
        if (dist < 0.2) return;

        const n = new THREE.Vector2(-vLine.y, vLine.x).normalize();
        const angle = Math.atan2(vLine.y, vLine.x);

        // 종이 쪼개기 및 지오메트리 생성 (생략된 로직은 이전과 동일)
        const W = 3, H = 3;
        const corners = [new THREE.Vector2(-W, H), new THREE.Vector2(-W, -H), new THREE.Vector2(W, -H), new THREE.Vector2(W, H)];
        let polyFront = [], polyBack = [];
        corners.forEach((A, i) => {
            let B = corners[(i + 1) % 4];
            let sideA = n.dot(new THREE.Vector2().subVectors(A, p1_pos));
            let sideB = n.dot(new THREE.Vector2().subVectors(B, p1_pos));
            if (sideA >= 0) polyFront.push(A);
            if (sideA <= 0) polyBack.push(A);
            if ((sideA > 0 && sideB < 0) || (sideA < 0 && sideB > 0)) {
                let t = sideA / (sideA - sideB);
                let I = new THREE.Vector2().lerpVectors(A, B, t);
                polyFront.push(I); polyBack.push(I);
            }
        });

        const sortPts = (pts) => {
            const c = pts.reduce((a, b) => ({x:a.x+b.x, y:a.y+b.y}), {x:0, y:0});
            c.x /= pts.length; c.y /= pts.length;
            return pts.sort((a, b) => Math.atan2(a.y-c.y, a.x-c.x) - Math.atan2(b.y-c.y, b.x-c.x));
        };

        if (fMeshF.geometry) fMeshF.geometry.dispose();
        if (bMeshF.geometry) bMeshF.geometry.dispose();
        if (polyFront.length > 2) fMeshF.geometry = fMeshB.geometry = new THREE.ShapeGeometry(new THREE.Shape(sortPts(polyFront)));
        if (polyBack.length > 2) {
            let localPoly = polyBack.map(P => {
                let dx = P.x - p1_pos.x, dy = P.y - p1_pos.y;
                let cos = Math.cos(-angle), sin = Math.sin(-angle);
                return new THREE.Vector2(dx * cos - dy * sin, dx * sin + dy * cos);
            });
            bMeshF.geometry = bMeshB.geometry = new THREE.ShapeGeometry(new THREE.Shape(sortPts(localPoly)));
        }

        hingeBase.position.set(p1_pos.x, p1_pos.y, 0);
        hingeBase.rotation.z = angle;
        hingeFold.rotation.x = -(slider.value * Math.PI) / 180;
        hingeFold.position.z = Math.sin((slider.value * Math.PI / 180) / 2) * 0.05;

        // 위치 업데이트
        p1Mesh.position.set(p1_pos.x, p1_pos.y, 0.1);
        p2Mesh.position.set(p2_pos.x, p2_pos.y, 0.1);
        p1Label.position.set(p1_pos.x, p1_pos.y + 0.35, 0.2);
        p2Label.position.set(p2_pos.x, p2_pos.y + 0.35, 0.2);

        // [추가] 선 라벨 L의 위치 계산
        // 두 점 P1, P2의 중앙에서 법선 방향으로 약간 띄운 위치에 배치
        const midPoint = new THREE.Vector2().addVectors(p1_pos, p2_pos).multiplyScalar(0.5);
        const labelOffset = n.clone().multiplyScalar(0.4); // 선에서 수직 방향으로 0.4만큼 띄움
        lineLabel.position.set(midPoint.x + labelOffset.x, midPoint.y + labelOffset.y, 0.15);

        // 접는 선 연장 표시
        foldLine.geometry.setFromPoints([
            new THREE.Vector3(p1_pos.x - Math.cos(angle)*10, p1_pos.y - Math.sin(angle)*10, 0.01),
            new THREE.Vector3(p1_pos.x + Math.cos(angle)*10, p1_pos.y + Math.sin(angle)*10, 0.01)
        ]);
        foldLine.computeLineDistances();
    }

    // 마우스 드래그 로직 및 애니메이션은 이전과 동일
    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();
    let dragging = null;

    container.addEventListener('pointerdown', (e) => {
        const rect = container.getBoundingClientRect();
        mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
        mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
        raycaster.setFromCamera(mouse, camera);
        const hits = raycaster.intersectObjects([p1Mesh, p2Mesh]);
        if (hits.length > 0) { dragging = hits[0].object; container.style.cursor = 'grabbing'; }
    });

    container.addEventListener('pointermove', (e) => {
        const rect = container.getBoundingClientRect();
        mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
        mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
        if (!dragging) return;
        raycaster.setFromCamera(mouse, camera);
        const intersectPoint = new THREE.Vector3();
        raycaster.ray.intersectPlane(new THREE.Plane(new THREE.Vector3(0,0,1), 0), intersectPoint);
        if (dragging === p1Mesh) p1_pos.set(Math.max(-2.9, Math.min(2.9, intersectPoint.x)), Math.max(-2.9, Math.min(2.9, intersectPoint.y)));
        else p2_pos.set(Math.max(-2.9, Math.min(2.9, intersectPoint.x)), Math.max(-2.9, Math.min(2.9, intersectPoint.y)));
        updatePaper();
    });

    container.addEventListener('pointerup', () => { dragging = null; container.style.cursor = 'default'; });
    slider.addEventListener('input', () => { foldValueTxt.innerText = slider.value + "°"; updatePaper(); });

    function animate() { requestAnimationFrame(animate); renderer.render(scene, camera); }
    updatePaper();
    animate();
});
</script>