idex.html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>GLOBAL INTELLIGENCE TERMINAL v2</title>

<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;500;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<!-- 🌍 Globe -->
<script src="https://unpkg.com/globe.gl"></script>

<style>
body {
margin: 0;
font-family: Inter;
background: radial-gradient(circle at top, #0f0f0f, #000);
color: white;
overflow: hidden;
}

/* HEADER */
header {
display: flex;
justify-content: space-between;
padding: 14px 18px;
border-bottom: 1px solid #222;
background: rgba(10,10,10,0.9);
}

.title {
color: #00ffcc;
font-weight: 700;
letter-spacing: 2px;
}

/* LAYOUT */
.container {
display: flex;
height: calc(100vh - 55px);
}

/* GLOBE */
#globe {
flex: 2;
}

/* PANEL */
.panel {
flex: 1;
background: rgba(15,15,15,0.95);
padding: 12px;
overflow-y: auto;
border-left: 1px solid #222;
}

.card {
background: linear-gradient(145deg, #111, #0a0a0a);
border: 1px solid #222;
padding: 10px;
border-radius: 10px;
margin-bottom: 10px;
transition: 0.2s;
}

.card:hover {
transform: scale(1.02);
border-color: #00ffcc;
}

input {
width: 100%;
padding: 10px;
background: #111;
border: 1px solid #333;
color: white;
border-radius: 8px;
margin-bottom: 10px;
}

.small {
font-size: 12px;
color: #aaa;
}

canvas {
height: 60px !important;
}
</style>
</head>

<body>

<header>
<div class="title">GLOBAL INTELLIGENCE TERMINAL</div>
<div id="clock"></div>
</header>

<div class="container">
<div id="globe"></div>

<div class="panel">
<input id="search" placeholder="Add stock (AAPL, TSLA, NVDA...)"/>

<h3>📊 MARKETS</h3>
<div id="stocks"></div>

<h3>📰 GLOBAL NEWS FEED</h3>
<div id="news"></div>
</div>
</div>

<script>

/////////////////////////////////////////////////////
// ⏰ CLOCK
////////////////////////////////////////////////////
setInterval(() => {
document.getElementById("clock").innerText = new Date().toLocaleTimeString();
}, 1000);

/////////////////////////////////////////////////////
// 🌍 GLOBE (interactive + news points)
////////////////////////////////////////////////////
const globe = Globe()(document.getElementById('globe'))
.globeImageUrl('//unpkg.com/three-globe/example/img/earth-night.jpg')
.bumpImageUrl('//unpkg.com/three-globe/example/img/earth-topology.png')
.backgroundColor('black')
.pointsData([
{ lat: 40.7, lng: -74, size: 0.5, color: "red", label: "New York News" },
{ lat: 51.5, lng: -0.1, size: 0.5, color: "orange", label: "London News" },
{ lat: 35.6, lng: 139.6, size: 0.5, color: "cyan", label: "Tokyo News" },
{ lat: 48.8, lng: 2.3, size: 0.5, color: "yellow", label: "Paris News" }
])
.pointColor(d => d.color)
.pointAltitude(0.02)
.pointLabel(d => d.label);

/////////////////////////////////////////////////////
// 💹 STOCK SYSTEM (SMOOTH ENGINE)
////////////////////////////////////////////////////
const API = "https://finnhub.io/api/v1/quote";
const TOKEN = "YOUR_API_KEY_HERE";

let stocks = ["AAPL","TSLA","NVDA","MSFT","AMZN","BTC","META"];

const history = {};
const charts = {};
const container = document.getElementById("stocks");

// create cards ONCE
stocks.forEach(s => {
const div = document.createElement("div");
div.className = "card";
div.id = `card-${s}`;
div.innerHTML = `
<b>${s}</b>
<div id="price-${s}">Loading...</div>
<canvas id="chart-${s}"></canvas>
<div class="small" id="ai-${s}">AI: analyzing...</div>
`;
container.appendChild(div);
history[s] = [];
});

async function updateStock(s) {
try {
const res = await fetch(`${API}?symbol=${s}&token=${TOKEN}`);
const data = await res.json();

const price = data.c;
const change = data.d;

if(!history[s]) history[s] = [];
history[s].push(price);
if(history[s].length > 25) history[s].shift();

document.getElementById(`price-${s}`).innerHTML =
`$${price.toFixed(2)} <span style="color:${change>=0?'lime':'red'}">
${change>=0?'▲':'▼'} ${change.toFixed(2)}
</span>`;

// AI fake sentiment (smarter feel)
document.getElementById(`ai-${s}`).innerText =
change >= 0
? "AI: bullish momentum detected 📈"
: "AI: bearish pressure detected 📉";

// chart
if(!charts[s]) {
const ctx = document.getElementById(`chart-${s}`);
charts[s] = new Chart(ctx, {
type: "line",
data: {
labels: history[s],
datasets: [{
data: history[s],
borderColor: "#00ffcc",
tension: 0.3,
pointRadius: 0
}]
},
options: {
responsive: true,
plugins: { legend: { display: false } },
scales: { x: { display:false }, y:{ display:false } }
}
});
} else {
charts[s].data.datasets[0].data = history[s];
charts[s].update();
}

} catch(err) {
console.log("stock error", err);
}
}

async function loopStocks() {
for(const s of stocks) {
await updateStock(s);
await new Promise(r => setTimeout(r, 600)); // prevents API spam
}
}

setInterval(loopStocks, 15000);
loopStocks();

/////////////////////////////////////////////////////
// 🔍 ADD STOCK
////////////////////////////////////////////////////
document.getElementById("search").addEventListener("keydown", e => {
if(e.key === "Enter") {
const val = e.target.value.toUpperCase();
if(!stocks.includes(val)) {
stocks.push(val);

const div = document.createElement("div");
div.className = "card";
div.id = `card-${val}`;
div.innerHTML = `
<b>${val}</b>
<div id="price-${val}">Loading...</div>
<canvas id="chart-${val}"></canvas>
<div class="small" id="ai-${val}">AI: analyzing...</div>
`;
container.appendChild(div);

history[val] = [];
}
e.target.value = "";
}
});

/////////////////////////////////////////////////////
// 📰 SIMPLE NEWS FEED
////////////////////////////////////////////////////
async function loadNews() {
const container = document.getElementById("news");

try {
const res = await fetch("https://newsapi.org/v2/top-headlines?language=en&pageSize=6&apiKey=YOUR_NEWS_KEY");
const data = await res.json();

container.innerHTML = "";

data.articles.forEach(a => {
const div = document.createElement("div");
div.className = "card";
div.innerHTML = `
<b>${a.title}</b>
<div class="small">global impact analyzing...</div>
`;
container.appendChild(div);
});

} catch(e) {
console.log(e);
}
}

loadNews();

</script>

</body>
</html>
