<template>
	<view>
		<cu-custom bgColor="bg-gradual-blue" :isBack="false">
			<block slot="content">记录</block>
		</cu-custom>
		<view>
			<uni-calendar class="uni-calendar--hook" :clear-date="true" :date="info.date" :lunar="info.lunar" :selected="info.selected"
			 :startDate="info.startDate" :endDate="info.endDate" :range="info.range"></uni-calendar>
		</view>
	</view>
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
	import uniCalendar from '@/components/uni-calendar/uni-calendar.vue';
	export default {
		name: 'record',
		data() {
			return {
				info: {
					lunar: true,
					range: true,
					selected: []
				}
			}
		},
		components: {
			uniCalendar
		},
		methods: {
			getFullDateStr(dayCount = 0) {
				return getDate(new Date(), dayCount).fullDate
			}
		},
		onShow() {
			let todayStr = getDate(new Date()).fullDate;
			this.info.date = todayStr;
			this.info.startDate = getDate(new Date(), -30).fullDate;
			this.info.endDate = todayStr;

			let records = uni.getStorageSync('records');
			if (records) {
				this.info.selected = records;
			}else{
				this.info.selected = [];
			}
			console.log('record onShow this.Info = ',this.info)
		}
	}
</script>

<style>
</style>
