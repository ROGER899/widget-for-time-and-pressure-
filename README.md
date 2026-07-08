# widget-for-time-and-pressure-

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Time & Pressure Widget</title>
    <style>
        /* Base styling to center the widget on the page */
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #1e1e2f; /* Dark background */
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
        }

        /* Widget container styling */
        .widget {
            background: linear-gradient(135deg, #4A00E0, #8E2DE2);
            color: white;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            text-align: center;
            width: 280px;
        }

        /* Time and date typography */
        .time {
            font-size: 3.5em;
            font-weight: bold;
            margin-bottom: 5px;
            letter-spacing: 2px;
            text-shadow: 0 2px 4px rgba(0,0,0,0.2);
        }
        
        .date {
            font-size: 1.1em;
            opacity: 0.85;
            margin-bottom: 25px;
        }

        /* Air pressure section styling */
        .pressure-container {
            background: rgba(255, 255, 255, 0.15);
            padding: 15px;
            border-radius: 15px;
            backdrop-filter: blur(5px);
        }

        .pressure-label {
            font-size: 0.9em;
            text-transform: uppercase;
            letter-spacing: 1.5px;
            margin-bottom: 8px;
            opacity: 0.9;
        }

        .pressure-value {
            font-size: 1.8em;
            font-weight: 600;
        }
    </style>
</head>
<body>

    <div class="widget">
        <div class="time" id="timeDisplay">--:--:--</div>
        <div class="date" id="dateDisplay">Loading date...</div>
        
        <div class="pressure-container">
            <div class="pressure-label">Air Pressure</div>
            <div class="pressure-value" id="pressureDisplay">Loading...</div>
        </div>
    </div>

    <script>
        // --- 1. CLOCK LOGIC ---
        function updateTime() {
            const now = new Date();
            
            // Format time and date nicely
            const timeOptions = { hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: false };
            const dateOptions = { weekday: 'long', month: 'short', day: 'numeric' };

            document.getElementById('timeDisplay').textContent = now.toLocaleTimeString([], timeOptions);
            document.getElementById('dateDisplay').textContent = now.toLocaleDateString([], dateOptions);
        }

        setInterval(updateTime, 1000); // Update every 1 second
        updateTime(); // Call immediately to avoid initial delay

        // --- 2. AIR PRESSURE LOGIC ---
        async function fetchPressure(lat, lon) {
            try {
                // Fetch data from Open-Meteo API
                const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=surface_pressure`;
                const response = await fetch(url);
                const data = await response.json();
                
                // Extract and display the pressure in hPa (Hectopascals)
                const pressure = data.current.surface_pressure;
                document.getElementById('pressureDisplay').textContent = `${pressure} hPa`;
            } catch (error) {
                document.getElementById('pressureDisplay').textContent = 'Error';
                console.error("Error fetching pressure data:", error);
            }
        }

        // Get user's coordinates to provide local air pressure
        function initializeWeather() {
            if ("geolocation" in navigator) {
                navigator.geolocation.getCurrentPosition(
                    (position) => {
                        // Success: Use user's exact coordinates
                        fetchPressure(position.coords.latitude, position.coords.longitude);
                    },
                    (error) => {
                        // Denied/Failed: Default to London
                        console.log("Geolocation unavailable, defaulting to London.");
                        fetchPressure(51.5074, -0.1278); 
                    }
                );
            } else {
                // Browser doesn't support geolocation
                fetchPressure(51.5074, -0.1278);
            }
        }

        initializeWeather();

        // Refresh the air pressure data every 30 minutes (1,800,000 milliseconds)
        setInterval(initializeWeather, 1800000);
    </script>

</body>
</html>
