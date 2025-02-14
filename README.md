<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>INCU Analyzer</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mqtt/4.3.7/mqtt.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            padding: 20px;
            background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
            color: #fff;
            min-height: 100vh;
        }

        .title {
            text-align: center;
            font-size: 2.5em;
            margin-bottom: 30px;
            color: #fff;
            text-shadow: 0 0 10px rgba(52, 152, 219, 0.5);
            animation: glow 2s ease-in-out infinite alternate;
        }

        @keyframes glow {
            from { text-shadow: 0 0 10px rgba(52, 152, 219, 0.5); }
            to { text-shadow: 0 0 20px rgba(52, 152, 219, 0.8); }
        }

        .mqtt-status {
            text-align: center;
            margin-bottom: 20px;
            padding: 10px;
            border-radius: 5px;
            transition: all 0.3s ease;
        }

        .mqtt-status.connected {
            background: rgba(46, 204, 113, 0.2);
            color: #2ecc71;
        }

        .mqtt-status.disconnected {
            background: rgba(231, 76, 60, 0.2);
            color: #e74c3c;
        }

        .broker-info {
            text-align: center;
            margin-bottom: 20px;
            color: #bdc3c7;
            font-size: 0.9em;
        }

        .control-buttons {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 30px;
        }

        .control-btn {
            padding: 15px 30px;
            font-size: 1.1em;
            border: none;
            border-radius: 8px;
            background: rgba(52, 152, 219, 0.8);
            color: white;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
        }

        .control-btn:hover {
            transform: translateY(-3px);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
            background: rgba(52, 152, 219, 1);
        }

        .control-btn.active {
            background: rgba(231, 76, 60, 0.8);
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }

        .input-container {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 30px;
        }

        .input-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }

        .input-group label {
            font-weight: bold;
            color: #fff;
        }

        .input-group input {
            padding: 10px;
            border: 2px solid rgba(52, 152, 219, 0.5);
            border-radius: 5px;
            font-size: 1em;
            background: rgba(255, 255, 255, 0.1);
            color: #fff;
            transition: all 0.3s ease;
        }

        .input-group input:focus {
            outline: none;
            border-color: rgba(52, 152, 219, 1);
            background: rgba(255, 255, 255, 0.2);
        }

        .timer-display {
            text-align: center;
            font-size: 3em;
            margin-bottom: 30px;
            color: #fff;
            text-shadow: 0 0 10px rgba(52, 152, 219, 0.5);
            font-family: 'Courier New', monospace;
        }

        .sensor-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
            max-width: 1200px;
            margin-left: auto;
            margin-right: auto;
        }

        .sensor-box {
            background: rgba(255, 255, 255, 0.1);
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
            text-align: center;
            transition: all 0.3s ease;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .sensor-box:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
            background: rgba(255, 255, 255, 0.15);
        }

        .sensor-box.updated {
            animation: highlight 1s ease-out;
        }

        @keyframes highlight {
            0% { background: rgba(52, 152, 219, 0.3); }
            100% { background: rgba(255, 255, 255, 0.1); }
        }

        .sensor-box h3 {
            margin-bottom: 10px;
            color: #fff;
        }

        .sensor-value {
            font-size: 1.5em;
            color: #3498db;
            text-shadow: 0 0 5px rgba(52, 152, 219, 0.5);
            transition: all 0.3s ease;
        }

        .data-table {
            width: 100%;
            border-collapse: separate;
            border-spacing: 0;
            margin-top: 20px;
            background: rgba(255, 255, 255, 0.1);
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
            border-radius: 10px;
            overflow: hidden;
        }

        .data-table th, .data-table td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
        }

        .data-table th {
            background: rgba(52, 152, 219, 0.8);
            color: white;
        }

        .data-table tr {
            transition: all 0.3s ease;
        }

        .data-table tr:hover {
            background: rgba(255, 255, 255, 0.15);
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .new-row {
            animation: fadeIn 0.5s ease-out;
        }
    </style>
    
    <div id="mqttStatus" class="mqtt-status disconnected">
        MQTT Status: Disconnected
    </div>
    
    <div class="control-buttons">
        <button id="saveBtn" class="control-btn">Play Saving Data</button>
        <button id="resetBtn" class="control-btn">Reset Data</button>
        <button id="exportBtn" class="control-btn">Export Data</button>
    </div>

    <div class="input-container">
        <div class="input-group">
            <label>Interval (seconds)</label>
            <input type="number" id="intervalInput" min="1" value="2">
        </div>
        <div class="input-group">
            <label>Timer (HH:MM:SS)</label>
            <input type="text" id="timerInput" placeholder="00:00:00" pattern="[0-9]{2}:[0-9]{2}:[0-9]{2}" value="00:01:00">
        </div>
    </div>

    <div class="timer-display" id="timerDisplay">00:00:00</div>

    <div class="sensor-grid">
        <div class="sensor-box">
            <h3>T1</h3>
            <div class="sensor-value" id="t1Value">0.0 °C</div>
        </div>
        <div class="sensor-box">
            <h3>T2</h3>
            <div class="sensor-value" id="t2Value">0.0 °C</div>
        </div>
        <div class="sensor-box">
            <h3>T3</h3>
            <div class="sensor-value" id="t3Value">0.0 °C</div>
        </div>
        <div class="sensor-box">
            <h3>T4</h3>
            <div class="sensor-value" id="t4Value">0.0 °C</div>
        </div>
        <div class="sensor-box">
            <h3>T5</h3>
            <div class="sensor-value" id="t5Value">0.0 °C</div>
        </div>
        <div class="sensor-box">
            <h3>TM</h3>
            <div class="sensor-value" id="tmValue">0.0 °C</div>
        </div>
        <div class="sensor-box">
            <h3>Flow</h3>
            <div class="sensor-value" id="flowValue">0.0 m/s</div>
        </div>
        <div class="sensor-box">
            <h3>Noise</h3>
            <div class="sensor-value" id="noiseValue">0.0 dB</div>
        </div>
        <div class="sensor-box">
            <h3>RH</h3>
            <div class="sensor-value" id="rhValue">0.0 %</div>
        </div>
    </div>

    <table class="data-table">
        <thead>
            <tr>
                <th>Date</th>
                <th>Time</th>
                <th>T1 (°C)</th>
                <th>T2 (°C)</th>
                <th>T3 (°C)</th>
                <th>T4 (°C)</th>
                <th>T5 (°C)</th>
                <th>TM (°C)</th>
                <th>Flow (m/s)</th>
                <th>Noise (dB)</th>
                <th>RH (%)</th>
            </tr>
        </thead>
        <tbody id="dataTableBody"></tbody>
    </table>

    <script>
        // MQTT Configuration
        const options = {
            protocol: 'ws',
            hostname: 'broker.hivemq.com',
            port: 8000,
            path: '/mqtt',
            clean: true,
            connectTimeout: 4000,
            reconnectPeriod: 1000,
            clientId: 'incu_analyzer_' + Math.random().toString(16).substr(2, 8)
        };

        const client = mqtt.connect(options);
        const topic = 'incu/sensors';
        let isRecording = false;
        let timerInterval;
        let remainingTime;
        let tableData = [];

        // MQTT Connection handling
        client.on('connect', () => {
            console.log('Connected to MQTT broker');
            document.getElementById('mqttStatus').className = 'mqtt-status connected';
            document.getElementById('mqttStatus').textContent = 'MQTT Status: Connected';
            client.subscribe(topic);
        });

        client.on('error', (error) => {
            console.error('MQTT Error:', error);
            document.getElementById('mqttStatus').className = 'mqtt-status disconnected';
            document.getElementById('mqttStatus').textContent = 'MQTT Status: Error - ' + error.message;
        });

        client.on('offline', () => {
            document.getElementById('mqttStatus').className = 'mqtt-status disconnected';
            document.getElementById('mqttStatus').textContent = 'MQTT Status: Offline';
        });

        client.on('message', (topic, message) => {
            if (isRecording) {
                try {
                    const data = JSON.parse(message.toString());
                    updateSensorValues(data);
                    addTableRow(data);
                    highlightUpdatedValues();
                } catch (e) {
                    console.error('Error parsing MQTT message:', e);
                }
            }
        });

        function highlightUpdatedValues() {
            const boxes = document.querySelectorAll('.sensor-box');
            boxes.forEach(box => {
                box.classList.add('updated');
                setTimeout(() => box.classList.remove('updated'), 1000);
            });
        }

        function updateSensorValues(data) {
            document.getElementById('t1Value').textContent = `${data.t1.toFixed(1)} °C`;
            document.getElementById('t2Value').textContent = `${data.t2.toFixed(1)} °C`;
            document.getElementById('t3Value').textContent = `${data.t3.toFixed(1)} °C`;
            document.getElementById('t4Value').textContent = `${data.t4.toFixed(1)} °C`;
            document.getElementById('t5Value').textContent = `${data.t5.toFixed(1)} °C`;
            document.getElementById('tmValue').textContent = `${data.tm.toFixed(1)} °C`;
            document.getElementById('flowValue').textContent = `${data.flow.toFixed(1)} m/s`;
            document.getElementById('noiseValue').textContent = `${data.noise.toFixed(1)} dB`;
            document.getElementById('rhValue').textContent = `${data.rh.toFixed(1)} %`;
        }

        function addTableRow(data) {
            const now = new Date();
            const row = {
                date: now.toLocaleDateString(),
                time: now.toLocaleTimeString(),
                ...data
            };
            tableData.push(row);
            
            const tr = document.createElement('tr');
            tr.classList.add('new-row');
            tr.innerHTML = `
                <td>${row.date}</td>
                <td>${row.time}</td>
                <td>${row.t1.toFixed(1)}</td>
                <td>${row.
