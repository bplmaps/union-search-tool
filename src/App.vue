
<template>
<section id="search">
  <union-search></union-search>
</section>
</template>

<script lang="ts">
import Vue from "vue";
import UnionSearch from "./UnionSearch";
import './style.scss';

  export default Vue.extend({

    components: {
      UnionSearch
    }

  });
</script>

<style>
    
</style>
