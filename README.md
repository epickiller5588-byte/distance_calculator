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
/* Base responsive typography */
body {
  font-size: clamp(14px, 2vw, 16px);
  line-height: 1.4;
}

/* Map ครึ่งจอเต็ม */
.map-wrap {
  order: -1;
  width: 100%;
  height: 55vh;
  margin-bottom: 10px;
}

/* Traffic Legend เล็กลง */
.legend {
  font-size: 11px;
  padding: 4px 6px;
  background: rgba(255,255,255,0.85);
  border-radius: 8px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.1);
  bottom: 8px;
  left: 8px;
}

/* ปุ่มไม่เกะกะ */
.controls-row {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}
.controls-row .btn {
  flex: 1;
  min-width: 100px;
  font-size: 14px;
  padding: 8px 10px;
  border-radius: 8px;
}

/* Sidebar ไม่ยาวเกิน */
.sidebar {
  width: auto;
  max-height: 45vh;
  overflow-y: auto;
  padding: 12px;
}
input, select, button {
  font-size: clamp(13px, 2.5vw, 15px);
  padding: clamp(8px, 2vw, 12px);
}

/* Scroll sidebar smoothly on mobile */
.sidebar {
  -webkit-overflow-scrolling: touch;
}

/* Responsive */
@media(max-width:1000px){ 
  .wrap{flex-direction:column;padding:12px;} 
  .map-wrap{order:-1;height:50vh;margin-bottom:12px;}
  .sidebar{width:auto;min-width:unset;max-height:50vh;overflow-y:auto;}
  .fare-table{right:12px;bottom:12px;min-width:150px;} 
  .controls-row{flex-direction:column;align-items:stretch;} 
  select, .btn{width:100%;}
  .routes{max-height:25vh;}
}

@media(max-width:600px){
  .map-wrap{height:42vh;}
  .sidebar{max-height:58vh;font-size:14px;padding:12px;}
  .search-row{flex-direction:column;gap:6px;}
  #search,#searchStart{width:100%;}
  .btn,.btn.alt{font-size:13px;padding:10px;width:100%;}
  .routes{max-height:22vh;}
  .fare-table{bottom:10px; right:10px; min-width:120px; max-width:90vw; font-size:13px; padding:10px;}
  .legend{bottom:10px; left:10px; font-size:12px; padding:6px;}
}

@media(max-width:480px){
  .map-wrap{height:38vh;}
  .sidebar{max-height:60vh;padding:10px;font-size:13px;}
  .logo{font-size:16px;}
  .btn,.btn.alt{font-size:12px;padding:9px;}
  .chip{font-size:12px;padding:6px 8px;}
  .route-card{padding:10px;}
}

</style>
</head>
<body>
<div class="wrap">
  <main class="map-wrap">
    <div id="map"></div>
    <div class="fare-table" id="fareTable">
      <div id="fare-title" style="font-weight:800;margin-bottom:6px">ตารางราคา</div>
      <table>
        <thead>
          <tr><th id="th-vehicle">Vehicle</th><th id="th-base">Base</th><th id="th-perkm">Per km</th></tr>
        </thead>
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

  <aside class="sidebar" aria-label="sidebar">
    <button id="btn-lang">🌐 English</button>

    <div class="section-title" id="lbl-origin">ต้นทาง</div>
    <div class="search-row">
      <input id="searchStart" placeholder="เลือกต้นทาง..." />
      <button id="btn-current" class="btn alt" title="ใช้ตำแหน่งปัจจุบัน">📍</button>
    </div>

    <div class="section-title" id="lbl-dest">ปลายทาง</div>
    <div class="search-row">
      <input id="search" placeholder="ค้นหาปลายทาง..." />
    </div>
    
    <div class="controls-row">
      <select id="vehicle"></select>
      <button id="btn-calc" class="btn">คำนวณ</button>
      <button id="btn-reset" class="btn alt">รีเซ็ต</button>
    </div>

    <button id="btn-toggle-fare" class="btn alt">แสดง/ซ่อนตารางราคา</button>

    <div class="routes" id="routesList"></div>
  </aside>
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
  document.getElementById("lbl-dest").textContent = i18n[currentLang].destinationLabel;
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

  const sel = document.getElementById("vehicle");
  sel.innerHTML = "";

  const placeholder = document.createElement("option");
  placeholder.textContent = (currentLang === "th" ? "เลือกประเภทยานพาหนะ" : "Select vehicle type");
  placeholder.disabled = true;
  placeholder.selected = true;
  sel.appendChild(placeholder);

  for(const k in vehicleRates){
    const opt = document.createElement("option");
    opt.value = k;
    opt.textContent = vehicleNames[currentLang][k] || k;
    sel.appendChild(opt);
  }

  if(!lastRoutes || lastRoutes.length===0){
    document.getElementById('routesList').textContent = i18n[currentLang].noRoute;
  }

  renderFareTable();
}

// --- ส่วน Map / Routing / Controls เหมือนเดิม ---


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
      triggerTextSearch(document.getElementById('search').value);
    }
  });

  if(navigator.geolocation){
    navigator.geolocation.getCurrentPosition(p=>{
      currentPos = {lat:p.coords.latitude, lng:p.coords.longitude};
      const posLatLng = new google.maps.LatLng(currentPos.lat, currentPos.lng);
      if(!phuketBounds.contains(posLatLng)) return;
      if(!markerA){ markerA = new google.maps.Marker({map, icon:{path:google.maps.SymbolPath.CIRCLE,scale:8,fillColor:'#1e88e5',fillOpacity:1,strokeWeight:0}}); }
      markerA.setPosition(posLatLng); markerA.setMap(map);
      markerA.setTitle('You');
      map.setCenter(posLatLng);
    });
  }

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
      });
    } else alert('Geolocation not supported');
  });

  document.getElementById('btn-calc').addEventListener('click', () => {
    if(currentPos && markerB && markerB.getPosition()) computeRoutes(currentPos, markerB.getPosition());
    renderFareTable();
  });

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

  document.getElementById('btn-toggle-fare').addEventListener('click', () => {
    const f = document.getElementById('fareTable');
    f.style.display = (f.style.display === 'none' ? 'block' : 'none');
  });

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

  selectedRouteIndex = 0;
  drawPolyline(routes[0], 0);
  highlightSelected();
}

function drawPolyline(route, index){
  polyLines.forEach(p=>p.setMap(null)); polyLines=[];
  const colors = ['#2ecc71','#f1c40f','#e74c3c','#1e88e5','#9b59b6'];
  const color = colors[index % colors.length];
  const pl = new google.maps.Polyline({
    path: route.overview_path,
    map: map,
    strokeColor: color,
    strokeOpacity: 0.8,
    strokeWeight: 6
  });
  polyLines.push(pl);
  const bounds = new google.maps.LatLngBounds();
  route.overview_path.forEach(p=>bounds.extend(p));
  map.fitBounds(bounds);
}

function highlightSelected(){
  const cards = document.querySelectorAll('.route-card');
  cards.forEach((c,i)=>{ c.style.borderColor = (i===selectedRouteIndex?'#1e88e5':'#ccc'); });
}

// ------------------ Update fare labels on vehicle change ------------------
function updateRouteFareLabels(){
  if(lastRoutes.length===0) return;
  const v = document.getElementById('vehicle').value;
  const cards = document.querySelectorAll('.route-card');
  lastRoutes.forEach((r,i)=>{
    const fareText = `${calculateFare(r, v)} ฿`;
    cards[i].querySelector('.fare-pill').textContent = fareText;
  });
}

</script>

<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places&callback=initMap" async defer></script>
</body>
</html>
