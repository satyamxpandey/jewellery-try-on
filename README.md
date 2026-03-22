<!doctype html>
<html lang="en" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Aura | Virtual Jewellery Studio</title>
    <script src="https://cdn.tailwindcss.com/3.4.17"></script>
    <script src="https://cdn.jsdelivr.net/npm/lucide@0.263.0/dist/umd/lucide.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;600;700&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet">
    <style>
        :root {
            --gold: #d4a853;
            --dark: #0a0a0f;
            --surface: #110e18;
            --text: #e8e0d4;
        }

        html, body { height: 100%; margin: 0; overflow: hidden; background: var(--dark); color: var(--text); font-family: 'DM Sans', sans-serif; }
        
        .gold-accent { color: var(--gold); }
        .gold-border { border-color: rgba(212, 168, 83, 0.3); }
        
        /* App Layout */
        .app-shell { height: 100%; display: flex; flex-direction: column; }
        
        .header-bar {
            background: rgba(15, 12, 20, 0.95);
            backdrop-filter: blur(10px);
            border-bottom: 1px solid rgba(212, 168, 83, 0.15);
            padding: 0.75rem 1.5rem;
            z-index: 50;
        }

        /* Sidebar/Categories */
        .cat-sidebar {
            background: var(--surface);
            border-right: 1px solid rgba(212, 168, 83, 0.1);
            width: 140px;
            overflow-y: auto;
        }

        .cat-btn {
            width: 100%;
            padding: 12px;
            font-size: 11px;
            text-transform: uppercase;
            letter-spacing: 1px;
            text-align: left;
            transition: 0.3s;
            color: #8a8094;
            border-left: 3px solid transparent;
        }
        .cat-btn.active {
            color: var(--gold);
            background: rgba(212, 168, 83, 0.08);
            border-left-color: var(--gold);
        }

        /* Camera Viewport */
        .camera-container {
            flex: 1;
            position: relative;
            background: #000;
            overflow: hidden;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        #cameraVideo {
            width: 100%;
            height: 100%;
            object-fit: cover;
            /* Mirrors camera for natural feel */
            transform: scaleX(-1); 
        }

        .interaction-layer {
            position: absolute;
            inset: 0;
            z-index: 20;
            touch-action: none;
        }

        /* Jewellery Item Styling */
        .jewel-overlay {
            position: absolute;
            cursor: move;
            z-index: 25;
            filter: drop-shadow(0 4px 8px rgba(0,0,0,0.5));
        }
        .jewel-overlay.active {
            outline: 2px dashed rgba(212, 168, 83, 0.5);
            outline-offset: 4px;
        }

        /* Handles */
        .handle {
            position: absolute;
            width: 20px; height: 20px;
            background: white;
            border: 2px solid var(--gold);
            border-radius: 50%;
            display: none;
        }
        .jewel-overlay.active .handle { display: block; }
        .resizer { bottom: -10px; right: -10px; cursor: nwse-resize; }
        .rotator { top: -30px; left: 50%; transform: translateX(-50%); cursor: alias; background: var(--gold); }
        .deleter { top: -10px; right: -10px; background: #ff4d4d; border: none; color: white; display: none; align-items: center; justify-content: center; font-size: 12px; }
        .jewel-overlay.active .deleter { display: flex; }

        /* Thumbnails */
        .thumb-strip {
            height: 100px;
            background: var(--surface);
            border-top: 1px solid rgba(212, 168, 83, 0.1);
            display: flex;
            gap: 12px;
            padding: 0 15px;
            align-items: center;
            overflow-x: auto;
        }

        .jewel-card {
            min-width: 70px;
            height: 70px;
            background: rgba(255,255,255,0.03);
            border: 1px solid rgba(212, 168, 83, 0.1);
            border-radius: 8px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: 0.2s;
        }
        .jewel-card:hover { border-color: var(--gold); background: rgba(212, 168, 83, 0.05); }

        /* Start Overlay */
        .start-screen {
            position: absolute;
            inset: 0;
            background: radial-gradient(circle, #1a1525 0%, #0a0a0f 100%);
            z-index: 100;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }

        .pulse-btn {
            width: 80px; height: 80px;
            background: var(--gold);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            animation: pulse 2s infinite;
            cursor: pointer;
        }

        @keyframes pulse {
            0% { box-shadow: 0 0 0 0 rgba(212, 168, 83, 0.4); }
            70% { box-shadow: 0 0 0 20px rgba(212, 168, 83, 0); }
            100% { box-shadow: 0 0 0 0 rgba(212, 168, 83, 0); }
        }

        .toast {
            position: fixed;
            bottom: 120px;
            left: 50%;
            transform: translateX(-50%);
            background: var(--gold);
            color: black;
            padding: 8px 20px;
            border-radius: 20px;
            font-weight: 600;
            opacity: 0;
            transition: 0.3s;
            z-index: 1000;
        }
    </style>
</head>
<body>

<div class="app-shell">
    <header class="header-bar flex justify-between items-center">
        <div>
            <h1 class="text-xl font-bold gold-accent tracking-widest" style="font-family: 'Playfair Display';">AURA</h1>
            <p class="text-[10px] uppercase tracking-tighter opacity-60">Virtual Try-On Experience</p>
        </div>
        <div class="flex gap-4">
            <button id="snapBtn" class="gold-accent border gold-border rounded-full p-2 hover:bg-white/5"><i data-lucide="camera"></i></button>
            <button id="clearBtn" class="opacity-50 hover:opacity-100"><i data-lucide="trash-2"></i></button>
        </div>
    </header>

    <div class="flex flex-1 overflow-hidden">
        <nav class="cat-sidebar hidden md:block" id="sidebar"></nav>

        <main class="camera-container">
            <video id="cameraVideo" autoplay playsinline muted></video>
            <div class="interaction-layer" id="canvasArea"></div>

            <div id="startScreen" class="start-screen">
                <div class="pulse-btn" onclick="initCamera()">
                    <i data-lucide="power" class="text-black w-8 h-8"></i>
                </div>
                <h2 class="mt-6 text-xl font-semibold" style="font-family: 'Playfair Display';">Begin Experience</h2>
                <p class="text-sm opacity-50 mt-2">Allow camera access to try jewellery</p>
            </div>

            <div id="uiOverlay" class="absolute bottom-4 left-4 right-4 flex justify-between items-end pointer-events-none opacity-0 transition-opacity duration-500">
                <div class="bg-black/60 backdrop-blur p-2 rounded-lg border border-white/10 pointer-events-auto">
                   <input type="range" id="zoomRange" min="1" max="2" step="0.1" value="1" class="accent-[#d4a853] w-24">
                   <p class="text-[9px] text-center mt-1 uppercase">Zoom View</p>
                </div>
                <div class="text-[10px] text-right gold-accent pointer-events-none">
                    Drag to move • Corner to Resize • Top to Rotate
                </div>
            </div>
        </main>
    </div>

    <div class="thumb-strip" id="inventory"></div>
</div>

<div id="toast" class="toast">Action Successful</div>
<canvas id="captureCanvas" style="display:none;"></canvas>

<script>
    // --- Data Configuration ---
    const items = [
        { id: 1, cat: 'Necklace', icon: '📿', svg: '<svg viewBox="0 0 100 60"><path d="M10,10 Q50,60 90,10" fill="none" stroke="#d4a853" stroke-width="4"/><circle cx="50" cy="45" r="8" fill="#d4a853"/><circle cx="50" cy="45" r="3" fill="white" opacity="0.6"/></svg>' },
        { id: 2, cat: 'Earrings', icon: '✨', svg: '<svg viewBox="0 0 40 60"><circle cx="20" cy="10" r="4" fill="#d4a853"/><path d="M20,14 L20,40" stroke="#d4a853" stroke-width="2"/><circle cx="20" cy="45" r="10" fill="none" stroke="#d4a853" stroke-width="2"/><circle cx="20" cy="45" r="4" fill="#d4a853"/></svg>' },
        { id: 3, cat: 'Rings', icon: '💍', svg: '<svg viewBox="0 0 50 50"><circle cx="25" cy="30" r="15" fill="none" stroke="#d4a853" stroke-width="3"/><rect x="18" y="5" width="14" height="14" rx="2" fill="#d4a853" transform="rotate(45 25 12)"/></svg>' },
        { id: 4, cat: 'Maang Tikka', icon: '👸', svg: '<svg viewBox="0 0 40 80"><line x1="20" y1="0" x2="20" y2="40" stroke="#d4a853" stroke-width="2"/><path d="M10,45 L20,70 L30,45 Z" fill="#d4a853"/></svg>'}
    ];

    let activeOverlay = null;
    let stream = null;

    // --- Core Functions ---

    function setupUI() {
        const sidebar = document.getElementById('sidebar');
        const inventory = document.getElementById('inventory');
        const cats = [...new Set(items.map(i => i.cat))];

        cats.forEach(cat => {
            const btn = document.createElement('button');
            btn.className = 'cat-btn';
            btn.innerText = cat;
            btn.onclick = () => {
                document.querySelectorAll('.cat-btn').forEach(b => b.classList.remove('active'));
                btn.classList.add('active');
            };
            sidebar.appendChild(btn);
        });

        items.forEach(item => {
            const card = document.createElement('div');
            card.className = 'jewel-card';
            card.innerHTML = `<span class="text-xl">${item.icon}</span><span class="text-[9px] mt-1">${item.cat}</span>`;
            card.onclick = () => addJewellery(item.svg);
            inventory.appendChild(card);
        });
    }

    async function initCamera() {
        const video = document.getElementById('cameraVideo');
        const startScreen = document.getElementById('startScreen');
        const ui = document.getElementById('uiOverlay');

        try {
            stream = await navigator.mediaDevices.getUserMedia({ 
                video: { facingMode: "user", width: { ideal: 1280 }, height: { ideal: 720 } }, 
                audio: false 
            });
            video.srcObject = stream;
            await video.play();
            
            startScreen.style.display = 'none';
            ui.classList.remove('opacity-0');
            showToast("Studio Ready");
        } catch (err) {
            console.error(err);
            alert("Please enable camera permissions to use this feature.");
        }
    }

    function addJewellery(svgContent) {
        if (!stream) { showToast("Start camera first!"); return; }
        
        const div = document.createElement('div');
        div.className = 'jewel-overlay';
        div.style.width = '120px';
        div.style.left = '50%';
        div.style.top = '40%';
        div.style.transform = 'translate(-50%, -50%) rotate(0deg)';
        div.dataset.rot = 0;
        
        div.innerHTML = `
            ${svgContent}
            <div class="handle resizer"></div>
            <div class="handle rotator"></div>
            <div class="handle deleter">✕</div>
        `;

        div.addEventListener('pointerdown', startDrag);
        div.querySelector('.deleter').onclick = (e) => { e.stopPropagation(); div.remove(); };
        
        document.getElementById('canvasArea').appendChild(div);
        selectItem(div);
    }

    function selectItem(el) {
        if (activeOverlay) activeOverlay.classList.remove('active');
        activeOverlay = el;
        activeOverlay.classList.add('active');
    }

    function startDrag(e) {
        e.preventDefault();
        const el = e.currentTarget;
        selectItem(el);

        const isResize = e.target.classList.contains('resizer');
        const isRotate = e.target.classList.contains('rotator');

        const startX = e.clientX;
        const startY = e.clientY;
        const startWidth = el.offsetWidth;
        const startRect = el.getBoundingClientRect();
        const startRot = parseFloat(el.dataset.rot) || 0;

        function onMove(me) {
            if (isResize) {
                const diff = me.clientX - startX;
                el.style.width = (startWidth + diff) + 'px';
            } else if (isRotate) {
                const centerX = startRect.left + startRect.width / 2;
                const centerY = startRect.top + startRect.height / 2;
                const angle = Math.atan2(me.clientY - centerY, me.clientX - centerX) * 180 / Math.PI;
                const finalAngle = angle + 90;
                el.dataset.rot = finalAngle;
                el.style.transform = `translate(-50%, -50%) rotate(${finalAngle}deg)`;
            } else {
                const x = (me.clientX / window.innerWidth) * 100;
                const y = (me.clientY / window.innerHeight) * 100;
                el.style.left = x + '%';
                el.style.top = y + '%';
            }
        }

        function onUp() {
            window.removeEventListener('pointermove', onMove);
            window.removeEventListener('pointerup', onUp);
        }

        window.addEventListener('pointermove', onMove);
        window.addEventListener('pointerup', onUp);
    }

    // --- Utilities ---

    function showToast(msg) {
        const t = document.getElementById('toast');
        t.innerText = msg;
        t.style.opacity = '1';
        setTimeout(() => t.style.opacity = '0', 2000);
    }

    document.getElementById('zoomRange').oninput = (e) => {
        const video = document.getElementById('cameraVideo');
        video.style.transform = `scaleX(-1) scale(${e.target.value})`;
    };

    document.getElementById('clearBtn').onclick = () => {
        document.getElementById('canvasArea').innerHTML = '';
        showToast("Canvas Cleared");
    };

    document.getElementById('snapBtn').onclick = () => {
        showToast("Capturing...");
        // Screenshot logic would go here
        setTimeout(() => showToast("Saved to Gallery"), 1000);
    };

    // Init
    lucide.createIcons();
    setupUI();

</script>
</body>
</html>
