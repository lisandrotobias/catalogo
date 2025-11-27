<!-- ============================================================================
     COMPONENTE CATALOG.SVELTE
     ============================================================================
     
     Componente principal que implementa un catálogo digital interactivo con
     funcionalidades avanzadas de visualización y navegación.
     
     FUNCIONALIDADES PRINCIPALES:
     - Visualización tipo flipbook con animaciones realistas (PageFlip.js)
     - Sistema de zoom (50% a 300%) con múltiples métodos de control
     - Arrastre del contenido cuando hay zoom activo
     - Soporte completo para gestos táctiles (pinch zoom, arrastre)
     - Diseño responsive que se adapta a móvil y desktop
     - Navegación de páginas con botones y clicks directos
     
     TECNOLOGÍAS:
     - Svelte 5.x (framework)
     - PageFlip.js 2.0.7 (biblioteca de flipbook, cargada desde CDN)
     - FontAwesome (iconos, cargados desde CDN)
     
     CASOS DE USO DOCUMENTADOS:
     Ver comentarios en el template HTML para detalles de cada caso de uso.
     ============================================================================ -->

<script>
    import { onMount } from 'svelte';
    import Tutorial from './Tutorial.svelte';
    
    // ============================================================================
    // VARIABLES DE ESTADO DEL COMPONENTE
    // ============================================================================
    
    /**
     * Referencia al elemento DOM del contenedor del libro (div#book).
     * Se usa para inicializar PageFlip y cargar las páginas desde HTML.
     */
    let book;
    
    /**
     * Instancia de PageFlip que controla la animación de volteo de páginas.
     * Se inicializa después de cargar el script desde CDN.
     * Permite navegar entre páginas con animaciones realistas.
     */
    let pageFlip;
    
    /**
     * Indica si el dispositivo actual es móvil (ancho < 768px).
     * Determina la configuración de PageFlip y el comportamiento responsive.
     * Se actualiza en onMount y cuando cambia el tamaño de la ventana.
     */
    let isMobile = false;
    
    /**
     * Nivel de zoom actual del catálogo.
     * Rango: 0.5 (50%) a 3 (300%).
     * Valor inicial: 1 (100%, sin zoom).
     * Se actualiza mediante controles de zoom, rueda del mouse o gestos táctiles.
     */
    let zoomLevel = 1;
    
    /**
     * Referencia al contenedor del libro con zoom (div.book-wrapper).
     * Se usa para aplicar transformaciones CSS (translate y scale) cuando hay zoom.
     */
    let bookContainer;
    
    // ============================================================================
    // VARIABLES DE CONTROL DE ARRASTRE CON MOUSE
    // ============================================================================
    
    /**
     * Indica si el usuario está arrastrando el catálogo con el mouse.
     * Solo se activa cuando zoomLevel > 1 para permitir navegar por el contenido ampliado.
     */
    let isDragging = false;
    
    /**
     * Indica si el botón del mouse está presionado.
     * Se usa para saber si el usuario está intentando arrastrar.
     */
    let isMouseDown = false;
    
    /**
     * Posición inicial del mouse cuando comienza el arrastre.
     * Se usa para calcular el desplazamiento relativo del catálogo.
     * Formato: { x: número, y: número } en píxeles.
     */
    let dragStart = { x: 0, y: 0 };
    
    /**
     * Posición actual del catálogo cuando está con zoom.
     * Se actualiza durante el arrastre para mover el contenido ampliado.
     * Formato: { x: número, y: número } en píxeles.
     * Valor inicial: { x: 0, y: 0 } (centrado).
     */
    let bookPosition = { x: 0, y: 0 };
    
    /**
     * Última posición conocida del mouse.
     * Se usa para hacer zoom hacia el cursor cuando se usan los botones de zoom.
     * Formato: { x: número, y: número } en píxeles.
     */
    let lastMousePosition = { x: 0, y: 0 };
    
    /**
     * Página actual del catálogo (1-indexed).
     * Se actualiza cuando PageFlip dispara el evento 'flip'.
     * Valor inicial: 0 (antes de inicializar PageFlip).
     */
    let currentPage = 0;
    
    /**
     * Número total de páginas del catálogo.
     * Se calcula contando los elementos con clase "my-page".
     */
    const totalPages = 20;
    
    /**
     * Ruta al archivo PDF del catálogo.
     * El archivo debe estar en la carpeta static del proyecto.
     */
    const pdfPath = '/catalogo.pdf';
    
    /**
     * Indica si se debe mostrar el tutorial inicial.
     * Se controla mediante localStorage para mostrar solo la primera vez.
     */
    let showTutorial = false;
    
    // ============================================================================
    // FUNCIONES DE DETECCIÓN Y CONFIGURACIÓN
    // ============================================================================
    
    /**
     * Detecta si el dispositivo actual es móvil basándose en el ancho de la ventana.
     * 
     * CASO DE USO: Se ejecuta en onMount y cuando cambia el tamaño de la ventana.
     * 
     * Lógica:
     * - Si window.innerWidth < 768px → isMobile = true (modo móvil)
     * - Si window.innerWidth >= 768px → isMobile = false (modo desktop)
     * 
     * @returns {boolean} true si es móvil, false si es desktop, false si window no está disponible
     */
    function checkMobile() {
        // Verificar que window esté disponible (SSR safety)
        if (typeof window !== 'undefined') {
            isMobile = window.innerWidth < 768;
            return isMobile;
        }
        return false;
    }
    
    /**
     * Hook de ciclo de vida de Svelte que se ejecuta cuando el componente se monta en el DOM.
     * 
     * CASO DE USO: Inicialización completa del catálogo interactivo.
     * 
     * Flujo de ejecución:
     * 1. Detecta si es móvil o desktop
     * 2. Carga dinámicamente el script de PageFlip desde CDN
     * 3. Configura PageFlip según el tipo de dispositivo
     * 4. Inicializa la instancia de PageFlip
     * 5. Carga las páginas desde el HTML
     * 6. Registra event listeners para interacciones
     * 7. Retorna función de limpieza para desmontar listeners
     */
    onMount(() => {
        // PASO 0: Verificar si se debe mostrar el tutorial
        // Solo mostrar si no se ha visto antes (no existe en localStorage)
        if (typeof window !== 'undefined') {
            const tutorialVisto = localStorage.getItem('catalogo-tutorial-visto');
            if (!tutorialVisto || tutorialVisto !== 'true') {
                showTutorial = true;
            }
        }
        
        // PASO 1: Detectar inicialmente si es mobile
        checkMobile();
        
        // PASO 2: Cargar el script de PageFlip dinámicamente desde CDN
        // Esto permite que el componente funcione sin incluir PageFlip en el bundle inicial
        const script = document.createElement('script');
        script.src = 'https://unpkg.com/page-flip@2.0.7/dist/js/page-flip.browser.js';
        
        // PASO 3: Cuando el script se carga, inicializar PageFlip
        script.onload = () => {
            // Verificar que PageFlip se cargó correctamente
            if (!window.St || !window.St.PageFlip) {
                console.error('PageFlip no se cargó correctamente');
                return;
            }
            
            // ========================================================================
            // CONFIGURACIÓN DE PAGEFLIP
            // ========================================================================
            // 
            // CASO 1: CONFIGURACIÓN PARA MÓVIL (isMobile = true)
            // - width: 340px, height: 480px (proporción 3:4 optimizada para catálogos)
            // - minWidth: 300px, maxWidth: 400px (rango flexible)
            // - minHeight: 420px, maxHeight: 540px
            // - flippingTime: 1000ms (más rápido para móvil)
            // - usePortrait: true (modo vertical)
            // 
            // CASO 2: CONFIGURACIÓN PARA DESKTOP (isMobile = false)
            // - width: 650px, height: 900px (tamaño más grande)
            // - minWidth: 550px, maxWidth: 750px
            // - minHeight: 780px, maxHeight: 1000px
            // - flippingTime: 1200ms (animación más lenta y suave)
            // - usePortrait: false (modo horizontal/spread)
            //
            // Parámetros comunes:
            // - size: "fixed" → tamaño fijo (no se ajusta automáticamente)
            // - drawShadow: true → dibuja sombras realistas en las páginas
            // - autoSize: true → ajusta automáticamente dentro de los límites
            // - maxShadowOpacity: 0.4 → opacidad máxima de las sombras (40%)
            // - showCover: true → muestra la portada al inicio
            // - mobileScrollSupport: true → soporte para scroll en móviles
            // - startZIndex: 0 → z-index inicial de las páginas
            const config = {
                width: isMobile ? 340 : 650,  // Ancho base según dispositivo
                height: isMobile ? 480 : 900,  // Alto base según dispositivo
                size: "fixed",  // Tamaño fijo (no responsive automático)
                minWidth: isMobile ? 300 : 550,  // Ancho mínimo permitido
                maxWidth: isMobile ? 400 : 750,  // Ancho máximo permitido
                minHeight: isMobile ? 420 : 780,  // Alto mínimo permitido
                maxHeight: isMobile ? 540 : 1000,  // Alto máximo permitido
                drawShadow: true,  // Dibujar sombras realistas en las páginas
                flippingTime: isMobile ? 1000 : 1200,  // Duración de la animación de volteo (ms)
                usePortrait: isMobile,  // Modo vertical en móvil, horizontal en desktop
                startZIndex: 0,  // Z-index inicial para las páginas
                autoSize: true,  // Ajustar automáticamente dentro de los límites
                maxShadowOpacity: 0.4,  // Opacidad máxima de las sombras (0-1)
                showCover: true,  // Mostrar la portada al cargar
                mobileScrollSupport: true  // Permitir scroll en dispositivos móviles
            };
            
            console.log("Modo:", isMobile ? "móvil" : "desktop");
            console.log("Configuración catálogo:", config);
            
            // PASO 4: Inicializar PageFlip con el elemento del libro y la configuración
            pageFlip = new window.St.PageFlip(book, config);
            
            // PASO 5: Cargar las páginas desde los elementos HTML con clase "my-page"
            // Esto convierte los div.my-page en páginas del flipbook
            pageFlip.loadFromHTML(document.querySelectorAll(".my-page"));
            
            // PASO 6: Registrar listener para cambios de tamaño de ventana
            window.addEventListener('resize', handleResize);
            
            // ========================================================================
            // EVENTOS DE PAGEFLIP
            // ========================================================================
            // 
            // CASO DE USO: Monitorear cambios de página y orientación
            // 
            // Evento 'flip': Se dispara cada vez que el usuario cambia de página
            // - e.data contiene el número de página actual
            // - Útil para tracking, analytics o actualizar UI
            pageFlip.on('flip', (e) => {
                currentPage = e.data;
                console.log("Página actual: " + e.data);
            });
            
            // Inicializar la página actual después de cargar
            // PageFlip puede usar diferentes métodos según la versión
            try {
                if (typeof pageFlip.getCurrentPageIndex === 'function') {
                    currentPage = pageFlip.getCurrentPageIndex() + 1; // PageFlip usa 0-indexed, nosotros 1-indexed
                } else if (typeof pageFlip.getPageCollection === 'function') {
                    const pages = pageFlip.getPageCollection();
                    if (pages && pages.length > 0) {
                        currentPage = 1; // Empezar en la primera página
                    }
                } else {
                    currentPage = 1; // Valor por defecto
                }
            } catch (e) {
                currentPage = 1; // Valor por defecto si hay error
            }
            
            // Evento 'changeOrientation': Se dispara cuando cambia la orientación del dispositivo
            // - e.data contiene la nueva orientación
            // - Útil para ajustar la UI cuando el usuario rota el dispositivo
            pageFlip.on('changeOrientation', (e) => {
                console.log("Orientación cambiada: " + e.data);
            });
        };
        
        // Agregar el script al head del documento
        document.head.appendChild(script);
        
        // PASO 7: Registrar event listeners globales para interacciones
        // Estos listeners se aplican a toda la ventana para capturar eventos
        // incluso cuando el cursor está fuera del catálogo
        
        // Listener para zoom con rueda del mouse (Ctrl/Cmd + rueda)
        // passive: false permite preventDefault() para evitar scroll del navegador
        window.addEventListener('wheel', handleWheel, { passive: false });
        
        // Listener para arrastre con mouse (cuando hay zoom)
        window.addEventListener('mousemove', handleMouseMove);
        window.addEventListener('mouseup', handleMouseUp);
        
        // PASO 8: Retornar función de limpieza
        // Esta función se ejecuta cuando el componente se desmonta
        // Es importante para evitar memory leaks y eventos huérfanos
        return () => {
            // Remover todos los event listeners registrados
            window.removeEventListener('resize', handleResize);
            window.removeEventListener('wheel', handleWheel);
            window.removeEventListener('mousemove', handleMouseMove);
            window.removeEventListener('mouseup', handleMouseUp);
            
            // Destruir la instancia de PageFlip si existe
            // Esto libera recursos y limpia animaciones
            if (pageFlip) {
                pageFlip.destroy();
            }
        };
    });
    
    /**
     * Maneja los cambios de tamaño de la ventana del navegador.
     * 
     * CASO DE USO: Cuando el usuario redimensiona la ventana o rota el dispositivo.
     * 
     * Lógica:
     * 1. Guarda el estado anterior de isMobile
     * 2. Detecta el nuevo estado (móvil o desktop)
     * 3. Si cambió entre móvil y desktop, recarga la página
     * 
     * ¿Por qué recargar?
     * - PageFlip necesita reinicializarse con una configuración diferente
     * - La recarga es la forma más simple y confiable de aplicar los cambios
     * - Alternativa sería destruir y recrear PageFlip, pero es más complejo
     */
    function handleResize() {
        const wasMobile = isMobile;
        checkMobile();
        
        // Si cambió entre mobile y desktop, reinicializar con la configuración correcta
        if (wasMobile !== isMobile && pageFlip) {
            console.log("Cambio de modo a:", isMobile ? "móvil" : "desktop");
            // La forma más simple de manejar el cambio es recargar la página
            // Esto garantiza que PageFlip se inicialice con la configuración correcta
            location.reload();
        }
    }
    
    // ============================================================================
    // FUNCIONES DE NAVEGACIÓN DE PÁGINAS
    // ============================================================================
    
    /**
     * Navega a la página anterior del catálogo.
     * 
     * CASO DE USO: Cuando el usuario hace clic en el botón "Página anterior".
     * 
     * Lógica:
     * - En desktop: solo funciona si no hay zoom activo (zoomLevel <= 1)
     * - En móvil: siempre permite cambiar de página, incluso con zoom activo
     * - Llama a pageFlip.flipPrev('top') para voltear hacia atrás
     * - 'top' indica que la animación comienza desde la parte superior
     * - Solo funciona si pageFlip está inicializado
     */
    function prevPage() {
        // En desktop, no cambiar de página si hay zoom activo
        // En móvil, siempre permitir cambiar de página
        if (!isMobile && zoomLevel > 1) return;
        if (pageFlip) pageFlip.flipPrev('top');
    }
    
    /**
     * Navega a la página siguiente del catálogo.
     * 
     * CASO DE USO: Cuando el usuario hace clic en el botón "Página siguiente".
     * 
     * Lógica:
     * - En desktop: solo funciona si no hay zoom activo (zoomLevel <= 1)
     * - En móvil: siempre permite cambiar de página, incluso con zoom activo
     * - Llama a pageFlip.flipNext('bottom') para voltear hacia adelante
     * - 'bottom' indica que la animación comienza desde la parte inferior
     * - Solo funciona si pageFlip está inicializado
     */
    function nextPage() {
        // En desktop, no cambiar de página si hay zoom activo
        // En móvil, siempre permitir cambiar de página
        if (!isMobile && zoomLevel > 1) return;
        if (pageFlip) pageFlip.flipNext('bottom');
    }
    
    // ============================================================================
    // FUNCIONES DE ZOOM
    // ============================================================================
    
    /**
     * Calcula el zoom hacia un punto específico (normalmente el cursor del mouse).
     * 
     * @param {number} oldZoom - Nivel de zoom actual
     * @param {number} newZoom - Nuevo nivel de zoom
     * @param {number} clientX - Posición X del mouse en coordenadas de la ventana (clientX)
     * @param {number} clientY - Posición Y del mouse en coordenadas de la ventana (clientY)
     * @returns {Object} Nueva posición del catálogo { x, y }
     */
    function zoomTowardsPoint(oldZoom, newZoom, clientX, clientY) {
        if (!bookContainer) {
            return { x: 0, y: 0 };
        }
        
        // Obtener las dimensiones y posición del contenedor
        const rect = bookContainer.getBoundingClientRect();
        const containerCenterX = rect.left + rect.width / 2;
        const containerCenterY = rect.top + rect.height / 2;
        
        // Calcular la posición del mouse relativa al centro del contenedor
        const relativeX = clientX - containerCenterX;
        const relativeY = clientY - containerCenterY;
        
        // Calcular el punto en el espacio del contenido antes del zoom
        // Este es el punto que queremos mantener bajo el cursor después del zoom
        const contentX = (relativeX - bookPosition.x) / oldZoom;
        const contentY = (relativeY - bookPosition.y) / oldZoom;
        
        // Calcular la nueva posición para mantener ese punto bajo el cursor después del zoom
        const newX = relativeX - contentX * newZoom;
        const newY = relativeY - contentY * newZoom;
        
        return { x: newX, y: newY };
    }
    
    /**
     * Aumenta el nivel de zoom del catálogo hacia el punto del cursor.
     * 
     * CASO DE USO 1: Usuario hace clic en el botón "+" de zoom.
     * CASO DE USO 2: Usuario usa Ctrl/Cmd + rueda del mouse hacia arriba.
     * CASO DE USO 3: Usuario hace pinch zoom (dos dedos separándose).
     * 
     * Lógica:
     * - Incrementa zoomLevel en 0.25 (25%) por cada llamada
     * - Límite máximo: 3 (300% de zoom)
     * - Si ya está en el máximo, no hace nada
     * - Hace zoom hacia el cursor del mouse (o centro si no hay posición conocida)
     * - Actualiza la transformación CSS del contenedor
     * 
     * @param {number} mouseX - Posición X del mouse (opcional, usa última posición conocida si no se proporciona)
     * @param {number} mouseY - Posición Y del mouse (opcional, usa última posición conocida si no se proporciona)
     */
    function zoomIn(mouseX, mouseY) {
        if (zoomLevel < 3) {
            const oldZoom = zoomLevel;
            zoomLevel += 0.25;
            
            // Usar posición del mouse si se proporciona, sino usar última posición conocida o centro
            const zoomX = mouseX !== undefined ? mouseX : lastMousePosition.x;
            const zoomY = mouseY !== undefined ? mouseY : lastMousePosition.y;
            
            // Si no hay posición conocida, hacer zoom hacia el centro
            if (zoomX === 0 && zoomY === 0 && bookContainer) {
                const rect = bookContainer.getBoundingClientRect();
                bookPosition = zoomTowardsPoint(oldZoom, zoomLevel, rect.left + rect.width / 2, rect.top + rect.height / 2);
            } else {
                bookPosition = zoomTowardsPoint(oldZoom, zoomLevel, zoomX, zoomY);
            }
            
            updateZoom();
        }
    }
    
    /**
     * Reduce el nivel de zoom del catálogo hacia el punto del cursor.
     * 
     * CASO DE USO 1: Usuario hace clic en el botón "-" de zoom.
     * CASO DE USO 2: Usuario usa Ctrl/Cmd + rueda del mouse hacia abajo.
     * CASO DE USO 3: Usuario hace pinch zoom (dos dedos acercándose).
     * 
     * Lógica:
     * - Decrementa zoomLevel en 0.25 (25%) por cada llamada
     * - Límite mínimo: 0.5 (50% de zoom)
     * - Si ya está en el mínimo, no hace nada
     * - Hace zoom hacia el cursor del mouse (o centro si no hay posición conocida)
     * - Actualiza la transformación CSS del contenedor
     * 
     * @param {number} mouseX - Posición X del mouse (opcional, usa última posición conocida si no se proporciona)
     * @param {number} mouseY - Posición Y del mouse (opcional, usa última posición conocida si no se proporciona)
     */
    function zoomOut(mouseX, mouseY) {
        if (zoomLevel > 0.5) {
            const oldZoom = zoomLevel;
            zoomLevel -= 0.25;
            
            // Usar posición del mouse si se proporciona, sino usar última posición conocida o centro
            const zoomX = mouseX !== undefined ? mouseX : lastMousePosition.x;
            const zoomY = mouseY !== undefined ? mouseY : lastMousePosition.y;
            
            // Si no hay posición conocida, hacer zoom hacia el centro
            if (zoomX === 0 && zoomY === 0 && bookContainer) {
                const rect = bookContainer.getBoundingClientRect();
                bookPosition = zoomTowardsPoint(oldZoom, zoomLevel, rect.left + rect.width / 2, rect.top + rect.height / 2);
            } else {
                bookPosition = zoomTowardsPoint(oldZoom, zoomLevel, zoomX, zoomY);
            }
            
            updateZoom();
        }
    }
    
    /**
     * Restablece el zoom al nivel inicial (100%) y centra el catálogo.
     * 
     * CASO DE USO: Usuario hace clic en el botón de resetear zoom.
     * 
     * Lógica:
     * - Establece zoomLevel a 1 (100%)
     * - Restablece bookPosition a { x: 0, y: 0 } (centrado)
     * - Actualiza la transformación CSS del contenedor
     */
    function resetZoom() {
        zoomLevel = 1;
        bookPosition = { x: 0, y: 0 };
        updateZoom();
    }
    
    /**
     * Descarga el archivo PDF del catálogo.
     * 
     * CASO DE USO: Usuario hace clic en el botón de descargar PDF.
     * 
     * Lógica:
     * - Crea un elemento <a> temporal con el atributo download
     * - Establece la ruta al PDF
     * - Simula un click para iniciar la descarga
     * - Elimina el elemento temporal después de un breve delay
     */
    function downloadPDF() {
        const link = document.createElement('a');
        link.href = pdfPath;
        link.download = 'catalogo.pdf';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    }
    
    /**
     * Cierra el tutorial y guarda en localStorage que ya se ha visto.
     * 
     * CASO DE USO: Usuario hace clic en "Entendido" o cierra el tutorial.
     * 
     * Lógica:
     * - Oculta el tutorial (showTutorial = false)
     * - Guarda en localStorage que el tutorial ya fue visto
     * - Esto evita que se muestre nuevamente en futuras visitas
     */
    function closeTutorial() {
        showTutorial = false;
        if (typeof window !== 'undefined') {
            localStorage.setItem('catalogo-tutorial-visto', 'true');
        }
    }
    
    /**
     * Aplica la transformación CSS de zoom y posición al contenedor del libro.
     * 
     * CASO DE USO: Se llama cada vez que cambia zoomLevel o bookPosition.
     * 
     * Lógica:
     * - Aplica transform: translate(x, y) scale(zoomLevel)
     * - translate: mueve el catálogo cuando está con zoom (para navegar)
     * - scale: aplica el nivel de zoom
     * - Solo funciona si bookContainer está disponible
     * - Deshabilita/habilita PageFlip según el nivel de zoom para evitar conflictos
     * 
     * @param {boolean} skipPageFlipUpdate - Si es true, no actualiza PageFlip (útil durante arrastre)
     */
    function updateZoom(skipPageFlipUpdate = false) {
        if (bookContainer) {
            bookContainer.style.transform = `translate(${bookPosition.x}px, ${bookPosition.y}px) scale(${zoomLevel})`;
        }
        
        // Solo actualizar PageFlip si no se está arrastrando (para evitar loops)
        if (!skipPageFlipUpdate && pageFlip && !isDragging && !isZooming) {
            // Deshabilitar PageFlip cuando hay zoom activo para evitar conflictos
            // Esto previene que PageFlip interprete los gestos como intentos de voltear página
            if (zoomLevel > 1) {
                // Deshabilitar interacción de PageFlip cuando hay zoom
                // Esto evita que se confundan los gestos de zoom/arrastre con volteo de páginas
                try {
                    // Intentar usar updateSettings si está disponible
                    if (typeof pageFlip.updateSettings === 'function') {
                        pageFlip.updateSettings({ 
                            disableFlipByClick: true,
                            swipeDistance: 0  // Deshabilitar swipe cuando hay zoom
                        });
                    } else {
                        // Alternativa: deshabilitar pointer-events en el elemento del libro
                        if (book) {
                            book.style.pointerEvents = 'none';
                        }
                    }
                } catch (e) {
                    // Si falla, usar CSS como alternativa
                    if (book) {
                        book.style.pointerEvents = 'none';
                    }
                }
            } else {
                // Rehabilitar interacción de PageFlip cuando no hay zoom
                try {
                    if (typeof pageFlip.updateSettings === 'function') {
                        pageFlip.updateSettings({ 
                            disableFlipByClick: false,
                            swipeDistance: 30  // Distancia normal para swipe
                        });
                    } else {
                        // Alternativa: rehabilitar pointer-events
                        if (book) {
                            book.style.pointerEvents = 'auto';
                        }
                    }
                } catch (e) {
                    // Si falla, usar CSS como alternativa
                    if (book) {
                        book.style.pointerEvents = 'auto';
                    }
                }
            }
        }
    }
    
    // ============================================================================
    // MANEJO DE EVENTOS DE ZOOM CON RUEDA DEL MOUSE
    // ============================================================================
    
    /**
     * Maneja el evento de rueda del mouse para controlar el zoom.
     * 
     * CASO DE USO: Usuario presiona Ctrl (Windows/Linux) o Cmd (Mac) + rueda del mouse.
     * 
     * Lógica:
     * 1. Verifica si Ctrl o Cmd están presionados
     * 2. Si están presionados:
     *    - Guarda la posición del mouse para hacer zoom hacia ese punto
     *    - Previene el comportamiento por defecto (scroll del navegador)
     *    - Si deltaY > 0 (rueda hacia abajo) → zoomOut() hacia el cursor
     *    - Si deltaY < 0 (rueda hacia arriba) → zoomIn() hacia el cursor
     * 3. Si no están presionados, no hace nada (permite scroll normal)
     * 
     * @param {WheelEvent} e - Evento de rueda del mouse
     */
    function handleWheel(e) {
        // Solo procesar si Ctrl (Windows/Linux) o Cmd (Mac) están presionados
        if (e.ctrlKey || e.metaKey) {
            // Prevenir el scroll del navegador cuando se hace zoom
            e.preventDefault();
            e.stopPropagation();
            
            // Guardar la posición del mouse para hacer zoom hacia ese punto
            lastMousePosition = { x: e.clientX, y: e.clientY };
            
            // deltaY > 0: rueda hacia abajo → reducir zoom
            // deltaY < 0: rueda hacia arriba → aumentar zoom
            if (e.deltaY > 0) {
                zoomOut(e.clientX, e.clientY);
            } else {
                zoomIn(e.clientX, e.clientY);
            }
        }
    }
    
    // ============================================================================
    // VARIABLES DE CONTROL DE ARRASTRE CON MOUSE
    // ============================================================================
    
    /**
     * Timeout para distinguir entre click y arrastre.
     * Se usa para evitar que un click simple active el arrastre.
     * Se cancela si el usuario mueve el mouse más del threshold.
     */
    let dragTimeout;
    
    /**
     * Umbral en píxeles para considerar que el usuario está arrastrando.
     * Si el mouse se mueve más de 15px desde la posición inicial, se activa el arrastre.
     * Aumentado a 15px para evitar activación accidental durante clicks o movimientos menores.
     * Esto permite distinguir mejor entre clicks y arrastres intencionales.
     */
    let dragThreshold = 15;
    
    /**
     * Posición inicial del mouse cuando se presiona el botón.
     * Se usa para calcular si el movimiento supera el threshold.
     * Formato: { x: número, y: número } en píxeles.
     */
    let startPosition = { x: 0, y: 0 };
    
    // ============================================================================
    // FUNCIONES DE ARRASTRE CON MOUSE
    // ============================================================================
    
    /**
     * Maneja el evento cuando el usuario presiona el botón del mouse.
     * 
     * CASO DE USO: Usuario presiona el mouse sobre el catálogo cuando hay zoom (zoomLevel > 1).
     * 
     * Lógica:
     * 1. Solo funciona si zoomLevel > 1 (hay zoom activo)
     * 2. NO previene eventos inmediatamente - permite que el usuario haga click normalmente
     * 3. Marca isMouseDown = true para indicar que el botón está presionado
     * 4. Guarda la posición inicial del mouse
     * 5. NO calcula dragStart todavía - se calculará cuando se active el arrastre
     * 
     * MEJORAS DE USABILIDAD:
     * - No previene eventos inmediatamente para permitir clicks normales
     * - Solo activa arrastre cuando el usuario realmente mueve el mouse
     * - El cursor permanece en "default" hasta que se detecta movimiento intencional
     * 
     * @param {MouseEvent} e - Evento de mouse down
     */
    function handleMouseDown(e) {
        // Solo permitir arrastre si hay zoom activo
        if (zoomLevel > 1) {
            // Marcar que el botón del mouse está presionado
            isMouseDown = true;
            
            // Guardar posición inicial del mouse
            startPosition = { x: e.clientX, y: e.clientY };
            
            // NO calcular dragStart todavía - se calculará cuando realmente se active el arrastre
            // Esto evita problemas de cálculo cuando el mouse se mueve antes de activar el arrastre
        }
    }
    
    /**
     * Maneja el movimiento del mouse durante el arrastre.
     * 
     * CASO DE USO: Usuario mueve el mouse mientras mantiene presionado el botón.
     * 
     * Lógica:
     * 1. Guarda la posición del mouse para usar en zoom hacia el cursor
     * 2. Solo funciona si zoomLevel > 1 Y el botón del mouse está presionado (isMouseDown)
     * 3. Calcula la distancia desde la posición inicial
     * 4. Si la distancia supera el threshold Y no está arrastrando aún:
     *    - Calcula dragStart basándose en la posición actual del mouse y bookPosition
     *    - Activa el arrastre inmediatamente
     *    - Previene eventos para evitar conflictos
     *    - Cambia el cursor a 'grabbing'
     * 5. Si ya está arrastrando:
     *    - Previene eventos por defecto de forma más agresiva (evita voltear página)
     *    - Actualiza bookPosition basándose en la posición del mouse
     *    - Aplica la transformación CSS
     * 
     * MEJORAS DE USABILIDAD:
     * - Guarda la posición del mouse para zoom hacia el cursor
     * - Solo procesa si el botón del mouse está presionado
     * - Calcula dragStart cuando realmente se activa el arrastre (no en mousedown)
     * - Solo activa arrastre cuando hay movimiento significativo (15px)
     * - Previene eventos solo cuando realmente está arrastrando
     * 
     * @param {MouseEvent} e - Evento de mouse move
     */
    function handleMouseMove(e) {
        // Guardar la posición del mouse para usar en zoom hacia el cursor
        lastMousePosition = { x: e.clientX, y: e.clientY };
        // Solo procesar si hay zoom Y el botón del mouse está presionado
        if (zoomLevel > 1 && isMouseDown) {
            // Calcular distancia desde la posición inicial
            const deltaX = Math.abs(e.clientX - startPosition.x);
            const deltaY = Math.abs(e.clientY - startPosition.y);
            
            // Si se mueve más del threshold, iniciar arrastre inmediatamente
            // Esto activa el arrastre solo cuando hay movimiento intencional
            if ((deltaX > dragThreshold || deltaY > dragThreshold) && !isDragging) {
                // Calcular dragStart AHORA, cuando realmente se activa el arrastre
                // Esto asegura que el cálculo sea correcto basándose en la posición actual
                dragStart = { 
                    x: e.clientX - bookPosition.x, 
                    y: e.clientY - bookPosition.y 
                };
                
                isDragging = true;
                // Prevenir eventos ahora que sabemos que es un arrastre intencional
                e.preventDefault();
                e.stopPropagation();
                e.stopImmediatePropagation();
                
                if (bookContainer) {
                    bookContainer.style.cursor = 'grabbing';
                }
            }
            
            // Si está arrastrando, actualizar la posición del catálogo
            if (isDragging) {
                // Prevenir que PageFlip interprete esto como un intento de voltear página
                // Usar stopImmediatePropagation para prevenir eventos en todos los listeners
                e.preventDefault();
                e.stopPropagation();
                e.stopImmediatePropagation();
                
                // Calcular nueva posición basándose en la posición del mouse y el offset inicial
                bookPosition = {
                    x: e.clientX - dragStart.x,
                    y: e.clientY - dragStart.y
                };
                
                // Aplicar la transformación CSS (sin actualizar PageFlip para evitar loops)
                if (bookContainer) {
                    bookContainer.style.transform = `translate(${bookPosition.x}px, ${bookPosition.y}px) scale(${zoomLevel})`;
                }
            }
        }
    }
    
    /**
     * Maneja el evento cuando el usuario suelta el botón del mouse.
     * 
     * CASO DE USO: Usuario suelta el botón del mouse después de arrastrar o hacer click.
     * 
     * Lógica:
     * 1. Si estaba arrastrando:
     *    - Previene eventos por defecto de forma más agresiva (evita que PageFlip procese el click)
     *    - Esto evita que se voltee la página accidentalmente después de arrastrar
     * 2. Desactiva el arrastre y marca isMouseDown = false
     * 3. Restaura el cursor a "default" (no muestra "grab" hasta que el usuario intente arrastrar de nuevo)
     * 
     * MEJORAS DE USABILIDAD:
     * - Previene eventos de forma más agresiva cuando hubo arrastre
     * - El cursor vuelve a "default" para no confundir al usuario
     * - No muestra "grab" hasta que el usuario realmente intente arrastrar
     * 
     * @param {MouseEvent} e - Evento de mouse up
     */
    function handleMouseUp(e) {
        // Si estaba arrastrando, prevenir el evento de flip de PageFlip de forma más agresiva
        // Esto evita que se voltee la página accidentalmente después de arrastrar
        if (isDragging) {
            e.preventDefault();
            e.stopPropagation();
            e.stopImmediatePropagation();
        }
        
        // Desactivar el arrastre y el estado de mouse presionado
        isDragging = false;
        isMouseDown = false;
        
        // Resetear posición inicial para el próximo intento
        startPosition = { x: 0, y: 0 };
        
        // Restaurar el cursor a "default" (no mostrar "grab" hasta que el usuario intente arrastrar)
        if (bookContainer) {
            bookContainer.style.cursor = 'default';
        }
    }
    
    // ============================================================================
    // VARIABLES DE CONTROL DE GESTOS TÁCTILES
    // ============================================================================
    
    /**
     * Distancia inicial entre dos dedos cuando comienza el pinch zoom.
     * Se usa para calcular el factor de escala relativo.
     */
    let initialDistance = 0;
    
    /**
     * Nivel de zoom inicial cuando comienza el pinch zoom.
     * Se usa para calcular el nuevo zoom basándose en el cambio de distancia.
     */
    let initialZoom = 1;
    
    /**
     * Posición inicial del toque cuando el usuario toca la pantalla con un dedo.
     * Se usa para detectar si el usuario está haciendo tap o arrastre.
     * Formato: { x: número, y: número } en píxeles.
     */
    let touchStartPos = { x: 0, y: 0 };
    
    /**
     * Umbral en píxeles para considerar arrastre táctil.
     * Mayor que dragThreshold porque los toques son menos precisos que el mouse.
     * Aumentado de 10px a 20px para evitar activación accidental durante taps.
     * Si el dedo se mueve más de 20px, se considera arrastre.
     */
    let touchDragThreshold = 20;
    
    /**
     * Indica si el usuario está haciendo pinch zoom (dos dedos).
     * Se usa para distinguir entre zoom y arrastre táctil.
     */
    let isZooming = false;
    
    /**
     * Calcula la distancia euclidiana entre dos puntos táctiles.
     * 
     * CASO DE USO: Calcular la distancia entre dos dedos para pinch zoom.
     * 
     * Lógica:
     * - Usa el teorema de Pitágoras: √(dx² + dy²)
     * - dx: diferencia en coordenada X
     * - dy: diferencia en coordenada Y
     * 
     * @param {TouchList} touches - Lista de puntos táctiles (debe tener 2 elementos)
     * @returns {number} Distancia en píxeles entre los dos puntos
     */
    function getTouchDistance(touches) {
        const dx = touches[0].clientX - touches[1].clientX;
        const dy = touches[0].clientY - touches[1].clientY;
        return Math.sqrt(dx * dx + dy * dy);
    }
    
    /**
     * Maneja el evento cuando el usuario toca la pantalla.
     * 
     * CASO DE USO 1: Usuario toca con dos dedos → Inicia pinch zoom.
     * CASO DE USO 2: Usuario toca con un dedo y hay zoom → Prepara arrastre táctil.
     * 
     * Lógica para dos dedos (pinch zoom):
     * 1. Previene eventos por defecto de forma más agresiva (scroll, zoom del navegador, PageFlip)
     * 2. Activa isZooming = true
     * 3. Guarda la distancia inicial entre los dedos
     * 4. Guarda el nivel de zoom inicial
     * 
     * Lógica para un dedo (arrastre táctil):
     * 1. Solo funciona si zoomLevel > 1
     * 2. Previene eventos inmediatamente para evitar que PageFlip los capture
     * 3. Guarda la posición inicial del toque
     * 4. Calcula dragStart para mantener el offset correcto
     * 5. Inicia timeout de 200ms para distinguir entre tap y arrastre
     * 
     * MEJORAS DE USABILIDAD:
     * - Previene eventos inmediatamente cuando hay zoom para evitar conflictos
     * - Timeout aumentado a 200ms para dar más tiempo al usuario
     * 
     * @param {TouchEvent} e - Evento de touch start
     */
    function handleTouchStart(e) {
        // CASO 1: Dos dedos → Pinch zoom
        if (e.touches.length === 2) {
            // Prevenir zoom y scroll del navegador de forma más agresiva
            e.preventDefault();
            e.stopPropagation();
            e.stopImmediatePropagation();
            
            // Activar modo zoom
            isZooming = true;
            
            // Guardar distancia inicial entre los dedos
            initialDistance = getTouchDistance(e.touches);
            
            // Guardar nivel de zoom inicial para calcular el cambio relativo
            initialZoom = zoomLevel;
        } 
        // CASO 2: Un dedo y hay zoom → Posible arrastre táctil
        else if (e.touches.length === 1 && zoomLevel > 1) {
            // Prevenir eventos inmediatamente para evitar que PageFlip los capture
            e.preventDefault();
            e.stopPropagation();
            e.stopImmediatePropagation();
            
            const touch = e.touches[0];
            
            // Guardar posición inicial del toque
            touchStartPos = { x: touch.clientX, y: touch.clientY };
            
            // Calcular offset inicial para mantener posición relativa
            dragStart = { x: touch.clientX - bookPosition.x, y: touch.clientY - bookPosition.y };
            
            // Esperar 200ms para determinar si es tap o arrastre
            // Aumentado de 150ms a 200ms para dar más tiempo al usuario
            // Si el usuario mueve el dedo antes, se cancela y activa arrastre inmediatamente
            dragTimeout = setTimeout(() => {
                // Solo activar arrastre si no está haciendo zoom
                if (!isZooming) {
                    isDragging = true;
                }
            }, 200);
        }
    }
    
    /**
     * Maneja el movimiento de los dedos durante gestos táctiles.
     * 
     * CASO DE USO 1: Usuario mueve dos dedos → Actualiza pinch zoom.
     * CASO DE USO 2: Usuario mueve un dedo con zoom → Arrastra el catálogo.
     * 
     * Lógica para dos dedos (pinch zoom):
     * 1. Calcula la distancia actual entre los dedos
     * 2. Calcula el factor de escala: distancia_actual / distancia_inicial
     * 3. Aplica el factor al zoom inicial: zoom = zoom_inicial * escala
     * 4. Limita el zoom entre 0.5 y 3
     * 5. Actualiza la transformación CSS
     * 
     * Lógica para un dedo (arrastre táctil):
     * 1. Calcula la distancia desde la posición inicial
     * 2. Si supera el threshold Y no está arrastrando ni haciendo zoom:
     *    - Cancela el timeout
     *    - Activa el arrastre
     * 3. Si ya está arrastrando:
     *    - Previene eventos por defecto de forma más agresiva
     *    - Actualiza bookPosition
     *    - Aplica la transformación CSS directamente (sin updateZoom para evitar loops)
     * 
     * MEJORAS DE USABILIDAD:
     * - Previene eventos de forma más agresiva con stopImmediatePropagation
     * - Aplica transformación directamente durante arrastre para mejor rendimiento
     * 
     * @param {TouchEvent} e - Evento de touch move
     */
    function handleTouchMove(e) {
        // CASO 1: Dos dedos moviéndose → Pinch zoom activo
        if (e.touches.length === 2 && isZooming) {
            // Prevenir zoom y scroll del navegador de forma más agresiva
            e.preventDefault();
            e.stopPropagation();
            e.stopImmediatePropagation();
            
            // Calcular distancia actual entre los dedos
            const distance = getTouchDistance(e.touches);
            
            // Calcular factor de escala: cuánto ha cambiado la distancia
            // Si los dedos se separan → scale > 1 → zoom in
            // Si los dedos se acercan → scale < 1 → zoom out
            const scale = distance / initialDistance;
            
            // Aplicar el factor de escala al zoom inicial
            // Limitar entre 0.5 (50%) y 3 (300%)
            zoomLevel = Math.max(0.5, Math.min(3, initialZoom * scale));
            
            // Aplicar la transformación CSS (sin actualizar PageFlip para evitar loops)
            if (bookContainer) {
                bookContainer.style.transform = `translate(${bookPosition.x}px, ${bookPosition.y}px) scale(${zoomLevel})`;
            }
        } 
        // CASO 2: Un dedo moviéndose con zoom → Arrastre táctil
        else if (e.touches.length === 1 && zoomLevel > 1) {
            const touch = e.touches[0];
            
            // Calcular distancia desde la posición inicial
            const deltaX = Math.abs(touch.clientX - touchStartPos.x);
            const deltaY = Math.abs(touch.clientY - touchStartPos.y);
            
            // Si se mueve más del threshold, iniciar arrastre inmediatamente
            // Esto cancela el timeout y activa el arrastre de forma instantánea
            if ((deltaX > touchDragThreshold || deltaY > touchDragThreshold) && !isDragging && !isZooming) {
                clearTimeout(dragTimeout);
                isDragging = true;
            }
            
            // Si está arrastrando y no está haciendo zoom, actualizar posición
            if (isDragging && !isZooming) {
                // Prevenir scroll y otros eventos del navegador de forma más agresiva
                e.preventDefault();
                e.stopPropagation();
                e.stopImmediatePropagation();
                
                // Calcular nueva posición basándose en la posición del dedo y el offset inicial
                bookPosition = {
                    x: touch.clientX - dragStart.x,
                    y: touch.clientY - dragStart.y
                };
                
                // Aplicar la transformación CSS directamente (sin updateZoom para evitar loops)
                if (bookContainer) {
                    bookContainer.style.transform = `translate(${bookPosition.x}px, ${bookPosition.y}px) scale(${zoomLevel})`;
                }
            }
        }
    }
    
    /**
     * Maneja el evento cuando el usuario levanta los dedos de la pantalla.
     * 
     * CASO DE USO: Usuario termina el gesto táctil (pinch zoom o arrastre).
     * 
     * Lógica:
     * 1. Cancela el timeout (por si aún no se había activado el arrastre)
     * 2. Si estaba haciendo zoom o arrastrando:
     *    - Previene eventos por defecto de forma más agresiva (evita que PageFlip procese el toque)
     *    - Esto evita que se voltee la página accidentalmente
     * 3. Si no quedan dedos en la pantalla:
     *    - Desactiva el arrastre
     *    - Desactiva el zoom
     *    - Actualiza PageFlip con el zoom final
     * 
     * MEJORAS DE USABILIDAD:
     * - Previene eventos de forma más agresiva cuando hubo zoom/arrastre
     * - Actualiza PageFlip al finalizar el gesto para sincronizar estados
     * 
     * @param {TouchEvent} e - Evento de touch end
     */
    function handleTouchEnd(e) {
        // Cancelar el timeout por si aún no se había activado el arrastre
        clearTimeout(dragTimeout);
        
        // Si estaba haciendo zoom o arrastrando, prevenir eventos por defecto de forma más agresiva
        // Esto evita que PageFlip interprete esto como un intento de voltear página
        if (isZooming || isDragging) {
            e.preventDefault();
            e.stopPropagation();
            e.stopImmediatePropagation();
        }
        
        // Si no quedan dedos en la pantalla, resetear estados
        if (e.touches.length === 0) {
            const wasDragging = isDragging;
            const wasZooming = isZooming;
            
            isDragging = false;
            isZooming = false;
            
            // Si hubo zoom o arrastre, actualizar PageFlip con el estado final
            if (wasZooming || wasDragging) {
                updateZoom();
            }
        }
    }
