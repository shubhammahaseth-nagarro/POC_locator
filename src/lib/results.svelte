<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  export let locationsList;
  export let activeTab;

  function handleClick(id) {
    dispatch('itemClicked', id);
  }
</script>

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
          role="presentation"
          data-index={i}
            on:click={() => handleClick(i)}
        >
          <strong
            >{loc.name || "Unknown Name"}
            {loc?.distanceInKm ? ` - ${loc.distanceInKm} km` : ""}
          </strong> <br />
          {loc.address || "Unknown Address"}
        </li>
      {/each}
    </ul>
  {:else}
    <p>No locations to display.</p>
  {/if}
</div>

<style>
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
