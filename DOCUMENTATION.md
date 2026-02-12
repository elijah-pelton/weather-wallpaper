# Weather Wallpaper Documentation

## Overview

Weather Wallpaper is a macOS application that transforms the desktop wallpaper into an interactive, real-time 3D globe. It integrates various data sources to display weather, radar, flight tracking, and air quality information.

## Architecture

The application is built using a hybrid approach:
- **Swift (macOS native):** Handles window management, system integration (menu bar, screens), location services, and basic app lifecycle.
- **Web (WebKit):** Uses Mapbox GL JS to render the 3D globe and handles data fetching from various web APIs.

### Component Interaction

1. **Swift to JavaScript:** The native app communicates with the web content using `evaluateJavaScript`. This is used to pass user settings (like Mapbox tokens or API keys), toggle features (flights, weather radar), and update the location.
2. **JavaScript to Swift:** Currently, the interaction is primarily one-way (Swift to JS). Location detection is handled in Swift and passed to JS.

## Key Components

### Native (Swift)

- **`main.swift`:** Entry point of the application. Sets up the `AppDelegate` and runs the app in accessory mode (no Dock icon).
- **`AppDelegate.swift`:** Manages the system menu bar, user preferences (using `UserDefaults`), and coordinates between the `LocationManager` and `DesktopWindowManager`.
- **`DesktopWindowManager.swift`:** Responsible for creating and managing a borderless, desktop-level `NSWindow` for each connected screen. Each window contains a `WKWebView` that loads the local `index.html`.
- **`LocationManager.swift`:** Uses CoreLocation to determine the user's current coordinates.

### Web (JavaScript/HTML/CSS)

- **`index.html`:** The main structure of the wallpaper, including the globe container and the information overlays (weather/allergy bars).
- **`globe.js`:**
    - Initializes Mapbox GL JS.
    - Manages the 3D globe rendering, including the day/night terminator.
    - Implements the "Spin Globe" functionality using `requestAnimationFrame`.
    - Handles flight tracking visualization and animation.
    - Manages the Mapbox layers for labels, roads, and weather radar.
- **`weather.js`:**
    - Fetches and renders weather data from Open-Meteo.
    - Fetches and renders air quality and pollen data.
    - Handles reverse geocoding to display the city name.
    - Manages a local cache for weather data in `localStorage`.
- **`styles.css`:** Defines the appearance of the overlays, markers, and animations.

## Data Sources & APIs

- **Mapbox GL JS:** 3D globe rendering and map data.
- **OpenSky Network:** Real-time aircraft positions.
- **RainViewer:** Weather radar tile overlays.
- **Open-Meteo:** Weather forecasts and air quality data.
- **Google Pollen API:** Pollen forecast data (requires optional API key).
- **Apple CoreLocation:** Native location services for the user's current position.

## Multi-Monitor Support

The app detects screen changes and creates a separate `NSWindow` and `WKWebView` for each monitor. This ensures the globe is visible on all desktops. Each `WKWebView` runs an independent instance of the web application.
