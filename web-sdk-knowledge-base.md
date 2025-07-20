# MapsIndoors Web SDK Knowledge Base

A community-driven collection of MapsIndoors implementation patterns and use cases.

## How to Contribute

Use the MapsIndoors Knowledge Base tools to add your implementations and help other developers.

## Knowledge Base Entries

*Entries will appear below as they are contributed by the community.*


---

## Hide Neighborhood Polygons and Extrusions with Display Rules

### Context
User needed to clean up a map view by removing visual clutter from Neighborhood polygons and extrusions while maintaining map functionality and keeping other location types visible

### Industry
corporate

### Problem
Need to disable polygon fill, stroke, and 3D extrusions for Neighborhood type locations while keeping the map clean and focused

### Solution
```javascript
// Initialize MapsIndoors with MapboxV3View
const mapViewOptions = {
    accessToken: 'your-mapbox-token',
    element: document.getElementById('map'),
    center: { lat: 40.75391251837155, lng: -73.97496487462143 },
    zoom: 15,
    maxZoom: 22,
};

const mapViewInstance = new mapsindoors.mapView.MapboxV3View(mapViewOptions);
const mapsIndoorsInstance = new mapsindoors.MapsIndoors({
    mapView: mapViewInstance,
});

// Wait for MapsIndoors to be ready before applying display rules
mapsIndoorsInstance.addListener('ready', async () => {
    console.log('MapsIndoors is ready');
    
    // Wait additional time to ensure everything is fully loaded
    setTimeout(async () => {
        try {
            // Get all Neighborhood type locations
            const neighborhoods = await mapsindoors.services.LocationsService.getLocations({
                types: ['Neighborhood'],
                take: 1000
            });
            
            console.log(`Found ${neighborhoods.length} neighborhoods`);
            
            if (neighborhoods.length > 0) {
                // Extract neighborhood IDs
                const neighborhoodIds = neighborhoods.map(neighborhood => neighborhood.id);
                
                // Apply comprehensive display rule to hide all visual elements
                mapsIndoorsInstance.setDisplayRule(neighborhoodIds, {
                    visible: true, // Keep location data accessible
                    polygonVisible: false, // Completely hide polygons
                    polygonFillOpacity: 0, // Disable polygon fill
                    polygonStrokeOpacity: 0, // Disable polygon stroke
                    polygonStrokeWidth: 0, // Set stroke width to 0
                    extrusionVisible: false, // Disable 3D extrusions
                    extrusionOpacity: 0, // Set extrusion opacity to 0
                    extrusionHeight: 0, // Set extrusion height to 0
                    iconVisible: true, // Keep icons if any
                    labelVisible: true, // Keep labels if any
                    zoomFrom: 1,
                    zoomTo: 25
                });
                
                console.log(`Applied display rules to ${neighborhoodIds.length} neighborhoods`);
            }
        } catch (error) {
            console.error('Error applying neighborhood display rules:', error);
        }
    }, 2000); // Wait 2 seconds after ready event
});
```

### Explanation
This code uses MapsIndoors display rules to selectively hide visual elements of Neighborhood type locations. The key is waiting for MapsIndoors to be fully ready, then fetching all Neighborhood locations and applying a comprehensive display rule that disables polygons, strokes, and 3D extrusions while keeping icons and labels visible. The setTimeout ensures all map elements are loaded before applying the rules.

### Use Cases
- Clean map interfaces without neighborhood visual clutter
- Custom styling for specific location types
- Hiding unwanted polygon elements in indoor maps
- Selective display rule application based on location type

### Important Notes
ÃÂÃÂ¢ÃÂÃÂÃÂÃÂ ÃÂÃÂ¯ÃÂÃÂ¸ÃÂÃÂ Must wait for MapsIndoors ready event before applying display rules
ÃÂÃÂ¢ÃÂÃÂÃÂÃÂ ÃÂÃÂ¯ÃÂÃÂ¸ÃÂÃÂ Additional setTimeout may be needed for complex solutions
ÃÂÃÂ¢ÃÂÃÂÃÂÃÂ ÃÂÃÂ¯ÃÂÃÂ¸ÃÂÃÂ Use MapboxV3View instead of deprecated MapboxView
ÃÂÃÂ¢ÃÂÃÂÃÂÃÂ ÃÂÃÂ¯ÃÂÃÂ¸ÃÂÃÂ Set multiple opacity and visibility properties to ensure complete hiding
ÃÂÃÂ¢ÃÂÃÂÃÂÃÂ ÃÂÃÂ¯ÃÂÃÂ¸ÃÂÃÂ Keep visible: true to maintain location data accessibility


