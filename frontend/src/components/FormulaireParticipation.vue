<template>
  <div class="conteneur-formulaire">
    <h2>Formulaire de participation</h2>

    <!-- Sélection de la personne -->
    <div class="groupe-formulaire">
      <label>Personne</label>
      <select v-model="personneSelectionnee">
        <option :value="null" disabled> Veuillez sélectionner une personne </option>
        <option
          v-for="pers in listePersonnes"
          :key="pers.matricule"
          :value="pers.matricule"
        >
          {{ pers.nom }} {{ pers.prenom }}
        </option>
      </select>
    </div>

    <!-- Sélection du projet -->
    <div class="groupe-formulaire">
      <label>Projet</label>
      <select v-model="projetSelectionne">
        <option :value="null" disabled> Veuillez sélectionner un projet </option>
        <option
          v-for="proj in listeProjets"
          :key="proj.id"
          :value="proj.id"
        >
          {{ proj.nom }}
        </option>
      </select>
    </div>

    <!-- Rôle -->
    <div class="groupe-formulaire">
      <label>Poste</label>
      <input type="text" v-model="poste" placeholder="Ex: Développeur front-end" />
    </div>

    <!-- Pourcentage (curseur) -->
    <div class="groupe-formulaire">
      <label>Taux d'implication</label>
      <div class="conteneur-curseur">
        <input
          type="range"
          min="0"
          max="100"
          step="1"
          v-model="tauxImplication"
          class="curseur"
        />
        <span class="valeur-curseur">{{ tauxImplication }}%</span>
      </div>
    </div>

    <!-- Bouton Valider -->
    <button class="bouton-valider" @click="sauvegarderParticipation">Valider</button>

    <!-- Affichage de l'erreur si besoin -->
    <div v-if="messageErreur" class="message-erreur">
      {{ messageErreur }}
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, toRefs } from 'vue'
import requeteAjax from '@/util/util.js'

const etat = reactive({
  listePersonnes: [],
  listeProjets: [],
  personneSelectionnee: null,
  projetSelectionne: null,
  poste: '',
  tauxImplication: 0,
  messageErreur: '',
})

const {
  listePersonnes,
  listeProjets,
  personneSelectionnee,
  projetSelectionne,
  poste,
  tauxImplication,
  messageErreur
} = toRefs(etat)

function chargerPersonnes() {
  requeteAjax('http://localhost:8989/api/personnes')
    .then((res) => {
      etat.listePersonnes = res._embedded ? res._embedded.personnes : []
    })
    .catch((err) => {
      etat.messageErreur = err.message
    })
}

function chargerProjets() {
  requeteAjax('http://localhost:8989/api/projets')
    .then((res) => {
      etat.listeProjets = res._embedded ? res._embedded.projets : []
    })
    .catch((err) => {
      etat.messageErreur = err.message
    })
}

function sauvegarderParticipation() {
  etat.messageErreur = ''

  if (!etat.personneSelectionnee || !etat.projetSelectionne) {
    etat.messageErreur = 'Sélectionnez une personne et un projet.'
    return
  }
  if (!etat.poste) {
    etat.messageErreur = 'Précisez le poste.'
    return
  }
  if (etat.tauxImplication <= 0) {
    etat.messageErreur = 'Le pourcentage doit être > 0.'
    return
  }

  const fraction = etat.tauxImplication / 100

  const url =
    `http://localhost:8989/api/gestion/participation?matricule=${etat.personneSelectionnee}` +
    `&codeProjet=${etat.projetSelectionne}` +
    `&poste=${encodeURIComponent(etat.poste)}` +
    `&taux=${fraction}`

  requeteAjax(url, { method: 'POST' })
    .then((result) => {
      console.log('Réponse du serveur :', result)
      alert('Participation enregistrée avec succès !')
      etat.personneSelectionnee = null
      etat.projetSelectionne = null
      etat.poste = ''
      etat.tauxImplication = 0
    })
    .catch((err) => {
      etat.messageErreur = err.message
    })
}

onMounted(() => {
  chargerPersonnes()
  chargerProjets()
})
</script>
<style scoped>

.conteneur-formulaire {
  max-width: 500px;
  margin: 2rem auto;
  padding: 2.5rem;
  background-color: #1a1a1a;
  color: #e0e0e0;
  border: 1px solid #2e2e2e;
}


.conteneur-formulaire h2 {
  margin-bottom: 2rem;
  font-size: 1.7rem;
  font-weight: 700;
  color: #fff;
  text-align: center;
  letter-spacing: 1px;
  text-transform: uppercase;
  text-shadow: 0 2px 4px rgba(0,0,0,0.3);
}


.groupe-formulaire {
  color: #c0c0c0;
  font-size: 1rem;
  margin-bottom: 0.5rem;
}


.groupe-formulaire label {
  display: block;
  margin-bottom: 0.75rem;
  font-weight: 500;
  color: #b0b0b0;
  font-size: 0.95rem;
}


.groupe-formulaire select,
.groupe-formulaire input[type="text"] {
  width: 100%;
  padding: 1rem;
  font-size: 1rem;
  background-color: #4A9EFFFF;
  border: 2px solid #3a3a3a;

  color: #fff;
  transition: all 0.3s ease;
}

.groupe-formulaire select:focus,
.groupe-formulaire input[type="text"]:focus {
  border-color: #4a9eff;
  outline: none;
  box-shadow: 0 0 0 2px rgba(74, 158, 255, 0.2);
}


.groupe-formulaire select option {
  background-color: #2a2a2a;
  color: #fff;
  padding: 8px;
}


.conteneur-curseur {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}


.curseur {
  flex: 1;
  height: 6px;
  background: #404040;
  border-radius: 3px;
  appearance: none;
}

.curseur::-webkit-slider-thumb {
  appearance: none;
  width: 20px;
  height: 20px;
  background: #4a9eff;
  border-radius: 50%;
  cursor: pointer;
  transition: background 0.2s ease;
}

.curseur::-webkit-slider-thumb:hover {
  background: #3a8eff;
}


.valeur-curseur {
  min-width: 60px;
  padding: 0.5rem;
  background: #2a2a2a;
  border-radius: 6px;
  text-align: center;
  font-weight: 500;
  color: #4a9eff;
}


.bouton-valider {
  width: 100%;
  padding: 0.9rem;
  margin-top: 1rem;
  font-size: 1rem;
  font-weight: 500;
  color: #fff;
  background-color: #5aa9fa;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  letter-spacing: 0.8px;
  box-shadow: 0 4px 15px rgba(90, 169, 250, 0.3);
}

.bouton-valider:hover {
  background-color: #3a8eff;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(74, 158, 255, 0.3);
}

.bouton-valider:active {
  transform: translateY(0);
}


.message-erreur {
  margin-top: 1rem;
  padding: 0.8rem;
  background-color: rgba(255, 69, 58, 0.1);
  border-left: 4px solid #ff453a;
  border-radius: 8px;
  color: #ff453a;
  font-size: 0.9rem;
  box-shadow: 0 2px 8px rgba(255, 69, 58, 0.2);
}
</style>
