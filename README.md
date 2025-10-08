<html lang="th">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Phuket Trip — Route & Fare (Modern)</title>
<style>
:root{
  --accent:#1e88e5; --muted:#6b7280; --card:#ffffff; --bg:#f6f8fb; --rounded:12px;
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
  <aside class="sidebar" aria-label="controls">
    <div class="logo">Trip Roule — Phuket demo
      <button id="btn-lang">🌐 English</button>
    </div>

    <div class="section-title" id="lbl-origin">ต้นทาง</div>
    <div class="search-row">
      <input id="searchStart" placeholder="เลือกต้นทาง..." aria-label="ต้นทาง" is="google-map-autocomplete">
      <button id="btn-current" class="btn alt" title="ใช้ตำแหน่งปัจจุบัน" aria-label="ใช้ตำแหน่งปัจจุบัน">📍</button>
    </div>

    <div class="section-title" id="lbl-dest">ปลายทาง</div>
    <div class="search-row">
      <input id="search" placeholder="ค้นหาปลายทาง..." aria-label="ปลายทาง" is="google-map-autocomplete">
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
/* -------------- Data & I18n -------------- */
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
  en:{ grabCar:"GrabCar", grabBike:"GrabBike", boltEconomy:"Bolt Economy", boltStandard:"Bolt Standard", boltVan:"Bolt Van", boltXL:"Bolt XL", boltTaxi:"Bolt Taxi", inDrive:"inDrive", taxiMeter:"Taxi Meter", smartBus:"Smart Bus", songthaew:"Songthaew", pinkBus:"Pink Bus", tukTuk:"TukTuk", motoTaxi:"MotoTaxi"}
};
const i18n = { th:{ onlyPhuket:"เฉพาะพื้นที่ภูเก็ตเท่านั้น", outsidePhuket:"อยู่นอกพื้นที่ภูเก็ต"}, en:{ onlyPhuket:"Only Phuket area", outsidePhuket:"Outside Phuket"}};
let currentLang = 'th';

/* -------------- Map & Places (Modern) -------------- */
let map,markerA,markerB,currentPos,trafficLayer;
async function initMap(){
  const phuketBounds = new google.maps.LatLngBounds({lat:7.7,lng:98.2},{lat:8.1,lng:98.6});
  const center = {lat:7.8804,lng:98.3923};
  map = new google.maps.Map(document.getElementById('map'), {center, zoom:11, mapTypeControl:false, streetViewControl:false});
  trafficLayer = new google.maps.TrafficLayer(); trafficLayer.setMap(map);

  // AdvancedMarkerElement
  function createMarker(loc,type='A'){
    return new google.maps.marker.AdvancedMarkerElement({
      map: map,
      position: loc,
      content: type==='A'? `<div style="font-size:20px">📍</div>` : `<div style="font-size:20px">📌</div>`
    });
  }

  // Autocomplete (Web Component)
  const acStart = document.getElementById('searchStart');
  acStart.addEventListener('place-changed', e=>{
    const place = e.detail.place;
    if(place?.geometry?.location){
      const loc = place.geometry.location;
      if(!phuketBounds.contains(loc)){ alert(i18n[currentLang].onlyPhuket); return; }
      if(!markerA) markerA=createMarker(loc,'A'); else markerA.position=loc;
      map.panTo(loc);
      currentPos = {lat:loc.lat(), lng:loc.lng()};
      if(markerB) computeRoutes(currentPos, markerB.position);
    }
  });

  const acEnd = document.getElementById('search');
  acEnd.addEventListener('place-changed', e=>{
    const place = e.detail.place;
    if(place?.geometry?.location){
      const loc = place.geometry.location;
      if(!phuketBounds.contains(loc)){ alert(i18n[currentLang].onlyPhuket); return; }
      if(!markerB) markerB=createMarker(loc,'B'); else markerB.position=loc;
      map.panTo(loc);
      if(currentPos) computeRoutes(currentPos, loc);
    }
  });

  // Text Search example
  async function triggerTextSearch(txt){
    if(!txt) return;
    const service = new google.maps.places.PlacesService(map);
    try{
      const results = await service.findPlaceFromQuery({query: txt, bounds: map.getBounds()});
      if(results && results.length>0){
        const loc = results[0].geometry.location;
        if(!markerB) markerB=createMarker(loc,'B'); else markerB.position=loc;
        map.panTo(loc);
        if(currentPos) computeRoutes(currentPos, loc);
      } else alert('No results found');
    }catch(err){console.error(err); alert('Search failed');}
  }

  // Current Location
  document.getElementById('btn-current').addEventListener('click', ()=>{
    if(!navigator.geolocation){ alert('Geolocation not supported'); return; }
    navigator.geolocation.getCurrentPosition(p=>{
      currentPos = {lat:p.coords.latitude, lng:p.coords.longitude};
      const pos = new google.maps.LatLng(currentPos.lat,currentPos.lng);
      if(!phuketBounds.contains(pos)){ alert(i18n[currentLang].outsidePhuket); return; }
      if(!markerA) markerA=createMarker(pos,'A'); else markerA.position=pos;
      map.panTo(pos);
      if(markerB) computeRoutes(currentPos, markerB.position);
    }, ()=>{ alert('Unable to retrieve your location'); });
  });
}

