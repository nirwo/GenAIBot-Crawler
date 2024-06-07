<template>
  <div class="chat-window">
    <div
      v-for="message in messages"
      :key="message.id"
      :class="['message', message.type]"
    >
      <div v-html="message.content"></div>
      <small>{{ message.timestamp }}</small>
    </div>
    <div v-if="loading" class="message bot">Loading...</div>
  </div>
</template>

<script>
export default {
  props: ["messages", "loading"],
  mounted() {
    console.log("ChatWindow mounted");
  },
};
</script>
