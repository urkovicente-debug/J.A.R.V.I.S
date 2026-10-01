<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>J.A.R.V.I.S. IA - Google Gemini</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            background-color: #030a16;
            color: #00f0ff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            overflow: hidden;
        }
        .hud-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 10;
        }
        .arc-reactor {
            position: relative;
            width: 180px;
            height: 180px;
            border-radius: 50%;
            border: 4px solid #00f0ff;
            box-shadow: 0 0 35px #00f0ff, inset 0 0 35px #00f0ff;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: all 0.3s ease;
            user-select: none;
        }
        .arc-reactor:hover {
            transform: scale(1.05);
            box-shadow: 0 0 60px #00f0ff, inset 0 0 60px #00f0ff;
        }
        .arc-reactor.escuchando {
            border-color: #ff3366;
            box-shadow: 0 0 60px #ff3366, inset 0 0 60px #ff3366;
            animation: pulso 1.5s infinite;
        }
        .arc-text {
            font-weight: bold;
            font-size: 1.2rem;
            letter-spacing: 3px;
            color: #ffffff;
            text-shadow: 0 0 10px #00f0ff;
        }
        @keyframes pulso {
            0% { transform: scale(1); }
            50% { transform: scale(1.08); }
            100% { transform: scale(1); }
        }
        .status {
            margin-top: 30px;
            font-size: 1.1rem;
            letter-spacing: 2px;
            text-transform: uppercase;
            text-align: center;
            min-height: 30px;
        }
        .console-box {
            margin-top: 20px;
            width: 90%;
            max-width: 500px;
            padding: 15px;
            border: 1px solid rgba(0, 240, 255, 0.3);
            background: rgba(0, 15, 30, 0.6);
            border-radius: 8px;
            text-align: center;
            color: #88c0d0;
            font-size: 0.95rem;
            line-height: 1.5;
            min-height: 60px;
        }
        .api-container {
            display: flex;
            gap: 10px;
            margin-top: 15px;
            width: 90%;
            max-width: 400px;
        }
        .api-input {
            flex: 1;
            padding: 8px;
            background: rgba(0, 15, 30, 0.8);
            border: 1px solid #00f0ff;
            color: #00f0ff;
            border-radius: 4px;
            text-align: center;
        }
        .btn-guardar {
            padding: 8px 15px;
            background: #00f0ff;
            color: #030a16;
            border: none;
            border-radius: 4px;
            font-weight: bold;
            cursor: pointer;
        }
        .btn-guardar:hover {
            background: #ffffff;
        }
    </style>
</head>
<body>

    <div class="hud-container">
        <div class="arc-reactor" id="reactor" onclick="iniciarEscucha()">
            <span class="arc-text">JARVIS</span>
        </div>

        <div class="status" id="estado">Comprobando clave de API...</div>
        
        <div class="api-container">
            <input type="password" id="apiKey" class="api-input" placeholder="Pega tu API Key de Gemini aquí">
            <button class="btn-guardar" onclick="guardarClave()">Guardar</button>
        </div>

        <div class="console-box" id="consola">Cargando Jarvis...</div>
    </div>

    <script>
        const estado = document.getElementById('estado');
        const consola = document.getElementById('consola');
        const reactor = document.getElementById('reactor');
        const apiKeyInput = document.getElementById('apiKey');

        // Al cargar la página, comprobar si la API Key ya está guardada en la memoria del navegador
        window.onload = () => {
            const claveGuardada = localStorage.getItem('jarvis_gemini_key');
            if (claveGuardada) {
                apiKeyInput.value = claveGuardada;
                estado.innerText = "Sistemas listos. Toca el núcleo para hablar";
                consola.innerText = "Clave de API cargada automáticamente.";
            } else {
                estado.innerText = "Introduce tu API Key y pulsa Guardar";
                consola.innerText = "Esperando que configures la API Key de Gemini.";
            }
        };

        function guardarClave() {
            const clave = apiKeyInput.value.trim();
            if (clave) {
                localStorage.setItem('jarvis_gemini_key', clave);
                estado.innerText = "¡Clave guardada con éxito!";
                consola.innerText = "La clave de API ha quedado guardada en este navegador.";
            } else {
                alert("Por favor, introduce una clave válida.");
            }
        }

        const Recognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        if (!Recognition) {
            estado.innerText = "Navegador no compatible";
            consola.innerText = "Usa Google Chrome para esta aplicación.";
        }

        const recognizer = new Recognition();
        recognizer.lang = 'es-ES';

        function iniciarEscucha() {
            const key = apiKeyInput.value.trim();
            if (!key) {
                estado.innerText = "Falta la API Key";
                consola.innerText = "Guarda tu API Key antes de comenzar a hablar.";
                return;
            }

            try {
                recognizer.start();
                reactor.classList.add('escuchando');
                estado.innerText = "Escuchando...";
                consola.innerText = "Escuchando comando...";
            } catch (e) {
                console.log(e);
            }
        }

        recognizer.onresult = async (event) => {
            reactor.classList.remove('escuchando');
            const texto = event.results[0][0].transcript;
            consola.innerText = 'Tú: "' + texto + '"';
            estado.innerText = "Pensando (Gemini)...";

            await consultarGemini(texto);
        };

        recognizer.onerror = () => {
            reactor.classList.remove('escuchando');
            estado.innerText = "Error de escucha";
        };

        recognizer.onend = () => {
            reactor.classList.remove('escuchando');
        };

        async function consultarGemini(mensaje) {
            const apiKey = apiKeyInput.value.trim();
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${apiKey}`;

            const prompt = `Eres J.A.R.V.I.S., la inteligencia artificial de Tony Stark. Responde de manera breve, educada y concisa (máximo 2 o 3 frases) como lo haría Jarvis en español. Pregunta del usuario: ${mensaje}`;

            try {
                const response = await fetch(url, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: prompt }] }]
                    })
                });

                const data = await response.json();

                if (data.candidates && data.candidates[0].content.parts[0].text) {
                    const respuesta = data.candidates[0].content.parts[0].text;
                    consola.innerText = "Jarvis: " + respuesta;
                    hablar(respuesta);
                } else {
                    estado.innerText = "Error en la clave de API";
                    consola.innerText = "No se pudo obtener respuesta. Verifica que la API Key sea correcta.";
                }
            } catch (error) {
                estado.innerText = "Error de conexión";
                consola.innerText = "Ocurrió un error al conectar con los servidores de Google.";
            }
        }

        function hablar(texto) {
            estado.innerText = "Respondiendo...";
            window.speechSynthesis.cancel();

            const utterance = new SpeechSynthesisUtterance(texto);
            utterance.lang = 'es-ES';
            utterance.rate = 1.0;

            utterance.onend = () => {
                estado.innerText = "Toca el núcleo para hablar";
            };

            window.speechSynthesis.speak(utterance);
        }
    </script>
</body>
</html>
