
<template>

  <div class="container is-fluid py-5 px-3">

    
        <div class="field">
      <p class="control is-expanded">
        <input v-model="searchInput" @keyup.enter="submitSearch" class="input is-large" type="text" placeholder="Enter titles, creators, or subjects">
      </p>
      </div>

        <div class="field has-addons">
          <p class="control is-expanded">
        <span class="select is-fullwidth">
          <select name="search-type" id="" v-model="searchType">
            <option value="both">Search both repositories</option>
            <option value="dc">Digital Collections only</option>
            <option value="ia">Internet Archive only</option>
          </select>
        </span>
      </p>
      <p class="control">
        <a class="button is-primary" v-bind:class="searchAllowed" v-on:click="submitSearch" >
          Search
        </a>
      </p>
        </div>
    


    <div id="results" :class="[this.digitalCollections.searchStatus === 'initial' && this.internetArchive.searchStatus === 'initial' ? 'is-hidden' : '']" class="has-background-white-ter px-5 py-4">
     
     <div v-if="searchType==='both' || searchType==='dc'" >
     <div :class="[this.digitalCollections.searchStatus === 'searching' ? '' : 'is-hidden']">
     Loading Digital Collections repository results <progress class="progress is-medium is-primary mt-2" max="100"></progress>
     </div>

     <div id="dc-search-results" class="mb-5" :class="[this.digitalCollections.searchStatus === 'returned' ? '' : 'is-hidden']">

      <h4 class="title is-4 mb-4">Results from Digital Collections repository</h4>
      <div class="columns">
       <div class="column is-one-quarter" v-bind:key="result" v-for="result in digitalCollections.results">
       <div class="card"  v-on:click="goToResult(result.url)">

         <div class="result-inner">
           <div class="card-image">
                <figure class="image is-4by3">
                  <img :src="result.image" :alt="'Thumbnail for' + result.title">
                </figure>
              </div>
          <div class="card-content">
            <h5>{{ result.title }}</h5>
            <h6 :class="[result.date === '' ? 'is-hidden' : '']">{{ result.date }}</h6>
          </div>
         </div>
        </div>
        </div>

        <div class="column is-one-quarter">
          <div class="box"><div class="mb-2">
            <strong>{{ digitalCollections.totalResults == 0 ? "No" : digitalCollections.totalResults }} results</strong> from Digital Collections repository
          </div><a class="button is-primary" :href="'https://collections.leventhalmap.org/search?utf8=✓&q=' + enteredSearchInput" target="_blank" :class="[digitalCollections.totalResults == 0 ? 'is-hidden' : '']">See all</a></div>
        </div>
        </div>
     </div>

     </div>


          <div v-if="searchType==='both' || searchType==='ia'" >


      <div :class="[this.internetArchive.searchStatus === 'searching' ? '' : 'is-hidden']">
     Loading Internet Archive results <progress class="progress is-medium is-primary mt-2" max="100"></progress>
     </div>

      <div id="ia-search-results" :class="[this.internetArchive.searchStatus === 'returned' ? '' : 'is-hidden']">

      <h4 class="title is-4 mb-4">Results from Internet Archive</h4>
      <div class="columns">
       <div class="column is-one-quarter" v-bind:key="result" v-for="result in internetArchive.results">
       <div class="card" v-on:click="goToResult(result.url)">
         <div class="result-inner">
           <div class="card-image">
                <figure class="image is-4by3">
                  <img class="thumbnail" :src="result.image" :alt="'Thumbnail for' + result.title">
                </figure>
              </div>
          <div class="card-content">
            <h5>{{ trimmer(result.title) }}</h5>
            <h6 :class="[result.date ? '' : 'is-hidden']">{{ result.date }}</h6>
          </div>
         </div>
        </div>
        </div>

        <div class="column is-one-quarter">
          <div class="box"><div class="mb-2">
            <strong>{{ internetArchive.totalResults == 0 ? "No" : internetArchive.totalResults }} results</strong> from Internet Archive
          </div><a class="button is-primary" :href="'https://archive.org/search.php?query=%28' + enteredSearchInput + '%29%20AND%20collection%3A%28normanbleventhalmapcenter%29'" target="_blank" :class="[internetArchive.totalResults == 0 ? 'is-hidden' : '']">See all</a></div>
        </div>
        </div>
     </div>
     </div>
    

    </div>

    

