<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Car Racing</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <!-- Chosen Palette: Asphalt & Neon -->
    <!-- Application Structure Plan: The application is a single-screen arcade game designed for quick, repetitive play. The user flow is straightforward: Start Screen -> Game -> Game Over Screen -> Restart. This structure is ideal for a de-stressing activity, as it minimizes cognitive load and allows for immediate action. All UI elements, including score displays and menus, are styled as transparent overlays on the central game canvas, maintaining a cohesive and immersive experience. The game logic is self-contained within the single HTML file, requiring no external assets or complex setup. The introduction of a "Nitro" power-up and sound effects adds a strategic layer and an enhanced sensory experience, specifically optimized for mobile touch controls and full-screen display. -->
    <!-- Visualization & Content Choices: The game serves as the primary interactive visualization. Report Info: N/A (Game Concept). Goal: Engage, Challenge, and Entertain. Viz/Presentation Method: A top-down 2D racing game rendered on an HTML Canvas. The road is a continuously moving background, and cars are simple geometric shapes. The player controls their car by moving it horizontally to avoid incoming obstacles (other cars) and to collect nitro boosts. Interaction: User presses arrow keys or taps on the sides of the screen to move the player's car left and right. The nitro boost is activated automatically on collection. Justification: This is a classic, highly effective arcade game mechanic that is easy to learn but difficult to master, perfect for a short mental break. The boost adds an element of reward and risk. Library/Method: Pure Vanilla JavaScript for all game logic and the Canvas API for all graphics rendering. Tone.js is used to create simple synthetic sound effects for an enhanced sensory experience without external files. Secondary Info: Score/High Score/Boost Bar. Goal: Inform/Motivate. Method: HTML text and a progress bar overlays. Interaction: Display only. Justification: Provides clear feedback on performance, resource management, and encourages replayability. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Press Start 2P', cursive;
            background-color: #121212;
            color: #e5e5e5;
            overflow: hidden;
        }
        canvas {
            background-color: #4a5568;
            display: block;
            border: 4px solid #e5e5e5;
            border-radius: 8px;
            box-shadow: 0 0 20px rgba(229, 229, 229, 0.3);
        }
        .ui-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            text-align: center;
            flex-direction: column;
            background-color: rgba(18, 18, 18, 0.8);
            backdrop-filter: blur(5px);
            z-index: 10;
        }
        .game-button {
            background-color: #6366f1;
            color: #fff;
            border: 2px solid #4f46e5;
            transition: all 0.2s ease-in-out;
            box-shadow: 0 4px 0 #4f46e5;
        }
        .game-button:active {
            transform: translateY(4px);
            box-shadow: 0 0px 0 #4f46e5;
        }
        .text-glow {
            text-shadow: 0 0 8px #6366f1, 0 0 12px #6366f1;
        }
        .control-area {
            position: absolute;
            width: 50%;
            height: 100%;
            top: 0;
            cursor: pointer;
            z-index: 5;
        }
        #nitro-bar-container {
            position: absolute;
            top: 4rem;
            left: 50%;
            transform: translateX(-50%);
            width: 80%;
            max-width: 200px;
            height: 20px;
            background-color: #333;
            border: 2px solid #e5e5e5;
            border-radius: 5px;
            overflow: hidden;
            z-index: 20;
        }
        #nitro-bar {
            height: 100%;
            width: 0%;
            background-color: #fca5a5;
            transition: width 0.1s linear;
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen m-0">

    <div class="relative w-screen h-screen flex justify-center items-center">
        <div id="startScreen" class="ui-overlay rounded-lg flex flex-col justify-center items-center p-4">
            <h1 class="text-4xl md:text-6xl font-bold text-glow mb-4">Car Racing</h1>
            <p class="text-lg md:text-xl mb-8 text-gray-300">Use left/right arrows or tap to steer.</p>
            <button id="startButton" class="game-button font-bold py-3 px-8 rounded-lg text-2xl">Start</button>
        </div>

        <div id="gameOverScreen" class="ui-overlay rounded-lg hidden flex-col justify-center items-center p-4">
            <h1 class="text-4xl md:text-6xl font-bold text-red-500 text-glow mb-4">CRASH!</h1>
            <p class="text-lg md:text-xl text-gray-300 mb-2">Final Score: <span id="finalScore">0</span></p>
            <p class="text-lg md:text-xl text-gray-300 mb-8">High Score: <span id="finalHighScore">0</span></p>
            <button id="restartButton" class="game-button font-bold py-3 px-8 rounded-lg text-2xl">Retry</button>
        </div>
        
        <div class="absolute top-4 left-4 text-xl md:text-2xl z-20">
            <span class="text-gray-400">SCORE:</span> <span id="scoreDisplay" class="text-white">0</span>
        </div>
        <div class="absolute top-4 right-4 text-xl md:text-2xl z-20">
            <span class="text-gray-400">HIGH:</span> <span id="highScoreDisplay" class="text-white">0</span>
        </div>

        <div id="nitro-bar-container">
            <div id="nitro-bar"></div>
        </div>

        <canvas id="gameCanvas"></canvas>
        <div class="absolute top-0 left-0 w-1/2 h-full" id="left-control"></div>
        <div class="absolute top-0 right-0 w-1/2 h-full" id="right-control"></div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const startButton = document.getElementById('startButton');
        const restartButton = document.getElementById('restartButton');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const highScoreDisplay = document.getElementById('highScoreDisplay');
        const finalScore = document.getElementById('finalScore');
        const finalHighScore = document.getElementById('finalHighScore');
        const leftControl = document.getElementById('left-control');
        const rightControl = document.getElementById('right-control');
        const nitroBar = document.getElementById('nitro-bar');

        let scaleFactor;
        let gameActive = false;
        let score = 0;
        let highScore = localStorage.getItem('carRacingHighScore') || 0;

        let player, obstacles, gameSpeed, roadOffset, obstacleSpawnRate, nitro, nitroActive, nitroDuration;

        const engineSynth = new Tone.Synth({
            oscillator: { type: "sawtooth" },
            envelope: { attack: 0.1, decay: 0.2, sustain: 0.8, release: 0.2 }
        }).toDestination();
        const boostSynth = new Tone.Synth({
            oscillator: { type: "square" },
            envelope: { attack: 0.01, decay: 0.2, sustain: 0.1, release: 0.5 }
        }).toDestination();
        const crashSynth = new Tone.NoiseSynth({
            noise: { type: "white" },
            envelope: { attack: 0.01, decay: 0.4, sustain: 0.1, release: 0.8 }
        }).toDestination();

        function resizeCanvas() {
            const aspectRatio = 16 / 9;
            let newWidth = window.innerWidth;
            let newHeight = newWidth / aspectRatio;
            
            if (newHeight > window.innerHeight) {
                newHeight = window.innerHeight;
                newWidth = newHeight * aspectRatio;
            }
            
            canvas.width = newWidth;
            canvas.height = newHeight;
            scaleFactor = canvas.height / 540;
            initializeGameVariables();
        }

        function initializeGameVariables() {
            gameSpeed = 5 * scaleFactor;
            roadOffset = 0;
            obstacleSpawnRate = 2000;
            nitroActive = false;
            nitroDuration = 0;
            player = {
                x: canvas.width / 2,
                y: canvas.height * 0.8,
                width: 30 * scaleFactor,
                height: 50 * scaleFactor,
                speed: 5 * scaleFactor,
                draw() {
                    const carWidth = this.width;
                    const carHeight = this.height;
                    const carX = this.x - carWidth / 2;
                    const carY = this.y - carHeight / 2;
                    ctx.fillStyle = '#6366f1';
                    ctx.fillRect(carX, carY + carHeight * 0.1, carWidth, carHeight * 0.8);
                    ctx.fillRect(carX + carWidth * 0.2, carY, carWidth * 0.6, carHeight * 0.3);
                    ctx.fillStyle = '#e5e5e5';
                    ctx.fillRect(carX, carY + carHeight * 0.8, carWidth * 0.2, carHeight * 0.2);
                    ctx.fillRect(carX + carWidth * 0.8, carY + carHeight * 0.8, carWidth * 0.2, carHeight * 0.2);
                    ctx.fillStyle = '#000000';
                    ctx.fillRect(carX + carWidth * 0.05, carY + carHeight * 0.85, carWidth * 0.1, carHeight * 0.1);
                    ctx.fillRect(carX + carWidth * 0.85, carY + carHeight * 0.85, carWidth * 0.1, carHeight * 0.1);
                },
                move(direction) {
                    if (direction === 'left') {
                        this.x = Math.max(canvas.width * 0.25, this.x - this.speed);
                    } else if (direction === 'right') {
                        this.x = Math.min(canvas.width * 0.75, this.x + this.speed);
                    }
                }
            };
            obstacles = [];
            nitro = null;
        }

        class Obstacle {
            constructor() {
                this.width = (20 + Math.random() * 20) * scaleFactor;
                this.height = (40 + Math.random() * 40) * scaleFactor;
                this.x = canvas.width * (0.25 + Math.random() * 0.5);
                this.y = -this.height;
                const colors = ['#f87171', '#fbbf24', '#34d399', '#60a5fa'];
                this.color = colors[Math.floor(Math.random() * colors.length)];
            }
            draw() {
                const carWidth = this.width;
                const carHeight = this.height;
                const carX = this.x - carWidth / 2;
                const carY = this.y;
                ctx.fillStyle = this.color;
                ctx.fillRect(carX, carY + carHeight * 0.1, carWidth, carHeight * 0.8);
                ctx.fillRect(carX + carWidth * 0.2, carY, carWidth * 0.6, carHeight * 0.3);
                ctx.fillStyle = '#e5e5e5';
                ctx.fillRect(carX, carY + carHeight * 0.8, carWidth * 0.2, carHeight * 0.2);
                ctx.fillRect(carX + carWidth * 0.8, carY + carHeight * 0.8, carWidth * 0.2, carHeight * 0.2);
                ctx.fillStyle = '#000000';
                ctx.fillRect(carX + carWidth * 0.05, carY + carHeight * 0.85, carWidth * 0.1, carHeight * 0.1);
                ctx.fillRect(carX + carWidth * 0.85, carY + carHeight * 0.85, carWidth * 0.1, carHeight * 0.1);
            }
            update() {
                this.y += gameSpeed;
            }
        }

        class Nitro {
            constructor() {
                this.width = 15 * scaleFactor;
                this.height = 30 * scaleFactor;
                this.x = canvas.width * (0.25 + Math.random() * 0.5);
                this.y = -this.height;
                this.color = '#f97316';
            }
            draw() {
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.moveTo(this.x, this.y);
                ctx.lineTo(this.x + this.width / 2, this.y + this.height);
                ctx.lineTo(this.x, this.y + this.height * 0.7);
                ctx.lineTo(this.x - this.width / 2, this.y + this.height);
                ctx.closePath();
                ctx.fill();
            }
            update() {
                this.y += gameSpeed;
            }
        }

        function spawnObstacle() {
            if (gameActive) {
                obstacles.push(new Obstacle());
                setTimeout(spawnObstacle, obstacleSpawnRate);
            }
        }

        function spawnNitro() {
            if (gameActive && nitro === null && Math.random() > 0.8) {
                nitro = new Nitro();
            }
        }

        function checkCollision(p, o) {
            return (
                p.x - p.width / 2 < o.x + o.width / 2 &&
                p.x + p.width / 2 > o.x - o.width / 2 &&
                p.y - p.height / 2 < o.y + o.height &&
                p.y + p.height / 2 > o.y
            );
        }

        function resetGame() {
            score = 0;
            obstacles = [];
            nitro = null;
            nitroActive = false;
            nitroBar.style.width = '0%';
            scoreDisplay.textContent = score;
            highScoreDisplay.textContent = highScore;
            resizeCanvas();
        }

        function startGame() {
            Tone.start();
            engineSynth.triggerAttack();
            resetGame();
            gameActive = true;
            startScreen.classList.add('hidden');
            gameOverScreen.classList.add('hidden');
            spawnObstacle();
            setInterval(spawnNitro, 5000);
            setInterval(() => {
                if(gameActive) {
                    score++;
                    scoreDisplay.textContent = score;
                }
            }, 100);
            animate();
        }

        function endGame() {
            gameActive = false;
            engineSynth.triggerRelease();
            crashSynth.triggerAttackRelease("8n");
            if (score > highScore) {
                highScore = score;
                localStorage.setItem('carRacingHighScore', highScore);
            }
            finalScore.textContent = score;
            finalHighScore.textContent = highScore;
            gameOverScreen.classList.remove('hidden');
        }

        function drawRoad() {
            const laneWidth = canvas.width * 0.5;
            const laneX = canvas.width / 2;
            const dashLength = 20 * scaleFactor;
            const gapLength = 20 * scaleFactor;
            const totalDash = dashLength + gapLength;
            ctx.fillStyle = '#333';
            ctx.fillRect(laneX - laneWidth / 2, 0, laneWidth, canvas.height);
            ctx.fillStyle = '#e5e5e5';
            roadOffset = (roadOffset + gameSpeed) % totalDash;
            for (let i = 0; i < canvas.height / totalDash; i++) {
                ctx.fillRect(laneX - 2 * scaleFactor, (i * totalDash) + roadOffset, 4 * scaleFactor, dashLength);
            }
            if (nitroActive) {
                ctx.fillStyle = 'rgba(99, 102, 241, 0.5)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            }
        }

        let keysPressed = {};
        window.addEventListener('keydown', (e) => {
            keysPressed[e.key] = true;
        });
        window.addEventListener('keyup', (e) => {
            keysPressed[e.key] = false;
        });

        function handleInput() {
            if (keysPressed['ArrowLeft']) {
                player.move('left');
            }
            if (keysPressed['ArrowRight']) {
                player.move('right');
            }
        }

        leftControl.addEventListener('touchstart', (e) => {
            e.preventDefault();
            keysPressed['ArrowLeft'] = true;
        });
        leftControl.addEventListener('touchend', (e) => {
            e.preventDefault();
            keysPressed['ArrowLeft'] = false;
        });
        rightControl.addEventListener('touchstart', (e) => {
            e.preventDefault();
            keysPressed['ArrowRight'] = true;
        });
        rightControl.addEventListener('touchend', (e) => {
            e.preventDefault();
            keysPressed['ArrowRight'] = false;
        });

        function animate() {
            if (!gameActive) return;
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawRoad();
            handleInput();
            player.draw();
            if (nitroActive) {
                gameSpeed = 10 * scaleFactor;
                nitroDuration -= 1;
                nitroBar.style.width = `${(nitroDuration / 200) * 100}%`;
                if (nitroDuration <= 0) {
                    nitroActive = false;
                }
            } else {
                gameSpeed = 5 * scaleFactor + (score * 0.05 * scaleFactor);
                nitroBar.style.width = '0%';
            }
            engineSynth.frequency.value = 100 + (gameSpeed * 10);
            obstacles.forEach((obstacle, index) => {
                obstacle.update();
                obstacle.draw();
                if (checkCollision(player, obstacle)) {
                    endGame();
                }
                if (obstacle.y > canvas.height) {
                    obstacles.splice(index, 1);
                }
            });
            if (nitro !== null) {
                nitro.update();
                nitro.draw();
                if (checkCollision(player, nitro)) {
                    nitroActive = true;
                    nitroDuration = 200;
                    nitro = null;
                    boostSynth.triggerAttackRelease("C4", "8n");
                }
                if (nitro !== null && nitro.y > canvas.height) {
                    nitro = null;
                }
            }
            requestAnimationFrame(animate);
        }
        window.addEventListener('resize', resetGame);
        startButton.addEventListener('click', startGame);
        restartButton.addEventListener('click', startGame);
        resetGame();
    </script>
</body>
</html>
