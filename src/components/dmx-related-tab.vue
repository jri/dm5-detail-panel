<template>
  <div class="dmx-related-tab">
    <dmx-topic-list v-if="!loading" :topics="topics" :sort-mode="sortMode" :marker-topic-ids="markerTopicIds"
      @topic-click="topicClick" @icon-click="iconClick" @sort-change="sortChange">
    </dmx-topic-list>
    <div v-else v-loading="true" class="dmx-spinner"></div>
  </div>
</template>

<script>
export default {

  created () {
    // console.log('dmx-related-tab created', this.markerTopicIds)
    this.fetchTopics()
  },

  destroyed () {
    // console.log('dmx-related-tab destroyed')
  },

  mixins: [
    require('./mixins/object').default,
    require('./mixins/tab').default
  ],

  props: {
    sortMode: String,                       // topic list sort mode: 'topic', 'type', 'assoc'
    markerTopicIds: Array                   // IDs of topics to render as "marked"
  },

  data () {
    return {
      topics: [],
      loading: false
    }
  },

  watch: {

    object () {
      this.topics = []        // enforce refetch
      this.fetchTopics()
    },

    tab () {
      this.fetchTopics()
    }
  },

  methods: {

    fetchTopics () {
      // console.log('fetchTopics', this.object.id, this.tab === 'related', !this.topics.length)
      // fetch only if "Related" tab is selected AND not fetched already
      if (this.tab === 'related' && !this.topics.length) {
        this.loading = true
        this.object.getRelatedTopicsWithoutChilds().then(topics => {
          this.topics = topics
          this.loading = false
        })
      }
    },

    topicClick (relTopic) {
      this.$emit('related-topic-click', relTopic)
    },

    iconClick (relTopic) {
      this.$emit('related-icon-click', relTopic)
    },

    sortChange (sortMode) {
      this.$emit('sort-change', sortMode)
    }
  },

  components: {
    'dmx-topic-list': require('dmx-topic-list').default
  }
}
</script>

<style>
.dmx-related-tab {
  overflow: auto;
  padding: var(--detail-panel-padding-all);
}

.dmx-related-tab .dmx-spinner {
  height: 42px;               /* see --loading-spinner-size in element-ui/packages/theme-chalk/src/common/var.scss */
}

.dmx-related-tab .dmx-spinner .el-loading-mask{
  background-color: unset;    /* no white mask */
}
</style>
