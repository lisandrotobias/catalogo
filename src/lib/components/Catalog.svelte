<script>
    import { onMount } from 'svelte';
    
    let book;
    let pageFlip;
    let isMobile = false;
    let zoomLevel = 1;
    let bookContainer;
    let isDragging = false;
    let dragStart = { x: 0, y: 0 };
    let bookPosition = { x: 0, y: 0 };
    
    // Detectar si estamos en mobile
    function checkMobile() {
        if (typeof window !== 'undefined') {
            isMobile = window.innerWidth < 768;
            return isMobile;
        }
        return false;
    }
    
    onMount(() => {
        // Detectar inicialmente si es mobile
        checkMobile();
        
        // Cargar el script según la documentación oficial
        const script = document.createElement('script');
        script.src = 'https://unpkg.com/page-flip@2.0.7/dist/js/page-flip.browser.js';
        script.onload = () => {
            if (!window.St || !window.St.PageFlip) {
                console.error('PageFlip no se cargó correctamente');
                return;
            }
            
            // Configuración optimizada para catálogo comercial
            const config = {
                width: isMobile ? 340 : 650,  // Optimizado para catálogo
                height: isMobile ? 480 : 900,  // Proporción ajustada para catálogos
                size: "fixed", 
                minWidth: isMobile ? 300 : 550,
                maxWidth: isMobile ? 400 : 750,
                minHeight: isMobile ? 420 : 780,
                maxHeight: isMobile ? 540 : 1000,
                drawShadow: true,
                flippingTime: isMobile ? 1000 : 1200,
                usePortrait: isMobile,
                startZIndex: 0,
                autoSize: true,
                maxShadowOpacity: 0.4,
                showCover: true,
                mobileScrollSupport: true
            };
            
            console.log("Modo:", isMobile ? "móvil" : "desktop");
            console.log("Configuración catálogo:", config);
            
            // Inicializar PageFlip con la sintaxis correcta
            pageFlip = new window.St.PageFlip(book, config);
            
            // Cargar desde HTML como en el ejemplo
            pageFlip.loadFromHTML(document.querySelectorAll(".my-page"));
            
            // Manejar cambios de tamaño de ventana
            window.addEventListener('resize', handleResize);
            
            // Agregar eventos según la documentación
            pageFlip.on('flip', (e) => {
                console.log("Página actual: " + e.data);
            });
            
            pageFlip.on('changeOrientation', (e) => {
                console.log("Orientación cambiada: " + e.data);
            });
        };
        
        document.head.appendChild(script);
        
        // Agregar event listeners para zoom y arrastre
        window.addEventListener('wheel', handleWheel, { passive: false });
        window.addEventListener('mousemove', handleMouseMove);
        window.addEventListener('mouseup', handleMouseUp);
        
        return () => {
            window.removeEventListener('resize', handleResize);
            window.removeEventListener('wheel', handleWheel);
            window.removeEventListener('mousemove', handleMouseMove);
            window.removeEventListener('mouseup', handleMouseUp);
            if (pageFlip) {
                pageFlip.destroy();
            }
        };
    });
    
    // Manejar cambios de tamaño de ventana
    function handleResize() {
        const wasMobile = isMobile;
        checkMobile();
        
        // Si cambió entre mobile y desktop, reinicializar con la configuración correcta
        if (wasMobile !== isMobile && pageFlip) {
            console.log("Cambio de modo a:", isMobile ? "móvil" : "desktop");
            location.reload(); // La forma más simple de manejar el cambio es recargar la página
        }
    }
    
    function prevPage() {
        if (pageFlip) pageFlip.flipPrev('top');
    }
    
    function nextPage() {
        if (pageFlip) pageFlip.flipNext('bottom');
    }
    
    // Funciones de zoom
    function zoomIn() {
        if (zoomLevel < 3) {
            zoomLevel += 0.25;
            updateZoom();
        }
    }
    
    function zoomOut() {
        if (zoomLevel > 0.5) {
            zoomLevel -= 0.25;
            updateZoom();
        }
    }
    
    function resetZoom() {
        zoomLevel = 1;
        bookPosition = { x: 0, y: 0 };
        updateZoom();
    }
    
    function updateZoom() {
        if (bookContainer) {
            bookContainer.style.transform = `translate(${bookPosition.x}px, ${bookPosition.y}px) scale(${zoomLevel})`;
        }
    }
    
    // Manejo de rueda del mouse para zoom
    function handleWheel(e) {
        if (e.ctrlKey || e.metaKey) {
            e.preventDefault();
            e.stopPropagation();
            if (e.deltaY > 0) {
                zoomOut();
            } else {
                zoomIn();
            }
        }
    }
    
    // Variables para controlar la interacción
    let dragTimeout;
    let dragThreshold = 5; // píxeles mínimos para considerar arrastre
    let startPosition = { x: 0, y: 0 };
    
    // Funciones para arrastrar cuando está con zoom
    function handleMouseDown(e) {
        if (zoomLevel > 1) {
            startPosition = { x: e.clientX, y: e.clientY };
            dragStart = { x: e.clientX - bookPosition.x, y: e.clientY - bookPosition.y };
            
            // Esperar un momento para determinar si es click o arrastre
            dragTimeout = setTimeout(() => {
                isDragging = true;
                bookContainer.style.cursor = 'grabbing';
                e.preventDefault();
            }, 150);
        }
    }
    
    function handleMouseMove(e) {
        if (zoomLevel > 1) {
            const deltaX = Math.abs(e.clientX - startPosition.x);
            const deltaY = Math.abs(e.clientY - startPosition.y);
            
            // Si se mueve más del threshold, iniciar arrastre inmediatamente
            if ((deltaX > dragThreshold || deltaY > dragThreshold) && !isDragging) {
                clearTimeout(dragTimeout);
                isDragging = true;
                bookContainer.style.cursor = 'grabbing';
            }
            
            if (isDragging) {
                e.preventDefault();
                e.stopPropagation();
                bookPosition = {
                    x: e.clientX - dragStart.x,
                    y: e.clientY - dragStart.y
                };
                updateZoom();
            }
        }
    }
    
    function handleMouseUp(e) {
        clearTimeout(dragTimeout);
        
        // Si estaba arrastrando, prevenir el evento de flip
        if (isDragging) {
            e.preventDefault();
            e.stopPropagation();
        }
        
        isDragging = false;
        if (bookContainer) {
            bookContainer.style.cursor = zoomLevel > 1 ? 'grab' : 'default';
        }
    }
    
    // Soporte para gestos táctiles (pinch zoom)
    let initialDistance = 0;
    let initialZoom = 1;
    let touchStartPos = { x: 0, y: 0 };
    let touchDragThreshold = 10;
    let isZooming = false;
    
    function getTouchDistance(touches) {
        const dx = touches[0].clientX - touches[1].clientX;
        const dy = touches[0].clientY - touches[1].clientY;
        return Math.sqrt(dx * dx + dy * dy);
    }
    
    function handleTouchStart(e) {
        if (e.touches.length === 2) {
            e.preventDefault();
            e.stopPropagation();
            isZooming = true;
            initialDistance = getTouchDistance(e.touches);
            initialZoom = zoomLevel;
        } else if (e.touches.length === 1 && zoomLevel > 1) {
            const touch = e.touches[0];
            touchStartPos = { x: touch.clientX, y: touch.clientY };
            dragStart = { x: touch.clientX - bookPosition.x, y: touch.clientY - bookPosition.y };
            
            // Esperar para determinar si es tap o arrastre
            dragTimeout = setTimeout(() => {
                if (!isZooming) {
                    isDragging = true;
                }
            }, 150);
        }
    }
    
    function handleTouchMove(e) {
        if (e.touches.length === 2 && isZooming) {
            e.preventDefault();
            e.stopPropagation();
            const distance = getTouchDistance(e.touches);
            const scale = distance / initialDistance;
            zoomLevel = Math.max(0.5, Math.min(3, initialZoom * scale));
            updateZoom();
        } else if (e.touches.length === 1 && zoomLevel > 1) {
            const touch = e.touches[0];
            const deltaX = Math.abs(touch.clientX - touchStartPos.x);
            const deltaY = Math.abs(touch.clientY - touchStartPos.y);
            
            // Si se mueve más del threshold, iniciar arrastre
            if ((deltaX > touchDragThreshold || deltaY > touchDragThreshold) && !isDragging && !isZooming) {
                clearTimeout(dragTimeout);
                isDragging = true;
            }
            
            if (isDragging && !isZooming) {
                e.preventDefault();
                e.stopPropagation();
                bookPosition = {
                    x: touch.clientX - dragStart.x,
                    y: touch.clientY - dragStart.y
                };
                updateZoom();
            }
        }
    }
    
    function handleTouchEnd(e) {
        clearTimeout(dragTimeout);
        
        if (isZooming || isDragging) {
            e.preventDefault();
            e.stopPropagation();
        }
        
        if (e.touches.length === 0) {
            isDragging = false;
            isZooming = false;
        }
    }
