<script>
  import { push } from "svelte-spa-router";

  export let reload = false;

  let email;
  let password;

  const isLogged = window.localStorage.getItem('token') != null;
  if ( isLogged ) {
      window.localStorage.removeItem('token');
      location.reload();
  }

  const handleSubmitForm = async (event) => {
      event.preventDefault();
      const token = await login();
      window.localStorage.setItem('token', token);

      if ( reload ) {
          location.reload();
      }
      else {
          // Naviguer vers la page d'accueil grace à svelte-spa-router
          push('/');
      }
  }

  const login = async () => {
      const endpoint = import.meta.env.VITE_URL_DIRECTUS + "/auth/login";

      const response = await fetch(endpoint, {
          method: "POST",
          headers: {
              'Content-Type': 'application/json',
          },
          body: JSON.stringify({
              "email": email,
              "password": password
          })
      });

      // Extraction du contenu du body de la réponse au format json
      const json = await response.json();

      // Lit uniquement le token du json
      const token = json.data.access_token
      return token;
  }
</script>




<form on:submit={handleSubmitForm} aria-label="Informations de connexion">
  <h2 id="title1">Se connecter</h2>
  <label for="email">Email</label>
  <input required type="email" name="email" id="email" bind:value={email} placeholder="ex : m.dupont@monmail.com">

  <label for="password">Mot de passe</label>
  <input required type="password" name="password" bind:value={password} id="password" placeholder="***********">

  <input type="submit" value="Se connecter">

</form>
<style>
    #title1 {
      font-size: 2rem;
      margin-bottom: 1rem;
      text-align: center;
      color: white;
    }
    
    form {
      display: flex;
      flex-direction: column;
      align-items: center;
      max-width: 400px;
      
      padding: 2rem;
      border: 1px solid #ddd;
      border-radius: 5px;
      color: white;
    }
    
    label {
      display: block;
      font-weight: bold;
      margin-bottom: 0.5rem;
    }
    
    input[type="email"],
    input[type="password"] {
      padding: 0.5rem;
      margin-bottom: 1rem;
      border: 1px solid #ccc;
      border-radius: 3px;
      width: 100%;
      max-width: 300px;
    }
    
    input[type="submit"] {
      background-color: #970c11;
      color: white;
      padding: 0.5rem;
      border: none;
      border-radius: 3px;
      width: 100%;
      max-width: 200px;
      cursor: pointer;
      transition: background-color 0.3s ease-in-out;
    }
    
    input[type="submit"]:hover {
      background-color: #b85255;
    }
  </style>