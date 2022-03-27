<script setup>
// @ is an alias to /src

import { reactive, onMounted } from "vue";

let data = reactive({
  countries: [],
});

defineExpose({
  // On expose la méthode 'refresh' pour être utilisée par le parent
  refresh,
});

// On définit les événements générés par le composant
const emit = defineEmits(["countryEdited"]);

function editCountry(country) {
  emit("countryEdited", country); // On notifie le parent qu'il faut modifier le pays
}

function deleteCountry(country) {
  console.log(country);
  const options = {
    method: "DELETE",
  };
  fetch(country._links.self.href, options)
    .then((response) => {
      if (!response.ok) {
        // status != 2XX
        throw new Error(response.status);
      }
      refresh();
    })
    .catch((error) => alert(error));
}

function refresh() {
  fetch("api/countries")
    .then((response) => {
      if (!response.ok) {
        // status != 2XX
        throw new Error(response.status);
      }
      return response.json();
    })
    .then((json) => {
      data.countries = json._embedded.countries;
    })
    .catch((error) => alert(error));
}
function getCities(country) {
  console.log(country);
  const options = {
    method: "GET",
  };
  fetch(country._links.cities.href, options)
    .then((response) => {
      if (!response.ok) {
        throw new Error(response.status);
      }
      refresh();
    })
    .catch((error) => alert(error));
}

function ajoutervilles() {
  let citiesHref = document.getElementById("countrySelector").value;
  console.log(citiesHref);
  fetch(citiesHref)
    .then((response) => {
      if (!response.ok) {
        // status != 2XX
        throw new Error(response.status);
      }
      return response.json();
    })
    .then((json) => {
      data.cities = json._embedded.cities;
    })
    .catch((error) => alert(error));
}
onMounted(refresh);
</script>
<template>
    <div class="container mt-3">
    <select id="countrySelector" :@onChange="ajoutervilles()">
      <option
        v-for="country in data.countries"
        :key="country.id"
        :value="country._links.cities.href"
      >
        {{ country.name }}
      </option>
    </select>
    <table class="table table-bordered table-sm table-hover">
      <thead>
        <tr>
          <th>Nom</th>
          <th>population</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="city in data.cities" :key="city.id">
          <td>{{ city.name }}</td>
          <td>{{city.population}}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