---

## Basic MapsIndoors Mall Demo with Debug Console

### Context
User was experiencing a blank screen when trying to load MapsIndoors. The issue was that the MapsIndoors and Mapbox scripts were being loaded at the bottom of the page after the JavaScript code tried to use them. Moving the scripts to the head section fixed the loading order issue.

### Industry
retail

### Problem
Getting a blank screen when trying to load a basic MapsIndoors mall demo due to improper script loading order

### Solution
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MapsIndoors Mall Demo - Debug Version</title>
    
    <!-- Load scripts in head to ensure they're available -->
    <script src="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.js"></script>
    <script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz?apikey=YOUR_API_KEY"></script>
    
    <!-- Mapbox CSS -->
    <link href="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.css" rel="stylesheet">
    
    <style>
        body {
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
        }
        
        #debug-info {
            position: fixed;
            top: 10px;
            left: 10px;
            background: white;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.2);
            z-index: 10000;
            max-width: 400px;
            font-size: 12px;
            max-height: 200px;
            overflow-y: auto;
        }
        
        #map {
            width: 100vw;
            height: 100vh;
            background-color: #e0e0e0;
        }
        
        .log-entry {
            margin: 2px 0;
            padding: 2px;
        }
        
        .log-error {
            color: red;
            font-weight: bold;
        }
        
        .log-success {
            color: green;
        }
        
        .log-info {
            color: blue;
        }
    </style>
