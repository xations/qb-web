<template>
  <v-dialog :value="true" @input="closeDialog" :fullscreen="phoneLayout" width="40em">
    <v-card>
      <v-card-title
        class="headline grey lighten-4"
      >
        <v-icon class="mr-2">mdi-delete</v-icon>
        <span>Delete torrents</span>
      </v-card-title>
      <v-card-text class="pb-0">
         Are you sure to delete selected torrents from transfer list?
        <ol class="torrents pt-6">
          <li v-for="(row, i) in torrents" :key="i">
            {{ row.name }}
          </li>
        </ol>

        <v-checkbox
          v-model="deleteFiles"
          prepend-icon="mdi-file-cancel"
          label="Also delete files"
        />
        <v-checkbox
          v-if="sameNamedTorrents.length > 0"
          v-model="deleteSameNamed"
          prepend-icon="mdi-file-multiple"
          class="mt-0"
          :label="`Also delete ${sameNamedTorrents.length} same named torrents`"
        />
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn text @click="closeDialog">Cancel</v-btn>
        <v-btn
          @click="submit"
          color="warning"
          :disabled="submitting"
          :loading="submitting"
        >
          Delete
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import _ from 'lodash';
import Vue, { PropType } from 'vue';
import { mapGetters } from 'vuex';
import api from '@/Api';
import { findSameNamedTorrents } from '@/utils';
import { Torrent } from '../../types';

export default Vue.extend({
  props: {
    value: Array as PropType<Torrent[]>,
  },
  data() {
    return {
      deleteFiles: false,
      deleteSameNamed: false,
      submitting: false,
      torrents: [],
      sameNamedTorrents: [],
    };
  },
  created() {
    this.torrents = this.value;
    this.sameNamedTorrents = findSameNamedTorrents(this.allTorrents, this.torrents);
  },
  computed: {
    ...mapGetters(['allTorrents']),
    phoneLayout() {
      return this.$vuetify.breakpoint.xsOnly;
    },
  },
  methods: {
    closeDialog() {
      this.$emit('input', []);
    },
    async submit() {
      if (this.submitting) {
        return;
      }

      this.submitting = true;

      let torrentsToDelete;
      if (this.deleteSameNamed) {
        torrentsToDelete = this.torrents.concat(this.sameNamedTorrents);
      } else {
        torrentsToDelete = this.torrents;
      }
      const hashes = torrentsToDelete.map((t: any) => t.hash);
      await api.deleteTorrents(hashes, this.deleteFiles);

      this.closeDialog();
    },
  },
});
</script>

<style lang="scss" scoped>
.torrents {
  overflow: auto;
}
.v-dialog--fullscreen {
  .v-card__text {
    padding-bottom: 52px;
  }

  .v-card__actions {
    position: absolute;
    bottom: 0;
    right: 0;
  }
}
</style>
