<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CYBER_GRID // SHADOW PROTOCOL</title>
    <style>
        :root {
            --bg-color: #0d1117;
            --panel-bg: #161b22;
            --border-color: #30363d;
            --ice-blue: #79c0ff;
            --deep-purple: #bc8cff;
            --dark-gray: #21262d;
            --text-dim: #8b949e;
            --glow-blue: rgba(121, 192, 255, 0.4);
            --glow-purple: rgba(188, 140, 255, 0.4);
            --danger: #ff7b72;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Courier New', Courier, monospace;
            user-select: none;
        }

        body {
            background-color: var(--bg-color);
            color: #c9d1d9;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            position: relative;
        }

        /* Ambient Fog & Scanlines */
        body::before {
            content: "";
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: 
                radial-gradient(circle at 50% 30%, rgba(188, 140, 255, 0.05) 0%, transparent 60%),
                radial-gradient(circle at 80% 80%, rgba(121, 192, 255, 0.05) 0%, transparent 50%);
            pointer-events: none;
            z-index: 1;
        }

        .scanlines {
            position: fixed;
            top: 0; left: 0; width: 100vw; height: 100vh;
            background: linear-gradient(
                rgba(18, 16, 16, 0) 50%, 
                rgba(0, 0, 0, 0.25) 50%
            );
            background-size: 100% 4px;
            z-index: 10;
            pointer-events: none;
            opacity: 0.6;
        }

        /* Container */
        .game-container {
            position: relative;
            z-index: 2;
            background-color: var(--panel-bg);
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 2rem;
            box-shadow: 0 0 30px rgba(0, 0, 0, 0.8), 0 0 15px var(--glow-purple);
            width: 100%;
            max-width: 440px;
            text-align: center;
            backdrop-filter: blur(5px);
        }

        /* Header */
        .title {
            font-size: 1.4rem;
            letter-spacing: 3px;
            color: var(--ice-blue);
            text-shadow: 0 0 8px var(--glow-blue);
            margin-bottom: 0.3rem;
            text-transform: uppercase;
        }

        .subtitle {
            font-size: 0.75rem;
            color: var(--text-dim);
            letter-spacing: 2px;
            margin-bottom: 1.5rem;
        }

        /* Stats Dashboard */
        .dashboard {
            display: flex;
            justify-content: space-between;
            background: var(--bg-color);
            border: 1px solid var(--border-color);
            padding: 0.75rem 1rem;
            border-radius: 4px;
            margin-bottom: 1.5rem;
            font-size: 0.85rem;
        }

        .stat-box {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .stat-label {
            color: var(--text-dim);
            font-size: 0.65rem;
            letter-spacing: 1px;
            margin-bottom: 0.2rem;
        }

        .stat-value {
            color: var(--deep-purple);
            font-weight: bold;
            font-size: 1.1rem;
            text-shadow: 0 0 5px var(--glow-purple);
        }

        /* Game Grid */
        .grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 12px;
            margin-bottom: 1.5rem;
            aspect-ratio: 1;
        }

        .node {
            background-color: var(--dark-gray);
            border: 1px solid var(--border-color);
            border-radius: 6px;
            cursor: pointer;
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
            position: relative;
            overflow: hidden;
        }

        .node::after {
            content: '';
            position: absolute;
            top: 50%; left: 50%;
            width: 6px; height: 6px;
            background: var(--text-dim);
            border-radius: 50%;
            transform: translate(-50%, -50%);
            opacity: 0.3;
            transition: all 0.2s;
        }

        .node:hover {
            border-color: var(--ice-blue);
            box-shadow: inset 0 0 10px var(--glow-blue);
        }

        .node:hover::after {
            background: var(--ice-blue);
            opacity: 0.8;
            box-shadow: 0 0 6px var(--ice-blue);
        }

        /* Active/State Classes */
        .node.active-blue {
            background-color: rgba(121, 192, 255, 0.2);
            border-color: var(--ice-blue);
            box-shadow: 0 0 15px var(--glow-blue), inset 0 0 10px var(--ice-blue);
        }
        .node.active-blue::after {
            background: var(--ice-blue);
            box-shadow: 0 0 10px var(--ice-blue);
            opacity: 1;
        }

        .node.active-purple {
            background-color: rgba(188, 140, 255, 0.25);
            border-color: var(--deep-purple);
            box-shadow: 0 0 15px var(--glow-purple), inset 0 0 10px var(--deep-purple);
        }
        .node.active-purple::after {
            background: var(--deep-purple);
            box-shadow: 0 0 10px var(--deep-purple);
            opacity: 1;
        }

        .node.error {
            background-color: rgba(255, 123, 114, 0.3);
            border-color: var(--danger);
            box-shadow: 0 0 15px rgba(255, 123, 114, 0.5);
        }

        /* Terminal Status Message */
        .status-terminal {
            font-size: 0.8rem;
            min-height: 1.2rem;
            color: var(--ice-blue);
            margin-bottom: 1.5rem;
            letter-spacing: 1px;
            text-shadow: 0 0 5px var(--glow-blue);
        }

        /* Controls */
        .btn {
            background: transparent;
            color: var(--ice-blue);
            border: 1px solid var(--ice-blue);
            padding: 0.75rem 1.5rem;
            font-size: 0.85rem;
            letter-spacing: 2px;
            border-radius: 4px;
            cursor: pointer;
            transition: all 0.3s ease;
            width: 100%;
            text-transform: uppercase;
        }

        .btn:hover {
            background: var(--ice-blue);
            color: var(--bg-color);
            box-shadow: 0 0 15px var(--glow-blue);
            font-weight: bold;
        }

        .btn:disabled {
            border-color: var(--border-color);
            color: var(--text-dim);
            cursor: not-allowed;
            box-shadow: none;
            background: transparent;
        }

        /* Footer */
        .footer {
            margin-top: 1.5rem;
            font-size: 0.65rem;
            color: var(--text-dim);
            letter-spacing: 1px;
        }
    </style>
