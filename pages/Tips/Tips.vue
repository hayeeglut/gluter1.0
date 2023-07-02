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
		<view class="refresh">
			<image src="../../static/tip/reply.png" mode="aspectFit"></image>
			<view class="">
				刷新
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
	<view class="replyZone" v-for="(item,index) in tipReplyDetail" :key="index">
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
</template>

<script>
	export default {
		data() {
			return {
				tipDetail: [], //帖子内容
				tipReplyDetail: [],
				photoServerUrl: "http://124.220.207.245:8080/images/" //图片服务器
			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad: function(options) {
			var that = this;
			//获取localStorage的openid
			that.tipDetail = uni.getStorageSync("tipDetail");
			console.log(that.tipDetail)
			//开始获取帖子Reply
			that.getTipDetailReply(that.tipDetail);
		},
		/**
		 * 生命周期函数--监听页面初次渲染完成
		 */
		onReady: function(e) {},
		methods: {
			//获取帖子详细信息的帖子
			getTipDetailReply(item) {
				var that = this;
				uni.request({
					url: "https://172.20.149.44:8088/chatArea/wechat/getTipContext",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					method: 'POST',
					data: {
						"tipId": item.tipId
					},
					success: (res) => {
						if (res.data.a) {
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
</style>