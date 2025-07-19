

---

## Use Case: Basic Map Initialization with Floor Selector

### Problem
You need to set up a MapsIndoors map with the basic controls that users expect, including a floor selector.

### Code Example
```js
// Initialize map with MapboxV3View
const mapViewOptions = {
    accessToken: 'your-mapbox-token',
    element: document.getElementById('map'),
    center: { lat: 30.3603212, lng: -97.7422623 },
    zoom: 19,
    maxZoom: 24
};

const mapView = new mapsindoors.mapView.MapboxV3View(mapViewOptions);
const mapsIndoorsInstance = new mapsindoors.MapsIndoors({
    mapView: mapView
});

// Add floor selector
const floorSelectorElement = document.createElement('div');
new mapsindoors.FloorSelector(floorSelectorElement, mapsIndoorsInstance);
mapView.getMap().addControl({
    onAdd: () => floorSelectorElement,
    onRemove: () => {}
});
```

### Explanation
- Creates a MapboxV3View instance (not MapboxView)
- Initializes MapsIndoors with the map view
- Adds the native floor selector as a Mapbox control
- The floor selector automatically updates when building changes

### Gotchas / Notes
ð ï¸ Always use `MapboxV3View` instead of `MapboxView` for SDK v4.41.1
ð ï¸ The floor selector won't appear until a building is loaded

---

## Use Case: Center Map on Specific Location

### Problem
You need to fly the camera to a specific location and ensure the correct floor is displayed.

### Code Example
```js
// Get location by ID
const locationId = '9acba35b0bce4ca89e867807';
const location = await mapsindoors.services.LocationsService.getLocation(locationId);

if (location) {
    // Extract coordinates
    const coords = location.properties.anchor.coordinates;
    
    // Set the floor first
    mapsIndoorsInstance.setFloor(location.properties.floor);
    
    // Then fly to location
    mapView.getMap().flyTo({
        center: [coords[0], coords[1]],
        zoom: 21,
        duration: 1000
    });
}
```

### Explanation
- Uses LocationsService to fetch location data
- Sets floor before moving camera (prevents jarring floor change)
- Uses Mapbox's flyTo for smooth animation
- Coordinates are in [lng, lat] format (not [lat, lng])

### Gotchas / Notes
ð ï¸ Always set floor before flying to location
ð ï¸ MapsIndoors uses [lng, lat] while some APIs use [lat, lng]

---

## Use Case: Filter and Display Locations by Type

### Problem
You want to show only specific types of locations (e.g., meeting rooms) on the map.

### Code Example
```js
// Get locations by type
const meetingRooms = await mapsindoors.services.LocationsService.getLocations({
    types: ['MeetingRoom', 'MeetingRoom Small', 'MeetingRoom Large'],
    venue: 'AUSTINOFFICE',
    take: 100
});

// Extract location IDs
const roomIds = meetingRooms.map(room => room.id);

// Apply custom display rules
mapsIndoorsInstance.setDisplayRule(roomIds, {
    visible: true,
    polygonVisible: true,
    polygonFillColor: '#3f51b5',
    polygonFillOpacity: 0.3,
    polygonStrokeColor: '#3f51b5',
    polygonStrokeOpacity: 0.8,
    polygonStrokeWidth: 2,
    zoomFrom: 16,
    zoomTo: 24
});

// Hide all other locations
mapsIndoorsInstance.setDisplayRule(null, {
    visible: false
});
```

### Explanation
- Fetches locations filtered by type
- Applies custom visual styling to matching locations
- Hides all other locations by setting display rule on null
- Display rules persist until changed

### Gotchas / Notes
ð ï¸ Setting display rule on `null` affects all locations not explicitly styled
ð ï¸ The `take` parameter limits results (default is 10)

---

## Use Case: Handle Location Click Events

### Problem
You need to respond when users click on locations and show relevant information.