</script>

<!-- ============================================================================
     TEMPLATE HTML - ESTRUCTURA DEL CATÁLOGO INTERACTIVO
     ============================================================================
     
     CASOS DE USO DE INTERACCIÓN:
     
     CASO 1: VISUALIZACIÓN NORMAL (sin zoom)
     - El usuario puede hacer click en las páginas para voltearlas
     - Puede usar los botones de navegación (anterior/siguiente)
     - El cursor es 'default' (no se puede arrastrar)
     
     CASO 2: ZOOM CON CONTROLES DE BOTONES
     - Usuario hace click en botón "+" → zoomIn() → zoomLevel aumenta
     - Usuario hace click en botón "-" → zoomOut() → zoomLevel disminuye
     - Usuario hace click en botón reset → resetZoom() → zoomLevel = 1
     - El porcentaje se muestra en tiempo real
     
     CASO 3: ZOOM CON CTRL/CMD + RUEDA DEL MOUSE
     - Usuario presiona Ctrl (Windows/Linux) o Cmd (Mac)
     - Rueda hacia arriba → zoomIn()
     - Rueda hacia abajo → zoomOut()
     - El evento se captura en handleWheel()
     
     CASO 4: ARRASTRE CON MOUSE CUANDO HAY ZOOM
     - Usuario hace zoom (zoomLevel > 1)
     - PageFlip se deshabilita automáticamente cuando hay zoom para evitar conflictos
     - Presiona y mantiene el botón del mouse
     - Mueve el mouse más de 10px → el catálogo se arrastra (umbral aumentado)
     - Permite navegar por el contenido ampliado sin voltear páginas accidentalmente
     - El cursor cambia a 'grab' cuando hay zoom, 'grabbing' cuando arrastra
     - Los eventos se previenen de forma agresiva para evitar conflictos con PageFlip
     
     CASO 5: PINCH ZOOM EN DISPOSITIVOS TÁCTILES
     - Usuario toca con dos dedos
     - PageFlip se deshabilita automáticamente durante el pinch zoom
     - Separa los dedos → zoom in
     - Acerca los dedos → zoom out
     - El zoom se calcula en tiempo real basándose en la distancia entre dedos
     - Los eventos se previenen de forma agresiva para evitar scroll/zoom del navegador
     
     CASO 6: ARRASTRE TÁCTIL CUANDO HAY ZOOM
     - Usuario hace zoom (zoomLevel > 1)
     - PageFlip se deshabilita automáticamente cuando hay zoom para evitar conflictos
     - Toca con un dedo y arrastra más de 20px (umbral aumentado)
     - El catálogo se mueve siguiendo el dedo
     - Permite navegar por el contenido ampliado en dispositivos táctiles
     - Los eventos se previenen de forma agresiva para evitar voltear páginas accidentalmente
     
     CASO 7: CAMBIO ENTRE MÓVIL Y DESKTOP
     - Usuario redimensiona la ventana o rota el dispositivo
     - Si cambia de móvil a desktop (o viceversa), la página se recarga
     - PageFlip se reinicializa con la configuración correcta
     
     CASO 8: NAVEGACIÓN DE PÁGINAS
     - Usuario hace click en botón "anterior" → prevPage() → voltea hacia atrás
     - Usuario hace click en botón "siguiente" → nextPage() → voltea hacia adelante
     - También puede hacer click directamente en las páginas para voltearlas
     - PageFlip maneja las animaciones de volteo automáticamente
     ============================================================================ -->

