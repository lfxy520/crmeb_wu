<template>
	<view :class="bgTabVal==2?'fullsize noRepeat':bgTabVal==1?'repeat ysize':'noRepeat ysize'"
		:style="'background-color:'+bgColor+';background-image: url('+bgPic+');min-height:'+windowHeight+'px;'">
		<skeleton :show="showSkeleton" :isNodes="isNodes" ref="skeleton" loading="chiaroscuro" selector="skeleton" bgcolor="#FFF"></skeleton>
		<view v-if="!errorNetwork" :style="{visibility: showSkeleton ? 'hidden' : 'visible'}">
			<view class="page-index skeleton" :class="{ bgf: navIndex > 0 }">
				<block>
					<!-- #ifdef H5 -->
					<view v-for="(item, index) in styleConfig" :key="index">
						<block v-if="item.name != 'headerSerch' && item.name != 'tabNav'">
							<component
								:is="item.name"
								:index="index"
								:isFixed="isFixed"
								:dataConfig="item"
								@changeBarg="changeBarg"
								:tempArr="tempArr"
								:userInfo="userInfo"
								:tabTitle="navTop"
								:navIndex="navIndex"
								@changeTab="changeTab"
								@detail="goDetail"
								v-show="navIndex == 0"
							></component>
						</block>
						<block v-if="item.name == 'headerSerch'">
							<component
								:is="'headerSerch'"
								:isFixed="isFixed"
								:dataConfig="item"
								@changeBarg="changeBarg"
								:tempArr="tempArr"
								:userInfo="userInfo"
								:tabTitle="navTop"
								@changeTab="changeTab"
								@detail="goDetail"
							></component>
						</block>
						<block v-if="item.name == 'tabNav'">
							<component
								:is="'tabNav'"
								:isFixed="isFixed"
								:dataConfig="item"
								@changeBarg="changeBarg"
								:tempArr="tempArr"
								:userInfo="userInfo"
								:tabTitle="navTop"
								@changeTab="changeTab"
								@detail="goDetail"
							></component>
						</block>
					</view>
					<!-- #endif -->
					<!-- #ifdef MP || APP-PLUS -->
					<block v-for="(item, index) in styleConfig" :key="index" >
						<view v-show="navIndex == 0">
							<activeParty v-if="item.name == 'activeParty'" :dataConfig="item"></activeParty>
							<bargain v-if="item.name == 'bargain'" :dataConfig="item" @changeBarg="changeBarg"></bargain>
							<blankPage v-if="item.name == 'blankPage'" :dataConfig="item"></blankPage>
							<combination v-if="item.name == 'combination'" :dataConfig="item"></combination>
							<topic v-if="item.name == 'topic'" :dataConfig="item"></topic>
							<coupon v-if="item.name == 'coupon'" :dataConfig="item"></coupon>
							<customerService v-if="item.name == 'customerService'" :dataConfig="item"></customerService>
							<goodList v-if="item.name == 'goodList'" :dataConfig="item" @detail="goDetail"></goodList>
							<guide v-if="item.name == 'guide'" :dataConfig="item"></guide>
							<liveBroadcast v-if="item.name == 'liveBroadcast'" :dataConfig="item"></liveBroadcast>
							<menus v-if="item.name == 'menus'" :dataConfig="item"></menus>
							<news v-if="item.name == 'news'" :dataConfig="item"></news>
							<presellList v-if="item.name == 'presellList'" :dataConfig="item"></presellList>
							<plantList v-if="item.name == 'plantList'" :dataConfig="item"></plantList>
							<promotionList v-if="item.name == 'promotionList'" :dataConfig="item" @changeTab="changeTab" :tempArr="tempArr" @detail="goDetail"></promotionList>
							<richText v-if="item.name == 'richText'" :dataConfig="item"></richText>
							<seckill v-if="item.name == 'seckill'" :dataConfig="item"></seckill>
							<swiperBg v-if="item.name == 'swiperBg'" :dataConfig="item"></swiperBg>
							<pictureCube v-if="item.name == 'pictureCube'" :dataConfig="item"></pictureCube>
							<titles v-if="item.name == 'titles'" :dataConfig="item"></titles>
							<shopList v-if="item.name == 'shopList' && hide_mer_status != 1" :dataConfig="item"></shopList>
						</view>
						<headerSerch v-if="item.name == 'headerSerch'" :dataConfig="item" :userInfo="userInfo"></headerSerch>
						<tabNav v-if="item.name == 'tabNav'" :dataConfig="item" class="tabNav" :tabTitle="navTop"
						@bindHeight="bindHeighta" @changeTab="changeTab" :isFixed="isFixed"></tabNav>
					</block>
					<!-- #endif -->
					<view class="main" v-show="navIndex == 0">
						<!-- 首页推荐 -->
						<view class="index-product-wrapper">
							<!-- 首发新品 -->
							<recommend :hostProduct="hostProduct" :indexP="true" :isLogin="isLogin"></recommend>
							<view class="loadingicon acea-row row-center-wrapper" v-if="hostProduct.length > 0">
								<text class="loading iconfont icon-jiazai" :hidden="hotLoading == false"></text>
								{{ hotTitle }}
							</view>
						</view>
					</view>
					<view v-show="navIndex == 0" class="loadingicon acea-row row-center-wrapper" v-if="tempArr.length && styleConfig[styleConfig.length - 1].name == 'promotionList'">
						<text class="loading iconfont icon-jiazai" :hidden="loading == false"></text>
						{{ loadTitle }}
					</view>
				</block>
				<!-- 分类页 -->
				<view class="productList" v-if="navIndex > 0" :style="'margin-top:' + prodeuctTop + 'px;'">
					<view class="sort acea-row" v-if="sortList.length>0"  :style="{ marginTop: sortMarTop + 'px' }">
						<navigator
							hover-class="none"
							v-if="index < 9"
							:url="'/pages/columnGoods/goods_list/index?id=' + item.store_category_id + '&title=' + item.cate_name"
							class="item"
							v-for="(item, index) in sortList"
							:key="item.store_category_id"
						>
							<view class="pictrue">
								<easy-loadimage :image-src="item.pic"></easy-loadimage>
							</view>
							<view class="text">{{ item.cate_name }}</view>
						</navigator>
						<view class="item" @click="bindMore" v-if="sortList.length >= 9">
							<view class="pictrues acea-row row-center-wrapper"><text class="iconfont icon-gengduo1"></text></view>
							<view class="text" style="margin-top: 22rpx;">更多</view>
						</view>
					</view>
					
					<block v-if="sortProduct.length > 0">
						<view class="list acea-row row-between-wrapper">
							<navigator @tap="goDetails(item)" class="item" v-for="(item, index) in sortProduct" :key="item.product_id">
								<view class="pictrue">
									<easy-loadimage :image-src="item.image"></easy-loadimage>
								</view>
								<view class="text">
									<view class="name line1">
										<text class="name_text line2">{{ item.store_name }}</text>
									</view>
									<view class="acea-row row-middle">
										<view class="money font-color-red">
											
											<text class="num">{{ item.price }}</text>
										</view>
									</view>
									<view class="item_tags acea-row">
										<text v-if="item.merchant.type_name && item.product_type == 0" class="font-bg-red">{{ item.merchant.type_name }}</text>
										<text v-else-if="item.merchant.is_trader && item.product_type == 0" class="font-bg-red">{{$t(`page.store.selfSupport`)}}</text>
										<text v-if="item.product_type != 0" :class="'font_bg-red type' + item.product_type">
											{{ item.product_type == 1 ? '秒杀' : item.product_type == 2 ? '预售' : item.product_type == 3 ? '助力' : item.product_type == 4 ? '拼团' : '' }}
										</text>
										<text class="tags_item ticket" v-if="item.issetCoupon">{{$t(`page.store.collar`)}}</text>
										<text class="tags_item delivery" v-if="item.delivery_free == 1">{{$t(`page.goodsDetail.shipping`)}}</text>
									</view>
								</view>
							</navigator>
							<view class="loadingicon acea-row row-center-wrapper" v-if="sortProduct.length > 0 || sortProductLoading">
								<text class="loading iconfont icon-jiazai" :hidden="loading == false"></text>
								{{ loadTitle }}
							</view>
						</view>
					</block>
					
					<block v-if="sortProduct.length == 0">
						<view class="noCommodity">
							<view class="pictrue" style="margin: 0 auto;"><image src="/static/images/noShopper.png"></image></view>
							<recommend :hostProduct="hostProduct"></recommend>
						</view>
					</block>
				</view>
				<!-- #ifdef H5 -->
				<a v-if="beian_sn" href="https://beian.miit.gov.cn/#/Integrated/index" class="copyRight">{{beian_sn}}</a>
				<!-- #endif -->
			</view>
		</view>
		<view v-else>
			<view class="error-network">
				<image src="/static/images/error-network.png"></image>
				<view class="title">网络连接断开</view>
				<view class="con">
					<view class="label">请检查情况：</view>
					<view class="item">· 在设置中是否已开启网络权限</view>
					<view class="item">· 当前是否处于弱网环境</view>
					<view class="item">· 版本是否过低，升级试试吧</view>
				</view>
				<view class="btn" @click="reconnect">重新连接</view>
			</view>
		</view>
		<!-- #ifdef APP-PLUS -->
		<view class="privacy-wrapper" v-if="privacyStatus">
			<view class="privacy-box">
				<view class="title">服务协议与隐私政策</view>
				<view class="content">
					请务必审慎阅读、充分理解“服务协议与 隐私政策”各条款，包括但不限于：为了 向你提供即时通讯、内容分享等服务，我 们需要收集你的设备信息、操作日志等个
					人信息。你可以在“设置”中查看、变更、 删除个人信息并管理你的授权。
					<br />
					你可以阅读
					<navigator url="/pages/users/privacy/index">《服务协议与隐私政策》</navigator>
					了解 详细信息。如你同意，请点击“我同意”开始接受我们的服务。
				</view>
				<view class="btn-box">
					<view class="btn-item" @click="confirmApp">我同意</view>
					<view class="btn" @click="closeModel">随便逛逛</view>
				</view>
			</view>
		</view>
		<!-- #endif -->
		<!-- 发送给朋友图片 -->
		<view class="share-box" v-if="isIntegral">
			<image src="/static/images/share-info.png" @click="closeShare"></image>
		</view>
		<!-- 优惠券弹窗 -->
		<view class="coupon_popups" v-if="showCoupon">
			<view class="bg"></view>
			<view class="con" :style="{ 'background-image': `url(${domain}/static/diy/couponWindow${keyColor}.png)` }">
				<scroll-view scroll-y="true" >
					<view v-for="(item, index) in couponArray" class="item">
						<view class="left">
							<view class="price"> <text>{{item.coupon_price}}</text></view>
							<view class="max_price" v-if="item.use_min_price > 0">满{{item.use_min_price}}可用</view>
							<view class="max_price" v-else>无门槛</view>
						</view>
						<view class="right">
							<view class="time line1" v-if="item.coupon_type">有效期{{item.use_start_time}} - {{item.use_end_time}}</view>
							<text class="coupon_type">{{couponTypeMsg[item.type] || '店铺券'}}</text>
							<navigator :url="'/pages/columnGoods/goods_coupon_list/index?coupon_id='+item.coupon_id" class='bnt1 b-color' hover-class="none"><text class="title titleSize">{{item.title}}</text>
							<view class="title titleColor">
								有效期:{{item.use_start_time}}-{{item.use_end_time}}
							</view>
							</navigator>
							<!-- <view>{{item.start_time}}-{{item.end_time}}</view> -->
						</view>
					</view>
				</scroll-view>
				<view class="text">优惠券已放入您的账户中，点击“优惠券”去使用</view>
				<view class='iconfont icon-guanbi3' @click="showCoupon = false"></view>
			</view>
		</view>
		<authorize @onLoadFun="onLoadFun" :isAuto="isAuto" :isShowAuth="isShowAuth" @authColse="authColse" :isGoIndex="false"></authorize>
		<!-- #ifndef H5 -->
		<passwordPopup></passwordPopup>
		<!-- #endif -->
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
var statusBarHeight = uni.getSystemInfoSync().statusBarHeight + 'px';
let app = getApp();
import { getUserInfo } from '@/api/user.js';
import { getDiy, getIndexData, getAppVersion ,getUserCoupons} from '@/api/api.js';
// #ifdef MP-WEIXIN
import { getTemlIds } from '@/api/api.js';
import { SUBSCRIBE_MESSAGE, TIPS_KEY } from '@/config/cache';
// #endif
import { getShare, getconfig } from '@/api/public.js';
import { goShopDetail } from '@/libs/order.js';
import { mapGetters } from 'vuex';
import { getProductslist, getProductHot, storeCategory } from '@/api/store.js';
import { initiateAssistApi } from '@/api/activity.js';
import { openBargainSubscribe } from '@/utils/SubscribeMessage.js';
import { setVisit, spread } from '@/api/user.js';
import recommend from '@/components/recommend';
// #ifndef H5
import passwordPopup from '@/components/passwordPopup';
// #endif
// #ifdef H5
import mConfig from './component/index.js';
// #endif
import authorize from '@/components/Authorize';
// #ifdef MP || APP-PLUS

