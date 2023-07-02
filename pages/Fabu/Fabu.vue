<template>
	<view class="content">
		<view class="container">
			<!-- 标题 -->
			<textarea placeholder="写一下标题- -" @blur="inputTitle" :focus="inputTitleFocus" :auto-height="true"
				:show-confirm-bar="false" maxlength="1500" v-model="form.title" class="post-title"></textarea>
			<!-- 输入框 -->
			<textarea placeholder="想说什么就说什么叭- -" @blur="inputBlur" :focus="inputFocus" :auto-height="true"
				:show-confirm-bar="false" maxlength="1500" v-model="form.content" class="post-txt"></textarea>

			<!-- 上传图片 -->
			<view class="img-list">
				<view v-for="(item,index) in form.imageFileList" :key='index' class="img-list-box">
					<image v-if="!form.video" :src="item.tempFilePath" class="img-item"></image>
				</view>
				<view v-if="form.imageFileList.length < 9" class="icon-camera" @tap="imageUpLoad()">
					<uni-icons type="camera-filled" size="27" color=#D3D4D6></uni-icons>
				</view>
			</view>

			<!-- 选择tag -->
			<view class="">
				<kevy-radio :items="items" :fontSize="26" color="#272727" :iconSize="26" iconColor="#4fc08d"
					:bordered="true" @click="myClick" :checkIndex="0"></kevy-radio>
			</view>

		</view>
		<!-- 选择板块 -->

		<!-- 底部确定按钮 -->
		<view @click="clickCreate()" class="yue-base-button">
			<view>确定发布~</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				// 默认输入框获得焦点
				inputFocus: true,
				inputTitleFocus: true,
				form: {
					title: "",
					content: '',
					address: '',
					imageFileList: [],
					tag:0
				},
				items: ["聊天灌水", "寻物启事/失物招领", "跳蚤市场", "bug反馈"],
				imageSuccessCount:0
			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad: function(options) {
			var that=this;
			//获取localStorage的openid
			that.openid=uni.getStorageSync("openid");
			console.log("openid："+that.openid)
			that.userName=uni.getStorageSync("forumUsername");
		},
		methods: {
			//测试图片选择
			imageUpLoad: function(res) {
				var that = this;
				wx.chooseMedia({
					count: 9,
					mediaType: ['image'],
					sourceType: ['album', 'camera'],
					maxDuration: 30,
					success(res) {
						//成功后存入本地
						that.form.imageFileList=res.tempFiles;
						console.log(that.form.imageFileList)
					}
				})
			},

			//点击事件
			myClick(idx) {
				var that=this;
				that.form.tag=idx;
			},

			//标题内容
			inputTitle(event) {
				this.inputTitleCursor = event.detail.cursor;
				this.inputTitleFocus = false;
			},
			// 文字内容
			inputBlur(event) {
				this.inputCursor = event.detail.cursor;
				this.inputFocus = false;
			},

			// 位置选择
			chooseLocation() {
				let that = this;
				uni.chooseLocation({
					success: function(res) {
						that.form.address = res.name;
					}
				});
			},
			//图片上传，多张
			upload_file: function(url,filePath,tipId) {
				var that = this;
				wx.uploadFile({
					url: url,
					filePath: filePath,
					name: 'upLoadFile',
					header: {
						'content-type': 'multipart/form-data'
					}, // 设置请求的 header
					formData: {
						tipId: tipId
					}, // HTTP 请求中其他额外的 form data
					success: function(res) {
						that.imageSuccessCount++;
						console.log("上传成功＋1")
					},
					fail: function(res) {}
				})
			},
			// 发布
			 clickCreate() {
				//加载动画
				uni.showLoading({
					title:"上传中..."
				})
				var that=this;
				console.log(that.form)
				/**
				 * 点击发布流程，首先向后端传送title，content，tag信息，等待后端回传创建好的tipId
				 * 然后再一张一张发送图片，图片需要附上tipId
				 * */
				 //首先判断title content 是不是空的
				 if(that.form.content==''){
					uni.showToast({
						title:"内容不能为空~",
						duration:2000
					}) 
				 }
				 if(that.form.title==''){
					 uni.showToast({
					 	title:"标题不能为空~",
					 	duration:2000
					 }) 
				 }
				 //内容齐全
				 else{
					 console.log("资料齐全，开始上传")
					 //首先进行title content推送
					 var that=this;
					 uni.request({
					 	url: "https://neeeeeeebs.top:8088/chatArea/wechat/upLoadChatTip",
					 	header: {
					 		'content-type': 'application/x-www-form-urlencoded'
					 	},
					 	method: 'POST',
					 	data: {
					 		openid:that.openid,
							title:that.form.title,
							userName:that.userName,
							context:that.form.content,
							tag:that.form.tag,
							upLoadTime:new Date().valueOf()
					 	},
					 	success: (res) => {
					 		if(res.data.a){
					 			console.log(res.data.data)
								//此时res.data.data就是返回的后端关于这条主键id
								//对图片进行逐张上传
								for(let i=0;i<that.form.imageFileList.length;i++){
									that.upload_file("https://neeeeeeebs.top:8088/chatArea/wechat/upLoadImage",that.form.imageFileList[i].tempFilePath,res.data.data)
								}
								/**
								 * 粗暴办法，每500ms检查一次是否上传完毕，上传完毕就退出，不然永远不退出
								 * */
								var interval = setInterval(function(){
									console.log(that.imageSuccessCount)
									console.log(that.form.imageFileList.length)
									if(that.imageSuccessCount==that.form.imageFileList.length)
									that.checkUpSuccess()
									//首先去结束定时器
									clearInterval(interval)
								},500)
					 		}
					 	}
					 })
				 }
			},
			//判断上传是否成功，并执行动作
			checkUpSuccess(){
				var that=this;
					//说明所有图片上传成功
					//关闭加载动画
					uni.hideLoading();
					//展示toast表示上传成功
					uni.showToast({
						title:"已上传，待审核",
						duration:2000
					})
				//无论怎么样，结束然后返回论坛主界面
				setTimeout(function () {
					uni.navigateBack();
				}, 2000);
			},
		}
	}
