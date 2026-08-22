c:\InternshipProject\02_Badminton_26_8_7\bluetooth_app\pages\index\index.vue
<template>
	<view class="container">
		<view class="header">
			<text class="title">羽毛球拾取小车</text>
			<text class="subtitle">智能控制中心</text>
		</view>
		
		<view class="status-card">
			<view class="status-row">
				<view class="status-dot" :class="{ active: isConnected }"></view>
				<text class="status-text">{{ isConnected ? '蓝牙已连接' : '蓝牙未连接' }}</text>
			</view>
			<view class="status-row" v-if="isConnected">
				<text class="mode-badge">{{ isAutoMode ? '自动模式' : '手动模式' }}</text>
			</view>
		</view>
		
		<view class="button-grid">
			<view class="btn-item" @click="goToBluetooth">
				<view class="btn-icon-wrap">
					<text class="btn-icon">🔗</text>
				</view>
				<text class="btn-text">蓝牙连接</text>
			</view>
			<view class="btn-item" @click="goToCountMonitor">
				<view class="btn-icon-wrap">
					<text class="btn-icon">📊</text>
				</view>
				<text class="btn-text">数量监控</text>
			</view>
			<view class="btn-item" @click="goToControl">
				<view class="btn-icon-wrap">
					<text class="btn-icon">🎮</text>
				</view>
				<text class="btn-text">小车控制</text>
			</view>
			<view class="btn-item" @click="goToMode">
				<view class="btn-icon-wrap" :class="{ active: isAutoMode }">
					<text class="btn-icon">{{ isAutoMode ? '🤖' : '🕹️' }}</text>
				</view>
				<text class="btn-text">模式转换</text>
			</view>
		</view>
		
		<view class="info-panel">
			<text class="info-title">使用说明</text>
			<view class="info-item">
				<text class="info-num">1</text>
				<text class="info-text">进入「蓝牙连接」搜索并连接小车蓝牙</text>
			</view>
			<view class="info-item">
				<text class="info-num">2</text>
				<text class="info-text">连接成功后可在「小车控制」中手动操控</text>
			</view>
			<view class="info-item">
				<text class="info-num">3</text>
				<text class="info-text">切换至「自动模式」让小车自主完成任务</text>
			</view>
			<view class="info-item">
				<text class="info-num">4</text>
				<text class="info-text">在「数量监控」中查看拾取球数统计</text>
			</view>
		</view>
	</view>
</template>

<script>
	import bluetoothManager from '@/utils/bluetooth.js'
	
	export default {
		data() {
			return {
				isConnected: false,
				isAutoMode: false
			}
		},
		onLoad() {
			const state = bluetoothManager.getState()
			this.isAutoMode = state.isAutoMode
			this.isConnected = bluetoothManager.getIsConnected()
			
			bluetoothManager.addListener(this.onConnectionChange)
			bluetoothManager.addStateListener(this.onStateChange)
		},
		onShow() {
			this.isConnected = bluetoothManager.getIsConnected()
			// 每次显示时同步最新状态
			this.isAutoMode = bluetoothManager.getState().isAutoMode
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
			
			goToBluetooth() {
				uni.navigateTo({ url: '/pages/bluetooth/bluetooth' })
			},
			goToControl() {
				if (!this.isConnected) {
					uni.showToast({ title: '请先连接蓝牙设备', icon: 'none' })
					return
				}
				uni.navigateTo({ url: '/pages/control/control' })
			},
			goToCountMonitor() {
				if (!this.isConnected) {
					uni.showToast({ title: '请先连接蓝牙设备', icon: 'none' })
					return
				}
				uni.navigateTo({ url: '/pages/threshold/threshold' })
			},
			goToMode() {
				if (!this.isConnected) {
					uni.showToast({ title: '请先连接蓝牙设备', icon: 'none' })
					return
				}
				uni.navigateTo({ url: '/pages/mode/mode' })
			}
		}
	}
</script>

<style lang="scss">
	.container {
		min-height: 100vh; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		padding: 30rpx; padding-top: calc(env(safe-area-inset-top) + 30rpx);
		box-sizing: border-box;
	}
	.header {
		display: flex; flex-direction: column; align-items: center;
		padding: 60rpx 0 40rpx;
	}
	.title { font-size: 48rpx; color: #fff; font-weight: bold; margin-bottom: 15rpx; }
	.subtitle { font-size: 28rpx; color: rgba(255,255,255,0.8); }
	
	.status-card {
		background: rgba(255,255,255,0.15); border-radius: 20rpx;
		padding: 30rpx; margin-bottom: 40rpx;
	}
	.status-row {
		display: flex; align-items: center; margin-bottom: 15rpx;
		&:last-child { margin-bottom: 0; }
	}
	.status-dot {
		width: 20rpx; height: 20rpx; border-radius: 50%;
		background: #ff5252; margin-right: 15rpx;
		&.active { background: #4caf50; }
	}
	.status-text { font-size: 28rpx; color: #fff; }
	.mode-badge {
		padding: 8rpx 24rpx; background: rgba(255,255,255,0.2);
		border-radius: 20rpx; font-size: 24rpx; color: #fff;
	}
	
	.button-grid {
		display: grid; grid-template-columns: 1fr 1fr; gap: 30rpx;
		margin-bottom: 40rpx;
	}
	.btn-item {
		background: rgba(255,255,255,0.15); border-radius: 24rpx;
		padding: 40rpx 20rpx; display: flex; flex-direction: column;
		align-items: center; transition: all 0.2s ease;
		&:active { transform: scale(0.95); background: rgba(255,255,255,0.25); }
	}
	.btn-icon-wrap {
		width: 100rpx; height: 100rpx; border-radius: 50%;
		background: rgba(255,255,255,0.2); display: flex;
		align-items: center; justify-content: center; margin-bottom: 15rpx;
		&.active { background: rgba(76, 175, 80, 0.5); }
	}
	.btn-icon { font-size: 48rpx; }
	.btn-text { font-size: 28rpx; color: #fff; font-weight: 600; }
	
	.info-panel {
		background: rgba(255,255,255,0.1); border-radius: 20rpx;
		padding: 30rpx;
	}
	.info-title { font-size: 32rpx; color: #fff; font-weight: bold; margin-bottom: 25rpx; }
	.info-item { display: flex; align-items: flex-start; margin-bottom: 20rpx; }
	.info-num {
		width: 40rpx; height: 40rpx; border-radius: 50%; background: rgba(255,255,255,0.3);
		display: flex; align-items: center; justify-content: center;
		font-size: 24rpx; color: #fff; margin-right: 15rpx; flex-shrink: 0;
	}
	.info-text { font-size: 26rpx; color: rgba(255,255,255,0.9); line-height: 1.5; }
</style>
