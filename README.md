# chaoistaoist-loco
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Live GPS Location</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
  <style>
    body { margin: 0; padding: 0; }
    #map { height: 100vh; width: 100%; }
    #status {
      position: fixed; top: 10px; left: 50%;
      transform: translateX(-50%);
      background: rgba(0,0,0,0.7); color: white;
      padding: 6px 14px; border-radius: 20px;
      font-family: sans-serif; font-size: 13px;
      z-index: 9999;
    }
  </style>
</head>
<body>
  <div id="status">Waiting for ping...</div>
  <div id="map"></div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    const NTFY_TOPIC = "chaoistaoist-loco"; // 🔁 replace this

    const map = L.map('map').setView([43.997, -76.189], 15);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap contributors'
    }).addTo(map);

    let marker = null;

    function parseCoordsFromMessage(msg) {
      // Extract from OsmAnd URL: ?pin=LAT,LON
      const match = msg.match(/[?&]pin=([-\d.]+),([-\d.]+)/);
      if (match) {
        return { lat: parseFloat(match[1]), lon: parseFloat(match[2]) };
      }
      return null;
    }

    function updateMap(lat, lon, timestamp) {
      if (marker) {
        marker.setLatLng([lat, lon]);
      } else {
        marker = L.marker([lat, lon]).addTo(map);
      }
      marker.bindPopup(`<b>Last ping:</b><br>${timestamp}`).openPopup();
      map.setView([lat, lon], 15);
      document.getElementById('status').textContent = `📍 Updated: ${timestamp}`;
    }

    function connectSSE() {
      const url = `https://ntfy.sh/${chaoistaoist-loco}/sse`;
      const es = new EventSource(url);

      es.onmessage = (event) => {
        try {
          const data = JSON.parse(event.data);
          const msg = data.message || "";
          const coords = parseCoordsFromMessage(msg);

          // Extract timestamp from message text
          const tsMatch = msg.match(/(Mon|Tue|Wed|Thu|Fri|Sat|Sun).+\d{4}/);
          const timestamp = tsMatch ? tsMatch[0] : new Date().toLocaleString();

          if (coords) {
            updateMap(coords.lat, coords.lon, timestamp);
          }
        } catch(e) {
          console.error("Parse error:", e);
        }
      };

      es.onerror = () => {
        document.getElementById('status').textContent = '⚠️ Reconnecting...';
        es.close();
        setTimeout(connectSSE, 5000); // auto-reconnect
      };
    }

    connectSSE();
  </script>
</body>
</html>

