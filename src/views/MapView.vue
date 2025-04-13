<template>
  <!--Div for map-->
  <div id="map" ref="mapElement" class="map-container"></div>
  <!--Div for engine start/stop button-->
  <div class="engine-button" @click="toggleEngine">
    <span>{{ engineStarted ? ' Engine ON' : ' Engine OFF' }}</span>
    <!--Uses simple if-statement to check what to show-->
  </div>
  <!--Div for toggeling waypoints paned-->
  <div class="waypoints-panel-toggle" @click="showWaypoints = !showWaypoints">Show Waypoints</div>
  <!--Div for popup-->
  <div v-if="showPopup" class="popup">
    <p>Start the engine first to move the UGV!</p>
  </div>
  <!--Div for waypoints panel-->
  <div v-if="showWaypoints" class="waypoints-panel">
    <h3>Saved Waypoints</h3>
    <table>
      <thead>
        <tr>
          <th>#</th>
          <th>Name</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(wp, index) in savedWaypoints" :key="index">
          <td>{{ index + 1 }}</td>
          <td>{{ wp.name }}</td>
          <td>
            <button @click="moveUGVTo(wp.latlng)">Drive</button>
            <button @click="renameWaypoint(index)">Rename</button>
            <button @click="deleteWaypoint(index)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
//Imports
import { ref, reactive, onMounted } from 'vue'
import L from 'leaflet'
import '@/assets/map.css'

//Variables
const mapElement = ref<HTMLElement | null>(null)
const engineStarted = ref(false)
const position = ref([58.38541016021584, 26.72474597117554]) //stating positsion for UGV
const showPopup = ref(false)
const speed = ref(0.00001) //Speed variable to change how fast UGV moves
const showWaypoints = ref(false)
const savedWaypoints = ref<{ name: string; latlng: L.LatLng }[]>([]) // List for saving waypoints

//References to the map and the UGV marker
let map: L.Map
let ugvMarker: L.Marker

//Functions
onMounted(() => {
  //My main function

  if (!mapElement.value) return

  map = L.map(mapElement.value).setView(position.value, 25) //Sets map view to my given positson value

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors',
  }).addTo(map)

  //Sets icon to my UGV icon
  const ugvIcon = L.icon({
    iconUrl: '/tank.png',
    iconSize: [32, 32],
    iconAnchor: [16, 32],
  })

  ugvMarker = L.marker(position.value, { icon: ugvIcon }).addTo(map) //Adds marker to map

  window.addEventListener('resize', () => map.invalidateSize())
  setTimeout(() => map.invalidateSize(), 500)

  let longPressTimeout: ReturnType<typeof setTimeout> | null = null
  map.on('mousedown', (e) => {
    //Adds waypoint if mouse button is held down on map
    longPressTimeout = setTimeout(() => addWaypoint(e.latlng), 600)
  })
  map.on('mouseup', () => {
    if (longPressTimeout) clearTimeout(longPressTimeout)
  })

  window.addEventListener('keydown', (e: KeyboardEvent) => {
    //Listener for keyboard commands
    if (!engineStarted.value) {
      //If engine is not started show popup for 5 seconds
      if (e.key.startsWith('Arrow')) {
        showPopup.value = true
        setTimeout(() => (showPopup.value = false), 5000)
      }
      return
    }
    switch (e.key) {
      case 'ArrowUp': //Moves UGV forward
        position.value[0] += speed.value
        break
      case 'ArrowDown': //Moves UGV backwars
        position.value[0] -= speed.value
        break
      case 'ArrowLeft': //Moves UGV left
        position.value[1] -= speed.value
        break
      case 'ArrowRight': //Moves UGV right
        position.value[1] += speed.value
        break
    }
    //Changes UGV position on map
    ugvMarker.setLatLng(position.value)
    map.setView(position.value)
  })
})
const toggleEngine = () => {
  // If button is clicked changes value of engineStarted
  engineStarted.value = !engineStarted.value
  if (engineStarted.value) showPopup.value = false //If value turns to true, sets popup to false
}

const moveUGVTo = (latlng: L.LatLng) => {
  //Moves UGV to given location and also sets map view to this location
  ugvMarker.setLatLng(latlng)
  map.setView(latlng)
  position.value = [latlng.lat, latlng.lng]
}

const renameWaypoint = (index: number) => {
  //Shows prompt to rename waypoint
  const wp = savedWaypoints.value[index]
  const newName = prompt('Enter new name for the waypoint:', wp.name)
  wp.name = newName
}

const deleteWaypoint = (index: number) => {
  // Deletes waypoint with given index
  const wp = savedWaypoints.value[index]

  Object.values(map._layers).forEach((layer) => {
    if (layer === ugvMarker) return // don't remove the UGV marker

    if ('getLatLng' in layer && typeof layer.getLatLng === 'function') {
      const marker = layer as L.Marker
      if (marker.getLatLng().equals(wp.latlng)) {
        map.removeLayer(marker)
      }
    }
  })

  savedWaypoints.value.splice(index, 1)
}

const addWaypoint = (latlng: L.LatLng) => {
  //Adds waypoint marker and open small popup
  const marker = L.marker(latlng).addTo(map)
  marker
    .bindPopup(
      `
    <div>
      <button id="drive">Drive</button>
      <button id="save">Save</button>
      <button id="discard">Discard</button>
    </div>
  `,
    )
    .openPopup()

  marker.getPopup()?.on('add', () => {
    const el = marker.getPopup()?.getElement()
    el?.querySelector('#drive')?.addEventListener('click', () => {
      //If drive is selected drives UGV to this location and removes marker
      moveUGVTo(latlng)
      map.closePopup()
      map.removeLayer(marker)
    })
    el?.querySelector('#save')?.addEventListener('click', () => {
      //If save is selected saves waypoint and switches it to savedMarker
      const name = `Waypoint ${savedWaypoints.value.length + 1}`
      const waypoint = reactive({ name, latlng })
      savedWaypoints.value.push(waypoint)

      const savedMarker = L.marker(latlng, {
        icon: L.icon({
          iconUrl: '/gps.png',
          iconSize: [36, 36],
          iconAnchor: [16, 36],
        }),
      }).addTo(map)

      const updatePopup = () => {
        //Updates popup for saved marker
        savedMarker.bindPopup(`
          <div>
            <strong>${waypoint.name}</strong>
            <button id="drive">Drive</button>
            <button id="rename">Rename</button>
            <button id="delete">Delete</button>
          </div>
        `)
      }

      updatePopup()

      savedMarker.on('popupopen', () => {
        const el = savedMarker.getPopup()?.getElement()
        el?.querySelector('#drive')?.addEventListener('click', () => {
          //Drives UGV
          moveUGVTo(waypoint.latlng)
          map.closePopup()
        })
        el?.querySelector('#rename')?.addEventListener('click', () => {
          //Renames waypoint
          const newName = prompt('Enter new name:', waypoint.name)
          if (newName) {
            waypoint.name = newName
            updatePopup()
          }
        })
        el?.querySelector('#delete')?.addEventListener('click', () => {
          //Deletes saved marker
          map.removeLayer(savedMarker)
          const index = savedWaypoints.value.indexOf(waypoint)
          if (index !== -1) savedWaypoints.value.splice(index, 1)
        })
      })

      map.removeLayer(marker)
      map.closePopup()
    })
    el?.querySelector('#discard')?.addEventListener('click', () => {
      //Discards waypoint marker
      map.removeLayer(marker)
    })
  })
}
</script>
