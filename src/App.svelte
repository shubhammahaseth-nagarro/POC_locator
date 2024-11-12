<script>
  import { onMount } from "svelte";

  const locationsData = {
    Electrician_Locator: [
      { lat: 51.508742, lng: -0.12085, title: "Location 1" },
      { lat: 40.7128, lng: -74.006, title: "Location 3 (New York)" },
    ],
    Shop_Locator: [{ lat: 48.8566, lng: 2.3522, title: "Location 2 (Paris)" }],
  };
  let Locator = locationsData.Electrician_Locator;
  let map;
  let activeTab = "Electrician_Locator";
  let markers = [];

  window.myMap = () => {
    const mapProp = {
      center: new google.maps.LatLng(51.508742, -0.12085),
      zoom: 7,
    };
    map = new google.maps.Map(document.getElementById("map"), mapProp);
    addMarkers();
  };

  function addMarkers() {
    markers.forEach((marker) => marker.setMap(null));
    markers = [];

    Locator.forEach((location) => {
      const marker = new google.maps.Marker({
        position: new google.maps.LatLng(location.lat, location.lng),
        map: map,
        title: location.title,
      });
      markers.push(marker);
    });
  }

  function changeTab(tab) {
    activeTab = tab;
    Locator = locationsData[tab];
    addMarkers();
  }

  onMount(() => {
    if (typeof google === "undefined") {
      const script = document.createElement("script");
      script.src = `https://maps.googleapis.com/maps/api/js?key=&callback=myMap`;
      script.async = true;
      script.defer = true;
      document.head.appendChild(script);
    } else {
      myMap();
    }
  });
</script>

<div id="map" class="map-container"></div>

<div class="tab-container">
  <div
    class="tab {activeTab === 'Electrician_Locator' ? 'active' : ''}"
    on:click={() => changeTab("Electrician_Locator")}
  >
    Electrician Locator
  </div>
  <div
    class="tab {activeTab === 'Shop_Locator' ? 'active' : ''}"
    on:click={() => changeTab("Shop_Locator")}
  >
    Shop Locator
  </div>
</div>

<style>
  #map {
    width: 100%;
    height: 50vh;
  }
  .tab-container {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    background-color: white;
    padding: 10px;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
    display: flex;
    gap: 20px;
    z-index: 1;
  }
  .tab {
    padding: 8px 16px;
    cursor: pointer;
    border-bottom: 2px solid transparent;
    font-weight: normal;
  }
  .active {
    border-bottom: 2px solid blue;
    font-weight: bold;
    color: blue;
  }
</style>
