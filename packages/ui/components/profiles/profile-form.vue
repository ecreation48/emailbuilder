<script>
import { validationMixin } from 'vuelidate';
import SENDINBLUEComponent from '../../components/profiles/esp/SENDINBLUEComponent';
import ACTITOComponent from '../../components/profiles/esp/ACTITOComponent';
import DSCComponent from '../../components/profiles/esp/DSCComponent';
import { groupsItem } from '~/helpers/api-routes';
import { ESP_TYPES } from '~/helpers/constants/esp-type';

export default {
  name: 'ProfileForm',
  components: {
    SENDINBLUEComponent,
    ACTITOComponent,
    DSCComponent,
  },
  mixins: [validationMixin],
  props: {
    title: { type: String, default: '' },
    profile: { type: Object, default: () => ({}) },
  },
  data: () => {
    return {
      group: {},
      authorizedEsps: [
        {
          text: 'SendinBlue',
          value: ESP_TYPES.SENDINBLUE,
        },
        {
          text: 'Actito',
          value: ESP_TYPES.ACTITO,
        },
        {
          text: 'DSC',
          value: ESP_TYPES.DSC,
        },
      ],
      selectedEsp: ESP_TYPES.SENDINBLUE,
      loading: false,
    };
  },
  computed: {
    selectedEspName() {
      return this.selectedEsp + 'Component';
    },
  },
  async mounted() {
    if (this.profile.type) {
      this.selectedEsp = this.profile.type;
    }
    await this.fetchGroup();
  },
  methods: {
    handleEspChange(value) {
      this.selectedEsp = value;
    },
    async fetchGroup() {
      const { $axios, $route } = this;
      const { params } = $route;
      try {
        this.group = await $axios.$get(groupsItem(params));
      } catch (error) {
        console.log(error);
      }
    },
    handleSubmit(newProfile) {
      this.$emit('submit', newProfile);
    },
  },
};
</script>

<template>
  <v-card flat tile tag="form">
    <v-card-title v-if="title">
      {{ title }}
    </v-card-title>
    <v-card-text>
      <v-row v-if="!group.downloadMailingWithFtpImages">
        <v-col cols="12">
          <v-alert
            dense
            border="left"
            type="warning"
            class="d-flex flex-row p-3"
          >
            {{ $t('profiles.warningNoFTP') }}
          </v-alert>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <v-select
            v-model="selectedEsp"
            :items="authorizedEsps"
            label="Esp"
            solo
            outlined
            flat
            :disabled="!group.downloadMailingWithFtpImages"
            @change="handleEspChange($event)"
          />
          <client-only>
            <component
              :is="selectedEspName"
              :profile-data="profile"
              :disabled="!group.downloadMailingWithFtpImages"
              :loading="loading"
              @submit="handleSubmit"
            />
          </client-only>
        </v-col>
      </v-row>
    </v-card-text>
  </v-card>
</template>
