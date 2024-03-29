<script>
import BsModalConfirm from '~/components/modal-confirm';
import { getTreeviewWorkspacesWithoutSubfolders } from '~/utils/workspaces';
import { SPACE_TYPE } from '~/helpers/constants/space-type';
import { mapState } from 'vuex';
import { FOLDER } from '~/store/folder';

export default {
  name: 'FolderMoveModal',
  components: {
    BsModalConfirm,
  },
  props: {
    confirmationInputLabel: { type: String, default: '' },
  },
  data() {
    return {
      folder: null,
      workspaceIsError: false,
      selectedLocation: {},
    };
  },
  computed: {
    ...mapState(FOLDER, ['workspacesHasRight', 'areLoadingWorkspaces']),
    treeviewLocationItems() {
      return getTreeviewWorkspacesWithoutSubfolders(this.workspacesHasRight);
    },
    isValidToBeMoved() {
      return (
        !!this.selectedLocation?.id &&
        this.selectedLocation?.id !== this.folder?.id
      );
    },
    folderName() {
      return this.folder?.name;
    },
  },
  methods: {
    submit() {
      if (this.isValidToBeMoved) {
        const location = this.selectedLocation;
        this.close();

        let destinationParam;
        if (location?.type === SPACE_TYPE.FOLDER) {
          destinationParam = {
            destinationFolderId: location?.id,
          };
        } else {
          destinationParam = {
            workspaceId: location?.id,
          };
        }
        this.$emit('confirm', {
          destinationParam,
          folderId: this.folder?.id,
        });
      }
    },
    handleSelectItemFromTreeView(selectedItems) {
      if (selectedItems[0]) {
        this.selectedLocation = selectedItems[0];
      }
    },
    open(selectedFolder) {
      this.folder = selectedFolder;
      this.$refs.moveFolderDialog.open();
    },
    close() {
      this.$refs.moveFolderDialog.close();
      this.selectedLocation = {};
    },
  },
};
</script>
<template>
  <bs-modal-confirm
    ref="moveFolderDialog"
    :title="`${this.$t(
      'global.moveFolderAction'
    )} <strong>${folderName}</strong>`"
    :is-form="true"
    class="modal-confirm-move-mail"
    @click-outside="close"
  >
    <slot />
    <v-skeleton-loader
      type="list-item, list-item, list-item"
      :loading="areLoadingWorkspaces"
    >
      <v-treeview
        ref="tree"
        item-key="id"
        activatable
        :items="treeviewLocationItems"
        hoverable
        open-all
        :dense="true"
        :return-object="true"
        class="pb-8"
        @update:active="handleSelectItemFromTreeView"
      >
        <template #prepend="{ item, open }">
          <v-icon v-if="!item.icon" color="accent">
            {{ open ? 'mdi-folder-open' : 'mdi-folder' }}
          </v-icon>
          <v-icon v-else color="accent">
            {{ item.icon }}
          </v-icon>
        </template>
        <template #label="{ item, active }">
          <div @click="active ? $event.stopPropagation() : null">
            {{ item.name }}
          </div>
        </template>
      </v-treeview>
    </v-skeleton-loader>
    <v-divider />
    <v-card-actions>
      <v-spacer />
      <v-btn color="primary" text @click="close">
        {{ $t('global.cancel') }}
      </v-btn>
      <v-btn
        :disabled="!isValidToBeMoved"
        elevation="0"
        color="accent"
        @click="submit"
      >
        {{ $t('global.move') }}
      </v-btn>
    </v-card-actions>
  </bs-modal-confirm>
</template>

<style scoped>
.v-treeview-node--active,
.v-treeview--hoverable {
  cursor: pointer;
}

.v-treeview {
  overflow-y: auto;
  max-height: 400px;
}
.v-treeview-node__label > div {
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
</style>
