# Project 2: Weather Dashboard 🌤️

**Difficulty:** Intermediate | **Estimated Time:** 12-16 hours | **Technologies:** React, API Integration, CSS

## 🎯 Project Overview

Build a comprehensive weather dashboard that fetches real-time weather data from APIs. This project demonstrates API integration, advanced React hooks, error handling, and responsive design - essential skills for modern React development.

## 🚀 What You'll Build

A modern weather application with these features:
- 🌍 **Current Weather** - Real-time weather for any city
- 📍 **Geolocation** - Auto-detect user's location
- 📊 **5-Day Forecast** - Extended weather predictions
- 🔍 **City Search** - Search and save favorite cities
- 📱 **Responsive Design** - Works on all devices
- 🎨 **Dynamic Backgrounds** - Changes based on weather conditions
- 💾 **Local Storage** - Remember user preferences
- ⚡ **Loading States** - Smooth user experience

## 🛠️ Technical Requirements

### Core Features

#### 1. Current Weather Display
```jsx
// Weather data structure
{
  location: "New York, NY",
  temperature: 22,
  condition: "Partly Cloudy",
  description: "Partly cloudy with light winds",
  humidity: 65,
  windSpeed: 12,
  pressure: 1013,
  visibility: 10,
  uvIndex: 6,
  feelsLike: 25,
  icon: "partly-cloudy-day"
}
```

#### 2. 5-Day Forecast
```jsx
// Forecast data structure
[
  {
    date: "2024-07-20",
    dayName: "Saturday",
    high: 28,
    low: 18,
    condition: "Sunny",
    icon: "sunny",
    precipitation: 0
  },
  // ... more days
]
```

#### 3. Search and Favorites
- Search cities by name
- Add/remove favorite cities
- Quick access to saved locations
- Search history

#### 4. Geolocation Integration
- Auto-detect user location
- Request location permission
- Fallback to default city

### API Integration

#### Recommended APIs
1. **OpenWeatherMap** (Free tier available)
   - Current weather
   - 5-day forecast
   - Geocoding for city search

2. **WeatherAPI** (Alternative)
   - More detailed data
   - Better free tier limits

#### API Setup Example
```javascript
// API configuration
const API_KEY = process.env.REACT_APP_WEATHER_API_KEY;
const BASE_URL = 'https://api.openweathermap.org/data/2.5';

// Fetch current weather
const fetchCurrentWeather = async (city) => {
  const response = await fetch(
    `${BASE_URL}/weather?q=${city}&appid=${API_KEY}&units=metric`
  );
  return response.json();
};

// Fetch forecast
const fetchForecast = async (city) => {
  const response = await fetch(
    `${BASE_URL}/forecast?q=${city}&appid=${API_KEY}&units=metric`
  );
  return response.json();
};
```

## 📁 Project Structure

```
weather-dashboard/
├── public/
│   ├── index.html
│   └── weather-icons/          # Custom weather icons
├── src/
│   ├── components/
│   │   ├── WeatherCard.js      # Main weather display
│   │   ├── ForecastCard.js     # Individual forecast day
│   │   ├── ForecastList.js     # 5-day forecast container
│   │   ├── SearchBar.js        # City search input
│   │   ├── FavoritesList.js    # Saved cities
│   │   ├── LoadingSpinner.js   # Loading component
│   │   └── ErrorMessage.js     # Error handling
│   ├── hooks/
│   │   ├── useWeather.js       # Custom weather hook
│   │   ├── useGeolocation.js   # Geolocation hook
│   │   └── useLocalStorage.js  # Local storage hook
│   ├── services/
│   │   ├── weatherAPI.js       # API calls
│   │   └── geolocation.js      # Location services
│   ├── utils/
│   │   ├── weatherHelpers.js   # Helper functions
│   │   └── dateHelpers.js      # Date formatting
│   ├── styles/
│   │   ├── App.css
│   │   ├── WeatherCard.css
│   │   ├── ForecastCard.css
│   │   └── SearchBar.css
│   ├── App.js
│   └── index.js
└── .env                        # API keys (not committed)
```

## 📋 Step-by-Step Implementation

### Phase 1: Setup and API Integration (3-4 hours)

1. **Project Setup**
   ```bash
   npx create-react-app weather-dashboard
   cd weather-dashboard
   npm install axios # for API calls
   ```

2. **Get API Key**
   - Sign up for OpenWeatherMap
   - Get free API key
   - Create `.env` file with API key

3. **Basic API Integration**
   - Create weather service functions
   - Test API calls
   - Handle API responses

4. **Environment Setup**
   ```javascript
   // .env file
   REACT_APP_WEATHER_API_KEY=your_api_key_here
   ```

