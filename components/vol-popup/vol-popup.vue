<template>
	<view>
		<!-- 弹框询问微信昵称, 头像授权 -->
		<u-popup :show="showAsk" mode="center">
			<view style="width: 520rpx; text-align: center;">
				<image src="@/static/imgs/popup.png" mode="widthFix" style="width: 100%;"></image>
				<view style="margin: 0 60rpx;">
					<u--text :text="popupContent.title" align="center" margin="10rpx 0" color="rgb(0,0,0)">
					</u--text>


					<u--text :text="popupContent.subTitle" align="center" margin="10rpx 0 0 0" size="12"
						color="rgb(121,121,121)">
					</u--text>
					<view style="margin: 40rpx 0;">
						<view style="margin-bottom: 10rpx;">
							<u-button type="primary" shape="circle" size="small" :text="popupContent.confirm"
								@click="confirm">
							</u-button>
						</view>
						<view style="border: 1px solid #eee; border-radius: 20px;">
							<u-button shape="circle" size="small" :text="popupContent.cancel" @click="cancel">
							</u-button>
						</view>
					</view>
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
