<template>
  <!-- Contenu du composant Utilisateur ici -->
  <div class="home-page">

    <div class="navbar">
      <div class="container">
        <router-link to="/" class="nav-link" exact>
          Dashboard
        </router-link>
        <router-link to="/orders" class="nav-link">
          Orders
        </router-link>
        <router-link to="/products" class="nav-link">
          Products
        </router-link>
        <router-link to="/customers" class="nav-link">
          Customers
        </router-link>
      </div>
    </div>

    <div class="container mt-4" v-if="!userExists">
      <div class="row">
        <div class="col-6">
          <h2 class="text-center"> Connexion </h2>
          <form @submit.prevent="submitUser">
            <div class="user-form">
              <div class="form-group">
                <label for="email">Email :</label>
                <input type="email" id="email" v-model="formData.email" required>
              </div>
              <div class="form-group">
                <label for="username">Nom d'utilisateur :</label>
                <input type="text" id="username" v-model="formData.username" required>
              </div>
              <button  type="submit">Récupérer les données utilisateur</button>
            </div>
          </form>
        </div>

        <div class="col-6">
          <h2 class="text-center"> Inscription </h2>
          <form class="  mx-auto user-form">
            <div class="form-group">
              <label for="email">Email :</label>
              <input type="email" class="form-control" id="email" v-model="user.email" placeholder=" entre votre email "/>
            </div>
            <div class="form-group">
              <label for="email">Nom :</label>
              <input type="text" class="form-control" id="nom" v-model="user.username"  placeholder=" entre votre nom "/>
            </div>
            <div class="form-group ">
              <button class="btn btn-success w-100" type="button" @click="createUser()">Creation</button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <div v-if="userExists" class="user-info">
      <div class="container">
        <div class="card">
          <div class="card-header">
            <h2 class="text-center">Informations de l'utilisateur </h2>
          </div>
          <div class="card-body">
            <table class="table table-bordered mt-3 text-center">
              <thead>
              <tr>
                <th>ID</th>
                <th>Nom d'utilisateur </th>
                <th>Email</th>
                <th>Actions</th>
              </tr>
              </thead>
              <tbody>
              <tr>
                <td>{{ userId }}</td>
                <td>{{ formData.username }}</td>
                <td>{{ formData.email }}</td>
                <td class="d-flex  justify-content-center gap-3 ">
                  <button @click="badgeClick()" class="btn btn-success">Je Badge !</button>
                  <button class="btn btn-secondary" type="button" @click="showWorkingTimes=true;getWorkingTimes()"> Mes Working Times</button>

                  <button type="button" @click="showModel" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#staticBackdrop">Modifier</button>
                  <button @click="supprimerUser(this.userId)" class="btn btn-danger">Supprimer</button>
                </td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <div v-if="userExists" class="user-info">
      <div class="container">
        <div class="card">
          <div class="card-header">
            <h2 class="text-center">Votre journée </h2>
          </div>
          <div class="card-body">
            <table class="table table-bordered mt-3 text-center">
              <thead>
              <tr>
                <th>Temps </th>
              </tr>
              </thead>
              <tbody>

              <tr v-for="(clock, index) in this.clocks" :key="index">
                <td> Badge n°{{++index}} :  {{ formattedDate(clock.time) }}</td>
              </tr>

              </tbody>
            </table>
          </div>
        </div>
      </div>

      <div v-if="showWorkingTimes" class="container">

        <div class="card">
          <div class="card-header">
            <h2 class="text-center">Vos working times</h2>
          </div>
          <div class="card-body">
            <table class="table table-bordered mt-3 text-center">
              <thead>
              <tr>
                <th>Début</th>
                <th>Fin</th>
                <th>Durée de travail</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="(working, index) in workingTimeData" :key="index">
                <td>{{ formatWorkingTime(working.start) }}</td>
                <td>{{ formatWorkingTime(working.end) }}</td>
                <td>{{ calculateWorkingDuration(working.start, working.end) }}</td>

              </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

    </div>

    <div class="user-info" v-if="showErrorMessage">
      <p>L'utilisateur n'existe pas dans la base de données.</p>
    </div>




    <div class="modal fade" id="staticBackdrop" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="staticBackdropLabel">Modifier l'utilisateur</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <form >
              <div class="user-form">
                <div class="form-group">
                  <label for="email">Email :</label>
                  <input type="email" id="email" v-model="formData.email" required>
                </div>
                <div class="form-group">
                  <label for="username">Nom d'utilisateur :</label>
                  <input type="text" id="username" v-model="formData.username" required>
                </div>
                <button class="btn btn-info" type="button" @click="editUser(this.userId)" data-bs-dismiss="modal" aria-label="Close">Modifier </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
    </div>

