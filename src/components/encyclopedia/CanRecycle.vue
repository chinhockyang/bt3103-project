<template>
  <div class=" container tab-pane fade show active mb-5" id="can-recycle" role="tabpanel" aria-labelledby="can-recycle-tab">
    <h1 class="mt-3">What {{ topic }} I Recycle</h1>

    <div class="input-group mt-3">
      <input type="text" class="form-control" v-model="search" placeholder="Search title.."/><br><br>      
    </div>    

    <button 
        class="collapsed btn btn-outline-secondary w-50 m-3"
        type="button"
        data-toggle="collapse" 
        data-target="#filteritem" 
        aria-expanded="false" 
        aria-controls="filteritem">
        Tools
    </button>
    
    <div 
      id="filteritem" 
      class="collapse w-75 mx-auto mb-3"
      aria-labelledby="filterheader" 
      data-parent="#filteraccordion">
      <div class="card">
        <div class="card-body">
          <span><strong>Filter Categories:</strong></span><br>
          <div class="form-check form-check-inline">
            <input class="form-check-input" type="checkbox" id="plastic" value="Plastic" v-model="checkedList" checked>
            <label class="form-check-label" for="plastic">Plastic</label>
          </div>          
          <div class="form-check form-check-inline">
            <input class="form-check-input" type="checkbox" id="paper" value="Paper" v-model="checkedList" checked>
            <label class="form-check-label" for="paper">Paper</label>          
          </div>
          <div class="form-check form-check-inline">
            <input class="form-check-input" type="checkbox" id="glass" value="Glass" v-model="checkedList" checked>
            <label class="form-check-label" for="glass">Glass</label>            
          </div>
          <div class="form-check form-check-inline">
            <input class="form-check-input" type="checkbox" id="metal" value="Metal" v-model="checkedList" checked>
            <label class="form-check-label" for="metal">Metal</label>            
          </div>
          <div class="form-check form-check-inline">
            <input class="form-check-input" type="checkbox" id="electronics" value="Electronics" v-model="checkedList" checked>
            <label class="form-check-label" for="electronics">Electronics</label>            
          </div>
          <div class="form-check form-check-inline">
            <input class="form-check-input" type="checkbox" id="others" value="Others" v-model="checkedList" checked>
            <label class="form-check-label" for="others">Others</label><br><br>            
          </div><br>
          <span><strong>Sort By:</strong></span>
          <div v-if="canRecycle" class="form-check">
              <input class="form-check-input" type="radio" id="recycled" value="recycled" v-model="sort">
              <label class="form-check-label" for="recycled">Sort By Amount Recycled</label><br>
          </div>          
          <div v-if="canRecycle" class="form-check">
              <input class="form-check-input" type="radio" id="searched" value="searched" v-model="sort">
              <label class="form-check-label" for="searched">Sort By Amount Searched</label><br>
          </div>
          <div v-else class="form-check">
              <input class="form-check-input" type="radio" id="searched" value="searched" v-model="sort" disabled>
              <label class="form-check-label" for="searched">Sort By Amount Searched</label>
          </div><br>          
          <span><strong>Items per Page:</strong></span><br><br>          
          <div class="form-group">
              <select class="form-control" v-model.number="pageSize">    
                <option v-for="n in 10" :key="n">
                  {{n}}
                </option>   
              </select>              
          </div>
        </div>
      </div>   
    </div>   

    <pagination 
      :items="filteredList" 
      @page:update="updatePage"
      :currentPage="currentPage"
      :pageSize="pageSize">
    </pagination>

    <ul class="row justify-content-center bg-light">
        <li v-for="item in visibleItems"
            class="col-s-8 col-md-6 col-l-4 m-3 justify-content-center"  
            style="max-width: 285px;"
            v-bind:key="item.name"            
            :title="'Click for more information on ' + item.name">            
            <item-card :item="item"></item-card>
        </li>
    </ul>
    
    <pagination 
      :items="filteredList" 
      @page:update="updatePage"
      :currentPage="currentPage"
      :pageSize="pageSize">
    </pagination>
  </div>
