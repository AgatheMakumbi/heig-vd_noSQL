<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>

<script lang="ts">
import PouchDB from "pouchdb";
export default {
  data() {
    return {
      total: 10,
    };
  },
  methods: {
    inc() {
      this.total++;
    },

    iniDatabase() {
      const db = new PouchDB("http://localhost:5984/post_insta");
      db.info().then(function (info) {
        console.log(info);
      });
    },

    fetchData() {},
  },

  mounted() {
    this.iniDatabase();
  },
};
</script>

<template>
  <div>
    <h1>My counter be like</h1>
    <p>Counter: {{ total }}</p>
    <button @click="inc">+1</button>
  </div>
</template>
