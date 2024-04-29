<script lang="ts" setup>
import { draggable } from '@atlaskit/pragmatic-drag-and-drop/element/adapter'
import { disableNativeDragPreview } from '@atlaskit/pragmatic-drag-and-drop/element/disable-native-drag-preview'
import { preventUnhandled } from '@atlaskit/pragmatic-drag-and-drop/prevent-unhandled'
import type { DragLocationHistory } from '@atlaskit/pragmatic-drag-and-drop/types'
import { ref, watchEffect } from 'vue'

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
  <div class="wrapper">
    <div
      ref="contentRef"
      class="sidebar"
      :style="{
        '--local-initial-width': `${initialWidth}px`
      }"
    >
      <ul>
        <li>Item 1</li>
        <li>Item 2</li>
        <li>Item 3</li>
      </ul>
    </div>
    <div ref="dividerRef" class="divider"></div>
  </div>
</template>

<style scoped>
.wrapper {
  width: var(--local-width);
  flex-shrink: 0;
  flex-grow: 0;
  display: flex;
  flex-direction: row;
}

.sidebar {
  flex-grow: 1;
  flex-shrink: 1;
  width: var(--local-resizing-width, var(--local-initial-width));
}

.sidebar ul {
  list-style: none;
  padding: 0;
  margin: 0;
  width: 100%;
}

.divider {
  width: 32px;
  cursor: ew-resize;
  flex-grow: 0;
  flex-shrink: 0;
  position: relative;
  background: transparent;
}
.divider::before {
  background: #0c66e4;
  content: '';
  position: absolute;
  top: 0;
  bottom: 0;
  width: 1px;
}
</style>
