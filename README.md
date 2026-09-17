# Pixel Game
Game to understand how pixels work

<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>משחק יצירת תמונות מפיקסלים</title>
    <!-- html2canvas library for saving element as image -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <style>
        :root {
            --bg-color: #f0f4f8;
            --card-bg: #ffffff;
            --text-color: #333333;
            --accent-color: #4a90e2;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            box-sizing: border-box;
        }

        h1 {
            margin-top: 0;
            color: #2c3e50;
        }

        .game-container {
            background-color: var(--card-bg);
            padding: 20px 30px;
            border-radius: 16px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            align-items: center;
            max-width: 650px;
            width: 100%;
        }

        .header-info {
            display: flex;
            justify-content: space-between;
            width: 100%;
            margin-bottom: 15px;
            font-size: 1.1rem;
            font-weight: bold;
        }

        .main-display {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 30px;
            margin: 20px 0;
            width: 100%;
        }

        /* הלוח הראשי */
        .pixel-grid {
            display: grid;
            gap: 2px;
            background-color: #bbb;
            border: 3px solid #888;
            border-radius: 4px;
            padding: 2px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }

        .pixel {
            width: 100%;
            height: 100%;
            background-color: #e0e0e0;
            box-sizing: border-box;
            border: 1px solid #ccc;
            transition: background-color 0.1s;
        }

        .pixel.active {
            border: 2px solid #ff3366;
            box-shadow: inset 0 0 5px rgba(255,51,102,0.8);
        }

        /* חלונית הפיקסל הבא (בצד ימין) */
        .target-panel {
            display: flex;
            flex-direction: column;
            align-items: center;
            background: #f8f9fa;
            padding: 15px;
            border-radius: 12px;
            border: 2px dashed #a0a0a0;
        }

        .target-pixel-display {
            width: 60px;
            height: 60px;
            border-radius: 8px;
            border: 2px solid #333;
            margin-top: 8px;
            box-shadow: 0 3px 6px rgba(0,0,0,0.15);
        }

        /* מקרא הצבעים/מספרים */
        .palette {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            justify-content: center;
            margin-top: 15px;
            width: 100%;
        }

        .palette-btn {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 50px;
            height: 60px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            box-shadow: 0 3px 6px rgba(0,0,0,0.1);
            transition: transform 0.1s, box-shadow 0.1s;
            color: #fff;
            text-shadow: 0 1px 2px rgba(0,0,0,0.8);
            font-weight: bold;
            font-size: 1.2rem;
        }

        .palette-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 10px rgba(0,0,0,0.15);
        }

        .palette-btn:active {
            transform: translateY(0);
        }

        .palette-btn span {
            font-size: 0.75rem;
            margin-top: 2px;
            opacity: 0.9;
        }

        /* מסך סיכום תוצאה */
        .summary-card {
            display: none;
            flex-direction: column;
            align-items: center;
            width: 100%;
            text-align: center;
        }

        .results-comparison {
            display: flex;
            gap: 20px;
            margin: 15px 0;
            justify-content: center;
        }

        .result-box {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .btn {
            color: white;
            border: none;
            padding: 12px 24px;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 25px;
            cursor: pointer;
            margin: 8px 5px;
            transition: background-color 0.2s, transform 0.1s;
        }
        
        .btn:active {
            transform: scale(0.98);
        }

        .btn-next {
            background-color: #2ecc71;
        }

        .btn-next:hover {
            background-color: #27ae60;
        }

        .btn-save {
            background-color: #3498db;
        }

        .btn-save:hover {
            background-color: #2980b9;
        }

        .btn-restart {
            background-color: #e67e22;
        }

        .btn-restart:hover {
            background-color: #d35400;
        }

        /* כרטיס תעודת סיכום לשמירה כתמונה */
        .summary-export-card {
            background: linear-gradient(135deg, #ffffff 0%, #f7f9fc 100%);
            border: 2px solid #e1e8ed;
            border-radius: 16px;
            padding: 25px;
            width: 90%;
            box-shadow: 0 4px 15px rgba(0,0,0,0.05);
            margin-bottom: 20px;
            box-sizing: border-box;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            margin: 15px 0;
            text-align: right;
        }

        .stat-item {
            background: #ffffff;
            padding: 10px 14px;
            border-radius: 10px;
            border: 1px solid #eef2f5;
            box-shadow: 0 2px 4px rgba(0,0,0,0.02);
        }

        .stat-label {
            font-size: 0.85rem;
            color: #7f8c8d;
            margin-bottom: 4px;
        }

        .stat-value {
            font-size: 1.15rem;
            font-weight: bold;
            color: #2c3e50;
        }

        .final-score-box {
            background: #2c3e50;
            color: #ffffff;
            padding: 15px;
            border-radius: 12px;
            margin-top: 15px;
        }

        .final-score-title {
            font-size: 0.9rem;
            text-transform: uppercase;
            letter-spacing: 1px;
            opacity: 0.8;
        }

        .final-score-number {
            font-size: 2.2rem;
            font-weight: bold;
            color: #f1c40f;
        }
    </style>
</head>
<body>

<div class="game-container">
    <!-- מסך המשחק -->
    <div id="game-screen" style="width: 100%; display: flex; flex-direction: column; align-items: center;">
        <div class="header-info">
            <span id="level-title">שלב 1 (4x4)</span>
            <span id="timer-display">זמן: 00:00</span>
        </div>

        <div class="main-display">
            <!-- לוח המשחק שמתמלא -->
            <div id="grid" class="pixel-grid"></div>

            <!-- חלונית הצגת הפיקסל הבא מימין -->
            <div class="target-panel">
                <span style="font-weight: bold; font-size: 0.9rem;">הפיקסל הבא:</span>
                <div id="target-pixel" class="target-pixel-display"></div>
            </div>
        </div>

        <p style="margin-bottom: 5px; font-weight: bold;">לחץ על המספר המתאים בצבעו:</p>
        <!-- מקרא המקשים/הצבעים -->
        <div id="palette" class="palette"></div>
    </div>

    <!-- מסך סיכום שלב יחיד -->
    <div id="summary-screen" class="summary-card">
        <h2>כל הכבוד! השלמת את השלב! 🎉</h2>
        <p id="summary-name" style="font-size: 1.2rem; font-weight: bold; color: #34495e;"></p>
        <p id="summary-stats"></p>

        <div class="results-comparison">
            <div class="result-box">
                <span>מה שיצרת:</span>
                <div id="user-result-grid" class="pixel-grid" style="margin-top: 5px;"></div>
            </div>
            <div class="result-box">
                <span>התמונה המקורית:</span>
                <div id="target-result-grid" class="pixel-grid" style="margin-top: 5px;"></div>
            </div>
        </div>

        <button class="btn btn-next" onclick="nextLevel()">לשלב הבא ⬅️</button>
    </div>

    <!-- מסך סיכום כללי סופי (תעודת סיום) -->
    <div id="final-summary-screen" class="summary-card">
        <h2>🏆 אלופים! סיימתם את כל השלבים! 🏆</h2>
        <p style="margin-top:0; color:#555;">הנה סיכום הביצועים המלא שלכם:</p>

        <!-- הכרטיס המעוצב שיינצל כתמונה -->
        <div id="export-card" class="summary-export-card">
            <h3 style="margin: 0 0 10px 0; color: #2c3e50;">תעודת אמן פיקסלים 🎨</h3>
            <div style="font-size:0.85rem; color:#7f8c8d; margin-bottom: 15px;">משחק צביעת פיקסלים (שלבים 4x4 עד 10x10)</div>
            
            <div class="stats-grid">
                <div class="stat-item">
                    <div class="stat-label">זמן כולל למשחק:</div>
                    <div class="stat-value" id="final-total-time">00:00</div>
                </div>
                <div class="stat-item">
                    <div class="stat-label">זמן ממוצע לפיקסל:</div>
                    <div class="stat-value" id="final-avg-time">0.00 שניות</div>
                </div>
                <div class="stat-item">
                    <div class="stat-label">פיקסלים נכונים:</div>
                    <div class="stat-value" id="final-correct-pixels" style="color: #2ecc71;">0 / 0</div>
                </div>
                <div class="stat-item">
                    <div class="stat-label">כמות טעויות:</div>
                    <div class="stat-value" id="final-wrong-pixels" style="color: #e74c3c;">0</div>
                </div>
            </div>

            <div class="final-score-box">
                <div class="final-score-title">ניקוד משוקלל סופי</div>
                <div class="final-score-number" id="final-score">0</div>
                <div style="font-size: 0.75rem; opacity: 0.8; margin-top: 4px;">חישוב: (דיוק % × 100) + בונוס מהירות</div>
            </div>
        </div>

        <div>
            <button class="btn btn-save" onclick="saveAsImage()">📸 שמור סיכום כתמונה</button>
            <button class="btn btn-restart" onclick="restartGame()">🔄 משחק חדש</button>
        </div>
    </div>
</div>

<script>
    // Audio Synth Context
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

    function playPixelSound() {
        if (audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = 'sine';
        osc.frequency.setValueAtTime(440, audioCtx.currentTime);
        osc.frequency.exponentialRampToValueAtTime(880, audioCtx.currentTime + 0.08);
        
        gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.08);
        
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        
        osc.start();
        osc.stop(audioCtx.currentTime + 0.08);
    }

    function playLevelCompleteSound() {
        if (audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
        const notes = [261.63, 329.63, 392.00, 523.25];
        notes.forEach((freq, idx) => {
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            const startTime = audioCtx.currentTime + idx * 0.1;
            
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(freq, startTime);
            
            gain.gain.setValueAtTime(0.2, startTime);
            gain.gain.exponentialRampToValueAtTime(0.01, startTime + 0.25);
            
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            
            osc.start(startTime);
            osc.stop(startTime + 0.25);
        });
    }

    function playGameCompleteSound() {
        if (audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
        const notes = [330, 392, 493.88, 523.25, 659.25, 783.99];
        notes.forEach((freq, idx) => {
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            const startTime = audioCtx.currentTime + idx * 0.12;
            
            osc.type = 'sine';
            osc.frequency.setValueAtTime(freq, startTime);
            
            gain.gain.setValueAtTime(0.25, startTime);
            gain.gain.exponentialRampToValueAtTime(0.01, startTime + 0.4);
            
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            
            osc.start(startTime);
            osc.stop(startTime + 0.4);
        });
    }

    const COLOR_MAP = {
        0: { code: '#FFFFFF', name: 'לבן' },
        1: { code: '#2ecc71', name: 'ירוק' },
        2: { code: '#f1c40f', name: 'צהוב' },
        3: { code: '#3498db', name: 'כחול' },
        4: { code: '#e74c3c', name: 'אדום' },
        5: { code: '#e67e22', name: 'כתום' },
        6: { code: '#9b59b6', name: 'סגול' },
        7: { code: '#795548', name: 'חום' },
        8: { code: '#34495e', name: 'שחור/כהה' }
    };

    const LEVELS = [
        {
            size: 4,
            name: "פרח קטן 🌸",
            data: [
                0, 2, 2, 0,
                2, 4, 4, 2,
                0, 1, 1, 0,
                0, 1, 0, 0
            ]
        },
        {
            size: 5,
            name: "בית קטן 🏠",
            data: [
                0, 0, 4, 0, 0,
                0, 4, 4, 4, 0,
                0, 2, 2, 2, 0,
                0, 2, 3, 2, 0,
                0, 2, 2, 2, 0
            ]
        },
        {
            size: 6,
            name: "סמיילי קורץ 😉",
            data: [
                0, 2, 2, 2, 2, 0,
                2, 8, 2, 8, 8, 2,
                2, 2, 2, 2, 2, 2,
                2, 4, 2, 2, 4, 2,
                2, 2, 4, 4, 2, 2,
                0, 2, 2, 2, 2, 0
            ]
        },
        {
            size: 7,
            name: "לב אדום ❤️",
            data: [
                0, 4, 4, 0, 4, 4, 0,
                4, 4, 4, 4, 4, 4, 4,
                4, 4, 4, 4, 4, 4, 4,
                0, 4, 4, 4, 4, 4, 0,
                0, 0, 4, 4, 4, 0, 0,
                0, 0, 0, 4, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0
            ]
        },
        {
            size: 8,
            name: "פטרייה קסומה 🍄",
            data: [
                0, 0, 4, 4, 4, 4, 0, 0,
                0, 4, 4, 0, 4, 0, 4, 0,
                4, 4, 0, 4, 4, 4, 4, 4,
                4, 4, 4, 4, 0, 4, 4, 4,
                0, 0, 0, 2, 2, 0, 0, 0,
                0, 0, 0, 2, 2, 0, 0, 0,
                0, 0, 0, 2, 2, 0, 0, 0,
                0, 1, 1, 1, 1, 1, 1, 0
            ]
        },
        {
            size: 9,
            name: "ברווז צהוב 🐤",
            data: [
                0, 0, 0, 2, 2, 2, 0, 0, 0,
                0, 0, 2, 2, 8, 2, 5, 5, 0,
                0, 0, 2, 2, 2, 2, 5, 0, 0,
                0, 0, 0, 2, 2, 2, 0, 0, 0,
                0, 2, 2, 2, 2, 2, 2, 0, 0,
                2, 2, 2, 2, 2, 2, 2, 2, 0,
                2, 2, 2, 2, 2, 2, 2, 2, 0,
                0, 2, 2, 2, 2, 2, 2, 0, 0,
                0, 0, 3, 3, 3, 3, 0, 0, 0
            ]
        },
        {
            size: 10,
            name: "ספינת חלל 🚀",
            data: [
                0, 0, 0, 0, 4, 4, 0, 0, 0, 0,
                0, 0, 0, 4, 0, 0, 4, 0, 0, 0,
                0, 0, 0, 0, 3, 3, 0, 0, 0, 0,
                0, 0, 0, 0, 3, 3, 0, 0, 0, 0,
                0, 0, 0, 3, 3, 3, 3, 0, 0, 0,
                0, 4, 0, 3, 8, 8, 3, 0, 4, 0,
                0, 4, 0, 3, 3, 3, 3, 0, 4, 0,
                4, 4, 4, 3, 3, 3, 3, 4, 4, 4,
                0, 0, 0, 5, 5, 5, 5, 0, 0, 0,
                0, 0, 0, 2, 0, 0, 2, 0, 0, 0
            ]
        }
    ];

    let currentLevelIdx = 0;
    let currentPixelSequence = [];
    let currentStep = 0;
    let userChoices = [];
    let timerInterval = null;
    let secondsElapsed = 0;

    let gameStats = {
        totalTime: 0,
        totalPixels: 0,
        correctPixels: 0,
        wrongPixels: 0
    };

    function startLevel(levelIdx) {
        currentLevelIdx = levelIdx;
        const level = LEVELS[currentLevelIdx];
        const size = level.size;

        document.getElementById('level-title').innerText = `שלב ${levelIdx + 1} (${size}x${size})`;
        document.getElementById('game-screen').style.display = 'flex';
        document.getElementById('summary-screen').style.display = 'none';
        document.getElementById('final-summary-screen').style.display = 'none';

        currentPixelSequence = [];
        for (let r = 0; r < size; r++) {
            for (let c = size - 1; c >= 0; c--) {
                currentPixelSequence.push(r * size + c);
            }
        }

        currentStep = 0;
        userChoices = new Array(size * size).fill(null);

        buildGrid(document.getElementById('grid'), size, 280);
        buildPalette();
        resetTimer();
        updateStepDisplay();
    }

    function buildGrid(gridEl, size, maxPixelAreaSize) {
        gridEl.innerHTML = '';
        gridEl.style.gridTemplateColumns = `repeat(${size}, 1fr)`;
        gridEl.style.gridTemplateRows = `repeat(${size}, 1fr)`;
        
        const cellSize = Math.floor(maxPixelAreaSize / size);
        gridEl.style.width = `${cellSize * size}px`;
        gridEl.style.height = `${cellSize * size}px`;

        for (let i = 0; i < size * size; i++) {
            const pixel = document.createElement('div');
            pixel.className = 'pixel';
            pixel.id = `grid-pixel-${i}`;
            gridEl.appendChild(pixel);
        }
    }

    function buildPalette() {
        const paletteEl = document.getElementById('palette');
        paletteEl.innerHTML = '';

        const levelData = LEVELS[currentLevelIdx].data;
        const usedColors = [...new Set(levelData)].sort();

        usedColors.forEach(colorNum => {
            const colorInfo = COLOR_MAP[colorNum];
            const btn = document.createElement('button');
            btn.className = 'palette-btn';
            btn.style.backgroundColor = colorInfo.code;
            
            if (colorNum === 0 || colorNum === 2) {
                btn.style.color = '#000';
                btn.style.textShadow = 'none';
            }

            btn.innerHTML = `${colorNum}<span>${colorInfo.name}</span>`;
            btn.onclick = () => handleColorSelection(colorNum);
            paletteEl.appendChild(btn);
        });
    }

    function updateStepDisplay() {
        const size = LEVELS[currentLevelIdx].size;
        
        for (let i = 0; i < size * size; i++) {
            const el = document.getElementById(`grid-pixel-${i}`);
            if (el) el.classList.remove('active');
        }

        if (currentStep < currentPixelSequence.length) {
            const pixelIndex = currentPixelSequence[currentStep];
            const currentEl = document.getElementById(`grid-pixel-${pixelIndex}`);
            if (currentEl) currentEl.classList.add('active');

            const targetColorNum = LEVELS[currentLevelIdx].data[pixelIndex];
            document.getElementById('target-pixel').style.backgroundColor = COLOR_MAP[targetColorNum].code;
        } else {
            finishLevel();
        }
    }

    function handleColorSelection(selectedColorNum) {
        if (currentStep >= currentPixelSequence.length) return;

        playPixelSound();

        const pixelIndex = currentPixelSequence[currentStep];
        userChoices[pixelIndex] = selectedColorNum;

        const pixelEl = document.getElementById(`grid-pixel-${pixelIndex}`);
        pixelEl.style.backgroundColor = COLOR_MAP[selectedColorNum].code;

        currentStep++;
        updateStepDisplay();
    }

    window.addEventListener('keydown', (e) => {
        const key = parseInt(e.key);
        if (!isNaN(key) && COLOR_MAP[key]) {
            const levelData = LEVELS[currentLevelIdx].data;
            if (levelData.includes(key)) {
                handleColorSelection(key);
            }
        }
    });

    function resetTimer() {
        clearInterval(timerInterval);
        secondsElapsed = 0;
        updateTimerDisplay();
        timerInterval = setInterval(() => {
            secondsElapsed++;
            updateTimerDisplay();
        }, 1000);
    }

    function updateTimerDisplay() {
        const mins = Math.floor(secondsElapsed / 60).toString().padStart(2, '0');
        const secs = (secondsElapsed % 60).toString().padStart(2, '0');
        document.getElementById('timer-display').innerText = `זמן: ${mins}:${secs}`;
    }

    function finishLevel() {
        clearInterval(timerInterval);
        playLevelCompleteSound();

        const level = LEVELS[currentLevelIdx];
        const size = level.size;
        const totalPixels = size * size;

        let correctCount = 0;
        let wrongCount = 0;
        for (let i = 0; i < totalPixels; i++) {
            if (userChoices[i] === level.data[i]) {
                correctCount++;
            } else {
                wrongCount++;
            }
        }

        gameStats.totalTime += secondsElapsed;
        gameStats.totalPixels += totalPixels;
        gameStats.correctPixels += correctCount;
        gameStats.wrongPixels += wrongCount;

        const accuracy = Math.round((correctCount / totalPixels) * 100);

        document.getElementById('game-screen').style.display = 'none';
        document.getElementById('summary-screen').style.display = 'flex';

        document.getElementById('summary-name').innerText = `תמונה: ${level.name}`;
        
        const mins = Math.floor(secondsElapsed / 60).toString().padStart(2, '0');
        const secs = (secondsElapsed % 60).toString().padStart(2, '0');
        document.getElementById('summary-stats').innerHTML = `
            זמן ביצוע: <b>${mins}:${secs}</b> | דיוק: <b>${accuracy}%</b> (${correctCount}/${totalPixels} נכונים)
        `;

        renderResultGrid('user-result-grid', size, userChoices, level.data);
        renderResultGrid('target-result-grid', size, level.data, null);
    }

    function renderResultGrid(elementId, size, data, targetData) {
        const gridEl = document.getElementById(elementId);
        buildGrid(gridEl, size, 180);

        for (let i = 0; i < size * size; i++) {
            const pixelEl = gridEl.children[i];
            const colorNum = data[i];
            pixelEl.style.backgroundColor = COLOR_MAP[colorNum] ? COLOR_MAP[colorNum].code : '#fff';
            
            if (targetData && data[i] !== targetData[i]) {
                pixelEl.style.position = 'relative';
                pixelEl.innerHTML = '<span style="color:red; font-weight:bold; position:absolute; top:50%; left:50%; transform:translate(-50%,-50%); font-size:10px;">✕</span>';
            }
        }
    }

    function nextLevel() {
        if (currentLevelIdx + 1 < LEVELS.length) {
            startLevel(currentLevelIdx + 1);
        } else {
            showFinalSummary();
        }
    }

    function showFinalSummary() {
        document.getElementById('summary-screen').style.display = 'none';
        document.getElementById('game-screen').style.display = 'none';
        document.getElementById('final-summary-screen').style.display = 'flex';

        playGameCompleteSound();

        const mins = Math.floor(gameStats.totalTime / 60).toString().padStart(2, '0');
        const secs = (gameStats.totalTime % 60).toString().padStart(2, '0');
        document.getElementById('final-total-time').innerText = `${mins}:${secs}`;

        const avgSeconds = gameStats.totalPixels > 0 ? (gameStats.totalTime / gameStats.totalPixels).toFixed(2) : 0;
        document.getElementById('final-avg-time').innerText = `${avgSeconds} שניות`;

        document.getElementById('final-correct-pixels').innerText = `${gameStats.correctPixels} / ${gameStats.totalPixels}`;
        document.getElementById('final-wrong-pixels').innerText = `${gameStats.wrongPixels}`;

        const accuracyPct = gameStats.totalPixels > 0 ? (gameStats.correctPixels / gameStats.totalPixels) : 0;
        const speedBonus = gameStats.totalTime > 0 ? Math.round((gameStats.correctPixels / gameStats.totalTime) * 500) : 0;
        const weightedScore = Math.round(accuracyPct * 10000 + speedBonus);

        document.getElementById('final-score').innerText = weightedScore.toLocaleString();
    }

    function saveAsImage() {
        const cardNode = document.getElementById('export-card');
        
        html2canvas(cardNode, {
            backgroundColor: null,
            scale: 2
        }).then(canvas => {
            const link = document.createElement('a');
            link.download = 'pixel-game-summary.png';
            link.href = canvas.toDataURL('image/png');
            link.click();
        });
    }

    function restartGame() {
        gameStats = {
            totalTime: 0,
            totalPixels: 0,
            correctPixels: 0,
            wrongPixels: 0
        };
        startLevel(0);
    }

    startLevel(0);
</script>

</body>
</html>
