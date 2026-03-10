<!DOCTYPE html>
<html lang="cs">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>WiFi QR Generátor</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: 'Segoe UI', sans-serif;
    background: #f0f4ff;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }
  .card {
    background: #fff;
    border-radius: 24px;
    box-shadow: 0 8px 40px rgba(80,100,200,0.13);
    padding: 40px 36px;
    width: 100%;
    max-width: 420px;
  }
  .header {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 28px;
  }
  .icon {
    width: 44px; height: 44px;
    background: linear-gradient(135deg, #6c8ef7, #a78bfa);
    border-radius: 12px;
    display: flex; align-items: center; justify-content: center;
  }
  .icon svg { width: 24px; height: 24px; fill: white; }
  h1 { font-size: 1.3rem; font-weight: 700; color: #1e1e2e; }
  p.sub { font-size: 0.82rem; color: #888; margin-top: 2px; }

  label { display: block; font-size: 0.82rem; font-weight: 600; color: #555; margin-bottom: 6px; margin-top: 16px; }
  input, select {
    width: 100%;
    padding: 11px 14px;
    border: 1.5px solid #e2e8f0;
    border-radius: 10px;
    font-size: 0.95rem;
    color: #1e1e2e;
    background: #fafbff;
    outline: none;
    transition: border 0.2s;
  }
  input:focus, select:focus { border-color: #6c8ef7; background: #fff; }

  .pw-wrap { position: relative; }
  .pw-wrap input { padding-right: 42px; }
  .toggle-pw {
    position: absolute; right: 12px; top: 50%; transform: translateY(-50%);
    background: none; border: none; cursor: pointer; color: #aaa; padding: 0;
    display: flex; align-items: center;
  }
  .toggle-pw:hover { color: #6c8ef7; }

  .btn {
    margin-top: 22px;
    width: 100%;
    padding: 13px;
    background: linear-gradient(135deg, #6c8ef7, #a78bfa);
    color: white;
    font-size: 1rem;
    font-weight: 600;
    border: none;
    border-radius: 12px;
    cursor: pointer;
    transition: opacity 0.2s, transform 0.1s;
  }
  .btn:hover { opacity: 0.92; transform: translateY(-1px); }
  .btn:active { transform: translateY(0); }

  .qr-section {
    display: none;
    flex-direction: column;
    align-items: center;
    margin-top: 28px;
    padding-top: 24px;
    border-top: 1.5px solid #f0f0f0;
  }
  .qr-section.show { display: flex; }
  .qr-box {
    background: #fff;
    border: 2px solid #e8eaff;
    border-radius: 16px;
    padding: 16px;
    box-shadow: 0 2px 12px rgba(108,142,247,0.1);
  }
  .qr-label {
    margin-top: 14px;
    font-size: 0.85rem;
    color: #888;
    text-align: center;
  }
  .qr-label strong { color: #1e1e2e; }

  .dl-btn {
    margin-top: 16px;
    padding: 10px 28px;
    background: #fff;
    color: #6c8ef7;
    font-size: 0.9rem;
    font-weight: 600;
    border: 2px solid #6c8ef7;
    border-radius: 10px;
    cursor: pointer;
    transition: background 0.2s, color 0.2s;
    display: flex; align-items: center; gap: 7px;
  }
  .dl-btn:hover { background: #6c8ef7; color: #fff; }
</style>
</head>
<body>
<div class="card">
  <div class="header">
    <div class="icon">
      <svg viewBox="0 0 24 24"><path d="M1 7.5C4.15 4.35 8.35 2.5 12 2.5s7.85 1.85 11 5l-2 2c-2.5-2.5-5.9-4-9-4s-6.5 1.5-9 4L1 7.5zm4 4C6.9 9.6 9.35 8.5 12 8.5s5.1 1.1 7 3l-2 2c-1.35-1.35-3.1-2-5-2s-3.65.65-5 2l-2-2zm4 4c1-.9 2.3-1.5 3-1.5s2 .6 3 1.5L12 19l-3-3.5z"/></svg>
    </div>
    <div>
      <h1>WiFi QR Generátor</h1>
      <p class="sub">Vygeneruj QR kód pro sdílení WiFi</p>
    </div>
  </div>

  <label for="ssid">Název sítě (SSID)</label>
  <input id="ssid" type="text" placeholder="např. MojeWiFi">

  <label for="security">Typ zabezpečení</label>
  <select id="security">
    <option value="WPA">WPA / WPA2 / WPA3</option>
    <option value="WEP">WEP</option>
    <option value="">Bez hesla (otevřená síť)</option>
  </select>

  <label for="password">Heslo</label>
  <div class="pw-wrap">
    <input id="password" type="password" placeholder="Zadej heslo">
    <button class="toggle-pw" onclick="togglePw()" title="Zobrazit/skrýt heslo">
      <svg id="eye-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/>
      </svg>
    </button>
  </div>

  <button class="btn" onclick="generate()">Generovat QR kód</button>

  <div class="qr-section" id="qrSection">
    <div class="qr-box" id="qrcode"></div>
    <div class="qr-label">Síť: <strong id="qrLabel"></strong></div>
    <button class="dl-btn" onclick="download()">
      <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
      Stáhnout PNG
    </button>
  </div>
</div>

<script>
  let qr = null;

  function togglePw() {
    const inp = document.getElementById('password');
    inp.type = inp.type === 'password' ? 'text' : 'password';
  }

  function escapeWifi(str) {
    return str.replace(/\\/g,'\\\\').replace(/"/g,'\\"').replace(/;/g,'\\;').replace(/,/g,'\\,').replace(/:/g,'\\:');
  }

  function generate() {
    const ssid = document.getElementById('ssid').value.trim();
    const pw = document.getElementById('password').value;
    const sec = document.getElementById('security').value;

    if (!ssid) { alert('Zadej prosím název sítě (SSID).'); return; }
    if (sec && !pw) { alert('Zadej prosím heslo.'); return; }

    const wifiStr = `WIFI:T:${sec};S:${escapeWifi(ssid)};P:${escapeWifi(pw)};;`;

    const box = document.getElementById('qrcode');
    box.innerHTML = '';
    if (qr) { qr = null; }

    qr = new QRCode(box, {
      text: wifiStr,
      width: 200,
      height: 200,
      colorDark: '#1e1e2e',
      colorLight: '#ffffff',
      correctLevel: QRCode.CorrectLevel.M
    });

    document.getElementById('qrLabel').textContent = ssid;
    document.getElementById('qrSection').classList.add('show');
  }

  function download() {
    const canvas = document.querySelector('#qrcode canvas');
    if (!canvas) { alert('Nejprve vygeneruj QR kód.'); return; }
    const ssid = document.getElementById('ssid').value.trim() || 'wifi-qr';
    const a = document.createElement('a');
    a.href = canvas.toDataURL('image/png');
    a.download = `${ssid}-qr.png`;
    a.click();
  }
</script>
</body>
</html>
