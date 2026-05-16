<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Control ATOM</title>
    <style>
        /* Estilo para que los botones se vean como cuadrados uniformes */
        .control-grid button {
            width: 60px;
            height: 60px;
            font-size: 20px;
            cursor: pointer;
        }
    </style>
</head>
<body>

    <div id="status" style="text-align: center; font-weight: bold; margin: 10px 0;">ESTADO: DESCONECTADO</div>

    <div class="control-grid" style="display: grid; grid-template-columns: repeat(3, 60px); grid-template-rows: repeat(3, 60px); gap: 5px; justify-content: center; margin: 20px auto;">
        
        <button style="grid-row: 1; grid-column: 2" 
                onmousedown="enviarComando('Y')" onmouseup="enviarComando('OK')" onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('Y')" ontouchend="event.preventDefault(); enviarComando('OK')">▲</button>

        <button style="grid-row: 2; grid-column: 1" 
                onmousedown="enviarComando('A')" onmouseup="enviarComando('OK')" onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('A')" ontouchend="event.preventDefault(); enviarComando('OK')">◀</button>

        <button id="btn-stop" style="grid-row: 2; grid-column: 2" onclick="enviarComando('OK')">■</button>

        <button style="grid-row: 2; grid-column: 3" 
                onmousedown="enviarComando('B')" onmouseup="enviarComando('OK')" onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('B')" ontouchend="event.preventDefault(); enviarComando('OK')">▶</button>

        <button style="grid-row: 3; grid-column: 2" 
                onmousedown="enviarComando('X')" onmouseup="enviarComando('OK')" onmouseleave="enviarComando('OK')"
                ontouchstart="event.preventDefault(); enviarComando('X')" ontouchend="event.preventDefault(); enviarComando('OK')">▼</button>
    </div>

    <div style="text-align: center; margin: 20px;">
        <button onclick="conectarBLE()" style="padding: 10px 20px; background-color: #007bff; color: white; border: none; border-radius: 5px;">CONECTAR ROBOT</button>
    </div>

    <div id="console" style="width: 320px; height: 200px; background-color: #1e1e1e; color: #ffffff; font-family: monospace; padding: 10px; margin: 0 auto; overflow-y: auto; border: 2px solid #333; border-radius: 5px;"></div>

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
            if (cleanMsg.includes("IR_IZQ")) newLine.style.color = "#88ff88"; 
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
                document.getElementById('status').innerText = "ESTADO: CONECTADO";
                document.getElementById('status').style.color = "#00ff88";
            } catch (e) { logToConsole("Error: " + e); }
        }

        async function enviarComando(letra) {
            if (caracteristicaRX) {
                try {
                    await caracteristicaRX.writeValue(new TextEncoder().encode(letra));
                } catch (e) { console.log(e); }
            }
        }
    </script>
</body>
</html>
