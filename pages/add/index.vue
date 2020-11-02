<template>
	<view>
		<cu-custom bgColor="bg-gradual-blue" :isBack="false">
			<block slot="content">添加项目</block>
		</cu-custom>
		<view class="cu-form-group padding-top-xl padding-bottom-xl">
			<view class="title">项目名称：</view>
			<input placeholder="请填写项目名称" v-model="itemName" type="text" maxlength="10"></input>
		</view>
		<view class="cu-form-group btn-wrapper justify-center">
			<view class="action">
				<button class="cu-btn bg-green shadow" @tap="addItem">添加</button>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				itemName: ''
			}
		},
		methods: {
			addItem() {
				if (this.itemName == '') {
					uni.showToast({
						icon: 'none',
						title: '请填写项目名称'
					})
					return;
				}
				let hasSubcripted = uni.getStorageSync('hasSubcripted') || "";
				if (hasSubcripted != 'hasSubcripted') {
					//没有订阅成功则跳转至vip页面
					uni.showModal({
						title: '温馨提示',
						content: "抱歉，您不是会员，不能添加项目",
						confirmText: '去购买',
						success: function(res) {
							if (res.confirm) {
								uni.navigateTo({
									url: '../vip/index'
								});
							}
						}
					});

					return;
				}
				try {
					let item_list = null;
					const app = getApp({
						allowDefault: true
					});
					item_list = uni.getStorageSync('item_list') || app.globalData.itemList;
					console.log('add item_list = ', item_list);
					let newItem = {
						cuIcon: 'picfill',
						color: 'yellow',
						badge: 0,
						name: this.itemName
					};
					item_list.push(newItem)
					uni.setStorageSync('item_list', item_list)
					console.log(item_list)
					this.itemName = '';
					uni.showToast({
						title: '添加成功',
						duration: 2000
					});
				} catch (e) {
					// error
				}
			}
		}
	}
</script>

<style>
	.btn-wrapper .cu-btn {
		margin-top: 600upx;
		width: 300upx;
		height: 80upx;
		line-height: 80upx;
		font-size: 32upx;
	}
</style>
