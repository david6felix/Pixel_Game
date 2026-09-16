# Pixel_Game
game to understand how pixels work

<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>משחק חלליות 90s - כולל סאונד</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #000;
            color: #fff;
            font-family: 'Courier New', Courier, monospace;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            overflow: hidden;
            user-select: none;
        }

        #game-container {
            position: relative;
            box-shadow: 0 0 20px rgba(0, 255, 204, 0.5);
            border: 4px solid #333;
        }

        canvas {
            background-color: #050510;
            display: block;
        }

        .ui-panel {
            position: absolute;
            top: 10px;
            left: 10px;
            right: 10px;
            display: flex;
            justify-content: space-between;
            font-size: 18px;
            font-weight: bold;
            color: #00ffcc;
            text-shadow: 2px 2px #ff0055;
            pointer-events: none;
        }

        #overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.85);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: #fff;
            text-align: center;
        }

        #overlay h1 {
            font-size: 36px;
            color: #ff0055;
            text-shadow: 3px 3px #00ffcc;
            margin-bottom: 10px;
        }

        #overlay p {
            font-size: 16px;
            margin: 5px 0;
            color: #ccc;
        }

        .btn {
            margin-top: 20px;
            padding: 12px 24px;
            font-size: 18px;
            font-family: inherit;
            background: #ff0055;
            color: white;
            border: none;
            cursor: pointer;
            box-shadow: 4px 4px 0px #00ffcc;
            transition: transform 0.1s;
        }

        .btn:hover {
            transform: translate(-2px, -2px);
            box-shadow: 6px 6px 0px #00ffcc;
        }

        .btn:active {
            transform: translate(2px, 2px);
            box-shadow: 2px 2px 0px #00ffcc;
        }

        .controls-hint {
            margin-top: 15px;
            font-size: 12px;
            color: #888;
        }
    </style>
</head>
<body>

<div id="game-container">
    <canvas id="gameCanvas" width="600" height="700"></canvas>
    
    <div class="ui-panel">
        <div id="scoreDisplay">ניקוד: 0</div>
        <div id="livesDisplay">חיים: ♥♥♥</div>
        <div id="stageDisplay">שלב: 1</div>
    </div>

    <div id="overlay">
        <h1 id="overlayTitle">חלליות 1990</h1>
        <p id="overlaySub">לחץ על התחל כדי לשחק!</p>
        <button class="btn" id="startBtn" onclick="startGame()">התחל משחק</button>
        <div class="controls-hint">
            מקשים: חיצים לזז | רווח לירי מתמשך | P להשהייה
        </div>
    </div>
</div>

