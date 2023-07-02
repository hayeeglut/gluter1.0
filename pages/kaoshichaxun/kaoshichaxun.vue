<template>
	<view v-for="(item,index) in examQuery" :key="index">
		<!-- thumb="isExamEnded(item) ? '/static/icons/blank-check-box.png' : '/static/icons/check-box.png'" -->
		<uni-countdown :day="item.day" :hour="item.hour" :minute="item.minute" :second="item.second" color="#ffffff"
			:background-color="item.color" />
		<view class="examTimeBlock">
			<text>课程名：{{item.couresName}}\n</text>
			<text>考试状态：{{item.examStatus}}\n</text>
			<text>考试教室：{{ item.examSchool }} {{ item.examBuilding }} {{item.examClassroom}}\n</text>
			<text>考试时间：{{item.examDate}} {{item.examTempTime}}</text>
			<!-- <text @click="details(item)" style="width: 15%; color: green">详情</text> -->
		</view>
		<!-- 分割线 -->
		<uv-divider text=""></uv-divider>
	</view>


	<!-- <uni-popup ref="showExamDetails" type="dialog">
		<uni-popup-dialog cancelText="关闭" title="成绩详细信息" @close="dialogClose" @confirm="dialogConfirm">
			<view class="examDetails">
				<view>
					<text>课程名</text>
					{{ showInfo.couresName }}
				</view>
				<view>
					<text>课程号</text>
					{{ showInfo.courseNumber }}
				</view>
				<view>
					<text>考试地点</text>
					{{ showInfo.examSchool }};
					{{ showInfo.examBuilding }};
					{{ showInfo.examClassroom }}
				</view>
				<view>
					<text>考试时间</text>
					{{ showInfo.examDate }} {{ showInfo.examTempTime }}
				</view>
				<view>
					<text>考试状态</text>
					{{ showInfo.examStatus }}
				</view>
			</view>
		</uni-popup-dialog>
	</uni-popup> -->

</template>

<script>
	export default {
		data() {
			return {
				//考试详情信息
				//showInfo: [],
				examQuery: [], //考试时间集合
				openid: "", //用户openid
			}
		},
		/**
		 * 生命周期函数 监听页面加载
		 */
		onLoad() {
			var that = this;
			uni.showLoading({
				title: '获取中...'
			});
			//首先需要获取用户的openid
			that.openid = uni.getStorageSync('openid');
			//成功获取openid
			uni.request({
				// url: 'https://localhost:8088/examTime/get',
				url: 'https://neeeeeeebs.fit:8088/examTime/get',
				header: {
					'content-type': 'application/x-www-form-urlencoded'
				},
				method: 'POST',
				data: {
					openid: that.openid
				},
				success(res) {
					console.log(res, '考试时间')
					//说明获取成功,直接将考试查询进行存取
					uni.hideLoading();
					//将返回值存入examQuery
					that.examQuery = res.data.data;
					that.countingDownExam();
				}
			});

		},
		methods: {
			/**
			 * 获取现在的时间与考试开始时间的时间差
			 */
			countingDownExam() {
				var that = this;
				//获得现在时间戳
				let currentTime = new Date().getTime();
				//获得考试时间列表
				this.examQuery.forEach((item, index) => {
					var leftTime = item.examTimeStamp - currentTime; //计算两日期之间相差的毫秒数
					if (leftTime >= 0) {
						let day = Math.floor(leftTime / 1000 / 60 / 60 / 24);
						let hour = Math.floor(leftTime / 1000 / 60 / 60 % 24);
						let minute = Math.floor(leftTime / 1000 / 60 % 60);
						let second = Math.floor(leftTime / 1000 % 60);
						that.$set(item, 'color', 'seagreen');
						that.$set(item, 'day', day);
						that.$set(item, 'hour', hour);
						that.$set(item, 'minute', minute);
						that.$set(item, 'second', second);
					} else {
						that.$set(item, 'color', 'red');
						that.$set(item, 'day', 0);
						that.$set(item, 'hour', 0);
						that.$set(item, 'minute', 0);
						that.$set(item, 'second', 0);
					}
				})
			},
			/**
			 * 详情点击用
			 * @param {Object} e
			 */
			/* details(e) {
				this.showInfo = e;
				this.$refs.showExamDetails.open();
			}, */
			/**
			 * 判断当前考试是否结束
			 * @param {Object} res
			 */
			isExamEnded(res) {
				//获取开始考试时间戳
				let startExamTime = res.examTimeStamp;
				let currentTime = new Date().getTime();
				//开始考试时间减去现在时间
				let temptimeDifference = startExamTime - currentTime;
				//若是已经过了考试开始时间，默认考试已完成
				if (temptimeDifference > 0) {
					return true;
				} else {
					return false;
				}
			}
		}
	}
</script>

<style>
	.examTimeBlock {
		font-size: 30rpx;
		font-weight: 600;
	}

	/* 详情详细信息 */
	.examDetails {
		width: 90%;
		margin: 0 auto;
		padding: 10rpx 0 0 0;
	}

	.examDetails text {
		margin-right: 20rpx;
		font-weight: 800;
	}
</style>