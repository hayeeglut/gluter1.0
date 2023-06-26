<template>
	<view>
		<!-- 头顶通知公告栏 -->
		<view>
			<uni-notice-bar @click="notice()" background-color="#ecf9ff" single="true" color="#2979FF"
				showGetMore="true" showIcon="true" text="通知公告栏！通知公告栏！通知公告栏！"></uni-notice-bar>
		</view>

		<view>
			<text class="Time_info" v-if="hours > 6 && hours < 11">早上好，</text>
			<text class="Time_info" v-else-if="hours > 11 && hours < 13">中午好，</text>
			<text class="Time_info" v-else-if="hours > 13 && hours < 18">下午好，</text>
			<text class="Time_info" v-else>晚上好，</text>
		</view>

		<view>
			<text class="Time_smallinfo" v-if="hours > 6 && hours < 11">一缕阳光穿过你的窗户，温暖地照在你的脸上，提醒你一天的开始。</text>
			<text class="Time_smallinfo" v-else-if="hours > 11 && hours < 13">享受美食的同时，也要坚持健康的饮食。</text>
			<text class="Time_smallinfo" v-else-if="hours > 13 && hours < 18">时光过得有些缓慢，但是不要放弃你的目标。</text>
			<text class="Time_smallinfo" v-else>现在是回家休息的时间，放松一下自己吧。</text>
		</view>

		<view class="personInfo">

			<view class="userInfo">
				<image class="image-bg" src="../../static/images/user/userinfobackground.jpg" mode="center"></image>
				<view>
					<image class="avatar" src="../../static/images/user/zhutou.jpg" mode="scaleToFill"></image>
				</view>
				<view class="username">用户名:{{forumUsername}}</view>
			</view>

			<view class="navMenu">
				<view class="navItem" v-for="(item,index) in serviceBarArray" :key="index">
					<view :class="'navContent ' + (active == index ? 'ac' : '')" @click="changeNav(index)">
						{{ item }}
					</view>
				</view>
			</view>



			<view @click="jump" class="keBiaoSimpleShow">
				<view class="simple_info">
					<text>今日课表</text>
				</view>
				<!-- 滚动条 -->
				<scroll-view scroll-y="true">
					<view :style="{'background-color':colorArrays[key]}" v-for="(item,key) in today_keBiao" :key="key"
						class="ke_info">
						<uni-row>
							<uni-col span="20">{{item.keChengMing}}</uni-col>
							<uni-col span="20">{{item.jiaoShi}}</uni-col>
							<uni-col span="20">第{{item.shangKeJieShu}},{{item.shangKeJieShu+1}}节</uni-col>
							<uni-col span="20">{{item.riQi}}</uni-col>
						</uni-row>
					</view>
				</scroll-view>
			</view>

			<!-- 个人小工具 -->
			<view style="display: flex">
				<!-- 小按钮 -->
				<view class="anNiu">
					<view v-for="item,key in tuBiao" :key="key" class="chengJi">
						<navigator :url="item.navigateUrl" hover-class="navigator-hover">
							<image :src="item.imgurl"></image>
							<view>{{item.name}}</view>
						</navigator>
					</view>
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
				<input type="text" @input="qieXueHao" class="qieXueHao" auto-focus="" placeholder="请输入学号" />
				<view class="qMiMa">密码:</view>
				<input type="text" @input="qieMiMa" :password="showPassword" class="qieMiMa" placeholder="请输入密码" />
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
			<uni-popup-dialog :type="msgType" cancelText="关闭" title="通知" @close="dialogClose" @confirm="dialogConfirm">
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
	</view>
</template>

