<template>
    <div>
      <navHeader></navHeader>
      <bread><span slot="goodlist">GoodList</span></bread>
      <div class="accessory-result-page accessory-page">
        <div class="container">
          <div class="filter-nav">
            <span class="sortby">Sort by:</span>
            <a href="javascript:void(0)" class="default cur">Default</a>
            <a @click="sortFunc" href="javascript:void(0)" class="price">Price <svg class="icon icon-arrow-short"><use xlink:href="#icon-arrow-short"></use></svg></a>
            <a @click="showPop" href="javascript:void(0)" class="filterby stopPop">Filter by</a>
          </div>
          <div class="accessory-result">
            <!-- filter -->
            <div class="filter stopPop" v-bind:class="{'filterby-show':overLayFlag}" id="filter">
              <dl class="filter-price">
                <dt>Price:</dt>
                <dd><a @click="filterFunc('All','All','All')" v-bind:class="{'cur':(checkedIndex=='All')}" href="javascript:void(0)">All</a></dd>
                <dd v-for="(price,index) in filterPrice">
                  <a @click="filterFunc(index,price.priceStart,price.priceEnd)" v-bind:class="{'cur':checkedIndex==index}" href="javascript:void(0)">{{price.priceStart}} - {{price.priceEnd}}</a>
                </dd>
              </dl>
            </div>

            <!-- search result accessories list -->
            <div class="accessory-list-wrap">
              <div class="accessory-list col-4">
                <ul>
                  <li v-for='item in resData'>
                    <div class="pic">
                      <a href="#"><img v-lazy="'static/'+item.prodcutImg" alt=""></a>
                    </div>
                    <div class="main">
                      <div class="name">{{item.productName}}</div>
                      <div class="price">{{item.prodcutPrice}}</div>
                      <div class="btn-area">
                        <a @click="addCartFunc(item)" href="javascript:;" class="btn btn--m">加入购物车</a>
                      </div>
                    </div>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="md-overlay" v-show="overLayFlag" @click="overLayFlag=false"></div>
      <div v-show="scrollLoading" style="text-align:center;height: 30px;background-color: transparent;width: 100%;margin: 0 auto;" v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="10">
        <img src="../../static/loading-svg/loading-balls.svg">
      </div>
      <navFooter loginProps="login"></navFooter>
      <modal v-bind:modalCover="showModal" v-on:hideModal="hideModal">
        <div class="modal center" slot="content">{{modalText}}</div>
        <a class="btn btn--m" href="javascript:;" slot="cancel" @click="hideModal" v-show="addCartSuccess">Continue</a>
        <router-link slot="confirm" class="confirm btn btn--m btn--red" href="javascript:;" to="/cart" @click="goCart" v-show="addCartSuccess">Check Cart</router-link>
      </modal>
    </div>
</template>

<script>
import '../assets/css/product.css';
import '../assets/css/base.css';
import NavHeader from '@/components/NavHeader';
import NavFooter from '@/components/navFooter';
import Bread from '@/components/Bread';
import axios from 'axios';
import Modal from '@/components/Modal';


export default {
  name: 'GoodList',
  data () {
    return {
      resData: [],
      pgData: [],
      filterPrice: [
        {
          priceStart: 0,
          priceEnd: 100
        },
        {
          priceStart: 100,
          priceEnd: 500
        },
        {
          priceStart: 500,
          priceEnd: 3000
        },
        {
          priceStart: 3000,
          priceEnd: 5000
        }
      ],
      checkedIndex: 'All',
      overLayFlag: false,
      busy: false,
      sort: 1,
      pageSize: 3,
      pageNum: 1,
      lastPage: 10,
      scrollLoading: false,
      priceStart: 'All',
      priceEnd: 'All',
      login: false,
      showModal: false,
      modalText: '',
      addCartSuccess: false
    }
  },
  mounted: function(){
     const userId = this.getCookies('userId');
    if(userId){
        this.login = true; 
    }
    this.init();
  },
  components: {
    NavHeader,
    NavFooter,
    Bread,
    Modal
  },
  methods: {
    init(){
      const param = {
        sort: this.sort,
        pageSize: this.pageSize,
        pageNum: this.pageNum,
        priceStart: this.priceStart,
        priceEnd: this.priceEnd
      };
      axios.get('/goods/sort',{
        params: param
      })
        .then((res)=>{
          this.resData = res.data.result.list;
          this.lastPage = res.data.result.lastPage
        })
    },
    render(){
      const param = {
        sort: this.sort,
        pageSize: this.pageSize,
        pageNum: this.pageNum,
        priceStart: this.priceStart,
        priceEnd: this.priceEnd
      };

      axios.get('/goods/sort',{
        params: param
      })
        .then((res)=>{
          this.pgData = res.data.result.list;
          this.resData=this.resData.concat(this.pgData);
          this.lastPage = res.data.result.lastPage;
          this.scrollLoading = false;
        })
    },
    showPop(){
      this.overLayFlag=true;
    },
    loadMore(){
        this.busy = true;
        setTimeout(() => {
              if(this.pageNum<this.lastPage){
                this.pageNum++;
                this.scrollLoading = true;
                this.render();
              };
          this.busy = false;
        }, 1000);
      },
    sortFunc(){
      this.pageNum = 1;
      this.resData = [];
      this.sort = (this.sort === -1)?1:-1;
      this.render();
    },
    filterFunc(checkedIndex,priceStart,priceEnd){
      this.pageNum = 1;
      this.resData = [];
      this.checkedIndex=checkedIndex;
      this.priceStart = priceStart;
      this.priceEnd = priceEnd;
      this.render();
    },
    getCookies(cname){
      let name = cname + "=";
      const decodeCookie = decodeURIComponent(document.cookie);
      let ca = decodeCookie.split(';');
      for(let i=0;i<ca.length;i++){
        let c = ca[i];
        while (c.charAt(0) == ' '){
          c = c.substring(1);
        }
        if (c.indexOf(name) == 0){
          return c.substring(name.length, c.length);
        }
      }
      return "";
    },
    addCartFunc(item){
      const cookieUserId = this.getCookies('userId');
       this.showModal = true;
       this.addCartSuccess =  false;
    if(!cookieUserId){
       this.modalText = "please login";
    }else{
        axios.post('/goods/cart',{
          productid: item.productId
        })
        .then(res => {
          if(res.status === '1001'){
            this.modalText = "please login first";
          }else{
            this.modalText = "Added to cart";
            this.addCartSuccess = true;
          }
        })
        .catch(error =>{
          this.modalText = "response error";
        });
      }
    },
    hideModal(){
      this.showModal = false;
    },
    goCart(){
      window.location.href = "/cart";
    }
  }
}
</script>

