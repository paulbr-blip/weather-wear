<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Weather Wear</title>
<style>
  :root{
    --accent:#0077b6;
    --card:#ffffff;
    --muted:#6b7280;
    --bg1:#e6f7fb;
    --bg2:#f7fbff;
  }
  html,body{height:100%;margin:0}
  body{
    font-family: "Segoe UI", Roboto, Arial, sans-serif;
    background: linear-gradient(180deg,var(--bg1),var(--bg2));
    display:flex;
    align-items:flex-start;
    justify-content:center;
    padding:28px;
    color:#1f2937;
  }

  .app {
    width:100%;
    max-width:560px;
    background:var(--card);
    border-radius:14px;
    box-shadow:0 10px 30px rgba(2,6,23,0.08);
    padding:16px;
  }

  /* Title */
  header { text-align:center; margin-bottom:12px; }
 h1 {
  font-size: 28px;
  font-weight: 400;
  text-align: center;
  margin-bottom: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px; /* spacing between icon and text */
}

.weather-icon-title {
  font-size: 32px;  /* make icon slightly larger */
  display: inline-block;
}

.weather-text {
  background: linear-gradient(90deg, #00b4d8, #0077b6);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  letter-spacing: 1.5px;
  text-shadow: 1px 1px 3px rgba(0,0,0,0.15);
}


  /* controls */
  .controls { display:flex; gap:8px; margin:14px 0; align-items:center; }
  .controls input[type="text"]{
    flex:1; padding:10px 12px; border-radius:8px; border:1px solid #e6eef7; font-size:14px;
  }
  .controls button{
    padding:10px 14px; background:var(--accent); color:white; border:none; border-radius:8px; font-weight:600; cursor:pointer;
  }

  .gender {
    display:flex; gap:12px; justify-content:center; margin-bottom:10px;
  }
  .gender label{ color:var(--accent); font-weight:600; cursor:pointer; }
  .gender input{ margin-right:6px; accent-color:var(--accent); }

  /* current */
  .current {
    display:flex; gap:14px; align-items:center; padding:14px; border-radius:12px;
    background:linear-gradient(180deg,#fbfdff,#f3f9ff);
    box-shadow:0 4px 12px rgba(7,18,34,0.04);
  }
  .current .left { flex:1; text-align:left; }
  .location { font-weight:700; color:var(--accent); font-size:17px; margin-bottom:4px; }
  .date { color:var(--muted); font-size:13px; margin-bottom:8px; }
  .temp { font-size:26px; font-weight:700; margin:0; }
  .cond { color:var(--muted); margin:8px 0; }
  .icon { width:74px; height:74px; object-fit:contain; flex:0 0 74px; }

  .today-details { font-size:13px; color:#333; margin-top:8px; }
  .today-details .precip { color:var(--accent); font-weight:700; }

  /* forecast */
  .forecast { margin-top:14px; display:grid; gap:10px; }
  .day-card {
    display:flex; align-items:center; justify-content:space-between; gap:12px;
    background:#fbfdff; border-radius:10px; padding:10px; border:1px solid #eef6ff;
  }
  .day-left { flex:1; text-align:left; }
  .day-title { font-weight:700; color:var(--accent); margin:0 0 6px 0; }
  .day-desc { font-size:13px; color:var(--muted); margin:0 0 6px 0; }
  .day-stats { font-size:13px; color:#333; margin:0; }
  .day-icon img { width:74px; height:74px; object-fit:contain; display:block; }
  .clothing { margin-top:8px; font-weight:700; color:#222; }

  footer { text-align:center; color:var(--muted); font-size:12px; margin-top:12px; }
  @media (max-width:520px){
    .current{flex-direction:row}
    .controls{flex-direction:column}
    .controls button{width:100%}
    .day-icon img, .icon { width:60px; height:60px; }
  }
</style>
</head>
<body>
  <main class="app" role="main">
    <header>
      <h1>
  <span class="weather-icon-title">☀️</span>
  <span class="weather-text">Weather Wear</span>
</h1>
    </header>

    <section class="section controls" aria-label="search and gender">
      <div style="flex:1">
        <input id="city-input" type="text" placeholder="Enter another city (optional)"/>
      </div>
      <div>
        <button id="city-btn" type="button">Check</button>
      </div>
    </section>

    <section class="section gender" aria-label="gender">
      <label><input type="radio" name="gender" value="male" checked id="gender-male"> Male</label>
      <label><input type="radio" name="gender" value="female" id="gender-female"> Female</label>
    </section>

    <section id="weather-box" class="section" aria-live="polite" style="display:none">
      <div class="current" role="region" aria-label="current weather">
        <div class="left">
          <div id="location" class="location"></div>
          <div id="today-date" class="date"></div>
          <div><span id="temperature" class="temp"></span></div>
          <div id="condition" class="cond"></div>
          <div id="today-details" class="today-details"></div>
          <div id="today-suggestion" class="today-details" style="font-weight:700;margin-top:6px;color:#111"></div>
        </div>
        <img id="weather-icon" class="icon" alt="weather icon" src="" />
      </div>

      <h3 style="margin:14px 0 8px;color:var(--accent)">3-Day Forecast</h3>
      <div id="forecast" class="forecast" aria-label="three day forecast"></div>
    </section>

    <footer>Data provided by OpenWeatherMap</footer>
  </main>

<script>
/* ====== CONFIG ====== */
const apiKey = "e8a4713f168477cdbda3e75616aa8721"; // your key
let latestCurrent = null;
let latestForecast = null;

/* ====== HELPERS ====== */
const $ = id => document.getElementById(id);
const safeAvg = arr => (Array.isArray(arr) && arr.length ? arr.reduce((a,b)=>a+b,0)/arr.length : 0);
const getIconUrl = code => code ? `https://openweathermap.org/img/wn/${code}@2x.png` : "";
const capitalize = s => typeof s === 'string' && s.length ? s[0].toUpperCase() + s.slice(1) : s;

/* clothing according to user's rules */
function clothingFromFeelsAndPop(feelsLike, pop, gender){
  // feelsLike in °C, pop 0..1, gender 'male'|'female'
  let s = "";
  if (gender === 'male') {
    if (feelsLike < 0) s = "Winter coat and thermals";
    else if (feelsLike < 5) s = "Coat, sweater and base layer";
    else if (feelsLike < 10) s = "Light coat, sweater and base layer";
    else if (feelsLike < 15) s = "Light jacket, sweater or overshirt";
    else if (feelsLike < 20) s = "Shirt or overshirt";
    else if (feelsLike < 25) s = "T-shirt and light trousers";
    else s = "T-shirt and shorts";
  } else {
    // female suggestions
    if (feelsLike < 0) s = "Warm winter coat, thermal leggings and boots";
    else if (feelsLike < 5) s = "Coat, knit sweater, scarf and warm trousers";
    else if (feelsLike < 10) s = "Light coat, jumper and jeans";
    else if (feelsLike < 15) s = "Jacket with blouse or light sweater";
    else if (feelsLike < 20) s = "Long-sleeve top or light dress with tights";
    else if (feelsLike < 25) s = "T-shirt with skirt, jeans or light trousers";
    else s = "T-shirt with shorts or summer dress";
  }

  if (typeof pop === 'number' && pop >= 0.25) s += ", plus a raincoat";
  return s;
}

/* ====== RENDERING ====== */
function renderCurrent(currentData, forecastData){
  if(!currentData) return;
  $('weather-box').style.display = 'block';
  $('location').textContent = `${currentData.name || ''}${currentData.sys && currentData.sys.country ? ', ' + currentData.sys.country : ''}`;

  const now = new Date();
  $('today-date').textContent = now.toLocaleDateString(undefined, { weekday:'long', year:'numeric', month:'short', day:'numeric' });

  const tempText = (currentData.main && typeof currentData.main.temp === 'number') ? Math.round(currentData.main.temp) + '°C' : '—';
  $('temperature').textContent = 'Temperature: ' + tempText;

  const desc = currentData.weather && currentData.weather[0] ? capitalize(currentData.weather[0].description) : '';
  $('condition').textContent = desc;

  const iconCode = currentData.weather && currentData.weather[0] ? currentData.weather[0].icon : '';
  $('weather-icon').src = getIconUrl(iconCode);
  $('weather-icon').alt = desc || 'weather';

  // compute today's min/max, feels, pop from forecastData if available
  if(forecastData && Array.isArray(forecastData.list)){
    const todayKey = new Date().toLocaleDateString('en-CA');
    const items = forecastData.list.filter(i => new Date(i.dt * 1000).toLocaleDateString('en-CA') === todayKey);
    if(items.length){
      const temps = items.map(i => Number(i.main && i.main.temp) || NaN).filter(n=>!Number.isNaN(n));
      const feels = items.map(i => Number(i.main && i.main.feels_like) || NaN).filter(n=>!Number.isNaN(n));
      const pops = items.map(i => (typeof i.pop === 'number' ? i.pop : 0));

      const minT = temps.length ? Math.round(Math.min(...temps)) : null;
      const maxT = temps.length ? Math.round(Math.max(...temps)) : null;
      const avgFeels = Math.round(safeAvg(feels));
      const avgPop = safeAvg(pops);

      const precipEmoji = avgPop >= 0.25 ? ((currentData.weather && currentData.weather[0] && currentData.weather[0].main && currentData.weather[0].main.toLowerCase().includes('snow')) ? '❄️' : '💧') : '';

      $('today-details').innerHTML =
        `${minT !== null ? 'Min: ' + minT + '°C' : ''}${minT !== null ? ' | ' : ''}${maxT !== null ? 'Max: ' + maxT + '°C' : ''}` +
        ` | Feels like (avg): ${!Number.isNaN(avgFeels) ? avgFeels + '°C' : '—'}` +
        ` | Precip: <span class="precip">${Math.round(avgPop * 100)}% ${precipEmoji}</span>`;

      // set clothing suggestion based on selected gender
      const gender = document.querySelector('input[name="gender"]:checked').value;
      $('today-suggestion').textContent = 'Clothing suggestion: ' + clothingFromFeelsAndPop(avgFeels, avgPop, gender);
    } else {
      // no todays items
      $('today-details').textContent = '';
      const feels = currentData.main && typeof currentData.main.feels_like === 'number' ? Math.round(currentData.main.feels_like) : null;
      const gender = document.querySelector('input[name="gender"]:checked').value;
      if(feels !== null) $('today-suggestion').textContent = 'Clothing suggestion: ' + clothingFromFeelsAndPop(feels, 0, gender);
      else $('today-suggestion').textContent = '';
    }
  } else {
    // no forecast data: fall back to current feels_like
    $('today-details').textContent = '';
    const feels = currentData.main && typeof currentData.main.feels_like === 'number' ? Math.round(currentData.main.feels_like) : null;
    const gender = document.querySelector('input[name="gender"]:checked').value;
    if(feels !== null) $('today-suggestion').textContent = 'Clothing suggestion: ' + clothingFromFeelsAndPop(feels, 0, gender);
    else $('today-suggestion').textContent = '';
  }
}

function renderForecast(forecastData){
  const container = $('forecast');
  container.innerHTML = '';
  if(!forecastData || !Array.isArray(forecastData.list) || forecastData.list.length === 0){
    const el = document.createElement('div');
    el.textContent = 'Forecast unavailable';
    container.appendChild(el);
    return;
  }

  // group by local date key YYYY-MM-DD
  const days = {};
  forecastData.list.forEach(item => {
    const key = new Date(item.dt * 1000).toLocaleDateString('en-CA');
    if(!days[key]) days[key] = [];
    days[key].push(item);
  });

  const keys = Object.keys(days).sort();
  const todayKey = new Date().toLocaleDateString('en-CA');
  const upcoming = keys.filter(k=>k !== todayKey).slice(0,3); // next 3 days

  upcoming.forEach(key => {
    const items = days[key];
    const temps = items.map(i => Number(i.main && i.main.temp) || NaN).filter(n=>!Number.isNaN(n));
    const feels = items.map(i => Number(i.main && i.main.feels_like) || NaN).filter(n=>!Number.isNaN(n));
    const pops = items.map(i => (typeof i.pop === 'number' ? i.pop : 0));

    const minT = temps.length ? Math.round(Math.min(...temps)) : null;
    const maxT = temps.length ? Math.round(Math.max(...temps)) : null;
    const avgFeels = Math.round(safeAvg(feels));
    const avgPop = safeAvg(pops);

    // midday item for icon + desc
    let midday = items[0];
    try {
      midday = items.reduce((prev,curr) => {
        const ph = new Date(prev.dt*1000).getHours();
        const ch = new Date(curr.dt*1000).getHours();
        return Math.abs(ch - 12) < Math.abs(ph - 12) ? curr : prev;
      }, items[0]);
    } catch(e) { midday = items[0]; }

    const desc = midday && midday.weather && midday.weather[0] ? capitalize(midday.weather[0].description) : '';
    const icon = midday && midday.weather && midday.weather[0] ? midday.weather[0].icon : '';

    const precipEmoji = avgPop >= 0.25 ? (midday && midday.weather && midday.weather[0] && midday.weather[0].main && midday.weather[0].main.toLowerCase().includes('snow') ? '❄️' : '💧') : '';

    // clothing using selected gender
    const gender = document.querySelector('input[name="gender"]:checked').value;
    const clothing = clothingFromFeelsAndPop(avgFeels, avgPop, gender);

    // build card
    const card = document.createElement('div');
    card.className = 'day-card';
    const dateObj = new Date(key);
    const title = dateObj.toLocaleDateString(undefined, { weekday:'short', month:'short', day:'numeric' });

    card.innerHTML = `
      <div class="day-left">
        <div class="day-title">${title}</div>
        <div class="day-desc">${desc}</div>
        <div class="day-stats">
          ${minT !== null ? 'Min: ' + minT + '°C' : ''}${minT !== null ? ' | ' : ''}${maxT !== null ? 'Max: ' + maxT + '°C' : ''}
          <br/>
          Feels like (avg): ${!Number.isNaN(avgFeels) ? avgFeels + '°C' : '—'}
          <br/>
          Precip: <span class="precip">${Math.round(avgPop * 100)}% ${precipEmoji}</span>
        </div>
        <div class="clothing">${clothing}</div>
      </div>
      <div class="day-icon"><img src="${getIconUrl(icon)}" alt="${desc}" /></div>
    `;
    container.appendChild(card);
  });
}

/* ====== FETCHING ====== */
function fetchByCoords(lat, lon){
  // fetch current first
  const currentUrl = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=metric&appid=${apiKey}`;
  fetch(currentUrl)
    .then(r => r.json())
    .then(currentData => {
      if(!currentData || (currentData.cod && Number(currentData.cod) !== 200)) {
        alert('Could not fetch current weather');
        return;
      }
      latestCurrent = currentData;
      // fetch forecast
      const fUrl = `https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&units=metric&appid=${apiKey}`;
      return fetch(fUrl).then(r=>r.json()).then(forecastData => {
        latestForecast = forecastData;
        renderCurrent(latestCurrent, latestForecast);
        renderForecast(latestForecast);
      });
    })
    .catch(err => {
      console.error(err);
      alert('Network error fetching weather');
    });
}

function fetchByCity(city){
  // geocode direct to get coords, then fetch by coords
  const geoUrl = `https://api.openweathermap.org/geo/1.0/direct?q=${encodeURIComponent(city)}&limit=1&appid=${apiKey}`;
  fetch(geoUrl)
    .then(r => r.json())
    .then(list => {
      if(!Array.isArray(list) || list.length === 0) {
        alert('City not found');
        return;
      }
      const { lat, lon, name, country } = list[0];
      fetchByCoords(lat, lon);
    })
    .catch(err => {
      console.error(err);
      alert('Error looking up city');
    });
}

/* ====== UI events ====== */
$('city-btn').addEventListener('click', ()=> {
  const city = $('city-input').value.trim();
  if(city) fetchByCity(city);
  else alert('Enter a city name or allow location');
});
$('city-input').addEventListener('keydown', e => {
  if(e.key === 'Enter') $('city-btn').click();
});

// on gender change, re-render suggestions if we have data
document.querySelectorAll('input[name="gender"]').forEach(r => {
  r.addEventListener('change', ()=> {
    if(latestCurrent) renderCurrent(latestCurrent, latestForecast);
    if(latestForecast) renderForecast(latestForecast);
  });
});

/* Try geolocation on load */
if(navigator.geolocation){
  navigator.geolocation.getCurrentPosition(pos => {
    fetchByCoords(pos.coords.latitude, pos.coords.longitude);
  }, err => {
    // do nothing; user can enter a city
    console.warn('Geolocation not available or denied');
  }, { timeout:10000 });
} else {
  console.warn('Geolocation not supported');
}

</script>
</body>
</html>
