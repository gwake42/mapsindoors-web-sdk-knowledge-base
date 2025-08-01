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


---

## Spatial Queries with MapsIndoors LocationsService

### Context
Spatial queries are essential for location-aware applications that need to find nearby points of interest

### Industry
corporate

### Problem
Finding relevant locations based on proximity and spatial context

### Solution
```javascript
// Proximity search with LocationsService
async function findNearbyLocations(lat, lng, floor) {
    try {
        const locations = await mapsindoors.services.LocationsService.getLocations({
            near: { lat: lat, lng: lng },
            radius: 5, // meters
            floor: floor,
            venue: 'AUSTINOFFICE',
            types: ['MeetingRoom', 'MeetingRoom Small', 'MeetingRoom Medium', 'MeetingRoom Large'],
            take: 10
        });
        
        return locations;
    } catch (error) {
        console.error('Error finding nearby locations:', error);
        return [];
    }
}

// Usage with draggable marker
marker.on('dragend', async () => {
    const lngLat = marker.getLngLat();
    const currentFloor = mapsIndoors.getFloor();
    
    const nearbyLocations = await findNearbyLocations(
        lngLat.lat, 
        lngLat.lng, 
        currentFloor
    );
    
    if (nearbyLocations.length > 0) {
        const closest = nearbyLocations[0];
        console.log('Closest location:', closest.properties.name);
        
        // Update UI with nearest location info
        popup.setHTML(`
            <div>
                <p>Nearest: ${closest.properties.name}</p>
                <p>Distance: ~${Math.round(calculateDistance(lngLat, closest))}m</p>
            </div>
        `);
    }
});

// Filter locations by type and venue
const meetingRooms = await mapsindoors.services.LocationsService.getLocations({
    types: ['MeetingRoom'],
    venue: 'AUSTINOFFICE',
    take: 100
});
```

### Explanation
The LocationsService provides powerful spatial query capabilities using the 'near' parameter with coordinates and radius. This enables finding locations within a specific distance of a point, essential for asset tracking, room finding, and proximity-based features in indoor mapping applications.

### Use Cases
- Asset tracking and assignment
- Room finder applications
- Wayfinding to nearest amenities
- Location-based content delivery
- Emergency evacuation routing

### Important Notes
⚠️ Radius parameter is in meters
⚠️ Always include floor parameter to limit results to current floor
⚠️ Use venue parameter for better performance and relevant results
⚠️ Limit results with take parameter to improve response time
⚠️ Handle async errors gracefully


---

## Route Optimization for Multi-Stop Indoor Navigation

### Context
Route optimization is crucial for efficiency in cleaning rounds, maintenance tasks, and guided tours

### Industry
healthcare

### Problem
Calculate optimized multi-stop routes for efficient navigation through indoor spaces

### Solution
```javascript
// Multi-stop route optimization with DirectionsService
const miDirectionsService = new mapsindoors.services.DirectionsService();
const miDirectionsRenderer = new mapsindoors.directions.DirectionsRenderer({
    mapsIndoors: mapsIndoorsInstance,
    fitBounds: true,
    fitBoundsPadding: 100
});

async function calculateOptimizedRoute(startRoom, endRoom, waypointRooms) {
    const routeRequest = {
        origin: {
            lat: startRoom.location.lat,
            lng: startRoom.location.lng,
            floor: startRoom.floor
        },
        destination: {
            lat: endRoom.location.lat,
            lng: endRoom.location.lng,
            floor: endRoom.floor
        },
        stops: waypointRooms.map(room => ({
            lat: room.location.lat,
            lng: room.location.lng,
            floor: room.floor
        })),
        optimize: true // Automatically reorders stops for efficiency
    };
    
    try {
        const route = await miDirectionsService.getRoute(routeRequest);
        
        if (route && route.legs && route.legs.length > 0) {
            // Display the route on the map
            miDirectionsRenderer.setRoute(route);
            
            // Log the optimized order
            console.log(`Route optimized: ${route.legs.length} legs`);
            
            return route;
        } else {
            throw new Error('No valid route found');
        }
    } catch (error) {
        console.error('Route calculation failed:', error);
        return null;
    }
}

// Navigate through route legs
let currentLegIndex = 0;

function nextLeg() {
    if (currentRoute && currentLegIndex < currentRoute.legs.length - 1) {
        currentLegIndex++;
        miDirectionsRenderer.nextLeg();
    }
}

function previousLeg() {
    if (currentRoute && currentLegIndex > 0) {
        currentLegIndex--;
        miDirectionsRenderer.previousLeg();
    }
}
```

### Explanation
The DirectionsService can optimize multi-stop routes by automatically reordering waypoints for maximum efficiency. Setting optimize: true calculates the best order to visit all stops, reducing travel time and distance. The DirectionsRenderer provides visual feedback and step-by-step navigation.

### Use Cases
- Cleaning route optimization
- Maintenance task scheduling
- Guided facility tours
- Emergency response planning
- Asset collection routes

### Important Notes
⚠️ Include floor parameter for proper 3D navigation
⚠️ Handle route calculation failures gracefully
⚠️ The optimize parameter reorders stops automatically
⚠️ Use DirectionsRenderer for visual route display
⚠️ Check for valid route before proceeding


---

## Custom Floor Selector Component

### Context
Custom floor selectors provide branded styling and enhanced functionality beyond the default MapsIndoors component

### Industry
corporate

### Problem
Create custom-styled floor navigation that matches application branding

### Solution
```javascript
// Custom floor selector implementation
class CustomFloorSelector {
    constructor(mapsIndoorsInstance) {
        this.mapsIndoors = mapsIndoorsInstance;
        this.element = this.createSelectorElement();
        this.floors = {};
    }

    createSelectorElement() {
        const container = document.createElement('div');
        container.style.cssText = `
            position: absolute;
            top: 20px;
            right: 20px;
            background: rgba(30, 30, 30, 0.8);
            padding: 10px;
            border-radius: 15px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
            display: flex;
            flex-direction: column;
            backdrop-filter: blur(10px);
        `;
        return container;
    }

    onShow() {
        this.element.style.display = 'flex';
        this.updateFloorButtons();
    }

    onHide() {
        this.element.style.display = 'none';
    }

    updateFloorButtons() {
        this.element.innerHTML = '';
        const currentFloor = this.mapsIndoors.getFloor();

        if (Object.keys(this.floors).length === 0) {
            const noFloorsMessage = document.createElement('div');
            noFloorsMessage.textContent = 'No floors available';
            this.element.appendChild(noFloorsMessage);
            return;
        }

        // Sort floors in descending order (highest first)
        const sortedFloors = Object.entries(this.floors)
            .sort(([a], [b]) => Number(b) - Number(a));

        sortedFloors.forEach(([floorIndex, floorName]) => {
            const button = document.createElement('button');
            button.textContent = floorName;
            button.style.cssText = `
                margin: 5px 0;
                padding: 10px 20px;
                border: none;
                background-color: ${floorIndex === currentFloor ? '#c8a028' : '#3c3c3c'};
                color: white;
                cursor: pointer;
                border-radius: 10px;
                transition: all 0.2s ease;
            `;
            button.onclick = () => this.changeFloor(floorIndex);
            this.element.appendChild(button);
        });
    }

    changeFloor(floorIndex) {
        this.mapsIndoors.setFloor(floorIndex);
        this.updateFloorButtons();
    }

    updateFloors(floors) {
        if (floors) {
            this.floors = Object.entries(floors).reduce((acc, [index, floorInfo]) => {
                acc[index] = floorInfo.name || `Floor ${index}`;
                return acc;
            }, {});
        } else {
            this.floors = {};
        }
        this.updateFloorButtons();
    }

    updateWithCurrentBuilding() {
        const currentBuilding = this.mapsIndoors.getBuilding();
        if (currentBuilding) {
            this.updateFloors(currentBuilding.floors);
        } else {
            this.updateFloors(null);
        }
    }
}

// Usage
const customFloorSelector = new CustomFloorSelector(mapsIndoorsInstance);
document.body.appendChild(customFloorSelector.element);

mapsIndoorsInstance.addListener('ready', () => {
    customFloorSelector.onShow();
    customFloorSelector.updateWithCurrentBuilding();
});

mapsIndoorsInstance.addListener('building_changed', () => {
    customFloorSelector.updateWithCurrentBuilding();
});
```

### Explanation
Custom floor selector components give you complete control over appearance and behavior. This pattern creates a reusable class that handles floor display, selection, and building changes while providing custom styling that matches your application's design.

### Use Cases
- Branded floor navigation
- Mobile-optimized interfaces
- Dashboard integrations
- Custom UI themes
- Enhanced accessibility features

### Important Notes
⚠️ Sort floors in descending order for intuitive display
⚠️ Update buttons when floor changes via setFloor
⚠️ Handle building changes to refresh available floors
⚠️ Check for empty floors object before rendering
⚠️ Use building_changed listener to update floor list


---

## Draggable Markers with Location Assignment

### Context
Draggable markers enable interactive asset assignment and location-based content management

### Industry
healthcare

### Problem
Interactive assignment of assets or resources to specific indoor locations

### Solution
```javascript
// Interactive draggable markers with location assignment
function addDraggableMarker(mapboxInstance, mapsIndoorsInstance) {
    const center = mapboxInstance.getCenter();
    
    // Create custom marker element
    const customMarkerElement = document.createElement('div');
    customMarkerElement.style.cssText = `
        background-image: url('https://media.mapsindoors.com/57e4e4992e74800ef8b69718/media/room-occupied.svg');
        background-size: cover;
        width: 40px;
        height: 40px;
        cursor: pointer;
    `;
    
    // Create draggable marker
    const marker = new mapboxgl.Marker({ 
        element: customMarkerElement,
        draggable: true 
    })
    .setLngLat([center.lng, center.lat])
    .addTo(mapboxInstance);

    // Create popup for location assignment
    const popup = new mapboxgl.Popup({
        closeButton: true,
        closeOnClick: false
    }); 
    
    marker.setPopup(popup);

    // Handle drag end to find nearest location
    marker.on('dragend', async () => {
        const lngLat = marker.getLngLat();
        console.log('Marker dragged to:', lngLat);

        try {
            // Find locations near the marker position
            const locations = await mapsindoors.services.LocationsService.getLocations({
                near: { lat: lngLat.lat, lng: lngLat.lng },
                radius: 1,
                take: 1
            });

            if (locations.length > 0) {
                const nearestLocation = locations[0];
                
                // Show assignment popup
                popup.setHTML(`
                    <div style="padding: 10px;">
                        <p>Assign to: ${nearestLocation.properties.name}</p>
                        <button id="assignButton" style="margin-top: 10px; padding: 5px 10px;">
                            Assign Asset
                        </button>
                    </div>
                `);
                popup.addTo(mapboxInstance);

                // Handle assignment
                document.getElementById('assignButton').addEventListener('click', () => {
                    // Apply visual feedback to assigned location
                    mapsIndoorsInstance.setDisplayRule(nearestLocation.id, {
                        polygonVisible: true,
                        polygonFillOpacity: 1,
                        polygonFillColor: "#7D49F3",
                        visible: true,
                        icon: "https://media.mapsindoors.com/57e4e4992e74800ef8b69718/media/room-occupied.svg"
                    });
                    
                    popup.remove();
                    console.log('Asset assigned to:', nearestLocation.properties.name);
                });
            } else {
                popup.setHTML('<p>No nearby locations found</p>');
                popup.addTo(mapboxInstance);
            }
        } catch (error) {
            console.error('Error finding locations:', error);
            popup.setHTML('<p>Error finding nearby locations</p>');
            popup.addTo(mapboxInstance);
        }
    });
    
    return marker;
}

// Usage
document.getElementById('addMarkerButton').addEventListener('click', () => {
    addDraggableMarker(mapboxInstance, mapsIndoorsInstance);
});
```

### Explanation
This pattern creates interactive draggable markers that automatically detect the nearest MapsIndoors location when moved. Users can assign assets, equipment, or other resources to specific locations through an intuitive drag-and-drop interface with immediate visual feedback.

### Use Cases
- Asset management and tracking
- Equipment assignment
- Resource allocation
- Interactive space planning
- Emergency equipment placement

### Important Notes
⚠️ Use small radius (1m) for precise location detection
⚠️ Handle async location search errors gracefully
⚠️ Remove event listeners to prevent memory leaks
⚠️ Provide visual feedback for successful assignments
⚠️ Consider floor-aware location searches


---

## Distance Matrix vs Directions API Usage Patterns

### Context
Understanding when to use Distance Matrix vs Directions API optimizes performance and provides appropriate data

### Industry
corporate

### Problem
Choosing the right API for distance calculations vs detailed navigation

### Solution
```javascript
// Distance Matrix vs Directions API comparison
const directionsService = new mapsindoors.services.DirectionsService();

// Use Distance Matrix for bulk distance calculations
async function calculateDistanceMatrix(origin, destinations) {
    try {
        const matrix = await mapsindoors.services.DistanceMatrixService.getDistanceMatrix({
            graphId: 'AUSTINOFFICE_Graph',
            origins: [`${origin.lat},${origin.lng},${origin.floor}`],
            destinations: destinations.map(dest => 
                `${dest.lat},${dest.lng},${dest.floor}`
            )
        });
        
        // Process results - matrix gives distance and duration for each destination
        const results = destinations.map((dest, index) => ({
            destination: dest,
            distance: matrix.rows[0].elements[index].distance.value, // meters
            duration: matrix.rows[0].elements[index].duration.value, // seconds
            distanceText: matrix.rows[0].elements[index].distance.text
        }));
        
        // Sort by distance to find nearest
        results.sort((a, b) => a.distance - b.distance);
        return results;
        
    } catch (error) {
        console.error('Distance Matrix error:', error);
        return [];
    }
}

// Use Directions API for actual navigation with turn-by-turn
async function getDetailedRoute(origin, destination) {
    try {
        const route = await directionsService.getRoute({
            origin: {
                lat: origin.lat,
                lng: origin.lng,
                floor: origin.floor
            },
            destination: {
                lat: destination.lat,
                lng: destination.lng,
                floor: destination.floor
            }
        });
        
        if (route && route.legs) {
            // Route provides detailed turn-by-turn instructions
            const totalDistance = route.legs.reduce((sum, leg) => sum + leg.distance.value, 0);
            const totalDuration = route.legs.reduce((sum, leg) => sum + leg.duration.value, 0);
            
            return {
                route: route,
                totalDistance: totalDistance,
                totalDuration: totalDuration,
                steps: route.legs.flatMap(leg => leg.steps || [])
            };
        }
        
        return null;
    } catch (error) {
        console.error('Directions error:', error);
        return null;
    }
}

// Example: Find nearest meeting rooms, then get route to closest
async function findNearestAndNavigate(userLocation, meetingRooms) {
    // Step 1: Use Distance Matrix to find nearest rooms
    const nearestRooms = await calculateDistanceMatrix(userLocation, meetingRooms);
    
    if (nearestRooms.length > 0) {
        console.log(`Nearest room: ${nearestRooms[0].destination.name} (${nearestRooms[0].distanceText})`);
        
        // Step 2: Use Directions API for actual navigation
        const route = await getDetailedRoute(userLocation, nearestRooms[0].destination);
        
        if (route) {
            console.log(`Route has ${route.steps.length} steps`);
            return { nearestRooms, route };
        }
    }
    
    return { nearestRooms, route: null };
}
```

### Explanation
Distance Matrix is ideal for bulk distance calculations and finding nearest locations, while Directions API provides detailed turn-by-turn navigation. Distance Matrix is more efficient for comparing multiple destinations, but Directions gives you the actual route geometry and step-by-step instructions needed for navigation.

### Use Cases
- Finding nearest amenities
- Route planning optimization
- Location comparison tools
- Navigation applications
- Proximity-based recommendations

### Important Notes
⚠️ Distance Matrix requires graphId parameter
⚠️ Use Distance Matrix for bulk comparisons, Directions for navigation
⚠️ Distance Matrix returns straight distances, Directions considers actual walkable paths
⚠️ Format coordinates as 'lat,lng,floor' strings for Distance Matrix
⚠️ Directions API provides route geometry and turn instructions


---

## MapsIndoors Event Handling Patterns

### Context
Proper event handling ensures responsive and interactive MapsIndoors applications

### Industry
corporate

### Problem
Creating responsive and interactive indoor mapping applications

### Solution
```javascript
// Event handling patterns for MapsIndoors
mapsIndoorsInstance.addListener('ready', () => {
    console.log('MapsIndoors is ready');
    
    // Safe to call MapsIndoors methods after ready event
    const building = mapsIndoorsInstance.getBuilding();
    const floor = mapsIndoorsInstance.getFloor();
    
    // Set initial display rules
    mapsIndoorsInstance.setDisplayRule(locationIds, {
        visible: true,
        iconVisible: false,
        polygonVisible: true
    });
});

// Handle location clicks
mapsIndoorsInstance.addListener('click', (event) => {
    if (event && event.id) {
        console.log('Clicked location:', event.id);
        
        // Get location details
        const location = event.location;
        if (location) {
            showLocationInfo(location);
        }
    }
});

// Handle floor changes
mapsIndoorsInstance.addListener('floor_changed', (event) => {
    const newFloor = event.floor;
    console.log('Floor changed to:', newFloor);
    
    // Update UI elements that depend on current floor
    updateFloorDependentContent(newFloor);
    updateMarkerVisibility(newFloor);
});

// Handle building changes
mapsIndoorsInstance.addListener('building_changed', (event) => {
    const newBuilding = event.building;
    console.log('Building changed to:', newBuilding?.id);
    
    // Update building-specific content
    if (newBuilding) {
        updateBuildingInfo(newBuilding);
        updateAvailableFloors(newBuilding.floors);
    }
});

// Handle mouse events for interactive features
mapsIndoorsInstance.addListener('mouse_over', (event) => {
    if (event && event.id) {
        showLocationTooltip(event);
    }
});

mapsIndoorsInstance.addListener('mouse_out', () => {
    hideLocationTooltip();
});

// Error handling
mapsIndoorsInstance.addListener('error', (error) => {
    console.error('MapsIndoors error:', error);
    
    // Handle different types of errors
    if (error.type === 'authentication') {
        showAuthError();
    } else if (error.type === 'network') {
        showNetworkError();
    } else {
        showGenericError(error.message);
    }
});

// Helper functions for event responses
function updateMarkerVisibility(currentFloor) {
    markers.forEach(marker => {
        const markerFloor = marker.floor;
        marker.getElement().style.display = 
            markerFloor === currentFloor ? 'block' : 'none';
    });
}

function showLocationInfo(location) {
    const infoPanel = document.getElementById('location-info');
    infoPanel.innerHTML = `
        <h3>${location.properties.name}</h3>
        <p>Floor: ${location.properties.floor}</p>
        <p>Type: ${location.properties.type}</p>
    `;
    infoPanel.style.display = 'block';
}
```

### Explanation
MapsIndoors provides several event listeners for building interactive applications. The 'ready' event is crucial for initialization, while 'click', 'floor_changed', and 'building_changed' events enable responsive user interactions. Mouse events add hover effects and tooltips for enhanced user experience.

### Use Cases
- Interactive location selection
- Floor-aware content updates
- Hover effects and tooltips
- Error handling and user feedback
- Building navigation interfaces

### Important Notes
⚠️ Always wait for 'ready' event before calling MapsIndoors methods
⚠️ Check if event.id exists before using it
⚠️ Handle error events to provide user feedback
⚠️ Floor and building change events may fire during initialization
⚠️ Remove event listeners when cleaning up components


---

## Real-time Data Sync with Integration API

### Context
Integration API enables real-time synchronization of external data with MapsIndoors locations

### Industry
healthcare

### Problem
Synchronizing real-time external data with MapsIndoors location database

### Solution
```javascript
// Integration API for real-time data synchronization
const apiConfig = {
    bearerToken: 'your-bearer-token',
    solutionId: 'your-solution-id',
    datasetId: 'your-dataset-id',
    baseUrl: 'https://integration.mapsindoors.com'
};

// Format location data for MapsIndoors Integration API
function formatLocationData(locations) {
    return locations.map(location => ({
        parentId: location.parentId, // Floor ID
        datasetId: apiConfig.datasetId,
        solutionId: apiConfig.solutionId,
        baseType: "poi",
        displayTypeId: location.displayTypeId,
        geometry: {
            coordinates: [location.lng, location.lat],
            type: "Point"
        },
        anchor: {
            coordinates: [location.lng, location.lat],
            type: "Point"
        },
        aliases: [],
        categories: [],
        status: 3,
        baseTypeProperties: {
            administrativeid: location.id,
            capacity: "0"
        },
        properties: {
            "name@en": location.name,
            "description@en": location.description || "",
            "name@generic": "generic"
        }
    }));
}

// Sync data to MapsIndoors
async function syncDataToMapsIndoors(locationData) {
    try {
        const formattedData = formatLocationData(locationData);
        
        const response = await fetch(`${apiConfig.baseUrl}/${apiConfig.solutionId}/api/geodata`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${apiConfig.bearerToken}`,
                'accept': 'application/json'
            },
            body: JSON.stringify(formattedData)
        });

        if (!response.ok) {
            throw new Error(`API request failed: ${response.status} ${response.statusText}`);
        }

        const result = await response.json();
        console.log('Sync successful:', result);

        // Trigger content synchronization to update the map
        await mapsindoors.MapsIndoors.synchronizeContent();
        console.log('Content synchronized');
        
        return result;
    } catch (error) {
        console.error('Sync error:', error);
        throw error;
    }
}

// Real-time data update pattern
async function updateLocationFromExternalAPI(externalApiData) {
    try {
        // Process external data
        const mappedLocations = externalApiData.map(item => ({
            id: item.deviceId,
            name: item.deviceName,
            lat: item.coordinates.latitude,
            lng: item.coordinates.longitude,
            parentId: getFloorId(item.floor),
            displayTypeId: 'your-display-type-id',
            description: `Status: ${item.status}`
        }));

        // Sync to MapsIndoors
        await syncDataToMapsIndoors(mappedLocations);
        
        // Update local UI
        updateLocationMarkers(mappedLocations);
        
        console.log(`Updated ${mappedLocations.length} locations`);
        
    } catch (error) {
        console.error('Failed to update locations:', error);
    }
}

// Periodic sync example
function startPeriodicSync(intervalMinutes = 5) {
    setInterval(async () => {
        try {
            // Fetch latest data from your external API
            const externalData = await fetchExternalLocationData();
            
            // Update MapsIndoors
            await updateLocationFromExternalAPI(externalData);
            
        } catch (error) {
            console.error('Periodic sync failed:', error);
        }
    }, intervalMinutes * 60 * 1000);
}

// Helper function to get floor ID from floor number
function getFloorId(floorNumber) {
    const building = mapsIndoorsInstance.getBuilding();
    if (building && building.floors && building.floors[floorNumber]) {
        return building.floors[floorNumber].id;
    }
    throw new Error(`Floor ${floorNumber} not found`);
}
```

### Explanation
The MapsIndoors Integration API allows you to programmatically create, update, and delete location data. This pattern shows how to format external data (IoT devices, assets, etc.) for the Integration API and keep MapsIndoors synchronized with real-time information from external systems.

### Use Cases
- IoT device tracking
- Asset management systems
- Real-time occupancy data
- Equipment status monitoring
- Dynamic content updates

### Important Notes
⚠️ Bearer token required for authentication
⚠️ Call synchronizeContent() after API updates
⚠️ Format coordinates as [lng, lat] for geometry
⚠️ Use correct parentId (floor ID) for location hierarchy
⚠️ Handle API rate limits and failures gracefully


---

## Hide Polygons and Extrusions for Location Types Using Display Rules

### Context
Developer needed to clean up MapsIndoors map display by hiding unwanted visual elements (polygons, strokes, and 3D extrusions) for specific location types while maintaining functionality. This is a common requirement when customizing indoor maps for cleaner interfaces.

### Industry
corporate

### Problem
Need to hide polygon fill, stroke, and 3D extrusions for specific location types (like Neighborhoods) to create cleaner map interfaces without visual clutter

### Solution
```javascript
// Hide Neighborhood Polygons and Extrusions - MapsIndoors Display Rules
// This pattern can be applied to any location type that needs visual elements hidden

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

