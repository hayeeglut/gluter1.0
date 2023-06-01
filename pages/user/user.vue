<template>
	<!-- 头顶通知公告栏 -->
	<view>
		<uni-notice-bar @click="notice()" background-color="#ecf9ff" single="true" color="#2979FF" showGetMore="true"
			showIcon="true" text="通知公告栏！通知公告栏！通知公告栏！"></uni-notice-bar>
	</view>
	
	<!-- <view class="">
		<button type="default" @click="getKeBiao()">课表</button>
	</view> -->
	
	<!-- 个人小工具 -->
	<view style="display: flex">
		<!-- 小按钮 -->
		<view class="anNiu">
			<view v-for="item,key in tuBiao" :key="key" class="chengJi">
				<navigator url="{{item.navigateUrl}}" hover-class="navigator-hover">
					<image :src="item.imgurl"></image>
					<view>{{item.name}}</view>
				</navigator>
			</view>
		</view>
	</view>

	<!-- 弹出层-填写账号密码 -->
	<uni-popup ref="userinfo" type="top" style="height: 80%;" background-color="#ecf9ff">
		<!-- 切换学号弹出层弹出层 -->
		<view>
			<uni-notice-bar text="目前已支持，教务处密码登录，和统一身份密码登录，有空就写个手机号登录"></uni-notice-bar>
			<uni-notice-bar text="教务处密码是字母+数字+特殊符号，统一身份密码好像是校园网密码"></uni-notice-bar>
			<view class="qXueHao">学号:</view>
			<input type="text" @blur="qieXueHao" class="qieXueHao" auto-focus="" placeholder="请输入学号" />
			<view class="qMiMa">密码:</view>
			<input type="text" @blur="qieMiMa" :password="showPassword" class="qieMiMa" placeholder="请输入密码" />
			<text :class="[!showPassword ? 'lookMiMa' : 'lookedMiMa']" @click="changePassword">查看密码</text>
			<!--单选框 -->
			<radio-group style="text-align: center; margin: 5%;" @change="login_type_confirm">
				<radio color="red" value="jwc">教务处密码登录</radio>
				<radio color="red" value="unit">统一身份密码登录</radio>
			</radio-group>
			<button type="default" @click="cunChuZHMM()">点击确认</button>
		</view>
	</uni-popup>

	<!-- 通知公告弹出窗口 -->
	<uni-popup ref="notice" type="dialog">
		<uni-popup-dialog :type="msgType" cancelText="关闭" confirmText="同意" title="通知" @close="dialogClose"
			@confirm="dialogConfirm">
			<!-- 通知公告栏 -->
			<view class="tongZhiGongGao">
				<scroll-view scroll-y="true">
					<view>
						<view class="neiRong" v-for="item,key in tongZhiGongGao" :key="key">{{item}}</view>
					</view>
				</scroll-view>
			</view>
		</uni-popup-dialog>
	</uni-popup>
</template>

