<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Smart Note</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    * {
      box-sizing: border-box;
      -webkit-touch-callout: none;
      -webkit-user-select: none;
      user-select: none;
    }

    body {
      margin: 0;
      padding: 0;
      background-color: #2b2b2b;
      font-family: sans-serif;
      display: flex;
      flex-direction: column;
      height: 100vh;
      overflow: hidden;
    }

    #toolbar {
      background-color: #1e1e1e;
      color: #fff;
      padding: 8px 10px;
      display: flex;
      gap: 8px;
      align-items: center;
      overflow-x: auto;
      white-space: nowrap;
      z-index: 10;
    }

    .tool-group {
      display: inline-flex;
      align-items: center;
      gap: 5px;
      border-right: 1px solid #444;
      padding-right: 8px;
    }

    select, input[type="color"], button {
      background: #333;
      color: #fff;
      border: 1px solid #555;
      padding: 6px 10px;
      border-radius: 4px;
      font-size: 13px;
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
      box-shadow: 0 6px 20px rgba(0, 0, 0, 0.5);
    }

    canvas {
      display: block;
      background-color: #f8f6f0;
    }

    #hint {
      position: absolute;
      bottom: 10px;
      right: 10px;
      background: rgba(0, 0, 0, 0.7);
      color: #ddd;
      padding: 5px 10px;
      border-radius: 15px;
      font-size: 11px;
      pointer-events: none;
    }
  </style>
