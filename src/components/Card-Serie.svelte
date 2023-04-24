<script>
  import axios from 'axios';
  import { onMount, setContext } from 'svelte';
  import { link } from 'svelte-spa-router';

  let series = [];
  let genres = [];
  let selectedCountry = '';
  
  // Année en cours
  const currentYear = new Date().getFullYear();

  // Options de filtre pour la recherche
  let selectedGenre = '';
  let selectedYear = currentYear;

  // Suivre la page actuelle pour charger les séries suivantes
  let page = 1;

  // Récupérer les genres disponibles dans l'API
  async function fetchGenres() {
    const response = await axios.get(`https://api.themoviedb.org/3/genre/tv/list?api_key=${import.meta.env.VITE_API_KEY}`);
    genres = response.data.genres.filter(genre => genre.id !== 10768); // Enlever le genre de la réalité
  }

  // Récupérer les séries selon les options de filtre, classées par popularité
  async function fetchSeries() {
    let response;
    if (selectedCountry === 'en') { // si le pays est anglais
      response = await axios.get(`https://api.themoviedb.org/3/discover/tv?api_key=${import.meta.env.VITE_API_KEY}&language=fr&sort_by=popularity.desc&first_air_date_year=${selectedYear}&with_genres=${selectedGenre}&page=${page}&with_original_language=en`);
    } else { // pour les autres pays
      response = await axios.get(`https://api.themoviedb.org/3/discover/tv?api_key=${import.meta.env.VITE_API_KEY}&language=fr&sort_by=popularity.desc&first_air_date_year=${selectedYear}&with_genres=${selectedGenre}&page=${page}`);
    }
    const results = response.data.results.filter(serie => serie.poster_path);
    // trier les séries américaines et européennes en premier
    series = results.sort((a, b) => {
      if (a.origin_country.includes('US') || a.origin_country.includes('GB')) {
        return -1;
      } else if (b.origin_country.includes('US') || b.origin_country.includes('GB')) {
        return 1;
      } else {
        return 0;
      }
    });
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }

  // Appliquer les options de filtre pour la recherche de séries
  function applyFilters() {
    page = 1; // Réinitialiser la page à la première page pour les nouveaux résultats de recherche
    selectedGenre = selectedGenre !== 'excludeReality' ? selectedGenre : ''; // Exclure le genre de la réalité
    selectedCountry = selectedCountry !== 'KR' ? selectedCountry : '';
    fetchSeries();
  }

  // Fonction pour obtenir le nom du genre à partir de son ID
  function getGenreName(genreId) {
    const genre = genres.find(g => g.id === genreId);
    return genre ? genre.name : '';
  }

  // Charger les séries suivantes
  function loadMoreSeries() {
    page++; // Incrémenter la page pour charger les séries suivantes
    fetchSeries();
  }

  // Charger les genres et les séries au moment du montage du composant
  onMount(async () => {
    await fetchGenres();
    await fetchSeries();
  });
  
    // Ajoute la fonctionnalité de chargement de séries à partir du contexte pour être utilisée par d'autres composants
    setContext('loadMoreSeries', loadMoreSeries);
  </script>
<div id="liste-series">
  <h1>Séries :</h1>
<section>
    <label>
      <p>Genre :</p>
      <select bind:value={selectedGenre} on:change={applyFilters}>
        <option value="">Tous les genres</option>
        {#each genres as genre}
        <option value={genre.id}>{genre.name}</option>
        {/each}
      </select>
    </label>
    <label>
      <p>Année :</p>
      <input type="number" bind:value={selectedYear} on:change={applyFilters} min="1900" max={currentYear} step="1" />
    </label>
    <label for="country-filter">
      <p>Pays :</p>
      <select id="country-filter" bind:value={selectedCountry} on:change={applyFilters}>
        <option value="">Tous</option>
        <option value="en">Anglais</option>
        <option value="fr">Français</option>
        <option value="es">Espagnol</option>
        <option value="it">Italien</option>
        <option value="de">Allemand</option>
      </select>
    </label>
  </section>
  
  <div class="serie">
    {#each series as serie}
    <div class="serie-card">
      <a use:link href={`/serie/${serie.id}`}>
        <img src={`https://image.tmdb.org/t/p/w500${serie.poster_path}`} alt={serie.title} />
        <div class="serie-card_description">
          <h3>{serie.name}</h3>
          <p>Note de popularité : {serie.popularity.toFixed(2)}</p>
        </div>
      </a>
    </div>
    {/each}
  </div>
  <div>
    <button on:click={loadMoreSeries}>Afficher les séries suivantes</button>
    <button on:click={() => {page--; fetchSeries()}} class="button is-primary" disabled={page === 1}>Retour</button>
  </div>
</div>
  
  <style lang="scss">
    #liste-series {
      display: flex;
      flex-direction: column;
        h1{
          display: flex;
          justify-content: center;
          margin: 0 30px 60px;
          font-size: 2rem;
          font-weight: bold;
          color: #e6e7eb;
        }
        section {
            display: flex;
            flex-wrap: wrap;
            justify-content: space-around;
            align-items: center;
            margin-bottom: 1rem;
            
            label {
              display: flex;
              align-items: center;
              margin-bottom: 0.5rem;
              
              p {
                margin-right: 0.5rem;
                font-weight: bold;
                color: #e6e7eb;
                font-size: 1rem;
              }              
              select,
              input[type="number"] {
                padding: 0.5rem;
                border-radius: 10px;
                border: 1px solid #ccc;
                font-size: 1rem;
                
                &:focus {
                  outline: none;
                  border-color: #555;
                }
              }             
              select {
                background-color: #fff;
              }            
              input[type="number"] {
                width: 6rem;
                background-color: #f9f9f9;
              }             
              select  {
                margin-left: 1rem;
              }
            }        
            label[for="country-filter"] {
              margin-right: 0.5rem;
              font-weight: bold;
              color: #e6e7eb;
              font-size: 1rem;
            }           
            select#country-filter {
              padding: 0.5rem;
              border-radius: 10px;
              border: 1px solid #ccc;
              font-size: 1rem;
              background-color: #fff;            
              &:focus {
                outline: none;
                border-color: #555;
              }
            }
          }
          .serie {
          display: flex;
          flex-wrap: wrap;
          justify-content: space-around;
          margin: 0 60px;
            .serie-card {
            position: relative; /* Ajout de la position relative */
            width: calc(20% - 30px);
            margin-bottom: 20px;
            border-radius: 15px;
            border: 3px solid #970c11;
            overflow: hidden;
            img {
              width: 100%;
              height: 100%;
              object-fit: cover;
            }
            .serie-card_description {
              display: flex;
              justify-content: center;
              align-items: center;
              position: absolute; /* Ajout de la position absolute */
              bottom: 0;
              left: 0;
              width: 100%;
              height: 50px;
              background-color: #970c11;
              h3 {
                font-size: 1rem;
                color: #e6e7eb;
                background-color: #970c11;
                padding: 5px;
                border-radius: 5px;
              }
              p {
                display: none;
              }
             }
            }
          }
          div {
            display: flex;
            justify-content: center;
            margin-top: 20px;            
            button {
              font-size: 1rem;
              padding: 10px 20px;
              border: none;
              border-radius: 5px;
              margin: 0 10px;
              cursor: pointer;
              transition: all 0.2s ease-in-out;
              margin-bottom: 50px;             
              &:hover {
                opacity: 0.8;
              }              
              &:disabled {
                opacity: 0.5;
                cursor: not-allowed;
              }
            }
          }
        }
    
  </style>