<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>T-Rex Runner Clone - Final Music Fix</title>

    <style>
        /* --- STYLE.CSS SECTION --- */
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            width: 100vw;
            margin: 0;
            background-color: #FFFFFF; 
            overflow: hidden;
            font-family: monospace;
            color: #1e3a8a; 
        }

        /* 1. GAME CONTAINER */
        .game-container {
            width: 900px; 
            height: 350px; /* Taller to fit jump */
            position: relative;
            overflow: hidden;
            
            background-color: rgba(255, 255, 255, 0.0); 
            
            /* ROAD EFFECT */
            border-bottom: 30px solid #4a4a4a; 
            box-shadow: 0 10px 0 0 #f0a71f; 
            margin-bottom: -25px; 
        }

        /* 2. DINO STYLES */
        .dino {
            width: 55px; 
            height: 85px; 
            position: absolute;
            bottom: 30px; 
            left: 50px; 
            background-image: url('https://i.postimg.cc/JhWv4hgg/Screenshot-2025-11-02-211959.png'); 
            background-size: 100% 100%; 
            background-position: center bottom;
            background-repeat: no-repeat;
            transition: all 0.25s; 
            filter: drop-shadow(0px 0px 0px rgba(0, 0, 0, 0)); 
        }

        /* JUMP HEIGHT */
        .jump {
            bottom: 230px; 
        }

        /* 3. CACTUS STYLES */
        .cactus {
            width: 45px; 
            height: 70px; 
            position: absolute;
            bottom: 30px; 
            right: -45px; 
            background-image: url('https://i.postimg.cc/qRqLkVkX/Screenshot-2025-11-02-212122.png'); 
            background-size: 100% 100%;
            background-position: center bottom;
            background-repeat: no-repeat;
            filter: drop-shadow(0px 0px 0px rgba(0, 0, 0, 0)); 
        }
        
        /* BIRD STYLES */
        .bird {
            width: 55px; 
            height: 55px; 
            position: absolute;
            right: -55px; 
            background-image: url('https://i.postimg.cc/3RwQpkwS/Screenshot-2025-11-02-201520.png');
            background-size: 100% 100%;
            background-position: center center;
            background-repeat: no-repeat;
            filter: drop-shadow(0px 0px 0px rgba(0, 0, 0, 0)); 
        }

        /* SCORE and GAME OVER */
        .score {
            position: absolute;
            top: 15px;
            right: 20px;
            font-size: 2em; 
            color: #1e3a8a; 
            z-index: 10; 
        }

        .game-over-screen {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            color: #1e3a8a; 
            background-color: rgba(255, 255, 255, 0.9);
            padding: 20px; 
            border-radius: 5px;
            border: 4px solid #1e3a8a; 
            z-index: 5; 
        }
        .game-over-screen h1 { margin: 0; font-size: 3em; }
        .game-over-screen p { margin: 10px 0 0; font-size: 1.5em; }

        /* 4. MUSIC ICON/BUTTON STYLES */
        #musicControl {
            position: absolute;
            bottom: 15px;
            right: 15px;
            font-size: 1.5em;
            cursor: pointer;
            color: #1e3a8a;
            border: 2px solid #1e3a8a;
            padding: 5px 10px;
            border-radius: 5px;
            z-index: 100;
        }
    </style>
