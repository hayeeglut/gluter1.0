<template>
	<view style="height: 100%">
		<!-- 年份选择 -->
		<view class="nianFenXuanZe">
			<input :placeholder="nianFen" @input="nfxz" type="text" />
			年
			<button @click="nianFenChaXun">点击查询</button>
		</view>
		<!-- 头部标签栏 -->
		<view class="biaoTou">
			<text style="width: 55%">课程名称</text>
			<text style="width: 15%">总评</text>
			<text style="width: 15%">绩点</text>
			<text style="width: 15%">详情</text>
		</view>
		<!-- 成绩信息 -->
		<view class="cjxx" v-for="item,index in grade" :key="index">
			<text style="width: 55%">{{ item.keChengMing }}</text>

			<text :class="item.zongPing < 60 || item.zongPing == '不及格' || item.zongPing == '无成绩' ? 'zongPing' : ''"
				style="width: 15%">{{ item.zongPing }}</text>

			<text style="width: 15%">{{ item.jiDian }}</text>

			<text @click="xiangQing(item)" style="width: 15%; color: green">详情</text>
		</view>
		<!-- 详情详细信息弹窗 -->
		<uni-popup ref="show_XiangQing" type="dialog">
			<uni-popup-dialog cancelText="关闭" title="成绩详细信息" @close="dialogClose" @confirm="dialogConfirm">
				<!-- 通知公告栏 -->
				<view class="keBiaoXinXi">
					<view>
						<text>学年</text>
						{{ nianFen }}
					</view>
					
					<view>
						<text>学期</text>
						{{ showInfo.chunQiu }}
					</view>
					
					<view>
						<text>课程</text>
						{{ showInfo.keChengMing }}
					</view>

					<view>
						<text>考核方式</text>
						{{ showInfo.kaoHeFangShi }}
					</view>
					
					<view>
						<text>主讲教师</text>
						{{ showInfo.teacher }}
					</view>

					<view>
						<text>总评</text>
						{{ showInfo.zongPing }}
					</view>

					<view>
						<text>绩点</text>
						{{ showInfo.jiDian }}
					</view>
					
					<view>
						<text>及格标志</text>
						{{ showInfo.xueYuan }}
					</view>
				</view>
			</uni-popup-dialog>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				condition: '不及格',
				nianFen: 2023,
				showInfo: {},
				grade: []
			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad(options) {
			this.onLoadClone3389(options);
		},
		methods: {
			dialogClose() {
				console.log('点击关闭')
			},
			dialogConfirm() {
				console.log('点击关闭')
			},
			/**
			 * 生命周期函数--监听页面加载
			 */
			onLoadClone3389(options) {
				var that = this;
				uni.showLoading({
					title: '获取中...'
				});
				var that = this;
				//写个新的获取成绩
				//首先需要获取用户的openid
				uni.getStorage({
					key: 'openid',
					success: (res) => {
						console.log(res, '是否成功获取id');
						console.log(that.nianFen, '年份');
						//成功获取openid
						uni.request({
							// url: 'https://localhost:8088/grade/wechat/get',
							url: 'https://172.20.149.44:8088/grade/wechat/get',
							header: {
								'content-type': 'application/x-www-form-urlencoded'
							},
							method: 'POST',
							data: {
								openid: res.data,
								year: that.nianFen - 1980
							},
							success(res) {
								console.log(res,'是否成功获取成绩');
								//只要这里是成功了，直接就把成绩给他存进去了
								uni.hideLoading();
								that.grade = JSON.parse(res.data.data);
							}
						});
					}
				})
			},

			// 年份选择器
			nfxz(res) {
				this.nianFen = res.detail.value;
			},

			//点击年份查询
			nianFenChaXun(res) {
				this.onLoadClone3389({});
			},

			//详情点击用
			xiangQing(e) {
				var that = this;
				that.showInfo = e;
				this.$refs.show_XiangQing.open();
			},
		}
	}
</script>

<style>
	page {
		background-color: #f5f5f5;
	}

	/* 年份选择 */
	.nianFenXuanZe {
		display: flex;
	}

	.nianFenXuanZe input {
		margin: 2px;
		width: 50%;
		height: 40rpx;
		border: 1px solid #c8cccf;
		border-radius: 4px;
	}

	.nianFenXuanZe button {
		height: 50rpx;
		line-height: 50rpx;
		margin: 0 0 0 0;
		color: #fffefe;
		background-color: #2d293e;
	}

	.biaoTou {
		width: 100%;
		display: flex;
	}

	.biaoTou text {
		text-align: center;
		font-weight: 900;
	}

	.cjxx {
		width: 100%;
		display: flex;
		margin-top: 15rpx;
	}

	.cjxx text {
		text-align: center;
	}

	.zongPing {
		color: red;
	}

	/* 详情详细信息 */
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