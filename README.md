# UGV Control Interface

This project is for Milrem Software engineering homework 2025.

## Project Setup

### 1. Clone repository

```bash
git clone https://github.com/raiomitt/ugv-milrem.git
cd ugv-milrem
```

### 2. Install dependencies

```sh
npm install
```

### 3. Run project

```sh
npm run dev
```

### 4. Open project

Go to website: http://localhost:5173/

## Instructions to use app

You can move the UGV on the map using the arrow keys, but you must turn on the engine first.

If you hold the cursor down on the map for 0.6 seconds, a waypoint will be created and a popup with several options will appear:

    Discard: Discards and removes the waypoint.

    Drive: Instantly teleports the UGV to the waypoint and removes the waypoint.

    Save: Saves the waypoint and changes its icon.

After saving a waypoint, you can:

    Rename: Rename waypoint button.

    Drive: Instantly teleports the UGV to the waypoint and removes the waypoint.

    Delete: delete permanently.

You can view all saved waypoints by clicking on Show Waypoints, where you will have access to the same options for each waypoint.
'Show Waypoints' tabel also refreshes automaticly after changing something.

## Most difficult part for me & Use of AI

The most challenging part for me was learning how to use TypeScript. I had previously worked with Vue, HTML, CSS, and JavaScript, so fortunately, picking up TypeScript wasn’t too difficult.
For this project, I used AI to find out how to import the Leaflet map into my project and how to make it work in full screen. I also asked AI how to change the marker icon image.
