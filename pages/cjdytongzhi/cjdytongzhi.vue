<template>
	<view style="height: 100%">
	    <view>点击订阅后，即可在成绩出来后收到通知</view>
	    <view>
	        订阅状态查询:
	        <text :style="'color: ' + sub_status_color + ';'">{{sub_status}}</text>
	    </view>
	    <!-- 订阅按钮 -->
	    <view>
	        <button type="primary" @click="subscrip_button">订阅按钮</button>
	    </view>
	    <!-- 删除订阅按钮 -->
	    <view>
	        <button type="warn" @click="delete_subscrip">取消订阅按钮</button>
	    </view>
		<view style="font-size: 50rpx;font-weight: 600;" class="">
			操作指引
		</view>
		<view v-for="(item,tipId) in helpInfo" :key="tipId">
			{{item}}
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				sub_status: '查询中',
				sub_status_color: ''
			}
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad(options) {
			var that=this;
		    this.onLoadClone3389(options);
			that.getHelpInfo();
		},
		methods: {
			// 获取操作帮助
			getHelpInfo(){
			    var that = this;
			    uni.request({
			        // url: 'https://neeeeeeebs.top:8088/exam_query_test/wechat/delete_subscrip',
			        url: 'https://neeeeeeebs.top:8088/tongZhi/getCJDY',
			        header: {
			            'content-type': 'application/x-www-form-urlencoded'
			        },
			        method: 'POST',
			        success(res) {
			            if (res.data.a) {
							console.log(res.data.data)
							that.helpInfo=res.data.data;
							console.log(that.helpInfo)
			            }
			        }
			    });
			},
			/**
			 * 生命周期函数--监听页面加载
			 */
			onLoadClone3389(options) {
			    var that = this;
			    //获取openid
			    //你猜我为什么写得这么麻烦，没办法，一切原因起源于js是异步进行的
			    //上一步没执行完他会去走下一步，这就会导致很多问题，所以只能这么写
			    that.get_openid();
			},
			
			//成绩通知删除
			delete_subscrip() {
			    var that = this;
			    uni.request({
			        // url: 'https://neeeeeeebs.top:8088/exam_query_test/wechat/delete_subscrip',
			        url: 'https://neeeeeeebs.top:8088/exam_query_test/wechat/delete_subscrip',
			        header: {
			            'content-type': 'application/x-www-form-urlencoded'
			        },
			        data: {
			            openid: that.openid
			        },
			        method: 'POST',
			        success(res) {
			            if (res.data == true) {
			                that.sub_status = '未订阅';
			                that.sub_status_color = 'red';
			            }
			        }
			    });
			},
			
			//订阅状态查询方法
			sub_status_query() {
			    var that = this;
			    uni.request({
			        // url: 'https://neeeeeeebs.top:8088/exam_query_test/wechat/sub_status_query',
			        url: 'https://neeeeeeebs.top:8088/exam_query_test/wechat/sub_status_query',
			        header: {
			            'content-type': 'application/x-www-form-urlencoded'
			        },
			        data: {
			            openid: that.openid
			        },
			        method: 'POST',
			        success(res) {
			            if (res.data == 1){
			                that.sub_status = '已订阅';
			                that.sub_status_color = 'green';
						}
			            else if (res.data == 0) {
			                that.sub_status = '未订阅';
			                that.sub_status_color = 'red';
			            } else {
			                //这里负责写，想要说的话
			                that.sub_status = res.data;
			                that.sub_status_color = 'red';
			            }
			        }
			    });
			},
			
			//获取openid
			get_openid() {
			    var that = this;
			    uni.getStorage({
			        key: 'openid',
			        success(res) {
						that.openid = res.data;
						that.sub_status_query();
			        }
			    });
			},
			
			//订阅按钮确认
			subscrip_button() {
			    console.log('go');
			    var that = this;
			    //首先进行前端界面订阅确认
			    uni.requestSubscribeMessage({
			        tmplIds: ['KFhaNa-O2jMu736fvL0GU2cgvGFYkX7w8H-XNbks3Z4'],
			        success(res) {
			            //订阅确认完了再向后端发送请求
			            uni.request({
			                // url: 'https://neeeeeeebs.top:8088/exam_query_test/wechat/subscrip_confirm',
			                url: 'https://neeeeeeebs.top:8088/exam_query_test/wechat/subscrip_confirm',
			                header: {
			                    'content-type': 'application/x-www-form-urlencoded'
			                },
			                data: {
			                    openid: that.openid
			                },
			                method: 'POST',
			                success(res) {
			                    if (res.data == '1') {
			                        //说明订阅成功
			                        that.onLoadClone3389({});
			                    } else if (res.data == 'already confirm') {
			                        //请勿重复订阅
			                        console.log('请勿重复订阅');
			                    }
			                }
			            });
			        }
			    });
			}
		}
	}
</script>

<style>

</style>
