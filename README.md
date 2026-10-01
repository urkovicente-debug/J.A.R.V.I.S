# J.A.R.V.I.S
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>J.A.R.V.I.S. System - Stark Industries</title>
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

    <div class="hud-container">
        <!-- Núcleo de Jarvis -->
        <div class="arc-reactor" id="reactor" onclick="iniciarEscucha()">
            <span class="arc-text">JARVIS</span>
        </div>

        <div class="status" id="estado">Haga clic en el núcleo para hablar</div>
        <div class="console-box" id="consola">Esperando comando de voz...</div>
    </div>

    <script>
        const estado = document.getElementById('estado');
        const consola = document.getElementById('consola');
        const reactor = document.getElementById('reactor');

        // Configuración del Reconocimiento de Voz
        const Recognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        
        if (!Recognition) {
            estado.innerText = "Navegador no compatible";
            consola.innerText = "Error: Tu navegador no admite el reconocimiento de voz. Usa Google Chrome.";
        }

        const recognizer = new Recognition();
        recognizer.lang = 'es-ES';
        recognizer.continuous = false;
        recognizer.interimResults = false;

        function iniciarEscucha() {
            try {
                recognizer.start();
                reactor.classList.add('escuchando');
                estado.innerText = "Escuchando...";
                consola.innerText = "Hable ahora, señor...";
            } catch (e) {
                console.log("Ya se está escuchando o error de inicio.");
            }
        }

        recognizer.onresult = (event) => {
            reactor.classList.remove('escuchando');
            const texto = event.results[0][0].transcript;
            consola.innerText = 'Tú: "' + texto + '"';
            estado.innerText = "Procesando...";

            // Procesar la orden
            procesarComando(texto.toLowerCase());
        };

        recognizer.onerror = (event) => {
            reactor.classList.remove('escuchando');
            estado.innerText = "Error al escuchar";
            consola.innerText = "No se ha podido procesar el audio. Inténtelo de nuevo.";
        };

        recognizer.onend = () => {
            reactor.classList.remove('escuchando');
        };

        // Lógica de Comandos de Jarvis
        function procesarComando(comando) {
            let respuesta = "";

            if (comando.includes("hola") || comando.includes("saludos")) {
                respuesta = "Saludos, señor. Todos los sistemas están funcionando al cien por ciento.";
            } 
            else if (comando.includes("quién eres") || comando.includes("quien eres") || comando.includes("tu nombre")) {
                respuesta = "Soy Jarvis, su asistente personal con interfaz web ejecutándose directamente desde GitHub.";
            } 
            else if (comando.includes("hora")) {
                const ahora = new Date();
                const horas = ahora.getHours();
                const minutos = ahora.getMinutes();
                respuesta = `Son las ${horas} horas con ${minutos} minutos.`;
            } 
            else if (comando.includes("fecha") || comando.includes("día")) {
                const opciones = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
                const hoy = new Date().toLocaleDateString('es-ES', opciones);
                respuesta = `Hoy es ${hoy}.`;
            } 
            else if (comando.includes("abrir youtube")) {
                respuesta = "Abriendo YouTube inmediatamente.";
                window.open("https://www.youtube.com", "_blank");
            } 
            else if (comando.includes("abrir google")) {
                respuesta = "Abriendo el buscador de Google.";
                window.open("https://www.google.com", "_blank");
            } 
            else if (comando.includes("estado del sistema")) {
                respuesta = "Núcleo reactivo estable. Conexión a GitHub Pages activa. Micrófono operativo.";
            } 
            else {
                respuesta = `He recibido su comando: "${comando}". No tengo una acción específica configurada para ello todavía.`;
            }

            consola.innerText = "Jarvis: " + respuesta;
            hablar(respuesta);
        }

        // Síntesis de Voz
        function hablar(texto) {
            estado.innerText = "Respondiendo...";
            window.speechSynthesis.cancel(); // Cancelar locuciones previas

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
