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
/* Layout: Desktop = split, Mobile = map top, controls bottom (locked viewport, no page scroll) */
.wrap{display:flex;height:100vh;gap:12px;padding:12px;box-sizing:border-box;}
.map-wrap{flex:1;position:relative;border-radius:var(--rounded);overflow:hidden;box-shadow:0 8px 24px rgba(15,23,42,.06);min-height:320px;}
#map{width:100%;height:100%}
.sidebar{width:420px;min-width:320px;background:var(--card);border-radius:var(--rounded);box-shadow:0 8px 24px rgba(15,23,42,.08);padding:16px;display:flex;flex-direction:column;gap:10px;overflow:auto;}
.logo{font-weight:700;font-size:18px;display:flex;align-items:center;gap:8px;position:relative;}
#btn-lang{position:absolute; top:0; right:0;padding:6px 10px;font-size:12px;background:var(--accent); color:#fff;border:none; border-radius:6px;cursor:pointer}
.search-row{display:flex;gap:8px;align-items:center}

/* <-- NEW: Style for the new Autocomplete web component */
gmp-place-autocomplete {
  flex: 1;
}
gmp-place-autocomplete input {
  padding: 10px 12px;
  border-radius: 10px;
  border: 1px solid #e6e9ee;
  font-size: 15px;
  width: 100%;
  box-sizing: border-box;
}

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

/* Responsive: mobile column layout with map on top and controls fixed bottom area */
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
            <gmp-place-autocomplete id="ac-start" placeholder="เลือกต้นทาง..." country="th"></gmp-place-autocomplete>
      <button id="btn-current" class="btn alt" title="ใช้ตำแหน่งปัจจุบัน" aria-label="ใช้ตำแหน่งปัจจุบัน">📍</button>
    </div>

    <div class="section-title" id="lbl-dest">ปลายทาง</div>
    <div class="search-row">
            <gmp-place-autocomplete id="ac-end" placeholder="ค้นหาปลายทาง..." country="th"></gmp-place-autocomplete>
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
/* -------------- Data (No changes needed) -------------- */
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
  th: { search:"ค้นหาปลายทาง...", start:"เลือกต้นทาง...", originLabel:"ต้นทาง", destinationLabel:"ปลายทาง", calc:"คำนวณ", reset:"รีเซ็ต", fareTable:"ตารางราคา", popular:["หาดป่าตอง","พระใหญ่ (Big Buddha)","วัดฉลอง (Wat Chalong)"], noRoute:"ยังไม่มีเส้นทาง", onlyPhuket:"กรุณาเลือกตำแหน่งในภูเก็ตเท่านั้น", outsidePhuket:"ตำแหน่งอยู่นอกภูเก็ต", routeLabel:"เส้นทาง", kmLabel:"กม.", minsLabel:"นาที", traffic:"การจราจร", fast:"คล่องตัว", moderate:"ปานกลาง", heavy:"หนาแน่น", vehicleCol:"ยานพาหนะ", baseCol:"ฐานเริ่มต้น", perKmCol:"ต่อ กม.", currentPosBtn: "ใช้ตำแหน่งปัจจุบัน", toggleFare: "แสดง/ซ่อนตารางราคา", },
  en: { search:"Search destination...", start:"Select origin...", originLabel:"Origin", destinationLabel:"Destination", calc:"Calculate", reset:"Reset", fareTable:"Fare Table", popular:["Patong Beach","Big Buddha","Wat Chalong"], noRoute:"No routes yet", onlyPhuket:"Please select a location within Phuket only", outsidePhuket:"Location is outside Phuket", routeLabel:"Route", kmLabel:"km", minsLabel:"mins", traffic:"Traffic", fast:"Fast", moderate:"Moderate", heavy:"Heavy", vehicleCol:"Vehicle", baseCol:"Base", perKmCol:"Per km", currentPosBtn: "Use current location", toggleFare: "Show/Hide Fare Table", }
};
let currentLang = 'th';

/* -------------- State -------------- */
let map, directionsService, trafficLayer;
let markerA = null, markerB = null; // These will now be AdvancedMarkerElements
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
  // <-- MODIFIED: Update placeholder for new autocomplete components
  document.getElementById("ac-end").placeholder = i18n[currentLang].search;
  document.getElementById("ac-start").placeholder = i18n[currentLang].start;
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
  const sel = document.getElementById("vehicle"); sel.innerHTML = "";
  const ph = document.createElement("option"); ph.value = ""; ph.textContent = (currentLang==='th'?'-- เลือกประเภทยานพาหนะ --':'-- Select vehicle --'); ph.disabled = true;
  sel.appendChild(ph);
  for(const k in vehicleRates){
    const opt = document.createElement("option");
    opt.value = k;
    opt.textContent = (vehicleNames[currentLang] && vehicleNames[currentLang][k]) ? vehicleNames[currentLang][k] : k;
    sel.appendChild(opt);
  }
  sel.value = sel.querySelector('option[value="grabCar"]') ? 'grabCar' : sel.querySelector('option:not([disabled])').value;
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
  document.getElementById('fareTable').style.display = 'none';
}

