<template>
   <div :class="`minimal-section-inner ${selected ? 'selected' : ''} ${showAll ? 'show-all': ''}`">
    <div class="section-items" v-if="selected || showAll">
      <Item
        v-for="(item, index) in items"
        :id="`${index}_${makeId(item.title)}`"
        :key="`${index}_${makeId(item.title)}`"
        :url="item.url"
        :title="item.title"
        :description="item.description"
        :icon="item.icon"
        :target="item.target"
        :color="item.color"
        :backgroundColor="item.backgroundColor"
        :statusCheckUrl="item.statusCheckUrl"
        :statusCheckHeaders="item.statusCheckHeaders"
        :itemSize="itemSize"
        :hotkey="item.hotkey"
        :enableStatusCheck="shouldEnableStatusCheck(item.statusCheck)"
        :statusCheckInterval="getStatusCheckInterval()"
        @itemClicked="$emit('itemClicked')"
        @triggerModal="triggerModal"
      />
    </div>
    <IframeModal
      :ref="`iframeModal-${groupId}`"
      :name="`iframeModal-${groupId}`"
      @closed="$emit('itemClicked')"
      @modalChanged="modalChanged"
    />
  </div>
</template>

<script>
import Item from '@/components/LinkItems/Item.vue';
import IframeModal from '@/components/LinkItems/IframeModal.vue';

export default {
  name: 'ItemGroup',
  inject: ['config'],
  props: {
    groupId: String,
    title: String,
    icon: String,
    displayData: Object,
    items: Array,
    itemSize: String,
    modalOpen: Boolean,
    index: Number,
    selected: Boolean,
    showAll: Boolean,
  },
  components: {
    Item,
    IframeModal,
  },
  methods: {
    selectSection(index) {
      this.$emit('sectionSelected', index);
    },
    /* Returns a unique lowercase string, based on name, for section ID */
    makeId(str) {
      return str.replace(/\s+/g, '-').replace(/[^a-zA-Z ]/g, '').toLowerCase();
    },
    /* Opens the iframe modal */
    triggerModal(url) {
      this.$refs[`iframeModal-${this.groupId}`].show(url);
    },
    modalChanged(changedTo) {
      this.$emit('change-modal-visibility', changedTo);
    },
    shouldEnableStatusCheck(itemPreference) {
      const globalPreference = this.config.appConfig.statusCheck || false;
      return itemPreference !== undefined ? itemPreference : globalPreference;
    },
    getStatusCheckInterval() {
      let interval = this.config.appConfig.statusCheckInterval;
      if (!interval) return 0;
      if (interval > 60) interval = 60;
      if (interval < 1) interval = 0;
      return interval;
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/media-queries.scss';
@import '@/styles/style-helpers.scss';

.minimal-section-inner {
  height: 100%;
  display: flex;
  flex-wrap: wrap;
  flex-direction: column;
  background: var(--minimal-view-group-background);
  border-radius: 0 0 var(--curve-factor) var(--curve-factor);
  .section-items {
    display: grid;
    @include phone { grid-template-columns: repeat(1, 1fr); }
    @include tablet { grid-template-columns: repeat(2, 1fr); }
    @include laptop { grid-template-columns: repeat(3, 1fr); }
    @include monitor { grid-template-columns: repeat(4, 1fr); }
    @include big-screen { grid-template-columns: repeat(5, 1fr); }
    @include big-screen-up { grid-template-columns: repeat(6, 1fr); }
  }
  &.selected {
    border: 1px solid var(--minimal-view-group-color);
    grid-column-start: span var(--col-count, 3);
  }
  &.show-all {
    border: none;
  }
}

</style>
