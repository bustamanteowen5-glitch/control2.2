<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ATOM Terminal Monitor - Full</title>
    <style>
        body { font-family: sans-serif; display: flex; flex-direction: column; align-items: center; background-color: #1a1a1a; color: white; margin: 0; padding: 20px; }
        .status { margin: 10px; padding: 10px; border-radius: 5px; background: #333; width: 90%; text-align: center; font-weight: bold; }
        
        /* Consola optimizada */
        #console {
            width: 90%; height: 350px; background: black; color: #00ff00;
            font-family: 'Courier New', monospace; font-size: 1rem; overflow-y: auto;
            border: 2px solid #444; padding: 15px; margin-bottom: 20px; border-radius: 8px;
            box-shadow: inset 0 0 10px #000;
        }

        .controls { display: grid; grid-template-columns: repeat(3, 85px); gap: 15px; }
        button { width: 85px; height: 85px; border: none; border-radius: 15px; background: #4a4a4a; color: white; font-size: 1.8rem; cursor: pointer; transition: 0.2s; }
        button:active { background: #00ff88; color: black; transform: scale(0.9); }
        .btn-conn { width: 90%; height: 55px; background: #007bff; margin-bottom: 20px; border-radius: 10px; color: white; border: none; font-weight: bold; font-size: 1.1rem; cursor: pointer; }
        #btn-stop { background: #dc3545; }
        h2 { margin-bottom: 5px; color: #007bff; }
    </style>
</head>
<body>

    <h2>ATOM MONITOR</h2>
    <div id="status" class="status">ESTADO: DESCONECTADO</div>
    <button class="btn-conn" onclick="conectarBLE()">CONECTAR ROBOT VIA BLUETOOTH</button>

    <div id="console">Sistema listo. Esperando conexión...<br></div>

    <div class="controls">
        <div style="grid-column: 2"><button onclick="enviarComando('A')">▲</button></div>
        <button style="grid-row: 2; grid-column: 1" onclick="enviarComando('B')">◀</button>
        <button id="btn-stop" style="grid-row: 2; grid-column: 2" onclick="enviarComando('OK')">■</button>
        <button style="grid-row: 2; grid-column: 3" onclick="enviarComando('X')">▶</button>
        <div style="grid-column: 2; grid-row: 3"><button onclick="enviarComando('Y')">▼</button></div>
    </div>

    <script>
        const UART_SERVICE_UUID = "6e400001-b5a3-f393-e0a9-e50e24dc0000";
        const RX_CHAR_UUID = "6e400002-b5a3-f393-e0a9-e50e24dc0000"; 
        const TX_CHAR_UUID = "6e400003-b5a3-f393-e0a9-e50e24dc0000"; 

        let caracteristicaRX;
        const consoleDiv = document.getElementById('console');

        function logToConsole(msg) {
            let cleanMsg = msg.trim();
            if (!cleanMsg) return;

            // Agregamos el prefijo de terminal ">"
            const newLine = document.createElement("div");
            newLine.textContent = "> " + cleanMsg;
            
            // Si es un dato de sensor, le damos un toque de color diferente si prefieres
            if (cleanMsg.includes("IR Izq")) {
                newLine.style.color = "#88ff88"; 
            }

            consoleDiv.appendChild(newLine);
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
                caracteristicaRX = await service.getCharacteristic(RX_CHAR_UUID);
                
                const charTX = await service.getCharacteristic(TX_CHAR_UUID);
                await charTX.startNotifications();
                
                charTX.addEventListener('characteristicvaluechanged', (event) => {
                    const value = new TextDecoder().decode(event.target.value);
                    logToConsole(value);
                });

                document.getElementById('status').innerText = "ESTADO: CONECTADO A ATOM";
                document.getElementById('status').style.color = "#00ff88";
                logToConsole("¡Conexión Exitosa!");

            } catch (e) { 
                logToConsole("Error de conexión: " + e); 
            }
        }

        async function enviarComando(letra) {
            if (caracteristicaRX) {
                try {
                    await caracteristicaRX.writeValue(new TextEncoder().encode(letra));
                } catch (e) {
                    logToConsole("Error al enviar: " + e);
                }
            } else {
                alert("Primero debes conectar el robot");
            }
        }
    </script>
</body>
</html>