### Code Example
```js
// Listen for click events
mapsIndoorsInstance.addListener('click', async (event) => {
    if (!event?.id) return;
    
    // Get full location data
    const location = await mapsindoors.services.LocationsService.getLocation(event.id);
    
    if (location) {
        // Show location info
        console.log('Clicked:', location.properties.name);
        console.log('Type:', location.properties.type);
        console.log('Floor:', location.properties.floor);
        
        // Highlight the clicked location
        mapsIndoorsInstance.setDisplayRule(event.id, {
            polygonFillOpacity: 0.7,
            polygonStrokeWidth: 3,
            polygonStrokeColor: '#ff5722'
        });
    }
});
```

### Explanation
- The click event provides only the location ID
- Fetch full location data using LocationsService
- Event object may be null if clicking empty space
- Can apply temporary highlight styling

### Gotchas / Notes
ð ï¸ Click event only provides `id`, not full location data
ð ï¸ Always check if `event` and `event.id` exist

---

## Use Case: Search Locations Programmatically

### Problem
You need to search for locations based on user input and display results.

### Code Example
```js
async function searchLocations(query) {
    try {
        const results = await mapsindoors.services.LocationsService.getLocations({
            q: query,           // Search query
            take: 20,          // Max results
            orderBy: 'relevance'
        });
        
        if (results.length > 0) {
            // Highlight search results
            const resultIds = results.map(loc => loc.id);
            
            // Dim all locations
            mapsIndoorsInstance.setDisplayRule(null, {
                polygonFillOpacity: 0.1
            });
            
            // Highlight results
            mapsIndoorsInstance.setDisplayRule(resultIds, {
                polygonFillOpacity: 0.6,
                polygonStrokeColor: '#4CAF50',
                polygonStrokeWidth: 2
            });
            
            // Fit bounds to show all results
            const bounds = new mapboxgl.LngLatBounds();
            results.forEach(location => {
                const coords = location.properties.anchor.coordinates;
                bounds.extend([coords[0], coords[1]]);
            });
            
            mapView.getMap().fitBounds(bounds, { padding: 50 });
        }
        
        return results;
    } catch (error) {
        console.error('Search failed:', error);
        return [];
    }
}
```

### Explanation
- Uses the `q` parameter for text search
- Can combine with filters like `types` or `categories`
- Highlights results while dimming other locations
- Fits map bounds to show all results

### Gotchas / Notes
ð ï¸ Search is case-insensitive but requires minimum 2 characters
ð ï¸ `orderBy: 'relevance'` gives better results than alphabetical

---

## Use Case: Get Nearby Locations with Distance

### Problem
You need to find locations near a point and know their distances.

### Code Example
```js
// Find locations near a coordinate
const nearbyLocations = await mapsindoors.services.LocationsService.getLocations({
    near: { lat: 30.3603212, lng: -97.7422623 },
    radius: 50,  // meters
    take: 10
});

// Calculate distances using Distance Matrix
if (nearbyLocations.length > 0) {
    const origin = `${30.3603212},${-97.7422623},0`; // lat,lng,floor
    const destinations = nearbyLocations.map(loc => 
        `${loc.properties.anchor.coordinates[1]},${loc.properties.anchor.coordinates[0]},${loc.properties.floor}`
    );
    
    const distanceMatrix = await mapsindoors.services.DistanceMatrixService.getDistanceMatrix({
        graphId: 'AUSTINOFFICE_Graph',
        origins: [origin],
        destinations: destinations
    });
    
    // Combine locations with distances
    const locationsWithDistance = nearbyLocations.map((loc, index) => ({
        ...loc,
        distance: distanceMatrix.rows[0].elements[index].distance.value,
        walkTime: distanceMatrix.rows[0].elements[index].duration.value
    }));
    
    // Sort by distance
    locationsWithDistance.sort((a, b) => a.distance - b.distance);
}
```

### Explanation
- `near` parameter accepts coordinates or 'location:ID' format
- Distance Matrix provides walking distance, not straight-line
- Format for matrix: "lat,lng,floor" as strings
- Returns distance in meters and duration in seconds

### Gotchas / Notes
ð ï¸ Distance Matrix requires the venue's graph ID
ð ï¸ Matrix coordinate format is different from location format

---

## Use Case: Listen for Floor Changes

### Problem
You need to update UI or data when the user changes floors.

