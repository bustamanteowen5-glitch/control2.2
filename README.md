<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Control ATOM + Consola + Sensores Ultrasónicos</title>
    <style>
        body { font-family: sans-serif; display: flex; flex-direction: column; align-items: center; background-color: #1a1a1a; color: white; margin: 0; padding: 20px; }
        .status { margin: 10px; padding: 10px; border-radius: 5px; background: #333; width: 80%; text-align: center; }
        
        /* Estilo de la Consola */
        #console {
            width: 90%; height: 120px; background: black; color: #00ff00;
            font-family: monospace; font-size: 0.9rem; overflow-y: auto;
            border: 2px solid #444; padding: 10px; margin-bottom: 20px; border-radius: 5px;
        }

        /* Estilo de sensores */
        .sensors-container {
            display: flex; gap: 20px; margin-bottom: 20px; width: 90%;
        }
        .sensor {
            flex: 1; padding: 15px; background: #333; border-radius: 10px; text-align: center;
            border: 2px solid #444;
        }
        .sensor h3 { margin: 0 0 10px 0; }
        .sensor-value {
            font-size: 2rem; font-weight: bold; color: #00ff00;
            font-family: monospace;
        }
        .sensor.active { border-color: #00ff88; background: #1a3a1a; }

        .controls { display: grid; grid-template-columns: repeat(3, 80px); gap: 15px; }
        button { width: 80px; height: 80px; border: none; border-radius: 15px; background: #4a4a4a; color: white; font-size: 1.5rem; cursor: pointer; }
        button:active { background: #00ff88; color: black; transform: scale(0.9); }
        .btn-conn { width: 90%; height: 50px; background: #007bff; margin-bottom: 20px; border-radius: 10px; color: white; border: none; font-weight: bold; }
        #btn-stop { background: #dc3545; }
    </style>
</head>
<body>

    <h2>ATOM Monitor + Sensores Ultrasónicos</h2>
    <div id="status" class="status">Estado: Desconectado</div>
    <button class="btn-conn" onclick="conectarBLE()">CONECTAR ROBOT</button>

    <div id="console">Esperando datos del sensor...<br></div>

    <!-- Sensores Ultrasónicos -->
    <div class="sensors-container">
        <div class="sensor">
            <h3>Sensor Ultrasónico (GPIO 16)</h3>
            <div class="sensor-value" id="sensorIzq">-</div>
            <p style="margin: 5px 0; font-size: 0.9rem;">Distancia (cm)</p>
        </div>
        <div class="sensor">
            <h3>Sensor Ultrasónico (GPIO 13)</h3>
            <div class="sensor-value" id="sensorDer">-</div>
            <p style="margin: 5px 0; font-size: 0.9rem;">Distancia (cm)</p>
        </div>
    </div>

    <div class="controls">
        <div style="grid-column: 2"><button onclick="enviar('A')">▲</button></div>
        <button style="grid-row: 2; grid-column: 1" onclick="enviar('B')">◀</button>
        <button id="btn-stop" style="grid-row: 2; grid-column: 2" onclick="enviar('OK')">■</button>
        <button style="grid-row: 2; grid-column: 3" onclick="enviar('X')">▶</button>
        <div style="grid-column: 2; grid-row: 3"><button onclick="enviar('Y')">▼</button></div>
    </div>

    <script>
        const UART_SERVICE_UUID = "6e400001-b5a3-f393-e0a9-e50e24dc0000";
        const RX_CHAR_UUID = "6e400002-b5a3-f393-e0a9-e50e24dc0000"; // Enviar al ESP32
        const TX_CHAR_UUID = "6e400003-b5a3-f393-e0a9-e50e24dc0000"; // Recibir del ESP32

        let caracteristicaRX;
        const consoleDiv = document.getElementById('console');

        function log(msg) {
            consoleDiv.innerHTML += msg + "<br>";
            consoleDiv.scrollTop = consoleDiv.scrollHeight;
        }

        function procesarDatos(value) {
            // Buscar formato: "USONIC_GPIO16:x,USONIC_GPIO13:y"
            const matchGpio16 = value.match(/USONIC_GPIO16:(\d+(?:\.\d+)?)/);
            const matchGpio13 = value.match(/USONIC_GPIO13:(\d+(?:\.\d+)?)/);

            if (matchGpio16) {
                const valGpio16 = parseFloat(matchGpio16[1]);
                document.getElementById('sensorIzq').textContent = valGpio16.toFixed(1);
                document.getElementById('sensorIzq').parentElement.classList.toggle('active', valGpio16 < 20);
            }

            if (matchGpio13) {
                const valGpio13 = parseFloat(matchGpio13[1]);
                document.getElementById('sensorDer').textContent = valGpio13.toFixed(1);
                document.getElementById('sensorDer').parentElement.classList.toggle('active', valGpio13 < 20);
            }
        }

        async function conectarBLE() {
            try {
                const device = await navigator.bluetooth.requestDevice({
                    filters: [{ name: 'ATOM' }],
                    optionalServices: [UART_SERVICE_UUID]
                });

                const server = await device.gatt.connect();
                const service = await server.getPrimaryService(UART_SERVICE_UUID);
                
                // Característica para enviar comandos
                caracteristicaRX = await service.getCharacteristic(RX_CHAR_UUID);
                
                // Característica para recibir datos (Sensor)
                const charTX = await service.getCharacteristic(TX_CHAR_UUID);
                await charTX.startNotifications();
                charTX.addEventListener('characteristicvaluechanged', (event) => {
                    const value = new TextDecoder().decode(event.target.value);
                    log("> " + value);
                    procesarDatos(value);
                });

                document.getElementById('status').innerText = "Conectado a ATOM";
                log("Sistema listo. Leyendo sensores ultrasónicos...");

            } catch (e) { log("Error: " + e); }
        }

        async function enviar(c) {
            if (caracteristicaRX) {
                await caracteristicaRX.writeValue(new TextEncoder().encode(c));
            }
        }
    </script>
</body>
</html>