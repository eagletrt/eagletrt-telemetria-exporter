<template>
  <div class="items-selector container">
    <items-selector-title />
    <items-selector-loading v-if="loading" :message="message" />
    <items-selector-error v-if="error" :message="message" />
    <items-selector-content
      v-if="editing"
      :databases="databases"
      :currentDatabase="currentDatabase"
      :collections="currentCollections"
      :selectedItems="selectedItems"
    />
    <items-selector-buttons :type="buttonType" :disabled="!editing" />
  </div>
</template>

<script>
import { Status } from '../../store';

import ItemsSelectorTitle from './title/ItemsSelectorTitle.vue';
import ItemsSelectorContent from './content/ItemsSelectorContent.vue';
import ItemsSelectorLoading from './loading/ItemsSelectorLoading.vue';
import ItemsSelectorError from './error/ItemsSelectorError.vue';
import ItemsSelectorButtons from './buttons/ItemsSelectorButtons.vue';

export default {
    name: 'ItemsSelector',
    components: {
        ItemsSelectorButtons,
        ItemsSelectorContent,
        ItemsSelectorLoading,
        ItemsSelectorError,
        ItemsSelectorTitle
    },
    computed: {
        databaseSchema: function () {
            return this.$store.state.databaseSchema;
        },
        currentDatabase: function () {
            return this.$store.state.currentDatabase;
        },
        databases: function () {
            return this.$store.getters.databases;
        },
        currentCollections: function () {
            return this.$store.getters.currentCollections;
        },
        selectedItems: function () {
            return this.$store.state.selectedItems;
        },
        status: function () {
            return this.$store.state.status;
        },
        loading: function () {
            const state = this.$store.state.status;
            return state === Status.FETCHING || state === Status.EXPORTING;
        },
        editing: function () {
            const state = this.$store.state.status;
            return state === Status.EDITING;
        },
        error: function () {
            const state = this.$store.state.status;
            return state === Status.FETCHING_ERROR || state === Status.EXPORTING_ERROR;
        },
        message: function () {
            return this.$store.state.message;
        },
        buttonType: function () {
            switch (this.$store.state.status) {
            case Status.NONE:
            case Status.FETCHING:
            case Status.EXPORTING:
            case Status.EDITING:
                return 'EDITING';
            case Status.FETCHING_ERROR:
                return 'REFRESH';
            case Status.EXPORTING_ERROR:
                return 'EXPORTING_ERROR';
            default:
                return 'EDITING';
            }
        }
    },
    created () {
        this.$store.dispatch('fetchDatabaseSchema');
    }
};
</script>

<style lang="scss" scoped>
.items-selector {
  background: #000000;
  padding: 24px;
  max-height: 95%;
  overflow-y: auto;
}
</style>