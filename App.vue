<script>
	import Vue from 'vue'
	export default {
		onLaunch: function() {
			uni.getSystemInfo({
				success: function(e) {
					// #ifndef MP
					Vue.prototype.StatusBar = e.statusBarHeight;
					if (e.platform == 'android') {
						Vue.prototype.CustomBar = e.statusBarHeight + 50;
					} else {
						Vue.prototype.CustomBar = e.statusBarHeight + 45;
					};
					// #endif
				}
			})
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		},
		globalData: {
			productIds: ['weekSub', 'yearSub'],
			flag: false,
			privacy: 'https://xiaogangdou.github.io/privacy.html',
			terms: 'https://xiaogangdou.github.io/terms.html',
			itemList: [{
				cuIcon: 'cardboardfill',
				color: 'red',
				badge: 0,
				name: '喝水'
			}, {
				cuIcon: 'recordfill',
				color: 'orange',
				badge: 0,
				name: '水果'
			}, {
				cuIcon: 'picfill',
				color: 'yellow',
				badge: 0,
				name: '护肤'
			}, {
				cuIcon: 'noticefill',
				color: 'olive',
				badge: 0,
				name: '健身'
			}, {
				cuIcon: 'upstagefill',
				color: 'cyan',
				badge: 0,
				name: '美容觉'
			}, {
				cuIcon: 'clothesfill',
				color: 'blue',
				badge: 0,
				name: '减肥餐'
			}]
		}

	}
</script>

<style>
	@import "colorui/main.css";
	@import "colorui/icon.css";

	page {
		height: 100%;
		font-size: 32rpx;
		line-height: 1.6;
	}

	.page-body {
		padding: 20rpx 0;
	}

</style>
