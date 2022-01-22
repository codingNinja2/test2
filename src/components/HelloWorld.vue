<template>

  <div>
<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <div class="container-fluid">
    <a class="navbar-brand" href="#">Navbar</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarSupportedContent">
      <ul class="navbar-nav me-auto mb-2 mb-lg-0">
        <li class="nav-item">
          <a class="nav-link active" aria-current="page" href="#">Home</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">Link</a>
        </li>
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
            Dropdown
          </a>
          <ul class="dropdown-menu" aria-labelledby="navbarDropdown">
            <li><a class="dropdown-item" href="#">Action</a></li>
            <li><a class="dropdown-item" href="#">Another action</a></li>
            <li><hr class="dropdown-divider"></li>
            <li><a class="dropdown-item" href="#">Something else here</a></li>
          </ul>
        </li>
        <li class="nav-item">
          <a class="nav-link disabled" href="#" tabindex="-1" aria-disabled="true">Disabled</a>
        </li>
      </ul>
      <form class="d-flex" @submit.prevent="submitSearch">
        <input class="form-control me-2" type="search" placeholder="Search" name="search" v-model="search" aria-label="Search">
        <button class="btn btn-outline-success" type="submit">Search</button>
      </form>
    </div>
  </div>
</nav>
  <div class="main">
  <div class="sidebar text-start p-5 pt-0">
    
    <div class="main-filter" v-for="(filter,index) in filters" :key="filter.index">
      <p>

        <a class="" data-bs-toggle="collapse" :href="`#collapse${index}`" v-if="index" role="button" aria-expanded="false" :aria-controls="'collapse' +index">
          {{filter.filter_lable}}
        </a>

         <a class="" data-bs-toggle="collapse" :href="`#collapse${index}`" v-else role="button" aria-expanded="true" :aria-controls="'collapse' +index">
          {{filter.filter_lable}}
        </a>
      </p>
      <div class="collapse" :id="'collapse' + index">
        <div >
           <div v-for="(filteroption, index) in filter.options" :key="index">
        <input type="checkbox" class="" @click="addFilters" :name="filteroption.code" :id="filteroption.value" :value="filteroption.value"  >
        <label class="" :for="filteroption.value">{{filteroption.value}}</label>
      </div>
        </div>
      </div>
      
    
    </div>
  </div>
  <div class="content">
    <div class="row" v-if="products.length">
      
      <div class="col-3" v-for="item in products" :key="item.id_product" >
          
        <div class="card" style="width:100%;margin-bottom:30px;height:450px" >
          <img :src="item.image" class="card-img-top" alt="">
          <div class="card-body">
            <h6 class="card-title">{{item.name}}</h6>
            <p class="card-text"><b>{{item.price}}</b></p>

            <p class="size start-0"><b>size-</b>{{item.size}}</p>
            
          </div>
      </div>  
      </div>  
    </div>
    <div class="row" v-else>
      <div class="col-12">
        <h3>No Product Found</h3>
      </div>
    </div>
  </div>
  </div>
  </div>
</template>


<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },

  data () {
    return {
      data: '',
      products: [],
      categories:[],
      page: 1,
      filters: [],
      search:'',
      appliedFilters: [],
      url : `https://pim.wforwoman.com/pim/pimresponse.php/?service=category&store=1&url_key=top-wear-kurtas&page=1&count=20&sort_by=&sort_dir=desc&filter=`
    }
  },

  mounted () {
    this.getShopData()
    this.getNextPageShopData()
  },

  methods : {
    getShopData : function () {
      const axios = require('axios');
      axios.get('https://pim.wforwoman.com/pim/pimresponse.php/?service=category&store=1&url_key=top-wear-kurtas&page=1&count=20&sort_by=&sort_dir=desc&filter=')
      .then( (response) => {
        // handle success
        if (response.status == 200) {
          this.data = response.data
          // console.log(this.data)
          this.products = this.data.result.products
          this.filters = this.data.result.filters
          // console.log( this.products)
        }
      })
      .catch(function (error) {
        // handle error
        console.log(error);
      });

    },

    getNextPageShopData : function () {
      const axios = require('axios');
      document.addEventListener('scroll', () => {
        let bottomOfWindow = document.documentElement.scrollTop + window.innerHeight === document.documentElement.offsetHeight+17;     
          if (bottomOfWindow) {          
            this.page++
                  
            axios.get(this.url).then( (response) => {
              if (response.status == 200) {
                this.products = this.products.concat(response.data.result.products)
              }
            })
            .catch(function (error) {
              // handle error
              console.log(error);
            });
          }

      });
    },
    
    addFilters: function (){
      
      let param = event.target.name+'-'+ event.target.value
      if (event.target.checked) {
        this.appliedFilters.push(param)
      } else {
        
        this.appliedFilters = this.appliedFilters.filter((filterItem) =>  {
            return filterItem !== param
        })
      }
      // console.log(this.appliedFilters);

      this.getFilteredProduct(this.appliedFilters)
    },

    getFilteredProduct: function (params){
      const axios = require('axios');
   
       this.url = `https://pim.wforwoman.com/pim/pimresponse.php/?service=category&store=1&url_key=top-wear-kurtas&page=${this.page}&count=20&sort_by=&sort_dir=desc&filter=${ params.join(',')}`
      
        axios.get(this.url).then( (response) => {
                
          this.products = [];
          if (response.status == 200) {
            
            if (response.data.response.error == "1") {
              // alert (response.data.response.error_message)
                console.log(response.data.response)
            } else {
              console.log(response.data)
              
              this.products = response.data.result.products
              console.log(this.products)
            }
          } else {
            // alert(response)
          }
        })
        .catch(function (error) {
          // handle error
          console.log(error);
        });
    },

    submitSearch: function () {
      alert('search url not found')
    }


  }
}



</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.main {
  display: grid;
  grid-template-columns: 0.3fr 0.7fr;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}

.main-filter {
  margin-bottom: 30px;
}

a {
  color: #42b983;
}
.size {
  font-size: 12px;
}

.card {
  height: 490px !important;
}
 .navbar {
  background-color: #fff;
  padding: .5rem 1rem;
  border-bottom: 1px solid #eee;
  margin-bottom: 60px;
}

.main-filter {
  border: 1px solid #f3f3f3;
  padding:15px;
}
</style>
