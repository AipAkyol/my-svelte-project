<!-- src/routes/TimezoneDisplay.svelte -->
<script>
    import { onMount , onDestroy} from 'svelte';
    let city;
    let currentTime;
    let continent;
    let continentOptions = ['Africa', 'America', 'Asia', 'Antarctica', 'Australia', 'Atlantic', 'Europe', 'Indian', 'Pasific']; // Add continent options
    let header = "Please enter the continent and the city";
    let tempCity;
    let tempContinent;
    let isLoading = false;
    
    function loadingState(){
        isLoading = true;
    }

    function updateCityandContinent(){
        city = tempCity;
        continent = tempContinent;
    }
    
    function updateHeader(){
        header = "Current Time in " + city;
    }
    // Define a function to fetch the current time
    async function fetchCurrentTime() {
        try {
            const response = await fetch(`https://worldtimeapi.org/api/timezone/${continent}/${city}`);
            const data = await response.json();
            console.log(data);
            currentTime = data.datetime.slice(11,19);
            console.log(currentTime);
            isLoading = false;
        } catch (error) {
            isLoading = false;
            console.error('Error fetching data:', error);       
        }
    }
     
    // Initialize the interval when the component is mounted
  let interval;
  onMount(() => {
    interval = setInterval(fetchCurrentTime, 1000); // Update every 1000ms (1 second)
  });

  // Clear the interval when the component is destroyed
  onDestroy(() => {
    clearInterval(interval);
  });
  
  onMount(fetchCurrentTime);
  </script>

<main>
    <h1>{header}</h1>
    {#if isLoading}
      <p>Loading...</p>
    {:else}
      {#if currentTime}
        <p>{currentTime}</p>
      {:else}
        <p>Loading...</p>
      {/if}
    {/if}
  </main>

  <select bind:value={tempContinent}>
    {#each continentOptions as option}
      <option value={option}>{option}</option>
    {/each}
  </select>
  <input bind:value={tempCity} />
  <button on:click={loadingState}
            on:click={updateCityandContinent} 
             on:click={fetchCurrentTime} 
              on:click={updateHeader} 
              
  >Submit</button>
  