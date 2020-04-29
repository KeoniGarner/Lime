<template>
  <div class="app flex-row align-items-center">
    <div class="container text-center">
      <b-card>
        <h1 ref="title" id="title" class="text-left">Lime Search</h1>
        <b-row class="justify-content-center">
          <b-col class="text-center">
            <b-form @submit="onSubmit" @reset="onReset">
              <b-form-group
                id="search"
                label=""
                label-for="search-field"
                description=""
                ><i class="fa fa-search fa-lg"></i>
                <b-form-input
                  id="search-field"
                  v-model="form.search"
                  type="text"
                  required
                  placeholder="Search..."
                >
                </b-form-input>
                <b-button id="submit" type="submit"></b-button>
              </b-form-group>
            </b-form>
          </b-col>
        </b-row>
      </b-card>
      <div v-if="loading" class="fingerprint-spinner w-100 align-items-center">
        <div class="spinner-ring"></div>
        <div class="spinner-ring"></div>
        <div class="spinner-ring"></div>
        <div class="spinner-ring"></div>
        <div class="spinner-ring"></div>
        <div class="spinner-ring"></div>
        <div class="spinner-ring"></div>
        <div class="spinner-ring"></div>
        <div class="spinner-ring"></div>
      </div>
      <b-card class="text-left" v-if="data.claims && data.claims.length">
        <h1>
          {{ search }}
          <span class="sour"
            >{{ evaluate(data.claims) }} SOUR
            <div id="lime">
              <img src="../assets/images/lime.png" alt="Lime" /></div
          ></span>
        </h1>

        <h3>True Claims</h3>
        <p v-if="!trueClaims.length">None</p>
        <p v-for="trueClaim in trueClaims">{{ trueClaim }}</p>
        <h3>False Claims</h3>
        <p v-if="!falseClaims.length">None</p>
        <p v-for="falseClaim in falseClaims">{{ falseClaim }}</p>
      </b-card>
    </div>
  </div>
</template>

<script>
const axios = require("axios");
export default {
  name: "home",
  components: {},
  data() {
    return {
      form: {
        search: ""
      },
      search: "",
      data: {
        claims: []
      },
      loading: false,
      trueClaims: [],
      falseClaims: []
    };
  },
  created() {},
  computed: {},
  methods: {
    onSubmit(event) {
      event.preventDefault();
      this.loading = true;
      axios
        .get(
          `https://factchecktools.googleapis.com/v1alpha1/claims:search?pageSize=1000&languageCode=en-US&query=${this.form.search}&key={apiKey}`
        )
        .then(response => {
          this.search = this.form.search;
          this.data = response.data;
        })
        .catch(error => {
          console.log(error);
        })
        .finally(() => {
          this.loading = false;
        });
    },
    onReset(event) {
      event.preventDefault();
    },
    evaluate(claims) {
      let claimantCount = {};
      let maxNumber = 0;
      let maxKey;
      this.trueClaims = [];
      this.falseClaims = [];
      for (const claim of claims) {
        if ("claimant" in claim && claim.claimant.includes(this.search)) {
          if (claim.claimant in claimantCount) {
            claimantCount[claim.claimant] += 1;
          } else {
            for (const claimReview of claim.claimReview) {
              if (claimReview.textualRating != "True") {
                this.falseClaims.push(claim.text);
                break;
              }
            }
            if (this.falseClaims[this.falseClaims.length - 1] != claim.text) {
              this.trueClaims.push(claim.text);
            }
            claimantCount[claim.claimant] = 1;
          }
          if (claimantCount[claim.claimant] > maxNumber) {
            maxNumber = claimantCount[claim.claimant];
            maxKey = claim.claimant;
          }
        }
      }
      this.search = maxKey;
      return (
        (
          (this.falseClaims.length /
            (this.trueClaims.length + this.falseClaims.length)) *
          100
        )
          .toFixed(2)
          .toString() + "%"
      );
    }
  }
};
</script>
