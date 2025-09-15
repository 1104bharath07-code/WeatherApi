<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title> animated Weather Dashboard with Favorites & Background</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: ;
      color: #333;
      padding: 2rem;
      min-height: 150vh;
    }
    .weather-container {
      background-color: white;
      background: linear-gradient(270deg, skyblue,rgb(255, 98, 0) , black);
      background-size: 600% 600%;
      animation: gradientShift 2s ease infinite;
      width: 450px;
      padding: 2rem;
      border-radius: 12px;
      box-shadow: 0 0 20px rgba(0,0,0,0.15);
      text-align: center;
      margin-right: 50rem;
      display: flex;
      flex-direction: column;
    }
    input[type="text"] {
      flex: 1;
      padding: 0.5rem;
      border-radius: 6px;
      border: 1px solid #fff8f8;
      font-size: 1rem;
      margin-right: 0.5rem;
    }
    .input-group {
      display: flex;
      margin-bottom: 0.8rem;
    }
    button {
      padding: 0.5rem 0.5rem;
      margin-left: 0.5rem;
      border: none; 
      background-color: #0680d7;
      color: white;
      border-radius: 6px;
      cursor: pointer;
      font-size: 1rem;
      transition: background-color 0.3s;
    }
    button:hover {
      background-color: #005fa3;
    }
    .current-weather {
      margin-top: 1.5rem;
      color: white;
    }
    .temp {
      font-size: 3rem;
      font-weight: bold;
      margin: 10px 0 5px 0;
      color: white;
    }
    .condition {
      font-size: 1.5rem;
      margin-bottom: 1rem;
      text-transform: capitalize;
      color: white;
    }
    .forecast {
      display: flex;
      justify-content: space-between;
      margin-top: 2rem;
      overflow-x: auto;
      flex: none;
      color: white;
      gap: 5px;
    }
    .day {
      width: 50px;
      color: white;
      text-align: center;
    }
    .day strong {
      display: block;
      margin-bottom: 0.5rem;
      font-size: 1rem;
      color: white;
    }
    .icon {
      width: 40px;
      height: 40px;
      margin-bottom: 0.5rem;
    }
    .favorites-container {
      margin-top: 1.5rem;
      text-align: left;
      flex: none;
    }
    .favorite-item {
        margin-left: 0.5rem;
      background: #cde0ff;
      padding: 5px 8px;
      border-radius: 6px;
      cursor: pointer;
      user-select: none;
      display: inline-flex;
      align-items: center;
      gap: 6px;
      font-size: 1rem;
      color: #004080;
    }
    .favorite-item:hover {
      background-color: #a2bbff;
    }
    .remove-fav-btn {
      background: #e57373;
      border-radius: 50%;
      border: none;
      color: white;
      font-weight: bold;
      cursor: pointer;
      width: 20px;
      height: 20px;
      line-height: 15px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 15px;
    }
    .unit-toggle-btn {
      margin-left: 0.5rem;
      padding: 1rem;
      font-size: 1rem;
      background-color: #444;
      border-radius: 6px;
      color: white;
    }
  </style>
</head>
<body>

  <div class="weather-container">
    <div class="input-group">
      <input id="cityInput" placeholder="Enter city" />
      <button id="searchBtn">Search</button>
      <button id="addFavoriteBtn" disabled>Add to Favorites</button>
      <button id="unitToggleBtn" class="unit-toggle-btn">°F</button>
    </div>
    <div class="current-weather" id="currentWeather">Current weather info will appear here</div>
    <div class="forecast" id="forecast"></div>
    <div class="favorites-container">
      <h3>Favorites</h3>
      <div id="favoritesList">No favorites added.</div>
    </div>
  </div>

