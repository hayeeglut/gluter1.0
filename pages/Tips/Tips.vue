<template>
	
</template>

<script>
	export default {
		data() {
			return {

			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad: function(options) {
			var that=this;
			//获取localStorage的openid
			that.tipDetail=uni.getStorageSync("tipDetail");
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
			getTipDetailReply(item){
				var that=this;
				uni.request({
					url: "https://172.20.129.4:8088/chatArea/wechat/getTipContext",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					method: 'POST',
					data: {
						"tipId":item.tipId
					},
					success: (res) => {
						if(res.data.a){
							console.log(res.data.data)
						}
					}
				})
			}
			
		}
	}
</script>

<style>

</style>
