<template>
	<view class="container">
		<select-form :platformClassification="formList" :form="singleSpecification" @input="input"></select-form>
		<view class="handle">
			<view class="button" @click="saveSingleSpecification">
				{{$t('page.product.save')}}
			</view>
		</view>
	</view>
</template>

<script>
	import selectForm from '../components/selectForm.vue';
	import { navigateTo, navigateBack, serialize, setStorage, getStorage} from '../../../libs/uniApi.js';
	// 单规格商品 attrValue
	export default {
		components:{
			selectForm
		},
		data() {
			return {
				singleSpecification: {
					price: '', // 售价
					stock: '', // 库存
					image: '',
					extension_one: '',
					extension_two: ''
				},
				moreThanFlag: true,
				formList: [
					{
						id: 1,
						label: this.$t('page.product.price'),
						type: 'input',
						model: 'price',
						holder: this.$t('page.product.pleasefil')+this.$t('page.product.price')
					},
					{
						id: 4,
						label: this.$t('page.product.inventory'),
						type: 'input',
						holder: this.$t('page.product.pleasefil')+this.$t('page.product.inventory'),
						model: 'stock'
					}
				],
				
			}
		},
		watch: {
			singleSpecification: {
				handler(val) {
					this.singleSpecification = val
				},
				deep: true
			}
		},
		onShow() {
			if(getStorage('addGoodsFormData').image) {
				this.singleSpecification.image = getStorage('addGoodsFormData').image;
			}
			if(getStorage('singleSpecification')) {
				Object.keys(this.singleSpecification).forEach(item => {
					if(getStorage('singleSpecification')[item]) {
						this.singleSpecification[item] = getStorage('singleSpecification')[item]
					}
				})
				
			}
		},
		methods: {
			selectMoreThan() {
				this.formList = this.formList.concat(this.moreThanList);
				this.moreThanFlag = false;
			},
			spliceMoreThan() {
				this.moreThanFlag = true;
				this.formList.splice(3, 4);
			},
			input(val) {
				this.singleSpecification = val
			},
			// 保存
			saveSingleSpecification() {
				setStorage('singleSpecification', this.singleSpecification);
				navigateBack(1);
			}
		}
	}
</script>

<style lang="scss" scoped>
	.container {
		padding-bottom: 126rpx;
	}
	.more_than {
		background: #FFFFFF;
		border-radius: 10px;
		margin: auto;
		margin-top: 15rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		width: 710rpx;
		height: 84rpx;
		color: #333333;
		font-size: 30rpx;
	}
	
	.handle {
		position: fixed;
		left: 0;
		bottom: 0;
		width: 100%;
		height: 126rpx;
		background: #FFFFFF;
		display: flex;
		align-items: center;
		justify-content: center;
		.button {
			display: flex;
			align-items: center;
			justify-content: center;
			color: #FFFFFF;
			font-size: 32rpx;
			width: 690rpx;
			height: 86rpx;
			background: #E93323;
			border-radius: 43rpx;
		}
	}
</style>
