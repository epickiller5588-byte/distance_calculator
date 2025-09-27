<!doctype html>
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
html,body{height:100%;margin:0;font-family:Inter,system-ui,-apple-system,"Sarabun",sans-serif;background:var(--bg);color:#112;}
.wrap{display:flex;height:100vh;gap:12px;padding:18px;box-sizing:border-box;}
.sidebar{width:420px;min-width:320px;background:var(--card);border-radius:var(--rounded);box-shadow:0 8px 24px rgba(15,23,42,.08);padding:20px;display:flex;flex-direction:column;gap:12px;overflow:auto;}
.logo{font-weight:700;font-size:20px;display:flex;align-items:center;gap:8px;position:relative;}
#btn-lang{position:absolute; top:0; right:0;padding:6px 10px;font-size:12px;background:#1e88e5; color:#fff;border:none; border-radius:6px;cursor:pointer}
.search-row{display:flex;gap:8px;align-items:center}
#search,#searchStart{flex:1;padding:10px 12px;border-radius:10px;border:1px solid #e6e9ee;font-size:15px}
.btn{padding:9px 12px;border-radius:10px;border:0;background:var(--accent);color:#fff;cursor:pointer;font-size:14px}
.btn.alt{background:#eef; color:var(--accent); border:1px solid #d6e6ff}
.chips{display:flex;gap:8px;flex-wrap:wrap}
.chip{display:flex;gap:8px;align-items:center;padding:8px 10px;border-radius:10px;border:1px solid #eef;background:#fff;cursor:pointer;font-size:14px}
.section-title{font-weight:700;margin:6px 0;color:#0b2540}
.controls-row{display:flex;gap:8px;align-items:center}
select{padding:8px;border-radius:8px;border:1px solid #e6e9ee;background:#fff}
.routes{margin-top:8px;display:flex;flex-direction:column;gap:8px;overflow:auto;max-height:30vh;padding-right:6px}
.route-card{background:#fbfdff;border-radius:10px;padding:12px;border:1px solid #eef;cursor:pointer;display:flex;justify-content:space-between;align-items:center;flex-direction:column}
.route-card.selected{outline:3px solid rgba(30,136,229,.12)}
.route-left{display:flex;flex-direction:column;width:100%;}
.route-title{font-weight:700}
.route-meta{color:var(--muted);font-size:13px}
.fare-pill{font-weight:700;color:var(--accent)}
#poiList{margin-top:6px;display:flex;flex-direction:column;gap:6px;max-height:150px;overflow:auto}
.map-wrap{flex:1;position:relative;border-radius:var(--rounded);overflow:hidden;box-shadow:0 8px 24px rgba(15,23,42,.06);}
#map{width:100%;height:100%}
.fare-table{position:absolute;right:24px;bottom:24px;background:var(--card);padding:14px;border-radius:12px;box-shadow:0 8px 24px rgba(15,23,42,.12);min-width:220px;max-width:90vw;overflow:auto;z-index:10;}
.fare-table table{border-collapse:collapse;width:100%;font-size:14px}
.fare-table th{font-weight:700;text-align:left;padding:6px 4px;color:#123}
.fare-table td{padding:6px 4px;color:#334}
.legend{position:absolute;left:20px;bottom:24px;background:#fff;padding:8px;border-radius:8px;border:1px solid #eef;display:flex;gap:8px;align-items:center;font-size:13px;flex-wrap:wrap;z-index:10;}
.dot{width:36px;height:6px;border-radius:6px}
.fast{background:linear-gradient(90deg,#2ecc71,#1faa4a)}
.moderate{background:linear-gradient(90deg,#f1c40f,#f39c12)}
.heavy{background:linear-gradient(90deg,#e74c3c,#c0392b)}

/* ---------- Responsive ---------- */
@media(max-width:1000px){ 
  .wrap{flex-direction:column;padding:12px;} 
  .sidebar{width:auto;min-width:unset;max-height:46vh;overflow:auto; -webkit-overflow-scrolling: touch;} 
  .fare-table{right:12px;bottom:12px;min-width:150px;} 
  .controls-row{flex-direction:column;align-items:stretch;} 
  select{width:100%;}
  .routes{max-height:25vh;overflow:auto; -webkit-overflow-scrolling: touch;}
}

@media(max-width:600px){
  .sidebar{
    padding:12px;
    max-height:50vh;
    font-size:14px;
    overflow:auto;
    -webkit-overflow-scrolling: touch; /* smooth scroll iOS */
  }
  .logo{font-size:18px;}
  .btn, .btn.alt{font-size:13px;padding:10px;} /* touch target ใหญ่ขึ้น */
  #search,#searchStart{font-size:14px;padding:10px;}
  .route-card{padding:10px;}
  .routes{
    max-height:22vh;
    overflow:auto;
    -webkit-overflow-scrolling: touch;
  }
  .fare-table{
    bottom:10px;
    right:10px;
    min-width:120px;
    max-width:90vw;
    font-size:13px;
    padding:10px;
  }
  .legend{
    bottom:10px;
    left:10px;
    font-size:12px;
    padding:6px;
  }
  select{
    font-size:14px;
    padding:8px;
  }
}

</style>

</head>
<body>
<div class="wrap">
  <aside class="sidebar" aria-label="sidebar">
    <div class="logo">Trip Roule — Phuket demo
      <button id="btn-lang">🌐 English</button>
    </div>

    <div class="section-title" id="lbl-origin">ต้นทาง</div>
    <div class="search-row">
      <input id="searchStart" placeholder="เลือกต้นทาง..." />
     <button 
  id="btn-current" 
  class="btn alt" 
  title="ใช้ตำแหน่งปัจจุบัน" 
  aria-label="ใช้ตำแหน่งปัจจุบัน"
>
  📍
</button>

    </div>

    <div class="section-title" id="lbl-dest">ปลายทาง</div>
    <div class="search-row">
      <input id="search" placeholder="ค้นหาปลายทาง..." />
    </div>

    <div class="chips" id="popular"></div>

    <div class="controls-row">
      <select id="vehicle"></select>
      <button id="btn-calc" class="btn">คำนวณ</button>
      <button id="btn-reset" class="btn alt">รีเซ็ต</button>
    </div>

    <button id="btn-toggle-fare" class="btn alt">แสดง/ซ่อนตารางราคา</button>

    <div class="section-title" style="margin-top:12px" id="lbl-recommend">เส้นทางที่แนะนำ</div>
    <div class="routes" id="routesList"></div>
  </aside>

  <main class="map-wrap">
    <div id="map"></div>

    <div class="fare-table" id="fareTable">
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
// ------------------ Data ------------------
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

// ------------------ i18n ------------------
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

let currentLang = "th";

// ------------------ Utilities ------------------
function km(m){ return (m/1000).toFixed(1); }
function calculateFare(route, vehicleKey){
  const v = vehicleRates[vehicleKey];
  if(!v) return 0;
  if(v.fixed) return v.min;
  const kmDistance = route.legs[0].distance.value / 1000;
  let fare = v.base + Math.max(0, kmDistance - v.freeKm) * v.perKm;
  return Math.max(Math.round(fare), v.min);
}
// ------------------ UI: language & fare table ------------------
function applyLanguage(){
  document.getElementById("btn-lang").textContent = (currentLang==="th" ? "🌐 English" : "🌐 ภาษาไทย");
  document.getElementById("search").placeholder = i18n[currentLang].search;
  document.getElementById("searchStart").placeholder = i18n[currentLang].start;
  document.getElementById("btn-calc").textContent = i18n[currentLang].calc;
  document.getElementById("btn-reset").textContent = i18n[currentLang].reset;
  document.getElementById("fare-title").textContent = i18n[currentLang].fareTable;
  document.getElementById("lbl-origin").textContent = i18n[currentLang].originLabel;
  document.getElementById("lbl-dest").textContent = i18n[currentLang].search;
  document.getElementById("lbl-recommend").textContent = i18n[currentLang].routeLabel;
  document.getElementById("legend-title").textContent = i18n[currentLang].traffic;
  document.getElementById("legend-fast").textContent = i18n[currentLang].fast;
  document.getElementById("legend-moderate").textContent = i18n[currentLang].moderate;
  document.getElementById("legend-heavy").textContent = i18n[currentLang].heavy;
  document.getElementById("th-vehicle").textContent = i18n[currentLang].vehicleCol;
  document.getElementById("th-base").textContent = i18n[currentLang].baseCol;
  document.getElementById("th-perkm").textContent = i18n[currentLang].perKmCol;
  document.getElementById("btn-toggle-fare").textContent = i18n[currentLang].toggleFare;

  const btnCurrent = document.getElementById('btn-current');
btnCurrent.title = i18n[currentLang].currentPosBtn;
btnCurrent.setAttribute('aria-label', i18n[currentLang].currentPosBtn);


  // populate vehicle select with placeholder
const sel = document.getElementById("vehicle");
sel.innerHTML = "";

// เพิ่ม placeholder เป็น option disabled
const placeholder = document.createElement("option");
placeholder.textContent = (currentLang === "th" ? "เลือกประเภทยานพาหนะ" : "Select vehicle type");
placeholder.disabled = true;
placeholder.selected = true;
sel.appendChild(placeholder);

// เพิ่มตัวเลือกรถ
for(const k in vehicleRates){
  const opt = document.createElement("option");
  opt.value = k;
  opt.textContent = vehicleNames[currentLang][k] || k;
  sel.appendChild(opt);
}


  // popular chips
  const pop = document.getElementById("popular"); pop.innerHTML = "";
  i18n[currentLang].popular.forEach(p => {
    const el = document.createElement("div"); el.className="chip"; el.textContent = p;
    el.onclick = ()=>{ document.getElementById('search').value = p; triggerTextSearch(p); };
    pop.appendChild(el);
  });

  // ensure routes list placeholder
  if(!lastRoutes || lastRoutes.length===0){
    document.getElementById('routesList').textContent = i18n[currentLang].noRoute;
  }

  renderFareTable();
}

function renderFareTable(){
  const tbody = document.getElementById('fareRows'); tbody.innerHTML = '';
  for(const k in vehicleRates){
    const v = vehicleRates[k];
    const name = vehicleNames[currentLang][k] || k;
    const row = document.createElement('tr');
    if(v.fixed){
      row.innerHTML = `<td>${name}</td><td>${v.min} ฿ (fixed)</td><td>-</td>`;
    } else {
      row.innerHTML = `<td>${name}</td><td>${v.base} ฿</td><td>${v.perKm} ฿</td>`;
    }
    tbody.appendChild(row);
  }
  document.getElementById('fareTable').style.display='block';
}

// ------------------ Map & routing ------------------
let map, markerA = null, markerB = null, directionsService, directionsRenderer, trafficLayer, placesService;
let currentPos = null, lastRoutes = [], selectedRouteIndex = 0, poiMarkers = [], polyLines = [];  

function initMap(){
  const phuketBounds = new google.maps.LatLngBounds({lat:7.7,lng:98.2},{lat:8.1,lng:98.6});
  const center = {lat:7.8804,lng:98.3923};
  map = new google.maps.Map(document.getElementById('map'), {center, zoom:11, mapTypeControl:false, streetViewControl:false});

  directionsService = new google.maps.DirectionsService();
  directionsRenderer = new google.maps.DirectionsRenderer({map, suppressMarkers:true, preserveViewport:true});
  trafficLayer = new google.maps.TrafficLayer(); trafficLayer.setMap(map);
  placesService = new google.maps.places.PlacesService(map);

  // Autocomplete start
  const autocompleteStart = new google.maps.places.Autocomplete(document.getElementById('searchStart'), {
    bounds: phuketBounds, componentRestrictions:{country:'th'}, fields:['place_id','geometry','name','formatted_address']
  });
  autocompleteStart.addListener('place_changed', () => {
    const place = autocompleteStart.getPlace();
    if(place?.geometry?.location){
      const loc = place.geometry.location;
      if(!phuketBounds.contains(loc)){ alert(i18n[currentLang].onlyPhuket); return; }
      if(!markerA){ markerA = new google.maps.Marker({map, icon:{path:google.maps.SymbolPath.CIRCLE,scale:8,fillColor:'#1e88e5',fillOpacity:1,strokeWeight:0}}); }
      markerA.setPosition(loc); markerA.setMap(map);
      map.panTo(loc);
      currentPos = {lat:loc.lat(), lng:loc.lng()};
      if(markerB && markerB.getPosition()) computeRoutes(currentPos, markerB.getPosition());
    }
  });

  // Autocomplete end
  const autocompleteEnd = new google.maps.places.Autocomplete(document.getElementById('search'), {
    bounds: phuketBounds, componentRestrictions:{country:'th'}, fields:['place_id','geometry','name','formatted_address']
  });
  autocompleteEnd.addListener('place_changed', () => {
    const place = autocompleteEnd.getPlace();
    if(place?.geometry?.location){
      const loc = place.geometry.location;
      if(!phuketBounds.contains(loc)){ alert(i18n[currentLang].onlyPhuket); return; }
      if(!markerB){ markerB = new google.maps.Marker({map, icon:'http://maps.google.com/mapfiles/ms/icons/red-dot.png'}); }
      markerB.setPosition(loc); markerB.setMap(map);
      map.panTo(loc);
      if(currentPos) computeRoutes(currentPos, loc);
    } else {
      // free-text fallback
      triggerTextSearch(document.getElementById('search').value);
    }
  });

  // try geolocation
  if(navigator.geolocation){
    navigator.geolocation.getCurrentPosition(p=>{
      currentPos = {lat:p.coords.latitude, lng:p.coords.longitude};
      const posLatLng = new google.maps.LatLng(currentPos.lat, currentPos.lng);
      if(!phuketBounds.contains(posLatLng)) return; // if outside phuket, ignore
      if(!markerA){ markerA = new google.maps.Marker({map, icon:{path:google.maps.SymbolPath.CIRCLE,scale:8,fillColor:'#1e88e5',fillOpacity:1,strokeWeight:0}}); }
      markerA.setPosition(posLatLng); markerA.setMap(map);
      markerA.setTitle('You');
      map.setCenter(posLatLng);
    }, err => {
      // ignore geolocation failure quietly
    });
  }

  // current button
  document.getElementById('btn-current').addEventListener('click', () => {
    if(navigator.geolocation){
      navigator.geolocation.getCurrentPosition(p=>{
        currentPos = {lat:p.coords.latitude, lng:p.coords.longitude};
        const posLatLng = new google.maps.LatLng(currentPos.lat, currentPos.lng);
        if(!phuketBounds.contains(posLatLng)){ alert(i18n[currentLang].outsidePhuket); return; }
        if(!markerA){ markerA = new google.maps.Marker({map, icon:{path:google.maps.SymbolPath.CIRCLE,scale:8,fillColor:'#1e88e5',fillOpacity:1,strokeWeight:0}}); }
        markerA.setPosition(posLatLng); markerA.setMap(map);
        map.panTo(posLatLng);
        if(markerB && markerB.getPosition()) computeRoutes(currentPos, markerB.getPosition());
      }, err => { alert('Unable to retrieve your location'); });
    } else alert('Geolocation not supported');
  });

  // calc button
  document.getElementById('btn-calc').addEventListener('click', () => {
    if(currentPos && markerB && markerB.getPosition()) computeRoutes(currentPos, markerB.getPosition());
    renderFareTable();
  });

  // reset button
  document.getElementById('btn-reset').addEventListener('click', () => {
    document.getElementById('search').value=''; document.getElementById('searchStart').value='';
    directionsRenderer.set('directions', null);
    lastRoutes = []; selectedRouteIndex = 0;
    document.getElementById('routesList').textContent = i18n[currentLang].noRoute;

    polyLines.forEach(p=>p.setMap(null)); polyLines = [];
    poiMarkers.forEach(m=>m.setMap(null)); poiMarkers = [];

    if(markerA){ markerA.setMap(null); markerA = null; }
    if(markerB){ markerB.setMap(null); markerB = null; }
  });

  // toggle fare table
  document.getElementById('btn-toggle-fare').addEventListener('click', () => {
    const f = document.getElementById('fareTable');
    f.style.display = (f.style.display === 'none' ? 'block' : 'none');
  });

  // vehicle change: update fare labels in route cards without recomputing route
  document.getElementById('vehicle').addEventListener('change', updateRouteFareLabels);

  applyLanguage();
}

// ------------------ Places text search fallback ------------------
function triggerTextSearch(txt){
  if(!txt) return;
  const service = new google.maps.places.PlacesService(map);
  service.textSearch({query:txt, bounds: map.getBounds(), region:'th'}, (results,status)=>{
    if(status==='OK' && results[0]){
      const loc = results[0].geometry.location;
      if(!markerB){ markerB = new google.maps.Marker({map, icon:'http://maps.google.com/mapfiles/ms/icons/red-dot.png'}); }
      markerB.setPosition(loc); markerB.setMap(map);
      map.panTo(loc);
      if(currentPos) computeRoutes(currentPos, loc);
    } else {
      alert('No results found');
    }
  });
}

// ------------------ Routing ------------------
function computeRoutes(origin, dest){
  const selectedVehicle = document.getElementById('vehicle').value;
  directionsService.route({
    origin,
    destination: dest,
    travelMode: google.maps.TravelMode.DRIVING,
    provideRouteAlternatives: true
  }, (res,status) => {
    if(status === 'OK'){
      // clear existing visuals
      directionsRenderer.set('directions', null);
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

  routes.forEach((r,i) => {
    const card = document.createElement('div'); card.className='route-card';
    card.dataset.routeIndex = i;
    const distText = `${km(r.legs[0].distance.value)} ${i18n[currentLang].kmLabel}`;
    const durText = `${Math.round(r.legs[0].duration.value/60)} ${i18n[currentLang].minsLabel}`;
    const fareText = `${calculateFare(r, vehicleKey)} ฿`;
    card.innerHTML = `<div class="route-left">
        <div class="route-title">${i18n[currentLang].routeLabel} ${i+1}</div>
        <div class="route-meta">${distText} | ${durText}</div>
        <div class="fare-pill">${fareText}</div>
      </div>`;
    card.onclick = () => { selectedRouteIndex = i; drawPolyline(routes[i], i); highlightSelected(); };
    container.appendChild(card);
  });

  // draw first route by default
  selectedRouteIndex = 0;
  drawPolyline(routes[0], 0);
  highlightSelected();
}

// draw polyline (use index to pick color consistently)
function drawPolyline(route, index){
  polyLines.forEach(p=>p.setMap(null)); polyLines=[];
  const colors = ['#2ecc71','#f1c40f','#e74c3c','#1e88e5','#9b59b6'];
  const color = colors[index % colors.length];
  const pl = new google.maps.Polyline({
    path: route.overview_path,
    map: map,
    strokeColor: color,
    strokeWeight: 6,
    strokeOpacity: 0.8  // <-- corrected property
  });
  polyLines.push(pl);
  map.fitBounds(route.bounds);
}

// highlight selected card
function highlightSelected(){
  document.querySelectorAll('.route-card').forEach((c,i)=>{ c.classList.toggle('selected', i===selectedRouteIndex); });
}

// update fare labels in route cards when vehicle selection changes
function updateRouteFareLabels(){
  const vehicleKey = document.getElementById('vehicle').value;
  document.querySelectorAll('.route-card').forEach(card=>{
    const idx = parseInt(card.dataset.routeIndex, 10);
    if(isNaN(idx) || !lastRoutes[idx]) return;
    const newFare = calculateFare(lastRoutes[idx], vehicleKey);
    const pill = card.querySelector('.fare-pill');
    if(pill) pill.textContent = `${newFare} ฿`;
  });
}

// initialize lang button and load map script callback
document.getElementById('btn-lang').addEventListener('click', ()=>{
  currentLang = (currentLang === 'th' ? 'en' : 'th');
  applyLanguage();
});

// expose initMap for Google callback
window.initMap = initMap;
window.applyLanguage = applyLanguage;
window.calculateFare = calculateFare;
</script>

<!-- Replace YOUR_API_KEY_HERE with your Google Maps API key (restrict it appropriately in Cloud Console) -->
<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places&callback=initMap" async defer></script>
</body>
</html>

