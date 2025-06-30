<script>
    import { onMount } from 'svelte';
    
    let book;
    let pageFlip;
    let isMobile = false;
    
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
            
            // Configuración específica para formato A4 (proporción 210:297)
            const config = {
                width: isMobile ? 320 : 600,  // Ancho A4 proporcionado
                height: isMobile ? 453 : 849, // Alto A4 proporcionado (210:297 ratio)
                size: "fixed", 
                minWidth: isMobile ? 280 : 500,
                maxWidth: isMobile ? 400 : 700,
                minHeight: isMobile ? 396 : 707, // Manteniendo proporción A4
                maxHeight: isMobile ? 566 : 990, // Manteniendo proporción A4
                drawShadow: true,
                flippingTime: isMobile ? 1000 : 1200,
                usePortrait: isMobile, // Esto es clave: true para móvil (una página), false para desktop (dos páginas)
                startZIndex: 0,
                autoSize: true,
                maxShadowOpacity: 0.5,
                showCover: true,
                mobileScrollSupport: true
            };
            
            console.log("Modo:", isMobile ? "móvil" : "desktop");
            console.log("Configuración A4:", config);
            
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
        
        return () => {
            window.removeEventListener('resize', handleResize);
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
</script>

<div class="catalog-container">
    <div class="flipbook-controls">
        <button class="control-btn" on:click={prevPage}>
            <i class="fas fa-arrow-circle-left"></i>
        </button>
        
        <!-- Contenedor de libro con id "book" como en la documentación -->
        <div id="book" bind:this={book} class="book">
            <!-- Páginas del trabajo integrador de Selena Sosa -->
            <div class="my-page" data-density="hard">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0001.jpg" alt="Página 1" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0002.jpg" alt="Página 2" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0003.jpg" alt="Página 3" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0004.jpg" alt="Página 4" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0005.jpg" alt="Página 5" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0006.jpg" alt="Página 6" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0007.jpg" alt="Página 7" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0008.jpg" alt="Página 8" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0009.jpg" alt="Página 9" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0010.jpg" alt="Página 10" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0011.jpg" alt="Página 11" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0012.jpg" alt="Página 12" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0013.jpg" alt="Página 13" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0014.jpg" alt="Página 14" />
            </div>
            
            <div class="my-page">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0015.jpg" alt="Página 15" />
            </div>
            
            <div class="my-page" data-density="hard">
                <img src="/images/trabajo_pou/TRABAJO INTEGRADOR FINAL-SELENA SOSA_page-0016.jpg" alt="Página 16" />
            </div>
        </div>
        
        <button class="control-btn" on:click={nextPage}>
            <i class="fas fa-arrow-circle-right"></i>
        </button>
    </div>
</div>

<style>
    .catalog-container {
        height: 100vh;
        width: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        background-color: #444444;
        overflow: hidden;
    }
    
    .flipbook-controls {
        display: flex;
        align-items: center;
        justify-content: center;
    }
    
    .book {
        /* Dimensiones adaptadas para formato A4 */
        margin: 0 20px;
        background-color: transparent;
        box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
    }
    
    .my-page {
        background-color: white;
        /* Asegurar que mantenga la proporción A4 */
        aspect-ratio: 210 / 297;
    }
    
    .my-page img {
        width: 100%;
        height: 100%;
        object-fit: contain;
        display: block;
        /* Mantener calidad de imagen para documentos */
        image-rendering: crisp-edges;
        image-rendering: -webkit-optimize-contrast;
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
    }
</style> 