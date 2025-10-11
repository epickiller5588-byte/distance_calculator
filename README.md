<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Phuket Trip — Route & Fare</title>
<style>
:root{
  --accent:#1e88e5; --muted:#6b7280; --card:#ffffff; --bg:#f6f8fb;
  --rounded:12px;
}
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
<aside class="sidebar">
  <div class="logo">Trip Route — Phuket Demo
    <button id="btn-lang">🌐 English</button>
  </div>
  <div class="section-title" id="lbl-origin">ต้นทาง</div>
  <div class="search-row">
    <input id="searchStart" placeholder="เลือกต้นทาง...">
    <button id="btn-current" class="btn alt">📍</button>
  </div>
  <div class="section-title" id="lbl-dest">ปลายทาง</div>
  <div class="search-row">
    <input id="search" placeholder="ค้นหาปลายทาง...">
  </div>
  <div class="controls-row">
    <select id="vehicle"></select>
    <button id="btn-calc" class="btn">คำนวณ</button>
    <button id="btn-reset" class="btn alt">รีเซ็ต</button>
  </div>
  <button id="btn-toggle-fare" class="btn alt">แสดง/ซ่อนตารางราคา</button>
  <div class="section-title" id="lbl-recommend">เส้นทางที่แนะนำ</div>
  <div class="routes" id="routesList"></div>
</aside>
<main class="map-wrap">
  <div id="map"></div>
  <div class="fare-table" id="fareTable">
    <div id="fare-title">ตารางราคา</div>
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
/* ---------- Data ---------- */
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

const vehicleNames = { th:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard", boltVan:"Bolt Van", boltXL:"Bolt XL", boltTaxi:"Bolt Taxi", inDrive:"inDrive", taxiMeter:"Taxi Meter", smartBus:"รถบัส Smart Bus", songthaew:"รถสองแถว", pinkBus:"รถบัสสีชมพู", tukTuk:"ตุ๊กตุ๊ก", motoTaxi:"มอเตอร์ไซค์รับจ้าง"}, en:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard", boltVan:"Bolt Van", boltXL:"Bolt XL", boltTaxi:"Bolt Taxi", inDrive:"inDrive", taxiMeter:"Taxi Meter", smartBus:"Smart Bus", songthaew:"Songthaew", pinkBus:"Pink Bus", tukTuk:"Tuk Tuk", motoTaxi:"Moto Taxi"}};

let currentLang='th';
let map, directionsService, trafficLayer, markerA=null, markerB=null, lastRoutes=[], polyLines=[];

/* ---------- Helpers ---------- */
function km(m){return (m/1000).toFixed(1);}
function calculateFare(route,key){
  const v = vehicleRates[key]; if(!v) return 0;
  if(v.fixed) return v.min;
  const kmDistance = route.legs[0].distance.value/1000;
  let fare = v.base + Math.max(0, kmDistance - v.freeKm)*v.perKm;
  return Math.max(Math.round(fare),v.min);
}

