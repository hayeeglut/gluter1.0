<template>
	<view style="height: 100%">
		<!-- 公告区 -->
		<uni-notice-bar left-icon="volume-o" text="开发中，不定期更新"></uni-notice-bar>
		<!-- 块级区域 -->
		<!-- 		<view class=""></view> -->
		<!-- 帖子区域 -->
		<view class="tieZiBaBa" v-for="(item,tipId) in chatAreaList" :key="tipId">
			<view class="zongTieZi">
				<navigator :url="'../Tips/Tips?id=' + index" hover-class="navigator-hover" class="content">
					<!-- 标题上面那个 -->
					<view class="upTitle">
						<!-- 帖子id -->
						<view class="upTitle_id">
							{{ item.tipId }}
						</view>
						<!-- 时间 -->
						<view class="lastTime">
							{{ item.upLoadTime }}
						</view>
					</view>
					<!-- 标题 -->
					<view class="title">
						{{ item.title }}
					</view>

					<navigator :url="'../Tips/Tips?id=' + index" hover-class="navigator-hover" class="content">
						{{ item.context }}
					</navigator>

					<!-- 展示图片 -->
					<view class="photo">
						<image mode="scaleToFill" v-for="Tieimage, index1 in item.photoUrlList" :key="index1"
							:src="Tieimage"></image>
					</view>
				</navigator>

				<!-- 点赞区域 -->
				<view class="dianZan">
					<navigator :url="'../Tips/Tips?id=' + index" hover-class="navigator-hover" class="reply">
						<icon size="40rpx" name="chat-o" info=""></icon>
						<!-- 评论数量计数 -->
						<view class="reply_number">
							{{ item.replyCount }}
						</view>
					</navigator>

					<view class="like">
						<!-- 点赞数量计数 -->
						<view class="like_number">
							{{ item.love }}
						</view>
					</view>
					<!-- 帖子最近更新时间 -->
					<view class="like_upTime"></view>
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
	export default {
		data() {
			return {
				//聊天区集合
				chatAreaList:[],
				//分页数
				startPage:0,
				//openid
				openid:""
				
			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad: function(options) {
			var that=this;
			//获取localStorage的openid
			that.openid=uni.getStorageSync("openid");
			//首先获取分页帖子
			that.getTipsByPage()
			
			
		},
		/**
		 * 生命周期函数--监听页面初次渲染完成
		 */
		onReady: function(e) {},
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
		onReachBottom: function() {},
		/**
		 * 用户点击右上角分享
		 */
		onShareAppMessage: function() {},
		// 下拉刷新事件
		onPullDownRefresh(e) {
			this.onLoadClone3389(this.options, {});
		},
		methods: {
			//分页获取聊天帖子集合
			getTipsByPage(e){
				var that=this
				uni.request({
					url: "https://172.20.129.4:8088/chatArea/wechat/getTipsByPage",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					method: 'POST',
					data: {
						"openid": that.openid,
						"startPage":that.startPage
					},
					success: (res) => {
						console.log(res)
						if(res.data.a){
						that.chatAreaList=res.data.data
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
	}

	/* 标题上面那个东西 */
	.upTitle {
		display: flex;
	}

	/* 时间 */
	.tieZiBaBa .upTitle .lastTime {
		flex: 5;
		font-size: 25rpx;
		color: #a9a9a9;
		position: relative;
		left: 120rpx;
	}

	/* 帖子id */
	.tieZiBaBa .upTitle .upTitle_id {
		flex: 5;
		font-size: 25rpx;
		color: #a9a9a9;
		position: relative;
		left: 15rpx;
	}

	/* 标题 */
	.tieZiBaBa .title {
		/* background-color: white; */
		border-radius: 10rpx;
		/* height: 60rpx; */
		margin-bottom: 10rpx;
		line-height: 60rpx;
		padding-left: 15rpx;
		font-weight: bold;
		font-size: large;
		margin-top: 10rpx;
		padding-right: 15rpx;
	}

	.tieZiBaBa .content {
		z-index: 1;
		/* background-color: white; */
		border-radius: 10rpx;
		height: auto;
		padding: 10rpx 15rpx 30rpx 15rpx;
	}

	/* 图片展示 */
	.tieZiBaBa .photo {
		padding-left: 15 rpx;
		margin-left: 10rpx;
	}

	.tieZiBaBa .photo van-image {
		padding-right: 20rpx;
		/* border: 1px solid black; */
	}

	/* 点赞 */
	.dianZan {
		width: 95%;
		height: 70rpx;
		/* background-color: white; */
		border-radius: 10rpx;
		line-height: 70rpx;
		display: flex;
		position: relative;
		left: 20rpx;
	}

	/* 进入评论区旁边那个计数 */
	.dianZan .reply_number {
		width: 20%;
		font-size: 25rpx;
	}

	/* 小爱心旁边那个计数 */
	.dianZan .like .like_number {
		width: 20%;
		font-size: 25rpx;
	}

	/* 小爱心 */
	.dianZan .reply,
	.like {
		width: 18.5%;
		text-align: center;
		display: flex;
		/* padding-left: 80rpx; */
	}

	.dianZan .like van-icon:hover {
		color: red;
	}

	/* 帖子最近更新时间 */
	.dianZan .like_upTime {
		width: 20%;
		position: relative;
		font-size: 28rpx;
		color: #a9a9a9;
		left: 305rpx;
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