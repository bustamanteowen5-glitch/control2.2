<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Control ATOM</title>
</head>
<body>

    <!-- CONTENEDOR DE CONTROL (Mantiene tu grid y diseño original intacto) -->
    <div style="display: grid; width: 200px; height: 200px; margin: 20px auto;">
        
        <button id="btn-stop" style="grid-row: 2; grid-column: 2" onclick="enviarComando('OK')">■</button>

        <button style="grid-row: 2; grid-column: 3" 
                onmousedown="enviarComando('X')" 
                onmouseup="enviarComando('OK')" 
                onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('X')" 
                ontouchend="event.preventDefault(); enviarComando('OK')">▶</button>

        <div style="grid-column: 2; grid-row: 3">
            <button 
                onmousedown="enviarComando('Y')" 
                onmouseup="enviarComando('OK')" 
                onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('Y')" 
                ontouchend="event.preventDefault(); enviarComando('OK')">▼</button>
        </div>

    </div>

    <!-- TERMINAL / CONSOLA ORIGINAL -->
    <div id="status" style="text-align: center; font-weight: bold; margin-bottom: 10px;">ESTADO: DESCONECTADO</div>
    <div id="console" style="width: 300px; height: 200px; background-color: #1e1e1e; color: #ffffff; font-family: monospace; padding: 10px; margin: 0 auto; overflow-y: auto; border-radius: 5px;"></div>

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
            
            // Detecta la telemetría para cambiar el color en la terminal
            if (cleanMsg.includes("IR_IZQ") || cleanMsg.includes("IR Izq")) {
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
