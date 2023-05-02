<template>
  <v-app>
    <v-app-bar color="#4a69bd" dark fixed app>
      <div
        style="background-color: #4a69bd"
        class="title text-h3 font-weight-bold white--text description"
      >
        <div class="mx-2">SciSimplify</div>
      </div>
      <v-spacer></v-spacer>
      <v-btn
        color="#1e3799"
        class="white--text my-2 mr-2"
        large
        style="box-shadow: none !important"
        @click="$refs.HomePage.togglePage(true)"
      >
        Home
      </v-btn>

      <v-btn href="tmp" target="_blank" text class="black--text">
        <span class="mr-2" style="color: #1e3799">About</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>
    <v-main
      style="
        background: linear-gradient(-45deg, #4a69bd, #60a3bc, #4a69bd, #60a3bc);
        background-size: 1000% 100%;
      "
    >
      <HomePage ref="HomePage" />
    </v-main>
    <v-footer
      fixed-bottom
      style="display: flex; z-index: 0; background-color: rgb(74, 105, 189)"
    >
    </v-footer>
  </v-app>
</template>


<script>
import HomePage from "./components/HomePage";

export default {
  name: "App",

  components: {
    HomePage,
  },

  data: () => ({}),
};
</script>

<style>
.v-application {
  font-family: "Gothic", sans-serif !important;
}
</style>