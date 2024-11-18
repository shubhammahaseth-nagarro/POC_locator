<script>
  import { onMount } from "svelte";
  const locationsData = {
    Electrician_Locator: [
      { lat: 28.7041, lng: 77.1025, title: "Delhi" },
      { lat: 19.076, lng: 72.8777, title: "Mumbai" },
    ],
    Shop_Locator: [
      { lat: 13.0827, lng: 80.2707, title: "Chennai" },
      { lat: 22.5726, lng: 88.3639, title: "Kolkata" },
    ],
  };

  let Locator = locationsData.Electrician_Locator;
  let map;
  let activeTab = "Electrician_Locator";
  let markers = [];
  let autocomplete;

  window.myMap = () => {
    const mapProp = {
      center: new google.maps.LatLng(20.5937, 78.9629), // Center map on India
      zoom: 5,
    };
    map = new google.maps.Map(document.getElementById("map"), mapProp);
    addMarkers();
    initAutocomplete();
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

  function initAutocomplete() {
    const input = document.getElementById("location-search");
    const options = {
      componentRestrictions: { country: "in" }, // Restrict to India
      fields: ["geometry", "name"],
    };
    autocomplete = new google.maps.places.Autocomplete(input, options);

    autocomplete.addListener("place_changed", () => {
      const place = autocomplete.getPlace();
      if (place.geometry && place.geometry.location) {
        map.setCenter(place.geometry.location);
        map.setZoom(12);
      }
    });
  }

  onMount(() => {
    if (typeof google === "undefined") {
      const script = document.createElement("script");
      script.src = `https://maps.googleapis.com/maps/api/js?key=${import.meta.env.VITE_GOOGLE_MAP_KEY}&libraries=places&callback=myMap`;
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

<div class="search-container">
  <input
    id="location-search"
    type="text"
    placeholder="Enter pincode or address"
  />
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
    color: black;
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
  .search-container {
    position: absolute;
    bottom: 100px;
    left: 50%;
    transform: translateX(-50%);
    width: 80%;
    background-color: white;
    padding: 10px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
    border-radius: 8px;
    z-index: 1;
  }
  #location-search {
    width: 100%;
    padding: 8px;
    font-size: 16px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
</style>
