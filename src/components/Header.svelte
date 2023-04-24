<script>
    import Menu from "./Menu.svelte";
    window.addEventListener('popstate', () => {
    window.location.reload();
  });
</script>

<header>
    <Menu/>
</header>

<style>
 
</style>