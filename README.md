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
/* ---------------- Data ---------------- */
const vehicleRates = { motorcycle:{perKm:10}, van:{perKm:15}, bus:{perKm:8} };
const vehicleNames = { th:{motorcycle:"มอเตอร์ไซค์",van:"รถตู้",bus:"รถบัส"}, en:{motorcycle:"Motorcycle",van:"Van",bus:"Bus"} };
const i18n = {
  th:{search:"ค้นหาปลายทาง...",start:"เลือกต้นทาง...",originLabel:"ต้นทาง",destinationLabel:"ปลายทาง",
      calc:"คำนวณ",reset:"รีเซ็ต",fareTable:"ตารางราคา",routeLabel:"เส้นทาง",kmLabel:"กม.",minsLabel:"นาที",
      traffic:"การจราจร",fast:"คล่องตัว",moderate:"ปานกลาง",heavy:"หนาแน่น",toggleFare:"แสดง/ซ่อนตารางราคา"},
  en:{search:"Search destination...",start:"Select origin...",originLabel:"Origin",destinationLabel:"Destination",
      calc:"Calculate",reset:"Reset",fareTable:"Fare Table",routeLabel:"Route",kmLabel:"km",minsLabel:"mins",
      traffic:"Traffic",fast:"Fast",moderate:"Moderate",heavy:"Heavy",toggleFare:"Show/Hide Fare Table"}
};
let currentLang='th';
let map, directionsService, trafficLayer, markerA=null, markerB=null, lastRoutes=[], selectedRouteIndex=0, polyLines=[];

/* ---------------- Helpers ---------------- */
function km(m){return (m/1000).toFixed(1);}
function calculateFare(route, vehicleKey){return Math.round(route.legs[0].distance.value/1000*vehicleRates[vehicleKey].perKm);}

/* ---------------- UI ---------------- */
function applyLanguage(){
  document.getElementById("btn-lang").textContent=currentLang==="th"?"🌐 English":"🌐 ภาษาไทย";
  document.getElementById("search").placeholder=i18n[currentLang].search;
  document.getElementById("searchStart").placeholder=i18n[currentLang].start;
  document.getElementById("btn-calc").textContent=i18n[currentLang].calc;
  document.getElementById("btn-reset").textContent=i18n[currentLang].reset;
  document.getElementById("fare-title").textContent=i18n[currentLang].fareTable;
  document.getElementById("lbl-origin").textContent=i18n[currentLang].originLabel;
  document.getElementById("lbl-dest").textContent=i18n[currentLang].destinationLabel;
  document.getElementById("lbl-recommend").textContent=i18n[currentLang].routeLabel;
  document.getElementById("legend-title").textContent=i18n[currentLang].traffic;
  document.getElementById("legend-fast").textContent=i18n[currentLang].fast;
  document.getElementById("legend-moderate").textContent=i18n[currentLang].moderate;
  document.getElementById("legend-heavy").textContent=i18n[currentLang].heavy;

  const sel=document.getElementById("vehicle"); sel.innerHTML="";
  const ph=document.createElement("option"); ph.value=""; ph.textContent=currentLang==='th'?'-- เลือกประเภทยานพาหนะ --':'-- Select vehicle --'; ph.disabled=true; sel.appendChild(ph);
  for(const k in vehicleRates){ const opt=document.createElement("option"); opt.value=k; opt.textContent=vehicleNames[currentLang][k]; sel.appendChild(opt); }
  sel.value="motorcycle";
}

/* ---------------- Map ---------------- */
function initMap(){
  const center={lat:7.8804,lng:98.3923};
  map=new google.maps.Map(document.getElementById("map"),{center,zoom:11,mapTypeControl:false,streetViewControl:false});
  directionsService=new google.maps.DirectionsService();
  trafficLayer=new google.maps.TrafficLayer(); trafficLayer.setMap(map);

  const acStart=new google.maps.places.Autocomplete(document.getElementById("searchStart"),{fields:['geometry','name']});
  acStart.addListener('place_changed',()=>{const place=acStart.getPlace(); if(place.geometry){const loc=place.geometry.location; if(!markerA) markerA=new google.maps.Marker({map,position:loc,title:"Origin"}); else markerA.setPosition(loc); map.panTo(loc); if(markerB) computeRoutes(loc,markerB.getPosition());}});
  const acEnd=new google.maps.places.Autocomplete(document.getElementById("search"),{fields:['geometry','name']});
  acEnd.addListener('place_changed',()=>{const place=acEnd.getPlace(); if(place.geometry){const loc=place.geometry.location; if(!markerB) markerB=new google.maps.Marker({map,position:loc,title:"Destination"}); else markerB.setPosition(loc); map.panTo(loc); if(markerA) computeRoutes(markerA.getPosition(),loc);}});

  document.getElementById("btn-calc").addEventListener("click",()=>{if(markerA && markerB) computeRoutes(markerA.getPosition(),markerB.getPosition());});
  document.getElementById("btn-lang").addEventListener("click",()=>{currentLang=currentLang==="th"?"en":"th";applyLanguage();});

  applyLanguage();
}

/* ---------------- Routing ---------------- */
function computeRoutes(origin,dest){
  directionsService.route({origin,destination:dest,travelMode:google.maps.TravelMode.DRIVING,provideRouteAlternatives:true},
    (res,status)=>{if(status==="OK"){polyLines.forEach(p=>p.setMap(null)); polyLines=[]; lastRoutes=res.routes; renderRoutes(res.routes); } else alert("Directions failed");});
}
function renderRoutes(routes){
  const container=document.getElementById("routesList"); container.innerHTML="";
  routes.forEach((r,i)=>{
    const poly=new google.maps.Polyline({path:r.overview_path.map(p=>({lat:p.lat(),lng:p.lng()})),map,strokeColor:"#1e88e5",strokeWeight:6,strokeOpacity:0.6});
    poly.routeIndex=i; polyLines.push(poly);
    const card=document.createElement("div"); card.className="route-card"; card.dataset.routeIndex=i;
    const distText=`${km(r.legs[0].distance.value)} ${i18n[currentLang].kmLabel}`; const durText=`${Math.round(r.legs[0].duration.value/60)} ${i18n[currentLang].minsLabel}`;
    const fare=calculateFare(r,document.getElementById("vehicle").value||"motorcycle");
    card.innerHTML=`<div class="route-left"><div class="route-title">${i18n[currentLang].routeLabel} ${i+1}</div><div class="route-meta">${distText} | ${durText}</div></div><div class="fare-pill">${fare} ฿</div>`;
    card.onclick=()=>{selectedRouteIndex=i; highlightSelected();};
    container.appendChild(card);
  });
  highlightSelected();
}
function highlightSelected(){document.querySelectorAll(".route-card").forEach((c,i)=>c.classList.toggle("selected",i===selectedRouteIndex));}
window.initMap=initMap;
</script>
<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places&callback=initMap" async defer></script>
</body>
</html>
