 <script> 
    import { onMount } from "svelte";
    
  
    let user = null;
    
  
    const fetchUser = async () => {
      const token = window.localStorage.getItem('token');
      const endpoint = `${import.meta.env.VITE_URL_DIRECTUS}/users/me`;
  
      const response = await fetch(endpoint, {
        headers: {
          Authorization: `Bearer ${token}`
        }
      });
  
      const json = await response.json();
      user = json.data;
      console.log(user);
      return user;
    }
  
    onMount(fetchUser);
  
    
  </script>
  
  <header>
    {#if user}
      <img class="avatar" src={import.meta.env.VITE_URL_DIRECTUS + '/admin/files/' + user.avatar + '?key=large'} alt="Avatar de {user.email}" />
      <h2>{user.first_name} {user.last_name}  </h2>
    {/if}
  </header>


  <style>
    header{
        display: flex;
        flex-direction: column;
        align-items: center;
        color: white;
    }
  
  </style> 