// Critical: Wait for MapsIndoors to be ready before applying display rules
mapsIndoorsInstance.addListener('ready', async () => {
    console.log('MapsIndoors is ready');
    
    // Additional wait time ensures all map elements are fully loaded
    setTimeout(async () => {
        try {
            // Fetch locations by type - replace 'Neighborhood' with any location type
            const neighborhoods = await mapsindoors.services.LocationsService.getLocations({
                types: ['Neighborhood'], // Can be any type: 'Building', 'Room', etc.
                take: 1000 // Adjust based on expected number of locations
            });
            
            console.log(`Found ${neighborhoods.length} neighborhoods`);
            
            if (neighborhoods.length > 0) {
                // Extract location IDs for bulk display rule application
                const neighborhoodIds = neighborhoods.map(neighborhood => neighborhood.id);
                
                // Comprehensive display rule to hide all visual polygon/extrusion elements
                mapsIndoorsInstance.setDisplayRule(neighborhoodIds, {
                    visible: true, // Keep location data accessible for search/routing
                    polygonVisible: false, // Primary setting to hide polygons
                    polygonFillOpacity: 0, // Backup: disable polygon fill
                    polygonStrokeOpacity: 0, // Backup: disable polygon stroke
                    polygonStrokeWidth: 0, // Backup: set stroke width to 0
                    extrusionVisible: false, // Primary setting to hide 3D extrusions
                    extrusionOpacity: 0, // Backup: set extrusion opacity to 0
                    extrusionHeight: 0, // Backup: set extrusion height to 0
                    iconVisible: true, // Keep icons visible if needed
                    labelVisible: true, // Keep labels visible if needed
                    zoomFrom: 1, // Apply at all zoom levels
                    zoomTo: 25
                });
                
                console.log(`Applied display rules to ${neighborhoodIds.length} neighborhoods - visual elements hidden`);
            } else {
                console.log('No neighborhoods found in this solution');
            }
        } catch (error) {
            console.error('Error applying neighborhood display rules:', error);
        }
    }, 2000); // 2-second delay after ready event - adjust if needed
});

// Error handling
mapsIndoorsInstance.addListener('error', (error) => {
    console.error('MapsIndoors error:', error);
});
```

### Explanation
This code demonstrates the proper way to hide visual elements for specific location types using MapsIndoors display rules. The key steps are: 1) Wait for MapsIndoors ready event, 2) Add additional timeout for full loading, 3) Fetch locations by type, 4) Apply comprehensive display rules that disable polygons and extrusions while keeping location data accessible. Multiple properties are set as both primary controls and backups to ensure complete visual hiding.

### Use Cases
- Creating clean map interfaces by hiding neighborhood boundaries
- Customizing indoor maps for specific use cases (retail, corporate, healthcare)
- Removing visual clutter while maintaining location functionality
- Selective hiding of location types based on business requirements
- Preparing maps for branded or white-label applications

### Important Notes
⚠️ Must wait for MapsIndoors ready event before applying display rules
⚠️ Additional setTimeout (2+ seconds) often needed for complex solutions to fully load
⚠️ Use MapboxV3View instead of deprecated MapboxView for SDK 4.41.1+
⚠️ Set both primary (polygonVisible: false) and backup properties (opacity: 0) for reliable hiding
⚠️ Keep visible: true to maintain location data for search and routing functionality
⚠️ Test with different location types as some may require different property combinations


---

## Get Buildings List Without Map Using VenuesService

### Context
User needed to get a complete list of buildings from their MapsIndoors solution without showing a map interface. We tried multiple approaches including SolutionsService.getBuildings() (which doesn't exist) and complex venue processing before settling on the direct VenuesService.getBuildings() method as the cleanest solution.

### Industry
facility-management

### Problem
Need to retrieve a complete list of all buildings in a MapsIndoors solution without initializing or displaying a map interface

### Solution
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MapsIndoors Buildings - No Map</title>
    <!-- MapsIndoors JavaScript -->
    <script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz?apikey=YOUR_API_KEY"></script>
    
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

        .building-card {
            background-color: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 6px;
            padding: 15px;
            margin: 15px 0;
            transition: transform 0.2s;
        }

        .building-name {
            font-size: 1.3em;
            font-weight: bold;
            color: #2c5aa0;
            margin-bottom: 10px;
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

        <div id="results"></div>
    </div>

    <script>
        // Using VenuesService.getBuildings() to get buildings directly
        async function getBuildingsViaVenues() {
            try {
                console.log('Fetching buildings via VenuesService.getBuildings()...');
                const buildings = await mapsindoors.services.VenuesService.getBuildings();
                console.log('Buildings data:', buildings);
                
                if (!buildings || buildings.length === 0) {
                    console.log('No buildings found or empty response');
                    return [];
                }
                
                // Process buildings data to ensure proper building names
                const processedBuildings = buildings.map(building => {
                    const floors = building.floors ? Object.keys(building.floors).sort((a, b) => parseInt(a) - parseInt(b)) : [];
                    
                    return {
                        id: building.id,
                        name: building.name || building.buildingInfo?.name || building.displayName || `Building ${building.id}`,
                        venue: building.venue || building.venueId,
                        floors: floors,
                        floorCount: floors.length,
                        address: building.address || building.buildingInfo?.address,
                        geometry: building.geometry,
                        bbox: building.bbox,
                        anchor: building.anchor,
                        floorData: building.floors
                    };
                });
                
                console.log('Processed buildings:', processedBuildings);
                return processedBuildings;
                
            } catch (error) {
                console.error('Error fetching buildings via VenuesService.getBuildings():', error);
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

        // Event listeners
        document.getElementById('fetch-buildings').addEventListener('click', fetchBuildings);
        document.getElementById('clear-results').addEventListener('click', () => {
            document.getElementById('results').innerHTML = '';
            document.getElementById('status').innerHTML = 'Click "Fetch Buildings" to load building data...';
        });

        // Auto-fetch on page load (optional)
        window.addEventListener('load', () => {
            setTimeout(() => {
                console.log('Page loaded, MapsIndoors should be ready');
                // Uncomment the line below to auto-fetch on page load
                // fetchBuildings();
            }, 1500);
        });
    </script>
</body>
</html>
```

### Explanation
This solution uses the VenuesService.getBuildings() method to retrieve all building data without initializing a map. The code loads only the MapsIndoors SDK (no Mapbox required) and provides a clean interface to fetch and display building information including names, IDs, floor counts, venues, and addresses. The building data is processed to ensure proper name extraction with multiple fallbacks, and floors are displayed as visual tags sorted numerically.

### Use Cases
- Administrative dashboards needing building lists
- Building management systems
- Data export utilities
- Reporting applications
- Building directory websites
- Asset management systems

### Important Notes
⚠️ Replace YOUR_API_KEY with actual MapsIndoors API key
⚠️ VenuesService.getBuildings() is the correct method - not SolutionsService.getBuildings()
⚠️ Only MapsIndoors SDK is needed - no Mapbox scripts required
⚠️ Add delay before fetching to ensure MapsIndoors is fully loaded
⚠️ Building names may be in different properties - use multiple fallbacks
⚠️ Floor data comes as object keys that need to be sorted numerically


---

## Emergency Response Dashboard with Real-time Status Updates

### Context
Creating an emergency response dashboard that provides real-time visual feedback for building emergencies with color-coded room status indicators

### Industry
healthcare

### Problem
Building emergency response systems need real-time visual status updates for rooms/zones during emergencies with different emergency types (fire, lockdown, medical, weather) and room statuses (safe, unsafe, unresponsive, cleared)

### Solution
```javascript
// Emergency response dashboard with status-based room coloring
const statusColors = {
    'safe': '#4caf50',
    'unsafe': '#f44336', 
    'unresponsive': '#9e9e9e',
    'cleared': '#2196f3',
    'lockdown': '#ff9800'
};

class EmergencyResponseDashboard {
    constructor(mapsIndoorsInstance) {
        this.mapsIndoors = mapsIndoorsInstance;
        this.roomsData = [];
        this.isEmergencyActive = false;
        this.emergencyType = null;
    }

    async loadRoomData() {
        try {
            const locations = await mapsindoors.services.LocationsService.getLocations({
                types: ['MeetingRoom', 'MeetingRoom Small', 'MeetingRoom Medium', 'MeetingRoom Large'],
                venue: 'AUSTINOFFICE',
                take: 100
            });
            
            this.roomsData = locations.map(location => ({
                id: location.id,
                name: location.properties.name || 'Unnamed Room',
                floor: location.properties.floor,
                location: {
                    lat: location.properties.anchor.coordinates[1],
                    lng: location.properties.anchor.coordinates[0]
                },
                status: 'unresponsive', // Initial status during emergency
                lastUpdated: null,
                occupancy: Math.floor(Math.random() * 20)
            }));
            
            this.updateRoomDisplayRules();
            
        } catch (error) {
            console.error('Error loading room data:', error);
        }
    }

    activateEmergency(type) {
        this.isEmergencyActive = true;
        this.emergencyType = type;
        
        // Reset all rooms to unresponsive status
        this.roomsData.forEach(room => {
            room.status = 'unresponsive';
            room.lastUpdated = null;
        });
        
        // Show emergency banner
        this.showEmergencyBanner(type);
        
        // Update room colors to indicate lockdown/emergency state
        this.updateRoomDisplayRules();
        
        // Start simulated status updates (replace with real data source)
        this.startStatusUpdates();
    }

    deactivateEmergency() {
        this.isEmergencyActive = false;
        
        // Set all rooms to safe
        this.roomsData.forEach(room => {
            room.status = 'safe';
            room.lastUpdated = new Date();
        });
        
        this.hideEmergencyBanner();
        this.updateRoomDisplayRules();
        this.stopStatusUpdates();
    }

    updateRoomStatus(roomId, newStatus) {
        const roomIndex = this.roomsData.findIndex(r => r.id === roomId);
        if (roomIndex !== -1) {
            this.roomsData[roomIndex].status = newStatus;
            this.roomsData[roomIndex].lastUpdated = new Date();
            this.updateRoomDisplayRules();
            this.updateStatusCounts();
        }
    }

    updateRoomDisplayRules() {
        // Group rooms by status
        const roomsByStatus = {
            safe: this.roomsData.filter(room => room.status === 'safe').map(room => room.id),
            unsafe: this.roomsData.filter(room => room.status === 'unsafe').map(room => room.id),
            unresponsive: this.roomsData.filter(room => room.status === 'unresponsive').map(room => room.id),
            cleared: this.roomsData.filter(room => room.status === 'cleared').map(room => room.id)
        };

        // Apply color-coded display rules for each status group
        Object.entries(roomsByStatus).forEach(([status, roomIds]) => {
            if (roomIds.length > 0) {
                this.mapsIndoors.setDisplayRule(roomIds, {
                    polygonVisible: true,
                    polygonFillColor: statusColors[status],
                    polygonFillOpacity: 0.5,
                    polygonStrokeColor: statusColors[status],
                    polygonStrokeOpacity: 0.8,
                    polygonStrokeWidth: 1,
                    zoomFrom: 16
                });
            }
        });
    }

    showEmergencyBanner(type) {
        const banner = document.getElementById('emergency-banner');
        const emergencyTexts = {
            'lockdown': 'Building Lockdown',
            'fire': 'Fire Emergency', 
            'medical': 'Medical Emergency',
            'weather': 'Weather Emergency'
        };
        
        banner.className = `emergency-banner active ${type}`;
        banner.innerHTML = `
            <i class="fas fa-exclamation-triangle"></i>
            <span>Active Emergency: ${emergencyTexts[type]}</span>
            <span style="opacity: 0.8; margin-left: 10px;">${new Date().toLocaleTimeString()}</span>
        `;
    }

    startStatusUpdates() {
        this.statusUpdateInterval = setInterval(() => {
            if (!this.isEmergencyActive) return;
            
            // Simulate random status updates (replace with real data)
            const updateCount = Math.floor(this.roomsData.length * 0.1);
            
            for (let i = 0; i < updateCount; i++) {
                const randomIndex = Math.floor(Math.random() * this.roomsData.length);
                const room = this.roomsData[randomIndex];
                
                // Skip recently updated rooms
                if (room.lastUpdated && (new Date() - room.lastUpdated) < 60000) continue;
                
                // Random status with emergency weighting
                const rand = Math.random();
                let newStatus;
                if (rand < 0.4) newStatus = 'unsafe';
                else if (rand < 0.7) newStatus = 'safe';
                else if (rand < 0.9) newStatus = 'cleared';
                else newStatus = 'unresponsive';
                
                this.updateRoomStatus(room.id, newStatus);
            }
        }, 10000); // Update every 10 seconds
    }

    updateStatusCounts() {
        const counts = {
            safe: this.roomsData.filter(room => room.status === 'safe').length,
            unsafe: this.roomsData.filter(room => room.status === 'unsafe').length,
            unresponsive: this.roomsData.filter(room => room.status === 'unresponsive').length,
            cleared: this.roomsData.filter(room => room.status === 'cleared').length,
            total: this.roomsData.length
        };
        
        document.getElementById('safe-count').textContent = counts.safe;
        document.getElementById('unsafe-count').textContent = counts.unsafe;
        document.getElementById('unresponsive-count').textContent = counts.unresponsive;
        document.getElementById('cleared-count').textContent = counts.cleared;
        document.getElementById('total-count').textContent = counts.total;
    }
}

// Usage
const emergencyDashboard = new EmergencyResponseDashboard(mapsIndoorsInstance);

mapsIndoorsInstance.addListener('ready', () => {
    emergencyDashboard.loadRoomData();
});

// Emergency activation
document.getElementById('activate-lockdown').addEventListener('click', () => {
    emergencyDashboard.activateEmergency('lockdown');
});

// Room status updates via click
mapsIndoorsInstance.addListener('click', (event) => {
    if (event?.id && emergencyDashboard.isEmergencyActive) {
        // Show context menu to update room status
        showStatusContextMenu(event.id, event.pixel);
    }
});
```

### Explanation
This solution creates a comprehensive emergency response system that visually represents room safety status during emergencies. It uses MapsIndoors display rules to color-code rooms based on their safety status (safe=green, unsafe=red, unresponsive=gray, cleared=blue). The system includes emergency type management, real-time status updates, and statistical dashboards for emergency coordinators.

### Use Cases
- Building emergency evacuation coordination
- Safety status monitoring during lockdowns
- Fire emergency response tracking
- Medical emergency area isolation
- Weather emergency shelter-in-place management

### Important Notes
⚠️ Must wait for MapsIndoors 'ready' event before applying display rules
⚠️ Status updates should be throttled to avoid overwhelming the display
⚠️ Color choices should be accessibility-friendly
⚠️ Emergency banner requires proper CSS z-index to appear above map


---

## Moving Asset Tracker with Path Visualization

### Context
Creating a real-time asset tracking system with moving markers that show path history and location awareness within MapsIndoors buildings

### Industry
transportation

### Problem
Real-time tracking of moving assets (people, equipment, vehicles) within buildings with path visualization and location awareness

### Solution
```javascript
// Asset tracking with moving markers and path visualization
class MovingAssetTracker {
    constructor(mapsIndoorsInstance, mapboxInstance) {
        this.mapsIndoors = mapsIndoorsInstance;
        this.mapbox = mapboxInstance;
        this.assets = new Map();
        this.selectedAssetId = null;
        this.buildingGeometry = null;
        
        this.initializePathVisualization();
    }

    initializePathVisualization() {
        // Add path source and layer for selected asset
        this.mapbox.addSource('asset-path', {
            type: 'geojson',
            data: {
                type: 'Feature',
                properties: {},
                geometry: {
                    type: 'LineString',
                    coordinates: []
                }
            }
        });

        this.mapbox.addLayer({
            id: 'asset-path-layer',
            type: 'line',
            source: 'asset-path',
            layout: {
                'line-join': 'round',
                'line-cap': 'round'
            },
            paint: {
                'line-color': '#4CAF50',
                'line-width': 3,
                'line-opacity': 0.8
            }
        });
    }

    createAsset(floor, assetType = 'device') {
        const assetId = this.generateAssetId();
        const initialPosition = this.getRandomPointInBuilding();
        
        if (!initialPosition) {
            console.error('Cannot create asset: Building geometry not available');
            return null;
        }

        // Create Mapbox marker
        const marker = new mapboxgl.Marker({
            color: this.getAssetColor(assetType)
        })
        .setLngLat(initialPosition)
        .addTo(this.mapbox);

        // Create asset object
        const asset = {
            id: assetId,
            type: assetType,
            floor: floor,
            marker: marker,
            position: initialPosition,
            targetPosition: this.getNewTargetPosition(),
            positions: [initialPosition], // Track movement history
            lastUpdate: Date.now(),
            isMoving: true
        };

        // Add click handler
        marker.getElement().addEventListener('click', (e) => {
            e.stopPropagation();
            this.selectAsset(assetId);
        });

        this.assets.set(assetId, asset);
        this.startAssetMovement(assetId);
        
        return assetId;
    }

    selectAsset(assetId) {
        // If clicking the same asset, deselect it
        if (this.selectedAssetId === assetId) {
            this.selectedAssetId = null;
            this.clearPathVisualization();
            return;
        }

        this.selectedAssetId = assetId;
        const asset = this.assets.get(assetId);
        
        if (asset) {
            // Show asset path
            this.updatePathVisualization(asset.positions);
            
            // Center map on asset
            this.mapbox.flyTo({
                center: asset.position,
                zoom: 20,
                duration: 1000
            });

            // Set to asset's floor
            this.mapsIndoors.setFloor(asset.floor);
            
            // Show asset info popup
            this.showAssetInfo(asset);
        }
    }

    updatePathVisualization(positions) {
        this.mapbox.getSource('asset-path').setData({
            type: 'Feature',
            properties: {},
            geometry: {
                type: 'LineString',
                coordinates: positions
            }
        });
    }

    clearPathVisualization() {
        this.mapbox.getSource('asset-path').setData({
            type: 'Feature',
            properties: {},
            geometry: {
                type: 'LineString',
                coordinates: []
            }
        });
    }

    startAssetMovement(assetId) {
        const moveAsset = async () => {
            const asset = this.assets.get(assetId);
            if (!asset || !asset.isMoving) return;

            // Calculate new position towards target
            const newPosition = this.getPointWithinRadius(
                asset.position, 
                asset.targetPosition, 
                20 // 20 feet movement radius
            );

            // Check if reached target
            const dx = asset.targetPosition[0] - newPosition[0];
            const dy = asset.targetPosition[1] - newPosition[1];
            const distanceToTarget = Math.sqrt(dx * dx + dy * dy);

            if (distanceToTarget < 0.0000001) {
                asset.targetPosition = this.getNewTargetPosition();
            }

            // Update asset position
            asset.marker.setLngLat(newPosition);
            asset.position = newPosition;
            asset.positions.push([...newPosition]);
            asset.lastUpdate = Date.now();

            // Update path visualization if this asset is selected
            if (this.selectedAssetId === assetId) {
                this.updatePathVisualization(asset.positions);
            }

            // Find nearest MapsIndoors location
            try {
                const nearestLocations = await mapsindoors.services.LocationsService.getLocations({
                    near: { lat: newPosition[1], lng: newPosition[0] },
                    radius: 1,
                    floor: asset.floor,
                    take: 1
                });

                if (nearestLocations.length > 0) {
                    asset.nearestLocation = nearestLocations[0].properties.name;
                }
            } catch (error) {
                console.error('Error finding nearest location:', error);
            }

            // Schedule next movement
            const delay = Math.random() * 5000 + 2000; // 2-7 seconds
            setTimeout(moveAsset, delay);
        };

        // Start with initial delay
        const initialDelay = Math.random() * 5000 + 2000;
        setTimeout(moveAsset, initialDelay);
    }

    showAssetInfo(asset) {
        const popup = new mapboxgl.Popup({
            closeButton: true,
            closeOnClick: false
        });

        const timeSinceUpdate = this.formatTimeSince(asset.lastUpdate);
        
        const popupContent = `
            <div style="padding: 10px; max-width: 200px;">
                <div style="font-weight: bold; margin-bottom: 8px;">Asset #${asset.id}</div>
                <div style="margin-bottom: 5px;">Type: ${asset.type}</div>
                <div style="margin-bottom: 5px;">Floor: ${asset.floor}</div>
                <div style="margin-bottom: 5px;">Last Update: ${timeSinceUpdate}</div>
                ${asset.nearestLocation ? `<div style="margin-bottom: 5px;">Near: ${asset.nearestLocation}</div>` : ''}
                <div style="margin-bottom: 5px;">Positions Tracked: ${asset.positions.length}</div>
                <button onclick="window.assetTracker.stopAssetMovement('${asset.id}')" 
                        style="padding: 5px 10px; margin-right: 5px; background: #f44336; color: white; border: none; border-radius: 3px;">
                    Stop
                </button>
                <button onclick="window.assetTracker.deleteAsset('${asset.id}')" 
                        style="padding: 5px 10px; background: #666; color: white; border: none; border-radius: 3px;">
                    Delete
                </button>
            </div>
        `;

        popup.setLngLat(asset.position)
            .setHTML(popupContent)
            .addTo(this.mapbox);

        asset.popup = popup;
    }

    stopAssetMovement(assetId) {
        const asset = this.assets.get(assetId);
        if (asset) {
            asset.isMoving = false;
            if (asset.popup) {
                asset.popup.remove();
            }
        }
    }

    deleteAsset(assetId) {
        const asset = this.assets.get(assetId);
        if (asset) {
            asset.marker.remove();
            if (asset.popup) {
                asset.popup.remove();
            }
            this.assets.delete(assetId);
            
            if (this.selectedAssetId === assetId) {
                this.selectedAssetId = null;
                this.clearPathVisualization();
            }
        }
    }

    updateAssetVisibility(currentFloor) {
        this.assets.forEach(asset => {
            const display = asset.floor === currentFloor ? 'block' : 'none';
            asset.marker.getElement().style.display = display;
        });
    }

    // Helper methods
    generateAssetId() {
        return Math.floor(Math.random() * 900 + 100).toString();
    }

    getAssetColor(type) {
        const colors = {
            'device': '#FF0000',
            'person': '#2196F3', 
            'equipment': '#FF9800',
            'vehicle': '#4CAF50'
        };
        return colors[type] || '#666666';
    }

    getRandomPointInBuilding() {
        if (!this.buildingGeometry) {
            const building = this.mapsIndoors.getBuilding();
            if (building) {
                this.buildingGeometry = building.geometry;
            } else {
                return null;
            }
        }

        const bounds = this.buildingGeometry.bbox;
        let point;
        let attempts = 0;
        const maxAttempts = 100;

        do {
            const lng = Math.random() * (bounds[2] - bounds[0]) + bounds[0];
            const lat = Math.random() * (bounds[3] - bounds[1]) + bounds[1];
            point = [lng, lat];
            attempts++;

            if (attempts > maxAttempts) {
                return [bounds[0], bounds[1]];
            }
        } while (!this.isPointInPolygon(point, this.buildingGeometry.coordinates[0]));

        return point;
    }

    getNewTargetPosition() {
        return this.getRandomPointInBuilding();
    }

    getPointWithinRadius(currentPoint, targetPoint, radiusFeet) {
        const FEET_TO_DEGREES = 0.0000003048;
        const maxDistance = radiusFeet * FEET_TO_DEGREES;

        const dx = targetPoint[0] - currentPoint[0];
        const dy = targetPoint[1] - currentPoint[1];
        const currentDistance = Math.sqrt(dx * dx + dy * dy);

        if (currentDistance <= maxDistance) {
            return targetPoint;
        }

        const ratio = maxDistance / currentDistance;
        return [
            currentPoint[0] + dx * ratio,
            currentPoint[1] + dy * ratio
        ];
    }

    isPointInPolygon(point, polygon) {
        let inside = false;
        for (let i = 0, j = polygon.length - 1; i < polygon.length; j = i++) {
            const xi = polygon[i][0], yi = polygon[i][1];
            const xj = polygon[j][0], yj = polygon[j][1];
            
            const intersect = ((yi > point[1]) !== (yj > point[1])) &&
                (point[0] < (xj - xi) * (point[1] - yi) / (yj - yi) + xi);
            if (intersect) inside = !inside;
        }
        return inside;
    }

    formatTimeSince(timestamp) {
        const seconds = Math.floor((Date.now() - timestamp) / 1000);
        if (seconds < 60) return `${seconds}s ago`;
        if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
        if (seconds < 86400) return `${Math.floor(seconds / 3600)}h ago`;
        return `${Math.floor(seconds / 86400)}d ago`;
    }
}

// Usage
let assetTracker;

mapsIndoorsInstance.addListener('ready', () => {
    assetTracker = new MovingAssetTracker(mapsIndoorsInstance, mapboxInstance);
    
    // Make accessible globally for popup buttons
    window.assetTracker = assetTracker;
});

// Floor change listener
mapsIndoorsInstance.addListener('floor_changed', () => {
    const currentFloor = mapsIndoorsInstance.getFloor();
    if (assetTracker) {
        assetTracker.updateAssetVisibility(currentFloor);
    }
});

// Add asset button
document.getElementById('add-asset').addEventListener('click', () => {
    const currentFloor = mapsIndoorsInstance.getFloor();
    assetTracker.createAsset(currentFloor, 'device');
});

// Map click to deselect asset
mapboxInstance.on('click', () => {
    if (assetTracker?.selectedAssetId) {
        assetTracker.selectAsset(assetTracker.selectedAssetId);
    }
});
```

