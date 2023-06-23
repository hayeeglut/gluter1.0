<template>
	<view v-for="(item,index) in examQuery" :key="index">
		<uni-collapse>
			<uni-collapse-item :title="item.couresName +' '+ item.examStatus" :show-animation="true"
				:thumb="isExamEnded(item) ? '/static/icons/blank-check-box.png' : '/static/icons/check-box.png'">
				<uni-countdown :day="item.examTimeStamps.d" :hour="item.examTimeStamps.hour" :minute="item.examTimeStamps.minute" :second="item.examTimeStamps.second" color="#ffffff" :background-color="item.color" />
				<view class="examTimeBlock">
					<text>课程名：{{item.couresName}}\n</text>
					<text>课程名：{{item.examClassroom}}\n</text>
					<text>考试时间：{{item.examDate}} {{item.examTempTime}}</text>
				</view>
			</uni-collapse-item>
		</uni-collapse>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				examQuery: [],
			}
		},
		/* 生命周期函数 监听页面加载 */
		onLoad() {
			var that = this;
			uni.showLoading({
				title: '获取中...'
			});
			//首先需要获取用户的openid
			var openid = uni.getStorageSync('openid');
			//成功获取openid
			uni.request({
				// url: 'https://localhost:8088/examTime/get',
				url: 'https://172.20.129.4:8088/examTime/get',
				header: {
					'content-type': 'application/x-www-form-urlencoded'
				},
				method: 'POST',
				data: {
					openid: openid
				},
				success(res) {
					console.log(res, '考试查询');
					//说明获取成功,直接将考试查询进行存取
					uni.hideLoading();
					that.examQuery = res.data.data;
				}
			});
		},
		onReady() {
			this.countingDownExam();
		},
		methods: {
			countingDownExam(){
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
						that.$set(item,'color','seagreen');
						that.$set(item.examTimeStamps,'day',day);
						that.$set(item.examTimeStamps,'hour',hour);
						that.$set(item.examTimeStamps,'minute',minute);
						that.$set(item.examTimeStamps,'second',second);
                    }else{
						that.$set(item,'color','red');
						that.$set(item.examTimeStamps,'day',0);
						that.$set(item.examTimeStamps,'hour',0);
						that.$set(item.examTimeStamps,'minute',0);
						that.$set(item.examTimeStamps,'second',0);
					}
                })
				this.examQuery = examQuery;
			},
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
</style>