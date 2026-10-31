# inkBOOK Home Assistant Dashboard

A lightweight, energy-efficient E-Ink dashboard for legacy Android 4.x e-readers (such as inkBOOK Classic, Prime, and Obsidian).

The setup uses **Server-Side Rendering (SSR)** to offload all dynamic interface rendering from the e-reader's dated hardware to a server. A headless Chromium instance periodically captures Home Assistant Lovelace views, converts them to 8-bit grayscale with custom dimensions and rotation, and saves a static image. A lightweight Caddy container serves the asset, allowing the reader to display the dashboard via a simple browser with tap-to-scroll navigation.


### Key Features

* **Zero Reader CPU Load:** Home Assistant's heavy JavaScript is executed entirely on the server. The e-reader only loads a static image.
* **Virtual Pagination (Tap-to-Scroll):** Supports multi-page heights (e.g., 1600 px for two 800 px pages). Tapping the screen shifts the viewport by exactly one screen height (800 px) and loops back to the top on the final screen.
* **Clock-Synchronized Refresh:** The client script aligns page refreshes with the server's cron intervals (accounting for render latency) rather than relying on uncalibrated timers.
* **Clean Kiosk Display:** Hides Home Assistant headers and menus directly at the source using the `kiosk-mode` integration.
* **E-Ink Optimization:** High-contrast grayscale output, 0 FPS static rendering, and no animations to minimize battery drain.


### Architecture Overview

1. **Home Assistant:** Hosts the dashboard view formatted with the `kiosk-mode` query parameter.
2. **`dashboard-scraper` (Docker):** Runs headless Puppeteer on a schedule, renders the view to a defined viewport, applies grayscale and 180° rotation, and writes the output image to the shared directory.
3. **`caddy` (Docker):** Exposes the output image and client viewer over HTTP on port 10103.
4. **inkBOOK (Lightning Browser):** Loads the client page in full-screen mode, manages periodic image refreshes, and handles tap navigation.


### Prerequisites

* **Home Assistant:**
* The `kiosk-mode` integration (via HACS or manual resource registration).
* A dedicated dashboard view sized to 600 px width and multiples of 800 px height.
* A Long-Lived Access Token.


* **Server:** Docker and Docker Compose installed.
* **inkBOOK:** Lightning Browser installed with full-screen mode enabled and connected to the local network (static IP recommended).


### Setup & Configuration

1. **Environment Variables (`.env`):**
* Specify `HA_BASE_URL`, `HA_SCREENSHOT_URL` (including the `?kiosk` query parameter), and `HA_TOKEN`.


2. **Docker Services (`compose.yaml`):**
```
docker compose up -d
```
* Run Caddy to serve `./html` on port 10103.
* Configure `sibbl/hass-lovelace-kindle-screensaver` with desired cron intervals, target dimensions (e.g., 600×1600), rotation (180°), and output path.


3. **Web Client (`html/index.html`):**
* Contains the image container and lightweight JavaScript handling the synchronized refresh schedule and tap-to-scroll logic.


### Reader Setup & Power Tuning

* Open Lightning Browser and navigate to `http://<SERVER_IP>:10103`.
* Enable Fullscreen / Kiosk mode to hide system bars.
