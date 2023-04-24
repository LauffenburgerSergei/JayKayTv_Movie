<script>
    import  Router from 'svelte-spa-router';
    
    import Home from './lib/Home.svelte';
    import Films from './lib/Films.svelte';
    import Series from './lib/Series.svelte';
    import Connexion from './lib/Connexion.svelte';
    import NotFound from './lib/NotFound.svelte';
    import MovieDetails from './lib/MovieDetails.svelte';
    import SerieDetails from './lib/SerieDetails.svelte';
    import Footer from './components/Footer.svelte';
    import Header from './components/Header.svelte';
    import SearchPage from './lib/SearchPage.svelte';
 
    
    
    
    
    
    const routes = {
        '/': Home,
        '/Films': Films,
        '/series' : Series,
        '/movie/:id': MovieDetails,
        '/serie/:id': SerieDetails, 
        '/connexion': Connexion,
        '/searchPage' : SearchPage,
        '*': NotFound,
    }
    
</script>
<Header/>
<main>
    <Router {routes} />
</main>
<Footer/>
<style>
    
</style>



