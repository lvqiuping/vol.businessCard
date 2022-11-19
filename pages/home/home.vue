<template>
	<view>
		<view style="position: relative;" class="bg-box">
			<image :src="imgBg" style="width: 100%;" mode="widthFix"></image>
			<view style="position: absolute;bottom: -8px; background-color: #fff; width: 210px; border-radius: 50px;
				margin-left: 15px;padding: 4px 0 4px 10px;">
				<u--text :text="resume.organization" size="18"></u--text>
			</view>
		</view>
		<view class="content">
			<view style="margin: 10px 0;display: flex;justify-content: space-between;">
				<view>
					<view v-for="(item, index) in resumeList" :key="index">
						<view class="t-text"> {{ item }}{{ resume[index] }}</view>
					</view>
					<view class="t-text" @click="getInfos(resume.url)">官网：{{ resume.url }}</view>
					<!-- <web-view :src="resume.url">{{resume.url}}</web-view> -->
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
					<view style="display: flex; align-items: center; justify-content: space-around; 
						border: 2px solid #0089cd;border-radius: 50px; padding: 6px;" @click="getBtn(item)">
						<image :src="item.url" style="width: 12px;height: 12px;"></image>
						<view style="font-size: 14px; padding-left: 4px;">{{ item.text }}</view>
					</view>
				</view>
			</view>
			<!-- <view style="width: 100%; height: 32px;margin-top: 10px;" class="bg-look"></view> -->
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
				isWeb: false,
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
				}

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

			getInfos(v) {
				console.log(v)
				uni.navigateTo({
					url: '/components/vol-webview/vol-webview?url=' + v
				})
			},
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

				// uni.chooseLocation({
				// 	success: function(res) {
				// 		console.log('success', res);
				// 	}
				// })


			},
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

			openMapRoute(lat, lon, cityName) {

				var url = '';

				if (plus.os.name == 'Android') {

					var hasBaiduMap = plus.runtime.isApplicationExist({

						pname: 'com.baidu.BaiduMap',

						action: 'baidumap://'

					});

					var hasAmap = plus.runtime.isApplicationExist({

						pname: 'com.autonavi.minimap',

						action: 'androidamap://'

					});

					var urlBaiduMap = 'baidumap://map/marker?location=' + lat + ',' + lon + '&title=' + cityName +
						'&src=婚梯';

					var urlAmap = 'androidamap://viewMap?sourceApplication=婚梯&poiname=' + cityName + '&lat=' +
						lat + '&lon=' + lon +

						'&dev=0';

					if (hasAmap && hasBaiduMap) {

						plus.nativeUI.actionSheet({

							title: '选择地图应用',

							cancel: '取消',

							buttons: [{

								title: '百度地图'

							}, {

								title: '高德地图'

							}]

						}, function(e) {

							switch (e.index) {

								case 1:

									plus.runtime.openURL(urlBaiduMap);

									break;

								case 2:

									plus.runtime.openURL(urlAmap);

									break;

							}

						});

					} else if (hasAmap) {

						plus.runtime.openURL(urlAmap);

					} else if (hasBaiduMap) {

						plus.runtime.openURL(urlBaiduMap);

					} else {

						url = 'geo:' + lat + ',' + lon + '?q=%e6%95%b0%e5%ad%97%e5%a4%a9%e5%a0%82';

						plus.runtime.openURL(url); //如果是国外应用，应该优先使用这个，会启动google地图。这个接口不能统一坐标系，进入百度地图时会有偏差

					}

				} else {

					// iOS上获取本机是否安装了百度高德地图，需要在manifest里配置，在manifest.json文件app-plus->distribute->apple->urlschemewhitelist节点下添加（如urlschemewhitelist:["iosamap","baidumap"]）

					plus.nativeUI.actionSheet({

						title: '选择地图应用',

						cancel: '取消',

						buttons: [{

							title: 'Apple地图'

						}, {

							title: '百度地图'

						}, {

							title: '高德地图'

						}]

					}, function(e) {

						console.log('e.index: ' + e.index);

						switch (e.index) {

							case 1:

								url = 'http://maps.apple.com/?q=%e6%95%b0%e5%ad%97%e5%a4%a9%e5%a0%82&ll=' +
									lat + ',' + lon +

									'&spn=0.008766,0.019441';

								break;

							case 2:

								url = 'baidumap://map/marker?location=' + lat + ',' + lon + '&title=' +
									cityName + '&src=婚梯';

								break;

							case 3:

								url = 'iosamap://viewMap?sourceApplication=婚梯&poiname=' + cityName +
									'&lat=' + lat + '&lon=' + lon + '&dev=0';

								break;

							default:

								break;

						}

						if (url != '') {

							plus.runtime.openURL(url, function(e) {

								plus.nativeUI.alert('本机未安装指定的地图应用');

							});

						}

					});

				}

			},
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
			//拨号
			makePhoneCall: function(phoneParmas) {
				console.log('电话', phoneParmas)
				uni.makePhoneCall({
					phoneNumber: phoneParmas,
				});
			},

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
