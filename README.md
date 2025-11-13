<html lang="th">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Phuket Trip — Route & Fare</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-sA+4v+3s8z1qgqkGk3mF2k2KJp2GkQmYh2MShb3f3Hg=" crossorigin=""/>
<style>
:root{
  --accent:#1e88e5; --muted:#6b7280; --card:#ffffff; --bg:#f6f8fb;
  --rounded:12px;
}
  h1, h2, h3 { display: none; }
html,body{height:100%;margin:0;font-family:Inter,system-ui,-apple-system,"Sarabun",sans-serif;background:var(--bg);color:#112;}
.wrap{display:flex;height:100vh;gap:12px;padding:12px;box-sizing:border-box;}
.map-wrap{flex:1;position:relative;border-radius:var(--rounded);overflow:hidden;box-shadow:0 8px 24px rgba(15,23,42,.06);min-height:320px;background:#e9eef6}
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

#map-error{position:absolute;left:12px;right:12px;top:12px;bottom:12px;background:rgba(255,255,255,0.95);z-index:9999;border-radius:12px;display:none;align-items:center;justify-content:center;padding:20px;text-align:center;box-shadow:0 8px 32px rgba(0,0,0,0.08)}
#map-error h2{margin:0 0 8px 0}
#map-error p{margin:0 0 12px 0;color:#334}
#map-error button{padding:8px 12px;border-radius:8px;border:0;background:var(--accent);color:#fff;cursor:pointer}

.marker-shadow{filter:drop-shadow(0 6px 8px rgba(17,24,39,0.12));}

@media(max-width:1000px){
  .wrap{flex-direction:column;padding:8px;}
  .sidebar{width:auto;min-width:unset;max-height:40vh;}
  .map-wrap{height:60vh;min-height:320px;}
  .fare-table{right:10px;bottom:10px;min-width:140px;}
}
@media(max-width:768px){
  .wrap{
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    gap: 0;
    padding: 0;
  }


  .map-wrap{
    order: -1; 
    width: 100%;
    height: 40vh;
    flex-shrink: 0;
  }

  .sidebar{
    flex: 1;
    overflow-y: auto;
    padding: 12px;
    box-sizing: border-box;
  }

  .routes{
    display: flex;
    flex-direction: column;
    gap: 8px;
    flex: none;
  }

  #btn-lang{
    position: fixed;
    top: 10px;
    right: 10px;
    z-index: 9999;
  }
  
  .legend {
    padding: 3px 5px;
    font-size: 8px;       
  }
  .dot {
    width: 10px;
    height: 3px;          
    margin-right: 3px;
  }

}

</style>
</head>
<body>
<div class="wrap">
  <aside class="sidebar" aria-label="controls">
    <div class="logo"> 
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

    <div id="map-error" role="alert">
      <div>
        <h2>แผนที่ไม่สามารถโหลดได้</h2>
        <p id="map-error-msg">ระบบตรวจพบปัญหาในการโหลด Google Maps (เช่น API key ไม่ถูกต้อง หรือถูกปิดการใช้งาน)</p>
        <div style="display:flex;gap:8px;justify-content:center;margin-top:10px">
          <button id="map-retry">ลองโหลดใหม่</button>
          <button id="map-doc" onclick="window.open('https://developers.google.com/maps/documentation/javascript/error-messages#invalid-key-map-error','_blank')">วิธีแก้ไข</button>
        </div>
      </div>
    </div>

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

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-o9N1j8g+8U5n2xQigb0kC4Jk5sZk2bS+3pA9u1r0i+M=" crossorigin=""></script>
<script>

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

let currentLang = 'th';

let map, directionsService, trafficLayer, placesService;
let markerA = null, markerB = null;
let currentPos = null, lastRoutes = [], selectedRouteIndex = 0, polyLines = [], poiMarkers = [];
let usingLeafletFallback = false;

function km(m){ return (m/1000).toFixed(1); }
function calculateFare(route, vehicleKey){
  const v = vehicleRates[vehicleKey];
  if(!v) return 0;
  if(v.fixed) return v.min;
  const kmDistance = route.legs[0].distance.value / 1000;
  let fare = v.base + Math.max(0, kmDistance - v.freeKm) * v.perKm;
  return Math.max(Math.round(fare), v.min);
}

const destinationSvg = `
<svg xmlns='http://www.w3.org/2000/svg' width='96' height='96' viewBox='0 0 24 24'>
  <defs>
    <linearGradient id='g' x1='0' y1='0' x2='1' y2='1'>
      <stop offset='0' stop-color='%231e88e5'/>
      <stop offset='1' stop-color='%233a9bf0'/>
    </linearGradient>
  </defs>
  <path d='M12 2C8.1 2 5 5.1 5 9c0 5.2 7 13 7 13s7-7.8 7-13c0-3.9-3.1-7-7-7z' fill='url(%23g)' stroke='%230b2540' stroke-opacity='0.12' stroke-width='0.5'/>
  <circle cx='12' cy='9' r='2.6' fill='%23fff' />
</svg>
`;
const destinationIconUrl = 'data:image/svg+xml;utf8,' + encodeURIComponent(destinationSvg);

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

function initMap() {
  const overlay = document.getElementById('map-error'); 
  if (overlay) overlay.style.display = 'none';

  const phuketBounds = new google.maps.LatLngBounds(
    { lat: 7.68, lng: 98.15 },
    { lat: 8.32, lng: 98.65 }
  );

  const center = { lat: 7.8804, lng: 98.3923 };
  map = new google.maps.Map(document.getElementById('map'), {
    center,
    zoom: 11,
    mapTypeControl: false,
    streetViewControl: false
  });

  directionsService = new google.maps.DirectionsService();
  trafficLayer = new google.maps.TrafficLayer();
  trafficLayer.setMap(map);
  placesService = new google.maps.places.PlacesService(map);

  function setMarkerA(latLng) {
    if (!markerA) {
      markerA = new google.maps.Marker({
        map,
        position: latLng,
        title: currentLang === 'th' ? 'ตำแหน่งปัจจุบันของคุณ' : 'Your current location',
        icon: {
          path: google.maps.SymbolPath.CIRCLE,
          scale: 9,
          fillColor: '#1e88e5',
          fillOpacity: 1,
          strokeColor: '#fff',
          strokeWeight: 2
        }
      });
    } else markerA.setPosition(latLng);
  }

  function setMarkerB(latLng, placeName, formattedAddress) {
    if (!markerB) {
      markerB = new google.maps.Marker({
        map,
        position: latLng,
        title: placeName || (currentLang === 'th' ? 'ปลายทาง' : 'Destination'),
        icon: {
          path: google.maps.SymbolPath.CIRCLE,
          scale: 9,
          fillColor: '#1e88e5',
          fillOpacity: 1,
          strokeColor: '#fff',
          strokeWeight: 2
        }
      });
      const info = new google.maps.InfoWindow({
        content: `<strong>${placeName || ''}</strong><div style='font-size:12px;color:#666'>${formattedAddress || ''}</div>`
      });
      markerB.addListener('click', () => info.open(map, markerB));
    } else {
      markerB.setPosition(latLng);
      markerB.setTitle(placeName || (currentLang === 'th' ? 'ปลายทาง' : 'Destination'));
    }
  }

  const acStart = new google.maps.places.Autocomplete(document.getElementById('searchStart'), {
    bounds: phuketBounds,
    componentRestrictions: { country: 'th' },
    fields: ['place_id', 'geometry', 'name', 'formatted_address']
  });

  acStart.addListener('place_changed', () => {
    const place = acStart.getPlace();
    if (place?.geometry?.location) {
      const loc = place.geometry.location;
      if (!phuketBounds.contains(loc)) {
        alert(i18n[currentLang].outsidePhuket);
        return;
      }
      setMarkerA(loc);
      map.panTo(loc);
      currentPos = { lat: loc.lat(), lng: loc.lng() };
      if (markerB && markerB.getPosition()) computeRoutes(currentPos, markerB.getPosition());
    }
  });


  const acEnd = new google.maps.places.Autocomplete(document.getElementById('search'), {
    bounds: phuketBounds,
    componentRestrictions: { country: 'th' },
    fields: ['place_id', 'geometry', 'name', 'formatted_address']
  });

  acEnd.addListener('place_changed', () => {
    const place = acEnd.getPlace();
    if (!place?.geometry?.location) return;

    const loc = place.geometry.location;
    if (!phuketBounds.contains(loc)) {
      alert(i18n[currentLang].outsidePhuket);
      return;
    }

    setMarkerB(loc, place.name, place.formatted_address);
    map.panTo(loc);

    if (markerA && markerA.getPosition()) {
      computeRoutes(markerA.getPosition(), loc);
    }
  });

  document.getElementById('btn-current').addEventListener('click', () => {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (pos) => {
          const loc = { lat: pos.coords.latitude, lng: pos.coords.longitude };
          if (!phuketBounds.contains(loc)) {
            alert(i18n[currentLang].outsidePhuket);
            return;
          }
          setMarkerA(loc);
          map.panTo(loc);
          currentPos = loc;
          if (markerB && markerB.getPosition()) computeRoutes(loc, markerB.getPosition());
        },
        () => alert('ไม่สามารถระบุตำแหน่งของคุณได้')
      );
    } else {
      alert('เบราว์เซอร์ของคุณไม่รองรับการระบุตำแหน่ง');
    }
  });

  document.getElementById('btn-calc').addEventListener('click', () => {
    if (markerA && markerB) computeRoutes(markerA.getPosition(), markerB.getPosition());
    else alert('กรุณาเลือกต้นทางและปลายทางก่อน');
  });

  document.getElementById('btn-reset').addEventListener('click', () => {
    if (markerA) markerA.setMap(null);
    if (markerB) markerB.setMap(null);
    markerA = markerB = null;
    document.getElementById('search').value = '';
    document.getElementById('searchStart').value = '';
    document.getElementById('routesList').innerHTML = '';
    polyLines.forEach(p => p.setMap(null));
    polyLines = [];
  });

  document.getElementById('btn-toggle-fare').addEventListener('click', () => {
    const table = document.getElementById('fareTable');
    table.style.display = (table.style.display === 'none' || !table.style.display) ? 'block' : 'none';
  });


}

