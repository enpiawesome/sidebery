<template lang="pug">
.HiddenPanelsBar(
  @dragenter="onDragEnter"
  @dragleave="onDragLeave"
  @dragover.prevent.stop="")
  .hidden-panel(
    v-for="(panel, i) in hiddenPanels"
    :data-color="panel.color"
    :data-active="selected === i"
    :title="panel.name"
    @click="clickHandler(panel)"
    @dragenter="onPanelDragEnter(panel, i)"
    @drop="onPanelDrop($event, panel, i)"
    @contextmenu.stop="onNavCtxMenu($event, panel.index)"
    @mousedown.right="onPanelRightClick(panel)"
    @mouseup.right="onNavRightMouseup($event, panel.index)")
    img(v-if="!!panel.customIcon" :src="panel.customIcon")
    svg(v-else): use(:xlink:href="'#' + panel.icon")
    .name {{panel.name}}
</template>

<script>
import EventBus from '../../event-bus'
import State from '../store/state'
import Actions from '../actions'

export default {
  data() {
    return {
      selected: -1,
    }
  },

  computed: {
    hiddenPanels() {
      return State.panels.filter(b => !b.bookmarks && b.inactive)
    },
  },

  created() {
    EventBus.$on('selectHiddenPanel', this.onSelectHiddenPanel)
    EventBus.$on('createTabInHiddenPanel', this.onCreateTabInHiddenPanel)
  },

  beforeDestroy() {
    EventBus.$off('selectHiddenPanel', this.onSelectHiddenPanel)
    EventBus.$off('createTabInHiddenPanel', this.onCreateTabInHiddenPanel)
  },

  methods: {
    clickHandler(panel) {
      State.hiddenPanelsBar = false
      Actions.createTabInPanel(panel)
    },

    onSelectHiddenPanel(dir) {
      if (this.selected + dir >= this.hiddenPanels.length) return
      if (this.selected + dir < 0) {
        State.hiddenPanelsBar = false
        State.panelIndex = State.lastPanelIndex
      }
      this.selected += dir
    },

    /**
     * Handle right mouseup event
     */
    onNavRightMouseup(e, i) {
      if (State.selected.length) return

      let panel = State.panels[i]
      if (!panel) return

      e.stopPropagation()

      if (State.ctxMenuNative) return

      let type
      if (panel.type === 'bookmarks') type = 'bookmarksPanel'
      else if (panel.type === 'default') type = 'tabsPanel'
      else if (panel.type === 'tabs') type = 'tabsPanel'

      State.selected = [panel]
      Actions.openCtxMenu(type, e.clientX, e.clientY)
    },

    /**
     * Handle context menu event
     */
    onNavCtxMenu(e, i) {
      if (!State.ctxMenuNative || e.ctrlKey || e.shiftKey) {
        e.stopPropagation()
        e.preventDefault()
        return
      }

      let panel = State.panels[i]
      if (!panel) return

      if (State.ctxMenuBlockTimeout) {
        e.stopPropagation()
        e.preventDefault()
        return
      }

      let nativeCtx = { showDefaults: false }
      browser.menus.overrideContext(nativeCtx)

      let type
      if (panel.type === 'bookmarks') type = 'bookmarksPanel'
      else if (panel.type === 'default') type = 'tabsPanel'
      else if (panel.type === 'tabs') type = 'tabsPanel'
      if (!State.selected.length) State.selected = [panel]

      Actions.openCtxMenu(type)
    },

    onCreateTabInHiddenPanel() {
      let panel = this.hiddenPanels[this.selected]
      if (!panel || panel.bookmarks) return
      Actions.createTabInPanel(panel)
      State.hiddenPanelsBar = false
    },

    onDragEnter(event) {
      this.dragEnterTarget = event.target
    },

    onDragLeave(event) {
      if (event.target !== this.dragEnterTarget) return
      State.hiddenPanelsBar = false
    },

    onPanelDragEnter(panel, i) {
      this.selected = i
    },

    onPanelDrop(event, panel) {
      State.panelIndex = panel.index
    },

    onPanelRightClick() {
      // ...
    },
  },
}
</script>
