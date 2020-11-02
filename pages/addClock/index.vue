<template>
	<view>
		<cu-custom bgColor="bg-gradual-blue" :isBack="true">
			<block slot="backText">返回</block>
			<block slot="content">{{itemName}}</block>
		</cu-custom>
		<view>
			<view class="cu-form-group" v-for="(timeItem,index) in timeList" :key="index" @tap="tapTimeItem(index)">
				<view class="title">时间选择</view>
				<view class="date-ctn">
					<picker mode="date" :value="timeItem.date" :start="startDate" :end="endDate" @change="dateChange">
						<view class="uni-input">{{timeItem.date}}</view>
					</picker>
				</view>
				<view class="time-ctn">
					<picker mode="time" :value="timeItem.time" @change="timeChange">
						<view class="picker">
							{{timeItem.time}}
						</view>
					</picker>
				</view>
				<view class="btn-del">
					<button class="cu-btn bg-green" @tap="removeTimeItem(index)">-</button>
				</view>
			</view>

			<view class="cu-form-group switch-wrap">
				<view class="title">是否提醒</view>
				<switch @change="switchClock" :class="alarmStatus?'checked':''" :checked="alarmStatus?true:false"></switch>
			</view>
			<view class="cu-form-group action-btns justify-center">
				<view class="action">
					<view class="cu-btn btn-add bg-green" @tap="addTimeList">添加时间</view>
					<view class="cu-btn btn-save bg-green" @tap="saveItem">保存</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	let utils = uni.requireNativePlugin('AD-Utils');

	function getDate(type) {
		const date = new Date();

		let year = date.getFullYear();
		let month = date.getMonth() + 1;
		let day = date.getDate();

		if (type === 'start') {
			year = year - 1;
		} else if (type === 'end') {
			year = year + 2;
		}
		month = month > 9 ? month : '0' + month;;
		day = day > 9 ? day : '0' + day;

		return `${year}-${month}-${day}`;
	}
	export default {
		data() {
			return {
				index: 0,
				itemName: '',
				startDate: getDate('start'),
				endDate: getDate('end'),
				weeks: [{
						value: '1',
						name: '周一'
					},
					{
						value: '2',
						name: '周二',
					},
					{
						value: '3',
						name: '周三'
					},
					{
						value: '4',
						name: '周四'
					},
					{
						value: '5',
						name: '周五'
					},
					{
						value: '6',
						name: '周六'
					},
					{
						value: '7',
						name: '周日'
					}
				],
				timeList: [],
				timeIndex: 0,
				alarmStatus: true,
				alarmList: []
			}
		},
		methods: {
			getItemName(index) {
				let item_list = uni.getStorageSync('item_list');
				if (item_list) {
					return item_list[index].name;
				} else {
					const app = getApp({
						allowDefault: true
					});
					return app.globalData.itemList[index].name;
				}
			},
			tapTimeItem(index) {
				uni.showToast({
					title: index
				})
				this.timeIndex = index;
			},
			dateChange(e) {
				console.log(e.detail.value)
				this.timeList[this.timeIndex].date = e.detail.value;
			},
			timeChange(e) {
				this.timeList[this.timeIndex].time = e.detail.value;
			},
			addTimeList() {
				let newTimeItem = {
					date: getDate({
						format: true
					}),
					time: '08:00',
					done: false
				}
				this.timeList.push(newTimeItem);
				console.log(this.timeList)
			},
			removeTimeItem(index) {
				this.timeList.splice(index, 1);
			},
			checkboxChange(e) {
				console.log('checkboxChange = ', e.detail.value);
				var weeks = this.weeks,
					values = e.detail.value;
				for (var i = 0, lenI = weeks.length; i < lenI; ++i) {
					const day = weeks[i]
					if (values.includes(day.value)) {
						this.$set(day, 'checked', true)
					} else {
						this.$set(day, 'checked', false)
					}
				}
			},
			switchClock(e) {
				console.log('switchClock = ', e.detail.value);
				this.alarmStatus = e.detail.value
			},
			saveItem() {
				if (this.timeList.length == 0) {
					uni.showToast({
						icon: 'none',
						title: '请先添加时间'
					});
					return;
				}
				let hasSubcripted = uni.getStorageSync('hasSubcripted') || "";
				if (hasSubcripted != 'hasSubcripted') {
					//没有订阅成功则跳转至vip页面
					if (this.timeList.length > 3) {
						this.goVipPage();
						return;
					} else {
						//所有timeList是否大于3
						let alarmLength = this.timeList.length;
						this.alarmList.forEach(alarmItem => {
							alarmItem.timeList.forEach(time => {
								alarmLength++;
							})
						})
						if (alarmLength > 3) {
							this.goVipPage();
							return;
						}
					}
				}
				if (this.alarmStatus) {
					this.saveCalendarBatch();
				} else {
					this.saveData();
					uni.showToast({
						title: '保存成功'
					});
				}
			},
			goVipPage() {
				uni.showModal({
					title: '温馨提示',
					content: '抱歉，您不是会员用户，不能添加超过3条提醒',
					confirmText: '去购买',
					success: function(res) {
						if (res.confirm) {
							uni.navigateTo({
								url: '../vip/index'
							});
						} else if (res.cancel) {
							console.log('用户点击取消');
						}
					}
				});
			},
			saveData() {
				let itemObject = {
					itemName: this.itemName,
					timeList: this.timeList,
					weeks: this.weeks,
					alarmStatus: this.alarmStatus
				};
				this.alarmList[this.index] = itemObject;
				console.log('saveData alarmList = ', this.alarmList);
				uni.setStorageSync('alarmList', this.alarmList);
			},
			saveCalendarBatch() {
				// 添加日历
				let title = this.itemName + "时间到了，请记得哦～";
				let todayStr = getDate();
				let notices = [];
				let notice = {};
				this.timeList.forEach((item, index) => {
					let alarmDate = item.date + " " + item.time + ":00";
					notice = {
						title: title,
						alarmDate: alarmDate,
						notes: ''
					};
					notices.push(notice);
				})
				utils.saveCalendarBatch(notices, result => {
					if (result.ok) {
						uni.showToast({
							title: '设置提醒成功'
						});
					}
					this.saveData();
				})
			}
		},
		onLoad(option) {
			console.log(option.index); //打印出上个页面传递的参数。
			this.index = option.index;
		},
		onShow() {
			this.itemName = this.getItemName(this.index);
			uni.getStorage({
				key: 'alarmList',
				success: (res) => {
					this.alarmList = res.data;
					console.log('onShow alarmList = ', this.alarmList);
					let itemAlarms = this.alarmList[this.index];
					if (itemAlarms) {
						this.timeList = itemAlarms.timeList;
						this.weeks = itemAlarms.weeks;
						this.alarmStatus = itemAlarms.alarmStatus;
					}
				}
			});
		}
	}
</script>

<style>
	.btn-del {
		margin-left: 20upx;
	}

	.action-btns {
		padding-top: 200upx;
		padding-bottom: 100upx;
	}

	.action-btns .cu-btn {
		height: 80upx;
		line-height: 80upx;
		font-size: 32upx;
	}

	.btn-add {
		margin-right: 40upx;
	}

	.btn-save {
		width: 240upx;
		width: 300upx;
	}

	.uni-list-cell {
		text-align: center;
	}

	.checkbox-wrap {
		padding-bottom: 20upx;
	}

	.checkbox-wrap .name {
		padding-top: 8upx;
	}

	.switch-wrap {
		border-top: 2upx solid rgba(0, 0, 0, 0.1);
	}

	.date-ctn uni-picker::after {
		display: none;
	}
</style>