### Explanation
This creates a comprehensive asset tracking system where markers move autonomously around the building, maintaining path history, and providing location-aware information. Assets can be selected to view their movement paths, and the system integrates with MapsIndoors floor management. Each asset tracks its position history and can identify its nearest MapsIndoors location.

### Use Cases
- Equipment tracking in hospitals
- Vehicle tracking in parking garages
- Personnel tracking during emergencies
- Valuable asset monitoring
- Delivery tracking within facilities

### Important Notes
⚠️ Building geometry must be available before creating assets
⚠️ Path visualization requires adding Mapbox sources/layers after map load
⚠️ Asset movement should be throttled to avoid performance issues
⚠️ Selected asset state needs to be cleared when asset is deleted


---

## Corporate Room Booking with Availability Visualization

### Context
Building a corporate room booking system with real-time availability visualization and integrated booking workflow

### Industry
corporate

### Problem
Corporate offices need efficient room booking systems with visual availability indicators and streamlined booking workflows

### Solution
```javascript
// Corporate room booking system with availability visualization
class CorporateRoomBooking {
    constructor(mapsIndoorsInstance) {
        this.mapsIndoors = mapsIndoorsInstance;
        this.rooms = [];
        this.selectedRoomId = null;
        
        // Mock booking data structure
        this.bookings = new Map();
        this.availabilityColors = {
            'available': '#4CAF50',
            'busy': '#f44336',
            'waitlist': '#ff9800'
        };
    }

    async loadRooms() {
        try {
            const locations = await mapsindoors.services.LocationsService.getLocations({
                types: ['MeetingRoom', 'MeetingRoom Small', 'MeetingRoom Medium', 'MeetingRoom Large'],
                venue: 'AUSTINOFFICE',
                take: 100
            });

            this.rooms = locations.map(location => {
                const capacity = this.getRoomCapacity(location.properties.type);
                return {
                    id: location.id,
                    name: location.properties.name || 'Unnamed Room',
                    type: location.properties.type || 'MeetingRoom',
                    capacity: capacity,
                    floor: location.properties.floor,
                    features: this.getRoomFeatures(location.properties.type),
                    location: {
                        lat: location.properties.anchor.coordinates[1],
                        lng: location.properties.anchor.coordinates[0]
                    },
                    availability: this.generateAvailability()
                };
            });

            this.generateTimeSlots();
            this.updateRoomDisplayRules();
            this.renderRoomList();

        } catch (error) {
            console.error('Error loading rooms:', error);
        }
    }

    generateTimeSlots() {
        const timeSlots = ['9AM', '10AM', '11AM', '12PM', '1PM', '2PM', '3PM', '4PM', '5PM'];
        
        this.rooms.forEach(room => {
            room.timeSlots = {};
            timeSlots.forEach(time => {
                // Random availability for demo
                const statuses = ['available', 'busy', 'available', 'available']; // Weight toward available
                room.timeSlots[time] = statuses[Math.floor(Math.random() * statuses.length)];
            });
            
            // Set current availability based on current time slot
            const currentHour = new Date().getHours();
            const currentTimeSlot = timeSlots.find(slot => {
                const hour = slot.includes('PM') && !slot.includes('12PM') ? 
                    parseInt(slot) + 12 : parseInt(slot);
                return hour === currentHour;
            });
            
            room.currentAvailability = currentTimeSlot ? 
                room.timeSlots[currentTimeSlot] : 'available';
        });
    }

    updateRoomDisplayRules() {
        // Group rooms by current availability
        const availableRooms = this.rooms.filter(room => room.currentAvailability === 'available').map(r => r.id);
        const busyRooms = this.rooms.filter(room => room.currentAvailability === 'busy').map(r => r.id);
        
        // Apply availability-based colors
        this.mapsIndoors.setDisplayRule(availableRooms, {
            polygonVisible: true,
            polygonFillColor: this.availabilityColors.available,
            polygonFillOpacity: 0.4,
            polygonStrokeColor: this.availabilityColors.available,
            polygonStrokeOpacity: 0.8,
            polygonStrokeWidth: 1,
            zoomFrom: 16
        });

        this.mapsIndoors.setDisplayRule(busyRooms, {
            polygonVisible: true,
            polygonFillColor: this.availabilityColors.busy,
            polygonFillOpacity: 0.4,
            polygonStrokeColor: this.availabilityColors.busy,
            polygonStrokeOpacity: 0.8,
            polygonStrokeWidth: 1,
            zoomFrom: 16
        });
    }

    renderRoomList() {
        const roomList = document.getElementById('room-list');
        if (!roomList) return;

        roomList.innerHTML = '';

        // Sort rooms by availability (available first)
        const sortedRooms = [...this.rooms].sort((a, b) => {
            if (a.currentAvailability === 'available' && b.currentAvailability !== 'available') return -1;
            if (a.currentAvailability !== 'available' && b.currentAvailability === 'available') return 1;
            return a.name.localeCompare(b.name);
        });

        sortedRooms.forEach(room => {
            const roomCard = document.createElement('div');
            roomCard.className = `room-card ${room.currentAvailability} ${this.selectedRoomId === room.id ? 'selected' : ''}`;
            roomCard.dataset.id = room.id;

            const availabilityText = room.currentAvailability === 'available' ? 'Available Now' : 'Currently Busy';
            const statusIcon = room.currentAvailability === 'available' ? 'fas fa-check-circle' : 'fas fa-clock';

            roomCard.innerHTML = `
                <div class="room-header">
                    <div class="room-name">${room.name}</div>
                    <div class="room-type">${this.formatRoomType(room.type)}</div>
                </div>
                <div class="room-details">
                    <div class="room-detail">
                        <i class="fas fa-users"></i>
                        <span>Capacity: ${room.capacity} people</span>
                    </div>
                    <div class="room-detail">
                        <i class="fas fa-building"></i>
                        <span>Floor ${room.floor}</span>
                    </div>
                    <div class="room-detail">
                        <i class="fas fa-list-ul"></i>
                        <span>${room.features.slice(0, 2).join(', ')}</span>
                    </div>
                </div>
                <div class="room-status">
                    <span class="availability ${room.currentAvailability}">
                        <i class="${statusIcon}"></i>
                        ${availabilityText}
                    </span>
                    <button class="waitlist-btn" onclick="bookingSystem.openBookingModal('${room.id}')">
                        <i class="fas fa-calendar-plus"></i>
                        ${room.currentAvailability === 'available' ? 'Book Now' : 'Join Waitlist'}
                    </button>
                </div>
            `;

            roomCard.addEventListener('click', () => {
                this.selectRoom(room.id);
            });

            roomList.appendChild(roomCard);
        });
    }

    selectRoom(roomId) {
        this.selectedRoomId = roomId;
        const room = this.rooms.find(r => r.id === roomId);
        if (!room) return;

        // Highlight selected room
        this.mapsIndoors.setDisplayRule(roomId, {
            polygonVisible: true,
            polygonFillColor: '#2196F3',
            polygonFillOpacity: 0.6,
            polygonStrokeColor: '#2196F3',
            polygonStrokeOpacity: 1,
            polygonStrokeWidth: 2,
            zoomFrom: 16
        });

        // Center map on room
        this.mapsIndoors.setFloor(room.floor);
        
        // Show room details panel
        this.showRoomDetails(room);
        
        // Update room cards selection
        document.querySelectorAll('.room-card').forEach(card => {
            if (card.dataset.id === roomId) {
                card.classList.add('selected');
            } else {
                card.classList.remove('selected');
            }
        });
    }

    showRoomDetails(room) {
        const detailPanel = document.getElementById('detail-panel');
        if (!detailPanel) return;

        // Update room info
        document.getElementById('detail-title').textContent = room.name;
        document.getElementById('detail-subtitle').textContent = `Floor ${room.floor} • ${this.formatRoomType(room.type)}`;

        // Update features
        const featureList = document.getElementById('feature-list');
        featureList.innerHTML = `
            <div class="feature-item">
                <i class="fas fa-users"></i>
                <span>Capacity: ${room.capacity} people</span>
            </div>
        `;
        
        room.features.forEach(feature => {
            const icon = this.getFeatureIcon(feature);
            featureList.innerHTML += `
                <div class="feature-item">
                    <i class="${icon}"></i>
                    <span>${feature}</span>
                </div>
            `;
        });

        // Update availability timeline
        const timeline = document.getElementById('availability-timeline');
        timeline.innerHTML = '';
        
        Object.entries(room.timeSlots).forEach(([time, status]) => {
            const timeSlot = document.createElement('div');
            timeSlot.className = `time-slot ${status}`;
            timeSlot.innerHTML = `<span>${time}</span>`;
            timeline.appendChild(timeSlot);
        });

        // Update time slot dropdown
        const timeSlotSelect = document.getElementById('time-slot');
        timeSlotSelect.innerHTML = '<option value="">Select a time slot</option>';
        
        Object.entries(room.timeSlots).forEach(([time, status]) => {
            const option = document.createElement('option');
            option.value = time;
            option.textContent = `${time} (${status === 'available' ? 'Available' : 'Waitlist'})`;
            timeSlotSelect.appendChild(option);
        });

        // Update availability status
        const availabilityEl = document.getElementById('room-availability');
        availabilityEl.textContent = room.currentAvailability === 'available' ? 'Available Now' : 'Currently Busy';
        availabilityEl.className = `availability ${room.currentAvailability}`;

        // Show panel
        detailPanel.classList.add('visible');
    }

    openBookingModal(roomId) {
        const room = this.rooms.find(r => r.id === roomId);
        if (!room) return;

        const modal = document.getElementById('booking-modal');
        if (!modal) return;

        // Populate modal with room details
        document.getElementById('modal-room').value = room.name;
        
        // Show modal
        modal.classList.add('show');
    }

    submitBooking(bookingData) {
        const { roomId, timeSlot, attendees, purpose, name, email } = bookingData;
        
        // Validate booking data
        if (!roomId || !timeSlot || !name || !email) {
            throw new Error('Missing required booking information');
        }

        // Create booking record
        const bookingId = Date.now().toString();
        const booking = {
            id: bookingId,
            roomId,
            timeSlot,
            attendees: parseInt(attendees) || 1,
            purpose,
            bookedBy: { name, email },
            timestamp: new Date(),
            status: 'confirmed'
        };

        // Store booking
        this.bookings.set(bookingId, booking);

        // Update room availability
        const room = this.rooms.find(r => r.id === roomId);
        if (room && room.timeSlots[timeSlot]) {
            room.timeSlots[timeSlot] = 'busy';
            
            // Update current availability if this is the current time slot
            const currentHour = new Date().getHours();
            const timeHour = timeSlot.includes('PM') && !timeSlot.includes('12PM') ? 
                parseInt(timeSlot) + 12 : parseInt(timeSlot);
            
            if (timeHour === currentHour) {
                room.currentAvailability = 'busy';
                this.updateRoomDisplayRules();
            }
        }

        // Update UI
        this.renderRoomList();
        if (this.selectedRoomId === roomId) {
            this.showRoomDetails(room);
        }

        return booking;
    }

    // Helper methods
    getRoomCapacity(type) {
        const capacities = {
            'MeetingRoom Small': Math.floor(Math.random() * 3) + 2, // 2-4
            'MeetingRoom Extra Small': Math.floor(Math.random() * 2) + 2, // 2-3
            'MeetingRoom Medium': Math.floor(Math.random() * 4) + 5, // 5-8
            'MeetingRoom Large': Math.floor(Math.random() * 8) + 10, // 10-17
            'MeetingRoom': Math.floor(Math.random() * 6) + 6 // 6-11
        };
        return capacities[type] || 8;
    }

    getRoomFeatures(type) {
        const allFeatures = ['Video Conferencing', 'Whiteboard', 'Display Screen', 'High-Speed Wi-Fi', 'Phone', 'AV Equipment'];
        const baseFeatures = ['High-Speed Wi-Fi'];
        
        if (type?.includes('Large')) {
            return [...baseFeatures, 'Video Conferencing', 'Whiteboard', 'Display Screen', 'AV Equipment'];
        } else if (type?.includes('Medium')) {
            return [...baseFeatures, 'Video Conferencing', 'Display Screen', 'Whiteboard'];
        } else if (type?.includes('Small')) {
            return [...baseFeatures, 'Display Screen'];
        }
        
        return [...baseFeatures, 'Video Conferencing', 'Display Screen'];
    }

    formatRoomType(type) {
        return type?.replace('MeetingRoom', 'Meeting Room') || 'Meeting Room';
    }

    getFeatureIcon(feature) {
        const icons = {
            'Video Conferencing': 'fas fa-video',
            'Whiteboard': 'fas fa-chalkboard',
            'Display Screen': 'fas fa-tv',
            'High-Speed Wi-Fi': 'fas fa-wifi',
            'Phone': 'fas fa-phone',
            'AV Equipment': 'fas fa-volume-up'
        };
        return icons[feature] || 'fas fa-check';
    }

    generateAvailability() {
        return Math.random() > 0.3 ? 'available' : 'busy';
    }
}

// Usage
let bookingSystem;

mapsIndoorsInstance.addListener('ready', () => {
    bookingSystem = new CorporateRoomBooking(mapsIndoorsInstance);
    bookingSystem.loadRooms();
    
    // Make globally accessible for onclick handlers
    window.bookingSystem = bookingSystem;
});

// Handle booking modal submission
document.getElementById('submit-booking')?.addEventListener('click', () => {
    try {
        const bookingData = {
            roomId: bookingSystem.selectedRoomId,
            timeSlot: document.getElementById('time-slot').value,
            attendees: document.getElementById('attendees').value,
            purpose: document.getElementById('meeting-purpose').value,
            name: document.getElementById('modal-name').value,
            email: document.getElementById('modal-email').value
        };

        const booking = bookingSystem.submitBooking(bookingData);
        
        // Show success notification
        console.log('Booking confirmed:', booking);
        document.getElementById('booking-modal').classList.remove('show');
        
        // Reset form
        document.querySelectorAll('#booking-modal input, #booking-modal select').forEach(el => el.value = '');
        
    } catch (error) {
        console.error('Booking error:', error);
        alert('Booking failed: ' + error.message);
    }
});
```

### Explanation
This system provides a comprehensive room booking interface with color-coded availability display, detailed room information, and booking workflow integration. Rooms are visually differentiated by availability status, and users can view detailed room features, time slot availability, and submit bookings. The system handles room capacity, features, and availability management with visual feedback through MapsIndoors display rules.

### Use Cases
- Meeting room reservations
- Workspace hot-desking
- Conference room scheduling
- Equipment room booking
- Client meeting space management

### Important Notes
⚠️ Room availability should be refreshed periodically
⚠️ Booking validation must check for conflicts
⚠️ Room features should be customizable per room type
⚠️ Time slot generation should align with business hours


---

## Temporal Heatmap Visualization with Time Controls

### Context
Creating temporal heatmap visualization for foot traffic, utilization patterns, or density analysis with time-based filtering and customizable color schemes

### Industry
retail

### Problem
Visualizing temporal density patterns, foot traffic flow, space utilization, or activity hotspots with time-based analysis

