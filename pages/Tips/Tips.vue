<template>
	<!-- 标题 -->
	<view class="title">
		{{tipDetail.title}}
	</view>
	<!-- 信息区 -->
	<view class="info">
		<!-- 头像 -->
		<view class="touXiang">
			<image src="../../static/luntan.png" mode="aspectFit"></image>
		</view>
		<view class="userInfo">
			<!-- 用户名 -->
			<view class="userName">
				同学&nbsp;&nbsp;{{tipDetail.userName}}
			</view>
			<!-- 时间信息 -->
			<view class="upLoadTime">
				上传&nbsp;:{{tipDetail.upLoadTime}}
			</view>
		</view>
	</view>

	<!-- 主要内容 -->
	<!-- 我拉个最新版本 -->
	<view class="content">
		{{tipDetail.context}}
	</view>
	<!-- 图片展示 -->
	<view class="photoList">
		<image class="tipImage" mode="widthFix" v-for="Tieimage, index1 in tipDetail.photoUrlList" :key="index1"
			:src="this.photoServerUrl+Tieimage"></image>
	</view>
	<!-- 功能区域 -->
	<view class="function">
		<!-- 帖子计数 -->
		<view class="tipCount">
			<image src="../../static/tip/tipsCount.png" mode="aspectFit"></image>
			<view class="">
				{{tipReplyDetail.length}}
			</view>
		</view>
		<!-- 刷新 -->
		<view class="refresh" @click="callUpReplyBox(item,1)">
			<image src="../../static/tip/reply.png" mode="aspectFit"></image>
			<view class="">
				回复
			</view>
		</view>
		<!-- 举报 -->
		<view class="report">
			<image src="../../static/tip/jubao.png" mode="aspectFit"></image>
			<view class="">
				举报
			</view>
		</view>
	</view>
	<!-- 分割线 -->
	<uv-divider text=""></uv-divider>
	<!-- 回复区域 -->
	<view @click="callUpReplyBox(item,2)" class="replyZone" v-for="(item,index) in tipReplyDetail" :key="index">
		<!-- 某回复区块 -->
		<view class="replyInfo1">
			<view class="replyInfo2">
				<!-- 用户名 -->
				<view class="userName">
					同学&nbsp;&nbsp;{{item.replyUserName}}
				</view>
				<!-- 时间信息 -->
				<view class="upLoadTime">
					第{{index+1}}楼&nbsp;回复时间&nbsp;:{{item.replyTime}}
				</view>
			</view>
			<!-- 对回复进行回复按钮 -->
			<view class="replyToReplyBtn">
				回复
			</view>
		</view>
		<!-- 回复内容 -->
		<view class="replyContent">
			{{item.replyContext}}
		</view>
		<!-- 对帖子某回复的楼中楼进行vfor -->
		<view class="replyToReplyList" >
			<view class="replyToReplyInfo" v-for="(item,index2) in item.tipReplyReplyList" :key="index2">
				同学&nbsp;{{item.replyUserName}}&nbsp;:&nbsp;{{item.context}}
			</view>
		</view>
		<!-- 分割线 -->
		<uv-divider text=""></uv-divider>
	</view>
	<!-- 回复弹窗 -->
			<uni-popup ref="reply" type="bottom">

				<view class="replyBox">
					<view class="replyplaceHolder">
						{{replyPlaceHolder}}
					</view>
					<textarea @input="replyContentSettings" class="replyBoxStyle" auto-focus="true" placeholder="{{replyPlaceHolder}}"
						placeholder-style="color: #cccccc" />
					<!-- 底部确定按钮 -->
					<view @click="reply()" class="yue-base-button">
						<view>回复</view>
					</view>
				</view>
			</uni-popup>
</template>

