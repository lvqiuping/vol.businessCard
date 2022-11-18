<template>
	<view class="page-container">
		<view v-show="userInfo.phone">
			<view>
				<view style="margin-bottom: 40rpx;">
					<u--text size="18" :text="newsInfo.title" bold></u--text>
				</view>
				<view>
					<u-row gutter="10" justify="space-between">
						<u-col span="4">
							<u--text type="success" size="12" :text="appName"></u--text>
						</u-col>
						<u-col span="8">
							<u--text type="info" size="12" :text="newsInfo.create_time"></u--text>
						</u-col>
					</u-row>
				</view>
			</view>
			<view class="content-info">
				<view style="font-size: 14px; line-height: 1.5; margin-bottom: 50rpx;">
					<u-parse :content="newsInfo.content"></u-parse>
				</view>
				<u-row gutter="10">
					<u-col span="5">
						<u-row gutter="10">
							<u-col textAlign="center" align="center" span="4" v-for="(item, index) in iconList" :key="index"
								@click="gridClick(item.value, index, item.name)">
								<view style="height: 44px;">
									<button :open-type="item.openType" class="u-reset-button"
										style="width: 56px; height: 44px; background: transparent; border: none">
										<u-icon :color="item.color" size="20" :name="item.icon"></u-icon>
										<u--text type="info" size="12" :text="item.name"></u--text>
									</button>
								</view>
							</u-col>
						</u-row>
					</u-col>
					<u-col span="4" offset="3" textAlign="center" justify="end">
						<view style="height: 44px;">
							<button class="u-reset-button"
								style="width: 56px; height: 44px; background: transparent; border: none">
								<u-icon size="20" name="eye" color="#909399"></u-icon>
								<u--text type="info" size="12" :text="newsInfo.read_count"></u--text>
							</button>
						</view>
					</u-col>
				</u-row>
			</view>
			<view class="footer-info">
				<view class="f-title">
					<u--text type="info" color="#909399" text="精选留言"></u--text>
					<u--text type="info" color="#909399" text="写留言" @click="getComment"></u--text>
				</view>
				<view class="f-list">
					<view v-for="(item, index) in commentList" style="display: flex; justify-content: space-between">
						<view>{{item.content}}</view>
						<view style="display: flex;" @click="support(item.id)">
							<u-icon :color="supportColor.color" size="22" name="thumb-up" />{{item.support}}
						</view>
					</view>
			
				</view>
			</view>
			
			<view style="height: 20rpx;"></view>
			<!-- 弹框 -->
			<u-popup :show="addComment" mode="bottom" @close="close" @open="open">
				<u--form labelPosition="left" :model="comment" ref="form1">
					<u-form-item prop="content" borderBottom>
						<u--textarea v-model="comment.content" placeholder="请输入内容"></u--textarea>
					</u-form-item>
				</u--form>
				<view style="margin: 60rpx 30rpx; display: flex;">
					<u-button icon="lock-open" size="mini" shape="circle" @click="cancel" type="info" text="取消">
					</u-button>
					<u-button icon="lock-open" size="mini" shape="circle" @click="submit" type="primary" text="确定">
					</u-button>
				</view>
			</u-popup>
		</view>
		<view v-show="!userInfo.phone">
			<u--text text="无权限查看" type="info"></u--text>
		</view>
		<u-popup :show="!userInfo.phone" mode="bottom" :round="10" closeable="true" @close="close" @open="open">
			<view class="slot-boxs">
				<view class="tips">
					是否确定继续前往查看更多内容？
				</view>
				<view class="slot-content">
					<view class="but1">
						<u-button type="info" @click="cancel">
							取消
						</u-button>
					</view>
					<view class="but1">
						<u-button type="primary" openType="getPhoneNumber" @getphonenumber="getPhoneNumber"
							class="but1">
							确定
						</u-button>
					</view>

				</view>
			</view>
		</u-popup>
	</view>
</template>

