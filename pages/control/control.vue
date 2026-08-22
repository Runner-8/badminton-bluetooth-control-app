c:\InternshipProject\02_Badminton_26_8_7\bluetooth_app\pages\control\control.vue
<template>
	<view class="container">
		<view class="status-bar">
			<view class="status-left">
				<text class="status-label">连接状态</text>
				<text class="status-value">{{ currentStatus }}</text>
			</view>
			<view class="connection-status" :class="{ connected: isConnected }">
				<text class="connection-icon">{{ isConnected ? '✅' : '❌' }}</text>
				<text class="connection-text">{{ isConnected ? '已连接' : '未连接' }}</text>
			</view>
		</view>
		
		<view class="tips-panel">
			<text class="tips-text">💡 长按按钮控制小车移动</text>
			<text class="tips-hint" v-if="!isConnected">请先连接蓝牙设备</text>
		</view>
		
		<view class="control-panel">
			<view class="left-panel">
				<view 
					class="control-btn btn-up" 
					@touchstart="onTouchStart('f', $event)" 
					@touchend="onTouchEnd"
					@touchcancel="onTouchEnd"
					@touchmove="onTouchMove"
				>
					<text class="btn-icon">▲</text>
					<text class="btn-text">前进</text>
					<view class="press-indicator" :class="{ active: pressing }"></view>
				</view>
				<view 
					class="control-btn btn-down" 
					@touchstart="onTouchStart('b', $event)" 
					@touchend="onTouchEnd"
					@touchcancel="onTouchEnd"
					@touchmove="onTouchMove"
				>
					<text class="btn-icon">▼</text>
					<text class="btn-text">后退</text>
					<view class="press-indicator" :class="{ active: pressing }"></view>
				</view>
			</view>
			
			<view class="right-panel">
				<view 
					class="control-btn btn-left" 
					@touchstart="onTouchStart('l', $event)" 
					@touchend="onTouchEnd"
					@touchcancel="onTouchEnd"
					@touchmove="onTouchMove"
				>
					<text class="btn-icon">◀</text>
					<text class="btn-text">左转</text>
					<view class="press-indicator" :class="{ active: pressing }"></view>
				</view>
				<view 
					class="control-btn btn-right" 
					@touchstart="onTouchStart('r', $event)" 
					@touchend="onTouchEnd"
					@touchcancel="onTouchEnd"
					@touchmove="onTouchMove"
				>
					<text class="btn-icon">▶</text>
					<text class="btn-text">右转</text>
					<view class="press-indicator" :class="{ active: pressing }"></view>
				</view>
			</view>
		</view>
		
		<view class="shift-panel">
			<view 
				class="control-btn btn-shift-left" 
				@touchstart="onTouchStart('sl', $event)" 
				@touchend="onTouchEnd"
				@touchcancel="onTouchEnd"
				@touchmove="onTouchMove"
			>
				<text class="btn-icon">⇐</text>
				<text class="btn-text">左移</text>
				<view class="press-indicator" :class="{ active: pressing }"></view>
			</view>
			<view 
				class="control-btn btn-shift-right" 
				@touchstart="onTouchStart('sr', $event)" 
				@touchend="onTouchEnd"
				@touchcancel="onTouchEnd"
				@touchmove="onTouchMove"
			>
				<text class="btn-icon">⇒</text>
				<text class="btn-text">右移</text>
				<view class="press-indicator" :class="{ active: pressing }"></view>
			</view>
		</view>
		
		<view class="stop-area" @touchstart="sendStopDirectly">
			<text class="stop-icon">⏹️</text>
			<text class="stop-text">停止</text>
		</view>
	</view>
</template>

