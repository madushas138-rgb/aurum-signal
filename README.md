
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>AURUM SIGNAL — Gold Trading</title>
<script src="https://unpkg.com/lightweight-charts@4.2.2/dist/lightweight-charts.standalone.production.js"></script>
<style>
:root{color-scheme:dark;--gold:#e8bd59;--bg:#090b10;--panel:#131720;--muted:#929bad}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:#f5f6fa;font:14px Arial,sans-serif}
main{max-width:850px;margin:auto;padding:18px}
header{border-bottom:1px solid #292d37;padding:8px 0 18px;margin-bottom:16px}
.brand{color:var(--gold);font-size:24px;font-weight:800;letter-spacing:2px}
.sub{color:var(--muted);font-size:11px;letter-spacing:2px;margin-top:6px}
.panel{background:var(--panel);border:1px solid #292e39;border-radius:14px;padding:16px;margin:12px 0}
.label{font-size:11px;letter-spacing:1px;color:var(--muted)}
.price{font-size:34px;font-weight:750;margin:8px 0}
.row{display:flex;justify-content:space-between;align-items:center;gap:10px;flex-wrap:wrap}
button{background:#202633;color:#fff;border:1px solid #394152;border-radius:9px;padding:11px 16px;font-weight:700;cursor:pointer}
button.active{background:var(--gold);color:#17130a;border-color:var(--gold)}
button:disabled{opacity:.5}
#chart{height:300px;width:100%;margin-top:16px}
.status{font-size:19px;font-weight:800;margin:12px 0}
.buy{color:#42d6a0}.sell{color:#ff6b76}.wait{color:var(--gold)}
.grid{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px;margin-top:16px}
.cell{background:#0c1017;padding:12px;border-radius:9px;min-width:0}
.val{font-size:16px;font-weight:700;margin-top:8px;overflow-wrap:anywhere}
.note{font-size:12px;line-height:1.6;color:var(--muted)}
button:focus-visible{outline:2px solid var(--gold)}
@media(max-width:480px){main{padding:12px}.grid{grid-template-columns:repeat(2,minmax(0,1fr))}.price{font-size:29px}#chart{height:260px}}
</style>
</head>
<body>
<main>
<header>
<div class="brand">AURUM SIGNAL</div>
<div class="sub">GOLD TRADING · XAU / USD</div>
</header>

<section class="panel">
<div class="row">
<div>
<div class="label">LATEST CANDLE CLOSE</div>
<div class="price" id="price">Loading...</div>
<div class="note" id="updated">Connecting to market data...</div>
</div>
<button id="refresh">↻ Refresh</button>
</div>
</section>

<section class="panel">
<div class="row">
<div class="label">XAU/USD CANDLE CHART</div>
<div class="row">
<button class="tf active" data-tf="5min">M5</button>
<button class="tf" data-tf="1min">M1</button>
</div>
</div>
<div id="chart"></div>
<div class="note" id="chartStatus">Waiting for candle data...</div>
</section>

<section class="panel">
<div class="label">TRADING SIGNAL · RULE-BASED</div>
<div class="status wait" id="signal">WAIT</div>
<div class="note" id="reason">Waiting for candle data.</div>
<div class="grid">
<div class="cell"><div class="label">ENTRY</div><div class="val" id="entry">--</div></div>
<div class="cell"><div class="label">STOP LOSS</div><div class="val" id="sl">--</div></div>
<div class="cell"><div class="label">TAKE PROFIT 1</div><div class="val" id="tp1">--</div></div>
<div class="cell"><div class="label">TAKE PROFIT 2</div><div class="val" id="tp2">--</div></div>
<div class="cell"><div class="label">TAKE PROFIT 3</div><div class="val" id="tp3">--</div></div>
<div class="cell"><div class="label">RISK / REWARD</div><div class="val" id="rr">1:1 · 1:2 · 1:3</div></div>
</div>
</section>

<section class="panel">
<div class="label">DATA & ALERTS</div>
<p class="note" id="dataStatus">Data source: Twelve Data XAU/USD via secure Worker.</p>
<p class="note">SL uses 2.5 × ATR. TP1/TP2/TP3 use 1R/2R/3R. Signals are technical rule outputs, not guaranteed predictions. Verify prices and risk before any trade. This page does not send background or push notifications.</p>
</section>
</main>

<script>
const API="https://aurum-candles-api.madushas138.workers.dev/";
let timeframe="5min",chart=null,candles=null,bars=[];
const $=id=>document.getElementById(id);
const fmt=n=>Number(n).toFixed(2);

function initChart(){
  if(!window.LightweightCharts){
    $("chartStatus").textContent="Chart library failed to load. Check internet connection.";
    return;
  }
  chart=LightweightCharts.createChart($("chart"),{
    width:$("chart").clientWidth,
    height:$("chart").clientHeight,
    layout:{background:{color:"#131720"},textColor:"#aab2c0"},
    grid:{vertLines:{color:"#222733"},horzLines:{color:"#222733"}},
    rightPriceScale:{borderColor:"#303644"},
    timeScale:{borderColor:"#303644",timeVisible:true,secondsVisible:false},
    localization:{priceFormatter:p=>p.toFixed(2)}
  });
  candles=chart.addCandlestickSeries({
    upColor:"#25c99a",downColor:"#ff6675",
    borderUpColor:"#25c99a",borderDownColor:"#ff6675",
    wickUpColor:"#25c99a",wickDownColor:"#ff6675"
  });
  window.addEventListener("resize",()=>{
    if(chart)chart.applyOptions({width:$("chart").clientWidth});
  });
}

function ema(data,period){
  const k=2/(period+1);
  let v=data[0].close;
  return data.map((b,i)=>{
    v=i===0?b.close:b.close*k+v*(1-k);
    return v;
  });
}

function rsi(data,period=14){
  if(data.length<period+1)return 50;
  let gains=0,losses=0;
  for(let i=data.length-period;i<data.length;i++){
    const d=data[i].close-data[i-1].close;
    if(d>0)gains+=d;
    else losses-=d;
  }
  if(losses===0)return 100;
  return 100-(100/(1+gains/losses));
}

function atr(data,period=14){
  if(data.length<period+1)return 0;
  let sum=0;
  for(let i=data.length-period;i<data.length;i++){
    const prev=data[i-1].close;
    sum+=Math.max(data[i].high-data[i].low,
      Math.abs(data[i].high-prev),
      Math.abs(data[i].low-prev));
  }
  return sum/period;
}

function setText(id,value){
  $(id).textContent=value;
}

function resetSignal(message){
  setText("signal","WAIT");
  $("signal").className="status wait";
  setText("reason",message);
  ["entry","sl","tp1","tp2","tp3"].forEach(id=>setText(id,"--"));
  setText("rr","1:1 · 1:2 · 1:3");
}

function makeSignal(data){
  resetSignal("No clear setup under the current EMA/RSI rules.");
  if(data.length<30)return;

  const e9=ema(data,9);
  const e21=ema(data,21);
  const last=data[data.length-1];
  const fast=e9[e9.length-1];
  const slow=e21[e21.length-1];
  const momentum=rsi(data);
  const volatility=atr(data);

  if(!volatility||!Number.isFinite(volatility))return;

  let side="";
  if(fast>slow&&last.close>fast&&momentum>=52&&momentum<=72){
    side="BUY";
  }else if(fast<slow&&last.close<fast&&momentum<=48&&momentum>=28){
    side="SELL";
  }else{
    return;
  }

  const entry=last.close;

  // WIDER STOP LOSS: 2.5 x ATR
  const risk=volatility*2.5;
  const sign=side==="BUY"?1:-1;

  setText("signal",side);
  $("signal").className="status "+(side==="BUY"?"buy":"sell");
  setText("reason",
    `EMA 9/21 trend and RSI ${momentum.toFixed(1)} agree. ATR-based SL = 2.5×ATR. Candle close: ${new Date(last.time*1000).toLocaleString()}. Rule-based setup; not a guarantee.`
  );

  setText("entry",fmt(entry));
  setText("sl",fmt(entry-sign*risk));
  setText("tp1",fmt(entry+sign*risk));
  setText("tp2",fmt(entry+sign*risk*2));
  setText("tp3",fmt(entry+sign*risk*3));
  setText("rr","1:1 · 1:2 · 1:3");
}

async function loadData(){
  $("refresh").disabled=true;
  setText("chartStatus",
    `Loading ${timeframe==="1min"?"M1":"M5"} candles...`
  );

  try{
    const response=await fetch(API+"?interval="+timeframe,{cache:"no-store"});
    const data=await response.json();

    if(!response.ok||data.status==="error"||!Array.isArray(data.values)){
      throw new Error(data.error||data.message||"Candle data unavailable");
    }

    bars=data.values.map(v=>({
      time:Math.floor(Date.parse(v.datetime.replace(" ","T")+"Z")/1000),
      open:Number(v.open),
      high:Number(v.high),
      low:Number(v.low),
      close:Number(v.close)
    })).filter(b=>
      Number.isFinite(b.time)&&Number.isFinite(b.open)&&
      Number.isFinite(b.high)&&Number.isFinite(b.low)&&
      Number.isFinite(b.close)
    ).sort((a,b)=>a.time-b.time);

    if(bars.length<25)throw new Error("Not enough candles for analysis");
    if(!candles||!chart)throw new Error("Chart library is not ready. Refresh the page.");

    candles.setData(bars);
    chart.timeScale().fitContent();

    setText("price","$"+fmt(bars[bars.length-1].close));
    setText("updated",
      "Last candle: "+new Date(bars[bars.length-1].time*1000).toLocaleString()
    );
    setText("chartStatus",
      `${bars.length} candles loaded · ${timeframe.toUpperCase()}`
    );
    setText("dataStatus","Data source: Twelve Data XAU/USD via Cloudflare Worker.");

    
const candleSeconds = timeframe==="1min" ? 60 : 300;
const now = Math.floor(Date.now()/1000);

const closedBars = bars.filter(
  b => b.time + candleSeconds <= now
);

makeSignal(closedBars);

  }catch(e){
    resetSignal("Live data failed. No signal is shown.");
    setText("chartStatus","Data error: "+e.message);
    setText("dataStatus","Check Worker deployment, API quota and internet connection.");
  }finally{
    $("refresh").disabled=false;
  }
}

document.querySelectorAll(".tf").forEach(btn=>{
  btn.addEventListener("click",()=>{
    timeframe=btn.dataset.tf;
    document.querySelectorAll(".tf").forEach(b=>
      b.classList.toggle("active",b===btn)
    );
    loadData();
  });
});

$("refresh").addEventListener("click",loadData);
initChart();
loadData();
</script>
</body>
</html>