<script>
	const app = getApp()
	export default {
		data() {
			return {
				appName: '',
				addComment: false,
				loading: true,
				collect: false,
				newsInfo: {},
				iconList: [{
					name: "分享",
					icon: "share",
					value: "share",
					color: "#909399",
					openType: "share"
				}, {
					name: "收藏",
					icon: "star",
					value: "collect",
					color: "#909399",
					openType: ""
				}, {
					name: "点赞",
					icon: "thumb-up",
					value: "support",
					color: "#909399",
					openType: ""
				}],
				commentList: [],
				id: 0,
				page: 0,
				comment: {
					content: '',
				},
				supportColor: {
					color: "#909399"
				},
				params: {
					openid: ''
				},
				userInfo: null
			}
		},
		// 分享给朋友
		onShareAppMessage: function(options) {
			// must return custom share data when user share.
			return {
				title: '吕秋萍',
				path: '/components/vol-list/detail/detail?id=' + this.newsInfo.id + '&openid=' + this.$store.state
					.userInfo.openid,
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
			var testQuery = `id=` + that.newsInfo.id + '&openid=' + that.$store.state.userInfo.openid
			return {
				title: '吕秋萍小程序',
				query: testQuery
			}

		},
		onLoad(option) {
			var that = this;
			uni.setNavigationBarTitle({
				title: option.title
			})
			this.appName = app.globalData.appName
			that.newsInfo.id = option.id
		},
		onShow() {
			var that = this;
				this.userInfo = uni.getStorageSync('userInfo')
				// if (this.userInfo.phone) {
				// 	return
				// }
			let params = {
				id: that.newsInfo.id
			}
			// 初始化新闻内容
			that.http.get("/News/read", params).then(result => {
					result.data.content = this.unescapeEntity(result.data.content)
					that.newsInfo = Object.assign({}, that.newsInfo, result.data);
					that.loading = false;
				}),
				that.getList();
			that.gridClick('read_count'); // 进入页面就加一次阅读量
		},
		onUnload(){
			uni.reLaunch({
				url: '/pages/home/home'
			})
		},
		methods: {
			getComment() {
				this.addComment = true
				this.comment = {
					content: '',
				}
			},
			open() {
				// console.log('open');
			},
			close() {
				this.show = false
				this.addComment = false
			},
			support(id) {
				var that = this
				let params = {
					id: id
				}
				that.http.get("/NewsComments/setInc", params).then(result => {
					that.getList()
				})
			},
			gridClick(p, index, name = '') {
				var that = this;
				let params = {
					id: that.newsInfo.id
				}
				params[p] = 1
				that.http.get("/News/setInc", params).then(result => {
					if (!['share', 'read_count'].includes(p)) {
						uni.showToast({
							title: name + '成功'
						})
						that.iconList[index].color = "#FC5C5B"
					}
				})
			},
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
			submit() {
				var that = this
				let params = {
					news_id: that.newsInfo.id,
					content: that.comment.content
				}
				that.http.post("/NewsComments/save", params).then(res => {
					that.addComment = false
					that.getList()
				});
			},
			cancel() {
				this.addComment = false
				return
			},
			getPhoneNumber(e) {
				this.show = false
				if (e.detail.errMsg === 'getPhoneNumber:ok') {
					this.http.post('/User/getUserPhoneNumber', {
						code: e.detail.code
					}).then(res => {
						if (res.code === 1) {
							const userInfo = uni.getStorageSync('userInfo')
							userInfo.phone = res.data.phoneNumber
							uni.setStorageSync('userInfo', userInfo)
							this.userInfo = userInfo
							// uni.navigateTo({
							// 	url: '/components/vol-list/detail/detail?id=' + this.newsInfo.id +
							// 		"&title=" + this.newsInfo.title
							// })
						}
					})
				} else if (e.detail.errMsg === 'getPhoneNumber:fail user deny') {
					console.log('拒绝获得手机号')
				}
			},
			unescapeEntity(str) {
				var reg =
					/&(?:nbsp|#160|lt|#60|gt|62|amp|#38|quot|#34|cent|#162|pound|#163|yen|#165|euro|#8364|sect|#167|copy|#169|reg|#174|trade|#8482|times|#215|divide|#247);/g,
					entity = {
						'&nbsp;': ' ',
						' ': ' ',
						'&lt;': '<',
						'<': '<',
						'&gt;': '>',
						'&62;': '>',
						'&amp;': '&',
						'&': '&',
						'&quot;': '"',
						'"': '"',
						'&cent;': '￠',
						'¢': '￠',
						'&pound;': '£',
						'£': '£',
						'&yen;': '¥',
						'¥': '¥',
						'&euro;': '€',
						'€': '€',
						'&sect;': '§',
						'§': '§',
						'&copy;': '©',
						'©': '©',
						'&reg;': '®',
						'®': '®',
						'&trade;': '™',
						'™': '™',
						'&times;': '×',
						'×': '×',
						'&divide;': '÷',
						'÷': '÷'
					};
				if (!str) {
					return '';
				}
				str = str.toString();
				return str.indexOf(';') < 0 ? str : str.replace(reg, function(chars) {
					return entity[chars];
				});
			}
		}
	}
</script>

<style lang="less" scoped>
	.slot-boxs {
		height: 350rpx;
		padding: 20rpx;
	}
	
	.tips {
		color: rgba(0, 0, 0, .6);
		padding: 20rpx;
		margin-bottom: 60rpx;
	
	}
	
	.slot-content {
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
	}
	
	.but1 {
		width: 28%;
		margin-right: 30rpx;
		padding-bottom: 40rpx;
	
	}
	button::after {
		border: none;
	}

	.text_indent_two {
		text-indent: 10rpx;
	}

	.over_two_lines {
		display: -webkit-box;
		word-break: break-all;
		text-overflow: ellipsis;
		overflow: hidden;
		white-space: pre-line;
		-webkit-box-orient: vertical;
		-webkit-line-clamp: 2;
	}

	.page-container {
		padding: 10rpx;
		height: calc(100vh - 110rpx);
		overflow: scroll;
		color: #333333;

		.title-info {
			font-size: 40rpx;

			.t-title {
				text-align: center;
				margin-bottom: 40rpx;
			}

			.t-tips {
				font-size: 30rpx;
				color: #999999;
				display: flex;
				justify-content: space-between;

				.t-user {
					color: #FC5C5B;
				}
			}
		}


		.content-info {
			background-color: #E1EFFE;
			padding: 20rpx;
			margin: 40rpx 0;

			.c-title {
				color: #333333;
				font-size: 40rpx;
				text-align: center;
				padding: 30rpx 0 20rpx;
			}

			.c-content {
				padding: 24rpx;
			}

		}

		.c-bottom {
			display: flex;
			padding: 20rpx;
			justify-content: space-between;


			.grid-list {
				width: 75%;
				display: inline-block;

				.grid-item {
					margin-right: 20rpx;
					text-align: center;
					float: left;
				}

				.grid-icon {
					height: 80rpx;
					position: relative;
				}

				.grid-text {
					font-size: 24rpx;
					color: #7e7e7e;
				}
			}

			.c-count {
				display: flex;
				flex-direction: cloumn;
				align-items: flex-end;
			}
		}

		.footer-info {
			.f-title {
				display: flex;
				justify-content: space-between;
			}

			.f-list {
				color: #999999;
				margin: 20rpx 0;
			}

		}

		.u-po {
			height: 300rpx;

			.u-po-title {
				display: flex;
				font-size: 50rpx;
				color: #333333;
				margin: 20rpx 40rpx;
			}
		}
	}
</style>
<style scoped>
	.grid-list /deep/ .u-icon {
		flex-direction: column !important;
		position: absolute;
		top: 48%;
		left: 50%;
		transform: translate(-50%, -50%);
	}
</style>