</head>
<body>

  <div id="toolbar">
    <div class="tool-group">
      <button id="modePen" class="active">✏️ Pen</button>
      <button id="modeLasso">⭕ Lasso Erase</button>
    </div>

    <div class="tool-group">
      <input type="color" id="penColor" value="#000000">
      <select id="penSize">
        <option value="2">Fine</option>
        <option value="4" selected>Medium</option>
        <option value="8">Bold</option>
      </select>
    </div>

    <div class="tool-group">
      <select id="styleSelect">
        <option value="lines" selected>Lined</option>
        <option value="todo">To-Do</option>
        <option value="grid">Grid</option>
        <option value="blank">Blank</option>
      </select>
    </div>

    <div class="tool-group">
      <button id="resetBtn">Reset View</button>
      <button id="saveImgBtn" class="btn-action">PNG</button>
      <button id="savePdfBtn" class="btn-action">PDF</button>
      <button id="clearBtn" class="btn-danger">Clear</button>
    </div>
  </div>

  <div id="viewport">
    <div id="sheet-container">
      <canvas id="noteCanvas"></canvas>
    </div>
    <div id="hint">1 Finger: Draw | 2 Fingers: Zoom | 3 Fingers: Pan</div>
  </div>

  <script>
    window.addEventListener('DOMContentLoaded', () => {
      const viewport = document.getElementById('viewport');
      const container = document.getElementById('sheet-container');
      const canvas = document.getElementById('noteCanvas');
      const ctx = canvas.getContext('2d');

      const penColor = document.getElementById('penColor');
      const penSize = document.getElementById('penSize');
      const styleSelect = document.getElementById('styleSelect');
      const resetBtn = document.getElementById('resetBtn');
      const clearBtn = document.getElementById('clearBtn');
      const saveImgBtn = document.getElementById('saveImgBtn');
      const savePdfBtn = document.getElementById('savePdfBtn');
      const modePen = document.getElementById('modePen');
      const modeLasso = document.getElementById('modeLasso');

      const CANVAS_WIDTH = 1200;
      const CANVAS_HEIGHT = 4000;
      canvas.width = CANVAS_WIDTH;
      canvas.height = CANVAS_HEIGHT;

      let scale = 1;
      let panX = 20;
      let panY = 20;

      let currentMode = 'pen';
      let strokes = [];
      let currentStroke = [];

      function applyTransform() {
        container.style.transform = `translate(${panX}px, ${panY}px) scale(${scale})`;
      }

      function drawPattern(style) {
        ctx.fillStyle = '#f8f6f0';
        ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);

        ctx.save();
        if (style === 'lines') {
          const spacing = 40;
          ctx.strokeStyle = '#d8dce2';
          ctx.lineWidth = 1;
          for (let y = spacing * 2; y < CANVAS_HEIGHT; y += spacing) {
            ctx.beginPath();
            ctx.moveTo(40, y);
            ctx.lineTo(CANVAS_WIDTH - 40, y);
            ctx.stroke();
          }
          ctx.strokeStyle = '#fca5a5';
          ctx.beginPath();
          ctx.moveTo(100, 0);
          ctx.lineTo(100, CANVAS_HEIGHT);
          ctx.stroke();
        } else if (style === 'todo') {
          const spacing = 46;
          ctx.strokeStyle = '#e2e8f0';
          ctx.lineWidth = 1;
          for (let y = spacing * 1.5; y < CANVAS_HEIGHT; y += spacing) {
            ctx.beginPath();
            ctx.moveTo(40, y);
            ctx.lineTo(CANVAS_WIDTH - 40, y);
            ctx.stroke();
            ctx.strokeStyle = '#94a3b8';
            ctx.strokeRect(50, y - 22, 16, 16);
            ctx.strokeStyle = '#e2e8f0';
          }
        } else if (style === 'grid') {
          const grid = 30;
          ctx.strokeStyle = '#e5e7eb';
          ctx.lineWidth = 1;
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

      function redraw() {
        drawPattern(styleSelect.value);
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
        const rect = viewport.getBoundingClientRect();
        if (rect.width > 0) {
          scale = Math.min((rect.width - 20) / CANVAS_WIDTH, 1);
          panX = (rect.width - CANVAS_WIDTH * scale) / 2;
          panY = 10;
          applyTransform();
        }
      }

      fitToWidth();
      redraw();

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

      function toCanvas(clientX, clientY) {
        const vRect = viewport.getBoundingClientRect();
        const screenX = clientX - vRect.left;
        const screenY = clientY - vRect.top;
        return {
          x: (screenX - panX) / scale,
          y: (screenY - panY) / scale
        };
      }

      function pointInPoly(pt, poly) {
        let inside = false;
        for (let i = 0, j = poly.length - 1; i < poly.length; j = i++) {
          const xi = poly[i].x, yi = poly[i].y;
          const xj = poly[j].x, yj = poly[j].y;
          const intersect = ((yi > pt.y) !== (yj > pt.y)) &&
            (pt.x < (xj - xi) * (pt.y - yi) / (yj - yi) + xi);
          if (intersect) inside = !inside;
        }
        return inside;
      }

      let isInteracting = false;
      let initDist = 0;
      let initScale = 1;
      let pinchMid = { x: 0, y: 0 };
      let panStart = { x: 0, y: 0 };
      let startPanPos = { x: 0, y: 0 };

      viewport.addEventListener('touchstart', (e) => {
        e.preventDefault();

        if (e.touches.length === 1) {
          isInteracting = true;
          const pt = toCanvas(e.touches[0].clientX, e.touches[0].clientY);
          currentStroke = [pt];

          if (currentMode === 'pen') {
            ctx.beginPath();
            ctx.moveTo(pt.x, pt.y);
            ctx.strokeStyle = penColor.value;
            ctx.lineWidth = parseInt(penSize.value, 10);
            ctx.lineCap = 'round';
            ctx.lineJoin = 'round';
          }
        } else if (e.touches.length === 2) {
          isInteracting = false;
          const t1 = e.touches[0], t2 = e.touches[1];
          initDist = Math.hypot(t1.clientX - t2.clientX, t1.clientY - t2.clientY);
          initScale = scale;
          const vRect = viewport.getBoundingClientRect();
          pinchMid = {
            x: ((t1.clientX + t2.clientX) / 2) - vRect.left,
            y: ((t1.clientY + t2.clientY) / 2) - vRect.top
          };
        } else if (e.touches.length === 3) {
          isInteracting = false;
          panStart = {
            x: (e.touches[0].clientX + e.touches[1].clientX + e.touches[2].clientX) / 3,
            y: (e.touches[0].clientY + e.touches[1].clientY + e.touches[2].clientY) / 3
          };
          startPanPos = { x: panX, y: panY };
        }
      }, { passive: false });

      viewport.addEventListener('touchmove', (e) => {
        e.preventDefault();

        if (e.touches.length === 1 && isInteracting) {
          const pt = toCanvas(e.touches[0].clientX, e.touches[0].clientY);
          currentStroke.push(pt);

          if (currentMode === 'pen') {
            ctx.lineTo(pt.x, pt.y);
            ctx.stroke();
          } else if (currentMode === 'lasso') {
            redraw();
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
          if (initDist > 0) {
            const nextScale = Math.min(Math.max(initScale * (dist / initDist), 0.2), 3);
            panX = pinchMid.x - (pinchMid.x - panX) * (nextScale / scale);
            panY = pinchMid.y - (pinchMid.y - panY) * (nextScale / scale);
            scale = nextScale;
            applyTransform();
          }
        } else if (e.touches.length === 3) {
          const curX = (e.touches[0].clientX + e.touches[1].clientX + e.touches[2].clientX) / 3;
          const curY = (e.touches[0].clientY + e.touches[1].clientY + e.touches[2].clientY) / 3;
          panX = startPanPos.x + (curX - panStart.x);
          panY = startPanPos.y + (curY - panStart.y);
          applyTransform();
        }
      }, { passive: false });

      viewport.addEventListener('touchend', (e) => {
        if (e.touches.length === 0 && isInteracting) {
          isInteracting = false;
          if (currentMode === 'pen') {
            if (currentStroke.length > 1) {
              strokes.push({
                points: [...currentStroke],
                color: penColor.value,
                size: parseInt(penSize.value, 10)
              });
            }
            ctx.closePath();
          } else if (currentMode === 'lasso') {
            if (currentStroke.length > 5) {
              const poly = [...currentStroke];
              strokes = strokes.filter(s => !s.points.some(p => pointInPoly(p, poly)));
            }
            currentStroke = [];
            redraw();
          }
        }
      });

      // Mouse support
      let isMouseDown = false;
      viewport.addEventListener('mousedown', (e) => {
        isMouseDown = true;
        const pt = toCanvas(e.clientX, e.clientY);
        currentStroke = [pt];
        if (currentMode === 'pen') {
          ctx.beginPath();
          ctx.moveTo(pt.x, pt.y);
          ctx.strokeStyle = penColor.value;
          ctx.lineWidth = parseInt(penSize.value, 10);
        }
      });

      viewport.addEventListener('mousemove', (e) => {
        if (!isMouseDown) return;
        const pt = toCanvas(e.clientX, e.clientY);
        currentStroke.push(pt);
        if (currentMode === 'pen') {
          ctx.lineTo(pt.x, pt.y);
          ctx.stroke();
        }
      });

      window.addEventListener('mouseup', () => {
        if (!isMouseDown) return;
        isMouseDown = false;
        if (currentMode === 'pen' && currentStroke.length > 1) {
          strokes.push({
            points: [...currentStroke],
            color: penColor.value,
            size: parseInt(penSize.value, 10)
          });
        }
        ctx.closePath();
      });

      styleSelect.addEventListener('change', redraw);
      resetBtn.addEventListener('click', fitToWidth);
      clearBtn.addEventListener('click', () => { strokes = []; redraw(); });

      saveImgBtn.addEventListener('click', () => {
        const link = document.createElement('a');
        link.download = 'notes.png';
        link.href = canvas.toDataURL('image/png');
        link.click();
      });

      savePdfBtn.addEventListener('click', () => {
        const { jsPDF } = window.jspdf;
        const doc = new jsPDF('p', 'pt', 'a4');
        const a4Width = 595.28;
        const a4Height = 841.89;
        const pageHeightCanvas = CANVAS_WIDTH * (a4Height / a4Width);
        const total = Math.ceil(CANVAS_HEIGHT / pageHeightCanvas);

        for (let i = 0; i < total; i++) {
          if (i > 0) doc.addPage('a4', 'p');
          const slice = document.createElement('canvas');
          slice.width = CANVAS_WIDTH;
          slice.height = pageHeightCanvas;
          const sCtx = slice.getContext('2d');
          sCtx.drawImage(canvas, 0, i * pageHeightCanvas, CANVAS_WIDTH, pageHeightCanvas, 0, 0, CANVAS_WIDTH, pageHeightCanvas);
          doc.addImage(slice.toDataURL('image/jpeg', 0.9), 'JPEG', 0, 0, a4Width, a4Height);
        }
        doc.save('notes.pdf');
      });
    });
  </script>
</body>
</html>
