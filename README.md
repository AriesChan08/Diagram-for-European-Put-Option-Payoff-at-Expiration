# Diagram-for-European-Put-Option-Payoff-at-Expiration
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>European Put Option Payoff Diagram</title>
<style>
  :root {
    --bg: #0b0f14;
    --panel: #121821;
    --grid: #233041;
    --axis: #9fb3c8;
    --curve: #4dd0e1;
    --loss: #ef5350;
    --profit: #66bb6a;
    --text: #e3eef9;
    --accent: #ffd166;
  }
  html, body {
    height: 100%;
    background: var(--bg);
    color: var(--text);
    font-family: system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji";
    margin: 0;
  }
  .wrap {
    max-width: 900px;
    margin: 0 auto;
    padding: 16px;
  }
  h1 {
    font-size: 1.25rem;
    margin: 0 0 8px;
  }
  p {
    margin: 6px 0 14px;
    color: #cdd9e5;
  }
  .card {
    background: var(--panel);
    border: 1px solid #1c2633;
    border-radius: 12px;
    padding: 12px;
    box-shadow: 0 4px 16px rgba(0,0,0,0.25);
  }
  .controls {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 8px;
    margin-bottom: 12px;
  }
  .control {
    display: flex;
    align-items: center;
    gap: 8px;
    background: #0f1520;
    border: 1px solid #1f2a3a;
    border-radius: 8px;
    padding: 8px;
  }
  .control label {
    font-size: 0.9rem;
    white-space: nowrap;
  }
  .control input {
    width: 100%;
    accent-color: var(--accent);
  }
  .control .val {
    min-width: 44px;
    text-align: right;
    font-variant-numeric: tabular-nums;
  }
  .canvas-wrap {
    position: relative;
    width: 100%;
    aspect-ratio: 16 / 9;
    background: linear-gradient(180deg, #0e1522, #0b0f14);
    border: 1px solid #1f2a3a;
    border-radius: 12px;
    overflow: hidden;
  }
  canvas {
    width: 100%;
    height: 100%;
    display: block;
  }
  .legend {
    display: flex;
    flex-wrap: wrap;
    gap: 8px 16px;
    margin-top: 10px;
    font-size: 0.95rem;
  }
  .key {
    display: inline-flex;
    align-items: center;
    gap: 6px;
  }
  .swatch {
    width: 14px; height: 14px; border-radius: 3px;
  }
  .sw-curve { background: var(--curve); }
  .sw-profit { background: var(--profit); }
  .sw-loss { background: var(--loss); }
  .foot {
    font-size: 0.9rem;
    color: #b9c6d6;
    margin-top: 8px;
  }
  @media (max-width: 640px) {
    .controls { grid-template-columns: 1fr; }
    h1 { font-size: 1.1rem; }
  }
</style>
</head>
<body>
  <div class="wrap">
    <h1>European Put Option Payoff at Expiration</h1>
    <p>This diagram shows the payoff for a long European put with strike K and premium. Default values: K = 50, premium = 8. Break-even = K − premium.</p>

    <div class="card">
      <div class="controls">
        <div class="control">
          <label for="strike">Strike (K)</label>
          <input id="strike" type="range" min="10" max="100" step="1" value="50" />
          <div class="val" id="strikeVal">50</div>
        </div>
        <div class="control">
          <label for="premium">Premium</label>
          <input id="premium" type="range" min="0" max="20" step="0.5" value="8" />
          <div class="val" id="premiumVal">8.0</div>
        </div>
        <div class="control">
          <label for="spotMax">Max Spot</label>
          <input id="spotMax" type="range" min="60" max="200" step="5" value="120" />
          <div class="val" id="spotMaxVal">120</div>
        </div>
      </div>

      <div class="canvas-wrap">
        <canvas id="chart"></canvas>
      </div>

      <div class="legend">
        <span class="key"><span class="swatch sw-curve"></span> Payoff = max(K − S, 0) − premium</span>
        <span class="key"><span class="swatch sw-profit"></span> Profit region</span>
        <span class="key"><span class="swatch sw-loss"></span> Loss region</span>
      </div>

      <div class="foot">Break-even at S = K − premium. At S = 0, profit ≈ K − premium. At S ≥ K, loss = premium.</div>
    </div>
  </div>

<script>
(function() {
  const canvas = document.getElementById('chart');
  const ctx = canvas.getContext('2d');

  const strikeEl = document.getElementById('strike');
  const premiumEl = document.getElementById('premium');
  const spotMaxEl = document.getElementById('spotMax');

  const strikeVal = document.getElementById('strikeVal');
  const premiumVal = document.getElementById('premiumVal');
  const spotMaxVal = document.getElementById('spotMaxVal');

  function updateVals() {
    strikeVal.textContent = Number(strikeEl.value).toFixed(0);
    premiumVal.textContent = Number(premiumEl.value).toFixed(1);
    spotMaxVal.textContent = Number(spotMaxEl.value).toFixed(0);
  }

  function dprResize() {
    const dpr = Math.max(1, Math.min(3, window.devicePixelRatio || 1));
    const rect = canvas.getBoundingClientRect();
    canvas.width = Math.round(rect.width * dpr);
    canvas.height = Math.round(rect.height * dpr);
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0); // scale drawing back to CSS pixels
  }

  function payoffPut(S, K, premium) {
    return Math.max(K - S, 0) - premium;
  }

  function draw() {
    updateVals();
    dprResize();

    const W = canvas.clientWidth;
    const H = canvas.clientHeight;

    // Margins for axes
    const m = { left: 48, right: 16, top: 16, bottom: 36 };

    const K = Number(strikeEl.value);
    const premium = Number(premiumEl.value);
    const Smax = Number(spotMaxEl.value);
    const Smin = 0;

    // Choose y-range to include min loss (-premium) and max profit (K - premium)
    const yMin = Math.min(-premium * 1.2, -1); // ensure some space below
    const yMax = Math.max(K - premium, 10); // ensure space above
    const xMin = Smin;
    const xMax = Smax;

    // Helpers
    const plotW = W - m.left - m.right;
    const plotH = H - m.top - m.bottom;
    const xScale = (x) => m.left + (x - xMin) / (xMax - xMin) * plotW;
    const yScale = (y) => m.top + (yMax - y) / (yMax - yMin) * plotH;

    // Background
    ctx.clearRect(0, 0, W, H);
    ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--panel');
    ctx.fillRect(0, 0, W, H);

    // Grid
    ctx.strokeStyle = getComputedStyle(document.documentElement).getPropertyValue('--grid');
    ctx.lineWidth = 1;
    ctx.beginPath();
    const gridXTicks = 8;
    const gridYTicks = 6;
    for (let i = 0; i <= gridXTicks; i++) {
      const x = xScale(xMin + (i / gridXTicks) * (xMax - xMin));
      ctx.moveTo(x, m.top); ctx.lineTo(x, H - m.bottom);
    }
    for (let j = 0; j <= gridYTicks; j++) {
      const y = yScale(yMin + (j / gridYTicks) * (yMax - yMin));
      ctx.moveTo(m.left, y); ctx.lineTo(W - m.right, y);
    }
    ctx.stroke();

    // Axes
    const axisColor = getComputedStyle(document.documentElement).getPropertyValue('--axis');
    ctx.strokeStyle = axisColor;
    ctx.lineWidth = 1.5;
    // X-axis at y=0
    const xAxisY = yScale(0);
    ctx.beginPath();
    ctx.moveTo(m.left, xAxisY); ctx.lineTo(W - m.right, xAxisY);
    ctx.stroke();
    // Y-axis at S=0
    ctx.beginPath();
    ctx.moveTo(m.left, m.top); ctx.lineTo(m.left, H - m.bottom);
    ctx.stroke();

    // Labels
    ctx.fillStyle = axisColor;
    ctx.font = '12px system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'top';
    // X ticks and labels
    const xTicks = 8;
    for (let i = 0; i <= xTicks; i++) {
      const S = xMin + (i / xTicks) * (xMax - xMin);
      const x = xScale(S);
      ctx.fillText(S.toFixed(0), x, H - m.bottom + 6);
    }
    // Y ticks and labels
    ctx.textAlign = 'right';
    ctx.textBaseline = 'middle';
    const yTicks = 6;
    for (let j = 0; j <= yTicks; j++) {
      const yVal = yMin + (j / yTicks) * (yMax - yMin);
      const y = yScale(yVal);
      ctx.fillText(yVal.toFixed(0), m.left - 6, y);
    }
    // Axis titles
    ctx.save();
    ctx.fillStyle = '#cfd9e8';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'bottom';
    ctx.fillText('Underlying Price S', (m.left + W - m.right) / 2, H - 4);
    ctx.translate(12, (m.top + H - m.bottom) / 2);
    ctx.rotate(-Math.PI / 2);
    ctx.fillText('Profit / Loss at Expiration', 0, 0);
    ctx.restore();

    // Break-even
    const breakeven = K - premium;
    const beX = xScale(Math.max(xMin, Math.min(xMax, breakeven)));
    ctx.strokeStyle = 'rgba(255, 209, 102, 0.7)';
    ctx.setLineDash([6, 6]);
    ctx.beginPath();
    ctx.moveTo(beX, m.top);
    ctx.lineTo(beX, H - m.bottom);
    ctx.stroke();
    ctx.setLineDash([]);
    ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--accent');
    ctx.textAlign = 'center';
    ctx.textBaseline = 'bottom';
    if (breakeven >= xMin && breakeven <= xMax) {
      ctx.fillText('Break-even S = ' + breakeven.toFixed(2), beX, xAxisY - 6);
    }

    // Strike line
    const kX = xScale(K);
    ctx.strokeStyle = 'rgba(159, 179, 200, 0.6)';
    ctx.setLineDash([4, 6]);
    ctx.beginPath();
    ctx.moveTo(kX, m.top);
    ctx.lineTo(kX, H - m.bottom);
    ctx.stroke();
    ctx.setLineDash([]);
    ctx.fillStyle = '#b7c6d9';
    ctx.textAlign = 'left';
    ctx.textBaseline = 'top';
    if (K >= xMin && K <= xMax) {
      ctx.fillText('Strike K = ' + K.toFixed(0), kX + 6, m.top + 6);
    }

    // Premium loss line at y = -premium
    const premY = yScale(-premium);
    ctx.strokeStyle = 'rgba(239, 83, 80, 0.6)';
    ctx.setLineDash([4, 6]);
    ctx.beginPath();
    ctx.moveTo(m.left, premY);
    ctx.lineTo(W - m.right, premY);
    ctx.stroke();
    ctx.setLineDash([]);
    ctx.fillStyle = '#ef9a9a';
    ctx.textAlign = 'right';
    ctx.textBaseline = 'bottom';
    ctx.fillText('Max loss = premium = ' + premium.toFixed(2), W - m.right - 6, premY - 4);

    // Payoff curve points
    const pts = [];
    const N = Math.max(200, Math.floor(plotW)); // smooth
    for (let i = 0; i <= N; i++) {
      const S = xMin + (i / N) * (xMax - xMin);
      const y = payoffPut(S, K, premium);
      pts.push({ x: xScale(S), y: yScale(y), S, yVal: y });
    }

    // Fill profit/loss regions under curve relative to y=0
    // Profit region where payoff > 0 and S < break-even
    ctx.beginPath();
    let started = false;
    for (let i = 0; i < pts.length; i++) {
      const p = pts[i];
      if (p.yVal > 0) {
        if (!started) {
          ctx.moveTo(p.x, yScale(0));
          started = true;
        }
        ctx.lineTo(p.x, p.y);
      } else if (started) {
        ctx.lineTo(p.x, yScale(0));
        ctx.closePath();
        break;
      }
    }
    ctx.fillStyle = 'rgba(102, 187, 106, 0.25)';
    ctx.fill();

    // Loss region where payoff < 0
    ctx.beginPath();
    // Start from left at S = 0 up to rightmost
    ctx.moveTo(pts[0].x, yScale(0));
    for (let i = 0; i < pts.length; i++) {
      const p = pts[i];
      const y0 = Math.min(p.y, yScale(0));
      ctx.lineTo(p.x, y0);
    }
    ctx.lineTo(pts[pts.length - 1].x, yScale(0));
    ctx.closePath();
    ctx.fillStyle = 'rgba(239, 83, 80, 0.18)';
    ctx.fill();

    // Payoff curve
    ctx.strokeStyle = getComputedStyle(document.documentElement).getPropertyValue('--curve');
    ctx.lineWidth = 2.5;
    ctx.beginPath();
    for (let i = 0; i < pts.length; i++) {
      const p = pts[i];
      if (i === 0) ctx.moveTo(p.x, p.y);
      else ctx.lineTo(p.x, p.y);
    }
    ctx.stroke();

    // Markers at notable points: (S=0, profit=K-premium), (S=K, -premium)
    const yAt0 = yScale(K - premium);
    const yAtK = yScale(-premium);
    const r = 4;
    // S=0
    ctx.fillStyle = '#ffffff';
    ctx.beginPath(); ctx.arc(xScale(0), yAt0, r, 0, Math.PI*2); ctx.fill();
    ctx.fillStyle = '#cfd9e8';
    ctx.textAlign = 'left'; ctx.textBaseline = 'bottom';
    ctx.fillText(`S=0, Profit=${(K - premium).toFixed(2)}`, xScale(0) + 6, yAt0 - 6);
    // S=K
    ctx.fillStyle = '#ffffff';
    ctx.beginPath(); ctx.arc(kX, yAtK, r, 0, Math.PI*2); ctx.fill();
    ctx.fillStyle = '#cfd9e8';
    ctx.textAlign = 'left'; ctx.textBaseline = 'top';
    ctx.fillText(`S=K, Profit=${(-premium).toFixed(2)}`, kX + 6, yAtK + 6);

    // Tooltip interaction
    setupTooltip(pts, xScale, yScale, xMin, xMax);
  }

  // Tooltip
  let tipEl;
  function setupTooltip(pts, xScale, yScale, xMin, xMax) {
    if (!tipEl) {
      tipEl = document.createElement('div');
      tipEl.style.position = 'absolute';
      tipEl.style.pointerEvents = 'none';
      tipEl.style.background = 'rgba(10,16,24,0.9)';
      tipEl.style.border = '1px solid #2a3a52';
      tipEl.style.color = '#e6eef7';
      tipEl.style.padding = '6px 8px';
      tipEl.style.borderRadius = '8px';
      tipEl.style.fontSize = '12px';
      tipEl.style.whiteSpace = 'nowrap';
      tipEl.style.transform = 'translate(-50%, -120%)';
      tipEl.style.display = 'none';
      canvas.parentElement.appendChild(tipEl);

      canvas.addEventListener('mouseleave', () => tipEl.style.display = 'none');
      canvas.addEventListener('mousemove', (e) => {
        const rect = canvas.getBoundingClientRect();
        const x = e.clientX - rect.left;
        const W = canvas.clientWidth;
        const H = canvas.clientHeight;
        const m = { left: 48, right: 16, top: 16, bottom: 36 };
        const plotW = W - m.left - m.right;
        const plotH = H - m.top - m.bottom;

        // Invert x to S
        const S = xMin + ((x - m.left) / plotW) * (xMax - xMin);
        if (S < xMin || S > xMax) { tipEl.style.display = 'none'; return; }

        // Find nearest point
        let nearest = pts[0];
        let best = Infinity;
        for (const p of pts) {
          const dx = Math.abs(p.x - x);
          if (dx < best) { best = dx; nearest = p; }
        }

        tipEl.style.display = 'block';
        tipEl.textContent = `S=${nearest.S.toFixed(2)} | Payoff=${nearest.yVal.toFixed(2)}`;
        tipEl.style.left = nearest.x + 'px';
        tipEl.style.top = nearest.y + 'px';
      });
    }
  }

  // Wire controls
  [strikeEl, premiumEl, spotMaxEl].forEach(el => el.addEventListener('input', draw));
  window.addEventListener('resize', draw);

  // Initial draw
  draw();
})();
</script>
</body>
</html>
