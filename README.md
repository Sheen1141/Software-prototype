<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Trash Detection UI</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body {
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: #f0f0f0;
      height: 100vh;
      font-family: sans-serif;
    }

    #screen {
      position: relative;
      width: 375px;
      height: 667px;
      border-radius: 10px;
      overflow: hidden;
      box-shadow: 0 0 20px rgba(0,0,0,0.4);
      background-image: url('https://images.pexels.com/photos/6281031/pexels-photo-6281031.jpeg?auto=compress&cs=tinysrgb&h=667');
      background-size: contain;
      background-position: center;
      background-repeat: no-repeat;
      background-color: black;
      cursor: pointer;
    }

    canvas {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 2;
    }

    .points-popup {
      position: absolute;
      color: #00cc44;
      font-weight: bold;
      font-size: 16px;
      animation: fadeUp 2s ease-out forwards;
      pointer-events: none;
      z-index: 3;
    }

    @keyframes fadeUp {
      0% {
        opacity: 1;
        transform: translateY(0);
      }
      100% {
        opacity: 0;
        transform: translateY(-40px);
      }
    }

    #message {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      color: white;
      font-size: 20px;
      font-weight: bold;
      text-align: center;
      display: none;
      z-index: 4;
    }

    #scan-text {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      color: white;
      font-size: 18px;
      font-weight: bold;
      z-index: 5;
      pointer-events: none;
    }
  </style>
</head>
<body>
  <div id="screen">
    <canvas id="canvas" width="375" height="667"></canvas>
    <div id="scan-text">Tap to scan</div>
    <div id="message">All litter has been collected</div>
  </div>

  <audio id="sound" src="http://cdn.freesound.org/previews/668/668791_6012605-lq.mp3"></audio>

  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const sound = document.getElementById('sound');
    const screen = document.getElementById('screen');
    const message = document.getElementById('message');
    const scanText = document.getElementById('scan-text');

    let scanned = false;

    // Red boxes = crumpled paper
    let redBoxes = [
      { x: 110, y: 400, w: 30, h: 30 },
      { x: 138, y: 420, w: 25, h: 35 },
      { x: 165, y: 400, w: 28, h: 28 }
    ];

    // Blue box = trash bin
    let blueBox = { x: 45, y: 335, w: 70, h: 100 };

    let showBoxes = false;

    function drawBoxes() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      if (!showBoxes) return;

      // Draw red boxes
      ctx.strokeStyle = 'red';
      ctx.lineWidth = 2;
      redBoxes.forEach(box => {
        ctx.strokeRect(box.x, box.y, box.w, box.h);
      });

      // Draw blue box
      ctx.strokeStyle = 'blue';
      ctx.lineWidth = 2;
      ctx.strokeRect(blueBox.x, blueBox.y, blueBox.w, blueBox.h);
    }

    function scanEffect(callback) {
      let y = 0;
      let speed = 5;

      function animate() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = 'rgba(0,255,0,0.3)';
        ctx.fillRect(0, y, canvas.width, 5);

        y += speed;

        if (y < canvas.height) {
          requestAnimationFrame(animate);
        } else {
          showBoxes = true;
          drawBoxes();
          if (callback) callback();
        }
      }

      animate();
    }

    function isInsideBox(x, y, box) {
      return (x >= box.x && x <= box.x + box.w && y >= box.y && y <= box.y + box.h);
    }

    function showPoints(x, y) {
      const popup = document.createElement('div');
      popup.className = 'points-popup';
      popup.innerText = '+5 green points';
      popup.style.left = `${x}px`;
      popup.style.top = `${y}px`;
      screen.appendChild(popup);
      setTimeout(() => {
        popup.remove();
      }, 2000);
    }

    function showCompletionUI() {
      message.style.display = 'block';
    }

    screen.addEventListener('click', (e) => {
      if (!scanned) {
        scanned = true;
        scanText.style.display = 'none';
        scanEffect(() => {
          screen.style.cursor = 'default';
        });
        return;
      }

      if (!showBoxes) return;

      const rect = canvas.getBoundingClientRect();
      const clickX = e.clientX - rect.left;
      const clickY = e.clientY - rect.top;

      for (let i = 0; i < redBoxes.length; i++) {
        if (isInsideBox(clickX, clickY, redBoxes[i])) {
          const box = redBoxes[i];
          sound.currentTime = 0;
          sound.play();
          showPoints(box.x + box.w / 2, box.y);
          redBoxes.splice(i, 1);
          drawBoxes();

          if (redBoxes.length === 0) {
            setTimeout(showCompletionUI, 500);
          }
          break;
        }
      }
    });

    drawBoxes(); // initially draw only background
  </script>
</body>
</html>

