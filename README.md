<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Game Kasir-Kasiran 3D | Belanja dan Bayar</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Arial', 'Segoe UI', sans-serif;
        }
        #info {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0,0,0,0.7);
            color: white;
            padding: 8px 15px;
            border-radius: 8px;
            pointer-events: none;
            z-index: 10;
            backdrop-filter: blur(5px);
            font-size: 14px;
            font-weight: bold;
            box-shadow: 0 2px 10px rgba(0,0,0,0.3);
        }
        #ui-panel {
            position: absolute;
            bottom: 20px;
            left: 20px;
            right: 20px;
            background: rgba(0,0,0,0.85);
            color: white;
            border-radius: 16px;
            padding: 12px 20px;
            display: flex;
            flex-wrap: wrap;
            justify-content: space-between;
            align-items: center;
            gap: 15px;
            backdrop-filter: blur(8px);
            pointer-events: auto;
            z-index: 20;
            font-weight: bold;
            border: 1px solid rgba(255,255,255,0.3);
            font-family: monospace;
        }
        .cash-display {
            background: #2e7d32;
            padding: 6px 12px;
            border-radius: 20px;
            color: gold;
            font-size: 1.3rem;
        }
        .cart-items {
            background: #1e1e2f;
            padding: 6px 12px;
            border-radius: 12px;
            max-width: 300px;
            overflow-x: auto;
            white-space: nowrap;
        }
        button {
            background: #ff9800;
            border: none;
            color: white;
            padding: 8px 16px;
            border-radius: 40px;
            font-weight: bold;
            cursor: pointer;
            transition: 0.2s;
            font-size: 1rem;
            box-shadow: 0 2px 6px black;
        }
        button:hover {
            background: #f57c00;
            transform: scale(1.02);
        }
        button:active {
            transform: scale(0.98);
        }
        .belanjaan-list {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
        }
        .item-btn {
            background: #4caf50;
            font-size: 0.9rem;
            padding: 5px 12px;
        }
        .total {
            background: #b71c1c;
            padding: 6px 12px;
            border-radius: 20px;
        }
        @media (max-width: 700px) {
            .ui-text { font-size: 12px; }
            button { padding: 6px 10px; font-size: 12px; }
        }
        #instruction {
            position: absolute;
            bottom: 100px;
            right: 15px;
            background: rgba(0,0,0,0.5);
            color: #ccc;
            font-size: 10px;
            padding: 4px 8px;
            border-radius: 8px;
            pointer-events: none;
        }
    </style>
