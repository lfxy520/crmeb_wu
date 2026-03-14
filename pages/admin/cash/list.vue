<template>
	<view :style="viewColor">
		<view class='commission-details'>
			<view class='promoterHeader'>
				<view class='headerCon acea-row row-between-wrapper'>
					<view>
						<view class='name'>{{$t(`page.users.userCash.totalextract`)}}</view>
						<view class='money'><text class='nums'>{{total_extract}}</text></view>
					</view>
					<navigator class="item" :url="`/pages/admin/cash/cash?merId=${mer_id}`" hover-class="none">
						<view>
							<view class='name' style="font-weight: ;">{{$t(`page.users.userCash.totalmoney`)}}</view>
							<view class='money'><text class='nums'>{{total_money}}</text></view>
						</view>
					</navigator>
				</view>
			</view>
			<view class='sign-record'>
				<block v-for="(item,index) in recordList" :key="index" v-if="recordList.length>0">
					<view class='list'>
						<view class='item'>
							<!-- <view class='data'>{{item.create_time}}</view> -->
							<view class='listn'>
								<view class='itemn acea-row row-between-wrapper'>
									<view>
										<block v-if="item.status<0">
											<view class='name line1'><text class="message" style="color: red;">{{getTypeName(item.financial_type)}} ({{getStatusName(item.status)}})</text></view>
										</block>
										<block v-else-if="item.status==1">
											<view class='name line1'><text class="message" style="color: green;">{{getTypeName(item.financial_type)}} ({{getStatusName(item.status)}})</text></view>
										</block>
										<block v-else>
											<view class='name line1'><text class="message" style="color: black;">{{getTypeName(item.financial_type)}} ({{getStatusName(item.status)}})</text></view>
										</block>
										<view>{{item.create_time}}</view>
									</view>
									<view>
										<view class='nums'>-{{item.extract_money}}</view>
										<view>{{$t(`page.users.userCash.balance`)}}:{{item.mer_money}}</view>
									</view>
								</view>
							</view>
						</view>
					</view>
				</block>
				<view v-if="recordList.length == 0"> 
					<emptyPage title='none~'></emptyPage> 
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
	import {
		extractLst,
	} from '@/api/admin.js';
	import { mapGetters } from "vuex";
	import authorize from '@/components/Authorize';
	import emptyPage from '@/components/emptyPage.vue'
	export default {
		components: {
			authorize,
			emptyPage
		},
		data() {
			return {
				type: 0,
				page: 1,
				limit: 12,
				total_extract:0,
				total_money:0,
				recordList: [],
				recordType: 0,
				recordCount: 0,
				status: false,
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				extractCount: 0,
				userInfo:'',
				mer_id: '',
			};
		},
		computed: mapGetters(['isLogin','viewColor']),
		onLoad(options) {
			this.mer_id = options.mer_id
		},
		onShow: function() {
			uni.setNavigationBarTitle({
				title: this.$t(`page.users.userCash.title`)
			});
			this.getRecordList(this.mer_id);
		},
		methods: {
			// 根据financial_type返回提现类型名称
			getTypeName(type) {
				switch(parseInt(type)) {
					case 1: return this.$t(`page.users.userSpread.bank`) || '银行卡';
					case 2: return '微信';
					case 3: return '支付宝';
					case 4: return 'USDT';
					case 5: return this.$t(`page.users.userCash.balance`) || '余额';
					default: return '未知';
				}
			},
			// 根据status返回状态名称
			getStatusName(status) {
				switch(parseInt(status)) {
					case -1: return this.$t(`page.users.userCash.statusRejected`) || '已拒绝';
					case 0: return this.$t(`page.users.userCash.statusPending`) || '审核中';
					case 1: return this.$t(`page.users.userCash.statusApproved`) || '已通过';
					default: return '未知';
				}
			},
			onLoadFun() {
				this.isShowAuth = false;
				this.getRecordList();
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			getRecordList: function(mer_id) {
				let that = this;
				let page = that.page;
				let limit = that.limit;
				let status = that.status;
				let recordList = that.recordList;
				let recordListNew = [];
				if (status == true) return;
				extractLst({
					page: page,
					limit: limit,
					mer_id: mer_id
				}, mer_id).then(res => {
					let len = res.data.list.length;
					let recordListData = res.data.list;
					that.total_extract = res.data.total_extract;
					that.total_money = res.data.total_money;
					recordListNew = recordList.concat(recordListData);
					that.status = limit > len;
					that.page+=1;
					that.$set(that, 'recordList', recordListNew);
				});
			}
		},
		onReachBottom: function() {
			this.getRecordList();
		}
	}
</script>

<style scoped lang="scss">
	.promoterHeader{
		background-image: linear-gradient(to right, var(--view-bntColor21) 0%, var(--view-bntColor22) 100%);
	}
	.commission-details .promoterHeader .headerCon .money {
		font-size: 36rpx;
	}
	.p-color {
		color: var(--view-priceColor);
	}
	.commission-details .listn .nums {
		font-size: 32rpx;
	}
	.message{
		font-size: 18rpx;
		color: #fc4141;
	}
</style>
