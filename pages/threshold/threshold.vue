c:\InternshipProject\02_Badminton_26_8_7\bluetooth_app\pages\threshold\threshold.vue
<template>
	<view class="container">
		<view class="count-display-section">
			<view class="count-header">
				<text class="count-title">拾取小球数量</text>
				<text class="count-status" :class="{ connected: isConnected }">
					{{ isConnected ? '实时监测中' : '未连接' }}
				</text>
			</view>
			<view class="count-value-wrap">
				<text class="count-value">{{ ballCount }}</text>
				<text class="count-unit">个</text>
			</view>
			<view class="count-bar">
				<view class="count-bar-fill" :style="{ width: countBarWidth + '%' }"></view>
			</view>
			<view class="count-info">
				<text class="info-text">当前桶: {{ currentBucket }}</text>
				<text class="info-text">当前桶收集数: {{ bucketCount }}/{{ maxPerBucket }}</text>
			</view>
		</view>
		
		<view class="bucket-section">
			<view class="section-title">收集桶状态</view>
			<view class="bucket-grid">
				<view class="bucket-item" v-for="(bucket, index) in buckets" :key="index" :class="{ active: index === currentBucketIndex }">
					<view class="bucket-icon">🪣</view>
					<text class="bucket-label">桶{{ index + 1 }}</text>
					<text class="bucket-count">{{ bucket }}个</text>
					<view class="bucket-bar">
						<view class="bucket-bar-fill" :style="{ width: (bucket / maxPerBucket * 100) + '%' }"></view>
					</view>
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
				ballCount: 0,
				currentBucketIndex: 0,
				buckets: [0, 0, 0],
				maxPerBucket: 2,
				isConnected: false
			}
		},
		computed: {
			countBarWidth() {
				const total = this.maxPerBucket * 3
				return Math.min((this.ballCount / total) * 100, 100)
			},
			currentBucket() {
				return '桶' + (this.currentBucketIndex + 1)
			},
			bucketCount() {
				return this.buckets[this.currentBucketIndex] || 0
			}
		},
		onLoad() {
			const state = bluetoothManager.getState()
			this.ballCount = state.ballCount
			this.buckets = [...state.buckets]
			this.currentBucketIndex = state.currentBucketIndex
			
			this.isConnected = bluetoothManager.getIsConnected()
			bluetoothManager.addListener(this.onConnectionChange)
			//bluetoothManager.addDataListener(this.onDataReceived)
			bluetoothManager.addStateListener(this.onStateChange)
		},
		onShow() {
			this.isConnected = bluetoothManager.getIsConnected()
		},
		onUnload() {
			bluetoothManager.removeListener(this.onConnectionChange)
			//bluetoothManager.removeDataListener(this.onDataReceived)
			bluetoothManager.removeStateListener(this.onStateChange)
		},
		methods: {
			onConnectionChange(device, isConnected) {
				this.isConnected = isConnected
			},

			onStateChange(state) {
				this.ballCount = state.ballCount
				this.buckets = [...state.buckets]
				this.currentBucketIndex = state.currentBucketIndex
			},
			
			//onDataReceived(cmd, data) {
				//console.log('收到数据:', cmd, data)
				
				//if (cmd === 0x08 && data.length >= 3) {
					//const newBuckets = [data[0], data[1], data[2]]
					
					// 自动判断当前活跃桶：第一个未满的桶
					//let activeBucket = 0
					//for (let i = 0; i < 3; i++) {
						//if (newBuckets[i] < this.maxPerBucket) {
							//activeBucket = i
							//break
						//}
					//}
					// 如果全满了，保持最后一个桶
					//if (newBuckets[0] >= this.maxPerBucket && 
					    //newBuckets[1] >= this.maxPerBucket && 
					    //newBuckets[2] >= this.maxPerBucket) {
						//activeBucket = 2
					//}
					
					// 写入全局状态
					//bluetoothManager.setState({
						//ballCount: newBuckets[0] + newBuckets[1] + newBuckets[2],
						//buckets: newBuckets,
						//currentBucketIndex: activeBucket
					//})
				//}
			//}
		}
	}
</script>

<style lang="scss">
	.container {
		min-height: 100vh; background: #f5f5f5; padding: 30rpx;
	}
	.count-display-section {
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		border-radius: 24rpx; padding: 40rpx; margin-bottom: 30rpx;
	}
	.count-header {
		display: flex; justify-content: space-between; align-items: center;
		margin-bottom: 30rpx;
	}
	.count-title { font-size: 32rpx; color: #fff; font-weight: bold; }
	.count-status {
		font-size: 24rpx; color: rgba(255,255,255,0.7);
		&.connected { color: #4caf50; }
	}
	.count-value-wrap {
		display: flex; align-items: baseline; justify-content: center;
		margin-bottom: 30rpx;
	}
	.count-value { font-size: 120rpx; color: #fff; font-weight: bold; }
	.count-unit { font-size: 36rpx; color: rgba(255,255,255,0.8); margin-left: 10rpx; }
	.count-bar {
		height: 12rpx; background: rgba(255,255,255,0.3); border-radius: 6rpx;
		overflow: hidden; margin-bottom: 20rpx;
	}
	.count-bar-fill {
		height: 100%; background: #4caf50; border-radius: 6rpx;
		transition: width 0.5s ease;
	}
	.count-info {
		display: flex; justify-content: space-between;
	}
	.info-text { font-size: 24rpx; color: rgba(255,255,255,0.8); }
	
	.bucket-section {
		background: #fff; border-radius: 24rpx; padding: 30rpx;
	}
	.section-title { font-size: 32rpx; color: #333; font-weight: bold; margin-bottom: 30rpx; }
	.bucket-grid { display: flex; gap: 20rpx; }
	.bucket-item {
		flex: 1; display: flex; flex-direction: column; align-items: center;
		padding: 30rpx 20rpx; background: #f8f9fa; border-radius: 16rpx;
		border: 2rpx solid transparent;
		&.active { border-color: #667eea; background: #f0f0ff; }
	}
	.bucket-icon { font-size: 48rpx; margin-bottom: 10rpx; }
	.bucket-label { font-size: 26rpx; color: #666; margin-bottom: 8rpx; }
	.bucket-count { font-size: 32rpx; color: #333; font-weight: bold; margin-bottom: 12rpx; }
	.bucket-bar {
		width: 100%; height: 8rpx; background: #e0e0e0; border-radius: 4rpx;
		overflow: hidden;
	}
	.bucket-bar-fill {
		height: 100%; background: #667eea; border-radius: 4rpx;
		transition: width 0.5s ease;
	}
</style>