
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Aurum Signal</title>
<style>
body{margin:0;background:#0b1020;color:#f5f5f5;font-family:Arial,sans-serif;padding:22px}
.card{background:#171f33;padding:20px;border-radius:16px;margin:16px 0}
.gold{color:#f5c451}
button{background:#f5c451;border:0;border-radius:10px;padding:12px 18px;font-weight:bold}
.small{color:#aab4c8;font-size:14px}
</style>
</head>
<body>
<h1 class="gold">✦ AURUM SIGNAL</h1>
<p>XAU/USD · Gold spot monitor</p>
<div class="card">
<p class="small">GOLD SPOT PRICE · USD / TROY OUNCE</p>
<h2 id="price">Loading...</h2>
<p id="status" class="small">Connecting to gold price source...</p>
<button onclick="loadPrice()">Refresh price</button>
<p>Price per gram: <b id="gram">—</b> USD</p>
<p class="small">Updated: <span id="time">—</span></p>
</div>
<div class="card">
<h2>Trading signals</h2>
<p>BUY / SELL: <b>NOT AVAILABLE</b></p>
<p class="small">Real 1-minute and 5-minute candle data is required before reliable signals can be generated. No simulated signals are shown.</p>
</div>
<p class="small">Educational monitor only. Spot prices may differ from your broker's MT5 price.</p>
<script>
async function loadPrice(){
 const status=document.getElementById('status');
 status.textContent='Loading price...';
 try{
  const r=await fetch('https://api.gold-api.com/price/XAU');
  if(!r.ok)throw Error('Price source unavailable');
  const d=await r.json();
  const p=Number(d.price);
  if(!Number.isFinite(p)||p<=0)throw Error('Invalid price');
  document.getElementById('price').textContent='$'+p.toLocaleString('en-US',{minimumFractionDigits:2,maximumFractionDigits:2});
  document.getElementById('gram').textContent=(p/31.1034768).toFixed(2);
  document.getElementById('time').textContent=new Date().toLocaleString();
  status.textContent='Live price received';
 }catch(e){
  status.textContent='Price unavailable. Try again later.';
  document.getElementById('price').textContent='—';
  document.getElementById('gram').textContent='—';
 }
}
loadPrice();
</script>
</body>
</html>

