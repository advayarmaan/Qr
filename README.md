<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <title>SUPRAYAS NGO - QR Scanner</title>
  
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>

  <style>
    @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800;900&display=swap');

    * {
      font-family: 'Plus Jakarta Sans', -apple-system, BlinkMacSystemFont, sans-serif;
      -webkit-tap-highlight-color: transparent;
      user-select: none;
      box-sizing: border-box;
    }

    html, body {
      height: 100%;
      height: 100dvh;
      overflow: hidden;
      margin: 0;
      padding: 0;
      touch-action: manipulation;
    }

    /* Suppress html5-qrcode default borders and shading to leave only 1 clean set of corners */
    #reader {
      border: none !important;
      position: relative !important;
      width: 100% !important;
      height: 100% !important;
    }

    #reader__scan_region {
      border: none !important;
      outline: none !important;
      width: 100% !important;
      height: 100% !important;
      display: flex !important;
      align-items: center !important;
      justify-content: center !important;
    }

    #reader__scan_region img,
    #reader__scan_region svg,
    #reader div[style*="border"],
    #reader__dashboard,
    #reader__header_message {
      display: none !important;
      border: none !important;
    }

    #reader video {
      object-fit: cover !important;
      width: 100% !important;
      height: 100% !important;
      border-radius: 1.25rem;
    }

    /* Single clean corner reticle setup */
    .reticle-corner {
      transition: border-color 0.15s ease-out, box-shadow 0.15s ease-out, transform 0.15s ease-out;
    }

    .reticle-flash {
      border-color: #22c55e !important;
      box-shadow: 0 0 24px rgba(34, 197, 94, 0.9), inset 0 0 12px rgba(34, 197, 94, 0.5);
      transform: scale(1.05);
    }
  </style>
