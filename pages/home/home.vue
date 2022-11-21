<template>
	<view>
		<view style="position: relative;" class="bg-box">
			<image :src="imgBg" style="width: 100%;" mode="widthFix" :show-menu-by-longpress="true"
				@click="getPreviewImage(imgBg)"></image>
			<view style="position: absolute;bottom: -8px; background-color: #fff; width: 210px; border-radius: 50px;
				margin-left: 15px;padding: 4px 0 4px 10px;">
				{{ resume.organization }}
			</view>
		</view>
		<view class="content">
			<view style="margin: 10px 0;display: flex;justify-content: space-between;">
				<view>
					<view v-for="(item, index) in resumeList" :key="index">
						<view class="t-text"> {{ item }}{{ resume[index] }}</view>
					</view>
					<view class="t-text" @click="getWebview(resume.url)">官网：{{ resume.url }}</view>
				</view>

				<view>
					<button class="custom-style" open-type="share"></button>
					<!-- <button class="custom-style" v-else @click="showAsk = true">
					</button> -->
				</view>



			</view>
			<view class="t-button">
				<view v-for="(item, index) in btnList" :key="index">
					<view style="display: flex; align-items: center; justify-content: space-between; 
						border: 2px solid #0089cd;border-radius: 50px; padding: 6px;" @click="getBtn(item)">
						<image :src="item.url" style="width: 12px;height: 12px;"></image>
						<view style="font-size: 14px; padding-left: 4px;">{{ item.text }}</view>
					</view>
				</view>
			</view>
			<view style="margin-top: 10px;display: flex;justify-content: space-between;font-size: 12px; color: #bbb;">
				<view style="display: flex;align-items: center;">
					<view v-for="(item, index) in supportList" :key="index" style="display: flex; align-items: center;">
						<image :src="item.avatarUrl" style="width: 12px;height: 12px;margin-right: 4px;"></image>
					</view>
					<view>最近{{ look }}人浏览</view>
				</view>
				<view style="display: flex;" @click="getSupport">
					点赞 {{ zan }}
					<u-icon :name="isSupport?'thumb-up-fill':'thumb-up'" :color="isSupport?'#0089cd':'#bbb'"></u-icon>
				</view>
			</view>


		</view>
		<vol-popup :showAsk="showAsk" :popupContent="popupContent" @cancel="cancelShowAsk" @confirm="wechatLogin">
		</vol-popup>
		<!-- <view>
			<wxml-to-canvas class="widget" id="widget" width="300" height="300" style=""></wxml-to-canvas>
		</view> -->
	</view>
</template>

