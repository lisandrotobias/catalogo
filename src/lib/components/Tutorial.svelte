<!-- ============================================================================
     COMPONENTE TUTORIAL.SVELTE
     ============================================================================
     
     Componente de presentación inicial que explica las funcionalidades
     principales del catálogo: zoom, cambio de páginas y descarga.
     
     Se muestra solo la primera vez que el usuario accede a la aplicación.
     Contenido específico para móvil y desktop.
     ============================================================================ -->

<script>
    /**
     * Prop que indica si el tutorial debe mostrarse.
     * Controlado desde el componente padre (Catalog).
     */
    export let show = false;
    
    /**
     * Prop que indica si el dispositivo es móvil.
     * Se usa para mostrar contenido específico según el dispositivo.
     */
    export let isMobile = false;
    
    /**
     * Evento que se dispara cuando el usuario cierra el tutorial.
     * El componente padre debe manejar el cierre y guardar en localStorage.
     */
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    
    /**
     * Función para cerrar el tutorial.
     * Dispara el evento 'close' para que el padre maneje el cierre.
     */
    function closeTutorial() {
        dispatch('close');
    }
    
    /**
     * Maneja clicks en el overlay (fondo).
     * Permite cerrar el tutorial haciendo click fuera del contenido.
     */
    function handleOverlayClick(e) {
        // Solo cerrar si se hace click directamente en el overlay (no en el contenido)
        if (e.target === e.currentTarget) {
            closeTutorial();
        }
    }
</script>