</script>

<style lang="scss" scoped>
	.content {
		height: 100vh;
		background-color: #FFFFFF;
		border-radius: 30upx 30upx 0upx 0upx;
	}

	.container {
		padding: 20rpx 40rpx;
		overflow: hidden;
	}

	.post-title {
		width: 100%;
		min-height: 100rpx;
		line-height: 1rpx;
	}

	.post-txt {
		width: 100%;
		min-height: 300rpx;
		line-height: 1rpx;
	}

	/* 选择位置 */
	.choose-location {
		display: inline-flex;
		align-items: center;
		background-color: #eee;
		font-size: 28rpx;
		border-radius: 100rpx;
		padding: 10rpx 20rpx;
		margin-left: 5rpx;
		line-height: 1;
		color: #616161;

		.txt {
			margin-left: 10rpx;
		}
	}

	.yue-base-button {
		position: fixed;
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

	// 相机icon
	.icon-camera {
		display: flex;
		justify-content: center;
		align-items: center;
		width: 210rpx;
		height: 210rpx;
		border-radius: 6rpx;
		margin: 5rpx 0rpx 0rpx 5rpx;
		background-color: #f4f5f7;
	}

	// 媒体列表
	.img-list {
		display: flex;
		flex-wrap: wrap;
		margin-bottom: 20rpx;
	}

	.img-list-box {
		display: flex;
		justify-content: center;
		align-items: center;
		position: relative;
	}

	.img-item {
		width: 210rpx;
		height: 210rpx;
		margin: 5rpx;
		border-radius: 6rpx
	}
</style>