function computeRoutes(origin, dest) {
  const selectedVehicle = document.getElementById('vehicle')?.value || 'grabCar';

  const directions = new google.maps.DirectionsService();
  directions.route(
    {
      origin,
      destination: dest,
      travelMode: google.maps.TravelMode.DRIVING,
      provideRouteAlternatives: true,
    },
    (res, status) => {
      if (status !== 'OK') return alert('ไม่สามารถค้นหาเส้นทางได้');
      lastRoutes = res.routes;
      const list = document.getElementById('routesList');
      list.innerHTML = '';

      polyLines.forEach(p => p.setMap(null));
      polyLines = [];

      res.routes.forEach((route, i) => {
        const leg = route.legs[0];
        const fare = calculateFare(route, selectedVehicle);

        const card = document.createElement('div');
        card.className = 'route-card';
        card.innerHTML = `
          <div class="route-left">
            <div class="route-title">${i18n[currentLang].routeLabel} ${i+1}</div>
            <div class="route-meta">${km(leg.distance.value)} ${i18n[currentLang].kmLabel} • ${leg.duration.text}</div>
          </div>
          <div class="fare-pill">${fare} ฿</div>
        `;

        card.addEventListener('click', () => {
          polyLines.forEach(p => p.setMap(null));
          const line = new google.maps.Polyline({
            path: route.overview_path,
            map,
            strokeColor: '#1e88e5',
            strokeOpacity: 0.8,
            strokeWeight: 6
          });
          polyLines.push(line);
          list.querySelectorAll('.route-card').forEach(c => c.classList.remove('selected'));
          card.classList.add('selected');

          showServiceOption(selectedVehicle);
        });

        list.appendChild(card);
      });

      if (res.routes[0]) {
        const line = new google.maps.Polyline({
          path: res.routes[0].overview_path,
          map,
          strokeColor: '#1e88e5',
          strokeOpacity: 0.8,
          strokeWeight: 6
        });
        polyLines.push(line);
      }
    }
  );
  
}
applyLanguage();

