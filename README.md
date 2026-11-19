<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>LOGIARCO v2.0 — Sistema Logístico</title>
  
  <!-- Configuración PWA para instalar como App en Celular -->
  <meta name="theme-color" content="#0066ff">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="LOGIARCO">
  
  <!-- Manifiesto PWA embebido -->
  <link rel="manifest" href='data:application/manifest+json,{
    "name": "LOGIARCO v2.0",
    "short_name": "LOGIARCO",
    "start_url": ".",
    "display": "standalone",
    "background_color": "#f8fbff",
    "theme_color": "#0066ff",
    "icons": [{
      "src": "https://cdn-icons-png.flaticon.com/512/854/854878.png",
      "sizes": "512x512",
      "type": "image/png"
    }]
  }'>

  <!-- Fuentes Futuristas Limpias -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@600;800&family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
  
  <!-- Mapa Interactivo Leaflet -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <style>
    :root {
      --bg-main: #f8fbff;
      --bg-card: #ffffff;
      --primary: #0066ff;
      --primary-dark: #0047b3;
      --primary-glow: rgba(0, 102, 255, 0.16);
      --accent-cyan: #00b4d8;
      --text-main: #0f172a;
      --text-muted: #64748b;
      --border-tech: #dbeafe;
      
      /* Estados de Paradas */
      --st-azul: #0066ff;
      --st-verde: #10b981;
      --st-amarillo: #f59e0b;
      --danger: #ef4444;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Plus Jakarta Sans', sans-serif;
      -webkit-tap-highlight-color: transparent;
    }

    body {
      background-color: var(--bg-main);
      color: var(--text-main);
      height: 100vh;
      overflow: hidden;
    }

    .brand-font {
      font-family: 'Orbitron', sans-serif;
      letter-spacing: 1.5px;
    }

    /* PANTALLA DE LOGIN */
    #auth-screen {
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      padding: 20px;
    }

    .auth-card {
      background: var(--bg-card);
      border: 1px solid var(--border-tech);
      border-radius: 20px;
      padding: 36px 26px;
      width: 100%;
      max-width: 420px;
      box-shadow: 0 20px 50px rgba(0, 102, 255, 0.08);
      position: relative;
      overflow: hidden;
    }

    .auth-card::before {
      content: "";
      position: absolute;
      top: 0; left: 0; right: 0;
      height: 4px;
      background: linear-gradient(90deg, var(--primary), var(--accent-cyan));
    }

    .logo-header {
      text-align: center;
      margin-bottom: 24px;
    }

    .logo-badge {
      display: inline-block;
      background: #eff6ff;
      color: var(--primary);
      font-size: 11px;
      font-weight: 700;
      padding: 4px 12px;
      border-radius: 99px;
      border: 1px solid var(--border-tech);
      margin-bottom: 8px;
    }

    .logo-header h1 {
      font-size: 28px;
      color: var(--primary);
    }

    .form-group {
      margin-bottom: 14px;
    }

    .form-group label {
      display: block;
      font-size: 12px;
      font-weight: 700;
      text-transform: uppercase;
      color: var(--text-muted);
      margin-bottom: 6px;
      letter-spacing: 0.5px;
    }

    input, select {
      width: 100%;
      padding: 13px 14px;
      border: 1.5px solid var(--border-tech);
      border-radius: 12px;
      font-size: 16px;
      background: #ffffff;
      color: var(--text-main);
      transition: all 0.2s;
    }

    input:focus {
      outline: none;
      border-color: var(--primary);
      box-shadow: 0 0 0 4px var(--primary-glow);
    }

    .btn {
      width: 100%;
      padding: 13px 16px;
      border: none;
      border-radius: 12px;
      font-weight: 700;
      font-size: 14px;
      cursor: pointer;
      transition: all 0.2s;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
    }

    .btn-primary {
      background: linear-gradient(135deg, var(--primary), #0052cc);
      color: #ffffff;
      box-shadow: 0 6px 16px rgba(0, 102, 255, 0.25);
    }

    .btn-outline {
      background: #ffffff;
      color: var(--primary);
      border: 1.5px solid var(--border-tech);
    }

    .btn-sm {
      padding: 8px 11px;
      font-size: 12px;
      border-radius: 10px;
      width: auto;
    }

    /* INTERFAZ PRINCIPAL */
    #app-screen {
      display: none;
      flex-direction: column;
      height: 100vh;
      overflow: hidden;
    }

    header.topbar {
      background: rgba(255, 255, 255, 0.95);
      backdrop-filter: blur(12px);
      border-bottom: 1px solid var(--border-tech);
      padding: 10px 14px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 8px;
      z-index: 1000;
      flex-shrink: 0;
    }

    .topbar-left {
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .topbar-left h2 {
      font-size: 17px;
      color: var(--primary);
    }

    .version-pill {
      background: #eff6ff;
      color: var(--primary);
      border: 1px solid var(--border-tech);
      padding: 2px 8px;
      border-radius: 99px;
      font-size: 10px;
      font-weight: 700;
    }

    .topbar-actions {
      display: flex;
      align-items: center;
      gap: 6px;
    }

    .user-chip {
      background: #f1f5f9;
      padding: 5px 10px;
      border-radius: 99px;
      font-size: 11px;
      font-weight: 700;
      color: var(--text-main);
    }

    /* WORKSPACE RESPONSIVO */
    .workspace {
      display: grid;
      grid-template-columns: 380px 1fr;
      flex: 1;
      overflow: hidden;
      position: relative;
    }

    .sidebar {
      background: #ffffff;
      border-right: 1px solid var(--border-tech);
      display: flex;
      flex-direction: column;
      overflow: hidden;
      z-index: 10;
    }

    .sidebar-controls {
      padding: 12px 14px;
      border-bottom: 1px solid var(--border-tech);
      background: #fafcff;
      flex-shrink: 0;
    }

    .view-tabs {
      display: grid;
      grid-template-columns: 1fr 1fr;
      background: #eff6ff;
      padding: 3px;
      border-radius: 10px;
      margin-bottom: 10px;
      border: 1px solid var(--border-tech);
    }

    .view-tab {
      padding: 7px;
      border: none;
      background: transparent;
      border-radius: 8px;
      font-size: 12px;
      font-weight: 700;
      color: var(--text-muted);
      cursor: pointer;
    }

    .view-tab.active {
      background: #ffffff;
      color: var(--primary);
      box-shadow: 0 2px 6px rgba(0, 102, 255, 0.12);
    }

    .legend-bar {
      display: flex;
      justify-content: space-between;
      font-size: 11px;
      color: var(--text-muted);
      margin-bottom: 10px;
      padding: 5px 10px;
      background: #ffffff;
      border-radius: 8px;
      border: 1px solid var(--border-tech);
    }

    .legend-item {
      display: flex;
      align-items: center;
      gap: 5px;
      font-weight: 600;
    }

    .dot {
      width: 9px;
      height: 9px;
      border-radius: 50%;
      display: inline-block;
    }

    .dot-azul { background: var(--st-azul); }
    .dot-verde { background: var(--st-verde); }
    .dot-amarillo { background: var(--st-amarillo); }

    .shift-banner {
      background: linear-gradient(135deg, #eff6ff, #ffffff);
      border: 1px solid var(--border-tech);
      border-radius: 10px;
      padding: 8px 12px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 12px;
    }

    .shift-stats strong {
      color: var(--primary);
      font-size: 13px;
    }

    .stops-list {
      flex: 1;
      overflow-y: auto;
      padding: 12px 14px;
      display: flex;
      flex-direction: column;
      gap: 10px;
    }

    .stop-card {
      background: #ffffff;
      border: 1.5px solid var(--border-tech);
      border-radius: 14px;
      padding: 12px 14px;
      position: relative;
      border-left-width: 6px;
    }

    .stop-card.estado-azul { border-left-color: var(--st-azul); }
    .stop-card.estado-verde { border-left-color: var(--st-verde); }
    .stop-card.estado-amarillo { border-left-color: var(--st-amarillo); background: #fffdf7; }

    .stop-header {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      margin-bottom: 4px;
    }

    .stop-title {
      font-weight: 700;
      font-size: 15px;
      color: var(--text-main);
    }

    .status-badge {
      font-size: 10px;
      font-weight: 800;
      text-transform: uppercase;
      padding: 3px 8px;
      border-radius: 99px;
    }

    .badge-azul { background: #dbeafe; color: #1d4ed8; }
    .badge-verde { background: #d1fae5; color: #047857; }
    .badge-amarillo { background: #fef3c7; color: #b45309; }

    .stop-meta {
      font-size: 12px;
      color: var(--text-muted);
      margin-bottom: 10px;
    }

    .stop-actions {
      display: grid;
      grid-template-columns: 1fr 1.3fr 1.2fr;
      gap: 6px;
    }

    .btn-visit {
      background: #fef3c7;
      color: #92400e;
      border: 1px solid #fcd34d;
    }

    .btn-remove {
      background: #fee2e2;
      color: #b91c1c;
      border: 1px solid #fca5a5;
    }

    /* MAPA */
    .map-wrapper {
      position: relative;
      height: 100%;
      width: 100%;
    }

    #map {
      height: 100%;
      width: 100%;
      z-index: 1;
      background: #eef4fb;
    }

    .map-controls-floating {
      position: absolute;
      top: 12px;
      right: 12px;
      z-index: 1000;
      display: flex;
      flex-direction: column;
      gap: 8px;
      pointer-events: auto;
    }

    .btn-gps {
      background: #ffffff;
      color: var(--primary);
      border: 1.5px solid var(--primary);
      padding: 10px 16px;
      border-radius: 99px;
      font-size: 13px;
      font-weight: 700;
      box-shadow: 0 4px 15px rgba(0, 102, 255, 0.25);
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
      touch-action: manipulation;
    }

    .adjust-bar {
      display: none;
      position: absolute;
      bottom: 14px;
      left: 12px;
      right: 12px;
      z-index: 1000;
      background: #ffffff;
      border: 2px solid var(--st-verde);
      border-radius: 16px;
      padding: 12px 14px;
      box-shadow: 0 10px 30px rgba(16, 185, 129, 0.25);
      flex-direction: column;
      gap: 8px;
    }

    .adjust-bar-title {
      font-size: 13px;
      font-weight: 700;
      color: #065f46;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .adjust-bar-actions {
      display: flex;
      gap: 8px;
    }

    /* MODALES */
    .modal-backdrop {
      display: none;
      position: fixed;
      inset: 0;
      background: rgba(15, 23, 42, 0.5);
      backdrop-filter: blur(4px);
      z-index: 3000;
      align-items: flex-end;
      justify-content: center;
      padding: 0;
    }

    .modal-box {
      background: #ffffff;
      border-top: 3px solid var(--primary);
      border-radius: 22px 22px 0 0;
      padding: 24px 20px 30px 20px;
      width: 100%;
      max-width: 480px;
      box-shadow: 0 -10px 40px rgba(0, 0, 0, 0.18);
    }

    @media (min-width: 768px) {
      .modal-backdrop {
        align-items: center;
        padding: 16px;
      }
      .modal-box {
        border-radius: 18px;
      }
    }

    .modal-box h3 {
      color: var(--primary);
      margin-bottom: 6px;
      font-size: 18px;
    }

    .modal-box p {
      font-size: 13px;
      color: var(--text-muted);
      margin-bottom: 14px;
    }

    .modal-footer {
      display: flex;
      gap: 10px;
      margin-top: 16px;
    }

    .gps-pulse {
      width: 16px;
      height: 16px;
      background: #0066ff;
      border: 3px solid #ffffff;
      border-radius: 50%;
      box-shadow: 0 0 0 6px rgba(0, 102, 255, 0.28);
    }

    #loader {
      display: none;
      position: fixed;
      top: 12px;
      left: 50%;
      transform: translateX(-50%);
      background: var(--primary);
      color: #fff;
      padding: 7px 16px;
      border-radius: 99px;
      font-size: 12px;
      font-weight: 700;
      z-index: 4000;
      box-shadow: 0 4px 15px rgba(0, 102, 255, 0.35);
    }

    @media (max-width: 850px) {
      .user-chip { display: none; }
      .workspace {
        grid-template-columns: 1fr;
        grid-template-rows: 45vh 1fr;
      }
      .map-wrapper { order: 1; }
      .sidebar {
        order: 2;
        border-right: none;
        border-top: 2px solid var(--border-tech);
        border-radius: 18px 18px 0 0;
        margin-top: -10px;
      }
    }
  </style>
</head>
<body>

  <div id="loader">Sincronizando...</div>

  <!-- PANTALLA DE LOGIN -->
  <div id="auth-screen">
    <div class="auth-card">
      <div class="logo-header">
        <span class="logo-badge">APP LOGÍSTICA PWA</span>
        <h1 class="brand-font">LOGIARCO</h1>
        <p style="font-size: 13px; color: var(--text-muted); margin-top: 4px;">Control inteligente de rutas y entregas</p>
      </div>

      <div id="login-form">
        <div class="form-group">
          <label>Usuario</label>
          <input type="text" id="login-user" placeholder="Ej. admin" />
        </div>
        <div class="form-group">
          <label>Contraseña</label>
          <input type="password" id="login-pass" placeholder="••••••" />
        </div>
        <button class="btn btn-primary" onclick="handleLogin()">Iniciar Sesión</button>
      </div>
    </div>
  </div>

  <!-- APLICACIÓN PRINCIPAL -->
  <div id="app-screen">
    <header class="topbar">
      <div class="topbar-left">
        <h2 class="brand-font">LOGIARCO</h2>
        <span class="version-pill">App</span>
        <span class="user-chip" id="current-user-label">Operador</span>
      </div>

      <div class="topbar-actions">
        <button class="btn btn-outline btn-sm" onclick="loadStops()">↻</button>
        <button class="btn btn-primary btn-sm" onclick="startPinPlacement()">+ Parada</button>
        <button class="btn btn-outline btn-sm" style="border-color:#f59e0b; color:#b45309;" onclick="finishShift()">Turno</button>
        <button class="btn btn-outline btn-sm" onclick="logout()">Salir</button>
      </div>
    </header>

    <div class="workspace">
      <aside class="sidebar">
        <div class="sidebar-controls">
          <div class="view-tabs">
            <button class="view-tab active" id="tab-all" onclick="setViewMode('all')">Todos</button>
            <button class="view-tab" id="tab-mine" onclick="setViewMode('mine')">Mis Paradas</button>
          </div>

          <div class="legend-bar">
            <span class="legend-item"><i class="dot dot-azul"></i> Reg.</span>
            <span class="legend-item"><i class="dot dot-verde"></i> Nueva</span>
            <span class="legend-item"><i class="dot dot-amarillo"></i> Entregada</span>
          </div>

          <div class="shift-banner">
            <div class="shift-stats">
              Turno: <strong id="shift-count">0</strong> vis. | 
              <strong id="shift-pieces">0</strong> pz | 
              <strong id="shift-money">$0.00</strong>
            </div>
          </div>
        </div>

        <div class="stops-list" id="stops-list"></div>
      </aside>

      <main class="map-wrapper">
        <div id="map"></div>

        <div class="map-controls-floating">
          <button class="btn-gps" id="btn-gps-trigger">📍 Mi Ubicación</button>
          <button class="btn-gps" style="border-color:#cbd5e1; color:#334155; background:#ffffff;" id="btn-map-style">🛰️ Satélite</button>
        </div>

        <div class="adjust-bar" id="adjust-bar">
          <div class="adjust-bar-title">
            <span>Arrastra el pin verde</span>
            <span id="draft-coords-text" style="font-size:11px; color:#64748b;"></span>
          </div>
          <div class="adjust-bar-actions">
            <button class="btn btn-outline btn-sm" style="flex:1;" onclick="cancelPinPlacement()">Cancelar</button>
            <button class="btn btn-primary btn-sm" style="flex:2; background:#10b981;" onclick="openAddModalFromDraft()">Confirmar</button>
          </div>
        </div>
      </main>
    </div>
  </div>

  <!-- MODAL: AGREGAR PARADA -->
  <div class="modal-backdrop" id="modal-add">
    <div class="modal-box">
      <h3>Registrar Nueva Parada</h3>
      <p>Se guardará en las coordenadas seleccionadas en color <strong>verde</strong>.</p>
      <div class="form-group">
        <label>Nombre de la Tienda / Punto *</label>
        <input type="text" id="new-stop-name" placeholder="Ej. Abarrotes El Sol" />
      </div>
      <div style="display:grid; grid-template-columns:1fr 1fr; gap:10px;">
        <div class="form-group">
          <label>Latitud</label>
          <input type="number" step="any" id="new-stop-lat" readonly style="background:#f8fafc; font-size:13px;" />
        </div>
        <div class="form-group">
          <label>Longitud</label>
          <input type="number" step="any" id="new-stop-lng" readonly style="background:#f8fafc; font-size:13px;" />
        </div>
      </div>
      <div class="modal-footer">
        <button class="btn btn-outline" onclick="closeModals()">Cancelar</button>
        <button class="btn btn-primary" onclick="confirmAddStop()">Guardar</button>
      </div>
    </div>
  </div>

  <!-- MODAL: OPERACIÓN -->
  <div class="modal-backdrop" id="modal-operation">
    <div class="modal-box">
      <h3 id="op-title">Registrar Entrega</h3>
      <p id="op-desc">Ingresa las piezas recogidas y el monto recibido.</p>
      <input type="hidden" id="op-stop-id" />
      <input type="hidden" id="op-type" />

      <div class="form-group">
        <label>Piezas recogidas *</label>
        <input type="number" inputmode="numeric" min="0" id="op-pieces" placeholder="Ej. 24" required />
      </div>
      <div class="form-group">
        <label>Monto recibido ($) *</label>
        <input type="number" inputmode="decimal" min="0" step="0.01" id="op-amount" placeholder="Ej. 1450.00" required />
      </div>

      <div class="modal-footer">
        <button class="btn btn-outline" onclick="closeModals()">Cancelar</button>
        <button class="btn btn-primary" id="op-confirm-btn" onclick="confirmStopOperation()">Confirmar</button>
      </div>
    </div>
  </div>

  <script>
    const SCRIPT_URL = "https://script.google.com/macros/s/AKfycby1aFad0qRI8z3sLSUFlvMFUAgoXEx1cKDSoegnxAnvmQb9BxLY3_g0nAoJmHWSAMPD1w/exec";

    let currentUser = null;
    let allStops = [];
    let viewMode = "all";
    let map = null;
    let streetLayer = null;
    let satelliteLayer = null;
    let isSatellite = false;
    let markersLayer = null;
    let myLocationMarker = null;
    let draftMarker = null;

    let userCoords = { lat: 21.1221, lng: -101.6826 };
    let draftCoords = { lat: 21.1221, lng: -101.6826 };

    let shiftSummary = { entregas: 0, piezas: 0, monto: 0 };

    function showLoader(show) {
      document.getElementById("loader").style.display = show ? "block" : "none";
    }

    async function apiCall(params) {
      showLoader(true);
      try {
        const query = new URLSearchParams(params).toString();
        const res = await fetch(`${SCRIPT_URL}?${query}`);
        const data = await res.json();
        showLoader(false);
        return data;
      } catch (err) {
        showLoader(false);
        alert("Error de conexión: " + err.message);
        return { status: "error" };
      }
    }

    async function handleLogin() {
      const user = document.getElementById("login-user").value.trim();
      const pass = document.getElementById("login-pass").value.trim();
      if (!user || !pass) return alert("Completa usuario y contraseña.");

      const res = await apiCall({ action: "login", user, pass });
      if (res.status === "ok") {
        currentUser = { user: res.user, name: res.name };
        startApp();
      } else {
        alert(res.message || "Credenciales incorrectas");
      }
    }

    function logout() {
      currentUser = null;
      document.getElementById("app-screen").style.display = "none";
      document.getElementById("auth-screen").style.display = "flex";
    }

    function startApp() {
      document.getElementById("auth-screen").style.display = "none";
      document.getElementById("app-screen").style.display = "flex";
      document.getElementById("current-user-label").textContent = currentUser.name;

      if (!map) {
        map = L.map("map", { zoomControl: true }).setView([userCoords.lat, userCoords.lng], 15);

        streetLayer = L.tileLayer("https://server.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer/tile/{z}/{y}/{x}", {
          maxZoom: 19, attribution: "Esri"
        });

        satelliteLayer = L.tileLayer("https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}", {
          maxZoom: 19, attribution: "Esri"
        });

        streetLayer.addTo(map);
        markersLayer = L.layerGroup().addTo(map);

        map.on("click", function(e) {
          if (draftMarker) {
            updateDraftPosition(e.latlng.lat, e.latlng.lng);
          }
        });

        document.getElementById("btn-gps-trigger").addEventListener("click", () => centerOnMyLocation(true));
        document.getElementById("btn-gps-trigger").addEventListener("touchend", (e) => { e.preventDefault(); centerOnMyLocation(true); });

        document.getElementById("btn-map-style").addEventListener("click", toggleMapLayer);
        document.getElementById("btn-map-style").addEventListener("touchend", (e) => { e.preventDefault(); toggleMapLayer(); });
      }

      setTimeout(() => {
        map.invalidateSize();
        centerOnMyLocation(true);
      }, 300);

      loadStops();
    }

    function toggleMapLayer() {
      const btn = document.getElementById("btn-map-style");
      if (isSatellite) {
        map.removeLayer(satelliteLayer);
        streetLayer.addTo(map);
        btn.textContent = "🛰️ Satélite";
        isSatellite = false;
      } else {
        map.removeLayer(streetLayer);
        satelliteLayer.addTo(map);
        btn.textContent = "🗺️ Calles";
        isSatellite = true;
      }
    }

    function centerOnMyLocation(fly = true) {
      if (!navigator.geolocation) {
        alert("Geolocalización no soportada en este navegador.");
        return;
      }

      navigator.geolocation.getCurrentPosition(
        (pos) => {
          userCoords = { lat: pos.coords.latitude, lng: pos.coords.longitude };

          if (myLocationMarker) {
            myLocationMarker.setLatLng([userCoords.lat, userCoords.lng]);
          } else {
            const gpsIcon = L.divIcon({
              className: "gps-marker",
              html: `<div class="gps-pulse"></div>`,
              iconSize: [16, 16],
              iconAnchor: [8, 8]
            });
            myLocationMarker = L.marker([userCoords.lat, userCoords.lng], { icon: gpsIcon })
              .bindPopup("Estás aquí")
              .addTo(map);
          }

          if (fly) {
            map.flyTo([userCoords.lat, userCoords.lng], 17);
          }
        },
        (err) => {
          alert("Permiso de ubicación denegado. Asegúrate de permitir el acceso al GPS en la configuración de tu navegador.");
        },
        { enableHighAccuracy: true, timeout: 10000, maximumAge: 0 }
      );
    }

    function startPinPlacement() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (pos) => {
            userCoords = { lat: pos.coords.latitude, lng: pos.coords.longitude };
            placeDraggableDraftPin(userCoords.lat, userCoords.lng);
          },
          () => {
            const center = map.getCenter();
            placeDraggableDraftPin(center.lat, center.lng);
          },
          { enableHighAccuracy: true, timeout: 5000 }
        );
      } else {
        const center = map.getCenter();
        placeDraggableDraftPin(center.lat, center.lng);
      }
    }

    function placeDraggableDraftPin(lat, lng) {
      if (draftMarker) map.removeLayer(draftMarker);

      const draggableIcon = L.divIcon({
        className: "draft-pin",
        html: `<div style="width: 28px; height: 28px; border-radius: 50%; background: #10b981; border: 4px solid #ffffff; box-shadow: 0 0 18px rgba(16, 185, 129, 0.9);"></div>`,
        iconSize: [28, 28],
        iconAnchor: [14, 14]
      });

      draftMarker = L.marker([lat, lng], { icon: draggableIcon, draggable: true }).addTo(map);
      updateDraftPosition(lat, lng);
      map.flyTo([lat, lng], 18);

      draftMarker.on("drag", function(e) {
        const pos = e.target.getLatLng();
        updateDraftPosition(pos.lat, pos.lng);
      });

      document.getElementById("adjust-bar").style.display = "flex";
    }

    function updateDraftPosition(lat, lng) {
      draftCoords = { lat: Number(lat).toFixed(6), lng: Number(lng).toFixed(6) };
      if (draftMarker) draftMarker.setLatLng([draftCoords.lat, draftCoords.lng]);
      document.getElementById("draft-coords-text").textContent = `${draftCoords.lat}, ${draftCoords.lng}`;
    }

    function cancelPinPlacement() {
      if (draftMarker) {
        map.removeLayer(draftMarker);
        draftMarker = null;
      }
      document.getElementById("adjust-bar").style.display = "none";
    }

    function openAddModalFromDraft() {
      document.getElementById("new-stop-name").value = "";
      document.getElementById("new-stop-lat").value = draftCoords.lat;
      document.getElementById("new-stop-lng").value = draftCoords.lng;
      document.getElementById("modal-add").style.display = "flex";
      setTimeout(() => document.getElementById("new-stop-name").focus(), 150);
    }

    async function confirmAddStop() {
      const nombre = document.getElementById("new-stop-name").value.trim();
      const lat = document.getElementById("new-stop-lat").value;
      const lng = document.getElementById("new-stop-lng").value;
      if (!nombre) return alert("Escribe el nombre de la tienda.");

      closeModals();
      cancelPinPlacement();

      const res = await apiCall({ action: "addStop", nombre, lat, lng, creador: currentUser.user });
      if (res.status === "ok") await loadStops();
    }

    async function loadStops() {
      const res = await apiCall({ action: "getStops" });
      if (res.status === "ok") {
        allStops = res.stops;
        renderStops();
      }
    }

    function setViewMode(mode) {
      viewMode = mode;
      document.getElementById("tab-all").classList.toggle("active", mode === "all");
      document.getElementById("tab-mine").classList.toggle("active", mode === "mine");
      renderStops();
    }

    function getMarkerIcon(estado) {
      let color = "#0066ff";
      if (estado === "verde") color = "#10b981";
      if (estado === "amarillo") color = "#f59e0b";

      return L.divIcon({
        className: "custom-pin",
        html: `<div style="width: 22px; height: 22px; border-radius: 50%; background: ${color}; border: 3px solid #ffffff; box-shadow: 0 2px 8px rgba(0,0,0,0.35);"></div>`,
        iconSize: [22, 22],
        iconAnchor: [11, 11]
      });
    }

    function renderStops() {
      const listEl = document.getElementById("stops-list");
      listEl.innerHTML = "";
      markersLayer.clearLayers();

      const filtered = viewMode === "mine"
        ? allStops.filter(s => String(s.creador).toLowerCase() === String(currentUser.user).toLowerCase())
        : allStops;

      if (filtered.length === 0) {
        listEl.innerHTML = `<p style="text-align:center; color:var(--text-muted); font-size:13px; margin-top:24px;">No hay paradas.</p>`;
        return;
      }

      filtered.forEach(stop => {
        const estado = stop.estado || "azul";
        const estadoTexto = estado === "verde" ? "Nueva" : (estado === "amarillo" ? "Entregada" : "Registrada");

        const marker = L.marker([stop.lat, stop.lng], { icon: getMarkerIcon(estado) })
          .bindPopup(`<strong>${stop.nombre}</strong><br/>Estado: <b>${estadoTexto}</b><br/>Registró: ${stop.creador}`);
        markersLayer.addLayer(marker);

        const card = document.createElement("div");
        card.className = `stop-card estado-${estado}`;
        card.innerHTML = `
          <div class="stop-header">
            <span class="stop-title">${stop.nombre}</span>
            <span class="status-badge badge-${estado}">${estadoTexto}</span>
          </div>
          <div class="stop-meta">Reg: <strong>${stop.creador}</strong> • ${stop.piezas || 0} pz / $${Number(stop.monto || 0).toFixed(2)}</div>
          <div class="stop-actions">
            <button class="btn btn-outline btn-sm" onclick="focusStop(${stop.lat}, ${stop.lng})">Ver</button>
            <button class="btn btn-visit btn-sm" onclick="openOperationModal('${stop.id}', 'deliver', '${stop.nombre.replace(/'/g, "")}')">Entregar</button>
            <button class="btn btn-remove btn-sm" onclick="openOperationModal('${stop.id}', 'remove', '${stop.nombre.replace(/'/g, "")}')">Quitar</button>
          </div>
        `;
        listEl.appendChild(card);
      });
    }

    function focusStop(lat, lng) {
      map.flyTo([lat, lng], 18);
    }

    function openOperationModal(id, type, nombre) {
      document.getElementById("op-stop-id").value = id;
      document.getElementById("op-type").value = type;
      document.getElementById("op-pieces").value = "";
      document.getElementById("op-amount").value = "";

      if (type === "deliver") {
        document.getElementById("op-title").textContent = `Entrega: ${nombre}`;
        document.getElementById("op-desc").textContent = "La parada pasará a color amarillo esta semana.";
        document.getElementById("op-confirm-btn").textContent = "Marcar Entregada";
      } else {
        document.getElementById("op-title").textContent = `Quitar: ${nombre}`;
        document.getElementById("op-desc").textContent = "Obligatorio: ingresa piezas y monto para cerrar.";
        document.getElementById("op-confirm-btn").textContent = "Cerrar y Quitar";
      }
      document.getElementById("modal-operation").style.display = "flex";
    }

    async function confirmStopOperation() {
      const id = document.getElementById("op-stop-id.value") || document.getElementById("op-stop-id").value;
      const type = document.getElementById("op-type").value;
      const piecesRaw = document.getElementById("op-pieces").value.trim();
      const amountRaw = document.getElementById("op-amount").value.trim();

      if (piecesRaw === "" || amountRaw === "" || isNaN(piecesRaw) || isNaN(amountRaw)) {
        alert("Indica obligatoriamente las piezas y el monto.");
        return;
      }

      closeModals();
      const action = type === "deliver" ? "deliverStop" : "removeStop";
      const res = await apiCall({ action, id, usuario: currentUser.user, piezas: Number(piecesRaw), monto: Number(amountRaw) });

      if (res.status === "ok") {
        shiftSummary.entregas += 1;
        shiftSummary.piezas += Number(piecesRaw);
        shiftSummary.monto += Number(amountRaw);
        updateShiftBanner();
        await loadStops();
      }
    }

    function updateShiftBanner() {
      document.getElementById("shift-count").textContent = shiftSummary.entregas;
      document.getElementById("shift-pieces").textContent = shiftSummary.piezas;
      document.getElementById("shift-money").textContent = `$${shiftSummary.monto.toFixed(2)}`;
    }

    async function finishShift() {
      if (!confirm(`¿Finalizar turno y enviar corte a Sheets?`)) return;
      const res = await apiCard({ action: "closeShift", usuario: currentUser.user, totalEntregas: shiftSummary.entregas, totalPiezas: shiftSummary.piezas, totalMonto: shiftSummary.monto });
      if (res.status === "ok") {
        alert("Corte enviado correctamente.");
        shiftSummary = { entregas: 0, piezas: 0, monto: 0 };
        updateShiftBanner();
      }
    }

    function closeModals() {
      document.getElementById("modal-add").style.display = "none";
      document.getElementById("modal-operation").style.display = "none";
    }
  </script>
</body>
</html>