### Solution
```javascript
// Heatmap data visualization with temporal filtering
class MapsIndoorsHeatmap {
    constructor(mapboxInstance, mapsIndoorsInstance) {
        this.mapbox = mapboxInstance;
        this.mapsIndoors = mapsIndoorsInstance;
        this.heatmapData = [];
        this.currentHour = 12;
        this.colorPalettes = {
            default: [
                'rgba(0, 0, 255, 0)',
                'royalblue',
                'cyan', 
                'lime',
                'yellow',
                'red'
            ],
            thermal: [
                'rgba(255, 255, 0, 0)',
                'yellow',
                'orange',
                'red',
                'purple',
                'darkblue'
            ],
            density: [
                'rgba(255, 255, 255, 0)',
                'lightgreen',
                'green',
                'darkgreen',
                'brown',
                'black'
            ]
        };
        
        this.initializeHeatmapLayer();
    }

    initializeHeatmapLayer() {
        // Wait for map to load before adding heatmap
        this.mapbox.on('load', () => {
            // Add empty GeoJSON source for heatmap data
            this.mapbox.addSource('heatmap-source', {
                type: 'geojson',
                data: {
                    type: "FeatureCollection",
                    features: []
                }
            });

            // Add heatmap layer
            this.mapbox.addLayer({
                id: 'heatmap-layer',
                type: 'heatmap',
                source: 'heatmap-source',
                paint: {
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
                        0, this.colorPalettes.default[0],
                        0.1, this.colorPalettes.default[1],
                        0.3, this.colorPalettes.default[2],
                        0.5, this.colorPalettes.default[3],
                        0.7, this.colorPalettes.default[4],
                        1, this.colorPalettes.default[5]
                    ],
                    'heatmap-radius': [
                        'interpolate',
                        ['linear'],
                        ['zoom'],
                        0, 2,
                        9, 10,
                        16, 20,
                        22, 30
                    ],
                    'heatmap-opacity': 0.8
                }
            });
        });
    }

    generateSampleHeatmapData() {
        console.log("Generating heatmap data...");
        const buildingGeometry = this.mapsIndoors.getBuilding()?.geometry;
        
        if (!buildingGeometry) {
            console.error("Building geometry not available");
            return;
        }

        this.heatmapData = [];
        const today = new Date();

        // Generate data for business hours (8 AM to 6 PM)
        for (let hour = 8; hour <= 18; hour++) {
            // Create 3-5 hotspots per hour
            const hotspotCount = Math.floor(Math.random() * 3) + 3;
            const hotspots = this.generateHotspots(buildingGeometry, hotspotCount);
            
            // Generate 200-500 data points per hour clustered around hotspots
            const pointsThisHour = Math.floor(Math.random() * 300) + 200;
            
            for (let i = 0; i < pointsThisHour; i++) {
                let point;
                
                if (Math.random() < 0.8) {
                    // 80% chance to be near a hotspot
                    const hotspot = hotspots[Math.floor(Math.random() * hotspots.length)];
                    point = this.generatePointNearHotspot(hotspot, 20); // 20m radius
                } else {
                    // 20% chance to be random
                    point = this.getRandomPointInBuilding(buildingGeometry);
                }

                if (point && this.isPointInBuilding(point, buildingGeometry)) {
                    const intensity = Math.floor(Math.random() * 10) + 1;
                    
                    this.heatmapData.push({
                        type: "Feature",
                        properties: {
                            intensity: intensity,
                            date: new Date(today.getFullYear(), today.getMonth(), today.getDate(), hour, 0, 0).toISOString(),
                            hour: hour,
                            unit_count: intensity
                        },
                        geometry: {
                            type: "Point",
                            coordinates: point
                        }
                    });
                }
            }
            
            console.log(`Generated data for hour ${hour}`);
        }

        console.log(`Total heatmap points generated: ${this.heatmapData.length}`);
        this.updateHeatmap();
    }

    generateHotspots(buildingGeometry, count) {
        const hotspots = [];
        for (let i = 0; i < count; i++) {
            const hotspot = this.getRandomPointInBuilding(buildingGeometry);
            if (hotspot) {
                hotspots.push(hotspot);
            }
        }
        return hotspots;
    }

    generatePointNearHotspot(center, maxDistanceMeters) {
        const angle = Math.random() * 2 * Math.PI;
        const distance = Math.random() * maxDistanceMeters;
        
        // Convert meters to approximate degrees
        const metersToDegreesLat = 1 / 111000;
        const metersToDegreesLng = 1 / (111000 * Math.cos(center[1] * Math.PI / 180));
        
        const deltaLat = distance * Math.sin(angle) * metersToDegreesLat;
        const deltaLng = distance * Math.cos(angle) * metersToDegreesLng;
        
        return [center[0] + deltaLng, center[1] + deltaLat];
    }

    getRandomPointInBuilding(buildingGeometry) {
        const bounds = buildingGeometry.bbox;
        let attempts = 0;
        const maxAttempts = 100;
        
        while (attempts < maxAttempts) {
            const lng = Math.random() * (bounds[2] - bounds[0]) + bounds[0];
            const lat = Math.random() * (bounds[3] - bounds[1]) + bounds[1];
            const point = [lng, lat];
            
            if (this.isPointInBuilding(point, buildingGeometry)) {
                return point;
            }
            attempts++;
        }
        
        // Fallback to center of bounding box
        return [(bounds[0] + bounds[2]) / 2, (bounds[1] + bounds[3]) / 2];
    }

    isPointInBuilding(point, buildingGeometry) {
        // Simple point-in-polygon check for building bounds
        if (!buildingGeometry.coordinates || !buildingGeometry.coordinates[0]) {
            return true; // Fallback: allow all points if no geometry
        }
        
        const polygon = buildingGeometry.coordinates[0];
        let inside = false;
        
        for (let i = 0, j = polygon.length - 1; i < polygon.length; j = i++) {
            const xi = polygon[i][0], yi = polygon[i][1];
            const xj = polygon[j][0], yj = polygon[j][1];
            
            if (((yi > point[1]) !== (yj > point[1])) &&
                (point[0] < (xj - xi) * (point[1] - yi) / (yj - yi) + xi)) {
                inside = !inside;
            }
        }
        
        return inside;
    }

    updateHeatmap(hour = this.currentHour, palette = 'default') {
        // Filter data for selected hour
        const filteredData = this.heatmapData.filter(feature => 
            feature.properties.hour === hour
        );

        const geojsonData = {
            type: "FeatureCollection",
            features: filteredData
        };

        // Update data source
        if (this.mapbox.getSource('heatmap-source')) {
            this.mapbox.getSource('heatmap-source').setData(geojsonData);
        }

        // Update color palette if heatmap layer exists
        if (this.mapbox.getLayer('heatmap-layer')) {
            this.mapbox.setPaintProperty('heatmap-layer', 'heatmap-color', [
                'interpolate',
                ['linear'],
                ['heatmap-density'],
                0, this.colorPalettes[palette][0],
                0.1, this.colorPalettes[palette][1],
                0.3, this.colorPalettes[palette][2],
                0.5, this.colorPalettes[palette][3],
                0.7, this.colorPalettes[palette][4],
                1, this.colorPalettes[palette][5]
            ]);
        }

        console.log(`Updated heatmap for hour ${hour} with ${filteredData.length} points`);
    }

    setTimeSlider(hour) {
        this.currentHour = hour;
        this.updateHeatmap(hour);
        
        // Update time display
        const timeDisplay = document.getElementById('time-display');
        if (timeDisplay) {
            timeDisplay.textContent = `${hour.toString().padStart(2, '0')}:00`;
        }
    }

    setColorPalette(paletteKey) {
        if (this.colorPalettes[paletteKey]) {
            this.updateHeatmap(this.currentHour, paletteKey);
        }
    }

    toggleHeatmapVisibility() {
        const layer = this.mapbox.getLayer('heatmap-layer');
        if (layer) {
            const visibility = this.mapbox.getLayoutProperty('heatmap-layer', 'visibility');
            this.mapbox.setLayoutProperty('heatmap-layer', 'visibility', 
                visibility === 'visible' ? 'none' : 'visible');
        }
    }

    loadCustomHeatmapData(geojsonData) {
        // Validate and process custom data
        if (!geojsonData || !geojsonData.features) {
            console.error('Invalid GeoJSON data provided');
            return;
        }

        // Ensure all features have required properties
        this.heatmapData = geojsonData.features.map(feature => {
            if (!feature.properties) {
                feature.properties = {};
            }
            
            // Set default intensity if not provided
            if (!feature.properties.intensity) {
                feature.properties.intensity = 1;
            }
            
            // Set default hour if not provided
            if (!feature.properties.hour) {
                feature.properties.hour = new Date().getHours();
            }
            
            return feature;
        });

        console.log(`Loaded ${this.heatmapData.length} custom heatmap points`);
        this.updateHeatmap();
    }

    exportHeatmapData() {
        return {
            type: "FeatureCollection",
            features: this.heatmapData
        };
    }
}

// UI Component for Heatmap Controls
function createHeatmapControls(heatmapInstance) {
    const controlsContainer = document.createElement('div');
    controlsContainer.style.cssText = `
        position: absolute;
        bottom: 20px;
        left: 20px;
        background: white;
        padding: 20px;
        border-radius: 5px;
        box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        z-index: 1000;
        min-width: 300px;
    `;

    controlsContainer.innerHTML = `
        <h3 style="margin-bottom: 15px;">Heatmap Controls</h3>
        
        <button id="generate-data" style="width: 100%; padding: 10px; margin-bottom: 15px; background: #4CAF50; color: white; border: none; border-radius: 4px; cursor: pointer;">
            Generate Sample Data
        </button>
        
        <div style="margin-bottom: 15px;">
            <label style="display: block; margin-bottom: 5px;">Time: <span id="time-display">12:00</span></label>
            <input type="range" id="time-slider" min="8" max="18" value="12" 
                   style="width: 100%;" oninput="updateTimeDisplay(this.value)">
        </div>
        
        <div style="margin-bottom: 15px;">
            <label style="display: block; margin-bottom: 5px;">Color Palette:</label>
            <select id="color-palette" style="width: 100%; padding: 5px;">
                <option value="default">Default</option>
                <option value="thermal">Thermal</option>
                <option value="density">Density</option>
            </select>
        </div>
        
        <button id="toggle-heatmap" style="width: 100%; padding: 8px; background: #2196F3; color: white; border: none; border-radius: 4px; cursor: pointer;">
            Toggle Heatmap
        </button>
    `;

    // Add event listeners
    const generateBtn = controlsContainer.querySelector('#generate-data');
    generateBtn.addEventListener('click', () => {
        heatmapInstance.generateSampleHeatmapData();
    });

    const timeSlider = controlsContainer.querySelector('#time-slider');
    timeSlider.addEventListener('input', (e) => {
        heatmapInstance.setTimeSlider(parseInt(e.target.value));
    });

    const colorPalette = controlsContainer.querySelector('#color-palette');
    colorPalette.addEventListener('change', (e) => {
        heatmapInstance.setColorPalette(e.target.value);
    });

    const toggleBtn = controlsContainer.querySelector('#toggle-heatmap');
    toggleBtn.addEventListener('click', () => {
        heatmapInstance.toggleHeatmapVisibility();
    });

    return controlsContainer;
}

// Global function for time display update
window.updateTimeDisplay = function(hour) {
    document.getElementById('time-display').textContent = `${hour.padStart(2, '0')}:00`;
};

// Usage
let heatmapInstance;

mapsIndoorsInstance.addListener('ready', () => {
    heatmapInstance = new MapsIndoorsHeatmap(mapboxInstance, mapsIndoorsInstance);
    
    // Add controls to page
    const controls = createHeatmapControls(heatmapInstance);
    document.body.appendChild(controls);
    
    console.log('Heatmap system initialized');
});
```

### Explanation
This system creates heatmap visualizations using Mapbox's native heatmap layer with temporal filtering capabilities. Users can view density patterns across different time periods, switch between color palettes, and load custom data. The system generates realistic sample data with hotspots and clustering, and provides interactive controls for time navigation and visual customization.

### Use Cases
- Retail foot traffic analysis
- Office space utilization studies
- Event crowd density monitoring
- Hospital patient flow analysis
- Restaurant seating pattern tracking

### Important Notes
⚠️ Heatmap layer must be added after map 'load' event
⚠️ Point coordinates must be in [lng, lat] format for GeoJSON
⚠️ Building geometry is required for realistic point generation
⚠️ Heatmap intensity values should be normalized for best visual results


---

## MapsIndoors PDF Export with Preview System

### Context
Users need to export MapsIndoors maps as high-quality images or PDFs for reports, documentation, or offline use. This provides a complete export system with preview functionality and quality controls.

### Industry
corporate

### Problem
Users need to export MapsIndoors floor plans as high-quality PDFs or PNG images with custom quality settings and preview functionality

### Solution
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MapsIndoors Export System</title>
    <!-- Mapbox CSS -->
    <link href="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.css" rel="stylesheet">
    <!-- MapsIndoors CSS -->
    <script src="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.js"></script>
    <script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz?apikey=YOUR_API_KEY"></script>
    <!-- jsPDF for PDF generation -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
</head>
<body>
    <div id="map" style="width: 100vw; height: 100vh;"></div>
    
    <div class="export-controls" style="position: absolute; top: 20px; left: 20px; z-index: 1000; background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 6px rgba(0,0,0,0.3);">
        <button id="preview-btn" class="btn">Preview Export</button>
        <button id="png-btn" class="btn">Export PNG</button>
        <button id="pdf-btn" class="btn">Export PDF</button>
    </div>

    <!-- Preview Modal -->
    <div id="preview-modal" class="modal-backdrop" style="display: none; position: fixed; z-index: 2000; left: 0; top: 0; width: 100%; height: 100%; background-color: rgba(0,0,0,0.5);">
        <div class="modal-content" style="background: white; margin: 5% auto; padding: 20px; border-radius: 8px; max-width: 800px; width: 80%;">
            <h2>Export Preview</h2>
            <div class="quality-control" style="margin: 15px 0;">
                <label>Quality Scale: <span id="scale-value">2x</span></label>
                <input type="range" id="scale-slider" min="1" max="4" value="2" style="width: 100%;">
            </div>
            <div class="preview-container" style="text-align: center; border: 2px dashed #ccc; padding: 20px;">
                <img id="preview-image" style="max-width: 100%; max-height: 400px;">
                <p id="preview-dimensions">Dimensions: --</p>
            </div>
            <div class="export-actions" style="margin-top: 20px; text-align: right;">
                <button id="modal-png-btn" class="btn">Export PNG</button>
                <button id="modal-pdf-btn" class="btn">Export PDF</button>
                <button id="close-preview" class="btn" style="background: #666;">Close</button>
            </div>
        </div>
    </div>

    <script>
        class MapsIndoorsExporter {
            constructor() {
                this.mapViewInstance = null;
                this.mapsIndoorsInstance = null;
                this.mapboxInstance = null;
                this.currentScale = 2;
            }

            async initialize() {
                // Initialize MapsIndoors with preserveDrawingBuffer for export
                const mapViewOptions = {
                    accessToken: 'pk.eyJ1IjoiZ2V3YS1tYXBzcGVvcGxlIiwiYSI6ImNsZzJudDB4ZTAwcnEzZnAwb2VvbTYwYnIifQ.w-cnsU-xP9jaly_qrgy_iA',
                    element: document.getElementById('map'),
                    center: { lat: 30.3603212, lng: -97.7422623 },
                    zoom: 19,
                    maxZoom: 24,
                    preserveDrawingBuffer: true // Critical for canvas export
                };

                this.mapViewInstance = new mapsindoors.mapView.MapboxV3View(mapViewOptions);
                this.mapsIndoorsInstance = new mapsindoors.MapsIndoors({
                    mapView: this.mapViewInstance
                });
                this.mapboxInstance = this.mapViewInstance.getMap();

                // Add floor selector
                const floorSelectorElement = document.createElement('div');
                new mapsindoors.FloorSelector(floorSelectorElement, this.mapsIndoorsInstance);
                this.mapboxInstance.addControl({
                    onAdd: function () { return floorSelectorElement },
                    onRemove: function () { }
                });

                this.setupEventListeners();
                await this.waitForReady();
            }

            async waitForReady() {
                return new Promise((resolve) => {
                    this.mapsIndoorsInstance.addListener('ready', () => {
                        console.log('MapsIndoors ready for export');
                        resolve();
                    });
                });
            }

            setupEventListeners() {
                document.getElementById('preview-btn').addEventListener('click', () => this.showPreview());
                document.getElementById('png-btn').addEventListener('click', () => this.exportPNG(4));
                document.getElementById('pdf-btn').addEventListener('click', () => this.exportPDF(4));
                
                document.getElementById('modal-png-btn').addEventListener('click', () => this.exportPNG(this.currentScale));
                document.getElementById('modal-pdf-btn').addEventListener('click', () => this.exportPDF(this.currentScale));
                document.getElementById('close-preview').addEventListener('click', () => this.hidePreview());
                
                document.getElementById('scale-slider').addEventListener('input', (e) => {
                    this.currentScale = parseInt(e.target.value);
                    document.getElementById('scale-value').textContent = `${this.currentScale}x`;
                    this.updatePreview();
                });
            }

            async showPreview() {
                document.getElementById('preview-modal').style.display = 'block';
                await this.updatePreview();
            }

            hidePreview() {
                document.getElementById('preview-modal').style.display = 'none';
            }

            async updatePreview() {
                try {
                    const preview = await this.generatePreview(this.currentScale);
                    if (preview) {
                        document.getElementById('preview-image').src = preview.dataUrl;
                        document.getElementById('preview-dimensions').textContent = 
                            `Dimensions: ${preview.dimensions.width} × ${preview.dimensions.height} pixels`;
                    }
                } catch (error) {
                    console.error('Error generating preview:', error);
                }
            }

            async generatePreview(scale) {
                const mapContainer = document.getElementById('map');
                const originalStyle = {
                    width: mapContainer.style.width,
                    height: mapContainer.style.height
                };

                // Store original state
                const originalCenter = this.mapboxInstance.getCenter();
                const originalZoom = this.mapboxInstance.getZoom();

                try {
                    // Calculate preview dimensions
                    const previewWidth = window.innerWidth * scale;
                    const previewHeight = window.innerHeight * scale;
                    
                    // Temporarily resize map for high-res capture
                    mapContainer.style.width = `${previewWidth}px`;
                    mapContainer.style.height = `${previewHeight}px`;
                    
                    // Update map and wait for stabilization
                    this.mapboxInstance.resize();
                    this.mapboxInstance.setCenter(originalCenter);
                    this.mapboxInstance.setZoom(originalZoom);
                    
                    await new Promise(resolve => setTimeout(resolve, 1000));
                    
                    // Generate data URL from canvas
                    const canvas = this.mapboxInstance.getCanvas();
                    const dataUrl = canvas.toDataURL('image/jpeg', 0.9);
                    
                    return {
                        dataUrl,
                        dimensions: { 
                            width: Math.round(previewWidth), 
                            height: Math.round(previewHeight) 
                        }
                    };
                } finally {
                    // Reset map to original state
                    mapContainer.style.width = originalStyle.width;
                    mapContainer.style.height = originalStyle.height;
                    this.mapboxInstance.resize();
                    this.mapboxInstance.setCenter(originalCenter);
                    this.mapboxInstance.setZoom(originalZoom);
                }
            }

            async exportPNG(scale) {
                const mapContainer = document.getElementById('map');
                const originalStyle = {
                    width: mapContainer.style.width,
                    height: mapContainer.style.height
                };

                try {
                    // Store original map state
                    const originalCenter = this.mapboxInstance.getCenter();
                    const originalZoom = this.mapboxInstance.getZoom();
                    const currentFloor = this.mapsIndoorsInstance.getFloor();
                    const floorName = currentFloor !== null ? `Floor_${currentFloor}` : 'Ground_Floor';

                    // Set larger dimensions for high-res export
                    mapContainer.style.width = `${window.innerWidth * scale}px`;
                    mapContainer.style.height = `${window.innerHeight * scale}px`;

                    // Resize and stabilize map
                    this.mapboxInstance.resize();
                    this.mapboxInstance.setCenter(originalCenter);
                    this.mapboxInstance.setZoom(originalZoom);

                    await new Promise(resolve => setTimeout(resolve, 2000));

                    // Generate high-res canvas export
                    const canvas = this.mapboxInstance.getCanvas();
                    const dataUrl = canvas.toDataURL('image/png', 1.0);

                    // Create download
                    const downloadLink = document.createElement('a');
                    downloadLink.href = dataUrl;
                    downloadLink.download = `mapsindoors_${floorName}_${Date.now()}.png`;
                    document.body.appendChild(downloadLink);
                    downloadLink.click();
                    document.body.removeChild(downloadLink);

                    console.log('PNG export completed');
                } finally {
                    // Reset map dimensions
                    mapContainer.style.width = originalStyle.width;
                    mapContainer.style.height = originalStyle.height;
                    this.mapboxInstance.resize();
                    this.hidePreview();
                }
            }

            async exportPDF(scale) {
                const { jsPDF } = window.jspdf;
                const mapContainer = document.getElementById('map');
                const originalStyle = {
                    width: mapContainer.style.width,
                    height: mapContainer.style.height
                };

                try {
                    // Store original map state
                    const originalCenter = this.mapboxInstance.getCenter();
                    const originalZoom = this.mapboxInstance.getZoom();
                    const currentFloor = this.mapsIndoorsInstance.getFloor();
                    const floorName = currentFloor !== null ? `Floor_${currentFloor}` : 'Ground_Floor';

                    // Set larger dimensions for higher quality
                    mapContainer.style.width = `${window.innerWidth * scale}px`;
                    mapContainer.style.height = `${window.innerHeight * scale}px`;

                    // Resize and stabilize map
                    this.mapboxInstance.resize();
                    this.mapboxInstance.setCenter(originalCenter);
                    this.mapboxInstance.setZoom(originalZoom);

                    await new Promise(resolve => setTimeout(resolve, 2000));

                    // Generate canvas data URL
                    const canvas = this.mapboxInstance.getCanvas();
                    const dataUrl = canvas.toDataURL('image/jpeg', 1.0);

                    // Create PDF in landscape orientation
                    const pdf = new jsPDF({
                        orientation: 'landscape',
                        unit: 'mm',
                        format: 'a4'
                    });

                    // Set PDF metadata
                    pdf.setProperties({
                        title: `MapsIndoors Floor Plan - ${floorName}`,
                        subject: 'Floor Plan Export',
                        creator: 'MapsIndoors Export Tool',
                        author: 'MapsIndoors'
                    });

                    // Calculate dimensions to fit page while maintaining aspect ratio
                    const pdfWidth = pdf.internal.pageSize.getWidth();
                    const pdfHeight = pdf.internal.pageSize.getHeight();
                    const imgWidth = canvas.width;
                    const imgHeight = canvas.height;
                    const ratio = Math.min(pdfWidth / imgWidth, pdfHeight / imgHeight);
                    
                    // Center the image on the page
                    const imgX = (pdfWidth - imgWidth * ratio) / 2;
                    const imgY = (pdfHeight - imgHeight * ratio) / 2;

                    // Add image to PDF
                    pdf.addImage(dataUrl, 'JPEG', imgX, imgY, imgWidth * ratio, imgHeight * ratio);

                    // Save the PDF
                    pdf.save(`mapsindoors_floorplan_${floorName}_${Date.now()}.pdf`);

                    console.log('PDF export completed');
                } finally {
                    // Reset map dimensions
                    mapContainer.style.width = originalStyle.width;
                    mapContainer.style.height = originalStyle.height;
                    this.mapboxInstance.resize();
                    this.hidePreview();
                }
            }
        }

        // Initialize the exporter
        const exporter = new MapsIndoorsExporter();
        window.addEventListener('load', () => {
            exporter.initialize();
        });
    </script>

    <style>
        .btn {
            padding: 10px 15px;
            margin: 5px;
            background: #2196F3;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
        }
        .btn:hover {
            background: #1976D2;
        }
        .btn:disabled {
            background: #ccc;
            cursor: not-allowed;
        }
    </style>
</body>
</html>
```

### Explanation
This code creates a comprehensive export system for MapsIndoors maps. Key features include: 1) Canvas-based high-resolution image generation using preserveDrawingBuffer, 2) Preview modal with quality scaling controls, 3) PNG export with downloadable links, 4) PDF generation using jsPDF library with proper aspect ratio handling, 5) Temporary map resizing for high-quality exports while preserving original view state. The system maintains map center and zoom during export operations.

### Use Cases
- Corporate floor plan documentation
- Emergency evacuation plan creation
- Facility management reports
- Real estate marketing materials
- Offline map reference creation

### Important Notes
⚠️ Must set preserveDrawingBuffer: true in MapboxV3View options for canvas export to work
⚠️ Temporary map resizing requires proper cleanup to avoid UI issues
⚠️ Large scale exports may consume significant memory
⚠️ PDF export requires jsPDF library to be loaded
⚠️ Export quality directly impacts file size - balance quality vs file size needs


---

## Category-Based Location Filtering with Dynamic UI

### Context
Users need to filter MapsIndoors locations by categories dynamically, with a UI that shows category counts and allows multiple category selection. This is essential for large venues with many different types of locations.

### Industry
retail

### Problem
Users need to filter large numbers of locations by category type with an intuitive UI showing counts and multiple selection capability

### Solution
```javascript
// MapsIndoors Category-Based Location Filtering with Dynamic UI
class CategoryLocationFilter {
    constructor(mapsIndoorsInstance) {
        this.mapsIndoors = mapsIndoorsInstance;
        this.categories = new Map();
        this.locations = new Map();
        this.activeFilters = new Set();
        this.filterContainer = null;
        
        this.initializeFilterUI();
    }

    async initialize() {
        await this.loadCategories();
        await this.loadLocations();
        this.renderFilterButtons();
        this.setupEventListeners();
    }

    async loadCategories() {
        try {
            // Use Integration API to fetch categories
            const response = await fetch(`https://integration.mapsindoors.com/${this.getSolutionId()}/api/categories`, {
                headers: { 'accept': 'application/json' }
            });
            
            if (!response.ok) throw new Error('Failed to fetch categories');
            
            const rawCategories = await response.json();
            const categoriesArray = Object.values(rawCategories);
            
            // Sort categories alphabetically
            categoriesArray.sort((a, b) => {
                const nameA = a.name?.['en'] || a.name || a.key;
                const nameB = b.name?.['en'] || b.name || b.key;
                return nameA.localeCompare(nameB);
            });
            
            categoriesArray.forEach(category => {
                this.categories.set(category.key, {
                    key: category.key,
                    name: category.name?.['en'] || category.name || category.key,
                    iconUrl: category.iconUrl,
                    count: 0
                });
            });
            
            console.log(`Loaded ${this.categories.size} categories`);
        } catch (error) {
            console.error('Error loading categories:', error);
        }
    }

    async loadLocations() {
        try {
            const locations = await mapsindoors.services.LocationsService.getLocations({
                take: 1000 // Adjust based on solution size
            });
            
            // Group locations by category
            locations.forEach(location => {
                if (location.categories && location.categories.length > 0) {
                    location.categories.forEach(categoryKey => {
                        if (this.categories.has(categoryKey)) {
                            // Increment category count
                            const category = this.categories.get(categoryKey);
                            category.count++;
                            
                            // Store location with category reference
                            if (!this.locations.has(categoryKey)) {
                                this.locations.set(categoryKey, []);
                            }
                            this.locations.get(categoryKey).push(location);
                        }
                    });
                }
            });
            
            console.log(`Processed ${locations.length} locations across categories`);
        } catch (error) {
            console.error('Error loading locations:', error);
        }
    }

    initializeFilterUI() {
        // Create filter container
        this.filterContainer = document.createElement('div');
        this.filterContainer.className = 'category-filter-container';
        this.filterContainer.style.cssText = `
            position: absolute;
            top: 20px;
            left: 20px;
            max-width: 300px;
            background: white;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            z-index: 1000;
            max-height: 80vh;
            overflow-y: auto;
        `;
        
        // Add title
        const title = document.createElement('h3');
        title.textContent = 'Filter by Category';
        title.style.cssText = 'margin: 0 0 15px 0; font-size: 1.1rem; color: #333;';
        this.filterContainer.appendChild(title);
        
        // Add "Show All" button
        const showAllBtn = document.createElement('button');
        showAllBtn.textContent = 'Show All';
        showAllBtn.className = 'filter-btn show-all active';
        showAllBtn.style.cssText = this.getButtonStyles(true);
        showAllBtn.onclick = () => this.showAllLocations();
        this.filterContainer.appendChild(showAllBtn);
        
        // Add clear filters button
        const clearBtn = document.createElement('button');
        clearBtn.textContent = 'Clear Filters';
        clearBtn.className = 'filter-btn clear-filters';
        clearBtn.style.cssText = this.getButtonStyles(false) + 'margin-top: 10px; background-color: #f44336;';
        clearBtn.onclick = () => this.clearAllFilters();
        this.filterContainer.appendChild(clearBtn);
        
        document.body.appendChild(this.filterContainer);
    }