function showServiceOption(vehicle) {
  const appLinks = {
    grabCar: 'grab://open?deeplink_id=ride',
    grabBike: 'grab://open?deeplink_id=ride',
    boltEconomy: 'bolt://open',
    boltStandard: 'bolt://open',
    boltVan: 'bolt://open',
    inDrive: 'indrive://open',
    taxiMeter: '',
    tukTuk: '',
    motoTaxi: ''
  };

  const webLinks = {
    grabCar: 'https://surl.li/ifaylz',
    grabBike: 'https://surl.li/ifaylz',
    boltEconomy: 'https://bolt.eu/th-th/rides/',
    boltStandard: 'https://bolt.eu/th-th/rides/',
    boltVan: 'https://bolt.eu/th-th/rides/',
    inDrive: 'https://indrive.com/en-th/city-rides',
    taxiMeter: 'https://www.phuket.go.th/',
    tukTuk: 'https://www.phuket.go.th/',
    motoTaxi: 'https://www.phuket.go.th/'
  };

  const storeLinks = {
    grabCar: {
      play: 'https://play.google.com/store/apps/details?id=com.grabtaxi.passenger',
      ios: 'https://apps.apple.com/app/grab/id647268330'
    },
    boltEconomy: {
      play: 'https://play.google.com/store/apps/details?id=com.bolt.android.passenger',
      ios: 'https://apps.apple.com/app/bolt/id675033630'
    },
    inDrive: {
      play: 'https://play.google.com/store/apps/details?id=sinet.startup.inDriver',
      ios: 'https://apps.apple.com/app/indrive/id996182390'
    },
    grabBike: {
      play: 'https://play.google.com/store/apps/details?id=com.grabtaxi.passenger',
      ios: 'https://apps.apple.com/app/grab/id647268330'
    },
    boltStandard: {
      play: 'https://play.google.com/store/apps/details?id=com.bolt.android.passenger',
      ios: 'https://apps.apple.com/app/bolt/id675033630'
    },
    boltVan: {
      play: 'https://play.google.com/store/apps/details?id=com.bolt.android.passenger',
      ios: 'https://apps.apple.com/app/bolt/id675033630'
    }
  };

  const vehicleName = vehicleNames[currentLang][vehicle] || vehicle;

  const modal = document.createElement('div');
  modal.style.position = 'fixed';
  modal.style.top = '0';
  modal.style.left = '0';
  modal.style.width = '100%';
  modal.style.height = '100%';
  modal.style.background = 'rgba(0,0,0,0.6)';
  modal.style.display = 'flex';
  modal.style.alignItems = 'center';
  modal.style.justifyContent = 'center';
  modal.style.zIndex = '9999';

  modal.innerHTML = `
    <div style="background:white;padding:20px 30px;border-radius:16px;text-align:center;max-width:300px;">
      <h3 style="margin-bottom:10px;">เปิดบริการ ${vehicleName}</h3>
      <p>คุณต้องการเปิดแอปหรือเว็บไซต์?</p>
      <div style="display:flex;gap:10px;justify-content:center;margin-top:15px;">
        <button id="openApp" style="background:#1e88e5;color:white;border:none;padding:8px 16px;border-radius:8px;">เปิดแอป</button>
        <button id="openWeb" style="background:#4caf50;color:white;border:none;padding:8px 16px;border-radius:8px;">เว็บไซต์</button>
      </div>
      <button id="closeModal" style="margin-top:15px;background:#ccc;border:none;padding:6px 12px;border-radius:8px;">ยกเลิก</button>
    </div>
  `;

  document.body.appendChild(modal);

  document.getElementById('openApp').addEventListener('click', () => {
    const userAgent = navigator.userAgent.toLowerCase();
    const isMobile = /android|iphone|ipad|ipod/i.test(userAgent);
    const isIOS = /iphone|ipad|ipod/i.test(userAgent);

    if (!isMobile) {
      window.open(webLinks[vehicle], '_blank');
      document.body.removeChild(modal);
      return;
    }

    const appLink = appLinks[vehicle];
    const playLink = storeLinks[vehicle]?.play;
    const iosLink = storeLinks[vehicle]?.ios;
    const storeLink = isIOS ? iosLink : playLink;

    if (!appLink) {
      window.open(webLinks[vehicle], '_blank');
      document.body.removeChild(modal);
      return;
    }

    const timeout = setTimeout(() => {
      if (storeLink) window.location.href = storeLink; 
    }, 1800);

    window.location.href = appLink;

    window.addEventListener('pagehide', () => clearTimeout(timeout));
    window.addEventListener('blur', () => clearTimeout(timeout));

    document.body.removeChild(modal);
  });

  document.getElementById('openWeb').addEventListener('click', () => {
    window.open(webLinks[vehicle], '_blank');
    document.body.removeChild(modal);
  });

  document.getElementById('closeModal').addEventListener('click', () => {
    document.body.removeChild(modal);
  });
}

