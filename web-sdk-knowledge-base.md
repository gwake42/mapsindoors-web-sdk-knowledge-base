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
Ã¢ÂÂ Ã¯Â¸Â Must wait for MapsIndoors ready event before applying display rules
Ã¢ÂÂ Ã¯Â¸Â Additional setTimeout may be needed for complex solutions
Ã¢ÂÂ Ã¯Â¸Â Use MapboxV3View instead of deprecated MapboxView
Ã¢ÂÂ Ã¯Â¸Â Set multiple opacity and visibility properties to ensure complete hiding
Ã¢ÂÂ Ã¯Â¸Â Keep visible: true to maintain location data accessibility


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
                debugLog('â MapboxV3View created', 'success');

                debugLog('Creating MapsIndoors instance...', 'info');
                const mapsIndoorsInstance = new mapsindoors.MapsIndoors({
                    mapView: mapView
                });
                debugLog('â MapsIndoors instance created', 'success');

                // Add ready listener
                mapsIndoorsInstance.addListener('ready', () => {
                    debugLog('â MapsIndoors is READY!', 'success');
                    
                    // Add floor selector
                    const floorSelectorElement = document.createElement('div');
                    new mapsindoors.FloorSelector(floorSelectorElement, mapsIndoorsInstance);
                    
                    const mapboxInstance = mapView.getMap();
                    mapboxInstance.addControl({
                        onAdd: function() { return floorSelectorElement; },
                        onRemove: function() {}
                    });
                    
                    debugLog('â Floor selector added', 'success');
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
            debugLog('â Mapbox GL JS loaded', 'success');

            debugLog('Checking for MapsIndoors SDK...', 'info');
            if (typeof mapsindoors === 'undefined') {
                debugLog('ERROR: MapsIndoors SDK not loaded!', 'error');
                return false;
            }
            debugLog('â MapsIndoors SDK loaded', 'success');

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
â ï¸ Script loading order is critical - MapsIndoors and Mapbox scripts must be loaded in the head before any JavaScript tries to use them
â ï¸ The MapboxV3View must be used instead of the older MapboxView
â ï¸ A timeout is needed to ensure scripts are fully loaded before initialization
â ï¸ Debug console helps identify exactly where initialization fails


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
⚠️ Must use MapboxV3View not MapboxView
⚠️ Load Mapbox scripts before MapsIndoors
⚠️ Wait for 'ready' event before using features
⚠️ Floor selector requires proper control setup