<!-- ============================================================================
     OVERLAY Y CONTENIDO DEL TUTORIAL
     ============================================================================
     El overlay cubre toda la pantalla con fondo semitransparente.
     El contenido está centrado y permite interacción con la app detrás.
     ============================================================================ -->
{#if show}
    <div 
        class="tutorial-overlay" 
        on:click={handleOverlayClick}
        role="dialog"
        aria-labelledby="tutorial-title"
        aria-modal="true"
    >
        <div class="tutorial-content" on:click|stopPropagation>
            <!-- Encabezado -->
            <div class="tutorial-header">
                <h2 id="tutorial-title" class="tutorial-title">
                    <i class="fas fa-info-circle"></i>
                    Bienvenido
                </h2>
                <button 
                    class="tutorial-close-btn" 
                    on:click={closeTutorial}
                    aria-label="Cerrar tutorial"
                >
                    <i class="fas fa-times"></i>
                </button>
            </div>
            
            <!-- Contenido principal - Específico por dispositivo -->
            <div class="tutorial-body">
                {#if isMobile}
                    <!-- Contenido para móvil -->
                    <div class="tutorial-item">
                        <i class="fas fa-hand-pointer tutorial-item-icon"></i>
                        <div class="tutorial-item-content">
                            <strong>Zoom:</strong> Separa dos dedos para acercar
                        </div>
                    </div>
                    
                    <div class="tutorial-item">
                        <i class="fas fa-arrows-alt tutorial-item-icon"></i>
                        <div class="tutorial-item-content">
                            <strong>Navegar:</strong> Toca para cambiar de página
                        </div>
                    </div>
                    
                    <div class="tutorial-item">
                        <i class="fas fa-download tutorial-item-icon"></i>
                        <div class="tutorial-item-content">
                            <strong>Descargar:</strong> Toca el botón <i class="fas fa-download"></i> arriba
                        </div>
                    </div>
                {:else}
                    <!-- Contenido para desktop -->
                    <div class="tutorial-item">
                        <i class="fas fa-search-plus tutorial-item-icon"></i>
                        <div class="tutorial-item-content">
                            <strong>Zoom:</strong> Usa los botones <i class="fas fa-search-plus"></i> <i class="fas fa-search-minus"></i> o <kbd>Ctrl</kbd> + rueda
                        </div>
                    </div>
                    
                    <div class="tutorial-item">
                        <i class="fas fa-arrow-circle-left tutorial-item-icon"></i>
                        <div class="tutorial-item-content">
                            <strong>Páginas:</strong> Usa las flechas o haz click en las páginas
                        </div>
                    </div>
                    
                    <div class="tutorial-item">
                        <i class="fas fa-download tutorial-item-icon"></i>
                        <div class="tutorial-item-content">
                            <strong>Descargar:</strong> Click en <i class="fas fa-download"></i> para PDF
                        </div>
                    </div>
                {/if}
            </div>
            
            <!-- Pie de página con botón de cierre -->
            <div class="tutorial-footer">
                <button class="tutorial-close-button" on:click={closeTutorial}>
                    Entendido
                </button>
            </div>
        </div>
    </div>
{/if}

<!-- ============================================================================
     ESTILOS CSS - DISEÑO RESPONSIVE DEL TUTORIAL
     ============================================================================ -->
<style>
    /* ============================================================================
       OVERLAY
       ============================================================================ */
    .tutorial-overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.8);
        backdrop-filter: blur(4px);
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 10000;
        animation: fadeIn 0.3s ease-out;
        padding: 20px;
        box-sizing: border-box;
    }
    
    @keyframes fadeIn {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }
    
    /* ============================================================================
       CONTENIDO DEL TUTORIAL
       ============================================================================ */
    .tutorial-content {
        background: #2c3e50;
        border-radius: 12px;
        box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
        max-width: 500px;
        width: 100%;
        animation: slideUp 0.4s ease-out;
        position: relative;
        display: flex;
        flex-direction: column;
    }
    
    @keyframes slideUp {
        from {
            transform: translateY(30px);
            opacity: 0;
        }
        to {
            transform: translateY(0);
            opacity: 1;
        }
    }
    
    /* ============================================================================
       ENCABEZADO
       ============================================================================ */
    .tutorial-header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 20px;
        border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    }
    
    .tutorial-title {
        margin: 0;
        color: white;
        font-size: 20px;
        font-weight: 600;
        display: flex;
        align-items: center;
        gap: 10px;
    }
    
    .tutorial-title i {
        color: #3498db;
        font-size: 22px;
    }
    
    .tutorial-close-btn {
        background: none;
        border: none;
        color: rgba(255, 255, 255, 0.7);
        font-size: 18px;
        cursor: pointer;
        padding: 6px;
        border-radius: 50%;
        transition: all 0.2s ease;
        display: flex;
        align-items: center;
        justify-content: center;
        width: 32px;
        height: 32px;
    }
    
    .tutorial-close-btn:hover {
        background-color: rgba(255, 255, 255, 0.1);
        color: white;
    }
    
    /* ============================================================================
       CUERPO DEL TUTORIAL
       ============================================================================ */
    .tutorial-body {
        padding: 20px;
        display: flex;
        flex-direction: column;
        gap: 16px;
    }
    
    /* ============================================================================
       ITEMS DEL TUTORIAL
       ============================================================================ */
    .tutorial-item {
        display: flex;
        align-items: flex-start;
        gap: 12px;
        color: white;
    }
    
    .tutorial-item-icon {
        font-size: 20px;
        color: #3498db;
        margin-top: 2px;
        flex-shrink: 0;
        width: 24px;
        text-align: center;
    }
    
    .tutorial-item-icon.fa-download {
        color: #4CAF50;
    }
    
    .tutorial-item-content {
        flex: 1;
        font-size: 14px;
        line-height: 1.5;
        color: rgba(255, 255, 255, 0.9);
    }
    
    .tutorial-item-content strong {
        color: white;
        font-weight: 600;
    }
    
    .tutorial-item-content i {
        color: #4CAF50;
        margin: 0 2px;
    }
    
    .tutorial-item-content kbd {
        background-color: rgba(0, 0, 0, 0.3);
        border: 1px solid rgba(255, 255, 255, 0.2);
        border-radius: 3px;
        padding: 2px 6px;
        font-family: monospace;
        font-size: 11px;
        color: white;
        margin: 0 2px;
    }
    
    /* ============================================================================
       PIE DE PÁGINA
       ============================================================================ */
    .tutorial-footer {
        padding: 16px 20px;
        border-top: 1px solid rgba(255, 255, 255, 0.1);
        display: flex;
        justify-content: center;
    }
    
    .tutorial-close-button {
        background: #3498db;
        border: none;
        color: white;
        font-size: 15px;
        font-weight: 600;
        padding: 10px 32px;
        border-radius: 6px;
        cursor: pointer;
        transition: all 0.2s ease;
        width: 100%;
    }
    
    .tutorial-close-button:hover {
        background: #2980b9;
    }
    
    .tutorial-close-button:active {
        transform: scale(0.98);
    }
    
    /* ============================================================================
       MEDIA QUERIES - DISEÑO RESPONSIVE PARA MÓVIL
       ============================================================================ */
    @media (max-width: 768px) {
        .tutorial-overlay {
            padding: 15px;
        }
        
        .tutorial-content {
            max-width: 100%;
            border-radius: 10px;
        }
        
        .tutorial-header {
            padding: 16px;
        }
        
        .tutorial-title {
            font-size: 18px;
        }
        
        .tutorial-title i {
            font-size: 20px;
        }
        
        .tutorial-body {
            padding: 16px;
            gap: 14px;
        }
        
        .tutorial-item {
            gap: 10px;
        }
        
        .tutorial-item-icon {
            font-size: 18px;
            width: 22px;
        }
        
        .tutorial-item-content {
            font-size: 13px;
        }
        
        .tutorial-footer {
            padding: 14px 16px;
        }
        
        .tutorial-close-button {
            font-size: 14px;
            padding: 12px 24px;
        }
    }
</style>
