# Tech Brief: Open-Meteo Weather Widget Implementation

## Overview
We'll implement a weather widget using Open-Meteo API for a static HTML/CSS website hosted on Vercel. Open-Meteo provides free weather data without requiring API keys or registration.

## Technical Architecture

### Components
- **Frontend**: HTML/CSS/JavaScript (client-side)
- **API**: Open-Meteo REST API
- **Hosting**: Vercel (static deployment)
- **Data Flow**: Browser → Open-Meteo API → Weather Widget Display

### API Specifications
- **Base URL**: `https://api.open-meteo.com/v1/forecast`
- **Method**: GET
- **Rate Limits**: None (unlimited free usage)
- **CORS**: Enabled (can call directly from browser)

### Key Features
- Current weather conditions
- Temperature, humidity, wind speed
- Weather codes (sunny, cloudy, rainy, etc.)
- Automatic location detection or manual coordinates
- Responsive design

### Data Structure
```json
{
  "current_weather": {
    "temperature": 22.5,
    "windspeed": 10.2,
    "winddirection": 180,
    "weathercode": 3,
    "time": "2023-12-07T15:00"
  }
}
```

## Technical Considerations

### Pros
- No API key management
- No rate limiting concerns
- CORS-enabled for direct browser calls
- Reliable European weather service
- ISO standards compliant

### Limitations
- Weather codes need manual mapping to descriptions
- Limited to basic weather parameters
- No historical data in free tier
- European-focused (but global coverage available)

---

# Step-by-Step Implementation Plan

## Phase 1: Setup and Basic Structure (15 minutes)

### Step 1: Create the HTML Structure
Add this to your existing HTML file:

```html
<!-- Weather Widget Container -->
<div class="weather-widget" id="weather-widget">
    <div class="weather-header">
        <h3>Current Weather</h3>
        <div class="location" id="location">Loading location...</div>
    </div>
    
    <div class="weather-content">
        <div class="temperature-section">
            <div class="temperature" id="temperature">--°</div>
            <div class="weather-icon" id="weather-icon">☀️</div>
        </div>
        
        <div class="weather-details">
            <div class="detail-item">
                <span class="label">Condition:</span>
                <span class="value" id="condition">--</span>
            </div>
            <div class="detail-item">
                <span class="label">Wind:</span>
                <span class="value" id="wind">-- mph</span>
            </div>
            <div class="detail-item">
                <span class="label">Updated:</span>
                <span class="value" id="updated">--</span>
            </div>
        </div>
    </div>
    
    <div class="weather-error" id="weather-error" style="display: none;">
        Unable to load weather data. Please try again later.
    </div>
</div>
```

### Step 2: Add CSS Styling
Add this to your CSS file:

```css
/* Weather Widget Styles */
.weather-widget {
    max-width: 300px;
    margin: 20px auto;
    background: linear-gradient(135deg, #74b9ff, #0984e3);
    border-radius: 15px;
    padding: 20px;
    color: white;
    font-family: 'Arial', sans-serif;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.weather-header {
    text-align: center;
    margin-bottom: 20px;
}

.weather-header h3 {
    margin: 0 0 5px 0;
    font-size: 18px;
    font-weight: 600;
}

.location {
    font-size: 14px;
    opacity: 0.9;
}

.weather-content {
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.temperature-section {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 15px;
}

.temperature {
    font-size: 48px;
    font-weight: bold;
    line-height: 1;
}

.weather-icon {
    font-size: 36px;
}

.weather-details {
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.detail-item {
    display: flex;
    justify-content: space-between;
    font-size: 14px;
}

.label {
    opacity: 0.9;
}

.value {
    font-weight: 600;
}

.weather-error {
    text-align: center;
    color: #ff6b6b;
    background: rgba(255, 255, 255, 0.1);
    padding: 10px;
    border-radius: 8px;
    font-size: 14px;
}

/* Responsive Design */
@media (max-width: 480px) {
    .weather-widget {
        margin: 20px 10px;
        max-width: none;
    }
    
    .temperature {
        font-size: 36px;
    }
    
    .weather-icon {
        font-size: 28px;
    }
}
```

## Phase 2: JavaScript Implementation (30 minutes)

### Step 3: Create Weather Code Mapping
Add this JavaScript to your HTML file (before closing `</body>` tag):

```javascript
<script>
// Weather code mapping for Open-Meteo
const weatherCodes = {
    0: { description: 'Clear sky', icon: '☀️' },
    1: { description: 'Mainly clear', icon: '🌤️' },
    2: { description: 'Partly cloudy', icon: '⛅' },
    3: { description: 'Overcast', icon: '☁️' },
    45: { description: 'Fog', icon: '🌫️' },
    48: { description: 'Depositing rime fog', icon: '🌫️' },
    51: { description: 'Light drizzle', icon: '🌦️' },
    53: { description: 'Moderate drizzle', icon: '🌦️' },
    55: { description: 'Dense drizzle', icon: '🌧️' },
    61: { description: 'Slight rain', icon: '🌧️' },
    63: { description: 'Moderate rain', icon: '🌧️' },
    65: { description: 'Heavy rain', icon: '🌧️' },
    71: { description: 'Slight snow', icon: '🌨️' },
    73: { description: 'Moderate snow', icon: '❄️' },
    75: { description: 'Heavy snow', icon: '❄️' },
    95: { description: 'Thunderstorm', icon: '⛈️' }
};

// Default weather info for fallback
const defaultWeather = {
    description: 'Unknown',
    icon: '🌡️'
};
</script>
```