</template>
<script>
let nbClick =0;

import moment from 'moment';
import axios from 'axios';
import 'moment-timezone';
import { onMounted } from 'vue';
import * as bootstrap from 'bootstrap';
import 'bootstrap';
window.bootstrap = bootstrap;
let myModal;

export default {
  setup() {

    onMounted(() => {
      myModal = new bootstrap.Modal(document.getElementById('staticBackdrop'));
    });

    const showModel = () => {
      console.log("tes");
      myModal.show();
    };

    return {
      // ...autres variables et méthodes que vous retournez
      showModel
    };
  },
  name: 'GestionUtilisateurs',

  data() {
    return {
      formData: {
        email: '',
        username: '',
      },
      user: {
        email: '',
        username: '',
        idUser: '',
      },
      data: {
        time: '',
        status: '',
        user_id: '',
      },
      clocks: {
        time: '',
        status: '',
        user_id: '',
      },
      workingTimeData: {
        start: '',
        end: '',
        user_id: ''
      },
      showWorkingTimes: false,

      userId: '',
      workingTime_id: null,
      userExists: false, // Ajout d'une variable pour suivre si l'utilisateur existe
      email: '',
      username: '',
      showErrorMessage: false,
      isEditing: false,
      startDay: null,
    };
  },
  computed: {},
  methods: {

    calculateWorkingDuration(start, end) {
      const startTime = moment(start);
      const endTime = moment(end);
      const duration = moment.duration(endTime.diff(startTime));
      const hours = Math.floor(duration.asHours());
      const minutes = duration.minutes();

      return `${hours} heures ${minutes} minutes`;
    },

    formatWorkingTime(date) {
      return moment(date).subtract(1, 'hour').format('YYYY-MM-DD HH:mm');
    },
    async createClock(user_id) {
      try {
        const currentDate = moment().tz('Europe/Paris');
        const data = {
          time: currentDate,
        }
        console.log(data);
        const response = await axios.post(`http://localhost:4000/api/clocks/${user_id}`, {clock: data});
        console.log('Clock créée avec succès:', response.data);

      } catch (error) {
        console.error('Erreur lors de la création de la clock :', error);
      }
    },


    badgeClick() {
      const today = moment().format('YYYY-MM-DD H:mm');

      if (nbClick === 0) {
        this.createClock(this.userId);
        this.startDay = today;
        this.workingTimeData.start = this.startDay
        this.workingTimeData.user_id = this.userId
        this.workingTimeData.end = "2023-10-30T16:34:00Z"
        console.log(this.workingTimeData)

        axios.post(`http://localhost:4000/api/working_times/${this.userId}`, {working_time: this.workingTimeData})
            .then(response => {
              //  //Traitement à effectuer après la création du working time
              const workingTimeId = response.data.id;
              console.log('Working time créé avec succès:', response.data);
              this.workingTime_id = workingTimeId;
              nbClick = 1;
            }).catch(error => {
          console.error('Erreur lors de la création du working time :', error);
        });

      }
      else if (nbClick == 1) {
        this.createClock(this.userId);
        this.workingTimeData.end = today
        console.log(this.workingTime_id)
        console.log("boucle 2")

        //JE DOIS RECUPERER L'ID DE WORKING TIME ET LE PASSER EN PARAM
        axios.put(`http://localhost:4000/api/working_times/${this.workingTime_id}`, {working_time: this.workingTimeData})
            .then(response => {
              //Traitement à effectuer après la création du working time
              console.log('Working time a été PUT avec succès:', response.data);
            }).catch(error => {
          console.error('Erreur lors de la création du working time :', error);
        });
      }
      nbClick = 0;
    },

  async getWorkingTimes() {
    try {
      // Effectuer une requête GET pour obtenir les détails de l'utilisateur
      const response = await axios.get(`http://localhost:4000/api/working_times/${this.userId}`);
      this.workingTimeData = response.data;
      console.log(this.workingTimeData)
      // Mettez en œuvre la logique nécessaire après avoir obtenu les détails de l'utilisateur ici
    } catch (error) {
      console.error('Erreur lors de la récupération des temps de travail :', error);
    }
  },


  async createUser() {
    try {
      let dataUser = this.user;
      // Effectuer une requête POST pour créer un nouvel utilisateur
      const response = await axios.post('http://localhost:4000/api/users', {"user": dataUser})
      console.log('Utilisateur créé avec succès:', response.data);
      // Mettez en œuvre la logique nécessaire après la création de l'utilisateur ici
      this.user = ''
    } catch (error) {
      console.error('Erreur lors de la création de l\'utilisateur :', error);
    }
  },

  formattedDate(date) {
    return moment.tz(date, 'Europe/Paris').format('DD-MM-YYYY H:mm');
  },

  async getClock() {
    try {
      // Effectuer une requête GET pour obtenir les détails de l'utilisateur
      const response = await axios.get(`http://localhost:4000/api/clocks/${this.userId}`);
      this.clocks = response.data;

      // Mettez en œuvre la logique nécessaire après avoir obtenu les détails de l'utilisateur ici
    } catch (error) {
      console.error('Erreur lors de la récupération des temps de travail :', error);
    }
  },

  async editUser(id) {
    try {
      const dataToUpdate = {
        email: this.formData.email,
        username: this.formData.username,
      };

      // Effectuer une requête PUT pour mettre à jour l'utilisateur existant
      const response = await axios.put(`http://localhost:4000/api/users/${id}`, {user: dataToUpdate});

      console.log('Utilisateur mis à jour avec succès :', response.data);
      // Mettez en œuvre la logique nécessaire après la mise à jour de l'utilisateur ici
      this.isEditing = false;
    } catch (error) {
      console.error('Erreur lors de la mise à jour de l\'utilisateur :', error);
    }
  },

  async getUser() {
    try {
      // Effectuer une requête GET pour obtenir les détails de l'utilisateur
      const response = await axios.get(`http://localhost:4000/api/users/${this.userId}`);
      console.log('Détails de l\'utilisateur récupérés avec succès:', response.data);
      // Mettez en œuvre la logique nécessaire après avoir obtenu les détails de l'utilisateur ici
    } catch (error) {
      console.error('Erreur lors de la récupération des détails de l\'utilisateur :', error);
    }
  },

  async supprimerUser(id) {
    try {
      if (confirm("Veuillez confirmer votre suppression")) {
        const response = await axios.delete(`http://localhost:4000/api/users/${id}`);
          console.log('Utilisateur supprimé avec succès:', response.data);

          this.userExists = false;
      }
    }  catch (error) {
            console.error('Erreur lors de la suppression de l\'utilisateur :', error);
    }
  },


  async submitUser() {
    const email = this.formData.email;
    const username = this.formData.username;
    try {
      // Effectuez une requête à l'API pour vérifier si l'utilisateur existe
      const response = await axios.get(`http://localhost:4000/api/users?email=${email}&username=${username}`);

      if (response.data) {
        this.userExists = true;
        this.email = email;
        this.username = username;
        this.showErrorMessage = false;
        // Récupérer l'ID de l'utilisateur
        this.userId = response.data.id

        this.getUser();
        this.getClock();
      } else {
        this.userExists = false;
        this.showErrorMessage = true;
        // L'utilisateur n'existe pas dans la base de données
        console.log(response.data);
      }
    } catch (error) {
      this.userExists = false;
      this.showErrorMessage = true;
    }
  },
}
};

