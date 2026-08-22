c:\InternshipProject\02_Badminton_26_8_7\bluetooth_app\pages\mode\mode.vue
<template>
	<view class="container">
		<view class="mode-display-section">
			<view class="mode-header">
				<text class="mode-title">当前模式</text>
				<text class="mode-status" :class="{ connected: isConnected }">
					{{ isConnected ? '已连接' : '未连接' }}
				</text>
			</view>
			<view class="mode-icon-wrap" :class="{ auto: isAutoMode }">
				<text class="mode-icon">{{ isAutoMode ? '🤖' : '🕹️' }}</text>
			</view>
			<text class="mode-text">{{ isAutoMode ? '自动模式' : '手动模式' }}</text>
			<text class="mode-desc">
				{{ isAutoMode ? '小车自动完成巡检、跟踪、吸取、返航等任务' : '通过APP手动控制小车移动' }}
			</text>
		</view>
		
		<view class="switch-section">
			<view class="switch-btn" :class="{ auto: isAutoMode }" @click="toggleMode">
				<text class="switch-icon">{{ isAutoMode ? '🔄' : '🔀' }}</text>
				<text class="switch-text">{{ isAutoMode ? '切换为手动模式' : '切换为自动模式' }}</text>
			</view>
		</view>
		
		<view class="info-section">
			<view class="info-title">模式说明</view>
			<view class="info-item">
				<text class="info-icon">🤖</text>
				<view class="info-content">
					<text class="info-label">自动模式</text>
					<text class="info-desc">小车自动完成巡检、跟踪移动、小球吸取、自动返航等任务，无需手动干预</text>
				</view>
			</view>
			<view class="info-item">
				<text class="info-icon">🕹️</text>
				<view class="info-content">
					<text class="info-label">手动模式</text>
					<text class="info-desc">通过APP蓝牙连接小车，使用方向按钮手动控制小车移动和转向</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import bluetoothManager from '@/utils/bluetooth.js'
	
	export default {
		data() {
			return {
				isAutoMode: false,
				isConnected: false
			}
		},
		onLoad() {
			this.isAutoMode = bluetoothManager.getState().isAutoMode
			
			this.isConnected = bluetoothManager.getIsConnected()
			bluetoothManager.addListener(this.onConnectionChange)
			bluetoothManager.addStateListener(this.onStateChange)
		},
		onUnload() {
			bluetoothManager.removeListener(this.onConnectionChange)
			bluetoothManager.removeStateListener(this.onStateChange)
		},
		methods: {
			onConnectionChange(device, isConnected) {
				this.isConnected = isConnected
			},
			
			onStateChange(state) {
				this.isAutoMode = state.isAutoMode
			},
			
			toggleMode() {
				if (!this.isConnected) {
					uni.showToast({ title: '请先连接蓝牙设备', icon: 'none' })
					return
				}
				
				const newMode = !this.isAutoMode
				
				uni.showLoading({ title: '切换中...' })
				
				bluetoothManager.sendCommand('modeSwitch', newMode).then(() => {
					uni.hideLoading()
					bluetoothManager.setState({ isAutoMode: newMode })
					uni.showToast({ 
						title: newMode ? '已切换为自动模式' : '已切换为手动模式', 
						icon: 'none' 
					})
				}).catch((err) => {
					uni.hideLoading()
					console.error('模式切换失败:', err)
					uni.showToast({ title: '切换失败', icon: 'none' })
				})
			}
		}
	}
</script>

<style lang="scss">
	.container {
		min-height: 100vh; background: #f5f5f5; padding: 30rpx;
	}
	.mode-display-section {
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		border-radius: 24rpx; padding: 40rpx; margin-bottom: 30rpx;
		display: flex; flex-direction: column; align-items: center;
	}
	.mode-header {
		width: 100%; display: flex; justify-content: space-between; align-items: center;
		margin-bottom: 30rpx;
	}
	.mode-title { font-size: 32rpx; color: #fff; font-weight: bold; }
	.mode-status {
		font-size: 24rpx; color: rgba(255,255,255,0.7);
		&.connected { color: #4caf50; }
	}
	.mode-icon-wrap {
		width: 160rpx; height: 160rpx; border-radius: 50%;
		background: rgba(255,255,255,0.2); display: flex; align-items: center;
		justify-content: center; margin-bottom: 20rpx;
		&.auto { background: rgba(76, 175, 80, 0.3); }
	}
	.mode-icon { font-size: 80rpx; }
	.mode-text { font-size: 40rpx; color: #fff; font-weight: bold; margin-bottom: 15rpx; }
	.mode-desc { font-size: 26rpx; color: rgba(255,255,255,0.8); text-align: center; }
	
	.switch-section { margin-bottom: 30rpx; }
	.switch-btn {
		display: flex; align-items: center; justify-content: center;
		padding: 40rpx; background: #fff; border-radius: 24rpx;
		transition: all 0.3s ease;
		&.auto { background: #4caf50; }
	}
	.switch-icon { font-size: 48rpx; margin-right: 20rpx; }
	.switch-text { font-size: 32rpx; color: #333; font-weight: 600; }
	.switch-btn.auto .switch-text { color: #fff; }
	
	.info-section {
		background: #fff; border-radius: 24rpx; padding: 30rpx;
	}
	.info-title { font-size: 32rpx; color: #333; font-weight: bold; margin-bottom: 30rpx; }
	.info-item {
		display: flex; align-items: flex-start; margin-bottom: 30rpx;
		&:last-child { margin-bottom: 0; }
	}
	.info-icon { font-size: 48rpx; margin-right: 20rpx; }
	.info-content { flex: 1; }
	.info-label { font-size: 28rpx; color: #333; font-weight: 600; display: block; margin-bottom: 8rpx; }
	.info-desc { font-size: 24rpx; color: #999; line-height: 1.6; }
</style>