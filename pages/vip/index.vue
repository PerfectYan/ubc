<template>
	<view class="vip-wrapper">
		<cu-custom bgColor="bg-gradual-blue" :isBack="true">
			<block slot="content">会员专属</block>
		</cu-custom>
		<view class="bg-vip flex1">
			<!-- <image style="width: 100%;height: 100%;" mode="aspectFill" :src="bgVip"></image> -->
			<view class="btn-re-sub" @tap="resumeSub">恢复订阅</view>
			<match-media :height="667" class="height667">
				<view class="bg-sub">
					<view class="sub-title">订阅后享受的特权</view>
					<view class="sub-desc">美容项目添加数量不限</view>
					<view class="sub-desc">闹钟添加数量不限</view>
					<view class="sub-items flex justify-around">
						<view class="sub-item" @tap="requestPayment('weekSub')">
							<view>周订阅</view>
							<view>8.00/周</view>
							<view class="f12">自动续费</view>
						</view>
						<view class="sub-item" @tap="requestPayment('yearSub')">
							<view>年订阅</view>
							<view>98.00/年</view>
							<view class="f12">自动续费</view>
						</view>
					</view>
				</view>
				<view class="sub-info f12">
					用户确认购买并付款后计入iTunes账户，除非您在到期日24小时前取消，否则我们将在您会员过期前24小时进行自动进行续费（含免费试用），扣费成功后订阅周期顺延一个订阅周期。
				</view>
				<view class="privacy-terms flex justify-center align-center">
					<view class="privacy" @tap="openUrl(privacy)">隐私策略</view>
					<view class="terms" @tap="openUrl(terms)">使用条款</view>
				</view>
			</match-media>
			<match-media :height="736" class="height736">
				<view class="bg-sub">
					<view class="sub-title">订阅后享受的特权</view>
					<view class="sub-desc">美容项目添加数量不限</view>
					<view class="sub-desc">闹钟添加数量不限</view>
					<view class="sub-items flex justify-around">
						<view class="sub-item" @tap="requestPayment('weekSub')">
							<view>周订阅</view>
							<view>8.00/周</view>
							<view class="f12">自动续费</view>
						</view>
						<view class="sub-item" @tap="requestPayment('yearSub')">
							<view>年订阅</view>
							<view>98.00/年</view>
							<view class="f12">自动续费</view>
						</view>
					</view>
				</view>
				<view class="sub-info f12">
					用户确认购买并付款后计入iTunes账户，除非您在到期日24小时前取消，否则我们将在您会员过期前24小时进行自动进行续费（含免费试用），扣费成功后订阅周期顺延一个订阅周期。
				</view>
				<view class="privacy-terms flex justify-center align-center">
					<view class="privacy" @tap="openUrl(privacy)">隐私策略</view>
					<view class="terms" @tap="openUrl(terms)">使用条款</view>
				</view>
			</match-media>
			<match-media :min-height="812" class="height812">
				<view class="bg-sub">
					<view class="sub-title">订阅后享受的特权</view>
					<view class="sub-desc">美容项目添加数量不限</view>
					<view class="sub-desc">闹钟添加数量不限</view>
					<view class="sub-items flex justify-around">
						<view class="sub-item" @tap="requestPayment('weekSub')">
							<view>周订阅</view>
							<view>{{orderList[0].price}}/周</view>
							<view class="f12">自动续费</view>
						</view>
						<view class="sub-item" @tap="requestPayment('yearSub')">
							<view>年订阅</view>
							<view>{{orderList[1].price}}/年</view>
							<view class="f12">自动续费</view>
						</view>
					</view>
				</view>
				<view class="sub-info f12">
					用户确认购买并付款后计入iTunes账户，除非您在到期日24小时前取消，否则我们将在您会员过期前24小时进行自动进行续费（含免费试用），扣费成功后订阅周期顺延一个订阅周期。
				</view>
				<view class="privacy-terms flex justify-center align-center">
					<view class="privacy" @tap="openUrl(privacy)">隐私策略</view>
					<view class="terms" @tap="openUrl(terms)">使用条款</view>
				</view>
			</match-media>

		</view>
	</view>
</template>