<script>
    // --- מנוע אודיו (Web Audio API) ---
    const AudioContext = window.AudioContext || window.webkitAudioContext;
    let audioCtx = null;
    let musicInterval = null;
    let isMuted = false;

    function initAudio() {
        if (!audioCtx) {
            audioCtx = new AudioContext();
        }
        if (audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
    }

    // אפקטי סאונד retro
    function playSound(type) {
        if (!audioCtx) return;

        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);

        const now = audioCtx.currentTime;

        if (type === 'shoot') {
            osc.type = 'square';
            osc.frequency.setValueAtTime(600, now);
            osc.frequency.exponentialRampToValueAtTime(100, now + 0.1);
            gain.gain.setValueAtTime(0.1, now);
            gain.gain.linearRampToValueAtTime(0.01, now + 0.1);
            osc.start(now);
            osc.stop(now + 0.1);
        } else if (type === 'explosion') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(150, now);
            osc.frequency.exponentialRampToValueAtTime(30, now + 0.3);
            gain.gain.setValueAtTime(0.2, now);
            gain.gain.linearRampToValueAtTime(0.01, now + 0.3);
            osc.start(now);
            osc.stop(now + 0.3);
        } else if (type === 'hit') {
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(120, now);
            osc.frequency.linearRampToValueAtTime(60, now + 0.15);
            gain.gain.setValueAtTime(0.2, now);
            gain.gain.linearRampToValueAtTime(0.01, now + 0.15);
            osc.start(now);
            osc.stop(now + 0.15);
        } else if (type === 'gameover') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(300, now);
            osc.frequency.linearRampToValueAtTime(80, now + 0.6);
            gain.gain.setValueAtTime(0.3, now);
            gain.gain.linearRampToValueAtTime(0.01, now + 0.6);
            osc.start(now);
            osc.stop(now + 0.6);
        }
    }

    // מוזיקת רקע 8-bit בלופ
    const bgNotes = [220, 247, 261, 293, 329, 293, 261, 247]; // סולם מינורי retro
    let noteIndex = 0;

    function startBgMusic() {
        stopBgMusic();
        musicInterval = setInterval(() => {
            if (!audioCtx || isPaused || !gameRunning) return;
            
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.type = 'triangle';
            
            let freq = bgNotes[noteIndex % bgNotes.length];
            osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
            
            gain.gain.setValueAtTime(0.03, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.2);
            
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            
            osc.start();
            osc.stop(audioCtx.currentTime + 0.2);
            
            noteIndex++;
        }, 200);
    }

    function stopBgMusic() {
        if (musicInterval) clearInterval(musicInterval);
    }

    // --- הגדרות משחק ---
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    const overlay = document.getElementById('overlay');
    const overlayTitle = document.getElementById('overlayTitle');
    const overlaySub = document.getElementById('overlaySub');
    const startBtn = document.getElementById('startBtn');

    let gameRunning = false;
    let isPaused = false;
    let score = 0;
    let lives = 3;
    let stage = 1;

    let keys = {};
    let lastShootTime = 0;
    const shootInterval = 150; // זמן בין יריות (מילישניות)

    // שחקן
    const player = {
        x: canvas.width / 2 - 20,
        y: canvas.height - 60,
        width: 40,
        height: 40,
        speed: 6,
        color: '#00ffcc'
    };

    let bullets = [];
    let enemies = [];
    let particles = [];
    let stars = [];

    // יצירת כוכבים לרקע
    for (let i = 0; i < 60; i++) {
        stars.push({
            x: Math.random() * canvas.width,
            y: Math.random() * canvas.height,
            size: Math.random() * 2 + 1,
            speed: Math.random() * 3 + 0.5
        });
    }

    // מאזיני מקלדת
    window.addEventListener('keydown', e => {
        keys[e.code] = true;
        
        if (e.code === 'KeyP' && gameRunning) {
            togglePause();
        }
    });

    window.addEventListener('keyup', e => {
        keys[e.code] = false;
    });

    function togglePause() {
        isPaused = !isPaused;
        if (isPaused) {
            overlayTitle.innerText = "מושהה";
            overlaySub.innerText = "לחץ P או המשך כדי לחזור";
            startBtn.innerText = "המשך";
            overlay.style.display = 'flex';
        } else {
            overlay.style.display = 'none';
            requestAnimationFrame(gameLoop);
        }
    }

    function startGame() {
        initAudio();
        
        if (isPaused) {
            togglePause();
            return;
        }

        score = 0;
        lives = 3;
        stage = 1;
        bullets = [];
        enemies = [];
        particles = [];
        player.x = canvas.width / 2 - 20;
        
        updateUI();
        overlay.style.display = 'none';
        gameRunning = true;

        startBgMusic();
        requestAnimationFrame(gameLoop);
    }

    function updateUI() {
        document.getElementById('scoreDisplay').innerText = `ניקוד: ${score}`;
        document.getElementById('livesDisplay').innerText = `חיים: ${'♥'.repeat(lives)}`;
        document.getElementById('stageDisplay').innerText = `שלב: ${stage}`;
    }

    function spawnEnemies() {
        if (enemies.length === 0) {
            const rows = 2 + stage;
            const cols = 7;
            for (let r = 0; r < rows; r++) {
                for (let c = 0; c < cols; c++) {
                    enemies.push({
                        x: 60 + c * 70,
                        y: 40 + r * 45,
                        width: 35,
                        height: 30,
                        color: r % 2 === 0 ? '#ff0055' : '#ffcc00',
                        vx: 1.5 + stage * 0.3
                    });
                }
            }
        }
    }

    function createExplosion(x, y, color) {
        playSound('explosion');
        for (let i = 0; i < 15; i++) {
            particles.push({
                x: x,
                y: y,
                vx: (Math.random() - 0.5) * 6,
                vy: (Math.random() - 0.5) * 6,
                size: Math.random() * 3 + 1,
                life: 20,
                color: color
            });
        }
    }

    // לולאת המשחק הראשי
    function gameLoop(timestamp) {
        if (!gameRunning || isPaused) return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // --- 1. עדכון וציור כוכבים ---
        ctx.fillStyle = '#fff';
        stars.forEach(star => {
            star.y += star.speed;
            if (star.y > canvas.height) star.y = 0;
            ctx.fillRect(star.x, star.y, star.size, star.size);
        });

        // --- 2. תנועת שחקן ---
        if (keys['ArrowLeft'] && player.x > 0) player.x -= player.speed;
        if (keys['ArrowRight'] && player.x < canvas.width - player.width) player.x += player.speed;

        // ירי רציף בלחיצה ממושכת
        if (keys['Space'] && timestamp - lastShootTime > shootInterval) {
            bullets.push({
                x: player.x + player.width / 2 - 3,
                y: player.y,
                width: 6,
                height: 12,
                speed: 8
            });
            playSound('shoot');
            lastShootTime = timestamp;
        }

        // ציור שחקן (חללית retro)
        ctx.fillStyle = player.color;
        ctx.beginPath();
        ctx.moveTo(player.x + player.width / 2, player.y);
        ctx.lineTo(player.x, player.y + player.height);
        ctx.lineTo(player.x + player.width, player.y + player.height);
        ctx.closePath();
        ctx.fill();

        // --- 3. עדכון קליעים ---
        ctx.fillStyle = '#ffff00';
        for (let i = bullets.length - 1; i >= 0; i--) {
            let b = bullets[i];
            b.y -= b.speed;
            ctx.fillRect(b.x, b.y, b.width, b.height);

            if (b.y < 0) bullets.splice(i, 1);
        }

        // --- 4. עדכון אויבים ---
        spawnEnemies();

        let shiftDown = false;
        enemies.forEach(e => {
            e.x += e.vx;
            if (e.x <= 10 || e.x + e.width >= canvas.width - 10) {
                shiftDown = true;
            }
        });

        if (shiftDown) {
            enemies.forEach(e => {
                e.vx *= -1;
                e.y += 15;
            });
        }

        // ציור אויבים ובדיקת פגיעות
        for (let ei = enemies.length - 1; ei >= 0; ei--) {
            let e = enemies[ei];

            // ציור אויב
            ctx.fillStyle = e.color;
            ctx.fillRect(e.x, e.y, e.width, e.height);

            // התנגשות עם קליעים
            for (let bi = bullets.length - 1; bi >= 0; bi--) {
                let b = bullets[bi];
                if (
                    b.x < e.x + e.width &&
                    b.x + b.width > e.x &&
                    b.y < e.y + e.height &&
                    b.y + b.height > e.y
                ) {
                    createExplosion(e.x + e.width / 2, e.y + e.height / 2, e.color);
                    enemies.splice(ei, 1);
                    bullets.splice(bi, 1);
                    score += 10;
                    updateUI();

                    // מעבר שלב
                    if (enemies.length === 0) {
                        stage++;
                        updateUI();
                    }
                    break;
                }
            }

            // התנגשות אויב בשחקן או הגעה לתחתית
            if (e.y + e.height >= player.y || e.y + e.height >= canvas.height) {
                lives--;
                playSound('hit');
                updateUI();
                createExplosion(player.x + player.width / 2, player.y, '#00ffcc');
                enemies.splice(ei, 1);

                if (lives <= 0) {
                    gameOver();
                    return;
                }
            }
        }

        // --- 5. חלקיקים (פיצוצים) ---
        for (let pi = particles.length - 1; pi >= 0; pi--) {
            let p = particles[pi];
            p.x += p.vx;
            p.y += p.vy;
            p.life--;
            ctx.fillStyle = p.color;
            ctx.fillRect(p.x, p.y, p.size, p.size);

            if (p.life <= 0) particles.splice(pi, 1);
        }

        requestAnimationFrame(gameLoop);
    }

    function gameOver() {
        gameRunning = false;
        stopBgMusic();
        playSound('gameover');
        
        overlayTitle.innerText = "GAME OVER";
        overlaySub.innerText = `הניקוד הסופי שלך: ${score}`;
        startBtn.innerText = "שחק שוב";
        overlay.style.display = 'flex';
    }
</script>

</body>
</html>
