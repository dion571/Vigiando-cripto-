<!DOCTYPE html><html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Crypto IA Trader PRO</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body{margin:0;font-family:Arial;background:#0f172a;color:white}
.menu{display:flex;background:#020617}
.menu button{flex:1;padding:15px;border:none;background:#020617;color:white;font-size:16px;cursor:pointer}
.menu button:hover{background:#1e293b}
.section{display:none;padding:20px}
.active{display:block}
.card{background:#1e293b;padding:20px;border-radius:12px;margin:auto;max-width:700px;margin-bottom:20px}
.green{color:#22c55e}
.red{color:#ef4444}
.yellow{color:#facc15}
h2{margin-top:0}
input{padding:8px;border-radius:6px;border:none;margin-bottom:10px;width:80%}
button.action{background:#22c55e;color:#0f172a;padding:10px 20px;border:none;border-radius:6px;cursor:pointer}
button.action:hover{background:#16a34a}
</style>
</head>
<body><div class="menu">
<button onclick="show('market')">📊 Mercado</button>
<button onclick="show('signals')">🤖 IA Trader</button>
<button onclick="show('simulator')">💰 Simulador</button>
</div><!-- Mercado --><div id="market" class="section active">
<div class="card">
<h2>📈 Preço e Tendência</h2>
<select id="crypto">
<option value="BTCBRL">Bitcoin</option>
<option value="ETHBRL">Ethereum</option>
<option value="SOLBRL">Solana</option>
<option value="XRPBRL">XRP</option>
<option value="ADABRL">Cardano</option>
<option value="DOGEBRL">Dogecoin</option>
<option value="BNBBRL">BNB</option>
</select>
<h1 id="price">Carregando...</h1>
<p id="variation"></p>
<p id="trend"></p>
<p id="rsi"></p>
<canvas id="chart"></canvas>
</div>
</div><!-- IA Trader --><div id="signals" class="section">
<div class="card">
<h2>🤖 IA Trader</h2>
<p id="signal" class="yellow">Carregando...</p>
<p id="prediction"></p>
<p id="monthlyForecast"></p>
<p id="suggestedInvestment"></p>
</div>
</div><!-- Simulador --><div id="simulator" class="section">
<div class="card">
<h2>💰 Simulador de Investimento</h2>
<input id="value" type="number" placeholder="Valor mensal em R$"><br>
<input id="years" type="number" placeholder="Número de anos"><br>
<button class="action" onclick="simulate()">Calcular</button>
<h3 id="result"></h3>
</div>
</div><script>
function show(id){
 document.querySelectorAll('.section').forEach(s=>s.classList.remove('active'));
 document.getElementById(id).classList.add('active');
}

let history=[];
let chart;
let lastPrice=0;

async function load(){
 let symbol=document.getElementById('crypto').value;
 try{
   let res=await fetch(`https://api.binance.com/api/v3/ticker/price?symbol=${symbol}`);
   let data=await res.json();
   let price=parseFloat(data.price);
   document.getElementById('price').innerHTML="R$ "+price.toLocaleString('pt-BR',{minimumFractionDigits:2});

   if(lastPrice!==0){
     let diff=((price-lastPrice)/lastPrice)*100;
     document.getElementById('variation').innerHTML=(diff>0?"📈 Subiu ":"📉 Caiu ")+diff.toFixed(2)+"%";
     document.getElementById('variation').className=diff>0?'green':'red';
   }
   lastPrice=price;

   history.push(price);
   if(history.length>100) history.shift();

   updateTrend();
   calculateRSI();
   updateSignal();
   updatePrediction();
   updateMonthlyForecast();
   chartUpdate();

 }catch{
   document.getElementById('price').innerHTML='Sem conexão';
 }
}

function updateTrend(){
 if(history.length<20) return;
 let avg=history.slice(-20).reduce((a,b)=>a+b)/20;
 let last=history[history.length-1];
 let t='';
 if(last>avg*1.03) t='🔥 Tendência forte de ALTA';
 else if(last<avg*0.97) t='⚠️ Tendência forte de QUEDA';
 else t='📊 Mercado lateral';
 document.getElementById('trend').innerHTML=t;
}

function calculateRSI(){
 if(history.length<15) return;
 let g=0,l=0;
 for(let i=1;i<15;i++){let d=history[i]-history[i-1]; if(d>0) g+=d; else l-=d;}
 let rsi=100-(100/(1+g/(l||1)));
 let txt='';
 if(rsi>70) txt=`🔴 RSI ${rsi.toFixed(0)} — Sobrecomprado`;
 else if(rsi<30) txt=`🟢 RSI ${rsi.toFixed(0)} — Sobrevendido`;
 else txt=`🟡 RSI ${rsi.toFixed(0)} — Neutro`;
 document.getElementById('rsi').innerHTML=txt;
}

function updateSignal(){
 if(history.length<30) return;
 let avg=history.slice(-30).reduce((a,b)=>a+b)/30;
 let price=history[history.length-1];
 let s='';
 if(price>avg*1.05) s='🔴 VENDER';
 else if(price<avg*0.95) s='🟢 COMPRAR';
 else s='🟡 AGUARDAR';
 document.getElementById('signal').innerHTML='Sinal: '+s;
}

function updatePrediction(){
 if(history.length<40) return;
 let first=history[0];
 let last=history[history.length-1];
 let change=((last-first)/first)*100;
 document.getElementById('prediction').innerHTML='Possível variação no próximo mês: '+change.toFixed(2)+'%';
}

function updateMonthlyForecast(){
 if(history.length<40) return;
 let first=history[0];
 let last=history[history.length-1];
 let change=((last-first)/first)*100;
 let suggested=0;
 if(change>2) suggested=1000;
 else if(change>0) suggested=500;
 else suggested=200;
 document.getElementById('monthlyForecast').innerHTML='Sugestão de investimento: R$ '+suggested;
 document.getElementById('suggestedInvestment').innerHTML='Momento: '+(change>0?'Bom para investir':'Aguarde melhor momento');
}

function chartUpdate(){
 let ctx=document.getElementById('chart').getContext('2d');
 if(!chart){
 chart=new Chart(ctx,{type:'line',data:{labels:history.map((_,i)=>i),datasets:[{data:history,borderColor:'#22c55e',fill:false}]},options:{responsive:true,plugins:{legend:{display:false}}}});
 }else{
 chart.data.datasets[0].data=history;
 chart.update();
 }
}

function simulate(){
 let v=parseFloat(document.getElementById('value').value);
 let y=parseInt(document.getElementById('years').value);
 if(!v||!y){alert('Preencha todos os campos'); return;}
 let months=y*12;
 let total=0;
 for(let i=0;i<months;i++) total=(total+v)*1.01;
 document.getElementById('result').innerHTML='Valor futuro estimado: R$ '+total.toLocaleString('pt-BR',{maximumFractionDigits:0});
}

load();
setInterval(load,5000);
</script></body>
</html>
