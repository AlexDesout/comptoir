<template>
  <main>
    <h1>Les produits</h1>

    <table>
      <caption>Liste des produits {{ data.page + 1 }} / {{ data.totalPages }}</caption>
      <thead>
        <tr>
          <th>Nom</th>
          <th>Prix</th>
          <th>Stock</th>
          <th>Commandés</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="data.listeProduits.length === 0">
          <td colspan="5">Veuillez patienter, chargement des produits...</td>
        </tr>
        <tr v-for="produit in data.listeProduits" :key="produit.reference">
          <td>{{ produit.nom }}</td>
          <td>{{ produit.prixUnitaire }}</td>
          <td>{{ produit.unitesEnStock }}</td>
          <td>{{ produit.unitesCommandees }}</td>
          <td>
            <button @click="deleteEntity(produit._links.self.href)">Supprimer</button>
          </td>
        </tr>
        <tr>
          <td colspan="5">
            <button @click="firstPage">🔙</button>
            <button @click="previousPage">⬅️</button>
            <button @click="nextPage">➡️</button>
            <button @click="lastPage">🔜</button>
          </td>
        </tr>
      </tbody>
    </table>
  </main>
</template>

<script setup>
import { reactive, onMounted } from "vue";
import { doAjaxRequest } from "@/api";

const data = reactive({
  listeProduits: [],
  page: 0,
  totalPages: 0,
  links: {
    first: null,
    previous: null,
    next: null,
    last: null
  }
});

function showError(error) {
  console.error("Erreur : status", error.status, error.body);
  alert(error.message);
}

function chargeProduits(url = null) {
  const defaultUrl = `/api/produits?page=${data.page}&size=5&sort=code,desc`;

  doAjaxRequest(url || defaultUrl)
    .then(json => {
      data.page = json.page.number;
      data.totalPages = json.page.totalPages;
      data.listeProduits = json._embedded.produits || [];
      data.links = {
        first: json._links.first?.href || null,
        previous: json._links.prev?.href || null,
        next: json._links.next?.href || null,
        last: json._links.last?.href || null
      };
    })
    .catch(showError);
}

const firstPage = () => {
  if (data.links.first) chargeProduits(data.links.first);
};

const lastPage = () => {
  if (data.links.last) chargeProduits(data.links.last);
};

const nextPage = () => {
  if (data.links.next) chargeProduits(data.links.next);
};

const previousPage = () => {
  if (data.links.previous) chargeProduits(data.links.previous);
};

onMounted(() => chargeProduits());
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
  color: white;
}
caption {
  font-weight: bold;
  margin-bottom: 10px;
}
button {
  margin: 2px;
  padding: 5px 10px;
  cursor: pointer;
}
</style>
