<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8" />
  <title>Noob Shooter</title>
  <style>
    body { margin: 0; overflow: hidden; background: #000; }
    canvas { display: block; }
    #info {
      position: absolute; top: 10px; left: 10px;
      color: white; font-family: sans-serif;
      background: rgba(0,0,0,0.5); padding: 5px 10px; border-radius: 6px;
    }
  </style>
</head>
<body>
  <div id="info">WASD: Hareket • Fare: Bakış • Sol Tık: Ateş</div>
  <script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.158.0/examples/js/controls/PointerLockControls.js"></script>
  <script>
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // Işık
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(1, 2, 3);
    scene.add(light);
    scene.add(new THREE.AmbientLight(0x404040));

    // Zemin
    const groundGeo = new THREE.PlaneGeometry(200, 200);
    const groundMat = new THREE.MeshPhongMaterial({color: 0x228B22});
    const ground = new THREE.Mesh(groundGeo, groundMat);
    ground.rotation.x = -Math.PI/2;
    scene.add(ground);

    // Kontroller
    const controls = new THREE.PointerLockControls(camera, document.body);
    document.body.addEventListener('click', () => controls.lock());
    camera.position.y = 1.7; // göz yüksekliği

    // Hareket
    const keys = {};
    document.addEventListener('keydown', e => keys[e.code] = true);
    document.addEventListener('keyup', e => keys[e.code] = false);

    // Botlar
    const bots = [];
    const botGeo = new THREE.BoxGeometry(1, 2, 1);
    const botMat = new THREE.MeshPhongMaterial({color: 0xff0000});
    for (let i=0; i<5; i++) {
      const bot = new THREE.Mesh(botGeo, botMat.clone());
      bot.position.set((Math.random()-0.5)*50, 1, (Math.random()-0.5)*50);
      bot.hp = 3;
      scene.add(bot);
      bots.push(bot);
    }

    // Mermiler
    const bullets = [];

    document.addEventListener('mousedown', e => {
      if (e.button === 0) {
        const bulletGeo = new THREE.SphereGeometry(0.1, 8, 8);
        const bulletMat = new THREE.MeshBasicMaterial({color: 0xffff00});
        const bullet = new THREE.Mesh(bulletGeo, bulletMat);
        bullet.position.copy(camera.position);
        const dir = new THREE.Vector3();
        camera.getWorldDirection(dir);
        bullet.velocity = dir.clone().multiplyScalar(0.5);
        scene.add(bullet);
        bullets.push(bullet);
      }
    });

    // Animasyon döngüsü
    function animate() {
      requestAnimationFrame(animate);

      // Hareket
      const speed = 0.1;
      if (keys['KeyW']) controls.moveForward(speed);
      if (keys['KeyS']) controls.moveForward(-speed);
      if (keys['KeyA']) controls.moveRight(-speed);
      if (keys['KeyD']) controls.moveRight(speed);

      // Bot hareketi (noob geziyor)
      bots.forEach(bot => {
        bot.position.x += (Math.random()-0.5)*0.05;
        bot.position.z += (Math.random()-0.5)*0.05;
      });

      // Mermiler
      bullets.forEach((bullet, i) => {
        bullet.position.add(bullet.velocity);
        // Bot çarpışma
        bots.forEach((bot, j) => {
          if (bot.position.distanceTo(bullet.position) < 1) {
            bot.hp -= 1;
            scene.remove(bullet);
            bullets.splice(i,1);
            if (bot.hp <= 0) {
              scene.remove(bot);
              bots.splice(j,1);
            }
          }
        });
      });

      renderer.render(scene, camera);
    }
    animate();
  </script>
</body>
</html>