</head>
<body class="bg-[#F8FAFC] text-slate-800 h-[100dvh] flex flex-col antialiased overflow-hidden selection:bg-teal-100">

  <!-- Main Container: Spans the full width of the device up to max-w-lg -->
  <main class="w-full max-w-lg mx-auto h-full flex flex-col justify-start px-4 sm:px-6 py-3 gap-2.5 sm:gap-3 overflow-hidden">
    
    <!-- 1. HEADER (Top of Screen) -->
    <header class="w-full text-center shrink-0 pt-1">
      <h1 class="text-2xl sm:text-3xl font-black tracking-wider text-slate-900 uppercase">
        SUPRAYAS NGO
      </h1>
    </header>

    <!-- 2. SCANNER (Directly below Header, full device width, flexible height) -->
    <section class="relative w-full flex-1 min-h-[220px] max-h-[340px] bg-slate-900/5 rounded-3xl p-2 shadow-[0_8px_30px_rgba(15,23,42,0.06)] border border-slate-200/80 backdrop-blur-sm overflow-hidden flex items-center justify-center shrink-0 cursor-pointer" onclick="handleViewfinderClick()">
      
      <div id="reader" class="w-full h-full rounded-2xl overflow-hidden bg-slate-950"></div>

      <!-- Exactly 1 Set of 90-degree Corner Reticles -->
      <div class="absolute inset-5 pointer-events-none z-20 flex flex-col justify-between">
        <div class="flex justify-between w-full">
          <span class="reticle-corner w-7 h-7 sm:w-8 sm:h-8 border-t-[4px] border-l-[4px] border-slate-400/90 rounded-tl-xl"></span>
          <span class="reticle-corner w-7 h-7 sm:w-8 sm:h-8 border-t-[4px] border-r-[4px] border-slate-400/90 rounded-tr-xl"></span>
        </div>
        <div class="flex justify-between w-full">
          <span class="reticle-corner w-7 h-7 sm:w-8 sm:h-8 border-b-[4px] border-l-[4px] border-slate-400/90 rounded-bl-xl"></span>
          <span class="reticle-corner w-7 h-7 sm:w-8 sm:h-8 border-b-[4px] border-r-[4px] border-slate-400/90 rounded-br-xl"></span>
        </div>
      </div>

      <!-- Initial Loading / Camera Permission Overlay -->
      <div id="cameraStatus" class="absolute inset-0 bg-slate-900/95 text-white flex flex-col items-center justify-center p-4 text-center z-30 transition-opacity duration-300">
        <div class="w-9 h-9 rounded-2xl bg-white/10 flex items-center justify-center mb-2">
          <svg class="w-5 h-5 text-teal-400 animate-spin" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"></path>
          </svg>
        </div>
        <p id="cameraStatusText" class="text-xs font-semibold text-slate-200">Initializing camera...</p>
        <button id="retryBtn" onclick="event.stopPropagation(); startCamera();" class="hidden mt-3 px-4 py-1.5 bg-teal-500 hover:bg-teal-600 text-white text-xs font-bold rounded-lg shadow transition-all active:scale-95">
          Allow Camera
        </button>
      </div>
    </section>

    <!-- 3. BUTTONS (Directly below Scanner, spanning across the full width) -->
    <nav aria-label="Scanner Controls" class="flex items-center justify-center gap-2.5 w-full shrink-0">
      
      <!-- Button 1: Torch -->
      <button id="torchBtn" onclick="toggleTorch(event)" class="flex-1 py-2 px-1 flex flex-col items-center justify-center gap-0.5 bg-white text-slate-700 rounded-xl border border-slate-200/90 shadow-sm active:scale-95 transition-all">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
        </svg>
        <span class="text-[10px] font-bold">Torch</span>
      </button>

      <!-- Button 2: Scan (Scan A Icon: Camera Silhouette with Central Crosshair Reticle) -->
      <button id="scanToggleBtn" onclick="toggleScannerState(event)" class="flex-1 py-2 px-1 flex flex-col items-center justify-center gap-0.5 bg-teal-50 text-teal-600 border border-teal-200/90 rounded-xl shadow-sm active:scale-95 transition-all">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6.827 6.175A2.31 2.31 0 015.186 7.23c-.38.054-.757.112-1.134.175C2.999 7.58 2.25 8.507 2.25 9.574V18a2.25 2.25 0 002.25 2.25h15A2.25 2.25 0 0021.75 18V9.574c0-1.067-.75-1.994-1.802-2.169a47.865 47.865 0 00-1.134-.175 2.31 2.31 0 01-1.64-1.055l-.822-1.316a2.192 2.192 0 00-1.736-1.039 48.774 48.774 0 00-5.232 0 2.192 2.192 0 00-1.736 1.039l-.821 1.316z" />
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16.5 12.75a4.5 4.5 0 11-9 0 4.5 4.5 0 019 0z" />
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 11.25v3M10.5 12.75h3" />
        </svg>
        <span class="text-[10px] font-bold">Scan</span>
      </button>

      <!-- Button 3: Sound -->
      <button id="soundToggleBtn" onclick="toggleSoundState(event)" class="flex-1 py-2 px-1 flex flex-col items-center justify-center gap-0.5 bg-teal-50 text-teal-600 border border-teal-200/90 rounded-xl shadow-sm active:scale-95 transition-all">
        <svg id="soundIcon" class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19.114 5.636a9 9 0 010 12.728M16.463 8.288a5.25 5.25 0 010 7.424M6.75 8.25l4.72-4.72a.75.75 0 011.28.53v15.88a.75.75 0 01-1.28.53l-4.72-4.72H4.51c-.88 0-1.704-.507-1.938-1.354A9.01 9.01 0 012.25 12c0-.83.112-1.633.322-2.396C2.806 8.756 3.63 8.25 4.51 8.25H6.75z" />
        </svg>
        <span class="text-[10px] font-bold">Sound</span>
      </button>

    </nav>

    <!-- 4. SCAN HISTORY (Directly below Buttons, up to 5 items, flowing Top-to-Bottom) -->
    <section aria-label="Scan History" class="w-full shrink-0 flex flex-col gap-1.5 pb-2">
      <div id="historyList" class="flex flex-col gap-1.5 w-full"></div>
    </section>

  </main>

  <script>
    const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbwnQX1GfrfBYOibVH8BtkBBSNBFeK_4EMAvwBD1NhX-7fPRPg_Ff_mc6E6ecZL__QHp/exec";

    let html5QrCode = null;
    let isScanning = true;
    let isTorchOn = false;
    let isSoundOn = true;
    let isProcessing = false;
    let lastScannedCode = null;
    let lastScanTime = 0;
    let currentVideoTrack = null;
    let history = [];
    let audioCtx = null;
    let hapticsUnlocked = false;

    function fireHaptic() {
      if ('vibrate' in navigator) {
        try { navigator.vibrate([120, 50, 120]); } catch (e) {}
      }
    }

    function primeHardware() {
      try {
        const AudioCtxClass = window.AudioContext || window.webkitAudioContext;
        if (AudioCtxClass && (!audioCtx || audioCtx.state === 'suspended')) {
          if (!audioCtx) audioCtx = new AudioCtxClass();
          audioCtx.resume();
        }
        if ('vibrate' in navigator && !hapticsUnlocked) {
          navigator.vibrate(20);
          hapticsUnlocked = true;
        }
      } catch (e) {}

      window.removeEventListener('touchstart', primeHardware);
      window.removeEventListener('pointerdown', primeHardware);
      window.removeEventListener('click', primeHardware);
    }

    window.addEventListener('touchstart', primeHardware, { passive: true });
    window.addEventListener('pointerdown', primeHardware, { passive: true });
    window.addEventListener('click', primeHardware, { passive: true });

    function flashCorners() {
      const corners = document.querySelectorAll('.reticle-corner');
      corners.forEach(corner => corner.classList.add('reticle-flash'));
      setTimeout(() => {
        corners.forEach(corner => corner.classList.remove('reticle-flash'));
      }, 350);
    }

    function playHardwareBeep() {
      if (!isSoundOn) return;
      try {
        const AudioCtxClass = window.AudioContext || window.webkitAudioContext;
        if (!AudioCtxClass) return;

        if (!audioCtx || audioCtx.state === 'closed') {
          audioCtx = new AudioCtxClass();
        }

        const emitBeep = () => {
          const now = audioCtx.currentTime;
          const osc = audioCtx.createOscillator();
          const gain = audioCtx.createGain();

          osc.type = 'sine';
          osc.frequency.setValueAtTime(2400, now);

          gain.gain.setValueAtTime(0.001, now);
          gain.gain.linearRampToValueAtTime(0.4, now + 0.003);
          gain.gain.setValueAtTime(0.4, now + 0.040);
          gain.gain.linearRampToValueAtTime(0.001, now + 0.045);

          osc.connect(gain);
          gain.connect(audioCtx.destination);

          osc.start(now);
          osc.stop(now + 0.045);
        };

        if (audioCtx.state === 'suspended') {
          audioCtx.resume().then(emitBeep);
        } else {
          emitBeep();
        }
      } catch (err) {}
    }

    function loadHistory() {
      const stored = localStorage.getItem('suprayas_qr_history');
      if (stored) {
        try { history = JSON.parse(stored).slice(0, 5); } catch (e) { history = []; }
      } else {
        history = [];
      }
      renderHistory();
    }

    function saveHistory() {
      localStorage.setItem('suprayas_qr_history', JSON.stringify(history.slice(0, 5)));
    }

    function addHistoryItem(text) {
      const isUrl = /^https?:\/\//i.test(text);
      history.unshift({ text, isUrl });
      if (history.length > 5) history = history.slice(0, 5);
      saveHistory();
      renderHistory();
    }

    function renderHistory() {
      const container = document.getElementById('historyList');
      if (!container) return;

      if (history.length === 0) {
        container.innerHTML = `
          <div class="w-full text-center py-2 text-xs font-semibold text-slate-400">
            Awaiting scans...
          </div>
        `;
        return;
      }

      container.innerHTML = history.map((item) => `
        <div onclick="handleItemClick('${encodeURIComponent(item.text)}', ${item.isUrl})" 
             class="flex items-center justify-between px-3.5 py-2.5 bg-white rounded-xl border border-slate-200/80 shadow-sm active:scale-[0.99] transition-all cursor-pointer">
          <p class="text-sm font-bold text-slate-800 truncate tracking-tight pr-2 w-full">
            ${item.text}
          </p>
          <div class="text-slate-400 shrink-0">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M9 5l7 7-7 7" />
            </svg>
          </div>
        </div>
      `).join('');
    }

    function handleItemClick(encodedText, isUrl) {
      fireHaptic();
      const text = decodeURIComponent(encodedText);
      if (isUrl) {
        window.open(text, '_blank', 'noopener,noreferrer');
      } else if (navigator.clipboard) {
        navigator.clipboard.writeText(text);
      }
    }

    async function toggleTorch(e) {
      if (e) e.stopPropagation();
      fireHaptic();
      
      if (!currentVideoTrack) {
        const videoEl = document.querySelector('#reader video');
        if (videoEl && videoEl.srcObject) currentVideoTrack = videoEl.srcObject.getVideoTracks()[0];
      }

      if (!currentVideoTrack) return;

      const capabilities = currentVideoTrack.getCapabilities ? currentVideoTrack.getCapabilities() : {};
      if (!capabilities.torch) {
        isTorchOn = !isTorchOn;
        updateTorchButtonUI();
        return;
      }

      try {
        isTorchOn = !isTorchOn;
        await currentVideoTrack.applyConstraints({ advanced: [{ torch: isTorchOn }] });
        updateTorchButtonUI();
      } catch (err) {}
    }

    function updateTorchButtonUI() {
      const btn = document.getElementById('torchBtn');
      if (isTorchOn) {
        btn.className = "flex-1 py-2 px-1 flex flex-col items-center justify-center gap-0.5 bg-amber-400 text-slate-900 border border-amber-300 rounded-xl shadow-sm active:scale-95 transition-all";
      } else {
        btn.className = "flex-1 py-2 px-1 flex flex-col items-center justify-center gap-0.5 bg-white text-slate-700 border border-slate-200/90 rounded-xl shadow-sm active:scale-95 transition-all";
      }
    }

    function toggleScannerState(e) {
      if (e) e.stopPropagation();
      fireHaptic();
      isScanning = !isScanning;
      const btn = document.getElementById('scanToggleBtn');

      if (isScanning) {
        btn.className = "flex-1 py-2 px-1 flex flex-col items-center justify-center gap-0.5 bg-teal-50 text-teal-600 border border-teal-200/90 rounded-xl shadow-sm active:scale-95 transition-all";
        if (html5QrCode && html5QrCode.getState() === 3) html5QrCode.resume();
      } else {
        btn.className = "flex-1 py-2 px-1 flex flex-col items-center justify-center gap-0.5 bg-slate-100 text-slate-400 border border-slate-200 rounded-xl shadow-sm active:scale-95 transition-all";
        if (html5QrCode && html5QrCode.getState() === 2) html5QrCode.pause(true);
      }
    }

    function toggleSoundState(e) {
      if (e) e.stopPropagation();
      fireHaptic();
      isSoundOn = !isSoundOn;
      const btn = document.getElementById('soundToggleBtn');
      const icon = document.getElementById('soundIcon');

      if (isSoundOn) {
        btn.className = "flex-1 py-2 px-1 flex flex-col items-center justify-center gap-0.5 bg-teal-50 text-teal-600 border border-teal-200/90 rounded-xl shadow-sm active:scale-95 transition-all";
        icon.innerHTML = `<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19.114 5.636a9 9 0 010 12.728M16.463 8.288a5.25 5.25 0 010 7.424M6.75 8.25l4.72-4.72a.75.75 0 011.28.53v15.88a.75.75 0 01-1.28.53l-4.72-4.72H4.51c-.88 0-1.704-.507-1.938-1.354A9.01 9.01 0 012.25 12c0-.83.112-1.633.322-2.396C2.806 8.756 3.63 8.25 4.51 8.25H6.75z" />`;
        playHardwareBeep();
      } else {
        btn.className = "flex-1 py-2 px-1 flex flex-col items-center justify-center gap-0.5 bg-slate-100 text-slate-400 border border-slate-200 rounded-xl shadow-sm active:scale-95 transition-all";
        icon.innerHTML = `<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.25 9.75L19.5 12m0 0l2.25 2.25M19.5 12l2.25-2.25M19.5 12l-2.25 2.25m-10.5-7.5l4.72-4.72a.75.75 0 011.28.53v15.88a.75.75 0 01-1.28.53l-4.72-4.72H4.51c-.88 0-1.704-.507-1.938-1.354A9.01 9.01 0 012.25 12c0-.83.112-1.633.322-2.396C2.806 8.756 3.63 8.25 4.51 8.25H6.75z" />`;
      }
    }

    function logToGoogleSheet(code) {
      if (!SCRIPT_URL.startsWith("http")) return;
      fetch(SCRIPT_URL, {
        method: "POST",
        mode: "no-cors",
        headers: { "Content-Type": "text/plain;charset=utf-8" },
        body: JSON.stringify({ code: code })
      }).catch(() => {});
    }

    function onScanSuccess(decodedText) {
      const now = Date.now();
      if (!isScanning || isProcessing) return;
      if (decodedText === lastScannedCode && (now - lastScanTime) < 3500) return;

      isProcessing = true;
      lastScannedCode = decodedText;
      lastScanTime = now;

      fireHaptic();
      flashCorners();
      playHardwareBeep();
      addHistoryItem(decodedText);
      logToGoogleSheet(decodedText);

      setTimeout(() => { isProcessing = false; }, 800);
    }

    function handleViewfinderClick() {
      if (!isScanning) return;
      const testCodes = [
        "https://suprayas.org/member/1042",
        "DONATION-RECEIPT-8891",
        "SUPRAYAS-INV-2026-0391",
        "https://suprayas.org/verify/A981",
        "VOLUNTEER-BADGE-092"
      ];
      const randomCode = testCodes[Math.floor(Math.random() * testCodes.length)];
      onScanSuccess(randomCode);
    }

    async function startCamera() {
      const statusOverlay = document.getElementById('cameraStatus');
      const statusText = document.getElementById('cameraStatusText');
      const retryBtn = document.getElementById('retryBtn');

      statusOverlay.classList.remove('opacity-0', 'pointer-events-none');
      statusText.textContent = "Connecting camera...";
      retryBtn.classList.add('hidden');

      if (!html5QrCode) {
        html5QrCode = new Html5Qrcode("reader", { verbose: false });
      }

      try {
        await html5QrCode.start(
          { facingMode: "environment" },
          { fps: 30, formatsToSupport: [Html5QrcodeSupportedFormats.QR_CODE] },
          onScanSuccess,
          () => {}
        );
        bindActiveTrack();
        statusOverlay.classList.add('opacity-0', 'pointer-events-none');
      } catch (firstErr) {
        try {
          const devices = await Html5Qrcode.getCameras();
          if (devices && devices.length > 0) {
            const rearCamera = devices.find(d => 
              d.label.toLowerCase().includes('back') || 
              d.label.toLowerCase().includes('rear') || 
              d.label.toLowerCase().includes('environment')
            );
            const targetId = rearCamera ? rearCamera.id : devices[0].id;
            
            await html5QrCode.start(
              targetId,
              { fps: 30, formatsToSupport: [Html5QrcodeSupportedFormats.QR_CODE] },
              onScanSuccess,
              () => {}
            );
            bindActiveTrack();
            statusOverlay.classList.add('opacity-0', 'pointer-events-none');
          } else {
            throw new Error("No cameras detected");
          }
        } catch (secondErr) {
          statusText.innerHTML = "Camera permission needed.<br><span class='text-[10px] text-teal-300'>Tap below to allow camera access.</span>";
          retryBtn.classList.remove('hidden');
        }
      }
    }

    function bindActiveTrack() {
      setTimeout(() => {
        const videoEl = document.querySelector('#reader video');
        if (videoEl && videoEl.srcObject) {
          currentVideoTrack = videoEl.srcObject.getVideoTracks()[0];
        }
      }, 500);
    }

    window.addEventListener('DOMContentLoaded', () => {
      loadHistory();
      startCamera();
    });
  </script>
</body>
</html># Qr
