
<!DOCTYPE html> 
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#0b0b0d">
<title>Aurum Signal</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#0b0b0d;color:#f7f2e8;font-family:-apple-system,BlinkMacSystemFont,Arial,sans-serif}
header{padding:22px 18px;border-bottom:1px solid #302817;display:flex;align-items:center;justify-content:space-between}
.brand{font-size:23px;font-weight:800;letter-spacing:1px;color:#e8c56c}
.brand span{display:block;font-size:10px;color:#aaa;letter-spacing:3px;margin-top:4px}
.pill{font-size:11px;color:#e8c56c;border:1px solid #59451d;padding:7px 10px;border-radius:20px}
main{max-width:650px;margin:auto;padding:18px}
.card{background:#141416;border:1px solid #302817;border-radius:17px;padding:19px;margin-bottom:15px}
.label{color:#aaa;font-size:12px;letter-spacing:1px}
.price{font-size:36px;font-weight:750;margin:12px 0 5px;overflow-wrap:anywhere}
.gold{color:#e8c56c}
.sub{color:#aaa;font-size:12px}
.row{display:flex;justify-content:space-between;gap:12px;align-items:center}
.grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.small{background:#1b1b1e;border-radius:12px;padding:14px}
.small strong{display:block;margin-top:8px;font-size:18px}
.tabs{display:flex;gap:10px;margin:14px 0}
button{font-size:14px;cursor:pointer}
.tab{flex:1;border:1px solid #40341d;background:#18181a;color:#ddd;padding:12px;border-radius:10px}
.tab.active{background:#e8c56c;color:#111;border-color:#e8c56c;font-weight:700}
.status{padding:13px;border-radius:10px;background:#211d13;color:#e8c56c;font-size:13px;line-height:1.6}
.signal{font-size:23px;font-weight:800;margin:12px 0}
.levels{display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-top:16px}
.level{background:#1b1b1e;border-radius:10px;padding:12px}
.level span{font-size:11px;color:#aaa;display:block}
.level strong{display:block;margin-top:7px;font-size:15px}
footer{text-align:center;color:#777;font-size:11px;line-height:1.7;padding:12px 10px 30px}
</style>
</head>
<body>
<header>
  <div class="brand">AURUM SIGNAL<span>GOLD TRADING</span></div>
  <div class="pill">● MARKET WATCH</div>
</header>
<main>
  <section class="card">
    <div class="row">
      <div class="label">XAU / USD · GOLD SPOT</div>
      <div class="pill" id="connection">CONNECTING</div>
    </div>
    <div class="price" id="price">Loading...</div>
    <div class="sub">USD per troy ounce</div>
    <div class="grid" style="margin-top:20px">
      <div class="small"><div class="label">PRICE / GRAM</div><strong id="gram">--</strong></div>
      <div class="small"><div class="label">DATA SOURCE</div><strong style="font-size:14px">Gold API</strong></div>
    </div>
    <p class="sub" id="updated" style="margin-top:15px">Waiting for price data...</p>
    <button class="tab" style="width:100%" onclick="loadPrice()">↻ Refresh price</button>
  </section>

  <section class="card">
    <div class="row">
      <div class="label">CHART TIMEFRAME</div>
      <div class="sub">XAU/USD</div>
    </div>
    <div class="tabs">
      <button class="tab active" id="m1" onclick="setTF('M1')">M1</button>
      <button class="tab" id="m5" onclick="setTF('M5')">M5</button>
    </div>
    <div class="status" id="chartStatus">Candle data not connected. Real candles will appear after a supported data source is configured.</div>
    <p class="sub" id="tfText">Selected timeframe: M1</p>
  </section>

  <section class="card">
    <div class="label">TRADING SIGNAL</div>
    <div class="signal">WAIT <span style="font-size:12px;color:#aaa">· NO VERIFIED SIGNAL</span></div>
    <div class="status">BUY / SELL is disabled until real M1/M5 candle data and signal rules are connected. No prediction is being generated.</div>
    <div class="levels">
      <div class="level"><span>ENTRY</span><strong>--</strong></div>
      <div class="level"><span>STOP LOSS</span><strong>--</strong></div>
      <div class="level"><span>TAKE PROFIT 1</span><strong>--</strong></div>
      <div class="level"><span>TAKE PROFIT 2</span><strong>--</strong></div>
      <div class="level"><span>TAKE PROFIT 3</span><strong>--</strong></div>
      <div class="level"><span>RISK / REWARD</span><strong>--</strong></div>
    </div>
  </section>

  <section class="card">
    <div class="label">SIGNAL ALERTS</div>
    <p class="sub">Alerts will be available after the signal engine is connected. This prototype does not send background notifications.</p>
  </section>
</main>
<footer>AURUM SIGNAL · PROTOTYPE<br>Market prices can be delayed or unavailable. Not financial advice.</footer>

<script>
let timeframe = 'M1';

function setTF(tf){
  timeframe = tf;
  document.getElementById('m1').classList.toggle('active',tf==='M1');
  document.getElementById('m5').classList.toggle('active',tf==='M5');
  document.getElementById('tfText').textContent='Selected timeframe: '+tf;
}

async function loadPrice(){
  const priceEl=document.getElementById('price');
  const statusEl=document.getElementById('connection');
  statusEl.textContent='CONNECTING';
  try{
    const response=await fetch('https://api.gold-api.com/price/XAU');
    if(!response.ok) throw new Error('Price unavailable');
    const data=await response.json();
    const price=Number(data.price);
    if(!Number.isFinite(price)||price<=0) throw new Error('Invalid price');
    priceEl.textContent='$'+price.toLocaleString('en-US',{minimumFractionDigits:2,maximumFractionDigits:2});
    document.getElementById('gram').textContent='$'+(price/31.1034768).toFixed(2);
    document.getElementById('updated').textContent='Updated: '+new Date().toLocaleString();
    statusEl.textContent='LIVE PRICE';
  }catch(error){
    statusEl.textContent='PRICE UNAVAILABLE';
    document.getElementById('updated').textContent='Could not load price. Tap Refresh to try again.';
    priceEl.textContent='--';
    document.getElementById('gram').textContent='--';
  }
}
loadPrice();
</script>
</body>
</html>