/* ---------- UI ---------- */
function applyLanguage(){
  document.getElementById("btn-lang").textContent=currentLang==="th"?"🌐 English":"🌐 ภาษาไทย";
  document.getElementById("search").placeholder=currentLang==="th"?"ค้นหาปลายทาง...":"Search destination...";
  document.getElementById("searchStart").placeholder=currentLang==="th"?"เลือกต้นทาง...":"Select origin...";
  document.getElementById("btn-calc").textContent=currentLang==="th"?"คำนวณ":"Calculate";
  document.getElementById("btn-reset").textContent=currentLang==="th"?"รีเซ็ต":"Reset";
  document.getElementById("fare-title").textContent=currentLang==="th"?"ตารางราคา":"Fare Table";
  document.getElementById("lbl-origin").textContent=currentLang==="th"?"ต้นทาง":"Origin";
  document.getElementById("lbl-dest").textContent=currentLang==="th"?"ปลายทาง":"Destination";
  document.getElementById("lbl-recommend").textContent=currentLang==="th"?"เส้นทางที่แนะนำ":"Route";
  document.getElementById("legend-title").textContent=currentLang==="th"?"การจราจร":"Traffic";
  document.getElementById("legend-fast").textContent=currentLang==="th"?"คล่องตัว":"Fast";
  document.getElementById("legend-moderate").textContent=currentLang==="th"?"ปานกลาง":"Moderate";
  document.getElementById("legend-heavy").textContent=currentLang==="th"?"หนาแน่น":"Heavy";
  document.getElementById("th-vehicle").textContent=currentLang==="th"?"ยานพาหนะ":"Vehicle";
  document.getElementById("th-base").textContent=currentLang==="th"?"ฐานเริ่มต้น":"Base";
  document.getElementById("th-perkm").textContent=currentLang==="th"?"ต่อ กม.":"Per km";

  const sel = document.getElementById("vehicle"); sel.innerHTML='';
  const ph=document.createElement("option"); ph.value=''; ph.textContent=currentLang==='th'?'-- เลือกประเภทยานพาหนะ --':'-- Select vehicle --'; ph.disabled=true; sel.appendChild(ph);
  for(const k in vehicleRates){const opt=document.createElement("option"); opt.value=k; opt.textContent=vehicleNames[currentLang][k]||k; sel.appendChild(opt);}
  sel.value='grabCar';

  const tbody=document.getElementById('fareRows'); tbody.innerHTML='';
  for(const k in vehicleRates){
    const v=vehicleRates[k]; const name=vehicleNames[currentLang][k]||k;
    const row=document.createElement('tr');
    row.innerHTML=v.fixed?`<td>${name}</td><td>${v.min} ฿ (fixed)</td><td>-</td>`:`<td>${name}</td><td>${v.base} ฿</td><td>${v.perKm} ฿</td>`;
    tbody.appendChild(row);
  }
}

/* ---------- Map ---------- */
function initMap(){
  const phuketBounds=new google.maps.LatLngBounds({lat:7.7,lng:98.2},{lat:8.1,lng:98.6});
  const center={lat:7.8804,lng:98.3923};
  map=new google.maps.Map(document.getElementById('map'),{center,zoom:11,mapTypeControl:false,streetViewControl:false});
  directionsService=new google.maps.DirectionsService();
  trafficLayer=new google.maps.TrafficLayer(); trafficLayer.setMap(map);

  const acStart=new google.maps.places.Autocomplete(document.getElementById('searchStart'),{bounds:phuketBounds,componentRestrictions:{country:'th'},fields:['place_id','geometry','name','formatted_address']});
  acStart.addListener('place_changed',()=>{
    const place=acStart.getPlace();
    if(place?.geometry?.location){
      const loc=place.geometry.location;
      if(!phuketBounds.contains(loc)){ alert('กรุณาเลือกตำแหน่งในภูเก็ต'); return; }
      if(!markerA) markerA=new google.maps.Marker({map,icon:{path:google.maps.SymbolPath.CIRCLE,scale:8,fillColor:'#1e88e5',fillOpacity:1,strokeWeight:0}});
      markerA.setPosition(loc); map.panTo(loc);
      if(markerB) computeRoutes(markerA.getPosition(), markerB.getPosition());
    }
  });

  const acEnd=new google.maps.places.Autocomplete(document.getElementById('search'),{bounds:phuketBounds,componentRestrictions:{country:'th'},fields:['place_id','geometry','name','formatted_address']});
  acEnd.addListener('place_changed',()=>{
    const place=acEnd.getPlace();
    if(place?.geometry?.location){
      const loc=place.geometry.location;
      if(!phuketBounds.contains(loc)){ alert('กรุณาเลือกตำแหน่งในภูเก็ต'); return; }
      if(!markerB) markerB=new google.maps.Marker({map,icon:'http://maps.google.com/mapfiles/ms/icons/red-dot.png'});
      markerB.setPosition(loc); map.panTo(loc);
      if(markerA) computeRoutes(markerA.getPosition(), markerB.getPosition());
    }
  });

  document.getElementById('btn-current').addEventListener('click',()=>{
    navigator.geolocation.getCurrentPosition(pos=>{
      const loc=new google.maps.LatLng(pos.coords.latitude,pos.coords.longitude);
      if(!phuketBounds.contains(loc)){ alert('คุณอยู่ไม่ในภูเก็ต'); return; }
      if(!markerA) markerA=new google.maps.Marker({map,icon:{path:google.maps.SymbolPath.CIRCLE,scale:8,fillColor:'#1e88e5',fillOpacity:1,strokeWeight:0}});
      markerA.setPosition(loc); map.panTo(loc);
      document.getElementById('searchStart').value='ตำแหน่งปัจจุบัน';
      if(markerB) computeRoutes(markerA.getPosition(), markerB.getPosition());
    },()=>alert('ไม่สามารถเข้าถึงตำแหน่งได้'));
  });

  document.getElementById('btn-calc').addEventListener('click',()=>{ if(markerA && markerB) computeRoutes(markerA.getPosition(), markerB.getPosition()); });
  document.getElementById('btn-reset').addEventListener('click',resetMap);
  document.getElementById('btn-toggle-fare').addEventListener('click',()=>{ const t=document.getElementById('fareTable'); t.style.display=t.style.display==='none'?'block':'none'; });
}