</script>

<div class="catalog-container" on:wheel={handleWheel}>
    <!-- Controles de zoom -->
    <div class="zoom-controls">
        <button class="zoom-btn" on:click={zoomOut} aria-label="Reducir zoom">
            <i class="fas fa-search-minus"></i>
        </button>
        <span class="zoom-level">{Math.round(zoomLevel * 100)}%</span>
        <button class="zoom-btn" on:click={zoomIn} aria-label="Aumentar zoom">
            <i class="fas fa-search-plus"></i>
        </button>
        <button class="zoom-btn" on:click={resetZoom} aria-label="Resetear zoom">
            <i class="fas fa-expand-arrows-alt"></i>
        </button>
    </div>
    
    <div class="flipbook-controls">
        <button class="control-btn" on:click={prevPage} aria-label="Página anterior">
            <i class="fas fa-arrow-circle-left"></i>
        </button>
        
        <!-- Contenedor de libro con zoom -->
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
             style="cursor: {zoomLevel > 1 ? 'grab' : 'default'}"
             tabindex="0">
            <div id="book" bind:this={book} class="book">
            <!-- Páginas del catálogo -->
            <div class="my-page" data-density="hard">
                <img src="/images/catalogo_01.jpg" alt="Página 1" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_02.jpg" alt="Página 2" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_03.jpg" alt="Página 3" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_04.jpg" alt="Página 4" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_05.jpg" alt="Página 5" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_06.jpg" alt="Página 6" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_07.jpg" alt="Página 7" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_08.jpg" alt="Página 8" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_09.jpg" alt="Página 9" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_10.jpg" alt="Página 10" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_11.jpg" alt="Página 11" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_12.jpg" alt="Página 12" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_13.jpg" alt="Página 13" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_14.jpg" alt="Página 14" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_15.jpg" alt="Página 15" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_16.jpg" alt="Página 16" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_17.jpg" alt="Página 17" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_18.jpg" alt="Página 18" />
            </div>
            
            <div class="my-page">
                <img src="/images/catalogo_19.jpg" alt="Página 19" />
            </div>
            
            <div class="my-page" data-density="hard">
                <img src="/images/catalogo_20.jpg" alt="Página 20" />
            </div>
        </div>
        </div>
        
        <button class="control-btn" on:click={nextPage} aria-label="Página siguiente">
            <i class="fas fa-arrow-circle-right"></i>
        </button>
    </div>