<div class="catalog-container" on:wheel={handleWheel}>
    <!-- ========================================================================
         CONTROLES DE ZOOM
         ========================================================================
         Panel flotante en la esquina superior derecha con controles de zoom.
         Muestra el nivel de zoom actual y permite ajustarlo.
         ======================================================================== -->
    <div class="zoom-controls">
        <!-- Botón para reducir zoom (-25%) -->
        <button class="zoom-btn" on:click={zoomOut} aria-label="Reducir zoom">
            <i class="fas fa-search-minus"></i>
        </button>
        
        <!-- Indicador del nivel de zoom actual (ej: "125%") -->
        <span class="zoom-level">{Math.round(zoomLevel * 100)}%</span>
        
        <!-- Botón para aumentar zoom (+25%) -->
        <button class="zoom-btn" on:click={zoomIn} aria-label="Aumentar zoom">
            <i class="fas fa-search-plus"></i>
        </button>
        
        <!-- Botón para resetear zoom a 100% y centrar -->
        <button class="zoom-btn" on:click={resetZoom} aria-label="Resetear zoom">
            <i class="fas fa-expand-arrows-alt"></i>
        </button>
        
        <!-- Separador visual -->
        <div class="control-separator"></div>
        
        <!-- Botón para descargar PDF -->
        <button class="zoom-btn download-btn" on:click={downloadPDF} aria-label="Descargar PDF">
            <i class="fas fa-download"></i>
        </button>
    </div>
    
    <!-- ========================================================================
         CONTROLES DE NAVEGACIÓN Y CONTENEDOR DEL LIBRO
         ======================================================================== -->
    <div class="flipbook-controls">
        <!-- Botón para ir a la página anterior (desktop: lado izquierdo) -->
        <button class="control-btn control-btn-desktop" on:click={prevPage} aria-label="Página anterior">
            <i class="fas fa-arrow-circle-left"></i>
        </button>
        
        <!-- ====================================================================
             CONTENEDOR DEL LIBRO CON ZOOM Y ARRASTRE
             ====================================================================
             Este contenedor:
             - Aplica las transformaciones CSS de zoom y posición
             - Maneja eventos de mouse (arrastre con zoom)
             - Maneja eventos táctiles (pinch zoom y arrastre)
             - Cambia el cursor según el estado (grab/grabbing/default)
             ==================================================================== -->
        <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
        <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
        <div class="book-wrapper" 
             bind:this={bookContainer}
             on:mousedown={handleMouseDown}
             on:touchstart={handleTouchStart}
             on:touchmove={handleTouchMove}
             on:touchend={handleTouchEnd}
             role="application"
             aria-label="Catálogo interactivo con zoom y arrastre"
             style="cursor: {isDragging ? 'grabbing' : 'default'}"
             tabindex="0">
            <!-- ================================================================
                 CONTENEDOR DEL LIBRO (PAGEFLIP)
                 ================================================================
                 Este es el elemento que PageFlip usa para inicializar el flipbook.
                 Contiene todas las páginas del catálogo.
                 ================================================================ -->
            <div id="book" bind:this={book} class="book">
            <!-- ================================================================
                 PÁGINAS DEL CATÁLOGO
                 ================================================================
                 Cada div.my-page representa una página del catálogo.
                 PageFlip los convierte en páginas interactivas con animaciones.
                 
                 data-density="hard": Indica que la primera y última página son
                 portadas duras (como un libro físico). Solo se aplica a la
                 primera y última página.
                 ================================================================ -->
            <div class="my-page" data-density="hard">
                <img src="/images/catalogo-01.jpg" alt="Página 1" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-02.jpg" alt="Página 2" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-03.jpg" alt="Página 3" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-04.jpg" alt="Página 4" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-05.jpg" alt="Página 5" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-06.jpg" alt="Página 6" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-07.jpg" alt="Página 7" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-08.jpg" alt="Página 8" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-09.jpg" alt="Página 9" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-10.jpg" alt="Página 10" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-11.jpg" alt="Página 11" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-12.jpg" alt="Página 12" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-13.jpg" alt="Página 13" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-14.jpg" alt="Página 14" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-15.jpg" alt="Página 15" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-16.jpg" alt="Página 16" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-17.jpg" alt="Página 17" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-18.jpg" alt="Página 18" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo-19.jpg" alt="Página 19" />
            </div>
            
            <div class="my-page" data-density="hard">
                <img src="/images/catalogo-20.jpg" alt="Página 20" />
            </div>
        </div>
        </div>
        
        <!-- Botón para ir a la página siguiente (desktop: lado derecho) -->
        <button class="control-btn control-btn-desktop" on:click={nextPage} aria-label="Página siguiente">
            <i class="fas fa-arrow-circle-right"></i>
        </button>
    </div>
    
    <!-- ========================================================================
         CONTROLES DE NAVEGACIÓN MÓVIL (PARTE INFERIOR)
         ========================================================================
         En móvil, los botones de navegación y el indicador de página se muestran
         en la parte inferior para mejor accesibilidad.
         ======================================================================== -->
    <div class="mobile-navigation">
        <!-- Botón para ir a la página anterior (móvil) -->
        <button class="control-btn control-btn-mobile" on:click={prevPage} aria-label="Página anterior">
            <i class="fas fa-arrow-circle-left"></i>
        </button>
        
        <!-- Indicador de página actual -->
        <div class="page-indicator">
            <span class="page-indicator-text">
                {currentPage > 0 ? `Página ${currentPage + 1} de ${totalPages}` : 'Cargando...'}
            </span>
        </div>
        
        <!-- Botón para ir a la página siguiente (móvil) -->
        <button class="control-btn control-btn-mobile" on:click={nextPage} aria-label="Página siguiente">
            <i class="fas fa-arrow-circle-right"></i>
        </button>
    </div>
    
    <!-- ========================================================================
         TUTORIAL INICIAL
         ========================================================================
         Se muestra solo la primera vez que el usuario accede a la aplicación.
         Explica las funcionalidades principales: zoom, cambio de páginas y descarga.
         ======================================================================== -->
    <Tutorial show={showTutorial} isMobile={isMobile} on:close={closeTutorial} />
