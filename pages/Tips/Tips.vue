<template>
	<!-- 标题 -->
	<view class="title">
		{{tipDetail.title}}
	</view>
	<!-- 用户名 -->
	<view class="userName">
		同学&nbsp;&nbsp;{{tipDetail.userName}}
	</view>
	<!-- 主要内容 -->
	<view class="content">
		{{tipDetail.context}}
	</view>
	<!-- 图片展示 -->
	<view class="photoList">
		<image class="tipImage" mode="aspectFit" v-for="Tieimage, index1 in tipDetail.photoUrlList"
			:key="index1" :src="this.photoServerUrl+Tieimage"></image>
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
	}

	/* 标题 */
	.title {
		font-size: 45rpx;
		font-weight: 600;
	}
</style>