</head>
<body>
    <div id="debug-info">
        <strong>Debug Console:</strong>
        <div id="debug-log"></div>
    </div>
    
    <div id="map"></div>

    <script>
        // Configuration
        const CONFIG = {
            MAPBOX_TOKEN: 'pk.eyJ1IjoiZ2V3YS1tYXBzcGVvcGxlIiwiYSI6ImNsZzJudDB4ZTAwcnEzZnAwb2VvbTYwYnIifQ.w-cnsU-xP9jaly_qrgy_iA',
            MAPSINDOORS_API_KEY: '350bfad2c9a944958b13d6d8',
            MAP_CENTER: {
                lat: 30.470393399091968,
                lng: -97.80686386810669
            },
            INITIAL_ZOOM: 18
        };

        // Debug logging function
        function debugLog(message, type = 'info') {
            const logDiv = document.getElementById('debug-log');
            const logEntry = document.createElement('div');
            logEntry.className = `log-entry log-${type}`;
            logEntry.textContent = `${new Date().toLocaleTimeString()}: ${message}`;
            logDiv.appendChild(logEntry);
            logDiv.scrollTop = logDiv.scrollHeight;
            console.log(`[${type.toUpperCase()}] ${message}`);
        }

        // Test MapsIndoors initialization
        function testMapsIndoors() {
            debugLog('Testing MapsIndoors initialization...', 'info');
            
            try {
                debugLog('Creating MapboxV3View...', 'info');
                
                const mapViewOptions = {
                    accessToken: CONFIG.MAPBOX_TOKEN,
                    element: document.getElementById('map'),
                    center: CONFIG.MAP_CENTER,
                    zoom: CONFIG.INITIAL_ZOOM
                };

                const mapView = new mapsindoors.mapView.MapboxV3View(mapViewOptions);
                debugLog('ÃÂ¢ÃÂÃÂ MapboxV3View created', 'success');

                debugLog('Creating MapsIndoors instance...', 'info');
                const mapsIndoorsInstance = new mapsindoors.MapsIndoors({
                    mapView: mapView
                });
                debugLog('ÃÂ¢ÃÂÃÂ MapsIndoors instance created', 'success');

                // Add ready listener
                mapsIndoorsInstance.addListener('ready', () => {
                    debugLog('ÃÂ¢ÃÂÃÂ MapsIndoors is READY!', 'success');
                    
                    // Add floor selector
                    const floorSelectorElement = document.createElement('div');
                    new mapsindoors.FloorSelector(floorSelectorElement, mapsIndoorsInstance);
                    
                    const mapboxInstance = mapView.getMap();
                    mapboxInstance.addControl({
                        onAdd: function() { return floorSelectorElement; },
                        onRemove: function() {}
                    });
                    
                    debugLog('ÃÂ¢ÃÂÃÂ Floor selector added', 'success');
                    debugLog('Mall demo ready!', 'success');
                });

                // Add error listener
                mapsIndoorsInstance.addListener('error', (error) => {
                    debugLog(`MapsIndoors error: ${error.message || error}`, 'error');
                });

                debugLog('Waiting for MapsIndoors ready event...', 'info');

            } catch (error) {
                debugLog(`MapsIndoors error: ${error.message}`, 'error');
                console.error('Full error:', error);
            }
        }

        // Check if required libraries are loaded
        function checkLibraries() {
            debugLog('Checking for Mapbox GL JS...', 'info');
            if (typeof mapboxgl === 'undefined') {
                debugLog('ERROR: Mapbox GL JS not loaded!', 'error');
                return false;
            }
            debugLog('ÃÂ¢ÃÂÃÂ Mapbox GL JS loaded', 'success');

            debugLog('Checking for MapsIndoors SDK...', 'info');
            if (typeof mapsindoors === 'undefined') {
                debugLog('ERROR: MapsIndoors SDK not loaded!', 'error');
                return false;
            }
            debugLog('ÃÂ¢ÃÂÃÂ MapsIndoors SDK loaded', 'success');

            return true;
        }

        // Global error handlers
        window.addEventListener('error', (e) => {
            debugLog(`JavaScript error: ${e.message}`, 'error');
        });

        window.addEventListener('unhandledrejection', (e) => {
            debugLog(`Unhandled promise rejection: ${e.reason}`, 'error');
        });

        // Start the testing process
        debugLog('Starting MapsIndoors mall demo...', 'info');

        // Wait for scripts to load, then start testing
        setTimeout(() => {
            if (checkLibraries()) {
                testMapsIndoors();
            } else {
                debugLog('Cannot proceed - required libraries missing', 'error');
            }
        }, 1000);

    </script>

</body>
</html>
```

### Explanation
This is a basic MapsIndoors mall demo with debugging capabilities to help troubleshoot loading issues. The key fix was moving the script tags to the head section to ensure proper loading order. The code includes a debug console that shows real-time status updates during initialization, making it easy to identify where problems occur.

### Use Cases
- Creating a basic mall mapping application
- Troubleshooting MapsIndoors loading issues
- Setting up a foundation for retail wayfinding demos
- Debugging script loading problems

### Important Notes
ÃÂ¢ÃÂÃÂ ÃÂ¯ÃÂ¸ÃÂ Script loading order is critical - MapsIndoors and Mapbox scripts must be loaded in the head before any JavaScript tries to use them
ÃÂ¢ÃÂÃÂ ÃÂ¯ÃÂ¸ÃÂ The MapboxV3View must be used instead of the older MapboxView
ÃÂ¢ÃÂÃÂ ÃÂ¯ÃÂ¸ÃÂ A timeout is needed to ensure scripts are fully loaded before initialization
ÃÂ¢ÃÂÃÂ ÃÂ¯ÃÂ¸ÃÂ Debug console helps identify exactly where initialization fails


---

## MapsIndoors SDK Initialization with MapboxV3View

### Context
Essential foundation pattern needed for all MapsIndoors + Mapbox applications

### Industry
universal

### Problem
Need to properly initialize MapsIndoors with Mapbox integration using the correct view class and script loading order

### Solution
```javascript
<!-- Always load Mapbox first -->
<link href="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.css" rel="stylesheet">
<script src="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.js"></script>

