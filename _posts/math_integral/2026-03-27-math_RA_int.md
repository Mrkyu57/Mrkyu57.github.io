---
layout: post
title: "미적분학"
koreantitle: 원통 셸 적분법 3D
englishtitle: 3D Cylindrical Shell Method
color: "#8354ce"
permalink: /calculus_integration_3d/
---

<!-- Three.js 라이브러리 매핑 (최신 브라우저 대응) -->
<script type="importmap">
  {
    "imports": {
      "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
      "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
    }
  }
</script>

<div id="physics-container-3d" style="width: 900px; margin: 20px auto; background: #fff; border-radius: 12px; box-shadow: 0 8px 24px rgba(0,0,0,0.1); border: 1px solid #eee; overflow: hidden;">
    <div style="padding: 15px; background: #fcfcfc; border-bottom: 1px solid #f0f0f0; display: flex; justify-content: center; align-items: center; gap: 10px;">
        <h3 style="margin: 0; font-family: 'Times New Roman', serif;">3D Cylindrical Shell Method</h3>
        <button id="start" style="width: 45px; height: 35px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px;">▶</button>
        <button id="reset" style="width: 45px; height: 35px; cursor: pointer; background: #475569; color: white; border: none; border-radius: 6px;">&#8635;</button>
        <div style="width: 1px; height: 20px; background: #ddd;"></div>
        <button id="btnCurve" style="padding: 5px 12px; cursor: pointer; background: #8354ce; color: white; border: none; border-radius: 6px;">f(x)</button>
        <button id="btnShell" style="padding: 5px 12px; cursor: pointer; background: #8354ce; color: white; border: none; border-radius: 6px;">Shell</button>
        <button id="btnFull" style="padding: 5px 12px; cursor: pointer; background: #f1f5f9; color: #333; border: 1px solid #ddd; border-radius: 6px;">Full Solid</button>
    </div>
    <div id="canvas-container" style="width: 900px; height: 500px; background: #f9fafb; position: relative;">
        <!-- 로딩 표시 (라이브러리 로딩 확인용) -->
        <div id="loading-msg" style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-family: sans-serif; color: #94a3b8;">
            Loading 3D Engine...
        </div>
    </div>
</div>

<script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

    // 1. 기본 셋팅
    const container = document.getElementById('canvas-container');
    const loadingMsg = document.getElementById('loading-msg');
    
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0xf9fafb);

    const camera = new THREE.PerspectiveCamera(45, 900 / 500, 0.1, 1000);
    camera.position.set(8, 6, 10);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(900, 500);
    container.appendChild(renderer.domElement);
    loadingMsg.style.display = 'none'; // 로딩 완료 시 메시지 숨김

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;

    // 2. 조명 및 가이드
    scene.add(new THREE.AmbientLight(0xffffff, 0.7));
    const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
    dirLight.position.set(5, 10, 7);
    scene.add(dirLight);
    
    scene.add(new THREE.GridHelper(12, 12, 0xddd, 0xeee));
    const axes = new THREE.AxesHelper(6);
    scene.add(axes);

    // 3. 수학 모델링 ( f(x) = -0.2x^2 + 4 )
    const maxX = 4;
    const f = (x) => -0.25 * x * x + 4;

    // (A) f(x) 곡선
    const points = [];
    for(let x=0; x<=maxX; x+=0.1) points.push(new THREE.Vector3(x, f(x), 0));
    const curveGeo = new THREE.BufferGeometry().setFromPoints(points);
    const curveLine = new THREE.Line(curveGeo, new THREE.LineBasicMaterial({ color: 0x334155, linewidth: 2 }));
    scene.add(curveLine);

    // (B) 원통 셸 (Cylinder)
    const shellGeo = new THREE.CylinderGeometry(1, 1, 1, 64, 1, true);
    const shellMat = new THREE.MeshPhongMaterial({ color: 0x8354ce, side: THREE.DoubleSide, transparent: true, opacity: 0.6 });
    const shell = new THREE.Mesh(shellGeo, shellMat);
    scene.add(shell);

    // (C) 전체 회전체 (Lathe)
    const lathePoints = [];
    for(let x=0; x<=maxX; x+=0.1) lathePoints.push(new THREE.Vector2(x, f(x)));
    lathePoints.push(new THREE.Vector2(0, 0));
    const fullSolid = new THREE.Mesh(
        new THREE.LatheGeometry(lathePoints, 64),
        new THREE.MeshPhongMaterial({ color: 0x8354ce, transparent: true, opacity: 0.2, side: THREE.DoubleSide })
    );
    fullSolid.visible = false;
    scene.add(fullSolid);

    // 4. 애니메이션 및 제어 로직
    let isMoving = false;
    let t = 0;

    const startBtn = document.getElementById('start');
    startBtn.onclick = () => {
        isMoving = !isMoving;
        startBtn.innerText = isMoving ? '▮▮' : '▶';
        startBtn.style.background = isMoving ? '#ffc107' : '#475569';
    };

    document.getElementById('reset').onclick = () => { t = 0; isMoving = false; startBtn.innerText = '▶'; startBtn.style.background = '#475569'; };
    document.getElementById('btnCurve').onclick = function() { curveLine.visible = !curveLine.visible; this.style.background = curveLine.visible ? '#8354ce' : '#f1f5f9'; this.style.color = curveLine.visible ? '#fff' : '#333'; };
    document.getElementById('btnShell').onclick = function() { shell.visible = !shell.visible; this.style.background = shell.visible ? '#8354ce' : '#f1f5f9'; this.style.color = shell.visible ? '#fff' : '#333'; };
    document.getElementById('btnFull').onclick = function() { fullSolid.visible = !fullSolid.visible; this.style.background = fullSolid.visible ? '#8354ce' : '#f1f5f9'; this.style.color = fullSolid.visible ? '#fff' : '#333'; };

    function animate() {
        requestAnimationFrame(animate);
        if (isMoving) {
            t += 0.02;
            if (t > maxX) t = 0;
            
            const currentR = t;
            const currentH = f(t);
            
            shell.scale.set(currentR, currentH, currentR);
            shell.position.y = currentH / 2;
        }
        controls.update();
        renderer.render(scene, camera);
    }
    animate();

    // 윈도우 리사이즈 대응
    window.addEventListener('resize', () => {
        const w = 900;
        const h = 500;
        renderer.setSize(w, h);
        camera.aspect = w / h;
        camera.updateProjectionMatrix();
    });
</script>