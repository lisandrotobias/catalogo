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
            
            // Configuración específica según el dispositivo
            const config = {
                width: isMobile ? 400 : 550, // Más ancho en desktop
                height: isMobile ? 600 : 733, // Más alto en desktop
                size: "fixed", 
                minWidth: isMobile ? 300 : 500,
                maxWidth: isMobile ? 500 : 1000,
                minHeight: isMobile ? 450 : 667,
                maxHeight: isMobile ? 700 : 1000,
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
            console.log("Configuración:", config);
            
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
            <!-- Estructura exacta según la documentación -->
            <div class="my-page" data-density="hard">
                <img src="/images/catalogo_01.jpg" alt="Portada" />
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
                <img src="/images/catalogo_20.jpg" alt="Contraportada" />
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
        /* Tamaño definido según la documentación */
        margin: 0 20px;
        background-color: transparent;
        box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
    }
    
    .my-page {
        background-color: white;
    }
    
    .my-page img {
        width: 100%;
        height: 100%;
        object-fit: contain;
        display: block;
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
    }
</style> 