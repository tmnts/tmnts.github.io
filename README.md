# tmnts.github.io

<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Интерактивные Шары для Любимого Дизайнера</title>
    <style>
        /* Стилизуем страницу как дорогой минималистичный лендинг */
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #0b0f19; /* Глубокий темный фон */
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
        }

        /* Холст на весь экран на заднем плане */
        canvas {
            display: block;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }

        /* Контент поверх шаров */
        .content {
            position: relative;
            z-index: 2;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 100%;
            color: #ffffff;
            text-align: center;
            pointer-events: none; /* Чтобы текст не мешал мышке взаимодействовать с шарами */
            user-select: none;
        }

        h1 {
            font-size: 3.5rem;
            margin-bottom: 10px;
            letter-spacing: -1px;
            background: linear-gradient(45deg, #ffc0cb, #ffb6c1, #00bfff);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        p {
            font-size: 1.2rem;
            color: #8f9cae;
            max-width: 600px;
            line-height: 1.6;
        }
    </style>
</head>
<body>

    <canvas id="ballCanvas"></canvas>

    <div class="content">
        <h1>Ты создаешь красоту, а не просто "продукт"</h1>
        <p>чис пиа.. Да, этот текст набрал Чиспа! Шарики слушаются только тебя. Подвигай мышкой!</p>
    </div>

    <script>
        const canvas = document.getElementById('ballCanvas');
        const ctx = canvas.getContext('2d');

        // Подгоняем размер холста под экран
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);

        // Объект для хранения координат мыши
        const mouse = {
            x: null,
            y: null,
            radius: 150 // Радиус действия мыши (на каком расстоянии шары начнут убегать)
        };

        // Отслеживаем движение мыши
        window.addEventListener('mousemove', function(event) {
            mouse.x = event.clientX;
            mouse.y = event.clientY;
        });

        // Очищаем координаты, когда мышь улетает с экрана
        window.addEventListener('mouseout', function() {
            mouse.x = null;
            mouse.y = null;
        });

        // Массив цветов для шаров (можешь поменять на ее любимую палитру!)
        const colors = [
            'rgba(34,180,34, 0.7)',  // Зеленый
            'rgba(0, 242, 254, 0.7)',  // Бирюзовый
            'rgba(255, 255, 255, 0.7)',  // Белый
            'rgba(251,160,227, 0.6)'   // Розовый
        ];

        // Класс (чертеж) для создания каждого шара
        class Ball {
            constructor() {
                this.radius = Math.random() * 30 + 15; // Случайный радиус от 15 до 45px
                this.x = Math.random() * (canvas.width - this.radius * 2) + this.radius;
                this.y = Math.random() * (canvas.height - this.radius * 2) + this.radius;
                // Скорость движения (вектор)
                this.vx = (Math.random() - 0.5) * 2; // от -1 до 1
                this.vy = (Math.random() - 0.5) * 2; 
                this.color = colors[Math.floor(Math.random() * colors.length)];
            }

            // Метод отрисовки шара
            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2, false);
                ctx.fillStyle = this.color;
                
                // Добавляем красивое легкое размытие/свечение
                ctx.shadowBlur = 15;
                ctx.shadowColor = this.color;
                
                ctx.fill();
                ctx.closePath();
            }

            // Метод обновления позиции и просчета физики
            update() {
                // Отскок от левой и правой стены
                if (this.x + this.radius > canvas.width || this.x - this.radius < 0) {
                    this.vx = -this.vx;
                }
                // Отскок от пола и потолка
                if (this.y + this.radius > canvas.height || this.y - this.radius < 0) {
                    this.vy = -this.vy;
                }

                // Двигаем шар
                this.x += this.vx;
                this.y += this.vy;

                // ВЗАИМОДЕЙСТВИЕ С МЫШКОЙ (Магия отталкивания)
                if (mouse.x !== null && mouse.y !== null) {
                    // Считаем расстояние между центром шара и курсором по теореме Пифагора
                    let dx = this.x - mouse.x;
                    let dy = this.y - mouse.y;
                    let distance = Math.sqrt(dx * dx + dy * dy);

                    // Если мышь подошла слишком близко
                    if (distance < mouse.radius + this.radius) {
                        // Вычисляем силу отталкивания (чем ближе мышь, тем сильнее толчок)
                        let force = (mouse.radius + this.radius - distance) / mouse.radius;
                        
                        // Нормализуем вектор направления движения от мыши
                        let forceX = dx / distance;
                        let forceY = dy / distance;

                        // Мягко ускоряем шар в противоположную от мыши сторону
                        this.x += forceX * force * 5;
                        this.y += forceY * force * 5;
                    }
                }

                this.draw();
            }
        }

        // Создаем массив и заполняем его шарами
        const ballsArray = [];
        const numberOfBalls = 40; // Сколько шаров выпустить на экран

        for (let i = 0; i < numberOfBalls; i++) {
            ballsArray.push(new Ball());
        }

        // Бесконечный цикл анимации
        function animate() {
            // Очищаем экран перед каждым новым кадром, сбрасывая тени
            ctx.shadowBlur = 0;
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Обновляем и рисуем каждый шар
            ballsArray.forEach(ball => ball.update());
            
            // Запрашиваем следующий кадр анимации у браузера (примерно 60-120 FPS)
            requestAnimationFrame(animate);
        }

        animate();
    </script>
</body>
</html>