import activeParty from './component/activeParty';
import headerSerch from './component/headerSerch';
import coupon from './component/coupon';
import bargain from './component/bargain';
import blankPage from './component/blankPage';
import combination from './component/combination';
import customerService from './component/customerService';
import goodList from './component/goodList';
import guide from './component/guide';
import liveBroadcast from './component/liveBroadcast';
import menus from './component/menus';
import news from './component/news';
import topic from './component/topic';
import presellList from './component/presellList';
import plantList from './component/plantList';
import pictureCube from './component/pictureCube'
import shopList from './component/shopList';
import richText from './component/richText';
import seckill from './component/seckill';
import swiperBg from './component/swiperBg';
import tabNav from './component/tabNav';
import titles from './component/titles';
// #endif
import { silenceBindingSpread, configMap, isWeixin } from '@/utils/index';
import history from '@/mixins/history';
import shareScence from '@/libs/spread';
import easyLoadimage from '@/components/easy-loadimage/easy-loadimage.vue';
import {getNewPeopleCouponLst} from '@/api/activity.js'
import { HTTP_REQUEST_URL } from '@/config/app';
export default {
	computed: configMap({hide_mer_status:0,beian_sn:'',share_title:'',site_name:''},
	mapGetters(['isLogin','uid','keyColor'])),
	mixins: [history],
	components: {
		easyLoadimage,
		recommend,
		authorize,
		// #ifndef H5
		passwordPopup,
		// #endif
		// #ifdef H5
		...mConfig,
		// #endif
		// #ifdef MP || APP-PLUS
		activeParty,
		headerSerch,
		bargain,
		coupon,
		blankPage,
		combination,
		customerService,
		goodList,
		guide,
		liveBroadcast,
		menus,
		news,
		topic,
		presellList,
		plantList,
		pictureCube,
		shopList,
		richText,
		seckill,
		swiperBg,
		tabNav,
		titles
		// #endif
	},
	data() {
		return {
			domain: HTTP_REQUEST_URL,
			couponTypeMsg:{
				10: '通用券',
				11: '品类券',
				12: '跨店券'
			},
			styleConfig: [],
			showSkeleton: true, //骨架屏显示隐藏
			isNodes: 0, //控制什么时候开始抓取元素节点,只要数值改变就重新抓取
			privacyStatus: false,
			errorNetwork: false,
			ad: {home_ad_pic: ''},
			userInfo: {},
			loading: false,
			isAuto: false, //没有授权的不会自动授权
			isShowAuth: false, //是否隐藏授权
			statusBarHeight: statusBarHeight,
			navIndex: 0,
			navTop: [],
			subscribe: false,
			followUrl: '',
			followHid: true,
			followCode: false,
			imgUrls: [{url:'',pic:'',id: '',name:''}],
			hot: [],
			sortList: [],
			menus: [{url:'',pic:'',id: '',name:''},{url:'',pic:'',id: '',name:''},{url:'',pic:'',id: '',name:''},{url:'',pic:'',id: '',name:''},{url:'',pic:'',id: '',name:''},{url:'',pic:'',id: '',name:''},{url:'',pic:'',id: '',name:''},{url:'',pic:'',id: '',name:''},{url:'',pic:'',id: '',name:''},{url:'',pic:'',id: '',name:''}],
			window: false,
			navH: '',
			couponList: [],
			marTop: 0,
			loadend: false,
			loading: false,
			loadTitle: this.$t(`page.goodsList.more`),
			sortProduct: [],
			where: {
				pid: 0,
				page: 1,
				limit: 6
			},
			is_switch: true,
			hostProduct: [],
			hotPage: 1,
			hotLimit: 30,
			hotScroll: true,
			hotLoading: false,
			hotTitle: this.$t(`page.goodsList.more`),
			explosiveMoney: [],
			prodeuctTop: 0,
			pinkInfo: '',
			searchH: 0,
			domOffsetTop: 50,
			// #ifdef APP-PLUS || MP
			isFixed: true,
			// #endif
			// #ifdef H5
			isFixed: false,
			// #endif
			goodScroll: true, //精品推荐开关
			params: {
				//精品推荐分页
				page: 1,
				limit: 10
			},
			tempArr: [], //精品推荐临时数组
			numConfig: 0,
			pageInfo: '', // 精品推荐全数据
			swiperCur: 0,
			d: '',
			h: '',
			m: '',
			s: '',
			sum_h: '',
			sortMarTop: 0,
			globalDatas: {},
			// #ifdef MP || APP-PLUS
			mpHeight: 108,
			// #endif
			// #ifndef MP || APP-PLUS
			mpHeight: 0,
			// #endif
			currSpid: '',
			_options:{},
			isIntegral: false,
			appUpdate: {},
			sortMpTop: 0,
			bgColor: '',
			bgPic: '',
			bgTabVal: '',
			windowHeight: 0,
			isHeaderSerch: false,
			showCoupon: false,
			couponArray: []
		};
	},
	/**
	 * 用户点击右上角分享
	 */
	// #ifdef MP
	onShareAppMessage: function() {
		let that = this;
		wx.showShareMenu({
			withShareTicket: true,
			menus: ['shareAppMessage', 'shareTimeline']
		});
		return {
			title: that.share_title,
			path: '/pages/index/index?spid=' + that.uid
		};
	},
	onShareTimeline: function() {
		let that = this;
		return {
			title: that.share_title,
			query: {
				spid: that.uid
			},
			imageUrl: ''
		};
	},
	// #endif
	onLoad(options) {
		let that = this
		this._options = options;
		// #ifdef APP-PLUS
		const pages = getCurrentPages();
		const page = pages[pages.length - 1];
		const currentWebview = page.$getAppWebview();
		currentWebview.setStyle({
		  pullToRefresh: {
		    support: !that.isSupport,
		    style: plus.os.name === 'Android' ? 'circle' : 'default'
		  }
		});
		that.isSupport = !that.isSupport;
		that.appVersionConfig(); //APP版本检测
		// #endif
		that.$nextTick(function() {
			uni.getSystemInfo({
				success: function(res) {
					that.windowHeight = res.windowHeight;
				}
			});
		})
		that.pageLoad(options);
		setTimeout(() => {
			that.isNodes++;
		}, 100);
	},
	onShow() {
		this.getConfig();
		this.isIntegral = uni.getStorageSync('isIntegral')
		// #ifdef APP-PLUS
		if(this.appUpdate.openUpgrade == '1'){
			this.alertAppUpdate();
		}
		// #endif
		this.loadCoupon();
		this.loadTabBar();
	},
	onHide() {
		uni.setStorageSync('isIntegral',false)
	},
	onReady() {

	},
	watch: {
		globalDatas(nVal, oVal) {
			// #ifdef H5
			this.ShareInfo(nVal);
			// #endif
		},
		privacyStatus(n) {
			if (n) {
				uni.hideTabBar();
			} else {
				uni.showTabBar();
			}
		},
	},
	onPullDownRefresh(){
		setTimeout(()=> {
			uni.stopPullDownRefresh();
		}, 100);
		this.reconnect();
	},
	
	methods: {
		loadTabBar()
		{
			uni.setNavigationBarTitle({
				title: this.$t('page.store.index')
			});
			uni.setTabBarStyle({
				color: '#282828',
				selectedColor: '#E93323',
				backgroundColor: '#ffffff',
				borderStyle: 'white',
			});
			uni.setTabBarItem({
				index: 0,
				pagePath: "pages/index/index",
				iconPath: "static/images/1-001.png",
				selectedIconPath: "static/images/1-002.png",
				text: this.$t('userDrawer.data[0].name'),
			});
			uni.setTabBarItem({
				index: 1,
				pagePath: "pages/goods_cate/goods_cate",
				iconPath: "static/images/2-001.png",
				selectedIconPath: "static/images/2-002.png",
				text: this.$t('userDrawer.data[2].name'),
			});
			uni.setTabBarItem({
				index: 2,
				pagePath: "pages/order_addcart/order_addcart",
				iconPath: "static/images/3-001.png",
				selectedIconPath: "static/images/3-002.png",
				text: this.$t('userDrawer.data[3].name'),
			});
			uni.setTabBarItem({
				index: 3,
				pagePath: "pages/user/index",
				iconPath: "static/images/4-001.png",
				selectedIconPath: "static/images/4-002.png",
				text: this.$t('userDrawer.data[5].name'),
			});
		},
		changeBarg(){
			
		},
		loadCoupon(){
			if(!this.showCoupon && !uni.getStorageSync('show_coupon') && uni.getStorageSync('is_new_user')){
				getNewPeopleCouponLst().then(res => {
					this.couponArray = res.data;
					setTimeout(() => {
						this.showCoupon = this.couponArray.length > 0;
						uni.setStorageSync('show_coupon', '1')
					}, 1500);
				})
			}
		},
		
		pageLoad(options){
			let that = this;
			if (options.spid) {
				that.currSpid = options.spid;
				app.globalData.spid = options.spid;
			}
			// #ifdef APP-PLUS
			try {
				let val = uni.getStorageSync('privacyStatus') || false;
				if (!val) {
					this.privacyStatus = true;
				}
			} catch (e) {}
			this.snycNetWork();
			// #endif
			// #ifdef MP
			if (options.scene) {
				let value = that.$util.getUrlParams(decodeURIComponent(options.scene));
				if (value.id) options.id = value.id;
				//记录推广人uid
				if (value.spid) {
					that.currSpid = value.spid;
					app.globalData.spid = value.spid;
				}
			}
			// #endif
			shareScence(that.currSpid, that.isLogin);
			this.isLogin && silenceBindingSpread();
			this.getIndexConfig()
			Promise.all([
				this.diyData(),
				this.get_host_product(),
			]);
			if (this.isLogin) {
				this.getUserInfo();
				// #ifdef MP
				this.getHistory();
				// #endif
			}
		},
		closeShare(){
			uni.setStorageSync('isIntegral',false);
			this.isIntegral = uni.getStorageSync('isIntegral');
		},
		bindHeighta(data) {
			// #ifdef APP-PLUS
			this.sortMpTop = data.top + data.height;
			// #endif
		},
		// 重新链接
		reconnect() {
			uni.getNetworkType({
				success: res => {
					this.errorNetwork = res.networkType === 'none';
					if(!this.errorNetwork){
						this.pageLoad(this._options);
					}
				}
			});
		},

		// #ifdef APP-PLUS
		snycNetWork() {
			uni.getNetworkType({
				success: res => {
					this.errorNetwork = res.networkType === 'none';
				}
			});
		},

		// 同意隐私协议
		confirmApp() {
			uni.setStorageSync('privacyStatus', true);
			this.privacyStatus = false;
		},
		// 关闭Model
		closeModel() {
			this.privacyStatus = false;
		},
		// #endif
		// 对象转数组
		objToArr(data) {
			let obj = Object.keys(data).sort();
			let m = obj.map(key => data[key]);
			return m;
		},
		diyData() {
			let that = this;
			getDiy().then(res => {
				setTimeout(() => {
					that.isNodes++;
				}, 0);
				that.errorNetwork = false
				uni.setNavigationBarTitle({
					title: res.data.title
				});
				let data = res.data;
				if (data.is_bg_color) {
					that.bgColor = data.color_picker
				}
				if (data.is_bg_pic) {
					that.bgPic = data.bg_pic
					that.bgTabVal = data.bg_tab_val
				}
				that.styleConfig = that.objToArr(res.data.value);
				that.styleConfig.forEach((item, index, arr) => {
					if (item.name == 'headerSerch') {
						that.isHeaderSerch = true
					}
				});

			});
		},
		// 微信登录回调
		onLoadFun: function(e) {
			this.isShowAuth = false;
		},
		// #ifdef APP-PLUS
		appVersionConfig() {
			var that = this;
			//app升级
			// 获取本地应用资源版本号
			getAppVersion().then(res => {
				that.$set(that.appUpdate, 'androidAddress', res.data.androidAddress);
				that.$set(that.appUpdate, 'appVersion', res.data.appVersion);
				that.$set(that.appUpdate, 'iosAddress', res.data.iosAddress);
				that.$set(that.appUpdate, 'openUpgrade', res.data.openUpgrade);
				plus.runtime.getProperty(plus.runtime.appid, function(inf) {
					let nowVersion = (inf.version).split('.').join('');
					let appVersion = (res.data.appVersion).split('.').join('');
					that.$set(that.appUpdate, 'alert', appVersion > nowVersion);
					that.alertAppUpdate();
				});
			})
		},
		alertAppUpdate(){
			if(this.appUpdate.alert) {
				uni.getSystemInfo({
					success: (res) => {
						uni.showModal({
							title: this.$t('page.index.updateTip'),
							content: this.$t('page.index.updateContent'),
							confirmText: this.$t('page.goodsDetail.confirm'),
							cancelText: this.$t('page.goodsDetail.cancel'),
							showCancel: this.appUpdate.openUpgrade !=
								'1',
							cancelColor: '#eeeeee',
							confirmColor: '#FF0000',
							success:(response)=> {
								if (response.confirm) {
									switch (res.platform) {
										case "android":
											plus.runtime.openURL(this
												.appUpdate
												.androidAddress);
											break;
										case "ios":
											plus.runtime.openURL(encodeURI(
												this.appUpdate
												.iosAddress));
											break;
									}

								}
							}
						});
					}
				})
			}
		},
		// #endif
		getConfig() {
			// 获取配置
			getconfig()
				.then(res => {
					let self = this;
					this.globalDatas = res.data;
					uni.$emit('configData', res.data);
				})
				.catch(err => {});
		},
		// 分类页更多
		bindMore() {
			console.log(this.navTop[this.navIndex]);
			try {
				uni.setStorageSync('storeIndex', this.navTop[this.navIndex].pid);
				uni.switchTab({
					url: '/pages/goods_cate/goods_cate'
				});
			} catch (e) {}
		},
		goDetails(item) {
			goShopDetail(item, this.uid).then(res => {
				if (this.isLogin) {
					initiateAssistApi(item.activity_id)
						.then(res => {
							let id = res.data.product_assist_set_id;
							uni.hideLoading();
							// #ifndef MP
							uni.navigateTo({
								url: '/pages/activity/assist_detail/index?id=' + id
							});
							// #endif
							// #ifdef MP
							openBargainSubscribe()
								.then(res => {
									uni.hideLoading();
									uni.navigateTo({
										url: '/pages/activity/assist_detail/index?id=' + id
									});
								})
								.catch(err => {
									uni.hideLoading();
								});
							// #endif
						})
						.catch(err => {
							uni.showToast({
								title: err,
								icon: 'none'
							});
						});
				} else {
                    this.isAuto = true;
                    this.isShowAuth = true
				}
			});
		},
		/**
		 * 获取个人用户信息
		 */
		getUserInfo: function() {
			let that = this;
			getUserInfo().then(res => {
				that.userInfo = res.data;
			});
		},
		// 记录会员访问
		setVisit() {
			setVisit({
				url: '/pages/index/index'
			}).then(res => {
				console.log(res);
			});
		},
		// 导航分类切换
		changeTab(e) {
			let self = this;
			if (this.navIndex == e.index) return;
			this.navIndex = e.index;
			if (e.index > 0) {
				storeCategory({
					pid: e.pid
				}).then(res => {
					this.sortList = res.data.length > 9 ? res.data.splice(0, 9) : res.data;
					if (this.sortList.length > 0) {
						this.where.pid = e.pid;
						this.where.page = 1;
						this.loadend = false;
						this.loading = false;
						this.sortProduct = [];
						this.get_product_list();
					}
				});
				self.sortMarTop = 10;
			}
		},
		//分类产品
		get_product_list: function() {
			let that = this;
			if (that.loading) return;
			that.loading = true;
			that.loadTitle = '';
			getProductslist(that.where)
				.then(res => {
					let list = res.data.list;
					let productList = that.$util.SplitArray(list, that.sortProduct);
					let loadend = list.length < that.where.limit;
					that.loadend = loadend;
					that.loading = false;
					that.loadTitle = loadend ? this.$t(`page.goodsList.no`) :  this.$t(`page.goodsList.more`);
					that.$set(that, 'sortProduct', productList);
					that.$set(that.where, 'page', that.where.page + 1);
				})
				.catch(err => {
					that.loading = false;
					that.loadTitle =  this.$t(`page.goodsList.more`);
				});
		},
		/**
		 * 获取我的推荐
		 */
		get_host_product: function() {
			let that = this;
			let num = that.hotLimit;
			if (!that.hotScroll) return;
			if (that.hotLoading) return;
			that.hotLoading = true;
			that.hotTitle = '';
			getProductHot(that.hotPage, that.hotLimit).then(res => {
				let list = res.data.list;
				let productList = that.$util.SplitArray(list, that.hostProduct);
				let hotScroll = list.length <= num && list.length != 0;
				that.hotScroll = hotScroll;
				that.hotLoading = false;
				that.hotTitle = !hotScroll ? this.$t(`page.goodsList.no`) :  this.$t(`page.goodsList.more`);
				that.$set(that, 'hostProduct', productList);
				that.$set(that, 'hotPage', that.hotPage + 1);
			});
		},
		// 首页数据
		getIndexConfig: function() {
			let that = this;
			getIndexData().then(res => {
				that.$set(that, 'imgUrls', res.data.banner);
				that.$set(that, 'menus', res.data.menu);
				that.$set(that, 'hot', res.data.hot);
				that.$set(that, 'ad', res.data.ad);
				res.data.category.unshift({
					cate_name: that.$t('page.store.index')
				});
				that.$set(that, 'navTop', res.data.category);
				// #ifdef H5
				that.subscribe = res.data.subscribe;
				// #endif
				// 小程序判断用户是否授权；
				// #ifdef MP
				uni.getSetting({
					success(res) {
						if (!res.authSetting['scope.userInfo']) {
							that.window = that.couponList.length ? true : false;
						} else {
							that.window = false;
						}
					}
				});
				// #endif
				// #ifndef MP
				if (that.isLogin) {
					that.window = false;
				} else {
					that.window = that.couponList.length ? true : false;
				}
				// #endif
				that.reloadData();
			});
		},
		reloadData() {
			setTimeout(() => {
				this.showSkeleton = false
			}, 500)
		},
		// 授权关闭
		authColse: function(e) {
			this.isShowAuth = e;
		},
		// 首发新品详情
		goDetail(item) {
			if (item.activity && item.activity.type === '2' && !this.isLogin) {
				// #ifdef H5
				uni.showModal({
					title: this.$t('page.index.tip'),
					content: this.$t('page.index.pleaseLogin'),
					confirmText: this.$t('page.goodsDetail.confirm'),
					cancelText: this.$t('page.goodsDetail.cancel'),
					success: function(res) {
						if (res.confirm) {
							uni.navigateTo({
								url: '/pages/users/login/index'
							});
						} else if (res.cancel) {
						}
					}
				});
				// #endif
				// #ifdef MP
				this.$set(this, 'isAuto', true);
				this.$set(this, 'isShowAuth', true);
				// #endif
				return;
			} else {
				goShopDetail(item, this.uid).then(res => {
					uni.navigateTo({
						url: `/pages/goods_details/index?id=${item.id}`
					});
				});
			}
		},
		// 分类详情
		godDetail(item) {
			uni.navigateTo({
				url: `/pages/goods_details/index?id=${item.id}`
			});
		},
		//拼团详情
		goCombinDetail(item) {
			uni.navigateTo({
				url: `/pages/activity/combination_details/index?id=${item.product_group_id}`
			});
		},
		countTime: function() {
			// 获取当前时间
			var date = new Date();
			var now = date.getTime();
			//设置截止时间
			var endDate = new Date('2020-10-22 23:23:23');
			var end = endDate.getTime();
			//时间差
			var leftTime = end - now;
			//定义变量 d,h,m,s保存倒计时的时间
			if (leftTime >= 0) {
				this.d = Math.floor(leftTime / 1000 / 60 / 60 / 24);
				this.h = Math.floor((leftTime / 1000 / 60 / 60) % 24);
				this.m = Math.floor((leftTime / 1000 / 60) % 60);
				this.s = Math.floor((leftTime / 1000) % 60);
				this.sum_h = this.d * 24 + this.h;
			}
			//递归每秒调用countTime方法，显示动态时间效果
			setTimeout(this.countTime, 1000);
		},
		//#ifdef H5
		ShareInfo(datas) {
			let data = this.storeInfo;
			let href = location.href;
			if (this.$wechat.isWeixin()) {
				if (this.isLogin) {
					href = href.indexOf('?') === -1 ? href + '?spid=' + this.uid : href + '&spid=' + this.uid;
				} else {
					href = href;
				}
				let configAppMessage = {
					desc: datas.share_info,
					title: datas.share_title,
					link: href,
					imgUrl: datas.share_pic
				};
				this.$wechat
					.wechatEvevt(['updateAppMessageShareData', 'updateTimelineShareData', 'onMenuShareAppMessage', 'onMenuShareTimeline'], configAppMessage)
					.then(res => {
						console.log(res, '=============================>>WXAPI');
					})
					.catch(err => {
						console.log(err);
					});
			}
		},
		//#endif
	},
	mounted() {

	},
	// 滚动到底部
	onReachBottom() {
		if (this.navIndex == 0) {
			// 首页加载更多
			this.get_host_product();
		} else {
			// 分类栏目加载更多
			if (this.sortProduct.length > 0) {
				this.get_product_list();
			} else {
				this.get_host_product();
			}
		}
	},
	// 滚动监听
	onPageScroll(e) {
		// #ifdef H5
		if (this.isHeaderSerch) {
			if (e.scrollTop > this.domOffsetTop) {
				this.isFixed = true;
			}
			if (e.scrollTop < this.domOffsetTop) {
				this.$nextTick(() => {
					this.isFixed = false;
				});
			}
		} else {
			this.isFixed = false
		}
		// #endif
		// 传入scrollTop值并触发所有easy-loadimage组件下的滚动监听事件
		uni.$emit('scroll');
	}
};
</script>
<style>

