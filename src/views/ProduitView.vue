<template>
    <main>
        <div>
            <h1>Les produits</h1>
        </div>
        <div>
            <table>
                <caption>Liste des produits {{ data.page + 1 }} / {{ data.totalPages }}</caption>
                <tr>
                    <th>Nom</th>
                    <th>Prix</th>
                    <th>Stock</th>
                    <th>Commandés</th>
                    <th>Action</th>
                </tr>
                <tr v-if="data.listeProduits.length === 0">
                    <td colspan="4">Veuillez patienter, chargement des produits...</td>
                </tr>
                <tr v-for="produit in data.listeProduits" :key="produit.reference">
                    <td>{{ produit.nom }}</td>
                    <td>{{ produit.prixUnitaire }}</td>
                    <td>{{ produit.unitesEnStock }}</td>
                    <td>{{ produit.unitesCommandees }}</td>
                    <td>
                        <button @click="deleteEntity(produit._links.self.href)">
                            Supprimer
                        </button>
                    </td>
                </tr>
                <tr>
                    <td><button @click="firstPage">🔙</button></td>
                    <td><button @click="previousPage">⬅️</button></td>
                    <td><button @click="nextPage">➡️</button></td>
                    <td><button @click="lastPage">🔜</button></td>
                </tr>
            </table>
        </div>
    </main>
</template>

<script setup>
import { reactive, onMounted } from "vue";
import { doAjaxRequest } from "@/api";

// Pour réinitialiser le formulaire
const produitVide = {
    nom: "",
    prixUnitaire: "",
    unitesEnStock: "",
    unitesCommandees: "",
};

let data = reactive({
    // Les données saisies dans le formulaire
    formulaireProduit: { ...produitVide },
    // La liste des catégories affichée sous forme de table
    listeProduits: [],
    page: 0,
    totalPages: 0
});

function showError(error) {
    console.log("Erreur : status %d", error.status)
    console.log(error.body);
    alert(error.message);
}

function chargeProduits() {
    // Appel à l'API pour avoir la liste des produits
    // Triés par code, descendant
    doAjaxRequest(`/api/produits?page=${data.page}&size=5&sort=code,desc`)
        .then((json) => {
            data.listeProduits = json._embedded.produits;
            data.totalPages = json.page.totalPages;
        })
        .catch(showError);
}

// function ajouteCategorie() {
//     // Ajouter une catégorie avec les données du formulaire
//     const options = {
//         method: "POST", // Verbe HTTP POST pour ajouter un enregistrement
//         body: JSON.stringify(data.formulaireCategorie),
//         headers: {
//             "Content-Type": "application/json",
//             "Accept": "application/json"
//         },
//     };
//     doAjaxRequest("/api/categories", options)
//         .then(() => {
//             // Réinitialiser le formulaire
//             data.formulaireCategorie = { ...categorieVide };
//             // Recharger la liste des catégories
//             chargeCategories();
//         })
//         .catch(showError);
// }
// /**
//  * Supprime une entité
//  * @param entityRef l'URI de l'entité à supprimer
//  */
// function deleteEntity(entityRef) {
//     doAjaxRequest(entityRef, { method: "DELETE", headers: { "Accept": "application/json" } })
//         .then(chargeCategories)
//         .catch(showError);
// }
function nextPage() {
    if (data.page < data.totalPages - 1) {
        data.page++;
        chargeProduits();
    } else {
        alert("Déjà à la dernière page");
    }
}

function previousPage() {
    if (data.page > 0) {
        data.page--;
        chargeProduits();
    } else {
        alert("Déjà à la première page");
    }
}

function firstPage() {
    data.page = 0;
    chargeProduits();
}

function lastPage() {
    data.page = data.totalPages -1;
    chargeProduits();
}


// A l'affichage du composant, on affiche la liste
onMounted(chargeProduits);

</script>


<style scoped>
td,
th {
    border: 1px solid #ddd;
    padding: 8px;
}

th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: left;
    background-color: #232623;
    color: rgb(255, 255, 255);
}
</style>
