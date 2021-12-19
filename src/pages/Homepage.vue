<template>
  <div>
    <h1>Eurosong voting app!</h1>
    <div class="center">
      <button class="cybr-btn" @click="goToPage('ranking')">
        Check the ranking<span aria-hidden>_</span>
        <span aria-hidden class="cybr-btn__glitch">Check the ranking</span>
        <span aria-hidden class="cybr-btn__tag">#2</span>
      </button>
      <button class="cybr-btn" @click="goToPage('game')">
        Check the votes<span aria-hidden>_</span>
        <span aria-hidden class="cybr-btn__glitch">Check the votes</span>
        <span aria-hidden class="cybr-btn__tag">#1</span>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "Homepage",
  methods: {
    goToPage(page) {
      this.$emit("change-page", page);
    },
  },
};
</script>
