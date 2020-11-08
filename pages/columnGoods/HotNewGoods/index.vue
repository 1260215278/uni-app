<template>
  <div class="quality-recommend">
    <div class="slider-banner swiper">
			<view class="swiper">
				<swiper indicator-dots="true" :autoplay="autoplay" :circular="circular" :interval="interval" :duration="duration"
				 indicator-color="rgba(255,255,255,0.6)" indicator-active-color="#fff">
					<block v-for="(item,index) in imgUrls" :key="index">
						<swiper-item>
							<image :src="item.pic" class="slide-image"></image>
						</swiper-item>
					</block>
				</swiper>
			</view>
    </div>
    <div class="title acea-row row-center-wrapper">
      <div class="line"></div>
      <div class="name">
        <span class="iconfont" :class="icon"></span>{{ name }}
      </div>
      <div class="line"></div>
    </div>
		<view class="wrapper">
			<GoodList  :bastList="goodsList" :is-sort="false"></GoodList>
			<view class="txt-bar" v-if="goodsList.length>0 && !isScroll">我是有底线的~</view>
			<emptyPage v-if="goodsList.length==0 && !isScroll" title="暂无数据~"></emptyPage>
		</view>
  </div>
</template>
<script>
import emptyPage from '@/components/emptyPage.vue'
import GoodList from "@/components/goodList";
import { getIndexData } from '@/api/api.js';
import { getGroomList,getHotBanner } from "@/api/store";
import util from "@/utils/util";

const typeData = {
	best:{
		icon:'icon-jingpintuijian',
		name:'精品推荐'
	},
	hot:{
		icon:'icon-remen',
		name:'热门榜单'
	},
	new:{
		icon:'icon-xinpin',
		name:'首页新品'
	},
	good:{
		icon:'icon-xinpin',
		name:'推荐单品'
	},
}

export default {
  name: "HotNewGoods",
  components: {
    GoodList,
		emptyPage,
  },
  props: {},
  data: function() {
    return {
      imgUrls: [],
      goodsList: [],
      name: "",
	  hotData:[],
	  loaded:false,
      icon: "",
			type:0,
			autoplay:true,
			circular:true,
			interval: 3000,
			duration: 500,
			page:1,
			limit:8,
			isScroll:true
    };
  },
  onLoad: function(option) {
	this.type = option.type
    this.getIndexGroomList();
	this.getHotBanner();
	getIndexData().then(res=>{
		this.hotData = res.data.hot;
	}).finally(e=>{
		this.loaded = true;
		this.titleInfo();
	});
  },
  watch:{
	name(n){
		uni.setNavigationBarTitle({
			title:n||'加载中'
		})
	}  
  },
  methods: {
    titleInfo: function() {
      if(!this.loaded){
      	this.name = '';
      	this.icon = '';
      }else{
      	let name = (typeData[this.type]||{}).name || '精品推荐';
		let url = this.$route ? this.$route.fullPath : ('/' + util.getNowUrl());
      	this.hotData.forEach(data=>{
      		if(data.url == url) name = data.title;
      	})
      	this.name = name;
      	this.icon = (typeData[this.type]||{}).icon || 'icon-jingpintuijian';
      }
    },
    getIndexGroomList: function() {
			if(!this.isScroll) return
      let that = this;
      let type = this.type;
      getGroomList(type,{
				page:this.page,
				limit:this.limit
			}).then(res => {
         
          that.goodsList = that.goodsList.concat(res.data.list);
					that.isScroll = res.data.list.length>=that.limit
					that.page++
        })
        .catch(function(res) {
          that.$util.Tips({ title: res });
        });
    },
	getHotBanner(){
		let that = this
		getHotBanner(this.type).then(res=>{
			that.imgUrls = res.data;
		})
		}
  },
	onReachBottom() {
		this.getIndexGroomList()
	}
};
</script>
<style lang="scss">
	/deep/ .empty-box{
		background-color: #f5f5f5;
	} 
	.swiper,swiper,swiper-item,.slide-image{
		width: 100%;
		height: 280rpx;
	}
	.quality-recommend {
		.wrapper{
			background: #fff;
		}
		.title {
	    height: 120rpx;
	    font-size:32rpx;
	    color: #282828;
	    background-color: #f5f5f5;
			.line{
				width: 230rpx;
				height: 2rpx;
				background-color: #e9e9e9;
			}
		}	
	}
	.txt-bar{
		padding: 20rpx 0;
		text-align: center;
		font-size: 26rpx;
		color: #666;
		background-color: #f5f5f5;
	}
</style>