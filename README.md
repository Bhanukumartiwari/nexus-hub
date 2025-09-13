# nexus-hub
Nexus Hub Productivity Dashboard
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nexus Hub - Productivity Dashboard</title>
    <style>
        body {
            background: #0a0a0a;
            color: #0ff;
            font-family: 'Arial', sans-serif;
            margin: 0;
            overflow-x: hidden;
        }
        #canvas { position: fixed; top: 0; left: 0; z-index: -1; opacity: 0.3; }
        .container {
            max-width: 1200px;
            margin: 20px auto;
            padding: 20px;
            background: rgba(0, 0, 0, 0.8);
            border-radius: 10px;
            box-shadow: 0 0 20px #0ff;
        }
        h1, h2 { text-align: center; text-shadow: 0 0 10px #0ff; }
        .role-selector, .pomodoro, .todo, .weather { margin: 20px; padding: 15px; border: 1px solid #0ff; border-radius: 5px; }
        .role-selector select, .todo input, .todo button { padding: 10px; margin: 5px; border: none; border-radius: 5px; background: #333; color: #0ff; }
        .todo-item { display: flex; justify-content: space-between; padding: 10px; margin: 5px 0; background: #222; border-radius: 5px; }
        .todo-item:hover { box-shadow: 0 0 10px #0ff; cursor: move; }
        .pomodoro-timer { font-size: 2em; text-align: center; text-shadow: 0 0 10px #0ff; }
        .weather { text-align: center; }
        button { cursor: pointer; transition: all 0.3s; }
        button:hover { background: #0ff; color: #000; }
        .ad-section { position: fixed; right: 10px; top: 100px; width: 160px; }
        @media (max-width: 600px) { .container { margin: 10px; padding: 10px; } .ad-section { display: none; } }
    </style>
</head>
<body>
    <canvas id="canvas"></canvas>
    <div class="container">
        <h1>Nexus Hub</h1>
        <div class="role-selector">
            <h2>Select Your Role</h2>
            <select id="role" onchange="updateDashboard()">
                <option value="student">Student</option>
                <option value="professional">Professional</option>
            </select>
        </div>
        <div class="pomodoro">
            <h2>Pomodoro Timer</h2>
            <div class="pomodoro-timer" id="timer">25:00</div>
            <button onclick="startTimer()">Start</button>
            <button onclick="resetTimer()">Reset</button>
        </div>
        <div class="todo">
            <h2>Tasks</h2>
            <input type="text" id="taskInput" placeholder="Add a task...">
            <button onclick="addTask()">Add</button>
            <div id="todoList"></div>
        </div>
        <div class="weather">
            <h2>Weather</h2>
            <div id="weatherInfo">Loading...</div>
        </div>
    </div>
    <div class="ad-section">
        <!-- Google AdSense Placeholder -->
        <!-- Replace with your AdSense code -->
        <p>Ad Space: <a href="https://www.coursera.org" target="_blank">Learn with Coursera (Affiliate)</a></p>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/particles.js/2.0.0/particles.min.js"></script>
    <script>
        // Particle.js Background
        particlesJS('canvas', {
            particles: { number: { value: 50 }, color: { value: '#0ff' }, shape: { type: 'circle' }, move: { speed: 2 } },
            interactivity: { events: { onhover: { enable: true, mode: 'repulse' } } }
        });

        // Adaptive Dashboard
        function updateDashboard() {
            const role = document.getElementById('role').value;
            const todoHeader = document.querySelector('.todo h2');
            todoHeader.textContent = role === 'student' ? 'Study Tasks' : 'Work Tasks';
            localStorage.setItem('role', role);
        }
        if (localStorage.getItem('role')) {
            document.getElementById('role').value = localStorage.getItem('role');
            updateDashboard();
        }

        // Pomodoro Timer
        let time = 25 * 60;
        let timerRunning = false;
        let timerInterval;
        function startTimer() {
            if (!timerRunning) {
                timerRunning = true;
                timerInterval = setInterval(() => {
                    time--;
                    const minutes = Math.floor(time / 60);
                    const seconds = time % 60;
                    document.getElementById('timer').textContent = `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
                    if (time <= 0) {
                        clearInterval(timerInterval);
                        timerRunning = false;
                        alert('Time’s up! Take a break.');
                        time = localStorage.getItem('role') === 'student' ? 25 * 60 : 50 * 60;
                    }
                }, 1000);
            }
        }
        function resetTimer() {
            clearInterval(timerInterval);
            timerRunning = false;
            time = localStorage.getItem('role') === 'student' ? 25 * 60 : 50 * 60;
            document.getElementById('timer').textContent = `${Math.floor(time / 60)}:00`;
        }

        // Todo List
        let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
        function renderTasks() {
            const todoList = document.getElementById('todoList');
            todoList.innerHTML = '';
            tasks.forEach((task, index) => {
                const div = document.createElement('div');
                div.className = 'todo-item';
                div.draggable = true;
                div.innerHTML = `${task} <button onclick="deleteTask(${index})">Delete</button>`;
                div.addEventListener('dragstart', (e) => e.dataTransfer.setData('text/plain', index));
                div.addEventListener('dragover', (e) => e.preventDefault());
                div.addEventListener('drop', (e) => {
                    e.preventDefault();
                    const fromIndex = e.dataTransfer.getData('text/plain');
                    const toIndex = index;
                    [tasks[fromIndex], tasks[toIndex]] = [tasks[toIndex], tasks[fromIndex]];
                    localStorage.setItem('tasks', JSON.stringify(tasks));
                    renderTasks();
                });
                todoList.appendChild(div);
            });
        }
        function addTask() {
            const taskInput = document.getElementById('taskInput');
            if (taskInput.value.trim()) {
                tasks.push(taskInput.value.trim());
                localStorage.setItem('tasks', JSON.stringify(tasks));
                taskInput.value = '';
                renderTasks();
            }
        }
        function deleteTask(index) {
            tasks.splice(index, 1);
            localStorage.setItem('tasks', JSON.stringify(tasks));
            renderTasks();
        }
        renderTasks();

        // Voice Control (Browser SpeechRecognition)
        if ('SpeechRecognition' in window || 'webkitSpeechRecognition' in window) {
            const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
            recognition.onresult = (event) => {
                const transcript = event.results[0][0].transcript.toLowerCase();
                if (transcript.includes('add task')) {
                    const task = transcript.replace('add task', '').trim();
                    if (task) {
                        tasks.push(task);
                        localStorage.setItem('tasks', JSON.stringify(tasks));
                        renderTasks();
                    }
                }
            };
            recognition.onerror = () => alert('Voice recognition not supported or failed.');
            document.addEventListener('keydown', (e) => {
                if (e.key === 'v') recognition.start();
            });
        }

        // Weather Widget with Provided API Key
        fetch('https://api.openweathermap.org/data/2.5/weather?q=London&units=metric&appid=8f870131b4e5932893eff8b4424b37cd')
            .then(response => response.json())
            .then(data => {
                document.getElementById('weatherInfo').innerHTML = `
                    ${data.name}: ${data.main.temp}°C, ${data.weather[0].description}
                `;
            })
            .catch(() => document.getElementById('weatherInfo').textContent = 'Weather unavailable');
    </script>
</body>
</html>

