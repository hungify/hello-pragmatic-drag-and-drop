<script lang="ts" setup></script>

<template>
  <div class="container">
    <div class="card">
      <svg
        class="svg"
        preserveAspectRatio="none"
        stroke="currentColor"
        fill="none"
        viewBox="0 0 200 200"
        aria-hidden="true"
      >
        <path vectorEffect="non-scaling-stroke" strokeWidth="2" d="M0 0l200 200M0 200L200 0"></path>
      </svg>
    </div>
  </div>
</template>

<style scoped>
:root {
  --card-background-color: rgb(255, 255, 255);
}

*,
::after,
::before {
  box-sizing: border-box;
}

.container {
  background-color: rgb(255, 255, 255);
  width: 100%;
  height: 100%;
}

.card {
  height: 100%;
  padding: 1.5rem;
}

.svg {
  background-color: rgb(255, 255, 255);
  border-color: rgba(128, 128, 128, 0.35);
  border-radius: 0.25rem;
  border-style: dashed;
  border-width: 2px;
  color: rgba(128, 128, 128, 0.35);
  height: 100%;
  width: 100%;
}
</style>