<script>
		import calculateWeek from "../../utils/calculateWeek.js"
	export default {
		data() {
			return {
				tipDetail: [], //帖子内容
				tipReplyDetail: [],
				photoServerUrl: "http://124.220.207.245:8080/images/" ,//图片服务器,
				replyPlaceHolder:""
			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad: function(options) {
			var that = this;
			//获取localStorage的openid
			that.tipDetail = uni.getStorageSync("tipDetail");
			that.openid=uni.getStorageSync("openid")
			that.userName=uni.getStorageSync("forumUsername")
			console.log(that.tipDetail)
			//开始获取帖子Reply
			that.getTipDetailReply(that.tipDetail);
		},
		/**
		 * 生命周期函数--监听页面初次渲染完成
		 */
		onReady: function(e) {},
		onPullDownRefresh(e) {
			var that = this;
			console.log("用户下拉论坛刷新")
			uni.redirectTo({
				url: "/pages/Tips/Tips"
			})
		},
		methods: {
			// 唤起回复框
						callUpReplyBox(res,status){
							var that=this;
							console.log(res)
							//用户选择创建帖子
							if(status==1){
							
								that.replyPlaceHolder="回复帖子"
								that.replyStatus=1;
							}
							//用户点击的是回复某个帖子
							if(status==2){
								that.tipReplyId=res.tipReplyId
								that.replyPlaceHolder="回复同学 "+res.replyUserName+":"+res.replyContext
								that.replyStatus=2;
							}
							this.$refs.reply.open();
						},
						// 提交回复内容(略，你应该写有)
						reply(res){
							var that=this;
							//对帖子进行回复
							if(that.replyStatus==1){
								uni.request({
									url: "https://neeeeeeebs.top:8088/chatArea/wechat/upReplyToTip",
									header: {
										'content-type': 'application/x-www-form-urlencoded'
									},
									method: 'POST',
									data: {
										"tipId": that.tipDetail.tipId,
										"replyUserOpenid":that.openid,
										"replyUserName":that.userName,
										"replyContext":that.replyContent,
										"replyTime":new Date().valueOf()
									},
									success: (res) => {
										if (res.data.a) {
											uni.showToast({
												title:"回复成功",
												duration:2000
											})
											this.$refs.reply.close();
											//成功后，重新获取帖子
											that.getTipDetailReply(that.tipDetail);
										}
									}
								})
							}
							//对某个回复进行楼中楼回复
							if(that.replyStatus==2){
								uni.request({
									url: "https://neeeeeeebs.top:8088/chatArea/wechat/upReplyReply",
									header: {
										'content-type': 'application/x-www-form-urlencoded'
									},
									method: 'POST',
									data: {
										"tipId":that.tipDetail.tipId,
										"tipReplyId": that.tipReplyId,
										"replyUserOpenid":that.openid,
										"replyUserName":that.userName,
										"replyContext":that.replyContent,
										"replyTime":new Date().valueOf()
									},
									success: (res) => {
										if (res.data.a) {
											uni.showToast({
												title:"回复成功",
												duration:2000
											})
											this.$refs.reply.close();
											//成功后，重新获取帖子
											that.getTipDetailReply(that.tipDetail);
										}
									}
								})
							}
						},
						// 回复内容更改
						replyContentSettings(res){
							this.replyContent = res.detail.value;
						},

			//获取帖子详细信息的帖子
			getTipDetailReply(item) {
				var that = this;
				uni.request({
					url: "https://neeeeeeebs.top:8088/chatArea/wechat/getTipContext",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					method: 'POST',
					data: {
						"tipId": item.tipId
					},
					success: (res) => {
						if (res.data.a) {
							// 把时间戳转换一下
							for (let i = 0; i < res.data.data.length; i++) {
								//转换他的更新时间1687755158000 1687706306351000
								let currentTime = new Date().valueOf()
								// console.log(currentTime)
								if ((currentTime - res.data.data[i].replyTime) / 1000 < 600) res.data.data[i]
									.replyTime = "十分钟内"
								if ((currentTime - res.data.data[i].replyTime) / 1000 < 1800) res.data.data[i]
									.replyTime = "三十分钟内"
								if ((currentTime - res.data.data[i].replyTime) / 1000 < 3600) res.data.data[i]
									.replyTime = "一小时内"
									if ((currentTime - res.data.data[i].replyTime) / 1000 < 43200) res.data.data[i]
										.replyTime = "半天之内"
								if ((currentTime - res.data.data[i].replyTime) / 1000 < 86400) res.data.data[
									i].replyTime = "一天之内"
								// 如果比一天还多 这条可以将时间戳转换成日期格式
								if ((currentTime - res.data.data[i].replyTime) / 1000 > 86400) res.data.data[
									i].replyTime = calculateWeek.formatTime(res.data.data[i].replyTime);
									
								// 无论怎么样，upLoadTime也要转化
								res.data.data[
									i].upLoadTime = calculateWeek.formatTime(res.data.data[i].upLoadTime);
							}
							that.tipReplyDetail = res.data.data
							console.log(that.tipReplyDetail)
							
						}
					}
				})
			}

		}
	}
</script>

<style>
	/* 全局 */
	page {
		z-index: 0;
		background-color: #f5f5f5;
		width: 95%;
		margin-left: 2.5%;
	}

	.info {
		display: flex;
		margin-top: 30rpx;
	}

	.userInfo {
		margin-left: 10rpx;
		width: 90%;
	}

	.userName {
		color: sienna;
	}

	.upLoadTime {
		color: cornflowerblue;
	}

	.touXiang {
		width: 10%;
	}

	.touXiang image {
		width: 65rpx;
		height: 65rpx;
	}

	/* 标题 */
	.title {
		margin-top: 30rpx;
		font-size: 45rpx;
		font-weight: 600;
	}

	/* 用户名 */
	.userName {}

	/* 主要内容 */
	.content {
		margin-top: 30rpx;
	}

	.photoList {
		margin-top: 30rpx;
	}

	/* 图片 */
	.tipImage {
		width: 100%;
	}

	/* 功能区 */
	.function {
		margin-top: 30rpx;
		display: flex;
	}

	.function .tipCount {
		width: 33%;
		text-align: center;

	}

	.function .refresh {
		width: 33%;
		text-align: center;

	}

	.function .report {
		width: 33%;
		text-align: center;

	}

	.function image {
		width: 65rpx;
		height: 65rpx;
	}
	
	/* 回复总设置 */
	.replyZone{
		
	}
		
	.replyInfo1{
		display: flex;
	}
	.replyInfo2{
		width: 80%;
	}
		
	.replyToReplyBtn{
		width: 20%;
	}
	.replyContent{
		margin: 10rpx 0 10rpx 0;
	}
	.replyToReplyList{
		    background-color: #9bb3b747;
		    width: 90%;
		    border-radius: 10rpx;
	}
	.replyToReplyInfo{
		margin: 5rpx 0 5rpx 10rpx;
	}
	.replyBox{
			padding: 25rpx;
			background-color: #ffffff;
			height: 500rpx;
		}
		
		.replyBox .replyBoxStyle{
			height: 60%;
			width: 100%;
			border-radius: 10rpx;
			background-color: #e3e3e3;
			
		}
		/* 回复框 */
		.yue-base-button {
			margin-top: 30rpx;
			display: flex;
			align-items: center;
			justify-content: center;
			bottom: 0;
			width: 100%;
			padding: 40rpx 0;
			z-index: 3;
		}
		
		.yue-base-button view {
			width: 560rpx;
			text-align: center;
			height: 96rpx;
			line-height: 96rpx;
			border-radius: 96rpx;
			font-size: 32rpx;
			font-weight: 400;
			color: #FFFFFF;
			background: #07C062;
		}
		.replyplaceHolder{
			    color: grey;
			    margin: 0 0 10rpx 0;
		}

</style>