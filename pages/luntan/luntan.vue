<template>
	<!-- 发布帖子按钮 -->
	<view class="fabuBTN" @click="jumpToFaBu()">
		<image src="../../static/chatArea/fabu.png" mode="aspectFit">发帖</image>
	</view>
	<view style="height: 100%">
		<!-- 公告区 -->
		<uni-notice-bar @click="notice()" background-color="#ecf9ff" single="true" color="#2979FF"
			showGetMore="true" showIcon="true" text="论坛规则(暂行)"></uni-notice-bar>
			
		<!-- 通知公告弹出窗口 -->
		<uni-popup ref="notice" type="dialog">
			<uni-popup-dialog :type="msgType" cancelText="关闭" title="论坛守则(必看!!!)" @close="dialogClose" @confirm="dialogConfirm">
				<!-- 通知公告栏 -->
				<view class="tongZhiGongGao">
					<scroll-view scroll-y="true">
						<view>
							<view class="neiRong" v-for="item,key in chatAreaRules" :key="key">{{item}}</view>
						</view>
					</scroll-view>
				</view>
			</uni-popup-dialog>
		</uni-popup>
		
		
		<!-- 块级区域 -->
		<!-- 帖子区域 -->
		<view class="tieZiBaBa" v-for="(item,tipId) in chatAreaList" :key="tipId" @click="jumpToTip(item)">
			<view class="zongTieZi">
				<!-- 标题上面那个 -->
				<view class="upTitle">
					<!-- 帖子id -->
					<view class="upTitle_id">
						<text style="font-weight: bold;">帖子ID: </text>{{ item.tipId }}
					</view>
					
					<!-- 时间 -->
					<view class="lastTime">
						<text style="font-weight: bold;">更新时间: </text>{{ item.upDateTime }}
					</view>
				</view>

				<!-- 标题 -->
				<view class="title">
					{{ item.title }}
				</view>

				<view class="content">
					{{ item.context }}
				</view>

				<!-- 展示图片 -->
				<view class="photo">
					<image class="tipImage" mode="aspectFit" v-for="Tieimage, index1 in item.photoUrlList.slice(0, 2)"
						:key="index1" :src="this.photoServerUrl+Tieimage"></image>
				</view>

				<!-- 点赞区域 -->
				<view class="dianZan">
					<!-- 点赞图片 -->
					<!-- <image class="like_love" src="../../static/chatArea/love.png" mode="scaleToFill"></image> -->
					
					<!-- 点赞数量计数 -->
					<!-- <view class="like_number">
						{{ item.love }}
					</view> -->
					<!-- 评论数量计数 -->
					<view class="reply_number">
						<!-- 评论图片 -->
						<image class="like_love" src="../../static/chatArea/list.png" mode="scaleToFill"></image>
						<text style="float: right;margin: 0 10rpx 0 0;">{{ item.replyCount }}</text>
					</view>
					
					<!-- 帖子tag -->
					<view class="commonTag">
						<text style="font-weight: bold;"></text>{{ item.tag }}
					</view>


				</view>
			</view>
		</view>

		<!-- 发布标签按钮 -->
		<!-- 随便写了一点注释17:52 -->
		<navigator url="../Fabu/Fabu">
			<view class="faBu">
				<icon name="upgrade" size="80rpx" color="green"></icon>
			</view>
		</navigator>
	</view>
</template>


