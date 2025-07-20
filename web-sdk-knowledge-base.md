

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Always use `MapboxV3View` instead of `MapboxView` for SDK v4.41.1
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ The floor selector won't appear until a building is loaded

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Always set floor before flying to location
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ MapsIndoors uses [lng, lat] while some APIs use [lat, lng]

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Setting display rule on `null` affects all locations not explicitly styled
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ The `take` parameter limits results (default is 10)

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Click event only provides `id`, not full location data
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Always check if `event` and `event.id` exist

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Search is case-insensitive but requires minimum 2 characters
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ `orderBy: 'relevance'` gives better results than alphabetical

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Distance Matrix requires the venue's graph ID
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Matrix coordinate format is different from location format

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Floor indices can be negative (basements)
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Floor change event fires before tiles load

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
    el.innerHTML = 'ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ';
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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Custom markers don't respect floor visibility automatically
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Must manually show/hide markers on floor change

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Don't call `getBuilding()` or `getVenue()` before ready
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Display rules can be set before ready but won't apply

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Maximum 25 waypoints per route
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Floor transitions require accessible paths in venue data

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Label collision detection works within zoom levels
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Labels follow language settings from SolutionConfig

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ `nextLeg()` and `previousLeg()` don't return values
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Floor changes happen at leg boundaries

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Enable `preserveDrawingBuffer: true` in Mapbox options
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Some browsers limit maximum canvas size

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Venue ID is different from venue name
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Floor filter applies across all buildings

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ GPS positions may need indoor mapping corrections
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Frequent updates can impact performance

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
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Distance Matrix has a limit on origin/destination combinations
ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ°ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¯ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ¸ÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂÃÂ Consider caching Distance Matrix results for performance

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

---

---

---

---

## Use Case: Location Search Implementation

### Context
MapsIndoors implementation discussed in Claude Desktop conversation



### Problem
You need to set up a MapsIndoors map with proper initialization and configuration.

### Code Example
```js
// MapsIndoors Display Rules - Hide Neighborhood Polygons and Extrusions
// This code demonstrates how to disable visual elements for specific location types

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
- Initializes MapsIndoors with proper map view configuration
- Implements asynchronous operations for better performance
- Includes proper error handling and recovery

### Gotchas / Notes
🛠️ Ensure proper API key configuration before initialization
🛠️ Map container element must exist in DOM before creating map view
🛠️ Always check if DOM elements exist before using them


## Use Case: Floor Management System

### Context
General MapsIndoors implementation - context shared through Claude Desktop conversation



### Problem
You need to set up a MapsIndoors map with proper initialization and configuration.

### Code Example
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MapsIndoors - Neighborhood Display Rules</title>
    <!-- Mapbox CSS -->
    <link href="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.css" rel="stylesheet">
    <!-- Mapbox JS -->
    <script src="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.js"></script>
    <!-- MapsIndoors JS -->
    <script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz?apikey=b3e7c729df7546b390103562"></script>
    
    <style>
        body {
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
        }
        
        #map {
            width: 100vw;
            height: 100vh;
        }
    </style>
</head>
<body>
    <div id="map"></div>

    <script>
        document.addEventListener('DOMContentLoaded', async function() {
            try {
                // Initialize map with MapboxV3View
                const mapViewOptions = {
                    accessToken: 'pk.eyJ1IjoiZ2V3YS1tYXBzcGVvcGxlIiwiYSI6ImNsZzJudDB4ZTAwcnEzZnAwb2VvbTYwYnIifQ.w-cnsU-xP9jaly_qrgy_iA',
                    element: document.getElementById('map'),
                    center: { lat: 40.75391251837155, lng: -73.97496487462143 },
                    zoom: 15,
                    maxZoom: 22,
                };

                // Create MapboxV3View instance
                const mapViewInstance = new mapsindoors.mapView.MapboxV3View(mapViewOptions);
                
                // Initialize MapsIndoors
                const mapsIndoorsInstance = new mapsindoors.MapsIndoors({
                    mapView: mapViewInstance,
                });
                
                // Get Mapbox instance
                const mapboxInstance = mapViewInstance.getMap();

                // Add floor selector
                const floorSelectorElement = document.createElement('div');
                new mapsindoors.FloorSelector(floorSelectorElement, mapsIndoorsInstance);
                mapboxInstance.addControl({ 
                    onAdd: function () { return floorSelectorElement },
                    onRemove: function () { }
                });

                // Wait for MapsIndoors to be ready
                mapsIndoorsInstance.addListener('ready', async () => {
                    console.log('MapsIndoors is ready');
                    
                    try {
                        // Map is now centered on the specified coordinates
                        console.log('Map centered on:', 40.75391251837155, -73.97496487462143);
                        
                        // Wait a bit more to ensure everything is fully loaded
                        setTimeout(async () => {
                            try {
                                // Get all Neighborhood type locations
                                const neighborhoods = await mapsindoors.services.LocationsService.getLocations({
                                    types: ['Neighborhood'],
                                    take: 1000 // Get up to 1000 neighborhoods
                                });
                                
                                console.log(`Found ${neighborhoods.length} neighborhoods`);
                                
                                if (neighborhoods.length > 0) {
                                    // Extract neighborhood IDs
                                    const neighborhoodIds = neighborhoods.map(neighborhood => neighborhood.id);
                                    
                                    // Apply display rule to disable polygon fill, stroke, and extrusions
                                    mapsIndoorsInstance.setDisplayRule(neighborhoodIds, {
                                        visible: true, // Keep the location visible
                                        polygonVisible: false, // Completely hide polygons
                                        polygonFillOpacity: 0, // Disable fill
                                        polygonStrokeOpacity: 0, // Disable stroke
                                        polygonStrokeWidth: 0, // Set stroke width to 0
                                        extrusionVisible: false, // Disable extrusions
                                        extrusionOpacity: 0, // Set extrusion opacity to 0
                                        extrusionHeight: 0, // Set extrusion height to 0
                                        iconVisible: true, // Keep icons if any
                                        labelVisible: true, // Keep labels if any
                                        zoomFrom: 1, // Show from zoom level 1
                                        zoomTo: 25 // Show up to max zoom
                                    });
                                    
                                    console.log(`Applied display rules to ${neighborhoodIds.length} neighborhoods - disabled polygons, strokes, and extrusions`);
                                } else {
                                    console.log('No neighborhoods found in this solution');
                                }
                                
                            } catch (error) {
                                console.error('Error applying neighborhood display rules:', error);
                            }
                        }, 2000); // Wait 2 seconds after ready event
                        
                    } catch (error) {
                        console.error('Error setting up neighborhoods display rules:', error);
                    }
                });

                // Handle MapsIndoors errors
                mapsIndoorsInstance.addListener('error', (error) => {
                    console.error('MapsIndoors error:', error);
                });

            } catch (error) {
                console.error('Error initializing MapsIndoors:', error);
            }
        });
    </script>
</body>
</html>
```

