<template>
  <div class="chatbot-container">
    <div class="preview">
      <iframe :src="url" frameborder="0"></iframe>
    </div>
    <div class="chat">
      <input v-model="url" placeholder="Enter URL" @keydown.enter="fetchContent">
      <textarea v-model="content" placeholder="Extracted content"></textarea>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      url: '',
      content: '',
      error: '',
    };
  },
  methods: {
    async fetchContent() {
      try {
        const response = await axios.post('http://192.168.68.54:5001/api/extract', { url: this.url });
        this.content = response.data.text;
      } catch (error) {
        console.error(error);
        this.error = 'Failed to fetch content';
      }
    },
  },
};
</script>

<style>
.chatbot-container {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: flex-start;
}

.preview {
  flex: 1;
  margin-right: 20px;
}

.preview iframe {
  width: 100%;
  height: 500px;
}

.chat {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.chat input, .chat textarea {
  margin-bottom: 10px;
  padding: 10px;
}
</style>
