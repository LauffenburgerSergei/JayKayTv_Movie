<script>
    //J'importe mon outil de travail
    import axios from "axios";
    
    //Je parametre le refresh sur non
    export let reload = false;

    //J'importe une fonctionnalité
    import {push} from "svelte-spa-router";

    //Je defini mes variables d'environnement qui vont transporter les infos du clients
    let last_name=""
    let first_name=""
    let email=""
    let password=""

    //Je parametre les erreur des champs en booleen sur non
    let error_boolean = false;

    /*
    Dans cette fonction, j'empeche l'execution par défaut, ensuite j'insères
    les variables d'environnement dans une variable en JSON afin de les utiliser avec
    Axios et ainsi les transmettre en API pour Directus. 
    Cependant pour cela si on envoie un objet JSON sérialisé en tant que données, Axios le considère 
    comme « application/x-www-form-urlencoded » (corps de requête codé par formulaire) alors il faut
    définir manuellement l'en-tête à l'aide de l'option de configuration "headers" si le type de contenu prévu est JSON.
    */
   function handleOnSubmit(event) {
        event.preventDefault()
        const data = JSON.stringify({last_name, first_name, email, password, status: "active", role: "d0490b37-d95c-4712-a471-04f281b360e7"});
        const options = {
            headers: {
                'Content-Type': 'application/json',
                //Coder une requete afin de recuperer le token d'admin à chaque fois qui donne l'autorisation de crée un user
                //ou bien donner la permissions au role "Public" de Directus de crée des users avec l'attribution du role "Private" (membre) par défaut et non modifiable par l'utilisateurs.
                //Dans ce cas plus besoin du token d'admin. 
                'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjgzNWM0ODdmLTc3Y2UtNDgxMi04ZWM0LTg5MWNkYjE0ZjU3ZiIsInJvbGUiOiJlZDFjODBkNy0wYmQzLTQwNjYtYTAxOS0wOWZiZDc4MDI4NzQiLCJhcHBfYWNjZXNzIjoxLCJhZG1pbl9hY2Nlc3MiOjEsImlhdCI6MTY4MTI4OTA5OCwiZXhwIjoxNjgxMjg5OTk4LCJpc3MiOiJkaXJlY3R1cyJ9.M1F3DZOeIIhKYRUdQsGqB91TVxDDHX4EfnXvJf31QU0'
            }
        }
        axios.post("http://localhost:8055/users", data, options)

        //Je verifie dans la console si tout est ok
        .then(function(res){
            console.log(res)
        })
        .catch(function(error){
            console.log(error)
        })

        if (reload) {
            location.reload();
        }
        else {
            //page register pour l'instant
            push('/')
        }
    }

    //Fonction qui me permet de verifier si l'adresse email est saisi et si elle est valide
    function validateMessageEmail(event) {
        let textbox = event.target;
        error_boolean = false;
        if (textbox.value === '') {
                textbox.setCustomValidity('Champ requis');
            } else if (textbox.validity.typeMismatch){
                error_boolean = true;
                textbox.setCustomValidity('Veuillez inserer une adresse email valide.');
            } else {
            textbox.setCustomValidity('');
            }
            return true;
    }
</script>

<form   on:submit={handleOnSubmit}
        on:invalid={validateMessageEmail}
        on:changed={validateMessageEmail}
        on:input={validateMessageEmail}>

<h2 class="titleregister">Inscription</h2>
    <div class="allform">

        <div class="infos">
            <label for="lastname">Nom :</label>
            <input required type="lastname" name="lastname" id="lastname" placeholder="Doe" bind:value={last_name}/>
        </div>

        <div class="infos">
            <label for="firstname">Prénom :</label>
            <input required type="firstname" name="firstname" id="firstname" placeholder="John" bind:value={first_name}/>
        </div>

        <div class="infos">
            <label for="email">Email :</label>
            <input required type="email" name="email" id="email" placeholder="johndoe@email.com" bind:value={email}/>
            {#if error_boolean}
                <h1>Erreur</h1>
            {/if}
        </div>

        <div class="infos">
            <label for="password">Mot de passe :</label>
            <input required type="password" name="password" id="password" placeholder="**********" bind:value={password}/>
        </div>

        <div class="divsubmitbtn">
            <button class="btnsubmit" type="submit">Crée un compte</button>
        </div>

    </div>

</form>