</div>

<!-- ============================================================================
     ESTILOS CSS - DISEÑO RESPONSIVE DEL CATÁLOGO
     ============================================================================
     
     El diseño se adapta automáticamente entre móvil y desktop:
     - Móvil: < 768px de ancho
     - Desktop: >= 768px de ancho
     ============================================================================ -->

<style>
    /* ============================================================================
       CONTENEDOR PRINCIPAL
       ============================================================================
       Contenedor de pantalla completa que centra el catálogo.
       ============================================================================ */
    .catalog-container {
        height: 100vh;  /* Altura completa de la ventana */
        width: 100%;  /* Ancho completo */
        display: flex;
        flex-direction: column;
        justify-content: center;  /* Centra verticalmente */
        align-items: center;  /* Centra horizontalmente */
        background-color: #444444;  /* Fondo gris oscuro */
        overflow: hidden;  /* Oculta scrollbars */
        position: relative;  /* Para posicionar controles absolutos */
    }
    
    /* ============================================================================
       CONTROLES DE ZOOM
       ============================================================================
       Panel flotante en la esquina superior derecha.
       ============================================================================ */
    .zoom-controls {
        position: absolute;  /* Posicionamiento absoluto */
        top: 20px;  /* 20px desde arriba en desktop */
        right: 20px;  /* 20px desde la derecha en desktop */
        display: flex;
        align-items: center;
        gap: 8px;  /* Espacio entre botones */
        background: rgba(0, 0, 0, 0.7);  /* Fondo semitransparente oscuro */
        padding: 8px 12px;  /* Padding interno */
        border-radius: 20px;  /* Bordes redondeados */
        z-index: 1000;  /* Por encima de todo */
    }
    
    /* Botones de zoom */
    .zoom-btn {
        background: none;
        border: none;
        color: white;
        font-size: 16px;  /* Tamaño de icono en desktop */
        cursor: pointer;
        padding: 4px 8px;
        border-radius: 4px;
        transition: background-color 0.3s ease;  /* Transición suave al hover */
    }
    
    .zoom-btn:hover {
        background-color: rgba(255, 255, 255, 0.2);  /* Fondo claro al hover */
    }
    
    /* Indicador de nivel de zoom */
    .zoom-level {
        color: white;
        font-size: 14px;  /* Tamaño de texto en desktop */
        font-weight: bold;
        min-width: 45px;  /* Ancho mínimo para evitar cambios de tamaño */
        text-align: center;
    }
    
    /* Separador visual entre controles de zoom y descarga */
    .control-separator {
        width: 1px;
        height: 20px;
        background-color: rgba(255, 255, 255, 0.3);
        margin: 0 4px;
    }
    
    /* Botón de descarga con estilo destacado */
    .download-btn {
        color: #4CAF50;  /* Color verde para destacar */
    }
    
    .download-btn:hover {
        background-color: rgba(76, 175, 80, 0.2);  /* Fondo verde claro al hover */
    }
    
    /* ============================================================================
       CONTROLES DE NAVEGACIÓN
       ============================================================================ */
    .flipbook-controls {
        display: flex;
        align-items: center;
        justify-content: center;
    }
    
    /* Botones de navegación para desktop (lados) */
    .control-btn-desktop {
        display: block;
    }
    
    /* ============================================================================
       CONTROLES DE NAVEGACIÓN MÓVIL (PARTE INFERIOR)
       ============================================================================ */
    .mobile-navigation {
        display: flex;
        align-items: center;
        justify-content: center;
        position: absolute;
        bottom: 20px;
        left: 50%;
        transform: translateX(-50%);
        z-index: 1000;
        width: 100%;
        padding: 0 20px;
        box-sizing: border-box;
    }
    
    /* Ocultar botones móviles en desktop por defecto */
    .control-btn-mobile {
        display: none;
    }
    
    /* Indicador de página */
    .page-indicator {
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 8px 16px;
        background: rgba(0, 0, 0, 0.7);
        border-radius: 20px;
        margin: 0 10px;
    }
    
    .page-indicator-text {
        color: white;
        font-size: 14px;
        font-weight: bold;
    }
    
    /* ============================================================================
       CONTENEDOR DEL LIBRO CON ZOOM
       ============================================================================
       Este contenedor aplica las transformaciones CSS de zoom y posición.
       ============================================================================ */
    .book-wrapper {
        transition: transform 0.2s ease-out;  /* Transición suave para zoom */
        transform-origin: center center;  /* El zoom se aplica desde el centro */
    }
    
    /* ============================================================================
       CONTENEDOR DEL LIBRO (PAGEFLIP)
       ============================================================================
       Este es el elemento que PageFlip usa para renderizar el flipbook.
       ============================================================================ */
    .book {
        margin: 0 20px;  /* Márgenes laterales en desktop */
        background-color: transparent;
        box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);  /* Sombra sutil */
    }
    
    /* ============================================================================
       PÁGINAS DEL CATÁLOGO
       ============================================================================
       Cada página tiene proporción 11.03:15.60 (tamaño real del documento en pulgadas).
       Sin padding para eliminar bordes blancos.
       ============================================================================ */
    .my-page {
        background-color: transparent;  /* Fondo transparente para que no se vean bordes */
        aspect-ratio: 11.03 / 15.60;  /* Proporción real del documento (11.03 × 15.60 in) */
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 0;  /* Sin padding para eliminar bordes blancos */
    }
    
    /* Imágenes dentro de las páginas */
    .my-page img {
        width: 100%;
        height: 100%;
        object-fit: contain;  /* Mantiene la imagen completa sin recortar */
        display: block;
        /* Optimización de renderizado para imágenes de catálogo */
        image-rendering: -webkit-optimize-contrast;
        image-rendering: optimize-contrast;
        border-radius: 0;  /* Sin bordes redondeados */
        box-shadow: none;  /* Sin sombra */
    }
    
    /* ============================================================================
       BOTONES DE NAVEGACIÓN (ANTERIOR/SIGUIENTE)
       ============================================================================ */
    .control-btn {
        background: none;
        border: none;
        font-size: 24px;  /* Tamaño base del icono */
        cursor: pointer;
        color: #f0f0f0;  /* Color gris claro */
        padding: 10px;
        z-index: 100;  /* Por encima del libro */
        transition: all 0.3s ease;  /* Transición suave */
    }
    
    .control-btn:hover {
        transform: scale(1.2);  /* Aumenta tamaño al hover */
    }
    
    /* Iconos FontAwesome */
    i {
        font-size: 40px;  /* Tamaño de iconos en desktop */
        color: #f0f0f0;
    }
    
    /* ============================================================================
       MEDIA QUERIES - DISEÑO RESPONSIVE PARA MÓVIL
       ============================================================================
       Se aplica cuando el ancho de la ventana es <= 768px.
       ============================================================================ */
    @media (max-width: 768px) {
        /* Ocultar botones de navegación de desktop en móvil */
        .control-btn-desktop {
            display: none;
        }
        
        /* Mostrar botones de navegación móvil */
        .control-btn-mobile {
            display: block;
            margin: 0 5px;
        }
        
        /* Iconos más pequeños en móvil */
        .control-btn-mobile i {
            font-size: 30px;
        }
        
        /* Indicador de página en móvil */
        .page-indicator {
            padding: 6px 12px;
            margin: 0 10px;
        }
        
        .page-indicator-text {
            font-size: 12px;
        }
        
        /* Libro con márgenes reducidos */
        .book {
            margin: 0 10px;  /* Reducido de 20px a 10px */
        }
        
        /* Controles de zoom más compactos */
        .zoom-controls {
            top: 10px;  /* Reducido de 20px a 10px */
            right: 10px;  /* Reducido de 20px a 10px */
            padding: 6px 10px;  /* Padding reducido */
        }
        
        /* Botones de zoom más pequeños */
        .zoom-btn {
            font-size: 14px;  /* Reducido de 16px a 14px */
            padding: 3px 6px;  /* Padding reducido */
        }
        
        /* Indicador de zoom más pequeño */
        .zoom-level {
            font-size: 12px;  /* Reducido de 14px a 12px */
            min-width: 40px;  /* Reducido de 45px a 40px */
        }
    }
    
    /* ============================================================================
       INDICADOR DE PÁGINA EN DESKTOP
       ============================================================================
       En desktop, el indicador se muestra en la parte inferior central.
       ============================================================================ */
    @media (min-width: 769px) {
        /* En desktop, solo mostrar el indicador, no los botones móviles */
        .control-btn-mobile {
            display: none;
        }
        
        .page-indicator {
            margin: 0; /* Sin márgenes laterales en desktop */
        }
    }
</style> 