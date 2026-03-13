<template>
	<div class="register absolute">
		<div class="shading">
			<div class="pictrue acea-row row-center-wrapper">
				<image :src="login_logo" v-if="login_logo" />
			</div>
		</div>
		<div class="whiteBg">
			<div class="title">{{$t(`page.users.login.forget`)}}</div>
			<div class="list">
				<div class="item">
					<div class="acea-row row-middle">
						<image src="/static/images/phone_1.png"></image>
						<input type="text" :placeholder="$t(`message.settled.emptyPhone`)" v-model="account" autocomplete="off" />
					</div>
				</div>
				<div class="item">
					<div class="acea-row row-middle">
						<image src="/static/images/code_2.png"></image>
						<input type="password" :placeholder="$t(`message.login.emptyPassword`)" v-model="password" autocomplete="off" />
					</div>
				</div>
				<div class="item">
					<div class="acea-row row-middle">
						<image src="/static/images/code_2.png"></image>
						<input type="password" :placeholder="$t(`message.login.againPassword`)" v-model="confirm_pwd" autocomplete="off" />
					</div>
				</div>
				<div class="item">
					<div class="acea-row row-middle">
						<image src="/static/images/code_2.png"></image>
						<input type="text" :placeholder="$t(`page.users.register.placeCode`)" class="codeIput" v-model="captcha" autocomplete="off" />
						<button class="code" :disabled="disabled" :class="disabled === true ? 'on' : ''" @click="code">
							{{ text }}
						</button>
					</div>
				</div>
				<div class="item" v-if="isShowCode">
					<div class="align-left">
						<input type="text" :placeholder="$t(`page.users.register.placeCode`)" class="codeIput" v-model="codeVal" />
						<div class="code" @click="again"><image :src="codeUrl" /></div>
					</div>
				</div>
			</div>
			<div class="logon" @click="registerReset">{{$t(`page.users.login.submit`)}}</div>
			<div class="tip">
				<span class="font-color-red" @click="back">{{$t(`page.users.register.sign`)}}</span>
			</div>
		</div>
		<div class="bottom"></div>
	</div>
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
	const app = getApp();
	import sendVerifyCode from "@/mixins/SendVerifyCode";
	import {
		registerVerify,
		registerForget,
		getCodeApi,
		getCaptcha
	} from "@/api/user";
	import {
		validatorDefaultCatch
	} from "@/utils/dialog";
	import attrs, {
		required,
		alpha_num,
		chs_phone
	} from "@/utils/validate";
	import { configMap } from '@/utils';

	export default {
		name: "RetrievePassword",
		mixins: [sendVerifyCode],
		data: function() {
			return {
				account: "",
				password: "",
				confirm_pwd: "",
				captcha: "",
				codeKey: "",
				codeUrl: "",
				codeVal: "",
				isShowCode: false,
			};
		},
		computed: configMap(['login_logo']),
		onReady() {
		},
		mounted: function() {
		},
		methods: {
			back() {
				uni.navigateBack();
			},
			again() {
				this.codeUrl =
					VUE_APP_API_URL + "/captcha?" + this.keyCode + Date.parse(new Date());
				console.log(this.codeUrl);
			},

			async code() {
				let that = this;
				// that.sendCode();

				if (!that.account) return that.$util.Tips({
					title: this.$t(`page.users.register.placePhone`)
				});

				await registerVerify({
						phone: that.account,
						type: 'change_pwd',
						key: that.codeKey,
						code: that.codeVal,
					})
					.then(res => {

						that.$util.Tips({
							title: res.message
						});
						that.sendCode();
					}).catch(res => {
						console.log(res, 'res')
						that.getcaptcha();
						that.$util.Tips({
							title: res
						});
					});

			},

			getcaptcha() {
				let that = this
				getCaptcha().then(data => {
					that.codeUrl = data.data.captcha; //图片路径
					that.codeVal = data.data.code; //图片验证码
					that.codeKey = data.data.key //图片验证码key
				})
				that.isShowCode = true;
			},
			async registerReset() {
				var that = this;
				if (!that.account) return that.$util.Tips({
					title: this.$t(`page.users.register.placePhone`)
				});
				if (that.password != that.confirm_pwd) return that.$util.Tips({
					title: this.$t(`message.login.diffPassword`)
				});
				if (!that.captcha) return that.$util.Tips({
					title: this.$t(`message.login.emptyCaptche`)
				});
				registerForget({
						phone: that.account,
						sms_code: that.captcha,
						pwd: that.password,
						confirm_pwd: that.confirm_pwd
					})
					.then(res => {
						that.$util.Tips({
							title: res.msg
						}, {
							tab: 3
						})
					})
					.catch(res => {
						that.$util.Tips({
							title: res
						})
					});
			},
		}
	};
</script>
<style scoped>
	.code img {
		width: 100%;
		height: 100%;
	}
</style>
