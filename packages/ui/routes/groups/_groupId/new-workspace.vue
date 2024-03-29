<script>
import mixinPageTitle from '~/helpers/mixins/mixin-page-title.js';
import { ERROR_CODES } from '~/helpers/constants/error-codes.js';
import { PAGE, SHOW_SNACKBAR } from '~/store/page.js';
import { mapMutations } from 'vuex';

import * as acls from '~/helpers/pages-acls.js';
import * as apiRoutes from '~/helpers/api-routes.js';
import WorkspaceForm from '~/components/workspaces/workspace-form';
import BsGroupMenu from '~/components/group/menu.vue';

export default {
  name: 'PageNewWorkspace',
  components: { WorkspaceForm, BsGroupMenu },
  mixins: [mixinPageTitle],
  meta: {
    acl: acls.ACL_GROUP_ADMIN,
  },
  async asyncData(nuxtContext) {
    const { $axios, params } = nuxtContext;

    try {
      const { items: users } = await $axios.$get(
        `/groups/${params?.groupId}/users`
      );
      const groupResponse = await $axios.$get(apiRoutes.groupsItem(params));
      return {
        groupUsers: users,
        isLoading: false,
        group: groupResponse,
      };
    } catch (error) {
      return { isLoading: false, isError: true };
    }
  },
  data() {
    return {
      group: {},
      isLoading: true,
      isError: false,
      groupUsers: [],
    };
  },
  head() {
    return { title: this.title };
  },

  computed: {
    title() {
      return `${this.$tc('global.settings', 1)} : ${this.$tc(
        'global.group',
        1
      )} ${this.group.name} - ${this.$t('global.newWorkspace')}`;
    },
    groupId() {
      return this.$route.params.groupId;
    },
  },
  methods: {
    ...mapMutations(PAGE, { showSnackbar: SHOW_SNACKBAR }),
    async createWorkspace(values) {
      const { $axios } = this;
      const groupId = this.$route.params?.groupId;
      try {
        this.isLoading = true;
        await $axios.$post('/workspaces', {
          groupId,
          ...values,
        });
        this.showSnackbar({
          text: this.$t('snackbars.created'),
          color: 'success',
        });

        this.$router.push(`/groups/${groupId}`);
      } catch (error) {
        const errorKey = `global.errors.${
          ERROR_CODES[error.response?.data] || 'errorOccured'
        }`;
        this.showSnackbar({
          text: this.$t(errorKey),
          color: 'error',
        });
      } finally {
        this.isLoading = false;
      }
    },
  },
};
</script>

<template>
  <bs-layout-left-menu>
    <template #menu>
      <bs-group-menu />
    </template>
    <workspace-form
      :title="$t('global.newWorkspace')"
      :group-users="groupUsers"
      :is-loading="isLoading"
      @submit="createWorkspace"
    />
  </bs-layout-left-menu>
</template>
