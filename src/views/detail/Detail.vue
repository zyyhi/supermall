<template>
  <div class="detail">
      <detail-nav-bar class="detail-bar" @tabClick="tabClick" ref="nav"/>
      <scroll 
        class="content" 
        ref="scroll"
        :probeType="3"
        @scroll="handleScrollContent"
      >
        <detail-swiper :topImages="topImages"/>
        <detail-base-info :baseInfo="baseInfo"/>
        <detail-shop-info :shopInfo="shopInfo"/>
        <detail-goods-info :goodsInfo = "goodsInfo" @goodsInfoImgLoad="goodsInfoImgLoad"/>
        <detail-goods-params :goodsParams="goodsParams" ref="goodsParams" />
        <detail-comment-info :commentInfo="commentInfo" ref="goodsComment"/>
        <detail-recommend :recommendList="recommendList" ref="goodsRecommend"/>
      </scroll>
      <detail-bottom-bar @addCart="addCart"/>
      <back-top @click.native="backTopClick" v-if="isShowBackTop" />
  </div>
</template>

<script>

import DetailNavBar from './childComps/DetailNavBar';
import DetailSwiper from './childComps/DetailSwiper';
import DetailBaseInfo from './childComps/DetailBaseInfo';
import DetailShopInfo from './childComps/DetailShopInfo';
import DetailGoodsInfo from './childComps/DetailGoodsInfo';
import DetailGoodsParams from './childComps/DetailGoodsParams';
import DetailCommentInfo from './childComps/DetailCommentInfo';
import DetailRecommend from './childComps/DetailRecommend';
import DetailBottomBar from './childComps/DetailBottomBar';

import Scroll from '@/components/common/scroll/Scroll';
import BackTop from '@/components/common/backTop/BackTop';

import { getGoodsDetail, getGoodsRecommend, Goods, Shop } from '@/network/detail.js';
import { debounce } from '@/common/utils';
// import {itemListenerMixin} from '@/common/mixin';
import {backTopMixin} from '@/common/mixin';
export default {
    name: 'Detail',
    data() {
        return {
            iid: '',
            topImages: [],
            baseInfo: {},
            shopInfo: {},
            goodsInfo:{},
            goodsParams:{},
            commentInfo:{},
            recommendList:[],
            imgItemListener: null, //???????????????????????????????????????listener
            themeTopYs:[0],    //????????????,???????????????????????????offsetTop
            currentIndex: 0, //???????????????????????????????????????
            // isShowBackTop: false,       
        }
    },
    components: {
        DetailNavBar,
        DetailSwiper,
        DetailBaseInfo,
        DetailShopInfo,
        DetailGoodsInfo,
        DetailGoodsParams,
        DetailCommentInfo,
        DetailRecommend,
        DetailBottomBar,
        Scroll,
        BackTop,
    },
    // mixins:[itemListenerMixin],
    mixins:[backTopMixin],
    created() {
        this.iid = this.$route.params.iid || this.$route.params.tradeItemId;
         // ??????????????????
        this.getGoodsDetail();

        // ????????????????????????
        getGoodsRecommend().then(res=> {
            this.recommendList = res.data.list;
        })
    },
    mounted() {
         const refresh = debounce(this.$refs.scroll.refresh, 50);
        this.imgItemListener = ()=>{
            refresh;
        }
        this.$bus.$on('itemImgLoad', this.imgItemListener);
    },
    destroyed() {
         // ?????????????????????????????????????????????????????????????????????????????????
        this.$bus.$off('itemImgLoad', this.imgItemListener);
    },
    watch:{
        // ????????????????????????????????????
        '$route'(to, from){
          if(to.fullPath.indexOf('detail')!==-1){
              this.iid = to.params.iid;
           this.getGoodsDetail();
          }
        }
    },
    methods: {
        // ??????????????????
        getGoodsDetail(){
            getGoodsDetail(this.iid).then(res =>{
                let data = res.result;
                // ?????????????????????
                this.topImages = data.itemInfo.topImages;
                // ???????????????????????????
                this.baseInfo = new Goods(data.itemInfo, data.columns, data.shopInfo.services);
                // ??????????????????
                this.shopInfo = new Shop(data.shopInfo);
                // ????????????????????????
                this.goodsInfo = data.detailInfo;
                // ????????????????????????
                this.goodsParams = data.itemParams
                //??????????????????
                this.commentInfo = data.rate;
            }).catch(err=>{
                console.log('Whoops,something bad happened~');
            });
        },
        //??????????????????????????????
        goodsInfoImgLoad(){
            // ???????????????????????????????????????????????????????????????offsetTop
            this.themeTopYs.push(this.$refs.goodsParams.$el.offsetTop,this.$refs.goodsComment.$el.offsetTop, this.$refs.goodsRecommend.$el.offsetTop);
            // ??????hack????????????????????????
           this.themeTopYs.push(Number.MAX_VALUE);
            this.$refs.scorll &&  this.$refs.scorll.refresh();
        },

        //?????????????????????
        tabClick(i) {
            this.$refs.scroll.scrollTo(0, -this.themeTopYs[i]);
        },
        //????????????????????????
        handleScrollContent(position){
            let positionY = -position.y;
            let length = this.themeTopYs.length;
            //hack?????????
            for(let i = 0; i<length; i++){
                if(this.currentIndex!==i && positionY>=this.themeTopYs[i] && positionY<this.themeTopYs[i+1]){
                    this.currentIndex = i;
                    this.$refs.nav.currentTitleIndex = i;

                }
            }

            // ????????????backTop??????
            this.isShowBackTop = Math.abs(position.y) > 1000;
        },
        //????????????
        // backTopClick(){
        //     this.$refs.scroll.scrollTo(0,0,500);
        // },

        //?????????????????????
        addCart(){ 

            const product = {};
            product.image = this.topImages[0];
            product.title = this.baseInfo.title;
            product.desc = this.baseInfo.desc;
            product.price = this.baseInfo.newPrice;
            product.iid = this.baseInfo.iid;
            product.realPrice = this.baseInfo.realPrice;

            this.$store.dispatch('addCart', product).then((res)=>{
                this.$toast.show(res, 2000);
            });
        }
    },
}
</script>

<style scoped>
.detail {
    height: 100vh;
    position: relative;
    background:#fff;
    position: relative;
    z-index: 9;
}
.detail-bar {
    position: relative;
    /* z-index: 9; */
}
.content {
    position: absolute;
    top: 44px;
    left: 0;
    right: 0;
    bottom: 58px;
    /* overflow: hidden; */
   /* height: 400px; */
   /* height: calc(100% - 44px - 58px);  */
   overflow: hidden;
}
</style>
