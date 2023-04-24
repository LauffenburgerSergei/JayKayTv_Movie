<script>
  import axios from 'axios';
  import { onMount } from 'svelte';

  export let params = {}

  let serie = {};
  let actors = [];
  let userScore = "N/A";
  let showActors = false;
  let trailerId = '';
  let trailerUrl = '';

  //La fonction "openTrailer" est appelée lorsqu'un utilisateur clique sur le bouton de lecture de la bande-annonce. 
  //Elle permet d'afficher la modal contenant la vidéo de la bande-annonce en définissant l'URL de la vidéo dans l'attribut "src" de l'élément "trailerVideo". 
  //Elle ajoute également un événement de clic sur le contenu de la modal qui permet de cacher la modal et d'arrêter la lecture de la vidéo si l'utilisateur 
  //clique sur le bouton Fermer ou en dehors de la modal.
  function openTrailer() {
    const modal = document.querySelector(".modal");
    const modalContent = document.querySelector(".modal-content");
    const trailerVideo = document.querySelector(".trailer-video");
    // @ts-ignore
    modal.style.display = "block"; 
    trailerVideo.setAttribute("src", trailerUrl); 
    modalContent.addEventListener("click", function(event) {
    // @ts-ignore
      if (event.target.classList.contains("close") || event.target.classList.contains("modal")) { 
        // @ts-ignore
        modal.style.display = "none"; 
        trailerVideo.removeAttribute("src"); 
      }
    });
  }
  
  //Ce code récupère la bande-annonce d'une série TV à partir de l'API The Movie Database en filtrant les résultats pour ne récupérer que les vidéos de type "Trailer". 
  //Si une bande-annonce est trouvée, elle est stockée dans la variable "trailerUrl" sous forme d'URL embed YouTube.
  async function fetchTrailer(serieId) {
    const response = await axios.get(`https://api.themoviedb.org/3/tv/${serieId}/videos?api_key=${import.meta.env.VITE_API_KEY}&language=fr`);
    const videos = response.data.results.filter(video => video.type === "Trailer");
    if (videos.length > 0) {
      trailerId = videos[0].key;
      trailerUrl = `https://www.youtube.com/embed/${trailerId}`;
    }
  }

  //Ce code est exécuté lors du montage du composant et récupère les informations d'une série TV depuis l'API TheMovieDB. 
  //Ensuite, il récupère la bande-annonce de la série, le score utilisateur et la liste des acteurs en utilisant d'autres fonctions asynchrones.
  onMount(async () => {
    const id = params.id;
    const response = await axios.get(`https://api.themoviedb.org/3/tv/${id}?api_key=${import.meta.env.VITE_API_KEY}&language=fr`);
    serie = response.data;
    await fetchTrailer(id);

    if (serie.vote_average !== undefined && serie.vote_count > 0) {
      userScore = (serie.vote_average * 10).toFixed(0) + "%";
    }
    actors = await fetchActors(id);
  });

  //Cette fonction asynchrone envoie une requête à l'API de TheMovieDB pour obtenir la liste des acteurs d'une série TV, 
  //en filtrant les acteurs qui n'ont pas de photo de profil et en ne gardant que les 10 premiers résultats.
  async function fetchActors(serieId) {
    const response = await fetch(`https://api.themoviedb.org/3/tv/${serieId}/credits?api_key=${import.meta.env.VITE_API_KEY}&language=fr`);
    const data = await response.json();
    return data.cast.filter(actor => actor.profile_path !== null).slice(0, 10);
  }

  //Cette fonction permet de changer l'état d'une variable booléenne 'showActors' en alternant entre vrai et faux.
  function toggleActors() {
    showActors = !showActors;
  }

  //Cette fonction permet de revenir en arrière dans l'historique de navigation du navigateur 
  //web lorsque l'utilisateur clique sur un bouton "retour" dans l'interface utilisateur.
  function goBack() {
    window.history.back();
  }
</script>

<div class="button-return">
    <span></span>
    <button class="return" on:click={goBack}>x</button>
</div>
<div class="bgc-info" style="background-image: url(https://image.tmdb.org/t/p/w1280/{serie.backdrop_path}); background-size: cover; ">
    <div class="info-serie">
      <img src={`https://image.tmdb.org/t/p/w500/${serie.poster_path}`} alt={`Affiche de ${serie.name}`} />
      <div class="info-serie_details">
        <h1>{serie.name}</h1>
        <p>{new Date(serie.first_air_date).toLocaleDateString('fr')}</p>
        <p>{serie.number_of_seasons} saisons</p>
        <p>{serie.number_of_episodes} épisodes</p>
        <p class=".synopsis">{serie.overview}</p>
        <div class="circle-score" style="position: relative; width: 55px; height: 55px;">
          <svg viewBox="0 0 36 36" style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;">
            <path class="circle-bg"
                  d="M18 2.0845
                     a 15.9155 15.9155 0 0 1 0 31.831
                     a 15.9155 15.9155 0 0 1 0 -31.831"
                  fill="none"
                  stroke="#2bcb7e"
                  stroke-width="3.8"
                  stroke-dasharray="100, 100"
                  transform="rotate(-90 18 18)" />
            <path class="circle"
                  d="M18 2.0845
                     a 15.9155 15.9155 0 0 1 0 31.831
                     a 15.9155 15.9155 0 0 1 0 -31.831"
                  fill="none"
                  stroke="#ccc"
                  stroke-width="3.8"
                  stroke-dasharray="{userScore}, 100"
                  stroke-linecap="round"
                  transform="rotate(-90 18 18)" />
          </svg>
          <div style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: flex; justify-content: center; align-items: center;">
            <p style="font-size: 18px; font-weight: bold; color: #fff;">{userScore}</p>
          </div>
        </div>
        <div class="button-info">
          <button on:click={openTrailer}>Bande annonce</button>
          <button on:click={toggleActors}>Liste des acteurs</button>
        </div>
      </div>
    </div>
