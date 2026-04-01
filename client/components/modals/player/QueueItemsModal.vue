<template>
  <modals-modal v-model="show" name="queue-items" :width="800" :height="'unset'">
    <template #outer>
      <div class="absolute top-0 left-0 p-5 w-2/3 overflow-hidden">
        <p class="text-3xl text-white truncate">{{ $strings.HeaderPlayerQueue }}</p>
      </div>
    </template>
    <div ref="container" class="w-full rounded-lg bg-bg box-shadow-md overflow-y-auto overflow-x-hidden py-4" style="max-height: 80vh">
      <div v-if="show" class="w-full h-full">
        <div class="pb-4 px-4 flex items-center">
          <p class="text-base text-gray-200">{{ $strings.HeaderPlayerQueue }}</p>
          <p class="text-base text-gray-400 px-4">{{ playerQueueItems.length }} Items</p>
          <div class="grow" />
          <ui-checkbox v-model="playerQueueAutoPlay" label="Auto Play" medium checkbox-bg="primary" border-color="gray-600" label-class="pl-2 mb-px" />
        </div>
        <draggable v-model="queueItemsCopy" v-bind="dragOptions" handle=".drag-handle" draggable=".queue-item" tag="div" @start="drag = true" @end="drag = false" @update="draggableUpdate">
          <transition-group type="transition" :name="!drag ? 'queue-item' : null">
            <modals-player-queue-item-row v-for="(item, index) in queueItemsCopy" :key="item.libraryItemId + '-' + (item.episodeId || index)" :is-dragging="drag" :item="item" :index="index" class="queue-item" @play="playItem(index)" @remove="removeItem" />
          </transition-group>
        </draggable>
      </div>
    </div>
  </modals-modal>
</template>

<script>
import draggable from 'vuedraggable'

export default {
  components: {
    draggable
  },
  props: {
    value: Boolean
  },
  data() {
    return {
      drag: false,
      dragOptions: {
        animation: 200,
        ghostClass: 'ghost',
        chosenClass: 'chosen'
      },
      queueItemsCopy: []
    }
  },
  computed: {
    show: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit('input', val)
      }
    },
    playerQueueAutoPlay: {
      get() {
        return this.$store.state.playerQueueAutoPlay
      },
      set(val) {
        this.$store.commit('setPlayerQueueAutoPlay', val)
      }
    },
    playerQueueItems() {
      return this.$store.state.playerQueueItems || []
    }
  },
  watch: {
    playerQueueItems: {
      handler() {
        this.initCopy()
      }
    },
    show(newVal) {
      if (newVal) this.initCopy()
    }
  },
  methods: {
    initCopy() {
      this.queueItemsCopy = this.playerQueueItems.map((i) => ({ ...i }))
    },
    draggableUpdate() {
      this.$store.commit('setPlayerQueueItems', this.queueItemsCopy)
    },
    playItem(index) {
      this.$eventBus.$emit('play-queue-item', {
        index
      })
      this.show = false
    },
    removeItem(item) {
      this.$store.commit('removeItemFromQueue', item)
    }
  },
  mounted() {
    this.initCopy()
  }
}
</script>

<style scoped>
.queue-item {
  transition: transform 0.2s ease;
}
.ghost {
  opacity: 0.3;
  border-top: 2px solid #4ade80;
}
.chosen {
  background: rgba(74, 222, 128, 0.1) !important;
  border-radius: 4px;
}
</style>
