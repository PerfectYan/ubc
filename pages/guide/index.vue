<template>
	<view class="guide">
		<swiper :current="tabIndex" class="flex1" duration="300" easing-function="easeInCubic" :show-indicators="false"
		 :auto-play="true" @change="sliderChange" :infinite="false" :forbid-slide-animation="false">
			<swiper-item class="flex1">
				<view :style="{backgroundImage:`url(${bg1})`, backgroundSize: 'cover'}" class="flex1 item">
					<view class="guide-title">
						<image style="width: 100%;height: 100%;" mode="scaleToFill" :src="title1"></image>
					</view>
					<view class="guide-desc">
						<image style="width: 100%;height: 100%;" mode="scaleToFill" :src="desc1"></image>
					</view>
					<view class="fixed">
						<view @click="tabIndex=1" class="open-btn  slide-btn">下一步</view>
					</view>
				</view>
			</swiper-item>
			<swiper-item class="flex1">
				<view :style="{backgroundImage:`url(${bg2})`, backgroundSize: 'cover'}" class="flex1 item">
					<view class="guide-title">
						<image style="width: 100%;height: 100%;" mode="scaleToFill" :src="title2"></image>
					</view>
					<view class="guide-desc">
						<image style="width: 100%;height: 100%;" mode="scaleToFill" :src="desc2"></image>
					</view>
					<view class="fixed">
						<view @click="tabIndex=2" class="open-btn  slide-btn">下一步</view>
					</view>
				</view>
			</swiper-item>

			<swiper-item class="flex1">
				<view :style="{backgroundImage:`url(${bg3})`, backgroundSize: 'cover'}" class="flex1 item">
					<view class="guide-title">
						<image style="width: 100%;height: 100%;" mode="scaleToFill" :src="title3"></image>
					</view>
					<view class="guide-desc guide-desc3">
						<image style="width: 100%;height: 100%;" mode="scaleToFill" :src="desc3"></image>
					</view>
					<view class="guide-desc3-btn">
						<view @click="requestPayment" class="open-btn  slide-btn">立即开启</view>
					</view>
					<view class="price">以{{orderList[0].price}}/周的价格收取费用</view>
					<view class="sub-info f12">
						用户确认购买并付款后计入iTunes账户，除非您在到期日24小时前取消，否则我们将在您会员过期前24小时进行自动进行续费（含免费试用），扣费成功后订阅周期顺延一个订阅周期。
					</view>
					<view class="privacy-terms flex justify-center align-center">
						<view class="privacy" @tap="openUrl(privacy)">隐私策略</view>
						<view class="terms" @tap="openUrl(terms)">使用条款</view>
					</view>
				</view>
			</swiper-item>
		</swiper>
	</view>
</template>
<script>
	import bg1 from '@/static/guide/bg-1.png';
	import bg2 from '@/static/guide/bg-2.png';
	import bg3 from '@/static/guide/bg-3.png';
	import title1 from '@/static/guide/title-1.png';
	import desc1 from '@/static/guide/desc-1.png';
	import title2 from '@/static/guide/title-2.png';
	import desc2 from '@/static/guide/desc-2.png';
	import title3 from '@/static/guide/title-3.png';
	import desc3 from '@/static/guide/desc-3.png';

	const app = getApp({
		allowDefault: true
	});
	var iapChannel = null,
		isReady = false,
		homePath = '/pages/home/index';
	export default {
		data() {
			return {
				orderList: [{price: 0}],
				productIds: app.globalData.productIds,
				privacy: app.globalData.privacy,
				terms: app.globalData.terms,
				bg1: bg1,
				bg2: bg2,
				bg3: bg3,
				title1: title1,
				desc1: desc1,
				title2: title2,
				desc2: desc2,
				title3: title3,
				desc3: desc3,
				currIndex: 0,
				tabIndex: 0, // 选中的
			};
		},
		onLoad: function() {
			// plus.payment.getChannels((channels) => {
			// 	console.log("获取到channel" + JSON.stringify(channels))
			// 	for (var i in channels) {
			// 		var channel = channels[i];
			// 		if (channel.id === 'appleiap') {
			// 			iapChannel = channel;
			// 			this.requestOrder();
			// 		}
			// 	}
			// 	if (!iapChannel) {
			// 		this.errorMsg()
			// 	}
			// }, (error) => {
			// 	this.errorMsg()
			// });
		},
		methods: {
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
					for (var index in orderList) {
						var OrderItem = orderList[index];
						console.log("Title:" + OrderItem.title + "Price:" + OrderItem.price + "Description:" + OrderItem.description +
							"ProductID:" + OrderItem.productid);
					}
					uni.hideLoading();
				}, (e) => {
					isReady = false;
					console.log('requestOrder failed: ' + JSON.stringify(e));
					let message = e.message.split(',')[0] || '暂不支持苹果 iap 支付';
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
						uni.navigateTo({
							url: '../home/index'
						})
					},
					fail: (e) => {
						console.log(e);
						uni.navigateTo({
							url: '../home/index'
						})
					},
					complete: () => {
						uni.hideLoading();
					}
				})
			},
			requestPayment() {
				let productId = this.orderList[0].productid;
				if (isReady) {
					this.paying(productId);
				} else {
					setTimeout(() => {
						this.paying(productId);
					}, 1000)
				}
			},
			openUrl(url) {
				plus.runtime.openURL(url)
			},
			sliderChange(e) {
				this.currIndex = e.detail.current;
			}
		}
	};
