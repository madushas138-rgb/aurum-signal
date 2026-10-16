
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
main{max-width:900px;margin:auto;padding:16px}
header{border-bottom:1px solid #292d37;padding:8px 0 18px;margin-bottom:16px}
.brand{color:var(--gold);font-size:24px;font-weight:800;letter-spacing:2px}
.sub{color:var(--muted);font-size:11px;letter-spacing:2px;margin-top:6px}
.panel{background:var(--panel);border:1px solid #292e39;border-radius:14px;padding:16px;margin:12px 0}
.label{font-size:11px;letter-spacing:1px;color:var(--muted)}
.price{font-size:32px;font-weight:750;margin:8px 0}
.row{display:flex;justify-content:space-between;align-items:center;gap:10px;flex-wrap:wrap}
button{background:#202633;color:#fff;border:1px solid #394152;border-radius:9px;padding:11px 14px;font-weight:700;cursor:pointer}
button.active{background:var(--gold);color:#17130a;border-color:var(--gold)}
button:disabled{opacity:.5}
#chart{height:280px;width:100%;margin-top:16px}
.status{font-size:21px;font-weight:800;margin:12px 0}
.buy,.win{color:#42d6a0}.sell,.loss{color:#ff6b76}.wait{color:var(--gold)}
.grid{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:9px;margin-top:15px}
.cell{background:#0c1017;padding:12px;border-radius:9px;min-width:0}
.val{font-size:16px;font-weight:700;margin-top:8px;overflow-wrap:anywhere}
.note{font-size:12px;line-height:1.6;color:var(--muted)}
.stats{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:8px;margin:12px 0}
.stat{background:#0c1017;border-radius:9px;padding:12px}
.stat strong{display:block;font-size:20px;margin-top:7px}
.tablewrap{overflow-x:auto}
table{width:100%;border-collapse:collapse;font-size:12px;min-width:650px}
th,td{text-align:left;padding:11px 8px;border-bottom:1px solid #292e39;white-space:nowrap}
th{color:var(--muted);font-weight:600}
.tag{font-weight:700}
.empty{padding:20px 0;color:var(--muted);font-size:13px}
@media(max-width:520px){
 main{padding:10px}
 .grid{grid-template-columns:repeat(2,minmax(0,1fr))}
 .stats{grid-template-columns:repeat(2,minmax(0,1fr))}
 .price{font-size:28px}
 #chart{height:250px}
}
</style>
</head>
<body>
<main>
<header>
 <div class="brand">AURUM SIGNAL</div>
 <div class="sub">XAU/USD · GOLD SIGNAL & TRADE TRACKER</div>
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
 <div class="note" id="chartStatus">Waiting for candles...</div>
</section>

<section class="panel">
 <div class="label">LATEST SIGNAL</div>
 <div class="status wait" id="signal">WAIT</div>
 <div class="note" id="reason">Waiting for confirmed candles.</div>
 <div class="grid">
  <div class="cell"><div class="label">ENTRY</div><div class="val" id="entry">--</div></div>
  <div class="cell"><div class="label">STOP LOSS</div><div class="val" id="sl">--</div></div>
  <div class="cell"><div class="label">TAKE PROFIT 1</div><div class="val" id="tp1">--</div></div>
  <div class="cell"><div class="label">TAKE PROFIT 2</div><div class="val" id="tp2">--</div></div>
  <div class="cell"><div class="label">TAKE PROFIT 3</div><div class="val" id="tp3">--</div></div>
  <div class="cell"><div class="label">RISK / REWARD</div><div class="val">1:1 · 1:2 · 1:3</div></div>
 </div>
 <p class="note">SL = 2.5 × ATR. Signals use EMA 9/21 and RSI. A signal is not a guarantee of profit.</p>
</section>

<section class="panel">
 <div class="row">
  <div>
   <div class="label">TRADE RESULTS</div>
   <h3 style="margin:7px 0">Signal History</h3>
  </div>
  <button id="clearHistory">Clear History</button>
 </div>

 <div class="stats">
  <div class="stat"><div class="label">TOTAL TRADES</div><strong id="totalCount">0</strong></div>
  <div class="stat"><div class="label">OPEN</div><strong id="openCount">0</strong></div>
  <div class="stat"><div class="label">TP1 HIT</div><strong id="tp1Count" class="win">0</strong></div>
  <div class="stat"><div class="label">SL HIT</div><strong id="slCount" class="loss">0</strong></div>
 </div>

 <div class="note" id="historyStatus">Trade records are saved in this browser on this device.</div>
 <div class="tablewrap">
  <table>
   <thead>
    <tr>
     <th>DATE / TIME</th>
     <th>SIDE</th>
     <th>ENTRY</th>
     <th>SL</th>
     <th>TP1</th>
     <th>TP2</th>
     <th>TP3</th>
     <th>RESULT</th>
    </tr>
   </thead>
   <tbody id="historyBody"></tbody>
  </table>
 </div>
 <div class="empty" id="emptyHistory">No tracked trades yet. Wait for a confirmed signal.</div>
</section>

<section class="panel">
 <div class="label">DATA & TRACKING NOTES</div>
 <p class="note" id="dataStatus">Twelve Data XAU/USD via Cloudflare Worker.</p>
 <p class="note">This tracker records simulated signals, not actual MT5 trades. Results are based on candle high/low prices. If SL and TP are both reached in one candle, SL is counted first as a conservative assumption.</p>
 <p class="note">The page must be open to refresh automatically. When reopened, it checks available candles for missed outcomes. Browser history can be lost if site data is cleared or a different browser/device is used.</p>
</section>
</main>

<script>
const API="https://aurum-candles-api.madushas138.workers.dev/";
const STORAGE_KEY="aurum_signal_trade_history_v2";
let timeframe="5min",chart=null,candles=null,bars=[];
let trades=loadTrades();
let lastDataKey="";

const $=id=>document.getElementById(id);
const fmt=n=>Number(n).toFixed(2);
const timeText=t=>new Date(t*1000).toLocaleString();

function loadTrades(){
 try{
  const x=JSON.parse(localStorage.getItem(STORAGE_KEY)||"[]");
  return Array.isArray(x)?x:[];
 }catch(e){return [];}
}

function saveTrades(){
 try{
  trades=trades.slice(0,300);
  localStorage.setItem(STORAGE_KEY,JSON.stringify(trades));
 }catch(e){
  $("historyStatus").textContent="Could not save browser history. Check browser storage settings.";
 }
}

function initChart(){
 if(!window.LightweightCharts){
  $("chartStatus").textContent="Chart library failed to load.";
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

function resetSignal(message){
 $("signal").textContent="WAIT";
 $("signal").className="status wait";
 $("reason").textContent=message;
 ["entry","sl","tp1","tp2","tp3"].forEach(id=>$(id).textContent="--");
}

function makeSignal(data){
 resetSignal("No confirmed setup at the latest closed candle.");
 if(data.length<32)return null;

 const e9=ema(data,9),e21=ema(data,21);
 const i=data.length-1;
 const last=data[i],prev=data[i-1];
 const fast=e9[i],slow=e21[i];
 const prevFast=e9[i-1],prevSlow=e21[i-1];
 const momentum=rsi(data);
 const volatility=atr(data);

 if(!volatility||!Number.isFinite(volatility))return null;

 const buy=fast>slow&&prevFast>prevSlow&&
  last.close>fast&&prev.close>prevFast&&
  momentum>=52&&momentum<=72;

 const sell=fast<slow&&prevFast<prevSlow&&
  last.close<fast&&prev.close<prevFast&&
  momentum<=48&&momentum>=28;

 const side=buy?"BUY":sell?"SELL":"";
 if(!side)return null;

 const entry=last.close;
 const risk=volatility*2.5;
 const sign=side==="BUY"?1:-1;
 const trade={
  id:side+"-"+last.time,
  side:side,
  entryTime:last.time,
  entry:entry,
  sl:entry-sign*risk,
  tp1:entry+sign*risk,
  tp2:entry+sign*risk*2,
  tp3:entry+sign*risk*3,
  hitTP1:false,
  hitTP2:false,
  hitTP3:false,
  status:"OPEN",
  lastProcessedTime:last.time,
  closeTime:null
 };

 $("signal").textContent=side;
 $("signal").className="status "+(side==="BUY"?"buy":"sell");
 $("reason").textContent=
  "Confirmed closed-candle setup · RSI "+momentum.toFixed(1)+
  " · SL distance 2.5 × ATR · "+timeText(last.time);

 $("entry").textContent=fmt(trade.entry);
 $("sl").textContent=fmt(trade.sl);
 $("tp1").textContent=fmt(trade.tp1);
 $("tp2").textContent=fmt(trade.tp2);
 $("tp3").textContent=fmt(trade.tp3);

 return trade;
}

function checkTrade(trade,bar){
 if(trade.status!=="OPEN"||bar.time<=trade.entryTime)return;

 const isBuy=trade.side==="BUY";
 const slHit=isBuy?bar.low<=trade.sl:bar.high>=trade.sl;
 const tp1Hit=isBuy?bar.high>=trade.tp1:bar.low<=trade.tp1;
 const tp2Hit=isBuy?bar.high>=trade.tp2:bar.low<=trade.tp2;
 const tp3Hit=isBuy?bar.high>=trade.tp3:bar.low<=trade.tp3;

 // Conservative assumption if SL and TP are both touched in the same candle.
 if(slHit){
  trade.status="SL HIT";
  trade.closeTime=bar.time;
  return;
 }

 if(tp1Hit)trade.hitTP1=true;
 if(tp2Hit)trade.hitTP2=true;
 if(tp3Hit){
  trade.hitTP3=true;
  trade.status="TP3 HIT";
  trade.closeTime=bar.time;
 }
}

function updateTrades(closedBars){
 let changed=false;

 for(const trade of trades){
  if(trade.status!=="OPEN")continue;

  const newBars=closedBars
   .filter(b=>b.time>trade.lastProcessedTime)
   .sort((a,b)=>a.time-b.time);

  for(const bar of newBars){
   checkTrade(trade,bar);
   trade.lastProcessedTime=bar.time;
   changed=true;
   if(trade.status!=="OPEN")break;
  }
 }

 if(changed)saveTrades();
}

function addTrade(trade){
 if(!trade)return;
 if(trades.some(t=>t.id===trade.id))return;

 // Keep one simulated open trade at a time.
 if(trades.some(t=>t.status==="OPEN")){
  $("reason").textContent+=" An existing simulated trade is still open; no extra trade was recorded.";
  return;
 }

 trades.unshift(trade);
 saveTrades();
}

function renderHistory(){
 const body=$("historyBody");
 body.replaceChildren();

 const total=trades.length;
 const open=trades.filter(t=>t.status==="OPEN").length;
 const tp1=trades.filter(t=>t.hitTP1).length;
 const sl=trades.filter(t=>t.status==="SL HIT").length;

 $("totalCount").textContent=total;
 $("openCount").textContent=open;
 $("tp1Count").textContent=tp1;
 $("slCount").textContent=sl;
 $("emptyHistory").style.display=total?"none":"block";

 $("historyStatus").textContent=
  "Saved on this browser · TP1 reached: "+tp1+
  " · TP3 reached: "+trades.filter(t=>t.hitTP3).length+
  " · SL hit: "+sl;

 for(const t of trades){
  const tr=document.createElement("tr");
  const result=t.status==="OPEN"
   ? "OPEN"
   : t.status==="SL HIT"
    ? "SL HIT"+(t.hitTP1?" · TP1 previously hit":"")
    : "TP3 HIT";

  const values=[
   timeText(t.entryTime),t.side,fmt(t.entry),fmt(t.sl),
   fmt(t.tp1)+(t.hitTP1?" ✓":""),
   fmt(t.tp2)+(t.hitTP2?" ✓":""),
   fmt(t.tp3)+(t.hitTP3?" ✓":""),
   result
  ];

  values.forEach((value,i)=>{
   const td=document.createElement("td");
   td.textContent=value;
   if(i===1||i===7){
    td.className="tag "+(
     t.side==="BUY"?"buy":
     t.side==="SELL"?"sell":
     t.status==="TP3 HIT"?"win":
     t.status==="SL HIT"?"loss":"wait"
    );
   }
   tr.appendChild(td);
  });

  body.appendChild(tr);
 }
}

async function loadData(){
 $("refresh").disabled=true;
 $("chartStatus").textContent=
  "Loading "+(timeframe==="1min"?"M1":"M5")+" candles...";

 try{
  const response=await fetch(API+"?interval="+timeframe,{cache:"no-store"});
  const data=await response.json();

  if(!response.ok||data.status==="error"||!Array.isArray(data.values)){
   throw new Error(data.error||data.message||"Candle data unavailable");
  }

  bars=data.values.map(v=>({
   time:Math.floor(Date.parse(v.datetime.replace(" ","T")+"Z")/1000),
   open:Number(v.open),high:Number(v.high),
   low:Number(v.low),close:Number(v.close)
  })).filter(b=>
   Number.isFinite(b.time)&&Number.isFinite(b.open)&&
   Number.isFinite(b.high)&&Number.isFinite(b.low)&&
   Number.isFinite(b.close)
  ).sort((a,b)=>a.time-b.time);

  const candleSeconds=timeframe==="1min"?60:300;
  const now=Math.floor(Date.now()/1000);
  const closedBars=bars.filter(b=>b.time+candleSeconds<=now);

  if(bars.length<25)throw new Error("Not enough candles for analysis");
  if(!chart||!candles)throw new Error("Chart library is not ready");

  candles.setData(bars);
  chart.timeScale().fitContent();

  const latest=bars[bars.length-1];
  $("price").textContent="$"+fmt(latest.close);
  $("updated").textContent="Last candle: "+timeText(latest.time);
  $("chartStatus").textContent=bars.length+" candles loaded · "+timeframe.toUpperCase();
  $("dataStatus").textContent="Data source: Twelve Data XAU/USD via Cloudflare Worker.";

  // Check existing simulated trades against newly available closed candles.
  updateTrades(closedBars);

  // Generate a new signal only from a closed candle.
  const signal=makeSignal(closedBars);
  if(signal)addTrade(signal);

  renderHistory();

  const dataKey=timeframe+"-"+(closedBars.length?closedBars[closedBars.length-1].time:0);
  lastDataKey=dataKey;
 }catch(e){
  $("chartStatus").textContent="Data error: "+e.message;
  $("dataStatus").textContent="Check internet, Worker deployment and API quota.";
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

$("clearHistory").addEventListener("click",()=>{
 if(!confirm("Delete all saved simulated trade history?"))return;
 trades=[];
 saveTrades();
 renderHistory();
 resetSignal("History cleared. Waiting for a new confirmed signal.");
});

initChart();
renderHistory();
loadData();

// Refresh data periodically while this page remains open.
setInterval(()=>{
 if(!document.hidden)loadData();
},60000);
</script>
</body>
</html>
