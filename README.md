<!doctype html>
<html lang="th">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Phuket Trip — Route & Fare (Beta API)</title>
<style>
:root{
  --accent:#1e88e5; --muted:#6b7280; --card:#ffffff; --bg:#f6f8fb;
  --rounded:12px; --glass: rgba(255,255,255,0.85);
}
*{box-sizing:border-box}
html,body{height:100%;margin:0;font-family:Inter,system-ui,-apple-system,"Sarabun",sans-serif;background:var(--bg);color:#112}
.wrap{display:flex;flex-direction:column;height:100vh;gap:10px;padding:10px}
.map-wrap{position:relative;border-radius:var(--rounded);overflow:hidden;box-shadow:0 12px 30px rgba(15,23,42,.06);height:60vh}
#map{width:100%;height:100%}
.controls{background:var(--card);border-radius:var(--rounded);padding:12px;box-shadow:0 8px 24px rgba(15,23,42,.06);display:flex;flex-direction:column;gap:8px}
.row{display:flex;gap:8px;align-items:center}
input,select,button{font-size:15px;padding:10px;border-radius:10px;border:1px solid #e6e9ee}
input[placeholder]{color:#667}
button{background:var(--accent);color:#fff;border:0;cursor:pointer}
.btn-alt{background:#eef;color:var(--accent);border:1px solid #d6e6ff}
.small{padding:8px 10px;font-size:13px;border-radius:8px}
.chips{display:flex;gap:8px;flex-wrap:wrap}
.route-list{display:flex;flex-direction:column;gap:8px;max-height:30vh;overflow:auto;padding-right:6px}
.route-card{background:#fbfdff;border-radius:10px;padding:10px;border:1px solid #eef;cursor:pointer;display:flex;justify-content:space-between;align-items:center}
.route-card.selected{outline:3px solid rgba(30,136,229,.12)}
.fare-pill{font-weight:700;color:var(--accent)}
.fare-table{position:fixed;right:14px;bottom:14px;background:var(--card);padding:12px;border-radius:12px;box-shadow:0 12px 30px rgba(15,23,42,.12);min-width:220px;z-index:110;display:none}
.fare-table table{border-collapse:collapse;width:100%;font-size:14px}
.fare-table th{font-weight:700;text-align:left;padding:6px 4px;color:#123}
.fare-table td{padding:6px 4px;color:#334}
.legend{position:fixed;left:14px;bottom:14px;background:var(--card);padding:8px;border-radius:8px;border:1px solid #eef;display:flex;gap:8px;align-items:center;font-size:13px;flex-wrap:wrap;z-index:110}
.dot{width:36px;height:6px;border-radius:6px}
.fast{background:linear-gradient(90deg,#2ecc71,#1faa4a)}
.moderate{background:linear-gradient(90deg,#f1c40f,#f39c12)}
.heavy{background:linear-gradient(90deg,#e74c3c,#c0392b)}

/* Mobile-first layout (map top, controls bottom) */
.header{display:flex;justify-content:space-between;align-items:center;gap:8px}
.logo{font-weight:700;font-size:18px}
.lang-btn{background:transparent;border:0;cursor:pointer;font-size:14px}
.controls-bottom{display:flex;flex-direction:column;gap:8px}
@media(min-width:1000px){
  .wrap{flex-direction:row;padding:12px;height:100vh}
  .map-wrap{flex:1;height:100%}
  .controls{width:420px;min-width:320px;height:100%;overflow:auto}
  .fare-table{right:22px;bottom:22px}
  .legend{left:22px;bottom:22px}
}
.note{font-size:13px;color:var(--muted)}
.small-muted{font-size:13px;color:var(--muted)}
</style>
</head>
<body>
<div class="wrap">
  <main class="map-wrap" aria-label="Map area">
    <div id="map" role="application" aria-label="Map"></div>

    <!-- Fare table floating -->
    <div class="fare-table" id="fareTable" aria-hidden="true">
      <div id="fare-title" style="font-weight:800;margin-bottom:6px">ตารางราคา</div>
      <table>
        <thead><tr><th id="th-vehicle">ยานพาหนะ</th><th id="th-base">ฐาน</th><th id="th-perkm">ต่อ กม.</th></tr></thead>
        <tbody id="fareRows"></tbody>
      </table>
    </div>

    <!-- Legend -->
    <div class="legend" id="legend">
      <div style="font-weight:700;margin-right:8px" id="legend-title">การจราจร</div>
      <div class="dot fast"></div><div class="small-muted" id="legend-fast">คล่องตัว</div>
      <div class="dot moderate"></div><div class="small-muted" id="legend-moderate">ปานกลาง</div>
      <div class="dot heavy"></div><div class="small-muted" id="legend-heavy">หนาแน่น</div>
    </div>
  </main>

  <aside class="controls" aria-label="controls">
    <div class="header">
      <div class="logo">Trip Roule — Phuket demo</div>
      <div>
        <button id="btn-lang" class="lang-btn" title="เปลี่ยนภาษา">ภาษา: ไทย</button>
      </div>
    </div>

    <!-- Place Autocomplete (Web Component) -->
    <label id="lbl-origin">ต้นทาง</label>
    <gmpx-place-autocomplete id="searchStart" placeholder="เลือกต้นทาง..." aria-label="ต้นทาง"></gmpx-place-autocomplete>
    <div class="row">
      <button id="btn-current" class="btn-alt small" title="ใช้ตำแหน่งปัจจุบัน">📍</button>
      <div class="note">หรือพิมพ์เพื่อค้นหา</div>
    </div>

    <label id="lbl-dest">ปลายทาง</label>
    <gmpx-place-autocomplete id="search" placeholder="ค้นหาปลายทาง..." aria-label="ปลายทาง"></gmpx-place-autocomplete>

    <div class="row">
      <select id="vehicle" aria-label="เลือกยานพาหนะ"></select>
      <button id="btn-calc" class="small">คำนวณ</button>
      <button id="btn-reset" class="btn-alt small">รีเซ็ต</button>
    </div>

    <div class="row">
      <button id="btn-toggle-fare" class="btn-alt small">แสดง/ซ่อนตารางราคา</button>
      <div style="flex:1"></div>
      <div class="small-muted" id="trafficStatus">Traffic: ON</div>
    </div>

    <div style="margin-top:8px;font-weight:700" id="lbl-recommend">เส้นทางที่แนะนำ</div>
    <div class="route-list" id="routesList" role="list">ยังไม่มีเส้นทาง</div>

    <div style="margin-top:8px" class="small-muted">Popular: <span id="popularList"></span></div>
  </aside>
</div>

<script>
/* ================================
   Data (Rates, Names, i18n)
   ================================ */
const vehicleRates = {
  grabCar:{base:35,perKm:10,freeKm:2,min:50},
  grabBike:{base:20,perKm:7,freeKm:1,min:25},
  boltEconomy:{base:50,perKm:10,freeKm:2,min:100},
  boltStandard:{base:100,perKm:12,freeKm:2,min:200},
  boltVan:{base:250,perKm:15,freeKm:2,min:300},
  taxiMeter:{base:50,perKm:12,freeKm:2,min:100},
  smartBus:{base:100,perKm:0,freeKm:0,min:100,fixed:true},
  songthaew:{base:40,perKm:0,freeKm:0,min:40,fixed:true},
  pinkBus:{base:15,perKm:0,freeKm:0,min:15,fixed:true},
  tukTuk:{base:200,perKm:0,freeKm:0,min:200,fixed:true},
  motoTaxi:{base:80,perKm:0,freeKm:0,min:80,fixed:true}
};

const vehicleNames = {
  th:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard", boltVan:"Bolt Van", taxiMeter:"Taxi Meter", smartBus:"รถบัส Smart Bus", songthaew:"รถสองแถว", pinkBus:"รถบัสสีชมพู", tukTuk:"ตุ๊กตุ๊ก", motoTaxi:"มอเตอร์ไซค์รับจ้าง"},
  en:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard", boltVan:"Bolt Van", taxiMeter:"Taxi Meter", smartBus:"Smart Bus", songthaew:"Songthaew", pinkBus:"Pink Bus", tukTuk:"Tuk Tuk", motoTaxi:"Moto Taxi"}
};

const i18n = {
  th: {
    search:"ค้นหาปลายทาง...",
    start:"เลือกต้นทาง...",
    originLabel:"ต้นทาง",
    destinationLabel:"ปลายทาง",
    calc:"คำนวณ",
    reset:"รีเซ็ต",
    fareTable:"ตารางราคา",
    popular:["หาดป่าตอง","พระใหญ่ (Big Buddha)","วัดฉลอง (Wat Chalong)"],
    noRoute:"ยังไม่มีเส้นทาง",
    onlyPhuket:"กรุณาเลือกตำแหน่งในภูเก็ตเท่านั้น",
    outsidePhuket:"ตำแหน่งอยู่นอกภูเก็ต",
    routeLabel:"เส้นทาง",
    kmLabel:"กม.",
    minsLabel:"นาที",
    traffic:"การจราจร",
    fast:"คล่องตัว",
    moderate:"ปานกลาง",
    heavy:"หนาแน่น",
    vehicleCol:"ยานพาหนะ",
    baseCol:"ฐานเริ่มต้น",
    perKmCol:"ต่อ กม.",
    currentPosBtn: "ใช้ตำแหน่งปัจจุบัน",
    toggleFare: "แสดง/ซ่อนตารางราคา",
    langBtn:"ภาษา: ไทย",
    priceEstimate:"ประมาณ"
  },
  en: {
    search:"Search destination...",
    start:"Select origin...",
    originLabel:"Origin",
    destinationLabel:"Destination",
    calc:"Calculate",
    reset:"Reset",
    fareTable:"Fare Table",
    popular:["Patong Beach","Big Buddha","Wat Chalong"],
    noRoute:"No routes yet",
    onlyPhuket:"Please select a location within Phuket only",
    outsidePhuket:"Location is outside Phuket",
    routeLabel:"Route",
    kmLabel:"km",
    minsLabel:"mins",
    traffic:"Traffic",
    fast:"Fast",
    moderate:"Moderate",
    heavy:"Heavy",
    vehicleCol:"Vehicle",
    baseCol:"Base",
    perKmCol:"Per km",
    currentPosBtn: "Use current location",
    toggleFare: "Toggle Fare Table",
    langBtn:"Language: EN",
    priceEstimate:"Est."
  }
};

let currentLang = 'th';

/* ================================
   State variables
   ================================ */
let map, directionsService;
let phuketBounds;
let markerA = null, markerB = null; // AdvancedMarkerElement
let currentPos = null;
let lastRoutes = [], polyLines = [], poiMarkers = [];
let selectedRouteIndex = 0;
let trafficLayer = null;

/* ================================
   Helpers
   ================================ */
function km(m){ return (m/1000).toFixed(1); }
function round(n){ return Math.round(n); }

function calculateFare(route, vehicleKey){
  const v = vehicleRates[vehicleKey];
  if(!v) return 0;
  if(v.fixed) return v.min;
  const kmDistance = route.legs[0].distance.value / 1000;
  let fare = v.base + Math.max(0, kmDistance - v.freeKm) * v.perKm;
  return Math.max(Math.round(fare), v.min);
}

/* ================================
   UI: language & fare table
   ================================ */
function applyLanguage(){
  document.getElementById("btn-lang").textContent = i18n[currentLang].langBtn;
  document.getElementById("search").setAttribute('placeholder', i18n[currentLang].search);
  document.getElementById("searchStart").setAttribute('placeholder', i18n[currentLang].start);
  document.getElementById("btn-calc").textContent = i18n[currentLang].calc;
  document.getElementById("btn-reset").textContent = i18n[currentLang].reset;
  document.getElementById("fare-title").textContent = i18n[currentLang].fareTable;
  document.getElementById("lbl-origin").textContent = i18n[currentLang].originLabel;
  document.getElementById("lbl-dest").textContent = i18n[currentLang].destinationLabel;
  document.getElementById("lbl-recommend").textContent = i18n[currentLang].routeLabel;
  document.getElementById("legend-title").textContent = i18n[currentLang].traffic;
  document.getElementById("legend-fast").textContent = i18n[currentLang].fast;
  document.getElementById("legend-moderate").textContent = i18n[currentLang].moderate;
  document.getElementById("legend-heavy").textContent = i18n[currentLang].heavy;
  document.getElementById("th-vehicle").textContent = i18n[currentLang].vehicleCol;
  document.getElementById("th-base").textContent = i18n[currentLang].baseCol;
  document.getElementById("th-perkm").textContent = i18n[currentLang].perKmCol;
  document.getElementById("btn-toggle-fare").textContent = i18n[currentLang].toggleFare;

  // populate vehicle select & default
  const sel = document.getElementById("vehicle"); sel.innerHTML = "";
  const ph = document.createElement("option"); ph.value=""; ph.disabled=true; ph.textContent = (currentLang==='th'?'-- เลือกประเภทยานพาหนะ --':'-- Select vehicle --');
  sel.appendChild(ph);
  for(const k in vehicleRates){
    const opt = document.createElement("option"); opt.value=k;
    opt.textContent = (vehicleNames[currentLang] && vehicleNames[currentLang][k]) ? vehicleNames[currentLang][k] : k;
    sel.appendChild(opt);
  }
  sel.value = 'grabCar' in vehicleRates ? 'grabCar' : Object.keys(vehicleRates)[0];

  // popular
  document.getElementById('popularList').textContent = i18n[currentLang].popular.join(', ');

  // routes placeholder
  if(!lastRoutes || lastRoutes.length===0){
    document.getElementById('routesList').textContent = i18n[currentLang].noRoute;
  }

  renderFareTable();
}

function renderFareTable(){
  const tbody = document.getElementById('fareRows'); tbody.innerHTML = '';
  for(const k in vehicleRates){
    const v = vehicleRates[k];
    const name = vehicleNames[currentLang] && vehicleNames[currentLang][k] ? vehicleNames[currentLang][k] : k;
    const row = document.createElement('tr');
    if(v.fixed){ row.innerHTML = `<td>${name}</td><td>${v.min} ฿ (fixed)</td><td>-</td>`; }
    else { row.innerHTML = `<td>${name}</td><td>${v.base} ฿</td><td>${v.perKm} ฿</td>`; }
    tbody.appendChild(row);
  }
  document.getElementById('fareTable').style.display = 'none';
}

/* ================================
   Map init & new-API wiring
   ================================ */
async function initMap(){
  phuketBounds = new google.maps.LatLngBounds({lat:7.7,lng:98.2},{lat:8.1,lng:98.6});
  const center = {lat:7.8804,lng:98.3923};
  map = new google.maps.Map(document.getElementById('map'), {center,zoom:11,mapTypeControl:false,streetViewControl:false, restriction:{latLngBounds:phuketBounds}});
  directionsService = new google.maps.DirectionsService();
  trafficLayer = new google.maps.TrafficLayer();
  trafficLayer.setMap(map);

  // AdvancedMarkerElement will be used by creating instances of google.maps.marker.AdvancedMarkerElement

  // PlaceAutocompleteElement (web component) events
  const acStart = document.getElementById('searchStart');
  const acEnd = document.getElementById('search');

  acStart.addEventListener('gmpx-placechange', async ()=> {
    const text = acStart.value;
    if(!text) return;
    try{
      const place = new google.maps.places.Place({ textQuery: text });
      const result = await place.fetchFields({ fields: ['location','name','formatted_address'] });
      const loc = result.location;
      if(!phuketBounds.contains(new google.maps.LatLng(loc.lat, loc.lng))){ alert(i18n[currentLang].onlyPhuket); return; }
      if(!markerA) markerA = new google.maps.marker.AdvancedMarkerElement({map, position: loc});
      else markerA.position = loc;
      currentPos = { lat: loc.lat, lng: loc.lng };
      map.panTo(loc);
      if(markerB && markerB.position) computeRoutes(currentPos, markerB.position);
    } catch(err){
      console.error('Place fetch error', err); alert('ค้นหาสถานที่ไม่สำเร็จ');
    }
  });

  acEnd.addEventListener('gmpx-placechange', async ()=> {
    const text = acEnd.value;
    if(!text) return;
    try{
      const place = new google.maps.places.Place({ textQuery: text });
      const result = await place.fetchFields({ fields: ['location','name','formatted_address'] });
      const loc = result.location;
      if(!phuketBounds.contains(new google.maps.LatLng(loc.lat, loc.lng))){ alert(i18n[currentLang].onlyPhuket); return; }
      if(!markerB) markerB = new google.maps.marker.AdvancedMarkerElement({map, position: loc, title: result.name});
      else markerB.position = loc;
      map.panTo(loc);
      if(currentPos) computeRoutes(currentPos, loc);
    } catch(err){
      console.error('Place fetch error', err); alert('ค้นหาสถานที่ไม่สำเร็จ');
    }
  });

  // Try initial geolocation (non-blocking)
  if(navigator.geolocation){
    navigator.geolocation.getCurrentPosition(p=>{
      const lat = p.coords.latitude, lng = p.coords.longitude;
      const posLatLng = new google.maps.LatLng(lat,lng);
      if(!phuketBounds.contains(posLatLng)) return; // ignore outside phuket
      currentPos = {lat,lng};
      if(!markerA) markerA = new google.maps.marker.AdvancedMarkerElement({map, position: currentPos, title:'You'});
      else markerA.position = currentPos;
      map.setCenter(currentPos);
    }, ()=>{ /* ignore permission denied */ });
  }

  // UI wiring
  document.getElementById('btn-current').addEventListener('click', ()=> {
    if(!navigator.geolocation){ alert('Geolocation not supported'); return; }
    navigator.geolocation.getCurrentPosition(p=>{
      const lat = p.coords.latitude, lng = p.coords.longitude;
      const pos = new google.maps.LatLng(lat,lng);
      if(!phuketBounds.contains(pos)){ alert(i18n[currentLang].outsidePhuket); return; }
      currentPos = {lat,lng};
      if(!markerA) markerA = new google.maps.marker.AdvancedMarkerElement({map, position: currentPos, title:'คุณอยู่ที่นี่'});
      else markerA.position = currentPos;
      map.panTo(pos);
      if(markerB && markerB.position) computeRoutes(currentPos, markerB.position);
    }, ()=>{ alert('ไม่สามารถดึงตำแหน่งปัจจุบันได้'); });
  });

  document.getElementById('btn-calc').addEventListener('click', ()=>{
    const selVehicle = document.getElementById('vehicle').value || 'grabCar';
    if(lastRoutes && lastRoutes.length>0){
      updateRouteFareLabels();
      // update selected route fare pill
      const pill = document.querySelector('.route-card.selected .fare-pill');
      if(pill){
        const fare = calculateFare(lastRoutes[selectedRouteIndex], selVehicle);
        pill.textContent = `${fare} ฿`;
      }
      return;
    }
    if(currentPos && markerB && markerB.position){
      computeRoutes(currentPos, markerB.position);
    } else {
      alert('กรุณาเลือกต้นทางและปลายทางก่อน (หรือใช้ปุ่ม 📍)');
    }
  });

  document.getElementById('btn-reset').addEventListener('click', ()=>{
    document.getElementById('search').value=''; document.getElementById('searchStart').value='';
    lastRoutes = []; selectedRouteIndex = 0;
    document.getElementById('routesList').innerHTML = i18n[currentLang].noRoute;
    polyLines.forEach(p=>p.setMap(null)); polyLines = [];
    poiMarkers.forEach(m=>m.setMap(null)); poiMarkers = [];
    if(markerA){ markerA.setMap(null); markerA=null; }
    if(markerB){ markerB.setMap(null); markerB=null; }
  });

  document.getElementById('btn-toggle-fare').addEventListener('click', ()=>{
    const f = document.getElementById('fareTable');
    const visible = f.style.display !== 'none';
    f.style.display = visible ? 'none' : 'block';
    f.setAttribute('aria-hidden', visible ? 'true' : 'false');
  });

  document.getElementById('vehicle').addEventListener('change', updateRouteFareLabels);

  document.getElementById('btn-lang').addEventListener('click', ()=>{
    currentLang = (currentLang === 'th' ? 'en' : 'th');
    applyLanguage();
  });

  // initial UI
  applyLanguage();
}

/* ================================
   Compute routes using DirectionsService
   (modern API supports promise-like .route returning a result object)
   ================================ */
async function computeRoutes(origin, dest){
  const req = {
    origin,
    destination: dest,
    travelMode: google.maps.TravelMode.DRIVING,
    provideRouteAlternatives: true
  };
  try{
    const res = await directionsService.route(req); // modern return includes routes
    if(!res || !res.routes || res.routes.length===0) { alert('ไม่พบเส้นทาง'); return; }
    lastRoutes = res.routes;
    // clear old visuals
    polyLines.forEach(p=>p.setMap(null)); polyLines = [];
    renderRoutes(res.routes, document.getElementById('vehicle').value || 'grabCar');
  } catch(err){
    console.error('Directions error', err);
    alert('Directions request failed');
  }
}

/* ================================
   Render routes: draw polylines + create cards
   ================================ */
function renderRoutes(routes, vehicleKey){
  const container = document.getElementById('routesList'); container.innerHTML = '';
  if(!routes || routes.length===0){ container.textContent = i18n[currentLang].noRoute; return; }

  const colors = ['#2ecc71','#f1c40f','#e74c3c','#1e88e5','#9b59b6'];
  routes.forEach((r,i) => {
    // polyline
    const path = r.overview_path.map(p => ({lat:p.lat(), lng:p.lng()}));
    const poly = new google.maps.Polyline({
      path,
      map,
      strokeColor: colors[i % colors.length],
      strokeWeight: 6,
      strokeOpacity: 0.6
    });
    poly.routeIndex = i;
    polyLines.push(poly);
    poly.addListener('click', ()=> {
      selectedRouteIndex = poly.routeIndex;
      highlightSelected();
      // fit bounds if available
      if(r.bounds) map.fitBounds(r.bounds);
    });

    // route card
    const distText = `${km(r.legs[0].distance.value)} ${i18n[currentLang].kmLabel}`;
    const durText = `${Math.round(r.legs[0].duration.value/60)} ${i18n[currentLang].minsLabel}`;
    const fare = calculateFare(r, vehicleKey || document.getElementById('vehicle').value || 'grabCar');
    const card = document.createElement('div'); card.className = 'route-card';
    card.dataset.routeIndex = i;
    card.innerHTML = `<div style="display:flex;flex-direction:column;">
        <div style="font-weight:700">${i18n[currentLang].routeLabel} ${i+1}</div>
        <div class="small-muted">${distText} | ${durText}</div>
      </div>
      <div class="fare-pill">${fare} ฿</div>`;
    card.onclick = () => { selectedRouteIndex = i; drawPolyline(i); highlightSelected(); map.fitBounds(r.bounds || r.overview_path.reduce((b,p)=>{ b.extend(p); return b; }, new google.maps.LatLngBounds())); };
    container.appendChild(card);
  });

  // select first by default
  selectedRouteIndex = 0;
  drawPolyline(0);
  highlightSelected();
}

/* emphasize the selected route */
function drawPolyline(index){
  polyLines.forEach((p, idx) => {
    p.setOptions({strokeOpacity: idx === index ? 1.0 : 0.4, strokeWeight: idx === index ? 8 : 6});
  });
}

/* highlight route cards and update fares */
function highlightSelected(){
  document.querySelectorAll('.route-card').forEach((c,i)=>{
    c.classList.toggle('selected', i === selectedRouteIndex);
  });
  updateRouteFareLabels();
}

/* update fare labels in cards & fare table */
function updateRouteFareLabels(){
  const vehicleKey = document.getElementById('vehicle').value || 'grabCar';
  document.querySelectorAll('.route-card').forEach(card=>{
    const idx = parseInt(card.dataset.routeIndex, 10);
    if(isNaN(idx) || !lastRoutes[idx]) return;
    const newFare = calculateFare(lastRoutes[idx], vehicleKey);
    const pill = card.querySelector('.fare-pill');
    if(pill) pill.textContent = `${newFare} ฿`;
  });
}

/* ================================
   Text search fallback using Place class (async/await)
   ================================ */
async function triggerTextSearch(txt, target='end'){
  if(!txt) return;
  try{
    const place = new google.maps.places.Place({ textQuery: txt });
    const res = await place.fetchFields({ fields: ['location','name','formatted_address'] });
    const loc = res.location;
    if(!phuketBounds.contains(new google.maps.LatLng(loc.lat, loc.lng))){ alert(i18n[currentLang].onlyPhuket); return; }
    if(target==='end'){
      if(!markerB) markerB = new google.maps.marker.AdvancedMarkerElement({map, position: loc});
      else markerB.position = loc;
      map.panTo(loc);
      if(currentPos) computeRoutes(currentPos, loc);
    } else {
      if(!markerA) markerA = new google.maps.marker.AdvancedMarkerElement({map, position: loc});
      else markerA.position = loc;
      currentPos = {lat:loc.lat, lng:loc.lng};
      map.panTo(loc);
      if(markerB && markerB.position) computeRoutes(currentPos, markerB.position);
    }
  } catch(err){
    console.error('textSearch error', err);
    alert('ไม่พบผลการค้นหา');
  }
}

/* ================================
   Expose initMap for callback & misc
   ================================ */
window.initMap = initMap;
window.triggerTextSearch = triggerTextSearch;
window.calculateFare = calculateFare;
</script>

<!-- Load Google Maps JS (Beta) with marker, places, geometry, directions libraries.
     Replace YOUR_API_KEY_HERE with your key. Keep v=beta to enable AdvancedMarker & gmpx components. -->
<script async
  src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=marker,places,geometry,directions&v=beta&callback=initMap"></script>
</body>
</html>
