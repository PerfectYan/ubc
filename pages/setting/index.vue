<template>
	<view>
		<cu-custom bgColor="bg-gradual-blue" :isBack="false">
			<block slot="content">设置</block>
		</cu-custom>
		<view class="cu-bar bg-white solid-bottom margin-top">
			<view class="action">
				<text class="cuIcon-title text-blue"></text>音量设置
			</view>
		</view>
		<view class="uni-padding-wrap uni-common-mt">
			<slider :value="volume" :max="maxVolume" @change="sliderChange" show-value activeColor="#0081ff" backgroundColor="#000000"
			 block-color="#8A6DE9" block-size="20" />
		</view>
		<view class="cu-bar bg-white solid-bottom margin-top">
			<view class="action">
				<text class="cuIcon-title text-blue"></text>完成度设置
			</view>
		</view>
		<view class="cu-form-group flex justify-between align-center">
			<view class="action col-1">完成: 大于</view>
			<view class="col-3 flex-sub flex align-center">
				<uni-number-box :min="minValue" :max="maxValue" :value="doneValue" @change="doneValueChange" />
			</view>
		</view>
		<view class="cu-form-group flex justify-between align-center">
			<view class="action col-1">基本完成: </view>
			<view class="col-3 flex-sub flex align-center">
				<uni-number-box :min="minValue2" :max="maxValue2" :value="almostDoneMinValue" @change="almostDoneMinValueChange" />
				<view class="split">至</view>
				<uni-number-box :min="minValue3" :max="maxValue3" :value="almostDoneMaxValue" @change="almostDoneMaxValueChange" />
			</view>
		</view>
		<view class="cu-form-group flex justify-between align-center">
			<view class="action col-1">未完成: 小于</view>
			<view class="col-3 flex-sub flex align-center">
				<uni-number-box :min="minValue4" :max="maxValue4" :value="unDoneValue" @change="unDoneValueChange" />
			</view>
		</view>
		<view class="cu-form-group">
			<view class="cu-btn bg-green" @tap="goVipPage">会员设置</view>
		</view>
	</view>
</template>

<script>
	import uniNumberBox from '@/components/uni-number-box/uni-number-box.vue';
	export default {
		data() {
			return {
				volume: null,
				maxVolume: 6,
				minValue: 80,
				maxValue: 100,
				minValue2: null,
				maxValue2: null,
				minValue3: null,
				maxValue3: null,
				minValue4: 1,
				maxValue4: null,
				doneValue: null,
				almostDoneMinValue: null,
				almostDoneMaxValue: null,
				unDoneValue: null
			}
		},
		components: {
			uniNumberBox
		},
		methods: {
			sliderChange(e) {
				uni.setStorageSync('volume_settings', e.detail.value)
				// #ifdef  APP-PLUS
				plus.device.setVolume(e.detail.value / 6);
				// #endif
			},
			doneValueChange(val) {
				val = parseInt(val);
				this.maxValue3 = val;
				this.almostDoneMaxValue = val;
				this.setDoneSettings(val,this.unDoneValue);
			},
			almostDoneMaxValueChange(val) {
				val = parseInt(val);
				this.maxValue2 = val;
				this.doneValue = val;
			},
			almostDoneMinValueChange(val) {
				val = parseInt(val);
				this.maxValue4 = val;
				this.unDoneValue = val;
			},
			unDoneValueChange(val) {
				val = parseInt(val);
				this.minValue2 = val;
				this.almostDoneMinValue = val;
				this.setDoneSettings(this.doneValue,val);
			},
			setDoneSettings(doneValue,unDoneValue) {
				let doneSettings = {
					doneValue: doneValue,
					unDoneValue: unDoneValue,
				}
				uni.setStorageSync('done_settings',doneSettings);
			},
			goVipPage() {
				uni.navigateTo({
					url: '../vip/index'
				})
			}
		},
		onShow() {
			let volume = uni.getStorageSync('volume_settings');
			if (volume) {
				this.volume = volume;
			} else {
				this.volume = 3;
			}
			let doneSettings = uni.getStorageSync('done_settings');
			if(doneSettings){
				this.doneValue = doneSettings.doneValue;
				this.unDoneValue = doneSettings.unDoneValue;
			}else{
				this.doneValue = 90;
				this.unDoneValue = 60;
			}
		}
	}
</script>

<style>
	.cu-form-group {
		padding: 2upx 14upx;
	}

	.cu-form-group .action {
		flex: 1;
	}

	.cu-form-group .flex-sub {
		flex: 3;
	}

	.cu-form-group .split {
		margin: 0 20upx;
	}
</style>
