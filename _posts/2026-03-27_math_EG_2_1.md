<div id="origami-axiom2-container" style="margin: 20px 0; text-align: center; font-family: 'Times New Roman', serif; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); position: relative;">
    
<div style="margin-bottom: 15px;">
        <h3 style="margin: 0; color: #333;">Huzita–Hatori axioms II</h3>
        <p style="margin: 5px 0; color: #666; font-size: 0.95em;">
            서로 다른 두 점이 있을 때, 두 점을 포갤 수 있는 접는 방법이 유일하게 존재한다
        </p>
    </div>

<div style="margin-bottom: 15px; display: flex; justify-content: center; align-items: center; gap: 15px;">
        <span style="font-weight: bold; font-size: 0.9em;">접기 (Fold)</span>
        <input type="range" id="foldSlider" min="0" max="180" value="0" style="width: 200px; cursor: pointer; accent-color: #e11d48;">
        <span id="foldValue" style="font-weight: bold; color: #e11d48; width: 45px;">0°</span>
    </div>

<div id="canvas-container" style="width: 100%; height: 450px; border: 1px solid #eee; border-radius: 8px; background: #f0f0f0; cursor: grab; overflow: hidden; position: relative;"></div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<script>
window.addEventListener('load', function() {
    const container = document.getElementById('canvas-container');
    const slider = document.getElementById('foldSlider');
    const foldValueTxt = document.getElementById('foldValue');

    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0xfafafa);

    const camera = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 1000);
    camera.position.set(0, 0, 10);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    const ambientLight = new THREE.AmbientLight(0xffffff, 1.0);
    scene.add(ambientLight);

    // 그룹 설정
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

    const foldLine = new THREE.Line(new THREE.BufferGeometry(), new THREE.LineDashedMaterial({ color: '#475569', dashSize: 0.2, gapSize: 0.1 }));
    scene.add(foldLine);

    // 라벨 생성 함수
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

    const p1Mesh = new THREE.Mesh(new THREE.SphereGeometry(0.1, 32, 32), new THREE.MeshBasicMaterial({ color: 0x3b82f6 }));
    const p2Mesh = new THREE.Mesh(new THREE.SphereGeometry(0.1, 32, 32), new THREE.MeshBasicMaterial({ color: 0xe11d48 }));
    
    const p1Label = createLabel('P₁', '#1d4ed8');
    const p2Label = createLabel('P₂', '#be123c');
    const lineLabel = createLabel('l', '#475569', 50);

    scene.add(p1Mesh, p2Mesh, p1Label, p2Label, lineLabel);

    let p1_orig = new THREE.Vector2(-1.5, 1.5);
    let p2_pos = new THREE.Vector2(1.5, -1.5);

    function updatePaper() {
        let N = new THREE.Vector2().subVectors(p2_pos, p1_orig);
        let dist = N.length();
        if (dist < 0.1) return;
        let n = N.clone().normalize();
        let M = new THREE.Vector2().addVectors(p1_orig, p2_pos).multiplyScalar(0.5);

        // 종이 쪼개기 로직 (생략 - 이전과 동일)
        const W = 3, H = 3;
        const pts = [new THREE.Vector2(-W, H), new THREE.Vector2(-W, -H), new THREE.Vector2(W, -H), new THREE.Vector2(W, H)];
        let polyFront = [], polyBack = [];
        pts.forEach((A, i) => {
            let B = pts[(i + 1) % 4];
            let sideA = new THREE.Vector2().subVectors(A, M).dot(n);
            let sideB = new THREE.Vector2().subVectors(B, M).dot(n);
            if (sideA >= 0) polyFront.push(A);
            if (sideA <= 0) polyBack.push(A);
            if ((sideA > 0 && sideB < 0) || (sideA < 0 && sideB > 0)) {
                let t = sideA / (sideA - sideB);
                let I = new THREE.Vector2().lerpVectors(A, B, t);
                polyFront.push(I); polyBack.push(I);
            }
        });

        const sortPts = (p) => {
            let c = p.reduce((a, b) => ({x:a.x+b.x, y:a.y+b.y}), {x:0, y:0});
            c.x/=p.length; c.y/=p.length;
            return p.sort((a,b) => Math.atan2(a.y-c.y, a.x-c.x) - Math.atan2(b.y-c.y, b.x-c.x));
        }

        if (fMeshF.geometry) fMeshF.geometry.dispose();
        if (bMeshF.geometry) bMeshF.geometry.dispose();
        if (polyFront.length > 2) fMeshF.geometry = fMeshB.geometry = new THREE.ShapeGeometry(new THREE.Shape(sortPts(polyFront)));
        if (polyBack.length > 2) {
            let localPoly = polyBack.map(P => {
                let dx = P.x - M.x, dy = P.y - M.y;
                return new THREE.Vector2(dx * n.x + dy * n.y, -dx * n.y + dy * n.x);
            });
            bMeshF.geometry = bMeshB.geometry = new THREE.ShapeGeometry(new THREE.Shape(sortPts(localPoly)));
        }

        hingeBase.position.set(M.x, M.y, 0);
        hingeBase.rotation.z = Math.atan2(n.y, n.x);
        
        const rad = (slider.value * Math.PI) / 180;
        hingeFold.rotation.y = rad;
        hingeFold.position.z = Math.sin(rad/2) * 0.15;

        // P2 고정 위치
        p2Mesh.position.set(p2_pos.x, p2_pos.y, 0.1);
        p2Label.position.set(p2_pos.x, p2_pos.y + 0.4, 0.2);

        // P1 3D 궤적 계산
        let p1_w = new THREE.Vector3(-dist/2, 0, 0);
        p1_w.applyQuaternion(hingeFold.quaternion).add(hingeFold.position).applyQuaternion(hingeBase.quaternion).add(hingeBase.position);
        p1Mesh.position.copy(p1_w);
        p1Label.position.copy(p1_w).add(new THREE.Vector3(0, 0.4, 0.1));

        // [추가] 선 라벨 L 위치 계산
        // 접는 선 위(M 지점 근처)에서 선 방향 벡터에 수직인 방향으로 배치
        let lineDir = new THREE.Vector2(-n.y, n.x); // 선의 방향 벡터
        lineLabel.position.set(M.x + lineDir.x * 0.8, M.y + lineDir.y * 0.8, 0.3);

        foldLine.geometry.setFromPoints([
            new THREE.Vector3(M.x + lineDir.x * 10, M.y + lineDir.y * 10, 0.01),
            new THREE.Vector3(M.x - lineDir.x * 10, M.y - lineDir.y * 10, 0.01)
        ]);
        foldLine.computeLineDistances();
    }

    // 마우스 이벤트 로직 (생략 - 이전과 동일)
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
        const nx = Math.max(-2.9, Math.min(2.9, intersectPoint.x));
        const ny = Math.max(-2.9, Math.min(2.9, intersectPoint.y));
        if (dragging === p1Mesh) p1_orig.set(nx, ny); else p2_pos.set(nx, ny);
        updatePaper();
    });

    container.addEventListener('pointerup', () => { dragging = null; container.style.cursor = 'default'; });
    slider.addEventListener('input', () => { foldValueTxt.innerText = slider.value + "°"; updatePaper(); });
    
    function animate() { requestAnimationFrame(animate); renderer.render(scene, camera); }
    updatePaper();
    animate();
});
</script>