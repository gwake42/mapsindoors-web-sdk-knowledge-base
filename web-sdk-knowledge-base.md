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

