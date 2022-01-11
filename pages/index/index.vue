<template>
	<view class="home">
		<!-- 文字 -->
		<view class="title">
			<span class="span">{{color}}</span>
			<view class="demo" :style="{backgroundColor: color}" @click="copy()" />
		</view>
		<!-- 操作条 -->
		<view class="buts">
			<span class="but1" @click="upfile()">
				上传
			</span>
			<span class="but2" :style="buttonStyle" @click="open()">
				拾色
			</span>
		</view>
		<!-- 图片 -->
		<canvas id="myCanvas" canvas-id="myCanvas" class="canvas" :style="canvasStyle" @touchstart="touchstart"
			@touchmove="touchmove">
		</canvas>
		<!-- 光标 -->
		<text v-show="anchorShow" class="anchor" :style="anchorStyle"></text>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				color: '#94b0ff',
				// canvas属性
				isImage: false, // 是否选择了图片
				canvasTop: 0,
				canvasWidth: 0,
				canvasHeight: 200,
				// 鼠标属性&定位
				isAnchor: false, // 是否可操作光标
				anchorShow: false, // 是否显示光标
				anchorTop: 300,
				anchorLeft: 300
			}
		},
		computed: {
			// 画布属性
			canvasStyle() {
				const hei = this.canvasHeight
				return {
					height: hei + 'px'
				}
			},
			// 拾色按钮样式
			buttonStyle() {
				const isAnchor = this.isAnchor
				return {
					color: isAnchor ? '#409EFF' : '#909399'
				}
			},
			// 光标属性
			anchorStyle() {
				const top = this.anchorTop
				const left = this.anchorLeft
				return {
					top: top + 'px',
					left: left + 'px'
				}
			}
		},
		mounted() {
			// 获取canvas属性
			this.reastCanvas()
		},
		methods: {
			// 复制当前颜色
			copy() {
				uni.setClipboardData({
					data: this.color
				})
			},
			// 开启&关闭拾色
			open() {
				if (this.isImage) {
					this.isAnchor = !this.isAnchor
				} else {
					uni.showToast({
						title: '请先上传图片',
						icon: 'error',
						duration: 2000
					})
				}
			},
			// 计算canvas属性
			reastCanvas() {
				uni.createSelectorQuery().in(this).select('#myCanvas').boundingClientRect(data => {
					this.canvasTop = data.top
					this.canvasWidth = data.width
					this.canvasHeight = data.height
					this.anchorShow = false // 画布计算后，隐藏光标
				}).exec()
			},
			// 上传或打开图片
			upfile() {
				uni.chooseImage({
					count: 1,
					success: (res) => {
						this.isImage = true // 上传图片成功标记
						this.isAnchor = true // 开启拾色状态
						this.canvasDraw(res.tempFilePaths[0])
					},
					fail: (err) => {
						console.log(err)
						uni.showToast({
							title: err || '选择图片异常',
							icon: 'error',
							duration: 2000
						})
					}
				})
			},
			// 绘制图形
			canvasDraw(file) {
				const canvas = uni.createCanvasContext('myCanvas')
				const cw = this.canvasWidth
				uni.getImageInfo({
					src: file,
					success: (img) => {
						const iw = img.width
						const ih = img.height
						const ch = parseInt(cw / iw * ih)
						this.canvasHeight = ch
						setTimeout(() => {
							canvas.drawImage(file, 0, 0, iw, ih, 0, 0, cw, ch)
							canvas.draw()
							// 重新计算定位
							setTimeout(() => this.reastCanvas(), 200)
						}, 200)
					}
				})
			},
			// 触摸开始
			touchstart(e) {
				if (this.isAnchor) {
					this.anchorShow = true
				}
			},
			// 触摸移动
			touchmove(e) {
				if (this.isAnchor) {
					// 触摸位置
					const x = e.changedTouches[0].x
					const y = e.changedTouches[0].y
					// canvas区域位置
					const ct = this.canvasTop
					const cw = this.canvasWidth
					const ch = this.canvasHeight
					const cb = ct + ch
					// 计算定位&赋值
					let left = x
					if (x > cw) left = cw
					if (x < 0) left = 0
					let top = y + ct
					if (y < 0) top = ct
					if (y > ch) top = cb
					this.anchorLeft = left
					this.anchorTop = top
					// 获取颜色&赋值
					uni.canvasGetImageData({
						canvasId: 'myCanvas',
						x,
						y,
						width: 1,
						height: 1,
						success: (res) => {
							this.color = RGBtoHEX(res.data[0], res.data[1], res.data[2])
						}
					})
				}
			}
		}
	}

	// rgb转色号
	const RGBtoHEX = (r, g, b) => {
		const to16 = (num) => {
			const str = num.toString(16)
			return str.length > 1 ? str : '0' + str
		}
		return '#' + to16(r) + to16(g) + to16(b)
	}
</script>

<style lang="scss">
	.home {
		width: 100%;
		height: 100%;
		overflow-x: hidden;
		overflow-y: hidden;

		// 顶部描述部分
		.title {
			width: 100%;
			height: 200upx;
			padding-top: 100upx;
			padding-bottom: 100upx;
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: space-around;

			.span {
				font-size: 48upx;
				letter-spacing: 6upx;
				font-weight: bold;
			}

			.demo {
				width: 40%;
				height: 140upx;
			}
		}

		// 操作条
		.buts {
			height: 120upx;
			padding-left: 20upx;
			padding-right: 20upx;
			display: flex;
			flex-direction: row;
			justify-content: space-between;
			align-items: center;

			.but1 {
				font-size: 32upx;
				color: #409EFF;
			}

			.but2 {
				font-size: 32upx;
			}
		}

		// 中部拾色部分 图片
		.canvas {
			width: 100%;
			padding: 0;
			margin: 0;
			background: url(@/static/nodata.png) no-repeat center;
		}

		.anchor {
			z-index: 1;
			width: 24rpx;
			height: 24rpx;
			border-radius: 50%;
			position: absolute;
			border: 4rpx solid #FFFFFF;
			background: rgba(0, 0, 0, .3);
			transform: translate(-50%, -50%);
		}
	}
</style>
