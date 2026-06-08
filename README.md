<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, orientation=landscape">
    <title>Anime Boxing - 3D Characters Edition</title>
    <style>
        :root {
            --anime-neon-blue: #00f0ff;
            --anime-neon-pink: #ff007f;
            --anime-neon-yellow: #ffea00;
        }

        body, html {
            margin: 0; padding: 0; width: 100%; height: 100%;
            overflow: hidden; background-color: #05050b;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            user-select: none; -webkit-user-select: none;
        }

        @media screen and (orientation: portrait) {
            #portrait-warning { display: flex !important; }
        }

        #portrait-warning {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: #0a0a12; color: white; z-index: 9999;
            justify-content: center; align-items: center; text-align: center;
            font-size: 20px; font-weight: bold; padding: 20px; box-sizing: border-box;
        }

        #game-ui {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            pointer-events: none; display: flex; flex-direction: column;
            justify-content: space-between; box-sizing: border-box; padding: 20px 40px;
        }

        .top-hud { display: flex; justify-content: space-between; align-items: center; width: 100%; }
        .status-container { width: 35%; display: flex; flex-direction: column; gap: 5px; pointer-events: auto; }
        .player-side { align-items: flex-start; }
        .enemy-side { align-items: flex-end; }
        .character-name { color: #fff; font-weight: bold; font-size: 16px; text-shadow: 0 0 8px rgba(255,255,255,0.6); }

        .bar-bg { width: 100%; height: 22px; background: rgba(0, 0, 0, 0.6); border: 2px solid #fff; transform: skew(-15deg); overflow: hidden; box-shadow: 0 0 15px rgba(0,0,0,0.7); }
        .hp-bar { height: 100%; width: 100%; transition: width 0.15s ease-out; }
        .player-side .hp-bar { background: linear-gradient(90deg, #39ff14, #00aa00); }
        .enemy-side .hp-bar { background: linear-gradient(90deg, #ff0000, #990000); }
        #regen-indicator { color: #ff4d4d; font-size: 12px; font-weight: bold; margin-top: 2px; opacity: 0; transition: opacity 0.3s; }
        .energy-bar { width: 0%; height: 6px; background: linear-gradient(90deg, var(--anime-neon-blue), #0055ff); transition: width 0.2s ease; }

        #combo-counter {
            position: absolute; top: 30%; left: 8%; color: var(--anime-neon-yellow);
            font-size: 36px; font-weight: 900; font-style: italic;
            text-shadow: 3px 3px 0px #000, 0 0 20px var(--anime-neon-pink);
            opacity: 0; transform: scale(0.5) rotate(-10deg); transition: all 0.1s ease;
        }
        #combo-counter.active { opacity: 1; transform: scale(1.2) rotate(-10deg); }

        .bottom-controls { display: flex; justify-content: space-between; align-items: flex-end; width: 100%; }
        .dpad-container { display: grid; grid-template-columns: repeat(3, 60px); grid-template-rows: repeat(2, 60px); gap: 8px; pointer-events: auto; }
        .action-container { display: grid; grid-template-columns: repeat(2, 75px); gap: 12px; pointer-events: auto; }

        .control-btn {
            background: rgba(10, 10, 25, 0.85); border: 2px solid var(--anime-neon-blue); border-radius: 14px;
            color: #fff; font-size: 15px; font-weight: bold; display: flex; align-items: center; justify-content: center;
            box-shadow: 0 0 15px rgba(0, 240, 255, 0.3); transition: all 0.1s ease;
        }
        .control-btn:active { background: var(--anime-neon-blue); color: #000; box-shadow: 0 0 25px var(--anime-neon-blue); transform: scale(0.9); }
        .action-btn { border-color: var(--anime-neon-pink); box-shadow: 0 0 15px rgba(255, 0, 127, 0.3); border-radius: 50%; height: 75px; }
        .action-btn:active { background: var(--anime-neon-pink); box-shadow: 0 0 25px var(--anime-neon-pink); }
        
        .ult-btn { grid-column: span 2; height: 50px; border-radius: 10px; border-color: var(--anime-neon-yellow); background: linear-gradient(185deg, rgba(50,50,0,0.8), rgba(20,20,0,0.95)); color: var(--anime-neon-yellow); font-size: 13px; }
        .ult-btn:active { background: var(--anime-neon-yellow); color: #000; }

        .btn-up { grid-column: 2; }
        .btn-left { grid-column: 1; grid-row: 2; }
        .btn-down { grid-column: 2; grid-row: 2; }
        .btn-right { grid-column: 3; grid-row: 2; }
    </style>
    
    <!-- استدعاء Three.js ومستدعي المجسمات GLTFLoader -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/loaders/GLTFLoader.js"></script>
</head>
<body>

    <div id="portrait-warning">🔄 يرجى تدوير الهاتف للوضع الأفقي!</div>

    <div id="game-ui">
        <div class="top-hud">
            <div class="status-container player-side">
                <div class="character-name">البطل (حسين)</div>
                <div class="bar-bg"><div class="hp-bar" id="player-hp"></div></div>
                <div class="bar-bg" style="width:65%; height:6px; margin-top:2px;"><div class="energy-bar" id="player-energy"></div></div>
            </div>
            <div class="status-container enemy-side">
                <div class="character-name">الخصم السايبورغ</div>
                <div class="bar-bg"><div class="hp-bar" id="enemy-hp"></div></div>
                <div id="regen-indicator">تجديد الطاقة نشط (+⚡)</div>
            </div>
        </div>

        <div id="combo-counter">0 HIT!</div>

        <div class="bottom-controls">
            <div class="dpad-container">
                <div class="control-btn btn-up" id="pad-up">▲</div>
                <div class="control-btn btn-left" id="pad-left">◀</div>
                <div class="control-btn btn-down" id="pad-down">▼</div>
                <div class="control-btn btn-right" id="pad-right">▶</div>
            </div>
            <div class="action-container">
                <div class="control-btn action-btn" id="attack-light">خفيفة</div>
                <div class="control-btn action-btn" id="attack-heavy">قوية</div>
                <div class="control-btn ult-btn" id="attack-ult">الضربة القاضية (ULT)</div>
            </div>
        </div>
    </div>

    <script>
        // --- 1. إعداد البيئة الثلاثية الأبعاد والإضاءة الحية لشخصيات الأنيمي ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x0a0a1a);
        scene.fog = new THREE.FogExp2(0x0a0a1a, 0.05); // ضباب خفيف لإعطاء عمق رهيب للحلبة

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true; // تفعيل الظلال لتبدو الشخصيات واقعية
        document.body.appendChild(renderer.domElement);

        // إضاءة سينمائية (Spotlight و Ambient) لتجعل مظهر الأنيمي يتوهج
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);

        const spotLight = new THREE.SpotLight(0x00f0ff, 2);
        spotLight.position.set(0, 10, 5);
        spotLight.castShadow = true;
        scene.add(spotLight);

        // أرضية الحلبة
        const ring = new THREE.Mesh(
            new THREE.CylinderGeometry(6, 6.5, 0.4, 32), 
            new THREE.MeshStandardMaterial({ color: 0x151528, roughness: 0.3 })
        );
        ring.position.y = -0.2;
        ring.receiveShadow = true;
        scene.add(ring);

        // --- 2. محرك استدعاء وتحريك الشخصيات ثلاثية الأبعاد (GLTF) ---
        const loader = new THREE.GLTFLoader();
        let playerMesh, enemyMesh;
        let playerMixer, enemyMixer; // لإدارة أنيميشن الشخصيات
        const clock = new THREE.Clock();

        // 💡 ملاحظة: استبدل هذه الروابط بروابط شخصياتك المرفوعة لاحقاً
        // حالياً قمت بوضع مجسمات تمهيدية ذكية تتفعل فور وضع رابط الـ GLB
        
        // استدعاء شخصية اللاعب
        loader.load('https://threejs.org/examples/models/gltf/RobotExpressive/RobotExpressive.glb', (gltf) => {
            playerMesh = gltf.scene;
            playerMesh.position.set(-1.8, 0, 0);
            playerMesh.scale.set(0.4, 0.4, 0.4); // ضبط الحجم
            playerMesh.rotation.y = Math.PI / 2; // جعلها تواجه الخصم
            playerMesh.castShadow = true;
            scene.add(playerMesh);

            // تفعيل الأنيميشن الخاص بالشخصية إذا كان مدمجاً بها
            playerMixer = new THREE.AnimationMixer(playerMesh);
            if(gltf.animations.length > 0) {
                // تشغيل حركة الوقوف الافتراضية للقتال (مثال: الإيماء أو الحركة التلقائية)
                playerMixer.clipAction(gltf.animations[0]).play();
            }
        });

        // استدعاء شخصية الخصم
        loader.load('https://threejs.org/examples/models/gltf/RobotExpressive/RobotExpressive.glb', (gltf) => {
            enemyMesh = gltf.scene;
            enemyMesh.position.set(1.8, 0, 0);
            enemyMesh.scale.set(0.4, 0.4, 0.4);
            enemyMesh.rotation.y = -Math.PI / 2;
            enemyMesh.castShadow = true;
            scene.add(enemyMesh);

            enemyMixer = new THREE.AnimationMixer(enemyMesh);
            if(gltf.animations.length > 0) {
                enemyMixer.clipAction(gltf.animations[3]).play(); // حركة مختلفة للخصم
            }
        });

        // ضبط الكاميرا السينمائية خلف اللاعب بمسافة جانبية
        camera.position.set(0, 2.5, 6);
        camera.lookAt(0, 0.8, 0);

        // --- 3. نظام قتال وحسابات الـ UI وتجدد الطاقة ---
        let combo = 0, playerHP = 100, enemyHP = 100, playerEnergy = 0, lastHitTime = 0;
        const enemyHpBar = document.getElementById('enemy-hp'), playerHpBar = document.getElementById('player-hp');
        const playerEnergyBar = document.getElementById('player-energy'), comboText = document.getElementById('combo-counter'), regenIndicator = document.getElementById('regen-indicator');

        function playerAttack(damage) {
            if(playerMesh) {
                // أنيميشن مبرمج سريع للاندفاع عند اللكم في حال لم تتوفر ملفات أنيميشن خارجية للكمة
                playerMesh.position.x += 0.5;
                setTimeout(() => playerMesh.position.x -= 0.5, 80);
            }

            enemyHP = Math.max(0, enemyHP - damage);
            enemyHpBar.style.width = enemyHP + '%';
            lastHitTime = Date.now();
            regenIndicator.style.opacity = 0;

            if(playerEnergy < 100) {
                playerEnergy = Math.min(100, playerEnergy + (damage * 1.5));
                playerEnergyBar.style.width = playerEnergy + '%';
            }

            combo++;
            comboText.innerText = combo + " HIT!";
            comboText.classList.add('active');
            clearTimeout(window.comboTimeout);
            window.comboTimeout = setTimeout(() => { comboText.classList.remove('active'); combo = 0; }, 1200);
        }

        // حلقة تجدد طاقة الخصم تلقائياً
        setInterval(() => {
            if (Date.now() - lastHitTime > 1200 && enemyHP > 0 && enemyHP < 100) {
                enemyHP = Math.min(100, enemyHP + 0.5);
                enemyHpBar.style.width = enemyHP + '%';
                regenIndicator.style.opacity = 1;
            } else {
                if (enemyHP >= 100 || enemyHP <= 0) regenIndicator.style.opacity = 0;
            }
        }, 100);

        // هجوم الخصم التلقائي
        setInterval(() => {
            if (enemyHP > 0 && enemyMesh) {
                playerHP = Math.max(0, playerHP - Math.floor(Math.random() * 5 + 2));
                playerHpBar.style.width = playerHP + '%';
                enemyMesh.position.x -= 0.4;
                setTimeout(() => enemyMesh.position.x += 0.4, 80);
            }
        }, 2000);

        // ربط الأزرار
        document.getElementById('attack-light').addEventListener('touchstart', (e) => { e.preventDefault(); playerAttack(5); });
        document.getElementById('attack-heavy').addEventListener('touchstart', (e) => { e.preventDefault(); playerAttack(12); });
        document.getElementById('attack-ult').addEventListener('touchstart', (e) => {
            e.preventDefault();
            if(playerEnergy >= 100) { playerAttack(30); playerEnergy = 0; playerEnergyBar.style.width = '0%'; }
        });

        // تحريك شخصية اللاعب في الحلبة عبر الـ D-Pad
        document.getElementById('pad-left').addEventListener('touchstart', (e) => { e.preventDefault(); if(playerMesh) playerMesh.position.z += 0.25; });
        document.getElementById('pad-right').addEventListener('touchstart', (e) => { e.preventDefault(); if(playerMesh) playerMesh.position.z -= 0.25; });
        document.getElementById('pad-up').addEventListener('touchstart', (e) => { e.preventDefault(); if(playerMesh) playerMesh.position.x += 0.25; });
        document.getElementById('pad-down').addEventListener('touchstart', (e) => { e.preventDefault(); if(playerMesh) playerMesh.position.x -= 0.25; });

        // Game Loop - تحديث حركة الأنيميشن للشخصيات في كل إطار
        function animate() {
            requestAnimationFrame(animate);
            const delta = clock.getDelta();
            
            // تحديث محرك الأنيميشن في كل فريم لتتحرك الشخصية بسلاسة
            if (playerMixer) playerMixer.update(delta);
            if (enemyMixer) enemyMixer.update(delta);

            renderer.render(scene, camera);
        }

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>

