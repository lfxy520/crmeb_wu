<template>
	<view :style="viewColor">
		<view class='sign-record'>
		   <view class='list' v-for="(item,index) in signList" :key="index">
		      <view class='item'>
		         <view class='data'>{{item.month}}</view>
		         <view class='listn'>
		            <view class='itemn acea-row row-between-wrapper' v-for="(itemn,indexn) in item.list" :key="indexn">
		               <view>
		                  <view class='name line1'>{{itemn.title}}</view>
		                  <view>{{itemn.create_time}}</view>
		               </view>
		               <view class='num t-color'>+{{itemn.number}}</view>
		            </view>
		         </view>
		      </view>
		   </view>
		    <view class='loadingicon acea-row row-center-wrapper'>
		        <text class='loading iconfont icon-jiazai' :hidden='loading==false'></text>{{loadtitle}}
		     </view>
		</view>
		<authorize @onLoadFun="onLoadFun" :isAuto="isAuto" :isShowAuth="isShowAuth" @authColse="authColse"></authorize>
	</view>
</template>

<script>
	// +----------------------------------------------------------------------
	// | CRMEB [ CRMEB赋能开发者，助力企业发展 ]
	// +----------------------------------------------------------------------
	// | Copyright (c) 2016~2021 https://www.crmeb.com All rights reserved.
	// +----------------------------------------------------------------------
	// | Licensed CRMEB并不是自由软件，未经许可不能去掉CRMEB相关版权
	// +----------------------------------------------------------------------
	// | Author: CRMEB Team <admin@crmeb.com>
	// +----------------------------------------------------------------------
	import { getSignMonthList } from '@/api/user.js';
	 import { mapGetters } from "vuex";
	 import authorize from '@/components/Authorize';
	export default {
		components: {
			authorize
		},
		data() {
			return {
				loading:false,
				    loadend:false,
				    loadtitle:this.$t(`page.goodsList.more`),
				    page:1,
				    limit:8,
				    signList:[],
					isAuto: false, //没有授权的不会自动授权
					isShowAuth: false //是否隐藏授权
			};
		},
		computed: mapGetters(['isLogin','viewColor']),
		onLoad(){
			if(this.isLogin){
				this.getSignMoneList();
			}else{
				this.isAuto = true;
                this.isShowAuth = true
			}
		},
		onReachBottom: function () {
		    this.getSignMoneList();
		  },
		methods: {
			  /**
			   *
			   * 授权回调
			  */
			  onLoadFun:function(){
				this.isShowAuth = false;
			    this.getSignMoneList();
			  },
			  // 授权关闭
			  authColse:function(e){
			  	this.isShowAuth = e
			  },
			  /**
			     * 获取签到记录列表
			    */
			    getSignMoneList:function(){
			      let that=this;
			      if(that.loading) return;
			      if(that.loadend) return;
				  that.loading = true;
				  that.loadtitle = "";
			      getSignMonthList({ page: that.page, limit: that.limit }).then(res=>{
			        let list = res.data;
			        let loadend = list.length < that.limit;
			        that.signList = that.$util.SplitArray(list, that.signList);
					that.$set(that,'signList',that.signList);
					that.loadend = loadend;
					that.loading = false;
					that.loadtitle = loadend ? this.$t(`page.goodsList.nono`) : this.$t(`page.goodsList.more`)
			      }).catch(err=>{
					that.loading = false;
					that.loadtitle =  this.$t(`page.goodsList.more`);
			      });
			    },
		}
	}
</script>
<style>
	.t-color{color:var(--view-theme)!important;}
</style>
