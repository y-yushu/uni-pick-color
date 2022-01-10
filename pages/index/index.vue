<template>
	<view class="home">
		<canvas id="canvas" />
		<button @click="test()">测试</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {}
		},
		onLoad() {

		},
		methods: {
			test() {
				console.log('测试')
				uni.showToast({
					title: '测试',
					duration: 2000
				})
			}
		}
	}
</script>

<style lang="scss">
	page,
	.home {
		width: 100%;
		height: 100%;
		background-color: #F0AD4E;

		#canvas {
			width: 200upx;
			height: 200upx;
			background-color: #2C405A;
		}
	}
</style>