<script>
  const apiKey = 'd9674c477b108e46eb480b45cb4329b9';

  const cityInput = document.getElementById('cityInput');
  const searchBtn = document.getElementById('searchBtn');
  const addFavoriteBtn = document.getElementById('addFavoriteBtn');
  const currentWeatherDiv = document.getElementById('currentWeather');
  const forecastDiv = document.getElementById('forecast');
  const favoritesList = document.getElementById('favoritesList');
  const unitToggleBtn = document.getElementById('unitToggleBtn');

  let isCelsius = true; // Track units: true = C, false = F
  let favorites = JSON.parse(localStorage.getItem('favorites')) || [];
  let currentCity = null;

  function celsiusToFahrenheit(c) {
    return (c * 9) / 5 + 32;
  }

  function formatTemp(tempC) {
    return isCelsius ? `${tempC.toFixed(1)}°C` : `${celsiusToFahrenheit(tempC).toFixed(1)}°F`;
  }

  function saveFavorites() {
    localStorage.setItem('favorites', JSON.stringify(favorites));
  }

  function renderFavorites() {
    favoritesList.innerHTML = '';
    if (favorites.length === 0) {
      favoritesList.textContent = 'No favorites added.';
      return;
    }
    favorites.forEach(city => {
      const fav = document.createElement('div');
      fav.className = 'favorite-item';
      fav.textContent = city;

      // Remove button
      const removeBtn = document.createElement('button');
      removeBtn.className = 'remove-fav-btn';
      removeBtn.textContent = '×';
      removeBtn.onclick = e => {
        e.stopPropagation(); // prevent parent click event
        favorites = favorites.filter(c => c !== city);
        saveFavorites();
        renderFavorites();
        if (city === currentCity) addFavoriteBtn.disabled = false;
      };
      fav.appendChild(removeBtn);

      fav.onclick = () => {
        cityInput.value = city;
        searchWeather(city);
      };

      favoritesList.appendChild(fav);
    });
  }

  function addCityToFavorites() {
    if (currentCity && !favorites.includes(currentCity)) {
      favorites.push(currentCity);
      saveFavorites();
      renderFavorites();
      addFavoriteBtn.disabled = true;
    }
  }

  async function searchWeather(city) {
    currentWeatherDiv.textContent = 'Loading...';
    forecastDiv.innerHTML = '';
    addFavoriteBtn.disabled = true;
    currentCity = null;

    try {
      let res = await fetch(`https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}`);
      let data = await res.json();
      if (data.cod !== 200) {
        currentWeatherDiv.textContent = data.message;
        return;
      }

      currentCity = data.name;
      addFavoriteBtn.disabled = favorites.includes(currentCity);
      displayCurrentWeather(data);
      res = await fetch(`https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apiKey}`);
      let forecastData = await res.json();
      if (forecastData.cod === "200") {
        displayForecast(forecastData);
      } else {
        forecastDiv.textContent = '';
      }
    } catch {
      currentWeatherDiv.textContent = 'Unable to fetch weather data';
      forecastDiv.textContent = '';
    }
  }

  function displayCurrentWeather(data) {
    const iconUrl = `https://openweathermap.org/img/wn/${data.weather[0].icon}@2x.png`;
    const tempC = data.main.temp - 273.15;
    currentWeatherDiv.innerHTML = `
      <img src="${iconUrl}" alt="${data.weather[0].description}" class="icon" />
      <div class="temp">${formatTemp(tempC)}</div>
      <div class="condition">${data.weather[0].description}</div>
      <div>${data.name}, ${data.sys.country}</div>
    `;
  }

  function displayForecast(data) {
    const filtered = data.list.filter(d => d.dt_txt.includes('12:00:00')).slice(0, 5);
    let html = '';
    filtered.forEach(d => {
      const day = new Date(d.dt_txt).toLocaleDateString(undefined, { weekday: 'short' });
      const iconUrl = `https://openweathermap.org/img/wn/${d.weather[0].icon}.png`;
      const tempC = d.main.temp - 273.15;
      html += `
        <div class="day">
          <strong>${day}</strong>
          <img alt="${d.weather[0].description}" src="${iconUrl}" class="icon"/>
          <div>${formatTemp(tempC)}</div>
        </div>
      `;
    });
    forecastDiv.innerHTML = html;
  }

  searchBtn.onclick = () => {
    const city = cityInput.value.trim();
    if (city) searchWeather(city);
  };

  addFavoriteBtn.onclick = addCityToFavorites;

  unitToggleBtn.onclick = () => {
    isCelsius = !isCelsius;
    unitToggleBtn.textContent = isCelsius ? 'Show °F' : 'Show °C';
    if (currentCity) searchWeather(currentCity);
  };

  
  renderFavorites();

  searchWeather('sivakasi');
</script>

</body>
</html>
