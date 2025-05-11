<template>
    <nav class="navbar navbar-expand-lg navbar-dark" aria-label="Twelfth navbar example">
        <div class="container-fluid">
            <!-- Logo visible solo en móviles -->
            <a class="nav-link d-lg-none mobile-logo" aria-current="page" href="/">
                <img class="me-2" src="/assets/images/logo.png" alt="logo" width="30" height="50">
                Machina<span style="color: #FFC72C;">Synth</span>Labs<span style="color: #C8102E;">X</span>
            </a>
            
            <button class="navbar-toggler collapsed" type="button" data-bs-toggle="collapse"
                data-bs-target="#navbarsExample10" aria-controls="navbarsExample10" aria-expanded="false"
                aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>

            <div class="navbar-collapse justify-content-md-center collapse" id="navbarsExample10">
                <ul class="navbar-nav">
                    <!-- Logo visible solo en desktop -->
                    <li class="nav-item d-none d-lg-block">
                        <a class="nav-link" aria-current="page" href="/">
                            <img class="me-3" src="/assets/images/logo.png" alt="logo" width="30" height="50">
                            Machina<span style="color: #FFC72C;">Synth</span>Labs<span style="color: #C8102E;">X</span>
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="/">Inicio</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="/dashboard">Dashboard</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="/home" tabindex="-1" aria-disabled="true">Simuladores</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="/registro_simuladores" tabindex="-1" aria-disabled="true">Expande la colección</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
</template>

<style scoped>
    .navbar {
        background-color: #002147;
        border-bottom: 1px solid #FFC72C;
        position: relative;
    }

    a {
        color: #fff;
    }
    
    a:hover {
        color: #fff;
    }

    /* Estilos para pantallas grandes (horizontal) */
    @media (min-width: 992px) {
        .navbar-nav {
            flex-direction: row;
        }
        
        .nav-item {
            display: flex;
            align-items: center;
            margin-right: 15px;
        }
        
        .nav-link {
            padding: 0.5rem 1rem;
            white-space: nowrap;
        }
    }

    /* Estilos para pantallas pequeñas (menú lateral izquierdo) */
    @media (max-width: 991.98px) {
        .navbar-collapse {
            position: fixed;
            top: 0;
            left: 0;
            bottom: 0;
            width: 250px;
            padding: 20px;
            background-color: #002147;
            border-right: 1px solid #FFC72C;
            z-index: 1050;
            transform: translateX(-100%);
            transition: transform 0.3s ease-in-out;
        }
        
        .navbar-collapse.show {
            transform: translateX(0);
        }
        
        .navbar-nav {
            flex-direction: column;
            height: 100%;
            padding-top: 60px; /* Espacio para el logo móvil */
        }
        
        .nav-item {
            display: block;
            margin-right: 0;
            margin-bottom: 15px;
        }
        
        .nav-link {
            padding: 0.8rem 1rem;
            display: block;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
        }
        
        /* Logo en móviles */
        .mobile-logo {
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            white-space: nowrap;
            display: flex;
            align-items: center;
            padding: 0.5rem 0;
        }
        
        /* Botón de toggle */
        .navbar-toggler {
            position: relative;
            z-index: 1051;
            order: -1; /* Lo coloca a la izquierda */
        }
        
        /* Overlay para el fondo cuando el menú está abierto */
        .navbar-collapse::before {
            content: '';
            position: fixed;
            top: 0;
            right: 0;
            bottom: 0;
            left: 250px;
            background-color: rgba(0, 0, 0, 0.5);
            z-index: 1049;
            display: none;
        }
        
        .navbar-collapse.show::before {
            display: block;
        }
    }
</style>