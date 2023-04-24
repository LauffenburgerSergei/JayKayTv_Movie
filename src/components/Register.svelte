<script>
  // Importation de la fonction "push" depuis le module "svelte-spa-router"
  import { push } from "svelte-spa-router";
  
  //////////// Register /////////////
  
  // Initialisation des variables pour stocker les données du formulaire
  let first_name = "";
  let last_name = "";
  let mail = "";
  let pwd = "";
  
  // Fonction qui génère un token aléatoire de 32 caractères
  function generateToken() {
    const tokenLength = 32;
    const characters = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    let token = '';
    for (let i = 0; i < tokenLength; i++) {
      token += characters.charAt(Math.floor(Math.random() * characters.length));
    }
    return token;
  }
  
  // Fonction qui gère la soumission du formulaire
  const handleSubmit = async (event) => {
    event.preventDefault(); // Empêche le formulaire d'être envoyé
  
    const data = {
      first_name: first_name,
      last_name: last_name,
      mail: mail,
      pwd: pwd,
    };
  
    try {
      const token = await register(data); // Appel de la fonction "register" qui envoie les données à l'API
      window.localStorage.setItem("token", token); // Stockage du token dans le localStorage
      console.log("Token:", token);
      push("/"); 
    } catch (error) {
      console.error(error); // Affichage d'une erreur éventuelle dans la console
    }
  };
  
  // Fonction qui envoie les données du formulaire à l'API pour enregistrer un nouvel utilisateur
  async function register(data) {
    const endpoint = import.meta.env.VITE_URL_DIRECTUS + "/users"; // URL de l'API pour enregistrer un nouvel utilisateur
    const membersRoleID = "d0490b37-d95c-4712-a471-04f281b360e7"; // ID du rôle "members" dans l'API Directus
  
    try {
      const response = await fetch(endpoint, { // Envoi des données à l'API avec la méthode POST
        method: "POST",
        headers: {
          "Content-Type": "application/json", // Format des données envoyées
        },
        body: JSON.stringify({
          email: data.mail,
          password: data.pwd,
          first_name: data.first_name,
          last_name: data.last_name,
          avatar: data.avatar,
          role: membersRoleID,
          status: "active",
        }),
      });
  
      if (response.ok) { // Si la réponse de l'API est OK
        const json = await response.json(); // Extraction des données de la réponse au format JSON
        const token = json.data.access_token; // Récupération du token d'authentification
        return token; // Retourne le token pour qu'il soit stocké dans le localStorage
      } else {
        throw new Error("Failed to register user"); // Sinon, affiche une erreur
      }
    } catch (error) {
      console.error(error); // Affiche une erreur éventuelle dans la console
    }
  }
</script>
  
  <main>
    
  
    <wrapper class="wrapper--right"
      ><form
        on:submit={handleSubmit}
        action=""
        method="post"
        id="inscription"
        name="inscription"
      >
        <section class="section--register" aria-labelledby="register">
          <h1 id="statistiques">S'enregistrer :</h1>
  
          
            
            <article
              class="article--register"
              aria-label="formulaire d'inscription"
            >
              <label for="first_name">Prénom :</label>
              <input 
                type="text" 
                name="first_name" 
                id="first_name" 
                bind:value={first_name} 
                required 
                />
            
              <label for="last_name">Nom :</label>
              <input
                type="text" 
                name="last_name" 
                id="last_name" 
                bind:value={last_name} 
                required 
                />
              <label for="email">E-mail : </label>
              <input
                type="email"
                name="email"
                id="email"
                bind:value={mail}
                required
              />
              <label for="pwd">Mot de passe : </label>
              <input
                type="password"
                name="pwd"
                id="pwd"
                bind:value={pwd}            
                required
              />
              <label for="avatar">Avatar : </label>
            <input type="file" name="avatar" id="avatar" accept="image/*">
            </article>
            <div class="buttons">
              <input
                class="submit"
                type="submit"
                name="submit"
                value="S'enregistrer"
                spellcheck="false"
                aria-label="s'enregistrer"
              />
              <input
                class="reset"
                type="reset"
                name="reset"
                value="Réinitialiser"
                aria-label="réinitialisation"
              />
            </div>
          
        </section>
      </form>
    </wrapper>
  </main>

  <style>
    /* Style pour le wrapper principal */
    main {
      display: flex;
      justify-content: center;
      align-items: center;
      
     
    }
  
    /* Style pour le formulaire */
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
  
    /* Style pour le titre */
    h1 {
      font-size: 2rem;
      margin-bottom: 1rem;
      text-align: center;
     
    }
  
    /* Style pour les champs de formulaire */
    label {
      display: block;
      text-align: center;
      font-size: 1rem;
      margin-bottom: 0.5rem;
      
      
    }
  
    input[type="text"],
    input[type="email"],
    input[type="password"] {
      width: 100%;
      padding: 0.5rem;
      border-radius: 5px;
      border: 1px solid #ccc;
      margin-bottom: 1rem;
     
      color: #282c34;
    }
  
    input[type="file"] {
      margin-bottom: 1rem;
    }
  
    /* Style pour les boutons */
    .buttons {
      display: flex;
      justify-content: space-between;
      width: 100%;
    }
  
    input[type="submit"],
    input[type="reset"] {
      padding: 0.5rem;
      border-radius: 5px;
      border: none;
      color: white;
      
      cursor: pointer;
      width: 45%;
    }
  
    input[type="submit"] {
      background-color: #970c11;
    }
  
    input[type="reset"] {
      background-color: #282c34;
    }
  </style>
  
  