### Code Example
```js
// Track current floor
let currentFloor = null;

// Listen for floor changes
mapsIndoorsInstance.addListener('floor_changed', () => {
    const newFloor = mapsIndoorsInstance.getFloor();
    console.log(`Floor changed from ${currentFloor} to ${newFloor}`);
    
    // Update any floor-dependent UI
    updateFloorIndicator(newFloor);
    
    // Refresh floor-specific data
    if (newFloor !== currentFloor) {
        loadFloorData(newFloor);
    }
    
    currentFloor = newFloor;
});

// Also listen for building changes
mapsIndoorsInstance.addListener('building_changed', () => {
    const building = mapsIndoorsInstance.getBuilding();
    if (building) {
        console.log('Building:', building.name);
        console.log('Available floors:', Object.keys(building.floors));
    }
});
```

### Explanation
- `floor_changed` fires when floor selection changes
- `getFloor()` returns the floor index as a number
- Building object contains floor metadata
- Floor 0 typically represents ground floor

### Gotchas / Notes
ð ï¸ Floor indices can be negative (basements)
ð ï¸ Floor change event fires before tiles load

---

## Use Case: Create Custom Location Markers

### Problem
You want to replace default location icons with custom markers.

### Code Example
```js
// Hide default MapsIndoors icons for specific locations
const customLocations = ['location-id-1', 'location-id-2'];

mapsIndoorsInstance.setDisplayRule(customLocations, {
    iconVisible: false,
    labelVisible: false,
    polygonVisible: true
});

// Add custom Mapbox markers
customLocations.forEach(async (locationId) => {
    const location = await mapsindoors.services.LocationsService.getLocation(locationId);
    if (!location) return;
    
    const coords = location.properties.anchor.coordinates;
    
    // Create custom marker element
    const el = document.createElement('div');
    el.className = 'custom-marker';
    el.innerHTML = 'ð';
    el.style.fontSize = '24px';
    el.style.cursor = 'pointer';
    
    // Add Mapbox marker
    const marker = new mapboxgl.Marker(el)
        .setLngLat([coords[0], coords[1]])
        .addTo(mapView.getMap());
    
    // Handle marker clicks
    el.addEventListener('click', () => {
        console.log('Custom marker clicked:', location.properties.name);
    });
});
```

### Explanation
- First hide MapsIndoors default icons
- Create custom HTML elements for markers
- Mapbox markers float above MapsIndoors layer
- Can use any HTML/CSS for marker styling

### Gotchas / Notes
ð ï¸ Custom markers don't respect floor visibility automatically
ð ï¸ Must manually show/hide markers on floor change

---

## Use Case: Wait for MapsIndoors Ready State

### Problem
You need to ensure MapsIndoors is fully loaded before executing code.

### Code Example
```js
// Promise wrapper for ready event
function waitForMapsIndoors(mapsIndoorsInstance) {
    return new Promise((resolve) => {
        mapsIndoorsInstance.addListener('ready', resolve);
    });
}

// Usage with async/await
async function initializeApp() {
    const mapView = new mapsindoors.mapView.MapboxV3View(mapViewOptions);
    const mapsIndoorsInstance = new mapsindoors.MapsIndoors({ mapView });
    
    // Wait for ready
    await waitForMapsIndoors(mapsIndoorsInstance);
    
    // Now safe to use MapsIndoors features
    const venue = mapsIndoorsInstance.getVenue();
    console.log('Venue loaded:', venue.name);
    
    // Get solution config
    const solutionConfig = mapsIndoorsInstance.getSolutionConfig();
    console.log('Default language:', solutionConfig.defaultLanguage);
}
```

### Explanation
- 'ready' event fires when venue and solution data is loaded
- Many methods fail if called before ready
- Can wrap in Promise for cleaner async code
- Solution config available after ready

### Gotchas / Notes
ð ï¸ Don't call `getBuilding()` or `getVenue()` before ready
ð ï¸ Display rules can be set before ready but won't apply

---

## Use Case: Calculate Route with Waypoints

### Problem
You need to calculate an optimized route through multiple locations.

