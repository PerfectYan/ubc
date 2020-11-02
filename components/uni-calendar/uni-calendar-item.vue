<template>
	<view class="uni-calendar-item__weeks-box" :class="{
		'uni-calendar-item--disable':weeks.disable,
		}">
		<view class="uni-calendar-item__weeks-box-item">
			<text v-if="selected&&weeks.extraInfo" class="uni-calendar-item__weeks-box-circle" :class="{
				'done': weeks.extraInfo.status && weeks.extraInfo.status == 0,
				'almost-done': weeks.extraInfo.status && weeks.extraInfo.status == 1,
				'undone': weeks.extraInfo.status && weeks.extraInfo.status == 2,
			}"></text>
			<text class="uni-calendar-item__weeks-box-text" :class="{
				'done': weeks.extraInfo && weeks.extraInfo.status && weeks.extraInfo.status == 0,
				'almost-done': weeks.extraInfo && weeks.extraInfo.status && weeks.extraInfo.status == 1,
				'undone': weeks.extraInfo && weeks.extraInfo.status && weeks.extraInfo.status == 2,
				'uni-calendar-item--disable':weeks.disable,
				}">{{weeks.date}}</text>
			<text v-if="!lunar&&!weeks.extraInfo && weeks.isDay" class="uni-calendar-item__weeks-lunar-text" :class="{
				}">今天</text>
			<text v-if="lunar&&!weeks.extraInfo" class="uni-calendar-item__weeks-lunar-text" :class="{
				'uni-calendar-item--disable':weeks.disable,
				}">{{weeks.isDay?'今天': (weeks.lunar.IDayCn === '初一'?weeks.lunar.IMonthCn:weeks.lunar.IDayCn)}}</text>
			<text v-if="weeks.extraInfo&&weeks.extraInfo.info" class="uni-calendar-item__weeks-lunar-text" :class="{
				'uni-calendar-item--extra':weeks.extraInfo.info,
				'done': weeks.extraInfo.status && weeks.extraInfo.status == 0,
				'almost-done': weeks.extraInfo.status && weeks.extraInfo.status == 1,
				'undone': weeks.extraInfo.status && weeks.extraInfo.status == 2,
				'uni-calendar-item--disable':weeks.disable,
				}">{{weeks.extraInfo.info}}</text>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			weeks: {
				type: Object,
				default () {
					return {}
				}
			},
			calendar: {
				type: Object,
				default: () => {
					return {}
				}
			},
			selected: {
				type: Array,
				default: () => {
					return []
				}
			},
			lunar: {
				type: Boolean,
				default: false
			}
		},
		methods: {
			choiceDate(weeks) {
				this.$emit('change', weeks)
			}
		}
	}
</script>

<style scoped>
	.uni-calendar-item__weeks-box {
		flex: 1;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.uni-calendar-item__weeks-box-text {
		font-size: 14px;
		color: #333;
	}

	.uni-calendar-item__weeks-lunar-text {
		font-size: 12px;
		color: #333;
	}

	.uni-calendar-item__weeks-box-item {
		position: relative;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: column;
		justify-content: center;
		align-items: center;
		width: 100rpx;
		height: 100rpx;
	}

	.uni-calendar-item__weeks-box-circle {
		position: absolute;
		top: 5px;
		right: 5px;
		width: 8px;
		height: 8px;
		border-radius: 8px;
		background-color: #0081FF;
	}
	
	.uni-calendar-item__weeks-box-circle.done{
		background-color: #0081FF;
	}
	
	.uni-calendar-item__weeks-box-circle.undone{
		background-color: #F00;
	}
	
	.uni-calendar-item__weeks-box-circle.almost-done{
		background-color: #F37B1D;
	}

	.uni-calendar-item--disable {
		background-color: rgba(249, 249, 249, 0.3);
		color: #c0c0c0;
	}

	.uni-calendar-item--extra {
		color: #0081FF;
		opacity: 0.8;
	}
	.uni-calendar-item--extra.done{
		color: #0081FF;
	}
	.uni-calendar-item--extra.undone{
		color: #F00;
	}
	
	.uni-calendar-item--extra.almost-done{
		color: #F37B1D;
	}

</style>