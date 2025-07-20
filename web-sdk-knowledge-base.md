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
⚠️ Must wait for MapsIndoors ready event before applying display rules
⚠️ Additional setTimeout may be needed for complex solutions
⚠️ Use MapboxV3View instead of deprecated MapboxView
⚠️ Set multiple opacity and visibility properties to ensure complete hiding
⚠️ Keep visible: true to maintain location data accessibility