### Phase 2: Core Weather Display (4-5 hours)

1. **Current Weather Component**
   ```jsx
   function WeatherCard({ weatherData, loading, error }) {
     if (loading) return <LoadingSpinner />;
     if (error) return <ErrorMessage message={error} />;
     
     return (
       <div className="weather-card">
         <div className="location">{weatherData.location}</div>
         <div className="temperature">{weatherData.temperature}°C</div>
         <div className="condition">{weatherData.condition}</div>
         {/* More weather details */}
       </div>
     );
   }
   ```

2. **Weather Data Fetching**
   ```jsx
   function App() {
     const [currentWeather, setCurrentWeather] = useState(null);
     const [loading, setLoading] = useState(true);
     const [error, setError] = useState(null);

     useEffect(() => {
       fetchWeatherData('London'); // Default city
     }, []);

     const fetchWeatherData = async (city) => {
       try {
         setLoading(true);
         setError(null);
         const data = await weatherAPI.getCurrentWeather(city);
         setCurrentWeather(data);
       } catch (err) {
         setError('Failed to fetch weather data');
       } finally {
         setLoading(false);
       }
     };
   }
   ```

### Phase 3: Search and Forecast (3-4 hours)

1. **Search Functionality**
   ```jsx
   function SearchBar({ onSearch, loading }) {
     const [query, setQuery] = useState('');

     const handleSubmit = (e) => {
       e.preventDefault();
       if (query.trim()) {
         onSearch(query.trim());
         setQuery('');
       }
     };

     return (
       <form onSubmit={handleSubmit} className="search-bar">
         <input
           type="text"
           value={query}
           onChange={(e) => setQuery(e.target.value)}
           placeholder="Search for a city..."
           disabled={loading}
         />
         <button type="submit" disabled={loading || !query.trim()}>
           Search
         </button>
       </form>
     );
   }
   ```

2. **5-Day Forecast**
   ```jsx
   function ForecastList({ forecastData }) {
     return (
       <div className="forecast-list">
         <h3>5-Day Forecast</h3>
         <div className="forecast-cards">
           {forecastData.map((day, index) => (
             <ForecastCard key={index} forecast={day} />
           ))}
         </div>
       </div>
     );
   }
   ```

### Phase 4: Advanced Features (3-4 hours)

1. **Geolocation Integration**
   ```jsx
   const useGeolocation = () => {
     const [location, setLocation] = useState(null);
     const [error, setError] = useState(null);

     const getCurrentLocation = () => {
       if (!navigator.geolocation) {
         setError('Geolocation is not supported');
         return;
       }

       navigator.geolocation.getCurrentPosition(
         (position) => {
           setLocation({
             lat: position.coords.latitude,
             lon: position.coords.longitude
           });
         },
         (error) => setError(error.message)
       );
     };

     return { location, error, getCurrentLocation };
   };
   ```

2. **Favorites Management**
   ```jsx
   function FavoritesList({ favorites, onSelectCity, onRemoveFavorite }) {
     return (
       <div className="favorites-list">
         <h3>Favorite Cities</h3>
         {favorites.map(city => (
           <div key={city} className="favorite-item">
             <button onClick={() => onSelectCity(city)}>
               {city}
             </button>
             <button onClick={() => onRemoveFavorite(city)}>
               ×
             </button>
           </div>
         ))}
       </div>
     );
   }
   ```

### Phase 5: Styling and Polish (2-3 hours)

1. **Responsive Design**
   ```css
   .weather-dashboard {
     display: grid;
     grid-template-columns: 1fr;
     gap: 20px;
     padding: 20px;
   }

   @media (min-width: 768px) {
     .weather-dashboard {
       grid-template-columns: 2fr 1fr;
     }
   }

   @media (min-width: 1024px) {
     .weather-dashboard {
       grid-template-columns: 3fr 1fr;
       max-width: 1200px;
       margin: 0 auto;
     }
   }
   ```

2. **Dynamic Backgrounds**
   ```jsx
   const getBackgroundClass = (condition) => {
     const conditionMap = {
       'clear': 'bg-sunny',
       'clouds': 'bg-cloudy',
       'rain': 'bg-rainy',
       'snow': 'bg-snowy',
       'thunderstorm': 'bg-stormy'
     };
     return conditionMap[condition.toLowerCase()] || 'bg-default';
   };
   ```

## 🎨 Design Requirements

### Visual Design
- **Modern, clean interface** with card-based layout
- **Weather-appropriate color schemes** (blue for rain, orange for sunny, etc.)
- **Smooth animations** for loading states and transitions
- **Intuitive icons** for weather conditions
- **Typography hierarchy** for easy reading

