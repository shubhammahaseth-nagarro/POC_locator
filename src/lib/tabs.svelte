<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();
  export let activeTab = "Electrician_Locator";

  function changeTab(tab) {
    activeTab = tab;
    dispatch("tabChanged", { activeTab });
  }
</script>

<div class="tab-container">
  <div role="presentation"
    class="tab {activeTab === 'Electrician_Locator' ? 'active' : ''}"
    on:click={() => changeTab("Electrician_Locator")}
  >
    Electrician Locator
  </div>
  <div role="presentation"
    class="tab {activeTab === 'Shop_Locator' ? 'active' : ''}"
    on:click={() => changeTab("Shop_Locator")}
  >Shop Locator</div>
</div>

<style>
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
</style>