### Explanation
- Initializes MapsIndoors with proper map view configuration
- Uses event listeners for interactive functionality
- Implements asynchronous operations for better performance
- Includes proper error handling and recovery

### Gotchas / Notes
ð ï¸ Ensure proper API key configuration before initialization
ð ï¸ Map container element must exist in DOM before creating map view
ð ï¸ Always check if DOM elements exist before using them


## Use Case: Floor Management System

### Context
General MapsIndoors implementation - context shared through Claude Desktop conversation



### Problem
You need to implement location search functionality for your indoor map.

### Code Example
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Austin Office Amenity Finder</title>
    
    <!-- Mapbox CSS (load first) -->
    <link href="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.css" rel="stylesheet">
    
    <!-- Font Awesome for icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.2/css/all.min.css">
    
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --primary-color: #0066cc;
            --primary-dark: #004499;
            --secondary-color: #28a745;
            --warning-color: #ffc107;
            --danger-color: #dc3545;
            --info-color: #17a2b8;
            --light-gray: #f8f9fa;
            --medium-gray: #e9ecef;
            --dark-gray: #6c757d;
            --border-radius: 8px;
            --box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--light-gray);
            overflow: hidden;
        }

        .container {
            display: flex;
            height: 100vh;
        }

        /* Sidebar Styles */
        .sidebar {
            width: 380px;
            background-color: white;
            box-shadow: var(--box-shadow);
            display: flex;
            flex-direction: column;
            z-index: 10;
        }

        .sidebar-header {
            background: linear-gradient(135deg, var(--primary-color), var(--primary-dark));
            color: white;
            padding: 20px;
        }

        .sidebar-title {
            font-size: 1.5rem;
            font-weight: 600;
            margin-bottom: 5px;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .sidebar-subtitle {
            font-size: 0.9rem;
            opacity: 0.9;
        }

        /* Search Section */
        .search-section {
            padding: 20px;
            border-bottom: 1px solid var(--medium-gray);
        }

        .search-container {
            position: relative;
            margin-bottom: 15px;
        }

        .search-input {
            width: 100%;
            padding: 12px 15px 12px 45px;
            border: 2px solid var(--medium-gray);
            border-radius: var(--border-radius);
            font-size: 1rem;
            transition: border-color 0.3s ease;
        }

        .search-input:focus {
            outline: none;
            border-color: var(--primary-color);
        }

        .search-icon {
            position: absolute;
            left: 15px;
            top: 50%;
            transform: translateY(-50%);
            color: var(--dark-gray);
            font-size: 1.1rem;
        }

        .clear-search {
            position: absolute;
            right: 15px;
            top: 50%;
            transform: translateY(-50%);
            background: none;
            border: none;
            color: var(--dark-gray);
            cursor: pointer;
            display: none;
        }

        /* Filter Section */
        .filter-section {
            padding: 20px;
            border-bottom: 1px solid var(--medium-gray);
        }

        .filter-title {
            font-weight: 600;
            margin-bottom: 15px;
            color: #333;
        }

        .filter-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
        }

        .filter-btn {
            padding: 10px 12px;
            border: 2px solid var(--medium-gray);
            border-radius: var(--border-radius);
            background: white;
            cursor: pointer;
            transition: all 0.3s ease;
            font-size: 0.9rem;
            font-weight: 500;
            display: flex;
            align-items: center;
            gap: 8px;
            text-align: left;
        }

        .filter-btn:hover {
            border-color: var(--primary-color);
            background-color: #f0f8ff;
        }

        .filter-btn.active {
            border-color: var(--primary-color);
            background-color: var(--primary-color);
            color: white;
        }

        .filter-btn.show-all {
            grid-column: 1 / -1;
            background: linear-gradient(135deg, var(--secondary-color), #20a742);
            color: white;
            border-color: var(--secondary-color);
        }

        .filter-btn.show-all:hover {
            background: linear-gradient(135deg, #20a742, #1e7e34);
        }

        .filter-btn.show-all.active {
            background: linear-gradient(135deg, #1e7e34, #155724);
        }

        /* Results Section */
        .results-section {
            flex: 1;
            overflow-y: auto;
            padding: 20px;
        }

        .results-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }

        .results-title {
            font-weight: 600;
            color: #333;
        }

        .results-count {
            background-color: var(--primary-color);
            color: white;
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 0.85rem;
            font-weight: 500;
        }

        .location-card {
            background: white;
            border-radius: var(--border-radius);
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            padding: 15px;
            margin-bottom: 12px;
            cursor: pointer;
            transition: all 0.3s ease;
            border-left: 4px solid transparent;
        }

        .location-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        }

        .location-card.selected {
            border-left-color: var(--primary-color);
            box-shadow: 0 4px 12px rgba(0, 102, 204, 0.2);
        }

        .location-header {
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            margin-bottom: 8px;
        }

        .location-name {
            font-weight: 600;
            color: #333;
            font-size: 1rem;
        }

        .location-type {
            background-color: var(--light-gray);
            color: var(--dark-gray);
            padding: 4px 8px;
            border-radius: 12px;
            font-size: 0.75rem;
            font-weight: 500;
        }

        .location-details {
            display: flex;
            align-items: center;
            gap: 15px;
            font-size: 0.9rem;
            color: var(--dark-gray);
        }

        .location-detail {
            display: flex;
            align-items: center;
            gap: 5px;
        }

        /* Map Container */
        .map-container {
            flex: 1;
            position: relative;
        }

        #map {
            width: 100%;
            height: 100%;
        }

        /* Map Controls */
        .map-controls {
            position: absolute;
            top: 20px;
            right: 20px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            z-index: 100;
        }

        .map-control-btn {
            width: 40px;
            height: 40px;
            background: white;
            border: none;
            border-radius: var(--border-radius);
            box-shadow: var(--box-shadow);
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .map-control-btn:hover {
            background-color: var(--light-gray);
            transform: scale(1.05);
        }

        /* Info Panel */
        .info-panel {
            position: absolute;
            bottom: 20px;
            right: 20px;
            background: white;
            border-radius: var(--border-radius);
            box-shadow: var(--box-shadow);
            padding: 20px;
            width: 300px;
            z-index: 100;
            transform: translateY(20px);
            opacity: 0;
            pointer-events: none;
            transition: all 0.3s ease;
        }

        .info-panel.visible {
            transform: translateY(0);
            opacity: 1;
            pointer-events: auto;
        }

        .info-panel-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }

        .info-panel-title {
            font-weight: 600;
            font-size: 1.1rem;
            color: #333;
        }

        .info-panel-close {
            background: none;
            border: none;
            font-size: 1.2rem;
            color: var(--dark-gray);
            cursor: pointer;
        }

        .info-panel-content {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .info-detail {
            display: flex;
            align-items: center;
            gap: 10px;
            font-size: 0.9rem;
        }

        .info-detail i {
            color: var(--primary-color);
            width: 16px;
        }

        /* Status Messages */
        .status-message {
            text-align: center;
            padding: 40px 20px;
            color: var(--dark-gray);
        }

        .status-message i {
            font-size: 3rem;
            margin-bottom: 15px;
            color: var(--medium-gray);
        }

        .status-message h3 {
            margin-bottom: 10px;
            color: #333;
        }

        /* Loading Animation */
        .loading {
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 40px;
        }

        .spinner {
            width: 40px;
            height: 40px;
            border: 4px solid var(--medium-gray);
            border-top: 4px solid var(--primary-color);
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        /* Responsive Design */
        @media (max-width: 768px) {
            .container {
                flex-direction: column;
            }

            .sidebar {
                width: 100%;
                height: 50vh;
            }

            .map-container {
                height: 50vh;
            }

            .info-panel {
                width: calc(100% - 40px);
                left: 20px;
                right: 20px;
            }

            .filter-grid {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Sidebar -->
        <div class="sidebar">
            <div class="sidebar-header">
                <div class="sidebar-title">
                    <i class="fas fa-map-marker-alt"></i>
                    Amenity Finder
                </div>
                <div class="sidebar-subtitle">Find restrooms, meeting rooms, and facilities in the Austin Office</div>
            </div>

            <!-- Search Section -->
            <div class="search-section">
                <div class="search-container">
                    <i class="fas fa-search search-icon"></i>
                    <input type="text" class="search-input" placeholder="Search locations..." id="searchInput">
                    <button class="clear-search" id="clearSearch">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
            </div>

            <!-- Filter Section -->
            <div class="filter-section">
                <div class="filter-title">Filter by Amenity Type</div>
                <div class="filter-grid" id="filterGrid">
                    <button class="filter-btn show-all active" data-type="all">
                        <i class="fas fa-th-large"></i>
                        Show All
                    </button>
                    <!-- Dynamic filter buttons will be added here -->
                </div>
            </div>

            <!-- Results Section -->
            <div class="results-section">
                <div class="results-header">
                    <div class="results-title">Locations</div>
                    <div class="results-count" id="resultsCount">0</div>
                </div>
                <div id="locationsList">
                    <div class="loading">
                        <div class="spinner"></div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Map Container -->
        <div class="map-container">
            <div id="map"></div>

            <!-- Map Controls -->
            <div class="map-controls">
                <button class="map-control-btn" id="zoomIn" title="Zoom In">
                    <i class="fas fa-plus"></i>
                </button>
                <button class="map-control-btn" id="zoomOut" title="Zoom Out">
                    <i class="fas fa-minus"></i>
                </button>
                <button class="map-control-btn" id="recenter" title="Recenter Map">
                    <i class="fas fa-crosshairs"></i>
                </button>
                <button class="map-control-btn" id="clearHighlights" title="Clear Highlights">
                    <i class="fas fa-eye-slash"></i>
                </button>
            </div>

            <!-- Info Panel -->
            <div class="info-panel" id="infoPanel">
                <div class="info-panel-header">
                    <div class="info-panel-title" id="infoPanelTitle">Location Details</div>
                    <button class="info-panel-close" id="infoPanelClose">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
                <div class="info-panel-content" id="infoPanelContent">
                    <!-- Dynamic content will be added here -->
                </div>
            </div>
        </div>
    </div>

    <!-- Scripts (load in correct order) -->
    <script src="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.js"></script>
    <script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz?apikey=mapspeople"></script>

    <script>
        class AmenityFinder {
            constructor() {
                this.mapsIndoorsInstance = null;
                this.mapboxInstance = null;
                this.allLocations = [];
                this.filteredLocations = [];
                this.selectedLocationId = null;
                this.currentFilter = 'all';
                this.searchTerm = '';
                this.locationTypes = new Map();
                
                // Type icons mapping
                this.typeIcons = {
                    'Restroom': 'fas fa-restroom',
                    'MeetingRoom': 'fas fa-users',
                    'MeetingRoom Small': 'fas fa-user-friends',
                    'MeetingRoom Medium': 'fas fa-users',
                    'MeetingRoom Large': 'fas fa-users-cog',
                    'MeetingRoom Extra Small': 'fas fa-user',
                    'Canteen': 'fas fa-utensils',
                    'Kitchen': 'fas fa-utensils',
                    'Elevator': 'fas fa-arrows-alt-v',
                    'Stairs': 'fas fa-walking',
                    'Entrance': 'fas fa-door-open',
                    'Exit': 'fas fa-sign-out-alt',
                    'Phone Booth': 'fas fa-phone',
                    'Default': 'fas fa-map-marker-alt'
                };

                this.init();
            }

            async init() {
                try {
                    await this.initializeMap();
                    this.setupEventListeners();
                } catch (error) {
                    console.error('Error initializing AmenityFinder:', error);
                    this.showError('Failed to initialize the application. Please refresh the page.');
                }
            }

            async initializeMap() {
                // Initialize Mapbox and MapsIndoors
                const mapViewOptions = {
                    accessToken: 'pk.eyJ1IjoiZ2V3YS1tYXBzcGVvcGxlIiwiYSI6ImNsZzJudDB4ZTAwcnEzZnAwb2VvbTYwYnIifQ.w-cnsU-xP9jaly_qrgy_iA',
                    element: document.getElementById('map'),
                    center: { lat: 30.3603212, lng: -97.7422623 }, // Austin Office
                    zoom: 20,
                    maxZoom: 22,
                };

                const mapViewInstance = new mapsindoors.mapView.MapboxV3View(mapViewOptions);
                this.mapsIndoorsInstance = new mapsindoors.MapsIndoors({
                    mapView: mapViewInstance,
                });
                this.mapboxInstance = mapViewInstance.getMap();

                // Add floor selector
                const floorSelectorElement = document.createElement('div');
                new mapsindoors.FloorSelector(floorSelectorElement, this.mapsIndoorsInstance);
                this.mapboxInstance.addControl({ 
                    onAdd: function () { return floorSelectorElement },
                    onRemove: function () { }
                });

                // Wait for MapsIndoors to be ready
                return new Promise((resolve, reject) => {
                    this.mapsIndoorsInstance.addListener('ready', async () => {
                        console.log('MapsIndoors is ready');
                        try {
                            await this.loadLocations();
                            this.setupMapEventListeners();
                            resolve();
                        } catch (error) {
                            reject(error);
                        }
                    });

                    this.mapsIndoorsInstance.addListener('error', (error) => {
                        console.error('MapsIndoors error:', error);
                        reject(error);
                    });
                });
            }

            async loadLocations() {
                try {
                    console.log('Loading locations...');
                    
                    // Get all locations from Austin Office
                    const locations = await mapsindoors.services.LocationsService.getLocations({
                        venue: 'AUSTINOFFICE',
                        take: 500 // Increase limit to get more locations
                    });

                    console.log(`Loaded ${locations.length} locations`);
                    
                    // Process and filter relevant locations
                    this.allLocations = locations.filter(location => {
                        // Only include locations with meaningful types and names
                        const hasType = location.properties.type && location.properties.type.trim() !== '';
                        const hasName = location.properties.name && location.properties.name.trim() !== '';
                        const hasGeometry = location.geometry && 
                            (location.geometry.type === 'Polygon' || 
                             location.geometry.type === 'Point' || 
                             location.geometry.type === 'MultiPolygon');
                        
                        return hasType && hasName && hasGeometry;
                    }).map(location => ({
                        id: location.id,
                        name: location.properties.name,
                        type: location.properties.type,
                        floor: location.properties.floor || 0,
                        building: location.properties.building,
                        venue: location.properties.venue,
                        coordinates: location.properties.anchor.coordinates,
                        geometry: location.geometry,
                        description: location.properties.description || ''
                    }));

                    console.log(`Processed ${this.allLocations.length} relevant locations`);

                    // Build location types map
                    this.buildLocationTypesMap();
                    
                    // Create filter buttons
                    this.createFilterButtons();
                    
                    // Initially show all locations
                    this.applyFilters();
                    
                } catch (error) {
                    console.error('Error loading locations:', error);
                    this.showError('Failed to load location data.');
                }
            }

            buildLocationTypesMap() {
                // Count locations by type
                const typeCounts = {};
                this.allLocations.forEach(location => {
                    const type = location.type;
                    typeCounts[type] = (typeCounts[type] || 0) + 1;
                });

                // Convert to Map and sort by count (descending)
                this.locationTypes = new Map(
                    Object.entries(typeCounts)
                        .sort((a, b) => b[1] - a[1]) // Sort by count descending
                        .filter(([type, count]) => count > 0) // Only include types with locations
                );

                console.log('Location types:', this.locationTypes);
            }

            createFilterButtons() {
                const filterGrid = document.getElementById('filterGrid');
                
                // Clear existing buttons except "Show All"
                const showAllBtn = filterGrid.querySelector('.show-all');
                filterGrid.innerHTML = '';
                filterGrid.appendChild(showAllBtn);

                // Add filter buttons for each location type
                this.locationTypes.forEach((count, type) => {
                    const button = document.createElement('button');
                    button.className = 'filter-btn';
                    button.dataset.type = type;
                    
                    const icon = this.typeIcons[type] || this.typeIcons['Default'];
                    button.innerHTML = `
                        <i class="${icon}"></i>
                        <span>${type} (${count})</span>
                    `;
                    
                    filterGrid.appendChild(button);
                });
            }

            setupEventListeners() {
                // Search input
                const searchInput = document.getElementById('searchInput');
                const clearSearch = document.getElementById('clearSearch');
                
                searchInput.addEventListener('input', (e) => {
                    this.searchTerm = e.target.value;
                    this.applyFilters();
                    
                    // Show/hide clear button
                    clearSearch.style.display = this.searchTerm ? 'block' : 'none';
                });

                clearSearch.addEventListener('click', () => {
                    searchInput.value = '';
                    this.searchTerm = '';
                    clearSearch.style.display = 'none';
                    this.applyFilters();
                });

                // Filter buttons
                document.getElementById('filterGrid').addEventListener('click', (e) => {
                    const filterBtn = e.target.closest('.filter-btn');
                    if (!filterBtn) return;

                    // Update active state
                    document.querySelectorAll('.filter-btn').forEach(btn => btn.classList.remove('active'));
                    filterBtn.classList.add('active');

                    // Update current filter
                    this.currentFilter = filterBtn.dataset.type;
                    this.applyFilters();
                });

                // Map controls
                document.getElementById('zoomIn').addEventListener('click', () => {
                    this.mapboxInstance.zoomIn();
                });

                document.getElementById('zoomOut').addEventListener('click', () => {
                    this.mapboxInstance.zoomOut();
                });

                document.getElementById('recenter').addEventListener('click', () => {
                    this.recenterMap();
                });

                document.getElementById('clearHighlights').addEventListener('click', () => {
                    this.clearHighlights();
                });

                // Info panel close
                document.getElementById('infoPanelClose').addEventListener('click', () => {
                    this.hideInfoPanel();
                });
            }

            setupMapEventListeners() {
                // Location click event
                this.mapsIndoorsInstance.addListener('click', (event) => {
                    if (event && event.id) {
                        const location = this.allLocations.find(loc => loc.id === event.id);
                        if (location) {
                            this.selectLocation(location.id);
                        }
                    }
                });
            }

            applyFilters() {
                // Filter locations based on current filter and search term
                this.filteredLocations = this.allLocations.filter(location => {
                    // Filter by type
                    const matchesType = this.currentFilter === 'all' || location.type === this.currentFilter;
                    
                    // Filter by search term
                    const matchesSearch = !this.searchTerm || 
                        location.name.toLowerCase().includes(this.searchTerm.toLowerCase()) ||
                        location.type.toLowerCase().includes(this.searchTerm.toLowerCase());
                    
                    return matchesType && matchesSearch;
                });

                console.log(`Filtered to ${this.filteredLocations.length} locations`);

                // Update display
                this.updateLocationsList();
                this.updateMapDisplay();
            }

            updateLocationsList() {
                const locationsList = document.getElementById('locationsList');
                const resultsCount = document.getElementById('resultsCount');
                
                // Update count
                resultsCount.textContent = this.filteredLocations.length;

                // Clear current list
                locationsList.innerHTML = '';

                if (this.filteredLocations.length === 0) {
                    locationsList.innerHTML = `
                        <div class="status-message">
                            <i class="fas fa-search"></i>
                            <h3>No locations found</h3>
                            <p>Try adjusting your search or filter criteria.</p>
                        </div>
                    `;
                    return;
                }

                // Group locations by floor for better organization
                const locationsByFloor = this.groupLocationsByFloor(this.filteredLocations);

                // Create location cards grouped by floor
                Object.entries(locationsByFloor)
                    .sort((a, b) => parseInt(a[0]) - parseInt(b[0])) // Sort floors numerically
                    .forEach(([floor, locations]) => {
                        // Add floor header if there are multiple floors
                        if (Object.keys(locationsByFloor).length > 1) {
                            const floorHeader = document.createElement('div');
                            floorHeader.style.cssText = `
                                font-weight: 600;
                                color: var(--dark-gray);
                                padding: 10px 0 5px 0;
                                font-size: 0.9rem;
                                border-bottom: 1px solid var(--medium-gray);
                                margin-bottom: 10px;
                            `;
                            floorHeader.textContent = `Floor ${floor}`;
                            locationsList.appendChild(floorHeader);
                        }

                        // Add location cards for this floor
                        locations.forEach(location => {
                            const locationCard = this.createLocationCard(location);
                            locationsList.appendChild(locationCard);
                        });
                    });
            }

            groupLocationsByFloor(locations) {
                return locations.reduce((groups, location) => {
                    const floor = location.floor || 0;
                    if (!groups[floor]) {
                        groups[floor] = [];
                    }
                    groups[floor].push(location);
                    return groups;
                }, {});
            }

            createLocationCard(location) {
                const card = document.createElement('div');
                card.className = `location-card ${this.selectedLocationId === location.id ? 'selected' : ''}`;
                card.dataset.locationId = location.id;

                const icon = this.typeIcons[location.type] || this.typeIcons['Default'];

                card.innerHTML = `
                    <div class="location-header">
                        <div class="location-name">${location.name}</div>
                        <div class="location-type">${location.type}</div>
                    </div>
                    <div class="location-details">
                        <div class="location-detail">
                            <i class="${icon}"></i>
                            <span>${location.type}</span>
                        </div>
                        <div class="location-detail">
                            <i class="fas fa-layer-group"></i>
                            <span>Floor ${location.floor}</span>
                        </div>
                    </div>
                `;

                card.addEventListener('click', () => {
                    this.selectLocation(location.id);
                });

                return card;
            }

            selectLocation(locationId) {
                this.selectedLocationId = locationId;
                const location = this.allLocations.find(loc => loc.id === locationId);
                
                if (!location) return;

                console.log('Selected location:', location);

                // Update card selection state
                document.querySelectorAll('.location-card').forEach(card => {
                    if (card.dataset.locationId === locationId) {
                        card.classList.add('selected');
                        card.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
                    } else {
                        card.classList.remove('selected');
                    }
                });

                // Highlight location on map
                this.highlightLocation(location);

                // Show info panel
                this.showInfoPanel(location);

                // Set floor and center on location
                this.mapsIndoorsInstance.setFloor(location.floor);
                this.mapboxInstance.flyTo({
                    center: location.coordinates,
                    zoom: 21,
                    duration: 1000
                });
            }

            highlightLocation(location) {
                // When selecting individual location, clear group highlights first
                this.mapsIndoorsInstance.highlight([]);
                
                // Then highlight just the selected location
                this.mapsIndoorsInstance.highlight([location.id]);
                
                console.log(`Highlighting individual location: ${location.id} (${location.name})`);
            }

            showInfoPanel(location) {
                const infoPanel = document.getElementById('infoPanel');
                const title = document.getElementById('infoPanelTitle');
                const content = document.getElementById('infoPanelContent');

                title.textContent = location.name;

                const icon = this.typeIcons[location.type] || this.typeIcons['Default'];

                content.innerHTML = `
                    <div class="info-detail">
                        <i class="${icon}"></i>
                        <span>${location.type}</span>
                    </div>
                    <div class="info-detail">
                        <i class="fas fa-layer-group"></i>
                        <span>Floor ${location.floor}</span>
                    </div>
                    <div class="info-detail">
                        <i class="fas fa-building"></i>
                        <span>${location.building || 'Austin Office'}</span>
                    </div>
                    ${location.description ? `
                        <div class="info-detail">
                            <i class="fas fa-info-circle"></i>
                            <span>${location.description}</span>
                        </div>
                    ` : ''}
                `;

                infoPanel.classList.add('visible');
            }

            hideInfoPanel() {
                document.getElementById('infoPanel').classList.remove('visible');
                this.selectedLocationId = null;
                
                // Update card selection state
                document.querySelectorAll('.location-card').forEach(card => {
                    card.classList.remove('selected');
                });

                // Clear highlights
                this.clearHighlights();
            }

            updateMapDisplay() {
                // Get all location IDs currently loaded
                const allLocationIds = this.allLocations.map(loc => loc.id);
                
                // Hide all locations first
                this.mapsIndoorsInstance.setDisplayRule(allLocationIds, { visible: false });

                // Show and highlight only filtered locations
                if (this.filteredLocations.length > 0) {
                    const filteredLocationIds = this.filteredLocations.map(loc => loc.id);
                    
                    // Make filtered locations visible
                    this.mapsIndoorsInstance.setDisplayRule(filteredLocationIds, { 
                        visible: true,
                        iconVisible: true,
                        labelVisible: true,
                        polygonVisible: true
                    });

                    // Highlight the filtered locations (this works with arrays of location IDs)
                    this.mapsIndoorsInstance.highlight(filteredLocationIds);
                    
                    console.log(`Highlighting ${filteredLocationIds.length} locations:`, filteredLocationIds);
                } else {
                    // Clear highlights when no locations match
                    this.mapsIndoorsInstance.highlight([]);
                }
            }

            clearHighlights() {
                this.mapsIndoorsInstance.highlight([]);
                this.hideInfoPanel();
                console.log('All highlights cleared');
            }

            recenterMap() {
                if (this.filteredLocations.length === 0) {
                    // Default center for Austin Office
                    this.mapboxInstance.flyTo({
                        center: [-97.7422623, 30.3603212],
                        zoom: 20,
                        duration: 1000
                    });
                    return;
                }

                // Calculate bounds for filtered locations
                const bounds = new mapboxgl.LngLatBounds();
                this.filteredLocations.forEach(location => {
                    bounds.extend(location.coordinates);
                });

                // Fit map to bounds
                this.mapboxInstance.fitBounds(bounds, {
                    padding: { top: 50, bottom: 50, left: 430, right: 50 }, // Account for sidebar
                    duration: 1000
                });
            }

            showError(message) {
                document.getElementById('locationsList').innerHTML = `
                    <div class="status-message">
                        <i class="fas fa-exclamation-triangle"></i>
                        <h3>Error</h3>
                        <p>${message}</p>
                    </div>
                `;
            }
        }

        // Initialize the application when the page loads
        document.addEventListener('DOMContentLoaded', () => {
            new AmenityFinder();
        });
    </script>
</body>
</html>
```

### Explanation
- Initializes MapsIndoors with proper map view configuration
- Uses event listeners for interactive functionality
- Implements asynchronous operations for better performance
- Includes proper error handling and recovery

### Gotchas / Notes
Ã°ÂÂÂ Ã¯Â¸Â Ensure proper API key configuration before initialization
Ã°ÂÂÂ Ã¯Â¸Â Map container element must exist in DOM before creating map view


## Use Case: MapsIndoors Custom Implementation

### Context
General MapsIndoors implementation - context shared through Claude Desktop conversation



### Problem
You need to implement custom functionality in your MapsIndoors application.

### Code Example
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MapsIndoors Buildings - No Map</title>
    <!-- MapsIndoors JavaScript -->
    <script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz?apikey=5d0f9d73dc5d4ca299a1eee3"></script>
    
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }

        .container {
            background: white;
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        h1 {
            color: #333;
            text-align: center;
            margin-bottom: 30px;
        }

        .loading {
            text-align: center;
            padding: 20px;
            color: #666;
        }

        .error {
            background-color: #fee;
            border: 1px solid #fcc;
            border-radius: 4px;
            padding: 15px;
            margin: 10px 0;
            color: #c33;
        }

        .success {
            background-color: #efe;
            border: 1px solid #cfc;
            border-radius: 4px;
            padding: 15px;
            margin: 10px 0;
            color: #3c3;
        }

        .building-card {
            background-color: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 6px;
            padding: 15px;
            margin: 15px 0;
            transition: transform 0.2s;
        }

        .building-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }

        .building-name {
            font-size: 1.3em;
            font-weight: bold;
            color: #2c5aa0;
            margin-bottom: 10px;
        }

        .building-info {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 10px;
            margin-bottom: 10px;
        }

        .info-item {
            background: white;
            padding: 8px 12px;
            border-radius: 4px;
            border-left: 3px solid #2c5aa0;
        }

        .info-label {
            font-weight: bold;
            color: #555;
            font-size: 0.9em;
        }

        .info-value {
            color: #333;
            margin-top: 2px;
        }

        .floors-list {
            background: white;
            padding: 10px;
            border-radius: 4px;
            margin-top: 10px;
        }

        .floor-tag {
            display: inline-block;
            background: #2c5aa0;
            color: white;
            padding: 4px 8px;
            border-radius: 3px;
            margin: 2px;
            font-size: 0.85em;
        }

        .controls {
            text-align: center;
            margin-bottom: 20px;
        }

        .btn {
            background: #2c5aa0;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            margin: 0 5px;
            font-size: 1em;
        }

        .btn:hover {
            background: #1e3f73;
        }

        .btn:disabled {
            background: #ccc;
            cursor: not-allowed;
        }

        .stats {
            background: #e8f4fd;
            border-radius: 6px;
            padding: 15px;
            margin-bottom: 20px;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 15px;
            text-align: center;
        }

        .stat-item {
            background: white;
            padding: 10px;
            border-radius: 4px;
        }

        .stat-number {
            font-size: 1.5em;
            font-weight: bold;
            color: #2c5aa0;
        }

        .stat-label {
            font-size: 0.9em;
            color: #666;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>MapsIndoors Buildings Data</h1>
        
        <div class="controls">
            <button class="btn" id="fetch-buildings">Fetch Buildings</button>
            <button class="btn" id="clear-results">Clear Results</button>
        </div>

        <div id="status" class="loading">
            Click "Fetch Buildings" to load building data...
        </div>

        <div id="stats" class="stats" style="display: none;">
            <div class="stats-grid">
                <div class="stat-item">
                    <div class="stat-number" id="total-buildings">0</div>
                    <div class="stat-label">Total Buildings</div>
                </div>
                <div class="stat-item">
                    <div class="stat-number" id="total-floors">0</div>
                    <div class="stat-label">Total Floors</div>
                </div>
                <div class="stat-item">
                    <div class="stat-number" id="avg-floors">0</div>
                    <div class="stat-label">Avg Floors/Building</div>
                </div>
                <div class="stat-item">
                    <div class="stat-number" id="load-time">0ms</div>
                    <div class="stat-label">Load Time</div>
                </div>
            </div>
        </div>

        <div id="results"></div>
    </div>

    <script>
        // Using VenuesService.getBuildings() to get buildings
        async function getBuildingsViaVenues() {
            try {
                console.log('Fetching buildings using VenuesService.getBuildings()...');
                const buildings = await mapsindoors.services.VenuesService.getBuildings();
                console.log('Buildings data:', buildings);
                
                const processedBuildings = [];
                
                if (buildings && buildings.length > 0) {
                    buildings.forEach(building => {
                        const floors = building.floors ? Object.keys(building.floors).sort((a, b) => parseInt(a) - parseInt(b)) : [];
                        
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
                } else {
                    console.log('No buildings found or empty response');
                }
                
                console.log('Processed buildings:', processedBuildings);
                return processedBuildings;
                
            } catch (error) {
                console.error('Error fetching buildings via VenuesService:', error);
                throw error;
            }
        }
                
        // Main fetch function
        async function fetchBuildings() {
            const startTime = Date.now();
            const statusDiv = document.getElementById('status');
            const resultsDiv = document.getElementById('results');
            const fetchBtn = document.getElementById('fetch-buildings');
            
            // Reset UI
            statusDiv.innerHTML = '<div class="loading">Loading buildings...</div>';
            resultsDiv.innerHTML = '';
            document.getElementById('stats').style.display = 'none';
            fetchBtn.disabled = true;
            
            try {
                const buildings = await getBuildingsViaVenues();
                const loadTime = Date.now() - startTime;
                
                if (!buildings || buildings.length === 0) {
                    statusDiv.innerHTML = '<div class="error">No buildings found or service returned empty result.</div>';
                    return;
                }
                
                // Display success message
                statusDiv.innerHTML = `<div class="success">Successfully loaded ${buildings.length} building${buildings.length !== 1 ? 's' : ''} using VenuesService.getBuildings()</div>`;
                
                // Display buildings
                displayBuildings(buildings);
                
                // Update stats
                updateStats(buildings, loadTime);
                
            } catch (error) {
                console.error('Failed to fetch buildings:', error);
                statusDiv.innerHTML = `<div class="error">Error: ${error.message}</div>`;
            } finally {
                fetchBtn.disabled = false;
            }
        }

        function displayBuildings(buildings) {
            const resultsDiv = document.getElementById('results');
            
            buildings.forEach(building => {
                const buildingCard = document.createElement('div');
                buildingCard.className = 'building-card';
                
                let floorsDisplay = '';
                if (building.floors && building.floors.length > 0) {
                    floorsDisplay = `
                        <div class="floors-list">
                            <strong>Floors:</strong><br>
                            ${building.floors.map(floor => `<span class="floor-tag">Floor ${floor}</span>`).join('')}
                        </div>
                    `;
                }
                
                buildingCard.innerHTML = `
                    <div class="building-name">${building.name}</div>
                    <div class="building-info">
                        <div class="info-item">
                            <div class="info-label">Building ID</div>
                            <div class="info-value">${building.id}</div>
                        </div>
                        <div class="info-item">
                            <div class="info-label">Floor Count</div>
                            <div class="info-value">${building.floorCount}</div>
                        </div>
                        ${building.venue ? `
                            <div class="info-item">
                                <div class="info-label">Venue</div>
                                <div class="info-value">${building.venue}</div>
                            </div>
                        ` : ''}
                        ${building.address ? `
                            <div class="info-item">
                                <div class="info-label">Address</div>
                                <div class="info-value">${building.address}</div>
                            </div>
                        ` : ''}
                    </div>
                    ${floorsDisplay}
                `;
                
                resultsDiv.appendChild(buildingCard);
            });
        }

        function updateStats(buildings, loadTime) {
            const totalFloors = buildings.reduce((sum, building) => sum + (building.floorCount || 0), 0);
            const avgFloors = buildings.length > 0 ? (totalFloors / buildings.length).toFixed(1) : 0;
            
            document.getElementById('total-buildings').textContent = buildings.length;
            document.getElementById('total-floors').textContent = totalFloors;
            document.getElementById('avg-floors').textContent = avgFloors;
            document.getElementById('load-time').textContent = loadTime + 'ms';
            document.getElementById('stats').style.display = 'block';
        }

        function clearResults() {
            document.getElementById('results').innerHTML = '';
            document.getElementById('status').innerHTML = 'Click "Fetch Buildings" to load building data...';
            document.getElementById('stats').style.display = 'none';
        }

        // Event listeners
        document.getElementById('fetch-buildings').addEventListener('click', fetchBuildings);
        document.getElementById('clear-results').addEventListener('click', clearResults);

        // Auto-fetch on page load (with delay to ensure MapsIndoors is ready)
        window.addEventListener('load', () => {
            setTimeout(() => {
                console.log('Page loaded, MapsIndoors should be ready');
                // Uncomment the line below to auto-fetch on page load
                // fetchBuildings();
            }, 1500);
        });

        // Debug: Log when MapsIndoors is available
        if (typeof mapsindoors !== 'undefined') {
            console.log('MapsIndoors is available:', mapsindoors);
        } else {
            console.log('MapsIndoors not yet loaded, waiting...');
        }
    </script>
</body>
</html>
```

### Explanation
- Uses event listeners for interactive functionality
- Implements asynchronous operations for better performance
- Includes proper error handling and recovery

### Gotchas / Notes
ÃÂ°ÃÂÃÂÃÂ ÃÂ¯ÃÂ¸ÃÂ Always check if DOM elements exist before using them