/* -------------- Route & Fare Calculation -------------- */
function computeRoutes(from,to){
  if(!from||!to) return;
  const distanceKm = haversine(from,to);
  displayRoutes(distanceKm);
}

function haversine(a,b){ // km
  const R=6371, dLat=(b.lat-a.lat)*Math.PI/180, dLng=(b.lng-a.lng)*Math.PI/180;
  const lat1=a.lat*Math.PI/180, lat2=b.lat*Math.PI/180;
  const x=Math.sin(dLat/2)**2 + Math.sin(dLng/2)**2 * Math.cos(lat1)*Math.cos(lat2);
  return R*2*Math.asin(Math.sqrt(x));
}

function displayRoutes(distance){
  const routesList=document.getElementById('routesList'); routesList.innerHTML='';
  Object.keys(vehicleRates).forEach(v=>{
    const rate=vehicleRates[v]; let fare;
    if(rate.fixed) fare=rate.min;
    else fare=Math.max(rate.min, rate.base + Math.max(0,distance-rate.freeKm)*rate.perKm);
    const card=document.createElement('div'); card.className='route-card';
    card.innerHTML=`<div class="route-left"><div class="route-title">${vehicleNames[currentLang][v]}</div><div class="route-meta">${distance.toFixed(1)} km</div></div><div class="fare-pill">${fare} ฿</div>`;
    card.addEventListener('click', ()=>{document.querySelectorAll('.route-card').forEach(c=>c.classList.remove('selected')); card.classList.add('selected');});
    routesList.appendChild(card);
  });
}

/* -------------- UI Controls -------------- */
document.getElementById('btn-toggle-fare').addEventListener('click', ()=>{const t=document.getElementById('fareTable'); t.style.display=(t.style.display==='none'?'block':'none');});

document.getElementById('btn-lang').addEventListener('click', ()=>{
  currentLang=currentLang==='th'?'en':'th';
  document.getElementById('btn-lang').textContent=currentLang==='th'?'🌐 English':'🌐 ไทย';
  document.getElementById('lbl-origin').textContent=currentLang==='th'?'ต้นทาง':'Origin';
  document.getElementById('lbl-dest').textContent=currentLang==='th'?'ปลายทาง':'Destination';
  document.getElementById('lbl-recommend').textContent=currentLang==='th'?'เส้นทางที่แนะนำ':'Recommended Routes';
  document.getElementById('legend-fast').textContent=currentLang==='th'?'เร็ว':'Fast';
  document.getElementById('legend-moderate').textContent=currentLang==='th'?'ปานกลาง':'Moderate';
  document.getElementById('legend-heavy').textContent=currentLang==='th'?'ช้า':'Heavy';
  const sel=document.getElementById('vehicle'); sel.innerHTML=''; Object.keys(vehicleNames[currentLang]).forEach(v=>{const o=document.createElement('option'); o.value=v;o.textContent=vehicleNames[currentLang][v]; sel.appendChild(o);});
});

document.getElementById('btn-reset').addEventListener('click', ()=>{
  markerA&&markerA.map=null; markerB&&markerB.map=null;
  markerA=markerB=null; currentPos=null; document.getElementById('search').value=''; document.getElementById('searchStart').value='';
  document.getElementById('routesList').innerHTML=''; document.getElementById('fareTable').style.display='none';
});

window.initMap=initMap;
</script>

<!-- Load Google Maps JS + Places -->
<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places&callback=initMap&libraries=places,marker"></script>
</body>
</html>
