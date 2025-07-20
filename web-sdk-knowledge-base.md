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