</script>
<style scoped>
	.guide {
		height: 100%;
	}

	.flex1 {
		height: 100%;
		flex: 1;
		position: relative;
		text-align: center;
	}

	.text {
		color: #fff;
		font-size: 30rpx;
		line-height: 90rpx;
	}

	.fixed {
		position: fixed;
		left: 0;
		width: 100%;
		text-align: center;
		bottom: 40upx;
	}

	.fixed-wrap {
		position: fixed;
		left: 0;
		width: 100%;
		height: 150upx;
		overflow: auto;
		bottom: 10upx;
	}

	.open-btn {
		width: 75%;
		margin: 60upx auto 0;
		height: 100upx;
		line-height: 100upx;
		text-align: center;
		background: #3C6BAE;
		color: #fff;
		font-size: 36upx;
		font-weight: bold;
		border-radius: 50upx;
	}

	.info {
		color: #fff;
		font-size: 26upx;
		margin: 20upx 0 5upx;
		text-align: center;
	}

	.des {
		color: #CCCCCC;
		opacity: 0.3;
		width: 60%;
		margin: 10upx auto;
		text-align: left;
		font-size: 24upx;
		line-height: 34upx;
	}

	.link-wrap {
		display: flex;
		font-size: 30upx;
		color: #ccc;
		margin-top: 10upx;
		justify-content: center;
	}

	.link {
		margin: 0 50upx;
		font-size: 24upx;
	}


	.apptestnnnn {
		border-width: 1px;
		border-color: #4cd964;
		border-style: solid;
	}

	.apptest {
		border-width: 1px;
		border-color: #007aff;
		border-style: solid;
	}

	.swiper-to-home {
		position: absolute;
		z-index: 999;
		right: 40upx;
		/* #ifndef MP */
		top: 80upx;
		/* #endif */
		/* #ifdef MP */
		top: 150upx;
		/* #endif */
	}

	.swiper-to-home-text {
		color: #ffffff;
		text-align: center;
		background-color: rgba(0, 0, 0, 0.5);
		border-width: 1upx;
		border-color: #ffffff;
		border-style: solid;
		border-radius: 30upx;
		font-size: 28upx;
		padding-top: 5upx;
		padding-bottom: 5upx;
		padding-left: 25upx;
		padding-right: 25upx;
	}

	.indicator {
		width: 714upx;
		height: 30upx;
		position: absolute;
		bottom: 50upx;
		left: 1upx;
		item-color: #f6f6f6;
		item-selected-color: #fd575c;
		item-size: 20upx;
	}

	.close {
		opacity: 0.6;
		position: fixed;
		left: 30upx;
		width: 40upx;
		height: 40upx;
		top: 50upx;
		background: url('~@/static/guide/close.png') no-repeat center;
		background-size: cover;
	}

	.guide-title {
		position: absolute;
		top: 60upx;
		right: 0;
		width: 655upx;
		height: 305upx;
		text-align: right;
	}

	.guide-desc {
		position: absolute;
		top: 390upx;
		left: 12.5%;
		width: 75%;
		height: 742upx;
	}
	
	.guide-desc.guide-desc3{
		height: 558upx;
	}
	
	.guide-desc3-btn{
		position: absolute;
		width: 100%;
		left: 0;
		top: 920upx;
	}
	
	.price{
		position: absolute;
		width: 100%;
		left: 0;
		top: 1090upx;
		font-size: 32upx;
		color: #fff;
		line-height: 46upx;
		text-align: center;
	}
	
	.sub-info {
		position: absolute;
		left: 7%;
		bottom: 5%;
		width: 86%;
		color: #fff;
		line-height: 32upx;
	}
	
	.privacy-terms {
		position: absolute;
		left: 0;
		bottom: 2%;
		width: 100%;
		font-size: 24upx;
		color: #fff;
		line-height: 34upx;
	}
	
	.terms {
		margin-left: 158upx;
	}
	.f12 {
		font-size: 24upx;
	}
	
</style>