function renderRoutes(routes, vehicleKey){
  console.log("renderRoutes called, routes:", routes);
  const container = document.getElementById('routesList');
  container.innerHTML = '';
  if(!routes || routes.length===0){
    container.textContent = i18n[currentLang].noRoute;
    return;
  }

  const colors = ['#2ecc71','#f1c40f','#e74c3c','#1e88e5','#9b59b6'];

  routes.forEach((r,i) => {

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

    poly.addListener('click', () => {
      selectedRouteIndex = poly.routeIndex;
      drawPolyline(i);
      highlightSelected();
      if(routes[selectedRouteIndex]?.bounds) map.fitBounds(routes[selectedRouteIndex].bounds);
    });

    const card = document.createElement('div');
    card.className='route-card';
    card.dataset.routeIndex = i;

    const distText = `${km(r.legs[0].distance.value)} ${i18n[currentLang].kmLabel}`;
    const durText = `${Math.round(r.legs[0].duration.value/60)} ${i18n[currentLang].minsLabel}`;
    const fare = calculateFare(r, vehicleKey);

    card.innerHTML = `<div class="route-left">
        <div class="route-title">${i18n[currentLang].routeLabel} ${i+1}</div>
        <div class="route-meta">${distText} | ${durText}</div>
      </div>
      <div class="fare-pill">${fare} ฿</div>`;

    card.onclick = () => {
      selectedRouteIndex = i;
      drawPolyline(i);
      highlightSelected();
      if(r.bounds) map.fitBounds(r.bounds);
    };

    container.appendChild(card);
  });

  selectedRouteIndex = 0;
  drawPolyline(0);
  highlightSelected();
}


