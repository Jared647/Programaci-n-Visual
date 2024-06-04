# Programaci-n-Visual
Trabajos de matería 
Snake4 

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Juego de Snake</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            background-color: #111;
            color: white;
            font-family: Arial, sans-serif;
        }
        #menu {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-bottom: 20px;
        }
        canvas {
            background-color: #333;
        }
        button {
            margin: 5px;
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div id="menu">
        <h1>Juego de Snake</h1>
        <label for="difficulty">Selecciona la dificultad:</label>
        <select id="difficulty">
            <option value="100">Fácil</option>
            <option value="80">Media</option>
            <option value="60">Difícil</option>
        </select>
        <label for="level">Selecciona el nivel:</label>
        <select id="level">
            <option value="1">Nivel 1</option>
            <option value="2">Nivel 2</option>
            <option value="3">Nivel 3</option>
            <option value="4">Nivel 4</option>
            <option value="5">Nivel 5</option>
            <option value="6">Nivel 6</option>
            <option value="7">Nivel 7</option>
            <option value="8">Nivel 8</option>
            <option value="9">Nivel 9</option>
            <option value="10">Nivel 10</option>
        </select>
        <button onclick="startGame()">Iniciar Juego</button>
    </div>
    <canvas id="gameCanvas" width="400" height="400" style="display: none;"></canvas>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const box = 20;
        const canvasSize = 400;

        let snake = [];
        let food;
        let hearts = [];
        let mines = [];
        let direction = null;
        let score = 0;
        let level = 1;
        let lives = 3;
        let game;
        let delay = 100;
        let showTongue = false;

        document.addEventListener('keydown', setDirection);

        function startGame() {
            const menu = document.getElementById('menu');
            menu.style.display = 'none';
            canvas.style.display = 'block';

            const difficulty = document.getElementById('difficulty');
            delay = parseInt(difficulty.value);

            const levelSelect = document.getElementById('level');
            level = parseInt(levelSelect.value);

            resetGame();
            game = setInterval(draw, delay);
        }

        function resetGame() {
            snake = [{ x: 9 * box, y: 10 * box }];
            direction = null;
            score = 0;
            lives = 3;
            hearts = [];
            mines = [];
            placeFood();
            placeHearts(level);
            placeMines(level);
        }

        function placeFood() {
            food = {
                x: Math.floor(Math.random() * 20) * box,
                y: Math.floor(Math.random() * 20) * box
            };
            if (isOccupied(food.x, food.y)) placeFood();
        }

        function placeHearts(number) {
            for (let i = 0; i < number; i++) {
                let heart = {
                    x: Math.floor(Math.random() * 20) * box,
                    y: Math.floor(Math.random() * 20) * box
                };
                if (!isOccupied(heart.x, heart.y)) {
                    hearts.push(heart);
                } else {
                    i--;
                }
            }
        }

        function placeMines(number) {
            for (let i = 0; i < number; i++) {
                let mine = {
                    x: Math.floor(Math.random() * 20) * box,
                    y: Math.floor(Math.random() * 20) * box
                };
                if (!isOccupied(mine.x, mine.y)) {
                    mines.push(mine);
                } else {
                    i--;
                }
            }
        }

        function setDirection(event) {
            let key = event.keyCode;
            if (key === 37 && direction !== 'RIGHT') direction = 'LEFT';
            else if (key === 38 && direction !== 'DOWN') direction = 'UP';
            else if (key === 39 && direction !== 'LEFT') direction = 'RIGHT';
            else if (key === 40 && direction !== 'UP') direction = 'DOWN';
            else if (key === 32) showTongue = true;
        }

        function draw() {
            ctx.clearRect(0, 0, canvasSize, canvasSize);

            for (let i = 0; i < snake.length; i++) {
                if (i === 0) {
                    drawSnakeHead(snake[i].x, snake[i].y);
                } else {
                    ctx.fillStyle = 'white';
                    ctx.fillRect(snake[i].x, snake[i].y, box, box);
                    ctx.strokeStyle = 'red';
                    ctx.strokeRect(snake[i].x, snake[i].y, box, box);
                }
            }

            ctx.fillStyle = 'red';
            ctx.fillRect(food.x, food.y, box, box);

            for (let heart of hearts) {
                ctx.fillStyle = 'pink';
                ctx.fillText('♥', heart.x + 4, heart.y + 16);
            }

            for (let mine of mines) {
                ctx.fillStyle = 'yellow';
                ctx.fillText('*', mine.x + 4, mine.y + 16);
            }

            let snakeX = snake[0].x;
            let snakeY = snake[0].y;

            if (direction === 'LEFT') snakeX -= box;
            if (direction === 'UP') snakeY -= box;
            if (direction === 'RIGHT') snakeX += box;
            if (direction === 'DOWN') snakeY += box;

            if (snakeX === food.x && snakeY === food.y) {
                score++;
                placeFood();
            } else {
                snake.pop();
            }

            let newHead = { x: snakeX, y: snakeY };

            if (collision(newHead, hearts)) {
                lives++;
                hearts = hearts.filter(heart => heart.x !== newHead.x || heart.y !== newHead.y);
            }

            if (collision(newHead, mines)) {
                lives--;
                mines = mines.filter(mine => mine.x !== newHead.x || mine.y !== newHead.y);
                if (lives === 0) {
                    clearInterval(game);
                    alert('Game Over');
                    showMenu();
                }
            }

            if (snakeX < 0 || snakeY < 0 || snakeX >= canvasSize || snakeY >= canvasSize || collision(newHead, snake)) {
                clearInterval(game);
                alert('Game Over');
                showMenu();
            }

            snake.unshift(newHead);

            ctx.fillStyle = 'white';
            ctx.font = '20px Arial';
            ctx.fillText('Score: ' + score, 10, canvasSize - 10);
            ctx.fillText('Level: ' + level, 320, canvasSize - 10);
            ctx.fillText('Lives: ' + lives, 160, canvasSize - 10);

            showTongue = false; // Reset tongue state after each draw
        }

        function drawSnakeHead(x, y) {
            ctx.fillStyle = 'green';
            ctx.fillRect(x, y, box, box);
            ctx.fillStyle = 'black';
            ctx.beginPath();
            ctx.arc(x + 10, y + 10, 4, 0, Math.PI * 2, true); // Draw eyes
            ctx.fill();
            ctx.beginPath();
            ctx.arc(x + 10, y + 14, 2, 0, Math.PI, false); // Draw mouth
            ctx.stroke();

            if (showTongue) {
                ctx.fillStyle = 'red';
                ctx.fillRect(x + 8, y + 20, 4, 10); // Draw tongue
            }
        }

        function collision(head, array) {
            for (let item of array) {
                if (head.x === item.x && head.y === item.y) return true;
            }
            return false;
        }

        function isOccupied(x, y) {
            for (let part of snake) {
                if (part.x === x && part.y === y) return true;
            }
            for (let heart of hearts) {
                if (heart.x === x && heart.y === y) return true;
            }
            for (let mine of mines) {
                if (mine.x === x && mine.y === y) return true;
            }
            return false;
        }

        function showMenu() {
            const menu = document.getElementById('menu');
            menu.style.display = 'flex';
            canvas.style.display = 'none';
        }
    </script>
</body>
</html>
