<?php
session_start();
date_default_timezone_set('America/Mexico_City');
 
 
$db = new SQLite3('transport.db');

?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SISTEMA STC | LIVE</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        :root {--bg: #e98b18;--panel: #f1ece7;--accent: #3b82f6;--success: #10b981;--warning: #f59e0b;--text: #e2e8f0;}
        body {font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; background-color: var(--bg); color: var(--text); margin: 0; overflow-x: hidden; -webkit-text-size-adjust: 100%; text-size-adjust: 100%;}
 
        header {background: var(--panel); padding: 1rem 3rem; display: flex; justify-content: space-between; align-items: center; border-bottom: 3px solid var(--accent); box-shadow: 0 4px 20px rgba(0,0,0,0.5);}
        .brand { font-size: 1.5rem; font-weight: 800; letter-spacing: 2px; color: var(--accent); }
        .brand span { color: white; }
        /* NAV SEMÁNTICO */
        nav a {color: var(--text); text-decoration: none; margin-left: 2rem; font-weight: 600; font-size: 0.9rem; transition: 0.3s; text-transform: uppercase;}
        nav a:hover { color: var(--accent); }
        /* DASHBOARD LAYOUT */
        .main-layout {display: grid; grid-template-columns: 1fr 380px; gap: 1.5rem; padding: 2rem; max-width: 1400px; margin: auto;}
        /* SECCIONES SEMÁNTICAS */
        section {background: var(--panel); border-radius: 15px; padding: 1.5rem; box-shadow: 0 8px 32px rgba(239, 232, 232, 0.3);}
        h2 { font-size: 1.1rem; margin-bottom: 1.5rem; display: flex; align-items: center; gap: 10px; color: #1e293b; }
        /* TABLA DE OPERACIONES */
        table { width: 100%; border-collapse: collapse; }
        th { text-align: left; padding: 1rem; color: #64748b; font-size: 0.8rem; text-transform: uppercase; border-bottom: 1px solid #2d3748; }
        td { padding: 1.2rem 1rem; border-bottom: 1px solid #1e293b; font-size: 0.9rem; }
        .status-pill {padding: 4px 12px; border-radius: 6px; font-size: 0.75rem; font-weight: bold; text-transform: uppercase;}
        .status-en-curso { background: rgba(16, 185, 129, 0.1); color: var(--success); border: 1px solid var(--success); }
        .status-proximo { background: rgba(245, 158, 11, 0.1); color: var(--warning); border: 1px solid var(--warning); }
        .status-finalizado { background: rgba(148, 163, 184, 0.1); color: #94a3b8; border: 1px solid #475569; }
        /* MULTIMEDIA PANEL */
        .media-stack {  display: flex;  flex-direction: column; gap: 1.5rem; width: 100%; /* Asegura que el contenedor ocupe el ancho disponible */max-width: 800px; /* Opcional: evita que se estire infinitamente en pantallas gigantes */ margin: 0 auto; /* Centra el contenedor si le pones un max-width */}
        .monitor-box {width: 100%; /* Cambiamos el alto fijo por una proporción (ej. 16:9 que es la de un monitor) */aspect-ratio: 18 / 9; /* Valores mínimos y máximos para proteger el diseño en extremos */min-height: 80px;  max-height: 450px; background: #000; border-radius: 10px;  overflow: hidden;  border: 1px solid #2d3748;/* Tip Pro: Si vas a meter contenido dentro, asegúrate de que se adapte */display: flex; object-fit: cover; /* Si metes imágenes o videos directos */} /* FOOTER SEMÁNTICO */
        footer {text-align: center; padding: 2rem; background: #070b14; color: #475569; font-size: 0.8rem; margin-top: 3rem;}
        .live-indicator {height: 10px; width: 10px; background: red; border-radius: 50%; display: inline-block; margin-right: 5px; animation: pulse 1s infinite;}
        @keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.3; } 100% { opacity: 1; } }
        /* Efecto de fila interactiva */
        .row-clickable { cursor: pointer; transition: 0.2s; }
        .row-clickable:hover { background: rgba(59, 130, 246, 0.1) !important; transform: scale(1.01); }
        /* Animación de flecha */
        .pulse-icon { animation: pulseArrow 1.5s infinite; color: var(--accent); }
        @keyframes pulseArrow { 0% { transform: translateX(0); opacity: 0.5; } 50% { transform: translateX(5px); opacity: 1; } 100% { transform: translateX(0); opacity: 0.5; } }
        /* Detalle de Telemetría */
        .telemetry-panel { background: #0f172a; border-left: 1px solid #1e293b; padding: 20px; position: sticky; top: 80px; height: calc(100vh - 120px); }
        #instruccion-inicial { text-align: center; margin-top: 100px; color: #475569; }
        .video-container { width: 100%; height: 180px; background: #000; border-radius: 8px; margin: 15px 0; position: relative; overflow: hidden; }
        .cam-overlay { position: absolute; top: 5px; left: 5px; color: #ef4444; font-size: 10px; font-weight: bold; background: rgba(0,0,0,0.5); padding: 2px 5px; } 
        /* SIMULACIÓN DE MAPA/PUESTA DE CARRERA */
        .map-simulation { height: 120px; background: #1e293b; border-radius: 8px; margin: 20px 0; position: relative; padding: 40px 20px; }
        .track { height: 6px; background: #334155; border-radius: 3px; position: relative; }
        .progress-bar-path { position: absolute; height: 100%; background: var(--accent); width: 0%; transition: 2s; box-shadow: 0 0 10px var(--accent); }
        .bus-icon-anim { position: absolute; top: -25px; left: 0%; transform: translateX(-50%); font-size: 20px; color: white; transition: 2s; text-shadow: 0 0 10px #000; }
        .marker { position: absolute; top: 15px; font-size: 10px; color: #94a3b8; }
        .marker.start { left: 0; }
        .marker.end { right: 0; }
        /* Grid de Conductores */
        .conductores-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .conductor-card { background: #1e293b; padding: 10px; border-radius: 8px; display: flex; align-items: center; gap: 10px; border: 1px solid transparent; transition: 0.3s; }
        .conductor-card:hover { border-color: var(--accent); }
        .avatar { width: 35px; height: 35px; background: var(--accent); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 12px; }
        .name { font-size: 13px; font-weight: bold; }
        .lic { font-size: 10px; color: #64748b; }
        .status-dot { width: 8px; height: 8px; border-radius: 50%; margin-left: auto; }
        .online { background: #22c55e; box-shadow: 0 0 5px #22c55e; }
        .busy { background: #ef4444; }
        @keyframes pulse-red { 0% { opacity: 1; } 50% { opacity: 0.3; } 100% { opacity: 1; }}
        .conductor-card { transition: transform 0.2s; cursor: default;}
        .conductor-card:hover { transform: translateY(-3px); box-shadow: 0 4px 12px rgba(0,0,0,0.1);}
        .flota-total h2, 
        .conductores-section h2,
        .table-container h2 { font-size: 1.1rem; color: #1e293b; border-left: 4px solid #38bdf8; padding-left: 10px; margin-bottom: 20px;}
        /* Estilo para que el panel no se mueva al bajar la página */
        .main-layout { display: flex; align-items: flex-start; gap: 20px; padding: 20px;}
        .left-column { flex: 1;}
        .telemetry-panel { width: 350px; background: white; border-radius: 12px; box-shadow: 0 10px 25px rgba(232, 216, 216, 0.1); padding: 20px; position: sticky; top: 20px;}
        @keyframes blink { 50% { opacity: 0; } }
        .modal-conductor { display: none; position: fixed; z-index: 999; left: 0; top: 0; width: 100%; height: 100%; background-color: rgba(0,0,0,0.5); backdrop-filter: blur(4px);}
        .modal-content { background-color: #fff; margin: 10% auto; padding: 25px; border-radius: 12px; width: 90%; max-width: 450px; text-align: center; box-shadow: 0 10px 25px rgba(0,0,0,0.2); position: relative;}
        .close-modal { position: absolute; right: 15px; top: 10px; font-size: 24px; cursor: pointer; color: #64748b;}
        .timer-display { font-size: 2rem; font-weight: 800; color: #1e293b; margin: 20px 0; font-family: monospace; background: #f1f5f9; padding: 10px; border-radius: 8px;}
 
        /* ========================================================
           CORRECCIÓN RESPONSIVA PARA EVITAR PELLIZCAR EL ZOOM
           ======================================================== */
        @media (max-width: 992px) {
            header { padding: 1rem; flex-direction: column; gap: 15px; text-align: center; }
            nav a { margin: 0 10px; }
            .main-layout { flex-direction: column; padding: 10px; gap: 20px; }
            .left-column { width: 100%; }
            .telemetry-panel { width: 100% !important; max-width: 100%; position: static !important; height: auto !important; box-sizing: border-box; }
            section { width: 100%; overflow-x: auto; -webkit-overflow-scrolling: touch; box-sizing: border-box; }
            .conductores-grid { grid-template-columns: 1fr; }
        }
 
        /* ========================================================
           TUTORIAL INTERACTIVO - ESTILO APP CDMX
           ======================================================== */
 
        /* Overlay oscuro del tutorial */
        #tutorial-overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 9000; pointer-events: none;}
 
        /* Capa oscura con hueco transparente para el spotlight */
        #tutorial-mask {
            position: absolute;
            top: 0; left: 0;
            width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.75);
            transition: all 0.4s ease;
        }
 
        /* Hueco brillante (spotlight) sobre el elemento resaltado */
        #tutorial-spotlight {
            position: absolute;
            border-radius: 12px;
            box-shadow: 0 0 0 9999px rgba(0,0,0,0.75), 0 0 0 3px #3b82f6, 0 0 20px rgba(59,130,246,0.6);
            transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
            pointer-events: none;
            z-index: 9001;
        }
 
        /* Tarjeta de tooltip del tutorial */
        #tutorial-card {
            position: fixed;
            background: linear-gradient(135deg, #909398 0%, #e6e5e5 100%);
            border: 1px solid #3b82f6;
            border-radius: 16px;
            padding: 20px 24px;
            max-width: 220px;
            width: 90%;
            z-index: 9100;
            pointer-events: all;
            box-shadow: 0 20px 60px rgba(0,0,0,0.8), 0 0 40px rgba(59,130,246,0.2);
            transition: all 0.35s cubic-bezier(0.25, 0.46, 0.45, 0.94);
            font-family: 'Segoe UI', sans-serif;
        }
 
        #tutorial-card .tut-header {
            display: flex;
            align-items: center;
            gap: 10px;
            margin-bottom: 10px;
        }
 
        #tutorial-card .tut-icon {
            width: 25px; height: 25px;
            background: linear-gradient(135deg, #3b83f600, #1d4fd800);
            border-radius: 10px;
            display: flex; align-items: center; justify-content: center;
            font-size: 1.1rem;
            flex-shrink: 0;
            box-shadow: 0 4px 12px rgba(59,130,246,0.4);
        }
 
        #tutorial-card .tut-title {
            font-size: 0.95rem;
            font-weight: 700;
            color: #f1f5f9;
            letter-spacing: 0.5px;
        }
 
        #tutorial-card .tut-subtitle {
            font-size: 0.65rem;
            color: #3b82f6;
            text-transform: uppercase;
            letter-spacing: 1px;
            font-weight: 600;
        }
 
        #tutorial-card .tut-body {
            font-size: 0.85rem;
            color: #000000;
            line-height: 1.6;
            margin-bottom: 1px;
        }
 
        #tutorial-card .tut-tip {
            background: rgba(59, 130, 246, 0.1);
            border-left: 3px solid #3b82f6;
            padding: 8px 12px;
            border-radius: 0 8px 8px 0;
            font-size: 0.78rem;
            color: #000000;
            margin-bottom: 16px;
        }
 
        #tutorial-card .tut-progress {
            display: flex;
            align-items: center;
            gap: 6px;
            margin-bottom: 14px;
        }
 
        #tutorial-card .tut-dot {
            width: 7px; height: 7px;
            border-radius: 50%;
            background: #334155;
            transition: all 0.3s;
        }
 
        #tutorial-card .tut-dot.active {
            background: #3b82f6;
            width: 18px;
            border-radius: 4px;
            box-shadow: 0 0 8px rgba(59,130,246,0.6);
        }
 
        #tutorial-card .tut-dot.done {
            background: #22c55e;
        }
 
        #tutorial-card .tut-step-label {
            font-size: 0.7rem;
            color: #475569;
            margin-left: auto;
        }
 
        .tut-buttons {
            display: flex;
            gap: 10px;
        }
 
        .tut-btn-skip {
            flex: 1;
            padding: 9px;
            background: transparent;
            border: 1px solid #334155;
            border-radius: 8px;
            color: #64748b;
            font-size: 0.8rem;
            cursor: pointer;
            transition: all 0.2s;
            font-weight: 600;
        }
        .tut-btn-skip:hover { border-color: #475569; color: #94a3b8; }
 
        .tut-btn-next {
            flex: 2;
            padding: 9px;
            background: linear-gradient(135deg, #3b82f6, #2563eb);
            border: none;
            border-radius: 8px;
            color: white;
            font-size: 0.85rem;
            cursor: pointer;
            font-weight: 700;
            transition: all 0.2s;
            box-shadow: 0 4px 12px rgba(59,130,246,0.4);
        }
        .tut-btn-next:hover { transform: translateY(-1px); box-shadow: 0 6px 18px rgba(59,130,246,0.5); }
 
        /* Botón flotante para abrir el tutorial */
        #btn-abrir-tutorial {
            position: fixed;
            bottom: 25px;
            right: 25px;
            z-index: 8000;
            width: 52px; height: 52px;
            background: linear-gradient(135deg, #3b82f6, #1d4fd8be);
            border-radius: 50%;
            border: none;
            color: white;
            font-size: 1.3rem;
            cursor: pointer;
            box-shadow: 0 4px 20px rgba(59,130,246,0.5);
            display: flex; align-items: center; justify-content: center;
            transition: all 0.3s;
            animation: tutPulse 3s infinite;
        }
/* Tu estilo original del botón  se queda igual */
#btn-abrir {
    position: fixed;
    bottom: 90px;
    right: 25px;
    z-index: 8000;
    width: 52px; height: 52px;
    background: linear-gradient(135deg, #3b82f6, #1d4fd8be);
    border-radius: 50%;
    border: none;
    color: white;
    font-size: 1.3rem;
    cursor: pointer;
    box-shadow: 0 4px 20px rgba(59,130,246,0.5);
    display: flex; align-items: center; justify-content: center;
    transition: all 0.3s;
    animation: tutPulse 3s infinite;
}

/* --- NUEVOS ESTILOS PARA LA SECCIÓN --- */

/* Oculta la sección por defecto */
/* Oculta la sección por defecto */
.report-section {
    display: none; 
}

/* Cuando se activa, se vuelve pantalla completa y tapa todo */
.report-section.active {
    display: block; /* O 'flex' si quieres centrar el contenido */
    
    /* Propiedades mágicas para pantalla completa */
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    
    /* Un z-index más alto que el botón para que quede por encima de todo */
    z-index: 9000; 
    
    /* Elige el fondo: blanco total o un color oscuro */
    background-color: #ffffff; 
    /* Si prefieres fondo oscuro tipo app: background-color: #111827; color: white; */

    /* Para que el usuario pueda hacer scroll si el formulario es largo */
    overflow-y: auto; 
    
    /* Un pequeño espacio interno para que no pegue a las orillas */
    padding: 20px; 
    box-sizing: border-box;
}
/* mapaaaa */
/* Tu estilo original para el botón */
#btn-abrir-mapa {
    position: fixed;
    bottom: 155px;
    right: 25px;
    z-index: 8000;
    width: 52px; height: 52px;
    background: linear-gradient(135deg, #3b82f6, #1d4fd8be);
    border-radius: 50%;
    border: none;
    color: white;
    font-size: 1.3rem;
    cursor: pointer;
    box-shadow: 0 4px 20px rgba(59,130,246,0.5);
    display: flex; align-items: center; justify-content: center;
    transition: all 0.3s;
    animation: tutPulse 3s infinite;
}

/* --- CLASES NUEVAS PARA EL CONTENEDOR --- */

/* Oculto por defecto */
.fullscreen-modal {
    display: none;
}

/* Pantalla completa activa que tapa todo */
.fullscreen-modal.active {
    display: block;
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: 9000; /* Por encima de todo */
    background-color: #111827; /* Fondo oscuro tipo app */
    overflow-y: auto; /* Permite scroll vertical para ver los 4 mapas */
}
        @keyframes tutPulse {
            0%, 100% { box-shadow: 0 4px 20px rgba(59,130,246,0.5); }
            50% { box-shadow: 0 4px 35px rgba(59,130,246,0.9), 0 0 0 8px rgba(59,130,246,0.15); }
        }
        #btn-abrir-tutorial:hover { transform: scale(1.1); animation: none; box-shadow: 0 6px 25px rgba(59,130,246,0.7); }
 
        /* Pantalla de bienvenida del tutorial */
        #tutorial-welcome {
            display: none;
            position: fixed;
            top: 0; left: 0;
            width: 100%; height: 100%;
            background: rgba(0,0,0,0.85);
            z-index: 9500;
            align-items: center;
            justify-content: center;
            backdrop-filter: blur(6px);
        }
 
        .welcome-card {
            background: linear-gradient(135deg, #909398 0%, #e6e5e5 100%);
            border: 1px solid #3b82f6;
            border-radius: 20px;
            padding: 40px 35px;
            max-width:420px;
            width: 90%;
            text-align: center;
            box-shadow: 0 30px 80px rgba(0,0,0,0.8), 0 0 60px rgba(59,130,246,0.15);
            animation: welcomeIn 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94);
        }
        @keyframes welcomeIn {
            from { transform: scale(0.85) translateY(30px); opacity: 0; }
            to { transform: scale(1) translateY(0); opacity: 1; }
        }
 
        .welcome-logo {
            width: 15px; height: 15px;
            background: linear-gradient(135deg, #ffffff, #ffffff);
            border-radius: 18px;
            margin: 0 auto 20px;
            display: flex; align-items: center; justify-content: center;
            font-size: 2rem;
            box-shadow: 0 8px 25px rgba(59,130,246,0.5);
        }
 
        .welcome-title {
            font-size: 1.4rem;
            font-weight: 800;
            color: #f1f5f9;
            margin-bottom: 8px;
            letter-spacing: 1px;
        }
 
        .welcome-subtitle {
            font-size: 0.85rem;
            color: #000000;
            margin-bottom: 10px;
            line-height: 1.5;
        }
 
        .welcome-features {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-bottom: 14px;
            text-align: left;
        }
 
        .welcome-feat {
            background: rgba(59,130,246,0.08);
            border: 1px solid rgba(59,130,246,0.2);
            border-radius: 10px;
            padding: 10px 12px;
            font-size: 0.78rem;
            color: #000000;
            display: flex;
            align-items: center;
            gap: 8px;
        }
 
        .welcome-feat i { color: #3b82f6; font-size: 0.9rem; }
 
        .welcome-btn-start {
            width: 100%;
            padding: 14px;
            background: linear-gradient(135deg, #3b82f6, #2563eb);
            border: none;
            border-radius: 12px;
            color: white;
            font-size: 1rem;
            font-weight: 700;
            cursor: pointer;
            letter-spacing: 0.5px;
            box-shadow: 0 6px 20px rgba(59,130,246,0.4);
            transition: all 0.2s;
            margin-bottom: 12px;
        }
        .welcome-btn-start:hover { transform: translateY(-2px); box-shadow: 0 10px 30px rgba(59,130,246,0.5); }
 
        .welcome-btn-skip {
            width: 100%;
            padding: 10px;
            background: transparent;
            border: 1px solid #334155;
            border-radius: 10px;
            color: #475569;
            font-size: 0.85rem;
            cursor: pointer;
            transition: all 0.2s;
        }
        .welcome-btn-skip:hover { border-color: #475569; color: #64748b; }
 
        /* Resaltado con glow en el elemento activo del tutorial */
        .tutorial-highlight {
            position: relative;
            z-index: 9050 !important;
        }
 
        /* ================================================================
           ESTILOS NUEVOS: LÍNEAS DE METRO Y METROBÚS
           ================================================================ */
        @keyframes parpadeo {
            from { opacity: 0.2; box-shadow: 0 0 0 0 rgba(0,255,0,0.7); }
            to   { opacity: 1;   box-shadow: 0 0 8px 4px rgba(0,255,0,0); }
        }
 
        /* Tabs de líneas */
        .lineas-tabs {
            display: flex;
            flex-wrap: wrap;
            gap: 6px;
            margin-bottom: 14px;
        }
        .linea-tab {
            padding: 5px 12px;
            border-radius: 20px;
            font-size: 0.72rem;
            font-weight: 800;
            cursor: pointer;
            border: 2px solid transparent;
            transition: all 0.2s;
            color: white;
            letter-spacing: 0.5px;
        }
        .linea-tab:hover { transform: translateY(-2px); filter: brightness(1.15); }
        .linea-tab.activa { border-color: white; box-shadow: 0 0 12px rgba(255,255,255,0.4); transform: translateY(-2px); }
 
        /* Panel de estaciones */
        .panel-linea {
            display: none;
            animation: fadeSlide 0.3s ease;
        }
        .panel-linea.visible { display: block; }
        @keyframes fadeSlide {
            from { opacity: 0; transform: translateY(8px); }
            to   { opacity: 1; transform: translateY(0); }
        }
 
        /* Simulador de trenes */
        .sim-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }
        .sim-live-badge {
            display: flex;
            align-items: center;
            gap: 6px;
            background: rgba(0,255,0,0.1);
            border: 1px solid #22c55e;
            border-radius: 20px;
            padding: 3px 10px;
            font-size: 0.65rem;
            color: #22c55e;
            font-weight: 700;
            letter-spacing: 1px;
        }
        .sim-dot {
            width: 7px; height: 7px;
            background: #22c55e;
            border-radius: 50%;
            animation: parpadeo 1s infinite alternate;
        }
 
        /* Track de estaciones */
        .estaciones-track {
            overflow-x: auto;
            padding: 18px 0 10px;
            scrollbar-width: thin;
            scrollbar-color: #dce3ed transparent;
        }
        .estaciones-track::-webkit-scrollbar { height: 4px; }
        .estaciones-track::-webkit-scrollbar-thumb { background: #334155; border-radius: 2px; }
 
        .estaciones-row {
            display: flex;
            align-items: flex-end;
            min-width: max-content;
            padding: 0 10px;
            position: relative;
        }
 
        /* Línea horizontal del track */
        .track-line {
            position: absolute;
            top: 28px;
            left: 30px;
            right: 30px;
            height: 5px;
            border-radius: 3px;
            z-index: 0;
        }
 
        .estacion-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 72px;
            flex-shrink: 0;
            position: relative;
            z-index: 1;
            cursor: pointer;
            transition: transform 0.2s;
        }
        .estacion-item:hover { transform: translateY(-4px); }
 
        .estacion-dot {
            width: 14px; height: 14px;
            border-radius: 50%;
            border: 3px solid white;
            background: #1e293b;
            transition: all 0.3s;
            z-index: 2;
            box-shadow: 0 2px 6px rgba(0,0,0,0.4);
        }
        .estacion-dot.tiene-tren {
            background: white;
            box-shadow: 0 0 10px rgba(255,255,255,0.8), 0 0 20px rgba(255,255,255,0.4);
            animation: trenpulse 0.8s infinite alternate;
        }
        @keyframes trenpulse {
            from { box-shadow: 0 0 6px rgba(255,255,255,0.6); }
            to   { box-shadow: 0 0 18px rgba(255,255,255,1), 0 0 30px rgba(255,255,255,0.5); }
        }
 
        .estacion-nombre {
            font-size: 0.55rem;
            margin-top: 6px;
            text-align: center;
            color: #cbd5e1;
            max-width: 68px;
            line-height: 1.2;
            font-weight: 600;
        }
 
        /* Ícono tren encima del dot */
        .tren-icon {
            position: absolute;
            top: -18px;
            font-size: 0.9rem;
            animation: trenmove 0.5s ease;
            filter: drop-shadow(0 0 4px rgba(255,255,255,0.8));
        }
        @keyframes trenmove {
            from { opacity: 0; transform: translateX(-10px); }
            to   { opacity: 1; transform: translateX(0); }
        }
 
        /* Stats de línea */
        .linea-stats {
            display: flex;
            gap: 10px;
            margin-top: 12px;
            flex-wrap: wrap;
        }
        .stat-chip {
            background: rgba(237, 14, 14, 0.06);
            border: 1px solid rgba(255,255,255,0.12);
            border-radius: 8px;
            padding: 6px 12px;
            font-size: 0.7rem;
            color: #94a3b8;
            display: flex;
            align-items: center;
            gap: 5px;
        }
        .stat-chip b { color: #f1f5f9; }
 
        /* Modal de estación */
        #modal-estacion {
            display: none;
            position: fixed;
            z-index: 9999;
            left: 0; top: 0;
            width: 100%; height: 100%;
            background: rgba(0,0,0,0.7);
            backdrop-filter: blur(6px);
            align-items: center;
            justify-content: center;
        }
        #modal-estacion.abierto { display: flex; }
        .modal-estacion-card {background: linear-gradient(135deg, #4b6082 0%, #0f172a 100%);border-radius: 20px;padding: 28px;max-width: 480px;width: 92%;position: relative;box-shadow: 0 30px 80px rgba(0,0,0,0.8);animation: welcomeIn 0.3s ease; }
        .modal-est-header {display: flex;align-items: center;gap: 14px;margin-bottom: 18px;}
        .modal-est-dot {
            width: 44px; height: 44px;
            border-radius: 50%;
            display: flex; align-items: center; justify-content: center;
            font-size: 1.3rem;
            font-weight: 900;
            color: white;
            flex-shrink: 0;
        }
        .modal-est-info h3 { margin: 0; color: #f1f5f9; font-size: 1.1rem; }
        .modal-est-info small { color: #64748b; font-size: 0.75rem; }
        .modal-est-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-bottom: 16px;
        }
        .modal-est-cell {
            background: rgba(255,255,255,0.04);
            border: 1px solid rgba(224, 2, 2, 0.08);
            border-radius: 10px;
            padding: 10px 12px;
        }
        .modal-est-cell span { font-size: 0.65rem; color: #64748b; text-transform: uppercase; display: block; margin-bottom: 3px; }
        .modal-est-cell b { font-size: 0.9rem; color: #f1f5f9; }
        .modal-est-prox {
            background: rgba(201, 210, 225, 0.08);
            border: 1px solid rgba(59,130,246,0.2);
            border-radius: 10px;
            padding: 12px;
            font-size: 0.8rem;
            color: #94a3b8;
        }
        .tren-en-camino {
            display: inline-flex;
            align-items: center;
            gap: 6px;
            background: rgba(34,197,94,0.12);
            border: 1px solid #22c55e;
            border-radius: 20px;
            padding: 4px 10px;
            font-size: 0.7rem;
            color: #22c55e;
            font-weight: 700;
            margin-top: 8px;
        }
        .close-est { position: absolute; top: 14px; right: 16px; background: none; border: none; color: #ed0909; font-size: 1.3rem; cursor: pointer; transition: color 0.2s; }
        .close-est:hover { color: #f1f5f9; }
 
        /* Tabla de líneas en la sección EN CURSO */
        .metro-lineas-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(130px, 1fr));
            gap: 10px;
            margin-top: 10px;
        }
        .metro-linea-card {
            border-radius: 10px;
            padding: 12px 10px;
            text-align: center;
            cursor: pointer;
            transition: all 0.2s;
            border: 2px solid transparent;
            position: relative;
            overflow: hidden;
        }
        .metro-linea-card:hover { transform: translateY(-3px); filter: brightness(1.1); border-color: rgba(255,255,255,0.4); }
        .metro-linea-card .num { font-size: 1.6rem; font-weight: 900; color: white; display: block; line-height: 1; }
        .metro-linea-card .nom { font-size: 0.6rem; color: rgba(255,255,255,0.8); margin-top: 4px; font-weight: 600; text-transform: uppercase; }
        .metro-linea-card .est-count { font-size: 0.55rem; color: rgba(255,255,255,0.6); margin-top: 2px; }
        .metro-linea-card .trenes-badge {
            position: absolute;
            top: 6px; right: 6px;
            background: rgba(238, 10, 10, 0.3);
            border-radius: 10px;
            padding: 1px 6px;
            font-size: 0.6rem;
            color: rgba(255,255,255,0.9);
            font-weight: 700;
        }
 
        /* Sección Metrobús viajes disponibles */
        .mb-lineas-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(130px, 1fr));
            gap: 10px;
            margin-top: 10px;
        }
        .mb-linea-card {
            border-radius: 10px;
            padding: 12px 10px;
            text-align: center;
            cursor: pointer;
            transition: all 0.2s;
            border: 2px solid transparent;
        }
        .mb-linea-card:hover { transform: translateY(-3px); filter: brightness(1.1); border-color: rgba(255,255,255,0.4); }
        .mb-linea-card .num { font-size: 1.6rem; font-weight: 900; color: white; }
        .mb-linea-card .nom { font-size: 0.58rem; color: rgba(255,255,255,0.8); margin-top: 4px; font-weight: 600; text-transform: uppercase; }
        .mb-linea-card .est-count { font-size: 0.55rem; color: rgba(255,255,255,0.6); }
 
        /* Panel expandible de estaciones (modal general) */
        #modal-linea-detalle {
            display: none;
            position: fixed;
            z-index: 9998;
            left: 0; top: 0;
            width: 100%; height: 100%;
            background: rgba(243, 228, 228, 0.6);
            backdrop-filter: blur(8px);
            align-items: center;
            justify-content: center;
        }
        #modal-linea-detalle.abierto { display: flex; }
        .modal-linea-card {
            background: #ffffffc1;
            border-radius: 20px;
            padding: 24px;
            max-width: 860px;
            width: 100%;
            max-height: 85vh;
            overflow-y: auto;
 position: relative; box-shadow: 0 30px 80px rgba(234, 222, 222, 0.65); animation: welcomeIn 0.3s ease;
        }
        .modal-linea-card::-webkit-scrollbar { width: 4px; }
        .modal-linea-card::-webkit-scrollbar-thumb { background: #e7ecf1; border-radius: 2px; }

        /* Estilos para el Footer AVISO DE PRIV */
footer {
    background-color: #222;
    color: #fff;
    padding: 15px;
    text-align: center;
    font-family: Arial, sans-serif;
}
footer a {
    color: #3498db;
    text-decoration: none;
    margin-left: 10px;
}
/* El contenedor de fondo (Modal) */
.modal {
    display: none; /* Se activa con JS */
    position: fixed;
    z-index: 9999; /* Asegura que se ponga por encima de todo */
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0, 0, 0, 0.75); /* Fondo oscuro semitransparente premium */
    backdrop-filter: blur(4px); /* Efecto difuminado de fondo elegante */
}

/* El Recuadro del Contenido (Caja Flotante) */
.modal-content {
    background-color: #121212; /* Negro oscuro mate estilizado (Modo oscuro puro) */
    color: #e0e0e0; /* Texto gris claro para alto contraste y lectura cómoda */
    margin: 5% auto; /* Centrado vertical y horizontal */
    padding: 30px;
    border: 1px solid #222222; /* Borde sutil sutil para dar profundidad */
    width: 80%;
    max-width: 700px; /* Ancho máximo para que no se desarme en PC */
    border-radius: 12px; /* Esquinas redondeadas modernas */
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5); /* Sombra pesada para destacar */
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    line-height: 1.6;
}

/* Título Principal */
.modal-content h2 {
    color: #ffffff;
    font-size: 24px;
    margin-top: 0;
    border-bottom: 2px solid #A31D1D; /* Línea de acento roja que combina con transporte */
    padding-bottom: 10px;
}

/* El botón de cerrar (X) */
.close-btn {color: #888888;float: right;font-size: 30px;font-weight: bold;cursor: pointer;transition: color 0.2s ease;line-height: 20px;}
/* Comportamiento al pasar el mouse por la X */
.close-btn:hover,
.close-btn:focus {color: #ffffff;text-decoration: none;}
/* Texto interno del Aviso */
.modal-content p {font-size: 14px;color: #b3b3b3;text-align: justify;white-space: pre-line; /* Mantiene los saltos de línea de tu texto */}
/* Resaltados dentro del texto */
.modal-content strong {color: #ffffff; /* Texto en negrita completamente blanco para resaltar */}
/* ===== SIMULADOR DE DOBLE VÍA ===== */
        .sim-doble-via { background: #121212; border-radius: 12px; padding: 14px 8px 10px; margin-top: 10px; }
        .sim-via-label { font-size: 0.62rem; font-weight: 700; letter-spacing: 1px; color: #ffffff; text-transform: uppercase; padding: 0 12px 4px; display: flex; align-items: center; gap: 6px; }
        .sim-via-label .via-arrow { font-size: 0.7rem; }
        .via-separator { height: 1px; background: #1e293b; margin: 12px 8px; }
        .estaciones-track { overflow-x: auto; padding: 8px 0 4px; }
        .estaciones-row { display: flex; align-items: flex-end; min-width: max-content; padding: 0 10px; position: relative; }
        .track-line { position: absolute; top: 22px; left: 30px; right: 30px; height: 5px; border-radius: 3px; z-index: 0; }
        .estacion-item { display: flex; flex-direction: column; align-items: center; width: 72px; flex-shrink: 0; position: relative; z-index: 1; cursor: pointer; transition: transform 0.2s; padding-top: 48px; }
        .estacion-item:hover { transform: translateY(-4px); }

        /* Estación cerrada en el track */
        .estacion-item.cerrada .estacion-dot { background: #ef4444 !important; border-color: #ef4444 !important; box-shadow: 0 0 8px rgba(239,68,68,0.6); }
        .estacion-item.cerrada .estacion-nombre { color: #ef4444; }
        .estacion-item.cerrada::before { content: '🚫'; position: absolute; top: 6px; font-size: 0.7rem; left: 50%; transform: translateX(-50%); }

        .est-logo { width: 26px; height: 26px; object-fit: contain; display: block; margin: 0 auto 4px; image-rendering: auto; }
        .estacion-dot { width: 14px; height: 14px; border-radius: 50%; border: 3px solid white; background: #1e293b; z-index: 2; box-shadow: 0 2px 6px rgba(0,0,0,0.4); transition: all 0.3s; flex-shrink: 0; }
        .estacion-dot.tiene-tren { background: white; box-shadow: 0 0 10px rgba(255,255,255,0.8), 0 0 20px rgba(255,255,255,0.4); animation: trenpulse 0.8s infinite alternate; }
        @keyframes trenpulse { from { box-shadow: 0 0 6px rgba(255,255,255,0.6); } to { box-shadow: 0 0 18px rgba(255,255,255,1), 0 0 30px rgba(255,255,255,0.5); } }
        .estacion-nombre { font-size: 0.52rem; margin-top: 5px; text-align: center; color: #cbd5e1; max-width: 68px; line-height: 1.2; font-weight: 600; }
        .tren-emoji-encima { position: absolute; font-size: 1rem; top: 10px; left: 50%; transform: translateX(-50%); z-index: 5; filter: drop-shadow(0 0 4px rgba(255,255,255,0.9)); pointer-events: none; animation: trenAparece 0.4s ease; }
        @keyframes trenAparece { from { opacity:0; transform:translateX(-50%) scale(0.5); } to { opacity:1; transform:translateX(-50%) scale(1); } }
        .via-progress-wrapper { position: relative; height: 5px; border-radius: 3px; overflow: visible; }
        .tren-en-tramo { position: absolute; top: 0; left: 0; height: 100%; background: rgba(255,255,255,0.9); border-radius: 3px; transition: width 5s linear; min-width: 4px; }
        .tren-emoji-tramo { position: absolute; top: -10px; font-size: 0.85rem; transform: translateX(-50%); z-index: 4; pointer-events: none; filter: drop-shadow(0 0 3px rgba(255,255,255,0.8)); transition: left 5s linear; }
    

/*------------- METRO1 ------------------*/
/* Contenedor del menú (Aquí se coloca la marca de agua fija) */
.dropdown-menu-container {
    position: relative;
    overflow: hidden;
    transition: all 0.4s ease-in-out;
    
    /* Movemos la marca de agua aquí para que sea visible desde el inicio */
    background-image: url('metr.png'); 
    background-position: center;
    background-repeat: no-repeat;
    background-size: 100% 160%;
}
/*------------- METROBUS2 ------------------*/
.dropdown-menu-container2 {
    position: relative;
    overflow: hidden;
    transition: all 0.4s ease-in-out;
    
    /* Movemos la marca de agua aquí para que sea visible desde el inicio */
    background-image: url('mtb.png'); 
    background-position: center;
    background-repeat: no-repeat;
    background-size: 100% 120%;
}
/*------------- CABLEBUS3 ------------------*/
.dropdown-menu-container3 {
    position: relative;
    overflow: hidden;
    transition: all 0.4s ease-in-out;
    
    /* Movemos la marca de agua aquí para que sea visible desde el inicio */
    background-image: url('cbb.png'); 
    background-position: center;
    background-repeat: no-repeat;
    background-size: 50% 100%;
}
/*------------- TROLEBUS4 ------------------*/
.dropdown-menu-container4 {
    position: relative;
    overflow: hidden;
    transition: all 0.4s ease-in-out;
    
    /* Movemos la marca de agua aquí para que sea visible desde el inicio */
    background-image: url('tlb.png'); 
    background-position: center;
    background-repeat: no-repeat;
    background-size: 100% 155%;
}
/*------------- RTP5 ------------------*/
.dropdown-menu-container5 {
    position: relative;
    overflow: hidden;
    transition: all 0.4s ease-in-out;
    
    /* Movemos la marca de agua aquí para que sea visible desde el inicio */
    background-image: url('rtt.png'); 
    background-position: center;
    background-repeat: no-repeat;
    background-size: 100% 130%;
}
/*------------- METRO ------------------*/
/* Máscara opaca sobre el contenedor principal */
.dropdown-menu-container::before {
    content: '';
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(255, 255, 255, 0.85); /* Cambia a negro si tu web es oscura */
    z-index: 1;
    transition: opacity 0.4s ease;
    pointer-events: none;
}
/*------------- METROBUS ------------------*/
/* Máscara opaca sobre el contenedor principal */
.dropdown-menu-container2::before {
    content: '';
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(255, 255, 255, 0.85); /* Cambia a negro si tu web es oscura */
    z-index: 1;
    transition: opacity 0.4s ease;
    pointer-events: none;
}
/*------------- CABLEBUS ------------------*/
/* Máscara opaca sobre el contenedor principal */
.dropdown-menu-container3::before {
    content: '';
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(255, 255, 255, 0.85); /* Cambia a negro si tu web es oscura */
    z-index: 1;
    transition: opacity 0.4s ease;
    pointer-events: none;
}
/*------------- CABLEBUS ------------------*/
/* Máscara opaca sobre el contenedor principal */
.dropdown-menu-container4::before {
    content: '';
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(255, 255, 255, 0.85); /* Cambia a negro si tu web es oscura */
    z-index: 1;
    transition: opacity 0.4s ease;
    pointer-events: none;
}
/*------------- RTP ------------------*/
/* Máscara opaca sobre el contenedor principal */
.dropdown-menu-container5::before {
    content: '';
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(255, 255, 255, 0.85); /* Cambia a negro si tu web es oscura */
    z-index: 1;
    transition: opacity 0.4s ease;
    pointer-events: none;
}
/* Encabezado interactivo (Subimos el z-index para que no quede abajo de la máscara) */
.menu-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
    padding: 10px 0;
    user-select: none;
    position: relative;
    z-index: 2; 
}

.arrow-icon {
    transition: transform 0.3s ease;
    color: #64748b;
}

/* CONTENIDO OCULTO */
.menu-content {
    max-height: 0; /* Totalmente oculto al inicio */
    opacity: 0;
    visibility: hidden;
    transition: max-height 0.5s ease, opacity 0.4s ease, visibility 0.4s;
    position: relative;
    z-index: 2; /* Asegura que tus cuadros queden por encima del fondo */
}



/* -------------*/

/* ===================================================
   ACCIONES: Cuando pasas el mouse o tiene la clase .open
   =================================================== */
/*------------- METRO ------------------*/
/*------------- METRO ------------------*/
/*------------- METRO ------------------*/
/* Desplegar el contenido */
.dropdown-menu-container:hover .menu-content,
.dropdown-menu-container.open .menu-content {
    max-height: 800px; 
    opacity: 1;
    visibility: visible;
}

/* Quitamos la marca de agua y la máscara al abrirse */
.dropdown-menu-container:hover,
.dropdown-menu-container.open {
    background-image: none; /* Desaparece la imagen */
}

.dropdown-menu-container:hover::before,
.dropdown-menu-container.open::before {
    opacity: 0; /* Desaparece la máscara blanca */
}

/* Rotar la flecha */
.dropdown-menu-container:hover .arrow-icon,
.dropdown-menu-container.open .arrow-icon {
    transform: rotate(180deg);
    
}
/*------------- METROBUS ------------------*/
/*------------- METROBUS ------------------*/
/*------------- METROBUS ------------------*/

.dropdown-menu-container2:hover .menu-content,
.dropdown-menu-container2.open .menu-content {
    max-height: 800px; 
    opacity: 1;
    visibility: visible;
}

/* Quitamos la marca de agua y la máscara al abrirse */
.dropdown-menu-container2:hover,
.dropdown-menu-container2.open {
    background-image: none; /* Desaparece la imagen */
}

.dropdown-menu-container2:hover::before,
.dropdown-menu-container2.open::before {
    opacity: 0; /* Desaparece la máscara blanca */
}

/* Rotar la flecha */
.dropdown-menu-container2:hover .arrow-icon,
.dropdown-menu-container2.open .arrow-icon {
    transform: rotate(180deg);
    
}
/*------------- CABLEBUS ------------------*/
/*------------- CABLEBUS ------------------*/
/*------------- CABLEBUS ------------------*/

.dropdown-menu-container3:hover .menu-content,
.dropdown-menu-container3.open .menu-content {
    max-height: 800px; 
    opacity: 1;
    visibility: visible;
}

/* Quitamos la marca de agua y la máscara al abrirse */
.dropdown-menu-container3:hover,
.dropdown-menu-container3.open {
    background-image: none; /* Desaparece la imagen */
}

.dropdown-menu-container3:hover::before,
.dropdown-menu-container3.open::before {
    opacity: 0; /* Desaparece la máscara blanca */
}

/* Rotar la flecha */
.dropdown-menu-container3:hover .arrow-icon,
.dropdown-menu-container3.open .arrow-icon {
    transform: rotate(180deg);
    
}
/*------------- TROLEBUS ------------------*/
/*------------- TROLEBUS ------------------*/
/*------------- TROLEBUS ------------------*/
/* Desplegar el contenido */
.dropdown-menu-container4:hover .menu-content,
.dropdown-menu-container4.open .menu-content {
    max-height: 800px; 
    opacity: 1;
    visibility: visible;
}

/* Quitamos la marca de agua y la máscara al abrirse */
.dropdown-menu-container4:hover,
.dropdown-menu-container4.open {
    background-image: none; /* Desaparece la imagen */
}

.dropdown-menu-container4:hover::before,
.dropdown-menu-container4.open::before {
    opacity: 0; /* Desaparece la máscara blanca */
}

/* Rotar la flecha */
.dropdown-menu-container4:hover .arrow-icon,
.dropdown-menu-container4.open .arrow-icon {
    transform: rotate(180deg);
    
}
/*------------- RTP ------------------*/
/*------------- RTP ------------------*/
/*------------- RTP ------------------*/
/* Desplegar el contenido */
.dropdown-menu-container5:hover .menu-content,
.dropdown-menu-container5.open .menu-content {
    max-height: 800px; 
    opacity: 1;
    visibility: visible;
}

/* Quitamos la marca de agua y la máscara al abrirse */
.dropdown-menu-container5:hover,
.dropdown-menu-container5.open {
    background-image: none; /* Desaparece la imagen */
}

.dropdown-menu-container5:hover::before,
.dropdown-menu-container5.open::before {
    opacity: 0; /* Desaparece la máscara blanca */
}

/* Rotar la flecha */
.dropdown-menu-container5:hover .arrow-icon,
.dropdown-menu-container5.open .arrow-icon {
    transform: rotate(180deg);
    
}
/* IMPORTANTE: Tu header debe tener posición relativa para controlar el fondo */
header {
    position: relative;
    overflow: hidden; 
    display: flex;
    justify-content: space-between;
    align-items: center;
}
#tut-nav {
    background-color: rgba(255, 255, 255, 0.75); /* Blanco con 75% de opacidad */
    padding: 8px 8px;
    border-radius: 30px; /* Bordes redondeados estilo cápsula */
    box-shadow: 0 2px 2px rgba(0, 0, 0, 0.1); /* Una ligera sombra para dar relieve */
    display: flex;
    gap: 15px; /* Espacio entre los enlaces */
}

#tut-brand, #tut-nav {
    position: relative;
    z-index: 2; 
}

/* --- CONTENEDOR DEL FONDO --- */
.header-slider-bg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 1; 
    opacity: 0.35; /* Ajusta la visibilidad de los carritos aquí */
    pointer-events: none; 
}

.slider-track {
    display: flex;
    height: 100%;
    /* 150px (ancho de cada carrito) * 12 imágenes totales = 1800px */
    width: calc(150px * 12); 
    /* Cambiamos la animación para que vaya a la derecha */
    animation: correrDerecha 20s linear infinite;
}

.slider-track img {
    height: 100%;      
    width: 150px; /* Si tus imágenes son más anchas, puedes subirlo a 180px o 200px (y ajustar los calcs) */
    object-fit: contain; /* 'contain' suele ser mejor para carritos/buses para que no se corten las llantas */
    padding: 0 10px; /* Espacio entre cada carrito */
}

/* --- ANIMACIÓN HACIA LA DERECHA --- */
@keyframes correrDerecha {
    0% {
        /* Empieza desplazado hacia la izquierda la mitad del tren (6 imágenes = -900px) */
        transform: translateX(calc(-150px * 6)); 
    }
    100% {
        /* Se mueve hacia la derecha hasta llegar a su posición original (0) */
        transform: translateX(0); 
    }
}

/* --- CONTENEDOR PRINCIPAL --- */
.report-section {
    margin-top: 25px;
    background: #ffffff;
    border-radius: 16px;
    padding: 1.5rem;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05); /* Sombra suave de app moderna */
    border: 1px solid #e2e8f0;
    font-family: 'Segoe UI', system-ui, sans-serif;
}

/* --- ENCABEZADO --- */
.report-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    border-bottom: 2px solid #f1f5f9;
    padding-bottom: 12px;
}

.report-header h2 {
    margin: 0;
    font-size: 1.2rem;
    color: #1e293b; /* Gris oscuro elegante */
    font-weight: 700;
    display: flex;
    align-items: center;
    gap: 10px;
}

.report-header h2 i {
    color: #ef4444; /* Rojo vibrante */
    font-size: 1.1rem;
}

.report-subtitle {
    font-size: 0.8rem;
    color: #64748b;
    font-weight: 500;
}

/* --- FORMULARIO Y FILAS --- */
.report-form {
    display: flex;
    flex-direction: column;
    gap: 16px;
}

.form-grid-3 {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); /* Responsivo automático */
    gap: 14px;
}

.form-grid-action {
    display: flex;
    gap: 14px;
    align-items: flex-end;
}

.flex-grow {
    flex: 3;
}

.button-group {
    flex: 1;
    min-width: 150px;
}

/* --- ETIQUETAS E INPUTS --- */
.input-group {
    display: flex;
    flex-direction: column;
    gap: 6px;
}

.input-group label {
    font-size: 0.75rem;
    font-weight: 600;
    color: #475569;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}

.hidden-label {
    display: block;
    font-size: 0.75rem;
}

/* Estilos compartidos para Selects e Inputs */
.report-form select, 
.report-form input[type="text"] {
    width: 100%;
    background: #f8fafc; /* Fondo gris ultra claro muy limpio */
    border: 1px solid #cbd5e1;
    color: #1e293b;
    padding: 10px 12px;
    border-radius: 8px;
    font-size: 0.9rem;
    outline: none;
    transition: all 0.25s ease;
    box-sizing: border-box;
}

/* Efecto cuando el usuario hace clic para escribir */
.report-form select:focus, 
.report-form input[type="text"]:focus {
    border-color: #3b82f6; /* Azul moderno en focus */
    background: #ffffff;
    box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.1);
}

/* Estilo cuando un select está deshabilitado */
.report-form select:disabled {
    background: #f1f5f9;
    color: #94a3b8;
    border-color: #e2e8f0;
    cursor: not-allowed;
}

/* --- BOTÓN ENVIAR --- */
.btn-submit {
    width: 100%;
    background: #ef4444;
    color: white;
    border: none;
    padding: 11px;
    border-radius: 8px;
    font-weight: 600;
    font-size: 0.9rem;
    cursor: pointer;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 8px;
    transition: all 0.2s ease;
}

.btn-submit:hover {
    background: #dc2626; /* Rojo un poco más oscuro al pasar el mouse */
    box-shadow: 0 4px 12px rgba(239, 68, 68, 0.2);
    transform: translateY(-1px); /* Efecto sutil de levantamiento */
}

.btn-submit:active {
    transform: translateY(0);
}

/* Animación sutil para el icono del megáfono */
@keyframes pulse {
    0%, 100% { transform: scale(1); }
    50% { transform: scale(1.08); }
}
.animate-pulse {
    animation: pulse 2s infinite;
}

/* ================================================================
   DROPDOWN MAESTRO — Contenedor padre que agrupa todos los sistemas
   ================================================================ */
.dropdown-maestro {
    background: var(--panel);
    border-radius: 15px;
    box-shadow: 0 8px 32px rgb(232, 232, 232);
    overflow: hidden;
    margin-bottom: 0;
}

.maestro-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 28px 20px;
    cursor: pointer;
    user-select: none;
    background: linear-gradient(135deg, #f5f5f5, #0f172a);
    border-bottom: 2px solid #38bdf8;
    transition: background 0.3s;
}
.maestro-header:hover {
    background: linear-gradient(135deg, #f5f5f5, #172033);
}

.maestro-titulo {
    display: flex;
    align-items: center;
    gap: 12px;
    font-size: 0.95rem;
    font-weight: 800;
    color: #f8fafc;
    letter-spacing: 1px;
    text-transform: uppercase;
}
.maestro-titulo .logos-fila {
    display: flex;
    align-items: center;
    gap: 1px;
}
.maestro-titulo .logos-fila img {
    height: 2.2rem;
    width: auto;
    background: transparent;
}

.maestro-badge {
    font-size: 0.62rem;
    background: rgba(34,197,94,0.15);
    color: #22c55e;
    border: 1px solid #22c55e;
    border-radius: 20px;
    padding: 1px 1px;
    font-weight: 700;
    letter-spacing: 1px;
    display: flex;
    align-items: center;
    gap: 6px;
}
.maestro-badge .dot-live {
    width: 7px; height: 7px;
    background: #22c55e;
    border-radius: 50%;
    animation: parpadeo 1s infinite alternate;
    display: inline-block;
}

.maestro-arrow {
    color: #38bdf8;
    font-size: 1rem;
    transition: transform 0.35s ease;
}
.dropdown-maestro.open .maestro-arrow {
    transform: rotate(180deg);
}

.maestro-content {
    max-height: 0;
    overflow: hidden;
    transition: max-height 0.2s ease-in-out;
}
.dropdown-maestro.open .maestro-content {
    max-height: 99999px; /* suficiente para todo el contenido */
}

/* Las secciones dentro del maestro no necesitan margen superior extra */
.maestro-content > section {
    margin-top: 0 !important;
    border-radius: 0 !important;
    border-top: 1px solid #1e293b;
}
.maestro-content > section:last-child {
    border-bottom: none;
}
    </style>
</head>
<body>
    
<!-- ================================================================
     PANTALLA DE BIENVENIDA DEL TUTORIAL
     ================================================================ -->
<div id="tutorial-welcome">
    <div class="welcome-card">
        <div class="welcome-logo"><img src="MI.JPG" alt="Metro Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"></i></div>
        <div class="welcome-title">¡Bienvenido al STC_LIVE!</div>
        <div class="welcome-subtitle">Movilidad Integral (MI) — <strong style="color: #f6923b">STC LIVE CDMX</strong><br>¿Te gustaría un recorrido rápido por el sistema?</div>
        <div class="welcome-features">
            <div class="welcome-feat"><img src="ME.png" alt="Metro Logo" style="height:1.9rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"> Metro CDMX</div>
            <div class="welcome-feat"><img src="MEB.png" alt="Metro Logo" style="height:1.9rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"> Metrobús CDMX</div>
            <div class="welcome-feat"><img src="CBBB.png" alt="Metro Logo" style="height:1.9rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">Cablebús</div>
            <div class="welcome-feat"><img src="TTB.png" alt="Metro Logo" style="height:1.9rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"> Trolebús STE</div>
            <div class="welcome-feat"><img src="RTTP.png" alt="Metro Logo" style="height:1.9rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"> RTP CDMX</div>
            <div class="welcome-feat"><img src="L.png" alt="Metro Logo" style="height:1.9rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"> Mapa de la Red</div>
        </div>
        <button class="welcome-btn-start" onclick="iniciarTutorial()"><i class="fas fa-play" style="margin-right:8px"></i> Comenzar Recorrido</button>
        <button class="welcome-btn-skip" onclick="cerrarBienvenida()"><i class="fas fa-forward" style="margin-right:6px"></i> Saltar introducción</button>
    </div>
</div>

<!-- ================================================================
     OVERLAY + SPOTLIGHT + TARJETA DEL TUTORIAL
     ================================================================ -->
<div id="tutorial-overlay">
    <div id="tutorial-mask"></div>
    <div id="tutorial-spotlight"></div>
</div>

<div id="tutorial-card" style="display:none;">
    <div class="tut-header">
        <div class="tut-icon" id="tut-icon"><i class="fas fa-info"></i></div>
        <div>
            <div class="tut-subtitle" id="tut-subtitle">SECCIÓN</div>
            <div class="tut-title" id="tut-title">Título</div>
        </div>
    </div>
    <div class="tut-body" id="tut-body">Descripción del elemento.</div>
    <div class="tut-tip" id="tut-tip"><i class="fas fa-lightbulb" style="margin-right:6px;color: #fbbf24"></i> Tip del tutorial</div>
    <div class="tut-progress" id="tut-progress"></div>
    <div class="tut-buttons">
        <button class="tut-btn-skip" onclick="terminarTutorial()"><i class="fas fa-times"></i> Salir</button>
        <button class="tut-btn-next" id="tut-btn-next" onclick="siguientePaso()">Siguiente <i class="fas fa-arrow-right"></i></button>
    </div>
</div>

<!-- ================================================================
     BOTÓN FLOTANTE TUTORIAL
     ================================================================ -->
<button id="btn-abrir-tutorial" onclick="abrirBienvenida()" title="Ver tutorial de la app">
    <i class="fas fa-question"></i>
</button>

<button id="btn-abrir" onclick="abrirReportes()" title="Ver reportes de la app">
    <i class="fas fa-exclamation-triangle"></i>
</button>

<button id="btn-abrir-mapa" onclick="abrirMapas()" title="Ver mapas de la app">
    <i class="fas fa-map-marked-alt"></i>
</button>
<!-- ================================================================
     HEADER (SIN MODIFICAR)
     ================================================================ -->
<header>
    <div class="brand" id="tut-brand">
        <img src="M.png" alt="Logo STC" style="width:24px;height:24px;vertical-align:middle;margin-right:8px;object-fit:contain;">
        STC<span>LIVE</span>
    </div>
    
    <div class="header-slider-bg">
        <div class="slider-track">
            <img src="metr.png" alt="1">
            <img src="mtb.png" alt="2">
            <img src="rtt.png" alt="3">
            <img src="tlb.png" alt="4">
            <img src="tl.png" alt="5">
            <img src="MXB.png" alt="6">
            
            <img src="metr.png" alt="1">
            <img src="mtb.png" alt="2">
            <img src="rtt.png" alt="3">
            <img src="tlb.png" alt="4">
            <img src="tl.png" alt="5">
            <img src="MXB.png" alt="6">
        </div>
    </div>

    <nav id="tut-nav">
        <a href="viaje.php" style="color: var(--accent);"><i class="fas fa-desktop"></i> Central</a>
        <a href="reportes.php" style="color: var(--accent);"><i class="fas fa-file-invoice"></i> Reportes</a>
        <a href="https://www.google.com/maps/d/u/0/edit?mid=1sTEmHz4HElQgoqnRMwp2gc4OTpqmdb8&usp=sharing" target="_blank" style="color: var(--accent);"><i class="fas fa-map"></i> Red</a>
        <a href="login.php" style="color:var(--accent);"><i class="fas fa-right-to-bracket"></i> Sesión</a>
    </nav>
</header>



<!-- ================================================================
     MAIN LAYOUT
     ================================================================ -->
<div class="main-layout">
    <div class="left-column">

        <!-- BARRA SUPERIOR: RELOJ Y FECHA -->
        <div id="tut-header-bar" style="display:flex;justify-content:space-between;align-items:center;background: #d5cab9c9;padding:20px;border-radius:12px;color:white;margin-bottom:25px;border-bottom:4px solid #38bdf8;">
            <div>
                <h1 style="margin:0;font-size:1.4rem;letter-spacing:1px;"><i class="fas fa-paper-plane"></i> CDMX</h1>
                <p id="fecha-actual" style="margin:0;font-size:0.8rem;color: #ffffff;text-transform:uppercase;"></p>
            </div>
            <div style="text-align:right;">
                <div id="reloj-digital" style="font-size:2rem;font-weight:bold;font-family:'Courier New',monospace;color: #38bdf8;text-shadow:0 0 10px rgba(56,189,248,0.3);">00:00:00</div>
                <span class="fas fa-globe-americas" style="font-size:0.7rem;padding:2px 8px;border-radius:10px;color: #22c55e;"> SISTEMA ONLINE</span>
            </div>
        </div>

<!-- ================================================================
     SECCIÓN: 12 LÍNEAS DEL METRO STC
     ================================================================ -->
     
<!-- ================================================================
     DROPDOWN MAESTRO — Agrupa todos los sistemas de transporte
     ================================================================ -->
<div class="dropdown-maestro" id="tut-maestro">

    <!-- Encabezado del maestro -->
    <div class="maestro-header" onclick="toggleMaestro()">
        <div class="maestro-titulo">
            <img src="Mii.png" alt="L1" style="height:2.4rem;width:auto;vertical-align:middle;background:transparent;"></i>MOVILIDAD CDMX
            <div class="logos-fila">
                <img src="ME.png" alt="Metro">
                <img src="MEB.png" alt="Metrobús">
                <img src="CBBB.png" alt="Cablebús">
                <img src="TTB.png" alt="Trolebús">
                <img src="RTTP.png" alt="RTP">
            </div>
        </div>
        <div style="display:flex;align-items:center;gap:12px;">
            <span class="maestro-badge">
                <span class="dot-live"></span>
            </span>
            <i class="fas fa-chevron-down maestro-arrow"></i>
        </div>
    </div>

    <!-- Contenido colapsable -->
    <div class="maestro-content">

<section class="table-container dropdown-menu-container" id="tut-tabla-viajes">
    <div class="menu-header" onclick="toggleMenu()">
        <h2>
            <i class="fas fa-broadcast-tower" style="color: #22c55e;"></i> EN CURSO
            <span style="font-size:0.7rem;color: #64748b;font-weight:400;margin-left:8px;display:inline-flex;align-items:center;">
                <img src="MM.png" alt="Metro Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">
                METRO CDMX — SIMULACIÓN EN TIEMPO REAL
            </span>
        </h2>
        <i class="fas fa-chevron-down arrow-icon"></i>
    </div>
    <div class="menu-content">
         <div class="metro-lineas-grid" id="tut-metro-lineas">

        <div class="metro-linea-card" style="background:linear-gradient(135deg,#e1548a,#c03070);" onclick="abrirLineaMetro(1)">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;"><img src="1.png" alt="L1" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num">L1</span>

            <div class="nom">Observatorio → Pantitlán</div>

            <div class="est-count">20 estaciones</div>

        </div>

        <div class="metro-linea-card" style="background:linear-gradient(135deg,#0077c8,#005fa3);" onclick="abrirLineaMetro(2)">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;"><img src="2.png" alt="L2" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num">L2</span>

            <div class="nom">Cuatro Caminos → Tasqueña</div>

            <div class="est-count">24 estaciones</div>

        </div>

        <div class="metro-linea-card" style="background:linear-gradient(135deg,#AF9800,#747816);" onclick="abrirLineaMetro(3)">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;"><img src="3.png" alt="L3" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num">L3</span>

            <div class="nom">Indios Verdes → Universidad</div>

            <div class="est-count">21 estaciones</div>

        </div>

        <div class="metro-linea-card" style="background:linear-gradient(135deg,#72c2d0,#4aa3b3);" onclick="abrirLineaMetro(4)">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;"><img src="4.png" alt="L4" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num">L4</span>

            <div class="nom">Martín Carrera → Santa Anita</div>

            <div class="est-count">10 estaciones</div>

        </div>

        <div class="metro-linea-card" style="background:linear-gradient(135deg,#fcd116,#e1ba09);color:#1e293b;" onclick="abrirLineaMetro(5)">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;color:#1e293b;"><img src="5.png" alt="L5" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num" style="color:#ffffff;">L5</span>

            <div class="nom" style="color:rgba(0,0,0,0.7);">Politécnico → Pantitlán</div>

            <div class="est-count" style="color:rgba(0,0,0,0.5);">13 estaciones</div>

        </div>

        <div class="metro-linea-card" style="background:linear-gradient(135deg,#e41f1f,#be1414);" onclick="abrirLineaMetro(6)">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;"><img src="6.png" alt="L6" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num">L6</span>

            <div class="nom">El Rosario → Martín Carrera</div>

            <div class="est-count">11 estaciones</div>

        </div>

        <div class="metro-linea-card" style="background:linear-gradient(135deg,#e77d22,#c06518);" onclick="abrirLineaMetro(7)">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;"><img src="7.png" alt="L7" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num">L7</span>

            <div class="nom">El Rosario → Barranca del Muerto</div>

            <div class="est-count">14 estaciones</div>

        </div>

        <div class="metro-linea-card" style="background:linear-gradient(135deg,#00a651,#008542);" onclick="abrirLineaMetro(8)">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;"><img src="8.png" alt="L8" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num">L8</span>

            <div class="nom">Garibaldi → Constitución de 1917</div>

            <div class="est-count">19 estaciones</div>

        </div>

        <div class="metro-linea-card" style="background:linear-gradient(135deg,#683b1b,#4d2b13);" onclick="abrirLineaMetro(9)">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;"><img src="9.png" alt="L9" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num">L9</span>

            <div class="nom">Tacubaya → Pantitlán</div>

            <div class="est-count">12 estaciones</div>

        </div>

        <div class="metro-linea-card" style="background:linear-gradient(135deg,#92278f,#731f71);" onclick="abrirLineaMetro('A')">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;"><img src="LA.png" alt="LA" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num">A</span>

            <div class="nom">Pantitlán → La Paz</div>

            <div class="est-count">10 estaciones</div>

        </div>

        <div class="metro-linea-card" style="background:linear-gradient(to bottom,#008542 50%,#b1b3b6 50%);" onclick="abrirLineaMetro('B')">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;"><img src="LB.png" alt="LB" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num">B</span>

            <div class="nom">Buenavista → Ciudad Azteca</div>

            <div class="est-count">21 estaciones</div>

        </div>

        <div class="metro-linea-card tiene-cierres" style="background:linear-gradient(135deg,#e3c153,#c7a53c);color:#1e293b;" onclick="abrirLineaMetro(12)">

            <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;color:#1e293b;"><img src="12.png" alt="L12" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>

            <span class="num" style="color:#ffffff;">L12</span>

            <div class="nom" style="color:rgba(0,0,0,0.7);">Mixcoac → Tláhuac</div>

            <div class="est-count" style="color:rgba(0,0,0,0.5);">20 estaciones</div>

        </div>

    </div>
        <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:16px;padding-top:14px;border-top:1px solid #1e293b;">
            <div class="stat-chip"><i class="fas fa-users" style="color:#3b82f6"></i> Servicio: <b>05:00 – 00:00 hrs</b></div>
            <div class="stat-chip"><i class="fas fa-circle-check" style="color:#22c55e"></i> Estado: <b style="color:#22c55e">OPERANDO</b></div>
            <div class="stat-chip"><i class="fas fa-triangle-exclamation" style="color:#f59e0b"></i> L12: <b style="color:#ef4444">CIERRES PARCIALES</b></div>
        </div>
    </div>
</section>
<!-- ================================================================
     SECCIÓN: 7 LÍNEAS METROBÚS
     ================================================================ -->
<section class="table-container dropdown-menu-container2" style="margin-top:20px; border-radius:15px; padding:1.5rem;" id="tut-metrobus">
    <div class="menu-header" onclick="toggleMenu()">
        <h2>
            <i class="fas fa-broadcast-tower" style="color:#22c55e;"></i> EN CURSO
            <span style="font-size:0.7rem;color:#64748b;font-weight:400;margin-left:8px;display:inline-flex;align-items:center;">
                <img src="mb.png" alt="Metrobús Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">
                METROBÚS CDMX — SIMULACIÓN EN TIEMPO REAL
            </span>
        </h2>
        <i class="fas fa-chevron-down arrow-icon"></i>
    </div>
    
    <div class="menu-content">
        <div class="mb-lineas-grid">
            <div class="mb-linea-card" style="background:linear-gradient(135deg,#e62215,#b8170d);color:#ffffff;" onclick="abrirLineaMetrobus(1)">
                <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;float:right;"><img src="M1.png" alt="MB1" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>
                <span class="num" style="color:#ffffff;">L1</span>
                <div class="nom" style="color:rgba(255,255,255,0.9);">Indios Verdes → El Caminero</div>
                <div class="est-count" style="color:rgba(255,255,255,0.6);">46 estaciones</div>
            </div>
            <div class="mb-linea-card" style="background:linear-gradient(135deg,#a3238e,#7d166c);color:#ffffff;" onclick="abrirLineaMetrobus(2)">
                <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;float:right;"><img src="M2.png" alt="MB2" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>
                <span class="num" style="color:#ffffff;">L2</span>
                <div class="nom" style="color:rgba(255,255,255,0.9);">Tacubaya → Tepalcates</div>
                <div class="est-count" style="color:rgba(255,255,255,0.6);">33 estaciones</div>
            </div>
            <div class="mb-linea-card" style="background:linear-gradient(135deg,#afca0b,#8da305);color:#ffffff;" onclick="abrirLineaMetrobus(3)">
                <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;float:right;"><img src="M3.png" alt="MB3" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>
                <span class="num" style="color:#ffffff;">L3</span>
                <div class="nom" style="color:rgba(255,255,255,0.9);">Tenayuca → Pueblo Sta. Cruz Atoyac</div>
                <div class="est-count" style="color:rgba(255,255,255,0.6);">40 estaciones</div>
            </div>
            <div class="mb-linea-card" style="background:linear-gradient(135deg,#f37021,#c75613);color:#ffffff;" onclick="abrirLineaMetrobus(4)">
                <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;float:right;"><img src="M4.png" alt="MB4" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>
                <span class="num" style="color:#ffffff;">L4</span>
                <div class="nom" style="color:rgba(255,255,255,0.9);">Buenavista → Aeropuerto T1</div>
                <div class="est-count" style="color:rgba(255,255,255,0.6);">41 estaciones</div>
            </div>
            <div class="mb-linea-card" style="background:linear-gradient(135deg,#1e3a8a,#111827);color:#ffffff;" onclick="abrirLineaMetrobus(5)">
                <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;float:right;"><img src="M5.png" alt="MB5" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>
                <span class="num" style="color:#ffffff;">L5</span>
                <div class="nom" style="color:rgba(255,255,255,0.9);">Río de los Remedios → Vaqueritos</div>
                <div class="est-count" style="color:rgba(255,255,255,0.6);">51 estaciones</div>
            </div>
            <div class="mb-linea-card" style="background:linear-gradient(135deg,#e5007d,#b50061);color:#ffffff;" onclick="abrirLineaMetrobus(6)">
                <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;float:right;"><img src="M6.png" alt="MB6" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>
                <span class="num" style="color:#ffffff;">L6</span>
                <div class="nom" style="color:rgba(255,255,255,0.9);">El Rosario → Villa de Aragón</div>
                <div class="est-count" style="color:rgba(255,255,255,0.6);">37 estaciones</div>
            </div>
            <div class="mb-linea-card" style="background:linear-gradient(135deg,#006837,#004d28);color:#ffffff;" onclick="abrirLineaMetrobus(7)">
                <span class="trenes-badge" style="background:transparent;box-shadow:none;padding:0;float:right;"><img src="M7.png" alt="MB7" style="height:1.4rem;width:auto;vertical-align:middle;background:transparent;"></span>
                <span class="num" style="color:#ffffff;">L7</span>
                <div class="nom" style="color:rgba(255,255,255,0.9);">Indios Verdes → Campo Marte</div>
                <div class="est-count" style="color:rgba(255,255,255,0.6);">31 estaciones</div>
            </div>
        </div>

        <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:14px;padding-top:12px;border-top:1px solid #1e293b;">
            <div class="stat-chip"><i class="fas fa-bus" style="color:#f59e0b"></i> Cobertura: <b>7 corredores</b></div>
            <div class="stat-chip"><i class="fas fa-clock" style="color:#f59e0b"></i> Servicio: <b>04:30 – 00:30 hrs</b></div>
            <div class="stat-chip"><i class="fas fa-circle-check" style="color:#22c55e"></i> Estado: <b style="color:#22c55e">OPERANDO</b></div>
        </div>
    </div>
</section>

<!-- ================================================================
     SECCIÓN: CABLEBÚS (3 LÍNEAS)
     ================================================================ -->

<section class="table-container dropdown-menu-container3" style="margin-top:20px; border-radius:15px; padding:1.5rem;" id="tut-cablebus">
    <div class="menu-header" onclick="toggleMenu()">
        <h2>
        <i class="fas fa-cable-car" style="color:#009b3a;"></i> EN CURSO
        <span style="font-size:0.7rem;color:#64748b;font-weight:400;margin-left:8px;display:inline-flex;align-items:center;">
            <img src="cb.png" alt="Metrobús Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">
               CABLEBÚS CDMX — SIMULACIÓN EN TIEMPO REAL
        </span>
    </h2>
     <i class="fas fa-chevron-down arrow-icon"></i>
    </div>
    <div class="menu-content">
    <div class="mb-lineas-grid">
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#e5007d,#b50061);color:#ffffff;" onclick="abrirLineaCablebus(1)">
            <span class="num" style="color:#ffffff;">L1</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Indios Verdes → Cuautepec</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">9 estaciones · 4.9 km</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#007dc3,#005a9e);color:#ffffff;" onclick="abrirLineaCablebus(2)">
            <span class="num" style="color:#ffffff;">L2</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Santa Martha → UAM Iztapalapa</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">10 estaciones · 5.5 km</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#f37021,#c75613);color:#ffffff;" onclick="abrirLineaCablebus(3)">
            <span class="num" style="color:#ffffff;">L3</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Constitución → Periferico Oriente</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">8 estaciones · 4.2 km</div>
        </div>
    </div>
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:14px;padding-top:12px;border-top:1px solid #1e293b;">
        <div class="stat-chip"><i class="fas fa-cable-car" style="color:#009b3a"></i> Cobertura: <b>3 líneas</b></div>
        <div class="stat-chip"><i class="fas fa-clock" style="color:#f59e0b"></i> Servicio: <b>06:00 – 22:00 hrs</b></div>
        <div class="stat-chip"><i class="fas fa-circle-check" style="color:#22c55e"></i> Estado: <b style="color:#22c55e">OPERANDO</b></div>
    </div>
</section>

<!-- ================================================================
     SECCIÓN: TROLEBÚS / STE (3 LÍNEAS PRINCIPALES)
     ================================================================ -->
<section class="table-container dropdown-menu-container4" style="margin-top:20px; border-radius:15px; padding:1.5rem;" id="tut-trolebus">
    <div class="menu-header" onclick="toggleMenu()">
    <h2>
        <i class="fas fa-bolt" style="color:#0033a0;"></i> EN CURSO
        <span style="font-size:0.7rem;color:#64748b;font-weight:400;margin-left:8px;display:inline-flex;align-items:center;">
             <img src="tb.png" alt="Metrobús Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">
            TROLEBÚS / STE CDMX — SIMULACIÓN EN TIEMPO REAL
        </span>
    </h2>
      <i class="fas fa-chevron-down arrow-icon"></i>
    </div>
    <div class="menu-content">
    <div class="mb-lineas-grid">
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#0033a0,#002070);color:#ffffff;" onclick="abrirLineaTrolebus(1)">
            <span class="num" style="color:#ffffff;">L1</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Tacubaya → Chapultepec</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">2 estaciones · 1.7 km</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#1e3a8a,#111827);color:#ffffff;" onclick="abrirLineaTrolebus(2)">
            <span class="num" style="color:#ffffff;">L2</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Buenavista → San Lázaro</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">15 estaciones · 8.8 km</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#4c1d95,#3b0764);color:#ffffff;" onclick="abrirLineaTrolebus(3)">
            <span class="num" style="color:#ffffff;">L3</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Etiopía → San Lázaro</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">13 estaciones · 6.4 km</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#065f46,#064e3b);color:#ffffff;" onclick="abrirLineaTrolebus(4)">
            <span class="num" style="color:#ffffff;">L4</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">San Lázaro → Peñón Viejo</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">10 estaciones · 5.5 km</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#78350f,#451a03);color:#ffffff;" onclick="abrirLineaTrolebus(5)">
            <span class="num" style="color:#ffffff;">L5</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Zócalo → Xochimilco</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">22 estaciones · 22 km</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#374151,#1f2937);color:#ffffff;" onclick="abrirLineaTrolebus(6)">
            <span class="num" style="color:#ffffff;">L6</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Pino Suárez → Huipulco</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">20 estaciones · 19 km</div>
        </div>
    </div>
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:14px;padding-top:12px;border-top:1px solid #1e293b;">
        <div class="stat-chip"><i class="fas fa-bolt" style="color:#0033a0"></i> Cobertura: <b>6 líneas eléctricas</b></div>
        <div class="stat-chip"><i class="fas fa-clock" style="color:#f59e0b"></i> Servicio: <b>05:00 – 23:30 hrs</b></div>
        <div class="stat-chip"><i class="fas fa-circle-check" style="color:#22c55e"></i> Estado: <b style="color:#22c55e">OPERANDO</b></div>
    </div>
</section>

<!-- ================================================================
     SECCIÓN: RTP — MÓDULO 1 (RUTAS SANTA FE / PONIENTE)
     ================================================================ -->
<section class="table-container dropdown-menu-container5" style="margin-top:20px; border-radius:15px; padding:1.5rem;" id="tut-rtp">
    <div class="menu-header" onclick="toggleMenu()">
    <h2>
        <i class="fas fa-broadcast-tower" style="color:#7ab800;"></i> EN CURSO
        <span style="font-size:0.7rem;color:#64748b;font-weight:400;margin-left:8px;display:inline-flex;align-items:center;">
             <img src="rtp.png" alt="Metrobús Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">
            &nbsp;RTP CDMX — MÓDULO 1 · PONIENTE / SANTA FE
        </span>
    </h2>
    <i class="fas fa-chevron-down arrow-icon"></i>
    </div>
    <div class="menu-content">
    <div class="mb-lineas-grid">
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#7ab800,#5a8a00);color:#ffffff;" onclick="abrirLineaRTP('M1-9C')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M1 · 9C</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">C.C. Santa Fe → Puerta Grande</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">25 paradas</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#5a8a00,#3d6000);color:#ffffff;" onclick="abrirLineaRTP('M1-76')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M1 · 76</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">C.C. Santa Fe → Metro Auditorio</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">39 paradas</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#84cc16,#65a30d);color:#ffffff;" onclick="abrirLineaRTP('M1-76A')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M1 · 76-A</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Santa Fe → Metro Auditorio</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">35 paradas</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#4d7c0f,#365314);color:#ffffff;" onclick="abrirLineaRTP('M1-110')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M1 · 110</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Metro Tacubaya → Chimalpa</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">45 paradas</div>
        </div>
    </div>
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:14px;padding-top:12px;border-top:1px solid #1e293b;">
        <div class="stat-chip"><i class="fas fa-truck-front" style="color:#7ab800"></i> Módulo: <b>1 — Poniente</b></div>
        <div class="stat-chip"><i class="fas fa-clock" style="color:#f59e0b"></i> Servicio: <b>05:00 – 22:00 hrs</b></div>
        <div class="stat-chip"><i class="fas fa-circle-check" style="color:#22c55e"></i> Estado: <b style="color:#22c55e">OPERANDO</b></div>
    </div>
</section>

<!-- ================================================================
     SECCIÓN: RTP — MÓDULO 2 (ORIENTE / IZTAPALAPA)
     ================================================================ -->
<section class="table-container dropdown-menu-container5" style="margin-top:20px; border-radius:15px; padding:1.5rem;" id="tut-rtp2">
    <div class="menu-header" onclick="toggleMenu()">
    <h2>
        <i class="fas fa-broadcast-tower" style="color:#7ab800;"></i> EN CURSO
        <span style="font-size:0.7rem;color:#64748b;font-weight:400;margin-left:8px;display:inline-flex;align-items:center;">
             <img src="rtp.png" alt="Metrobús Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">
            &nbsp;RTP CDMX — MÓDULO 2 · ORIENTE / IZTAPALAPA
        </span>
    </h2>
    <i class="fas fa-chevron-down arrow-icon"></i>
    </div>
    <div class="menu-content">
    <div class="mb-lineas-grid">
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#ca8a04,#92400e);color:#ffffff;" onclick="abrirLineaRTP('M2-36')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M2 · 36</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Zócalo → La Paz</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">28 paradas</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#b45309,#78350f);color:#ffffff;" onclick="abrirLineaRTP('M2-36A')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M2 · 36-A</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Zócalo → Iztapalapa</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">32 paradas</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#d97706,#b45309);color:#ffffff;" onclick="abrirLineaRTP('M2-50')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M2 · 50</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Santa Anita → Pantitlán</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">18 paradas</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#f59e0b,#d97706);color:#ffffff;" onclick="abrirLineaRTP('M2-116')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M2 · 116</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Metro Constitución → Tláhuac</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">22 paradas</div>
        </div>
    </div>
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:14px;padding-top:12px;border-top:1px solid #1e293b;">
        <div class="stat-chip"><i class="fas fa-truck-front" style="color:#7ab800"></i> Módulo: <b>2 — Oriente</b></div>
        <div class="stat-chip"><i class="fas fa-clock" style="color:#f59e0b"></i> Servicio: <b>05:00 – 22:00 hrs</b></div>
        <div class="stat-chip"><i class="fas fa-circle-check" style="color:#22c55e"></i> Estado: <b style="color:#22c55e">OPERANDO</b></div>
    </div>
</section>

<!-- ================================================================
     SECCIÓN: RTP — MÓDULO 3 (NORTE / TLALNEPANTLA)
     ================================================================ -->

<section class="table-container dropdown-menu-container5" style="margin-top:20px; border-radius:15px; padding:1.5rem;" id="tut-rtp">
    <div class="menu-header" onclick="toggleMenu()">
    <h2>
        <i class="fas fa-broadcast-tower" style="color:#7ab800;"></i> EN CURSO
        <span style="font-size:0.7rem;color:#64748b;font-weight:400;margin-left:8px;display:inline-flex;align-items:center;">
             <img src="rtp.png" alt="Metrobús Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">
            &nbsp;RTP CDMX — MÓDULO 3 · NORTE / TLALNEPANTLA
        </span>
    </h2>
    <i class="fas fa-chevron-down arrow-icon"></i>
    </div>
    <div class="menu-content">
    <div class="mb-lineas-grid">
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#0f766e,#0d5c55);color:#ffffff;" onclick="abrirLineaRTP('M3-3')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M3 · 3</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Metro Indios Verdes → Tlalnepantla</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">14 paradas</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#0e7490,#0c5f73);color:#ffffff;" onclick="abrirLineaRTP('M3-23')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M3 · 23</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">Vallejo → Aeropuerto</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">20 paradas</div>
        </div>
        <div class="mb-linea-card" style="background:linear-gradient(135deg,#1d4ed8,#1e3a8a);color:#ffffff;" onclick="abrirLineaRTP('M3-105')">
            <span class="num" style="color:#ffffff;font-size:0.9rem;">M3 · 105</span>
            <div class="nom" style="color:rgba(255,255,255,0.9);">La Raza → Cuautitlán</div>
            <div class="est-count" style="color:rgba(255,255,255,0.6);">25 paradas</div>
        </div>
    </div>
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:14px;padding-top:12px;border-top:1px solid #1e293b;">
        <div class="stat-chip"><i class="fas fa-truck-front" style="color:#7ab800"></i> Módulo: <b>3 — Norte</b></div>
        <div class="stat-chip"><i class="fas fa-clock" style="color:#f59e0b"></i> Servicio: <b>05:00 – 22:00 hrs</b></div>
        <div class="stat-chip"><i class="fas fa-circle-check" style="color:#22c55e"></i> Estado: <b style="color:#22c55e">OPERANDO</b></div>
    </div>
</section>


    </div><!-- /maestro-content -->
</div><!-- /dropdown-maestro -->

<!-- ================================================================
     SECCIÓN: REPORTES
     ================================================================ -->
<section id="stc-live-quejas-mi" class="report-section">
    
    <div class="report-header">
        <h2>
            <i class="fas fa-bullhorn animate-pulse"></i> 
            Reportes STC LIVE
        </h2>
            <!-- NUESTRO BOTÓN PARA SALIR -->
    <button type="button" onclick="abrirReportes()" style="background: none; border: none; font-size: 1.5rem; cursor: pointer; color: #ef4444;">
        <i class="fas fa-times"></i> Cerrar
    </button>
        <span class="report-subtitle"><img src="MII.png" alt="Metro Logo" style="height:2.3rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"></span>
    </div>




    <form id="form-queja-live" class="report-form">
        <div class="form-grid-3">
            <div class="input-group">
                <label for="rep-servicio">Sistema</label>
                <select id="rep-servicio" required>
                    <option value="">Selecciona sistema</option>
                    <option value="metro">🚇 Metro</option>
                    <option value="mb">🚌 Metrobús</option>
                    <option value="cablebus">🚡 Cablebús</option>
                    <option value="trolebus">🚎 Trolebús</option>
                    <option value="rtp">🚌 RTP</option>
                </select>
            </div>
            
            <div class="input-group">
                <label for="rep-linea">Línea</label>
                <select id="rep-linea" required disabled>
                    <option value="">Selecciona línea</option>
                </select>
            </div>

            <div class="input-group">
                <label for="rep-estacion">Estación</label>
                <select id="rep-estacion" required disabled>
                    <option value="">Selecciona estación</option>
                </select>
            </div>
        </div>

        <div class="form-grid-action">
            <div class="input-group flex-grow">
                <label for="rep-descripcion">Detalle de la incidencia</label>
                <input type="text" id="rep-descripcion" placeholder="Ej. Retraso de 10 min, dirección Universidad..." required>
            </div>
            
            <div class="button-group">
                <label class="hidden-label">&nbsp;</label>
                <button type="submit" class="btn-submit">
                    <i class="fas fa-paper-plane"></i> Enviar Reporte
                </button>
            </div>
        </div>
    </form>
</section>


<script>
// 1. FUNCIONES GLOBALES (Para que los botones onclick="abrir..." funcionen de inmediato)
window.abrirReportes = function() {
    const seccionReportes = document.getElementById('stc-live-quejas-mi');
    if (seccionReportes) {
        seccionReportes.classList.toggle('active');
    }
};

window.abrirMapas = function() {
    const contenedorMapas = document.getElementById('contenedor-mapas');
    if (contenedorMapas) {
        contenedorMapas.classList.toggle('active');
    }
};

// 2. LÓGICA DE LOS SELECTORES Y ENVÍO (Espera a que cargue el HTML)
document.addEventListener('DOMContentLoaded', function() {
    const SISTEMAS_DATOS = {
        metro: typeof METRO_LINEAS !== 'undefined' ? METRO_LINEAS : {},
        mb: typeof METROBUS_LINEAS !== 'undefined' ? METROBUS_LINEAS : {},
        cablebus: typeof CABLEBUS_LINEAS !== 'undefined' ? CABLEBUS_LINEAS : {},
        trolebus: typeof TROLEBUS_LINEAS !== 'undefined' ? TROLEBUS_LINEAS : {},
        rtp: typeof RTP_LINEAS !== 'undefined' ? RTP_LINEAS : {}
    };

    const selectServicio = document.getElementById('rep-servicio');
    const selectLinea = document.getElementById('rep-linea');
    const selectEstacion = document.getElementById('rep-estacion');

    // Cambiar Líneas al elegir Sistema
    if (selectServicio) {
        selectServicio.addEventListener('change', function() {
            selectLinea.innerHTML ='<option value=""> Línea </option>';
            selectEstacion.innerHTML ='<option value=""> Estación </option>';
            selectEstacion.disabled = true;

            const sistema = this.value;
            if (!sistema || !SISTEMAS_DATOS[sistema]) {
                selectLinea.disabled = true;
                return;
            }

            const lineas = SISTEMAS_DATOS[sistema];
            for (const clave in lineas) {
                const option = document.createElement('option');
                option.value = clave;
                option.textContent = lineas[clave].nombre;
                selectLinea.appendChild(option);
            }
            selectLinea.disabled = false;
        });
    }

    // Cambiar Estaciones al elegir Línea
    if (selectLinea) {
        selectLinea.addEventListener('change', function() {
            selectEstacion.innerHTML = '<option value="">-- Estación --</option>';
            
            const sistema = selectServicio.value;
            const lineaId = this.value;

            if (!lineaId) {
                selectEstacion.disabled = true;
                return;
            }

            const listaEstaciones = SISTEMAS_DATOS[sistema][lineaId].estaciones;
            listaEstaciones.forEach(estacion => {
                const option = document.createElement('option');
                option.value = estacion;
                option.textContent = estacion;
                selectEstacion.appendChild(option);
            });
            selectEstacion.disabled = false;
        });
    }

    // Enviar Reporte
    const formQueja = document.getElementById('form-queja-live');
    if (formQueja) {
        formQueja.addEventListener('submit', function(e) {
            e.preventDefault();

            const tipo = selectServicio.value;
            const estacion = selectEstacion.value;
            const descripcion = document.getElementById('rep-descripcion').value.trim();
            
            const ahora = new Date();
            const horaStr = ahora.getHours().toString().padStart(2, '0') + ':' + ahora.getMinutes().toString().padStart(2, '0');

            const llaveLimpia = estacion.toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "").replace(/\s+/g, '');

            const paquete = { tipo, estacion, llave: llaveLimpia, descripcion, hora: horaStr };

            fetch('api_quejas.php', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(paquete)
            })
            .then(res => res.json())
            .then(data => {
                if(data.status === 'success') {
                    alert(`✅ Reporte guardado para: ${estacion}`);
                    formQueja.reset();
                    selectLinea.disabled = true;
                    selectEstacion.disabled = true;
                    
                    // Cierra la pantalla completa tras enviar
                    window.abrirReportes();
                }
            })
            .catch(err => console.error("Error al guardar:", err));
        });
    }
});
</script>

    </div><!-- /left-column -->

    <!-- ================================================================
         PANEL LATERAL: ANUNCIOS Y ESTADO (TELEMETRÍA) style="margin-top:20px; border-radius:15px; padding:1.5rem;"
         ================================================================ -->
 <aside class="telemetry-panel" id="tut-telemetria" style="position: sticky; top: 20px; align-self: start; background: #1e293bc9; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.5); padding: 18px; width: 340px; border: 1px solid #553333; color: #f8fafc; font-family: sans-serif; backdrop-filter: blur(8px); display: flex; flex-direction: column; min-height: 85vh;">
    
    <!-- ENCABEZADO DEL PANEL -->
    <div class="panel-header" style="display: flex; align-items: center; justify-content: space-between; border-bottom: 1px solid #334155; padding-bottom: 12px; margin-bottom: 15px;">
        <div style="display: flex; align-items: center; gap: 10px;">
            <i class="fas fa-bullhorn" style="color: #fbbf24; font-size: 1.3rem; animation: pulse 2s infinite;"></i>
            <div>
                <h3 style="margin: 0; font-size: 0.9rem; letter-spacing: 1px; color: #f8fafc; font-weight: bold;">ANUNCIOS Y ESTADO</h3>
                <span style="font-size: 0.65rem; color: #94a3b8; font-family: monospace;">STC LIVE • MOVILIDAD CDMX</span>
            </div>
        </div>
        <span id="red-status-global" style="font-size: 0.65rem; background: #15803d; color: #f7d9bb; padding: 2px 8px; border-radius: 20px; font-weight: bold;">EN VIVO</span>
    </div>

    <!-- SELECTOR DE TRANSPORTE (COLORES OFICIALES SEMOVI) -->
    <div class="transport-selector" style="display: grid; grid-template-columns: repeat(5, 1fr); gap: 3px; margin-bottom: 10px;">
        <button onclick="cambiarCanalX('MetroCDMX', 'METRO')" title="Metro" style="background: #f26704e1; color: white; border: none; padding: 1px 0; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 0.75rem; transition: transform 0.2s;"><img src="ME.png" alt="Metro Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"></button>
        <button onclick="cambiarCanalX('MetrobusCDMX', 'METROBÚS')" title="Metrobús" style="background: #cb1306; color: white; border: none; padding: 1px 0; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 0.75rem; transition: transform 0.2s;"><img src="MEB.png" alt="Metro Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"></button>
        <button onclick="cambiarCanalX('MICablebusCDMX', 'CABLEBÚS')" title="Cablebús" style="background: #38d5fccf; color: white; border: none; padding: 1px 0; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 0.75rem; transition: transform 0.2s;"><img src="CBBB.png" alt="Metro Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"></button>
        <button onclick="cambiarCanalX('RTP_CiudadDeMex', 'RTP')" title="RTP" style="background: #7ab800; color: white; border: none; padding: 1px 0; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 0.75rem; transition: transform 0.2s;"><img src="RTTP.png" alt="Metro Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"></button>
        <button onclick="cambiarCanalX('STECDMX', 'TROLEBÚS/STE')" title="Transportes Eléctricos" style="background: #096ed3e1; color: white; border: none; padding: 1px 0; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 0.75rem; transition: transform 0.2s;"><img src="TTB.png" alt="Metro Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;"></button>
    </div>

    <!-- CONTENEDOR EN MODO LISTA DE ANUNCIOS (AHORA CRECE VERTICALMENTE) -->
    <div class="x-container" style="background: #0f172a; border-radius: 8px; overflow: hidden; position: relative; flex-grow: 1; min-height: 200px; border: 1px solid #334155; display: flex; flex-direction: column;">
        
        <div class="feed-header" style="font-size: 0.7rem; padding: 6px 12px; background: #1e293b; border-bottom: 1px solid #334155; display: flex; justify-content: space-between; align-items: center; z-index: 10; position: relative;">
            <span style="color: #fbbf24; font-weight: bold;"><i class="fas fa-list-ul"></i> NOTIFICACIONES RECIENTES</span>
            <span id="current-handle" style="color: #b8a494; font-family: monospace; font-weight: bold;">@MetroCDMX</span>
        </div>

        <!-- Ajuste de scroll e iframe para ocupar el 100% de la altura disponible -->
        <div style="flex-grow: 1; width: 100%; overflow-y: hidden; overflow-x: hidden; position: relative;">
            <div style="margin-top: -510px; width: 100%; height:1500px;"> 
                <iframe id="iframe-x-feed" src="https://nitter.net/MetroCDMX?theme=dark&hide_replies=1" frameborder="0" style="width: 100%; height: 100%; background: transparent; border: none; pointer-events: auto;"></iframe>
            </div>
        </div>
    </div>

    <!-- TARJETAS DE ESTADO INFERIOR -->
    <div class="network-grid" style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 15px; background: #1e293b; padding: 5px; border-radius: 8px; border: 1px solid #334155;">
        <div style="border-right: 1px solid #334155; padding-right: 5px;">
            <span style="font-size: 0.6rem; color: #94a3b8; text-transform: uppercase; font-weight: bold;">Red Activa</span><br>
            <b id="info-sistema-activo" style="font-size: 0.8rem; color: #38bdf8;">METRO (STC)</b>
        </div>
        <div style="padding-left: 5px;">
            <span style="font-size: 0.6rem; color: #94a3b8; text-transform: uppercase; font-weight: bold;">Formato</span><br>
            <b style="font-size: 0.8rem; color: #fbbf24;"><i class="fas fa-bullhorn"></i> ANUNCIOS 24H</b>
        </div>
    </div>
    
    <!-- BOTÓN DE RESET -->
    <button onclick="limpiarMonitor()" style="width: 100%; margin-top: 12px; padding: 8px; background: transparent; color: #94a3b8; border: 1px solid #334155; border-radius: 6px; cursor: pointer; font-size: 0.7rem; transition: all 0.3s; font-weight: bold; letter-spacing: 0.5px;">
        <i class="fas fa-sync" style="margin-right: 6px; color: #38bdf8;"></i> ACTUALIZAR LISTA
    </button>
</aside>




        <!-- SEPARADOR -->
       
            <div class="media-stack">
                <section id="tut-camara">
                    <h2><span class="live-indicator"></span>
                    N+ MEDIOS D COMUNICACIÓN
                </h2>
                    <div class="monitor-box" >
                        <iframe width="100%" height="100%" src="https://www.youtube.com/embed/p2AzyIEuFak?autoplay=1&mute=1" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
                    </div>
                </section>
           

<div id="contenedor-mapas" class="fullscreen-modal">
    
    <div style="display: flex; justify-content: flex-end; padding: 10px 20px; background: #111827;">
        <button type="button" onclick="abrirMapas()" style="background: #ef4444; border: none; color: white; padding: 10px 20px; font-size: 1rem; border-radius: 6px; cursor: pointer; font-weight: bold;">
            <i class="fas fa-times"></i> Cerrar Mapas
        </button>
    </div>

    <section class="tut-mapa" style="position:relative; padding: 20px; background: #111827; color: white;">
        <h2>
            <img src="/stc/M.png" alt="Logo Metro" style="width:24px;height:24px;vertical-align:middle;margin-right:8px;object-fit:contain;">
            LÍNEAS STC METRO
        </h2>
        <div style="position:absolute;top:70px;left:40px;z-index:10;background:rgba(0,0,0,0.75);color:#00ff00;padding:5px 10px;border-radius:4px;font-family:monospace;font-size:12px;display:flex;align-items:center;gap:8px;">
            <span style="width:10px;height:10px;background: #00ff00;border-radius:50%;display:inline-block;animation:parpadeo 1s infinite alternate;"></span>
            MAPA (ACTUALIZANDO...)
        </div>
        <div class="monitor-box" style="width:100%;height:550px;overflow:hidden;border-radius:8px;">
            <iframe width="100%" height="100%" style="border:0;" src="https://www.google.com/maps/d/embed?mid=1HbWBfY6lGUgi6eZqtDApdBGPWPpuWNo" loading="lazy"></iframe>
        </div>
    </section>

    <section class="tut-mapa" style="position:relative; padding: 20px; background: #111827; color: white;">
        <h2>
            <img src="/stc/L.png" alt="Logo Metro" style="width:24px;height:24px;vertical-align:middle;margin-right:8px;object-fit:contain;">
            LINEA STC METRO
        </h2>
        <div style="position:absolute;top:70px;left:40px;z-index:10;background:rgba(0,0,0,0.75);color: #00ff00;padding:5px 10px;border-radius:4px;font-family:monospace;font-size:12px;display:flex;align-items:center;gap:8px;">
            <span style="width:10px;height:10px;background:#00ff00;border-radius:50%;display:inline-block;animation:parpadeo 1s infinite alternate;"></span>
            MAPA (ACTUALIZANDO...)
        </div>
        <div class="monitor-box" style="width:100%;height:550px;overflow:hidden;border-radius:8px;">
            <iframe width="100%" height="100%" style="border:0;" src="https://www.google.com/maps/d/embed?mid=1XWgnRco3nUN-9xQqKvT_z2NEt-5S-Rw&ll=19.46422244178118%2C-99.05978096668679&z=11" loading="lazy"></iframe>
        </div>
    </section>

    <section class="tut-mapa" style="position:relative; padding: 20px; background: #111827; color: white;">
        <h2><i class="fas fa-location-arrow"></i> METROBÚS</h2>
        <div style="position:absolute;top:70px;left:40px;z-index:10;background:rgba(0,0,0,0.75);color:#00ff00;padding:5px 10px;border-radius:4px;font-family:monospace;font-size:12px;display:flex;align-items:center;gap:8px;">
            <span style="width:10px;height:10px;background:#00ff00;border-radius:50%;display:inline-block;animation:parpadeo 1s infinite alternate;"></span>
            MAPA (ACTUALIZANDO...)
        </div>
        <div class="monitor-box" style="width:100%;height:550px;overflow:hidden;border-radius:8px;">
            <iframe width="100%" height="100%" style="border:0;" src="https://www.google.com/maps/d/embed?mid=1ruA_R5lS6JpQrbJTTfHi0hIPk-xQv1I" loading="lazy"></iframe>
        </div>
    </section>

     <section class="tut-mapa" style="position:relative; padding: 20px; background: #111827; color: white;">
        <h2><i class="fas fa-location-arrow"></i> METROBÚS</h2>
        <div style="position:absolute;top:70px;left:40px;z-index:10;background:rgba(0,0,0,0.75);color:#00ff00;padding:5px 10px;border-radius:4px;font-family:monospace;font-size:12px;display:flex;align-items:center;gap:8px;">
            <span style="width:10px;height:10px;background:#00ff00;border-radius:50%;display:inline-block;animation:parpadeo 1s infinite alternate;"></span>
            MAPA (ACTUALIZANDO...)
        </div>
        <div class="monitor-box" style="width:100%;height:550px;overflow:hidden;border-radius:8px;">
            <iframe width="100%" height="100%" style="border:0;" src="https://www.google.com/maps/d/embed?mid=1_QVAPzVXt_RWRUQ4RPZ_l8hJWfNdoKE&ehbc=2E312F&noprof=1" loading="lazy"></iframe>
        </div>
    </section>
</div>



        <section style="background:linear-gradient(to right,#1e3a8a,#1e40af);">
            <h2 style="color:white;margin:0">CONTACTO AGENTE</h2>
            <p style="font-size:0.75rem;color:#bfdbfe;margin-top:10px;">
                VITEM RV — 56 6101 2729<br>
                Av. Primero de Mayo 238, San Luis Tlatilco,<br>
                53529 Naucalpan de Juárez, Méx.
                <span class="fas fa-laptop-code" style="color:#4ade80"> 24HRS</span>
            </p>
        </section>


<!-- ================================================================
     FOOTER
     ================================================================ -->
<footer>
    <i class="fas fa-network-wired"></i> 2026 STC LIVE S.A. DE C.V. | TODOS LOS DERECHOS RESERVADOS 
    | <a href="#" id="openPrivacy">Aviso de Privacidad</a>
</footer>

<!-- El Recuadro Oculto (Modal) -->
<div id="privacyModal" class="modal">
    <div class="modal-content">
        <span class="close-btn">&times;</span>
        <h2>AVISO DE PRIVACIDAD</h2>
        <p><strong>STC LIVE S.A. DE C.V.</strong>, es responsable del uso y protección de sus datos personales...</p>
        <p>Con fundamento en la Ley Federal de Protección de Datos Personales en Posesión de Particulares (en lo sucesivo, la “Ley”), STC LIVE México, S.A. de C.V. (en lo sucesivo, denominada como “STC LIVE”), reconoce que la Privacidad y la Seguridad de su información personal es un derecho humano muy importante para Usted, por lo cual STC LIVE está comprometida a resguardar su información personal con los más altos índices de seguridad legal, tecnológica y administrativa. Asimismo, a no vender, alquilar, compartir o divulgar su información personal a terceros con fines ilícitos o contrarios a los de su titular. De acuerdo a lo anterior, el presente Aviso de Privacidad se aplica a toda la información, incluyendo la información personal recopilada por STC LIVE.  
<strong>Información recopilada.</strong>
En general, STC LIVE recopila datos personales de Usted que se relacionen o deriven, ya sea de manera directa o indirecta, con la relación que lo une con STC LIVE, así como con el desempeño de las actividades necesarias para dar cumplimiento a las obligaciones acordadas. En la recopilación y tratamiento de dicha información, STC LIVE se compromete y obliga a observar y cumplir los principios de licitud, consentimiento, información, calidad, finalidad, lealtad, proporcionalidad y responsabilidad. De esta forma, los datos que STC LIVE le solicita son recabados de manera lícita conforme a la Ley, a través de la obtención de su consentimiento, recopilando única y exclusivamente los datos pertinentes y necesarios para dar cumplimiento a las obligaciones adquiridas. Asimismo, le comunicamos que STC LIVE tiene implementadas medidas de seguridad administrativas, técnicas y físicas para proteger sus datos personales, incluyendo los datos personales sensibles, a efecto de garantizar la confidencialidad y seguridad de la información que Usted nos proporciona. Los datos personales que usted proporcione a STC LIVE serán compilados y resguardados en una base de datos propiedad exclusiva de STC LIVE. 
<strong>Utilización de la información recopilada.</strong>  
La información recopilada por STC LIVE, se utiliza para los siguientes fines, según sea el caso: Clientes y personas interesadas en convertirse en nuevos clientes: la información es utilizada para realizar procesos de revisión documental, análisis de transportación, contacto y consulta. En caso de que ser cliente, se realizan procesos de llenado del formato de Solicitud de Alta, alta en nuestro sistema, elaboración de usuario, contacto, oferta de servicios, y aquellas que resulten estrictamente necesaria para dar cumplimiento a la relación que lo une con nosotros. Proveedores y personas interesadas en convertirse en nuevos proveedores: la información es utilizada para validación de tiempos, nombres y en general, verificar que la información proporcionada es verídica y correcta, así como contacto y consulta. Asimismo, en caso de que se requieran sus servicios, la información se utiliza para dar de alta en nuestro sistema y aquellas que resulten estrictamente necesaria para dar cumplimiento a la relación que lo une con nosotros. la información recopilada se utiliza para fines de contacto, registro de información en nuestra base de datos, creación de usuarios, alta ante las diferentes dependencias que como patrón nos obliga la legislación de México, actualizaciones y aquellas que resulten estrictamente necesaria para dar cumplimiento a la relación que lo une con nosotros. 
<strong>Consentimiento. </strong>
Al momento de proporcionar sus Datos Personales a STC LIVE, Usted ha sido informado del contenido del presente Aviso de Privacidad, por lo que manifiesta su conformidad con el mismo y otorga su consentimiento para que STC LIVE recopile y utilice sus Datos Personales de conformidad con los fines y términos del presente Aviso. Por su parte, al amparo del presente Aviso de Privacidad, Usted declara que los Datos Personales suministrados a STC LIVE son exactos, auténticos y completos, liberando a STC LIVE de cualquier responsabilidad relacionada al respecto, en virtud de que es el particular responsable de la exactitud, veracidad, autenticidad y vigencia de dichos Datos Personales. Ejercicio de los derechos ARCO. De conformidad con la Ley, le recordamos que en materia de Protección de Datos Personales Usted puede ejercer ante STC LIVE sus derechos denominados “ARCO” de acuerdo a lo siguiente: 
<strong>Acceso.</strong>
Es la facultad que tiene todo particular, de solicitar a STC LIVE le informe si en su Base de Datos tienen alguno de sus Datos Personales, así como de conocer el Aviso de Privacidad al que se sujeta el tratamiento de su información. 
<strong>Rectificación.</strong>
Es el derecho que los particulares tienen para que se corrijan sus Datos Personales en posesión de STC LIVE, cuando éstos sean incorrectos o incompletos.  
<strong>Cancelación.</strong>
Es la facultad que los particulares tienen de solicitar la cancelación de sus Datos Personales en posesión de STC LIVE. Lo anterior, da lugar a un periodo de bloqueo tras el cual procederá la supresión del (los) dato(s). 
<strong>Oposición.</strong>
Es la facultad de los particulares para solicitar a STC LIVE que se abstenga de realizar el tratamiento de sus Datos Personales en determinadas situaciones.  
<strong>Revocación del Consentimiento.</strong> 
El consentimiento puede ser revocado únicamente si no existe obligación pendiente por cumplir. Para esto, Usted deberá enviar la solicitud de revocación del consentimiento en atención a STC LIVE, a través de los medios que se señalan más adelante. De las solicitudes. El particular que desee ejercer cualquiera de sus derechos ARCO, podrá realizarlo a través de una solicitud que deberá cumplir, por lo menos, con los siguientes requisitos: El nombre del titular y correo u otro medio para comunicarle la respuesta a su solicitud; Los documentos que acrediten la identidad o, en su caso, la representación legal del titular; La descripción clara y precisa de los datos personales respecto de los que se busca ejercer alguno de los derechos antes mencionados, y Cualquier otro elemento o documento que facilite la localización de los datos personales. 
<strong>Transferencia de datos.</strong>
Salvo instrucción en contrario, como titular de los datos personales Usted consiente el tratamiento de su información por parte de STC LIVE, tanto dentro como fuera de los Estados Unidos Mexicanos, respecto de cualquier sociedad controladora, subsidiaria, matriz, sociedad del grupo y/o afiliadas bajo el control o en la que forme parte STC LIVE, así como aquellas sociedades con las que STC LIVE tenga relación contractual, las cuales operan bajo los mismos procesos y políticas internas. 
<strong>Contacto.</strong>
En caso de que tenga alguna pregunta o desee ejercer cualquiera de sus derechos “ARCO” arriba mencionados, puede contactarnos por los siguientes medios: 

<strong>Ciudad de México, México. Teléfono: 5661012927 
Correo electrónico: stclive@gmail.com
Cualquier modificación al presente Aviso de Privacidad, será notificado a través de nuestra página web:  
https://unexaggerative-katheryn-postscholastic.ngrok-free.dev/stc/2.php
ACTUALIZACIÓN 23 DE MAYO 2026 / SUJETO A CAMBIOS SIN PREVIO AVISO</strong>
</p>
    </div>
</div>

<!-- ================================================================
     MODAL: DETALLE DE LÍNEA METRO / METROBÚS / OTRAS
     ================================================================ -->
<div id="modal-linea-detalle">
    <div class="modal-linea-card">
        <button class="close-est" onclick="cerrarModalLinea()"><i class="fas fa-times"></i></button>
        <div id="ml-header" style="margin-bottom:16px;"></div>
        <div id="ml-simulador"></div>
        <div id="ml-info" style="margin-top:14px;"></div>
    </div>
</div>

<!-- ================================================================
     MODAL: DETALLE DE ESTACIÓN
     ================================================================ -->
<div id="modal-estacion">
    <div class="modal-estacion-card">
        <button class="close-est" onclick="cerrarModalEstacion()"><i class="fas fa-times"></i></button>
        <div class="modal-est-header">
            <div class="modal-est-dot" id="me-dot">M</div>
            <div class="modal-est-info">
                <h3 id="me-nombre">Estación</h3>
                <small id="me-linea">Línea —</small>
            </div>
        </div>
        <div id="me-cierre-banner" class="est-cerrada-banner" style="display:none;">
            <i class="fas fa-triangle-exclamation"></i> ESTACIÓN CERRADA TEMPORALMENTE
        </div>
        <div class="modal-est-grid">
            <div class="modal-est-cell"><span>Estado</span><b id="me-estado">—</b></div>
            <div class="modal-est-cell"><span>Próximo tren</span><b id="me-prox">—</b></div>
            <div class="modal-est-cell"><span>Afluencia</span><b id="me-afluencia">—</b></div>
            <div class="modal-est-cell"><span>Correspondencia</span><b id="me-corr">—</b></div>
        </div>
        <div class="modal-est-prox" id="me-info-extra">
            <i class="fas fa-info-circle" style="margin-right:5px;color:#3b82f6"></i>
            Información adicional de la estación.
            <div class="tren-en-camino" id="me-tren-badge" style="display:none;">
                <span style="width:6px;height:6px;background:#22c55e;border-radius:50%;display:inline-block;animation:parpadeo 0.8s infinite alternate;"></span>
                TREN APROXIMÁNDOSE
            </div>
        </div>
    </div>
</div>

<!-- ================================================================
     JAVASCRIPT
     ================================================================ -->
<script>

    /*
    
    function toggleMenu() {
    const container = document.getElementById('tut-tabla-viajes');
    // Alterna la clase 'open' para abrir y cerrar con clicks
    container.classList.toggle('close');
}  */

// ================================================================
//  TOGGLE MAESTRO — Abre/cierra el contenedor padre de todos los sistemas
// ================================================================
function toggleMaestro() {
    const maestro = document.getElementById('tut-maestro');
    if (maestro) maestro.classList.toggle('open');
}

// ================================================================
//  ESTACIONES CERRADAS — Fuente: anuncios oficiales STC
//  Cualquier estación listada aquí se muestra como CERRADA en
//  verEstacion() y en el track del simulador.
// ================================================================
const ESTACIONES_CERRADAS = {
    // Línea 12 — cierres históricos y actuales (tramo oriente)
    12: [
        'Nopalera', 'Zapotitlán', 'Tlaltenco', 'Tláhuac',
        'Olivos', 'Tezonco'
    ],
    // Línea 1 — rehabilitación (descomenta cuando aplique)
    // 1: ['Observatorio'],
    // Agrega más líneas y estaciones según anuncios oficiales:
    // 2: ['Xola'],
};

// ================================================================
//  DATOS DE LÍNEAS DE METRO
// ================================================================
const METRO_LINEAS = {
    1: {
        nombre: 'Línea 1', color: '#e1548a',
        terminal_a: 'Observatorio', terminal_b: 'Pantitlán',
        kmTotal: 18.7, frecuencia: '3 min',
        info: 'Primera línea del metro, opera desde 1969.',
        estaciones: ['Observatorio','Tacubaya','Juanacatlán','Chapultepec','Sevilla','Insurgentes','Cuauhtémoc','Balderas','Salto del Agua','Isabel la Católica','Pino Suárez','Merced','Candelaria','San Lázaro','Moctezuma','Balbuena','Boulevard Puerto Aéreo','Gómez Farías','Zaragoza','Pantitlán']
    },
    2: {
        nombre: 'Línea 2', color: '#0077c8',
        terminal_a: 'Cuatro Caminos', terminal_b: 'Tasqueña',
        kmTotal: 22.4, frecuencia: '3 min',
        info: 'Cruza la ciudad de norte a sur por el centro histórico.',
        estaciones: ['CuatroCaminos','Panteones','Tacuba','Cuitláhuac','Popotla','ColegioMilitar','Normal','SanCosme','Revolución','Hidalgo','BellasArtes','Allende','Zócalo','PinoSuárez','SanAntonioAbad','Chabacano','Viaducto','Xola','VilladeCortés','Nativitas','Portales','Ermita','GeneralAnaya','Tasqueña']
    },
    3: {
        nombre: 'Línea 3', color: '#7B8123',
        terminal_a: 'Indios Verdes', terminal_b: 'Universidad',
        kmTotal: 23.0, frecuencia: '3 min',
        info: 'Conecta el norte con el sur universitario de la CDMX.',
        estaciones: ['IndiosVerdes','Deportivo18deMarzo','Potrero','LaRaza','Tlatelolco','Guerrero','Hidalgo','Juárez','Balderas','PoderJudicial','HospitalGeneral','CentroMédico','Etiopía','Eugenia','DivisióndelNorte','Zapata','Coyoacán','ViverosDerechosHumanos','MiguelÁngeldeQuevedo','Copilco','Universidad']
    },
    4: {
        nombre: 'Línea 4', color: '#72c2d0',
        terminal_a: 'Martín Carrera', terminal_b: 'Santa Anita',
        kmTotal: 10.6, frecuencia: '4 min',
        info: 'Une oriente y nororiente de la ciudad.',
        estaciones: ['MartínCarrera','Talismán','Bondojito','Consulado','CanaldelNorte','Morelos','Candelaria','FrayServando','Jamaica','SantaAnita']
    },
    5: {
        nombre: 'Línea 5', color: '#f5e600',
        terminal_a: 'Politécnico', terminal_b: 'Pantitlán',
        kmTotal: 14.4, frecuencia: '4 min',
        info: 'Sirve la zona norte y conecta con el aeropuerto histórico.',
        estaciones: ['Politécnico','InstitutodelPetróleo','AutobusesdelNorte','LaRaza','Misterios','ValleGómez','Consulado','EduardoMolina','Aragón','Oceanía','TerminalAérea','Hangares','Pantitlán']
    },
    6: {
        nombre: 'Línea 6', color: '#e0393e',
        terminal_a: 'El Rosario', terminal_b: 'Martín Carrera',
        kmTotal: 17.0, frecuencia: '4 min',
        info: 'Conecta el norte de la CDMX de poniente a oriente.',
        estaciones: ['ElRosario','Tezozómoc','UAMAzcapotzalco','Ferrería','Norte45','Vallejo','InstitutodelPetróleo','Lindavista','Deportivo18deMarzo','LaVillaBasílica','MartínCarrera']
    },
    7: {
        nombre: 'Línea 7', color: '#e77d22',
        terminal_a: 'El Rosario', terminal_b: 'Barranca del Muerto',
        kmTotal: 19.4, frecuencia: '4 min',
        info: 'Atraviesa la ciudad de norte a sur por el poniente.',
        estaciones: ['ElRosario','AquilesSerdán','Camarones','Refinería','Tacuba','SanJoaquín','Polanco','Auditorio','Constituyentes','Tacubaya','SanPedrodelosPinos','SanAntonio','Mixcoac','BarrancadelMuerto']
    },
    8: {
        nombre: 'Línea 8', color: '#00a651',
        terminal_a: 'Garibaldi/Lagunilla', terminal_b: 'Constitución de 1917',
        kmTotal: 19.3, frecuencia: '4 min',
        info: 'Conecta el centro histórico con el oriente de la ciudad.',
        estaciones: ['GaribaldiLagunilla','BellasArtes','SanJuandeLetrán','SaltodelAgua','Doctores','Obrera','Chabacano','LaViga','SantaAnita','Coyuya','Iztacalco','Apatlaco','Aculco','Escuadrón201','Atlalilco','Iztapalapa','CerrodelaEstrella','UAM','Constituciónde1917']
    },
    9: {
        nombre: 'Línea 9', color: '#4d3c2e',
        terminal_a: 'Tacubaya', terminal_b: 'Pantitlán',
        kmTotal: 14.1, frecuencia: '4 min',
        info: 'Conecta poniente con oriente pasando por el centro.',
        estaciones: ['Tacubaya','Patriotismo','Chilpancingo','CentroMédico','LázaroCárdenas','Chabacano','Jamaica','Mixiuhca','Velódromo','CiudadDeportiva','Puebla','Pantitlán']
    },
    A: {
        nombre: 'Línea A', color: '#92278f',
        terminal_a: 'Pantitlán', terminal_b: 'La Paz',
        kmTotal: 31.0, frecuencia: '5 min',
        info: 'La línea más larga, conecta CDMX con el Estado de México.',
        estaciones: ['Pantitlán','AgrícolaOriental','CanaldeSanJuan','Telpacates','Guelatao','PeñónViejo','Acatitla','SantaMarta','LosReyes','LaPaz']
    },
    B: {
        nombre: 'Línea B', color: '#007360',
        terminal_a: 'Buenavista', terminal_b: 'Ciudad Azteca',
        kmTotal: 12.0, frecuencia: '5 min',
        info: 'Línea diagonal que sirve zonas de alta demanda al norte.',
        estaciones: ['Buenavista','Guerrero','Lagunilla','Tepito','Morelos','SanLázaro','RicardoFloresMagón','RomeroRubio','Oceanía','DeportivoOceanía','BosquedeAragón','VilladeAragón','Nezahualcóyotl','Impulsora','RíodelosRemedios','Múzquiz','Ecatepec','Olímpica','PlazaAragón','CiudadAzteca']
    },
    12: {
        nombre: 'Línea 12', color: '#e3c153',
        terminal_a: 'Mixcoac', terminal_b: 'Tláhuac',
        kmTotal: 24.7, frecuencia: '4 min',
        info: 'La Línea Dorada: tramo oriente con cierres temporales.',
        estaciones: ['Mixcoac','InsurgentesSur','Zapata','ParquedelosVenados','EjeCentral','Ermita','Mexicaltzingo','Atlalilco','Culhuacán','SanAndrésTomatlán','LomasEstrella','Calle11','PeriféricoOriente','Tezonco','Olivos','Nopalera','Zapotitlán','Tlaltenco','Tláhuac']
    }
};

// ================================================================
//  DATOS DE LÍNEAS METROBÚS
// ================================================================
const METROBUS_LINEAS = {
    1: {
        nombre: 'Metrobús L1', color: '#c0392b',
        terminal_a: 'Indios Verdes', terminal_b: 'El Caminero',
        kmTotal: 20.5, frecuencia: '5 min',
        info: 'Primer corredor BRT de CDMX inaugurado en 2005 por Insurgentes.',
        estaciones: ['Indios Verdes','Politécnico','Autobuses del Norte','Fortuna','La Villa','Tlatelolco','Buenavista','Revolución','Hidalgo','Bellas Artes','Salto del Agua','Doctores','Centro Médico','Sonora','Campeche','Insurgentes','Hamburgo','Florencia','Reforma','Sevilla','Patriotismo','Barranca del Muerto','Viveros','Miguel Ángel de Quevedo','Coyoacán','Parroquia','Altavista','La Bombilla','El Caminero']
    },
    2: {
        nombre: 'Metrobús L2', color: '#e67e22',
        terminal_a: 'Tacubaya', terminal_b: 'Tepalcates',
        kmTotal: 19.6, frecuencia: '6 min',
        info: 'Corredor Eje 4 Sur — une poniente con oriente.',
        estaciones: ['Tacubaya','Jalisco','Mixcoac','Barranca del Muerto','Parque Hundido','La Joya','Metro Eje 10','Ciudad Jardín','UAM Iztapalapa','Constitución','Tepalcates']
    },
    3: {
        nombre: 'Metrobús L3', color: '#8e44ad',
        terminal_a: 'Tenayuca', terminal_b: 'Etiopía/Plaza de la Transparencia',
        kmTotal: 17.2, frecuencia: '5 min',
        info: 'Eje 1 Poniente — une el norte con el centro-sur.',
        estaciones: ['Tenayuca','Periférico Norte','Vallejo Norte','La Raza Norte','Flores Magón','Ricardo Flores Magón','Tlatelolco','Guerrero','Hidalgo Sur','Bellas Artes Sur','Artes Plásticas','Dr. Lavista','Chilpancingo','Campeche','Sonora','Insurgentes Sur','Etiopía/Plaza de la Transparencia']
    },
    4: {
        nombre: 'Metrobús L4', color: '#27ae60',
        terminal_a: 'Buenavista', terminal_b: 'Aeropuerto T1',
        kmTotal: 10.9, frecuencia: '7 min',
        info: 'Conecta el centro con el Aeropuerto Internacional.',
        estaciones: ['Buenavista','Guerrero','Lagunilla','Tepito','Morelos','Moctezuma','Boulevard P. Aéreo','Hangares','Aeropuerto T1']
    },
    5: {
        nombre: 'Metrobús L5', color: '#2980b9',
        terminal_a: 'Río de los Remedios', terminal_b: 'Vaqueritos',
        kmTotal: 10.2, frecuencia: '6 min',
        info: 'Corredor norte — sirve la alcaldía Gustavo A. Madero.',
        estaciones: ['Río de los Remedios','Acueducto de Guadalupe','La Villa/Basílica','Deportivo 18 de Marzo','Lindavista','Politécnico','Indios Verdes','Alcaldía GAM','Vaqueritos']
    },
    6: {
        nombre: 'Metrobús L6', color: '#d4ac0d',
        terminal_a: 'El Rosario', terminal_b: 'Martín Carrera',
        kmTotal: 11.7, frecuencia: '6 min',
        info: 'Corredor norte oriente — circuito urbano nororiente.',
        estaciones: ['El Rosario','Vallejo','Instituto del Petróleo','Lindavista','Deportivo 18 de Marzo','La Villa','Tepeyac','Martín Carrera']
    },
    7: {
        nombre: 'Metrobús L7', color: '#1abc9c',
        terminal_a: 'Tláloc', terminal_b: 'Auditorio',
        kmTotal: 4.4, frecuencia: '8 min',
        info: 'Corredor Reforma–Chapultepec, conecta museos con el Auditorio.',
        estaciones: ['Tláloc','Museos','Chapultepec','Gandhi','Constituyentes','Auditorio']
    }
};

// ================================================================
//  DATOS DE LÍNEAS CABLEBÚS
// ================================================================
const CABLEBUS_LINEAS = {
    1: {
        nombre: 'Cablebús L1', color: '#e5007d',
        terminal_a: 'Indios Verdes', terminal_b: 'Cuautepec',
        kmTotal: 4.9, frecuencia: '8 min',
        info: 'Une Metro Indios Verdes con la cima de Cuautepec.',
        estaciones: ['Indios Verdes','Tlalpexco','Santa Bárbara','Tlalpexco Norte','La Presa','Arbolillo','El Meyahual','Paraje San Juan','Cuautepec']
    },
    2: {
        nombre: 'Cablebús L2', color: '#007dc3',
        terminal_a: 'Santa Martha', terminal_b: 'UAM Iztapalapa',
        kmTotal: 5.5, frecuencia: '8 min',
        info: 'Conecta Santa Martha con UAM Iztapalapa en el oriente.',
        estaciones: ['Santa Martha','Acueducto','Ermita Iztapalapa','Escuadrón 201','Eje 6 Sur','Chimalhuacán','Cerro de la Estrella','Constitución de 1917','Culhuacán','UAM Iztapalapa']
    },
    3: {
        nombre: 'Cablebús L3', color: '#f37021',
        terminal_a: 'Constitución de 1917', terminal_b: 'Periférico Oriente',
        kmTotal: 4.2, frecuencia: '9 min',
        info: 'Conecta la zona de Iztapalapa con Periférico Oriente.',
        estaciones: ['Constitución de 1917','Vista Hermosa','Acueducto Oriente','Las Torres','Periférico Oriente Norte','Peñón Nuevo','Rincón Oriente','Periférico Oriente']
    }
};

// ================================================================
//  DATOS DE LÍNEAS TROLEBÚS / STE
// ================================================================
const TROLEBUS_LINEAS = {
    1: {
        nombre: 'Trolebús L1', color: '#0033a0',
        terminal_a: 'Tacubaya', terminal_b: 'Chapultepec',
        kmTotal: 1.7, frecuencia: '10 min',
        info: 'Corredor corto Tacubaya–Chapultepec.',
        estaciones: ['Tacubaya','Chapultepec']
    },
    2: {
        nombre: 'Trolebús L2', color: '#1e3a8a',
        terminal_a: 'Buenavista', terminal_b: 'San Lázaro',
        kmTotal: 8.8, frecuencia: '8 min',
        info: 'Conecta Buenavista con San Lázaro por Reforma.',
        estaciones: ['Buenavista','Reforma Norte','Hidalgo','Alameda','Bellas Artes','Zócalo','Pino Suárez','Merced','San Lázaro']
    },
    3: {
        nombre: 'Trolebús L3', color: '#4c1d95',
        terminal_a: 'Etiopía', terminal_b: 'San Lázaro',
        kmTotal: 6.4, frecuencia: '8 min',
        info: 'Corredor Eje 1 Poniente Norte.',
        estaciones: ['Etiopía','Centro Médico','Hospital General','Salto del Agua','Isabel la Católica','Correo Mayor','Loreto','San Lázaro']
    },
    4: {
        nombre: 'Trolebús L4', color: '#065f46',
        terminal_a: 'San Lázaro', terminal_b: 'Peñón Viejo',
        kmTotal: 5.5, frecuencia: '10 min',
        info: 'Conecta el oriente con San Lázaro.',
        estaciones: ['San Lázaro','Cantera','Aeropuerto','Hangares','Terminal Aérea','Peñón de los Baños','Peñón Viejo']
    },
    5: {
        nombre: 'Trolebús L5', color: '#78350f',
        terminal_a: 'Zócalo', terminal_b: 'Xochimilco',
        kmTotal: 22.0, frecuencia: '12 min',
        info: 'Electrobus — el más largo de América Latina.',
        estaciones: ['Zócalo','Pino Suárez','Salto del Agua','Niños Héroes','Xola','Villa de Cortés','Nativitas','Portales','Ermita','General Anaya','Tasqueña','Estadio Azteca','Huipulco','Pericoapa','Tepepan','San Luis Tlaxialtemalco','Tulyehualco','La Noria','Embarcadero','Xochimilco']
    },
    6: {
        nombre: 'Trolebús L6', color: '#374151',
        terminal_a: 'Pino Suárez', terminal_b: 'Huipulco',
        kmTotal: 19.0, frecuencia: '10 min',
        info: 'Corredor poniente Eje Central hacia el sur.',
        estaciones: ['Pino Suárez','Fray Servando','Chabacano','Jamaica','Santa Anita','Viaducto','Xola','Villa de Cortés','Nativitas','Portales','Ermita','General Anaya','Tasqueña','Tenorios','Miramontes','El Vergel','Huipulco']
    }
};

// ================================================================
//  DATOS DE LÍNEAS RTP
// ================================================================
const RTP_LINEAS = {
    'M1-9C':  { nombre: 'RTP M1 · 9C',   color: '#7ab800', terminal_a: 'C.C. Santa Fe', terminal_b: 'Puerta Grande',           kmTotal: 12.5, frecuencia: '15 min', info: 'Módulo 1 Poniente.', estaciones: ['C.C. Santa Fe','Vasco de Quiroga','Camino a Santa Fe','Boulevard de la Luz','Torres Bicentenario','Centro Comercial','Puerta Grande'] },
    'M1-76':  { nombre: 'RTP M1 · 76',   color: '#5a8a00', terminal_a: 'C.C. Santa Fe',  terminal_b: 'Metro Auditorio',         kmTotal: 18.0, frecuencia: '15 min', info: 'Módulo 1 Poniente por Las Palmas.', estaciones: ['C.C. Santa Fe','Vasco de Quiroga','Las Palmas','Tecamachalco','Bosques de las Lomas','Polanco','Metro Auditorio'] },
    'M1-76A': { nombre: 'RTP M1 · 76-A', color: '#84cc16', terminal_a: 'Santa Fe',       terminal_b: 'Metro Auditorio',         kmTotal: 15.0, frecuencia: '15 min', info: 'Módulo 1 Poniente variante.', estaciones: ['Santa Fe','Pedregal de Santa Fe','Xicotencatl','Palmas','Auditorio'] },
    'M1-110': { nombre: 'RTP M1 · 110',  color: '#4d7c0f', terminal_a: 'Metro Tacubaya', terminal_b: 'Chimalpa',                kmTotal: 22.0, frecuencia: '20 min', info: 'Módulo 1 Poniente, la más larga.', estaciones: ['Metro Tacubaya','Constituyentes','Santa Fe','La Loma','Cuajimalpa','Chimalpa'] },
    'M2-36':  { nombre: 'RTP M2 · 36',   color: '#ca8a04', terminal_a: 'Zócalo',          terminal_b: 'La Paz',                  kmTotal: 20.0, frecuencia: '15 min', info: 'Módulo 2 Oriente.', estaciones: ['Zócalo','Pino Suárez','Anillo de Circunvalación','Iztapalapa','La Paz'] },
    'M2-36A': { nombre: 'RTP M2 · 36-A', color: '#b45309', terminal_a: 'Zócalo',          terminal_b: 'Iztapalapa',              kmTotal: 18.5, frecuencia: '15 min', info: 'Módulo 2 Oriente variante.', estaciones: ['Zócalo','Merced','Jamaica','Santa Anita','Iztapalapa'] },
    'M2-50':  { nombre: 'RTP M2 · 50',   color: '#d97706', terminal_a: 'Santa Anita',     terminal_b: 'Pantitlán',               kmTotal: 8.0,  frecuencia: '12 min', info: 'Módulo 2 Oriente corto.', estaciones: ['Santa Anita','Coyuya','Iztacalco','Pantitlán'] },
    'M2-116': { nombre: 'RTP M2 · 116',  color: '#f59e0b', terminal_a: 'Metro Constitución', terminal_b: 'Tláhuac',             kmTotal: 16.0, frecuencia: '18 min', info: 'Módulo 2 hacia Tláhuac.', estaciones: ['Metro Constitución','Atlalilco','Iztapalapa','Cerro de la Estrella','Tláhuac'] },
    'M3-3':   { nombre: 'RTP M3 · 3',    color: '#0f766e', terminal_a: 'Indios Verdes',   terminal_b: 'Tlalnepantla',            kmTotal: 14.0, frecuencia: '15 min', info: 'Módulo 3 Norte.', estaciones: ['Indios Verdes','Acueducto','Vallejo','Lechería','Tlalnepantla'] },
    'M3-23':  { nombre: 'RTP M3 · 23',   color: '#0e7490', terminal_a: 'Vallejo',          terminal_b: 'Aeropuerto',              kmTotal: 12.0, frecuencia: '15 min', info: 'Módulo 3 Norte al aeropuerto.', estaciones: ['Vallejo','Instituto del Petróleo','Lindavista','Politécnico','Aeropuerto'] },
    'M3-105': { nombre: 'RTP M3 · 105',  color: '#1d4ed8', terminal_a: 'La Raza',          terminal_b: 'Cuautitlán',              kmTotal: 22.0, frecuencia: '20 min', info: 'Módulo 3 Norte largo.', estaciones: ['La Raza','Vallejo','Cuautitlán Izcalli','Cuautitlán'] }
};

// ================================================================
//  CORRESPONDENCIAS
// ================================================================
const CORRESPONDENCIAS = {
    'Pantitlán':                    'L1, L5, L9, LA',
    'Tacubaya':                     'L1, L7, L9, MB L2',
    'Pino Suárez':                  'L1, L2',
    'Balderas':                     'L1, L3',
    'Salto del Agua':               'L1, L8',
    'Cuatro Caminos':               'L2',
    'Tacuba':                       'L2, L7',
    'Hidalgo':                      'L2, L3',
    'Bellas Artes':                 'L2, L8',
    'Chabacano':                    'L2, L8, L9',
    'Tasqueña':                     'L2, Tren Ligero',
    'Indios Verdes':                'L3, MB L5',
    'La Raza':                      'L3, L5',
    'Centro Médico':                'L3, L9',
    'Etiopía/Plaza de la Transparencia': 'L3, MB L3',
    'Deportivo 18 de Marzo':        'L3, L6',
    'Instituto del Petróleo':       'L5, L6',
    'Politécnico':                  'L3, L5',
    'El Rosario':                   'L6, L7',
    'Martín Carrera':               'L4, L6',
    'Mixcoac':                      'L7, L12',
    'Santa Anita':                  'L4, L8, L9',
    'Jamaica':                      'L4, L9',
    'Oceanía':                      'L5, L9',
    'Garibaldi/Lagunilla':          'L8, MB L4',
    'Buenavista':                   'Tren Suburbano, MB L3',
    'Tlatelolco':                   'L3',
    'Insurgentes':                  'L1',
    'Insurgentes Sur':              'L3, L12',
    'La Villa/Basílica':            'L6, MB L5',
};

const AFLUENCIAS = ['Alta','Alta','Media-Alta','Media','Media-Alta','Alta','Baja','Media'];

// ================================================================
//  HORARIOS DE SERVICIO
// ================================================================
function getHorarioServicio() {
    const ahora = new Date();
    const dia = ahora.getDay();
    const minutos = ahora.getHours() * 60 + ahora.getMinutes();
    let inicio, etiqueta;
    if (dia === 0)      { inicio = 7 * 60; etiqueta = 'Dom/Fest 07:00–00:00'; }
    else if (dia === 6) { inicio = 6 * 60; etiqueta = 'Sábado 06:00–00:00'; }
    else                { inicio = 5 * 60; etiqueta = 'Lun–Vie 05:00–00:00'; }
    const enServicio = minutos >= inicio && minutos < 24 * 60;
    return { enServicio, etiqueta };
}

// ================================================================
//  HELPERS
// ================================================================
function logoSlug(nombre) {
    return nombre.toLowerCase()
        .normalize('NFD').replace(/[\u0300-\u036f]/g, '')
        .replace(/[^a-z0-9]+/g, '_')
        .replace(/^_+|_+$/, '') + '.png';
}

const VEL_METRO_KMH = 36;
const VEL_MB_KMH    = 18;
const VEL_CB_KMH    = 12;
const VEL_TE_KMH    = 16;

function segsTramo(kmTotal, n, velocidad) {
    if (n < 2) return 180;
    const kmTramo = kmTotal / (n - 1);
    return Math.max(30, Math.round((kmTramo / velocidad) * 3600));
}

// ================================================================
//  SIMULADOR DE TRENES / UNIDADES
// ================================================================
function getTrenesActivos(kmTotal, estaciones, freqMin, velocidad) {
    const { enServicio } = getHorarioServicio();
    if (!enServicio) return [];
    const n = estaciones.length;
    const freqSeg  = freqMin * 60;
    const segTramo = segsTramo(kmTotal, n, velocidad);
    const durTotal = (n - 1) * segTramo;
    const ahora = new Date();
    const segDia = ahora.getHours() * 3600 + ahora.getMinutes() * 60 + ahora.getSeconds();
    const dia = ahora.getDay();
    let inicioSeg;
    if (dia === 0)      inicioSeg = 7 * 3600;
    else if (dia === 6) inicioSeg = 6 * 3600;
    else                inicioSeg = 5 * 3600;
    const trenes = [];
    for (let salida = inicioSeg; salida <= segDia; salida += freqSeg) {
        const viajando = segDia - salida;
        if (viajando > durTotal) continue;
        const posFloat = viajando / segTramo;
        if (posFloat < n) trenes.push({ dir: 1, posFloat });
    }
    const desfase = Math.floor(freqSeg / 2);
    for (let salida = inicioSeg + desfase; salida <= segDia; salida += freqSeg) {
        const viajando = segDia - salida;
        if (viajando > durTotal) continue;
        const posFloat = (n - 1) - viajando / segTramo;
        if (posFloat >= 0) trenes.push({ dir: -1, posFloat });
    }
    return trenes;
}

// ================================================================
//  RENDER VÍA — con soporte a estaciones cerradas
// ================================================================
function renderVia(lineaData, trenes, tipo, lineaNum, dir, color, emoji) {
    const est = lineaData.estaciones;
    const n   = est.length;
    const cerradasLinea = ESTACIONES_CERRADAS[lineaNum] || [];
    const trenesDir = trenes.filter(t => t.dir === dir);
    const dotActivo = new Set();
    trenesDir.forEach(t => {
        const idx  = Math.round(t.posFloat);
        const frac = t.posFloat - Math.floor(t.posFloat);
        if (frac < 0.15 || frac > 0.85) dotActivo.add(idx);
    });
    const orden = dir === 1
        ? Array.from({length: n}, (_, i) => i)
        : Array.from({length: n}, (_, i) => n - 1 - i);
    let html = `<div class="estaciones-track"><div class="estaciones-row" style="gap:0;"><div class="track-line" style="background:${color};"></div>`;
    orden.forEach((realIdx, visualPos) => {
        const estNombre  = est[realIdx];
        const logo       = logoSlug(estNombre);
        const esTermA    = realIdx === 0;
        const esTermB    = realIdx === n - 1;
        const esTerminal = esTermA || esTermB;
        const tieneEnDot = dotActivo.has(realIdx);
        const esCerrada  = cerradasLinea.includes(estNombre);
        let tramoPct = -1;
        trenesDir.forEach(t => {
            const fraccion = dir === 1 ? t.posFloat - realIdx : realIdx - t.posFloat;
            if (fraccion > 0 && fraccion < 1) {
                const pct = Math.round(fraccion * 100);
                if (tramoPct < 0 || pct > tramoPct) tramoPct = pct;
            }
        });
        const tramoId = `tramo-${tipo}-l${lineaNum}-d${dir === 1 ? 'i' : 'r'}-${realIdx}`;
        html += `
            <div class="estacion-item${esCerrada ? ' cerrada' : ''}"
                 onclick="abrirEstacion('${estNombre.replace(/'/g,"\\'")}', '${lineaNum}', ${realIdx}, '${tipo}')"
                 style="position:relative;">
                ${tieneEnDot && !esCerrada ? `<span class="tren-emoji-encima">${emoji}</span>` : ''}
                <img src="${logo}" alt="" class="est-logo" onerror="this.style.display='none'">
                <div class="estacion-dot ${(tieneEnDot && !esCerrada) ? 'tiene-tren' : ''}" style="border-color:${color};"></div>
                <div class="estacion-nombre" style="${esTerminal ? 'color:white;font-weight:800;' : ''}">
                    ${estNombre}${esCerrada ? ' 🚫' : ''}
                </div>
                ${(tramoPct >= 0 && visualPos < n - 1 && !esCerrada) ? `
                <div id="${tramoId}" style="position:absolute;top:22px;left:50%;width:72px;height:5px;z-index:3;pointer-events:none;">
                    <div class="tren-en-tramo" style="width:${tramoPct}%;"></div>
                    <span class="tren-emoji-tramo" style="left:${tramoPct}%;">${emoji}</span>
                </div>` : ''}
            </div>`;
    });
    html += `</div></div>`;
    return html;
}

// ================================================================
//  CONSTRUIR SIMULADOR COMPLETO (2 vías)
// ================================================================
function construirSimulador(lineaData, tipo, lineaNum) {
    const velMap = { metro: VEL_METRO_KMH, mb: VEL_MB_KMH, cablebus: VEL_CB_KMH, trolebus: VEL_TE_KMH, rtp: VEL_MB_KMH };
    const emojiMap = { metro: '🚇', mb: '🚌', cablebus: '🚡', trolebus: '🚎', rtp: '🚍' };
    const frecuenciaMin = parseInt(lineaData.frecuencia);
    const velocidad = velMap[tipo] || VEL_MB_KMH;
    const emoji     = emojiMap[tipo] || '🚌';
    const color     = lineaData.color;
    const n         = lineaData.estaciones.length;
    const termA     = lineaData.terminal_a;
    const termB     = lineaData.terminal_b;
    const trenes = getTrenesActivos(lineaData.kmTotal, lineaData.estaciones, frecuenciaMin, velocidad);
    const trenIda     = trenes.filter(t => t.dir ===  1).length;
    const trenRegreso = trenes.filter(t => t.dir === -1).length;
    const { enServicio } = getHorarioServicio();
    let html = `<div class="sim-doble-via">`;
    html += `<div class="sim-via-label"><span class="via-arrow">➡</span><span> ${termA} → ${termB}</span><span style="margin-left:auto;color:${enServicio ? '#22c55e':'#ef4444'};">${emoji} ${trenIda} en ruta</span></div>`;
    html += renderVia(lineaData, trenes, tipo, lineaNum, 1, color, emoji);
    html += `<div class="via-separator"></div>`;
    html += `<div class="sim-via-label"><span class="via-arrow">⬅</span><span> ${termB} → ${termA}</span><span style="margin-left:auto;color:${enServicio ? '#22c55e':'#ef4444'};">${emoji} ${trenRegreso} en ruta</span></div>`;
    html += renderVia(lineaData, trenes, tipo, lineaNum, -1, color, emoji);
    html += `</div>`;
    return html;
}

// ================================================================
//  TICK — actualiza simulador cada 5 segundos
// ================================================================
let modalLineaActual = 0;
let modalTipoActual  = 'metro';
let _simInterval     = null;

function iniciarTickSimulador() {
    if (_simInterval) clearInterval(_simInterval);
    _simInterval = setInterval(() => {
        if (!modalLineaActual) return;
        const lineasMap = {
            metro:    METRO_LINEAS,
            mb:       METROBUS_LINEAS,
            cablebus: CABLEBUS_LINEAS,
            trolebus: TROLEBUS_LINEAS,
            rtp:      RTP_LINEAS
        };
        const lineaData = lineasMap[modalTipoActual] ? lineasMap[modalTipoActual][modalLineaActual] : null;
        if (!lineaData) return;
        document.getElementById('ml-simulador').innerHTML = construirSimulador(lineaData, modalTipoActual, modalLineaActual);
    }, 5000);
}
iniciarTickSimulador();

// ================================================================
//  FUNCIÓN GENÉRICA PARA ABRIR MODAL DE CUALQUIER LÍNEA
// ================================================================
function abrirModalLinea(lineaData, tipo, lineaNum) {
    if (!lineaData) return;
    modalLineaActual = lineaNum;
    modalTipoActual  = tipo;
    const { enServicio, etiqueta } = getHorarioServicio();
    const n = lineaData.estaciones.length;
    const kmTramo = (lineaData.kmTotal / (n - 1)).toFixed(2);
    const unidad = (tipo === 'metro') ? 'estaciones' : 'paradas';
    const cerradasLinea = ESTACIONES_CERRADAS[lineaNum] || [];
    const tieneCierres = cerradasLinea.length > 0;

    document.getElementById('modal-linea-detalle').classList.add('abierto');
    document.getElementById('ml-header').innerHTML = `
        <div style="display:flex;align-items:center;gap:14px;margin-bottom:4px;">
            <div style="width:50px;height:50px;border-radius:14px;background:${lineaData.color};display:flex;align-items:center;justify-content:center;font-size:1.3rem;font-weight:900;color:white;box-shadow:0 4px 18px ${lineaData.color}88;flex-shrink:0;">${lineaNum}</div>
            <div>
                <h2 style="margin:0;color:#000000;font-size:1.1rem;">${lineaData.nombre}</h2>
                <p style="margin:0;color:#64748b;font-size:0.78rem;">
                    ${lineaData.terminal_a} <i class="fas fa-arrow-right" style="margin:0 4px;color:${lineaData.color}"></i> ${lineaData.terminal_b} — ${n} ${unidad}
                </p>
            </div>
            <div style="margin-left:auto;text-align:right;">
                <div class="sim-live-badge" style="${enServicio ? '' : 'border-color:#ef4444;color:#ef4444;'}">
                    <div class="sim-dot" style="${enServicio ? '' : 'background:#ef4444;animation:none;'}"></div>
                    ${enServicio ? 'SERVICIO' : 'CERRADO'}
                </div>
                <div style="font-size:0.65rem;color:#64748b;margin-top:4px;">Frec: ${lineaData.frecuencia} · ~${kmTramo} km/tramo</div>
            </div>
        </div>
        ${tieneCierres ? `<div style="background:rgba(239,68,68,0.12);border:1px solid #ef4444;border-radius:8px;padding:7px 10px;font-size:0.75rem;color:#ef4444;font-weight:700;margin-top:8px;"><i class="fas fa-triangle-exclamation"></i> CIERRES TEMPORALES: ${cerradasLinea.join(', ')}</div>` : ''}
        <p style="font-size:0.78rem;color:#000000;margin:8px 0 0;">${lineaData.info} &nbsp;·&nbsp; <span style="color:#f59e0b;">${etiqueta}</span></p>
    `;
    document.getElementById('ml-simulador').innerHTML = construirSimulador(lineaData, tipo, lineaNum);
    document.getElementById('ml-info').innerHTML = `
        <div style="display:flex;gap:10px;flex-wrap:wrap;">
            <div class="stat-chip"><i class="fas fa-road" style="color:${lineaData.color}"></i><b>${lineaData.kmTotal} km</b></div>
            <div class="stat-chip"><i class="fas fa-clock" style="color:${lineaData.color}"></i>Frec. <b>${lineaData.frecuencia}</b></div>
            <div class="stat-chip"><i class="fas fa-circle-${enServicio ? 'check' : 'xmark'}" style="color:${enServicio ? '#22c55e' : '#ef4444'}"></i><b style="color:${enServicio ? '#22c55e' : '#ef4444'}">${enServicio ? 'OPERANDO' : 'CERRADO'}</b></div>
        </div>
        <p style="font-size:0.72rem;color:#000000;margin-top:10px;">
            <i class="fas fa-info-circle" style="margin-right:4px;color:#3b82f6"></i>
            Haz clic en cualquier ${unidad.slice(0,-1)} para ver su información detallada. Las marcadas con 🚫 están cerradas temporalmente.
        </p>
    `;
}

function abrirLineaMetro(num)    { abrirModalLinea(METRO_LINEAS[String(num)],    'metro',    String(num)); }
function abrirLineaMetrobus(num) { abrirModalLinea(METROBUS_LINEAS[num],         'mb',        num); }
function abrirLineaCablebus(num) { abrirModalLinea(CABLEBUS_LINEAS[num],         'cablebus',  num); }
function abrirLineaTrolebus(num) { abrirModalLinea(TROLEBUS_LINEAS[num],         'trolebus',  num); }
function abrirLineaRTP(clave)    { abrirModalLinea(RTP_LINEAS[clave],            'rtp',       clave); }

function cerrarModalLinea() {
    document.getElementById('modal-linea-detalle').classList.remove('abierto');
    modalLineaActual = 0;
}

// ================================================================
//  ABRIR MODAL ESTACIÓN — con soporte de cierres
// ================================================================
// ================================================================
//  ABRIR MODAL ESTACIÓN — Sincronizado Milimétricamente con DB
// ================================================================
function abrirEstacion(nombre, lineaNum, idx, tipo) {
    cerrarModalEstacion();
    const claveLinea = String(lineaNum);
    const lineasMap = {
        metro:    METRO_LINEAS,
        mb:       METROBUS_LINEAS,
        cablebus: CABLEBUS_LINEAS,
        trolebus: TROLEBUS_LINEAS,
        rtp:      RTP_LINEAS
    };
    const L = lineasMap[tipo] ? lineasMap[tipo][claveLinea] : null;
    if (!L) return;

    const modal = document.getElementById('modal-estacion');
    modal.style.display = 'flex';
    setTimeout(() => { modal.classList.add('abierto'); }, 10);

    const { enServicio, etiqueta } = getHorarioServicio();
    const velMap = { metro: VEL_METRO_KMH, mb: VEL_MB_KMH, cablebus: VEL_CB_KMH, trolebus: VEL_TE_KMH, rtp: VEL_MB_KMH };
    const velocidad = velMap[tipo] || VEL_MB_KMH;
    const freqMin = parseInt(L.frecuencia);
    const n = L.estaciones.length;
    const esTerminal = (idx === 0 || idx === n - 1);
    const corr = CORRESPONDENCIAS[nombre] || 'Ninguna';
    const afluencia = AFLUENCIAS[idx % AFLUENCIAS.length];

    const cerradasLinea = ESTACIONES_CERRADAS[lineaNum] || [];
    const esCerrada = cerradasLinea.includes(nombre);

    const banner = document.getElementById('me-cierre-banner');
    if (banner) banner.style.display = esCerrada ? 'flex' : 'none';

    let proxTren = '—';
    let hayTrenCerca = false;

    if (esCerrada) {
        proxTren = 'Estación cerrada';
    } else if (!enServicio) {
        proxTren = 'Servicio cerrado';
    } else {
        const trenes  = getTrenesActivos(L.kmTotal, L.estaciones, freqMin, velocidad);
        const segTramo = segsTramo(L.kmTotal, n, velocidad);
        let minSeg = Infinity;
        trenes.forEach(t => {
            let distTramos;
            if (t.dir === 1) {
                distTramos = idx - t.posFloat;
                if (distTramos < 0) distTramos += n;
            } else {
                distTramos = t.posFloat - idx;
                if (distTramos < 0) distTramos += n;
            }
            const seg = distTramos * segTramo;
            if (seg >= 0 && seg < minSeg) minSeg = seg;
        });
        if (minSeg === Infinity || minSeg > 3600) {
            proxTren = 'Sin datos';
        } else if (minSeg < 60) {
            proxTren = Math.max(0, Math.round(minSeg)) + ' seg';
            hayTrenCerca = true;
        } else {
            const min = Math.ceil(minSeg / 60);
            proxTren = min + ' min';
            hayTrenCerca = min <= 3;
        }
    }

    const meDot = document.getElementById('me-dot');
    meDot.innerHTML = '';
    meDot.style.background = 'transparent';
    const nombreArchivo = nombre.toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "").replace(/\s+/g, '');
    meDot.innerHTML = `<img src="${nombreArchivo}.png" alt="${nombre}" style="width:100%;height:100%;object-fit:contain;">`;

    document.getElementById('me-nombre').textContent = nombre;
    document.getElementById('me-linea').textContent  = L.nombre + (esTerminal ? ' — TERMINAL' : ` — Parada ${idx + 1} de ${n}`);

    const estadoEl = document.getElementById('me-estado');
    if (esCerrada) {
        estadoEl.textContent = 'CERRADA ✗';
        estadoEl.className = 'cerrada';
    } else {
        estadoEl.textContent = enServicio ? 'ABIERTA ✓' : 'CERRADA ✗';
        estadoEl.className = '';
    }

    document.getElementById('me-prox').textContent = proxTren;
    document.getElementById('me-afluencia').textContent = esCerrada ? '—' : afluencia;
    document.getElementById('me-corr').textContent = corr;

    const badge = document.getElementById('me-tren-badge');
    badge.style.display = (hayTrenCerca && !esCerrada) ? 'inline-flex' : 'none';

    // Guardar estructura base estática en variable
    const infoBaseHTML = `
        <i class="fas fa-info-circle" style="margin-right:5px;color:#3b82f6"></i>
        ${esCerrada
            ? `<b style="color:#ef4444">ESTACIÓN FUERA DE SERVICIO</b> — Por obras de rehabilitación.`
            : esTerminal
                ? `<b style="color:#f59e0b">TERMINAL</b> — Cabecera de línea.<br>Horario: <b>${etiqueta}</b>`
                : `Posición: parada <b>${idx + 1}</b> de <b>${n}</b> en la ${L.nombre}.`
        }
        <br>Correspondencias: <b style="color:#38bdf8">${corr}</b>
        ${(hayTrenCerca && !esCerrada) ? `
        <div class="tren-en-camino" style="display:inline-flex;margin-top:6px;">
            <span style="width:6px;height:6px;background:#22c55e;border-radius:50%;display:inline-block;animation:parpadeo 0.8s infinite alternate;"></span>
            &nbsp;${tipo === 'metro' ? 'TREN' : 'UNIDAD'} APROXIMÁNDOSE — ${proxTren}
        </div>` : ''}
    `;

    const infoExtraContenedor = document.getElementById('me-info-extra');
    infoExtraContenedor.innerHTML = infoBaseHTML;

    // CONSULTA ASÍNCRONA A LA BASE DE DATOS USANDO LA VARIABLE NORMALIZADA
    fetch(`api_quejas.php?llave=${nombreArchivo}`)
        .then(response => response.json())
        .then(reportesEstacion => {
            if (Array.isArray(reportesEstacion) && reportesEstacion.length > 0) {
                let quejasHTML = `
                    <div style="margin-top: 12px; border-top: 1px dashed #ef4444; padding-top: 8px;">
                        <b style="color: #ef4444; display: flex; align-items: center; gap: 5px; font-size:0.85rem; margin-bottom:5px;">
                            <i class="fas fa-exclamation-circle"></i> REPORTES CIUDADANOS (${reportesEstacion.length}):
                        </b>
                        <div style="display: flex; flex-direction: column; gap: 4px; max-height: 90px; overflow-y: auto;">
                `;
                
                reportesEstacion.forEach(rep => {
                    quejasHTML += `
                        <div style="background: rgba(239, 68, 68, 0.15); border: 1px solid rgba(239, 68, 68, 0.3); padding: 5px 8px; border-radius: 4px; font-size: 0.75rem; text-align: left;">
                            <div style="display:flex; justify-content:space-between; color:#fca5a5; margin-bottom:2px;">
                                <span style="text-transform: uppercase; font-weight:bold;">⚠️ Incidencia</span>
                                <span style="font-family:monospace; color:#ef4444;">${rep.hora || '--:--'}</span>
                            </div>
                            <span style="color:#f1f5f9;">${rep.descripcion}</span>
                        </div>
                    `;
                });

                quejasHTML += `</div></div>`;
                infoExtraContenedor.innerHTML = infoBaseHTML + quejasHTML;
            }
        })
        .catch(err => console.error("Error al recuperar quejas de la DB:", err));
}

function cerrarModalEstacion() {
    const modal = document.getElementById('modal-estacion');
    modal.classList.remove('abierto');
    setTimeout(() => { if (!modal.classList.contains('abierto')) modal.style.display = 'none'; }, 250);
}

document.getElementById('modal-estacion').addEventListener('click', function(e) {
    if (e.target === this) cerrarModalEstacion();
});

// ================================================================
//  RELOJ
// ================================================================
function actualizarReloj() {
    const ahora = new Date();
    const reloj = document.getElementById('reloj-digital');
    const fecha = document.getElementById('fecha-actual');
    if (reloj) reloj.textContent = ahora.toLocaleTimeString('es-MX', { hour12: false });
    if (fecha) fecha.textContent = ahora.toLocaleDateString('es-MX', { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' });
}
setInterval(actualizarReloj, 1000);
actualizarReloj();

// ================================================================
//  CAMBIAR CANAL DE ANUNCIOS (X / NITTER)
//  Aplicado también a los botones de cada sección de transporte
// ================================================================
function cambiarCanalX(cuentaX, nombreSistema) {
    if (typeof ROL_ACTUAL !== 'undefined' && ROL_ACTUAL === 'publico') return;

    const iframe = document.getElementById('iframe-x-feed');
    const handleText = document.getElementById('current-handle');
    const infoSistema = document.getElementById('info-sistema-activo');

    if (handleText) {
        handleText.innerText = "@" + cuentaX;
    }

    if (infoSistema) {
        infoSistema.innerText = nombreSistema;
    }

    // Volvemos a inyectar la URL limpia con el feed en lista de Nitter
    if (iframe) {
        iframe.src = "https://nitter.net/" + cuentaX + "?theme=dark&hide_replies=1";
    }
}


function limpiarMonitor() {
    cambiarCanalX('MetroCDMX', 'METRO (STC)');
}

// Inicia el monitor con el feed del Metro automáticamente en cuanto cargue el sitio
window.addEventListener('DOMContentLoaded', () => {
    limpiarMonitor();
});


// ================================================================
//  CAMBIAR FUCNION YA NO USO
// ================================================================
function abrirDetalle(nombre, estado, segundos, extra) {
    let modal = document.getElementById('modal-conductor-dinamico');
    if (!modal) {
        modal = document.createElement('div');
        modal.id = 'modal-conductor-dinamico';
        modal.className = 'modal-conductor';
        modal.innerHTML = `
            <div class="modal-content">
                <span class="close-modal" onclick="cerrarModal()">&times;</span>
                <div id="modal-icon" style="font-size:2.5rem;margin-bottom:10px;"></div>
                <h3 id="modal-nombre" style="margin:0 0 5px;color:#1e293b;"></h3>
                <p id="modal-extra" style="font-size:0.8rem;color:#64748b;margin:0 0 10px;"></p>
                <div id="modal-estado-badge" style="display:inline-block;padding:4px 14px;border-radius:20px;font-size:0.8rem;font-weight:bold;margin-bottom:15px;"></div>
                <div id="modal-timer-wrapper" style="display:none;">
                    <p style="font-size:0.75rem;color:#64748b;margin:0 0 5px;">TIEMPO RESTANTE</p>
                    <div class="timer-display" id="modal-timer">00:00:00</div>
                </div>
                <p id="modal-msg" style="font-size:0.82rem;color:#475569;margin-top:10px;"></p>
            </div>`;
        document.body.appendChild(modal);
        modal.addEventListener('click', function(e) { if (e.target === modal) cerrarModal(); });
    }
    const estadoUpper = (estado || '').toUpperCase();
    let icon = '🟢', color = '#22c55e', bg = 'rgba(34,197,94,0.15)', msg = 'Disponible para asignación de viaje.';
    if (estadoUpper === 'OCUPADO' || estadoUpper === 'EN RUTA') { icon = '🔴'; color = '#ef4444'; bg = 'rgba(239,68,68,0.15)'; msg = 'Actualmente en servicio activo.'; }
    else if (estadoUpper === 'DESCANSO') { icon = '🟡'; color = '#f59e0b'; bg = 'rgba(245,158,11,0.15)'; msg = 'En período de descanso obligatorio.'; }
    else if (estadoUpper === 'ESTACIONADO') { icon = '⚪'; color = '#64748b'; bg = 'rgba(100,116,139,0.15)'; msg = 'Unidad en terminal, sin asignación.'; }
    document.getElementById('modal-icon').textContent  = icon;
    document.getElementById('modal-nombre').textContent = nombre;
    document.getElementById('modal-extra').textContent  = extra || '';
    document.getElementById('modal-msg').textContent    = msg;
    const badge = document.getElementById('modal-estado-badge');
    badge.textContent   = estadoUpper;
    badge.style.background = bg;
    badge.style.color      = color;
    badge.style.border     = '1px solid ' + color;
    const timerWrapper = document.getElementById('modal-timer-wrapper');
    if (segundos > 0) {
        timerWrapper.style.display = 'block';
        iniciarCuentaRegresiva('modal-timer', segundos);
    } else {
        timerWrapper.style.display = 'none';
    }
    modal.style.display = 'block';
}

function cerrarModal() {
    const modal = document.getElementById('modal-conductor-dinamico');
    if (modal) modal.style.display = 'none';
}

let _timerInterval = null;
function iniciarCuentaRegresiva(elId, segundosIniciales) {
    if (_timerInterval) clearInterval(_timerInterval);
    let seg = segundosIniciales;
    function render() {
        const h = Math.floor(seg / 3600);
        const m = Math.floor((seg % 3600) / 60);
        const s = seg % 60;
        const el = document.getElementById(elId);
        if (el) el.textContent = String(h).padStart(2,'0') + ':' + String(m).padStart(2,'0') + ':' + String(s).padStart(2,'0');
        if (seg <= 0) clearInterval(_timerInterval);
        seg--;
    }
    render();
    _timerInterval = setInterval(render, 1000);
}

// ================================================================
//  TUTORIAL INTERACTIVO
// ================================================================
const TUTORIAL_STEPS = [
    {
        id: 'tut-brand',
        icon: `<img src="MII.png" alt="Metro Logo" style="height:2.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">`,
        subtitle: 'BIENVENIDO A TU SISTEMA',
        title: 'STC LIVE CDMX',
        body: 'Es un sistema de gestión de transportes del STC.',
        tip: 'Sistema Transporte Colectivo.',
        position: 'bottom'
    },
    {
        id: 'tut-nav',
        icon: '<img src="MII.png" alt="Metro Logo" style="height:2.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">',
        subtitle: 'MENÚ DE NAVEGACIÓN',
        title: 'Accesos Rápidos',
        body: 'Secciones del sistema:\n• Central: Inicio\n• Reportes: Registra incidentes\n• Red: Mapa de Movilidad\n• Sesión: Inicia con usuario',
        tip: 'Puedes navegar entre módulos.',
        position: 'bottom'
    },
    {
        id: 'tut-header-bar',
        icon: '<img src="MX.png" alt="Metro Logo" style="height:3.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">',
        subtitle: 'ESTADO',
        title: 'Horario Tiempo Centro',
        body: '(UTC-06:00) Ciudad de México.',
        tip: 'Si el reloj no coincide, revisa la zona horaria.',
        position: 'bottom'
    },
    {
        id: 'tut-maestro',
        icon: '<img src="M.png" alt="Metro Logo" style="height:1.5rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">',
        subtitle: 'METRO EN CURSO',
        title: 'STC Metro CDMX',
        body: 'Haz clic para ver su recorrido.',
        tip: 'Los puntos brillantes son la posición del tren.',
        position: 'bottom'
    },
    {
        id: 'tut-maestro',
        icon: '<img src="mtbb.png" alt="Metro Logo" style="height:1.5rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">',
        subtitle: 'METROBÚS',
        title: 'Líneas Metrobús CDMX',
        body: 'Haz clic para ver su recorrido.',
        tip: 'Horario de 05:00 a 00:00 hrs.',
        position: 'bottom'
    },
    {
        id: 'tut-maestro',
        icon: '<img src="CBBB.png" alt="Metro Logo" style="height:2.5rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">',
        subtitle: 'CABLEBÚS',
        title: 'Líneas Cablebús CDMX',
        body: 'Haz clic para ver su recorrido.',
        tip: 'Horario de 05:00 a 23:00 hrs.',
        position: 'bottom'
    },
    {
        id: 'tut-maestro',
        icon: '<img src="TTB.png" alt="Metro Logo" style="height:2.5rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">',
        subtitle: 'TROLEBÚS / STE',
        title: 'Líneas Trolebús Eléctrico',
        body: 'Haz clic para ver su recorrido.',
        tip: 'Horario de 05:00 a 00:00 hrs.',
        position: 'bottom'
    },
    {
        id: 'tut-maestro',
        icon: '<img src="RTTP.png" alt="Metro Logo" style="height:2.5rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">',
        subtitle: 'RTP CDMX',
        title: 'Red de Transporte de Pasajeros',
        body: 'RTP cubre algunas zonas.',
        tip: 'Horario de 05:00 a 00:00 hrs.',
        position: 'bottom'
    },
    {
        id: 'tut-telemetria',
        icon: '<img src="MII.png" alt="Metro Logo" style="height:2.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">',
        subtitle: 'PANEL DE ANUNCIOS',
        title: 'Anuncios en Tiempo Real',
        body: 'El panel muestra anuncios oficiales de cada transporte.',
        tip: 'Si hay cierres de estaciones, aparecerán primero aquí.',
        position: 'left'
    },
    {
        id: 'btn-abrir',
        icon: '<img src="Inc.png" alt="Metro Logo" style="height:2.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">',
        subtitle: 'REPORTES',
        title: 'Reporta algún incidente',
        body: 'Realiza aportes en todo el sistema',
        tip: 'Tu contribución puede ayudarnos a todos.',
        position: 'top'
    },
    {
        id: 'btn-abrir-mapa',
        icon: '<img src="MII.png" alt="Metro Logo" style="height:2.1rem;width:auto;vertical-align:middle;margin-right:4px;background:transparent;">',
        subtitle: 'MAPA',
        title: 'MAPAS DE LA RED',
        body: 'MAPAS INDEPENDIENTES.',
        tip: 'MOVILIDAD INTEGRADA.',
        position: 'top'
    }
];

let pasoActual = 0;
let elementoAnterior = null;

function abrirBienvenida() {
    const welcome = document.getElementById('tutorial-welcome');
    welcome.style.display = 'flex';
}
function cerrarBienvenida() {
    document.getElementById('tutorial-welcome').style.display = 'none';
    localStorage.setItem('tms_tutorial_visto', '1');
}
function iniciarTutorial() {
    cerrarBienvenida();
    pasoActual = 0;
    document.getElementById('tutorial-overlay').style.display = 'block';
    document.getElementById('btn-abrir-tutorial').style.display = 'none';
    mostrarPaso(pasoActual);
}

function mostrarPaso(index) {
    const paso = TUTORIAL_STEPS[index];
    if (!paso) { terminarTutorial(); return; }
    if (elementoAnterior) {
        elementoAnterior.classList.remove('tutorial-highlight');
        elementoAnterior.style.removeProperty('z-index');
    }
    const el = document.getElementById(paso.id);
    if (!el) { siguientePaso(); return; }
    el.scrollIntoView({ behavior: 'smooth', block: 'center' });
    setTimeout(function() {
        el.classList.add('tutorial-highlight');
        elementoAnterior = el;
        const rect = el.getBoundingClientRect();
        const spotlight = document.getElementById('tutorial-spotlight');
        const pad = 8;
        spotlight.style.top    = (rect.top    - pad + window.scrollY) + 'px';
        spotlight.style.left   = (rect.left   - pad + window.scrollX) + 'px';
        spotlight.style.width  = (rect.width  + pad * 2) + 'px';
        spotlight.style.height = (rect.height + pad * 2) + 'px';

        // ==========================================
        // NUEVA LÓGICA INTELIGENTE PARA EL ICONO
        // ==========================================
        const elIcono = document.getElementById('tut-icon');
        if (paso.icon.trim().startsWith('<img')) {
            // Si el texto empieza con "<img", lo inyectamos directamente
            elIcono.innerHTML = paso.icon;
        } else {
            // Si es un icono normal (ej: "fas fa-bus"), usamos la etiqueta <i> como antes
            elIcono.innerHTML = '<i class="' + paso.icon + '"></i>';
        }
        // ==========================================

        document.getElementById('tut-subtitle').textContent = paso.subtitle;
        document.getElementById('tut-title').textContent    = paso.title;
        document.getElementById('tut-body').innerHTML = paso.body.replace(/\n/g, '<br>');
        document.getElementById('tut-tip').innerHTML = '<i class="fas fa-lightbulb" style="margin-right:6px;color:#fbbf24"></i>' + paso.tip;
        const btnNext = document.getElementById('tut-btn-next');
        if (index === TUTORIAL_STEPS.length - 1) {
            btnNext.innerHTML = '<i class="fas fa-check"></i> ¡Listo!';
        } else {
            btnNext.innerHTML = 'Siguiente <i class="fas fa-arrow-right"></i>';
        }
        let dotsHTML = '';
        for (let i = 0; i < TUTORIAL_STEPS.length; i++) {
            let cls = 'tut-dot';
            if (i < index)      cls += ' done';
            else if (i === index) cls += ' active';
            dotsHTML += '<div class="' + cls + '"></div>';
        }
        dotsHTML += '<span class="tut-step-label">' + (index + 1) + ' / ' + TUTORIAL_STEPS.length + '</span>';
        document.getElementById('tut-progress').innerHTML = dotsHTML;
        const card = document.getElementById('tutorial-card');
        card.style.display = 'block';
        posicionarTarjeta(rect, paso.position || 'bottom');
    }, 400);
}

function posicionarTarjeta(rect, pos) {
    const card  = document.getElementById('tutorial-card');
    const cardW = 320;
    const cardH = 300;
    const margin = 18;
    const vw = window.innerWidth;
    const vh = window.innerHeight;
    let top, left;
    if (pos === 'bottom')      { top = rect.bottom + margin; left = rect.left + rect.width / 2 - cardW / 2; }
    else if (pos === 'top')    { top = rect.top - cardH - margin; left = rect.left + rect.width / 2 - cardW / 2; }
    else if (pos === 'left')   { top = rect.top + rect.height / 2 - cardH / 2; left = rect.left - cardW - margin; }
    else                       { top = rect.top + rect.height / 2 - cardH / 2; left = rect.right + margin; }
    left = Math.max(10, Math.min(left, vw - cardW - 10));
    top  = Math.max(10, Math.min(top,  vh - cardH - 10));
    card.style.top  = top  + 'px';
    card.style.left = left + 'px';
}

function siguientePaso() {
    pasoActual++;
    if (pasoActual >= TUTORIAL_STEPS.length) terminarTutorial();
    else mostrarPaso(pasoActual);
}

function terminarTutorial() {
    document.getElementById('tutorial-overlay').style.display = 'none';
    document.getElementById('tutorial-card').style.display = 'none';
    document.getElementById('btn-abrir-tutorial').style.display = 'flex';
    if (elementoAnterior) { elementoAnterior.classList.remove('tutorial-highlight'); elementoAnterior = null; }
    localStorage.setItem('tms_tutorial_visto', '1');
    window.scrollTo({ top: 0, behavior: 'smooth' });
}

window.addEventListener('DOMContentLoaded', function() {
    if (!localStorage.getItem('tms_tutorial_visto')) {
        setTimeout(function() { abrirBienvenida(); }, 800);
    }
});

document.addEventListener('keydown', function(e) {
    if (e.key === 'Escape') {
        const overlay = document.getElementById('tutorial-overlay');
        if (overlay && overlay.style.display === 'block') terminarTutorial();
        cerrarModalEstacion();
        cerrarModalLinea();
    }
});

// ================================================================
//  AVISO DE PRIVACIDAD
// ================================================================
const privModal = document.getElementById("privacyModal");
const btnPriv   = document.getElementById("openPrivacy");
const spanClose = document.getElementsByClassName("close-btn")[0];

btnPriv.onclick = function(e) {
    e.preventDefault();
    privModal.style.display = "block";
};
spanClose.onclick = function() {
    privModal.style.display = "none";
};
window.onclick = function(event) {
    if (event.target === privModal) privModal.style.display = "none";
};
</script>
</body>
</html>