/* ---------- Routing ---------- */
function computeRoutes(origin,dest){
  directionsService.route({origin,destination:dest,travelMode:google.maps.TravelMode.DRIVING,provideRouteAlternatives:true},(res,status)=>{
    if(status!=='OK'){ alert('ไม่สามารถหาทางได้'); return; }
    clearPolylines(); lastRoutes=res.routes;
    const list=document.getElementById('routesList'); list.innerHTML='';
    res.routes.forEach((r,i)=>{
      const pl=new google.maps.Polyline({path:r.overview_path,strokeColor:'#1e88e5',strokeOpacity:0.6,strokeWeight:6,map:map});
      polyLines.push(pl);
      pl.addListener('click',()=>highlightRoute(i));
      const card=document.createElement('div'); card.className='route-card'; card.innerHTML=`<div class="route-left"><div class="route-title">${currentLang==='th'?'เส้นทาง':'Route'} ${i+1}</div><div class="route-meta">${km(r.legs[0].distance.value)} กม. — ${r.legs[0].duration.text}</div></div><div class="fare-pill">${calculateFare(r,document.getElementById('vehicle').value)} ฿</div>`;
      card.addEventListener('click',()=>highlightRoute(i));
      list.appendChild(card);
    });
  });
}

function clearPolylines(){ polyLines.forEach(p=>p.setMap(null)); polyLines=[]; lastRoutes=[]; document.getElementById('routesList').innerHTML=''; }
function highlightRoute(idx){ polyLines.forEach((p,i)=>p.setOptions({strokeOpacity:i===idx?0.9:0.4,strokeWeight:i===idx?8:6})); const cards=document.querySelectorAll('.route-card'); cards.forEach((c,i)=>c.classList.toggle('selected',i===idx)); }

/* ---------- Reset ---------- */
function resetMap(){
  if(markerA) markerA.setMap(null); markerA=null;
  if(markerB) markerB.setMap(null); markerB=null;
  clearPolylines();
  document.getElementById('search').value='';
  document.getElementById('searchStart').value='';
}

/* ---------- Init ---------- */
applyLanguage();
document.getElementById('btn-lang').addEventListener('click',()=>{ currentLang=currentLang==='th'?'en':'th'; applyLanguage(); });
</script>
<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places&v=weekly"></script>
</body>
</html>
