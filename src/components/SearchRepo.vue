<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
       <v-autocomplete  
        v-model="model"
        :items="items"
        :loading="isLoading"
        :search-input.sync="search"
        color="#009270"
        hide-no-data
        hide-selected
        label="VueJS repository"
        placeholder="Start typing to Search"
        prepend-icon="mdi-database-search"
        return-object>
       <template v-slot:append-item>
         <v-row>
           <v-col class="text-center">
             <v-btn color="#009270" class="text-white" dark @click="more">Afficher plus de résultats</v-btn>

           </v-col>
         </v-row>
       </template>
    
        </v-autocomplete>
      </v-col>
      <v-row>
        
        <v-col v-for="(select, i) of selected" :key="`item-${i}`">
          <v-card>
            <v-card-title>{{select.name}}</v-card-title>
            <v-card-text>{{select.description}}</v-card-text>
          </v-card>
        </v-col>
      </v-row>
    
    </v-row>
  </v-container>
</template>

<script>
const { Octokit } = require("@octokit/core");
  export default {
    name: 'SearchRepo',

    data: () => ({
      entries: [],
      model: undefined,
      isLoading: false,
      search: undefined,
      currentPageNbr: 1,
      selected: []
    }),

    computed: {
      items () {
        return this.entries.map(entry => {
         
          return {text: entry.name, value: entry.id}
        })
      },
    },

    methods: {
      /**
       * search projects   
       **/
      async call(name, pageNbr = 1) {
        
        const octokit = new Octokit({
          auth: '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'
        })

        const res = await octokit.request(`GET /search/repositories?page=${pageNbr}&per_page=10`, {
          q: `${name} in:file language:vue`,
          sort: 'stars',
          order: 'desc'
        })
        
        return res.data
        
      
      },
      async more() {
        this.currentPageNbr++

          this.isLoading = true
        const res = await this.call(this.search, this.currentPageNbr )
        const { total_count, items } = res
        
        this.count = total_count
        
        this.entries = this.entries.concat(items) 
        this.isLoading = false
        
      }
    },
    watch: {
      async search (val) {

        if(val) {

          // Items have already been loaded
          if (this.items.length > 0) return
  
          // Items have already been requested
          if (this.isLoading) return
  
          this.isLoading = true
  
          // Lazily load input items
          try {
            const res = await this.call(val)
  
            const { total_count, items } = res
        
            this.count = total_count
            this.entries = items
          }catch(err) {
            console.error(err)
          }finally {
            this.isLoading = false
          }
        }
         
      },
      model(val) {        
        this.selected.push(this.entries.find(entry => entry.id === val.value))
      }
    }
  }
</script>
