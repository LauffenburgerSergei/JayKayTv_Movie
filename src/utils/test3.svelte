<script>
    var details = document.querySelectorAll("details");
  
    details.forEach(function (detail) {
      detail.addEventListener("click", function () {
        details.forEach(function (otherDetail) {
          if (otherDetail !== detail) {
            otherDetail.open = false;
          }
        });
      });
    });
  
    // Importation de la fonction "push" depuis le module "svelte-spa-router"
    import { push } from "svelte-spa-router";
  
    // Déclaration de la variable "reload" initialisée à "false"
    export let reload = false;
  
    // Déclaration des variables "email" et "password" pour le formulaire de login
    let email = "";
    let password = "";
  
    // Vérifie si l'utilisateur est déjà connecté en vérifiant la présence du token d'authentification dans le local storage
    const isLogged = window.localStorage.getItem("token") != null;
  
    // Si l'utilisateur est déjà connecté, on supprime le token du local storage et on recharge la page
    if (isLogged) {
      window.localStorage.removeItem("token");
      location.reload();
    }
  
    // Fonction appelée lors de la soumission du formulaire de login
  const handleSubmitForm = async (event) => {
    event.preventDefault(); // Empêche la soumission du formulaire
  
    // Appelle la fonction "login" qui retourne un objet contenant le token d'authentification et l'ID du rôle
    const { token, roleID } = await login();
  
    // Enregistre le token d'authentification dans le local storage
    window.localStorage.setItem("token", token);
    console.log("Token:", token);
  
    // Si la variable "reload" est vraie, on recharge la page
    if (reload) {
      location.reload();
    } else {
      // Redirige l'utilisateur vers la page de profil correspondante en fonction de l'ID du rôle
      if (roleID === "213b3c24-fb05-446d-ab79-fd05adbbd6e2") {
        push("/");
      } else if (roleID === "645cbe7e-cdf9-409c-bc58-863ce065dfbb") {
        push("/");
      } else {
        console.error("Unknown role ID:", roleID);
      }
    }
  };  const response = await fetch(endpoint, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          email: email,
          password: password,
        }),
      });
  
      // Extrait le contenu de la réponse au format JSON
      const json = await response.json();
  
      // Récupère le token d'authentification depuis le JSON
      const token = json.data.access_token;
  
      // Retourne le token d'authentification
      return token;
    };
    //////////// Register /////////////
  
   // Initialisation des variables pour stocker les données du formulaire
  let pseudo = "";
  let mail = "";
  let pwd = "";
  
  // Fonction qui gère la soumission du formulaire
  const handleSubmit = async (event) => {
    event.preventDefault(); // Empêche le formulaire d'être envoyé
  
    const data = { // Création de l'objet qui contient les données du formulaire
      pseudo: pseudo,
      mail: mail,
      pwd: pwd,
    };
  
    try {
      const token = await register(data); // Appel de la fonction "register" qui envoie les données à l'API
      window.localStorage.setItem("token", token); // Stockage du token dans le localStorage
      console.log("Token:", token);
      push("/profil-membre"); // Redirection vers la page du profil membre
    } catch (error) {
      console.error(error); // Affichage d'une erreur éventuelle dans la console
    }
  };
  
  // Fonction qui envoie les données du formulaire à l'API pour enregistrer un nouvel utilisateur
  async function register(data) {
    const endpoint = import.meta.env.VITE_URL_DIRECTUS + "/users"; // URL de l'API pour enregistrer un nouvel utilisateur
    const membersRoleID = "213b3c24-fb05-446d-ab79-fd05adbbd6e2"; // ID du rôle "members" dans l'API Directus
  
    try {
      const response = await fetch(endpoint, { // Envoi des données à l'API avec la méthode POST
        method: "POST",
        headers: {
          "Content-Type": "application/json", // Format des données envoyées
        },
        body: JSON.stringify({ // Encodage des données du formulaire en JSON
          email: data.mail,
          password: data.pwd,
          pseudo: data.pseudo,
          role: membersRoleID,
          status: "active", // Statut du nouvel utilisateur
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
    <wrapper class="wrapper--left">
      <!-- dans action mettre le nom de la page (ex: /profil)  -->
      <form
        on:submit={handleSubmit}
        action=""
        method="post"
        id="connexion"
        name="connexion"
      >
        <section class="section--login" aria-labelledby="login">
          <h1 id="login">Se connecter</h1>
  
          <details open>
            <summary>Voir le formulaire</summary>
            <article class="article--login" aria-label="formulaire de connexion">
              <label for="email">E-mail : </label>
              <input
                type="email"
                name="email"
                bind:value={mail}
                id="email"
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
            </article>
            <div class="buttons">
              <input
                class="submit"
                type="submit"
                name="submit"
                value="Se connecter"
                spellcheck="false"
                aria-label="Se connecter"
              />
              <input
                class="reset"
                type="reset"
                name="reset"
                value="Réinitialiser"
                aria-label="réinitialisation"
              />
            </div>
          </details>
        </section>
      </form>
    </wrapper>
  
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
  
          <details>
            <summary>Voir le formulaire</summary>
            <article
              class="article--register"
              aria-label="formulaire d'inscription"
            >
              <label for="pseudo">Pseudo : </label>
              <input
                type="text"
                name="pseudo"
                id="pseudo"
                bind:value={pseudo}
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
          </details>
        </section>
      </form>
    </wrapper>
  </main>
  
  