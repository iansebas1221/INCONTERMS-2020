<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Manual Interactivo Incoterms 2020 (Canvas)</title>
    <!-- Carga de Tailwind CSS para el body y elementos fuera del Canvas (aunque el contenido principal va en Canvas) -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Estilos base para centrar el canvas y hacerlo visualmente agradable */
        body {
            font-family: 'Inter', sans-serif;
            transition: background-color 0.3s;
        }
        /* Contenedor principal que se ajusta a la pantalla */
        #canvasContainer {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            padding: 16px;
            box-sizing: border-box;
            background-color: #e5e7eb; /* Fondo suave fuera del canvas */
        }
        /* El elemento canvas principal */
        #manualCanvas {
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
            border-radius: 12px;
            display: block;
            touch-action: manipulation; /* Mejora la experiencia táctil */
            cursor: pointer; /* Cursor de puntero para indicar interactividad */
        }
        /* Estilos para los botones de control (fuera del canvas) */
        .control-panel {
            position: absolute;
            top: 20px;
            right: 20px;
            display: flex;
            gap: 10px;
        }
        .control-button {
            padding: 8px 12px;
            border-radius: 8px;
            background-color: #3b82f6;
            color: white;
            font-weight: 600;
            cursor: pointer;
            transition: background-color 0.2s;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .control-button:hover {
            background-color: #2563eb;
        }
        .dark-mode {
            background-color: #1f2937;
        }
        /* Indicador de carga para la API */
        .loading-indicator {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 20px 40px;
            background: rgba(0, 0, 0, 0.8);
            color: white;
            border-radius: 10px;
            font-size: 1.2rem;
            z-index: 1000;
            display: none;
        }
    </style>
</head>
<body class="bg-gray-100">

    <div id="canvasContainer">
        <canvas id="manualCanvas"></canvas>
    </div>

    <!-- Panel de Controles -->
    <div class="control-panel">
        <div id="darkModeToggle" class="control-button">🌙 Modo Oscuro</div>
        <div id="textZoomIn" class="control-button text-xl">+ Aumentar</div>
        <div id="textZoomOut" class="control-button text-xl">– Reducir</div>
    </div>
    
    <div id="loadingIndicator" class="loading-indicator">Cargando...</div>

    <script type="module">
        // =================================================================================
        // CONFIGURACIÓN DE LA APLICACIÓN Y CONSTANTES
        // =================================================================================

        const canvas = document.getElementById('manualCanvas');
        const ctx = canvas.getContext('2d');
        const loadingIndicator = document.getElementById('loadingIndicator');

        // Dimensiones base del diseño (usadas para el escalado responsivo)
        const BASE_WIDTH = 900;
        const BASE_HEIGHT = 1200;
        
        let scaleFactor = 1.0;
        let currentScreenId = 'inicio';
        let isTransitioning = false;
        let transitionProgress = 0; // 0 (start) to 1 (end)
        let fontSizeScale = 1.0; // Factor de zoom de texto
        let isDarkMode = false;
        let scrollOffset = 0; // Para la simulación de scroll interno

        // Definición de colores base
        const COLORS = {
            light: {
                background: '#ffffff',
                primary: '#3b82f6', // Azul para encabezados y botones
                secondary: '#10b981', // Verde para acentos
                text: '#1f2937', // Gris oscuro
                textSoft: '#6b7280', // Gris medio
                boxBackground: '#f8f8f8',
                shadow: 'rgba(0, 0, 0, 0.1)'
            },
            dark: {
                background: '#1f2937',
                primary: '#60a5fa',
                secondary: '#34d399',
                text: '#f3f4f6',
                textSoft: '#9ca3af',
                boxBackground: '#374151',
                shadow: 'rgba(255, 255, 255, 0.1)'
            }
        };

        // Función auxiliar para obtener el esquema de color actual
        const getCurrentColors = () => isDarkMode ? COLORS.dark : COLORS.light;


        // =================================================================================
        // 📚 CONTENIDO DEL MANUAL (SECTIONS_CONTENT)
        // =================================================================================

        const MANUAL_CONTENT = {
            inicio: {
                title: "Manual Incoterms 2020",
                subtitle: "Reglas de la Cámara de Comercio Internacional (CCI)",
                content: [
                    { type: 'heading', level: 2, text: "Introducción a los Términos Comerciales Internacionales" },
                    { type: 'paragraph', text: "Los Incoterms (International Commercial Terms) son un conjunto de reglas internacionales, publicadas por la CCI, que definen las responsabilidades del vendedor y del comprador en el contrato de compraventa para la entrega de mercancías." },
                    { type: 'paragraph', text: "Están diseñados para evitar malentendidos en los contratos de comercio internacional sobre los costes, riesgos y responsabilidades que implican la entrega de la mercancía." },
                    { type: 'button', text: "Ir al Índice General", target: "indice" }
                ],
                nav: []
            },
            indice: {
                title: "Índice General",
                subtitle: "Navegación por Capítulos",
                content: [
                    { type: 'index_item', text: "Sección 1: Reglas para cualquier modo de transporte", target: "seccion1" },
                    { type: 'index_item', text: "Sección 2: Reglas marítimas y vías navegables interiores", target: "seccion2" },
                    { type: 'index_item', text: "Sección 3: Aplicación práctica y ejemplos (FOB)", target: "seccion3" },
                    { type: 'index_item', text: "Búsqueda de Información en Línea", target: "search_section" },
                ],
                nav: [{ text: "Inicio", target: "inicio" }]
            },
            seccion1: {
                title: "Sección 1: Reglas Multimodales (Grupo E, F, C y D)",
                subtitle: "EXW, FCA, CPT, CIP, DAP, DPU, DDP",
                content: [
                    { type: 'heading', level: 3, text: "EXW (Ex Works / En Fábrica)" },
                    { type: 'paragraph', text: "El vendedor pone las mercancías a disposición del comprador en sus propias instalaciones. Es el término que implica la menor responsabilidad para el vendedor, que solo debe embalar y poner la mercancía lista para ser recogida." },
                    { type: 'list', items: [
                        "Menor riesgo para el vendedor.",
                        "El comprador asume todos los costes y riesgos desde la recogida.",
                        "No apto para pagos documentarios (cartas de crédito)."
                    ]},
                    { type: 'box', text: "Cuadro Informativo: El riesgo y la propiedad se transfieren en el almacén del vendedor. Se recomienda usar FCA si el vendedor carga la mercancía en el vehículo del comprador." },
                    { type: 'image_placeholder', text: "Diagrama: Transferencia de Riesgo en EXW", width: 400, height: 200 }
                ],
                nav: [
                    { text: "Índice", target: "indice" },
                    { text: "Siguiente", target: "seccion2" }
                ]
            },
            seccion2: {
                title: "Sección 2: Reglas Marítimas (Grupo F y C)",
                subtitle: "FAS, FOB, CFR, CIF",
                content: [
                    { type: 'heading', level: 3, text: "FOB (Free On Board / Franco a Bordo)" },
                    { type: 'paragraph', text: "El vendedor cumple con su obligación de entrega cuando la mercancía pasa la borda del buque en el puerto de embarque convenido. El riesgo de pérdida o daño de las mercancías se transmite cuando las mercancías están a bordo del buque, y el comprador asume todos los costos a partir de ese momento." },
                    { type: 'list', items: [
                        "Uso exclusivo para transporte marítimo o fluvial.",
                        "Requiere que la mercancía sea entregada en el buque.",
                        "Es uno de los términos más antiguos y comúnmente mal utilizados."
                    ]},
                    { type: 'paragraph', text: "El vendedor debe despachar las mercancías para la exportación. El comprador debe contratar el transporte principal y el seguro (aunque no es obligatorio)." },
                ],
                nav: [
                    { text: "Anterior", target: "seccion1" },
                    { text: "Índice", target: "indice" },
                    { text: "Siguiente", target: "seccion3" }
                ]
            },
            seccion3: {
                title: "Sección 3: Aplicación de DDP (Delivered Duty Paid)",
                subtitle: "Máxima responsabilidad para el vendedor",
                content: [
                    { type: 'heading', level: 3, text: "DDP (Entregada Derechos Pagados)" },
                    { type: 'paragraph', text: "DDP es el término que impone la máxima obligación al vendedor. Significa que el vendedor asume todos los riesgos y costes, incluidos los derechos, impuestos y cualquier otro cargo, hasta que las mercancías han sido entregadas en el lugar de destino convenido, listas para ser descargadas." },
                    { type: 'box', text: "Advertencia: El comprador no tiene ninguna obligación de importación o aduana. Esto puede ser riesgoso para el vendedor si no conoce bien las regulaciones del país de destino." },
                    { type: 'image_placeholder', text: "Ícono: Un barco llegando a puerto con una bandera de 'DDP'", width: 300, height: 150 },
                    { type: 'list', items: [
                        "Vendedor gestiona la importación.",
                        "Riesgo y coste hasta la entrega en destino.",
                        "Solo recomendado si el vendedor tiene presencia o agente en el país de destino."
                    ]},
                ],
                nav: [
                    { text: "Anterior", target: "seccion2" },
                    { text: "Índice", target: "indice" }
                ]
            },
            search_section: {
                title: "Búsqueda de Información",
                subtitle: "Consulta en línea sobre Incoterms",
                content: [
                    { type: 'heading', level: 2, text: "Obtener las Reglas Oficiales" },
                    { type: 'paragraph', text: "Puedes usar esta sección para buscar información en tiempo real sobre Incoterms y sus aplicaciones. Haré una consulta usando la API de Google Search para obtener datos actualizados sobre el tema." },
                    { type: 'button', text: "Buscar Reglas de Incoterms 2020 (en línea)", target: "search_rules" }
                ],
                nav: [{ text: "Índice", target: "indice" }]
            },
            search_results: {
                title: "Resultados de la Búsqueda",
                subtitle: "Datos en Tiempo Real",
                content: [
                    { type: 'paragraph', text: "Aquí se mostrarán los resultados de la búsqueda una vez finalice la consulta a la API de Google." }
                ],
                nav: [{ text: "Índice", target: "indice" }]
            }
        };

        // =================================================================================
        // 💻 CLASE PRINCIPAL DEL RENDERIZADO Y LÓGICA (CanvasRenderer)
        // =================================================================================

        class ManualApp {
            constructor() {
                this.resizeCanvas();
                window.addEventListener('resize', () => this.resizeCanvas());
                canvas.addEventListener('click', (e) => this.handleClick(e));
                
                // Eventos para el panel de control (fuera del canvas)
                document.getElementById('darkModeToggle').addEventListener('click', () => this.toggleDarkMode());
                document.getElementById('textZoomIn').addEventListener('click', () => this.zoomText(0.1));
                document.getElementById('textZoomOut').addEventListener('click', () => this.zoomText(-0.1));

                this.initializeAuthAndStart();
            }

            // Inicializa Firebase Auth (aunque no se use Firestore en este ejemplo, es la práctica estándar)
            async initializeAuthAndStart() {
                // No se necesita Firebase para esta aplicación, pero mantenemos la estructura
                this.render();
            }

            // --- Utilidades de Escalado y Responsividad ---

            // Ajusta el tamaño del canvas al contenedor y recalcula el factor de escala
            resizeCanvas() {
                const container = document.getElementById('canvasContainer');
                const ratio = BASE_WIDTH / BASE_HEIGHT;
                
                let w = container.clientWidth - 32; // -32px por el padding
                let h = container.clientHeight - 32;

                // Mantener el aspect ratio (vertical, tipo libro)
                if (w / h > ratio) {
                    w = h * ratio;
                } else {
                    h = w / ratio;
                }

                canvas.width = w;
                canvas.height = h;

                // Recalcular el factor de escala
                scaleFactor = canvas.width / BASE_WIDTH;
                this.render();
            }

            // Convierte una coordenada base (BASE_WIDTH/HEIGHT) a una coordenada real del canvas
            s(value) {
                return value * scaleFactor;
            }

            // Convierte un valor de fuente base a un tamaño real
            sf(value) {
                return this.s(value) * fontSizeScale;
            }

            // --- Lógica de Diseño y Dibujo ---

            // Dibuja un rectángulo con bordes redondeados (utilidad central)
            drawRoundedRect(x, y, w, h, radius, fillColor, strokeColor = null, shadow = true) {
                const r = this.s(radius);
                ctx.beginPath();
                ctx.moveTo(this.s(x) + r, this.s(y));
                ctx.arcTo(this.s(x) + this.s(w), this.s(y), this.s(x) + this.s(w), this.s(y) + this.s(h), r);
                ctx.arcTo(this.s(x) + this.s(w), this.s(y) + this.s(h), this.s(x), this.s(y) + this.s(h), r);
                ctx.arcTo(this.s(x), this.s(y) + this.s(h), this.s(x), this.s(y), r);
                ctx.arcTo(this.s(x), this.s(y), this.s(x) + this.s(w), this.s(y), r);
                ctx.closePath();

                if (shadow) {
                    const colors = getCurrentColors();
                    ctx.shadowColor = colors.shadow;
                    ctx.shadowBlur = this.s(8);
                    ctx.shadowOffsetX = this.s(4);
                    ctx.shadowOffsetY = this.s(4);
                }

                ctx.fillStyle = fillColor;
                ctx.fill();
                
                // Desactivar la sombra para el stroke y contenido posterior
                ctx.shadowBlur = 0;
                ctx.shadowOffsetX = 0;
                ctx.shadowOffsetY = 0;

                if (strokeColor) {
                    ctx.strokeStyle = strokeColor;
                    ctx.lineWidth = this.s(2);
                    ctx.stroke();
                }
            }

            // Dibuja texto con ajuste de línea (line wrapping)
            drawWrappedText(text, x, y, maxWidth, lineHeight, fontStyle, color) {
                ctx.font = fontStyle;
                ctx.fillStyle = color;
                const words = text.split(' ');
                let line = '';
                let yPos = this.s(y);
                const maxW = this.s(maxWidth);

                // Función para añadir una línea
                const addLine = (currentLine, currentY) => {
                    ctx.fillText(currentLine, this.s(x), currentY);
                    return currentY + this.s(lineHeight);
                };

                for (let n = 0; n < words.length; n++) {
                    let testLine = line + words[n] + ' ';
                    let metrics = ctx.measureText(testLine);
                    let testWidth = metrics.width;

                    if (testWidth > maxW && n > 0) {
                        yPos = addLine(line, yPos);
                        line = words[n] + ' ';
                    } else {
                        line = testLine;
                    }
                }
                yPos = addLine(line, yPos);
                return yPos;
            }

            // --- Renderizado de Elementos de la Sección ---

            // Renderiza el contenido de la sección actual
            renderSectionContent(contentData, startY) {
                const colors = getCurrentColors();
                let yPos = startY;
                const contentX = 100;
                const contentW = BASE_WIDTH - 200;
                const lineHeight = 30;

                contentData.forEach(item => {
                    // Padding y espaciado inicial
                    yPos += this.s(20);

                    switch (item.type) {
                        case 'heading':
                            const size = item.level === 2 ? 32 : 24;
                            const font = `${this.sf(size)}px 'Inter', sans-serif`;
                            
                            ctx.font = `bold ${font}`;
                            ctx.fillStyle = item.level === 2 ? colors.primary : colors.text;
                            ctx.fillText(item.text, this.s(contentX), yPos);
                            
                            yPos += this.s(10); // Espacio después del encabezado
                            break;

                        case 'paragraph':
                            const pFont = `${this.sf(18)}px 'Inter', sans-serif`;
                            yPos = this.drawWrappedText(item.text, contentX, yPos, contentW, lineHeight, pFont, colors.textSoft);
                            break;
                        
                        case 'list':
                            const listFont = `${this.sf(18)}px 'Inter', sans-serif`;
                            item.items.forEach(listItem => {
                                // Dibujar viñeta (círculo simple)
                                ctx.fillStyle = colors.secondary;
                                ctx.beginPath();
                                ctx.arc(this.s(contentX - 10), yPos - this.s(4), this.s(4), 0, Math.PI * 2);
                                ctx.fill();
                                // Dibujar texto de la lista
                                yPos = this.drawWrappedText(listItem, contentX, yPos, contentW, lineHeight, listFont, colors.text);
                            });
                            break;

                        case 'box':
                            const boxHeight = 100;
                            const boxY = yPos - this.s(10);
                            
                            // Dibuja el cuadro con sombra suave
                            this.drawRoundedRect(contentX - 10, boxY, contentW + 20, boxHeight, 10, colors.boxBackground, colors.secondary);
                            
                            const boxFont = `${this.sf(18)}px 'Inter', sans-serif`;
                            const boxTextX = contentX + 5;
                            const boxTextY = boxY + 35;
                            
                            // Dibuja el texto dentro del cuadro
                            ctx.fillStyle = colors.text;
                            this.drawWrappedText(item.text, boxTextX, boxTextY, contentW, 25, boxFont, colors.text);
                            
                            yPos = boxY + this.s(boxHeight) + this.s(10);
                            break;
                        
                        case 'image_placeholder':
                            const imgW = item.width || 400;
                            const imgH = item.height || 200;
                            const imgX = contentX + (contentW - imgW) / 2;

                            // Dibuja el marco del placeholder
                            this.drawRoundedRect(imgX, yPos, imgW, imgH, 10, '#eee', colors.textSoft, true);

                            // Dibuja el texto central
                            ctx.fillStyle = colors.textSoft;
                            ctx.font = `${this.sf(20)}px 'Inter', sans-serif`;
                            ctx.textAlign = 'center';
                            ctx.fillText(item.text, this.s(imgX + imgW / 2), this.s(yPos + imgH / 2 + 5));
                            ctx.textAlign = 'left'; // Restaurar
                            
                            yPos += this.s(imgH) + this.s(10);
                            break;

                        case 'button':
                            const btnW = 300;
                            const btnH = 50;
                            const btnX = contentX + (contentW - btnW) / 2;
                            const btnY = yPos;
                            item.bounds = { x: btnX, y: btnY, w: btnW, h: btnH }; // Almacenar límites
                            
                            // Dibuja el botón
                            this.drawRoundedRect(btnX, btnY, btnW, btnH, 10, colors.primary, null, true);
                            
                            // Dibuja el texto del botón
                            ctx.fillStyle = 'white';
                            ctx.font = `bold ${this.sf(20)}px 'Inter', sans-serif`;
                            ctx.textAlign = 'center';
                            ctx.fillText(item.text, this.s(btnX + btnW / 2), this.s(btnY + btnH / 2 + 7));
                            ctx.textAlign = 'left'; // Restaurar
                            
                            yPos += this.s(btnH);
                            break;

                        case 'index_item':
                            const indexW = contentW + 20;
                            const indexH = 60;
                            const indexX = contentX - 10;
                            const indexY = yPos;
                            item.bounds = { x: indexX, y: indexY, w: indexW, h: indexH }; // Almacenar límites
                            
                            // Dibuja el cuadro del índice
                            this.drawRoundedRect(indexX, indexY, indexW, indexH, 8, colors.boxBackground, colors.textSoft);
                            
                            // Dibuja el texto
                            ctx.fillStyle = colors.text;
                            ctx.font = `${this.sf(18)}px 'Inter', sans-serif`;
                            ctx.fillText(item.text, this.s(indexX + 20), this.s(indexY + indexH / 2 + 7));
                            
                            // Dibuja un ícono de flecha (>)
                            ctx.fillStyle = colors.primary;
                            ctx.font = `${this.sf(24)}px sans-serif`;
                            ctx.textAlign = 'right';
                            ctx.fillText('›', this.s(indexX + indexW - 10), this.s(indexY + indexH / 2 + 7));
                            ctx.textAlign = 'left';
                            
                            yPos += this.s(indexH);
                            break;
                    }
                    // Espacio entre elementos
                    yPos += this.s(15);
                });
                return yPos;
            }

            // Dibuja la barra de navegación inferior
            renderNavBar(navItems, currentY) {
                const colors = getCurrentColors();
                const navH = 80;
                const navY = BASE_HEIGHT - navH;

                // Fondo de la barra de navegación (semi-transparente y elevado)
                this.drawRoundedRect(0, navY, BASE_WIDTH, navH, 0, colors.background, null, true);

                const itemW = BASE_WIDTH / navItems.length;
                
                navItems.forEach((item, index) => {
                    const btnW = itemW - 40;
                    const btnH = 40;
                    const btnX = 20 + index * itemW + (itemW - btnW) / 2 - 20;
                    const btnY = navY + 20;
                    item.bounds = { x: btnX, y: btnY, w: btnW, h: btnH }; // Almacenar límites

                    // Dibuja el botón de navegación
                    this.drawRoundedRect(btnX, btnY, btnW, btnH, 8, colors.primary, null, true);
                    
                    // Dibuja el texto
                    ctx.fillStyle = 'white';
                    ctx.font = `bold ${this.sf(18)}px 'Inter', sans-serif`;
                    ctx.textAlign = 'center';
                    ctx.fillText(item.text, this.s(btnX + btnW / 2), this.s(btnY + btnH / 2 + 6));
                    ctx.textAlign = 'left';
                });
            }

            // --- Lógica de Navegación y Transición ---

            // Inicia la transición a una nueva sección
            goToSection(newScreenId) {
                if (isTransitioning || !MANUAL_CONTENT[newScreenId]) return;
                
                // Si es la búsqueda en línea, manejamos la API
                if (newScreenId === 'search_rules') {
                    this.searchOnline();
                    return;
                }

                currentScreenId = newScreenId;
                isTransitioning = true;
                transitionProgress = 0;
                this.animateTransition();
            }

            // Animación simple de fade-in
            animateTransition() {
                if (transitionProgress < 1) {
                    transitionProgress += 0.05; // Velocidad de la transición
                    this.render();
                    requestAnimationFrame(() => this.animateTransition());
                } else {
                    isTransitioning = false;
                    transitionProgress = 1;
                    this.render();
                }
            }

            // Lógica para manejar clics en el canvas
            handleClick(event) {
                const rect = canvas.getBoundingClientRect();
                // Calcular las coordenadas del clic en el sistema de coordenadas base (0-BASE_WIDTH, 0-BASE_HEIGHT)
                const clickX = (event.clientX - rect.left) / scaleFactor;
                const clickY = (event.clientY - rect.top) / scaleFactor;
                
                const currentContent = MANUAL_CONTENT[currentScreenId];

                // 1. Revisar clics en el contenido (botones, items de índice)
                currentContent.content.forEach(item => {
                    if (item.bounds) {
                        const { x, y, w, h, target } = item.bounds;
                        if (clickX >= x && clickX <= x + w && clickY >= y && clickY <= y + h) {
                            this.goToSection(target);
                            return;
                        }
                    }
                });

                // 2. Revisar clics en la barra de navegación inferior
                currentContent.nav.forEach(item => {
                    if (item.bounds) {
                        const { x, y, w, h, target } = item.bounds;
                        if (clickX >= x && clickX <= x + w && clickY >= y && clickY <= y + h) {
                            this.goToSection(target);
                            return;
                        }
                    }
                });
            }

            // --- Funcionalidades Opcionales ---

            // Alternar modo claro/oscuro
            toggleDarkMode() {
                isDarkMode = !isDarkMode;
                document.body.classList.toggle('dark-mode', isDarkMode);
                document.getElementById('darkModeToggle').textContent = isDarkMode ? '☀️ Modo Claro' : '🌙 Modo Oscuro';
                this.render();
            }

            // Ajustar el zoom del texto
            zoomText(delta) {
                fontSizeScale = Math.max(0.8, Math.min(1.5, fontSizeScale + delta)); // Limitar el zoom
                this.render();
            }

            // Búsqueda en línea usando Google Search API (simulado, ya que es un fetch)
            async searchOnline() {
                loadingIndicator.style.display = 'block';
                const userQuery = "Resumen de las reglas Incoterms 2020";
                const systemPrompt = "Actúa como un experto en comercio internacional y derecho mercantil. Proporciona un resumen conciso y bien estructurado (máximo 4 párrafos) de las diferencias clave entre los grupos de Incoterms (E, F, C, D) en la versión 2020.";
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;

                // Implementación de Backoff para reintentos
                const fetchWithRetry = async (url, options, maxRetries = 5) => {
                    for (let i = 0; i < maxRetries; i++) {
                        try {
                            const response = await fetch(url, options);
                            if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
                            return response;
                        } catch (error) {
                            if (i < maxRetries - 1) {
                                const delay = Math.pow(2, i) * 1000;
                                await new Promise(resolve => setTimeout(resolve, delay));
                            } else {
                                throw error;
                            }
                        }
                    }
                };

                const payload = {
                    contents: [{ parts: [{ text: userQuery }] }],
                    tools: [{ "google_search": {} }],
                    systemInstruction: { parts: [{ text: systemPrompt }] },
                };

                try {
                    const response = await fetchWithRetry(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });
                    
                    const result = await response.json();
                    const candidate = result.candidates?.[0];
                    let generatedText = "No se pudo obtener una respuesta.";
                    
                    if (candidate && candidate.content?.parts?.[0]?.text) {
                        generatedText = candidate.content.parts[0].text;
                    }

                    // Procesar la respuesta para el Canvas
                    const paragraphs = generatedText.split('\n\n').filter(p => p.trim().length > 0);
                    MANUAL_CONTENT.search_results.content = paragraphs.map(p => ({ type: 'paragraph', text: p }));
                    
                    if (MANUAL_CONTENT.search_results.content.length === 0) {
                        MANUAL_CONTENT.search_results.content.push({ type: 'paragraph', text: "La búsqueda retornó un resultado vacío o no válido." });
                    }

                } catch (error) {
                    console.error("Error al llamar a la API de Gemini:", error);
                    MANUAL_CONTENT.search_results.content = [{ type: 'paragraph', text: "Error en la conexión o la API. Inténtalo de nuevo." }];
                } finally {
                    loadingIndicator.style.display = 'none';
                    this.goToSection('search_results');
                }
            }


            // --- Función de Dibujo Principal ---

            render() {
                const colors = getCurrentColors();
                const content = MANUAL_CONTENT[currentScreenId];

                // Limpiar el canvas y establecer el fondo
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                
                // Dibujar el fondo del "manual" (blanco o gris oscuro)
                this.drawRoundedRect(0, 0, BASE_WIDTH, BASE_HEIGHT, 12, colors.background);
                
                // Aplicar la transición de fade-in
                if (isTransitioning) {
                    ctx.globalAlpha = transitionProgress;
                } else {
                    ctx.globalAlpha = 1.0;
                }

                // --- Cabecera ---
                const headerY = 100;

                // Título principal
                ctx.fillStyle = colors.primary;
                ctx.font = `bold ${this.sf(48)}px 'Inter', sans-serif`;
                ctx.textAlign = 'center';
                ctx.fillText(content.title, this.s(BASE_WIDTH / 2), this.s(headerY));
                
                // Subtítulo
                ctx.fillStyle = colors.textSoft;
                ctx.font = `${this.sf(22)}px 'Inter', sans-serif`;
                ctx.fillText(content.subtitle, this.s(BASE_WIDTH / 2), this.s(headerY + 40));
                ctx.textAlign = 'left'; // Restaurar

                // --- Contenido ---
                const contentStartY = headerY + 80;
                // Renderizar el contenido y obtener la posición Y final
                this.renderSectionContent(content.content, contentStartY);

                // --- Navegación ---
                if (content.nav.length > 0) {
                    this.renderNavBar(content.nav, BASE_HEIGHT - 80);
                }

                // Restaurar alpha
                ctx.globalAlpha = 1.0;
            }
        }

        // Inicializar la aplicación al cargar el script
        window.manualApp = new ManualApp();
        
    </script>
</body>
</html>