<script>
	import colors from '../../utils/colors.js';
	import calculateWeek from '../../utils/calculateWeek.js';
	export default {
		data() {
			return {
				j_username: "",
				j_password: "",
				forumUsername: "请登录",
				weeklyData: [{}],
				openid: "",
				todayDay: 0,
				todayMonth: 0,
				day: 0,
				keInfoList: [],
				month: 0,
				hours: 0,
				active: 0,
				serviceBarArray: [
					'常用服务',
					'账号设置'
				],
				openBoolen: false,
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
				today_keBiao: [],
				JESSIONID: '',
				//图标信息
				tuBiao: [{
						"imgurl": "../../static/images/tool_images/chengJi.png",
						"name": "成绩查询",
						"navigateUrl": "../chengjichaxun/chengjichaxun"
					}, {
						"imgurl": "../../static/images/tool_images/kaoshi.png",
						"name": "考试查询",
						"navigateUrl": "../kaoshichaxun/kaoshichaxun"
					}, {
						"imgurl": "../../static/images/tool_images/xdjc.png",
						"name": "修读进程",
						"navigateUrl": "../xiudujincheng/xiudujincheng"
					}, {
						"imgurl": "../../static/images/tool_images/chengjichaxun.png",
						"name": "成绩订阅通知",
						"navigateUrl": "../cjdytongzhi/cjdytongzhi"
					}, {
						"imgurl": "../../static/images/tool_images/shezhi.png",
						"name": "设置",
						"navigateUrl": "../shezhi/shezhi"
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
						url: 'https://172.20.129.4:8088/wx_get_true_openid/get',
						header: {
							'content-type': 'application/x-www-form-urlencoded'
						},
						data: {
							"wx_login_code": event.code
						},
						success: (res) => {
							console.log(res, '获取openid')
							//获得token完成登录
							uni.setStorageSync('openid', res.data);
							that.inputopenid(res.data);
							console.log('Page Load')
							//去userinfo表查找是否存在用户
							uni.request({
								url: "https://172.20.129.4:8088/userInfo/wechat/find_user",
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
										uni.request({
											url: "https://172.20.129.4:8088/userInfo/wechat/getUserName",
											header: {
												'content-type': 'application/x-www-form-urlencoded'
											},
											method: 'POST',
											data: {
												"openid": that.openid
											},
											success: (res) => {
												uni.setStorageSync(
													'forumUsername',
													res.data.data);
												that.forumUsername =
													res.data.data;
											}
										})
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
			this.colorArrays = colors;
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
				// 1 获取单选框中的值
				// 2 把值赋值给 data 中的数据
				this.login_type = e.detail.value;
			},
			changePassword() {
				this.showPassword = !this.showPassword;
			},
			dialogClose() {
				console.log('点击关闭')
			},
			/**
			 * 更改导航菜单
			 */
			changeNav(index) {
				this.active = index;

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
			getSystemTime() {
				this.todayDay = new Date().getDate();
				this.todayMonth = new Date().getMonth() + 1;
				this.day = new Date().getDate();
				this.month = new Date().getMonth() + 1;
				this.hours = new Date().getHours();
			},
			//将获取通知公告请求独立
			getTongZhi() {
				uni.request({
					url: 'https://172.20.129.4:8088/tongZhi/getToolTop',
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
			jump() {
				uni.switchTab({
					url: "/pages/kebiao/kebiao"
				})
			},
			// 将弹出层的账号密码存储
			qieXueHao(res) {
				this.j_username = res.detail.value;

			},
			qieMiMa(res) {
				this.j_password = res.detail.value;
			},
			// 弹出层的确认按钮
			cunChuZHMM(res) {
				//将账号密码存入storage中，然后重载本页面
				uni.setStorageSync('j_username', this.j_username);
				uni.setStorageSync('j_password', this.j_password);
				this.createUser();
			},
			//创建用户并发送后台
			createUser(res) {
				uni.showLoading({
					title: '少女祈祷中...',
				})
				var that = this;
				uni.request({
					url: "https://172.20.129.4:8088/userInfo/wechat/createUser",
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
						console.log(res, '获得用户名')
						uni.hideLoading();
						if (res.data.a = true) {
							//说明后端返回true，用户创建成功
							//重新调用本页面
							uni.reLaunch({
								url: '/pages/user/user'
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
			// 将课表请求独立出来
			getKeBiao(res) {
				var that = this;
				uni.request({
					url: "https://172.20.129.4:8088/keBiao/wechat/getKeBiao",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					data: {
						"openid": that.openid,
					},
					method: 'POST',
					success: (result) => {
						console.log(result, '请求课表');
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

			//将获取本日课表独立出来
			getBenRiKeBiao(res) {
				var that = this;
				//开始对课表进行裁剪，获得当前日课表
				//这条函数可以用来获取当前日期-开学日期的间隔天数
				let phone_type = uni.getSystemInfoSync();
				if (phone_type.osName == 'ios') {
					var temp = Math.floor(((new Date().valueOf() - new Date("2023/2/19").valueOf()) / 1000 / 60 / 60 /
						24) / 7);
				} else {
					var temp = Math.floor(((new Date().valueOf() - new Date("2023-2-19").valueOf()) / 1000 / 60 / 60 /
						24) / 7);
				}
				console.log(temp);
				console.log(phone_type);
				//此时temp就是我们定位的那一周，不过我们依然用temp
				let a = new Array();
				//获取今天是星期几
				let xqj = calculateWeek.getWeekByDate(new Date());
				for (let j = 0; j < that.weeklyData[temp].keInfoList.length; j++) {
					if (that.weeklyData[temp].keInfoList[j].xingQiJi == xqj) {
						//说明这节课属于今天，给他存一下
						a.push(that.weeklyData[temp].keInfoList[j]);
					}
				}
				//成功拿到今天的数据,给他存一下
				this.today_keBiao = a;
			},
		},

	}
</script>

<style>
	.ac{
		border-bottom: 1px solid red;
		color: red;
		/* border-bottom: 1px solid #ffac05;
		color: #ffac05; */
	}

	.Time_info {
		font-size: 45rpx;
	}

	.Time_smallinfo {
		font-size: 25rpx;
	}

	/* 个人信息栏 */
	.personInfo {
		width: 100%;
		border-radius: 10rpx;
		float: left;
	}

	/* userInfo */
	.personInfo .userInfo {
		width: 100%;
		height: 500rpx;
		margin-top: 20rpx;
		color: white;
	}

	/* 背景图片 */
	.personInfo .userInfo .image-bg {
		border-top-left-radius: 4%;
		border-top-right-radius: 4%;
		position: absolute;
		z-index: -1;
		width: 100%;
	}

	/* 个人中心 */
	.personInfo .userInfo .avatar {
		width: 120rpx;
		height: 120rpx;
		border-radius: 50%;
		margin: 40rpx 0 0 320rpx;
		text-align: center;
	}

	.personInfo .userInfo .username {
		line-height: 50rpx;
		font-size: 30rpx;
		text-align: center;
	}

	/* 切换栏 */
	.navMenu {
		display: flex;
		white-space: nowrap;
		height: 80rpx;
		font-size: 35rpx;
	}

	.navItem {
		width: 40%;
		text-align: center;
		margin: 20rpx 12rpx 0rpx;
		padding: 5rpx 25rpx;
		line-height: 38rpx;
	}
	
	.navItem .navContent {
	    padding: 5rpx 20rpx;
	    line-height: 38rpx;
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

	/* 简化课表 */
	.keBiaoSimpleShow {
		background-color: aliceblue;
		height: 300rpx;
	}


	.keBiaoSimpleShow scroll-view {
		height: 85%;
	}

	/* 今日课表 */
	.keBiaoSimpleShow .simple_info {
		margin: 5rpx 5rpx 5rpx 5rpx;
		color: white;
		width: 100%;
		text-align: center;
	}

	.keBiaoSimpleShow .simple_info text {
		margin-left: 32rpx;
	}

	.keBiaoSimpleShow .ke_info {
		border-radius: 10rpx;
		padding-left: 10rpx;
		margin: 10rpx auto;
		width: 90%;
		height: auto;
		background-color: rgba(0, 0, 0, 0.432);
		/* border-radius:10px */
	}

	.keBiaoSimpleShow van-row {
		height: 15rpx;
	}

	.keBiaoSimpleShow van-col {
		padding: 5rpx 5rpx 5rpx 5rpx;
		color: white;
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