<template>
	<view :style="viewColor">
		<view class='payment-status'>
			<!--失败时： 用icon-iconfontguanbi fail替换icon-duihao2 bg-color-->
			<view class='iconfont icon-duihao2 icon' v-if ="order_pay_info.paid !=0"></view>
			<view class='iconfont icon-iconfontguanbi icon' v-else></view>
			<!-- 失败时：订单支付失败 -->
			<view class='status'>{{order_pay_info.paid?$t(`page.orderPayStatus.success`):$t(`page.orderPayStatus.fail`)}}</view>
			<!-- <view class='status' v-else>订单创建成功</view> -->
			<view class='wrapper'>
				<view class='item acea-row row-between-wrapper'>
					<view>{{$t(`page.orderPayStatus.orderTime`)}}</view>
					<view class='itemCom'>{{order_pay_info.create_time}}</view>
				</view>
				<view class='item acea-row row-between-wrapper'>
					<view>{{$t(`page.orderPayStatus.payType`)}}</view>
					<view v-if="order_pay_info.pay_type == 0" class='itemCom'>{{$t(`page.orderPayStatus.balance`)}}</view>
					<view v-else class='itemCom'>{{$t(`page.orderPayStatus.online`)}}</view>
				</view>
				<view class='item acea-row row-between-wrapper'>
					<view>{{$t(`page.orderPayStatus.payPrice`)}}</view>
					<view class='itemCom'>{{order_pay_info.pay_price}}</view>
				</view>
				<view v-if="order_pay_info.give_integral > 0" class='item acea-row row-between-wrapper'>
					<view>赠送积分</view>
					<view class='itemCom t-color'>{{order_pay_info.give_integral}}个</view>
				</view>
				<!--失败时加上这个  -->
				<view class='item acea-row row-between-wrapper' v-if="order_pay_info.paid==0 && order_pay_info.pay_type != 'offline' && msg != 'success'">
					<view>{{$t(`page.orderPayStatus.failReason`)}}</view>
					<view class='itemCom'>{{order_pay_info.pay_type==0 ? $t(`message.orderConfirm.notenough`):msg}}</view>
				</view>
			</view>
			<!--失败时： 重新购买 -->
			<view @tap="goOrderDetails">
				<button formType="submit" class='returnBnt gColor' hover-class='none'>{{$t(`page.orderPayStatus.viewOrder`)}}</button>
			</view>
			<!-- <view @tap="goOrderDetails" v-if="order_pay_info.paid==0 && status==1">
				<button class='returnBnt bg-color' hover-class='none'>重新购买</button>
			</view> -->
			<!-- <view @tap="goOrderDetails" v-if="order_pay_info.paid==0 && status==2">
				<button class='returnBnt bg-color' hover-class='none'>重新支付</button>
			</view> -->
			<button @click="goPink(order_pay_info.activity_id)" class='returnBnt s-Color' formType="submit" hover-class='none' v-if="order_pay_info.activity_type == 4 && order_pay_info.paid">邀请好友参团</button>
			<button v-if="!order_pay_info.paid || order_pay_info.activity_type != 4" @click="goIndex" class='returnBnt s-Color' formType="submit" hover-class='none'>{{$t(`page.orderPayStatus.backHome`)}}</button>
			<view class="coupon-wrapper" v-if="couponList.length>0 && order_pay_info.paid">
				<view class="hd">
					<view class="line"></view>
					<view class="txt">赠送优惠券</view>
					<view class="line"></view>
				</view>
				<view class="coupon-box" :class="{on:isOpen}">
					<block v-for="(item,index) in couponList" :key="index">
						<view class="coupon-item flex">
							<view class="left-bg" :style="{ 'background-image': `url(${domain}/static/diy/couponBg${keyColor}.png)` }"><text></text>{{item.coupon_price}}</view>
							<view class="info">
								<view class="title">{{item.title}}</view>
								<view class="des">满{{item.use_min_price}}元可用</view>
								<block v-if="item.coupon_type == 1">
									<view class="des">有效期:{{ item.use_start_time |timeYMD }}-{{ item.use_end_time |timeYMD}}</view>
								</block>
								<block v-if="item.coupon_type == 0">
									<view class="des">领取后{{ item.coupon_time}}天内可用</view>
								</block>
							</view>
						</view>
					</block>
				</view>
				<view class="more" v-if="couponList.length>2" @click="bindMore">
					{{ text }}
					<text v-if="!isOpen" class="iconfont icon-xiangxia"></text>
					<text v-else class="iconfont icon-xiangshang"></text>
				</view>
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
	import {getPayOrder} from '@/api/order.js';
	import {openOrderSubscribe} from '@/utils/SubscribeMessage.js';
	import {mapGetters} from "vuex";
	import authorize from '@/components/Authorize';
	import { HTTP_REQUEST_URL } from '@/config/app';
	export default {
		components: {
			authorize,
		},
		filters: {
		  timeYMD: function (value) {
				if(value){
					var newDate=/\d{4}-\d{1,2}-\d{1,2}/g.exec(value)
					return newDate[0]
				}
		  }
		},
		data() {
			return {
				domain: HTTP_REQUEST_URL,
				orderId: '',
				order_type: '',
				order_pay_info: {},
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				status: 0,
				msg: '',
				couponList:[], //优惠券列表
				isOpen:false ,//展开
				moneyBg: '/static/images/couponBg',
				text: '展开更多'
			};
		},
		computed: mapGetters(['isLogin','viewColor','keyColor']),
		onLoad: function(options) {
			if (!options.order_id) return this.$util.Tips({
				title: '缺少参数无法查看订单支付状态'
			}, {
				tab: 3,
				url: 1
			});
			this.orderId = options.order_id;
			this.order_type = options.order_type;
			this.status = options.status || 0;
			this.msg = options.msg || '';
			if (this.isLogin) {
				this.getOrderPayInfo();
			} else {
                this.isAuto = true;
                this.isShowAuth = true
			}
		},
		methods: {
			// 优惠券展开
			bindMore(){
				this.isOpen = !this.isOpen
				this.text = this.text == '展开更多' ? '收起' : '展开更多';
			},
			onLoadFun: function() {
				this.isShowAuth = false;
				this.getOrderPayInfo();
			},
            // 授权关闭
            authColse: function(e) {
                this.isShowAuth = e;
            },
			/**
			 *
			 * 支付完成查询支付状态
			 *
			 */
			getOrderPayInfo: function() {
				let that = this;
				uni.showLoading({
					title: this.$t(`message.tips.loding`)
				});
				getPayOrder(that.orderId).then(res => {
					uni.hideLoading();
					that.$set(that, 'order_pay_info', res.data);
					that.couponList = res.data.give_coupon
					uni.setNavigationBarTitle({
						title: res.data.paid ? this.$t('page.orderPayStatus.success') : this.$t('page.orderPayStatus.fail')
					});
				}).catch(err => {
					uni.hideLoading();
				});
			},
			/**
			 * 去首页关闭当前所有页面
			 */
			goIndex: function(e) {
				uni.switchTab({
					url: '/pages/index/index'
				});
			},
			// 去参团页面；
			goPink: function(id) {
				uni.navigateTo({
					url: '/pages/activity/combination_status/index?id=' + id
				});
			},
			/**
			 *
			 * 去订单详情页面
			 */
			goOrderDetails: function(e) {
				let that = this;
				if (this.order_pay_info.paid == 0) {
					uni.redirectTo({
						url: '/pages/users/order_list/index'
					})
				} else {
					// // #ifdef MP
					// uni.showLoading({
					// 	title: this.$t(`message.tips.loding`),
					// })
					// openOrderSubscribe().then(res => {
					// 	uni.hideLoading();
					// 	uni.redirectTo({
					// 		url: '/pages/order_details/index?order_id=' + that.orderId
					// 	});
					// }).catch(() => {
					// 	nui.hideLoading();
					// });
					// // #endif
					// // #ifndef MP

					// // #endif
					uni.redirectTo({
						url: '/pages/users/order_list/index?status=1'
					})
				}

			}
		}
	}
