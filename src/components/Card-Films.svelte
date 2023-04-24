<script> 
    import axios from 'axios';
    import { onMount, setContext } from 'svelte';
    import { link } from 'svelte-spa-router';
    

    let movies = [];
    let genres = [];
    let selectedCountry = '';
    
    // Année en cours
    //initialisation d'une variable qui stocke l'année actuelle en utilisant l'objet Date
    const currentYear = new Date().getFullYear();
    
    // Options de filtre pour la recherche
    let selectedGenre = '';
    let selectedYear = currentYear;
    
    // Suivre la page actuelle pour charger les films suivants
    let page = 1;
    
    // Récupérer les genres disponibles dans l'API
    //Cette fonction asynchrone utilise l'API de TheMovieDB pour récupérer la liste des genres de film disponibles, 
    //les filtre pour retirer le genre adulte, puis stocke les genres dans la variable "genres".
    async function fetchGenres() {
      const response = await axios.get(`https://api.themoviedb.org/3/genre/movie/list?api_key=${import.meta.env.VITE_API_KEY}`);
      genres = response.data.genres.filter(genre => genre.id !== 27);// Enlever le genre adulte (id 27)
    }
    
    
    // Récupérer les films selon les options de filtre, classés par popularité
    //Cette fonction exécute une requête asynchrone vers l'API de TheMovieDB pour récupérer une liste de films selon les critères de recherche (année de sortie, genre, pays, page) spécifiés par l'utilisateur,
    // puis filtre les résultats en excluant les films coréens et en enlevant les films sans affiche. 
    //Enfin, elle met à jour la variable "movies" avec les résultats de la requête et fait défiler la page vers le haut.
    async function fetchMovies() {
      const response = await axios.get(`https://api.themoviedb.org/3/discover/movie?api_key=${import.meta.env.VITE_API_KEY}&language=fr&sort_by=popularity.desc&primary_release_year=${selectedYear}&with_genres=${selectedGenre}&with_original_language=${selectedCountry}&page=${page}`);
  movies = response.data.results.filter(movie => 
    movie.poster_path && 
    movie.original_language !== 'kr' 
    // Exclure les films coréens
  ); 
      window.scrollTo({ top: 0, behavior: 'smooth' }); 
      // Ajout de la méthode scrollTo() pour faire remonter la page en haut
    }
    
    // Appliquer les options de filtre pour la recherche de films
    //Cette fonction applique les filtres sélectionnés pour la recherche de films, réinitialise la page à la première page pour les nouveaux résultats de recherche,
    //exclut le genre adulte et le pays KR, et exécute la fonction fetchMovies() pour récupérer les nouveaux résultats de recherche.
    function applyFilters() {
      page = 1; // Réinitialiser la page à la première page pour les nouveaux résultats de recherche
      selectedGenre = selectedGenre !== 'excludeAdult' ? selectedGenre : ''; // Exclure le genre adulte
      selectedCountry = selectedCountry !== 'KR' ? selectedCountry : ''; // Exclure le pays KR
      fetchMovies();
      
    }
    
    
    // Fonction pour obtenir le nom du genre à partir de son ID
    //Cette fonction prend en paramètre un identifiant de genre et renvoie le nom correspondant en recherchant dans la liste des genres récupérée depuis l'API de TheMovieDB.
    //Si le genre est trouvé, le nom est renvoyé, sinon une chaîne de caractères vide est retournée.
    function getGenreName(genreId) {
      const genre = genres.find(g => g.id === genreId);
      return genre ? genre.name : '';
    }
    
    // Charger les films suivants
    //La fonction "loadMoreMovies" incrémente le numéro de page pour charger les films suivants en appelant la fonction "fetchMovies".
    function loadMoreMovies() {
      page++; // Incrémenter la page pour charger les films suivants
      fetchMovies();
    }
    
    //Ce code exécute deux fonctions asynchrones, "fetchGenres" et "fetchMovies", au moment du montage du composant Svelte,
    //en attendant que chaque fonction se termine avant de passer à la suivante.
    onMount(async () => {
      await fetchGenres();
      await fetchMovies();
    });
    
    // Ajoute la fonctionnalité de chargement de films à partir du contexte pour être utilisée par d'autres composants
    //La fonction "setContext" définit un contexte pour les composants enfants de Svelte. Ici, 
    //elle définit un contexte nommé "loadMoreMovies" qui est égal à la fonction "loadMoreMovies".
    setContext('loadMoreMovies', loadMoreMovies);
    </script>
    
    <div id="liste-films">
      <h1>Films :</h1>
      
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

      <div class="movie">
        {#each movies as movie}
          <div class="movie-card">
            <a use:link href={`/movie/${movie.id}`}>
              <img src={`https://image.tmdb.org/t/p/w500${movie.poster_path}`} alt={movie.title} />
              <div class="movie-card_description">
                <h3>{movie.title}</h3>
                <p>Note de popularité : {movie.popularity.toFixed(2)}</p>
              </div>
            </a>
          </div>
        {/each}
      </div>
      <div>
        <button on:click={loadMoreMovies}>Afficher les films suivants</button>
        <button on:click={() => {page--; fetchMovies()}} class="button is-primary" disabled={page === 1}>Retour</button>
      </div>
    </div>

  <style lang="scss">
    #liste-films {
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
          .movie {
          display: flex;
          flex-wrap: wrap;
          justify-content: space-around;
          margin: 0 60px;

            .movie-card {
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
            .movie-card_description {
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