<!-- Then load MapsIndoors -->
<script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz?apikey=YOUR_API_KEY"></script>

<script>
// Use MapboxV3View (NOT MapboxView)
const mapViewOptions = {
    accessToken: 'your-mapbox-token',
    element: document.getElementById('map'),
    center: { lat: 30.3603212, lng: -97.7422623 },
    zoom: 20,
    maxZoom: 22,
};

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

// Wait for ready before using features
mapsIndoorsInstance.addListener('ready', () => {
    console.log('MapsIndoors is ready');
    // Initialize your application logic here
});
</script>
```

### Explanation
This establishes the correct initialization pattern for MapsIndoors with Mapbox. Key points: load Mapbox scripts first, use MapboxV3View (not MapboxView), wait for the 'ready' event before using MapsIndoors features, and properly add the floor selector control.

### Use Cases
- Web application setup
- Map initialization
- SDK integration

### Important Notes
Ã¢ÂÂ Ã¯Â¸Â Must use MapboxV3View not MapboxView
Ã¢ÂÂ Ã¯Â¸Â Load Mapbox scripts before MapsIndoors
Ã¢ÂÂ Ã¯Â¸Â Wait for 'ready' event before using features
Ã¢ÂÂ Ã¯Â¸Â Floor selector requires proper control setup


---

## Interactive Location Highlighting with Status-Based Display Rules

### Context
Interactive dashboard where users need to visualize different room statuses and select specific rooms for detailed interaction

### Industry
healthcare

### Problem
Need to visually represent different room statuses (clean, dirty, in-progress) and allow users to interact with specific rooms

### Solution
```javascript
// Dynamic room highlighting with status-based colors
function updateRoomDisplayRules(rooms, selectedRoomId = null) {
    // Group rooms by status
    const roomsByStatus = {
        clean: rooms.filter(room => room.status === 'clean').map(room => room.id),
        needsCleaning: rooms.filter(room => room.status === 'needs-cleaning').map(room => room.id),
        inProgress: rooms.filter(room => room.status === 'in-progress').map(room => room.id)
    };

    // Apply status-based display rules
    mapsIndoorsInstance.setDisplayRule(roomsByStatus.clean, {
        polygonVisible: true,
        polygonFillColor: '#4CAF50', // Green for clean
        polygonFillOpacity: 0.3,
        polygonStrokeColor: '#4CAF50',
        polygonStrokeOpacity: 0.8,
        polygonStrokeWidth: 1,
        zoomFrom: 16
    });

    mapsIndoorsInstance.setDisplayRule(roomsByStatus.needsCleaning, {
        polygonVisible: true,
        polygonFillColor: '#f44336', // Red for needs cleaning
        polygonFillOpacity: 0.3,
        polygonStrokeColor: '#f44336',
        polygonStrokeOpacity: 0.8,
        polygonStrokeWidth: 1,
        zoomFrom: 16
    });

    mapsIndoorsInstance.setDisplayRule(roomsByStatus.inProgress, {
        polygonVisible: true,
        polygonFillColor: '#ff9800', // Orange for in progress
        polygonFillOpacity: 0.3,
        polygonStrokeColor: '#ff9800',
        polygonStrokeOpacity: 0.8,
        polygonStrokeWidth: 1,
        zoomFrom: 16
    });

    // Highlight selected room with enhanced styling
    if (selectedRoomId) {
        mapsIndoorsInstance.setDisplayRule(selectedRoomId, {
            polygonVisible: true,
            polygonFillOpacity: 0.6,
            polygonStrokeOpacity: 1,
            polygonStrokeWidth: 3,
            zoomFrom: 16
        });
    }
}

// Click handler for room selection
mapsIndoorsInstance.addListener('click', (event) => {
    if (event && event.id) {
        const room = rooms.find(r => r.id === event.id);
        if (room) {
            selectRoom(room.id);
            updateRoomDisplayRules(rooms, room.id);
        }
    }
});

