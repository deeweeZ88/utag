<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Map Tagging App</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.css" />
    <style>
        #map {
            height: 100vh;
            width: 100%;
        }
        #formContainer {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: white;
            padding: 20px;
            border: 1px solid #ccc;
            z-index: 1000;
            display: none;
        }
        #tagForm label,
        #tagForm select,
        #tagForm input {
            width: 100%;
            margin-bottom: 10px;
        }
    </style>
</head>
<body>
    <div id="map"></div>

    <div id="formContainer">
        <form id="tagForm">
            <label for="status">Select Status:</label><br />
            <select id="status" name="status">
                <option value="power_out">Power Out</option>
                <option value="power_back_on">Power Back On</option>
                <option value="robbed">Robbed</option>
                <option value="garage_sale">Garage Sale</option>
            </select>
            <br /><br />
            <label for="datetime">Date and Time:</label><br />
            <input type="datetime-local" id="datetime" name="datetime" />
            <br /><br />
            <button type="submit">Submit</button>
            <button type="button" onclick="closeForm()">Cancel</button>
        </form>
    </div>

    <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
    <script src="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.js"></script>
    <script>
        // Initialize the map
        var map = L.map('map').fitWorld();
        var userMarker;
        var tagMarkers = []; // Store all markers

        // Set up the OpenStreetMap layer
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 18,
        }).addTo(map);

        // Color map for different statuses
        var statusColors = {
            'power_out': 'red',
            'power_back_on': 'green',
            'robbed': 'black',
            'garage_sale': 'blue'
        };

        // Initialize geocoder
        var geocoder = L.Control.Geocoder.nominatim();

        // Handle location found
        function onLocationFound(e) {
            var latlng = e.latlng;

            // Add a marker at the user's location if not already added
            if (!userMarker) {
                userMarker = L.marker(latlng).addTo(map);
            } else {
                userMarker.setLatLng(latlng);
            }

            // Bind a popup with a form
            userMarker.bindPopup(
                `<b>Your Location</b><br>
                 <button onclick="openForm()">Add Tag</button>`
            ).openPopup();

            // Center the map on the user's location
            map.setView(latlng, 16);
        }

        // Handle location error
        function onLocationError(e) {
            alert(e.message);
        }

        // Locate the user
        map.on('locationfound', onLocationFound);
        map.on('locationerror', onLocationError);

        map.locate({ setView: true, maxZoom: 16 });

        // Open the form
        function openForm() {
            document.getElementById('formContainer').style.display = 'block';
        }

        // Close the form
        function closeForm() {
            document.getElementById('formContainer').style.display = 'none';
        }

        // Handle form submission
        document.getElementById('tagForm').addEventListener('submit', function (e) {
            e.preventDefault();

            var status = document.getElementById('status').value;
            var datetime = document.getElementById('datetime').value;
            var latlng = userMarker.getLatLng();

            // Get address using geocoder
            geocoder.reverse(latlng, map.options.crs.scale(map.getZoom()), function(results) {
                var address = results[0] ? results[0].name : 'Address not found';

                // Add a marker with the tag information at the user's location
                var newMarker = L.circleMarker(latlng, {
                    color: statusColors[status],
                    radius: 8
                }).addTo(map)
                .bindPopup(
                    `<b>Status:</b> ${status}<br><b>Date and Time:</b> ${datetime}<br><b>Address:</b> ${address}`
                );

                tagMarkers.push(newMarker); // Store the marker for later use

                alert('Tag Submitted!');

                // Close the form
                closeForm();
            });
        });

        // Sample data
        var sampleTags = [
            {
                latlng: [51.505, -0.09],
                status: 'power_out',
                datetime: '2024-10-15T09:00',
            },
            {
                latlng: [51.51, -0.1],
                status: 'garage_sale',
                datetime: '2024-10-16T14:00',
            },
        ];

        // Function to display sample tags
        function displaySampleTags() {
            sampleTags.forEach(function (tag) {
                geocoder.reverse(tag.latlng, map.options.crs.scale(map.getZoom()), function(results) {
                    var address = results[0] ? results[0].name : 'Address not found';
                    var marker = L.circleMarker(tag.latlng, {
                        color: statusColors[tag.status],
                        radius: 8
                    }).addTo(map)
                    .bindPopup(
                        `<b>Status:</b> ${tag.status}<br><b>Date and Time:</b> ${tag.datetime}<br><b>Address:</b> ${address}`
                    );

                    tagMarkers.push(marker); // Store the marker for later use
                });
            });
        }

        // Call the function
        displaySampleTags();
    </script>
</body>
</html>