</head>
<body>
    <div id="info">
        🛒 SIMULASI KASIR 3D 🧾 | Klik barang, lalu klik "Bayar"
    </div>
    <div id="ui-panel">
        <div>💰 Uang Kasir: <span id="uang-kasir" class="cash-display">Rp 50000</span></div>
        <div>🛍️ Keranjang: <span id="cart-count">0</span> item | Total: <span id="cart-total" class="total">Rp 0</span></div>
        <div class="belanjaan-list">
            <button id="beli-apel" class="item-btn">🍎 Apel (Rp 3000)</button>
            <button id="beli-susu" class="item-btn">🥛 Susu (Rp 7000)</button>
            <button id="beli-roti" class="item-btn">🍞 Roti (Rp 5000)</button>
            <button id="beli-keju" class="item-btn">🧀 Keju (Rp 12000)</button>
        </div>
        <div>
            <button id="btn-bayar">💵 BAYAR</button>
            <button id="btn-reset">🔄 Reset Toko</button>
        </div>
    </div>
    <div id="instruction">
        🖱️ Geser untuk rotasi kamera | Klik tombol belanja
    </div>

    <!-- Import Three.js core & add-ons -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
        import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

        // --- Setup Scene, Camera, Renderers ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x111122);
        scene.fog = new THREE.FogExp2(0x111122, 0.008);

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(6, 4, 8);
        camera.lookAt(0, 0, 0);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true; // bayangan realistis
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        document.body.appendChild(renderer.domElement);

        // CSS2DRenderer untuk teks label
        const labelRenderer = new CSS2DRenderer();
        labelRenderer.setSize(window.innerWidth, window.innerHeight);
        labelRenderer.domElement.style.position = 'absolute';
        labelRenderer.domElement.style.top = '0px';
        labelRenderer.domElement.style.left = '0px';
        labelRenderer.domElement.style.pointerEvents = 'none';
        document.body.appendChild(labelRenderer.domElement);

        // --- Controls ---
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.autoRotate = false;
        controls.enableZoom = true;
        controls.zoomSpeed = 1.2;
        controls.target.set(0, 1.2, 0);

        // --- Lighting ---
        // Ambient light
        const ambientLight = new THREE.AmbientLight(0x404060);
        scene.add(ambientLight);
        // Main directional light
        const dirLight = new THREE.DirectionalLight(0xfff5e6, 1);
        dirLight.position.set(5, 10, 7);
        dirLight.castShadow = true;
        dirLight.receiveShadow = true;
        dirLight.shadow.mapSize.width = 1024;
        dirLight.shadow.mapSize.height = 1024;
        scene.add(dirLight);
        // Fill light from below
        const fillLight = new THREE.PointLight(0x4466cc, 0.3);
        fillLight.position.set(0, -1, 0);
        scene.add(fillLight);
        // Warm back light
        const backLight = new THREE.PointLight(0xffaa66, 0.4);
        backLight.position.set(-2, 2, -3);
        scene.add(backLight);
        
        // Ground (lantai toko)
        const floorMat = new THREE.MeshStandardMaterial({ color: 0x8b5a2b, roughness: 0.7, metalness: 0.1 });
        const floor = new THREE.Mesh(new THREE.PlaneGeometry(12, 12), floorMat);
        floor.rotation.x = -Math.PI / 2;
        floor.position.y = -0.05;
        floor.receiveShadow = true;
        scene.add(floor);
        
        // Grid helper
        const gridHelper = new THREE.GridHelper(12, 20, 0xccaa88, 0xaa8866);
        gridHelper.position.y = -0.02;
        scene.add(gridHelper);
        
        // Meja kasir (counter)
        const counterBase = new THREE.Mesh(
            new THREE.BoxGeometry(1.8, 0.6, 1.5),
            new THREE.MeshStandardMaterial({ color: 0xbc9a6c, roughness: 0.4, metalness: 0.1 })
        );
        counterBase.position.set(2.2, 0.2, -1.5);
        counterBase.castShadow = true;
        counterBase.receiveShadow = true;
        scene.add(counterBase);
        
        const counterTop = new THREE.Mesh(
            new THREE.BoxGeometry(2.0, 0.1, 1.6),
            new THREE.MeshStandardMaterial({ color: 0xddbb99, roughness: 0.2, metalness: 0.05 })
        );
        counterTop.position.set(2.2, 0.55, -1.5);
        counterTop.castShadow = true;
        scene.add(counterTop);
        
        // Kasir (sebuah karakter sederhana + CSS2D)
        const cashierGroup = new THREE.Group();
        // badan
        const body = new THREE.Mesh(new THREE.BoxGeometry(0.6, 0.8, 0.5), new THREE.MeshStandardMaterial({ color: 0x4a90e2, roughness: 0.3 }));
        body.position.y = 0.4;
        body.castShadow = true;
        cashierGroup.add(body);
        // kepala
        const head = new THREE.Mesh(new THREE.SphereGeometry(0.35, 32, 32), new THREE.MeshStandardMaterial({ color: 0xffccaa }));
        head.position.y = 0.85;
        head.castShadow = true;
        cashierGroup.add(head);
        // topi
        const hat = new THREE.Mesh(new THREE.CylinderGeometry(0.4, 0.45, 0.12, 8), new THREE.MeshStandardMaterial({ color: 0xcc5555 }));
        hat.position.y = 1.05;
        hat.castShadow = true;
        cashierGroup.add(hat);
        cashierGroup.position.set(2.2, 0, -1.3);
        cashierGroup.castShadow = true;
        scene.add(cashierGroup);
        
        // Label CSS2D untuk kasir
        const cashierDiv = document.createElement('div');
        cashierDiv.textContent = '🧑‍💼 Kasir Ani';
        cashierDiv.style.backgroundColor = 'rgba(0,0,0,0.7)';
        cashierDiv.style.color = 'white';
        cashierDiv.style.padding = '2px 8px';
        cashierDiv.style.borderRadius = '20px';
        cashierDiv.style.fontSize = '14px';
        cashierDiv.style.fontWeight = 'bold';
        cashierDiv.style.border = '1px solid gold';
        const cashierLabel = new CSS2DObject(cashierDiv);
        cashierLabel.position.set(2.2, 1.2, -1.3);
        scene.add(cashierLabel);
        
        // --- Pembeli (karakter pelanggan) dengan animasi idle sederhana
        const buyerGroup = new THREE.Group();
        const buyerBody = new THREE.Mesh(new THREE.BoxGeometry(0.65, 0.85, 0.55), new THREE.MeshStandardMaterial({ color: 0x66bb6a }));
        buyerBody.position.y = 0.425;
        buyerBody.castShadow = true;
        buyerGroup.add(buyerBody);
        const buyerHead = new THREE.Mesh(new THREE.SphereGeometry(0.38, 32, 32), new THREE.MeshStandardMaterial({ color: 0xfdd0a4 }));
        buyerHead.position.y = 0.9;
        buyerHead.castShadow = true;
        buyerGroup.add(buyerHead);
        // rambut
        const hair = new THREE.Mesh(new THREE.CylinderGeometry(0.4, 0.42, 0.15, 8), new THREE.MeshStandardMaterial({ color: 0x8b5a2b }));
        hair.position.y = 1.05;
        buyerGroup.add(hair);
        buyerGroup.position.set(-1.8, 0, 0.5);
        buyerGroup.castShadow = true;
        scene.add(buyerGroup);
        
        // Label pelanggan
        const buyerDiv = document.createElement('div');
        buyerDiv.textContent = '🛒 Pembeli: Budi';
        buyerDiv.style.backgroundColor = 'rgba(0,0,0,0.6)';
        buyerDiv.style.color = '#ffd966';
        buyerDiv.style.padding = '2px 8px';
        buyerDiv.style.borderRadius = '20px';
        buyerDiv.style.fontSize = '13px';
        const buyerLabel = new CSS2DObject(buyerDiv);
        buyerLabel.position.set(-1.8, 1.25, 0.5);
        scene.add(buyerLabel);
        
        // --- Rak Belanjaan (display barang 3D)
        function createShelf(x, z, color, itemName) {
            const shelfMat = new THREE.MeshStandardMaterial({ color: 0xc9a87c });
            const shelf = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.8, 0.8), shelfMat);
            shelf.position.set(x, 0.4, z);
            shelf.castShadow = true;
            shelf.receiveShadow = true;
            scene.add(shelf);
            // barang kecil di rak
            const itemGeo = new THREE.SphereGeometry(0.2, 16, 16);
            const itemMat = new THREE.MeshStandardMaterial({ color: color });
            const item = new THREE.Mesh(itemGeo, itemMat);
            item.position.set(x + 0.3, 0.65, z + 0.2);
            item.castShadow = true;
            scene.add(item);
            // label CSS2D
            const div = document.createElement('div');
            div.textContent = itemName;
            div.style.backgroundColor = '#222';
            div.style.color = 'white';
            div.style.padding = '2px 6px';
            div.style.borderRadius = '12px';
            div.style.fontSize = '10px';
            const labelObj = new CSS2DObject(div);
            labelObj.position.set(x, 0.9, z);
            scene.add(labelObj);
            return item;
        }
        
        createShelf(-2, -1.2, 0xdd3333, "🍎 Apel");
        createShelf(-2.2, 1.0, 0xeeeeff, "🥛 Susu");
        createShelf(0.5, -1.8, 0xdd9944, "🍞 Roti");
        createShelf(1.5, 1.2, 0xffaa44, "🧀 Keju");
        
        // Tambahan beberapa kotak belanjaan dekorasi
        const decorBox = new THREE.Mesh(new THREE.BoxGeometry(0.6, 0.6, 0.6), new THREE.MeshStandardMaterial({ color: 0x66cc99 }));
        decorBox.position.set(-0.5, 0.3, -1.2);
        decorBox.castShadow = true;
        scene.add(decorBox);
        
        // --- Data game state ---
        // Daftar barang yang tersedia
        const itemsCatalog = [
            { id: 'apel', name: '🍎 Apel', price: 3000 },
            { id: 'susu', name: '🥛 Susu', price: 7000 },
            { id: 'roti', name: '🍞 Roti', price: 5000 },
            { id: 'keju', name: '🧀 Keju', price: 12000 }
        ];
        
        let cart = []; // array of {id, name, price}
        let cashierMoney = 50000; // uang kasir awal
        
        // DOM elements
        const uangKasirSpan = document.getElementById('uang-kasir');
        const cartCountSpan = document.getElementById('cart-count');
        const cartTotalSpan = document.getElementById('cart-total');
        
        function updateUI() {
            uangKasirSpan.textContent = `Rp ${cashierMoney.toLocaleString()}`;
            const total = cart.reduce((sum, item) => sum + item.price, 0);
            cartTotalSpan.textContent = `Rp ${total.toLocaleString()}`;
            cartCountSpan.textContent = cart.length;
        }
        
        // Tambah barang ke keranjang
        function addToCart(itemId) {
            const item = itemsCatalog.find(i => i.id === itemId);
            if (item) {
                cart.push({ ...item });
                updateUI();
                // efek animasi sederhana: getar sedikit pelanggan
                buyerGroup.position.y = 0.02;
                setTimeout(() => { buyerGroup.position.y = 0; }, 100);
                // suara (opsional) - tidak pakai audio agar tidak perlu ijin
                // tampilkan notifikasi lewat label sementara
                showFloatingMessage(`+ ${item.name}`, buyerGroup.position);
            }
        }
        
        function showFloatingMessage(msg, pos) {
            const div = document.createElement('div');
            div.textContent = msg;
            div.style.position = 'absolute';
            div.style.backgroundColor = '#4caf50';
            div.style.color = 'white';
            div.style.padding = '4px 8px';
            div.style.borderRadius = '20px';
            div.style.fontSize = '14px';
            div.style.fontWeight = 'bold';
            div.style.pointerEvents = 'none';
            div.style.transition = 'opacity 1s, transform 1s';
            div.style.opacity = '1';
            document.body.appendChild(div);
            // mapping 3d ke 2d
            const vector = pos.clone();
            vector.project(camera);
            const x = (vector.x * 0.5 + 0.5) * window.innerWidth;
            const y = (-vector.y * 0.5 + 0.5) * window.innerHeight;
            div.style.left = `${x - 30}px`;
            div.style.top = `${y - 40}px`;
            setTimeout(() => {
                div.style.opacity = '0';
                div.style.transform = 'translateY(-30px)';
                setTimeout(() => div.remove(), 1000);
            }, 50);
        }
        
        // Proses pembayaran
        function processPayment() {
            if (cart.length === 0) {
                showNotification("Keranjang kosong! Belanja dulu yuk.", "warning");
                return;
            }
            const total = cart.reduce((sum, i) => sum + i.price, 0);
            if (cashierMoney >= total) {
                cashierMoney -= total;
                // Tampilkan daftar barang yang dibeli
                const itemNames = cart.map(i => i.name).join(', ');
                showNotification(`✅ Pembayaran berhasil! Total: Rp ${total.toLocaleString()}\nBarang: ${itemNames}`, "success");
                // Kosongkan keranjang
                cart = [];
                updateUI();
                // Animasi senang pelanggan
                buyerGroup.position.y = 0.05;
                setTimeout(() => buyerGroup.position.y = 0, 200);
                // tambah efek uang melayang
                createFlyingCoinEffect();
            } else {
                showNotification(`❌ Uang kasir tidak cukup! Butuh Rp ${total.toLocaleString()}, uang kasir Rp ${cashierMoney.toLocaleString()}.`, "error");
            }
        }
        
        function createFlyingCoinEffect() {
            const coinGeo = new THREE.SphereGeometry(0.12, 8, 8);
            const coinMat = new THREE.MeshStandardMaterial({ color: 0xffaa33, emissive: 0x442200 });
            const coin = new THREE.Mesh(coinGeo, coinMat);
            coin.position.copy(buyerGroup.position);
            coin.position.y += 0.7;
            scene.add(coin);
            const targetPos = new THREE.Vector3(2.2, 1.0, -1.3);
            let t = 0;
            const duration = 400;
            const startPos = coin.position.clone();
            function animateCoin(time) {
                if (!coin.parent) return;
                t += 16;
                const progress = Math.min(1, t / duration);
                coin.position.lerpVectors(startPos, targetPos, progress);
                if (progress >= 1) {
                    scene.remove(coin);
                } else {
                    requestAnimationFrame(animateCoin);
                }
            }
            requestAnimationFrame(animateCoin);
        }
        
        function showNotification(msg, type) {
            // floating notification dengan style
            const notif = document.createElement('div');
            notif.textContent = msg;
            notif.style.position = 'fixed';
            notif.style.bottom = '90px';
            notif.style.left = '20px';
            notif.style.right = '20px';
            notif.style.maxWidth = '350px';
            notif.style.backgroundColor = type === 'success' ? '#2e7d32' : (type === 'error' ? '#c62828' : '#f9a825');
            notif.style.color = 'white';
            notif.style.padding = '12px';
            notif.style.borderRadius = '16px';
            notif.style.fontWeight = 'bold';
            notif.style.zIndex = '1000';
            notif.style.boxShadow = '0 4px 12px black';
            notif.style.backdropFilter = 'blur(4px)';
            notif.style.fontSize = '14px';
            notif.style.border = '1px solid gold';
            document.body.appendChild(notif);
            setTimeout(() => {
                notif.style.opacity = '0';
                notif.style.transition = 'opacity 0.5s';
                setTimeout(() => notif.remove(), 600);
            }, 2500);
        }
        
        function resetToko() {
            cart = [];
            cashierMoney = 50000;
            updateUI();
            showNotification("Toko direset! Uang kasir kembali Rp 50000, keranjang kosong.", "success");
            // efek animasi semangat
            cashierGroup.position.y = 0.03;
            setTimeout(() => cashierGroup.position.y = 0, 150);
        }
        
        // Koneksi tombol
        document.getElementById('beli-apel').addEventListener('click', () => addToCart('apel'));
        document.getElementById('beli-susu').addEventListener('click', () => addToCart('susu'));
        document.getElementById('beli-roti').addEventListener('click', () => addToCart('roti'));
        document.getElementById('beli-keju').addEventListener('click', () => addToCart('keju'));
        document.getElementById('btn-bayar').addEventListener('click', processPayment);
        document.getElementById('btn-reset').addEventListener('click', resetToko);
        
        // Animasi idle untuk karakter (gerakan ringan)
        let time = 0;
        
        // Partikel uang / coin hiasan di sekitar kasir
        const coinParticles = [];
        for (let i = 0; i < 30; i++) {
            const coinP = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 0.02, 6), new THREE.MeshStandardMaterial({ color: 0xffcc44, metalness: 0.8 }));
            coinP.userData = { speed: 0.005 + Math.random() * 0.01, angle: Math.random() * Math.PI * 2, radius: 0.8 + Math.random() * 1.2, yOffset: Math.random() * 1.5 };
            coinP.castShadow = true;
            scene.add(coinP);
            coinParticles.push(coinP);
        }
        
        // Animasi floating particles
        function animateParticles() {
            coinParticles.forEach((p, idx) => {
                const data = p.userData;
                data.angle += data.speed;
                const x = 2.2 + Math.cos(data.angle) * data.radius;
                const z = -1.3 + Math.sin(data.angle) * data.radius * 0.8;
                p.position.x = x;
                p.position.z = z;
                p.position.y = 0.6 + Math.sin(Date.now() * 0.003 + idx) * 0.15;
            });
        }
        
        // Sederhana animasi buyer dan kasir (idle bounce)
        let bounce = 0;
        
        // --- Animasi Render Loop ---
        function animate() {
            requestAnimationFrame(animate);
            time += 0.016;
            // idle bounce sangat halus
            bounce = Math.sin(time * 2.5) * 0.003;
            buyerGroup.position.y = bounce;
            cashierGroup.position.y = bounce * 0.8;
            // label ikut
            buyerLabel.position.y = 1.25 + bounce;
            cashierLabel.position.y = 1.2 + bounce * 0.8;
            
            animateParticles();
            
            controls.update(); // update orbit
            renderer.render(scene, camera);
            labelRenderer.render(scene, camera);
        }
        
        animate();
        
        // Resize handler
        window.addEventListener('resize', onWindowResize, false);
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            labelRenderer.setSize(window.innerWidth, window.innerHeight);
        }
        
        // Inisialisasi UI awal
        updateUI();
        
        // sedikit efek kilau background lampu
        const ambientGlow = new THREE.PointLight(0x88aaff, 0.5);
        ambientGlow.position.set(1, 2, 2);
        scene.add(ambientGlow);
        
        // menambahkan meja kecil untuk display uang
        const moneyPlate = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.05, 0.5), new THREE.MeshStandardMaterial({ color: 0xddbb77 }));
        moneyPlate.position.set(2.5, 0.2, -1.2);
        moneyPlate.castShadow = true;
        scene.add(moneyPlate);
        
        console.log("Game Kasir-Kasiran 3D Siap!");
    </script>
</body>
</html>
