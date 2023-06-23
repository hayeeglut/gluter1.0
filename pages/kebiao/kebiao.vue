<template>
	<view>
		<!-- 导航区域 -->
		<scroll-view class="navScroll" scroll-x="true" enable-flex="true"
			:scroll-into-view="'a' + nowWeek ? nowWeek - 3 : pageNum > 3 ? pageNum - 2 : 0" scroll-with-animation="true">
			<view class="navItem" v-for="(item,index) in weekArray" :key="index" :id=" 'a' + index ">
				<view :class="'navContent ' + (todayWeek == index + 1 || index == pageNum ? 'active' : '')"
					@click="changeNav" :data-page="index">
					{{ todayWeek == index + 1 ? item + '(本周)' : item }}
				</view>
			</view>
		</scroll-view>

		<!-- 日期区域 -->
		<view class="date">
			<view class="month">
				<view>{{month}}</view>
				<view>月</view>
			</view>
			<view class="day">
				<view :class="'week ' + (todayMonth == monthNum && day == item ? 'todayDate' : '')"
					v-for="(item,index) in nowDay" :key="index">
					<view class="week-item">周{{week[index]}}</view>
					<view class="day-item">
						{{ nowDay[index] == 1 ? (monthNum < 12 ? monthNum + 1 : 1) + '月' : nowDay[index] + '日' }}
					</view>
				</view>
			</view>
		</view>

		<!-- 课表区域 -->
		<scroll-view scroll-y="true" :scroll-top="scrollTop" class="courseScroll">
			<view class="courseContent">
				<view class="courseTime">
					<!-- 这个应该是左边的时间条 -->
					<view v-for="(item,index) in 12" :key="index" class="left">
						<view class="number">{{ item + 1 }}</view>
						<view class="course-time">
							<view class="time-start">{{ course_time[index][0] }}</view>
							<view class="time-end">{{ course_time[index][1] }}</view>
						</view>
					</view>
					<view>
						<view>其他课程</view>
					</view>
				</view>
				<view class="course">
					<!-- 课程信息块 -->
					<view v-for="(item,index) in weeklyData[nowWeek-1].keInfoList" :key="index" class="kcb-item"
						@click="FadeOutCLick"
						:style="'margin-left:'+(item.xingQiJi!=7?(item.xingQiJi)*100:0)+'rpx;margin-top:'+((item.shangKeJieShu-1)*110+4)+'rpx;height:'+(2*110 - 8)+'rpx;'">
						<!-- 告警标志 -->
						<view class="attention"
							:style="'width:40rpx;height: 40rpx; position: absolute;display:'+ item.zhuangKe">
							<image src="../../static/images/kebiao/attention.png"></image>
						</view>
						<!-- 课表块 -->
						<view class="smalltext" :style="'background-color:'+ colorArrays[item.id-1]">
							<view class="smalltextName">{{item.keChengMing}}</view>
							<view class="smalltextAddress">{{item.jiaoShi}}</view>
						</view>
					</view>
				</view>
			</view>
		</scroll-view>
	</view>

	<uni-popup ref="keInfo" type="dialog">
		<uni-popup-dialog cancelText="关闭" title="课程详细信息" @close="keInfo_onClose">
			<!-- 课表栏 -->
			<view class="keBiaoXinXi" v-for="(item, index) in showKeInfo" :key="index">
				<view>
					<text>课程</text>
					{{ item.keChengMing }}
				</view>
				<view>
					<text>教室</text>
					{{ item.jiaoShi }}
				</view>
				<view>
					<text>日期</text>
					{{ item.riQi }}
				</view>
				<view>
					<text>课节</text>
					{{ item.shangKeJieShu }}-{{ item.shangKeJieShu + 1 }}节
				</view>
				<view>
					<text>星期</text>
					星期{{ item.xingQiJi }}
				</view>
				<view>
					<text>属性</text>
					{{ item.xuanKeShuXing }}
				</view>
			</view>
		</uni-popup-dialog>
	</uni-popup>
</template>