</div>

<style>
    .catalog-container {
        height: 100vh;
        width: 100%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        background-color: #444444;
        overflow: hidden;
        position: relative;
    }
    
    .zoom-controls {
        position: absolute;
        top: 20px;
        right: 20px;
        display: flex;
        align-items: center;
        gap: 8px;
        background: rgba(0, 0, 0, 0.7);
        padding: 8px 12px;
        border-radius: 20px;
        z-index: 1000;
    }
    
    .zoom-btn {
        background: none;
        border: none;
        color: white;
        font-size: 16px;
        cursor: pointer;
        padding: 4px 8px;
        border-radius: 4px;
        transition: background-color 0.3s ease;
    }
    
    .zoom-btn:hover {
        background-color: rgba(255, 255, 255, 0.2);
    }
    
    .zoom-level {
        color: white;
        font-size: 14px;
        font-weight: bold;
        min-width: 45px;
        text-align: center;
    }
    
    .flipbook-controls {
        display: flex;
        align-items: center;
        justify-content: center;
    }
    
    .book-wrapper {
        transition: transform 0.2s ease-out;
        transform-origin: center center;
    }
    
    .book {
        /* Dimensiones adaptadas para catálogo */
        margin: 0 20px;
        background-color: transparent;
        box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
    }
    
    .my-page {
        background-color: white;
        /* Proporción optimizada para catálogo comercial */
        aspect-ratio: 3 / 4;
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 8px;
    }
    
    .my-page img {
        width: 100%;
        height: 100%;
        object-fit: contain;
        display: block;
        /* Optimizado para imágenes de catálogo */
        image-rendering: -webkit-optimize-contrast;
        image-rendering: optimize-contrast;
        border-radius: 4px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    }
    
    .control-btn {
        background: none;
        border: none;
        font-size: 24px;
        cursor: pointer;
        color: #f0f0f0;
        padding: 10px;
        z-index: 100;
        transition: all 0.3s ease;
    }
    
    .control-btn:hover {
        transform: scale(1.2);
    }
    
    i {
        font-size: 40px;
        color: #f0f0f0;
    }
    
    @media (max-width: 768px) {
        .control-btn {
            margin: 0 5px;
        }
        
        i {
            font-size: 30px;
        }
        
        .book {
            margin: 0 10px;
        }
        
        .zoom-controls {
            top: 10px;
            right: 10px;
            padding: 6px 10px;
        }
        
        .zoom-btn {
            font-size: 14px;
            padding: 3px 6px;
        }
        
        .zoom-level {
            font-size: 12px;
            min-width: 40px;
        }
    }
</style> 