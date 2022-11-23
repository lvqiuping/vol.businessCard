<template>
	<view>
		<view style="position: relative;" class="bg-box">
			<image :src="cardData.header_img" style="width: 100%;" mode="widthFix"
				:show-menu-by-longpress="showmenubyLongpress" @click="getPreviewImage(cardData.header_img)"></image>
			<view style="position: absolute;bottom: -4px; background-color: #fff; border-radius: 50px; padding: 4px 12px;margin-left: 18px;
				font-size: 18px;
				">
				{{ cardData.company }}
			</view>
		</view>
		<view class="content">
			<view style="margin: 10px 0;display: flex;justify-content: space-between;">
				<view>
					<view class="t-text">手机：{{ cardData.phone }}</view>
					<view class="t-text">邮箱：{{ cardData.email }}</view>
					<view class="t-text">地址：{{ cardData.address }}</view>
					<view class="t-text" @click="getWebview(cardData.website)">官网：{{ cardData.website }}</view>
				</view>
				<view>
					<button class="custom-style" open-type="share" v-if="userInfo"></button>
					<button class="custom-style" @click="showAsk = true" v-else></button>
				</view>
			</view>
			<view class="t-button">
				<view v-for="(item, index) in btnList" :key="index">
					<view style="display: flex; align-items: center; justify-content: space-between; 
						border: 2px solid #0089cd;border-radius: 50px; padding: 4px 8px;" @click="getBtn(item)">
						<image :src="item.url" style="width: 12px;height: 12px;"></image>
						<view style="font-size: 14px; padding-left: 4px;">{{ item.text }}</view>
					</view>
				</view>
			</view>
			<view style="margin-top: 14px;display: flex;justify-content: space-between;font-size: 12px; color: #bbb;">
				<view style="display: flex;align-items: center;">
					<view style="margin-right: 4px;">
						<u-avatar-group :urls="userCardHistory" size="16" gap="0.1"></u-avatar-group>
					</view>
					<view>最近{{ cardData.user_card_history_count }}人浏览</view>
				</view>
				<view style="display: flex;" @click="getSupport">
					点赞 {{ cardData.user_card_support_count }}
					<u-icon :name="hasSupport?'thumb-up-fill':'thumb-up'" :color="hasSupport?'#0089cd':'#bbb'"></u-icon>
				</view>
			</view>
		</view>
		<vol-popup :showAsk="showAsk" :popupContent="popupContent" @cancel="cancelShowAsk" @confirm="wechatLogin">
		</vol-popup>
		<!-- 个性化海报分享 -->
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
				showmenubyLongpress: false,
				userInfo: null, // 一份存本地内存，一份存组件调用
				showAsk: false,
				popupContent: {
					title: '您还未登录',
					subTitle: '(请先登录再进行此操作)',
					cancel: '稍后登录',
					confirm: '登录'
				},
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
				timer: null,
				hasSupport: false,
				cardData: {},
				userCardHistory: [],
				startViewTime: 0,
				endViewTime: 0
			}
		},
		// 分享给朋友
		async onShareAppMessage(options) {
			// must return custom share data when user share.
			var that = this
			return {
				provider: "weixin",
				scene: "WXSceneSession",
				title: '您好，我是' + that.cardData.username + '，这是我的名片请惠存。',
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
				id: that.cardData.id,
				openid: that.userInfo.openid
			}
			return {
				title: '您好，我是' + that.cardData.username + '，这是我的名片请惠存。',
				query: query
			}
		},
		onLoad() {
			var that = this
			that.getCard()
			that.userInfo = uni.getStorageSync('userInfo')
			if (!that.userInfo) {
				that.timer = setTimeout(function() {
					that.showAsk = true
				}, 3000)
			}
		},
		// 	onShow() {
		// 		this.startViewTime = Date.now()
		// 	},
		// 	onHide() {
		// 		// 刷新浏览记录
		// 		console.log('this.userInfo', this.userInfo)
		// 		if (uni.getStorageSync('userInfo')) {
		// 			this.endViewTime = Date.now()
		// 			that.getLook()
		// }
		// },
		methods: {
			// 卡片信息
			getCard() {
				var that = this
				that.http.get("/card/read").then((result) => {
					console.log(result.data)
					that.cardData = result.data
					var a = that.cardData.userCardHistory
					var b = []
					a.forEach((item) => {
						b.push(item.user)
					})
					b.forEach((item) => {
						that.userCardHistory.push(item.header_img)
					})

					if (that.userInfo) {
						that.getHasSupport()
					}
				})
			},
			// 是否已点赞
			getHasSupport() {
				var that = this
				let params = {
					id: that.cardData.id,
					openid: that.userInfo.openid
				}
				that.http.get("/card/hasSupport", params).then((result) => {
					that.hasSupport = result.data
				})
			},
			// 授权登录
			wechatLogin() {
				var that = this
				that.showAsk = false
				uni.getUserProfile({ // 调起微信询问是否登录，拿到用户信息
					desc: '用于完善会员信息',
					lang: 'zh_CN',
					success: res => {
						if (res) {
							res.userInfo.card_id = that.cardData.id
							wx.login({ // 拿到code
								success(res2) {
									let params = {
										userInfo: res.userInfo,
										code: res2.code
									}
									that.http.post("/user/login", params, true)
										.then((result) => {
											if (result.code != 1) {
												return that.$toast(result.msg)
											}
											uni.showToast({
												title: '登录成功'
											})
											that.showAsk = false
											uni.setStorageSync('userInfo', result.data)
											that.userInfo = result.data
											that.getLook()
											that.getHasSupport()

										}).catch((err) => {
											console.log('err', err)
										});
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
			// 浏览记录
			getLook() {
				console.log('记录')
				var that = this
				let params = {
					id: that.cardData.id,
					openid: that.userInfo.openid,
					view_time: that.endViewTime - that.startViewTime
				}
				that.http.get("/card/setViewHistory", params)
			},
			// 拨号+通讯录+跳转
			getBtn(v) {
				var that = this
				if (!that.userInfo) {
					that.showAsk = true
					return
				}
				if (v.type === 'call') {
					that.makeCall(that.cardData.phone)
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
					organization: that.cardData.company,
					title: that.cardData.position,
					firstName: that.cardData.username,
					mobilePhoneNumber: that.cardData.phone,
					email: that.cardData.email,
					workAddressStreet: that.cardData.address,
					url: that.cardData.website,
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
				console.log(this.cardData)
				uni.openLocation({
					latitude: Number(this.cardData.latitude),
					longitude: Number(this.cardData.longitude),
					address: this.cardData.address,
					success: function() {
						console.log('success');
					},
					fail: function() {
						uni.showToast({
							title: 'error'
						})
					}
				})
			},
			// 外部链接跳转
			getWebview(v) {
				var that = this
				if (!that.userInfo) {
					that.showAsk = true
					return
				}
				uni.navigateTo({
					url: '/components/vol-webview/vol-webview?url=' + v
				})
			},
			// 点赞
			getSupport() {
				var that = this
				if (!that.userInfo) {
					that.showAsk = true
					return
				}
				if (!that.hasSupport) {
					let params = {
						id: that.cardData.id,
						openid: that.userInfo.openid,
						support: 1
					}
					that.http.post("/card/setInc", params).then(result => {
						that.hasSupport = true
						that.cardData.user_card_support_count++
					})
				} else {
					console.log('已点过赞')
				}

			},
			//长按识别二维码
			getPreviewImage(e) {
				var that = this
				if (!that.userInfo) {
					that.showAsk = true
					return
				} else {
					that.showmenubyLongpress = true
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
		justify-content: space-between;
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
		box-shadow: 0 4px 7px 0 rgba(0, 0, 0, 0.2)
	}
</style>
