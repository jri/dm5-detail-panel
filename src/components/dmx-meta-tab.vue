<template>
  <div class="dmx-meta-tab">
    <div class="flex">
      <div>
        <div class="field-label">ID</div>
        <el-link type="primary" :underline="false" title="Show JSON" @click="idClick">{{object.id}}</el-link>
      </div>
      <div>
        <div class="field-label">URI</div>
        <div>{{object.uri || 'n/a'}}</div>
      </div>
    </div>
    <div class="flex">
      <div>
        <div class="field-label">Created</div>
        <div>{{created || 'n/a'}}</div>
      </div>
      <div>
        <div class="field-label">User</div>
        <div>{{creator || 'n/a'}}</div>
      </div>
    </div>
    <div class="flex">
      <div>
        <div class="field-label">Modified</div>
        <div>{{modified || 'n/a'}}</div>
      </div>
      <div>
        <div class="field-label">User</div>
        <div>{{modifier || 'n/a'}}</div>
      </div>
    </div>
    <!-- Workspace -->
    <div class="flex">
      <div>
        <div class="field-label">Workspace</div>
        <dmx-inline-edit :disabled="!writable" @save="assignToWorkspace">
          <template #info>
            <div>{{workspace && workspace.value || 'n/a'}}</div>
          </template>
          <template #form>
            <dmx-workspace-select :workspace="workspace" ref="workspaceSelect"></dmx-workspace-select>
          </template>
        </dmx-inline-edit>
      </div>
      <div>
        <div class="field-label">Owner</div>
        <div>{{owner || 'n/a'}}</div>
      </div>
    </div>
    <!-- Type -->
    <dmx-topic-list :topics="types" no-sort-menu :marker-topic-ids="markerTopicIds"
      @topic-click="topicClick" @icon-click="iconClick">
    </dmx-topic-list>
    <!-- Topicmaps -->
    <dmx-topic-list :topics="topicmapTopics" no-sort-menu :marker-topic-ids="markerTopicIds"
      @topic-click="topicClick" @icon-click="iconClick">
    </dmx-topic-list>
  </div>
</template>

<script>
import dmx from 'dmx-api'

export default {

  created () {
    // console.log('dmx-meta-tab created')
    this.fetchMetaData()
  },

  destroyed () {
    // console.log('dmx-meta-tab destroyed')
  },

  mixins: [
    require('./mixins/object').default,
    require('./mixins/writable').default,
    require('./mixins/tab').default
  ],

  props: {
    markerTopicIds: Array     // IDs of topics to render as "marked"
  },

  data () {
    return {
      created:   undefined,
      modified:  undefined,
      creator:   undefined,
      modifier:  undefined,
      workspace: undefined,
      owner:     undefined,
      types: [],
      topicmapTopics: []
    }
  },

  watch: {

    object () {
      // console.log('object watcher', this.object.id)
      this.fetchMetaData()
    },

    tab () {
      // console.log('tab watcher', this.tab)
      this.fetchMetaData()
    }
  },

  methods: {

    fetchMetaData () {
      // Optimization: don't fetch if "Meta" tab is not selected
      if (this.tab !== 'meta') {
        return
      }
      // TODO: suppress unnecessary refetching when browsing between tabs and revisit the "Meta" tab
      //
      this.object.getCreationTime()
        .then(created => {
          this.created = created && new Date(created).toLocaleString()
        })
      this.object.getModificationTime()
        .then(modified => {
          this.modified = modified && new Date(modified).toLocaleString()
        })
      this.object.getCreator()
        .then(creator => {
          this.creator = creator
        })
      this.object.getModifier()
        .then(modifier => {
          this.modifier = modifier
        });
      // if the selected object is a workspace the workspace is the object itself
      (this.object.typeUri === 'dmx.workspaces.workspace' ? Promise.resolve(this.object) : this.object.getWorkspace())
        .then(workspace => {
          return this.workspace = workspace
        })
        .then(workspace => {
          if (workspace) {
            dmx.rpc.getWorkspaceOwner(workspace.id).then(owner => {
              this.owner = owner
            })
          } else {
            this.owner = undefined
          }
        })
      // manual traversal gives us a RelatedTopic as needed for revelation
      this.object.getRelatedTopics({
        assocTypeUri:      'dmx.core.instantiation',
        myRoleTypeUri:     'dmx.core.instance',
        othersRoleTypeUri: 'dmx.core.type'
      }).then(types => {
        this.types = types
      })
      this.object.getTopicmapTopics().then(topicmapTopics => {
        this.topicmapTopics = topicmapTopics
      })
    },

    assignToWorkspace () {
      this.workspace = this.$refs.workspaceSelect.workspace_    // update client state
      // console.log('assignToWorkspace', this.workspace.id)
      this.object.assignToWorkspace(this.workspace.id)          // update server
      // TODO: process directives
    },

    topicClick (relTopic) {
      this.$emit('related-topic-click', relTopic)
    },

    iconClick (relTopic) {
      this.$emit('related-icon-click', relTopic)
    },

    idClick () {
      this.$emit('object-id-click', this.object)
    }
  },

  components: {
    'dmx-topic-list':       require('dmx-topic-list').default,
    'dmx-inline-edit':      require('dmx-inline-edit').default,
    'dmx-workspace-select': require('./dmx-workspace-select').default
  }
}
</script>

<style>
.dmx-meta-tab {
  overflow: auto;
  padding: var(--detail-panel-padding-all);
}

.dmx-meta-tab .flex {
  display: flex;
}

.dmx-meta-tab .flex > div + div {
  margin-left: 2.5em;
}

.dmx-meta-tab .flex + .flex,
.dmx-meta-tab .dmx-topic-list {
  margin-top: var(--field-spacing);
}
</style>