### User Experience
- **Fast loading** with skeleton screens
- **Error handling** with retry options
- **Offline support** (bonus feature)
- **Keyboard navigation** support
- **Touch-friendly** mobile interface

## 🏆 Bonus Features

### Level 1 Bonuses
- **Weather Alerts** - Display weather warnings
- **Hourly Forecast** - 24-hour detailed forecast
- **Weather Maps** - Integration with weather radar
- **Unit Conversion** - Celsius/Fahrenheit, mph/kmh

### Level 2 Bonuses
- **Weather History** - Historical weather data
- **Weather Comparison** - Compare multiple cities
- **Push Notifications** - Weather alerts
- **Dark/Light Mode** - Theme switching

### Level 3 Bonuses
- **PWA Features** - Offline support, installable
- **Voice Search** - Search cities by voice
- **Weather Widgets** - Embeddable weather widgets
- **Social Sharing** - Share weather conditions

## 🧪 Testing Strategy

### Unit Tests
```jsx
// Example test for weather service
import { fetchCurrentWeather } from '../services/weatherAPI';

test('fetches current weather data', async () => {
  const mockData = { temperature: 22, condition: 'Sunny' };
  global.fetch = jest.fn(() =>
    Promise.resolve({
      json: () => Promise.resolve(mockData),
    })
  );

  const result = await fetchCurrentWeather('London');
  expect(result).toEqual(mockData);
});
```

### Integration Tests
- Test API integration with mock data
- Test user interactions (search, favorites)
- Test responsive design breakpoints
- Test error handling scenarios

## 📱 Responsive Design Breakpoints

### Mobile (320px - 768px)
- Single column layout
- Stacked forecast cards
- Collapsible favorites section
- Touch-optimized buttons

### Tablet (768px - 1024px)
- Two-column layout
- Side-by-side forecast
- Expanded search bar
- Hover effects

### Desktop (1024px+)
- Multi-column layout
- Dashboard-style interface
- Keyboard shortcuts
- Advanced features visible

## 🔧 Performance Optimization

### API Optimization
- **Caching** - Cache API responses
- **Debouncing** - Debounce search requests
- **Rate Limiting** - Respect API limits
- **Error Retry** - Implement retry logic

### React Optimization
- **useMemo** - Memoize expensive calculations
- **useCallback** - Optimize event handlers
- **Code Splitting** - Lazy load components
- **Image Optimization** - Optimize weather icons

## 📚 Learning Resources

### Weather APIs
- [OpenWeatherMap API](https://openweathermap.org/api)
- [WeatherAPI Documentation](https://www.weatherapi.com/docs/)
- [Weather Icons](https://erikflowers.github.io/weather-icons/)

### React Concepts
- Advanced hooks (useEffect, custom hooks)
- API integration patterns
- Error boundary implementation
- Performance optimization

### Design Inspiration
- [Weather App Designs](https://dribbble.com/tags/weather_app)
- [Material Design Weather](https://material.io/design/color/the-color-system.html)

## ✅ Completion Checklist

### Core Features
- [ ] Display current weather for any city
- [ ] Show 5-day weather forecast
- [ ] Implement city search functionality
- [ ] Add/remove favorite cities
- [ ] Geolocation integration
- [ ] Responsive design across devices
- [ ] Loading states and error handling

### Advanced Features
- [ ] Local storage for preferences
- [ ] Dynamic backgrounds based on weather
- [ ] Weather icons and animations
- [ ] Unit conversion options
- [ ] Search history

### Code Quality
- [ ] Clean, organized component structure
- [ ] Custom hooks for reusable logic
- [ ] Proper error handling
- [ ] Performance optimizations
- [ ] Comprehensive testing

## 🚀 Deployment and Portfolio

### Deployment Steps
1. **Build for production**
   ```bash
   npm run build
   ```

2. **Deploy to Netlify/Vercel**
   - Connect GitHub repository
   - Set environment variables
   - Configure build settings

3. **Custom Domain** (optional)
   - Purchase domain
   - Configure DNS settings

### Portfolio Presentation
- **Live Demo** - Fully functional application
- **GitHub Repository** - Clean, documented code
- **Case Study** - Document your development process
- **Screenshots** - Show responsive design
- **Technical Highlights** - API integration, performance optimization

---

**Ready to build your weather dashboard?** This project will significantly enhance your React skills and demonstrate your ability to work with APIs and create professional applications.

**Next Project:** [E-commerce Product Catalog](../03-ecommerce-catalog/README.md) - Advanced state management and complex UI patterns.

---

*This project showcases essential skills that employers look for: API integration, error handling, responsive design, and modern React patterns. Take your time to build something you're proud of!*