/* -------------- Map & Places (MODIFIED VERSION) -------------- */
async function initMap() { // <-- MODIFIED: Made async for new libraries
  // NEW: Import libraries
  const { Map } = await google.maps.importLibrary("maps");
  const { AdvancedMarkerElement } = await google.maps.importLibrary("marker");
  const { Place } = await google.maps.importLibrary("places");

  const phuketBounds = new google.maps.LatLngBounds( { lat: 7.7, lng: 98.2 }, { lat: 8.1, lng: 98.6 } );
  const center = { lat: 7.8804, lng: 98.3923 };

  map = new Map(document.getElementById("map"), { // <-- MODIFIED: Use imported Map
    center,
    zoom: 11,
    mapId: 'DEMO_MAP_ID', // <-- NEW: Advanced Markers require a Map ID. Replace with your own.
    mapTypeControl: false,
    streetViewControl: false,
  });

  directionsService = new google.maps.DirectionsService();
  trafficLayer = new google.maps.TrafficLayer();
  trafficLayer.setMap(map);

  // NEW: Create HTML elements for custom marker icons
  const startIcon = document.createElement('img');
  startIcon.src = "https://maps.google.com/mapfiles/ms/icons/blue-dot.png";
  startIcon.style.cssText = "width:40px; height:40px;";
  
  const endIcon = document.createElement('img');
  endIcon.src = "https://maps.google.com/mapfiles/ms/icons/red-dot.png";
  endIcon.style.cssText = "width:40px; height:40px;";

  // NEW: Function to create or update a marker
  function createOrUpdateMarker(markerRef, position, iconContent) {
    if (!markerRef) {
      return new AdvancedMarkerElement({ map, position, content: iconContent });
    }
    markerRef.position = position;
    return markerRef;
  }

  // -------------------- AUTOCOMPLETE START (NEW) --------------------
  const acStart = document.getElementById('ac-start');
  acStart.addEventListener('gmp-placechange', (event) => {
    const place = event.target.place;
    if (place?.location) {
      if (!phuketBounds.contains(place.location)) {
        alert(i18n[currentLang].onlyPhuket);
        return;
      }
      markerA = createOrUpdateMarker(markerA, place.location, startIcon);
      map.panTo(place.location);
      currentPos = place.location;
      if (markerB?.position) computeRoutes(currentPos, markerB.position);
    }
  });

  // -------------------- AUTOCOMPLETE END (NEW) --------------------
  const acEnd = document.getElementById('ac-end');
  acEnd.addEventListener('gmp-placechange', (event) => {
    const place = event.target.place;
    if (place?.location) {
      if (!phuketBounds.contains(place.location)) {
        alert(i18n[currentLang].onlyPhuket);
        return;
      }
      markerB = createOrUpdateMarker(markerB, place.location, endIcon);
      map.panTo(place.location);
      if (currentPos) computeRoutes(currentPos, place.location);
    } else {
       triggerTextSearch(acEnd.value);
    }
  });

  // -------------------- INITIAL LOCATION --------------------
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (p) => {
        currentPos = { lat: p.coords.latitude, lng: p.coords.longitude };
        if (!phuketBounds.contains(currentPos)) return;
        markerA = createOrUpdateMarker(markerA, currentPos, startIcon);
        map.setCenter(currentPos);
      },
      () => {}
    );
  }

  // -------------------- BUTTON: CURRENT LOCATION --------------------
  document.getElementById("btn-current").addEventListener("click", () => {
    if (!navigator.geolocation) return alert("Geolocation not supported");
    navigator.geolocation.getCurrentPosition(
      (p) => {
        currentPos = { lat: p.coords.latitude, lng: p.coords.longitude };
        if (!phuketBounds.contains(currentPos)) return alert(i18n[currentLang].outsidePhuket);
        markerA = createOrUpdateMarker(markerA, currentPos, startIcon);
        map.panTo(currentPos);
        if (markerB?.position) computeRoutes(currentPos, markerB.position);
      },
      () => alert("Unable to retrieve your location")
    );
  });

  // -------------------- BUTTON: CALCULATE FARE --------------------
  document.getElementById("btn-calc").addEventListener("click", () => {
    if (lastRoutes.length > 0) {
      updateRouteFareLabels();
      return;
    }
    if (currentPos && markerB?.position) {
      computeRoutes(currentPos, markerB.position);
    } else {
      alert("กรุณาเลือกต้นทางและปลายทางก่อน (หรือใช้ปุ่ม 📍)");
    }
  });

  // -------------------- BUTTON: RESET --------------------
  document.getElementById("btn-reset").addEventListener("click", () => {
    acStart.value = "";
    acEnd.value = "";
    lastRoutes = [];
    selectedRouteIndex = 0;
    document.getElementById("routesList").innerHTML = i18n[currentLang].noRoute;
    polyLines.forEach((p) => p.setMap(null)); polyLines = [];
    poiMarkers.forEach((m) => m.setMap(null)); poiMarkers = [];
    if (markerA) { markerA.map = null; markerA = null; }
    if (markerB) { markerB.map = null; markerB = null; }
    currentPos = null;
  });

  // -------------------- BUTTON: TOGGLE FARE --------------------
  document.getElementById("btn-toggle-fare").addEventListener("click", () => {
    const f = document.getElementById("fareTable");
    const visible = f.style.display !== "none";
    f.style.display = visible ? "none" : "block";
    f.setAttribute("aria-hidden", visible ? "true" : "false");
  });

  document.getElementById("vehicle").addEventListener("change", updateRouteFareLabels);
  applyLanguage();
}

