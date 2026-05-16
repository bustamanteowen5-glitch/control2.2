<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Control ATOM</title>
</head>
<body>
    <!-- Se asume la existencia de las etiquetas previas de estructura como #status, #console, etc. -->

    <!-- CONTROLES DE MOVIMIENTO -->
    <div style="display: grid; width: 200px; height: 200px; margin: 20px auto;">
        <!-- Botón Adelante -->
        <div style="grid-column: 2; grid-row: 1;">
            <button 
                onmousedown="enviarComando('Y')" 
                onmouseup="enviarComando('OK')" 
                onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('Y')" 
                ontouchend="event.preventDefault(); enviarComando('OK')">▲</button>
        </div>

        <!-- Botón Izquierda -->
        <button style="grid-row: 2; grid-column: 1" 
                onmousedown="enviarComando('A')" 
                onmouseup="enviarComando('OK')" 
                onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('A')" 
                ontouchend="event.preventDefault(); enviarComando('OK')">◀</button>

        <!-- Botón Stop (Opcional por seguridad) -->
        <button id="btn-stop" style="grid-row: 2; grid-column: 2" onclick="enviarComando('OK')">■</button>

        <!-- Botón Derecha -->
        <button style="grid-row: 2; grid-column: 3" 
                onmousedown="enviarComando('B')" 
                onmouseup="enviarComando('OK')" 
                onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('B')" 
                ontouchend="event.preventDefault(); enviarComando('OK')">▶</button>

        <!-- Botón Atrás -->
        <div style="grid-column: 2; grid-row: 3">
            <button 
                onmousedown="enviarComando('X')" 
                onmouseup="enviarComando('OK')" 
                onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('X')" 
                ontouchend="event.preventDefault(); enviarComando('OK')">▼</button>
        </div>
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

            const newLine = document.createElement("div");
            newLine.textContent = "> " + cleanMsg;
            
            // Actualizado para detectar la cadena del nuevo formato de MicroPython
            if (cleanMsg.includes("IR_IZQ")) {
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
                console.log("Primero debes conectar el robot");
            }
        }
    </script>
</body>
</html>
