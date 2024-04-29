<script lang="ts" setup>
import { draggable } from '@atlaskit/pragmatic-drag-and-drop/element/adapter'
import { disableNativeDragPreview } from '@atlaskit/pragmatic-drag-and-drop/element/disable-native-drag-preview'
import { preventUnhandled } from '@atlaskit/pragmatic-drag-and-drop/prevent-unhandled'
import type { DragLocationHistory } from '@atlaskit/pragmatic-drag-and-drop/types'
import { ref, watchEffect } from 'vue'
import Content from './components/Content.vue'

type State =
  | {
      type: 'idle'
    }
  | {
      type: 'dragging'
    }

const widths = {
  start: 260,
  min: 150,
  max: 450
}

function getProposedWidth({
  initialWidth,
  location
}: {
  initialWidth: number
  location: DragLocationHistory
}): number {
  const diffX = location.current.input.clientX - location.initial.input.clientX
  const proposedWidth = initialWidth + diffX

  // ensure we don't go below the min or above the max allowed widths
  return Math.min(Math.max(widths.min, proposedWidth), widths.max)
}

const initialWidth = ref(widths.start)
const dividerRef = ref<HTMLDivElement | null>(null)
const state = ref<State>({ type: 'idle' })
const contentRef = ref<HTMLDivElement | null>(null)

watchEffect(() => {
  if (!dividerRef.value) return

  draggable({
    element: dividerRef.value!,
    onGenerateDragPreview: ({ nativeSetDragImage }) => {
      // we will be moving the line to indicate a drag
      // we can disable the native drag preview
      disableNativeDragPreview({ nativeSetDragImage })
      // we don't want any native drop animation for when the user
      // does not drop on a drop target. we want the drag to finish immediately
      preventUnhandled.start()
    },
    onDragStart() {
      state.value = { type: 'dragging' }
    },
    onDrag({ location }) {
      contentRef.value?.style.setProperty(
        '--local-resizing-width',
        `${getProposedWidth({ initialWidth: initialWidth.value, location })}px`
      )
    },
    onDrop({ location }) {
      preventUnhandled.stop()
      state.value = { type: 'idle' }

      initialWidth.value = getProposedWidth({ initialWidth: initialWidth.value, location })
      contentRef.value?.style.removeProperty('--local-resizing-width')
    }
  })
})
</script>

<template>
  <div class="container">
    <div class="split-view">
      <div class="sash-container" ref="contentRef">
        <div
          class="sash"
          ref="dividerRef"
          :style="{
            '--local-initial-width': `${initialWidth}px`,
            cursor: state.type === 'dragging' ? 'ew-resize' : 'col-resize'
          }"
        ></div>
      </div>
      <div
        :style="{
          '--local-initial-width': `${initialWidth}px`
        }"
      >
        <Content
          :style="{
            width: `${initialWidth}px`
          }"
        />
      </div>
      <div
        :style="{
          width: `${initialWidth}px`
        }"
      >
        <Content
          :style="{
            '--local-initial-width': `${initialWidth}px`
          }"
        />
      </div>
    </div>
  </div>
</template>

<style>
:root {
  --focus-border: #007fd4;
  --sash-size: 8px;
  --sash-hover-size: 4px;
}

.container {
  border: 1px solid rgba(128, 128, 128, 0.35);
  box-shadow:
    0 1px 3px 0 rgba(0, 0, 0, 0.1),
    0 1px 2px 0 rgba(0, 0, 0, 0.06);
  font-family: sans-serif;
  height: 80vh;
  overflow: scroll;
  resize: both;
  width: 80vw;
}
.split-view {
  height: 100%;
  overflow: hidden;
  position: relative;
  width: 100%;
}
.sash-container {
  height: 100%;
  pointer-events: none;
  position: absolute;
  width: 100%;
}

.sash {
  position: absolute;
  z-index: 35;
  touch-action: none;
  pointer-events: auto;
  text-align: initial;

  cursor: ew-resize;
  top: 0;
  width: var(--sash-size);
  height: 100%;

  cursor: col-resize;
  left: var(--local-resizing-width, var(--local-initial-width));
}

.sash::before {
  content: '';
  pointer-events: none;
  position: absolute;
  width: 100%;
  height: 100%;
  transition: background-color 0.1s ease-out;
  background-color: rgba(128, 128, 128, 0.35);
  width: var(--sash-hover-size);
  left: calc(50% - (var(--sash-hover-size) / 2));
}
</style>
