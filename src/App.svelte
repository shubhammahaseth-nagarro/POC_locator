<script>
  import { onMount } from "svelte";

  let Locator;
  let map;
  let activeTab = "Electrician_Locator";
  let markers = [];
  let autocomplete;
  let locationsList = [];

  // API base URL
  const API_BASE_URL = "https://electrician-poc-backend.vercel.app/api/stores";

  window.myMap = () => {
    const mapProp = {
      center: new google.maps.LatLng(20.5937, 78.9629), // Center map on India
      zoom: 5,
    };
    map = new google.maps.Map(document.getElementById("map"), mapProp);
    fetchLocations("electrician");
    initAutocomplete();
  };

  async function fetchLocations(role, pincode = "", radius = "") {
    try {
      const url = new URL(API_BASE_URL);
      if (pincode) url.searchParams.append("pincode", pincode);
      url.searchParams.append("role", role);
      if (radius) url.searchParams.append("radius", radius);

      const response = await fetch(url);
      const data = await response.json();

      if (data[0]?.location) {
        Locator = data?.map((loc) => ({
          lat: loc.location.coordinates[1],
          lng: loc.location.coordinates[0],
          title: loc.address || "Unknown",
        }));
        locationsList = data;
        addMarkers();
      } else if (!data?.stores?.length) {
        locationsList = [];
        alert("No stores found!");
      } else {
        console.error(
          "Failed to fetch locations:",
          data.message || "Unknown error"
        );
      }
    } catch (error) {
      console.error("Error fetching locations:", error);
    }
  }

  function addMarkers() {
    markers.forEach((marker) => marker.setMap(null));
    markers = [];

    Locator.forEach((location) => {
      const marker = new google.maps.Marker({
        position: new google.maps.LatLng(location.lat, location.lng),
        map: map,
        title: location.title,
      });

      marker.addListener("click", () => {
        map.setZoom(12);
        map.setCenter(marker.getPosition());
      });

      markers.push(marker);
    });
  }

  function zoomToMarker(index) {
    const marker = markers[index];
    if (marker) {
      map.setCenter(marker.getPosition());
      map.setZoom(12);
      scrollToTop()
    }
  }

  function changeTab(tab) {
    activeTab = tab;
    const role = tab === "Electrician_Locator" ? "electrician" : "shop";
    Locator = [];
    locationsList = [];
    markers.forEach((marker) => marker.setMap(null));
    markers = [];
    map.setZoom(5);
    map.setCenter(new google.maps.LatLng(20.5937, 78.9629));
    document.getElementById("location-search").value = "";
    document.getElementById("radius-input").value = "";
    fetchLocations(role);
  }

  function initAutocomplete() {
    const input = document.getElementById("location-search");
    const radiusInput = document.getElementById("radius-input");
    const options = {
      // componentRestrictions: { country: "in" }, // Restrict to India
      fields: ["geometry", "name"],
    };
    autocomplete = new google.maps.places.Autocomplete(input, options);

    autocomplete.addListener("place_changed", () => {
      const place = autocomplete.getPlace();
      if (place.geometry && place.geometry.location) {
        const { lat, lng } = place.geometry.location.toJSON();
        map.setCenter(new google.maps.LatLng(lat, lng));
        map.setZoom(12);
        const radius = radiusInput?.value || "";
        fetchLocations(
          activeTab === "Electrician_Locator" ? "electrician" : "shop",
          place.name,
          radius
        );
      }
    });
  }

  function extractPincode(address) {
    const pincodeRegex = /\b\d{6}\b/;
    const match = address.match(pincodeRegex);
    return match ? match[0] : "";
  }

  function handleSearch() {
    const input = document.getElementById("location-search");
    const radiusInput = document.getElementById("radius-input");
    const address = input.value.trim();
    const pincode = extractPincode(address);
    const radius = radiusInput.value.trim();

    if (pincode) {
      fetchLocations(
        activeTab === "Electrician_Locator" ? "electrician" : "shop",
        pincode,
        radius
      );
    } else {
      alert("Please enter a valid pincode or address.");
    }
  }

  // Function to scroll to the top
  const scrollToTop = () => {
    window.scrollTo({
      top: 0,
      behavior: "smooth"
    });
  };


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
  <input id="radius-input" type="text" placeholder="Enter radius (optional)" />
  <button on:click={handleSearch}>Search</button>
</div>

<div class="results-container">
  {#if locationsList.length > 0}
    <h3>
      Available {activeTab === "Electrician_Locator"
        ? "Electricians"
        : "Shops"}:
    </h3>
    <ul>
      {#each locationsList as loc, i}
        <li
          class="location-item"
          data-index={i}
          on:click={() => zoomToMarker(i)}
        >
          <strong>{loc.name || "Unknown Name"}</strong><br />
          {loc.address || "Unknown Address"}
        </li>
      {/each}
    </ul>
  {:else}
    <p>No locations to display.</p>
  {/if}
</div>

<style>
  body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f5f5f5;
  }
  #map {
    width: 100%;
    height: 50vh;
  }

  .tab-container {
    display: flex;
    justify-content: center;
    margin-top: 20px;
    padding: 10px 20px;
    background-color: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  }

  .tab {
    padding: 10px 20px;
    cursor: pointer;
    border-bottom: 2px solid transparent;
    font-size: 16px;
    transition:
      color 0.3s ease,
      border-color 0.3s ease;
  }

  .tab:hover {
    color: #007bff;
  }

  .active {
    border-bottom: 2px solid #007bff;
    font-weight: bold;
    color: #007bff;
  }

  .search-container {
    margin: 20px auto;
    padding: 15px 20px;
    background-color: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    display: flex;
    justify-content: space-between;
    gap: 15px;
    max-width: 800px;
  }

  #location-search,
  #radius-input {
    flex: 1;
    padding: 10px;
    font-size: 16px;
    border: 1px solid #ddd;
    border-radius: 4px;
    outline: none;
    transition: border-color 0.3s ease;
  }

  #location-search:focus,
  #radius-input:focus {
    border-color: #007bff;
  }

  button {
    padding: 10px 20px;
    font-size: 16px;
    color: #fff;
    background-color: #007bff;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: #0056b3;
  }

  .results-container {
    max-width: 800px;
    margin: 20px auto;
    padding: 20px;
    background: #fff;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  }

  .results-container h3 {
    margin-bottom: 10px;
    font-size: 18px;
    color: #333;
  }

  .results-container ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .results-container li {
    margin-bottom: 10px;
    font-size: 16px;
    line-height: 1.5;
  }

  .results-container li strong {
    display: block;
    color: #007bff;
  }

  .location-item {
    cursor: pointer;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    margin-bottom: 10px;
    transition: background-color 0.3s ease;
  }

  .location-item:hover {
    background-color: #f1f1f1;
  }
</style>
