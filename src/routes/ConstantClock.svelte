<!-- Clock.svelte -->
<script>
    import { onMount, onDestroy } from 'svelte';
    export let constantCity;
    export let constantContinent;
    let currentTime = "";
    let isLoading = true; // Initialize as loading
  
    async function updateTime() {
      try {
        const response = await fetch(`https://worldtimeapi.org/api/timezone/${constantContinent}/${constantCity}`);
        const data = await response.json();
        currentTime = data.datetime.slice(11, 19);
        isLoading = false;
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    }
  
    let interval;
    onMount(() => {
      updateTime(); // Initial update
      interval = setInterval(updateTime, 1000);
    });
  
    onDestroy(() => {
      clearInterval(interval); // Cleanup the interval when the component is unmounted
    });
  </script>
  
  <div class="clock"> <!-- Add class here -->
    <h2>Current time in {constantCity.replace(/_/g, " ")}</h2>
    {#if isLoading}
      <p>Loading...</p>
    {:else}
      <p>{currentTime}</p>
    {/if}
  </div>
  
  <style>
    .clock {
      text-align: center;
      font-size: 16px;
      font-weight: bold;
      margin: 20px;
      display: inline-block;
    }
    p {
        font-weight: normal; /* Change 'normal' to the desired font weight */
    }
  </style>