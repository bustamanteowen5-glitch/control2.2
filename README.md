<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: #0d1117;
            color: #ffffff;
        }
        
        #console {
            width: 300px;
            height: 200px;
            background-color: #1e1e1e;
            color: #ffffff;
            font-family: monospace;
            padding: 10px;
            margin: 20px auto;
            overflow-y: auto;
            border-radius: 5px;
            border: 1px solid #30363d;
            font-size: 12px;
        }
        
        button {
            padding: 10px 20px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            font-weight: bold;
        }
        
        button:hover {
            background-color: #0056b3;
        }
        
        button:active {
            background-color: #003f87;
        }
    </style>
</head>
<body>
    <h1>🤖 CONTROL ROBOT ATOM</h1>
    
    <div style="margin-top: 20px;">
        <button onclick="conectarBLE()" style="padding: 15px 30px; font-size: 18px; width: 200px;">
            🔌 CONECTAR ROBOT ATOM
        </button>
    </div>

    <div id="status" style="font-weight: bold; margin: 15px 0; font-size: 18px; color: #ff6b6b;">
        ESTADO: DESCONECTADO
    </div>

    <h2>Control D-Pad</h2>
    <div style="display: grid; width: 220px; height: 220px; margin: 20px auto; grid-template-columns: repeat(3, 1fr); grid-template-rows: repeat(3, 1fr); gap: 5px;">
        
        <!-- Arriba -->
        <button style="grid-row: 1; grid-column: 2; font-size: 24px;" 
                onmousedown="enviarComando('W')" 
                onmouseup="enviarComando('OK')" 
                onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('W')" 
                ontouchend="event.preventDefault(); enviarComando('OK')">▲</button>

        <!-- Izquierda -->
        <button style="grid-row: 2; grid-column: 1; font-size: 24px;"
                onmousedown="enviarComando('A')" 
                onmouseup="enviarComando('OK')" 
                onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('A')" 
                ontouchend="event.preventDefault(); enviarComando('OK')">◀</button>

        <!-- Centro (STOP) -->
        <button id="btn-stop" style="grid-row: 2; grid-column: 2; font-size: 32px; background-color: #dc3545;" onclick="enviarComando('OK')">■</button>

        <!-- Derecha -->
        <button style="grid-row: 2; grid-column: 3; font-size: 24px;" 
                onmousedown="enviarComando('X')" 
                onmouseup="enviarComando('OK')" 
                onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('X')" 
                ontouchend="event.preventDefault(); enviarComando('OK')">▶</button>

        <!-- Abajo -->
        <button style="grid-row: 3; grid-column: 2; font-size: 24px;"
                onmousedown="enviarComando('Y')" 
                onmouseup="enviarComando('OK')" 
                onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('Y')" 
                ontouchend="event.preventDefault(); enviarComando('OK')">▼</button>
    </div>

    <h2>Terminal</h2>
    <div id="console"></div>

    <script>
        const UART_SERVICE_UUID = "6e400001-b5a3-f393-e0a9-e50e24dc0000";
        const RX_CHAR_UUID = "6e400002-b5a3-f393-e0a9-e50e24dc0000"; 
        const TX_CHAR_UUID = "6e400003-b5a3-f393-e0a9-e50e24dc0000"; 

        let caracteristicaRX;
        const consoleDiv = document.getElementById('console');

        function logToConsole(msg) {
            let cleanMsg = msg.trim();
            if (!cleanMsg) return;
            const newLine = document.createElement("div");
            newLine.textContent = "> " + cleanMsg;
            
            // Color para telemetría
            if (cleanMsg.includes("IR_IZQ") || cleanMsg.includes("Dist")) {
                newLine.style.color = "#88ff88"; 
            }
            consoleDiv.appendChild(newLine);
            consoleDiv.scrollTop = consoleDiv.scrollHeight;
        }

        async function conectarBLE() {
            try {
                logToConsole("Buscando a ATOM...");
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

                document.getElementById('status').innerText = "ESTADO: CONECTADO A ATOM ✅";
                document.getElementById('status').style.color = "#00ff88";
                logToConsole("¡Conexión Exitosa!");

            } catch (e) { 
                logToConsole("Error: " + e.message); 
            }
        }

        async function enviarComando(letra) {
            if (caracteristicaRX) {
                try {
                    await caracteristicaRX.writeValue(new TextEncoder().encode(letra));
                } catch (e) {
                    console.log("Error al enviar: " + e);
                }
            } else if (letra !== 'OK') {
                logToConsole("Aviso: Primero conecta el robot.");
            }
        }
    </script>
</body>
</html>