<script>
	import calculateWeek from "../../utils/calculateWeek.js"

	export default {
		data() {
			return {
				//聊天区集合
				chatAreaList: [],
				//分页数
				startPage: 0,
				//openid
				openid: "",
				chatAreaRules:[],//社区守则
				photoServerUrl: "http://124.220.207.245:8080/images/" //图片服务器

			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad: function(options) {
			var that = this;
			//获取localStorage的openid
			that.openid = uni.getStorageSync("openid");
			console.log(that.openid)
			if(that.openid==""){
				uni.reLaunch({
					url: "/pages/user/user"
				})
			}
			//首先获取分页帖子
			that.getTipsByPage()
			//获取社区规则
			that.getChatAreaRules()
		},
		/**
		 * 生命周期函数--监听页面初次渲染完成
		 */
		onReady: function(e) {
			console.log("页面加载完成")
		},
		/**
		 * 生命周期函数--监听页面显示
		 */
		onShow: function() {},
		/**
		 * 生命周期函数--监听页面隐藏
		 */
		onHide: function() {},
		/**
		 * 生命周期函数--监听页面卸载
		 */
		onUnload: function() {},
		/**
		 * 页面相关事件处理函数--监听用户下拉动作
		 */
		onPullDownRefresh: function() {},
		/**
		 * 页面上拉触底事件的处理函数
		 */
		onReachBottom: function() {
			var that = this;
			console.log("用户触底")
			that.startPage += 10;
			//拼接帖子
			that.getTipsByPage()
		},
		/**
		 * 用户点击右上角分享
		 */
		onShareAppMessage: function() {},
		// 下拉刷新事件
		onPullDownRefresh(e) {
			var that = this;
			console.log("用户下拉论坛刷新")
			uni.reLaunch({
				url: "/pages/luntan/luntan"
			})
		},
		methods: {
			// 通知显示与关闭
			notice(type) {
				this.msgType = type
				this.$refs.notice.open()
			},
			//获取论坛规则
			getChatAreaRules(){
				var that = this
				uni.request({
					url: "https://neeeeeeebs.fit:8088/tongZhi/getChatAreaRules",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					method: 'POST',
					success: (res) => {
						if(res.data.a){
							that.chatAreaRules=res.data.data;
							//当thatAreaRules准备好后调用notice，让用户强制浏览论坛守则
							var interval = setInterval(function(){
								if(that.chatAreaRules!=null)
								that.notice()
								//首先去结束定时器
								clearInterval(interval)
							},200)
						}
					}
				})
			},
			//跳转至发布帖子页面
			jumpToFaBu(){
				uni.navigateTo({
					url:"/pages/Fabu/Fabu"
				})
			},
			//跳转去详细帖子
			jumpToTip(item) {
				//直接把详细铁存到localStorage
				uni.setStorageSync("tipDetail", item)
				uni.navigateTo({
					url: "/pages/Tips/Tips"
				})
			},
			//分页获取聊天帖子集合
			getTipsByPage(e) {
				var that = this
				uni.request({
					url: "https://neeeeeeebs.fit:8088/chatArea/wechat/getTipsByPage",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					method: 'POST',
					data: {
						"openid": that.openid,
						"startPage": that.startPage
					},
					success: (res) => {
						console.log(res)
						if (res.data.a) {
							// that.chatAreaList=res.data.data
							//成功后需要把data的tag数据进行转换一下
							for (let i = 0; i < res.data.data.length; i++) {
								if (res.data.data[i].tag == 0) res.data.data[i].tag = "#聊天灌水"
								if (res.data.data[i].tag == 1) res.data.data[i].tag = "#寻物/失物"
								if (res.data.data[i].tag == 2) res.data.data[i].tag = "#跳蚤市场"
								if (res.data.data[i].tag == 3) res.data.data[i].tag = "#bug反馈"

								//转换他的更新时间1687755158000 1687706306351000
								let currentTime = new Date().valueOf()
								// console.log(currentTime)
								if ((currentTime - res.data.data[i].upDateTime) / 1000 < 600) res.data.data[i]
									.upDateTime = "十分钟内"
								if ((currentTime - res.data.data[i].upDateTime) / 1000 < 1800) res.data.data[i]
									.upDateTime = "三十分钟内"
								if ((currentTime - res.data.data[i].upDateTime) / 1000 < 3600) res.data.data[i]
									.upDateTime = "一小时内"
									if ((currentTime - res.data.data[i].upDateTime) / 1000 < 43200) res.data.data[i]
										.upDateTime = "半天之内"
								if ((currentTime - res.data.data[i].upDateTime) / 1000 < 86400) res.data.data[
									i].upDateTime = "一天之内"
								// 如果比一天还多 这条可以将时间戳转换成日期格式
								if ((currentTime - res.data.data[i].upDateTime) / 1000 > 86400) res.data.data[
									i].upDateTime = calculateWeek.formatTime(res.data.data[i].upDateTime);
									
								// 无论怎么样，upLoadTime也要转化
								res.data.data[
									i].upLoadTime = calculateWeek.formatTime(res.data.data[i].upLoadTime);
							}
							Array.prototype.push.apply(that.chatAreaList, res.data.data);
						}
					}
				})
			},
			getUserInfo(event) {
				console.log(event.detail);
			},
		}
	}
</script>

<style>
	page {
		z-index: 0;
		background-color: #f5f5f5;
	}
	
	/* 通知公告栏 */
	.tongZhiGongGao {
	width: 100%;
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
	/* 发布按钮 */
	.fabuBTN image{
		    width: 80rpx;
		    height: 80rpx;
		    position: fixed;
		    left: 85%;
		    top: 80%;
	}
	/* 图片展示 */
	.tieZiBaBa .photo {
		margin-top: 5px;
	}

	/* 帖子中每张图片的大小 */
	.tipImage {
		width: 50%;
	}

	.searchTab {
		padding-top: 10rpx;
		border-radius: 10rpx;
	}

	.van-tabs {
		padding: 10rpx 0 20rpx;
		width: 100%;
	}

	.pingLun {
		padding: 20rpx 0 20rpx;
		border-radius: 1px solid grey;
	}

	/* 帖子位置 */
	.tieZiBaBa {
		/* position: absolute; */
		width: 100%;
		height: auto;
		padding: 15rpx 0 0rpx;
	}

	.zongTieZi {
		/* width: 95%; */
		background-color: white;
		padding-left: 0rpx;
		margin: 0px 5px 5px 5px;
		border-radius: 10px;
	}

	/* 标题上面那个东西 */
	.upTitle {
		display: flex;
	}

	/* 时间 */
	.tieZiBaBa .upTitle .lastTime {

		font-size: 25rpx;
		color: #a9a9a9;
		position: relative;
		width: 50%;

	}

	/* 帖子id */
	.tieZiBaBa .upTitle .upTitle_id {

		font-size: 25rpx;
		color: #a9a9a9;
		position: relative;
		width: 20%;

	}

	/* 标题 */
	.tieZiBaBa .title {
		/* background-color: white; */
		border-radius: 10rpx;
		/* height: 60rpx; */
		margin-bottom: 10rpx;
		line-height: 60rpx;
		/* padding-left: 15rpx; */
		font-weight: bold;
		font-size: large;
		margin-top: 10rpx;
		padding-right: 15rpx;

		/* 展示一部分 */
		/* 数据展示 */
		overflow: hidden;

		text-overflow: ellipsis;

		display: -webkit-box;

		-webkit-line-clamp: 2;

		-webkit-box-orient: vertical;
	}

	.tieZiBaBa .content {
		z-index: 1;
		/* background-color: white; */
		border-radius: 10rpx;
		height: auto;
		/* margin: 10rpx 15rpx 30rpx 15rpx; */

		/* 控制内容展示多少行 */
		overflow: hidden;

		text-overflow: ellipsis;

		display: -webkit-box;

		-webkit-line-clamp: 4;

		-webkit-box-orient: vertical;
	}

	/* 点赞 */
	.dianZan {
		width: 95%;
		border-radius: 10rpx;
		line-height: 50rpx;
		padding-top: 10rpx;
		padding-bottom: 10rpx;
		padding-left: 20rpx;
		padding-right: 20rpx;
		display: flex;
		position: relative;
	}
	
	/* 主要tag */
	.dianZan .mainTag{
		width: 25%;
		text-align: center;
		background-color: lightgreen;
		border-radius: 15% / 50%;
		font-size: 30rpx;
	}
	
	/* 次要tag */
	.dianZan .commonTag{
		margin-left: 10%;
		width: 25%;
		text-align: center;
		background-color: #8fe4ff;
		border-radius: 2% / 50%;
		font-size: 30rpx;
	}
	
	/* 进入评论区旁边那个计数 */
	.dianZan .reply_number {
		width: 12%;
		font-size: 30rpx;
		/* text-align: center; */

	}

	/* 小爱心旁边那个计数 */
	.dianZan .like_number {
		margin-left: 1%;
		width: 20%;
		font-size: 30rpx;
	}

	/* 小爱心 */
	.dianZan .like_love {
		width: 45rpx;
		height: 100%;
		text-align: center;
	}

	/* 帖子最近更新时间 */
	.dianZan .like_upTime {
		margin-left: 10%;
		width: 20%;
		position: relative;
		font-size: 28rpx;
		color: #a9a9a9;
	}

	/* 发布按钮 */
	.faBu {
		width: 100rpx;
		height: 100rpx;
		position: fixed;
		/* float: right; */
		z-index: 3;
		top: 900rpx;
		left: 650rpx;
		flex-wrap: nowrap;
		text-align: center;
	}
</style>