</div>
{#if showActors}
<div id="modal-actors" class="actor_modal">
  <div class="modal-content_actor">
    <button class="close-modal" on:click={toggleActors}>X</button>  
    <ul class="actors">
      {#each actors as actor}
        <li>
          <img src={`https://image.tmdb.org/t/p/w500/${actor.profile_path}`} alt={`Photo de ${actor.name}`} />
          <div class="name-character">
            <p>{actor.name} <br/> ({actor.character})</p>
          </div>
        </li>
      {/each}
    </ul>
  </div>
</div>
{/if}   
<div class="modal">
  <div class="modal-content">
    <span class="close">&times;</span>
    <iframe class="trailer-video" width="300" height="300" frameborder="0" allowfullscreen title="bande annonce"></iframe>
  </div>
</div>

<style lang="scss">
     .button-return {
  display: flex;
  justify-content: space-between;
  button {
    display: flex;
    justify-content: center;
    height: 40px;
    width: 40px;
    background-color: #970c11;
    color: #fff;
    border: none;
    border-radius: 50px;
    margin: 30px;
    cursor: pointer;
    transition: all 0.3s ease;
    font-size: 2rem;
    margin-left: 50px;
    transform: translateY(80px); 
  }
}
div {
  .info-serie {
    display: flex;
    padding: 40px 150px;
    background-image: linear-gradient(to right, rgba(31.5, 31.5, 31.5, 1) calc((50vw - 170px) - 340px), rgba(31.5, 31.5, 31.5, 0.84) 50%, rgba(31.5, 31.5, 31.5, 0.84) 100%);
    img {
      height: 450px;
      margin: 0 40px;
      border-radius: 15px;
      border: 3px solid #970C11;
    }
    .info-serie_details {
      color: #fff;
      font-size: 1.2rem;
      border-radius: 15px;
      padding: 0 40px;
      h1 {
        font-size: 2.2rem;
        font-weight: bold;
        color: #ffffff;
      }
      p {
        padding: 10px 0;
        line-height: 1.3rem;

        .synopsis {
          width: 250px;
        }
      }
      button {
        background-color: #970C11;
        border: none;
        margin-top: 20px;
        padding: 15px;
        border-radius: 15px;
        color: #fff;
        cursor: pointer;
      }
    }
  }
}
ul {
  list-style: none;
  padding: 0;
  margin: 0;
}
li {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}
.no-image {
  width: auto;
  height: 150px;
  background-color: #ccc;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 20px;
  color: #000;
  font-size: 0.8em;
}
p {
  margin: 0;
  font-size: 1em;
}
.button-info {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
/*----------------------------------------------------------*/
/*------------------Modal pour les acteurs------------------*/
/*----------------------------------------------------------*/
.actor_modal {
position: fixed;
z-index: 9999;
left: 0;
top: 0;
width: 100%;
height: 100%;
overflow: auto;
background-color: rgba(0, 0, 0, 0.4);
display: flex;
justify-content: center;
align-items: center;
}
.modal-content_actor {
background-color: rgba(255, 255, 255, 0.8);
padding: 0 10px 15px 10px;
border-radius: 5px;
box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.3);
max-width: 80%;
max-height: 80%;
overflow: auto;
overflow: hidden;
}
.close-modal {
  background-color: #970c11;
  border: none;
  color: white;
  padding: 5px 10px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin-top: 20px;
  border-radius: 10px;
  cursor: pointer;
}
.actors {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  list-style: none;
  padding: 0;
  margin: 0;

  li {
    width: calc(15% - 40px);
    margin: 0 10px 10px 10px;
    background-color: #fff;
    box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.3);
    border-radius: 5px;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

    img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
  }

  .name-character {
    padding: 10px;
    text-align: center;
    width: 100%;
    background-color: #970c11;
    color : white;
  }
}

/*----------------------------------------------------------*/
/*------------------Modal pour les vidéos-------------------*/
/*----------------------------------------------------------*/
.modal {
display: none; 
position: fixed; 
z-index: 1; 
left: 0;
top: 0;
width: 100%;
height: 100%;
background-color: rgba(0,0,0,0.5); 
}
.modal-content {
background-color: #fff;
padding: 20px;
border: 1px solid #888;
width: 80%;
height: 490px;
max-width: 863px;/* La largeur maximale de la modal */
position: relative;
transform: translate(-50%, -50% );
top: 50%;
left: 50%;
}
.close {
position: absolute;
top: 0;
right: 0;
font-size: 30px;
font-weight: bold;
color: #aaa;
padding: 5px 10px;
background-color: #970C11;
border-radius:50%;
}
.close:hover,
.close:focus {
color: #000;
text-decoration: none;
cursor: pointer;
}
.trailer-video {
width: 100%;
height: 100%;
}
</style>