<script>
	import permision from "@/components/Android/permission.js";
	export default {
		name: 'Home',
		data() {
			return {
				userInfo: {},
				isLogin: false,
				shareOpenType: '',
				zan: 0,
				look: 0,
				isSupport: false,
				isWeb: false,
				showAsk: false,
				popupContent: {
					title: '您还未登录',
					subTitle: '(请先登录再进行此操作)',
					cancel: '稍后登录',
					confirm: '登录'
				},
				code: require('@/static/imgs/code.png'),
				imgBg: require('@/static/imgs/bg.png'),
				share: require('@/static/imgs/bb2.png'),
				btnList: [{
						type: 'call',
						text: '一键拨打',
						url: require('@/static/imgs/bb1.png')
					},
					{
						type: 'add',
						text: '添加通讯录',
						url: require('@/static/imgs/bb2.png')
					},
					{
						type: 'adress',
						text: '地址导航',
						url: require('@/static/imgs/bb3.png')
					}
				],
				resume: {
					organization: '宁波德曼压缩机有限公司',
					firstName: '何立庆',
					title: '直销中心总监',
					mobilePhoneNumber: '13805810569',
					email: 'hlq@deman1998.com',
					workAddressStreet: '浙江慈溪市匡堰越慈路188号',
					url: 'www.deman1998.com'
				},
				resumeList: {
					mobilePhoneNumber: '手机：',
					email: '邮箱：',
					workAddressStreet: '地址：'
					// url: '官网：'
				},
				supportList: [],
				supportFlag: false

			}
		},
		created() {

		},
		// 分享给朋友
		async onShareAppMessage(options) {
			// must return custom share data when user share.
			var that = this
			return {
				provider: "weixin",
				scene: "WXSceneSession",
				title: '您好，我是' + that.resume.organization + '，这是我的名片请惠存。',
				imageUrl: that.imgBg,
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
				title: '您好，我是' + that.resume.organization + '，这是我的名片请惠存。',
				query: query
			}
		},

		methods: {
			// 授权登录
			wechatLogin() {
				var that = this
				that.showAsk = false
				uni.getUserProfile({ // 调起微信询问是否登录，拿到用户信息
					desc: '用于完善会员信息',
					lang: 'zh_CN',
					success: res => {
						if (res) {
							console.log('成功', res)
							that.userInfo.news_id = res.userInfo.news_id
							that.userInfo.avatarUrl = res.userInfo.avatarUrl
							// userInfoList.push({})
							// console.log('that.userInfo', that.userInfo)
							// uni.setStorageSync('userInfos', that.userInfo)
							// console.log(uni.getStorageInfoSync('userInfos'))
							that.supportList.push(that.userInfo)
							wx.login({ // 拿到code
								success(res2) {
									console.log('code', res2)
									if (res2) {
										that.isLogin = true
										that.look = that.look + 1
										uni.showToast({
											text: '登录成功！'
										})
									}

								}
							})

						}
					},
					fail: err => {
						console.log('err', err)
					}
				})
			},
			// 取消登录
			cancelShowAsk(v) {
				var that = this
				that.showAsk = v
			},
			// 分享
			getShare() {
				if (this.isLogin) {
					this.shareOpenType = 'share'
				} else {
					this.showAsk = true
				}
			},
			// 拨号+通讯录+跳转
			getBtn(v) {
				var that = this
				if (v.type === 'call') {
					that.makeCall(that.resume.mobilePhoneNumber)
				} else if (v.type === 'add') {
					that.getAdd()
				} else if (v.type === 'adress') {
					that.getMap()
				}
			},
			//拨号
			makeCall: function(mobile) {
				//安卓端：询问电话权限
				switch (uni.getSystemInfoSync().platform) {
					case 'android':
						permision.requestAndroidPermission("android.permission.CALL_PHONE")
						break;
					case 'ios':
						permision.judgeIosPermission("record")
						permision.judgeIosPermission("camera")
						break;
				};
				this.makePhoneCall(mobile)
			},
			makePhoneCall: function(phoneParmas) {
				uni.makePhoneCall({
					phoneNumber: phoneParmas,
				});
			},
			// 通讯录
			getAdd() {
				var that = this
				uni.addPhoneContact({
					organization: that.resume.organization,
					title: that.resume.title,
					firstName: that.resume.firstName,
					mobilePhoneNumber: that.resume.mobilePhoneNumber,
					email: that.resume.email,
					workAddressStreet: that.resume.workAddressStreet,
					url: that.resume.url,
					success(res) {
						console.log('添加到通讯', res)

					},
					fail() {

					},
					complete() {

					}
				})
			},
			// 导航
			getMap() {
				uni.openLocation({
					latitude: 30.163972,
					longitude: 121.326373,
					address: '浙江慈溪市匡堰越慈路188号',
					success: function() {
						console.log('success');
					},
					fail: function() {
						uni.showToast({
							title: 'error'
						})
					}
				})
				// uni.getLocation({
				// 	type: 'gcj02', //返回可以用于uni.openLocation的经纬度
				// 	success: function(res) {
				// 		console.log('99999', res)
				// 		const latitude = res.latitude;
				// 		const longitude = res.longitude;
				// 		uni.openLocation({
				// 			latitude: latitude,
				// 			longitude: longitude,
				// 			success: function() {
				// 				console.log('success');
				// 			},
				// 			fail: function() {
				// 				uni.showToast({
				// 					title: 'error'
				// 				})
				// 			}
				// 		})
				// 	},

				// })
			},
			// 外部链接跳转
			getWebview(v) {
				uni.navigateTo({
					url: '/components/vol-webview/vol-webview?url=' + v
				})
			},
			// 点赞
			getSupport() {
				if (!this.isLogin) {
					this.showAsk = true
				} else {
					this.supportFlag = !this.supportFlag
					this.isSupport = !this.isSupport
					if (this.supportFlag) {
						this.zan = this.zan + 1
					} else if (!this.supportFlag) {
						this.zan = this.zan - 1
					}
				}
				// let params = {
				// 	id: id
				// }
				// that.http.get("/NewsComments/setInc", params).then(result => {
				// 	// 获取点赞列表
				// 	that.getList()
				// })
			},
			// 点赞列表
			getList() {
				var that = this;
				let params = {
					news_id: that.newsInfo.id
				}
				that.http.get("/NewsComments/index", params).then(result => {
					result.data.color = "#333"
					that.commentList = result.data;
					that.loading = false;
				})
			},
			//长按识别二维码
			getPreviewImage(e) {
				console.log('e', e)
				uni.previewImage({
					urls: ['@/static/imgs/imgBg.png'], // 数组
					current: '@/static/imgs/imgBg.png', // 数组中的第一张
					success: res => {
						console.log('res', res)
					},
					fail: err => {
						console.log('err', err)
					}
				})
			}
			// getImage() {
			// 	const cw = 140
			// 	const ch = 112
			// 	const widget = this.selectComponent('#widget')
			// 	const cre = widget.renderToCanvas({
			// 		wxml: `
			// 			<view class="view">
			// 			<
			// 			 src="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Flmg.jj20.com%2Fup%2Fallimg%2F1114%2F042421133312%2F210424133312-1-1200.jpg&refer=http%3A%2F%2Flmg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1671414695&t=0476dcc72a3dfdf6e035233a6fb37d95" class="bg">
			// 			</view>
			// 			<text class="text">ok</text>
			// 			`,
			// 		style: {
			// 			view: {
			// 				width: cw,
			// 				height: ch,
			// 				display: 'flex',
			// 				textAlign: 'center'
			// 			},
			// 			bg: {
			// 				position: 'absolute',
			// 				width: cw,
			// 				height: ch,
			// 				top: 0,
			// 				left: 0
			// 			},
			// 			text: {
			// 				fontSize: 9,
			// 				color: 'red'
			// 			}

			// 		}
			// 	})
			// 	const re = widget.canvasToTempFilePath()
			// 	console.log('返回', re)
			// 	this.testImg = re.tempFilePath
			// },


		}


	}
</script>
<style lang="scss" scoped>
	.t-text {
		color: #0089cd;
		font-size: 14px;
		margin-bottom: 4px;
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
