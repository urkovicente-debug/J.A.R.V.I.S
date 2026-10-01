<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>J.A.R.V.I.S. AI System - Stark Industries</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

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

        /* Configuración de API Key */
        .api-container {
            position: absolute;
            top: 20px;
            display: flex;
            gap: 10px;
            z-index: 20;
        }

        .api-input {
            background: rgba(0, 15, 30, 0.8);
            border: 1px solid #00f0ff;
            color: #00f0ff;
            padding: 8px 12px;
            border-radius: 4px;
            outline: none;
            font-size: 0.85rem;
            width: 250px;
        }

        .api-button {
            background: #00f0ff;
            color: #030a16;
            border: none;
            padding: 8px 15px;
            border-radius: 4px;
            font-weight: bold;
            cursor: pointer;
        }

        /* Núcleo Reactivo de Jarvis */
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
            margin-top: 40px;
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

        /* Textos de Estado */
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
    </style>
</head>
<body>

    <div class="api-container">
        <input type="password" id="apiKeyInput" class="api-input" placeholder="Pega tu Gemini API Key aquí">
        <button class="api-button" onclick="guardarApiKey()">Guardar Clave</button>
    </div>

    <div class="hud-container">
        <div class="arc-reactor" id="reactor" onclick="iniciarEscucha()">
            <span class="arc-text">JARVIS</span>
        </div>

        <div class="status" id="estado">Haga clic en el núcleo para hablar</div>
        <div class="console-box" id="consola">Esperando clave de API de Gemini o comando...</div>
    </div>

    <script>
        const estado = document.getElementById('estado');
        const consola = document.getElementById('consola');
        const reactor = document.getElementById('reactor');
        const apiKeyInput = document.getElementById('apiKeyInput');

        // Cargar clave guardada
        let geminiApiKey = localStorage.getItem('GEMINI_API_KEY') || '';
        if (geminiApiKey) {
            apiKeyInput.value = geminiApiKey;
            consola.innerText = "Sistemas listos. Conexión con IA establecida.";
        } else {
            consola.innerText = "Por favor, ingresa tu API Key de Gemini en la parte superior para conectar la IA.";
        }

        function guardarApiKey() {
            geminiApiKey = apiKeyInput.value.trim();
            localStorage.setItem('GEMINI_API_KEY', geminiApiKey);
            alert("Clave API guardada correctamente.");
            consola.innerText = "Sistemas listos. Conexión con IA establecida.";
        }

        // Configuración del Reconocimiento de Voz
        const Recognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        const recognizer = new Recognition();
        recognizer.lang = 'es-ES';
        recognizer.continuous = false;

        function iniciarEscucha() {
            if (!geminiApiKey) {
                alert("Primero ingresa y guarda tu API Key de Gemini en la parte superior.");
                return;
            }
            try {
                recognizer.start();
                reactor.classList.add('escuchando');
                estado.innerText = "Escuchando...";
                consola.innerText = "Hable ahora, señor...";
            } catch (e) {
                console.log(e);
            }
        }

        recognizer.onresult = async (event) => {
            reactor.classList.remove('escuchando');
            const texto = event.results[0][0].transcript;
            consola.innerText = 'Tú: "' + texto + '"';
            estado.innerText = "Pensando (Consultando a la IA)...";

            // Consultar a Gemini API
            await consultarGemini(texto);
        };

        recognizer.onerror = () => {
            reactor.classList.remove('escuchando');
            estado.innerText = "Error al escuchar";
        };

        // Función para conectar con la API de Gemini
        async function consultarGemini(pregunta) {
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${geminiApiKey}`;

            const promptSistema = `Eres J.A.R.V.I.S., el asistente de inteligencia artificial de Iron Man. Responde de forma muy breve (máximo 2 o 3 frases cortos), educada, formal, llamándome 'señor', y manteniendo siempre el personaje de una IA futurista de alta tecnología. Pregunta del usuario: ${pregunta}`;

            try {
                const response = await fetch(url, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{
                            parts: [{ text: promptSistema }]
                        }]
                    })
                });

                const data = await response.json();
                
                if (data.candidates && data.candidates[0].content.parts[0].text) {
                    const respuesta = data.candidates[0].content.parts[0].text;
                    consola.innerText = "Jarvis: " + respuesta;
                    hablar(respuesta);
                } else {
                    estado.innerText = "Error en la respuesta";
                    consola.innerText = "Error en el formato de respuesta de la API.";
                }
            } catch (error) {
                estado.innerText = "Error de conexión";
                consola.innerText = "Hubo un error al conectar con Gemini. Revisa tu clave API.";
            }
        }

        // Síntesis de Voz
        function hablar(texto) {
            estado.innerText = "Respondiendo...";
            window.speechSynthesis.cancel();

            const utterance = new SpeechSynthesisUtterance(texto);
            utterance.lang = 'es-ES';
            utterance.rate = 1.0;

            utterance.onend = () => {
                estado.innerText = "Haga clic en el núcleo para hablar";
            };

            window.speechSynthesis.speak(utterance);
        }
    </script>
</body>
</html>
