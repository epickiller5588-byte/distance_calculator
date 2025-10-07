<html lang="th">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Phuket Trip — Route & Fare</title>
<style>
:root{
  --accent:#1e88e5; --muted:#6b7280; --card:#ffffff; --bg:#f6f8fb; --rounded:12px;
}
h1,h2,h3{display:none;}
body,html{margin:0; padding:0; font-family:Inter,system-ui,-apple-system,"Sarabun",sans-serif; background:var(--bg); color:#112; height:100%;}
.wrap{display:flex; flex-direction:column; height:100vh;}
.map-wrap{flex:0 0 60%; position:relative; border-radius:var(--rounded); overflow:hidden;}
#map{width:100%; height:100%;}
.controls{flex:1; display:flex; flex-direction:column; gap:8px; padding:8px;}
.row{display:flex; gap:6px; flex-wrap:wrap;}
input,select,button{padding:8px 10px; font-size:14px; border-radius:8px; border:1px solid #e6e9ee;}
input{flex:1;}
select{flex:1;}
button{cursor:pointer; border:none; background:var(--accent); color:#fff;}
button.alt{background:#eef; color:var(--accent); border:1px solid #d6e6ff;}
.routes{flex:1; overflow:auto; margin-top:4px; display:flex; flex-direction:column; gap:6px;}
.route-card{background:#fbfdff; border-radius:8px; padding:8px; display:flex; justify-content:space-between; align-items:center; cursor:pointer; border:1px solid #eef;}
.route-card.selected{outline:2px solid rgba(30,136,229,.2);}
.fare-table{position:absolute; bottom:12px; right:12px; background:var(--card); padding:10px; border-radius:10px; box-shadow:0 6px 16px rgba(0,0,0,0.12); display:none; max-width:90vw; overflow:auto;}
.fare-table table{border-collapse:collapse; width:100%;}
.fare-table th,.fare-table td{padding:4px; text-align:left; font-size:13px;}
.legend{position:absolute; left:12px; bottom:12px; background:#fff; padding:6px 8px; border-radius:8px; border:1px solid #eef; display:flex; gap:6px; align-items:center; font-size:12px;}
.dot{width:24px; height:4px; border-radius:4px;}
.fast{background:linear-gradient(90deg,#2ecc71,#1faa4a);}
.moderate{background:linear-gradient(90deg,#f1c40f,#f39c12);}
.heavy{background:linear-gradient(90deg,#e74c3c,#c0392b);}
@media(max-width:600px){
  .row{flex-direction:column;}
  .map-wrap{flex:0 0 55%;}
}
</style>
</head>
<body>
<div class="wrap">
  <div class="map-wrap">
    <div id="map"></div>
    <div class="fare-table" id="fareTable">
      <table>
        <thead><tr><th id="th-vehicle">Vehicle</th><th id="th-base">Base</th><th id="th-perkm">Per km</th></tr></thead>
        <tbody id="fareRows"></tbody>
      </table>
    </div>
    <div class="legend" id="legend">
      <div class="dot fast"></div><span id="legend-fast">Fast</span>
      <div class="dot moderate"></div><span id="legend-moderate">Moderate</span>
      <div class="dot heavy"></div><span id="legend-heavy">Heavy</span>
    </div>
  </div>
  <div class="controls">
    <div class="row">
      <input id="searchStart" placeholder="เลือกต้นทาง...">
      <input id="search" placeholder="ค้นหาปลายทาง...">
      <button id="btn-current" class="alt">📍</button>
    </div>
    <div class="row">
      <select id="vehicle"></select>
      <button id="btn-calc">คำนวณ</button>
      <button id="btn-reset" class="alt">รีเซ็ต</button>
      <button id="btn-toggle-fare" class="alt">ตารางราคา</button>
      <button id="btn-lang">🌐 English</button>
    </div>
    <div class="routes" id="routesList">ยังไม่มีเส้นทาง</div>
  </div>
</div>

<script>
// ----- Data -----
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
const vehicleNames = {th:{grabCar:"GrabCar",grabBike:"GrabBike",boltEconomy:"Bolt Economy",boltStandard:"Bolt Standard",boltVan:"Bolt Van",boltXL:"Bolt XL",boltTaxi:"Bolt Taxi",inDrive:"inDrive",taxiMeter:"Taxi Meter",smartBus:"Smart Bus",songthaew:"รถสองแถว",pinkBus:"รถชมพู",tukTuk:"ตุ๊กตุ๊ก",motoTaxi:"มอเตอร์ไซค์รับจ้าง"},en:{grabCar:"GrabCar",grabBike:"GrabBike",boltEconomy:"Bolt Economy",boltStandard:"Bolt Standard",boltVan:"Bolt Van",boltXL:"Bolt XL",boltTaxi:"Bolt Taxi",inDrive:"inDrive",taxiMeter:"Taxi Meter",smartBus:"Smart Bus",songthaew:"Songthaew",pinkBus:"Pink Bus",tukTuk:"Tuk Tuk",motoTaxi:"Moto Taxi"}};
const i18n = {
  th:{search:"ค้นหาปลายทาง...", start:"เลือกต้นทาง...", calc:"คำนวณ", reset:"รีเซ็ต", toggleFare:"ตารางราคา", noRoute:"ยังไม่มีเส้นทาง", kmLabel:"กม.", minsLabel:"นาที", fast:"คล่องตัว", moderate:"ปานกลาง", heavy:"หนาแน่น"},
  en:{search:"Search destination...", start:"Select origin...", calc:"Calculate", reset:"Reset", toggleFare:"Fare Table", noRoute:"No routes yet", kmLabel:"km", minsLabel:"mins", fast:"Fast", moderate:"Moderate", heavy:"Heavy"}
};
let currentLang='th';

// ----- State -----
let map,directionsService,markerA,markerB,lastRoutes=[],selectedRouteIndex=0,polyLines=[],currentPos=null;

// ----- Helpers -----
function km(m){return (m/1000).toFixed(1);}
function calculateFare(route, vehicleKey){const v=vehicleRates[vehicleKey]; if(!v) return 0; if(v.fixed) return v.min; const kmDistance=route.legs[0].distance.value/1000; let fare=v.base+Math.max(0,kmDistance-v.freeKm)*v.perKm; return Math.max(Math.round(fare),v.min);}
function updateRouteFareLabels(){const vehicleKey=document.getElementById('vehicle').value||'grabCar';document.querySelectorAll('.route-card').forEach(card=>{const idx=parseInt(card.dataset.routeIndex,10); if(isNaN(idx)||!lastRoutes[idx]) return; const newFare=calculateFare(lastRoutes[idx],vehicleKey); const pill=card.querySelector('.fare-pill'); if(pill) pill.textContent=`${newFare} ฿`;});}

// ----- UI -----
function applyLanguage(){
  document.getElementById('search').placeholder=i18n[currentLang].search;
  document.getElementById('searchStart').placeholder=i18n[currentLang].start;
  document.getElementById('btn-calc').textContent=i18n[currentLang].calc;
  document.getElementById('btn-reset').textContent=i18n[currentLang].reset;
  document.getElementById('btn-toggle-fare').textContent=i18n[currentLang].toggleFare;
  document.getElementById('legend-fast').textContent=i18n[currentLang].fast; // Added legend translation
  document.getElementById('legend-moderate').textContent=i18n[currentLang].moderate; // Added legend translation
  document.getElementById('legend-heavy').textContent=i18n[currentLang].heavy; // Added legend translation

  const sel=document.getElementById('vehicle'); sel.innerHTML=''; for(const k in vehicleRates){const opt=document.createElement('option'); opt.value=k; opt.textContent=vehicleNames[currentLang][k]||k; sel.appendChild(opt);} sel.value='grabCar';
  renderFareTable();
}
function renderFareTable(){const tbody=document.getElementById('fareRows'); tbody.innerHTML=''; for(const k in vehicleRates){const v=vehicleRates[k]; const name=vehicleNames[currentLang][k]||k; const row=document.createElement('tr'); if(v.fixed){row.innerHTML=`<td>${name}</td><td>${v.min} ฿ (fixed)</td><td>-</td>`;}else{row.innerHTML=`<td>${name}</td><td>${v.base} ฿</td><td>${v.perKm} ฿</td>`;} tbody.appendChild(row);} document.getElementById('fareTable').style.display='none';}

// ----- Map -----
function initMap(){
  const center={lat:7.8804,lng:98.3923}; // Center on Phuket
  map=new google.maps.Map(document.getElementById('map'),{center, zoom:11, mapTypeControl:false, streetViewControl:false});
  directionsService=new google.maps.DirectionsService();

  // Create markers
  markerA = new google.maps.Marker({map, icon:'https://maps.gstatic.com/mapfiles/ms2/micons/blue-dot.png', title:"ต้นทาง"});
  markerB = new google.maps.Marker({map, icon:'https://maps.gstatic.com/mapfiles/ms2/micons/red-dot.png', title:"ปลายทาง"});

  // Autocomplete (Restricted to Thailand)
  const options = { componentRestrictions:{country:'th'} };

  const acStart=new google.maps.places.Autocomplete(document.getElementById('searchStart'), options);
  acStart.addListener('place_changed',()=>{
    const place=acStart.getPlace();
    if(place?.geometry?.location){
      const loc=place.geometry.location;
      markerA.setPosition(loc);
      markerA.setMap(map); // Ensure marker is visible
      currentPos={lat:loc.lat(), lng:loc.lng()};
      map.panTo(loc);
      if(markerB.getPosition()) computeRoutes(currentPos, markerB.getPosition());
    }
  });
  const acEnd=new google.maps.places.Autocomplete(document.getElementById('search'), options);
  acEnd.addListener('place_changed',()=>{
    const place=acEnd.getPlace();
    if(place?.geometry?.location){
      const loc=place.geometry.location;
      markerB.setPosition(loc);
      markerB.setMap(map); // Ensure marker is visible
      if(currentPos) computeRoutes(currentPos, markerB.getPosition());
      map.panTo(loc);
    }
  });

  // Get current location on load
  if(navigator.geolocation){
    navigator.geolocation.getCurrentPosition(p=>{
      currentPos={lat:p.coords.latitude,lng:p.coords.longitude};
      const pos=new google.maps.LatLng(currentPos.lat,currentPos.lng);
      markerA.setPosition(pos);
      markerA.setMap(map);
      map.setCenter(pos);
    },()=>{
      // Handle Geolocation failure on load (e.g., user denied)
      console.log('Geolocation failed or denied. Starting at map center.');
    });
  }

  // Buttons
  document.getElementById('btn-current').addEventListener('click',()=>{
    if(!navigator.geolocation){alert('Geolocation not supported by your browser'); return;}
    navigator.geolocation.getCurrentPosition(p=>{
      currentPos={lat:p.coords.latitude,lng:p.coords.longitude};
      const pos=new google.maps.LatLng(currentPos.lat,currentPos.lng);
      markerA.setPosition(pos);
      markerA.setMap(map); // Ensure marker is visible
      map.panTo(pos);
      // Optional: Reverse geocode to put location name in input field
      const geocoder = new google.maps.Geocoder();
      geocoder.geocode({ 'location': pos }, (results, status) => {
        if (status === 'OK' && results[0]) {
          document.getElementById('searchStart').value = results[0].formatted_address;
        }
      });
      if(markerB.getPosition()) computeRoutes(currentPos, markerB.getPosition());
    },()=>{alert('Unable to retrieve your location. Please check your browser permissions.');});
  });
  document.getElementById('btn-calc').addEventListener('click',()=>{
    if(currentPos && markerB.getPosition()){
      computeRoutes(currentPos, markerB.getPosition());
    } else {
      alert('กรุณาเลือกต้นทางและปลายทางก่อน (Please select both origin and destination)');
    }
  });
  document.getElementById('btn-reset').addEventListener('click',()=>{
    document.getElementById('search').value='';
    document.getElementById('searchStart').value='';
    lastRoutes=[]; selectedRouteIndex=0;
    document.getElementById('routesList').innerHTML=i18n[currentLang].noRoute;
    polyLines.forEach(p=>p.setMap(null)); polyLines=[];
    // Reset markers and current position state
    if(markerA) markerA.setMap(null); 
    if(markerB) markerB.setMap(null);
    currentPos = null; // Important: reset currentPos to force re-selection
  });
  document.getElementById('btn-toggle-fare').addEventListener('click',()=>{
    const f=document.getElementById('fareTable'); f.style.display=(f.style.display==='none')?'block':'none';
  });
  document.getElementById('btn-lang').addEventListener('click',()=>{
    currentLang=(currentLang==='th'?'en':'th'); 
    document.getElementById('btn-lang').textContent = currentLang === 'th' ? '🌐 English' : '🌐 ไทย';
    applyLanguage();
  });

  // Add listener to vehicle selection to update fares
  document.getElementById('vehicle').addEventListener('change', updateRouteFareLabels);

  applyLanguage();
}

// ----- Routing -----
function computeRoutes(origin,dest){
  const vehicleKey=document.getElementById('vehicle').value||'grabCar';
  directionsService.route({origin,destination:dest,travelMode:google.maps.TravelMode.DRIVING,provideRouteAlternatives:true},(res,status)=>{
    if(status==='OK'){ 
      polyLines.forEach(p=>p.setMap(null)); polyLines=[]; 
      lastRoutes=res.routes; 
      renderRoutes(res.routes, vehicleKey);
    }
    else {
      alert(`Directions request failed: ${status}. Check if Directions API is enabled.`);
      document.getElementById('routesList').innerHTML=i18n[currentLang].noRoute;
    }
  });
}
function renderRoutes(routes,vehicleKey){
  const container=document.getElementById('routesList'); container.innerHTML='';
  if(!routes||routes.length===0){container.textContent=i18n[currentLang].noRoute; return;}
  const colors=['#2ecc71','#f1c40f','#e74c3c','#1e88e5','#9b59b6']; // Colors for different routes
  routes.forEach((r,i)=>{
    const path=r.overview_path.map(p=>({lat:p.lat(),lng:p.lng()}));
    const poly=new google.maps.Polyline({path, map, strokeColor:colors[i%colors.length], strokeWeight:6, strokeOpacity:0.6});
    poly.routeIndex=i; polyLines.push(poly);
    poly.addListener('click',()=>{selectedRouteIndex=poly.routeIndex; drawPolyline(routes[selectedRouteIndex]); highlightSelected();});
    const card=document.createElement('div'); card.className='route-card'; card.dataset.routeIndex=i;
    const distText=`${km(r.legs[0].distance.value)} ${i18n[currentLang].kmLabel}`;
    const durText=`${Math.round(r.legs[0].duration.value/60)} ${i18n[currentLang].minsLabel}`;
    const fare=calculateFare(r,vehicleKey);
    card.innerHTML=`<div>${distText} | ${durText}</div><div class="fare-pill">${fare} ฿</div>`;
    card.onclick=()=>{selectedRouteIndex=i; drawPolyline(r); highlightSelected();};
    container.appendChild(card);
  });
  selectedRouteIndex=0; drawPolyline(routes[0]); highlightSelected();
}
function drawPolyline(route){polyLines.forEach((p,i)=>{p.setOptions({strokeOpacity:(i===selectedRouteIndex?1:0.4), strokeWeight:(i===selectedRouteIndex?8:6)});});}
function highlightSelected(){document.querySelectorAll('.route-card').forEach((c,i)=>{c.classList.toggle('selected',i===selectedRouteIndex);}); updateRouteFareLabels();}

window.initMap=initMap;
</script>
<script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDcAtU6iQwn7aUsNwCHST73U2pqKbImiJM&libraries=places&callback=initMap" async defer></script>
</body>
</html>
