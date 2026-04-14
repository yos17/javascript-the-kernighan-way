# Chapter 10: Async and APIs

Modern web applications are asynchronous by nature. When your program needs to fetch data from a server, read a file, or wait for user input, it can't just pause and wait—that would freeze the entire interface. JavaScript handles this through asynchronous programming: the ability to start long-running operations and continue executing other code while waiting for results. This chapter explores how async/await makes asynchronous code readable, and how the Fetch API lets JavaScript programs communicate with web services. Together, these tools transform static pages into dynamic applications that talk to the world.

The Weather Dashboard demonstrates these concepts in a real-world context. It fetches weather data from a public API, handles errors gracefully, shows loading feedback to users, and stores search history in the browser. Along the way, you'll see how to parse JSON responses, destructure complex nested data, and build a responsive UI that responds to network events. This is the foundation of modern web development.

[Open weather_dashboard.html](weather_dashboard.html) to try it.

## The Complete Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather Dashboard</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            transition: background 0.5s ease;
            min-height: 100vh;
            display: flex;
        }

        .container {
            display: flex;
            width: 100%;
        }

        .sidebar {
            width: 250px;
            padding: 20px;
            background: rgba(0, 0, 0, 0.1);
            overflow-y: auto;
            box-shadow: 2px 0 10px rgba(0, 0, 0, 0.1);
        }

        .sidebar h3 {
            margin-bottom: 15px;
            font-size: 14px;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: #333;
        }

        .history-list {
            list-style: none;
        }

        .history-item {
            padding: 10px;
            margin-bottom: 8px;
            background: white;
            border-radius: 6px;
            cursor: pointer;
            font-size: 14px;
            transition: all 0.2s;
            box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
        }

        .history-item:hover {
            transform: translateX(5px);
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
        }

        .main-content {
            flex: 1;
            padding: 40px;
            overflow-y: auto;
        }

        .search-container {
            display: flex;
            gap: 10px;
            margin-bottom: 30px;
        }

        .search-container input {
            flex: 1;
            padding: 12px 16px;
            border: 2px solid rgba(255, 255, 255, 0.3);
            border-radius: 8px;
            font-size: 16px;
            background: rgba(255, 255, 255, 0.9);
            color: #333;
            transition: all 0.3s;
        }

        .search-container input:focus {
            outline: none;
            border-color: rgba(255, 255, 255, 0.7);
            background: white;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        }

        .search-container button {
            padding: 12px 32px;
            background: rgba(255, 255, 255, 0.3);
            border: 2px solid rgba(255, 255, 255, 0.5);
            border-radius: 8px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s;
            color: white;
        }

        .search-container button:hover {
            background: rgba(255, 255, 255, 0.5);
            border-color: white;
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
        }

        .error-message {
            background: rgba(220, 53, 69, 0.9);
            color: white;
            padding: 16px;
            border-radius: 8px;
            margin-bottom: 20px;
            display: none;
            animation: slideIn 0.3s ease;
        }

        .error-message.show {
            display: block;
        }

        .loading {
            display: none;
            text-align: center;
            padding: 40px;
        }

        .loading.show {
            display: block;
        }

        .spinner {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-top: 4px solid white;
            border-radius: 50%;
            width: 50px;
            height: 50px;
            animation: spin 0.8s linear infinite;
            margin: 0 auto 16px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        @keyframes slideIn {
            from { opacity: 0; transform: translateY(-10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .weather-content {
            display: none;
            animation: fadeIn 0.5s ease;
        }

        .weather-content.show {
            display: block;
        }

        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }

        .current-weather {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 12px;
            padding: 30px;
            margin-bottom: 30px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
        }

        .city-title {
            font-size: 36px;
            font-weight: bold;
            margin-bottom: 10px;
            color: #333;
        }

        .current-temp {
            display: flex;
            align-items: baseline;
            gap: 15px;
            margin: 20px 0;
        }

        .temp-value {
            font-size: 64px;
            font-weight: bold;
            color: #333;
        }

        .temp-unit {
            font-size: 24px;
            color: #666;
        }

        .condition-emoji {
            font-size: 80px;
        }

        .weather-details {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 20px;
            margin-top: 30px;
            padding-top: 30px;
            border-top: 1px solid rgba(0, 0, 0, 0.1);
        }

        .detail-card {
            background: rgba(0, 0, 0, 0.02);
            padding: 15px;
            border-radius: 8px;
            text-align: center;
        }

        .detail-label {
            font-size: 12px;
            text-transform: uppercase;
            color: #666;
            margin-bottom: 8px;
            letter-spacing: 0.5px;
        }

        .detail-value {
            font-size: 24px;
            font-weight: bold;
            color: #333;
        }

        .forecast {
            margin-top: 30px;
        }

        .forecast-title {
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 20px;
            color: #333;
        }

        .forecast-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 15px;
        }

        .forecast-card {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 12px;
            padding: 20px;
            text-align: center;
            box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
            transition: all 0.3s;
        }

        .forecast-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
        }

        .forecast-day {
            font-weight: bold;
            color: #333;
            margin-bottom: 10px;
            font-size: 16px;
        }

        .forecast-emoji {
            font-size: 48px;
            margin: 10px 0;
        }

        .forecast-temps {
            display: flex;
            justify-content: space-around;
            margin-top: 10px;
            padding-top: 10px;
            border-top: 1px solid rgba(0, 0, 0, 0.1);
        }

        .forecast-temp {
            font-size: 14px;
            color: #666;
        }

        .forecast-temp-high {
            font-weight: bold;
            color: #333;
        }

        .bg-sunny  { background: linear-gradient(135deg, #FFD89B 0%, #19547B 100%); }
        .bg-cloudy { background: linear-gradient(135deg, #A8A8A8 0%, #5A5A5A 100%); }
        .bg-rainy  { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
        .bg-cold   { background: linear-gradient(135deg, #667eea 0%, #4facfe 100%); }
        .bg-hot    { background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); }

        @media (max-width: 768px) {
            .container { flex-direction: column; }
            .sidebar { width: 100%; max-height: 150px; display: flex; gap: 20px; }
            .sidebar h3 { display: none; }
            .history-list { display: flex; gap: 10px; overflow-x: auto; flex: 1; }
            .history-item { white-space: nowrap; flex-shrink: 0; }
            .main-content { padding: 20px; }
            .current-temp { flex-wrap: wrap; }
            .temp-value { font-size: 48px; }
            .city-title { font-size: 28px; }
            .search-container { flex-direction: column; }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="sidebar">
            <h3>Recent Searches</h3>
            <ul class="history-list" id="historyList"></ul>
        </div>

        <div class="main-content">
            <div class="search-container">
                <input type="text" id="cityInput" placeholder="Enter a city name..." autocomplete="off">
                <button id="searchBtn">Search</button>
            </div>

            <div class="error-message" id="errorMessage"></div>
            <div class="loading" id="loading">
                <div class="spinner"></div>
                <p>Fetching weather data...</p>
            </div>

            <div class="weather-content" id="weatherContent">
                <div class="current-weather">
                    <div class="city-title" id="cityTitle"></div>
                    <div class="current-temp">
                        <div class="condition-emoji" id="emoji"></div>
                        <div>
                            <div class="temp-value">
                                <span id="tempF"></span><span class="temp-unit">°F</span>
                            </div>
                            <div class="temp-value" style="font-size: 40px; color: #999;">
                                <span id="tempC"></span><span class="temp-unit">°C</span>
                            </div>
                        </div>
                    </div>
                    <div class="weather-details">
                        <div class="detail-card">
                            <div class="detail-label">Condition</div>
                            <div class="detail-value" id="condition"></div>
                        </div>
                        <div class="detail-card">
                            <div class="detail-label">Feels Like</div>
                            <div class="detail-value" id="feelsLike"></div>
                        </div>
                        <div class="detail-card">
                            <div class="detail-label">Humidity</div>
                            <div class="detail-value" id="humidity"></div>
                        </div>
                        <div class="detail-card">
                            <div class="detail-label">Wind Speed</div>
                            <div class="detail-value" id="windSpeed"></div>
                        </div>
                        <div class="detail-card">
                            <div class="detail-label">Wind Direction</div>
                            <div class="detail-value" id="windDirection"></div>
                        </div>
                    </div>
                </div>

                <div class="forecast">
                    <div class="forecast-title">3-Day Forecast</div>
                    <div class="forecast-grid" id="forecastGrid"></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        const API_BASE = 'https://wttr.in';
        const MAX_HISTORY = 5;

        const cityInput = document.getElementById('cityInput');
        const searchBtn = document.getElementById('searchBtn');
        const errorMessage = document.getElementById('errorMessage');
        const loading = document.getElementById('loading');
        const weatherContent = document.getElementById('weatherContent');
        const historyList = document.getElementById('historyList');

        loadHistory();
        setupEventListeners();

        function setupEventListeners() {
            searchBtn.addEventListener('click', handleSearch);
            cityInput.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') handleSearch();
            });
        }

        async function handleSearch() {
            const city = cityInput.value.trim();
            if (!city) { showError('Please enter a city name'); return; }
            await fetchWeather(city);
            addToHistory(city);
        }

        async function fetchWeather(city) {
            try {
                showLoading();
                hideError();
                const response = await fetch(`${API_BASE}/${city}?format=j1`);
                if (!response.ok) {
                    throw new Error('City not found. Please check the spelling and try again.');
                }
                const data = await response.json();
                displayWeather(data, city);
                cityInput.value = '';
            } catch (error) {
                showError(error.message || 'Unable to fetch weather data. Please try again.');
            } finally {
                hideLoading();
            }
        }

        function displayWeather(data, cityName) {
            try {
                const { current_condition, forecast } = data.current_condition
                    ? { current_condition: data.current_condition, forecast: data.forecast }
                    : extractWeatherData(data);

                const {
                    temp_C, temp_F, weatherDesc, humidity,
                    windspeedKmph, windDirection16Point, FeelsLikeC, FeelsLikeF
                } = current_condition;

                document.getElementById('cityTitle').textContent = cityName;
                document.getElementById('tempF').textContent = Math.round(temp_F);
                document.getElementById('tempC').textContent = Math.round(temp_C);
                document.getElementById('condition').textContent = weatherDesc;
                document.getElementById('humidity').textContent = `${humidity}%`;
                document.getElementById('windSpeed').textContent = `${Math.round(windspeedKmph)} km/h`;
                document.getElementById('windDirection').textContent = windDirection16Point;
                document.getElementById('feelsLike').textContent = `${Math.round(FeelsLikeF)}°F`;
                document.getElementById('emoji').textContent = getWeatherEmoji(weatherDesc);

                updateBackgroundGradient(temp_F, weatherDesc);
                displayForecast(forecast);
                weatherContent.classList.add('show');
            } catch (error) {
                showError('Error displaying weather data: ' + error.message);
            }
        }

        function extractWeatherData(data) {
            const { current_condition: [current], forecast } = data;
            return {
                current_condition: {
                    temp_C: current.temp_C,
                    temp_F: current.temp_F,
                    weatherDesc: current.weatherDesc[0].value,
                    humidity: current.humidity,
                    windspeedKmph: current.windspeedKmph,
                    windDirection16Point: current.windDirection16Point,
                    FeelsLikeC: current.FeelsLikeC,
                    FeelsLikeF: current.FeelsLikeF
                },
                forecast
            };
        }

        function displayForecast(forecast) {
            const forecastGrid = document.getElementById('forecastGrid');
            forecastGrid.innerHTML = '';
            forecast.slice(0, 3).forEach((day) => {
                const date = new Date(day.date);
                const dayName = date.toLocaleDateString('en-US', { weekday: 'short' });
                const maxTemp = day.day.maxtemp_F;
                const minTemp = day.day.mintemp_F;
                const condition = day.day.weatherDesc[0].value;

                const card = document.createElement('div');
                card.className = 'forecast-card';
                card.innerHTML = `
                    <div class="forecast-day">${dayName}</div>
                    <div class="forecast-emoji">${getWeatherEmoji(condition)}</div>
                    <div style="font-size: 14px; color: #666; margin-bottom: 10px;">${condition}</div>
                    <div class="forecast-temps">
                        <div class="forecast-temp">High<br><span class="forecast-temp-high">${Math.round(maxTemp)}°</span></div>
                        <div class="forecast-temp">Low<br><span class="forecast-temp-high">${Math.round(minTemp)}°</span></div>
                    </div>
                `;
                forecastGrid.appendChild(card);
            });
        }

        function getWeatherEmoji(condition) {
            const lower = condition.toLowerCase();
            if (lower.includes('sunny') || lower.includes('clear')) return '☀️';
            if (lower.includes('cloud')) return '☁️';
            if (lower.includes('rain') || lower.includes('drizzle')) return '🌧️';
            if (lower.includes('snow')) return '❄️';
            if (lower.includes('thunder') || lower.includes('storm')) return '⛈️';
            if (lower.includes('fog') || lower.includes('mist')) return '🌫️';
            if (lower.includes('sleet')) return '🌨️';
            if (lower.includes('hail')) return '🧊';
            return '🌤️';
        }

        function updateBackgroundGradient(tempF, condition) {
            const lower = condition.toLowerCase();
            document.body.classList.remove('bg-sunny', 'bg-cloudy', 'bg-rainy', 'bg-cold', 'bg-hot');
            if (lower.includes('rain') || lower.includes('thunder')) document.body.classList.add('bg-rainy');
            else if (lower.includes('cloud')) document.body.classList.add('bg-cloudy');
            else if (tempF < 40) document.body.classList.add('bg-cold');
            else if (tempF > 85) document.body.classList.add('bg-hot');
            else document.body.classList.add('bg-sunny');
        }

        function addToHistory(city) {
            let history = JSON.parse(localStorage.getItem('weatherHistory') || '[]');
            history = history.filter(c => c.toLowerCase() !== city.toLowerCase());
            history.unshift(city);
            history = history.slice(0, MAX_HISTORY);
            localStorage.setItem('weatherHistory', JSON.stringify(history));
            renderHistory();
        }

        function loadHistory() { renderHistory(); }

        function renderHistory() {
            const history = JSON.parse(localStorage.getItem('weatherHistory') || '[]');
            historyList.innerHTML = '';
            history.forEach(city => {
                const li = document.createElement('li');
                li.className = 'history-item';
                li.textContent = city;
                li.addEventListener('click', () => { cityInput.value = city; fetchWeather(city); });
                historyList.appendChild(li);
            });
        }

        function showLoading() {
            loading.classList.add('show');
            weatherContent.classList.remove('show');
            errorMessage.classList.remove('show');
        }
        function hideLoading() { loading.classList.remove('show'); }
        function showError(message) {
            errorMessage.textContent = message;
            errorMessage.classList.add('show');
            weatherContent.classList.remove('show');
        }
        function hideError() { errorMessage.classList.remove('show'); }
    </script>
</body>
</html>
```

## How It Works

The Weather Dashboard is a single-file application that combines HTML, CSS, and JavaScript. It demonstrates five core patterns: making asynchronous HTTP requests, parsing JSON data structures, handling errors with try/catch, managing UI state (loading, success, error), and storing data locally in the browser. Let's walk through each.

### Promises and async/await

JavaScript is single-threaded: it executes one statement at a time. When you fetch data from a server, that request takes time—potentially seconds. If JavaScript blocked and waited, nothing else could happen: buttons wouldn't respond, animations would freeze, the page would appear dead.

Promises represent a value that may not be available yet but will be eventually. They have three states: pending (the operation is in progress), fulfilled (it completed successfully with a result), or rejected (it failed with an error).

For years, developers used `.then()` chains to work with Promises:

```javascript
fetch('/api/weather')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

This works but gets unwieldy with multiple steps. `async/await` is cleaner syntax for the same concept:

```javascript
async function getWeather() {
  const response = await fetch('/api/weather');
  const data = await response.json();
  console.log(data);
}
```

The `async` keyword marks a function that will use `await`. The `await` keyword pauses execution at that line until the Promise settles (succeeds or fails). Your code reads top-to-bottom, like synchronous code, but the browser remains responsive because JavaScript's event loop keeps running. This is the power of async/await: code that's easy to read and doesn't block the UI.

In the Weather Dashboard:

```javascript
async function fetchWeather(city) {
  try {
    showLoading();
    const response = await fetch(`${API_BASE}/${city}?format=j1`);
    if (!response.ok) {
      throw new Error('City not found.');
    }
    const data = await response.json();
    displayWeather(data, city);
  } catch (error) {
    showError(error.message);
  } finally {
    hideLoading();
  }
}
```

Notice the structure: show a loading spinner, fetch the data, check if the request succeeded, parse the JSON, display results. If anything goes wrong, catch the error and show a user-friendly message. The `finally` block always runs, regardless of success or failure, to hide the spinner.

### The Fetch API

The Fetch API is the modern standard for HTTP requests in JavaScript. It replaces older technologies like XMLHttpRequest and is built on Promises.

`fetch(url)` initiates a request and returns a Promise that resolves to a Response object:

```javascript
const response = await fetch('https://wttr.in/London?format=j1');
```

The Response object has properties and methods:
- `response.ok`: boolean; true if status was 200–299
- `response.status`: the HTTP status code (200, 404, 500, etc.)
- `response.json()`: a method that reads the response body and parses it as JSON; returns a Promise
- `response.text()`: reads the body as plain text
- `response.blob()`: reads the body as binary data (for images, files, etc.)

A critical detail: **fetch does not reject the Promise on HTTP errors like 404 or 500**. It only rejects if the request itself fails (network error, CORS block, etc.). You must check `response.ok` or `response.status` manually:

```javascript
const response = await fetch(url);
if (!response.ok) {
  throw new Error(`HTTP error: ${response.status}`);
}
const data = await response.json();
```

The Weather Dashboard fetches from the free wttr.in service: `https://wttr.in/CityName?format=j1`. The `format=j1` parameter tells the API to return JSON instead of text weather data.

### Working with JSON

JSON (JavaScript Object Notation) is the universal language for web data. It looks like JavaScript objects but is plain text, so any language can parse it. JSON has six types: strings, numbers, booleans, null, arrays `[...]`, and objects `{...}`.

The Fetch API's `response.json()` method parses the JSON string into a JavaScript object:

```javascript
const data = await response.json();
// data is now a JavaScript object
console.log(data.current_condition.temp_C);
```

The wttr.in API returns deeply nested JSON structure. To extract data, you navigate the object tree and use destructuring to make it cleaner:

```javascript
const { current_condition, forecast } = data;
const { temp_C, temp_F, humidity } = current_condition[0];
console.log(temp_C); // Example: 15
```

In the dashboard, we handle API response quirks by normalizing the data structure with the `extractWeatherData` function, which uses destructuring to pull out the exact fields we need.

### Error Handling with try/catch

Network requests can fail for many reasons: no internet connection, server is down, user entered an invalid city, firewall blocks the request. The try/catch/finally pattern handles all of them:

```javascript
try {
  // code that might throw an error
  const response = await fetch(url);
  if (!response.ok) throw new Error('HTTP error: ' + response.status);
  const data = await response.json();
  console.log(data);
} catch (error) {
  // error is either a network error, a thrown Error, or a JSON parse error
  console.error(error.message);
} finally {
  // runs whether success or failure
  console.log('Request complete');
}
```

The `try` block contains code that might fail. If any statement throws an error (or an `await` Promise rejects), execution jumps to the `catch` block. The `error` parameter is an Error object with a `message` property and a `stack` property (showing where it came from).

In the dashboard, try/catch catches three types of errors: network errors if the user is offline, HTTP errors if the server returns 404 or 500, and JSON parse errors if the response isn't valid JSON. The `finally` block always runs, whether success or failure. In the dashboard, it hides the loading spinner so the user sees the result (or error message) without waiting for cleanup.

### Loading States

When a network request starts, the user has no idea it's happening—the interface looks frozen. Good UX requires showing loading feedback immediately.

In the dashboard:

```javascript
function showLoading() {
  loading.classList.add('show');
  weatherContent.classList.remove('show');
  errorMessage.classList.remove('show');
}

function hideLoading() {
  loading.classList.remove('show');
}
```

The CSS shows/hides the spinner with a `.show` class. When the user clicks "Search": show the spinner, start the async fetch, and while waiting, the spinner animates—the page is still responsive. When data arrives (or error occurs), the finally block hides the spinner and shows results or error message. This three-state pattern (loading, success, error) is standard in modern UX.

### Destructuring API Responses

Destructuring is a syntax for extracting values from objects and arrays. It makes code cleaner and more readable:

```javascript
// Without destructuring
const temp = data.current_condition[0].temp_C;
const humidity = data.current_condition[0].humidity;

// With destructuring
const {
  temp_C: temp,
  humidity
} = data.current_condition[0];
```

The second version is more concise. You can rename fields with `:`, and extract nested values by nesting destructure patterns. In the dashboard, we extract eight pieces of data in a single statement:

```javascript
const {
  temp_C,
  temp_F,
  weatherDesc,
  humidity,
  windspeedKmph,
  windDirection16Point,
  FeelsLikeC,
  FeelsLikeF
} = current_condition;
```

This makes the rest of the function clean and readable.

## Try It

Here are three ways to extend the dashboard:

1. **Add a Celsius/Fahrenheit toggle**: Currently, both temperatures display. Add a button that hides one and shows only the other.

2. **Color the forecast cards by temperature**: The current weather background changes color based on temperature. Apply the same logic to the forecast cards.

3. **Add hourly forecast**: The wttr.in API returns hourly data. Display 6 hourly forecasts (next 6 hours) in a scrollable row.

## Exercises

### Exercise 1: Temperature Unit Toggle

Add a button that lets users switch between viewing temperatures in Fahrenheit only, Celsius only, or both. Store the preference in `localStorage` so it persists across page reloads.

### Exercise 2: Weather Description Styling

Different weather conditions should have different visual styles. For example, rainy days get a blue/purple gradient; sunny days get warm orange/yellow; snowy days get icy blue/white. Add a function that returns a gradient color based on the weather description, and apply it to the background and cards.

### Exercise 3: Geolocation

Use the browser's Geolocation API (`navigator.geolocation.getCurrentPosition`) to detect the user's location on page load. Convert latitude/longitude to a city name using a reverse-geocoding API, and auto-load the weather for that city.

## Solutions

### Solution 1: Temperature Unit Toggle

```javascript
// Add to HTML: <button id="tempToggle">Show °F only</button>

const tempToggle = document.getElementById('tempToggle');
let showFahrenheit = true;
let showCelsius = true;

tempToggle.addEventListener('click', () => {
  const modes = [
    { f: true, c: true, label: 'Show °F only' },
    { f: true, c: false, label: 'Show °C only' },
    { f: false, c: true, label: 'Show both' }
  ];

  const current = modes.find(m => m.f === showFahrenheit && m.c === showCelsius);
  const next = modes[(modes.indexOf(current) + 1) % modes.length];

  showFahrenheit = next.f;
  showCelsius = next.c;
  tempToggle.textContent = next.label;

  document.getElementById('tempF').parentElement.style.display = showFahrenheit ? 'block' : 'none';
  document.getElementById('tempC').parentElement.style.display = showCelsius ? 'block' : 'none';

  localStorage.setItem('tempMode', JSON.stringify({ f: showFahrenheit, c: showCelsius }));
});

const saved = JSON.parse(localStorage.getItem('tempMode') || '{"f":true,"c":true}');
showFahrenheit = saved.f;
showCelsius = saved.c;
```

### Solution 2: Weather Description Styling

```javascript
function getGradientFromCondition(condition, tempF) {
  const lower = condition.toLowerCase();

  if (lower.includes('sunny') || lower.includes('clear')) {
    return 'linear-gradient(135deg, #FFD89B 0%, #19547B 100%)';
  } else if (lower.includes('rain')) {
    return 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)';
  } else if (lower.includes('snow')) {
    return 'linear-gradient(135deg, #E0F7FA 0%, #80DEEA 100%)';
  } else if (lower.includes('cloud')) {
    return 'linear-gradient(135deg, #A8A8A8 0%, #5A5A5A 100%)';
  } else if (lower.includes('thunder')) {
    return 'linear-gradient(135deg, #2C3E50 0%, #34495E 100%)';
  }

  if (tempF > 85) {
    return 'linear-gradient(135deg, #f093fb 0%, #f5576c 100%)';
  } else if (tempF < 40) {
    return 'linear-gradient(135deg, #667eea 0%, #4facfe 100%)';
  }

  return 'linear-gradient(135deg, #FFD89B 0%, #19547B 100%)';
}

const gradient = getGradientFromCondition(condition, temp_F);
document.body.style.background = gradient;

document.querySelectorAll('.forecast-card').forEach((card, i) => {
  const dayForecast = forecast[i].day;
  const dayCondition = dayForecast.weatherDesc[0].value;
  const dayTemp = dayForecast.maxtemp_F;
  const dayGradient = getGradientFromCondition(dayCondition, dayTemp);
  card.style.background = dayGradient;
  card.style.color = 'white';
});
```

### Solution 3: Geolocation

```javascript
async function autoDetectLocation() {
  if (!navigator.geolocation) {
    console.log('Geolocation not supported');
    return;
  }

  navigator.geolocation.getCurrentPosition(
    async (position) => {
      const { latitude, longitude } = position.coords;
      try {
        const response = await fetch(
          `https://geocoding-api.open-meteo.com/v1/search?latitude=${latitude}&longitude=${longitude}&count=1&language=en&format=json`
        );
        const data = await response.json();
        if (data.results && data.results.length > 0) {
          const city = data.results[0].name;
          cityInput.value = city;
          await fetchWeather(city);
        }
      } catch (error) {
        console.error('Reverse geocoding failed:', error);
      }
    },
    (error) => {
      console.log('Geolocation denied or unavailable:', error);
    }
  );
}

document.addEventListener('DOMContentLoaded', autoDetectLocation);
```

## What You Learned

| Concept | How We Used It |
|---------|---------------|
| async/await | Made fetchWeather non-blocking so the UI stays responsive |
| fetch() | Made HTTP GET requests to wttr.in without external libraries |
| Response.json() | Parsed JSON data from the API response |
| Promises | Understood that fetch returns a Promise; await waits for it |
| try/catch/finally | Caught network and parse errors; always hid the spinner |
| Error handling | Showed user-friendly messages instead of raw error codes |
| Destructuring | Extracted nested API data cleanly into variables |
| DOM manipulation | Updated the page with weather data and loading states |
| localStorage | Stored search history without a server |
| CSS transitions | Made loading spinners and color changes smooth |
| Responsive design | Ensured the dashboard works on mobile and desktop |
| Object/array methods | Used forEach to build forecast cards; slice to limit history |

## Building with Claude

When you're building features that interact with APIs, Claude can help you understand what data the API actually returns. Instead of guessing at nested structure, paste an example API response and ask Claude to explain the shape of the data. Claude can then show you the best destructuring patterns and help you navigate complex nested objects without trial and error.

Debugging async code is tricky because errors happen after your code runs. Use Claude to help read error messages and trace where they come from. When you see "Cannot read property of undefined," Claude can help you print intermediate values and find exactly which nested property is missing.

Adding new data sources is straightforward once you understand one API. If you find another weather API, a COVID tracker, a stock price feed, or any public JSON endpoint, Claude can help you adapt the dashboard to work with it. The fetch pattern stays the same; only the URL and the data extraction changes.

Modern web development is asynchronous development. These patterns—fetching, parsing, handling errors, managing loading states—appear in every production web app. Master them here, and you've built the foundation for APIs, real-time updates, file uploads, and countless other features that make the web interactive.
