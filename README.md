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
h1,h2,h3{display:none;}
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
      <place-autocomplete id="searchStart" placeholder="เลือกต้นทาง..." country="th"></place-autocomplete>
      <button id="btn-current" class="btn alt" title="ใช้ตำแหน่งปัจจุบัน" aria-label="ใช้ตำแหน่งปัจจุบัน">📍</button>
    </div>

    <div class="section-title" id="lbl-dest">ปลายทาง</div>
    <div class="search-row">
      <place-autocomplete id="search" placeholder="ค้นหาปลายทาง..." country="th"></place-autocomplete>
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

<script type="module">
import { AdvancedMarkerElement } from "https://unpkg.com/@googlemaps/marker/dist/marker.esm.js";

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

const i18n = { /* ...เหมือนเดิม... */ };
let currentLang = 'th';

let map, directionsService, trafficLayer, markerA=null, markerB=null, lastRoutes=[], polyLines=[];

/* --------- Helpers --------- */
function km(m){return (m/1000).toFixed(1);}
function calculateFare(route, vehicleKey){
  const v = vehicleRates[vehicleKey]; if(!v) return 0;
  if(v.fixed) return v.min;
  const kmDistance = route.legs[0].distance.value/1000;
  let fare = v.base + Math.max(0, kmDistance-v.freeKm)*v.perKm;
  return Math.max(Math.round(fare),v.min);
}

/* --------- Map & Routing --------- */
async function initMap(){
  const phuketBounds = new google.maps.LatLngBounds({lat:7.7,lng:98.2},{lat:8.1,lng:98.6});
  map = new google.maps.Map(document.getElementById('map'),{center:{lat:7.8804,lng:98.3923},zoom:11,mapTypeControl:false,streetViewControl:false});
  directionsService = new google.maps.DirectionsService();
  trafficLayer = new google.maps.TrafficLayer(); trafficLayer.setMap(map);

  // Current location
  if(navigator.geolocation){
    navigator.geolocation.getCurrentPosition(p=>{
      const pos = {lat:p.coords.latitude,lng:p.coords.longitude};
      const latLng = new google.maps.LatLng(pos.lat,pos.lng);
      if(!phuketBounds.contains(latLng)) return;
      markerA = new AdvancedMarkerElement({map, position:latLng, title:'You'});
      map.setCenter(latLng);
    });
  }

  // Buttons
  document.getElementById('btn-current').addEventListener('click', async ()=>{
    if(!navigator.geolocation) return alert('Geolocation not supported');
    navigator.geolocation.getCurrentPosition(p=>{
      const pos={lat:p.coords.latitude,lng:p.coords.longitude};
      const latLng=new google.maps.LatLng(pos.lat,pos.lng);
      if(!phuketBounds.contains(latLng)) return alert('ตำแหน่งอยู่นอกภูเก็ต');
      if(!markerA) markerA=new AdvancedMarkerElement({map,position:latLng,title:'You'});
      markerA.position=latLng;
      map.panTo(latLng);
      if(markerB) computeRoutes(markerA.position, markerB.position);
    },()=>alert('ไม่สามารถรับตำแหน่งของคุณได้'));
  });

  document.getElementById('btn-reset').addEventListener('click',()=>{
    lastRoutes=[]; polyLines.forEach(p=>p.setMap(null)); polyLines=[];
    if(markerA){markerA.setMap(null);markerA=null;}
    if(markerB){markerB.setMap(null);markerB=null;}
    document.getElementById('search').value=''; document.getElementById('searchStart').value='';
    document.getElementById('routesList').textContent='ยังไม่มีเส้นทาง';
  });
  
  document.getElementById('btn-calc').addEventListener('click',()=>{
    if(markerA && markerB) computeRoutes(markerA.position, markerB.position);
    else alert('กรุณาเลือกต้นทางและปลายทางก่อน');
  });

  applyLanguage();
}

async function computeRoutes(origin,dest){
  const vehicleKey=document.getElementById('vehicle').value||'grabCar';
  try{
    const res=await directionsService.route({origin,destination:dest,travelMode:'DRIVING',provideRouteAlternatives:true});
    if(res.routes.length===0) return alert('No routes');
    polyLines.forEach(p=>p.setMap(null)); polyLines=[];
    lastRoutes=res.routes;
    renderRoutes(res.routes,vehicleKey);
  }catch(e){alert('Directions request failed');}
}

function renderRoutes(routes,vehicleKey){
  const container=document.getElementById('routesList'); container.innerHTML='';
  routes.forEach((r,i)=>{
    const path=r.overview_path.map(p=>({lat:p.lat(),lng:p.lng()}));
    const poly=new google.maps.Polyline({path,map,strokeColor:'#'+Math.floor(Math.random()*16777215).toString(16),strokeWeight:6,strokeOpacity:0.6});
    poly.routeIndex=i; polyLines.push(poly);
    poly.addListener('click',()=>{selectedRouteIndex=i; highlightSelected(); map.fitBounds(r.bounds);});
    const card=document.createElement('div'); card.className='route-card'; card.dataset.routeIndex=i;
    const distText=`${km(r.legs[0].distance.value)} กม.`; const durText=`${Math.round(r.legs[0].duration.value/60)} นาที`;
    const fare=calculateFare(r,vehicleKey);
    card.innerHTML=`<div class="route-left"><div class="route-title">เส้นทาง ${i+1}</div><div class="route-meta">${distText} | ${durText}</div></div><div class="fare-pill">${fare} ฿</div>`;
    card.onclick=()=>{drawPolyline(r,i); highlightSelected(); map.fitBounds(r.bounds);}
    container.appendChild(card);
  });
  selectedRouteIndex=0; drawPolyline(routes[0],0); highlightSelected();
}

function drawPolyline(route,index){
  polyLines.forEach((p,i)=>{p.setOptions({strokeOpacity:i===index?1:0.4,strokeWeight:i===index?8:6});});
}

function highlightSelected(){
  document.querySelectorAll('.route-card').forEach((c,i)=>c.classList.toggle('selected',i===selectedRouteIndex));
}

window.initMap=initMap;
</script>

<script src="https://maps.googleapis.com/maps-api-v3/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&callback=initMap" async defer></script>
<script type="module" src="https://unpkg.com/@googlemaps/marker/dist/marker.esm.js"></script>
<script type="module" src="https://unpkg.com/@googlemaps/places/dist/places.esm.js"></script>
</body>
</html>
