<template>
	<view class="biaoTou">
	    <text style="width: 27%">课程名称</text>
	    <text style="width: 10%">属性</text>
	    <text style="width: 12%">学分要求</text>
	    <text style="width: 12%">已获学分</text>
	    <text style="width: 12%">门数要求</text>
	    <text style="width: 12%">已通过数</text>
	    <text style="width: 15%">是否合格</text>
	</view>
	
	<view class="xdjcxx" v-for="(item, index) in xdjc" :key="index">
	    <text style="width: 27%">{{ item.keZuMingCheng }}</text>
	
	    <text style="width: 10%">{{ item.shuXing }}</text>
	
	    <text style="width: 12%">{{ item.xueFenYaoQiu }}</text>
	
	    <text style="width: 12%">{{ item.yiHuoXueFen }}</text>
	
	    <text style="width: 12%">{{ item.menShuYaoQiu }}</text>
	
	    <text style="width: 15%">{{ item.yiTongGuoMenShu }}</text>
	
	    <text :class="item.shiFouHeGe == '未通过' ? 'shiFouHeGe_no' : 'shiFouHeGe_yes'" style="width: 12%">{{ item.shiFouHeGe }}</text>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				xdjc: []
			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad(options) {
		    var that = this;
		    uni.showLoading({
		        title: '获取中...'
		    });
		    //首先需要获取用户的openid
		    var openid = uni.getStorageSync('openid');
		    //成功获取openid
		    uni.request({
		        // url: 'https://localhost:8088/xdjc/wechat/get',
		        url: 'https://172.20.129.4:8088/xdjc/wechat/get',
		        header: {
		            'content-type': 'application/x-www-form-urlencoded'
		        },
		        method: 'POST',
		        data: {
		            openid: openid
		        },
		        success(res) {
					console.log(res,'修读进程')
		            //说明获取成功,直接将修读进程进行存取
		            uni.hideLoading();
		            that.xdjc = JSON.parse(res.data.data);
		        }
		    });
		},
		methods: {}
	}
</script>

<style>
page {
    background-color: #f5f5f5;
}
.biaoTou {
    width: 100%;
    display: flex;
}
.biaoTou text {
    text-align: center;
    font-weight: 900;
}
.xdjcxx {
    width: 100%;
    display: flex;
}
.xdjcxx text {
    text-align: center;
}
.shiFouHeGe_no {
    color: red;
}
.shiFouHeGe_yes {
    color: green;
}
</style>