### Code Example
```js
const directionsService = new mapsindoors.services.DirectionsService();
const directionsRenderer = new mapsindoors.directions.DirectionsRenderer({
    mapsIndoors: mapsIndoorsInstance,
    fitBounds: true
});

// Define route points
const origin = { lat: 30.3605466, lng: -97.7424251, floor: 0 };
const destination = { lat: 30.3602822, lng: -97.7424575, floor: 20 };
const waypoints = [
    { lat: 30.3603212, lng: -97.7422623, floor: 0 },
    { lat: 30.3604212, lng: -97.7421623, floor: 0 }
];

// Calculate optimized route
const route = await directionsService.getRoute({
    origin: origin,
    destination: destination,
    stops: waypoints,
    optimize: true  // Optimize waypoint order
});

if (route) {
    // Display route
    directionsRenderer.setRoute(route);
    
    // Get route info
    console.log('Total distance:', route.distance.text);
    console.log('Total time:', route.duration.text);
    console.log('Number of legs:', route.legs.length);
}
```

### Explanation
- `optimize: true` reorders waypoints for shortest path
- Route includes floor changes (elevators/stairs)
- DirectionsRenderer handles multi-floor visualization
- Each leg represents a route segment

### Gotchas / Notes
ð ï¸ Maximum 25 waypoints per route
ð ï¸ Floor transitions require accessible paths in venue data

---

## Use Case: Control Location Label Visibility

### Problem
You want to show/hide location labels based on zoom level or user preference.

### Code Example
```js
// Hide all labels by default
mapsIndoorsInstance.setDisplayRule(null, {
    labelVisible: false,
    labelZoomFrom: 22,  // Only show at high zoom
    labelZoomTo: 24
});

// Show labels for specific location types
const importantTypes = ['Entrance', 'Elevator', 'Stairs', 'Restroom'];

const importantLocations = await mapsindoors.services.LocationsService.getLocations({
    types: importantTypes,
    take: 1000
});

const importantIds = importantLocations.map(loc => loc.id);

mapsIndoorsInstance.setDisplayRule(importantIds, {
    labelVisible: true,
    labelZoomFrom: 19,  // Show at lower zoom
    labelZoomTo: 24,
    labelMaxWidth: 100,
    iconVisible: true
});

// Toggle labels with button
document.getElementById('toggle-labels').addEventListener('click', () => {
    const currentVisibility = !labelsVisible;
    mapsIndoorsInstance.setDisplayRule(null, {
        labelVisible: currentVisibility
    });
    labelsVisible = currentVisibility;
});
```

### Explanation
- Labels can be controlled separately from icons
- `labelZoomFrom/To` sets zoom range for visibility
- `labelMaxWidth` prevents long labels from overlapping
- Can toggle all labels with null selector

### Gotchas / Notes
ð ï¸ Label collision detection works within zoom levels
ð ï¸ Labels follow language settings from SolutionConfig

---

## Use Case: Navigate Route Step by Step

### Problem
You need to provide turn-by-turn navigation through a route.

### Code Example
```js
let currentRoute = null;
let currentLegIndex = 0;

// Calculate and start navigation
async function startNavigation(originId, destId) {
    const origin = await mapsindoors.services.LocationsService.getLocation(originId);
    const dest = await mapsindoors.services.LocationsService.getLocation(destId);
    
    currentRoute = await directionsService.getRoute({
        origin: {
            lat: origin.properties.anchor.coordinates[1],
            lng: origin.properties.anchor.coordinates[0],
            floor: origin.properties.floor
        },
        destination: {
            lat: dest.properties.anchor.coordinates[1],
            lng: dest.properties.anchor.coordinates[0],
            floor: dest.properties.floor
        }
    });
    
    if (currentRoute) {
        directionsRenderer.setRoute(currentRoute);
        currentLegIndex = 0;
        updateNavigationUI();
    }
}

// Navigate to next/previous leg
function nextLeg() {
    if (currentRoute && currentLegIndex < currentRoute.legs.length - 1) {
        currentLegIndex++;
        directionsRenderer.nextLeg();
        updateNavigationUI();
    }
}

function previousLeg() {
    if (currentRoute && currentLegIndex > 0) {
        currentLegIndex--;
        directionsRenderer.previousLeg();
        updateNavigationUI();
    }
}

function updateNavigationUI() {
    const leg = currentRoute.legs[currentLegIndex];
    console.log(`Step ${currentLegIndex + 1} of ${currentRoute.legs.length}`);
    console.log('Distance:', leg.distance.text);
    console.log('Start floor:', leg.start_location.zLevel);
    console.log('End floor:', leg.end_location.zLevel);
    
    // Set map to show current leg floor
    mapsIndoorsInstance.setFloor(leg.start_location.zLevel);
}
```

