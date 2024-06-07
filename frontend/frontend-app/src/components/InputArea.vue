<template>
  <div class="input-area">
    <input
      v-model="internalUrl"
      placeholder="Enter URL"
      @input="$emit('updateUrl', internalUrl)"
      class="input-field"
    />
    <button @click="fetchContent" class="btn">Submit</button>
    <input
      v-model="internalSearchQuery"
      placeholder="Search extracted content"
      @input="$emit('updateSearchQuery', internalSearchQuery)"
      class="input-field"
    />
    <button @click="$emit('clearSearch')" class="btn">Clear Search</button>
    <button @click="$emit('clearHistory')" class="btn">Clear History</button>
    <button @click="$emit('showDifferences')" class="btn">
      Show Differences
    </button>
  </div>
</template>

<script>
export default {
  props: ["url", "searchQuery"],
  data() {
    return {
      internalUrl: this.url,
      internalSearchQuery: this.searchQuery,
    };
  },
  methods: {
    fetchContent() {
      this.$emit("fetchContent");
    },
  },
  watch: {
    url(newVal) {
      this.internalUrl = newVal;
    },
    searchQuery(newVal) {
      this.internalSearchQuery = newVal;
    },
  },
};
</script>