<script>
	import colors from '../../utils/colors.js';
	import calculateWeek from '../../utils/calculateWeek.js';
	export default {
		data() {
			return {
				j_username: "",
				j_password: "",
				weeklyData: "e",
				openid: "",
				todayDay: 0,
				todayMonth: 0,
				day: 0,
				month: 0,
				// 弹出层
				show: false,
				show_tongZhi: false,
				login_type: 'unit',
				showPassword: true,
				//本学期开学日
				schoolTime: ['2022', '08', '29'],
				//通知公告
				tongZhiGongGao: [''],
				colorArrays: [''],
				today_keBiao: '',
				JESSIONID: '',
				//图标信息
				tuBiao: [{
						"imgurl": "../../static/images/tool_images/chengJi.png",
						"name": "成绩查询",
						"navigateUrl": ""
					}, {
						"imgurl": "../../static/images/tool_images/kaoshi.png",
						"name": "考试查询",
						"navigateUrl": ""
					}, {
						"imgurl": "../../static/images/tool_images/xdjc.png",
						"name": "修读进程",
						"navigateUrl": ""
					}, {
						"imgurl": "../../static/images/tool_images/chengjichaxun.png",
						"name": "成绩订阅通知",
						"navigateUrl": ""
					}, {
						"imgurl": "../../static/images/tool_images/shezhi.png",
						"name": "设置",
						"navigateUrl": ""
					}, {
						"imgurl": "../../static/images/tool_images/shigongzhong.png",
						"name": "施工中...",
						"navigateUrl": ""
					},
					{
						"imgurl": "../../static/images/tool_images/shigongzhong.png",
						"name": "施工中...",
						"navigateUrl": ""
					},
					{
						"imgurl": "../../static/images/tool_images/shigongzhong.png",
						"name": "施工中...",
						"navigateUrl": ""
					}
				]
			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad: function(option) {
			uni.clearStorageSync("weeklyData");
			var that = this;
			console.log('看看我运行了吗')
			//开局获取openid
			//获取uni.login
			uni.showLoading({
				title: '少女祈祷中...',
			});
			uni.login({
				"provider": "weixin",
				"onlyAuthorize": true, // 微信登录仅请求授权认证
				success: function(event) {
					console.log(event, '获取code');
					//客户端成功获取授权临时票据（code）,向业务服务器发起登录请求。
					uni.request({
						url: 'https://172.20.142.219:8088/wx_get_true_openid/get',
						header: {
						        'content-type': 'application/x-www-form-urlencoded'
						      },
						data: {
							"wx_login_code": event.code
						},
						success: (res) => {
							console.log(res, '获取token')
							//获得token完成登录
							uni.setStorageSync('token', res.data);
							that.inputopenid(res.data);
							console.log('Page Load')
							//去userinfo表查找是否存在用户
							uni.request({
								url: "https://172.20.142.219:8088/userInfo/wechat/find_user",
								header: {
								        'content-type': 'application/x-www-form-urlencoded'
								      },
								method: 'POST',
								data: {
									"openid": res.data
								},
								success: (res) => {
									console.log(res, '获取a')
									if (res.data.a == true) {
										//说明这名用户存在
										//可以进行课表请求
										that.getKeBiao();
									} else if (res.data.a == false) {
										//说明后端不存在这个用户
										that.popup();
									}
								}
							})
							uni.hideLoading();
						}
					})
				}
			})
			//开局获取通知
			that.getTongZhi();
			//将课表块颜色存储
			this.colorArrays = colors
		},
		onShow: function() {
			
		},
		/**
		 * 生命周期函数--监听页面初次渲染完成
		 */
		onReady: function(option) {
			var that = this;
			this.getSystemTime();
		},
		methods: {
			//登录方式单选按钮
			login_type_confirm(e) {
				let type = e.detail.value;
				// 1 获取单选框中的值
				// 2 把值赋值给 data 中的数据
				this.login_type = type;
			},
			changePassword() {
				this.showPassword = !this.showPassword;
			},
			dialogConfirm() {
				console.log('点击确认')
			},
			dialogClose() {
				console.log('点击关闭')
			},
			/**
			 * 打开更新个人信息的弹窗
			 */
			popup() {
				this.$refs.userinfo.open()
			},
			/**
			 * 存储openid变量
			 */
			inputopenid(openid) {
				this.openid = openid;
			},
			/**
			 * 打开通知公告弹窗
			 */
			notice(type) {
				this.msgType = type
				this.getTongZhi();
				this.$refs.notice.open()
			},
			/**
			 * 获取系统时间
			 */
			getSystemTime: function() {
				this.todayDay = new Date().getDate();
				this.todayMonth = new Date().getMonth() + 1;
				this.day = new Date().getDate();
				this.month = new Date().getMonth() + 1;
			},
			//将获取通知公告请求独立
			getTongZhi: function() {
				uni.request({
					url: 'https://172.20.142.219:8088/tongZhi/getToolTop',
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					method: "POST",
					success: (res) => {
						console.log(res, '具体通知')
						this.tongZhiGongGao = res.data.data;
					}
				})
			},
			// 点击jump跳转
			jump: function (e) {
			  uni.navigateTo({
			    url: '../keBiao/keBiao',
			  })
			},
			// 将弹出层的账号密码存储
			qieXueHao: function (res) {
				this.j_username = res.detail.value;
				
			},
			qieMiMa: function (res) {
			    this.j_password = res.detail.value;
				console.log(this, '看看账号密码存进去了吗')
			},
			// 弹出层的确认按钮
			cunChuZHMM: function (res) {
			  //将账号密码存入storage中，然后重载本页面
			  uni.setStorageSync('j_username', this.j_username);
			  uni.setStorageSync('j_password', this.j_password);
			  this.createUser();
			},
			// 将课表请求独立出来
			getKeBiao: function(res) {
				var that = this;
				uni.request({
					url: "https://172.20.142.219:8088/keBiao/wechat/getKeBiao",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					data: {
						"openid": that.openid,
					},
					method: 'POST',
					success: (result) => {
						if (result.data.a == false) {
							//说明账号密码错误
							console.log("居然有账号密码错误?");
							uni.showToast({
								title: '账号密码错误哦',
							})
							uni.removeStorage({
								key: 'j_username',
							})
							uni.removeStorage({
								key: 'j_password',
							})
							uni.removeStorage({
								key: 'weeklyData',
							})
							setTimeout(() => {
								uni.reLaunch({
									url: '/pages/user/user',
								})
							}, 1500);
						} else {
							that.weeklyData = JSON.parse(result.data.data);
							that.JESSIONID = result.data.jessionid;
							uni.setStorageSync('JESSIONID', result.data.jessionid);
							uni.setStorageSync('weeklyData', JSON.parse(result.data.data));
							that.getBenRiKeBiao();
							uni.hideLoading();
						}
					},
				})
			},
			//创建用户并发送后台
			createUser: function(res) {
				uni.showLoading({
					title: '少女祈祷中...',
				})
				var that = this;
				uni.request({
					url: "https://172.20.142.219:8088/userInfo/wechat/createUser",
					method: "POST",
					 header: {
					        'content-type': 'application/x-www-form-urlencoded'
					      },
					data: {
						"j_username": that.j_username,
						"j_password": that.j_password,
						"openid": that.openid,
						"login_type": that.login_type
					},
					success: (res) => {
						console.log(res)
						uni.hideLoading();
						if (res.data.a = true) {
							//说明后端返回true，用户创建成功
							//重新调用本页面
							console.log("重载页面！！！");
							uni.reLaunch({
								url:'/pages/user/user'
							})
						}
						if (res.data.a = false) {
							//说明账号密码错了，因为我后端拿到密码后会进行教务处认证
							uni.hideLoading();
							that.popup();
						}
					}
				})
			},

			//将获取本日课表独立出来
			/* getBenRiKeBiao: function(res) {
				var that = this;
				//开始对课表进行裁剪，获得当前日课表
				//这条函数可以用来获取当前日期-开学日期的间隔天数
				let phone_type = uni.getSystemInfoSync();
				if (phone_type.platform == 'ios') {
					var temp = Math.floor(((new Date().valueOf() - new Date("2023/2/19").valueOf()) / 1000 / 60 / 60 /
						24) / 7);
				} else {
					var temp = Math.floor(((new Date().valueOf() - new Date("2023-2-19").valueOf()) / 1000 / 60 / 60 /
						24) / 7);
				}
				// console.log(temp);
				// console.log(phone_type);
				//此时temp就是我们定位的那一周，不过我们依然用temp
				let a = new Array();
				//获取今天是星期几
				let xqj = calWeek.getWeekByDate(new Date());
				for (let j = 0; j < that.data.weeklyData[temp].keInfoList.length; j++) {
					if (that.data.weeklyData[temp].keInfoList[j].xingQiJi == xqj) {
						//说明这节课属于今天，给他存一下
						a.push(that.data.weeklyData[temp].keInfoList[j]);
					}
				}
				//成功拿到今天的数据,给他存一下
				that.setData({
					today_keBiao: a,
				})
			}, */
		},
		
	}	
</script>

<style>
	/* 个人信息栏 */
	.personInfo {
		width: 350rpx;
		background-color: rgb(43 38 66 / 82%);
		/* background-color: rgb(6 171 234 / 82%); */
		height: 300rpx;
		border-radius: 10rpx;
		float: left;
	}

	/* userInfo */
	.personInfo .userInfo {
		background-color: #944949;
		width: 93%;
		height: 100rpx;
		margin: 0 auto;
		border-radius: 10rpx;
		margin-top: 20rpx;
		color: white;
	}

	/* 同学 */
	.personInfo .tongXue {
		line-height: 50rpx;
		font-weight: 700;
		font-size: 35rpx;
		margin-left: 15rpx;
	}

	.personInfo .id {
		line-height: 50rpx;
		margin-left: 15rpx;
	}

	/* 教务处绑定 */
	.personInfo .jwc_bd {
		width: 130rpx;
		height: 130rpx;
		background-color: teal;
		border-radius: 20rpx;
		color: white;
		line-height: 64rpx;
		margin: 20rpx 0 0 12rpx;
		text-align: center;

	}

	/* 设置里面的东西 */
	.personInfo .sheZhi {
		text-align: center;
		width: max-content;
		margin: 0 auto;
		margin-top: 20rpx;
		color: white;
	}

	.personInfo .sheZhi image {
		width: 80rpx;
		height: 80rpx;
	}

	/* 那些小按钮 */
	.anNiu {
		display: flex;
		flex-wrap: wrap;
	}

	.anNiu .chengJi {
		text-align: center;
		width: 25%;
	}

	.anNiu .chengJi image {
		width: 80rpx;
		height: 80rpx;
	}

	/* 弹出层 */
	/* 统一身份认证 */
	.qContext {
		text-align: center;
	}

	/* 输入框 */
	.qXueHao {
		color: rgb(170, 0, 255);
		width: 600rpx;
		margin-top: 20rpx;
		margin-left: 50rpx;
	}

	.qMiMa {
		color: rgb(170, 0, 255);
		width: 600rpx;
		margin-top: 20rpx;
		margin-left: 50rpx;
	}

	.qieXueHao {
		min-height: 1.8rem;
		color: black;
		width: 600rpx;
		border-radius: 10rpx;
		margin-left: 50rpx;
		border: 1rpx solid #c8cccf;
	}

	.qieMiMa {
		min-height: 1.8rem;
		color: black;
		width: 600rpx;
		border-radius: 10rpx;
		margin-left: 50rpx;
		border: 1rpx solid #c8cccf;
	}

	.lookMiMa {
		color: rgb(170, 85, 255);
		width: 600rpx;
		margin-top: 20rpx;
		margin-left: 50rpx;
	}

	.lookedMiMa {
		color: rgb(170, 0, 255);
		width: 600rpx;
		margin-top: 20rpx;
		margin-left: 50rpx;
	}

	/* 通知公告栏 */
	.tongZhiGongGao {
		width: 90%;
		margin: 0 auto;
		padding: 10rpx 0 0 0;
	}

	.tongZhiGongGao .neiRong {
		padding-right: 20rpx;
		padding-top: 5rpx;
	}

	.tongZhiGongGao scroll-view {
		height: 800rpx;
	}
</style>