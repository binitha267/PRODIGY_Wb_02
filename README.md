<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Stopwatch Web Application</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <style>
        * {
            font-family: 'Inter', sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            overflow-x: hidden;
        }
        
        .stopwatch-container {
            backdrop-filter: blur(20px);
            background: rgba(255, 255, 255, 0.1);
            border: 1px solid rgba(255, 255, 255, 0.2);
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
        }
        
        .time-display {
            font-size: 4rem;
            font-weight: 300;
            letter-spacing: 0.1em;
            color: white;
            text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
            transition: all 0.3s ease;
        }
        
        .control-button {
            transition: all 0.3s ease;
            transform: translateY(0);
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
        }
        
        .control-button:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
        }
        
        .control-button:active {
            transform: translateY(1px);
        }
        
        .start-btn {
            background: linear-gradient(45deg, #4CAF50, #45a049);
        }
        
        .pause-btn {
            background: linear-gradient(45deg, #ff9800, #e68900);
        }
        
        .reset-btn {
            background: linear-gradient(45deg, #f44336, #d32f2f);
        }
        
        .lap-btn {
            background: linear-gradient(45deg, #2196F3, #1976D2);
        }
        
        .lap-list {
            max-height: 300px;
            overflow-y: auto;
            backdrop-filter: blur(10px);
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        
        .lap-item {
            transition: all 0.3s ease;
            border-left: 3px solid transparent;
        }
        
        .lap-item:hover {
            background: rgba(255, 255, 255, 0.1);
            border-left-color: #4CAF50;
        }
        
        .running .time-display {
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.8; }
        }
        
        .fade-in {
            animation: fadeIn 0.5s ease-in-out;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        @media (max-width: 640px) {
            .time-display {
                font-size: 2.5rem;
            }
        }
        
        /* Custom scrollbar */
        .lap-list::-webkit-scrollbar {
            width: 6px;
        }
        
        .lap-list::-webkit-scrollbar-track {
            background: rgba(255, 255, 255, 0.1);
            border-radius: 3px;
        }
        
        .lap-list::-webkit-scrollbar-thumb {
            background: rgba(255, 255, 255, 0.3);
            border-radius: 3px;
        }
        
        .lap-list::-webkit-scrollbar-thumb:hover {
            background: rgba(255, 255, 255, 0.5);
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">
    <div class="stopwatch-container rounded-3xl p-8 w-full max-w-md mx-auto">
        <!-- Header -->
        <div class="text-center mb-8">
            <h1 class="text-3xl font-bold text-white mb-2">
                <i class="fas fa-stopwatch mr-2"></i>
                Stopwatch
            </h1>
            <p class="text-gray-200 opacity-80">Precise time measurement</p>
        </div>
        
        <!-- Time Display -->
        <div class="text-center mb-8" id="stopwatchDisplay">
            <div class="time-display" id="timeDisplay">00:00.000</div>
        </div>
        
        <!-- Control Buttons -->
        <div class="grid grid-cols-2 gap-4 mb-6">
            <button id="startStopBtn" class="control-button start-btn text-white font-semibold py-4 px-6 rounded-xl">
                <i class="fas fa-play mr-2"></i>
                Start
            </button>
            <button id="resetBtn" class="control-button reset-btn text-white font-semibold py-4 px-6 rounded-xl">
                <i class="fas fa-redo mr-2"></i>
                Reset
            </button>
            <button id="pauseBtn" class="control-button pause-btn text-white font-semibold py-4 px-6 rounded-xl opacity-50 cursor-not-allowed" disabled>
                <i class="fas fa-pause mr-2"></i>
                Pause
            </button>
            <button id="lapBtn" class="control-button lap-btn text-white font-semibold py-4 px-6 rounded-xl opacity-50 cursor-not-allowed" disabled>
                <i class="fas fa-flag mr-2"></i>
                Lap
            </button>
        </div>
        
        <!-- Lap Times -->
        <div id="lapSection" class="hidden">
            <h3 class="text-white font-semibold mb-3 flex items-center">
                <i class="fas fa-list mr-2"></i>
                Lap Times
            </h3>
            <div class="lap-list rounded-xl p-4" id="lapList">
                <!-- Lap times will be inserted here -->
            </div>
        </div>
    </div>

    <script>
        class Stopwatch {
            constructor() {
                this.startTime = 0;
                this.elapsedTime = 0;
                this.timerInterval = null;
                this.isRunning = false;
                this.isPaused = false;
                this.lapCount = 0;
                this.lapTimes = [];
                
                this.initElements();
                this.bindEvents();
                this.updateDisplay();
            }
            
            initElements() {
                this.timeDisplay = document.getElementById('timeDisplay');
                this.startStopBtn = document.getElementById('startStopBtn');
                this.resetBtn = document.getElementById('resetBtn');
                this.pauseBtn = document.getElementById('pauseBtn');
                this.lapBtn = document.getElementById('lapBtn');
                this.lapSection = document.getElementById('lapSection');
                this.lapList = document.getElementById('lapList');
                this.stopwatchDisplay = document.getElementById('stopwatchDisplay');
            }
            
            bindEvents() {
                this.startStopBtn.addEventListener('click', () => this.toggleStartStop());
                this.resetBtn.addEventListener('click', () => this.reset());
                this.pauseBtn.addEventListener('click', () => this.togglePause());
                this.lapBtn.addEventListener('click', () => this.recordLap());
            }
            
            toggleStartStop() {
                if (!this.isRunning) {
                    this.start();
                } else {
                    this.stop();
                }
            }
            
            start() {
                this.startTime = performance.now() - this.elapsedTime;
                this.isRunning = true;
                this.isPaused = false;
                
                this.timerInterval = setInterval(() => {
                    if (!this.isPaused) {
                        this.elapsedTime = performance.now() - this.startTime;
                        this.updateDisplay();
                    }
                }, 10);
                
                this.updateButtons();
                this.stopwatchDisplay.classList.add('running');
            }
            
            stop() {
                clearInterval(this.timerInterval);
                this.isRunning = false;
                this.isPaused = false;
                
                this.updateButtons();
                this.stopwatchDisplay.classList.remove('running');
            }
            
            togglePause() {
                this.isPaused = !this.isPaused;
                this.updateButtons();
            }
            
            reset() {
                clearInterval(this.timerInterval);
                this.startTime = 0;
                this.elapsedTime = 0;
                this.isRunning = false;
                this.isPaused = false;
                this.lapCount = 0;
                this.lapTimes = [];
                
                this.updateDisplay();
                this.updateButtons();
                this.clearLaps();
                this.stopwatchDisplay.classList.remove('running');
            }
            
            recordLap() {
                if (this.isRunning && !this.isPaused) {
                    this.lapCount++;
                    const lapTime = this.elapsedTime;
                    const previousLapTime = this.lapTimes.length > 0 ? this.lapTimes[this.lapTimes.length - 1].time : 0;
                    const splitTime = lapTime - previousLapTime;
                    
                    this.lapTimes.push({
                        number: this.lapCount,
                        time: lapTime,
                        split: splitTime
                    });
                    
                    this.addLapToDisplay(this.lapCount, lapTime, splitTime);
                    this.showLapSection();
                }
            }
            
            updateDisplay() {
                const time = this.formatTime(this.elapsedTime);
                this.timeDisplay.textContent = time;
            }
            
            formatTime(milliseconds) {
                const totalSeconds = Math.floor(milliseconds / 1000);
                const minutes = Math.floor(totalSeconds / 60);
                const seconds = totalSeconds % 60;
                const ms = Math.floor((milliseconds % 1000));
                
                return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}.${ms.toString().padStart(3, '0')}`;
            }
            
            updateButtons() {
                if (!this.isRunning) {
                    // Stopped state
                    this.startStopBtn.innerHTML = '<i class="fas fa-play mr-2"></i>Start';
                    this.startStopBtn.className = 'control-button start-btn text-white font-semibold py-4 px-6 rounded-xl';
                    
                    this.pauseBtn.disabled = true;
                    this.pauseBtn.className = 'control-button pause-btn text-white font-semibold py-4 px-6 rounded-xl opacity-50 cursor-not-allowed';
                    
                    this.lapBtn.disabled = true;
                    this.lapBtn.className = 'control-button lap-btn text-white font-semibold py-4 px-6 rounded-xl opacity-50 cursor-not-allowed';
                } else if (this.isPaused) {
                    // Paused state
                    this.startStopBtn.innerHTML = '<i class="fas fa-play mr-2"></i>Resume';
                    this.startStopBtn.className = 'control-button start-btn text-white font-semibold py-4 px-6 rounded-xl';
                    
                    this.pauseBtn.innerHTML = '<i class="fas fa-play mr-2"></i>Resume';
                    this.pauseBtn.disabled = false;
                    this.pauseBtn.className = 'control-button start-btn text-white font-semibold py-4 px-6 rounded-xl';
                    
                    this.lapBtn.disabled = true;
                    this.lapBtn.className = 'control-button lap-btn text-white font-semibold py-4 px-6 rounded-xl opacity-50 cursor-not-allowed';
                } else {
                    // Running state
                    this.startStopBtn.innerHTML = '<i class="fas fa-stop mr-2"></i>Stop';
                    this.startStopBtn.className = 'control-button reset-btn text-white font-semibold py-4 px-6 rounded-xl';
                    
                    this.pauseBtn.innerHTML = '<i class="fas fa-pause mr-2"></i>Pause';
                    this.pauseBtn.disabled = false;
                    this.pauseBtn.className = 'control-button pause-btn text-white font-semibold py-4 px-6 rounded-xl';
                    
                    this.lapBtn.disabled = false;
                    this.lapBtn.className = 'control-button lap-btn text-white font-semibold py-4 px-6 rounded-xl';
                }
            }
            
            addLapToDisplay(lapNumber, totalTime, splitTime) {
                const lapItem = document.createElement('div');
                lapItem.className = 'lap-item flex justify-between items-center py-3 px-4 text-white fade-in';
                
                lapItem.innerHTML = `
                    <div class="flex items-center">
                        <span class="bg-white bg-opacity-20 rounded-full w-8 h-8 flex items-center justify-center text-sm font-semibold mr-3">
                            ${lapNumber}
                        </span>
                        <div>
                            <div class="font-medium">Lap ${lapNumber}</div>
                            <div class="text-sm opacity-75">Split: ${this.formatTime(splitTime)}</div>
                        </div>
                    </div>
                    <div class="text-right">
                        <div class="font-mono font-medium">${this.formatTime(totalTime)}</div>
                        <div class="text-sm opacity-75">Total</div>
                    </div>
                `;
                
                this.lapList.insertBefore(lapItem, this.lapList.firstChild);
            }
            
            showLapSection() {
                if (this.lapSection.classList.contains('hidden')) {
                    this.lapSection.classList.remove('hidden');
                    this.lapSection.classList.add('fade-in');
                }
            }
            
            clearLaps() {
                this.lapList.innerHTML = '';
                this.lapSection.classList.add('hidden');
            }
        }
        
        // Initialize the stopwatch when the page loads
        document.addEventListener('DOMContentLoaded', () => {
            new Stopwatch();
        });
        
        // Add keyboard shortcuts
        document.addEventListener('keydown', (event) => {
            if (event.code === 'Space') {
                event.preventDefault();
                document.getElementById('startStopBtn').click();
            } else if (event.key === 'r' || event.key === 'R') {
                document.getElementById('resetBtn').click();
            } else if (event.key === 'p' || event.key === 'P') {
                if (!document.getElementById('pauseBtn').disabled) {
                    document.getElementById('pauseBtn').click();
                }
            } else if (event.key === 'l' || event.key === 'L') {
                if (!document.getElementById('lapBtn').disabled) {
                    document.getElementById('lapBtn').click();
                }
            }
        });
    </script>
</body>
</html># PRODIGY_Wb_02
task 02 of my internship on web development at infotech
