<template>
	<view>
		<u-action-sheet round="30" cancelText="取消" :actions="actionList" :title="actionInfos.title"
			:closeOnClickOverlay="actionInfos.closeOnClickOverlay" :show="showAction" @select="selectActionsSheet"
			@close="closeActionsSheet">
		</u-action-sheet>
	</view>
</template>

<script>
	export default {
		name: 'ActionSheet',
		props: {
			actionList: {
				type: Array,
				default: []
			},
			actionInfos: {
				type: Object,
				default: ''
			},
			showAction: {
				type: Boolean,
				default: false
			}
		},
		methods: {
			selectActionsSheet(index) {
				this.$emit('selectActionsSheet2', index)
			},
			closeActionsSheet() {
				this.$emit('closeActionsSheet2', false)
			}
		}

	}
</script>

<style>
</style>
