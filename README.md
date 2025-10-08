<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Phuket Trip — Route & Fare</title>
<style>
:root{
  --accent:#1e88e5; --muted:#6b7280; --card:#fff; --bg:#f6f8fb; --rounded:12px;
}
html,body{margin:0;height:100%;font-family:sans-serif;background:var(--bg)}
.wrap{display:flex;height:100vh;gap:12px;padding:12px;box-sizing:border-box}
.map-wrap{flex:1;position:relative;border-radius:var(--rounded);overflow:hidden;box-shadow:0 8px 24px rgba(0,0,0,.06)}
#map{width:100%;height:100%}
.sidebar{width:380px;background:var(--card);border-radius:var(--rounded);padding:12px;display:flex;flex-direction:column;gap:8px;overflow:auto;box-shadow:0 8px 24px rgba(0,0,0,.08)}
.btn{padding:8px 12px;border-radius:10px;border:0;background:var(--accent);color:#fff;cursor:pointer}
.btn.alt{background:#eef;color:var(--accent);border:1px solid #d6e6ff}
.route-card{padding:8px;border:1px solid #eef;border-radius:10px;display:flex;justify-content:space-between;cursor:pointer}
.route-card.selected{outline:3px solid rgba(30,136,229,.12)}
.fare-table{position:absolute;right:12px;bottom:12px;background:var(--card);padding:12px;border-radius:12px;box-shadow:0 8px 24px rgba(0,0,0,.12);display:none;z-index:10;max-width:90vw;overflow:auto}
.fare-table table{width:100%;border-collapse:collapse;font-size:14px}
.fare-table th{font-weight:700;text-align:left;padding:4px;color:#123}
.fare-table td{padding:4px;color:#334}
</style>
</head>
<body>
<div class="wrap">
  <aside class="sidebar">
    <div>
      <strong>Trip Route — Phuket demo</strong>
      <button id="btn-lang" class="btn alt">🌐 English</button>
    </div>
    <div id="lbl-origin">ต้นทาง</div>
    <input id="searchStart" placeholder="เลือกต้นทาง..." style="width:100%;padding:8px"/>
    <button id="btn-current" class="btn alt">📍 ใช้ตำแหน่งปัจจุบัน</button>
    <div id="lbl-dest">ปลายทาง</div>
    <input id="search" placeholder="ค้นหาปลายทาง..." style="width:100%;padding:8px"/>
    <div>
      <select id="vehicle" style="width:100%;padding:6px;margin-top:6px"></select>
      <button id="btn-calc" class="btn">คำนวณ</button>
      <button id="btn-reset" class="btn alt">รีเซ็ต</button>
    </div>
    <button id="btn-toggle-fare" class="btn alt">แสดง/ซ่อนตารางราคา</button>
    <div id="lbl-recommend">เส้นทางที่แนะนำ</div>
    <div id="routesList" style="max-height:30vh;overflow:auto">ยังไม่มีเส้นทาง</div>
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
  </main>
</div>

<script>
/* ---------- Data ---------- */
const vehicleRates = {
  grabCar:{base:35,perKm:10,freeKm:2,min:50},
  grabBike:{base:20,perKm:7,freeKm:1,min:25},
  boltEconomy:{base:50,perKm:10,freeKm:2,min:100},
  boltStandard:{base:100,perKm:12,freeKm:2,min:200},
  smartBus:{base:100,perKm:0,freeKm:0,min:100,fixed:true}
};
const vehicleNames = {
  th:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard", smartBus:"Smart Bus" },
  en:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard", smartBus:"Smart Bus" }
};
const i18n = {
  th:{ search:"ค้นหาปลายทาง...", start:"เลือกต้นทาง...", originLabel:"ต้นทาง", destinationLabel:"ปลายทาง", calc:"คำนวณ", reset:"รีเซ็ต", fareTable:"ตารางราคา", noRoute:"ยังไม่มีเส้นทาง", toggleFare:"แสดง/ซ่อนตารางราคา", routeLabel:"เส้นทาง", kmLabel:"กม.", minsLabel:"นาที" },
  en:{ search:"Search destination...", start:"Select origin...", originLabel:"Origin", destinationLabel:"Destination", calc:"Calculate", reset:"Reset", fareTable:"Fare Table", noRoute:"No routes yet", toggleFare:"Show/Hide Fare Table", routeLabel:"Route", kmLabel:"km", minsLabel:"mins" }
};
let currentLang='th';

/* ---------- State ---------- */
let map, directionsService;
let markerA=null, markerB=null, currentPos=null;
let lastRoutes=[], selectedRouteIndex=0, polyLines=[];

/* ---------- Helpers ---------- */
function km(m){ return (m/1000).toFixed(1); }
function calculateFare(route, vehicleKey){ 
  const v = vehicleRates[vehicleKey]; if(!v) return 0;
  if(v.fixed) return v.min;
  const kmDistance = route.legs[0].distance.value/1000;
  return Math.max(Math.round(v.base + Math.max(0, kmDistance-v.freeKm)*v.perKm), v.min);
}

/* ---------- UI ---------- */
function applyLanguage(){
  document.getElementById("btn-lang").textContent = currentLang==="th" ? "🌐 English" : "🌐 ภาษาไทย";
  document.getElementById("search").placeholder = i18n[currentLang].search;
  document.getElementById("searchStart").placeholder = i18n[currentLang].start;
  document.getElementById("btn-calc").textContent = i18n[currentLang].calc;
  document.getElementById("btn-reset").textContent = i18n[currentLang].reset;
  document.getElementById("fare-title").textContent = i18n[currentLang].fareTable;
  document.getElementById("lbl-origin").textContent = i18n[currentLang].originLabel;
  document.getElementById("lbl-dest").textContent = i18n[currentLang].destinationLabel;
  document.getElementById("lbl-recommend").textContent = i18n[currentLang].routeLabel;
  document.getElementById("th-vehicle").textContent = i18n[currentLang].routeLabel;
  document.getElementById("th-base").textContent = "Base";
  document.getElementById("th-perkm").textContent = "Per km";
  document.getElementById("btn-toggle-fare").textContent = i18n[currentLang].toggleFare;

  const sel=document.getElementById("vehicle"); sel.innerHTML="";
  for(const k in vehicleRates){
    const opt=document.createElement("option");
    opt.value=k; opt.textContent=vehicleNames[currentLang][k]; sel.appendChild(opt);
  }
  sel.value='grabCar';
  renderFareTable();
}
function renderFareTable(){
  const tbody=document.getElementById('fareRows'); tbody.innerHTML='';
  for(const k in vehicleRates){
    const v=vehicleRates[k];
    const row=document.createElement('tr');
    if(v.fixed){ row.innerHTML=`<td>${vehicleNames[currentLang][k]}</td><td>${v.min} ฿</td><td>fixed</td>`;}
    else row.innerHTML=`<td>${vehicleNames[currentLang][k]}</td><td>${v.base} ฿</td><td>${v.perKm} ฿</td>`;
    tbody.appendChild(row);
  }
}

/* ---------- Map & Directions ---------- */
function initMap(){
  const phuket={lat:7.8804,lng:98.3923};
  map=new google.maps.Map(document.getElementById("map"),{center:phuket,zoom:11});
  directionsService=new google.maps.DirectionsService();

  // Markers
  markerA = new google.maps.marker.AdvancedMarkerElement({map, position:phuket, content:'📍A'});

  // Autocomplete origin
  const acStart = new google.maps.places.Autocomplete(document.getElementById('searchStart'), {componentRestrictions:{country:'th'}});
  acStart.addListener('place_changed',()=>{
    const place=acStart.getPlace();
    if(place.geometry){ markerA.position=place.geometry.location; map.panTo(place.geometry.location); currentPos={lat:place.geometry.location.lat(),lng:place.geometry.location.lng()};
      if(markerB) computeRoutes(markerA.position, markerB.position);
    }
  });

  // Autocomplete destination
  const acEnd = new google.maps.places.Autocomplete(document.getElementById('search'), {componentRestrictions:{country:'th'}});
  acEnd.addListener('place_changed',()=>{
    const place=acEnd.getPlace();
    if(place.geometry){
      if(!markerB) markerB=new google.maps.marker.AdvancedMarkerElement({map,position:place.geometry.location,content:'📍B'});
      else markerB.position=place.geometry.location;
      if(currentPos) computeRoutes(markerA.position, markerB.position);
    }
  });

  document.getElementById('btn-current').addEventListener('click',()=>{
    if(navigator.geolocation){navigator.geolocation.getCurrentPosition(p=>{
      const pos={lat:p.coords.latitude,lng:p.coords.longitude};
      markerA.position=pos; map.panTo(pos); currentPos=pos;
      if(markerB) computeRoutes(markerA.position, markerB.position);
    });}
  });

  document.getElementById('btn-calc').addEventListener('click',()=>{
    if(currentPos && markerB) computeRoutes(markerA.position, markerB.position);
    else alert(i18n[currentLang].noRoute);
  });

  document.getElementById('btn-reset').addEventListener('click',()=>{
    document.getElementById('search').value=''; document.getElementById('searchStart').value='';
    lastRoutes=[]; polyLines.forEach(p=>p.setMap(null)); polyLines=[];
    if(markerA) markerA.setMap(null); markerA=null;
    if(markerB) markerB.setMap(null); markerB=null;
    document.getElementById('routesList').textContent=i18n[currentLang].noRoute;
    markerA=new google.maps.marker.AdvancedMarkerElement({map,position:phuket,content:'📍A'});
  });

  document.getElementById('btn-toggle-fare').addEventListener('click',()=>{
    const f=document.getElementById('fareTable'); f.style.display=f.style.display==='none'?'block':'none';
  });

  document.getElementById('btn-lang').addEventListener('click',()=>{ currentLang=currentLang==='th'?'en':'th'; applyLanguage(); });

  applyLanguage();
}

/* ---------- Compute Routes ---------- */
function computeRoutes(origin,dest){
  const vehicleKey=document.getElementById('vehicle').value;
  directionsService.route({origin,destination:dest,travelMode:google.maps.TravelMode.DRIVING,provideRouteAlternatives:true},(res,status)=>{
    if(status==='OK'){
      polyLines.forEach(p=>p.setMap(null)); polyLines=[];
      lastRoutes=res.routes; renderRoutes(res.routes,vehicleKey);
    }else alert('Directions request failed: '+status);
  });
}

function renderRoutes(routes,vehicleKey){
  const container=document.getElementById('routesList'); container.innerHTML='';
  routes.forEach((r,i)=>{
    const path=r.overview_path.map(p=>({lat:p.lat(),lng:p.lng()}));
    const poly=new google.maps.Polyline({path,map,strokeColor:'#1e88e5',strokeWeight:6,strokeOpacity:0.6});
    polyLines.push(poly);
    const fare=calculateFare(r,vehicleKey);
    const card=document.createElement('div'); card.className='route-card'; card.dataset.idx=i;
    card.innerHTML=`<div>${i18n[currentLang].routeLabel} ${i+1}</div><div>${fare} ฿</div>`;
    card.onclick=()=>{highlightRoute(i);};
    container.appendChild(card);
  });
}

function highlightRoute(idx){ 
  polyLines.forEach((p,i)=>p.setOptions({strokeOpacity:i===idx?1:0.4,strokeWeight:i===idx?8:6}));
  document.querySelectorAll('.route-card').forEach((c,i)=>c.classList.toggle('selected',i===idx));
}

window.initMap=initMap;
</script>
<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places,marker&callback=initMap" async defer></script>
</body>
</html>
