<template>
  <v-app>
    <v-main>
      <PartManager />
    </v-main>
  </v-app>
</template>

<script>
import PartManager from "./components/PartManager.vue";

export default {
  name: "App",

  components: {
    PartManager,
  },

  data: () => ({
    //
  }),
};
</script>
