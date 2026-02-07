<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Vòng quay lì xì Tết</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    canvas { background: #fff7ed; }
  </style>
</head>
<body class="min-h-screen flex items-center justify-center bg-orange-50">
  <div class="text-center space-y-4">
    <h1 class="text-2xl font-bold text-orange-600">🎉 Vòng quay lì xì Tết 🎉</h1>
    <canvas id="wheel" width="400" height="400" class="mx-auto rounded-full shadow-lg"></canvas>
    <div>
      <button id="spinBtn" class="px-6 py-2 bg-red-600 text-white rounded-xl shadow hover:bg-red-700">Quay số</button>
    </div>
    <p id="result" class="text-lg font-semibold text-green-700"></p>
  </div>

  <script>
    // Giá trị tiền (VND)
    const values = [5000, 10000, 20000, 30000, 40000, 50000];

    // Trọng số xác suất (càng lớn → càng dễ trúng)
    // 5k, 10k nhiều hơn
    const weights = [35, 30, 15, 10, 7, 3];

    const colors = ['#fde68a', '#fca5a5', '#fdba74', '#86efac', '#93c5fd', '#f9a8d4'];
    const canvas = document.getElementById('wheel');
    const ctx = canvas.getContext('2d');
    const center = canvas.width / 2;
    const radius = center - 10;

    let angle = 0;
    let spinning = false;
    let selectedIndex = 0;

    function drawWheel() {
      const slice = (2 * Math.PI) / values.length;
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      values.forEach((val, i) => {
        ctx.beginPath();
        ctx.moveTo(center, center);
        ctx.arc(center, center, radius, angle + i * slice, angle + (i + 1) * slice);
        ctx.fillStyle = colors[i];
        ctx.fill();

        ctx.save();
        ctx.translate(center, center);
        ctx.rotate(angle + i * slice + slice / 2);
        ctx.textAlign = 'right';
        ctx.fillStyle = '#7c2d12';
        ctx.font = 'bold 18px sans-serif';
        ctx.fillText(val / 1000 + 'k', radius - 20, 5);
        ctx.restore();
      });

      // Kim chỉ
      ctx.fillStyle = '#dc2626';
      ctx.beginPath();
      ctx.moveTo(center - 10, 10);
      ctx.lineTo(center + 10, 10);
      ctx.lineTo(center, 40);
      ctx.fill();
    }

    function getWeightedIndex() {
      const total = weights.reduce((a, b) => a + b, 0);
      let rand = Math.random() * total;
      for (let i = 0; i < weights.length; i++) {
        if (rand < weights[i]) return i;
        rand -= weights[i];
      }
      return weights.length - 1;
    }

    drawWheel();

    document.getElementById('spinBtn').onclick = () => {
      if (spinning) return;
      spinning = true;

      selectedIndex = getWeightedIndex();
      const sliceDeg = 360 / values.length;
      const targetAngle = 360 * 5 + (values.length - selectedIndex) * sliceDeg + sliceDeg / 2;

      const duration = 3000;
      const start = performance.now();

      function animate(time) {
        const progress = Math.min((time - start) / duration, 1);
        angle = (targetAngle * (1 - Math.pow(1 - progress, 3))) * Math.PI / 180;
        drawWheel();

        if (progress < 1) {
          requestAnimationFrame(animate);
        } else {
          spinning = false;
          // Tính kết quả THEO KIM CHỈ (chuẩn, không lệch)
          const sliceRad = (2 * Math.PI) / values.length;
          const normalizedAngle = (2 * Math.PI - (angle % (2 * Math.PI)) + Math.PI / 2) % (2 * Math.PI);
          const index = Math.floor(normalizedAngle / sliceRad);
          document.getElementById('result').innerText = `🎁 Chúc mừng! Anh trúng ${values[index] / 1000}k`;
        }
      }

      requestAnimationFrame(animate);
    };

    // Test đơn giản: log 1000 lượt quay để kiểm tra phân bố xác suất
    function testDistribution(times = 1000) {
      const count = Array(values.length).fill(0);
      for (let i = 0; i < times; i++) {
        count[getWeightedIndex()]++;
      }
      console.log('Kết quả test xác suất:', count);
    }
    // testDistribution(); // bỏ comment nếu anh muốn test
  </script>
</body>
</html>
