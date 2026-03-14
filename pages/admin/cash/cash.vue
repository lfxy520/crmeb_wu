<template>
	<view :style="viewColor">
		<view class='cash-withdrawal' :hidden='!loading'>
			<view class='nav acea-row  row-between-wrapper'>
				<view class='name'>{{$t(`page.users.userCash.method`)}}:</view>
				<view class='input'>
					<view class="more-content" @click="goPay(currentTab)">
						<view class="more-content-left">
							<view class="type-icon" :style="[{backgroundColor: handlePayColor()}]">
								<view class="iconfont bankicon" :class="navList[currentTab]['icon']"></view>
							</view>
							<text style="position: absolute;top: 22px;">{{navList[currentTab]["name"]}}</text>
						 </view>
						 <view class="moreicon">
							<view class='iconfont icon-jiantou'></view>
						 </view>
					</view>
				</view>
			</view>
			<view class='wrapper'>
				<view :hidden='currentTab != 1' class='list'>
					<form @submit="subCash" report-submit='true'>
						<view class='item acea-row row-between-wrapper'>
							<view class='name'>{{$t(`page.users.userCash.address`)}}:</view>
							<view class='input'><input placeholder='' placeholder-class='placeholder' name="address"></input></view>
						</view>
						<view class='item acea-row row-between-wrapper'>
							<view class='name'>{{$t(`page.users.userCash.amount`)}}:</view>
							<view class='input'><input placeholder='' placeholder-class='placeholder' name="extract_money" type='digit' v-model="extract_money"></input></view>
						</view>
						<view class="btn-submit">
							<button formType="submit" :disabled="load" class='bnt b-color' :class="load ? 'disabled' : ''" >{{$t(`page.users.userCash.submit`)}}</button>
						</view>
					</form>
				</view>
				<view :hidden='currentTab != 0' class='list'>
					<form @submit="subCash" report-submit='true'>
						<view class='item acea-row row-between-wrapper'>
							<view class='name'>{{$t(`page.users.userSpread.ckr`)}}:</view>
							<view class='input'><input placeholder='' placeholder-class='placeholder' name="name"></input></view>
						</view>
						<view class='item acea-row row-between-wrapper'>
							<view class='name'>{{$t(`page.users.userSpread.cardno`)}}:</view>
							<view class='input'><input type='number' placeholder='' placeholder-class='placeholder' name="bank_code"></input></view>
						</view>
						<view class='item acea-row row-between-wrapper'>
							<view class='name'>{{$t(`page.users.userSpread.bank`)}}:</view>
							<view class='input'><input type='text' placeholder='' placeholder-class='placeholder' name="bank"></input></view>
						</view>
						<view class='item acea-row row-between-wrapper'>
							<view class='name'>{{$t(`page.users.userSpread.amount`)}}:</view>
							<view class='input'><input placeholder='' placeholder-class='placeholder' name="extract_money"
								 type='digit' v-model="extract_money"></input></view>
						</view>
						<view class="btn-submit">
							<button formType="submit" :disabled="load" class='bnt b-color' :class="load ? 'disabled' : ''" >{{$t(`page.users.userSpread.tixian`)}}</button>
						</view>
					</form>
				</view>
				<!-- 余额提现表单 -->
				<view :hidden='currentTab != 2' class='list'>
					<form @submit="subCash" report-submit='true'>
						<view class='item acea-row row-between-wrapper'>
							<view class='name'>{{$t(`page.users.userSpread.amount`)}}:</view>
							<view class='input'><input placeholder='' placeholder-class='placeholder' name="extract_money"
								 type='digit' v-model="extract_money"></input></view>
						</view>
						<view class="btn-submit">
							<button formType="submit" :disabled="load" class='bnt b-color' :class="load ? 'disabled' : ''" >{{$t(`page.users.userSpread.tixian`)}}</button>
						</view>
					</form>
				</view>
			</view>
		</view>
		<cash :payMode='pay_type' :pay_close="pay_close" @payClose="payClose" @onChangeFun="onChangeFun" :order_id="currentTab"></cash>
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
		extractCash
	} from '@/api/admin.js';
	import { mapGetters } from "vuex";
	import cash from '@/components/cash';
	import { configMap } from '@/utils';
	export default { 
		components: {
			cash
		},
		data() {
			return {
				navList: [
					{
						    'id': '0',
						    'ids': 1,
							'name': this.$t(`page.users.userSpread.bank`),
							'icon': 'icon-yinhangqia',
							'bg_color': '#FE960F'
					},
					{
						    'id': '1',
						    'ids': 2,
							'name': 'USDT',
							'icon': 'icon-yinhangqia',
							'bg_color': '#FE960F'
						},
					{
						    'id': '2',
						    'ids': 5,
							'name': this.$t(`page.users.userCash.balance`) || '余额',
							'icon': 'icon-yue',
							'bg_color': '#41B035'
						},
				],
				currentTab: '0',
				extract_money: '',
				index: 0,
				array: [], //提现银行
				minPrice: 0.00, //最低提现金额
				userInfo: [],
				isClone: false,
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				loading: true,
				load: false,
				pics: [], //收款码
				extract_pic: '',
				placeholderValue: '0.00',
				payColor: '',
				pay_close: false,
				pay_type: [],
				mer_id: '',
			};
		},
		computed: {
			...mapGetters(['isLogin','viewColor','uid']),
			... configMap(['sys_extension_type', 'withdraw_type'])
		},
		watch: {
			withdraw_type: {
				handler(){
					this.loadType()
				},
				immediate: true
			}
		},
		onLoad(options) {
			this.mer_id = options.merId;
		},
		methods: {
			loadType(){
				this.pay_type = []
				this.navList.map((value, index) => {
					this.pay_type.push(value)
				})
				this.currentTab = this.pay_type[0].id
			},

			onLoadFun: function() {
				this.isShowAuth = false;
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},

			subCash: function(e) {
				let that = this,
					value = e.detail.value;
				if(that.load) return;
				if (that.currentTab == 0) { //银行卡
					if (value.name.length == 0) return this.$util.Tips({
						title: this.$t(`page.users.userSpread.tips1`)
					});
					if (value.bank_code.length == 0) return this.$util.Tips({
						title: this.$t(`page.users.userSpread.tips2`)
					});
					if (value.bank.length == 0) return this.$util.Tips({
						title: this.$t(`page.users.userSpread.tips6`)
					});
					value.financial_type = 1;
				}else if (that.currentTab == 1) { //USDT
					if (value.address.length == 0) return this.$util.Tips({
						title: this.$t(`page.users.userSpread.tips5`)
					});
					value.financial_type = 4;
				}else if (that.currentTab == 2) { //余额提现
					value.financial_type = 5;
				}
				
				if (value.extract_money.length == 0) return this.$util.Tips({
					title: this.$t(`page.users.userCash.tips2`)
				});
				if (Number(value.extract_money) < that.minPrice) return this.$util.Tips({
					//title: this.$t(`page.users.userCash.tips1`) + that.minPrice
				});
				that.load = true;
				extractCash(value,that.mer_id,that.uid).then(res => {
					that.load = false;
					// 余额提现直接到账，其他需要审核
					let successMsg = (that.currentTab == 2)
						? (this.$t(`page.users.userCash.balanceWithdrawSuccess`) || '提现成功，已到余额')
						: this.$t(`page.users.userCash.tips6`);
					that.$util.Tips({
						title: successMsg,
						icon: 'success'
					});
					setTimeout(function(){
						uni.navigateTo({
							url: '/pages/admin/business/index'
						});
					},1000)
				}).catch(err => {
					that.load = false;
					// 如果是翻译key则翻译
					let errMsg = err;
					if (typeof err === 'string' && err.startsWith('page.')) {
						// page.userCash.xxx -> page.users.userCash.xxx
						// page.financial.xxx -> page.users.financial.xxx
						let key = err.replace('page.', 'page.users.');
						errMsg = this.$t(key) || this.$t(err) || err;
					}
					return that.$util.Tips({
						title: errMsg
					});
				});
			},
			handlePayColor: function() {
				let str = ''
				if (this.currentTab == 1) {
					str = '#41B035'
				} else if (this.currentTab == 2) {
					str = '#00A9F2'
				} else if (this.currentTab == 3) {
					str = '#0000ff'
				}else {
					str = '#FE960F '
				}
				return str
			},
			/**
			 * 打开支付组件
			 *
			 */
			goPay: function(pay_price) {
				this.$set(this, 'pay_close', true);
				// this.$set(this, 'pay_order_id', );
			
			},
			/**
			 * 关闭支付组件
			 *
			 */
			payClose: function() {
				this.pay_close = false;
			},
			onChangeFun: function(e) {
				let opt = e;
				let action = opt.action || null;
				let value = opt.value != undefined ? opt.value : null;
				(action && this[action]) && this[action](value);
				this.currentTab = opt.type
			},
		}
	}
