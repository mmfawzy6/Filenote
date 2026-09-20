<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <meta name="mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="theme-color" content="#1e1e1e">
  <title>Smart Note</title>
  
  <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiU21hcnQgTm90ZSIsInNob3J0X25hbWUiOiJOb3RlcyIsImRpc3BsYXkiOiJzdGFuZGFsb25lIiwic3RhcnRfdXJsIjoiLiIsImJhY2tncm91bmRfY29sb3IiOiIjMWExYTFhIiwidGhlbWVfY29sb3IiOiIjMWUxZTFlIn0=">

  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

  <style>
    * {
      box-sizing: border-box;
      -webkit-touch-callout: none;
      -webkit-user-select: none;
      user-select: none;
      touch-action: none; /* إلغاء تأخير واستجابة النظام الافتراضية لتحسين اللمس */
    }

    body {
      margin: 0;
      padding: 0;
      background-color: #1a1a1a;
      font-family: system-ui, -apple-system, sans-serif;
      display: flex;
      flex-direction: column;
      height: 100vh;
      overflow: hidden;
    }

    #toolbar {
      background-color: #242424;
      color: #fff;
      padding: 8px 12px;
      display: flex;
      gap: 10px;
      align-items: center;
      overflow-x: auto;
      white-space: nowrap;
      z-index: 100;
      touch-action: auto;
    }

    .tool-group {
      display: inline-flex;
      align-items: center;
      gap: 6px;
      border-left: 1px solid #444;
      padding-left: 8px;
    }

    label {
      font-size: 12px;
      color: #bbb;
    }

    select, input[type="color"], button {
      background: #333;
      color: #fff;
      border: 1px solid #555;
      padding: 6px 10px;
      border-radius: 6px;
      font-size: 13px;
      cursor: pointer;
    }

    button.active {
      background: #2563eb;
      border-color: #3b82f6;
    }

    button.btn-action {
      background: #059669;
      border: none;
    }

    button.btn-danger {
      background: #dc2626;
      border: none;
    }

    #viewport {
      flex: 1;
      position: relative;
      overflow: hidden;
      background-color: #2b2b2b;
      touch-action: none;
    }

    #sheet-container {
      position: absolute;
      top: 0;
      left: 0;
      transform-origin: 0 0;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.4);
    }

    canvas {
      display: block;
      background-color: #f8f6f0;
    }

    #status-overlay {
      position: absolute;
      bottom: 12px;
      right: 12px;
      background: rgba(0, 0, 0, 0.75);
      color: #eee;
      padding: 6px 12px;
      border-radius: 20px;
      font-size: 11px;
      pointer-events: none;
      z-index: 50;
    }
  </style>
