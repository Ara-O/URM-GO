<template>
  <section class="foapa-number-section">
    <div class="logo-container">
      <img src="../../assets/detroit-mercy-logo.png" class="udmercy-logo" @click="$router.push('/dashboard')"
        alt="Detroit Mercy logo" />
    </div>
    <h3>Foapa Numbers</h3>
    <br />
    <div class="foapa-number-wrapper">
      <div class="foapa-number" v-for="foapa in userFoapaNumbers"
        style="display: flex; flex-direction: column; align-items: start">
        <span class="foapa-number-title">
          <img src="../../assets/trash-icon.png" alt="Trash" @click="deleteFoapa(foapa)" />
          <h3 style="font-weight: 500">
            {{ foapa.foapa_name }}
          </h3>
        </span>
        <div class="foapa-details-container">
          <h3 style="margin-top: 10px">{{ formatUserFoapa(foapa) }}</h3>
        </div>
      </div>
      <div class="filter" @click="goToFoapaPage">
        <h3>Manage Foapa Details</h3>
      </div>
    </div>
  </section>
</template>

<script lang="ts" setup>
import { onMounted, ref, computed } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import { FoapaData } from "../../types/types";

let userFoapaNumbers = ref<FoapaData[]>([]);

const router = useRouter();

function formatUserFoapa(foapa: FoapaData) {
  // console.log(foapa);
  return `${foapa.fund}-${foapa.organization || "XXXX"}-${foapa.account}-${foapa.program || "XXXX"
    }-${foapa.activity || "XXXX"}`;
}

function retrieveUserFoapaDetails() {
  axios
    .get(
      `${import.meta.env.VITE_API_URL}/api/retrieve-foapa-details`
    )
    .then((res) => {
      console.log(res.data)
      userFoapaNumbers.value = res.data;
    })
    .catch((err) => {
      console.log(err);
    });
}




function deleteFoapa(foapa_detail: FoapaData) {
  let index = userFoapaNumbers.value.findIndex(
    (foapa) => foapa.foapa_name === foapa_detail.foapa_name && foapa.fund === foapa_detail.fund
  );

  if (index > -1) {
    userFoapaNumbers.value.splice(index, 1);
  }

  axios
    .post(
      `${import.meta.env.VITE_API_URL}/api/update-foapa-details`,
      userFoapaNumbers.value
    )
    .then(() => {
      retrieveUserFoapaDetails();
    })
    .catch((err) => {
      console.log(err);
      alert(err.response?.data || "There was an error deleting this FOAPA number");
    });
}

function goToFoapaPage() {
  router.push("/add-foapa");
}

onMounted(() => {
  retrieveUserFoapaDetails();
});
</script>

<style scoped>
.foapa-details-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-around;
}
</style>
