<template>
	<view class="content">
		<!-- <view class="content-bg">
			<image class="content-bg-image" :src='bgUrl'></image>
		</view> -->
		<view class="logo">
			<image class="logo-image" :src='logoUrl'></image>
			<image class="scrm-image" :src='scrmUrl'></image>
		</view>
		<view class="text-area">
			<text class="title">在线客服即时沟通</text>
			<text>与专业人员沟通，随时随地解决您的问题</text>
			<text>点击按钮立即发起咨询</text>
		</view>
		<view class="turn" v-show="customerVisble">
			<button v-if="!agent_id&&!group_id" class="turn-btn" open-type="contact" size="mini"
				session-from="udesk|{{userInfo.nickName}}|{{userInfo.avatarUrl}}|weifeng^{{weifeng}}"
				>
				{{service_btn}}
			</button>
			<button v-if="agent_id" class="turn-btn" open-type="contact" size="mini"
				session-from="udesk|{{userInfo.nickName}}|{{userInfo.avatarUrl}}|weifeng^{{weifeng}}|assign^{{agent_id}}"
				>
				{{service_btn}}
			</button>
			<button v-if="group_id" class="turn-btn" open-type="contact" size="mini"
				session-from="udesk|{{userInfo.nickName}}|{{userInfo.avatarUrl}}|weifeng^{{weifeng}}|assign^{{group_id}}"
				>
				{{service_btn}}
			</button>
		</view>
		<view class="turn" v-show="acceptVisble">
			<button class="turn-btn" lang="zh_CN" @click='onGotUserInfo'>点击授权</button>
		</view>
	</view>
</template>

<script>
	import common from '@/common/request.js'
	export default {
		data() {
			return {
				title: '基于企业微信的新一代CRM',
				service_btn: '点击接入在线客服',
				// bgUrl: '/static/bg-img.png',
				logoUrl: '/static/logo.png',
				scrmUrl: '/static/scrmbg.png',
				//获取用户openid的域名
				weifeng: '',
				userInfo: {},
				customerVisble: false,
				acceptVisble: false,
				codeType: 1, // 转客服类型 1:单聊转客服 2:群聊转客服
				loginType:1, // 登陆环境  1微信  2企业微信
				agent_id:null,
				group_id:null,
			}
		},
		onLoad(option) {
			// console.log(option);
			let that = this;
			wx.getSystemInfo({
				success(res){
					if(res.environment){
						that.loginType = 2;
					}
				}
			})
			// 获取转客服参数
			if (option.scene) {
				let weifengSessionId = decodeURIComponent(option.scene);
				if (weifengSessionId.slice(-1) === "1") {
					this.weifeng = weifengSessionId
				} else {
					this.codeType = 2;
					this.weifeng = weifengSessionId+'openId:';
				}
			}
			if(option.agent){
				let agentId = option.agent;
				this.agent_id = JSON.stringify({agent_id:agentId}) ;
			}
			if(option.group){
				let groupId = option.group;
				this.group_id = JSON.stringify({group_id:groupId}) ;
			}
			this.acceptVisble = true;
		},
		methods: {
			onGotUserInfo() {
				if(this.loginType ===1){
					let that = this;
					uni.getUserProfile({
						desc: '登录', // 不能省略 会报错
						success: res => {
							console.log(res);
							that.userInfo = res.userInfo;
							if (that.codeType === 1) {
								that.acceptVisble = false;
								that.customerVisble = true;
							} else {
								uni.login({
									provider: 'weixin',
									success: res => {
										uni.showLoading({
										    title: '加载中'
										});
										uni.request({
										    url: common.requestUrl+'/backend/customerServiceTurn/jscode2session',
											data: {
											    code: res.code
											},
										    success: (res) => {
												that.weifeng+=res.data.data.openid;
												console.log(that.weifeng);
												that.acceptVisble = false;
												that.customerVisble = true;
												uni.hideLoading();
										    },
											fail:(res)=>{console.log('失败');}
										});
									}
								})
							}
						},
						fail: function() {
							console.log('失败');
						}
					})
				}else{
					let that = this;
					uni.showToast({
					    title: '请您在微信中打开～',
					    icon: 'none'
					});
					// uni.getUserInfo({
					// 	success:function(res){
					// 		that.userInfo = res.userInfo;
					// 		if (that.codeType === 1) {
					// 			that.acceptVisble = false;
					// 			that.customerVisble = true;
					// 		}else{
					// 			uni.login({
					// 			      success: function(res) {
					// 					  uni.showLoading({
					// 					      title: '加载中'
					// 					  });
					// 			        if (res.code) {
					// 						console.log(res)
					// 			          //发起网络请求
					// 			          uni.request({
					// 			            url: common.requestUrl+'/backend/customerServiceTurn/jscode2session',
					// 			            data: {
					// 			              code: res.code
					// 			            },
					// 						success: (res) => {
					// 							that.weifeng+=res.data.data.open_userid;
					// 							console.log(that.weifeng);
					// 							that.acceptVisble = false;
					// 							that.customerVisble = true;
					// 							uni.hideLoading();
					// 						},
					// 						fail:(res)=>{console.log('失败');}
					// 			          })
										 
					// 			        } else {
					// 			          console.log('登录失败！' + res.errMsg)
					// 			        }
					// 			      }
					// 			});
					// 		}
					// 	}
					// })
					
				}
				

			}
		}
	}
</script>

<style>
	.content {
		position: fixed;
		top: 0px;
		bottom: 0px;
		left: 0px;
		right: 0px;
		height: 100%;
	}

	/* .content-bg-image {
		display: block;
		width: 100%;
		height: 100%;
		top: 0px;
		bottom: 0px;
		left: 0px;
		right: 0px;
		position: absolute;
		z-index: -1;
	} */

	.logo {
		width: 100%;
		text-align: center;
	}

	.logo-image {
		height: 28px;
		width: 184px;
		margin-top: 80rpx;
		margin-bottom: 20rpx;
	}
	.scrm-image{
		width: 288px;
		height: 288px;
		margin-bottom: 30rpx;
	}

	.text-area {
		display: flex;
		flex-direction: column;
		align-items: center;
		font-size: 14px;
		color: rgba(0, 0, 0, 0.45);
		line-height: 24px;
	}

	.title {
		font-size: 16px;
		color: rgba(0, 0, 0, 0.85);
		font-weight: 500;
		line-height: 36px;
	}

	.turn {
		position: absolute;
		bottom: 120rpx;
		width: 100%;
		display: flex;
		justify-content: center;
	}

	.turn-btn {
		display: block;
		width: 200px;
		height: 40px;
		background-color: #2C7CF6;
		border-radius: 4px;
		line-height: 40px !important;
		text-align: center;
		margin: 0 auto;
		font-size: 32rpx;
		color: #fff;
	}
	@media screen and (max-width: 350px) {
		.scrm-image{
			width: 240px;
			height: 240px;
		}
		.turn{
			bottom: 80rpx;
		}
	}
	@media screen and (min-width: 400px) {
		.text-area{
			margin-top: 40rpx;
		}
	}
</style>
