<html lang="th">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Phuket Trip — Route & Fare</title>
<style>
:root{
  --accent:#1e88e5; --muted:#6b7280; --card:#ffffff; --bg:#f6f8fb;
  --rounded:12px;
}
h1, h2, h3 { display: none; }
html,body{height:100%;margin:0;font-family:Inter,system-ui,-apple-system,"Sarabun",sans-serif;background:var(--bg);color:#112;}
.wrap{display:flex;height:100vh;gap:12px;padding:12px;box-sizing:border-box;}
.map-wrap{flex:1;position:relative;border-radius:var(--rounded);overflow:hidden;box-shadow:0 8px 24px rgba(15,23,42,.06);min-height:320px;}
#map{width:100%;height:100%}
.sidebar{width:420px;min-width:320px;background:var(--card);border-radius:var(--rounded);box-shadow:0 8px 24px rgba(15,23,42,.08);padding:16px;display:flex;flex-direction:column;gap:10px;overflow:auto;}
.logo{font-weight:700;font-size:18px;display:flex;align-items:center;gap:8px;position:relative;}
#btn-lang{position:absolute; top:0; right:0;padding:6px 10px;font-size:12px;background:var(--accent); color:#fff;border:none; border-radius:6px;cursor:pointer}
.search-row{display:flex;gap:8px;align-items:center}
#search,#searchStart{flex:1;padding:10px 12px;border-radius:10px;border:1px solid #e6e9ee;font-size:15px}
.btn{padding:9px 12px;border-radius:10px;border:0;background:var(--accent);color:#fff;cursor:pointer;font-size:14px}
.btn.alt{background:#eef; color:var(--accent); border:1px solid #d6e6ff}
.chips{display:flex;gap:8px;flex-wrap:wrap}
.chip{display:flex;gap:8px;align-items:center;padding:8px 10px;border-radius:999px;border:1px solid #eef;background:#fff;cursor:pointer;font-size:14px}
.section-title{font-weight:700;margin:6px 0;color:#0b2540}
.controls-row{display:flex;gap:8px;align-items:center}
select{padding:8px;border-radius:8px;border:1px solid #e6e9ee;background:#fff}
.routes{margin-top:8px;display:flex;flex-direction:column;gap:8px;overflow:auto;max-height:30vh;padding-right:6px}
.route-card{background:#fbfdff;border-radius:10px;padding:10px;border:1px solid #eef;cursor:pointer;display:flex;justify-content:space-between;align-items:center}
.route-card.selected{outline:3px solid rgba(30,136,229,.12)}
.route-left{display:flex;flex-direction:column;width:100%;}

.route-title{font-weight:700}
.route-meta{color:var(--muted);font-size:13px}
.fare-pill{font-weight:700;color:var(--accent)}
.fare-table{position:absolute;right:18px;bottom:18px;background:var(--card);padding:12px;border-radius:12px;box-shadow:0 8px 24px rgba(15,23,42,.12);min-width:200px;max-width:90vw;overflow:auto;z-index:10;display:none}
.fare-table table{border-collapse:collapse;width:100%;font-size:14px}
.fare-table th{font-weight:700;text-align:left;padding:6px 4px;color:#123}
.fare-table td{padding:6px 4px;color:#334}
.legend{position:absolute;left:18px;bottom:18px;background:#fff;padding:8px;border-radius:8px;border:1px solid #eef;display:flex;gap:8px;align-items:center;font-size:13px;flex-wrap:wrap;z-index:10;}
.dot{width:36px;height:6px;border-radius:6px}
.fast{background:linear-gradient(90deg,#2ecc71,#1faa4a)}
.moderate{background:linear-gradient(90deg,#f1c40f,#f39c12)}
.heavy{background:linear-gradient(90deg,#e74c3c,#c0392b)}
@media(max-width:1000px){
  .wrap{flex-direction:column;padding:8px;}
  .sidebar{width:auto;min-width:unset;max-height:40vh;}
  .map-wrap{height:60vh;min-height:320px;}
  .fare-table{right:10px;bottom:10px;min-width:140px;}
}
</style>
</head>
<body>
<div class="wrap">
  <aside class="sidebar" aria-label="controls">
    <div class="logo">Trip Roule — Phuket demo
      <button id="btn-lang">🌐 English</button>
    </div>
    <div class="section-title" id="lbl-origin">ต้นทาง</div>
    <div class="search-row">
      <input id="searchStart" placeholder="เลือกต้นทาง..." aria-label="ต้นทาง">
      <button id="btn-current" class="btn alt" title="ใช้ตำแหน่งปัจจุบัน" aria-label="ใช้ตำแหน่งปัจจุบัน">📍</button>
    </div>
    <div class="section-title" id="lbl-dest">ปลายทาง</div>
    <div class="search-row">
      <input id="search" placeholder="ค้นหาปลายทาง..." aria-label="ปลายทาง">
    </div>
    <div class="controls-row">
      <select id="vehicle" aria-label="เลือกยานพาหนะ"></select>
      <button id="btn-calc" class="btn">คำนวณ</button>
      <button id="btn-reset" class="btn alt">รีเซ็ต</button>
    </div>
    <button id="btn-toggle-fare" class="btn alt">แสดง/ซ่อนตารางราคา</button>
    <div class="section-title" style="margin-top:8px" id="lbl-recommend">เส้นทางที่แนะนำ</div>
    <div class="routes" id="routesList" role="list"></div>
  </aside>
  <main class="map-wrap">
    <div id="map" role="application" aria-label="Map"></div>
    <div class="fare-table" id="fareTable" aria-hidden="true">
      <div id="fare-title" style="font-weight:800;margin-bottom:6px">ตารางราคา</div>
      <table>
        <thead><tr><th id="th-vehicle">Vehicle</th><th id="th-base">Base</th><th id="th-perkm">Per km</th></tr></thead>
        <tbody id="fareRows"></tbody>
      </table>
    </div>
    <div class="legend" id="legend">
      <div style="font-weight:700;margin-right:8px" id="legend-title">Traffic</div>
      <div class="dot fast"></div><div style="font-size:13px;color:var(--muted)" id="legend-fast">Fast</div>
      <div class="dot moderate"></div><div style="font-size:13px;color:var(--muted)" id="legend-moderate">Moderate</div>
      <div class="dot heavy"></div><div style="font-size:13px;color:var(--muted)" id="legend-heavy">Heavy</div>
    </div>
  </main>
</div>

<script>
/* ---------------- Data ---------------- */
const vehicleRates = {
  grabCar:{base:35,perKm:10,freeKm:2,min:50}, grabBike:{base:20,perKm:7,freeKm:1,min:25},
  boltEconomy:{base:50,perKm:10,freeKm:2,min:100}, boltStandard:{base:100,perKm:12,freeKm:2,min:200},
  boltVan:{base:250,perKm:15,freeKm:2,min:300}, boltXL:{base:200,perKm:15,freeKm:2,min:300},
  boltTaxi:{base:100,perKm:12,freeKm:2,min:200}, inDrive:{base:30,perKm:9,freeKm:1,min:80},
  taxiMeter:{base:50,perKm:12,freeKm:2,min:100}, smartBus:{base:100,perKm:0,freeKm:0,min:100,fixed:true},
  songthaew:{base:40,perKm:0,freeKm:0,min:40,fixed:true}, pinkBus:{base:15,perKm:0,freeKm:0,min:15,fixed:true},
  tukTuk:{base:200,perKm:0,freeKm:0,min:200,fixed:true}, motoTaxi:{base:80,perKm:0,freeKm:0,min:80,fixed:true}
};
const vehicleNames = {
  th:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard",
       boltVan:"Bolt Van", boltXL:"Bolt XL", boltTaxi:"Bolt Taxi", inDrive:"inDrive", taxiMeter:"Taxi Meter",
       smartBus:"รถบัส Smart Bus", songthaew:"รถสองแถว", pinkBus:"รถบัสสีชมพู", tukTuk:"ตุ๊กตุ๊ก", motoTaxi:"มอเตอร์ไซค์รับจ้าง"},
  en:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard",
       boltVan:"Bolt Van", boltXL:"Bolt XL", boltTaxi:"Bolt Taxi", inDrive:"inDrive", taxiMeter:"Taxi Meter",
       smartBus:"Smart Bus", songthaew:"Songthaew", pinkBus:"Pink Bus", tukTuk:"Tuk Tuk", motoTaxi:"Moto Taxi"}
};
const i18n = {
  th:{search:"ค้นหาปลายทาง...", start:"เลือกต้นทาง...", originLabel:"ต้นทาง", destinationLabel:"ปลายทาง",
      calc:"คำนวณ", reset:"รีเซ็ต", fareTable:"ตารางราคา", popular:["หาดป่าตอง","พระใหญ่ (Big Buddha)","วัดฉลอง (Wat Chalong)"],
      noRoute:"ยังไม่มีเส้นทาง", onlyPhuket:"กรุณาเลือกตำแหน่งในภูเก็ตเท่านั้น",
      routeLabel:"เส้นทาง", kmLabel:"กม.", minsLabel:"นาที", traffic:"การจราจร", fast:"คล่องตัว",
      moderate:"ปานกลาง", heavy:"หนาแน่น", vehicleCol:"ยานพาหนะ", baseCol:"ฐานเริ่มต้น", perKmCol:"ต่อ กม.",
      toggleFare:"แสดง/ซ่อนตารางราคา"},
  en:{search:"Search destination...", start:"Select origin...", originLabel:"Origin", destinationLabel:"Destination",
      calc:"Calculate", reset:"Reset", fareTable:"Fare Table", popular:["Patong Beach","Big Buddha","Wat Chalong"],
      noRoute:"No routes yet", onlyPhuket:"Please select a location within Phuket only", routeLabel:"Route",
      kmLabel:"km", minsLabel:"mins", traffic:"Traffic", fast:"Fast", moderate:"Moderate", heavy:"Heavy",
      vehicleCol:"Vehicle", baseCol:"Base", perKmCol:"Per km", toggleFare:"Show/Hide Fare Table"}
};
let currentLang = 'th';

/* ---------------- State ---------------- */
let map,directionsService,trafficLayer,placesService;
let markerA=null, markerB=null;
let currentPos=null,lastRoutes=[],selectedRouteIndex=0,polyLines=[],poiMarkers=[];

/* ---------------- Helpers ---------------- */
function km(m){ return (m/1000).toFixed(1); }
function calculateFare(route, vehicleKey){
  const v = vehicleRates[vehicleKey]; if(!v) return 0;
  if(v.fixed) return v.min;
  const kmDistance = route.legs.reduce((sum,leg)=>sum+leg.distance.value,0)/1000;
  let fare = v.base + Math.max(0, kmDistance - v.freeKm)*v.perKm;
  return Math.max(Math.round(fare),v.min);
}

/* ---------------- UI ---------------- */
function applyLanguage(){
  document.getElementById("btn-lang").textContent = (currentLang==="th"?"🌐 English":"🌐 ภาษาไทย");
  document.getElementById("search").placeholder = i18n[currentLang].search;
  document.getElementById("searchStart").placeholder = i18n[currentLang].start;
  document.getElementById("btn-calc").textContent = i18n[currentLang].calc;
  document.getElementById("btn-reset").textContent = i18n[currentLang].reset;
  document.getElementById("fare-title").textContent = i18n[currentLang].fareTable;
  document.getElementById("lbl-origin").textContent = i18n[currentLang].originLabel;
  document.getElementById("lbl-dest").textContent = i18n[currentLang].destinationLabel;
  document.getElementById("lbl-recommend").textContent = i18n[currentLang].routeLabel;
  document.getElementById("btn-toggle-fare").textContent = i18n[currentLang].toggleFare;
}

/* ---------------- Map Init ---------------- */
function initMap(){
  const center = {lat:7.8804,lng:98.3923};
  map = new google.maps.Map(document.getElementById('map'),{center,zoom:11,gestureHandling:'greedy'});
  directionsService = new google.maps.DirectionsService();
  trafficLayer = new google.maps.TrafficLayer(); trafficLayer.setMap(map);
  placesService = new google.maps.places.PlacesService(map);

  // Autocomplete
  const acStart = new google.maps.places.Autocomplete(document.getElementById("searchStart"),{bounds:map.getBounds()});
  const acDest = new google.maps.places.Autocomplete(document.getElementById("search"),{bounds:map.getBounds()});
  acStart.addListener('place_changed',()=>{ const place = acStart.getPlace(); setMarker('A',place); });
  acDest.addListener('place_changed',()=>{ const place = acDest.getPlace(); setMarker('B',place); });
}

/* ---------------- Markers ---------------- */
function setMarker(type, place){
  const pos = place.geometry.location;
  if(type==='A'){
    if(!markerA) markerA = new google.maps.Marker({map,icon:'http://maps.google.com/mapfiles/ms/icons/green-dot.png'});
    markerA.setPosition(pos); markerA.setMap(map);
  } else {
    if(!markerB) markerB = new google.maps.Marker({map,icon:'http://maps.google.com/mapfiles/ms/icons/red-dot.png'});
    markerB.setPosition(pos); markerB.setMap(map);
  }
  map.panTo(pos);
}

/* ---------------- Routes ---------------- */
function clearPolylines(){ polyLines.forEach(p=>p.setMap(null)); polyLines=[]; }
function clearMarkers(){ if(markerA){ markerA.setMap(null); markerA=null; } if(markerB){ markerB.setMap(null); markerB=null; } }
function calcRoutes(){
  if(!markerA||!markerB){ alert(i18n[currentLang].onlyPhuket); return; }
  directionsService.route({origin:markerA.getPosition(),destination:markerB.getPosition(),travelMode:'DRIVING',provideRouteAlternatives:true},(res,status)=>{
    if(status!=="OK"){ alert("Error:"+status); return; }
    lastRoutes = res.routes; selectedRouteIndex=0;
    drawRoutes();
  });
}
function drawRoutes(){
  clearPolylines();
  const bounds = new google.maps.LatLngBounds();
  const routesList = document.getElementById("routesList"); routesList.innerHTML="";
  lastRoutes.forEach((r,i)=>{
    const color = ['#1e88e5','#29b6f6','#00acc1'][i%3];
    const poly = new google.maps.Polyline({path:r.overview_path,map,strokeColor:color,strokeOpacity:0.7,strokeWeight:6});
    polyLines.push(poly); bounds.extend(r.bounds||r.overview_path[0]); r.overview_path.forEach(p=>bounds.extend(p));
    poly.addListener('click',()=>selectRoute(i)); poly.addListener('touchstart',()=>selectRoute(i));
    const card = document.createElement("div"); card.className="route-card"+(i===0?" selected":"");
    card.innerHTML = `<div class="route-left"><div class="route-title">${i18n[currentLang].routeLabel} ${i+1}</div>
      <div class="route-meta">${km(r.legs.reduce((sum,leg)=>sum+leg.distance.value,0))} ${i18n[currentLang].kmLabel} | ${r.legs.reduce((sum,leg)=>sum+leg.duration.value,0)/60|0} ${i18n[currentLang].minsLabel}</div></div>
      <div class="fare-pill">${calculateFare(r,document.getElementById("vehicle").value)} ฿</div>`;
    card.addEventListener('click',()=>selectRoute(i));
    routesList.appendChild(card);
  });
  map.fitBounds(bounds);
}
function selectRoute(i){
  selectedRouteIndex=i;
  document.querySelectorAll(".route-card").forEach((c,idx)=>c.classList.toggle("selected",idx===i));
}

/* ---------------- Events ---------------- */
document.getElementById("btn-lang").addEventListener("click",()=>{
  currentLang = currentLang==="th"?"en":"th"; applyLanguage(); drawRoutes();
});
document.getElementById("btn-calc").addEventListener("click",calcRoutes);
document.getElementById("btn-reset").addEventListener("click",()=>{ clearMarkers(); clearPolylines(); lastRoutes=[]; document.getElementById("routesList").innerHTML=""; });
document.getElementById("btn-toggle-fare").addEventListener("click",()=>{
  const ft = document.getElementById("fareTable"); ft.style.display=(ft.style.display==='none'?'block':'none');
});

/* ---------------- Vehicle Options ---------------- */
function initVehicles(){
  const sel = document.getElementById("vehicle"); sel.innerHTML="";
  Object.keys(vehicleRates).forEach(k=>{
    const o = document.createElement("option"); o.value=k; o.textContent=vehicleNames[currentLang][k]||k;
    sel.appendChild(o);
  });
  sel.addEventListener("change",()=>drawRoutes());
}

/* ---------------- Init ---------------- */
applyLanguage(); initVehicles(); window.initMap=initMap;
</script>
<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places&callback=initMap" async defer></script>
</body>
</html>
