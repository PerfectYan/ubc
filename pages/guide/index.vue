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
					<view class="guide-desc">
						<image style="width: 100%;height: 100%;" mode="scaleToFill" :src="desc3"></image>
					</view>
					<view class="fixed">
						<view @click="requestPayment" class="open-btn  slide-btn">立即开启</view>
					</view>
				</view>
			</swiper-item>
		</swiper>
	</view>
</template>
<script>
	const SystemInfo = uni.getSystemInfoSync();
	import bg1 from '@/static/guide/bg-1.png';
	import bg2 from '@/static/guide/bg-2.png';
	import bg3 from '@/static/guide/bg-3.png';
	import title1 from '@/static/guide/title-1.png';
	import desc1 from '@/static/guide/desc-1.png';
	import title2 from '@/static/guide/title-2.png';
	import desc2 from '@/static/guide/desc-2.png';
	import title3 from '@/static/guide/title-3.png';
	import desc3 from '@/static/guide/desc-3.png';
	let iapChannel = null,
		homePath = '/pages/home/index',
		isReady = false;
	export default {
		data() {
			return {
				productIds: getApp().globalData.authsub || [],
				privacy: getApp().globalData.privacy,
				terms: getApp().globalData.terms,
				price: '',
				flag: getApp().globalData.flag,
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
				screenHeight: SystemInfo.screenHeight
			};
		},
		onLoad: function() {
			console.log(this.screenHeight);
		},
		methods: {
			closeHandle() {
				uni.showModal({
					title: '温馨提示',
					content: '三天内免费试用哦，确定要关闭吗？',
					confirmText: '免费试用',
					cancelText: '狠心放弃',
					success: (showResult) => {
						if (showResult.confirm) {
							this.paying();
						} else {
							uni.redirectTo({
								url: homePath
							});
						}
					}
				})
			},
			requestOrder() {
				iapChannel.requestOrder(this.productIds, (orderList) => { //必须调用此方法才能进行 iap 支付
					isReady = true;
					// console.log('requestOrder success666: ' + JSON.stringify(orderList));
				}, (e) => {
					// console.log('requestOrder failed: ' + JSON.stringify(e));
					this.errorMsg()
				});
			},
			paying() {
				uni.showLoading();
				uni.requestPayment({
					provider: 'appleiap',
					orderInfo: {
						productid: this.productIds[1] //默认第二个设置
					},
					success: (e) => {
						// 订阅成功，下次不显示启动图
						uni.setStorageSync('hasSubcripted', 'hasSubcripted')
						uni.setStorageSync('launch', 'launch');
						setTimeout(() => {
							uni.redirectTo({
								url: homePath
							});
						}, 2000)
					},
					fail: (e) => {
						console.log(e);
						uni.hideLoading();
					},
					complete: () => {}
				})
			},
			requestPayment() {
				if (this.flag) {
					this.paying();
				} else {
					uni.switchTab({
						url: homePath
					});
				}
			},
			errorMsg() {
				uni.showToast({
					icon: 'none',
					title: 'iap is  not  supported'
				});
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
<style lang="scss" scoped>
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
		bottom: 40rpx;
	}

	.fixed-wrap {
		position: fixed;
		left: 0;
		width: 100%;
		height: 150rpx;
		overflow: auto;
		bottom: 10rpx;
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
		font-size: 26rpx;
		margin: 20rpx 0 5rpx;
		text-align: center;
	}

	.des {
		color: #CCCCCC;
		opacity: 0.3;
		width: 60%;
		margin: 10rpx auto;
		text-align: left;
		font-size: 24rpx;
		line-height: 34rpx;
	}

	.link-wrap {
		display: flex;
		font-size: 30rpx;
		color: #ccc;
		margin-top: 10rpx;
		justify-content: center;
	}

	.link {
		margin: 0 50rpx;
		font-size: 24rpx;
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
		right: 40rpx;
		/* #ifndef MP */
		top: 80rpx;
		/* #endif */
		/* #ifdef MP */
		top: 150rpx;
		/* #endif */
	}

	.swiper-to-home-text {
		color: #ffffff;
		text-align: center;
		background-color: rgba(0, 0, 0, 0.5);
		border-width: 1rpx;
		border-color: #ffffff;
		border-style: solid;
		border-radius: 30rpx;
		font-size: 28rpx;
		padding-top: 5rpx;
		padding-bottom: 5rpx;
		padding-left: 25rpx;
		padding-right: 25rpx;
	}

	.indicator {
		width: 714rpx;
		height: 30rpx;
		position: absolute;
		bottom: 50rpx;
		left: 1rpx;
		item-color: #f6f6f6;
		item-selected-color: #fd575c;
		item-size: 20rpx;
	}

	.close {
		opacity: 0.6;
		position: fixed;
		left: 30rpx;
		width: 40rpx;
		height: 40rpx;
		top: 50rpx;
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
</style>