    renderFilterButtons() {
        // Remove existing category buttons (keep title and control buttons)
        const existingBtns = this.filterContainer.querySelectorAll('.category-btn');
        existingBtns.forEach(btn => btn.remove());
        
        // Add category filter buttons
        this.categories.forEach((category, key) => {
            if (category.count > 0) { // Only show categories with locations
                const button = document.createElement('button');
                button.className = 'filter-btn category-btn';
                button.dataset.category = key;
                
                button.innerHTML = `
                    <div style="display: flex; align-items: center; justify-content: space-between;">
                        <span>${category.name}</span>
                        <span style="background: rgba(0,0,0,0.1); padding: 2px 6px; border-radius: 10px; font-size: 0.8em;">
                            ${category.count}
                        </span>
                    </div>
                `;
                
                button.style.cssText = this.getButtonStyles(false);
                button.onclick = () => this.toggleCategoryFilter(key);
                
                this.filterContainer.appendChild(button);
            }
        });
    }

    getButtonStyles(isActive) {
        return `
            display: block;
            width: 100%;
            padding: 10px 12px;
            margin: 5px 0;
            border: 2px solid ${isActive ? '#1976D2' : '#e0e0e0'};
            border-radius: 6px;
            background-color: ${isActive ? '#1976D2' : 'white'};
            color: ${isActive ? 'white' : '#333'};
            cursor: pointer;
            font-size: 0.9rem;
            transition: all 0.2s ease;
            text-align: left;
        `;
    }

    setupEventListeners() {
        // Add hover effects
        this.filterContainer.addEventListener('mouseover', (e) => {
            if (e.target.classList.contains('filter-btn') && !e.target.classList.contains('active')) {
                e.target.style.backgroundColor = '#f5f5f5';
                e.target.style.borderColor = '#ccc';
            }
        });
        
        this.filterContainer.addEventListener('mouseout', (e) => {
            if (e.target.classList.contains('filter-btn') && !e.target.classList.contains('active')) {
                e.target.style.backgroundColor = 'white';
                e.target.style.borderColor = '#e0e0e0';
            }
        });
    }

    toggleCategoryFilter(categoryKey) {
        if (this.activeFilters.has(categoryKey)) {
            this.activeFilters.delete(categoryKey);
        } else {
            this.activeFilters.add(categoryKey);
        }
        
        this.updateFilterUI();
        this.applyLocationFilters();
    }

    showAllLocations() {
        this.activeFilters.clear();
        this.updateFilterUI();
        
        // Reset all location display rules to visible
        const allLocationIds = Array.from(this.locations.values())
            .flat()
            .map(location => location.id);
        
        this.mapsIndoors.setDisplayRule(allLocationIds, {
            visible: true,
            iconVisible: true,
            polygonVisible: true
        });
        
        console.log('Showing all locations');
    }

    clearAllFilters() {
        this.activeFilters.clear();
        this.updateFilterUI();
        this.applyLocationFilters();
    }

    updateFilterUI() {
        // Update button states
        const buttons = this.filterContainer.querySelectorAll('.filter-btn');
        
        buttons.forEach(button => {
            const isShowAll = button.classList.contains('show-all');
            const isClearBtn = button.classList.contains('clear-filters');
            const categoryKey = button.dataset.category;
            
            if (isShowAll) {
                const isActive = this.activeFilters.size === 0;
                button.classList.toggle('active', isActive);
                button.style.cssText = this.getButtonStyles(isActive);
            } else if (!isClearBtn && categoryKey) {
                const isActive = this.activeFilters.has(categoryKey);
                button.classList.toggle('active', isActive);
                button.style.cssText = this.getButtonStyles(isActive);
            }
        });
    }

    applyLocationFilters() {
        if (this.activeFilters.size === 0) {
            this.showAllLocations();
            return;
        }
        
        // Get locations for active categories
        const visibleLocationIds = new Set();
        const hiddenLocationIds = new Set();
        
        this.activeFilters.forEach(categoryKey => {
            if (this.locations.has(categoryKey)) {
                this.locations.get(categoryKey).forEach(location => {
                    visibleLocationIds.add(location.id);
                });
            }
        });
        
        // Get all location IDs to determine which to hide
        this.locations.forEach(categoryLocations => {
            categoryLocations.forEach(location => {
                if (!visibleLocationIds.has(location.id)) {
                    hiddenLocationIds.add(location.id);
                }
            });
        });
        
        // Apply display rules
        if (visibleLocationIds.size > 0) {
            this.mapsIndoors.setDisplayRule(Array.from(visibleLocationIds), {
                visible: true,
                iconVisible: true,
                polygonVisible: true,
                iconSize: { width: 32, height: 32 },
                zoomFrom: 15
            });
        }
        
        if (hiddenLocationIds.size > 0) {
            this.mapsIndoors.setDisplayRule(Array.from(hiddenLocationIds), {
                visible: false,
                iconVisible: false,
                polygonVisible: false
            });
        }
        
        // Fit bounds to visible locations if any are selected
        if (visibleLocationIds.size > 0) {
            this.fitBoundsToVisibleLocations(Array.from(visibleLocationIds));
        }
        
        console.log(`Applied filters: ${this.activeFilters.size} categories, ${visibleLocationIds.size} visible locations`);
    }

    fitBoundsToVisibleLocations(locationIds) {
        // Get bounds for visible locations
        const bounds = new mapboxgl.LngLatBounds();
        let hasValidBounds = false;
        
        this.locations.forEach(categoryLocations => {
            categoryLocations.forEach(location => {
                if (locationIds.includes(location.id)) {
                    const coords = location.properties.anchor.coordinates;
                    bounds.extend([coords[0], coords[1]]);
                    hasValidBounds = true;
                }
            });
        });
        
        if (hasValidBounds) {
            // Get mapbox instance through MapsIndoors
            const mapboxInstance = this.mapsIndoors.getMapView().getMap();
            mapboxInstance.fitBounds(bounds, {
                padding: 50,
                duration: 1000
            });
        }
    }

    getActiveCategoryNames() {
        return Array.from(this.activeFilters).map(key => {
            const category = this.categories.get(key);
            return category ? category.name : key;
        });
    }

    getSolutionId() {
        // Extract solution ID from MapsIndoors instance if available
        // This is a placeholder - you'll need to provide your actual solution ID
        return 'your-solution-id-here';
    }

    exportFilterState() {
        return {
            activeFilters: Array.from(this.activeFilters),
            categories: Array.from(this.categories.entries()),
            timestamp: new Date().toISOString()
        };
    }

    importFilterState(filterState) {
        if (filterState.activeFilters) {
            this.activeFilters = new Set(filterState.activeFilters);
            this.updateFilterUI();
            this.applyLocationFilters();
        }
    }
}

// Usage Example
let categoryFilter;

mapsIndoorsInstance.addListener('ready', async () => {
    categoryFilter = new CategoryLocationFilter(mapsIndoorsInstance);
    await categoryFilter.initialize();
    
    console.log('Category filter system initialized');
    
    // Optional: Set up keyboard shortcuts
    document.addEventListener('keydown', (e) => {
        if (e.key === 'Escape') {
            categoryFilter.clearAllFilters();
        } else if (e.key === 'a' && e.ctrlKey) {
            e.preventDefault();
            categoryFilter.showAllLocations();
        }
    });
});

// Helper function to create search input for filtering categories
function addCategorySearch(filterInstance) {
    const searchInput = document.createElement('input');
    searchInput.type = 'text';
    searchInput.placeholder = 'Search categories...';
    searchInput.style.cssText = `
        width: 100%;
        padding: 8px 12px;
        margin-bottom: 10px;
        border: 1px solid #ddd;
        border-radius: 4px;
        font-size: 0.9rem;
    `;
    
    searchInput.addEventListener('input', (e) => {
        const searchTerm = e.target.value.toLowerCase();
        const categoryButtons = filterInstance.filterContainer.querySelectorAll('.category-btn');
        
        categoryButtons.forEach(button => {
            const categoryName = button.textContent.toLowerCase();
            const shouldShow = categoryName.includes(searchTerm);
            button.style.display = shouldShow ? 'block' : 'none';
        });
    });
    
    // Insert after title
    const title = filterInstance.filterContainer.querySelector('h3');
    title.after(searchInput);
}
```

### Explanation
This code creates a complete category-based filtering system for MapsIndoors locations. It fetches categories via the Integration API, counts locations per category, and provides an interactive UI with filter buttons. Key features include: 1) Dynamic category loading with location counts, 2) Multiple category selection with visual feedback, 3) Show all/clear filters functionality, 4) Automatic bounds fitting to visible locations, 5) Export/import filter states for saving user preferences. The system uses display rules to show/hide locations based on selected categories.

### Use Cases
- Shopping mall store type filtering
- Hospital department filtering
- Airport amenity filtering
- Office space type filtering
- Educational facility room filtering

### Important Notes
⚠️ Categories must be properly configured in the MapsIndoors CMS
⚠️ Locations need to have categories assigned for filtering to work
⚠️ Large numbers of locations may impact performance
⚠️ Solution ID must be provided for Integration API access
⚠️ Display rules are applied in bulk for better performance


---

## Real-time Location Updates with WebSocket Integration

### Context
Organizations need real-time location updates for IoT devices, people tracking, or dynamic content that changes frequently. This requires WebSocket integration with MapsIndoors to show live data without page refreshes.

### Industry
healthcare

### Problem
Need real-time updates of device locations, people positions, or dynamic content on MapsIndoors maps through WebSocket connections

### Solution
```javascript
// Real-time Location Updates with WebSocket Integration
class MapsIndoorsRealtimeUpdater {
    constructor(mapsIndoorsInstance) {
        this.mapsIndoors = mapsIndoorsInstance;
        this.websocket = null;
        this.locationMarkers = new Map();
        this.updateQueue = [];
        this.isProcessingQueue = false;
        this.reconnectAttempts = 0;
        this.maxReconnectAttempts = 5;
        this.reconnectDelay = 1000;
        
        this.statusCallbacks = new Set();
        this.locationUpdateCallbacks = new Set();
    }

    // Connect to WebSocket server for real-time updates
    connect(websocketUrl, options = {}) {
        const {
            heartbeatInterval = 30000,
            autoReconnect = true,
            apiKey = null
        } = options;

        try {
            // Add API key to URL if provided
            const wsUrl = apiKey ? `${websocketUrl}?apiKey=${apiKey}` : websocketUrl;
            
            this.websocket = new WebSocket(wsUrl);
            this.autoReconnect = autoReconnect;
            
            this.websocket.onopen = () => {
                console.log('Connected to real-time location updates');
                this.reconnectAttempts = 0;
                this.notifyStatusCallbacks('connected');
                
                // Send heartbeat to keep connection alive
                if (heartbeatInterval > 0) {
                    this.startHeartbeat(heartbeatInterval);
                }
            };

            this.websocket.onmessage = (event) => {
                try {
                    const message = JSON.parse(event.data);
                    this.handleIncomingMessage(message);
                } catch (error) {
                    console.error('Error parsing WebSocket message:', error);
                }
            };

            this.websocket.onclose = (event) => {
                console.log('WebSocket connection closed:', event.code, event.reason);
                this.notifyStatusCallbacks('disconnected');
                
                if (this.autoReconnect && this.reconnectAttempts < this.maxReconnectAttempts) {
                    setTimeout(() => {
                        this.reconnectAttempts++;
                        console.log(`Reconnection attempt ${this.reconnectAttempts}/${this.maxReconnectAttempts}`);
                        this.connect(websocketUrl, options);
                    }, this.reconnectDelay * this.reconnectAttempts);
                }
            };

            this.websocket.onerror = (error) => {
                console.error('WebSocket error:', error);
                this.notifyStatusCallbacks('error', error);
            };

        } catch (error) {
            console.error('Failed to establish WebSocket connection:', error);
            this.notifyStatusCallbacks('error', error);
        }
    }

    // Handle different types of incoming messages
    handleIncomingMessage(message) {
        switch (message.type) {
            case 'location_update':
                this.queueLocationUpdate(message.data);
                break;
            case 'location_batch':
                message.data.forEach(update => this.queueLocationUpdate(update));
                break;
            case 'status_update':
                this.handleStatusUpdate(message.data);
                break;
            case 'heartbeat':
                // Respond to server heartbeat
                this.sendMessage({ type: 'heartbeat_response' });
                break;
            default:
                console.log('Unknown message type:', message.type);
        }
    }

    // Queue location updates to prevent overwhelming the map
    queueLocationUpdate(updateData) {
        this.updateQueue.push({
            ...updateData,
            timestamp: Date.now()
        });
        
        if (!this.isProcessingQueue) {
            this.processUpdateQueue();
        }
    }

    // Process queued updates in batches
    async processUpdateQueue() {
        if (this.isProcessingQueue || this.updateQueue.length === 0) {
            return;
        }

        this.isProcessingQueue = true;
        
        try {
            // Process updates in batches of 10
            const batchSize = 10;
            while (this.updateQueue.length > 0) {
                const batch = this.updateQueue.splice(0, batchSize);
                
                await Promise.all(batch.map(update => this.processLocationUpdate(update)));
                
                // Small delay between batches to prevent overwhelming the map
                if (this.updateQueue.length > 0) {
                    await new Promise(resolve => setTimeout(resolve, 100));
                }
            }
        } finally {
            this.isProcessingQueue = false;
        }
    }

    // Process a single location update
    async processLocationUpdate(updateData) {
        const {
            locationId,
            coordinates,
            status,
            properties,
            action = 'update'
        } = updateData;

        try {
            switch (action) {
                case 'create':
                    await this.createLocation(updateData);
                    break;
                case 'update':
                    await this.updateLocation(locationId, updateData);
                    break;
                case 'delete':
                    await this.deleteLocation(locationId);
                    break;
                default:
                    console.warn('Unknown action:', action);
            }
            
            // Notify location update callbacks
            this.notifyLocationUpdateCallbacks(updateData);
            
        } catch (error) {
            console.error('Error processing location update:', error);
        }
    }

    // Create new location with real-time data
    async createLocation(data) {
        const { locationId, coordinates, properties, status } = data;
        
        // Create marker for new location
        const marker = new mapboxgl.Marker({
            color: this.getStatusColor(status)
        })
        .setLngLat([coordinates.lng, coordinates.lat])
        .addTo(this.getMapboxInstance());

        // Add popup with location info
        const popup = new mapboxgl.Popup({ offset: 25 })
            .setHTML(`
                <div style="padding: 10px;">
                    <h4>${properties.name || 'Unknown Location'}</h4>
                    <p>Status: ${status || 'Unknown'}</p>
                    <p>Last Update: ${new Date().toLocaleTimeString()}</p>
                </div>
            `);
        
        marker.setPopup(popup);
        
        this.locationMarkers.set(locationId, {
            marker,
            popup,
            data: data,
            lastUpdate: Date.now()
        });

        console.log(`Created new location: ${locationId}`);
    }

    // Update existing location
    async updateLocation(locationId, data) {
        const existingLocation = this.locationMarkers.get(locationId);
        
        if (existingLocation) {
            // Update marker position if coordinates changed
            if (data.coordinates) {
                existingLocation.marker.setLngLat([data.coordinates.lng, data.coordinates.lat]);
            }
            
            // Update marker color if status changed
            if (data.status) {
                const newColor = this.getStatusColor(data.status);
                existingLocation.marker.getElement().style.backgroundColor = newColor;
            }
            
            // Update popup content
            if (data.properties || data.status) {
                const properties = { ...existingLocation.data.properties, ...data.properties };
                existingLocation.popup.setHTML(`
                    <div style="padding: 10px;">
                        <h4>${properties.name || 'Unknown Location'}</h4>
                        <p>Status: ${data.status || existingLocation.data.status}</p>
                        <p>Last Update: ${new Date().toLocaleTimeString()}</p>
                    </div>
                `);
            }
            
            // Update stored data
            existingLocation.data = { ...existingLocation.data, ...data };
            existingLocation.lastUpdate = Date.now();
            
        } else {
            // Location doesn't exist, try to get it from MapsIndoors
            try {
                const location = await mapsindoors.services.LocationsService.getLocation(locationId);
                if (location) {
                    // Apply display rule update
                    this.mapsIndoors.setDisplayRule(locationId, {
                        visible: true,
                        iconVisible: data.status !== 'hidden',
                        polygonFillColor: this.getStatusColor(data.status),
                        polygonVisible: true,
                        polygonFillOpacity: 0.6
                    });
                }
            } catch (error) {
                console.warn(`Location ${locationId} not found in MapsIndoors:`, error);
            }
        }
    }

    // Delete location
    async deleteLocation(locationId) {
        const existingLocation = this.locationMarkers.get(locationId);
        
        if (existingLocation) {
            existingLocation.marker.remove();
            this.locationMarkers.delete(locationId);
            console.log(`Deleted location: ${locationId}`);
        } else {
            // Hide location in MapsIndoors if it exists
            this.mapsIndoors.setDisplayRule(locationId, {
                visible: false,
                iconVisible: false,
                polygonVisible: false
            });
        }
    }

    // Get color based on status
    getStatusColor(status) {
        const statusColors = {
            'online': '#4CAF50',
            'offline': '#f44336',
            'warning': '#ff9800',
            'maintenance': '#9e9e9e',
            'active': '#2196f3',
            'inactive': '#607d8b'
        };
        
        return statusColors[status] || '#666666';
    }

    // Send message to WebSocket server
    sendMessage(message) {
        if (this.websocket && this.websocket.readyState === WebSocket.OPEN) {
            this.websocket.send(JSON.stringify(message));
        } else {
            console.warn('WebSocket not connected, cannot send message');
        }
    }

    // Start heartbeat to keep connection alive
    startHeartbeat(interval) {
        this.heartbeatInterval = setInterval(() => {
            this.sendMessage({ type: 'heartbeat' });
        }, interval);
    }

    // Stop heartbeat
    stopHeartbeat() {
        if (this.heartbeatInterval) {
            clearInterval(this.heartbeatInterval);
            this.heartbeatInterval = null;
        }
    }

    // Handle status updates
    handleStatusUpdate(statusData) {
        console.log('Received status update:', statusData);
        this.notifyStatusCallbacks('status_update', statusData);
    }

    // Event callback management
    onStatusChange(callback) {
        this.statusCallbacks.add(callback);
        return () => this.statusCallbacks.delete(callback);
    }

    onLocationUpdate(callback) {
        this.locationUpdateCallbacks.add(callback);
        return () => this.locationUpdateCallbacks.delete(callback);
    }

    notifyStatusCallbacks(status, data = null) {
        this.statusCallbacks.forEach(callback => {
            try {
                callback(status, data);
            } catch (error) {
                console.error('Error in status callback:', error);
            }
        });
    }

    notifyLocationUpdateCallbacks(updateData) {
        this.locationUpdateCallbacks.forEach(callback => {
            try {
                callback(updateData);
            } catch (error) {
                console.error('Error in location update callback:', error);
            }
        });
    }

    // Get Mapbox instance from MapsIndoors
    getMapboxInstance() {
        return this.mapsIndoors.getMapView().getMap();
    }

    // Request location data from server
    requestLocationData(filters = {}) {
        this.sendMessage({
            type: 'request_locations',
            data: filters
        });
    }

    // Subscribe to specific location updates
    subscribeToLocation(locationId) {
        this.sendMessage({
            type: 'subscribe',
            data: { locationId }
        });
    }

    // Unsubscribe from location updates
    unsubscribeFromLocation(locationId) {
        this.sendMessage({
            type: 'unsubscribe',
            data: { locationId }
        });
    }

    // Get connection status
    getConnectionStatus() {
        if (!this.websocket) return 'not_connected';
        
        switch (this.websocket.readyState) {
            case WebSocket.CONNECTING:
                return 'connecting';
            case WebSocket.OPEN:
                return 'connected';
            case WebSocket.CLOSING:
                return 'closing';
            case WebSocket.CLOSED:
                return 'closed';
            default:
                return 'unknown';
        }
    }

    // Clean up and disconnect
    disconnect() {
        this.autoReconnect = false;
        this.stopHeartbeat();
        
        if (this.websocket) {
            this.websocket.close();
            this.websocket = null;
        }
        
        // Clean up markers
        this.locationMarkers.forEach(({ marker }) => {
            marker.remove();
        });
        this.locationMarkers.clear();
        
        console.log('Real-time updater disconnected');
    }

    // Get statistics
    getStats() {
        return {
            connectionStatus: this.getConnectionStatus(),
            activeMarkers: this.locationMarkers.size,
            queueLength: this.updateQueue.length,
            reconnectAttempts: this.reconnectAttempts,
            isProcessingQueue: this.isProcessingQueue
        };
    }
}

// Usage Example
let realtimeUpdater;

mapsIndoorsInstance.addListener('ready', () => {
    realtimeUpdater = new MapsIndoorsRealtimeUpdater(mapsIndoorsInstance);
    
    // Set up event listeners
    realtimeUpdater.onStatusChange((status, data) => {
        console.log('Connection status changed:', status, data);
        updateConnectionUI(status);
    });
    
    realtimeUpdater.onLocationUpdate((updateData) => {
        console.log('Location updated:', updateData);
        // Handle location update in your UI
    });
    
    // Connect to WebSocket server
    realtimeUpdater.connect('wss://your-websocket-server.com/locations', {
        heartbeatInterval: 30000,
        autoReconnect: true,
        apiKey: 'your-api-key'
    });
});

// Example UI update function
function updateConnectionUI(status) {
    const statusIndicator = document.getElementById('connection-status');
    if (statusIndicator) {
        statusIndicator.textContent = status;
        statusIndicator.className = `status-indicator ${status}`;
    }
}

// Example cleanup on page unload
window.addEventListener('beforeunload', () => {
    if (realtimeUpdater) {
        realtimeUpdater.disconnect();
    }
});

