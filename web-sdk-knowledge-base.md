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


---

## Status-Based Location Styling with Display Rules

### Context
Display rules provide immediate visual feedback for location status in real-time management applications

### Industry
healthcare

### Problem
Visual indication of location status for real-time monitoring and management

### Solution
```javascript
// Status-based location styling with display rules
const statusColors = {
    'clean': '#4caf50',
    'needs-cleaning': '#f44336', 
    'in-progress': '#ff9800',
    'available': '#4caf50',
    'occupied': '#f44336',
    'safe': '#4caf50',
    'unsafe': '#f44336'
};

// Group locations by status
const cleanRooms = roomsData.filter(room => room.status === 'clean').map(room => room.id);
const dirtyRooms = roomsData.filter(room => room.status === 'needs-cleaning').map(room => room.id);
const inProgressRooms = roomsData.filter(room => room.status === 'in-progress').map(room => room.id);

// Apply display rules for each status group
mapsIndoorsInstance.setDisplayRule(cleanRooms, {
    polygonVisible: true,
    polygonFillColor: statusColors.clean,
    polygonFillOpacity: 0.5,
    polygonStrokeColor: statusColors.clean,
    polygonStrokeOpacity: 0.8,
    polygonStrokeWidth: 1,
    zoomFrom: 16
});

mapsIndoorsInstance.setDisplayRule(dirtyRooms, {
    polygonVisible: true,
    polygonFillColor: statusColors['needs-cleaning'],
    polygonFillOpacity: 0.5,
    polygonStrokeColor: statusColors['needs-cleaning'],
    polygonStrokeOpacity: 0.8,
    polygonStrokeWidth: 1,
    zoomFrom: 16
});

// Highlight selected location with enhanced styling
mapsIndoorsInstance.setDisplayRule(selectedLocationId, {
    polygonVisible: true,
    polygonFillOpacity: 0.8,
    polygonStrokeOpacity: 1,
    polygonStrokeWidth: 3,
    zoomFrom: 16
});

// Reset to default styling
mapsIndoorsInstance.setDisplayRule(locationId, null);
```

### Explanation
Display rules allow dynamic visual styling of locations based on their status. This pattern groups locations by status and applies consistent color coding, essential for dashboards showing cleaning status, room availability, emergency conditions, or any real-time location state.

### Use Cases
- Cleaning management dashboards
- Room booking systems
- Emergency response applications
- Asset tracking systems
- Space utilization monitoring

### Important Notes
⚠️ Use polygonVisible: true to show location boundaries
⚠️ Set appropriate zoomFrom level to avoid performance issues at high zoom
⚠️ Reset display rules with null to return to default styling
⚠️ Group locations by status for efficient batch updates
⚠️ Use consistent color schemes across your application

