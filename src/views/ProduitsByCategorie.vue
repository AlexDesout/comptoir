<template>
    <main>
        <div>
            <h1>Les catégories de produits</h1>

            <p v-if="data.listeCategories.length === 0">
                Veuillez patienter, chargement des catégories...
            </p>

            <select v-model="data.selectedCategorie">
                <option disabled value="">Sélectionner une catégorie</option>
                <option v-for="cat in data.listeCategories" :key="cat.code" :value="cat._links.produits.href">
                    {{ cat.libelle }}
                </option>
            </select>


        </div>
        <div>
            <table>
                <caption>Liste des produits</caption>
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
            </table>
        </div>
    </main>
</template>

<script setup>
import { reactive, onMounted, watch } from "vue";
import { doAjaxRequest } from "@/api";

const produitVide = {
    nom: "",
    prixUnitaire: "",
    unitesEnStock: "",
    unitesCommandees: "",
};

let data = reactive({
    listeCategories: [],
    selectedCategorie: "",
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
    let url = data.selectedCategorie
        ? `${data.selectedCategorie}?sort=code,desc`
        : `/api/produits?sort=code,desc`;

    doAjaxRequest(url)
        .then((json) => {
            console.log(json)
            data.listeProduits = json._embedded.produits;
            if(!data.totalPages) {
                data.totalPages = json.page.totalPages;
            }
        })
        .catch(showError);
}

function chargeCategories() {
    doAjaxRequest(`/api/categories?sort=code,desc`)
        .then((json) => {
            data.listeCategories = json._embedded.categories;
        })
        .catch(showError);
}

onMounted(() => {
    chargeCategories();
    chargeProduits();
});

watch(() => data.selectedCategorie, () => {
    data.page = 0;
    chargeProduits();
});


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