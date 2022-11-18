<template>
	<view>
		<view style="position: relative;" class="bg-box">
			<image :src="imgBg" style="width: 100%;" mode="widthFix"></image>
			<view style="position: absolute;bottom: -8px; background-color: #fff; width: 210px; border-radius: 50px;
				margin-left: 15px;padding: 4px 0 4px 10px;">
				<u--text :text="companyTitle" size="18"></u--text>
			</view>
		</view>
		<view class="content">
			<view style="margin: 10px 0;display: flex;justify-content: space-between;">
				<view>
					<view v-for="(item, index) in resumeList" :key="index">
						<view class="t-text"> {{ item }}{{ resume[index] }}</view>
					</view>
				</view>

				<view>
					<button class="custom-style" v-if="isLogin" open-type="share">
					</button>
					<button class="custom-style" v-else @click="showAsk = true">
					</button>
				</view>



			</view>
			<view class="t-button">
				<view v-for="(item, index) in btnList" :key="index">
					<view
						style="display: flex; align-items: center; justify-content: space-around; border: 2px solid #0089cd;border-radius: 50px; padding: 6px;">
						<image :src="item.url" style="width: 12px;height: 12px;"></image>
						<view style="font-size: 14px; padding-left: 4px;">{{ item.text }}</view>
					</view>
				</view>
			</view>
			<view style="width: 100%; height: 32px;margin-top: 10px;" class="bg-look"></view>
		</view>
		<vol-popup :showAsk="showAsk" :popupContent="popupContent" @cancel="cancelShowAsk" @confirm="wechatLogin">
		</vol-popup>
		<view v-if="flag">
			<wxml-to-canvas class="widget" id="widget" width="300" height="300"></wxml-to-canvas>
		</view>
	</view>
</template>

<script>
	export default {
		name: 'Home',
		data() {
			return {
				flag: false,
				isLogin: true,
				showAsk: false,
				popupContent: {
					title: '您还未登录',
					subTitle: '(请先登录再进行此操作)',
					cancel: '稍后登录',
					confirm: '登录'
				},
				imgBg: require('@/static/imgs/bg.png'),
				share: require('@/static/imgs/bb2.png'),
				companyTitle: '宁波德曼压缩机有限公司',
				btnList: [{
						text: '一键拨打',
						url: require('@/static/imgs/bb1.png')
					},
					{
						text: '添加通讯录',
						url: require('@/static/imgs/bb2.png')
					},
					{
						text: '地址导航',
						url: require('@/static/imgs/bb3.png')
					}
				],
				resume: {
					tel: '13805810569',
					email: 'hlq@deman1998.com',
					address: '浙江慈溪市匡堰越慈路188号',
					cn: 'www.deman1998.com'
				},
				resumeList: {
					tel: '手机：',
					email: '邮箱：',
					address: '地址：',
					cn: '官网：'
				}

			}
		},
		created() {

		},
		// 分享给朋友
		onShareAppMessage(options) {
			var that = this
			that.flag = true
			const cw = 140
			const ch = 112
			// const {wxml, style} = require('./demo.js')
			const widget = that.selectComponent('.widget')
			console.log('widget', widget)
			const cre = widget.renderToCanvas({
				wxml: `
				<view class="view">
				<image src="@/static/imgs/bb3.png" class="bg"></view>
				<text class="text">ok</text>
				`,
				style: {
					view: {
						width: cw,
						height: ch,
						display: 'flex',
						textAlign: 'center'
					},
					bg: {
						position: 'absolute',
						width: cw,
						height: ch,
						top: 0,
						left: 0
					},
					text: {
						fontSize: 9,
						color: 'red'
					}

				}
			})
			const re = widget.canvasToTempFilePath()
			// must return custom share data when user share.
			return {
				provider: "weixin",
				scene: "WXSceneSession",
				title: '您好，我是' + that.companyTitle + '，这是我的名片请惠存。',
				imageUrl: re.tempFilePath,
				// path: '/components/vol-list/detail/detail?id=' + that.newsInfo.id + '&openid=' + that.userInfo.openid,
				path: '/pages/home/home', // 默认是当前页面，必须是以‘/'开头的完整路径
				success(res) {
					console.log('分享成功', res)
				},
				fail(res) {
					console.log('分享失败', res)
				}
			}

		},
		// 分享到朋友圈，加上这个上面微信自带的按钮才会能选择
		onShareTimeline: function() {
			var that = this
			var query = {
				id: that.newsInfo.id,
				openid: that.userInfo.openid
			}
			return {
				title: that.newsInfo.title,
				query: query
			}
		},
		methods: {
			cancelShowAsk(v) {
				var that = this
				that.showAsk = v
			},
			wechatLogin() {
				var that = this
				that.showAsk = false
				uni.getUserProfile({ // 调起微信询问是否登录，拿到用户信息
					desc: '用于完善会员信息',
					lang: 'zh_CN',
					success: res => {
						if (res) {
							console.log('成功', res)
							res.userInfo.news_id = that.detailId
							wx.login({ // 拿到code
								success(res2) {
									console.log('code', res2)
									if (res2) {
										this.onShareAppMessage()
									}
									// let params = {
									// 	userInfo: res.userInfo,
									// 	code: res2.code
									// }
									// 						that.http.post("/User/login", params, true)
									// 							.then((result) => {
									// 								that.showAsk = false
									// 								that.loading = false;

									// 								if (result.code != 1) {
									// 									that.loading = false;
									// 									return that.$toast(result.msg)
									// 								}
									// 								uni.setStorageSync('userInfo', result.data)
									// 								// 成功后跳转,扫码进来和好友从分享进来
									// 								that.userInfo = result.data

									// 								if (result.data.phone) {
									// 									let url = '/pages/home/home'
									// 									if (that.detailId > 0) {
									// 										url =
									// 											'/components/vol-list/detail/detail?id=' +
									// 											that.detailId
									// 									}

									// 									uni.reLaunch({
									// 										url: url
									// 									})
									// 								}
									// 							});
								}
							})

						}
					},
					fail: err => {
						console.log('err', err)
					}
				})
			},

		}


	}
</script>
<style lang="scss" scoped>
	.t-text {
		color: #0089cd;
		font-size: 14px;
		margin-bottom: 4px;
	}



	.bg-look {
		background-image: url('../../static/imgs/look.png');
		background-size: 100% 100%;
		background-position: center center;
		background-repeat: no-repeat;
	}

	.t-button {
		display: flex;
		justify-content: space-around;
	}

	.content {
		width: 90%;
		margin: 0 5%;
	}

	.custom-style {
		width: 50px;
		height: 50px;
		background-image: url('../../static/imgs/share.png');
		background-size: 100% 100%;
		background-position: center center;
		background-repeat: no-repeat;
	}
</style>
