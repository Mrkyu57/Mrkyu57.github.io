---
layout: post
title: "물리학 - 파동 (벡터 시각화)"
---

<style>
    #wave-container {
        width: 100%;
        max-width: 850px;
        margin: 20px auto;
        background: #ffffff;
        border-radius: 12px;
        box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
        border: 1px solid #eee;
        overflow: hidden;
        user-select: none;
    }
    .wave-header {
        padding: 15px;
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 20px;
        background: #fcfcfc;
        border-bottom: 1px solid #eee;
    }
    #wave-canvas {
        display: block;
        width: 100%;
        height: auto;
        background: #ffffff;
        touch-action: none;
        cursor: crosshair;
    }
    .status-badge {
        font-size: 13px;
        font-weight: bold;
        color: #555;
        font-family: 'Times New Roman', serif;
    }
</style>

<div id="wave-container">
    <div class="wave-header">
        <span class="status-badge" id="txt-n">Mode: n = 1</span>
        <button id="btn-stop" style="padding: 6px 12px; cursor: pointer; background: #000; color: #fff; border: none; border-radius: 4px; font-size: 12px;">MUTE / RESET</button>
        <span style="font-size: 12px; color: #999;">현을 당겨보세요 (끝으로 갈수록 고음/노드증가)</span>
    </div>
    <canvas id="wave-canvas" width="800" height="400"></canvas>
</div>

<script>
(function() {
    const canvas = document.getElementById('wave-canvas');
    const ctx = canvas.getContext('2d');
    const stopBtn = document.getElementById('btn-stop');
    const statusTxt = document.getElementById('txt-n');

    let audioCtx = null, oscillator = null, gainNode = null;
    const L_START = 100, L_END = 700, L = 600, CENTER_Y = 200;
    const BASE_FREQ = 220;

    let amplitude = 0, phase = 0, n = 1, isDragging = false;

    function initAudio() {
        if (audioCtx) return;
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        oscillator = audioCtx.createOscillator();
        gainNode = audioCtx.createGain();
        oscillator.type = 'sine';
        gainNode.gain.setValueAtTime(0, audioCtx.currentTime);
        oscillator.connect(gainNode);
        gainNode.connect(audioCtx.destination);
        oscillator.start();
    }

    function drawArrow(x, y, vx, vy, color, label, sub) {
        if (Math.abs(vx) < 1 && Math.abs(vy) < 1) return;
        ctx.save();
        ctx.strokeStyle = color;
        ctx.fillStyle = color;
        ctx.lineWidth = 1.5;

        // 화살표 선
        ctx.beginPath();
        ctx.moveTo(x, y);
        ctx.lineTo(x + vx, y + vy);
        ctx.stroke();

        // 화살표 머리
        const angle = Math.atan2(vy, vx);
        const headLen = 8;
        ctx.beginPath();
        ctx.moveTo(x + vx, y + vy);
        ctx.lineTo(x + vx - headLen * Math.cos(angle - Math.PI / 6), y + vy - headLen * Math.sin(angle - Math.PI / 6));
        ctx.lineTo(x + vx - headLen * Math.cos(angle + Math.PI / 6), y + vy - headLen * Math.sin(angle + Math.PI / 6));
        ctx.closePath();
        ctx.fill();

        // 라벨 (Times New Roman 스타일)
        ctx.font = "italic 16px 'Times New Roman'";
        const textX = x + vx / 2 + 10 * Math.cos(angle - Math.PI / 2);
        const textY = y + vy / 2 + 10 * Math.sin(angle - Math.PI / 2);
        ctx.fillText(label, textX, textY);
        if (sub) {
            ctx.font = "bold 10px 'Times New Roman'";
            ctx.fillText(sub, textX + 12, textY + 5);
        }
        ctx.restore();
    }

    function getMousePos(e) {
        const rect = canvas.getBoundingClientRect();
        const clientX = e.touches ? e.touches[0].clientX : e.clientX;
        const clientY = e.touches ? e.touches[0].clientY : e.clientY;
        const scaleX = canvas.width / rect.width;
        const scaleY = canvas.height / rect.height;
        return { x: (clientX - rect.left) * scaleX, y: (clientY - rect.top) * scaleY };
    }

    function update(pos) {
        amplitude = pos.y - CENTER_Y;
        const dist = Math.abs(pos.x - 400);
        n = Math.floor(1 + (dist / 300) * 8);
        n = Math.max(1, Math.min(n, 8));
        statusTxt.innerText = `Mode: n = ${n}`;
        
        if (oscillator) oscillator.frequency.setTargetAtTime(BASE_FREQ * n, audioCtx.currentTime, 0.05);
        if (gainNode) gainNode.gain.setTargetAtTime(Math.min(Math.abs(amplitude) / 250, 0.4), audioCtx.currentTime, 0.05);
    }

    canvas.addEventListener('mousedown', (e) => {
        initAudio();
        if (audioCtx.state === 'suspended') audioCtx.resume();
        isDragging = true;
        update(getMousePos(e));
    });
    window.addEventListener('mousemove', (e) => { if (isDragging) update(getMousePos(e)); });
    window.addEventListener('mouseup', () => isDragging = false);
    canvas.addEventListener('touchstart', (e) => { initAudio(); isDragging = true; update(getMousePos(e)); e.preventDefault(); }, {passive: false});
    window.addEventListener('touchmove', (e) => { if (isDragging) { update(getMousePos(e)); e.preventDefault(); } }, {passive: false});
    window.addEventListener('touchend', () => isDragging = false);

    stopBtn.addEventListener('click', () => { amplitude = 0; if (gainNode) gainNode.gain.setTargetAtTime(0, audioCtx.currentTime, 0.05); });

    function render() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        // 평형 가이드
        ctx.strokeStyle = "#eee";
        ctx.beginPath(); ctx.moveTo(L_START, CENTER_Y); ctx.lineTo(L_END, CENTER_Y); ctx.stroke();

        const timeFactor = isDragging ? 1 : Math.cos(phase);
        const currentAmp = amplitude * timeFactor;

        // 현 그리기
        ctx.beginPath();
        ctx.lineWidth = 2.5;
        ctx.strokeStyle = "#000";
        for (let x = L_START; x <= L_END; x += 1) {
            const relX = x - L_START;
            const y = CENTER_Y + currentAmp * Math.sin((n * Math.PI * relX) / L);
            if (x === L_START) ctx.moveTo(x, y); else ctx.lineTo(x, y);
        }
        ctx.stroke();

        // 벡터 화살표 계산
        // 1. 진폭 벡터 (첫 번째 마루 위치)
        const ampPosX = L_START + L / (2 * n);
        const ampVal = currentAmp; 
        drawArrow(ampPosX, CENTER_Y, 0, ampVal, "#e74c3c", "A", "");

        // 2. 파장 벡터 (한 주기 거리)
        const lambda = (2 * L) / n;
        const lambdaShow = Math.min(lambda, L_END - L_START); // 화면 밖으로 나가지 않게 조절
        drawArrow(L_START, CENTER_Y - 40, lambdaShow, 0, "#3498db", "λ", "");

        // 고정점
        ctx.fillStyle = "#000";
        ctx.beginPath(); ctx.arc(L_START, CENTER_Y, 5, 0, 7); ctx.fill();
        ctx.beginPath(); ctx.arc(L_END, CENTER_Y, 5, 0, 7); ctx.fill();

        if (!isDragging) phase += 0.07 * (1 + n * 0.05);
        requestAnimationFrame(render);
    }
    render();
})();
</script>