<script>
import BsSnackBar from '~/components/snackbar.vue';

export default {
  name: 'BsLayoutAuthentication',
  components: { BsSnackBar },
};
</script>

<template>
  <v-app id="inspire">
    <v-main>
      <v-container class="fill-height secondary" fluid>
        <v-row align="center" justify="center">
          <v-col cols="12" sm="8" md="5">
            <nuxt />
          </v-col>
        </v-row>
      </v-container>
    </v-main>
    <bs-snack-bar />
  </v-app>
</template>