<script>
	import bgVip from '@/static/vip/bg-vip.png';
	const app = getApp({
		allowDefault: true
	});
	var iapChannel = null,
		isReady = false;
	export default {
		data() {
			return {
				bgVip: bgVip,
				privacy: app.globalData.privacy,
				terms: app.globalData.terms,
				productIds: app.globalData.productIds,
				orderList: []
			}
		},
		onLoad: function() {
			plus.payment.getChannels((channels) => {
				console.log("获取到channel" + JSON.stringify(channels))
				for (var i in channels) {
					var channel = channels[i];
					if (channel.id === 'appleiap') {
						iapChannel = channel;
						this.requestOrder();
					}
				}
				if (!iapChannel) {
					this.errorMsg()
				}
			}, (error) => {
				this.errorMsg()
			});
		},
		methods: {
			openUrl(url) {
				plus.runtime.openURL(url)
			},
			resumeSub() {
				uni.showModal({
					title: '提示',
					showCancel:false,
					content: "确定要恢复订阅吗？",
					success: function(res) {
						if (res.confirm) {
							uni.showLoading()
							iapChannel.restoreComplateRequest({
								"username": ""
							}, function(trancs) {
								console.log("获取已经购买项目成功：" + JSON.stringify(trancs));
								uni.showToast({
									title: '已恢复订阅'
								})
							}, function(errormsg) {
								console.log("获取支付通道失败：" + errormsg.message);
								uni.showToast({
									title: errormsg.message
								})
							});
						}
					}
				});
			},
			errorMsg() {
				uni.showToast({
					icon: 'none',
					title: '暂不支持苹果 iap 支付'
				});
			},
			requestOrder() {
				uni.showLoading({
					title: '检测支付环境...'
				})
				iapChannel.requestOrder(this.productIds, (orderList) => { //必须调用此方法才能进行 iap 支付
					isReady = true;
					this.orderList = orderList;
					console.log('requestOrder success: ' + JSON.stringify(orderList));
					uni.hideLoading();
				}, (e) => {
					isReady = false;
					console.log('requestOrder failed: ' + JSON.stringify(e));
					let message = e.message.split(',')[0] || '不支持苹果内购';
					uni.showToast({
						icon: 'none',
						title: message
					});
					uni.hideLoading();
				});
			},
			paying(productId) {
				uni.showLoading();
				console.log(productId);
				uni.requestPayment({
					provider: 'appleiap',
					orderInfo: {
						productid: productId
					},
					success: (e) => {
						uni.setStorageSync('hasSubcripted', 'hasSubcripted')
						uni.showToast({
							title: '订阅成功'
						});
					},
					fail: (e) => {
						console.log(e);
						uni.showToast({
							title: '订阅失败'
						});
					},
					complete: () => {
						uni.hideLoading();
					}
				})
			},
			requestPayment(productId) {
				if (isReady) {
					this.paying(productId);
				} else {
					setTimeout(() => {
						this.paying(productId);
					}, 1000)
				}
			},
		}
	}
</script>

<style>
	.vip-wrapper {
		height: 100%;
		text-align: center;
	}

	.bg-vip {
		position: relative;
		height: calc(100% - 90upx);
		background: url('@/static/vip/bg-vip.png') no-repeat center;
		background-size: cover;
	}

	.btn-re-sub {
		position: absolute;
		top: 40upx;
		right: 30upx;
		width: 138upx;
		height: 56upx;
		line-height: 56upx;
		box-sizing: border-box;
		background-color: #FBF7F3;
		border-radius: 30upx;
		border: 1px solid #DC6A9D;
		font-size: 24upx;
		color: #DB649A;
		cursor: pointer;
	}

	.bg-sub {
		position: absolute;
		top: 420upx;
		left: 13%;
		width: 74%;
		height: 494upx;
		background: url(../../static/vip/bg-dy.png) no-repeat center;
		background-size: cover;
		padding: 60upx 50upx;
	}

	.height667 .bg-sub {
		top: 420upx;
	}

	.height736 .bg-sub {
		top: 430upx;
	}

	.height812 .bg-sub {
		top: 540upx;
	}

	.bg-sub .sub-title {
		height: 68upx;
		font-size: 48upx;
		color: #FF8277;
		line-height: 68upx;
		text-align: center;
	}

	.bg-sub .sub-desc {
		position: relative;
		padding-top: 10upx;
		padding-left: 36upx;
		text-align: left;
		font-size: 32upx;
		color: #FF8277;
		line-height: 46upx;
	}

	.sub-desc::before {
		position: absolute;
		left: 0;
		top: 20upx;
		display: block;
		content: '';
		width: 20upx;
		height: 20upx;
		background: #EB6877;
		border: 4upx solid #fff;
		border-radius: 100%;
	}

	.sub-items {
		margin-top: 30upx;
	}

	.sub-items .sub-item {
		width: 196upx;
		padding: 20upx 0;
		background: #EB6B79;
		border-radius: 8upx;
		color: #fff;
	}

	.f12 {
		font-size: 24upx;
	}

	.sub-info {
		position: absolute;
		left: 7%;
		bottom: 12%;
		width: 86%;
		color: #FF746C;
		line-height: 32upx;
	}

	.privacy-terms {
		position: absolute;
		left: 0;
		bottom: 6%;
		width: 100%;
		font-size: 24upx;
		color: #FF746C;
		line-height: 34upx;
	}

	.terms {
		margin-left: 158upx;
	}
</style>