// Mock WebSocket server message examples for testing:
/*
// Location update message
{
    "type": "location_update",
    "data": {
        "locationId": "device-123",
        "coordinates": { "lat": 30.3603212, "lng": -97.7422623 },
        "status": "online",
        "properties": { "name": "Temperature Sensor 1", "temperature": 72 },
        "action": "update"
    }
}

// Batch update message
{
    "type": "location_batch",
    "data": [
        {
            "locationId": "device-124",
            "coordinates": { "lat": 30.3604212, "lng": -97.7423623 },
            "status": "warning",
            "action": "update"
        },
        {
            "locationId": "device-125",
            "coordinates": { "lat": 30.3605212, "lng": -97.7424623 },
            "status": "offline",
            "action": "update"
        }
    ]
}
*/
```

### Explanation
This code creates a complete real-time location update system using WebSockets. Key features include: 1) WebSocket connection with auto-reconnect and heartbeat, 2) Queued update processing to prevent map overwhelming, 3) Support for create/update/delete operations, 4) Dynamic marker creation and updates with status-based colors, 5) Event callback system for status and location changes, 6) Integration with MapsIndoors display rules for existing locations. The system handles connection failures gracefully and provides statistics for monitoring.

### Use Cases
- IoT device tracking
- People location monitoring
- Equipment status updates
- Emergency response tracking
- Delivery and logistics monitoring

### Important Notes
⚠️ WebSocket server must send properly formatted JSON messages
⚠️ Large volumes of updates can overwhelm the map rendering
⚠️ Connection failures require proper reconnection logic
⚠️ Markers should be cleaned up to prevent memory leaks
⚠️ Status colors should be consistent with your application design




---

## MapsIndoors Venues Service Tester - No Map Required

### Context
Developer needed a lightweight way to test MapsIndoors API keys and see what venues are available in their solution without the overhead of initializing maps and complex UI components


### Problem
Need to quickly test MapsIndoors API keys and explore available venues without setting up a full map interface

### Solution
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MapsIndoors Venues Tester</title>
    <!-- MapsIndoors SDK -->
    <script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz"></script>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }

        .container {
            background-color: white;
            border-radius: 8px;
            padding: 30px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }

        .input-group {
            margin-bottom: 20px;
        }

        .input-group label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
            color: #333;
        }

        .input-group input {
            width: 100%;
            padding: 12px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
            box-sizing: border-box;
        }

        .btn {
            background-color: #4CAF50;
            color: white;
            padding: 12px 24px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            font-weight: bold;
            width: 100%;
        }

        .btn:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }

        .venue-card {
            background-color: white;
            border-radius: 6px;
            padding: 15px;
            margin-bottom: 15px;
            border-left: 4px solid #4CAF50;
            box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
        }

        .venue-name {
            font-weight: bold;
            font-size: 18px;
            color: #333;
            margin-bottom: 8px;
        }

        .venue-details {
            color: #666;
            font-size: 14px;
            line-height: 1.5;
        }

        .error {
            background-color: #ffebee;
            color: #c62828;
            padding: 15px;
            border-radius: 4px;
            border-left: 4px solid #f44336;
        }

        .raw-response pre {
            background-color: #f4f4f4;
            border: 1px solid #ddd;
            border-radius: 4px;
            padding: 15px;
            overflow-x: auto;
            white-space: pre-wrap;
            word-wrap: break-word;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>MapsIndoors Venues Tester</h1>
            <p>Enter your MapsIndoors API key to test the getVenues service</p>
        </div>

        <div class="input-group">
            <label for="apiKey">MapsIndoors API Key:</label>
            <input type="text" id="apiKey" placeholder="Enter your API key here..." value="mapspeople">
        </div>

        <button class="btn" id="testBtn" onclick="testGetVenues()">
            Test Get Venues
        </button>

        <div id="loading" style="display: none;">
            <p>Loading venues...</p>
        </div>

        <div id="results" style="display: none;">
            <h3>Results</h3>
            <div id="resultsContent">
                <!-- Results will be populated here -->
            </div>
        </div>
    </div>

    <script>
        function showLoading() {
            document.getElementById('loading').style.display = 'block';
            document.getElementById('results').style.display = 'none';
            document.getElementById('testBtn').disabled = true;
        }

        function hideLoading() {
            document.getElementById('loading').style.display = 'none';
            document.getElementById('testBtn').disabled = false;
        }

        function showResults(content) {
            document.getElementById('resultsContent').innerHTML = content;
            document.getElementById('results').style.display = 'block';
        }

        function showError(message) {
            const errorContent = `<div class="error"><strong>Error:</strong> ${message}</div>`;
            showResults(errorContent);
        }

        function showSuccess(venues, rawResponse) {
            let content = '';

            if (venues && venues.length > 0) {
                content += `<div class="success"><strong>Success!</strong> Found ${venues.length} venue${venues.length !== 1 ? 's' : ''}.</div>`;

                venues.forEach(venue => {
                    content += `
                        <div class="venue-card">
                            <div class="venue-name">${venue.name || 'Unnamed Venue'}</div>
                            <div class="venue-details">
                                <div><strong>ID:</strong> ${venue.id || 'N/A'}</div>
                                <div><strong>Country:</strong> ${venue.country || 'N/A'}</div>
                                <div><strong>City:</strong> ${venue.city || 'N/A'}</div>
                                <div><strong>Address:</strong> ${venue.address || 'N/A'}</div>
                                <div><strong>Default Floor:</strong> ${venue.defaultFloor !== undefined ? venue.defaultFloor : 'N/A'}</div>
                                <div><strong>Has Tiles:</strong> ${venue.tilesUrl ? 'Yes' : 'No'}</div>
                                ${venue.anchor ? `<div><strong>Coordinates:</strong> ${venue.anchor.coordinates ? venue.anchor.coordinates.join(', ') : 'N/A'}</div>` : ''}
                            </div>
                        </div>
                    `;
                });
            } else {
                content += `<div class="success"><strong>Success!</strong> API call completed but no venues found.</div>`;
            }

            content += `
                <div class="raw-response">
                    <h4>Raw API Response:</h4>
                    <pre>${JSON.stringify(rawResponse, null, 2)}</pre>
                </div>
            `;

            showResults(content);
        }

        async function testGetVenues() {
            const apiKey = document.getElementById('apiKey').value.trim();
            
            if (!apiKey) {
                showError('Please enter an API key');
                return;
            }

            showLoading();

            try {
                console.log('Testing venues service with API key:', apiKey);
                
                // Create a temporary script element with the API key
                const script = document.createElement('script');
                script.src = `https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz?apikey=${apiKey}`;
                document.head.appendChild(script);

                // Wait for script to load
                await new Promise((resolve, reject) => {
                    script.onload = resolve;
                    script.onerror = () => reject(new Error('Failed to load MapsIndoors SDK'));
                });

                // Test the venues service
                const venues = await mapsindoors.services.VenuesService.getVenues();
                console.log('Venues response:', venues);

                // Clean up
                document.head.removeChild(script);

                // Show results
                showSuccess(venues, venues);

            } catch (error) {
                console.error('Error testing venues:', error);
                
                let errorMessage = 'Unknown error occurred';
                
                if (error.message) {
                    errorMessage = error.message;
                } else if (error.status) {
                    errorMessage = `HTTP ${error.status}: ${error.statusText || 'Request failed'}`;
                }

                if (errorMessage.includes('401') || errorMessage.includes('Unauthorized')) {
                    errorMessage = 'Invalid API key or unauthorized access';
                } else if (errorMessage.includes('404')) {
                    errorMessage = 'API endpoint not found - check your API key';
                }

                showError(errorMessage);
            } finally {
                hideLoading();
            }
        }

        // Allow Enter key to trigger test
        document.getElementById('apiKey').addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                testGetVenues();
            }
        });
    </script>
</body>
</html>
```

### Explanation
This tool provides a simple HTML interface to test the MapsIndoors VenuesService.getVenues() method. It dynamically loads the MapsIndoors SDK with the provided API key and displays venue information in a user-friendly format. The tool shows both formatted venue cards and raw JSON responses for debugging purposes.

### Use Cases
- API key validation
- Venue discovery
- Quick data exploration
- Debugging venue data
- Demo for stakeholders

### Important Notes
⚠️ Must include API key in SDK URL for proper authentication
⚠️ Requires internet connection to load SDK
⚠️ Some venues may not have all properties populated
⚠️ Error handling should account for various API response scenarios


---

## AI-Powered Airport Navigation Assistant with MapsIndoors & Gemini Integration

### Context
Advanced conversational airport navigation system combining real-time location services with AI-powered natural language processing

### Industry
transportation

### Problem
Airport passengers need intuitive, conversational assistance to find amenities, calculate travel times, and navigate complex terminal layouts efficiently

### Solution
```javascript
// Key Implementation Patterns from AirportLLM

// 1. AI-Enhanced Query Processing with Venue Context
async function parseInstructionsFromGemini(query) {
  const prompt = `
    You are an AI assistant in San Francisco International Airport.
    Available location types: ${venueContext.types.join(", ")}
    Common categories: ${venueContext.categories.join(", ")}
    
    User query: "${query}"
    
    Determine query type: location_search, travel_time, distance, or directions
    Extract search parameters and format response as JSON
  `;
  
  const response = await fetch(GEMINI_API_ENDPOINT, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      contents: [{ parts: [{ text: prompt }] }],
      generationConfig: { temperature: 0.2, maxOutputTokens: 1024 }
    })
  });
  
  return JSON.parse(response.candidates[0].content.parts[0].text);
}

// 2. Intelligent Proximity Search with Distance Matrix
async function searchLocations(query, options = {}) {
  const baseParams = {
    venue: "76949746a5873f0090fd8d8e839d8726",
    take: options.take || 100,
    near: { lat: USER_POSITION.lat, lng: USER_POSITION.lng },
    radius: options.radius || 500,
    floor: options.floor || USER_POSITION.floor
  };
  
  // Smart search strategy: type-first, then text search
  let locations = await mapsindoors.services.LocationsService.getLocations({
    ...baseParams,
    types: [query]
  });
  
  if (!locations?.length) {
    locations = await mapsindoors.services.LocationsService.getLocations({
      ...baseParams,
      q: query
    });
  }
  
  // Enhanced with distance matrix calculations
  const origins = [`${USER_POSITION.lat},${USER_POSITION.lng},${USER_POSITION.floor}`];
  const destinations = locations.map(loc => {
    const coords = loc.properties.anchor.coordinates;
    return `${coords[1]},${coords[0]},${loc.properties.floor}`;
  });
  
  const matrix = await mapsindoors.services.DistanceMatrixService.getDistanceMatrix({
    graphId: "airport_graph",
    origins,
    destinations,
    travelMode: "walking"
  });
  
  return locations.map((location, index) => ({
    ...location,
    distance: matrix.rows[0].elements[index].distance?.value,
    duration: matrix.rows[0].elements[index].duration?.value
  })).sort((a, b) => (a.distance || Infinity) - (b.distance || Infinity));
}

// 3. Contextual Route Display with AI-Generated Instructions
async function showRouteToLocation(location) {
  const route = await miDirectionsService.getRoute({
    origin: {
      lat: USER_POSITION.lat,
      lng: USER_POSITION.lng,
      floor: USER_POSITION.floor
    },
    destination: {
      lat: location.properties.anchor.coordinates[1],
      lng: location.properties.anchor.coordinates[0],
      floor: location.properties.floor
    },
    graphId: "airport_graph",
    travelMode: "walking"
  });
  
  miDirectionsRenderer.setRoute(route);
  
  // AI-enhanced route communication
  addAIMessage(`
    Route to ${location.properties.name}: ${formatDistance(route.distance.value)}
    Estimated time: ${formatDuration(route.duration.value)}
    ${formatRouteInstructions(route)}
  `);
}

// 4. Dynamic Venue Context Loading for AI Prompts
async function fetchVenueContext() {
  const sampleLocations = await mapsindoors.services.LocationsService.getLocations({
    venue: "76949746a5873f0090fd8d8e839d8726",
    take: 200,
    near: { lat: USER_POSITION.lat, lng: USER_POSITION.lng }
  });
  
  const types = new Set();
  const categories = new Set();
  
  sampleLocations.forEach(location => {
    if (location.properties.type) types.add(location.properties.type);
    if (location.properties.categories) {
      Object.keys(location.properties.categories).forEach(cat => categories.add(cat));
    }
  });
  
  venueContext = {
    types: Array.from(types),
    categories: Array.from(categories)
  };
}

// 5. Multi-Modal Response Generation
async function processUserMessage(message) {
  const instructions = await parseInstructionsFromGemini(message);
  const locations = await searchLocations(instructions.searchQuery);
  
  if (instructions.queryType === "travel_time") {
    const target = locations[0];
    const time = formatDuration(target.duration);
    
    addAIMessage(`
      It will take approximately **${time}** to reach ${target.properties.name}.
      <div class="location-card" onclick="showRoute(${target.id})">
        ${target.properties.name} • ${formatDistance(target.distance)}
      </div>
    `);
    
    addResultMarker(target, 0);
  } else {
    // Display multiple options with interactive cards
    locations.slice(0, 5).forEach((loc, i) => {
      addResultMarker(loc, i);
      addLocationCard(loc, i);
    });
  }
}
```

### Explanation
This implementation showcases advanced integration between MapsIndoors spatial services and AI language models, creating a conversational navigation experience. The system uses Gemini AI to parse natural language queries and extract intent (finding locations vs. calculating travel time), then applies intelligent search strategies with proximity-based filtering. The Distance Matrix API provides accurate walking times and distances, while the AI generates contextual responses. Key innovations include dynamic venue context loading for better AI prompts, multi-modal response generation with interactive UI elements, and seamless integration between map visualization and conversational interface. The system handles complex queries like 'How long to the nearest coffee shop?' by combining location search, distance calculation, and route visualization in a single conversational flow.

### Use Cases
- Airport passenger assistance systems
- Transit hub navigation with AI agents
- Large venue wayfinding with natural language
- Accessibility-focused navigation systems
- Multi-language airport assistance platforms
- Emergency evacuation guidance systems

### Important Notes
⚠️ Graph ID must match venue routing configuration (airport_graph)
⚠️ Distance Matrix requires careful coordinate formatting (lat,lng,floor)
⚠️ AI prompts need venue-specific context for accurate query parsing
⚠️ Floor synchronization between user position and search results critical
⚠️ Environment variable injection required for production deployment
⚠️ Proximity search radius affects performance vs accuracy tradeoff



---

## Booking Integration with External Systems

### Context
Facilities requiring appointment scheduling and resource booking connected to external databases

### Industry
healthcare

### Problem
Users need to book rooms, equipment, or appointments through MapsIndoors while maintaining synchronization with external booking systems