### Step 4: Implement Location Detection
```javascript
<script>
// Get user's location
function getUserLocation() {
    return new Promise((resolve, reject) => {
        if (!navigator.geolocation) {
            reject('Geolocation not supported');
            return;
        }
        
        navigator.geolocation.getCurrentPosition(
            (position) => {
                resolve({
                    latitude: position.coords.latitude,
                    longitude: position.coords.longitude
                });
            },
            (error) => {
                console.log('Location access denied, using default location');
                // Fallback to New York coordinates
                resolve({
                    latitude: 40.7128,
                    longitude: -74.0060
                });
            }
        );
    });
}

// Reverse geocoding to get city name (simplified)
async function getCityName(lat, lon) {
    try {
        // Using a simple reverse geocoding service
        const response = await fetch(`https://api.bigdatacloud.net/data/reverse-geocode-client?latitude=${lat}&longitude=${lon}&localityLanguage=en`);
        const data = await response.json();
        return data.city || data.locality || 'Unknown Location';
    } catch (error) {
        console.error('Error getting city name:', error);
        return 'Unknown Location';
    }
}
</script>
```

### Step 5: Implement Weather Data Fetching
```javascript
<script>
// Fetch weather data from Open-Meteo
async function fetchWeatherData(latitude, longitude) {
    const url = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true&hourly=relativehumidity_2m&timezone=auto`;
    
    try {
        const response = await fetch(url);
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Error fetching weather data:', error);
        throw error;
    }
}

// Update the weather widget with data
function updateWeatherWidget(weatherData, cityName) {
    const current = weatherData.current_weather;
    const weatherInfo = weatherCodes[current.weathercode] || defaultWeather;
    
    // Update DOM elements
    document.getElementById('location').textContent = cityName;
    document.getElementById('temperature').textContent = `${Math.round(current.temperature)}°C`;
    document.getElementById('weather-icon').textContent = weatherInfo.icon;
    document.getElementById('condition').textContent = weatherInfo.description;
    document.getElementById('wind').textContent = `${Math.round(current.windspeed)} km/h`;
    
    // Format update time
    const updateTime = new Date(current.time).toLocaleTimeString('en-US', {
        hour: '2-digit',
        minute: '2-digit'
    });
    document.getElementById('updated').textContent = updateTime;
}

// Show error state
function showError() {
    document.querySelector('.weather-content').style.display = 'none';
    document.getElementById('weather-error').style.display = 'block';
}

// Hide error state
function hideError() {
    document.querySelector('.weather-content').style.display = 'block';
    document.getElementById('weather-error').style.display = 'none';
}
</script>
```

### Step 6: Main Initialization Function
```javascript
<script>
// Main function to initialize weather widget
async function initWeatherWidget() {
    try {
        hideError();
        
        // Get user location
        const location = await getUserLocation();
        console.log('Location obtained:', location);
        
        // Get city name
        const cityName = await getCityName(location.latitude, location.longitude);
        console.log('City name:', cityName);
        
        // Fetch weather data
        const weatherData = await fetchWeatherData(location.latitude, location.longitude);
        console.log('Weather data:', weatherData);
        
        // Update widget
        updateWeatherWidget(weatherData, cityName);
        
    } catch (error) {
        console.error('Error initializing weather widget:', error);
        showError();
    }
}

// Initialize when page loads
document.addEventListener('DOMContentLoaded', initWeatherWidget);

// Optional: Refresh weather data every 10 minutes
setInterval(initWeatherWidget, 10 * 60 * 1000);
</script>
```

## Phase 3: Testing and Deployment (15 minutes)

### Step 7: Local Testing
1. Open your HTML file in a browser
2. Check browser console for any errors
3. Verify location permission prompt appears
4. Confirm weather data displays correctly

### Step 8: Deploy to Vercel
1. Commit your changes to your repository
2. Push to your connected Git repository
3. Vercel will automatically deploy
4. Test the live version

### Step 9: Optional Enhancements
- Add temperature unit toggle (°C/°F)
- Include weather forecast
- Add loading animations
- Implement manual location search

## Troubleshooting Common Issues

1. **Location not detected**: Widget falls back to New York coordinates
2. **CORS errors**: Open-Meteo supports CORS, but check browser console
3. **API errors**: Check network connectivity and API response
4. **Styling issues**: Verify CSS is properly linked

## Performance Considerations
- API calls are cached by browser
- Widget refreshes every 10 minutes
- Minimal JavaScript footprint
- Responsive design for all devices

This implementation provides a fully functional weather widget that will work seamlessly with your existing Vercel-hosted website!