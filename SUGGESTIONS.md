# Performance Improvement Suggestions

This document outlines several strategies to reduce CPU, memory, and API usage for the Weather Wallpaper application.

## 1. Reduce Resource Consumption for Multi-Monitor Setups

Currently, the app creates a full `WKWebView` and Mapbox GL instance for every connected monitor. This scales linearly with the number of screens, significantly increasing memory and GPU usage.

*   **Suggestion:** Implement a "Master-Slave" architecture for data fetching.
    *   **How:** Instead of each `WKWebView` independently fetching weather, flight, and radar data, the Swift `AppDelegate` could handle all API requests. Once the data is received, it can be injected into all active `WKWebViews` simultaneously using `evaluateJavaScript`.
    *   **Benefit:** Reduces API calls by `(N-1)` where `N` is the number of monitors. It also ensures data consistency across all screens.

## 2. Optimize GPU Usage and Rendering

Mapbox GL JS is a WebGL-based engine that can be demanding on the GPU.

*   **Suggestion:** Implement a "Sleep" or "Low Power" mode when the desktop is obscured.
    *   **How:** Use macOS APIs (or JavaScript's `IntersectionObserver` / `Document.visibilityState`, though these are tricky for desktop wallpapers) to detect if the user is currently looking at the desktop. If a full-screen application is covering the wallpaper, or if the user is away, the app could:
        1.  Pause the globe rotation (`setSpinEnabled(false)`).
        2.  Pause flight animations.
        3.  Reduce the Mapbox render frame rate.
    *   **Benefit:** Significantly reduces GPU load and battery consumption on laptops.

*   **Suggestion:** Limit Frame Rate.
    *   **How:** Mapbox GL JS often tries to render at 60 FPS. For a wallpaper, 20-30 FPS is often sufficient. This can be controlled by throttling the `requestAnimationFrame` loops in `globe.js`.
    *   **Benefit:** Lower CPU and GPU usage.

## 3. Efficient Flight Tracking

The current flight tracking implementation fetches all global flights and animates them every 200ms.

*   **Suggestion:** Filter flights by visibility.
    *   **How:** Only animate flights that are within the current map view. As the user zooms in, the number of flights to track and render should decrease.
    *   **Benefit:** Reduces CPU overhead of calculating new positions for thousands of aircraft that aren't even visible on the screen.

*   **Suggestion:** Increase Fetch Intervals.
    *   **How:** Flight data from OpenSky is currently fetched every 5 minutes. Depending on the zoom level, this could be even less frequent if the user is at a global view where movement is less perceptible.
    *   **Benefit:** Reduces load on the OpenSky API and saves bandwidth.

## 4. Optimized Weather and Radar Updates

*   **Suggestion:** Synchronized Radar Fetching.
    *   **How:** Similar to flight data, the RainViewer radar tile URL should be fetched once in Swift and distributed to all WebViews.
    *   **Benefit:** Avoids redundant network requests.

*   **Suggestion:** Cache Geocoding Results.
    *   **How:** The app reverse-geocodes coordinates to get city names. These results should be cached persistently (e.g., in `UserDefaults` or `localStorage`) so that switching between locations doesn't trigger a new API call for a previously visited coordinate.
    *   **Benefit:** Faster UI updates and fewer API calls.

## 5. WebKit Optimization

*   **Suggestion:** Shared Web Process.
    *   **How:** Ensure that all `WKWebView` instances share the same `WKProcessPool`.
    *   **Benefit:** This allows the WebViews to share some memory and network resources, reducing the overall footprint of the application.

## 6. Minimize JavaScript Overhead

*   **Suggestion:** Consolidate Timers.
    *   **How:** Instead of having multiple `setInterval` calls (for the clock, terminator, weather updates), use a single management loop or leverage Swift to trigger these updates.
    *   **Benefit:** Reduces the number of times the JavaScript engine needs to wake up and execute code.
