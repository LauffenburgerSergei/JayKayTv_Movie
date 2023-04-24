<script>
    import { link } from 'svelte-spa-router';
    import logo from '../assets/logo_site.png';

    const isLogged = window.localStorage.getItem('token') != null;

   
    const nav_links = [
        {
            text: "Accueil",
            url: "/",
            label: ""
        },
        {
            text: "Films",
            url: "/films",
            label: "Affiche tous les films"
        },
        {
            text: "Séries",
            url: "/series",
            label: "Affiche toutes les séries"
        },
        {
            text: "Recherche",
            url: "/searchPage",
            label: "Rechercher un film ou une série"
        },        
        {
            text: isLogged ? "Se déconnecter" : "Se connecter",
            url: "/connexion",
            label: ""
        },
    ];
</script>

<nav aria-label="menu principal">
    <a use:link href="/" title="Co'clock Working" aria-label="accueil du site">
        <img src="{logo}" alt="Logo de l'entreprise">
    </a>

    <ul id="menu"  aria-labelledby="toggle-menu">

        {#each nav_links as nav}
            <li role="none">
                <a use:link href="{nav.url}" role="menuitem" aria-label="{nav.label}">{nav.text}</a>
            </li>
        {/each}
    </ul>

    <button type="button" id="toggle-menu" aria-controls="menu" aria-haspopup="true">
        <i id="menubutton" class="fa-solid fa-bars"></i>
    </button>
</nav>

<style lang="scss">
    nav{
        color: #970c11;
        padding: 0 40px;
        height: 100px;
        li{
            background-color: #970c11;
            border: none;
            border-radius: 10px;
            padding: 10px;
            margin-left: 10px;
            a{
                color: white;
            }
        }             
    }
</style>