### Explanation
- DirectionsRenderer has built-in leg navigation
- Each leg may span multiple floors
- Renderer highlights current leg automatically
- Can track progress through route

### Gotchas / Notes
ð ï¸ `nextLeg()` and `previousLeg()` don't return values
ð ï¸ Floor changes happen at leg boundaries

---

## Use Case: Export Map as Image

### Problem
You need to capture the current map view as a high-resolution image.

### Code Example
```js
async function exportMapAsImage(scale = 2) {
    const mapContainer = document.getElementById('map');
    const mapboxMap = mapView.getMap();
    
    // Store current state
    const originalDimensions = {
        width: mapContainer.style.width,
        height: mapContainer.style.height
    };
    const originalCenter = mapboxMap.getCenter();
    const originalZoom = mapboxMap.getZoom();
    
    try {
        // Set higher resolution
        mapContainer.style.width = `${mapContainer.offsetWidth * scale}px`;
        mapContainer.style.height = `${mapContainer.offsetHeight * scale}px`;
        
        // Resize and wait for render
        mapboxMap.resize();
        await new Promise(resolve => setTimeout(resolve, 1000));
        
        // Reset view (resize can shift it)
        mapboxMap.setCenter(originalCenter);
        mapboxMap.setZoom(originalZoom);
        await new Promise(resolve => setTimeout(resolve, 500));
        
        // Capture canvas
        const canvas = mapboxMap.getCanvas();
        const dataUrl = canvas.toDataURL('image/png');
        
        // Create download link
        const link = document.createElement('a');
        link.download = `map_${Date.now()}.png`;
        link.href = dataUrl;
        link.click();
        
    } finally {
        // Restore original dimensions
        mapContainer.style.width = originalDimensions.width;
        mapContainer.style.height = originalDimensions.height;
        mapboxMap.resize();
        mapboxMap.setCenter(originalCenter);
        mapboxMap.setZoom(originalZoom);
    }
}
```

### Explanation
- Temporarily increases canvas size for higher resolution
- Must wait for map to re-render at new size
- Canvas captures both Mapbox and MapsIndoors layers
- Always restore original dimensions

### Gotchas / Notes
ð ï¸ Enable `preserveDrawingBuffer: true` in Mapbox options
ð ï¸ Some browsers limit maximum canvas size

---

## Use Case: Search Within Specific Building/Venue

### Problem
You need to limit search results to a specific building or venue.

### Code Example
```js
// Search only in specific venue
const venueResults = await mapsindoors.services.LocationsService.getLocations({
    q: 'conference',
    venue: 'AUSTINOFFICE',  // Venue ID
    take: 50
});

// Search only on specific floor
const floorResults = await mapsindoors.services.LocationsService.getLocations({
    q: 'meeting',
    floor: 2,
    building: 'building-id',  // Optional building filter
    take: 50
});

// Search with multiple filters
const filteredResults = await mapsindoors.services.LocationsService.getLocations({
    q: 'room',
    types: ['MeetingRoom', 'ConferenceRoom'],
    categories: ['Bookable'],
    venue: 'AUSTINOFFICE',
    floor: 1,
    take: 100
});

// Search and group by building
const allResults = await mapsindoors.services.LocationsService.getLocations({
    q: searchTerm,
    take: 1000
});

const resultsByBuilding = allResults.reduce((groups, location) => {
    const building = location.properties.building || 'Unknown';
    if (!groups[building]) groups[building] = [];
    groups[building].push(location);
    return groups;
}, {});
```

### Explanation
- Venue filter is most restrictive (single venue)
- Can combine multiple filters
- Building property may be null for outdoor locations
- Categories filter requires exact match

### Gotchas / Notes
ð ï¸ Venue ID is different from venue name
ð ï¸ Floor filter applies across all buildings

---

## Use Case: Monitor Real-time Location Updates