</style>
<style lang="scss">
	
	.bnt1{
		color: red;
	}
page {
	display: flex;
	flex-direction: column;
	height: 100%;
}
.main {
	padding: 0 20rpx;
}
.colum0{
	white-space: nowrap; 
	display: flex;
}
.ysize {
	background-size: 100%;
}
.fullsize {
	background-size: 100% 100%;
}
.repeat {
	background-repeat: repeat;
}
.noRepeat {
	background-repeat: no-repeat;
}
.privacy-wrapper {
	z-index: 999;
	position: fixed;
	left: 0;
	top: 0;
	bottom: 0;
	right: 0;
	width: 100%;
	height: 100%;
	background: #7f7f7f;
	.privacy-box {
		position: absolute;
		left: 50%;
		top: 50%;
		transform: translate(-50%, -50%);
		width: 560rpx;
		padding: 50rpx 45rpx 0;
		background: #fff;
		border-radius: 20rpx;
		.title {
			text-align: center;
			font-size: 32rpx;
			text-align: center;
			color: #333;
			font-weight: 700;
		}
		.content {
			margin-top: 20rpx;
			line-height: 1.5;
			font-size: 26rpx;
			color: #666;
			navigator {
				display: inline-block;
				color: #e93323;
			}
		}
		.btn-box {
			margin-top: 40rpx;
			text-align: center;
			font-size: 30rpx;
			.btn-item {
				height: 82rpx;
				line-height: 82rpx;
				background: linear-gradient(90deg, #f67a38 0%, #f11b09 100%);
				color: #fff;
				border-radius: 41rpx;
			}
			.btn {
				padding: 30rpx 0;
			}
		}
	}
}
.coupon_popups{
	z-index: 999;
	position: fixed;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
	text-align: center;
	.bg{
		position: absolute;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0,0,0,.5);
	}
	.con{
		position: absolute;
		left: 50%;
		top: 50%;
		transform: translate(-50%,-50%);
		// width:750rpx;
		width: 730rpx;
		height:900rpx;
		// background-image: url('~@/static/images/couponWindow3.png');
		background-size: 100% 700rpx;
		background-repeat: no-repeat;
		scroll-view{
			width: 610rpx;
			// height: 440rpx;
			height: 306rpx;
			padding-top: 20rpx;
			margin: 300rpx auto 0;
		}
		.item{
			display: flex;
			align-items: center;
			width: 100%;
			height: 164rpx;
			background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABDgAAAFICAMAAACC1L7bAAAAilBMVEUAAAD////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////2N2iNAAAALXRSTlMA9xnrZyf0790ev4d1Egr7cl9TI+XNbVpAtKSUjX0NB6ucgnpDMdjRxUw7LdTK5bIiAAAF40lEQVR42uzcWW7CMBSF4UNImAIBEcpUKJQixt79b68qD1VLgNi8VHL+bxHH9tH1VUE+SeedVs1QkMmNAYGotTrzdNJWiSRtGe7J5aRnQFh2m1z3ndeGB05ycjYgOPNct2WLvuGRoZxMDAhPbbnSDQmPlDKpnCwMCNE+UsGxYSixl5MXA4JUP+lKbCgXyUHbgED1J/pjbHAwloONAcGKqfP8jVYqlTUNCNdQPxKDmw+V6hoQsEZOm+et0VOJHiUzwnbgjPQ3Uwlm6BC6WBcRZ6SHpa7QMqNimj19ezd42OqBLcO3CF/K2IG/5lR3Tbm8oQIuVd/S4CfmnYJqiyXtDJ5eM92QdAyohI6UG7zVB7qWpQZUxRufVJ4z6kb65dytG1AZR80NzzkMTkkkKY/XpAaqJWVqFICvmfiSBcDTp1hnDsDTiFX+AHzVCA4A3ggOAAQHgAKCA8D/Izi+2KmDAQAAAAZC/tYDGMB9CiJAHMARB9ATByAO4IgD6IkDEAdwxAH0xAGIAzjiAHriAMQBHHEAPXEA4gCOOICeOABxAEccQE8cgDiAIw6gJw5AHMARB9ATByAOxk4dDAAAADAQ8rcewADuUxDBEQfQEwcgDuCIA+iJAxAHcMQB9MQBiAM44gB64gDEARxxAD1xAOIAjjiAnjgAcQBHHEBPHIA4gCMOoCcOQBzAEQfQEwcgDuCIA+iJAxAHcMQB9MQBiAM44gB64gDEARxxAD1xAOIAjjiAnjgAcQBHHEBPHIA4gCMOoCcOQBzAEQfQEwcgDuCIA+iJAxAHcMQB9MQBjJ06GAAAAGAg5G89gAHcpyASB3DEAfTEAYgDOOIAeuIAxAEccQA9cQDiAI44gJ44AHEARxxATxyAOIAjDqAnDkAcwBEH0BMHIA7giAPoiQMQB3DEAfTEAYgDOOIAeuIAxAEccQA9cQDiAI44gJ44AHEARxxATxyAOIAjDqAnDkAcwBEH0BMHIA7giAPoiQMQB3DEAfTEAYgDOOIAeuIAxAEccQA9cTB26mAAAACAgZC/9QAOYJ+CCMQBhDiAP3EA4gBCHMCfOABxACEO4E8cgDiAEAfwJw5AHECIA/gTByAOIMQB/IkDEAcQ4gD+xAGIAwhxAH/iAMQBhDiAP3EA4gBCHMCfOABxACEO4E8cgDiAEAfwJw5AHECIA/gTByAOIMQB/IkDEAcQ4gD+xAGIAwhxAH/iAMQBhDiAP3EA4gBCHMCfOABxACEOYOzUwQAAAAADIX/rAQzgPgVRTxyAOIAjDqAnDkAcwBEH0BMHIA7giAPoiQMQB3DEAfTEAYgDOOIAeuIAxAEccQA9cQDiAI44gJ44AHEARxxATxyAOIAjDqAnDkAcwBEH0BMHIA7giAPoiQMQB3DEAfTEAYgDOOIAeuIAxAEccQA9cQDiAI44gJ44AHEARxxATxyAOIAjDqAnDkAcwBEH0BMHIA7giIOxU4cEAIBAEMBAIUny/esR4BTqzBZi0CcOQBxAEAfQJw5AHEAQB9AnDkAcQBAH0CcOQBxAEAfQJw5AHEAQB9AnDkAcQBAH0CcOQBxAEAfQt+4AfNnrDK+dO8tNGIaiMHwSoBAKlERAQ4FQhnSQeve/vaoPRVWayTza/7cI2/ecKwNwMtDEAMDJRFMDACdb7Qz3WZ3SRJqX2fJ1YEBI9no23GGyGemP8sLZgYBkig3uLqpaLDk6EIy5tDY42iaqsTgZEISppJnBSZSqwdKAEAwlZhVHg1KNcrZiEICHQhK9ipOoVIucRVz476wfcWToLVOroQGeG8wlkXI42akF7zeE4BbyvRv6WS3UoaCVhd+m+jUy9JPqhmoFQRrHukkNfUzU7bgywF+fXJPONmpFZATvDSttAD1it6hQMwY/+O8pVcWV3aVOW/XBFyfw1TjXP8mXod1MvewN8NE6Vo3jmU2wdrlaEDXDb9HhqHqjF0OLWL1cDfDOKVGz5PBmaCKJdBQhWl9idUiy/QczSy31UxjgjYfpYxar6hufchGebA48QgAAAABJRU5ErkJggg==');
			background-repeat: no-repeat;
			background-size: 100% 100%;
			margin-bottom: 20rpx;
			border-radius: 20rpx;
			position: relative;
			.left{
				width: 160rpx;
				text-align: center;

				&::after{
					content: "";
					display: block;
					height: 110rpx;
					border-right: 1rpx dashed #E6E6E6;
					position: absolute;
					left: 178rpx;
					top: 25rpx;
				}
				.price{
					color: #E93323;
					font-size: 26rpx;
					font-weight: bold;
					text{
						font-size: 46rpx;
					}
				}
				.max_price{
					color: #999999;
					font-size: 18rpx;
					margin-top: 8rpx;
				}
			}
			.right{
				width: 440rpx;
				padding: 0 20rpx;
				.title{
					color: #333333;
					font-size: 26rpx;
				}
				.time{
					color: #999999;
					font-size: 22rpx;
					margin-top: 20rpx;
				}
				.bnt1{
					.titleSize{
						font-weight: 700;
						font-size: 33rpx;
					}
					.titleColor{
						margin-top: 20rpx;
						color: #999999;
					}
				}
				.coupon_type{
					position: absolute;
					color: #fff;
					font-size: 15rpx;
					width: 30rpx;
					text-align: center;
					height: auto;
					background: #E93323;
					top: 0;
					right: 20rpx;
					line-height: 26rpx;
					padding: 5rpx 0 15rpx;
					&::after{
						content: "";
						width: 0;
						height: 0;
						border-left: 14rpx solid transparent;
						border-right: 14rpx solid transparent;
						border-bottom: 14rpx solid #fff;
						position: absolute;
						bottom: -4rpx;
						left: 2rpx;
					}

				}
			}
		}
		.text{
			width: 600rpx;
			margin: 30rpx auto 0;
			font-size: 20rpx;
			color: #ffffff;
		}
		.icon-guanbi3{
			color: #ffffff;
			font-size: 70rpx;
			position: absolute;
			bottom: -80rpx;
			left: 320rpx;
		}
	}
}
.copyRight{
	margin: 60rpx auto 20rpx;
	max-width: 690rpx;
	font-size: 20rpx;
	color: #707070;
	text-decoration: none;
}
.share-box {
	z-index: 1000;
	position: fixed;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
	image {
		width: 100%;
		height: 100%;
	}
}
.error-network {
	position: fixed;
	left: 0;
	top: 0;
	display: flex;
	flex-direction: column;
	align-items: center;
	width: 100%;
	height: 100%;
	padding-top: 40rpx;
	background: #fff;
	padding-top: 30%;

	image {
		width: 414rpx;
		height: 336rpx;
	}
	.title {
		position: relative;
		top: -40rpx;
		font-size: 32rpx;
		color: #666;
	}
	.con {
		font-size: 24rpx;
		color: #999;
		.label {
			margin-bottom: 20rpx;
		}
		.item {
			margin-bottom: 20rpx;
		}
	}
	.btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 508rpx;
		height: 86rpx;
		margin-top: 100rpx;
		border: 1px solid #d74432;
		color: #e93323;
		font-size: 30rpx;
		border-radius: 120rpx;
	}
}
.area-row {
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
	display: block;
	width: 100%;
	text-align: center;
}
.page-index {
	display: flex;
	flex-direction: column;
	min-height: 100%;
	.page_content {
		/* #ifdef MP || APP-PLUS */
		padding-top: 270rpx;
		/* #endif */
		.page_bg{
			background: linear-gradient(180deg, #fff 0%, #f5f5f5 100%);
		}
		.nav {
			padding: 0 0rpx 30rpx;
			flex-wrap: wrap;
			/* #ifdef MP */
			margin-top: 0;
			/* #endif */
			/* #ifdef H5 */
			margin-top: 0;

			/* #endif */
			.item {
				display: flex;
				flex-direction: column;
				align-items: center;
				justify-content: center;
				width: 20%;
				margin-top: 30rpx;
				image {
					width: 82rpx;
					height: 82rpx;
				}
			}
		}
		.index-product-wrapper {
			.nav-bd {
				display: flex;
				align-items: center;
				.item {
					display: flex;
					flex-direction: column;
					align-items: center;
					justify-content: center;
					width: 25%;
					.txt {
						font-size: 32rpx;
						color: #282828;
					}
					.label {
						display: flex;
						align-items: center;
						justify-content: center;
						width: 124rpx;
						height: 32rpx;
						margin-top: 5rpx;
						font-size: 24rpx;
						color: #999;
					}
					&.active {
						color: $theme-color;
						.label {
							background: linear-gradient(90deg, $bg-star 0%, $bg-end 100%);
							border-radius: 16rpx;
							color: #fff;
						}
					}
				}
			}
			.list-box {
				display: flex;
				flex-wrap: wrap;
				justify-content: space-between;
				.item {
					width: 345rpx;
					margin-bottom: 20rpx;
					background-color: #fff;
					border-radius: 10px;
					overflow: hidden;
					image {
						width: 100%;
						height: 345rpx;
					}
					.text-info {
						padding: 10rpx 20rpx 15rpx;
						.title {
							color: #222222;
						}
						.old-price {
							margin-top: 8rpx;
							font-size: 26rpx;
							color: #aaaaaa;
							text-decoration: line-through;
							text {
								margin-right: 2px;
								font-size: 20rpx;
							}
						}
						.price {
							display: flex;
							align-items: flex-end;
							color: $theme-color;
							font-size: 34rpx;
							font-weight: 800;
							text {
								padding-bottom: 4rpx;
								font-size: 24rpx;
								font-weight: normal;
							}
							.txt {
								display: flex;
								align-items: center;
								justify-content: center;
								width: 28rpx;
								height: 28rpx;
								margin-left: 15rpx;
								margin-bottom: 10rpx;
								border: 1px solid $theme-color;
								border-radius: 4rpx;
								font-size: 22rpx;
								font-weight: normal;
							}
						}
					}
				}
				&.on {
					display: flex;
				}
			}
		}
	}
}
.productList {
	background-color: #f1f1f1;
	min-height: 70vh;
	.sort {
		width: 710rpx;
		max-height: 380rpx;
		background: rgba(255, 255, 255, 1);
		border-radius: 16rpx;
		padding: 8rpx 0rpx 30rpx;
		flex-wrap: wrap;
		margin: 25rpx auto 0 auto;
		.item {
			width: 20%;
			margin-top: 30rpx;
			text-align: center;
			.pictrues {
				width: 90rpx;
				height: 90rpx;
				background: rgba(248, 248, 248, 1);
				border-radius: 50%;
				margin: 0 auto;
			}
			.easy-loadimage {
				width: 90rpx;
				height: 90rpx;
				display: inline-block;
			}
			.text {
				color: #272727;
				font-size: 24rpx;
				margin-top: 10rpx;
				overflow: hidden;
				white-space: nowrap;
				text-overflow: ellipsis;
			}
		}
	}
}
.productList .list {
	padding: 0 20rpx;
}
.productList .list.on {
	background-color: #fff;
	border-top: 1px solid #f6f6f6;
}
.productList .list .item {
	width: 345rpx;
	margin-top: 20rpx;
	background-color: #fff;
	border-radius: 10rpx;
	.name{
		display: flex;
		align-items: center;
	}
}
.productList .list .item.on {
	width: 100%;
	display: flex;
	border-bottom: 1rpx solid #f6f6f6;
	padding: 30rpx 0;
	margin: 0;
}
.productList .list .item .pictrue {
	position: relative;
	width: 100%;
	height: 345rpx;
}
.productList .list .item .pictrue.on {
	width: 180rpx;
	height: 180rpx;
}
.productList .list .item .pictrue image,
.productList .list .item .pictrue uni-image,
.productList .list .item .pictrue .easy-loadimage{
	width: 100%;
	height: 100%;
	border-radius: 10rpx 10rpx 0 0;
}
/deep/.productList .list .item .pictrue uni-image.origin-img{
	border-radius: 10rpx 10rpx 0 0;
}
.productList .list .item .pictrue image.on {
	border-radius: 6rpx;
}
.productList .list .item .text {
	padding: 14rpx 17rpx 26rpx 17rpx;
	font-size: 28rpx;
	color: #212121;
}
.productList .list .item .text.on {
	width: 508rpx;
	padding: 0 0 0 22rpx;
}
.productList .list .item .text .money {
	font-size: 26rpx;
	font-weight: bold;
	margin-top: 8rpx;
}
.productList .list .item .text .coupon {
	background: rgba(255, 248, 247, 1);
	border: 1px solid rgba(233, 51, 35, 1);
	border-radius: 4rpx;
	font-size: 20rpx;
	margin-left: 18rpx;
	padding: 1rpx 4rpx;
}
.productList .list .item .text .money.on {
	margin-top: 50rpx;
}
.productList .list .item .text .money .num {
	font-size: 34rpx;
}

.pictrue {
	position: relative;
}
.fixed {
	z-index: 100;
	position: fixed;
	left: 0;
	top: 0;
	background-color: #fff;
	box-shadow: 0 10rpx 20rpx -5rpx rgba(0, 0, 0, 0.06);
}

</style>
