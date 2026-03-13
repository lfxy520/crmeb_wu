<template>
	<view class="change_lang">
		<view class="list">
			<view class='item acea-row row-between-wrapper' @click="switchZh">
				<view>{{ $t('userDrawer.language[0].name') }}</view>
				<text class='iconfont icon-you'></text>
			</view>
		</view>
		<view class="list other">
			<view class='item acea-row row-between-wrapper' @click="switchEn">
				<view>{{ $t('userDrawer.language[1].name') }}</view>
				<text class='iconfont icon-you'></text>
			</view>
		</view>
	</view>
</template>

<script>
	export default{
		name:'change_lang',
		data(){
			return{
				
			}
		},
		onShow() {
			uni.setNavigationBarTitle({
				title: this.$t('userDrawer.data[6].name')
			});
		},
		methods:{
				
			switchZh() {
				//中文
				this._i18n.locale = 'zh';
				uni.setStorageSync('lang', 'zh');
				uni.setNavigationBarTitle({
					title: this.$t('userDrawer.data[6].name')
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
					text: this.$t('tabbar.Home'),
				});
				uni.setTabBarItem({
					index: 1,
					pagePath: "pages/goods_cate/goods_cate",
					iconPath: "static/images/2-001.png",
					selectedIconPath: "static/images/2-002.png",
					text: this.$t('tabbar.Category'),
				});
				uni.setTabBarItem({
					index: 2,
					pagePath: "pages/order_addcart/order_addcart",
					iconPath: "static/images/3-001.png",
					selectedIconPath: "static/images/3-002.png",
					text: this.$t('tabbar.Cart'),
				});
				uni.setTabBarItem({
					index: 3,
					pagePath: "pages/user/index",
					iconPath: "static/images/4-001.png",
					selectedIconPath: "static/images/4-002.png",
					text: this.$t('tabbar.Mine'),
				});	
				uni.switchTab({
					url: '/pages/user/index'
				})
			},
			switchEn() {
				//英文
				this._i18n.locale = 'en';
				uni.setStorageSync('lang', 'en');
				uni.setNavigationBarTitle({
					title: this.$t('userDrawer.data[6].name')
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
				uni.switchTab({
					url: '/pages/user/index'
				})
			}
		}
	} 
</script>

<style scoped lang="scss">
	.change_lang {
		margin-top: 2rpx;
		.other {
			margin-top: 15rpx;
		}

		.list {
			background-color: #fff;

			.item {
				padding: 30rpx 30rpx 30rpx 0;
				border-bottom: 1rpx solid #f2f2f2;
				margin-left: 30rpx;
				font-size: 32rpx;
				color: #242424;

				.iconfont {
					font-size: 30rpx;
					color: #8A8A8A;
				}
			}
		}
	}
</style>