### Problem
You need to show live positions that update in real-time (e.g., asset tracking).

### Code Example
```js
// Subscribe to position updates
const positionProvider = new mapsindoors.PositionControl(element, {
    mapsIndoors: mapsIndoorsInstance,
    positionOptions: {
        enableHighAccuracy: true,
        maximumAge: 1000,
        timeout: 15000
    }
});

// Track custom positions
const liveMarkers = new Map();

function updateLivePosition(deviceId, lat, lng, floor) {
    if (!liveMarkers.has(deviceId)) {
        // Create marker element
        const el = document.createElement('div');
        el.className = 'live-marker';
        el.innerHTML = `
            <div style="
                width: 20px;
                height: 20px;
                background: #4CAF50;
                border-radius: 50%;
                border: 3px solid white;
                box-shadow: 0 2px 4px rgba(0,0,0,0.3);
                animation: pulse 2s infinite;
            "></div>
        `;
        
        const marker = new mapboxgl.Marker(el)
            .setLngLat([lng, lat])
            .addTo(mapView.getMap());
            
        liveMarkers.set(deviceId, { marker, floor });
    } else {
        // Update existing marker
        const { marker, floor: currentFloor } = liveMarkers.get(deviceId);
        marker.setLngLat([lng, lat]);
        liveMarkers.set(deviceId, { marker, floor });
    }
    
    // Show/hide based on current floor
    updateLiveMarkerVisibility();
}

// Update visibility on floor change
mapsIndoorsInstance.addListener('floor_changed', updateLiveMarkerVisibility);

function updateLiveMarkerVisibility() {
    const currentFloor = mapsIndoorsInstance.getFloor();
    liveMarkers.forEach(({ marker, floor }) => {
        marker.getElement().style.display = 
            floor === currentFloor ? 'block' : 'none';
    });
}
```

### Explanation
- PositionControl handles device GPS positioning
- Custom markers for third-party position data
- Must manage floor visibility manually
- CSS animations indicate live status

### Gotchas / Notes
ð ï¸ GPS positions may need indoor mapping corrections
ð ï¸ Frequent updates can impact performance

---

## Use Case: Highlight Path to Nearest Amenity

### Problem
User needs to find the nearest restroom, exit, or other amenity from their location.

### Code Example
```js
async function findNearestAmenity(currentPosition, amenityType) {
    // Find all amenities of the type
    const amenities = await mapsindoors.services.LocationsService.getLocations({
        types: [amenityType],
        venue: 'AUSTINOFFICE',
        take: 100
    });
    
    if (amenities.length === 0) {
        console.log('No amenities found');
        return null;
    }
    
    // Use Distance Matrix to find actual walking distances
    const origin = `${currentPosition.lat},${currentPosition.lng},${currentPosition.floor}`;
    const destinations = amenities.map(loc => 
        `${loc.properties.anchor.coordinates[1]},${loc.properties.anchor.coordinates[0]},${loc.properties.floor}`
    );
    
    const distanceMatrix = await mapsindoors.services.DistanceMatrixService.getDistanceMatrix({
        graphId: 'AUSTINOFFICE_Graph',
        origins: [origin],
        destinations: destinations
    });
    
    // Find nearest by walking distance
    let nearestIndex = 0;
    let shortestDistance = Infinity;
    
    distanceMatrix.rows[0].elements.forEach((element, index) => {
        if (element.distance.value < shortestDistance) {
            shortestDistance = element.distance.value;
            nearestIndex = index;
        }
    });
    
    const nearestAmenity = amenities[nearestIndex];
    
    // Calculate and display route
    const route = await directionsService.getRoute({
        origin: currentPosition,
        destination: {
            lat: nearestAmenity.properties.anchor.coordinates[1],
            lng: nearestAmenity.properties.anchor.coordinates[0],
            floor: nearestAmenity.properties.floor
        }
    });
    
    if (route) {
        directionsRenderer.setRoute(route);
        
        // Highlight destination
        mapsIndoorsInstance.setDisplayRule(nearestAmenity.id, {
            polygonFillColor: '#4CAF50',
            polygonFillOpacity: 0.8,
            iconScale: 1.5
        });
    }
    
    return {
        amenity: nearestAmenity,
        distance: shortestDistance,
        route: route
    };
}

// Usage
const userPosition = { lat: 30.3603212, lng: -97.7422623, floor: 0 };
const nearest = await findNearestAmenity(userPosition, 'Restroom');
console.log(`Nearest restroom: ${nearest.amenity.properties.name}`);
console.log(`Distance: ${nearest.distance} meters`);
```

