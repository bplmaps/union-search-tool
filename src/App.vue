<template>
  <div id="app">
    <UnionSearch/>
  </div>
</template>

<script>
import UnionSearch from './components/UnionSearch.vue'


export default {
  name: 'App',
  components: {
    UnionSearch
  }
}
</script>

<style lang="scss">
@import "~bulma/sass/utilities/initial-variables.sass";
$primary: #006688;

@import "~bulma/bulma.sass";

#app {
  font-family: BlinkMacSystemFont, -apple-system, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", "Helvetica", "Arial", sans-serif;
}


</style>
