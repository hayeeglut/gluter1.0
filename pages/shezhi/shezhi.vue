<template>
	<view style="height: 100%">
		<view>
			<view class="xuanXiang">
				<view @click="xiuGaiXueHaoMiMa" class="zhuXiao">
					<text class="alltext">注销用户，重新登陆</text>
					<image src="../../static/images/shezhi/xiaolian.png"></image>
				</view>
				<!-- 反馈 -->
				<view @click="fanKuiKuang" class="fankui">
					<text class="alltext">给开发者反馈</text>
					<image src="../../static/images/shezhi/fanzhuanxiaolian.png"></image>
				</view>
				<view @click="gengXinKeBiao" class="gengxin">
					<text class="alltext">更新课表</text>
					<image src="../../static/images/shezhi/zhangzuixiao.png"></image>
				</view>
			</view>
		</view>

		<!-- 弹出层-填写账号密码 -->
		<uni-popup ref="fanKui" type="bottom" @close="onClose">
			<view class="tanchuceng">
				<textarea @input="fanKuiNeiRongFun" class="shuRuKuang" placeholder="也可以联系开发者QQ哦:2695743645"
					auto-height></textarea>
				<button class="fanKuiAnNiu" color="#00aaff" @click="fanKui">点击反馈</button>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				fanKuiNeiRong: '',
			}
		},
		methods: {
			// 改绑学号
			xiuGaiXueHaoMiMa(res) {
				uni.request({
					// url: 'https://localhost:8088/userInfo/wechat/deleteUser',
					url: 'https://172.20.129.4:8088/userInfo/wechat/deleteUser',
					method: 'POST',
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					data: {
						openid: uni.getStorageSync('openid')
					},
					success: (res) => {
						console.log(res, '改学号');
						//确认成功后
						if (res.data.a = true) {
							//跳转回user页面
							uni.reLaunch({
								url: '/pages/user/user'
							});
						}
					}
				});
			},

			// 实时获取反馈内容
			fanKuiNeiRongFun(res) {
				this.fanKuiNeiRong = res.detail.value;
			},

			// 点击开发者反馈弹出上拉框
			fanKuiKuang(res) {
				this.$refs.fanKui.open();
			},

			// 反馈按钮
			fanKui(result) {
				var that = this;
				if (that.fanKuiNeiRong != null) {
					uni.showToast({
						title: '反馈已提交啦'
					});
					uni.request({
						//  url: 'https://localhost:8088/jianYi/set1',
						url: 'https://172.20.129.4:8088/jianYi/set1',
						method: 'POST',
						header: {
							'content-type': 'application/x-www-form-urlencoded'
						},
						data: {
							message: that.fanKuiNeiRong
						},
						success: (res) => {
							//关闭上拉框
							that.$refs.fanKui.close();
							uni.hideToast({});
						}
					});
				}
			},

			//更新课表，直接去后台把它旧课表给删了就好了
			gengXinKeBiao(res) {
				uni.setStorageSync('weeklyData', 'needReNew');
				//后台请求删除课表
				uni.request({
					// url: 'https://localhost:8088/keBiao/wechat/reNewKeBiao',
					url: 'https://172.20.129.4:8088/keBiao/wechat/reNewKeBiao',
					method: 'POST',
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					data: {
						openid: uni.getStorageSync('openid')
					},
					success: (res) => {
						uni.reLaunch({
							url: '/pages/user/user'
						});
					}
				});


			},

			onClose() {
				console.log('占位：函数 onClose 未声明');
			}
		}
	}
</script>

<style>
	/* 修改学号密码 */
	.xuanXiang {
		padding-top: 30rpx;
		width: 90%;
		background-color: white;
		border-radius: 30rpx;
		height: 300rpx;
		margin: 0 auto;
	}

	.alltext {
		margin-left: 30rpx;
		margin-top: 70rpx;
	}

	.xuanXiang image {
		margin-top: 10rpx;
		width: 40rpx;
		height: 40rpx;
		float: right;
		padding-right: 50rpx;
	}
	
	.zhuXiao{
		margin-left: 40rpx;
		width: 80%;
		height: 80rpx;
		background-color: bisque;
		border-radius: 10rpx;
		align-items: center;
		vertical-align: center;
		justify-content: center;
	}
	.fankui{
		margin-top: 10rpx;
		margin-left: 40rpx;
		width: 80%;
		height: 80rpx;
		background-color: bisque;
		border-radius: 10rpx;
		align-items: center;
		vertical-align: center;
		justify-content: center;
	}
	.gengxin{
		margin-top: 10rpx;
		margin-left: 40rpx;
		width: 80%;
		height: 80rpx;
		background-color: bisque;
		border-radius: 10rpx;
		align-items: center;
		vertical-align: center;
		justify-content: center;
	}

	/* 给我反馈 */
	.tanchuceng {
		background-color: darkseagreen;
		border-radius: 8px;
		height: 50%;
		width: 100%;
	}

	.shuRuKuang {
		background-color: aliceblue;
		border: 1px solid #00000059;
		margin: 10px auto;
		width: 90%;
		border-radius: 8px;
	}

	.fanKuiAnNiu {
		display: flex;
		margin: 0 auto;
	}
</style>