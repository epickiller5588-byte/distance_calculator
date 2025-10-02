
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
html,body{
  height:100%;
  margin:0;
  font-family:Inter,system-ui,-apple-system,"Sarabun",sans-serif;
  background:var(--bg);
  color:#112;
  overflow:hidden;
}
/* Layout */
.wrap{
  display:flex;
  height:100vh;
  gap:12px;
  padding:8px;
  box-sizing:border-box;
  flex-direction:column;
}
.map-wrap{
  flex:1;
  position:relative;
  border-radius:var(--rounded);
  overflow:hidden;
  box-shadow:0 8px 24px rgba(15,23,42,.06);
  min-height:50vh;
}
#map{width:100%;height:100%}
.sidebar{
  width:100%;
  max-height:45vh;
  background:var(--card);
  border-radius:var(--rounded);
  box-shadow:0 8px 24px rgba(15,23,42,.08);
  padding:12px;
  display:flex;
  flex-direction:column;
  gap:10px;
  overflow-y:auto;
  scrollbar-width:thin;
  scrollbar-color:var(--accent) #e6e9ee;
}
.sidebar::-webkit-scrollbar{
  width:6px;
}
.sidebar::-webkit-scrollbar-track{
  background:#e6e9ee;
  border-radius:6px;
}
.sidebar::-webkit-scrollbar-thumb{
  background:var(--accent);
  border-radius:6px;
}
.logo{
  font-weight:700;
  font-size:18px;
  display:flex;
  align-items:center;
  gap:8px;
  position:relative;
  margin-bottom:4px;
}
#btn-lang{
  position:absolute;
  top:0;
  right:0;
  padding:6px 10px;
  font-size:12px;
  background:var(--accent);
  color:#fff;
  border:none;
  border-radius:6px;
  cursor:pointer;
}
.search-row{
  display:flex;
  gap:8px;
  align-items:center;
}
#search,#searchStart{
  flex:1;
  padding:10px 12px;
  border-radius:10px;
  border:1px solid #e6e9ee;
  font-size:15px;
  outline:none;
}
#search:focus,#searchStart:focus{border-color:var(--accent);}
.btn{
  padding:9px 12px;
  border-radius:10px;
  border:0;
  background:var(--accent);
  color:#fff;
  cursor:pointer;
  font-size:14px;
  flex-shrink:0;
}
.btn.alt{
  background:#eef;
  color:var(--accent);
  border:1px solid #d6e6ff;
}
.controls-row{
  display:flex;
  gap:8px;
  align-items:center;
  flex-wrap:wrap;
}
select{
  padding:8px;
  border-radius:8px;
  border:1px solid #e6e9ee;
  background:#fff;
  flex:1;
}
.routes{
  margin-top:8px;
  display:flex;
  flex-direction:column;
  gap:8px;
  overflow-y:auto;
  max-height:20vh;
  padding-right:4px;
}
.route-card{
  background:#fbfdff;
  border-radius:10px;
  padding:10px;
  border:1px solid #eef;
  cursor:pointer;
  display:flex;
  justify-content:space-between;
  align-items:center;
}
.route-card.selected{outline:3px solid rgba(30,136,229,.12);}
.route-left{display:flex;flex-direction:column;width:100%;}
.route-title{font-weight:700}
.route-meta{color:var(--muted);font-size:13px}
.fare-pill{font-weight:700;color:var(--accent)}
.fare-table{
  position:absolute;
  right:12px;
  bottom:12px;
  background:var(--card);
  padding:12px;
  border-radius:12px;
  box-shadow:0 8px 24px rgba(15,23,42,.12);
  min-width:200px;
  max-width:90vw;
  overflow:auto;
  z-index:10;
  display:none;
  font-size:14px;
}
.fare-table table{border-collapse:collapse;width:100%}
.fare-table th{font-weight:700;text-align:left;padding:6px 4px;color:#123}
.fare-table td{padding:6px 4px;color:#334}
.legend{
  position:absolute;
  left:12px;
  bottom:12px;
  background:#fff;
  padding:8px;
  border-radius:8px;
  border:1px solid #eef;
  display:flex;
  gap:6px;
  align-items:center;
  font-size:12px;
  flex-wrap:wrap;
  z-index:10;
}
.dot{width:36px;height:6px;border-radius:6px}
.fast{background:linear-gradient(90deg,#2ecc71,#1faa4a)}
.moderate{background:linear-gradient(90deg,#f1c40f,#f39c12)}
.heavy{background:linear-gradient(90deg,#e74c3c,#c0392b)}
@media(min-width:1000px){
  .wrap{flex-direction:row;}
  .map-wrap{height:auto;flex:1;}
  .sidebar{width:420px;max-height:100%;overflow:auto;}
  .routes{max-height:30vh;}
  .fare-table{right:18px;bottom:18px;}
  .legend{left:18px;bottom:18px;}
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
/* -------------- Data -------------- */
const vehicleRates = {
  grabCar:{base:35,perKm:10,freeKm:2,min:50},
  grabBike:{base:20,perKm:7,freeKm:1,min:25},
  boltEconomy:{base:50,perKm:10,freeKm:2,min:100},
  boltStandard:{base:100,perKm:12,freeKm:2,min:200},
  boltVan:{base:250,perKm:15,freeKm:2,min:300},
  boltXL:{base:200,perKm:15,freeKm:2,min:300},
  boltTaxi:{base:100,perKm:12,freeKm:2,min:200},
  inDrive:{base:30,perKm:9,freeKm:1,min:80},
  taxiMeter:{base:50,perKm:12,freeKm:2,min:100},
  smartBus:{base:100,perKm:0,freeKm:0,min:100,fixed:true},
  songthaew:{base:40,perKm:0,freeKm:0,min:40,fixed:true},
  pinkBus:{base:15,perKm:0,freeKm:0,min:15,fixed:true},
  tukTuk:{base:200,perKm:0,freeKm:0,min:200,fixed:true},
  motoTaxi:{base:80,perKm:0,freeKm:0,min:80,fixed:true}
};

const vehicleNames = {
  th:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard", boltVan:"Bolt Van", boltXL:"Bolt XL", boltTaxi:"Bolt Taxi", inDrive:"inDrive", taxiMeter:"Taxi Meter", smartBus:"รถบัส Smart Bus", songthaew:"รถสองแถว", pinkBus:"รถบัสสีชมพู", tukTuk:"ตุ๊กตุ๊ก", motoTaxi:"มอเตอร์ไซค์รับจ้าง"},
  en:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard", boltVan:"Bolt Van", boltXL:"Bolt XL", boltTaxi:"Bolt Taxi", inDrive:"inDrive", taxiMeter:"Taxi Meter", smartBus:"Smart Bus", songthaew:"Songthaew", pinkBus:"Pink Bus", tukTuk:"Tuk Tuk", motoTaxi:"Moto Taxi"}
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
    toggleFare: "Show/Hide Fare Table",
  }
};

let currentLang = 'th';

/* -------------- State -------------- */
let map, directionsService, trafficLayer, placesService;
let markerA = null, markerB = null;
let currentPos = null, lastRoutes = [], selectedRouteIndex = 0, polyLines = [], poiMarkers = [];

/* -------------- Helpers -------------- */
function km(m){ return (m/1000).toFixed(1); }

function calculateFare(route, vehicleKey){
  const v = vehicleRates[vehicleKey];
  if(!v) return 0;
  if(v.fixed) return v.min;
  const kmDistance = route.legs[0].distance.value / 1000;
  let fare = v.base + Math.max(0, kmDistance - v.freeKm) * v.perKm;
  return Math.max(Math.round(fare), v.min);
}

/* -------------- UI: language, populate vehicle, fares -------------- */
function applyLanguage(){
  document.getElementById("btn-lang").textContent = (currentLang==="th" ? "🌐 English" : "🌐 ภาษาไทย");
  document.getElementById("search").placeholder = i18n[currentLang].search;
  document.getElementById("searchStart").placeholder = i18n[currentLang].start;
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

  // populate vehicle select & set default to first usable option (avoid empty value bug)
  const sel = document.getElementById("vehicle"); sel.innerHTML = "";
  // placeholder (optional, not selected)
  const ph = document.createElement("option"); ph.value = ""; ph.textContent = (currentLang==='th'?'-- เลือกประเภทยานพาหนะ --':'-- Select vehicle --'); ph.disabled = true;
  sel.appendChild(ph);
  for(const k in vehicleRates){
    const opt = document.createElement("option");
    opt.value = k;
    opt.textContent = (vehicleNames[currentLang] && vehicleNames[currentLang][k]) ? vehicleNames[currentLang][k] : k;
    sel.appendChild(opt);
  }
  // set a sensible default so price calculation works immediately
  sel.value = sel.querySelector('option[value="grabCar"]') ? 'grabCar' : sel.querySelector('option:not([disabled])').value;


  // ensure routes placeholder
  if(!lastRoutes || lastRoutes.length === 0){
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
  // leave displayed state controlled by toggle (initially hidden to reduce clutter on mobile)
  document.getElementById('fareTable').style.display = 'none';
}

/* -------------- Map & Places -------------- */
function initMap(){
  const phuketBounds = new google.maps.LatLngBounds({lat:7.7,lng:98.2},{lat:8.1,lng:98.6});
  const center = {lat:7.8804,lng:98.3923};
  map = new google.maps.Map(document.getElementById('map'), {center, zoom:11, mapTypeControl:false, streetViewControl:false});
  directionsService = new google.maps.DirectionsService();
  trafficLayer = new google.maps.TrafficLayer(); trafficLayer.setMap(map);
  placesService = new google.maps.places.PlacesService(map);

  // Autocomplete for start
  const acStart = new google.maps.places.Autocomplete(document.getElementById('searchStart'), {
    bounds: phuketBounds, componentRestrictions:{country:'th'}, fields:['place_id','geometry','name','formatted_address']
  });
  acStart.addListener('place_changed', () => {
    const place = acStart.getPlace();
    if(place?.geometry?.location){
      const loc = place.geometry.location;
      if(!phuketBounds.contains(loc)){ alert(i18n[currentLang].onlyPhuket); return; }
      if(!markerA) markerA = new google.maps.Marker({map, icon:{path:google.maps.SymbolPath.CIRCLE,scale:8,fillColor:'#1e88e5',fillOpacity:1,strokeWeight:0}});
      markerA.setPosition(loc); markerA.setMap(map);
      map.panTo(loc);
      currentPos = {lat:loc.lat(), lng:loc.lng()};
      if(markerB && markerB.getPosition()) computeRoutes(currentPos, markerB.getPosition());
    }
  });

  // Autocomplete for end
  const acEnd = new google.maps.places.Autocomplete(document.getElementById('search'), {
    bounds: phuketBounds, componentRestrictions:{country:'th'}, fields:['place_id','geometry','name','formatted_address']
  });
  acEnd.addListener('place_changed', () => {
    const place = acEnd.getPlace();
    if(place?.geometry?.location){
      const loc = place.geometry.location;
      if(!phuketBounds.contains(loc)){ alert(i18n[currentLang].onlyPhuket); return; }
      if(!markerB) markerB = new google.maps.Marker({map, icon:'http://maps.google.com/mapfiles/ms/icons/red-dot.png'});
      markerB.setPosition(loc); markerB.setMap(map);
      map.panTo(loc);
      if(currentPos) computeRoutes(currentPos, loc);
    } else {
      // fallback to text search
      triggerTextSearch(document.getElementById('search').value);
    }
  });

  // try initial geolocation (non-blocking)
  if(navigator.geolocation){
    navigator.geolocation.getCurrentPosition(p=>{
      currentPos = {lat:p.coords.latitude, lng:p.coords.longitude};
      const posLatLng = new google.maps.LatLng(currentPos.lat, currentPos.lng);
      if(!phuketBounds.contains(posLatLng)) return;
      if(!markerA) markerA = new google.maps.Marker({map, icon:{path:google.maps.SymbolPath.CIRCLE,scale:8,fillColor:'#1e88e5',fillOpacity:1,strokeWeight:0}});
      markerA.setPosition(posLatLng); markerA.setMap(map); markerA.setTitle('You');
      map.setCenter(posLatLng);
    },()=>{ /* ignore */ });
  }

  // wire UI buttons
  document.getElementById('btn-current').addEventListener('click', ()=> {
    if(!navigator.geolocation){ alert('Geolocation not supported'); return; }
    navigator.geolocation.getCurrentPosition(p=>{
      currentPos = {lat:p.coords.latitude, lng:p.coords.longitude};
      const pos = new google.maps.LatLng(currentPos.lat, currentPos.lng);
      if(!phuketBounds.contains(pos)){ alert(i18n[currentLang].outsidePhuket); return; }
      if(!markerA) markerA = new google.maps.Marker({map, icon:{path:google.maps.SymbolPath.CIRCLE,scale:8,fillColor:'#1e88e5',fillOpacity:1,strokeWeight:0}});
      markerA.setPosition(pos); markerA.setMap(map); map.panTo(pos);
      if(markerB && markerB.getPosition()) computeRoutes(currentPos, markerB.getPosition());
    }, ()=>{ alert('Unable to retrieve your location'); });
  });

  document.getElementById('btn-calc').addEventListener('click', ()=>{
    // If we already have routes, just update fares; else compute if both markers present
    const selVehicle = document.getElementById('vehicle').value;
    if(lastRoutes && lastRoutes.length>0){
      updateRouteFareLabels(); // refresh prices in cards
      // Also update fare shown for selected route
      const pill = document.querySelector('.route-card.selected .fare-pill');
      if(pill){
        const idx = selectedRouteIndex;
        const fare = calculateFare(lastRoutes[idx], selVehicle || 'grabCar');
        pill.textContent = `${fare} ฿`;
      }
      return;
    }
    if(currentPos && markerB && markerB.getPosition()){
      computeRoutes(currentPos, markerB.getPosition());
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

  applyLanguage();
}

/* -------------- Places text fallback -------------- */
function triggerTextSearch(txt){
  if(!txt) return;
  const service = new google.maps.places.PlacesService(map);
  service.textSearch({query:txt, bounds: map.getBounds(), region:'th'}, (results,status)=>{
    if(status==='OK' && results[0]){
      const loc = results[0].geometry.location;
      if(!markerB) markerB = new google.maps.Marker({map, icon:'http://maps.google.com/mapfiles/ms/icons/red-dot.png'});
      markerB.setPosition(loc); markerB.setMap(map);
      map.panTo(loc);
      if(currentPos) computeRoutes(currentPos, loc);
    } else {
      alert('No results found');
    }
  });
}

/* -------------- Routing & render -------------- */
function computeRoutes(origin, dest){
  const selectedVehicle = document.getElementById('vehicle').value || 'grabCar';
  directionsService.route({
    origin: origin,
    destination: dest,
    travelMode: google.maps.TravelMode.DRIVING,
    provideRouteAlternatives: true
  }, (res,status) => {
    if(status === 'OK'){
      // remove old visuals
      polyLines.forEach(p=>p.setMap(null)); polyLines=[];
      lastRoutes = res.routes;
      renderRoutes(res.routes, selectedVehicle);
    } else {
      alert('Directions request failed: ' + status);
    }
  });
}

function renderRoutes(routes, vehicleKey){
  const container = document.getElementById('routesList'); container.innerHTML = '';
  if(!routes || routes.length===0){
    container.textContent = i18n[currentLang].noRoute; return;
  }

  // draw polylines and create cards
  const colors = ['#2ecc71','#f1c40f','#e74c3c','#1e88e5','#9b59b6'];
  routes.forEach((r,i) => {
    // polyline
    const path = r.overview_path.map(p => ({lat:p.lat(), lng:p.lng()}));
    const poly = new google.maps.Polyline({
      path: path,
      map: map,
      strokeColor: colors[i % colors.length],
      strokeWeight: 6,
      strokeOpacity: 0.6
    });
    poly.routeIndex = i;
    polyLines.push(poly);
    poly.addListener('click', ()=> {
      selectedRouteIndex = poly.routeIndex;
      highlightSelected();
      map.fitBounds(routes[selectedRouteIndex].bounds);
    });

    // card
    const card = document.createElement('div'); card.className='route-card';
    card.dataset.routeIndex = i;
    const distText = `${km(r.legs[0].distance.value)} ${i18n[currentLang].kmLabel}`;
    const durText = `${Math.round(r.legs[0].duration.value/60)} ${i18n[currentLang].minsLabel}`;
    const fare = calculateFare(r, vehicleKey || document.getElementById('vehicle').value || 'grabCar');
    card.innerHTML = `<div class="route-left">
        <div class="route-title">${i18n[currentLang].routeLabel} ${i+1}</div>
        <div class="route-meta">${distText} | ${durText}</div>
      </div>
      <div class="fare-pill">${fare} ฿</div>`;
    card.onclick = () => { selectedRouteIndex = i; drawPolyline(routes[i], i); highlightSelected(); map.fitBounds(routes[i].bounds); };
    container.appendChild(card);
  });

  // default select first
  selectedRouteIndex = 0;
  drawPolyline(routes[0], 0);
  highlightSelected();
}

/* emphasize the selected route, dim others */
function drawPolyline(route, index){
  // set all polylines opacity to dim
  polyLines.forEach((p, idx) => {
    p.setOptions({strokeOpacity: idx === index ? 1.0 : 0.4, strokeWeight: idx === index ? 8 : 6});
  });
}

/* highlight card matching selectedRouteIndex */
function highlightSelected(){
  document.querySelectorAll('.route-card').forEach((c,i)=>{
    c.classList.toggle('selected', i === selectedRouteIndex);
  });
  // ensure fare pills updated for current vehicle
  updateRouteFareLabels();
}

/* update fare text in route cards and fareTable summary */
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

/* -------------- Init wiring -------------- */
document.getElementById('btn-lang').addEventListener('click', ()=>{
  currentLang = (currentLang === 'th' ? 'en' : 'th');
  applyLanguage();
});

/* expose some functions for debugging if needed */
window.initMap = initMap;
window.calculateFare = calculateFare;
window.triggerTextSearch = triggerTextSearch;
</script>

<!-- Replace YOUR_API_KEY_HERE with your Google Maps JS key (restrict it appropriately) -->
<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places&callback=initMap" async defer></script>
</body>
</html>