</template>


<script>
import database from '../../firebase.js'
import ItemCard from './ItemCard.vue'
import Pagination from './Pagination.vue'

export default {

  data(){
    return{
        // whether the page shows "can" or "cannot"
        topic: 'can',

        // whether the page renders items for "can" or "cannot" recycle
        canRecycle: true,

        // what is entered in the search
        search: '',

        //itemsList is all the items that can/cannot be recycled retrieved from database
        //visibleItems is the items shown all the current page (of pagination)
        itemsList: [],        
        visibleItems: [],        
        
        //currentPage: page of pagination - 1
        //pageSize: number of items shown per page
        currentPage: 0,
        pageSize: 6,        

        //items to be shown (filter)
        checkedList: ['Plastic', 'Paper', 'Glass', 'Metal', 'Electronics', 'Others'],
        sort: "searched"
    }
  },
  
  methods:{
    fetchItems:function(){
      var sortReq = (this.sort == "searched") ? "amountSearched" : "amountRecycled";
      database.collection('items')
      .orderBy(sortReq, "desc")    
      .get().then((querySnapShot)=>{
        let item={}
        querySnapShot.forEach(doc=>{
            item=doc.data()
            item.show=false
            item.id=doc.id            
            if (this.canRecycle == true) {
              if (item.approved & item.recyclable) {                
                this.itemsList.push(item);
              } 
            } else {
              if (item.approved & !item.recyclable) {                
                this.itemsList.push(item);
              }
            }            
            })})    
      },

      //updates the page number of pagination
      updatePage: function(pageNumber) {
        this.currentPage = pageNumber;        
      },

      //slice filteredList to get only the items on the current page
      updateVisibleItems: function() {
        this.visibleItems = this.filteredList.slice(this.currentPage * this.pageSize, parseInt((this.currentPage * this.pageSize) + this.pageSize));

        // if there are 0 items on current page, go back 1 page
        if (this.visibleItems.length == 0 && this.currentPage > 0) {
          this.updatePage(this.currentPage - 1);
        }
      },
  },

  computed: {
    // filter itemsList based on the search data
    filteredList: function() {
      return this.itemsList
      .filter(item => this.checkedList.includes(item.category))
      .filter(item => {
        return item.name.toLowerCase().includes(this.search.toLowerCase())
        });
    }
  },

  watch: {
    //render "can" or "cannot" based on $route param 
    canRecycle: function(val) {      
      return this.topic = eval(val) ? "Can" : "Can't";
    }, 

    // if $route param changed to false (cannot recycle)
    // clear the itemsList and retrieve from database again
    $route: function(val) {
      this.canRecycle = eval(val.params.id);      
      this.sort = "searched";
      this.itemsList = [];
      this.visibleItems = [];
      this.fetchItems();
      this.updateVisibleItems();
    },

    sort: function() {      
      this.itemsList = [];
      this.visibleItems = [];
      this.fetchItems();
      this.updateVisibleItems();
    }

  },

  created() {     
    if (this.$route.params.category) {
      this.checkedList = [this.$route.params.category];
    }
    this.canRecycle = eval(this.$route.params.id);    
    this.fetchItems();    
  },  

  beforeUpdate() {
    this.updateVisibleItems();
  },

  components: {
    'item-card': ItemCard,
    'pagination': Pagination
  }
}

</script>

<style scoped>

#itemsList{
    width: 100%;
    max-width: 1000px;
    margin: 30px auto;
    padding: 0 5px;
    box-sizing: border-box;
}

ul{
    display: flex;
    flex-wrap: wrap;
    list-style-type: none;
    padding: 0;    
}

li{  
    flex-grow: 1;
    flex-basis: 300px;
    text-align: center;
    margin-left: 3%;
    margin-right: 3%;    
}

li:hover {
  border-color:lightgreen;
  border-style: ridge;
  border-width: unset;
}

</style>