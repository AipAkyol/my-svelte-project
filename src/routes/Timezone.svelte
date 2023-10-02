<!-- src/routes/TimezoneDisplay.svelte -->
<script>
    import { onMount , onDestroy} from 'svelte';
    let city="";
    let currentTime;
    let continent="";
    let header = "Please enter the continent and the city";
    let tempCity;
    let tempContinent;
    let isLoading = false;
    let continentsData = {};
    let citiesOfSelectedContinent = [];
    let citySelectPlaceholder = "Please select a continent";
    
    function loadingState(){
        isLoading = true;
     }

    function updateCityandContinent(){
        city = tempCity;
        continent = tempContinent;
        tempCity = undefined;
        tempContinent = undefined;
    }
    
    function updateHeader(){
        header = "Current Time in " + city;
    }

    async function fetchData() {
    try {
      const response = await fetch('https://worldtimeapi.org/api/timezone');
      const data = await response.json();

      // Iterate through the data and organize it into continents and cities
      data.forEach(timezone => {
        const parts = timezone.split('/');
        const dataContinent = parts[0];
        const dataCity = parts[1];
        if ((dataCity == undefined) || (dataContinent == "Etc")){
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

  function changePlaceholder(){
    citySelectPlaceholder = "Please select a city";
  }
  function handleContinentChange(event) {
    continent = event.target.value;
    // Enable the city select and populate it with cities of the selected continent
    citiesOfSelectedContinent = continentsData[continent] || [];
    city = ''; // Reset selected city when the continent changes
  }  
    // Define a function to fetch the current time
    async function fetchCurrentTime() {
        try {
          const response = await fetch(`https://worldtimeapi.org/api/timezone/${continent}/${city}`);
            const data = await response.json();
            currentTime = data.datetime.slice(11,19);
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
  <select id="continentSelect" bind:value={tempContinent} on:change={handleContinentChange} on:change={changePlaceholder}>
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
            on:click={updateCityandContinent} 
             on:click={fetchCurrentTime} 
              on:click={updateHeader} 
              disabled={!tempCity}
  >Submit</button>
  