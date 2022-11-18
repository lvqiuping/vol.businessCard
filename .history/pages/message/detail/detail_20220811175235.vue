<template>
	<view class="page-container">
		<!-- <u-skeleton rows="15" :loading="loading" :title="false"> -->
		<view class="title-info">
			<view class="t-title over_two_lines">{{data.title}}</view>
			<view class="t-tips">
				<text
					class="t-user">{{data.creator}}</text><text>{{data.createDate}}</text><text>{{data.creator}}</text>
			</view>
		</view>
		<view class="content-info">
			<view class="c-title">东莞新病毒最新报告</view>
			<view class="text_indent_two" style="margin-bottom: 20rpx;">来源：今日头条</view>
			<view class="text_indent_two">来源：今日头条</view>
			<view class="text_indent_two">
				<u-parse :content="data.content"></u-parse>
			</view>
			<view class="c-bottom">
				<view class="grid-list" v-for="(item,index) in grid">
					<view class="grid-title">
						<view class="grid-title-text">
							{{item.name}}
						</view>
					</view>
					<view class="grid-item" @click="gridClick(data.path)" v-for="(data,dindex) in item.data"
						:key="dindex">
						<view class="grid-icon" :class="data.bg">
							<u-icon color="#333" size="22" :name="data.icon"></u-icon>
						</view>
						<view class="grid-text">
							{{data.name}}
						</view>
					</view>
				</view>
				<view class="c-count">阅读20万+</view>
			</view>
		</view>
		<view class="footer-info">
			<view class="f-title">
				<text>精选留言</text>
				<text>写留言</text>
			</view>
			<view class="f-list">
				<message-list :list="list"></message-list>
			</view>
		</view>

		<view style="height: 20rpx;"></view>
		<!-- </u-skeleton> -->
		<!-- 弹框 -->
		<u-popup :show="show" mode="bottom" @close="close" @open="open">
			<view class="u-po">
		 	<view class="u-po-title">东莞本地宝
					<u-icon name="arrow-right" color="#333333" size="20" style="margin-left: 20rpx;" />
				</view>
				<view class="c-bottom">
					<view class="grid-list" v-for="(item,index) in grid">
						<view class="grid-title">
							<view class="grid-title-text">
								{{item.name}}
							</view>
						</view>
						<view class="grid-item" @click="gridClick(data.path)" v-for="(data,dindex) in item.data"
							:key="dindex">
							<view class="grid-icon" :class="data.bg">
								<u-icon color="#333" size="22" :name="data.icon"></u-icon>
							</view>
							<view class="grid-text">
								{{data.name}}
							</view>
						</view>
					</view>
				</view>
			</view>
		</u-popup>
	</view>
</template>

<script>
	import messagelist from '@/components/vol-list/message-list.vue'
	export default {
		components: {
			'message-list': messagelist
		},
		data() {
			return {
				show: false,
				loading: true,
				data: {},
				grid: [{
					data: [{
						name: "分享",
						icon: "share",
						bg: "color1",
						path: "0"
					}, {
						name: "收藏",
						icon: "star",
						bg: "color2",
						path: "1"
					}, {
						name: "点赞",
						icon: "thumb-up",
						bg: "color3",
						path: "2"
					}]
				}],
				list: [],
				id: 0,
				page: 0,
			}
		},


		onLoad(option) {
			uni.setNavigationBarTitle({
				title: option.title
			})
			this.http.get("api/app_news/getDetail?id=" + option.id, {}, false).then(result => {
					this.data = result;
					this.loading = false;
				}),
				this.getList(); //初始列表
		},
		methods: {
			open() {
				// console.log('open');
			},
			close() {
				this.show = false
				// console.log('close');
			},
			gridClick(p) {
				if (p === '0') {
					this.show = true
				}
			},
			getList() {
				if (this.page < 0) {
					return;
				}
				this.page++;
				let url = 'api/app_news/getList?newsType=' + 1 + "&page=" + this.page;
				this.http.get(url, {}, false).then(result => {
					result.forEach(item => {
						item.imageUrl = this.http.ipAddress + item.imageUrl;
					})
					if (!result.length) {
						this.page = -1;
					}
					this.list.push(...result);
					this.loading = false;
				})
			},
		}
	}
</script>

<style lang="less" scoped>
	.text_indent_two {
		text-indent: 20rpx;
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
		margin: 20rpx 40rpx;
		height: calc(100vh - 110rpx);
		overflow: scroll;
		color: #333333;

		.title-info {
			font-size: 60rpx;

			.t-title {
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
			margin: 27rpx 0;

			.c-title {
				color: #333333;
				font-size: 40rpx;
				text-align: center;
				padding: 30rpx 0 20rpx;
			}




		}

		.c-bottom {
			display: flex;
			padding: 20rpx 0;

			.grid-list {
				width: 75%;
				display: inline-block;

				.grid-item {
					width: 20%;
					text-align: center;
					float: left;
				}

				.grid-icon {
					width: 80rpx;
					height: 80rpx;
					position: relative;
					left: 0;
					right: 0;
					margin: auto;
				}

				.color1 {
					// background-image: linear-gradient(#e8b752, #e8b752);
				}

				.grid-text {
					font-size: 24rpx;
					color: #7e7e7e;
				}
			}

			.c-count {
				align-self: end;
			}
		}

		.footer-info {
			.f-title {
				display: flex;
				justify-content: space-between;
			}

			.f-list {
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