<script>
	import bluetoothManager from '@/utils/bluetooth.js'
	
	export default {
		data() {
			return {
				currentStatus: '待机',
				isConnected: false,
				pressing: false,
				pressTimer: null,
				pressStartTime: 0,
				currentCmd: '',
				isLongPress: false,
				THRESHOLD: 300 // 长按触发阈值(ms)
			}
		},
		onLoad() {
			this.isConnected = bluetoothManager.getIsConnected()
			bluetoothManager.addListener(this.onConnectionChange)
		},
		onUnload() {
			bluetoothManager.removeListener(this.onConnectionChange)
			this.clearTimer()
		},
		methods: {
			onConnectionChange(device, isConnected) {
				this.isConnected = isConnected
			},
			
			clearTimer() {
				if (this.pressTimer) {
					clearTimeout(this.pressTimer)
					this.pressTimer = null
				}
			},
			
			onTouchStart(cmd, event) {
				if (!this.isConnected) {
					uni.showToast({ title: '请先连接蓝牙设备', icon: 'none' })
					return
				}
				
				// 阻止默认滚动行为
				event.preventDefault()
				
				this.pressing = true
				this.currentCmd = cmd
				this.pressStartTime = Date.now()
				this.isLongPress = false
				
				// 超过阈值后触发长按发送
				this.pressTimer = setTimeout(() => {
					this.sendCommand(cmd)
					this.isLongPress = true
				}, this.THRESHOLD)
			},
			
			onTouchEnd() {
				this.pressing = false
				this.clearTimer()
				
				// 长按松手后自动发送停止命令
				if (this.isLongPress && this.currentCmd) {
					this.sendStop()
				}
				
				this.currentCmd = ''
				this.isLongPress = false
			},
			
			onTouchMove() {
				// 触摸移动时不做处理
			},
			
			sendCommand(cmd) {
				const statusMap = { 'f': '前进', 'b': '后退', 'l': '左转', 'r': '右转', 'sl': '左移', 'sr': '右移', 's': '待机' }
				this.currentStatus = statusMap[cmd] || cmd
				
				bluetoothManager.sendCommand(this.getCmdType(cmd)).then(() => {
					console.log('发送命令:', cmd)
				}).catch((err) => {
					console.error('发送失败:', err)
				})
			},
			
			sendStop() {
				this.currentStatus = '待机'
				
				bluetoothManager.sendCommand('stop').then(() => {
					console.log('停止命令已发送')
				}).catch((err) => {
					console.error('停止命令发送失败:', err)
				})
			},
			
			sendStopDirectly() {
				if (!this.isConnected) {
					uni.showToast({ title: '请先连接蓝牙设备', icon: 'none' })
					return
				}
				
				this.sendStop()
			},
			
			getCmdType(cmd) {
				const cmdMap = {
					'f': 'forward',
					'b': 'backward',
					'l': 'left',
					'r': 'right',
					'sl': 'leftShift',
					'sr': 'rightShift'
				}
				return cmdMap[cmd] || cmd
			}
		}
	}
</script>

<style lang="scss">
	.container {
		min-height: 100vh; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
		display: flex; flex-direction: column; padding: 30rpx;
		padding-top: calc(env(safe-area-inset-top) + 30rpx); box-sizing: border-box;
	}
	.status-bar {
		display: flex; justify-content: space-between; align-items: center;
		background: rgba(255,255,255,0.1); border-radius: 20rpx; padding: 30rpx 40rpx;
		margin-bottom: 20rpx;
	}
	.status-left { display: flex; flex-direction: column; }
	.status-label { font-size: 28rpx; color: rgba(255,255,255,0.7); margin-bottom: 8rpx; }
	.status-value { font-size: 48rpx; color: #fff; font-weight: bold; }
	.connection-status {
		display: flex; flex-direction: column; align-items: center;
		padding: 20rpx 30rpx; background: rgba(255, 255, 255, 0.1); border-radius: 16rpx;
		&.connected { background: rgba(76, 175, 80, 0.3); }
	}
	.connection-icon { font-size: 32rpx; margin-bottom: 8rpx; }
	.connection-text { font-size: 24rpx; color: #fff; }
	
	.tips-panel {
		background: rgba(255, 193, 7, 0.2); border-radius: 16rpx; padding: 20rpx 30rpx;
		margin-bottom: 30rpx; text-align: center;
	}
	.tips-text { font-size: 26rpx; color: #ffc107; display: block; }
	.tips-hint { font-size: 24rpx; color: #ff9800; margin-top: 8rpx; }
	
	.control-panel { flex: 1; display: flex; justify-content: space-between; padding: 0 20rpx; }
	.left-panel, .right-panel { display: flex; flex-direction: column; gap: 40rpx; }
	.control-btn {
		width: 180rpx; height: 180rpx; border-radius: 24rpx;
		display: flex; flex-direction: column; align-items: center; justify-content: center;
		transition: all 0.2s ease; box-shadow: 0 8rpx 24rpx rgba(0,0,0,0.3);
		position: relative; overflow: hidden;
		&:active { transform: scale(0.95); }
	}
	.btn-up { background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%); }
	.btn-down { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
	.btn-left { background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%); }
	.btn-right { background: linear-gradient(135deg, #fa709a 0%, #fee140 100%); }
	.btn-shift-left { background: linear-gradient(135deg, #a18cd1 0%, #fbc2eb 100%); }
	.btn-shift-right { background: linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%); }
	.btn-icon { font-size: 56rpx; color: #fff; margin-bottom: 8rpx; }
	.btn-text { font-size: 28rpx; color: #fff; font-weight: 600; }
	.press-indicator {
		position: absolute; bottom: 0; left: 0; right: 0; height: 0;
		background: rgba(255,255,255,0.3);
		transition: height 0.3s ease;
		&.active { height: 100%; }
	}
	
	.shift-panel {
		display: flex; justify-content: center; gap: 40rpx;
		margin-top: 30rpx;
	}
	.stop-area {
		display: flex; flex-direction: column; align-items: center;
		padding: 30rpx; background: rgba(255,255,255,0.1);
		border-radius: 20rpx; margin-top: 30rpx;
	}
	.stop-icon { font-size: 64rpx; margin-bottom: 10rpx; }
	.stop-text { font-size: 28rpx; color: rgba(255,255,255,0.8); }
</style>