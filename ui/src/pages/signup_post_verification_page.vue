<template>
  <registration-layout>
    <h3 class="signup-title-description" v-if="surveyProgress === 1">
      Password Information
    </h3>
    <h3 class="signup-title-description" v-if="surveyProgress === 2">
      Address Information
    </h3>
    <h3 class="signup-title-description" v-if="surveyProgress === 3">
      User Foapa Information (Not Required). Hover Over Question Mark Icon For
      Help
    </h3>

    <section class="signup-form">
      <!-- SECTION 2 -->
      <section v-show="surveyProgress === 1" class="signup-form">
        <PasswordSection :user-signup-data="userSignupData" @continue="surveyProgress++" />
      </section>

      <!-- SECTION 3 -->
      <section v-show="surveyProgress === 2" class="signup-form">
        <AddressSection :user-signup-data="userSignupData" @continue="surveyProgress++" @go-back="surveyProgress--" />
      </section>

      <!-- SECTION 4 -->
      <section v-show="surveyProgress === 3" class="signup-form">
        <FoapaSection :user-signup-data="userSignupData" @finish="registerUser" @go-back="surveyProgress--" />
      </section>
      <br />
      <h5 v-if="creatingAccountFeedback" class="creating-account-feedback">
        Creating Account...
      </h5>
    </section>
  </registration-layout>
</template>

<script lang="ts" setup>
import PasswordSection from "../components/signup-flow/PasswordSection.vue";
import AddressSection from "../components/signup-flow/AddressSection.vue";
import FoapaSection from "../components/signup-flow/FoapaSection.vue";
import axios from "axios";
import RegistrationLayout from "../layouts/registration_layout.vue";
import { onMounted, reactive, ref } from "vue";
import { useRouter, useRoute } from "vue-router";
import { UserData } from "../types/types";
let surveyProgress = ref<number>(1);

const router = useRouter();
const route = useRoute();
let creatingAccountFeedback = ref<boolean>(false);
let userSignupData = reactive<UserData>({
  first_name: "",
  last_name: "",
  work_email: "",
  employment_number: null,
  department: "",
  mailing_address: "mail",
  phone_number: 12345678,
  password: "b",
  postal_code: "Postcode",
  city: "Det",
  state: "MI",
  country: "USA",
  foapa_details: [],
});

function registerUser() {
  creatingAccountFeedback.value = true;

  axios
    .post(
      `${import.meta.env.VITE_API_URL}/api/register`,
      userSignupData, {
      headers: {
        'Authorization': `Bearer ${route.params.token}`
      }
    }
    )
    .then((res) => {
      localStorage.setItem("token", res.data);
      axios.defaults.headers.common["Authorization"] = `BEARER ${res.data}`
      router.push("/dashboard");
    })
    .catch((err) => {
      if (err?.response.status === 409) {
        alert("A faculty with your credentials already exists")
      }

      if (err?.response.status === 500) {
        alert("An error has occured, please try again")
      }

      if (err?.response.status === 400) {
        alert("Invalid request, please try again")
      }

      if (err?.response.status === 403) {
        alert("Invalid token, please restart the registration process")
      }
    })
    .finally(() => {
      creatingAccountFeedback.value = false;
    });
}


onMounted(() => {
  if (localStorage.getItem("token")?.length ?? 0 > 0) {
    router.push("/dashboard");
  }

  const axiosConfig = {
    headers: {
      'Authorization': `Bearer ${route.params.token}`
    }
  };

  if (route.params.token.length === 0) {
    alert("Missing registration token, please restart the signup process")
    router.push("/signup")
    return
  }

  axios
    .get(
      `${import.meta.env.VITE_API_URL}/api/verify-user-registration-token`, axiosConfig)
    .then((res) => {
      userSignupData = Object.assign(userSignupData, res.data);
    })
    .catch((err) => {
      console.log(err);
      alert(err?.response?.data || "An error has occured, please restart the registration process");
      router.push("/signup");
    });

});
</script>

<style scoped>
@import url("../assets/styles/signup-page.css");
</style>
