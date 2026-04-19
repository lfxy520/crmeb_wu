<template>
	<view :style="viewColor">
		<form @submit="submitSub" report-submit='true'>
			<view class="payment-top acea-row row-column row-center-wrapper">
				<span class="name">{{$t(`page.users.userBill.Balance`)}}</span>
				<view class="pic">
					<span class="pic-font">{{ userinfo.now_money || 0 }}</span>
				</view>
			</view>
			<view class="payment">
				<view class="nav acea-row row-around row-middle">
					<view class="item" :class="active==index?'on':''" v-for="(item,index) in navRecharge" :key="index" @click="navRecharges(index)">{{item}}</view>
				</view>

				<view class='tip picList' v-if='!active'>
					<view class="pic-box pic-box-color acea-row row-center-wrapper row-column" :class="activePic == index ? 'pic-box-color-active' : ''"
					 v-for="(item, index) in picList" :key="index" @click="picCharge(index, item)">
						<view class="pic-number-pic">
							{{ item.data.price }}<span class="pic-number"> </span>
						</view>
					</view>
					<view class="pic-box pic-box-color acea-row row-center-wrapper" :class="rechar_id == 0 ? 'pic-box-color-active' : ''"
					 @click="picCharge(picList.length)">
						<input type="number" :placeholder="otherValue" v-model="money" class="pic-box-money pic-number-pic" :class="rechar_id == 0 ? 'pic-box-color-active' : ''" />
					</view>
					<view class="tips-box">
						<view class="tips mt-30">{{$t(`page.users.userPayment.method`)}}：</view>
						<view class="tips-samll">
							<view style="position: relative;">
								<image src="../../../static/images/usdt.png" style="width: 22px;height: 22px;"></image>
								<text style="position: absolute;top: 3px;left: 32px;">USDT</text>
								<checkbox class="checkbox" style="padding-left: 15px;float:right" @click="checkin()"/>
							</view>
						</view>
						<view class="tips-samll">
							<view style="position: relative;" @click="goMenuPage('/pages/chat/customer_list/chat?mer_id=0')">
								<image src="../../../static/images/bank.png" style="width: 22px;height: 22px;"></image>
								<text style="position: absolute;top: 3px;left: 32px;">{{$t(`page.users.userPayment.bank`)}}</text>
							</view>
						</view>
					</view>

				</view>
				<view class="tip" v-else>
				</view>
				<view v-if="is_checkin">
					<view class="nav acea-row row-around row-middle">
						<view class="item" :class="checkinex==index?'on':''" v-for="(item,index) in nav_checkin" :key="index" @click="CheckRecharges(index)">{{item}}</view>
					</view>					
					<view class='tip picList' v-if='!checkinex'>
						<image :src="trc20"></image>
					</view>
					<view class="tip" v-else>
						<image :src="erc20"></image>
					</view>
					<button class="copy" :data-copy="copydata" @tap="copyText">{{$t(`page.orderDetails.copy`)}}</button>
					<view class="item no-border">
						<view class='acea-row row-middle'>
							<text class="item-title">{{$t(`page.users.userPayment.voucher`)}} : </text>
							<view class="upload" style="padding-left: 15px;">
								<view class='pictrue' v-for="(item,index) in pics" :key="index" :data-index="index" @click="getPhotoClickIdx">
									<image :src='item'></image>
									<text class='iconfont icon-guanbi1' @click.stop='DelPic(index)'></text>
								</view>
								<view class='pictrue acea-row row-center-wrapper row-column' @click='uploadpic' v-if="pics.length < 10">
									<text class='iconfont icon-icon25201'></text>
									<view>{{$t(`page.users.goodsCommentCon.upload`)}}</view>
								</view>
							</view>
						</view>
					</view>
				</view>
				<button class='but' formType="submit">{{$t(`page.users.userPayment.confirm`)}}</button>
			</view>
		</form>
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
	import {
		spreadInfo,
		rechargeRoutine,
		rechargeWechat,
		getRechargeApi,
		rechargeBrokerage
	} from '@/api/user.js';
	import {
		mapGetters
	} from "vuex";
	import authorize from '@/components/Authorize';
	export default {
		components: {
			authorize
		},
		data() {
			let that = this;
			return {
				copydata:'',
				otherValue: this.$t(`page.users.userPayment.other`),
				now_money: 0,
				navRecharge: [this.$t(`page.users.userBill.Recharge`)],
				is_checkin:false,
				checkinex:0,
				nav_checkin:['trc20','erc20'],
				trc20:'',
				trc20_address:'',
				erc20:'',
				erc20_address:'',
				active: 0,
				number: '',
				userinfo: {},
				placeholder: "0.00",
				from: '',
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				picList: [],
				activePic: 0,
				money: "",
				numberPic: '',
				rechar_id: 0,
				rechargeAttention: [],
				pics: [],
			};
		},
		computed: mapGetters(['isLogin','viewColor']),
		onLoad(options) {
			// #ifdef H5
			this.from = this.$wechat.isWeixin() ? "weixin" : "h5"
			// #endif
			if (this.isLogin) {
				this.getUserInfo();
				this.getRecharge();
			} else {
				this.isAuto = true;
                this.isShowAuth = true
			}
		},
		methods: {
			copyText: function (e) {
			    var copy = e.currentTarget.dataset.copy;  //data-copy传过来的数值
			    wx.setClipboardData({
			      data: copy,
			      success: function (res) {
			        wx.getClipboardData({
			          success: function (res) {
			            wx.showToast({
			              title: 'copied'
			            })
			          }
			        })
			      }
			    })
			},
			/**
			 * 上传文件
			 *
			 */
			uploadpic: function() {
				let that = this;
				that.$util.uploadImageOne('upload/image', function(res) {
					that.pics.push(res.data.path);
					that.$set(that, 'pics', that.pics);
				});

			},	
			// 图片预览
			// 获得相册 idx
			getPhotoClickIdx(e) {
				let _this = this;
				let idx = e.currentTarget.dataset.index;
				_this.imgPreview(_this.pics, idx);
			},
			// 图片预览
			imgPreview: function(list, idx) {
				// list：图片 url 数组
				if (list && list.length > 0) {
					uni.previewImage({
						current: list[idx], //  传 Number H5端出现不兼容
						urls: list
					});
				}
			},
			/**
			 * 删除图片
			 *
			 */
			DelPic: function(index) {
				let that = this,
					pic = this.pics[index];
				that.pics.splice(index, 1);
				that.$set(that, 'pics', that.pics);
			},
			goMenuPage(url) {
				if (this.isLogin) {
					uni.navigateTo({
						url
					})
				} else {
					this.openAuto()
				}
			},
			checkin()
			{
				this.is_checkin = this.is_checkin?false:true;
				this.copydata = this.trc20_address;
			},
			picCharge(idx, item) {
				this.activePic = idx;
				console.log('item:'+item);
				if (item === undefined) {
					this.rechar_id = 0;
					this.numberPic = "";
					this.otherValue = ''
				} else {
					this.otherValue = this.$t(`page.users.userPayment.other`);
					this.money = "";
					this.rechar_id = item.id;
					this.numberPic = item.data.price;
				}
			},

			/**
			 * 充值额度选择
			 */
			getRecharge() {
				getRechargeApi()
					.then(res => {
						this.trc20 = res.data.usdt[0].data.trc20;
						this.erc20 = res.data.usdt[0].data.erc20;
						this.trc20_address = res.data.usdt[0].data.trc20_address;
						this.erc20_address = res.data.usdt[0].data.erc20_address;
						this.picList = res.data.recharge_quota;
						if (this.picList[0]) {
							this.rechar_id = this.picList[0].id;
							this.numberPic = this.picList[0].data.price;
						}
						this.rechargeAttention = res.data.recharge_attention || [];
					})
					.catch(res => {
						this.$dialog.toast({
							mes: res
						});
					});
			},

			onLoadFun: function() {
				this.isShowAuth = false;
				this.getUserInfo();
				this.getRecharge();
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			navRecharges: function(index) {
				this.active = index;
			},
			CheckRecharges: function(index) {
				this.checkinex = index;
				this.copydata = index?this.erc20_address:this.trc20_address;
			},
			/**
			 * 获取用户信息
			 */
			getUserInfo: function() {
				let that = this;
				spreadInfo().then(res => {
					that.$set(that, 'userinfo', res.data);
				})
			},
			/*
			 * 用户充值
			 */
			submitSub: function(e) {
				let that = this
				let value = e.detail.value.number;
				// 转入余额
				if (that.active) {
				} else {
					uni.showLoading({
						title: this.$t(`page.users.userPayment.submit`),
						mask:true
					})

					if (this.rechar_id == 0) {
						if (parseFloat(that.money)=== 0) {
							return that.$util.Tips({
								title: this.$t(`page.users.userPayment.tips1`)
							});
						}
						if (!that.money) {
							return that.$util.Tips({
								title: this.$t(`page.users.userPayment.tips2`)
							});
						}
						if (!Number(that.money)) {
							return that.$util.Tips({
								title: this.$t(`page.users.userPayment.tips3`)
							});
						}
					}
					if(that.pics.length == 0)
					{
						return that.$util.Tips({
							title: this.$t(`page.users.userPayment.tips4`)
						});							
					}
					// #ifdef H5
					rechargeWechat({
						price: that.rechar_id == 0 ? that.money : that.numberPic,
						type: that.from,
						pics:that.pics,
						recharge_id: that.rechar_id
					}).then(res => {
							return that.$util.Tips({
								title: this.$t(`page.users.userPayment.tips5`),
								icon: 'success'
							}, {
								tab: 5,
								url: '/pages/users/user_money/index'
							});
					}).catch(error=>{
						return that.$util.Tips({
							title: error
						});
					})
				// #endif
				}
			}
		}
	}
</script>

<style lang="scss">
	page {
		width: 100%;
		height: 100%;
		background-color: #fff;
	}
	.item {
		border-bottom: 1rpx solid #eee;
		position: relative;
		margin: 0 20px;
		&.no-border {
			border-bottom: none;
			padding-left: 0;
			padding-right: 0;
		}
		.item-title {
			color: #666666;
			font-size: 28rpx;
			display: block;
		}
		.item-desc {
			color: #B2B2B2;
			font-size: 22rpx;
			display: block;
			margin-top: 9rpx;
			line-height: 36rpx;
		}
	}	
	.acea-row.row-middle {
		-webkit-box-align: center;
		-moz-box-align: center;
		-o-box-align: center;
		-ms-flex-align: center;
		-webkit-align-items: center;
		align-items: center;
		padding-left: 2px;
	}
	.acea-row,.upload {
		display: -webkit-box;
		display: -moz-box;
		display: -webkit-flex;
		display: -ms-flexbox;
		display: flex;
		-webkit-box-lines: multiple;
		-moz-box-lines: multiple;
		-o-box-lines: multiple;
		-webkit-flex-wrap: wrap;
		-ms-flex-wrap: wrap;
		flex-wrap: wrap;
	}
	.upload {
		margin-top: 20rpx;
	}	
	.pictrue {
		width: 130rpx;
		height: 130rpx;
		margin: 24rpx 22rpx 0 0;
		position: relative;
		font-size: 11px;
		color: #bbb;
		&:nth-child(4n) {
			margin-right: 0;
		}
		&:nth-last-child(1) {
			border: 0.5px solid #ddd;
			box-sizing: border-box;
		}
		uni-image,
		image {
			width: 100%;
			height: 100%;
			border-radius: 1px;
			img {
				-webkit-touch-callout: none;
				-webkit-user-select: none;
				-moz-user-select: none;
				display: block;
				position: absolute;
				top: 0;
				left: 0;
				opacity: 0;
				width: 100%;
				height: 100%;
			}
		}
		.icon-guanbi1 {
			font-size: 33rpx;
			position: absolute;
			top: -10px;
			right: -10px;
		}
	}
	.payment {
		position: relative;
		top: -60rpx;
		width: 100%;
		background-color: #fff;
		border-radius: 10rpx;
		padding-top: 25rpx;
		border-top-right-radius: 39rpx;
		border-top-left-radius: 39rpx;
	}

	.payment .nav {
		height: 75rpx;
		line-height: 75rpx;
		padding: 0 100rpx;
	}

	.payment .nav .item {
		font-size: 30rpx;
		color: #333;
	}

	.payment .nav .item.on {
		font-weight: bold;
		border-bottom: 4rpx solid var(--view-theme);
	}
	.t-color{color:var(--view-theme);}
	.payment .input {
		display: flex;
		align-items: center;
		justify-content: center;
		border-bottom: 1px dashed #dddddd;
		margin: 60rpx auto 0 auto;
		padding-bottom: 20rpx;
		font-size: 56rpx;
		color: #333333;
		flex-wrap: nowrap;
	}
	.payment .input text {
		padding-left: 106rpx;
	}
	.payment .input input {
		padding-right: 106rpx;
		width: 300rpx;
		height: 94rpx;
		text-align: center;
		font-size: 70rpx;
	}
	.payment .placeholder {
		color: #d0d0d0;
		height: 100%;
		line-height: 94rpx;
	}

	.payment .tip {
		font-size: 26rpx;
		color: #888888;
		padding: 0 30rpx;
		margin-top: 25rpx;
	}

	.payment .but {
		color: #fff;
		font-size: 30rpx;
		width: 700rpx;
		height: 86rpx;
		border-radius: 50rpx;
		margin: 46rpx auto 0 auto;
		line-height: 86rpx;
		background-color: var(--view-theme);
	}

	.payment-top {
		width: 100%;
		height: 350rpx;
		background-color: var(--view-theme);

		.name {
			font-size: 26rpx;
			color: rgba(255, 255, 255, 0.8);
			margin-top: -38rpx;
			margin-bottom: 30rpx;
		}

		.pic {
			font-size: 32rpx;
			color: #fff;
		}

		.pic-font {
			font-size: 78rpx;
			color: #fff;
		}
	}
	.copy {
		font-size: 20rpx;
		color: #868686;
		border-radius: 3rpx;
		border: 1px solid #868686;
		padding: 0rpx 15rpx;
		margin-left: 24rpx;
		height: 40rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 16rpx;
		width: 100px;
		margin: 0 auto;
	}
	.picList {
		display: flex;
		flex-wrap: wrap;
		margin: 30rpx 0;

		.pic-box {
			width: 32%;
			height: auto;
			border-radius: 20rpx;
			margin-top: 21rpx;
			padding: 20rpx 0;
			margin-right: 12rpx;

			&:nth-child(3n) {
				margin-right: 0;
			}
		}

		.pic-box-color {
			background-color: #f4f4f4;
			color: #656565;
		}

		.pic-number {
			font-size: 22rpx;
		}

		.pic-number-pic {
			font-size: 38rpx;
			margin-right: 10rpx;
			text-align: center;
		}

		.pic-box-color-active {
			background-color: var(--view-theme);
			color: #fff;
		}
	}

	.tips-box {
		width: 100%;
		.tips {
			font-size: 28rpx;
			color: #333333;
			font-weight: 800;
			margin-bottom: 14rpx;
			margin-top: 20rpx;
		}

		.tips-samll {
			padding: 15px;
			border-bottom: 1px solid #eee;
			font-size: 12px;
			color: #020202;
		}

		.tip-box {
			margin-top: 30rpx;
		}
	}

	.tips-title {
		margin-top: 20rpx;
		font-size: 24rpx;
		color: #333;
	}
</style>