function drawPolyline(index){
  polyLines.forEach((p, idx) => {
    p.setOptions({strokeOpacity: idx === index ? 1.0 : 0.2, strokeWeight: idx === index ? 8 : 5});
  });
}

function highlightSelected(){
  const cards = document.querySelectorAll('.route-card');
  cards.forEach(c=>{
    if(+c.dataset.routeIndex === selectedRouteIndex) c.classList.add('selected');
    else c.classList.remove('selected');
  });
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

document.getElementById('btn-lang').addEventListener('click', ()=>{
  currentLang = (currentLang === 'th' ? 'en' : 'th');
  applyLanguage();
});

document.getElementById('map-retry').addEventListener('click', ()=>{
  const existing = document.querySelector('script[data-gm-retry]');
  if(existing) existing.remove();
  const tag = document.createElement('script');
  tag.async = true; tag.defer = true;
  tag.setAttribute('data-gm-retry', '1');
  tag.src = 'https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY_HERE&libraries=places&callback=initMap';
  tag.onerror = ()=>{ showMapError('เกิดข้อผิดพลาดในการโหลด Google Maps — กรุณาตรวจสอบ API key และการตั้งค่าบัญชี Google Cloud'); };
  document.body.appendChild(tag);
});

window.initMap = initMap;
window.calculateFare = calculateFare;
window.triggerTextSearch = triggerTextSearch;
window.gm_authFailure = gm_authFailure;

setTimeout(handleMapsNotLoaded, 3000);
</script>

<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places&callback=initMap" async defer onerror="(function(){document.getElementById('map-error').style.display='flex';document.getElementById('map-error-msg').textContent='Failed to load Google Maps script (network error or blocked). Using fallback map.';})()"></script>
</body>
</html>
