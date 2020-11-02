<template>
	<scroll-view scroll-y="true" class="page show">
		<cu-custom bgColor="bg-gradual-blue" :isBack="false">
			<block slot="content">首页</block>
		</cu-custom>
		<view class="home-top flex flex-wrap align-center">
			<view class="avatar-wrap padding-xl margin-left">
				<view class="cu-avatar xl round" style="background-image:url(../../static/logo.png);"></view>
				<view class="avatar-text">{{customName}}</view>
			</view>
			<view class="progress flex flex-sub align-center justify-end padding-right-xl">
				<text>当前完成：</text>
				<view>{{donePercent}}%</view>
			</view>
		</view>
		<view class="cu-list grid" :class="['col-' + gridCol,gridBorder?'':'no-border']">
			<view class="cu-item" v-for="(item,index) in cuIconList" :key="index" v-if="index<gridCol*10" @tap="goModifyClock(index)">
				<view :class="['cuIcon-' + item.cuIcon,'text-' + item.color]">
					<view class="cu-tag badge" v-if="item.badge!=0">
						<block v-if="item.badge!=1">{{item.badge>99?'99+':item.badge}}</block>
					</view>
				</view>
				<text>{{item.name}}</text>
			</view>
		</view>
		<view class="cu-list setted-time-list">
			<view class="time-item flex justify-between align-center" v-for="(item,index) in settedTimeList" :key="index">
				<view :class="item.done?'done':''">{{item.name+'时间 '}}</view>
				<view :class="item.done?'done':''">{{item.time}}</view>
				<view v-show="item.done!=true" class="cu-btn bg-green" @tap="setItemDone(index)">完成</view>
			</view>
		</view>
	</scroll-view>
</template>

<script>
	/**
	 * 获取任意时间
	 */
	function getDate(date, AddDayCount = 0) {
		if (!date) {
			date = new Date()
		}
		if (typeof date !== 'object') {
			date = date.replace(/-/g, '/')
		}
		const dd = new Date(date)

		dd.setDate(dd.getDate() + AddDayCount) // 获取AddDayCount天后的日期

		const y = dd.getFullYear()
		const m = dd.getMonth() + 1 < 10 ? '0' + (dd.getMonth() + 1) : dd.getMonth() + 1 // 获取当前月份的日期，不足10补0
		const d = dd.getDate() < 10 ? '0' + dd.getDate() : dd.getDate() // 获取当前几号，不足10补0
		return {
			fullDate: y + '-' + m + '-' + d,
			year: y,
			month: m,
			date: d,
			day: dd.getDay()
		}
	}

	function unique(arr) {
		for (var i = 0; i < arr.length; i++) {
			for (var j = i + 1; j < arr.length; j++) {
				if (arr[i].name == arr[j].name && arr[i].time == arr[j].time) {
					arr.splice(j, 1);
					j--;
				}
			}
		}
		return arr;
	}
	function getUniqueRecord(arr){
		for (var i = 0; i < arr.length; i++) {
			for (var j = i + 1; j < arr.length; j++) {
				if (arr[i].date == arr[j].date) {
					arr.splice(i, 1);
					j--;
				}
			}
		}
		return arr;
	}
	export default {
		data() {
			return {
				customName: '美容时钟',
				gridCol: 3,
				gridBorder: true,
				cuIconList: [],
				settedTimeList: [],
				donePercent: 0
			}
		},
		methods: {
			goModifyClock(index) {
				console.log(index)
				uni.navigateTo({
					url: '/pages/addClock/index?index=' + index
				});
			},
			initData() {
				try {
					let item_list = uni.getStorageSync('item_list');
					if (item_list) {
						this.cuIconList = item_list;
					} else {
						const app = getApp({
							allowDefault: true
						});
						this.cuIconList = app.globalData.itemList;
					}
					let alarmList = uni.getStorageSync('alarmList') || [];
					console.log('alarmList = ',alarmList);
					let settedTimeList = [];
					alarmList.forEach((alarm, index) => {
						if (alarm && alarm.itemName && (alarm.itemName == this.cuIconList[index].name)) {
							alarm.timeList.forEach((time, i) => {
								let settedItem = {
									name: alarm.itemName,
									time: time.date + ' ' + time.time,
									done: time.done
								}
								settedTimeList.push(settedItem)
							})
						}
					});
					let storagedSettedTimeList = uni.getStorageSync('settedTimeList') || [];
					let arr = storagedSettedTimeList.concat(settedTimeList);
					console.log('home onShow arr = ', arr);
					let uniqueArr = unique(arr);
					let todayStr = getDate().fullDate;
					this.settedTimeList = uniqueArr.filter((item) => {
						return item.time.indexOf(todayStr) > -1
					})
					console.log('home onShow this.settedTimeList = ', this.settedTimeList);
					this.setPercent();
				} catch (e) {
					console.log(e);
				}
			},
			setItemDone(index) {
				this.settedTimeList[index].done = true;
				this.setPercent();
			},
			setPercent() {
				let doneCount = 0;
				this.settedTimeList.forEach((item) => {
					if (item.done) {
						doneCount++;
					}
				});
				this.donePercent = parseInt((doneCount / this.settedTimeList.length) * 100) || 0;
				uni.setStorageSync('settedTimeList', this.settedTimeList);
				let todayStr = getDate().fullDate;
				let todayDoneObj = {
					date: todayStr,
					info: '',
					status: null,
				};
				let done_settings = uni.getStorageSync('done_settings');
				let doneValue = done_settings.doneValue || 90;
				let unDoneValue = done_settings.unDoneValue || 60;
				if (this.donePercent > doneValue) {
					todayDoneObj.status = 0;
					todayDoneObj.info = '已完成';
				} else if (this.donePercent < unDoneValue) {
					todayDoneObj.status = 2;
					todayDoneObj.info = '未完成';
				} else {
					todayDoneObj.status = 1;
					todayDoneObj.info = '基本完成';
				}
				let todayRecord = [todayDoneObj];
				console.log("todayRecord = ", todayRecord)
				let records = uni.getStorageSync('records') || [];
				console.log("records = ", records)
				let newRecords = [];
				
				if(records.length == 0){
					newRecords = todayRecord;
				}else{
					newRecords = getUniqueRecord(records.concat(todayRecord));
				}
				console.log("newRecords = ", newRecords)
				uni.setStorageSync('records', newRecords);
			}
		},
		onShow() {
			this.initData();
		}
	}
</script>

<style>
	.avatar-text {
		padding-top: 16upx;
		text-align: center;
	}

	.progress {
		padding-right: 40upx;
	}

	.cu-list {
		border-top: 2upx solid rgba(0, 0, 0, .1);
	}

	.setted-time-list {
		padding: 48upx 40upx 200upx;
	}

	.setted-time-list .time-item {
		position: relative;
		padding: 12upx 0;
	}

	.done {
		position: relative;
		color: #999;
	}

	.done::after {
		display: block;
		position: absolute;
		top: 19upx;
		left: 0;
		content: '';
		width: 100%;
		height: 2upx;
		background-color: #999;
	}
</style>