</head>
<body>

    <audio id="bgMusic" loop>
        <source src="https://biological-coral-cv0u9l5snt.edgeone.app/udta%20hi%20phiru%20%20in%20habao%20me%20kahin%20%20%20%20#funny%20#modi%20ji%27s%20favourite%20song%20on%20his%20own%20voice.mp3" type="audio/mp3">
        Your browser does not support the audio element.
    </audio>

    <div class="game-container">
        <div class="dino" id="dino"></div>
        <div class="score" id="score">Score: 0</div>
        <div class="game-over-screen" id="gameOver">
            <h1>GAME OVER</h1>
            <p>Press SPACE to Restart</p>
        </div>
    </div>
    
    <div id="musicControl">🔇 Enable Music</div>

    <script>
        // --- JAVASCRIPT SECTION ---
        const dino = document.getElementById('dino');
        const gameContainer = document.querySelector('.game-container');
        const scoreDisplay = document.getElementById('score');
        const gameOverScreen = document.getElementById('gameOver');
        const music = document.getElementById('bgMusic'); 
        const musicControl = document.getElementById('musicControl');
        
        const CONTAINER_WIDTH = 900; 
        const ROAD_OFFSET_PIXELS = 30; 

        let isJumping = false;
        let isGameOver = true;
        let gameSpeed = 5;
        let score = 0;
        let lastObstacleTime = 0;
        const OBSTACLE_INTERVAL_MIN = 1000;
        const OBSTACLE_INTERVAL_MAX = 3000;
        
        const BIRD_HEIGHTS = [90]; 

        let allowBirdSpawn = false;
        const BIRD_SPAWN_DELAY_MS = 120000; // 2 minutes
        
        let isMusicPlaying = false; // New state tracker

        // --- Music Control Function ---
        function toggleMusic() {
            if (isMusicPlaying) {
                music.pause();
                isMusicPlaying = false;
                musicControl.innerHTML = '🔇 Enable Music';
            } else {
                music.play().then(() => {
                    isMusicPlaying = true;
                    musicControl.innerHTML = '🔊 Music On';
                }).catch(error => {
                    console.log("Playback failed even with click, audio source likely invalid or inaccessible.");
                    musicControl.innerHTML = '❌ Audio Error';
                });
            }
        }
        
        // --- Core Game Functions ---

        function jump() {
            if (isJumping || isGameOver) return;
            isJumping = true;
            dino.classList.add('jump'); 

            setTimeout(() => {
                dino.classList.remove('jump');
                setTimeout(() => {
                    isJumping = false; 
                }, 50); 
            }, 250); 
        }

        function createObstacle() {
            const now = Date.now(); 
            if (now - lastObstacleTime < Math.random() * (OBSTACLE_INTERVAL_MAX - OBSTACLE_INTERVAL_MIN) + OBSTACLE_INTERVAL_MIN) {
                return;
            }
            lastObstacleTime = now;

            let isBird = false;
            if (allowBirdSpawn) { 
                isBird = Math.random() > 0.7; 
            }
            
            const obstacle = document.createElement('div');
            obstacle.classList.add(isBird ? 'bird' : 'cactus');
            gameContainer.appendChild(obstacle);

            if (isBird) {
                const randomIndex = Math.floor(Math.random() * BIRD_HEIGHTS.length);
                const birdHeight = BIRD_HEIGHTS[randomIndex];
                
                obstacle.style.bottom = `${birdHeight + ROAD_OFFSET_PIXELS}px`;
            } else {
                obstacle.style.bottom = `${ROAD_OFFSET_PIXELS}px`; 
            }
            
            let obstacleX = CONTAINER_WIDTH; 

            function moveObstacle() {
                if (isGameOver) {
                    obstacle.remove();
                    return;
                }

                obstacleX -= gameSpeed;
                obstacle.style.right = `${CONTAINER_WIDTH - obstacleX}px`;

                if (checkCollision(dino, obstacle)) {
                    endGame();
                    return;
                }
                
                if (obstacleX < -obstacle.clientWidth) {
                    obstacle.remove();
                    score++;
                    scoreDisplay.textContent = `Score: ${score}`;
                } else {
                    requestAnimationFrame(moveObstacle);
                }
            }

            requestAnimationFrame(moveObstacle);
        }

        function checkCollision(dino, obstacle) {
            const dinoRect = dino.getBoundingClientRect();
            const obsRect = obstacle.getBoundingClientRect();
            
            const JUMP_HEIGHT_PIXELS_TOTAL = 230; 

            const dinoBottom = dino.classList.contains('jump') ? dinoRect.bottom - JUMP_HEIGHT_PIXELS_TOTAL : dinoRect.bottom;

            return (
                dinoRect.left < obsRect.right &&
                dinoRect.right > obsRect.left &&
                dinoRect.top < obsRect.bottom &&
                dinoBottom > obsRect.top
            );
        }

        function gameLoop() {
            if (isGameOver) return;

            createObstacle();
            gameSpeed += 0.005;

            requestAnimationFrame(gameLoop);
        }

        function startGame() {
            isGameOver = false;
            gameOverScreen.style.display = 'none';
            score = 0;
            gameSpeed = 5;
            scoreDisplay.textContent = `Score: 0`;
            
            document.querySelectorAll('.cactus, .bird').forEach(obs => obs.remove());
            
            dino.classList.remove('jump');
            isJumping = false;
            
            allowBirdSpawn = false; 
            setTimeout(() => {
                allowBirdSpawn = true;
            }, BIRD_SPAWN_DELAY_MS);
            
            // Do NOT try to play music here. It must be manually clicked.
            if (isMusicPlaying) {
                 music.play().catch(e => console.log("Failed to auto-resume music."));
            }

            gameLoop();
        }

        function endGame() {
            isGameOver = true;
            gameOverScreen.style.display = 'block';
            music.pause(); 
            music.currentTime = 0; 
        }

        // --- Event Listeners ---
        document.addEventListener('keydown', (event) => {
            if (event.code === 'Space' || event.code === 'ArrowUp') {
                if (isGameOver) {
                    startGame();
                } else {
                    jump();
                }
            }
        });
        
        // NEW: Event listener for the manual music button
        musicControl.addEventListener('click', toggleMusic);


        endGame();
    </script>

</body>
</html>
