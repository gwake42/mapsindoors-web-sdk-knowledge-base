# MapsIndoors Web SDK Knowledge Base

A community-driven collection of MapsIndoors implementation patterns and use cases.

## How to Contribute

Use the MapsIndoors Knowledge Base tools to add your implementations and help other developers.

## Knowledge Base Entries

*Entries will appear below as they are contributed by the community.*


---

## MapsIndoors SDK Initialization with MapboxV3View

### Context
MapsIndoors 4.41.1 requires MapboxV3View for Mapbox 3.8.0 compatibility

### Industry
healthcare

### Problem
Proper initialization of MapsIndoors with Mapbox integration using correct SDK versions

### Solution
```javascript
// Correct script loading order in HTML
<!-- Mapbox CSS (always load first) -->
<link href="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.css" rel="stylesheet">
<!-- Mapbox JavaScript -->
<script src="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.js"></script>
<!-- MapsIndoors JavaScript -->
<script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz?apikey=YOUR_API_KEY"></script>

// JavaScript initialization
const mapViewOptions = {
    accessToken: 'your-mapbox-token',
    element: document.getElementById('map'),
    center: { lat: 30.3603212, lng: -97.7422623 },
    zoom: 20,
    maxZoom: 22,
};

// IMPORTANT: Use MapboxV3View (not MapboxView)
const mapViewInstance = new mapsindoors.mapView.MapboxV3View(mapViewOptions);
const mapsIndoorsInstance = new mapsindoors.MapsIndoors({
    mapView: mapViewInstance,
});
const mapboxInstance = mapViewInstance.getMap();

// Add floor selector
const floorSelectorElement = document.createElement('div');
new mapsindoors.FloorSelector(floorSelectorElement, mapsIndoorsInstance);
mapboxInstance.addControl({ 
    onAdd: function () { return floorSelectorElement },
    onRemove: function () { }
});

// Wait for ready event
mapsIndoorsInstance.addListener('ready', () => {
    console.log('MapsIndoors is ready');
    // Your initialization code here
});
```

### Explanation
This pattern ensures proper compatibility between MapsIndoors and Mapbox. The key is using MapboxV3View instead of the deprecated MapboxView, loading scripts in the correct order, and waiting for the ready event before executing initialization code.

### Use Cases
- Any MapsIndoors application with Mapbox integration
- Indoor mapping applications
- Location-based services
- Navigation applications

### Important Notes
⚠️ Always use MapboxV3View with current SDK versions
⚠️ Load Mapbox CSS first, then JS, then MapsIndoors
⚠️ Wait for 'ready' event before using MapsIndoors methods
⚠️ API key must be included in the MapsIndoors script URL