</script>

<style lang="scss">
	.payment-status {
		background-color: #fff;
		margin: 92rpx 30rpx 30rpx;
		border-radius: 10rpx;
		padding: 1rpx 0 28rpx 0;
	}
	
	.payment-status .icon {
		font-size: 70rpx;
		width: 140rpx;
		height: 140rpx;
		border-radius: 50%;
		color: #fff;
		text-align: center;
		line-height: 140rpx;
		border: 6rpx solid #f5f5f5;
		margin: -76rpx auto 0 auto;
		background-color: #999;
		&.icon-duihao2{
			background-color: var(--view-theme);
		}
	}
	.payment-status .icon.fail {
		text-shadow: 0px 4px 0px #7a7a7a;
	}
	.payment-status .status {
		font-size: 32rpx;
		font-weight: bold;
		text-align: center;
		margin: 25rpx 0 37rpx 0;
	}
	.payment-status .wrapper {
		border: 1rpx solid #eee;
		margin: 0 30rpx 47rpx 30rpx;
		padding: 35rpx 0;
		border-left: 0;
		border-right: 0;
	}
	.payment-status .wrapper .item {
		font-size: 28rpx;
		color: #282828;
	}
	.payment-status .wrapper .item~.item {
		margin-top: 20rpx;
	}
	.payment-status .wrapper .item .itemCom {
		color: #666;
	}
	.payment-status .returnBnt {
		width: 630rpx;
		height: 86rpx;
		border-radius: 50rpx;
		color: #fff;
		font-size: 30rpx;
		text-align: center;
		line-height: 86rpx;
		margin: 0 auto 20rpx auto;
		&.gColor{
			background-color: var(--view-theme);
		}
		&.s-Color{
			color: var(--view-theme);
			 border: 1rpx solid var(--view-theme);
		}
	}
	.t-color {
		color: var(--view-theme)!important;
	}
	.coupon-wrapper{
		.hd{
			display: flex;
			align-items: center;
			justify-content: center;
			margin: 30rpx 0;
			color: #999999;
			font-size: 24rpx;
			.line{
				width: 70rpx;
				height: 1px;
				background: #DCDCDC;
			}
			.txt{
				margin: 0 20rpx;
			}
		}
		.coupon-box{
			height: 356rpx;
			padding: 0 20rpx;
			overflow: hidden;
			&.on{
				height: auto;
			}
			.coupon-item{
				width: 100%;
				margin-bottom: 20rpx;
				box-shadow:0px 2px 10px 0px rgba(0, 0, 0, 0.06);
				.left-bg{
					display: flex;
					align-items: center;
					justify-content: center;
					width: 236rpx;
					height: 160rpx;
					color: #fff;
					font-size: 64rpx;
					text{
						margin-top: 26rpx;
						font-size: 36rpx;
					}
				}
				.info{
					display: flex;
					flex-direction: column;
					justify-content: space-between;
					margin-left: 20rpx;
					padding: 20rpx 0;
					.title{
						color: #282828;
						font-size: 30rpx;
					}
					.des{
						font-size: 24rpx;
						color: #999999;
					}
				}
			}
		}
		.more{
			display: flex;
			align-items: center;
			justify-content: center;
			padding-top: 24rpx;
			font-size: 24rpx;
			color: #999999;
			.iconfont{
				margin-top: 6rpx;
				margin-left: 10rpx;
				font-size: 20rpx;
			}
		}
	}
</style>
