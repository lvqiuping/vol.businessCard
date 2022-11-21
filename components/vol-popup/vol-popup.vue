<template>
	<view>
		<!-- 弹框询问微信昵称, 头像授权 -->
		<u-popup :show="showAsk" mode="center">
			<view style="width: 500rpx; height: 500rpx; text-align: center;border-radius: 20px;">
				<image src="@/static/imgs/popup.png" mode="aspectFill" style="width: 100%;height: 110px;"></image>

				<view style="margin: 0 60rpx;">
					<u--text :text="popupContent.title" align="center" margin="10rpx 0" color="rgb(0,0,0)">
					</u--text>


					<u--text :text="popupContent.subTitle" align="center" margin="10rpx 0 0 0" size="12"
						color="rgb(121,121,121)">
					</u--text>
					<view style="margin: 20rpx 0;">
						<u-button type="primary" shape="circle" size="small" :text="popupContent.confirm"
							@click="confirm">
						</u-button>
					</view>
					<u-button type="info" shape="circle" size="small" :text="popupContent.cancel" @click="cancel">
					</u-button>

				</view>
			</view>
		</u-popup>
	</view>
</template>

<script>
	export default {
		name: 'Popup',
		props: {
			showAsk: {
				type: Boolean,
				default: false
			},
			popupContent: {
				type: Object,
				default: null
			}
		},
		methods: {
			cancel() {
				this.$emit('cancel', false)
			},
			confirm() {
				this.$emit('confirm')
			}
		}
	}
</script>

<style>
</style>