### Explanation
- Combines location search with distance matrix
- Finds nearest by actual walking distance, not straight line
- Automatically shows route to nearest option
- Can be used for any amenity type

### Gotchas / Notes
ð ï¸ Distance Matrix has a limit on origin/destination combinations
ð ï¸ Consider caching Distance Matrix results for performance

---

This series provides practical, annotated examples covering the most common MapsIndoors SDK use cases. Each example is self-contained and includes the specific gotchas developers often encounter. The format is consistent and ideal for chunking into an AI assistant's context.

## Review Queue

### VenuesService Building Data Retrieval

Complete example from codeExample.html showing how to fetch building data using VenuesService.getBuildings() without map rendering. Includes data normalization, floor sorting, and performance tracking.

Key features:
- No map initialization required
- Comprehensive building metadata extraction
- Floor data processing and sorting
- Error handling and fallback values
- Performance measurement

Example usage:
```javascript
const buildings = await mapsindoors.services.VenuesService.getBuildings();
const processed = buildings.map(building => ({
  id: building.id,
  name: building.name || `Building ${building.id}`,
  floors: Object.keys(building.floors || {}).sort((a,b) => parseInt(a) - parseInt(b)),
  floorCount: Object.keys(building.floors || {}).length
}));
```

Perfect for backend data processing, building inventories, and facility management systems.

### Building Data Retrieval Without Map - Complete Implementation

Real-world example from codeExample.html showing comprehensive building data fetching using VenuesService.getBuildings() without map rendering. Includes data processing, normalization, performance tracking, and UI display.

Code demonstrates:
- Async building data retrieval
- Data normalization with fallbacks
- Floor data sorting and processing
- Performance measurement
- Error handling
- Statistical calculations
- UI display patterns

Core function:
```javascript
async function getBuildingsViaVenues() {
    try {
        const buildings = await mapsindoors.services.VenuesService.getBuildings();
        const processedBuildings = [];
        
        if (buildings && buildings.length > 0) {
            buildings.forEach(building => {
                const floors = building.floors ? 
                    Object.keys(building.floors).sort((a, b) => parseInt(a) - parseInt(b)) : [];
                
                processedBuildings.push({
                    id: building.id,
                    name: building.name || building.buildingInfo?.name || `Building ${building.id}`,
                    floors: floors,
                    floorCount: floors.length,
                    address: building.address || building.buildingInfo?.address,
                    geometry: building.geometry,
                    bbox: building.bbox,
                    anchor: building.anchor,
                    venue: building.venue || building.venueId,
                    floorData: building.floors
                });
            });
        }
        
        return processedBuildings;
    } catch (error) {
        console.error("Error fetching buildings via VenuesService:", error);
        throw error;
    }
}
```

Perfect for: Backend processing, building inventories, facility management, data analysis applications that need building metadata without map visualization.




### AI-Powered Airport Navigation Chatbot with Natural Language Search

Complete implementation of an intelligent airport navigation system that combines Gemini AI with MapsIndoors for natural language location search, multi-strategy filtering, and conversational routing instructions.

Key features:
- Natural language query processing using Gemini AI
- Multi-strategy location search (gate matching, type filtering, proximity search)
- Real-time distance matrix calculations for travel times
- Conversational route instructions with turn-by-turn guidance
- Gate-specific regex matching for airport terminals
- Error handling with fallback mechanisms

