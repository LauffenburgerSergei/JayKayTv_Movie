<script>
  import axios from 'axios';
  import { link } from 'svelte-spa-router';
 
  let query = '';
  let results = [];
  let searchComplete = false;

  function handleSearch(event) {
    event.preventDefault();
    axios.get('https://api.themoviedb.org/3/search/multi', {
      params: {
        api_key: import.meta.env.VITE_API_KEY,
        query: query
      }
    })
    .then(response => {
      results = response.data.results.filter(result => result.poster_path);
      searchComplete = true;
    })
    .catch(error => {
      searchComplete = true;
    });
  }
</script>

<section>
  <form on:submit={handleSearch} class="search-form">
    <input type="text" placeholder="Recherche de film ou de série" class="search-input" bind:value={query}>
    <button type="submit" class="search-button">Rechercher</button>
  </form>
  <div class="movie-results">
    {#each results as result}
      {#if result.media_type === 'movie'}
        <div class="movie-card">
          <a use:link href={`/movie/${result.id}`} >
          {#if result.poster_path}
            <img src={`https://image.tmdb.org/t/p/w500${result.poster_path}`} alt={result.title} />
          {:else}
            <div class="no-image">Pas d'image disponible</div>
          {/if}
          <div class="movie-details">
            <h2>{result.title}</h2>
          </div>
        </a>
        </div>
      {:else if result.media_type === 'tv'}
        <div class="movie-card">
          <a use:link href={`/serie/${result.id}`} >
          {#if result.poster_path}
            <img src={`https://image.tmdb.org/t/p/w500${result.poster_path}`} alt={result.name} />
          {:else}
            <div class="no-image">Pas d'image disponible</div>
          {/if}
          <div class="movie-details">
            <h2>{result.name}</h2>
          </div>
        </a>
        </div>
      {/if}
    {/each}
    {#if searchComplete && results.length === 0 && query}
      <p>Aucun résultat trouvé pour "{query}"</p>
    {/if}
  </div>
</section>
  <style >
    form{
      display: flex;
      justify-content: center;
      margin: 60px;
    }
    .search-form {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 20px 0;
  }
  .search-input {
    padding: 10px;
    font-size: 16px;
    border: none;
    border-radius: 5px 0 0 5px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
    outline: none;
    width: 40%;
    margin-right: -5px;
  }
  .search-button {
    padding: 10px;
    background-color: #970c11;
    color: #fff;
    font-size: 16px;
    border: none;
    border-radius: 0 5px 5px 0;
    cursor: pointer;
    outline: none;
    transition: background-color 0.3s ease;
  }
  .search-button:hover {
    background-color: #e6e7eb;
    color: #970c11;
    font-weight: bold;
  }
   .movie-results{
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
    margin: 0 60px;
   }
   .movie-card{
        position: relative; /* Ajout de la position relative */
        width: calc(20% - 30px);
        margin-bottom: 20px;
        border-radius: 15px;
        overflow: hidden;    
      }
      img {
        width: 100%;
        height: 100%;
        object-fit: cover;
      }
      .movie-details{
        display: flex;
        justify-content: center;
        align-items: center;
        position: absolute; /* Ajout de la position absolute */
        bottom: 0;
        left: 0;
        width: 100%;
        height: 50px;
        background-color: #970c11;
      }
      h2 {
        text-align: center;
        font-size: 1rem;
        color: #e6e7eb;
        background-color: #970c11;
        padding: 5px;
        border-radius: 5px;
      }
      P{
        color: #e6e7eb;
        background-color: #970c11;
        height: auto;
        width: auto;
        padding: 10px;
        border-radius: 10px;
      }
  </style>

  
  
