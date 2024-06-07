<template>
  <div class="history">
    <h3>History of URLs Fetched</h3>
    <ul>
      <li v-for="item in history" :key="item.url">
        <a @click="$emit('loadFromHistory', item.url)">{{ item.url }}</a>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  props: ["history"],
};
</script>

<style scoped>
.history {
  width: 100%;
}

.history h3 {
  color: #fff;
}

.history ul {
  list-style-type: none;
  padding: 0;
}

.history li {
  background: #333;
  margin-bottom: 5px;
  padding: 10px;
  border: 1px solid #444;
  border-radius: 5px;
}

.history li a {
  text-decoration: none;
  color: #1e90ff;
  cursor: pointer;
}

.history li a:hover {
  text-decoration: underline;
}
</style>
