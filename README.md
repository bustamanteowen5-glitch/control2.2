
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Control ATOM + Consola</title>
    <style>
        body { font-family: sans-serif; display: flex; flex-direction: column; align-items: center; background-color: #1a1a1a; color: white; margin: 0; padding: 20px; }
        .status { margin: 10px; padding: 10px; border-radius: 5px; background: #333; width: 80%; text-align: center; }
        
        /* Estilo de la Consola */
        #console {
            width: 90%; height: 120px; background: black; color: #00ff00;
            font-family: monospace; font-size: 0.9rem; overflow-y: auto;
            border: 2px solid #444; padding: 10px; margin-bottom: 20px; border-radius: 5px;
        }

        .controls { display: grid; grid-template-columns: repeat(3, 80px); gap: 15px; }
        button { width: 80px; height: 80px; border: none; border-radius: 15px; background: #4a4a4a; color: white; font-size: 1.5rem; cursor: pointer; }
        button:active { background: #00ff88; color: black; transform: scale(0.9); }
        .btn-conn { width: 90%; height: 50px; background: #007bff; margin-bottom: 20px; border-radius: 10px; color: white; border: none; font-weight: bold; }
        #btn-stop { background: #dc3545; }
    </style>
</head>
<body>

    <h2>ATOM Monitor</h2>
    <div id="status" class="status">Estado: Desconectado</div>
    <button class="btn-conn" onclick="conectarBLE()">CONECTAR ROBOT</button>

    <div id="console">Esperando datos del sensor...<br></div>

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
                });

                document.getElementById('status').innerText = "Conectado a ATOM";
                log("Sistema listo.");

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
