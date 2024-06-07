<template>
  <div class="chatbot-container">
    <ChatWindow :messages="messages" :loading="loading" />
    <InputArea
      :url="url"
      :searchQuery="searchQuery"
      @updateUrl="url = $event"
      @updateSearchQuery="searchQuery = $event"
      @fetchContent="fetchContent"
      @searchContent="searchContent"
      @clearSearch="clearSearch"
      @clearHistory="clearHistory"
      @showDifferences="showDifferences"
    />
    <HistoryList :history="history" @loadFromHistory="loadFromHistory" />
  </div>
</template>

<script>
import axios from "axios";
import { diffWords } from "diff";
import ChatWindow from "./ChatWindow.vue";
import InputArea from "./InputArea.vue";
import HistoryList from "./HistoryList.vue";

const allowedDomains = ["example.com", "another-trusted-domain.com"];

function isAllowedDomain(url) {
  const { hostname } = new URL(url);
  return allowedDomains.includes(hostname);
}

function isValidUrl(url) {
  const pattern = new RegExp(
    "^(https?:\\/\\/)?" + // protocol
      "((([a-zA-Z\\d]([a-zA-Z\\d-]*[a-zA-Z\\d])*)\\.)+[a-zA-Z]{2,}|" + // domain name
      "((\\d{1,3}\\.){3}\\d{1,3}))" + // OR ip (v4) address
      "(\\:\\d+)?(\\/[-a-zA-Z\\d%_.~+]*)*" + // port and path
      "(\\?[;&a-zA-Z\\d%_.~+=-]*)?" + // query string
      "(\\#[-a-zA-Z\\d_]*)?$",
    "i" // fragment locator
  );
  return !!pattern.test(url);
}

export default {
  components: {
    ChatWindow,
    InputArea,
    HistoryList,
  },
  data() {
    return {
      url: "",
      content: "",
      previousContent: "",
      searchQuery: "",
      filteredContent: "",
      loading: false,
      error: "",
      history: [],
      messages: [],
    };
  },
  computed: {
    formattedUrl() {
      let formattedUrl = this.url.trim();
      if (
        !formattedUrl.startsWith("http://") &&
        !formattedUrl.startsWith("https://")
      ) {
        formattedUrl = "https://" + formattedUrl;
      }
      return formattedUrl;
    },
    highlightedContent() {
      if (!this.searchQuery) return this.filteredContent;
      const regex = new RegExp(
        `(${this.escapeRegExp(this.searchQuery)})`,
        "gi"
      );
      return this.filteredContent.replace(regex, "<mark>$1</mark>");
    },
    wordCount() {
      return this.filteredContent
        ? this.filteredContent.split(/\s+/).filter((word) => word).length
        : 0;
    },
    summary() {
      if (!this.filteredContent) return "";
      const sentences = this.filteredContent.split(". ");
      const summarySentences = sentences.slice(
        0,
        Math.min(3, sentences.length)
      );
      return summarySentences.join(". ") + (sentences.length > 3 ? "..." : "");
    },
  },
  methods: {
    async fetchContent() {
      this.loading = true;
      this.error = "";
      try {
        if (!isAllowedDomain(this.formattedUrl)) {
          throw new Error("Domain not allowed");
        }
        if (!isValidUrl(this.formattedUrl)) {
          throw new Error("Invalid URL");
        }
        const response = await axios.post(
          "http://192.168.68.54:5001/api/extract",
          { url: this.formattedUrl }
        );
        this.previousContent = this.content;
        this.content = response.data.text;
        this.filteredContent = this.content;
        this.addToHistory(this.formattedUrl, this.content);
        this.addMessage(
          "bot",
          `
          <p>${this.filteredContent}</p>
          <p><strong>Word Count:</strong> ${this.wordCount}</p>
          <p><strong>Summary:</strong> ${this.summary}</p>
        `
        );
      } catch (error) {
        console.error(error);
        this.error =
          "Failed to fetch content. Please check the URL and try again.";
        this.content = "";
        this.filteredContent = "";
        this.addMessage("bot", `<p class="error">${this.error}</p>`);
      } finally {
        this.loading = false;
      }
    },
    searchContent() {
      if (!this.searchQuery) {
        this.filteredContent = this.content;
      } else {
        const regex = new RegExp(this.escapeRegExp(this.searchQuery), "gi");
        this.filteredContent = this.content
          .split("\n")
          .filter((line) => regex.test(line))
          .join("\n");
      }
    },
    clearSearch() {
      this.searchQuery = "";
      this.filteredContent = this.content;
    },
    clearHistory() {
      this.history = [];
      this.saveHistory();
    },
    resetContent() {
      this.content = "";
      this.filteredContent = "";
      this.error = "";
      this.searchQuery = "";
    },
    escapeRegExp(string) {
      return string.replace(/[.*+?^${}()|[\]\\]/g, "\\$&");
    },
    addToHistory(url, content) {
      const existingIndex = this.history.findIndex((item) => item.url === url);
      if (existingIndex === -1) {
        this.history.push({ url, content });
      } else {
        this.history[existingIndex].content = content;
      }
      this.saveHistory();
    },
    saveHistory() {
      localStorage.setItem("urlHistory", JSON.stringify(this.history));
    },
    loadHistory() {
      const history = localStorage.getItem("urlHistory");
      if (history) {
        this.history = JSON.parse(history);
      }
    },
    loadFromHistory(url) {
      const historyItem = this.history.find((item) => item.url === url);
      if (historyItem) {
        this.previousContent = this.content;
        this.content = historyItem.content;
        this.filteredContent = this.content;
        this.url = url;
        this.addMessage(
          "bot",
          `
          <p>${this.highlightedContent}</p>
          <p><strong>Word Count:</strong> ${this.wordCount}</p>
          <p><strong>Summary:</strong> ${this.summary}</p>
        `
        );
      }
    },
    showDifferences() {
      const diffResult = diffWords(this.previousContent, this.content);
      const diffHtml = diffResult
        .map((part) => {
          const color = part.added ? "green" : part.removed ? "red" : "grey";
          return `<span style="color:${color}">${part.value}</span>`;
        })
        .join("");
      this.addMessage(
        "bot",
        `<p>Differences between current and previous content:</p><p>${diffHtml}</p>`
      );
    },
    addMessage(type, content) {
      const timestamp = new Date().toLocaleTimeString();
      this.messages.push({ id: Date.now(), type, content, timestamp });
    },
  },
  created() {
    this.loadHistory();
  },
};
</script>