</script>

<style lang="scss">
	page {
		background-color: #F5F5F5 !important;
	}
    .mt25{
		margin-top: 25rpx;
	}
	.cash-withdrawal .nav {
		height: 130rpx;
		padding: 0 30rpx;
		font-size: 30rpx;
		margin-bottom: 20rpx;
		background-color: #fff;
	}
	.b-color{background-color: var(--view-theme);}
	.cash-withdrawal .nav .input {
		width: 505rpx;
		.more-content{
			display: flex;
			align-items: center;
			.more-content-left{
				width: 90%;
				.type-icon{
					display: inline-block;
					width: 56rpx;
					height: 56rpx;
					text-align: center;
					line-height: 56rpx;
					background-color: #FE960F;
					margin-right: 20rpx;
					border-radius: 50%;
					.bankicon{
						font-size: 36rpx;
						color: #FFFFFF;
					}
				}
			}
			.moreicon{
				width: 10%;
				text-align: right;
			}
		}
	}

	.cash-withdrawal .nav .item {
		font-size: 26rpx;
		flex: 1;
		text-align: center;
	}

	.cash-withdrawal .nav .item~.item {
		border-left: 1px solid #f0f0f0;
	}

	.cash-withdrawal .nav .item .iconfont {
		width: 40rpx;
		height: 40rpx;
		border-radius: 50%;
		border: 2rpx solid #e93323;
		text-align: center;
		line-height: 37rpx;
		margin: 0 auto 6rpx auto;
		font-size: 22rpx;
		box-sizing: border-box;
	}

	.cash-withdrawal .nav .item .iconfont.on {
		background-color: #e93323;
		color: #fff;
		border-color: #e93323;
	}

	.cash-withdrawal .nav .item .line {
		width: 2rpx;
		height: 20rpx;
		margin: 0 auto;
		transition: height 0.3s;
	}

	.cash-withdrawal .nav .item .line.on {
		height: 39rpx;
	}

	.cash-withdrawal .wrapper .list {
		padding: 0 30rpx;
		background-color: #fff;
	}
	.cash-withdrawal .wrapper .list .item {
		border-bottom: 1rpx solid #eee;
		height: 107rpx;
		font-size: 30rpx;
		color: #333;

		&.uploadItem {
			border-bottom: none;
			height: auto;

			.name {
				height: 107rpx;
				;
			}
		}
	}

	.picture {
		width: 70px;
		height: 70px;
		margin: 0 0 17px 0;
		position: relative;
		font-size: 11px;
		color: #bbb;
		border: 0.5px solid #ddd;
		box-sizing: border-box;
		margin-top: 40rpx;

		uni-image,image {
			width: 100%;
			height: 100%;
			border-radius: 1px;
		}

		.icon-guanbi1 {
			font-size: 22px;
			position: absolute;
			top: -10px;
			right: -10px;
			color: #fc4141;
		}
	}

	.cash-withdrawal .wrapper .list .item .name {
		width: 140rpx;
	}

	.cash-withdrawal .wrapper .list .item .input {
		width: 505rpx;
	}

	.cash-withdrawal .wrapper .list .item .input .placeholder {
		color: #bbb;
	}
	.cash-withdrawal .placeholder1 {
		font-size: 46rpx;
	}
	.cash-withdrawal .wrapper .list .tip {
		font-size: 26rpx;
		color: #999;
		margin-bottom: 25rpx;
	}
   .cash-withdrawal .wrapper .list .btn-submit{
	   background-color: #F5F5F5;
	   margin: 0 -30rpx;
	   padding: 64rpx 30rpx;
   }
	.cash-withdrawal .wrapper .list .bnt {
		font-size: 32rpx;
		color: #fff;
		width: 690rpx;
		height: 90rpx;
		text-align: center;
		border-radius: 50rpx;
		line-height: 90rpx;
		/deep/ &.disabled {
			background: #E3E3E3!important;
			pointer-events: none;
		}
	}



	.cash-withdrawal .wrapper .list .tip2 {
		font-size: 26rpx;
		color: #999;
		text-align: center;
		margin: 44rpx 0 20rpx 0;
	}

	.cash-withdrawal .wrapper .list .value {
		height: 135rpx;
		line-height: 135rpx;
		border-bottom: 1rpx solid #eee;
		width: 690rpx;
		margin: 0 auto;
	}

	.cash-withdrawal .wrapper .list .value input {
		font-size: 80rpx;
		color: #282828;
		height: 135rpx;
		text-align: center;
	}
	.cash-withdrawal .wrapper .list .value .placeholder2 {
		color: #bbb;
	}
	.price {
		color: var(--view-priceColor);
	}
	.Bank {
		display: block;
		width: 100%;
		text-overflow: ellipsis;
		overflow: hidden;
		white-space: nowrap;
	}
	.auto_arrival{
		text-align: center;
		padding: 20rpx 0 0 0;
		.input{
			width: 100%;
			border-bottom: 1rpx solid #eee;
			margin-top: 10rpx;
			padding: 20rpx 0;
			font-size: 60rpx;
			color: #999;
			height: 100rpx;
			uni-input{
				height: 100%;
			}
		}
	}
	 uni-toast.uni-mask{
		background-color: rgba(0,0,0,0.5) !important;
	}
</style>