</head>
<body>

    <div class="scanlines"></div>

    <div class="game-container">
        <div class="title">SHADOW PROTOCOL</div>
        <div class="subtitle">CYBER_GRID // MEMORY OVERRIDE</div>

        <div class="dashboard">
            <div class="stat-box">
                <span class="stat-label">LEVEL</span>
                <span class="stat-value" id="level">01</span>
            </div>
            <div class="stat-box">
                <span class="stat-label">SCORE</span>
                <span class="stat-value" id="score">0000</span>
            </div>
            <div class="stat-box">
                <span class="stat-label">HIGHSCORE</span>
                <span class="stat-value" id="highscore">0000</span>
            </div>
        </div>

        <div class="grid" id="grid">
            <div class="node" data-id="0"></div>
            <div class="node" data-id="1"></div>
            <div class="node" data-id="2"></div>
            <div class="node" data-id="3"></div>
            <div class="node" data-id="4"></div>
            <div class="node" data-id="5"></div>
            <div class="node" data-id="6"></div>
            <div class="node" data-id="7"></div>
            <div class="node" data-id="8"></div>
        </div>

        <div class="status-terminal" id="status">PRESS START TO INITIATE</div>

        <button class="btn" id="start-btn">INITIALIZE PROTOCOL</button>

        <div class="footer">SYSTEM STATUS: ENCRYPTED // NIGHT MODE ACTIVE</div>
    </div>

    <script>
        // Web Audio API Synthesizer
        const AudioContext = window.AudioContext || window.webkitAudioContext;
        let audioCtx;

        function initAudio() {
            if (!audioCtx) {
                audioCtx = new AudioContext();
            }
        }

        function playTone(freq, duration = 0.15, type = 'sine') {
            if (!audioCtx) return;
            try {
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                
                osc.type = type;
                osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
                
                gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + duration);
                
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                
                osc.start();
                osc.stop(audioCtx.currentTime + duration);
            } catch (e) {
                console.error(e);
            }
        }

        // Frequencies for 9 nodes (Dark Ambient Synth Scale)
        const nodeNotes = [220, 246.94, 261.63, 293.66, 329.63, 349.23, 392.00, 440.00, 493.88];

        // Game Logic State
        let sequence = [];
        let playerSequence = [];
        let level = 1;
        let score = 0;
        let highScore = localStorage.getItem('shadow_protocol_hs') || 0;
        let isPlaying = false;
        let isPlayerTurn = false;

        // UI Elements
        const levelEl = document.getElementById('level');
        const scoreEl = document.getElementById('score');
        const highscoreEl = document.getElementById('highscore');
        const statusEl = document.getElementById('status');
        const startBtn = document.getElementById('start-btn');
        const nodes = document.querySelectorAll('.node');

        highscoreEl.textContent = String(highScore).padStart(4, '0');

        startBtn.addEventListener('click', () => {
            initAudio();
            resetGame();
            startNextRound();
        });

        nodes.forEach(node => {
            node.addEventListener('click', (e) => {
                if (!isPlayerTurn || !isPlaying) return;
                
                const id = parseInt(node.getAttribute('data-id'));
                handlePlayerInput(id, node);
            });
        });

        function resetGame() {
            sequence = [];
            playerSequence = [];
            level = 1;
            score = 0;
            isPlaying = true;
            updateStats();
            startBtn.disabled = true;
            startBtn.textContent = 'TRANSMITTING...';
        }

        function updateStats() {
            levelEl.textContent = String(level).padStart(2, '0');
            scoreEl.textContent = String(score).padStart(4, '0');
            highscoreEl.textContent = String(highScore).padStart(4, '0');
        }

        function startNextRound() {
            playerSequence = [];
            isPlayerTurn = false;
            statusEl.textContent = 'RECEIVING DATA PATTERN...';
            
            // Add next node to sequence
            const nextNode = Math.floor(Math.random() * 9);
            sequence.push(nextNode);

            // Play sequence
            let i = 0;
            const interval = setInterval(() => {
                if (i >= sequence.length) {
                    clearInterval(interval);
                    setTimeout(() => {
                        isPlayerTurn = true;
                        statusEl.textContent = 'OVERRIDE: REPEAT PATTERN';
                        startBtn.textContent = 'AWAITING INPUT...';
                    }, 400);
                    return;
                }
                
                activateNode(sequence[i], 'active-purple', 300);
                i++;
            }, 600);
        }

        function activateNode(id, cssClass, duration = 200) {
            const node = nodes[id];
            node.classList.add(cssClass);
            playTone(nodeNotes[id], duration / 1000, 'sine');
            
            setTimeout(() => {
                node.classList.remove(cssClass);
            }, duration);
        }

        function handlePlayerInput(id, node) {
            playerSequence.push(id);
            const currentIndex = playerSequence.length - 1;

            if (playerSequence[currentIndex] !== sequence[currentIndex]) {
                // Wrong Input
                gameOver();
                return;
            }

            // Correct Input
            activateNode(id, 'active-blue', 150);

            if (playerSequence.length === sequence.length) {
                // Round Completed
                isPlayerTurn = false;
                score += level * 100;
                if (score > highScore) {
                    highScore = score;
                    localStorage.setItem('shadow_protocol_hs', highScore);
                }
                level++;
                updateStats();
                statusEl.textContent = 'PATTERN MATCHED. SYNCING...';
                
                setTimeout(() => {
                    startNextRound();
                }, 1000);
            }
        }

        function gameOver() {
            isPlaying = false;
            isPlayerTurn = false;
            playTone(100, 0.5, 'sawtooth');

            nodes.forEach(node => node.classList.add('error'));
            statusEl.textContent = 'SYSTEM FAILURE // CONNECTION LOST';
            
            setTimeout(() => {
                nodes.forEach(node => node.classList.remove('error'));
                startBtn.disabled = false;
                startBtn.textContent = 'RETRY PROTOCOL';
            }, 1000);
        }
    </script>
</body>
</html>
