<!-- src/routes/TimezoneDisplay.svelte -->
<script>
  import { onMount, onDestroy } from 'svelte';

  let continent;
  let city;
  let currentTime;
  let header = "Please enter the continent and the city";
  let tempCity = "";
  let tempContinent = "";
  let isLoading = false;
  let continentsData = {};
  let citiesOfSelectedContinent = [];
  let citySelectPlaceholder = "Please select a continent first";
  let isClockStarted = false; // Add a flag to track whether the clock should be updated

  function loadingState() {
    isLoading = true;
  }

  function updateURL() {
    continent = tempContinent;
    city = tempCity;
  }
  async function fetchData() {
    try {
      const response = await fetch('https://worldtimeapi.org/api/timezone');
      const data = await response.json();

      // Iterate through the data and organize it into continents and cities
      data.forEach(timezone => {
        const parts = timezone.split('/');
        const dataContinent = parts[0];
        let dataCity;
        if (parts.length>2){
          dataCity = parts[1] + "/" + parts[2];
        }
        else{
          dataCity = parts[1];
        }
        if ((dataCity == undefined) || (dataContinent == "Etc")) {
          return;
        }
        // Create a continent entry if it doesn't exist
        if (!continentsData[dataContinent]) {
          continentsData[dataContinent] = [];
        }

        continentsData[dataContinent].push(dataCity);
      });
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  }

  // Fetch data when the component is mounted
  onMount(() => {
    fetchData();
  });

  function handleContinentChange(event) {
    tempContinent = event.target.value;
    // Enable the city select and populate it with cities of the selected continent
    citiesOfSelectedContinent = continentsData[tempContinent] || [];
    tempCity = ''; // Reset selected city when the continent changes

    // Set the city select placeholder based on whether a continent is selected
    citySelectPlaceholder = tempContinent ? "Please select a city" : "Please select a continent first";
  }

  async function fetchCurrentTime() {
    if (!isClockStarted) return; // Do not fetch the time if the flag is not set
    try {
      const response = await fetch(`https://worldtimeapi.org/api/timezone/${continent}/${city}`);
      const data = await response.json();
      currentTime = data.datetime.slice(11, 19);
      isLoading = false;
    } catch (error) {
      isLoading = false;
      console.error('Error fetching data:', error);
    }
  }

  let interval;
  onMount(() => {
    interval = setInterval(fetchCurrentTime, 1000); // Update every 1000ms (1 second)
  });

  onDestroy(() => {
    clearInterval(interval);
  });

</script>

<main>
  <h1>{header.replace(/_/g, " ")}</h1>
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

<label for="continentSelect">Select a Continent:</label>
<select id="continentSelect" bind:value={tempContinent} on:change={handleContinentChange}>
  {#each Object.keys(continentsData) as continent}
    <option value={continent}>{continent}</option>
  {/each}
</select>

<label for="citySelect">Select a City:</label>
<select id="citySelect" bind:value={tempCity} disabled={!tempContinent}>
  <option value="" disabled>{citySelectPlaceholder}</option>
  {#each citiesOfSelectedContinent as city}
    <option value={city}>{city.replace(/_/g, " ")}</option>
  {/each}
</select>

<button
  on:click={loadingState}
  on:click={updateURL}
  on:click={() => {
    isClockStarted = true; // Set the flag to start updating the clock
    fetchCurrentTime(); // Manually fetch the time when the button is clicked
  }}
  on:click={()=> {
    header = "Time in " + city.replace(/\//g, " ") + " is:"
  }
  } 
  disabled={!tempCity}
>Submit</button>