Core implementation:
```javascript
// 1. AI-Enhanced Query Processing
async function parseInstructionsFromGemini(query) {
  const prompt = `Analyze this airport query: "${query}"
  Return JSON with searchQuery, queryType, floor, etc.`;
  
  const response = await fetch(GEMINI_API_ENDPOINT, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      contents: [{ parts: [{ text: prompt }] }],
      generationConfig: { temperature: 0.2, maxOutputTokens: 1024 }
    })
  });
  
  const data = await response.json();
  return JSON.parse(data.candidates[0].content.parts[0].text);
}

// 2. Multi-Strategy Location Search
async function searchLocations(query, options = {}) {
  const instructions = await parseInstructionsFromGemini(query);
  
  const baseSearchParams = {
    venue: '76949746a5873f0090fd8d8e839d8726',
    take: options.take || 100,
    near: { lat: USER_POSITION.lat, lng: USER_POSITION.lng },
    radius: options.radius || 500,
    floor: USER_POSITION.floor
  };
  
  // Strategy 1: Gate-specific regex matching
  const gateMatch = query.match(/(?:gate\s*)?([A-Z]\d+)/i);
  if (gateMatch) {
    const gateNumber = gateMatch[1].toUpperCase();
    const gateSearchParams = { ...baseSearchParams };
    gateSearchParams.q = gateNumber;
    locations = await mapsindoors.services.LocationsService.getLocations(gateSearchParams);
  }
  
  // Strategy 2: Type-based search for categories
  else if (query.toLowerCase().includes('restaurant')) {
    const restaurantSearchParams = { ...baseSearchParams };
    restaurantSearchParams.types = ['Restaurants'];
    locations = await mapsindoors.services.LocationsService.getLocations(restaurantSearchParams);
  }
  
  // Strategy 3: General text search with proximity
  else {
    const searchParams = { ...baseSearchParams };
    searchParams.q = instructions.searchQuery;
    locations = await mapsindoors.services.LocationsService.getLocations(searchParams);
  }
  
  // Calculate travel times using distance matrix
  const origins = [`${USER_POSITION.lat},${USER_POSITION.lng},${USER_POSITION.floor}`];
  const destinations = locations.map(loc => {
    const coords = loc.properties.anchor.coordinates;
    return `${coords[1]},${coords[0]},${loc.properties.floor}`;
  });
  
  const matrix = await mapsindoors.services.DistanceMatrixService.getDistanceMatrix({
    graphId: "airport_graph",
    origins: origins,
    destinations: destinations,
    travelMode: "walking"
  });
  
  return locations.sort((a, b) => (a.distance || 0) - (b.distance || 0));
}

// 3. Conversational Route Display
async function showRouteToLocation(location) {
  const coords = location.properties.anchor.coordinates;
  
  const route = await miDirectionsService.getRoute({
    origin: { lat: USER_POSITION.lat, lng: USER_POSITION.lng, floor: USER_POSITION.floor },
    destination: { lat: coords[1], lng: coords[0], floor: location.properties.floor },
    graphId: "airport_graph",
    travelMode: "walking"
  });
  
  // Display conversational route instructions
  const distance = formatDistance(route.distance.value);
  const time = formatDuration(route.duration.value);
  
  addAIMessage(`Route to ${location.properties.name}: ${distance} in ${time}\n${formatRouteInstructions(route)}`);
  
  miDirectionsRenderer.setRoute(route);
  
  // Highlight destination
  mapsIndoorsInstance.setDisplayRule(location.id, {
    iconVisible: true,
    polygonVisible: true,
    polygonFillColor: "#4ade80",
    polygonFillOpacity: 0.4
  });
}
```

Perfect for: Airport navigation systems, large venue wayfinding, AI-powered location search, conversational interfaces, transportation hubs, and any application requiring natural language interaction with indoor mapping.

Technologies: MapsIndoors SDK 4.40.0, Mapbox GL JS 3.8.0, Google Gemini 1.5 Flash API, JavaScript ES6+ (async/await)



---

## Use Case: floor management Implementation

### Problem
You need to implement custom floor switching functionality in your MapsIndoors application.

### Code Example
```js
mapsIndoorsInstance.setFloor(2);
mapView.getMap().flyTo({center: [lng, lat], zoom: 20, duration: 1000});
```

### Explanation
- Demonstrates MapsIndoors implementation patterns
- Provides reusable code for common use cases

### Gotchas / Notes
🛠️ Test thoroughly in your specific MapsIndoors environment
🛠️ Verify API compatibility with your MapsIndoors SDK version

