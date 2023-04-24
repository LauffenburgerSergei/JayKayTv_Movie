<script>
  import { onMount } from "svelte";
  import axios from "axios";

  let posters = [];

  const getMoviePosters = async () => {
    
    const current_year = new Date().getFullYear();
    let posters_by_year = [];

    for (let year = current_year; year >= current_year - 49; year--) {
      try {
        const response = await axios.get(`https://api.themoviedb.org/3/discover/movie?api_key=${import.meta.env.VITE_API_KEY}&sort_by=popularity.desc&primary_release_year=${year}&include_adult=false`);
        const most_popular_movie = response.data.results.find(movie => movie.original_language !== "ko" && movie.adult === false);
        
        if (most_popular_movie) {
          posters_by_year.push(most_popular_movie.poster_path);
        }

        if (posters_by_year.length >= 33) {
          break;
        }
        
      } catch (error) {
        console.error(error);
      }
    }

    posters = shuffleArray(posters_by_year);
  };
  

  const shuffleArray = (array) => {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  };

  onMount(async () => {
    await getMoviePosters();
  });
</script>
<div class="container">
  {#if posters.length}
    <div class="poster-grid">
      {#each posters as poster}
      <img class="poster" src={`https://image.tmdb.org/t/p/w300${poster}`} alt="movie poster">
      {/each}
      <div class="logo-overlay">
        <img src="/src//assets/logo_site.png" alt="logo">
      </div>
    </div>
  {:else}
    <p></p>
  {/if}
</div>
<style lang="scss">
  .container {
    position: relative;
    margin: 20px 0;
  } 
  .poster-grid {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    position: relative;
    overflow: hidden;
  }
  .poster {
    width: 150px;
    margin: 10px;
    transform: rotate(10deg);
    border: 2px solid #970c11;
    border-radius: 15px;
    opacity: 0.5;
  } 
  .logo-overlay {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
 
  .logo-overlay img {
    width: auto;
    height: 700px;
  }
</style>