</script>

<style scoped>
/* Styles pour la page d'accueil */

/* Style pour la barre de navigation */
.navbar {
  background-color: beige;
  padding: 15px 0; /* Ajoutez un espace autour du texte dans la navbar */
  width: 100%; /* Faites en sorte que la navbar occupe toute la largeur de son conteneur */
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1); /* Ajoute une légère ombre */
}

/* Style pour les liens dans la navbar */
.navbar a {
  margin: 0 20px; /* Ajoute un espace entre chaque lien */
  color: black; /* Couleur du texte des liens */
  text-decoration: none; /* Supprime la soulignement des liens */
  font-weight: bold; /* Mettez en gras le texte des liens */
  font-size: 1.2rem; /* Taille de la police des liens */
}





/* Changez la couleur du texte des liens au survol */
.navbar a:hover {
  color: darkred; /* Couleur du texte des liens au survol */
}



/* Vos autres styles restants... */

/* Style pour le formulaire utilisateur */
.user-form {
  max-width: 400px;
  margin: 20px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  text-align: center;
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.user-form .form-group {
  margin-bottom: 15px;
}

.user-form .form-group label {
  display: block;
  font-weight: bold;
  margin-bottom: 5px;
}

.user-form .form-group input[type="email"],
.user-form .form-group input[type="text"] {
  width: calc(100% - 20px);
  padding: 8px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-bottom: 10px;
}

.user-form button[type="submit"] {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.user-form button[type="submit"]:hover {
  background-color: #0056b3;
}
</style>
