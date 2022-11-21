<template>
	<view>
		<view>
			<!-- url为要跳转外链的地址-->
			<web-view :src="url">
			</web-view>
		</view>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				url: ''
			}
		},
		onLoad(val) {
			// 传过来的地址一定要补全 https://， 网页会自动补全，但是小程序不会
			this.url = 'https://' + val.url
			// 设置当前的title 如果外链中有的话将被覆盖
			if (this.isNotEmpty(val.title)) {
				this.setTitle(val.title);
			}
		},
		methods: {
			isNotEmpty(obj) {
				if (typeof obj == undefined || obj == null || obj == "" || obj == "undefined" || obj.length == 0) {
					return false;
				} else {
					return true;
				}
			},
			// 设置title
			setTitle(title) {
				uni.setNavigationBarTitle({
					title: title
				})
			},
		}
	}
</script>