// Update room status and refresh display
function updateRoomStatus(roomId, newStatus) {
    const room = rooms.find(r => r.id === roomId);
    if (room) {
        room.status = newStatus;
        room.lastUpdated = new Date();
        updateRoomDisplayRules(rooms, roomId);
    }
}
```

### Explanation
This pattern enables dynamic visual feedback by applying different colors and styles to locations based on their status. The setDisplayRule method can target multiple locations at once using arrays of IDs, and selected items get enhanced styling. Click events allow users to interact with locations directly on the map.

### Use Cases
- Cleaning management dashboards
- Room booking systems
- Emergency response applications
- Asset tracking interfaces

### Important Notes
â ï¸ setDisplayRule accepts both single IDs and arrays
â ï¸ Display rules are cumulative - later rules override earlier ones
â ï¸ zoomFrom parameter controls when styling appears
â ï¸ polygonStrokeWidth affects visual hierarchy


---

## Time-Controlled Heatmap Visualization with Mapbox Integration

### Context
Analytics dashboard showing foot traffic or density patterns throughout a building over different time periods

### Industry
retail

### Problem
Need to visualize density or activity patterns that change over time periods within indoor spaces

### Solution
```javascript
// Add heatmap source and layer to Mapbox
mapboxInstance.addSource('heatmap-source', {
    'type': 'geojson',
    'data': {
        type: "FeatureCollection",
        features: [] // Will be populated with data points
    }
});

mapboxInstance.addLayer({
    'id': 'heatmap-layer',
    'type': 'heatmap',
    'source': 'heatmap-source',
    'paint': {
        'heatmap-weight': [
            'interpolate',
            ['linear'],
            ['get', 'intensity'],
            0, 0,
            5, 0.5,
            10, 1
        ],
        'heatmap-intensity': [
            'interpolate',
            ['linear'],
            ['zoom'],
            0, 1,
            9, 3,
            16, 5,
            22, 10
        ],
        'heatmap-color': [
            'interpolate',
            ['linear'],
            ['heatmap-density'],
            0, 'rgba(33,102,172,0)',
            0.2, 'rgb(103,169,207)',
            0.4, 'rgb(209,229,240)',
            0.6, 'rgb(253,219,199)',
            0.8, 'rgb(239,138,98)',
            1, 'rgb(178,24,43)'
        ],
        'heatmap-radius': [
            'interpolate',
            ['linear'],
            ['zoom'],
            0, 2,
            9, 20,
            16, 40,
            22, 80
        ],
        'heatmap-opacity': 0.8
    }
});

// Update heatmap with time-filtered data
function updateHeatmap(hour) {
    const filteredData = heatmapData.filter(feature => feature.properties.Hour === hour);
    
    const geojsonData = {
        type: "FeatureCollection",
        features: filteredData
    };

    mapboxInstance.getSource('heatmap-source').setData(geojsonData);
}

// Time slider control
const timeSlider = document.createElement('input');
timeSlider.type = 'range';
timeSlider.min = 8;
timeSlider.max = 22;
timeSlider.value = 12;
timeSlider.oninput = (e) => {
    const hour = parseInt(e.target.value);
    updateHeatmap(hour);
};

// Generate sample data points within building bounds
function generateHeatmapData() {
    const data = [];
    for (let hour = 8; hour <= 22; hour++) {
        for (let i = 0; i < 100; i++) {
            const point = getRandomPointInBuilding();
            data.push({
                type: "Feature",
                properties: {
                    intensity: Math.floor(Math.random() * 10) + 1,
                    Hour: hour
                },
                geometry: {
                    type: "Point",
                    coordinates: point
                }
            });
        }
    }
    return data;
}
```

### Explanation
This creates a time-controlled heatmap overlay using Mapbox's native heatmap layer. The heatmap visualizes density data that changes over time, with a slider control allowing users to see patterns at different hours. The intensity property controls heat values, and the layer styling creates smooth color gradients from low to high density areas.

### Use Cases
- Foot traffic analysis
- Equipment usage patterns
- Crowd density monitoring
- Space utilization studies

### Important Notes
⚠️ Heatmap layer must be added after map loads
⚠️ GeoJSON features need 'intensity' property for heatmap-weight
⚠️ Radius scales with zoom level for proper visualization
⚠️ Filter data by time before updating source