<script>
	import colors from '../../utils/colors.js';
	const app = getApp();
	export default {
		data() {
			return {
				weeklyData: [
					{
					    keInfoWeek: [''],
					    keInfoList: ['']
					},
				],
				showKeInfo: [{}],
				//课表信息
				weekArray: [
					'第1周',
					'第2周',
					'第3周',
					'第4周',
					'第5周',
					'第6周',
					'第7周',
					'第8周',
					'第9周',
					'第10周',
					'第11周',
					'第12周',
					'第13周',
					'第14周',
					'第15周',
					'第16周',
					'第17周',
					'第18周',
					'第19周',
					'第20周'
				],
				
				// 当前所在分类的索引
				pageNum: 0,

				// 今日日期
				todayDay: 0,

				// 今日月份
				todayMonth: 0,

				// 今日周
				todayWeek: 0,

				// 今日日期
				day: 0,

				// 月份
				month: 0,
				
				monthNum: 1,

				// 周日为起始日
				week: ['日', '一', '二', '三', '四', '五', '六'],

				// 本周的七天日期
				nowDay: [1, 2, 3, 4, 5, 6, 7],

				// 本学期开学时间
				schoolTime: ['2023', '02', '20'],

				// 当前周
				nowWeek: 0,
				
				course_time: [
					['8:30', ''],
					['', '10:05'],
					['10:20', ''],
					['', '12:00'],
					['14:30', ''],
					['', '16:10'],
					['16:25', ''],
					['', '18:00'],
					['18:30', ''],
					['', '20:00'],
					['20:20', ''],
					['', '21:50']
				],
				colorArrays: [''],
				scrollTop: 0
			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad(options) {
			var that = this;
			this.nowWeek = this.getNowWeek();
			this.nowDay = this.getDayOfWeek(this.nowWeek);
			this.pageNum = this.nowWeek - 1;
			this.month = this.getMonth((this.nowWeek - 1) * 7);
			this.todayWeek = this.nowWeek;
			this.monthNum = this.month / 1;
			// 当前月份数字类型，用于数字运算
			this.colorArrays = colors; // 课表颜色
			//获取storage里面的课表数据
			uni.getStorage({
				key: 'weeklyData',
				success(res) {
					console.log(res);
					that.weeklyData = res.data;
				}
			});
		},
		/**
		 * 生命周期函数--监听页面初次渲染完成
		 */
		onReady(option) {
			var that = this;
			that.getSystemTime();
		},

		methods: {
			/**
			 * 获取系统时间
			 */
			getSystemTime() {
				this.todayDay = new Date().getDate();
				this.todayMonth = new Date().getMonth() + 1;
				this.day = new Date().getDate();
				this.month = new Date().getMonth() + 1;
			},
			// 获取课表块详细信息用
			FadeOutCLick(e) {
				/*
				这里面涉及到了安卓和苹果手机不同的屏幕宽度的问题
				rpx单位数值 * wx.getSystemInfoSync().windowWidth/750 -> px单位数值
				let pxValue = RPX * (wx.getSystemInfoSync().windowWidth/750);
				*/
				console.log(e,'kke')
				var that = this;
				let zhou = parseInt((e.currentTarget.offsetLeft + 4) / (uni.getSystemInfoSync().windowWidth / 750) / 100);
				let jie = parseInt(((e.currentTarget.offsetTop + 4) / (uni.getSystemInfoSync().windowWidth / 750) - 4) / 110 + 1);
				let j = 0;
				console.log(that.weeklyData[that.pageNum ].keInfoList);
				for (let i = 0; i < that.weeklyData[that.pageNum].keInfoList.length; i++) {
					if(that.weeklyData[that.pageNum].keInfoList[i].shangKeJieShu == jie && that.weeklyData[that
							.pageNum].keInfoList[i].xingQiJi == zhou) {
						that.$set(that.showKeInfo, j, that.weeklyData[that.pageNum].keInfoList[i]);
						j++;
					}
				}
				that.$refs.keInfo.open();
			},
			
			// 获取第几周后的月份
			getMonth(days) {
			    let [year, month, day] = this.schoolTime;
			    var date = new Date(year, month - 1, day);
			    date.setDate(date.getDate() + days); //获取n天后的日期
			    if (date.getMonth() + 1 < 10) {
			        var m = '0' + (date.getMonth() + 1);
			    } else {
			        var m = date.getMonth() + 1;
			    }
			    return m;
			},
			
			// 获取第几周后的星期几的日期
			getDay(days) {
			    let [year, month, day] = this.schoolTime;
			    var date = new Date(year, month - 1, day);
			    date.setDate(date.getDate() + days); //获取n天后的日期
			    if (date.getDate() < 10) {
			        var d = '0' + date.getDate();
			    } else {
			        var d = date.getDate();
			    } //获取当前几号，不足10补0
			    return d;
			},
			
			// 获取当前周
			getNowWeek() {
			    var date = new Date();
			    let [year, month, day] = this.schoolTime;
			    var start = new Date(year, month - 1, day);
			    //计算时间差
			    var left_time = parseInt((date.getTime() - start.getTime()) / 1000) + 86400;
			    var days = parseInt(left_time / 3600 / 24);
			    var week = Math.floor(days / 7) + 1;
			    var result = week;
			    if (week > 20 || week <= 0) {
			        result = this.now_week;
			    }
			    return result;
			},
			
			//获取一周的日期
			getDayOfWeek(week) {
			    var day = [];
			    for (var i = -1; i < 6; i++) {
			        var days = (week - 1) * 7 + i;
			        day.push(this.getDay(days));
			    }
			    return day;
			},
			
			// 获取课表数据
			async getCourseList() {},
			
			// 点击切换导航的回调
			changeNav(event) {
				console.log(event,'dijizhou')
			    this.pageNum = event.currentTarget.dataset.page;
			    this.nowWeek = this.pageNum + 1;
			    this.nowDay = this.getDayOfWeek(this.nowWeek);
			    this.month = this.getMonth((this.nowWeek - 1) * 7);
				this.monthNum = this.month / 1;
			},
			
			keInfo_onClose() {
			    console.log('占位：函数 keInfo_onClose 未声明');
			}
		}
	}
</script>

<style>
.navScroll {
    display: flex;
    white-space: nowrap;
    height: 75rpx;
    font-size: 25rpx;
}

.navScroll .navItem {
    margin: 20rpx 12rpx 0rpx;
}

.navScroll .navItem .navContent {
    padding: 5rpx 25rpx;
    line-height: 38rpx;
}

.active {
    border-bottom: 1px solid red;
    color: red;
}

.date {
    display: flex;
    height: 90rpx;
    padding: 6rpx 0;
    color: #3f3f3f;
    line-height: 1.4;
}

.date .month {
    width: 7%;
    font-size: 24rpx;
    display: inline-block;
    color: #3f3f3f;
    text-align: center;
    height: 100%;
    padding: 8rpx 0;
}

.date .day {
    width: calc(100% - 8%);
    margin: 0 auto;
}
.date .day .week {
    width: calc(100% / 7);
    display: inline-block;
    height: 100%;
    text-align: center;
}
.date .day .week-item {
    font-size: 24rpx;
    font-weight: 700;
    line-height: 1.8;
}
.date .day .day-item {
    font-size: 22rpx;
    color: #333;
}

.date .day .todayDate {
    background: #1380ff;
    border-radius: 8rpx;
    color: #fff;
}

.date .day .todayDate .day-item {
    color: #fff;
}

.courseScroll {
    width: 100%;
    height: 100%;
    z-index: 2;
    position: fixed;
}

.courseScroll .courseContent {
    height: 1580rpx;
    width: 100%;
    display: flex;
}

.courseScroll .courseContent .courseTime {
    width: 7%;
    font-size: 24rpx;
    display: inline-block;
    color: #3f3f3f;
    text-align: center;
}

.courseScroll .courseContent .courseTime .left {
    width: 100%;
    height: 110rpx;
    justify-content: center;
    display: flex;
    position: relative;
    align-items: center;
    z-index: 0;
    flex-direction: column;
}

.courseScroll .courseContent .courseTime .left .course-time {
    height: 110rpx;
    width: 100%;
    position: absolute;
    text-align: center;
    top: 0;
    right: 0;
}

.courseScroll .courseContent .courseTime .left .course-time .time-start,
.courseScroll .courseContent .courseTime .left .course-time .time-end {
    color: #8799a3;
    font-size: 20rpx;
    position: absolute;
    width: 100%;
    left: 0;
}

.courseScroll .courseContent .courseTime .left .course-time .time-start {
    top: 12rpx;
}

.courseScroll .courseContent .courseTime .left .course-time .time-end {
    bottom: 12rpx;
}

.courseScroll .courseContent .courseTime .left .number {
    font-size: 24rpx;
    line-height: 110rpx;
}

.course {
    width: 92%;
    margin: 0 auto;
    position: relative;
}
/* 课程快css参数 */
.course .kcb-item {
    width: calc(92% / 7);
    position: absolute;
    justify-content: center;
    display: flex;
}
/* 课程块淡入淡出用 */
.fadeout {
    -webkit-transition: all 1.5s;
    -moz-transition: all 1.5s;
    -ms-transition: all 1.5s;
    -o-transition: all 1.5s;
    transition: all 1.5s;
    opacity: 0;
}
.fadein {
    -webkit-transition: all 1.5s;
    -moz-transition: all 1.5s;
    -ms-transition: all 1.5s;
    -o-transition: all 1.5s;
    transition: all 1.5s;
    opacity: 1;
}
/* 告警图标 */
.attention image {
    width: 40rpx;
    height: 40rpx;
    position: absolute;
    /* display: flex; */
    margin-left: 100%;
    margin-top: -30%;
    z-index: 99;
}
.course .kcb-item .smalltext {
    height: 100%;
    width: 100%;
    margin: 0 4rpx;
    font-size: 24rpx;
    line-height: 36rpx;
    text-align: center;
    color: #fff;
    border-radius: 8rpx;
    padding: 2rpx 4rpx;
    display: flex;
    flex-direction: column;
    position: absolute;
}

.course .kcb-item .smalltext .smalltextName {
    flex: 1 0 auto;
}

.course .kcb-item .smalltext .smalltextAddress {
    border-top: 1px solid #fff;
    flex: 0 0 auto;
}
/* 课表详细信息 */
.keBiaoXinXi {
    width: 90%;
    margin: 0 auto;
    padding: 10rpx 0 0 0;
}
.keBiaoXinXi text {
    margin-right: 20rpx;
    font-weight: 800;
}
</style>