### Solution
```javascript
// Initialize MapsIndoors with booking integration
const mapsIndoors = new MapsIndoors.MapsIndoors({
  apiKey: "your-api-key"
});

// Setup location selection for booking
const map = new MapsIndoors.Map({
  elementId: "map",
  mapsIndoors: mapsIndoors
});

// Configure booking service with external system
class BookingService {
  constructor(externalApiEndpoint, authToken) {
    this.apiEndpoint = externalApiEndpoint;
    this.authToken = authToken;
  }

  async checkAvailability(locationId, externalId, timeSlot) {
    try {
      const response = await fetch(`${this.apiEndpoint}/availability`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${this.authToken}`
        },
        body: JSON.stringify({
          external_id: externalId, // Critical: maps to external system
          time_slot: timeSlot,
          location_id: locationId
        })
      });
      return await response.json();
    } catch (error) {
      console.error("Availability check failed:", error);
      return { available: false, error: true };
    }
  }

  async createBooking(locationId, externalId, bookingData) {
    try {
      const response = await fetch(`${this.apiEndpoint}/bookings`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${this.authToken}`
        },
        body: JSON.stringify({
          external_id: externalId, // Key field for system integration
          location_id: locationId,
          user_id: bookingData.userId,
          start_time: bookingData.startTime,
          end_time: bookingData.endTime,
          booking_type: bookingData.type
        })
      });
      return await response.json();
    } catch (error) {
      console.error("Booking creation failed:", error);
      return { success: false, error: true };
    }
  }
}

// Initialize booking service
const bookingService = new BookingService(
  "https://your-external-system.com/api",
  "your-auth-token"
);

// Handle location selection and booking flow
map.on("location_selected", async (location) => {
  const externalId = location.properties.externalId; // Must be set in MapsIndoors
  
  if (!externalId) {
    console.warn("Location missing external ID - booking not available");
    return;
  }

  // Show booking interface
  const bookingUI = document.getElementById("booking-interface");
  bookingUI.style.display = "block";
  
  // Populate with location details
  document.getElementById("location-name").textContent = location.properties.name;
  
  // Check current availability
  const availability = await bookingService.checkAvailability(
    location.id,
    externalId,
    new Date().toISOString()
  );
  
  updateAvailabilityDisplay(availability);
});

// Handle booking submission
document.getElementById("submit-booking").addEventListener("click", async () => {
  const selectedLocation = map.getSelectedLocation();
  const externalId = selectedLocation.properties.externalId;
  
  const bookingData = {
    userId: getCurrentUser().id,
    startTime: document.getElementById("start-time").value,
    endTime: document.getElementById("end-time").value,
    type: document.getElementById("booking-type").value
  };
  
  const result = await bookingService.createBooking(
    selectedLocation.id,
    externalId,
    bookingData
  );
  
  if (result.success) {
    showBookingConfirmation(result.booking_id);
    // Update location display with booking status
    updateLocationBookingStatus(selectedLocation.id, "booked");
  } else {
    showBookingError(result.error);
  }
});

// Update location styling based on booking status
function updateLocationBookingStatus(locationId, status) {
  const displayRule = {
    id: `booking-status-${status}`,
    types: ["ROOM"],
    filter: {
      propertyName: "id",
      propertyValue: locationId
    },
    style: {
      fillColor: status === "booked" ? "#ff6b6b" : "#51cf66",
      strokeColor: "#333",
      strokeWidth: 2
    }
  };
  
  map.addDisplayRule(displayRule);
}

function updateAvailabilityDisplay(availability) {
  const statusElement = document.getElementById("availability-status");
  if (availability.available) {
    statusElement.textContent = "Available";
    statusElement.className = "status-available";
  } else {
    statusElement.textContent = "Unavailable";
    statusElement.className = "status-unavailable";
  }
}
```

### Explanation
This implementation integrates MapsIndoors with external booking systems using the external ID as the critical linking field. The external ID stored in location properties must match the identifier used in your external database (e.g., room_id in your booking system). The BookingService class handles API communication with any external system, while the MapsIndoors map provides the visual interface for location selection and booking status display.

### Use Cases
- Hospital room and equipment booking with EMR integration
- Conference room scheduling in corporate offices
- Classroom and lab booking in educational institutions
- Meeting space reservation in coworking facilities
- Equipment checkout in manufacturing facilities
- Service appointment scheduling in retail locations

### Important Notes
⚠️ External ID is mandatory: Every bookable location must have the externalId property set to match your external system identifier
⚠️ API authentication: Implement proper token management and refresh logic for external system access
⚠️ Real-time sync: Consider implementing webhooks or polling to keep booking status updated
⚠️ Error handling: Always provide fallback UI when external booking system is unavailable
⚠️ Timezone handling: Ensure consistent timezone handling between MapsIndoors and external system
⚠️ Permission validation: Verify user permissions both in MapsIndoors and external system before allowing bookings
⚠️ Display rule conflicts: Booking status styling may conflict with other display rules - use specific filters


---

## Corporate Room Booking Demo with Waitlist Feature

### Context
Built as a demo for a banking corporate office environment, focusing on professional aesthetics and waitlist functionality rather than direct booking. Uses actual MapsIndoors room data from the AUSTINOFFICE venue to create a realistic corporate room finder application.

### Industry
corporate

### Problem
Creating a polished corporate room booking application with waitlist functionality instead of traditional booking, suitable for banking/corporate environments

### Solution
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Corporate Room Finder</title>
    <!-- Mapbox CSS -->
    <link href="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.css" rel="stylesheet">
    <!-- Font Awesome for icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.2/css/all.min.css">
    <!-- Google Fonts - Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --primary-color: #0a4d7e;
            --primary-light: #d0e3f0;
            --secondary-color: #1c86c5;
            --tertiary-color: #e9f3fc;
            --accent-color: #f0b941;
            --success-color: #28a745;
            --warning-color: #ffc107;
            --danger-color: #dc3545;
            --gray-100: #f8f9fa;
            --gray-200: #e9ecef;
            --gray-300: #dee2e6;
            --gray-400: #ced4da;
            --gray-500: #adb5bd;
            --gray-600: #6c757d;
            --gray-700: #495057;
            --gray-800: #343a40;
            --gray-900: #212529;
            --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
            --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
            --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
            --radius-sm: 0.25rem;
            --radius-md: 0.5rem;
            --radius-lg: 0.75rem;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
            color: var(--gray-800);
            background-color: var(--gray-100);
            line-height: 1.5;
        }

        .app-container {
            display: flex;
            height: 100vh;
            max-height: 100vh;
            overflow: hidden;
        }

        .sidebar {
            flex: 0 0 400px;
            background-color: white;
            box-shadow: var(--shadow-md);
            z-index: 10;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        .sidebar-header {
            padding: 20px;
            background-color: var(--primary-color);
            color: white;
        }

        .sidebar-title {
            font-size: 1.5rem;
            font-weight: 600;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .search-container {
            padding: 20px;
            background-color: white;
            border-bottom: 1px solid var(--gray-300);
        }

        .search-input {
            width: 100%;
            padding: 12px 15px 12px 40px;
            border: 1px solid var(--gray-300);
            border-radius: var(--radius-md);
            font-size: 0.95rem;
            background-color: var(--gray-100);
            transition: all 0.2s ease;
        }

        .search-input:focus {
            outline: none;
            border-color: var(--secondary-color);
            background-color: white;
            box-shadow: 0 0 0 3px rgba(28, 134, 197, 0.15);
        }

        .room-list {
            flex: 1;
            overflow-y: auto;
            padding: 10px;
        }

        .room-card {
            background-color: white;
            border-radius: var(--radius-md);
            border: 1px solid var(--gray-300);
            padding: 15px;
            margin-bottom: 15px;
            cursor: pointer;
            transition: all 0.2s ease;
            box-shadow: var(--shadow-sm);
        }

        .room-card:hover {
            box-shadow: var(--shadow-md);
            border-color: var(--gray-400);
        }

        .room-card.selected {
            border-color: var(--primary-color);
            box-shadow: 0 0 0 2px rgba(10, 77, 126, 0.15);
        }

        .map-container {
            flex: 1;
            height: 100%;
            position: relative;
        }

        #map {
            width: 100%;
            height: 100%;
        }

        .floor-selector {
            position: absolute;
            right: 20px;
            top: 160px;
            background-color: white;
            box-shadow: var(--shadow-md);
            border-radius: var(--radius-md);
            overflow: hidden;
            z-index: 100;
        }

        .floor-btn {
            width: 40px;
            height: 40px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            border: none;
            background: white;
            border-bottom: 1px solid var(--gray-200);
            font-weight: 500;
            transition: all 0.2s ease;
        }

        .floor-btn:hover {
            background-color: var(--gray-100);
        }

        .floor-btn.active {
            background-color: var(--primary-color);
            color: white;
        }
    </style>
</head>
<body>
    <div class="app-container">
        <div class="sidebar">
            <!-- Sidebar content with room list -->
        </div>
        <div class="map-container">
            <div id="map"></div>
        </div>
    </div>

    <!-- Scripts -->
    <script src="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.js"></script>
    <script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.39.2/mapsindoors-4.39.2.js.gz?apikey=mapspeople"></script>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // Initialize MapsIndoors
            const mapViewOptions = {
                accessToken: 'YOUR_MAPBOX_TOKEN',
                element: document.getElementById('map'),
                center: { lat: 30.3603212, lng: -97.7422623 },
                zoom: 20,
                maxZoom: 22,
            };

            const mapViewInstance = new mapsindoors.mapView.MapboxView(mapViewOptions);
            const mapsIndoorsInstance = new mapsindoors.MapsIndoors({
                mapView: mapViewInstance,
            });
            const mapboxInstance = mapViewInstance.getMap();

            // Custom floor selector function
            function addCustomFloorSelector() {
                const floorSelectorContainer = document.createElement('div');
                floorSelectorContainer.className = 'floor-selector';
                document.querySelector('.map-container').appendChild(floorSelectorContainer);
                
                function updateFloorSelector() {
                    floorSelectorContainer.innerHTML = '';
                    const building = mapsIndoorsInstance.getBuilding();
                    if (!building || !building.floors) return;
                    
                    const currentFloor = mapsIndoorsInstance.getFloor();
                    const floors = Object.entries(building.floors)
                        .sort(([a], [b]) => parseInt(b) - parseInt(a));
                    
                    floors.forEach(([floorIndex, floorInfo]) => {
                        const button = document.createElement('button');
                        button.className = `floor-btn ${floorIndex == currentFloor ? 'active' : ''}`;
                        button.textContent = floorInfo.name || floorIndex;
                        button.onclick = () => {
                            mapsIndoorsInstance.setFloor(parseInt(floorIndex));
                        };
                        floorSelectorContainer.appendChild(button);
                    });
                }
                
                mapsIndoorsInstance.addListener('building_changed', updateFloorSelector);
                mapsIndoorsInstance.addListener('floor_changed', updateFloorSelector);
                setTimeout(updateFloorSelector, 1000);
            }

            // Load actual MapsIndoors rooms
            mapsIndoorsInstance.addListener('ready', async function() {
                addCustomFloorSelector();
                
                try {
                    // Get meeting rooms from specific venue
                    const rooms = await mapsindoors.services.LocationsService.getLocations({
                        venue: 'AUSTINOFFICE',
                        types: ['MeetingRoom', 'MeetingRoom Small', 'MeetingRoom Extra Small', 'MeetingRoom Medium', 'MeetingRoom Large'],
                        take: 100
                    });

                    // Set display rules for room highlighting
                    mapsIndoorsInstance.setDisplayRule(rooms.map(room => room.id), {
                        visible: true,
                        polygonVisible: true,
                        polygonFillColor: '#0a4d7e',
                        polygonFillOpacity: 0.3,
                        polygonStrokeColor: '#0a4d7e',
                        polygonStrokeOpacity: 0.8,
                        polygonStrokeWidth: 1,
                        zoomFrom: 16
                    });

                    // Handle room selection
                    mapsIndoorsInstance.addListener('click', (event) => {
                        if (event && event.id) {
                            const room = rooms.find(r => r.id === event.id);
                            if (room) {
                                // Highlight selected room
                                mapsIndoorsInstance.setDisplayRule(event.id, {
                                    visible: true,
                                    polygonVisible: true,
                                    polygonFillColor: '#f0b941',
                                    polygonFillOpacity: 0.5,
                                    polygonStrokeColor: '#f0b941',
                                    polygonStrokeOpacity: 1,
                                    polygonStrokeWidth: 2,
                                    zoomFrom: 16
                                });
                            }
                        }
                    });
                } catch (error) {
                    console.error('Error loading rooms:', error);
                }
            });
        });
    </script>
</body>
</html>
```

### Explanation
This code creates a professional room booking demo that integrates actual MapsIndoors location data with a polished UI suitable for corporate environments. Key features include:

1. **MapsIndoors Integration**: Uses real meeting room data from AUSTINOFFICE venue with specific room types
2. **Custom Floor Selector**: Clean, positioned floor selector that integrates with the overall design
3. **Room Display Rules**: Visual highlighting of rooms with different states (normal vs selected)
4. **Professional Styling**: Uses a corporate color scheme with proper spacing, shadows, and typography
5. **Responsive Layout**: Sidebar with room list and full-width map container
6. **Waitlist Functionality**: Instead of booking, users join waitlists for rooms (shown in full implementation)

The application fetches actual MapsIndoors meeting rooms and displays them as clickable polygons on the map. When clicked, rooms are highlighted with visual feedback. The custom floor selector provides intuitive navigation between building floors.

### Use Cases
- Corporate office room booking systems
- Banking facility management
- Enterprise space reservation
- Meeting room availability tracking
- Professional office wayfinding
- Corporate real estate management

### Important Notes
⚠️ Must use specific venue parameter ('AUSTINOFFICE') to get relevant room data
⚠️ Custom floor selector requires building_changed and floor_changed listeners for proper updates
⚠️ Display rules must be set after MapsIndoors ready event
⚠️ Room types should match actual types in the venue (MeetingRoom variations)
⚠️ MapboxView initialization requires valid Mapbox access token
⚠️ Floor selector positioning may need adjustment based on other map controls


---

## Bank Desk Booking System with MapsIndoors

### Context
Banking institutions need efficient desk booking systems for modern flexible work environments where employees don't have assigned permanent desks

### Industry
financial services

### Problem
Banks need a comprehensive desk booking system for employees to reserve workspaces, especially in modern flexible work environments where hot-desking is common.

### Solution
```javascript
class BankDeskBooking {
    constructor(mapsIndoorsInstance) {
        this.mapsIndoors = mapsIndoorsInstance;
        this.desks = [];
        this.selectedDeskId = null;
        
        this.availabilityColors = {
            'available': '#4CAF50',
            'busy': '#f44336',
            'selected': '#2196F3'
        };
    }

    async loadDesks() {
        // Load desk locations from MapsIndoors
        const locations = await mapsindoors.services.LocationsService.getLocations({
            types: ['Desk', 'Workstation'],
            venue: 'BANK_VENUE',
            take: 100
        });

        this.desks = locations.map(location => ({
            id: location.id,
            name: location.properties.name,
            floor: location.properties.floor,
            department: location.properties.department,
            availability: this.checkAvailability(location.id),
            equipment: this.getEquipment(location),
            timeSlots: this.generateTimeSlots()
        }));

        this.updateDeskDisplayRules();
        this.renderDeskList();
    }

    updateDeskDisplayRules() {
        // Visual feedback on map
        const availableDesks = this.desks.filter(desk => desk.availability === 'available').map(d => d.id);
        const busyDesks = this.desks.filter(desk => desk.availability === 'busy').map(d => d.id);
        
        this.mapsIndoors.setDisplayRule(availableDesks, {
            polygonVisible: true,
            polygonFillColor: this.availabilityColors.available,
            polygonFillOpacity: 0.4,
            polygonStrokeColor: this.availabilityColors.available,
            polygonStrokeOpacity: 0.8,
            polygonStrokeWidth: 1,
            zoomFrom: 16
        });

        this.mapsIndoors.setDisplayRule(busyDesks, {
            polygonVisible: true,
            polygonFillColor: this.availabilityColors.busy,
            polygonFillOpacity: 0.4,
            polygonStrokeColor: this.availabilityColors.busy,
            polygonStrokeOpacity: 0.8,
            polygonStrokeWidth: 1,
            zoomFrom: 16
        });
    }

    selectDesk(deskId) {
        this.selectedDeskId = deskId;
        const desk = this.desks.find(d => d.id === deskId);
        if (!desk) return;

        // Highlight selected desk
        this.mapsIndoors.setDisplayRule(deskId, {
            polygonVisible: true,
            polygonFillColor: this.availabilityColors.selected,
            polygonFillOpacity: 0.6,
            polygonStrokeColor: this.availabilityColors.selected,
            polygonStrokeOpacity: 1,
            polygonStrokeWidth: 2,
            zoomFrom: 16
        });

        // Center map on desk
        this.mapsIndoors.setFloor(desk.floor);
        this.showDeskDetails(desk);
    }

    async submitBooking(bookingData) {
        const { deskId, date, timeSlot, purpose } = bookingData;
        
        // In real implementation, integrate with booking API
        const booking = {
            id: Date.now().toString(),
            deskId,
            date,
            timeSlot,
            purpose,
            status: 'confirmed',
            timestamp: new Date()
        };

        // Update desk availability
        const desk = this.desks.find(d => d.id === deskId);
        if (desk && desk.timeSlots[timeSlot]) {
            desk.timeSlots[timeSlot] = 'busy';
            desk.availability = 'busy';
            this.updateDeskDisplayRules();
        }

        return booking;
    }
}

// Usage
mapsIndoorsInstance.addListener('ready', () => {
    const bookingSystem = new BankDeskBooking(mapsIndoorsInstance);
    bookingSystem.loadDesks();
});
```

### Explanation
This bank desk booking system provides a complete solution for workspace reservation in banking environments. It uses MapsIndoors to visualize desk availability with color-coded polygons, allows filtering by department/floor, and handles booking workflows. The system integrates visual feedback with the map, showing available (green), busy (red), and selected (blue) desks.

### Use Cases
- Hot-desking in bank branches
- Flexible workspace management
- Department-specific desk allocation
- Meeting room-style booking for individual desks
- Employee workspace planning
- Office capacity management

### Important Notes
⚠️ Ensure real-time availability updates to prevent double bookings
⚠️ Consider integration with Active Directory for employee authentication
⚠️ Handle time zone considerations for multi-location banks
⚠️ Implement booking cancellation policies
⚠️ Consider desk equipment/accessibility requirements
⚠️ Plan for peak booking periods (Monday mornings, post-holidays)


---

## MapsIndoors Web SDK Filtering Approaches - No setFilter Method Available

### Context
Developer was asking about filtering capabilities in MapsIndoors Web SDK, specifically looking for a setFilter method similar to other mapping libraries. This solution provides comprehensive alternatives since no direct setFilter method exists.

### Industry
corporate

### Problem
The MapsIndoors Web SDK does not have a direct setFilter() method like other mapping SDKs. Developers need alternative approaches to filter and display locations based on various criteria such as type, floor, capacity, or custom properties.

### Solution
```javascript
// 1. SERVER-SIDE FILTERING via LocationsService (Most Efficient)
// Filter by type
const meetingRooms = await mapsindoors.services.LocationsService.getLocations({
    types: ['MeetingRoom', 'MeetingRoom Small', 'MeetingRoom Large'],
    venue: 'YOUR_VENUE_ID',
    take: 100
});

// Filter by categories
const filteredByCategory = await mapsindoors.services.LocationsService.getLocations({
    categories: ['category-key-1', 'category-key-2'],
    take: 50
});

// Filter by search query
const searchResults = await mapsindoors.services.LocationsService.getLocations({
    q: 'conference room',
    venue: 'YOUR_VENUE_ID'
});

// Filter by floor
const floorLocations = await mapsindoors.services.LocationsService.getLocations({
    floor: 2,
    venue: 'YOUR_VENUE_ID'
});

// Filter by proximity
const nearbyLocations = await mapsindoors.services.LocationsService.getLocations({
    near: { lat: 30.3603212, lng: -97.7422623 },
    radius: 100, // meters
    take: 10
});

// 2. VISUAL FILTERING via Display Rules
// Hide/show specific locations
mapsIndoorsInstance.setDisplayRule(['location-id-1', 'location-id-2'], {
    visible: false
});

// Hide all locations of certain types
const allMeetingRooms = await mapsindoors.services.LocationsService.getLocations({
    types: ['MeetingRoom']
});
const roomIds = allMeetingRooms.map(room => room.id);
mapsIndoorsInstance.setDisplayRule(roomIds, {
    visible: false
});

// 3. CLIENT-SIDE FILTERING with JavaScript
// Get all locations then filter client-side
const allLocations = await mapsindoors.services.LocationsService.getLocations({
    venue: 'YOUR_VENUE_ID',
    take: 1000
});

// Filter by capacity (custom property)
const largeRooms = allLocations.filter(location => {
    const capacity = location.properties.capacity;
    return capacity && parseInt(capacity) > 10;
});

// Filter by name pattern
const conferenceRooms = allLocations.filter(location => 
    location.properties.name.toLowerCase().includes('conference')
);

// 4. CUSTOM FILTER CLASS (Comprehensive Solution)
class LocationFilter {
    constructor(mapsIndoorsInstance) {
        this.mapsIndoors = mapsIndoorsInstance;
        this.allLocations = [];
        this.filteredLocations = [];
    }

    async initialize() {
        // Load all locations
        this.allLocations = await mapsindoors.services.LocationsService.getLocations({
            venue: 'YOUR_VENUE_ID',
            take: 1000
        });
    }

    filterByType(types) {
        this.filteredLocations = this.allLocations.filter(location => 
            types.includes(location.properties.type)
        );
        this.updateDisplay();
    }

    filterByFloor(floor) {
        this.filteredLocations = this.allLocations.filter(location => 
            location.properties.floor === floor
        );
        this.updateDisplay();
    }

    filterByCapacity(minCapacity, maxCapacity) {
        this.filteredLocations = this.allLocations.filter(location => {
            const capacity = parseInt(location.properties.capacity || 0);
            return capacity >= minCapacity && capacity <= maxCapacity;
        });
        this.updateDisplay();
    }

    filterByMultipleCriteria(criteria) {
        this.filteredLocations = this.allLocations.filter(location => {
            let matches = true;
            
            if (criteria.types && !criteria.types.includes(location.properties.type)) {
                matches = false;
            }
            
            if (criteria.floor && location.properties.floor !== criteria.floor) {
                matches = false;
            }
            
            if (criteria.searchTerm && !location.properties.name.toLowerCase().includes(criteria.searchTerm.toLowerCase())) {
                matches = false;
            }
            
            if (criteria.minCapacity) {
                const capacity = parseInt(location.properties.capacity || 0);
                if (capacity < criteria.minCapacity) matches = false;
            }
            
            return matches;
        });
        this.updateDisplay();
    }

    updateDisplay() {
        // Hide all locations first
        const allIds = this.allLocations.map(loc => loc.id);
        this.mapsIndoors.setDisplayRule(allIds, { visible: false });
        
        // Show filtered locations
        const filteredIds = this.filteredLocations.map(loc => loc.id);
        this.mapsIndoors.setDisplayRule(filteredIds, { visible: true });
    }

    clearFilter() {
        const allIds = this.allLocations.map(loc => loc.id);
        this.mapsIndoors.setDisplayRule(allIds, { visible: true });
        this.filteredLocations = [...this.allLocations];
    }

    getFilteredLocations() {
        return this.filteredLocations;
    }
}

// USAGE EXAMPLE
const filter = new LocationFilter(mapsIndoorsInstance);
await filter.initialize();

// Filter by type
filter.filterByType(['MeetingRoom', 'MeetingRoom Large']);

// Filter by multiple criteria
filter.filterByMultipleCriteria({
    types: ['MeetingRoom'],
    floor: 2,
    searchTerm: 'conference',
    minCapacity: 8
});

// Clear all filters
filter.clearFilter();
```

### Explanation
The MapsIndoors Web SDK doesn't provide a setFilter() method, but offers four main filtering approaches:

1. **Server-side filtering** using LocationsService.getLocations() with parameters like types, categories, floor, search query (q), and proximity (near/radius). This is the most efficient approach as it reduces data transfer.

2. **Visual filtering** using setDisplayRule() to hide/show locations on the map without re-fetching data. Good for interactive filtering.

3. **Client-side filtering** using JavaScript array methods after fetching all locations. Useful for complex custom criteria but requires loading all data first.

4. **Custom filter classes** that combine multiple approaches for comprehensive filtering systems.

The LocationFilter class example provides a reusable solution that loads all locations once, then applies various filters client-side while updating the visual display accordingly.

### Use Cases
- Interactive room booking systems where users filter by capacity, floor, or amenities
- Wayfinding applications that show only relevant location types
- Facility management dashboards with dynamic location filtering
- Emergency response systems filtering by room status or type
- Corporate space management with complex filtering criteria

### Important Notes
⚠️ No direct setFilter method exists in MapsIndoors Web SDK
⚠️ LocationsService.getLocations() has a 'take' parameter limit - use multiple calls for large datasets
⚠️ setDisplayRule() only controls visibility, doesn't remove locations from memory
⚠️ Client-side filtering requires loading all data first, which may impact performance
⚠️ Always use venue parameter in LocationsService calls for better performance
⚠️ Floor parameter in LocationsService is a number, not a string
⚠️ Display rules are applied per location ID, so batch operations are more efficient


---

## Dynamic 2D/3D View Switcher with Smooth Wall Extrusion

### Context
This solution addresses the need for intuitive 2D/3D mode switching in indoor mapping applications where users expect smooth visual transitions rather than abrupt mode changes. It's particularly valuable for applications where users need to examine floor plans in detail (2D) and then get spatial context through 3D visualization without jarring transitions.

### Industry
Multi-Industry

### Problem
Creating a smooth transition between 2D and 3D viewing modes in MapsIndoors based on camera tilt angle, with dynamic wall height extrusion that grows progressively as users tilt the map from top-down to perspective view

### Solution
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MapsIndoors Dynamic 2D/3D View Switcher with Wall Extrusion</title>
    
    <!-- Mapbox CSS (load first) -->
    <link href="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.css" rel="stylesheet">
    
    <!-- Mapbox JavaScript (load before MapsIndoors) -->
    <script src="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.js"></script>
    
    <!-- MapsIndoors JavaScript (load after Mapbox) -->
    <script src="https://app.mapsindoors.com/mapsindoors/js/sdk/4.41.1/mapsindoors-4.41.1.js.gz?apikey=mapspeople3d"></script>
    
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

        .view-indicator {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(255, 255, 255, 0.9);
            padding: 10px 15px;
            border-radius: 5px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            z-index: 1000;
            font-weight: bold;
            transition: all 0.3s ease;
        }

        .view-indicator.view-2d {
            border-left: 4px solid #4CAF50;
            color: #4CAF50;
        }

        .view-indicator.view-transition {
            border-left: 4px solid #FF9800;
            color: #FF9800;
        }

        .view-indicator.view-3d {
            border-left: 4px solid #FF5722;
            color: #FF5722;
        }

        .controls {
            position: absolute;
            top: 20px;
            right: 20px;
            background: rgba(255, 255, 255, 0.9);
            padding: 15px;
            border-radius: 5px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            z-index: 1000;
            min-width: 250px;
        }

        .control-group {
            margin-bottom: 10px;
        }

        .control-label {
            font-size: 12px;
            color: #666;
            margin-bottom: 5px;
        }

        .control-value {
            font-weight: bold;
            font-size: 14px;
        }

        .tilt-bar {
            width: 100%;
            height: 8px;
            background: #eee;
            border-radius: 4px;
            overflow: hidden;
            margin-top: 5px;
            position: relative;
        }

        .tilt-progress {
            height: 100%;
            background: linear-gradient(90deg, #4CAF50 0%, #FF9800 70%, #FF5722 100%);
            transition: width 0.2s ease;
            border-radius: 4px;
        }

        .tilt-threshold {
            position: absolute;
            top: 0;
            height: 100%;
            width: 2px;
            background: rgba(0, 0, 0, 0.5);
        }

        .threshold-45 {
            left: 75%; /* 45 degrees out of 60 */
        }

        .toggle-button {
            width: 100%;
            padding: 8px;
            background: #2196F3;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            margin-top: 10px;
        }

        .toggle-button:hover {
            background: #1976D2;
        }

        .info-panel {
            position: absolute;
            bottom: 20px;
            left: 20px;
            background: rgba(255, 255, 255, 0.9);
            padding: 15px;
            border-radius: 5px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            z-index: 1000;
            max-width: 350px;
            font-size: 14px;
        }

        .info-title {
            font-weight: bold;
            margin-bottom: 10px;
            color: #333;
        }

        .info-text {
            color: #666;
            line-height: 1.4;
        }

        .loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(255, 255, 255, 0.9);
            padding: 20px;
            border-radius: 5px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            z-index: 2000;
            text-align: center;
        }

        .spinner {
            border: 3px solid #f3f3f3;
            border-top: 3px solid #2196F3;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
            margin: 0 auto 10px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .extrusion-info {
            color: #FF9800;
            font-size: 12px;
            margin-top: 3px;
        }
    </style>
</head>
<body>
    <div id="map"></div>
    
    <div class="loading" id="loading">
        <div class="spinner"></div>
        <div>Loading MapsIndoors...</div>
    </div>
    
    <div class="view-indicator view-2d" id="view-indicator">
        <span id="view-mode">2D View</span>
    </div>
    
    <div class="controls">
        <div class="control-group">
            <div class="control-label">Camera Tilt</div>
            <div class="control-value" id="tilt-value">0°</div>
            <div class="tilt-bar">
                <div class="tilt-progress" id="tilt-progress"></div>
                <div class="tilt-threshold threshold-45"></div>
            </div>
            <div class="extrusion-info" id="extrusion-info">Walls Height: 0.0</div>
        </div>
        
        <div class="control-group">
            <div class="control-label">Zoom Level</div>
            <div class="control-value" id="zoom-value">20</div>
        </div>
        
        <div class="control-group">
            <div class="control-label">Current Floor</div>
            <div class="control-value" id="floor-value">0</div>
        </div>
        
        <div class="control-group">
            <div class="control-label">View State</div>
            <div class="control-value" id="view-state">2D Mode</div>
        </div>
        
        <button class="toggle-button" id="manual-toggle">
            Toggle 2D/3D Mode
        </button>
        
        <button class="toggle-button" id="feature-toggle" style="margin-top: 5px;">
            Toggle Features
        </button>
    </div>
    
    <div class="info-panel">
        <div class="info-title">Dynamic View Switcher with Wall Extrusion</div>
        <div class="info-text">
            • <strong>0° - 1°:</strong> Pure 2D mode<br>
            • <strong>Feature Toggle:</strong> Use proper MapsIndoors 2D/3D feature hiding<br>
            • <strong>Wall Height Mode:</strong> Smooth wall growth with camera tilt<br>
            • <strong>45°+:</strong> Full 3D mode with all 3D features<br>
            • Wall height grows smoothly from 0 to 1.5<br>
            • Drag to rotate/tilt the map and see the smooth transitions<br>
            • Use the manual toggle button to override automatic mode
        </div>
    </div>

    <script>
        class CustomMap {
            constructor() {
                // View state
                this.currentViewMode = '2d'; // '2d', 'transition', '3d'
                this.manualOverride = false;
                
                // Threshold angles (in degrees)
                this.transitionStartAngle = 1;
                this.full3DAngle = 45;
                this.maxWallsHeight = 1.5;
                
                // 2D/3D toggle state
                this.is3DMode = false;
                this.useFeatureToggle = false;
                
                // Feature arrays for 2D/3D toggle - will be initialized after MapsIndoors loads
                this.features3DToHide = [];
                this.features2DToHide = [];
                
                // MapsIndoors instances
                this.mapsIndoorsInstance = null;
                this.mapboxInstance = null;
                this.locationIds = [];
                
                // Initialize
                this.init();
            }
            
            // Initialize feature types after MapsIndoors is loaded
            initializeFeatureTypes() {
                try {
                    // Check if FeatureType exists and initialize arrays
                    if (typeof mapsindoors !== 'undefined' && mapsindoors.FeatureType) {
                        this.features3DToHide = [
                            mapsindoors.FeatureType.WALLS3D,
                            mapsindoors.FeatureType.MODEL3D,
                            mapsindoors.FeatureType.EXTRUSION3D,
                            mapsindoors.FeatureType.EXTRUDEDBUILDINGS
                        ].filter(type => type !== undefined); // Filter out any undefined types
                        
                        this.features2DToHide = [
                            mapsindoors.FeatureType.MODEL2D,
                            mapsindoors.FeatureType.WALLS2D
                        ].filter(type => type !== undefined);
                        
                        console.log('Feature types initialized:', {
                            features3DToHide: this.features3DToHide,
                            features2DToHide: this.features2DToHide
                        });
                    } else {
                        console.warn('FeatureType not available, using string constants');
                        // Fallback to string constants if FeatureType enum doesn't exist
                        this.features3DToHide = ['WALLS3D', 'MODEL3D', 'EXTRUSION3D', 'EXTRUDEDBUILDINGS'];
                        this.features2DToHide = ['MODEL2D', 'WALLS2D'];
                    }
                } catch (error) {
                    console.error('Error initializing feature types:', error);
                    // Fallback to empty arrays
                    this.features3DToHide = [];
                    this.features2DToHide = [];
                }
            }
            
            async init() {
                try {
                    await this.initializeMapsIndoors();
                    this.setupEventListeners();
                } catch (error) {
                    console.error('Error initializing map:', error);
                    this.hideLoading();
                    alert('Error loading map. Please refresh the page.');
                }
            }
            
            async initializeMapsIndoors() {
                // Initialize map with MapboxV3View
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
                this.mapView = mapViewInstance; // Store reference for feature toggling

                    // Wait for MapsIndoors to be ready
                await new Promise((resolve) => {
                    this.mapsIndoorsInstance.addListener('ready', async () => {
                        console.log('MapsIndoors is ready');
                        
                        // Configure solution for 3D capability
                        try {
                            const solutionConfig = this.mapsIndoorsInstance.getSolutionConfig();
                            // Enable 3D features in solution config
                            solutionConfig.is3D = true;
                            this.mapsIndoorsInstance.setSolutionConfig(solutionConfig);
                            console.log('3D mode enabled in solution config');
                            
                            // Initialize feature arrays now that MapsIndoors is loaded
                            this.initializeFeatureTypes();
                            
                            // Initialize in 2D mode (hide 3D features)
                            this.hide3DFeatures();
                        } catch (error) {
                            console.warn('Could not configure 3D mode:', error);
                        }
                        
                        await this.onMapReady();
                        resolve();
                    });
                });

                // Add floor selector
                const floorSelectorElement = document.createElement('div');
                new mapsindoors.FloorSelector(floorSelectorElement, this.mapsIndoorsInstance);
                this.mapboxInstance.addControl({ 
                    onAdd: () => floorSelectorElement,
                    onRemove: () => {} 
                });

                // Handle errors
                this.mapsIndoorsInstance.addListener('error', (error) => {
                    console.error('MapsIndoors error:', error);
                });
            }
            
            async onMapReady() {
                try {
                    // Get all locations to apply display rules
                    const locations = await mapsindoors.services.LocationsService.getLocations({
                        take: 1000 // Get a large number of locations
                    });
                    
                    this.locationIds = locations.map(location => location.id);
                    console.log(`Found ${this.locationIds.length} locations`);
                    
                    // Set initial 2D view mode
                    await this.setViewModeByAngle(0);
                    
                    // Update UI
                    this.hideLoading();
                    
                } catch (error) {
                    console.error('Error in onMapReady:', error);
                    this.hideLoading();
                }
            }
            
            setupEventListeners() {
                // Throttle function for performance
                let updateTimeout = null;
                const throttledUpdate = () => {
                    if (updateTimeout) clearTimeout(updateTimeout);
                    updateTimeout = setTimeout(() => {
                        this.updateViewBasedOnTilt();
                        this.updateCameraInfo();
                    }, 16); // ~60fps
                };
                
                // Add camera event listeners
                this.mapboxInstance.on('move', throttledUpdate);
                this.mapboxInstance.on('rotate', throttledUpdate);
                this.mapboxInstance.on('pitch', throttledUpdate);
                this.mapboxInstance.on('zoom', () => this.updateCameraInfo());
                
                // Manual toggle button
                document.getElementById('manual-toggle').addEventListener('click', () => {
                    this.manualOverride = !this.manualOverride;
                    if (this.manualOverride) {
                        // Cycle through modes: 2d -> transition -> 3d -> 2d
                        const currentPitch = this.mapboxInstance.getPitch();
                        if (this.currentViewMode === '2d') {
                            this.setViewModeByAngle(25); // Mid-transition
                        } else if (this.currentViewMode === 'transition') {
                            this.setViewModeByAngle(50); // Full 3D
                        } else {
                            this.setViewModeByAngle(0); // Back to 2D
                        }
                    } else {
                        // Return to automatic mode
                        this.updateViewBasedOnTilt();
                    }
                });
                
                // Feature toggle button
                document.getElementById('feature-toggle').addEventListener('click', () => {
                    this.useFeatureToggle = !this.useFeatureToggle;
                    
                    if (this.useFeatureToggle) {
                        // Use feature-based 2D/3D toggle
                        this.is3DMode = !this.is3DMode;
                        
                        if (this.is3DMode) {
                            this.hide2DFeatures();
                            this.show3DFeatures();
                        } else {
                            this.hide3DFeatures();
                            this.show2DFeatures();
                        }
                        
                        console.log(`Feature toggle mode: ${this.is3DMode ? '3D' : '2D'}`);
                    } else {
                        // Return to wall height based mode
                        this.showAllFeatures();
                        this.updateViewBasedOnTilt();
                    }
                });
                
                // Floor change listener
                this.mapsIndoorsInstance.addListener('floor_changed', () => {
                    this.updateCameraInfo();
                });
                
                // Initial camera info update
                this.updateCameraInfo();
            }
            
            async updateViewBasedOnTilt() {
                if (!this.mapsIndoorsInstance || this.manualOverride || this.useFeatureToggle) return;
                
                const pitch = this.mapboxInstance.getPitch();
                await this.setViewModeByAngle(pitch);
            }
            
            async setViewModeByAngle(pitch) {
                if (!this.locationIds || this.locationIds.length === 0) {
                    console.warn('No location IDs available for display rules');
                    return;
                }
                
                let newViewMode;
                
                if (pitch <= this.transitionStartAngle) {
                    // Pure 2D mode (0° - 1°) - ALL 3D features OFF, 2D features ON
                    newViewMode = '2d';
                    
                    // Hide all 3D features, show all 2D features
                    this.hideAll3DFeatures();
                    this.showAll2DFeatures();
                    
                    // Also set display rule to ensure walls are off
                    const displayRule = {
                        walls3D: false,
                        wallsVisible: false,
                        wallsHeight: 0,
                        extrusionVisible: false,
                        model3DVisible: false,
                        iconVisible: true,
                        polygonVisible: true,
                       // labelVisible: true,
                        zoomFrom: 16
                    };
                    this.mapsIndoorsInstance.setDisplayRule(this.locationIds, displayRule);
                    
                } else if (pitch < this.full3DAngle) {
                    // Transition mode (1° - 44°) - 3D walls with growing height, keep 2D features
                    newViewMode = 'transition';
                    
                    // Hide most 3D features but allow walls, keep 2D features visible
                    this.hideNon3DWallFeatures();
                    this.showAll2DFeatures();
                    
                    // Calculate wall height based on angle (1° to 44° maps to 0 to 1.5)
                    const transitionProgress = (pitch - this.transitionStartAngle) / (this.full3DAngle - this.transitionStartAngle);
                    const wallsHeight = transitionProgress * this.maxWallsHeight;
                    
                    const displayRule = {
                        walls3D: true, // Enable 3D walls
                        wallsVisible: true,
                        wallsHeight: wallsHeight,
                        extrusionVisible: false, // Keep other 3D features off
                        model3DVisible: false, // Keep 3D models off until full 3D
                        iconVisible: true, // Keep 2D features visible
                        polygonVisible: true,
                        zoomFrom: 16
                    };
                    this.mapsIndoorsInstance.setDisplayRule(this.locationIds, displayRule);
                    
                } else {
                    // Full 3D mode (45°+) - ALL 2D OFF, ALL 3D ON
                    newViewMode = '3d';
                    
                    // Hide all 2D features, show all 3D features
                    this.hideAll2DFeatures();
                    this.showAll3DFeatures();
                    
                    const displayRule = {
                        walls3D: true,
                        wallsVisible: true,
                        wallsHeight: this.maxWallsHeight,
                        extrusionVisible: false,
                        model3DVisible: true,
                        iconVisible: false, // Hide 2D icons in full 3D
                        polygonVisible: true,
                        zoomFrom: 16
                    };
                    this.mapsIndoorsInstance.setDisplayRule(this.locationIds, displayRule);
                }
                
                // Only update if the view mode has changed or if we're in transition mode
                if (newViewMode !== this.currentViewMode || newViewMode === 'transition') {
                    this.currentViewMode = newViewMode;
                    console.log(`Setting view mode: ${newViewMode}, pitch: ${pitch.toFixed(1)}°, walls height: ${this.currentViewMode === 'transition' ? ((pitch - this.transitionStartAngle) / (this.full3DAngle - this.transitionStartAngle) * this.maxWallsHeight).toFixed(2) : (this.currentViewMode === '3d' ? this.maxWallsHeight : 0)}`);
                    this.updateViewIndicator();
                }
            }
            
            updateViewIndicator() {
                const indicator = document.getElementById('view-indicator');
                const modeText = document.getElementById('view-mode');
                const viewState = document.getElementById('view-state');
                
                let statusText = '';
                if (this.useFeatureToggle) {
                    statusText = ' (Feature Toggle)';
                } else if (this.manualOverride) {
                    statusText = ' (Manual)';
                } else {
                    statusText = ' (Auto)';
                }
                
                if (this.useFeatureToggle) {
                    if (this.is3DMode) {
                        indicator.className = 'view-indicator view-3d';
                        modeText.textContent = '3D Features' + statusText;
                        viewState.textContent = '3D Features Mode';
                    } else {
                        indicator.className = 'view-indicator view-2d';
                        modeText.textContent = '2D Features' + statusText;
                        viewState.textContent = '2D Features Mode';
                    }
                } else {
                    if (this.currentViewMode === '2d') {
                        indicator.className = 'view-indicator view-2d';
                        modeText.textContent = '2D View' + statusText;
                        viewState.textContent = '2D Mode';
                    } else if (this.currentViewMode === 'transition') {
                        indicator.className = 'view-indicator view-transition';
                        modeText.textContent = 'Transition' + statusText;
                        viewState.textContent = 'Transition Mode';
                    } else {
                        indicator.className = 'view-indicator view-3d';
                        modeText.textContent = '3D View' + statusText;
                        viewState.textContent = '3D Mode';
                    }
                }
            }
            
            updateCameraInfo() {
                if (!this.mapboxInstance) return;
                
                // Get camera properties
                const pitch = this.mapboxInstance.getPitch();
                const zoom = this.mapboxInstance.getZoom();
                const currentFloor = this.mapsIndoorsInstance ? this.mapsIndoorsInstance.getFloor() : 0;
                
                // Calculate current walls height
                let wallsHeight = 0;
                if (pitch > this.transitionStartAngle && pitch < this.full3DAngle) {
                    const transitionProgress = (pitch - this.transitionStartAngle) / (this.full3DAngle - this.transitionStartAngle);
                    wallsHeight = transitionProgress * this.maxWallsHeight;
                } else if (pitch >= this.full3DAngle) {
                    wallsHeight = this.maxWallsHeight;
                }
                
                // Update tilt display
                document.getElementById('tilt-value').textContent = `${Math.round(pitch)}°`;
                
                // Update walls height display
                document.getElementById('extrusion-info').textContent = `Walls Height: ${wallsHeight.toFixed(2)}`;
                
                // Update tilt progress bar (0-60 degrees range)
                const tiltProgress = Math.min(pitch / 60 * 100, 100);
                document.getElementById('tilt-progress').style.width = `${tiltProgress}%`;
                
                // Update zoom display
                document.getElementById('zoom-value').textContent = zoom.toFixed(1);
                
                // Update floor display
                document.getElementById('floor-value').textContent = currentFloor || 0;
            }
            
            hideLoading() {
                const loading = document.getElementById('loading');
                if (loading) {
                    loading.style.display = 'none';
                }
            }
            
            // 2D/3D Feature Toggle Methods
            hideAll3DFeatures() {
                if (this.mapView && this.mapView.hideFeatures && this.features3DToHide.length > 0) {
                    try {
                        this.mapView.hideFeatures(this.features3DToHide);
                        console.log('Hidden all 3D features:', this.features3DToHide);
                    } catch (error) {
                        console.error('Error hiding 3D features:', error);
                    }
                }
            }
            
            hideAll2DFeatures() {
                if (this.mapView && this.mapView.hideFeatures && this.features2DToHide.length > 0) {
                    try {
                        this.mapView.hideFeatures(this.features2DToHide);
                        console.log('Hidden all 2D features:', this.features2DToHide);
                    } catch (error) {
                        console.error('Error hiding 2D features:', error);
                    }
                }
            }
            
            hideNon3DWallFeatures() {
                // Hide 3D features except walls (for transition mode)
                if (this.mapView && this.mapView.hideFeatures) {
                    try {
                        const non3DWallFeatures = this.features3DToHide.filter(feature => {
                            // Keep WALLS3D visible, hide others
                            const featureStr = feature.toString ? feature.toString() : feature;
                            return !featureStr.includes('WALLS3D') && !featureStr.includes('walls3d');
                        });
                        
                        if (non3DWallFeatures.length > 0) {
                            this.mapView.hideFeatures(non3DWallFeatures);
                            console.log('Hidden non-wall 3D features:', non3DWallFeatures);
                        }
                    } catch (error) {
                        console.error('Error hiding non-wall 3D features:', error);
                    }
                }
            }
            
            showAll3DFeatures() {
                // Show all 3D features by ensuring 2D features are hidden
                this.hideAll2DFeatures();
                // Reset any feature hiding to show all 3D
                if (this.mapView && this.mapView.hideFeatures) {
                    try {
                        // Only hide 2D features, let all 3D show
                        this.mapView.hideFeatures(this.features2DToHide);
                        console.log('Showing all 3D features, hiding 2D features');
                    } catch (error) {
                        console.error('Error showing 3D features:', error);
                    }
                }
            }
            
            showAll2DFeatures() {
                // Show all 2D features by not hiding them
                // In transition mode, we want 2D features visible alongside 3D walls
                console.log('Keeping 2D features visible');
            }
            
            hide3DFeatures() {
                this.hideAll3DFeatures();
            }
            
            hide2DFeatures() {
                this.hideAll2DFeatures();
            }
            
            show3DFeatures() {
                this.showAll3DFeatures();
            }
            
            show2DFeatures() {
                this.showAll2DFeatures();
            }
            
            showAllFeatures() {
                if (this.mapView && this.mapView.hideFeatures) {
                    try {
                        this.mapView.hideFeatures([]);
                        console.log('Showing all features');
                    } catch (error) {
                        console.error('Error showing all features:', error);
                    }
                }
            }
        }
        
        // Initialize the custom map when the page loads
        document.addEventListener('DOMContentLoaded', () => {
            console.log('DOM loaded, initializing custom map...');
            new CustomMap();
        });
        
        // Fallback initialization if DOMContentLoaded already fired
        if (document.readyState === 'loading') {
            // Document is still loading
        } else {
            // Document is already loaded
            console.log('Document already loaded, initializing custom map...');
            new CustomMap();
        }
    </script>
</body>
</html>
```

### Explanation
This implementation creates a sophisticated 2D/3D view switching system with three distinct modes:

**Core Functionality:**
- **Pure 2D Mode (0°-1° tilt):** All 3D features disabled, 2D features visible, no wall extrusion
- **Transition Mode (1°-45° tilt):** Gradual wall height growth from 0 to 1.5 units, 2D features remain visible alongside growing 3D walls
- **Full 3D Mode (45°+ tilt):** Complete 3D experience with all 3D features enabled, 2D features hidden

**Technical Implementation:**
- Uses Mapbox camera pitch events to detect user interaction
- Dynamically calculates wall height using linear interpolation based on tilt angle
- Applies MapsIndoors display rules to control feature visibility
- Implements both feature-based hiding (using FeatureType enums) and display rule-based control
- Includes performance optimization with throttled event handlers (60fps updates)
- Provides manual override and feature toggle modes for testing

**Advanced Features:**
- Real-time visual feedback with color-coded indicators and progress bars
- Automatic feature type initialization with fallback support
- Comprehensive error handling and console logging
- Responsive UI controls showing current camera state
- Floor selector integration

### Use Cases
- Interactive building exploration where users transition from 2D floor plans to 3D spatial understanding
- Wayfinding applications that need both detailed 2D layout views and 3D context
- Real estate applications showing properties from floor plan to architectural visualization
- Campus navigation systems transitioning between map view and building interior visualization
- Healthcare facility navigation with both schematic and spatial views
- Conference venue exploration combining floor plans with 3D venue layouts

### Important Notes
⚠️ Must load Mapbox CSS and JS before MapsIndoors SDK to prevent styling conflicts
⚠️ Requires solution configuration with is3D: true to enable 3D features
⚠️ FeatureType enums may not be available in all SDK versions - includes fallback to string constants
⚠️ Display rules and feature hiding work differently - this solution uses both for maximum compatibility
⚠️ Performance sensitive - uses throttled event handlers to maintain 60fps during camera movements
⚠️ Location IDs must be retrieved before applying display rules
⚠️ Manual override and feature toggle modes will conflict with automatic tilt-based switching
⚠️ Wall height calculations are linear - consider easing functions for smoother visual transitions
⚠️ Some 3D features may not be available depending on the dataset and solution configuration



## Examples


### Moving Asset Tracking with Custom Markers - Mapbox Integration

**Description:** Real-time asset tracking system using MapsIndoors with Mapbox as the base map. Shows smooth animation between positions and custom marker styling for vehicles, equipment, or personnel tracking.

**Code:**
```javascript
// Initialize MapsIndoors with Mapbox
const mapViewOptions = {
    accessToken: 'your-mapbox-token',
    element: document.getElementById('map'),
    center: { lat: 30.3603212, lng: -97.7422623 },
    zoom: 18,
    maxZoom: 22,
};

// Use MapboxV3View
const mapViewInstance = new mapsindoors.mapView.MapboxV3View(mapViewOptions);
const mapsIndoorsInstance = new mapsindoors.MapsIndoors({
    mapView: mapViewInstance,
});
const mapboxInstance = mapViewInstance.getMap();

// Create custom marker with custom icon
function createAssetMarker(position, iconUrl) {
    const customElement = document.createElement('div');
    customElement.style.cssText = `
        width: 40px;
        height: 40px;
        background-image: url('${iconUrl}');
        background-size: contain;
        background-repeat: no-repeat;
        background-position: center;
        cursor: pointer;
    `;
    
    const marker = new mapboxgl.Marker({
        element: customElement,
        anchor: 'center'
    })
    .setLngLat([position.lng, position.lat])
    .addTo(mapboxInstance);
    
    return marker;
}

// Animate marker movement
function animateMarkerMovement(marker, fromCoords, toCoords, duration = 800) {
    const startTime = Date.now();
    
    const animate = () => {
        const elapsed = Date.now() - startTime;
        const progress = Math.min(elapsed / duration, 1);
        
        const currentLat = fromCoords.lat + (toCoords.lat - fromCoords.lat) * progress;
        const currentLng = fromCoords.lng + (toCoords.lng - fromCoords.lng) * progress;
        
        marker.setLngLat([currentLng, currentLat]);
        
        if (progress < 1) {
            requestAnimationFrame(animate);
        }
    };
    
    animate();
}

// Update position from API
async function updateAssetPosition(assetId) {
    try {
        const response = await fetch(`/api/assets/${assetId}/position`, {
            method: "POST" // or GET depending on your API
        });
        
        const result = await response.json();
        const { lat, lng, floor } = result.data;
        
        const newPosition = { lat, lng };
        const oldPosition = currentPosition;
        
        // Update floor if changed
        if (floor !== currentFloor) {
            mapsIndoorsInstance.setFloor(floor);
            currentFloor = floor;
        }
        
        // Animate marker to new position
        animateMarkerMovement(assetMarker, oldPosition, newPosition);
        
        currentPosition = newPosition;
    } catch (error) {
        console.error('Error updating position:', error);
    }
}
```


### Moving Asset Tracking with Custom Markers - Google Maps Integration

**Description:** Real-time asset tracking system using MapsIndoors with Google Maps as the base map provider. Shows smooth animation and integration with Google's mapping services for vehicles, equipment, or personnel tracking.

**Code:**
```javascript
// Load Google Maps API with geometry library
// <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_KEY&libraries=geometry"></script>

// Initialize MapsIndoors with Google Maps
const mapViewOptions = {
    element: document.getElementById('map'),
    center: { lat: 30.3603212, lng: -97.7422623 },
    zoom: 18,
    maxZoom: 22,
    mapTypeId: google.maps.MapTypeId.ROADMAP
};

// Use GoogleMapsView
const mapViewInstance = new mapsindoors.mapView.GoogleMapsView(mapViewOptions);
const mapsIndoorsInstance = new mapsindoors.MapsIndoors({
    mapView: mapViewInstance,
});
const googleMap = mapViewInstance.getMap();

// Create Google Maps marker with custom icon
function createAssetMarker(position, iconUrl) {
    const marker = new google.maps.Marker({
        position: position,
        map: googleMap,
        icon: {
            url: iconUrl,
            scaledSize: new google.maps.Size(40, 40),
            anchor: new google.maps.Point(20, 20)
        },
        title: 'Asset Tracker'
    });
    
    return marker;
}

// Animate marker movement using Google Maps
function animateMarkerMovement(marker, fromCoords, toCoords, duration = 800) {
    const startTime = Date.now();
    
    const animate = () => {
        const elapsed = Date.now() - startTime;
        const progress = Math.min(elapsed / duration, 1);
        
        const currentLat = fromCoords.lat + (toCoords.lat - fromCoords.lat) * progress;
        const currentLng = fromCoords.lng + (toCoords.lng - fromCoords.lng) * progress;
        
        marker.setPosition({ lat: currentLat, lng: currentLng });
        
        if (progress < 1) {
            requestAnimationFrame(animate);
        }
    };
    
    animate();
}

// Calculate distance using Google geometry
function calculateDistance(coord1, coord2) {
    const point1 = new google.maps.LatLng(coord1.lat, coord1.lng);
    const point2 = new google.maps.LatLng(coord2.lat, coord2.lng);
    return google.maps.geometry.spherical.computeDistanceBetween(point1, point2);
}

// Update position from API
async function updateAssetPosition(assetId) {
    try {
        const response = await fetch(`/api/assets/${assetId}/position`, {
            method: "POST"
        });
        
        const result = await response.json();
        const { lat, lng, floor } = result.data;
        
        const newPosition = { lat, lng };
        const oldPosition = currentPosition;
        
        // Update floor if changed
        if (floor !== currentFloor) {
            mapsIndoorsInstance.setFloor(floor);
            currentFloor = floor;
        }
        
        // Animate marker to new position
        animateMarkerMovement(assetMarker, oldPosition, newPosition);
        
        currentPosition = newPosition;
    } catch (error) {
        console.error('Error updating position:', error);
    }
}
```


### MapsIndoors Solution Config - Custom Selection Icon

**Description:** How to customize the selection state icon in MapsIndoors by modifying the solution configuration. This example shows how to change the default selection icon to a custom room-occupied icon.

**Code:**
```javascript
// Wait for MapsIndoors to be ready
return new Promise((resolve, reject) => {
    this.mapsIndoorsInstance.addListener('ready', async () => {
        console.log('MapsIndoors is ready');
        try {
            // Get the Solution Config object
            const solutionConfig = this.mapsIndoorsInstance.getSolutionConfig();
            console.log('Original selection icon:', solutionConfig.stateDisplayRules.selection.icon);
            
            // Get the selection state DisplayRule
            const selectionIconDisplayRule = solutionConfig.stateDisplayRules.selection;
            
            // Update the selection icon
            selectionIconDisplayRule.icon = "https://media.mapsindoors.com/57e4e4992e74800ef8b69718/media/room-occupied.svg";
            
            // Update the SolutionConfig to apply the changes
            this.mapsIndoorsInstance.setSolutionConfig(solutionConfig);
            console.log('Updated selection icon:', solutionConfig.stateDisplayRules.selection.icon);
            
            // Continue with your existing logic
            await this.loadLocations();
            this.setupMapEventListeners();
            resolve();
        } catch (error) {
            console.error('Error in MapsIndoors ready handler:', error);
            reject(error);
        }
    });

    // Optional: Add error listener for additional error handling
    this.mapsIndoorsInstance.addListener('error', (error) => {
        console.error('MapsIndoors error:', error);
        reject(error);
    });
});
```