</div>

</template>

<script lang="ts">
import Vue from "vue";


  export default Vue.extend({

    data: function() {
      return {
        state: 'initial',
        searchInput: '',
        enteredSearchInput: '',
        searchType: 'both',
        digitalCollections: {
          searchStatus: 'initial',
          results: [],
          totalResults: 0
        },
        internetArchive: {
          searchStatus: 'initial',
          results: [],
          totalResults: 0
        }
      }
    },

    computed: {
      searchAllowed: function() {
        if(this.digitalCollections.searchStatus === 'searching' || this.internetArchive.searchStatus === 'searching') { 
          return "is-static";
        } else { return "" }
      }
      
      

    },

    methods: {
            trimmer: function(s){ 
        if( s.length > 160 )
          { return s.substr(0,120) + "…"; }
        else 
          { return s; }
      },

      submitSearch: function(){
        this.enteredSearchInput = this.searchInput;
        this.digitalCollections.searchStatus = 'searching';
        this.digitalCollections.results = [];
        this.internetArchive.searchStatus = 'searching';
        this.internetArchive.results = [];

        fetch(`https://collections.leventhalmap.org/search.json?&per_page=3&q=${this.searchInput}`)
          .then(response => response.json())
          .then(result => {
              this.digitalCollections.searchStatus = 'returned';
              this.digitalCollections.totalResults = result.response.pages.total_count;
              result.response.docs.forEach(r => {
                this.digitalCollections.results.push({
                  title: r.title_info_primary_tsi,
                  date: r.date_start_tsim[0],
                  id: r.id,
                  image: `https://fedora.digitalcommonwealth.org/fedora/objects/${r.exemplary_image_ssi}/datastreams/thumbnail300/content
`,
                  url: `https://collections.leventhalmap.org/search/${r.id}`
                });
              });
          })
        .catch(() => {this.digitalCollections.searchStatus = 'failed'; this.digitalColletions.totalResults = 0; });

        fetch(`https://archive.org/advancedsearch.php?q=%28${this.searchInput}%29+AND+collection%3A%28normanbleventhalmapcenter%29&fl%5B%5D=identifier&fl%5B%5D=title&fl%5B%5D=year&sort%5B%5D=&sort%5B%5D=&sort%5B%5D=&rows=3&page=1&output=json&save=yes`)
          .then(response => response.json())
          .then(result => {
              this.internetArchive.searchStatus = 'returned';
              this.internetArchive.totalResults = result.response.numFound;
              result.response.docs.forEach(r => {
                this.internetArchive.results.push({
                  title: r.title,
                  date: r.year,
                  id: r.identifier,
                  image: `https://archive.org/services/img/${r.identifier}`,
                  url: `https://archive.org/details/${r.identifier}`
                });
              });
          })
        .catch(() => {this.internetArchive.searchStatus = 'failed'; this.internetArchive.totalResults = 0; });
      }
      ,
      
      goToResult: function(u){
      window.open(u);
    },
    
    },



  });
</script>

<style scoped>

.container {
  background-color: rgb(248, 248, 248);
  box-shadow: inset 0px 0px 5px 1px rgba(0,0,0,0.5);

}

.card {
    transition: background-color 0.5s ease;
    cursor: pointer;

    img { 
        object-fit: cover;
    }

    h6 {
        color: $grey-light;
        font-size: 80%;
    }
}

.card:hover {
  background: rgba(204, 198, 226, 0.199);
}

.card-content h5 {
  font-weight: 600;
  overflow: hidden;
}

.card-content h6 {
  color: #555;
  font-weight: 300;
}

.thumbnail {
  object-fit: contain;
}

@media (max-width: 768px) {

    .result-inner {
        display: flex;
        flex-direction: row;
    }

    .card-image {
        display: block;
        width: 25%;
    }

    .card-content {
        display: block;
        width: 75%;
        padding: 5px;
        
    }
}
</style>
