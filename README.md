# ShadeRoute

Shade-optimal walking routes. Projects real building shadows from OpenStreetMap
footprints + live sun position (SunCalc), scores every sidewalk segment by shade,
and routes with a shade-weighted Dijkstra. Includes GPS turn-by-turn navigation
with voice prompts.

Single HTML file, no build step, no API keys.

## Deploy to GitHub Pages (5 minutes)

1. Create a new public repo (e.g. `shaderoute`).
2. Upload these files to the repo root:
   - `index.html`
   - `manifest.webmanifest`
   - `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`
3. Repo Settings -> Pages -> Source: "Deploy from a branch" -> `main` / root -> Save.
4. Wait ~1 minute. Your app is live at `https://<username>.github.io/shaderoute/`.

## Install on iPhone

1. Open the URL in Safari.
2. Share button -> "Add to Home Screen".
3. Launch from the home screen icon — it runs fullscreen like a native app.

HTTPS (which GitHub Pages provides) is required: the browser GPS API,
wake lock, and voice synthesis all refuse to run on insecure pages.

## Using it

1. Type a From and To address (or landmark / intersection) -> "Route it".
   Or pan the map, "Load area", then tap two points.
2. Purple = shade-optimal route. Orange dashed = shortest route. The table
   compares distance, meters in sun, and shade %.
3. Drag the time slider to preview shadows at any time of day.
4. Sun aversion (lambda): how many shaded meters you'd walk to avoid 1 sunny meter.
5. "Start walking" begins turn-by-turn: follows your GPS, speaks turns
   ("In 200 ft, turn left onto Norman Ave"), reroutes automatically if you
   stray 30m+ off route, keeps the screen awake, and refreshes shadows with
   the real clock every 5 minutes.

Grant location permission when prompted ("Allow While Using App" + Precise on iOS).

## Data sources & limits

- Buildings + walking network: Overpass API (public OSM servers, rate-limited —
  if a load fails, wait ~30s and retry).
- Geocoding: Nominatim (max 1 request/second — the app throttles itself).
- Building heights: OSM `height` or `building:levels` tags; buildings missing
  both use the default height in the Tuning panel (12 m ~= 4 floors).
- Corridor loads are capped at ~3 km^2 (about a 20-25 minute walk). Longer
  routes need a precomputed backend — the natural next upgrade, along with
  swapping OSM heights for NYC Open Data's building footprints (`heightroof`)
  and adding the street tree canopy.

## Tuning knobs

Top of the `<script>` in `index.html`:

```js
const CONFIG = {
  maxAreaKm2: 3.0,      // max Overpass load area
  sampleStepM: 8,       // shade sampling density along sidewalks
  maxShadowLenM: 400,   // shadow length cap at low sun angles
  gridCellM: 64,        // spatial index cell size
  walkableHighways: ... // which OSM way types count as walkable
};
```

Turn sensitivity lives in `classifyTurn()` (25deg / 60deg / 135deg bands),
off-route threshold in `onPosition()` (30 m, 3 consecutive fixes), and voice
prompt distances there too (60 m heads-up, 15 m action call).