</head>
<body>

  <div id="toolbar">
    <div class="tool-group">
      <button id="modePen" class="active">✏️ قلم</button>
      <button id="modeLasso">⭕ مسح بالتطويق</button>
    </div>

    <div class="tool-group">
      <label>القلم:</label>
      <input type="color" id="penColor" value="#111827">
      <select id="penSize">
        <option value="2">رفيع</option>
        <option value="4" selected>متوسط</option>
        <option value="8">عريض</option>
        <option value="22">تظليل</option>
      </select>
    </div>

    <div class="tool-group">
      <label>النمط:</label>
      <select id="styleSelect">
        <option value="lines" selected>أسطر (Lined)</option>
        <option value="todo">قائمة مهام (To-Do)</option>
        <option value="grid">مربعات (Grid)</option>
        <option value="blank">فارغ</option>
      </select>
    </div>

    <div class="tool-group">
      <button id="resetViewBtn">إعادة ضبط</button>
      <button id="saveImgBtn" class="btn-action">حفظ صورة</button>
      <button id="savePdfBtn" class="btn-action">حفظ PDF</button>
      <button id="clearBtn" class="btn-danger">مسح الكل</button>
    </div>
  </div>

  <div id="viewport">
    <div id="sheet-container">
      <canvas id="noteCanvas"></canvas>
    </div>
    <div id="status-overlay">👆 رسم | ✌️ تكبير/تصغير | 🖐️ 3 أصابع تحريك</div>
  </div>

  <script>
    const viewport = document.getElementById('viewport');
    const container = document.getElementById('sheet-container');
    const canvas = document.getElementById('noteCanvas');
    const ctx = canvas.getContext('2d', { desynchronized: true }); // تقليل زمن تأخير الرسم

    const penColor = document.getElementById('penColor');
    const penSize = document.getElementById('penSize');
    const styleSelect = document.getElementById('styleSelect');
    const resetViewBtn = document.getElementById('resetViewBtn');
    const clearBtn = document.getElementById('clearBtn');
    const saveImgBtn = document.getElementById('saveImgBtn');
    const savePdfBtn = document.getElementById('savePdfBtn');
    const modePen = document.getElementById('modePen');
    const modeLasso = document.getElementById('modeLasso');

    const CANVAS_WIDTH = 1200;
    const CANVAS_HEIGHT = 5000;
    canvas.width = CANVAS_WIDTH;
    canvas.height = CANVAS_HEIGHT;

    let scale = 1;
    let panX = 40;
    let panY = 40;

    let currentMode = 'pen';
    let strokes = [];
    let currentStroke = [];

    function applyTransform() {
      container.style.transform = `translate3d(${panX}px, ${panY}px, 0) scale(${scale})`;
    }

    function renderPaperPattern(style) {
      ctx.fillStyle = '#f8f6f0';
      ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);

      ctx.save();
      if (style === 'lines') {
        const spacing = 40;
        ctx.strokeStyle = '#d8dce2';
        ctx.lineWidth = 1.2;
        for (let y = spacing * 2; y < CANVAS_HEIGHT; y += spacing) {
          ctx.beginPath();
          ctx.moveTo(60, y);
          ctx.lineTo(CANVAS_WIDTH - 60, y);
          ctx.stroke();
        }
        ctx.strokeStyle = '#fca5a5';
        ctx.lineWidth = 1.5;
        ctx.beginPath();
        ctx.moveTo(120, 0);
        ctx.lineTo(120, CANVAS_HEIGHT);
        ctx.stroke();
      } else if (style === 'todo') {
        const spacing = 48;
        ctx.strokeStyle = '#e2e8f0';
        ctx.lineWidth = 1.2;
        for (let y = spacing * 1.5; y < CANVAS_HEIGHT; y += spacing) {
          ctx.beginPath();
          ctx.moveTo(50, y);
          ctx.lineTo(CANVAS_WIDTH - 50, y);
          ctx.stroke();
          ctx.strokeStyle = '#94a3b8';
          ctx.strokeRect(65, y - 24, 18, 18);
          ctx.strokeStyle = '#e2e8f0';
        }
      } else if (style === 'grid') {
        const grid = 30;
        ctx.strokeStyle = '#e5e7eb';
        ctx.lineWidth = 0.8;
        for (let x = grid; x < CANVAS_WIDTH; x += grid) {
          ctx.beginPath();
          ctx.moveTo(x, 0);
          ctx.lineTo(x, CANVAS_HEIGHT);
          ctx.stroke();
        }
        for (let y = grid; y < CANVAS_HEIGHT; y += grid) {
          ctx.beginPath();
          ctx.moveTo(0, y);
          ctx.lineTo(CANVAS_WIDTH, y);
          ctx.stroke();
        }
      }
      ctx.restore();
    }

    function redrawCanvas() {
      renderPaperPattern(styleSelect.value);
      for (const stroke of strokes) {
        if (stroke.points.length < 2) continue;
        ctx.save();
        ctx.beginPath();
        ctx.strokeStyle = stroke.color;
        ctx.lineWidth = stroke.size;
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';
        ctx.moveTo(stroke.points[0].x, stroke.points[0].y);
        for (let i = 1; i < stroke.points.length; i++) {
          ctx.lineTo(stroke.points[i].x, stroke.points[i].y);
        }
        ctx.stroke();
        ctx.restore();
      }
    }

    function fitToWidth() {
      const availableWidth = viewport.clientWidth - 40;
      scale = Math.min(Math.max(availableWidth / CANVAS_WIDTH, 0.3), 1);
      panX = (viewport.clientWidth - CANVAS_WIDTH * scale) / 2;
      panY = 20;
      applyTransform();
    }
    window.addEventListener('resize', fitToWidth);
    fitToWidth();
    redrawCanvas();

    modePen.addEventListener('click', () => {
      currentMode = 'pen';
      modePen.classList.add('active');
      modeLasso.classList.remove('active');
    });

    modeLasso.addEventListener('click', () => {
      currentMode = 'lasso';
      modeLasso.classList.add('active');
      modePen.classList.remove('active');
    });

    function isPointInsidePolygon(point, polygon) {
      let inside = false;
      for (let i = 0, j = polygon.length - 1; i < polygon.length; j = i++) {
        const xi = polygon[i].x, yi = polygon[i].y;
        const xj = polygon[j].x, yj = polygon[j].y;
        const intersect = ((yi > point.y) !== (yj > point.y)) &&
                          (point.x < (xj - xi) * (point.y - yi) / (yj - yi) + xi);
        if (intersect) inside = !inside;
      }
      return inside;
    }

    let isInteracting = false;
    let initialPinchDistance = 0;
    let initialPinchScale = 1;
    let pinchCenter = { x: 0, y: 0 };
    let threeFingerStart = { x: 0, y: 0 };
    let initialPan = { x: 0, y: 0 };

    function screenToCanvas(screenX, screenY) {
      return {
        x: (screenX - panX) / scale,
        y: (screenY - panY) / scale
      };
    }

    viewport.addEventListener('pointerdown', (e) => {
      if (e.pointerType === 'touch') return;
      handleDesktopStart(e);
    });

    viewport.addEventListener('touchstart', (e) => {
      e.preventDefault();
      if (e.touches.length === 1) {
        isInteracting = true;
        const pt = screenToCanvas(e.touches[0].clientX, e.touches[0].clientY);
        currentStroke = [pt];
        if (currentMode === 'pen') {
          const size = parseInt(penSize.value, 10);
          ctx.beginPath();
          ctx.moveTo(pt.x, pt.y);
          ctx.strokeStyle = size >= 20 ? penColor.value + '44' : penColor.value;
          ctx.lineWidth = size;
          ctx.lineCap = 'round';
          ctx.lineJoin = 'round';
        }
      } else if (e.touches.length === 2) {
        isInteracting = false;
        const t1 = e.touches[0], t2 = e.touches[1];
        initialPinchDistance = Math.hypot(t1.clientX - t2.clientX, t1.clientY - t2.clientY);
        initialPinchScale = scale;
        pinchCenter = { x: (t1.clientX + t2.clientX) / 2, y: (t1.clientY + t2.clientY) / 2 };
      } else if (e.touches.length === 3) {
        isInteracting = false;
        threeFingerStart = {
          x: (e.touches[0].clientX + e.touches[1].clientX + e.touches[2].clientX) / 3,
          y: (e.touches[0].clientY + e.touches[1].clientY + e.touches[2].clientY) / 3
        };
        initialPan = { x: panX, y: panY };
      }
    }, { passive: false });

    viewport.addEventListener('touchmove', (e) => {
      e.preventDefault();
      if (e.touches.length === 1 && isInteracting) {
        const pt = screenToCanvas(e.touches[0].clientX, e.touches[0].clientY);
        currentStroke.push(pt);

        if (currentMode === 'pen') {
          ctx.lineTo(pt.x, pt.y);
          ctx.stroke();
        } else if (currentMode === 'lasso') {
          redrawCanvas();
          ctx.save();
          ctx.strokeStyle = '#ef4444';
          ctx.lineWidth = 2;
          ctx.setLineDash([6, 6]);
          ctx.beginPath();
          ctx.moveTo(currentStroke[0].x, currentStroke[0].y);
          for (let i = 1; i < currentStroke.length; i++) {
            ctx.lineTo(currentStroke[i].x, currentStroke[i].y);
          }
          ctx.stroke();
          ctx.restore();
        }
      } else if (e.touches.length === 2) {
        const t1 = e.touches[0], t2 = e.touches[1];
        const dist = Math.hypot(t1.clientX - t2.clientX, t1.clientY - t2.clientY);
        if (initialPinchDistance > 0) {
          const newScale = Math.min(Math.max(initialPinchScale * (dist / initialPinchDistance), 0.25), 3.0);
          panX = pinchCenter.x - (pinchCenter.x - panX) * (newScale / scale);
          panY = pinchCenter.y - (pinchCenter.y - panY) * (newScale / scale);
          scale = newScale;
          applyTransform();
        }
      } else if (e.touches.length === 3) {
        const currentCenter = {
          x: (e.touches[0].clientX + e.touches[1].clientX + e.touches[2].clientX) / 3,
          y: (e.touches[0].clientY + e.touches[1].clientY + e.touches[2].clientY) / 3
        };
        panX = initialPan.x + (currentCenter.x - threeFingerStart.x);
        panY = initialPan.y + (currentCenter.y - threeFingerStart.y);
        applyTransform();
      }
    }, { passive: false });

    viewport.addEventListener('touchend', (e) => {
      if (e.touches.length === 0) {
        if (!isInteracting) return;
        isInteracting = false;

        if (currentMode === 'pen') {
          if (currentStroke.length > 1) {
            const size = parseInt(penSize.value, 10);
            strokes.push({
              points: [...currentStroke],
              color: size >= 20 ? penColor.value + '44' : penColor.value,
              size: size
            });
          }
          ctx.closePath();
        } else if (currentMode === 'lasso') {
          if (currentStroke.length > 5) {
            const loopPolygon = [...currentStroke];
            strokes = strokes.filter(s => !s.points.some(pt => isPointInsidePolygon(pt, loopPolygon)));
          }
          currentStroke = [];
          redrawCanvas();
        }
      }
    });

    styleSelect.addEventListener('change', redrawCanvas);
    resetViewBtn.addEventListener('click', fitToWidth);
    clearBtn.addEventListener('click', () => { strokes = []; redrawCanvas(); });

    saveImgBtn.addEventListener('click', () => {
      const link = document.createElement('a');
      link.download = 'smart-note.png';
      link.href = canvas.toDataURL('image/png');
      link.click();
    });

    savePdfBtn.addEventListener('click', () => {
      const { jsPDF } = window.jspdf;
      const a4Width = 595.28;
      const a4Height = 841.89;
      const pageHeightOnCanvas = CANVAS_WIDTH * (a4Height / a4Width);

      const doc = new jsPDF('p', 'pt', 'a4');
      const totalPages = Math.ceil(CANVAS_HEIGHT / pageHeightOnCanvas);

      for (let i = 0; i < totalPages; i++) {
        if (i > 0) doc.addPage('a4', 'p');
        const sliceCanvas = document.createElement('canvas');
        sliceCanvas.width = CANVAS_WIDTH;
        sliceCanvas.height = pageHeightOnCanvas;
        const sliceCtx = sliceCanvas.getContext('2d');
        sliceCtx.drawImage(canvas, 0, i * pageHeightOnCanvas, CANVAS_WIDTH, pageHeightOnCanvas, 0, 0, CANVAS_WIDTH, pageHeightOnCanvas);
        doc.addImage(sliceCanvas.toDataURL('image/jpeg', 0.95), 'JPEG', 0, 0, a4Width, a4Height);
      }
      doc.save('smart-note.pdf');
    });
  </script>
</body>
</html>