/* -------------- Places text fallback (MODIFIED) -------------- */
async function triggerTextSearch(txt){ // <-- MODIFIED: async function
  if(!txt) return;
  const request = {
    textQuery: txt,
    fields: ['location'],
    region: 'th',
    locationBias: map.getBounds()
  };
  try {
    const { places } = await google.maps.places.Place.searchByText(request); // <-- NEW: Await search
    if(places && places.length > 0){
      const loc = places[0].location;
      const endIcon = document.createElement('img');
      endIcon.src = "http://maps.google.com/mapfiles/ms/icons/red-dot.png"; // fallback icon
      endIcon.style.cssText = "width:40px; height:40px;";
      
      if (!markerB) markerB = new google.maps.marker.AdvancedMarkerElement({ map });
      markerB.position = loc;
      markerB.content = endIcon;
      map.panTo(loc);
      if(currentPos) computeRoutes(currentPos, loc);
    } else {
      alert('No results found');
    }
  } catch (error) {
    console.error("Text Search failed", error);
    alert('Search failed');
  }
}

/* -------------- Routing & render (No significant changes) -------------- */
function computeRoutes(origin, dest){
  const selectedVehicle = document.getElementById('vehicle').value || 'grabCar';
  directionsService.route({
    origin: origin,
    destination: dest,
    travelMode: google.maps.TravelMode.DRIVING,
    provideRouteAlternatives: true
  }, (res,status) => {
    if(status === 'OK'){
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
  const colors = ['#2ecc71','#f1c40f','#e74c3c','#1e88e5','#9b59b6'];
  routes.forEach((r,i) => {
    const path = r.overview_path.map(p => ({lat:p.lat(), lng:p.lng()}));
    const poly = new google.maps.Polyline({path: path, map: map, strokeColor: colors[i % colors.length], strokeWeight: 6, strokeOpacity: 0.6 });
    poly.routeIndex = i;
    polyLines.push(poly);
    poly.addListener('click', ()=> {
      selectedRouteIndex = poly.routeIndex;
      highlightSelected();
      map.fitBounds(routes[selectedRouteIndex].bounds);
    });
    const card = document.createElement('div'); card.className='route-card';
    card.dataset.routeIndex = i;
    const distText = `${km(r.legs[0].distance.value)} ${i18n[currentLang].kmLabel}`;
    const durText = `${Math.round(r.legs[0].duration.value/60)} ${i18n[currentLang].minsLabel}`;
    const fare = calculateFare(r, vehicleKey || document.getElementById('vehicle').value || 'grabCar');
    card.innerHTML = `<div class="route-left"><div class="route-title">${i18n[currentLang].routeLabel} ${i+1}</div><div class="route-meta">${distText} | ${durText}</div></div><div class="fare-pill">${fare} ฿</div>`;
    card.onclick = () => { selectedRouteIndex = i; drawPolyline(routes[i], i); highlightSelected(); map.fitBounds(routes[i].bounds); };
    container.appendChild(card);
  });
  selectedRouteIndex = 0;
  drawPolyline(routes[0], 0);
  highlightSelected();
}
function drawPolyline(route, index){
  polyLines.forEach((p, idx) => { p.setOptions({strokeOpacity: idx === index ? 1.0 : 0.4, strokeWeight: idx === index ? 8 : 6}); });
}
function highlightSelected(){
  document.querySelectorAll('.route-card').forEach((c,i)=>{ c.classList.toggle('selected', i === selectedRouteIndex); });
  updateRouteFareLabels();
}
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
window.initMap = initMap;
</script>

<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places,marker&callback=initMap" async></script>
</body>
</html>
