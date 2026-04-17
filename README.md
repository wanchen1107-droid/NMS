# NMS
小恐龍遊戲
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>變速小恐龍 - 自定義圖片版</title>
    <style>
        body {
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f7f7f7;
            font-family: 'Arial', sans-serif;
            overflow: hidden;
        }
        #game-container {
            position: relative;
            text-align: center;
        }
        canvas {
            background-color: #fff;
            border-bottom: 2px solid #535353;
            box-shadow: 0 10px 20px rgba(0,0,0,0.1);
        }
        #ui-layer {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            display: none;
            background: rgba(255, 255, 255, 0.9);
            padding: 20px 40px;
            border-radius: 10px;
            border: 2px solid #535353;
        }
        .instruction {
            margin-top: 10px;
            color: #777;
            font-size: 14px;
        }
    </style>
</head>
<body>

    <div id="game-container">
        <canvas id="gameCanvas"></canvas>
        <div id="ui-layer">
            <h1 id="status-text">GAME OVER</h1>
            <p>按「空白鍵」重新開始</p>
        </div>
        <div class="instruction">按 空白鍵 跳躍 / 開始</div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const uiLayer = document.getElementById('ui-layer');

        canvas.width = 800;
        canvas.height = 250;

        // --- 1. 圖片處理 ---
        const dinoImg = new Image();
        // 這裡替換成你提供的灰色小恐龍圖片網址
        dinoImg.src = 'https://i.imgur.com/7b7vOZA.png'; 

        let imgLoaded = false;
        dinoImg.onload = () => { imgLoaded = true; };

        // --- 2. 遊戲變數 ---
        let dino, obstacles, score, gameActive, frame, baseSpeed;
        const gravity = 0.6;

        function init() {
            dino = {
                x: 50,
                y: 200,
                width: 60,  // 根據圖片比例稍微調整
                height: 60,
                dy: 0,
                jumpForce: 12,
                grounded: false
            };
            obstacles = [];
            score = 0;
            frame = 0;
            baseSpeed = 6;
            gameActive = true;
            uiLayer.style.display = 'none';
            animate();
        }

        function spawnObstacle() {
            let h = 30 + Math.random() * 30;
            obstacles.push({
                x: canvas.width,
                y: canvas.height - h,
                width: 20,
                height: h
            });
        }

        function animate() {
            if (!gameActive) {
                uiLayer.style.display = 'block';
                return;
            }

            ctx.clearRect(0, 0, canvas.width, canvas.height);
            frame++;

            // --- 3. 忽快忽慢邏輯 ---
            // 使用正弦波讓速度在 baseSpeed ± 3 之間變動
            let currentSpeed = baseSpeed + Math.sin(frame * 0.03) * 3;

            // 恐龍物理運動
            if (!dino.grounded) {
                dino.dy += gravity;
                dino.y += dino.dy;
            }

            if (dino.y > canvas.height - dino.height) {
                dino.y = canvas.height - dino.height;
                dino.dy = 0;
                dino.grounded = true;
            }

            // 繪製主角 (圖片)
            if (imgLoaded) {
                ctx.drawImage(dinoImg, dino.x, dino.y, dino.width, dino.height);
            } else {
                ctx.fillStyle = "#ccc"; // 圖片未載入前的備用色塊
                ctx.fillRect(dino.x, dino.y, dino.width, dino.height);
            }

            // 障礙物產生與更新
            if (frame % 90 === 0) spawnObstacle();

            for (let i = obstacles.length - 1; i >= 0; i--) {
                let o = obstacles[i];
                o.x -= currentSpeed;

                ctx.fillStyle = "#535353";
                ctx.fillRect(o.x, o.y, o.width, o.height);

                // 碰撞偵測 (縮小一點點判定範圍，體感更公平)
                if (dino.x + 10 < o.x + o.width &&
                    dino.x + dino.width - 10 > o.x &&
                    dino.y + 10 < o.y + o.height) {
                    gameActive = false;
                }

                if (o.x + o.width < 0) {
                    obstacles.splice(i, 1);
                    score++;
                }
            }

            // UI 資訊
            ctx.fillStyle = "#535353";
            ctx.font = "16px Arial";
            ctx.fillText(`SCORE: ${score}`, 20, 30);
            ctx.fillText(`SPEED: ${currentSpeed.toFixed(1)}`, 700, 30);

            requestAnimationFrame(animate);
        }

        // 鍵盤監聽
        window.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                if (gameActive) {
                    if (dino.grounded) {
                        dino.dy = -dino.jumpForce;
                        dino.grounded = false;
                    }
                } else {
                    init(); // 快速重新開始
                }
            }
        });

        // 啟動提示
        ctx.fillStyle = "#535353";
        ctx.font = "20px Arial";
        ctx.textAlign = "center";
        ctx.fillText("按 空白鍵 開始遊戲", canvas.width/2, canvas.height/2);

    </script>
</body>
</html>
