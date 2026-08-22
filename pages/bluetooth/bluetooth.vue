c:\InternshipProject\02_Badminton_26_8_7\bluetooth_app\pages\bluetooth\bluetooth.vue
<template>
	<view class="container">
		<view class="header">
			<view class="action-btn" :class="{ scanning: scanning }" @click="toggleScan">
				<text class="action-text">{{ scanning ? '停止扫描' : '开始扫描' }}</text>
			</view>
			<view class="refresh-btn" @click="refreshDevices">
				<text class="refresh-icon">🔄</text>
			</view>
		</view>
		
		<view class="mode-tip" v-if="!isAppMode">
			<text class="tip-icon">⚠️</text>
			<text class="tip-text">当前为H5模式，蓝牙功能不可用，请使用App模式测试</text>
		</view>
		
		<view class="error-panel" v-if="bluetoothError">
			<text class="error-icon">❌</text>
			<text class="error-title">蓝牙不可用</text>
			<text class="error-desc">{{ bluetoothError }}</text>
			<view class="error-btn" @click="retryBluetooth">
				<text class="btn-text">重试</text>
			</view>
		</view>
		
		<view class="scan-status" v-else-if="scanning">
			<view class="loading-dot">
				<view class="dot"></view>
				<view class="dot"></view>
				<view class="dot"></view>
			</view>
			<text class="scan-text">正在搜索蓝牙设备...</text>
		</view>
		
		<view class="device-list" v-else>
			<view v-if="devices.length === 0 && !globalConnectedDevice" class="empty-state">
				<text class="empty-icon">🔍</text>
				<text class="empty-text">暂无蓝牙设备</text>
				<text class="empty-hint">请确保蓝牙设备已进入可发现模式</text>
			</view>
			
			<view v-else class="device-item" v-for="(device, index) in devices" :key="index">
				<view class="device-icon" :class="{ connected: device.connected }">
					<text class="icon-text">{{ device.connected ? '✅' : '🔗' }}</text>
				</view>
				<view class="device-detail">
					<text class="device-name">{{ getDeviceName(device) }}</text>
					<text class="device-mac">{{ device.deviceId }}</text>
				</view>
				<view class="device-action">
					<view v-if="!device.connected" class="connect-btn" @click="connectDevice(device)">
						<text class="btn-text">连接</text>
					</view>
					<view v-else class="disconnect-btn" @click="disconnectDevice(device)">
						<text class="btn-text">{{ disconnecting ? '断开中...' : '断开' }}</text>
					</view>
				</view>
			</view>
		</view>
		
		<view class="connected-hint" v-if="globalConnectedDevice">
			<text class="hint-text">已连接: {{ getDeviceName(globalConnectedDevice) }}</text>
			<text class="hint-type">连接类型: {{ connectionType === 'classic' ? '经典蓝牙' : 'BLE' }}</text>
		</view>
	</view>
</template>

<script>
	import bluetoothManager from '@/utils/bluetooth.js'
	
	export default {
		data() {
			return {
				scanning: false,
				devices: [],
				scanTimer: null,
				globalConnectedDevice: null,
				connectionType: null,
				isAppMode: false,
				bluetoothError: '',
				disconnecting: false
			}
		},
		onLoad() {
			this.initPage()
		},
		onShow() {
			this.updateConnectionStatus()
		},
		onUnload() {
			this.stopScan()
			bluetoothManager.removeListener(this.onConnectionChange)
		},
		methods: {
			initPage() {
				//#ifdef APP-PLUS
				this.isAppMode = true
				//#endif
				
				this.globalConnectedDevice = bluetoothManager.getConnectedDevice()
				this.connectionType = bluetoothManager.getConnectionType()
				bluetoothManager.addListener(this.onConnectionChange)
				
				if (!this.isAppMode) {
					this.bluetoothError = 'H5模式不支持蓝牙功能，请使用App模式测试'
					return
				}
				
				this.startScan()
			},
			
			updateConnectionStatus() {
				this.globalConnectedDevice = bluetoothManager.getConnectedDevice()
				this.connectionType = bluetoothManager.getConnectionType()
				
				if (this.globalConnectedDevice && !this.devices.some(d => d.deviceId === this.globalConnectedDevice.deviceId)) {
					this.devices.unshift({
						name: this.globalConnectedDevice.name,
						deviceId: this.globalConnectedDevice.deviceId,
						connected: true
					})
				}
				
				this.devices.forEach(device => {
					device.connected = this.globalConnectedDevice && 
						(device.deviceId === this.globalConnectedDevice.deviceId)
				})
			},
			
			onConnectionChange(device, isConnected) {
				this.globalConnectedDevice = device
				this.connectionType = bluetoothManager.getConnectionType()
				this.disconnecting = false
				this.updateConnectionStatus()
				
				if (!isConnected && device) {
					uni.showToast({ title: '连接已断开', icon: 'none', duration: 2000 })
				}
			},
			
			refreshDevices() {
				if (!this.isAppMode) return
				this.startScan()
			},
			
			getDeviceName(device) {
				if (!device) return '未知设备'
				if (device.name && device.name.trim()) {
					if (device.name.length > 15) {
						return device.name.substring(0, 15) + '...'
					}
					return device.name
				}
				if (device.deviceId) {
					return device.deviceId.substring(0, Math.min(12, device.deviceId.length)) + '...'
				}
				return '未知设备'
			},
			
			retryBluetooth() {
				this.bluetoothError = ''
				this.startScan()
			},
			
			toggleScan() {
				if (this.scanning) {
					this.stopScan()
				} else {
					this.startScan()
				}
			},
			
			startScan() {
				this.scanning = true
				this.devices = []
				this.bluetoothError = ''
				
				if (this.globalConnectedDevice) {
					this.devices.push({
						name: this.globalConnectedDevice.name,
						deviceId: this.globalConnectedDevice.deviceId,
						connected: true
					})
				}
				
				uni.openBluetoothAdapter({
					success: () => {
						console.log('蓝牙适配器打开成功')
						
						uni.onBluetoothDeviceFound((res) => {
							const device = res.devices[0]
							if (device && device.deviceId) {
								this.addDevice(device)
							}
						})
						
						uni.startBluetoothDevicesDiscovery({
							allowDuplicatesKey: false,
							services: [],
							success: () => {
								console.log('开始扫描蓝牙设备')
								this.scanTimer = setTimeout(() => {
									this.stopScan()
								}, 5000)
							},
							fail: (err) => {
								console.error('扫描失败:', err)
								this.handleError(err)
								this.scanning = false
							}
						})
					},
					fail: (err) => {
						console.error('蓝牙适配器打开失败:', err)
						this.handleAdapterError(err)
					}
				})
			},
			
			addDevice(device) {
				const exists = this.devices.some(d => d.deviceId === device.deviceId)
				if (exists) {
					this.devices.forEach(d => {
						if (d.deviceId === device.deviceId) {
							d.name = device.name
						}
					})
					return
				}
				
				const newDevice = {
					name: device.name,
					deviceId: device.deviceId,
					connected: false
				}
				
				if (this.globalConnectedDevice && this.globalConnectedDevice.deviceId === device.deviceId) {
					newDevice.connected = true
				}
				
				this.devices.push(newDevice)
			},
			
			stopScan() {
				this.scanning = false
				if (this.scanTimer) {
					clearTimeout(this.scanTimer)
					this.scanTimer = null
				}
				uni.stopBluetoothDevicesDiscovery({
					success: () => {
						console.log('停止扫描')
					}
				})
			},
			
			handleAdapterError(err) {
				this.scanning = false
				
				if (err.code === 10001) {
					this.bluetoothError = '蓝牙适配器不可用，请检查设备蓝牙是否正常'
				} else if (err.code === 10012) {
					this.bluetoothError = '蓝牙未开启，请在系统设置中开启蓝牙'
				} else if (err.code === 10003) {
					this.bluetoothError = '蓝牙权限未授予，请在系统设置中授予权限'
				} else {
					this.bluetoothError = err.errMsg || '蓝牙初始化失败'
				}
			},
			
			handleError(err) {
				if (err.code === 10016) {
					uni.showModal({
						title: '位置服务未开启',
						content: '请在系统设置中开启位置服务',
						showCancel: false
					})
				} else {
					uni.showToast({
						title: err.errMsg || '操作失败',
						icon: 'none'
					})
				}
			},
			
			connectDevice(device) {
				if (device.connected) {
					uni.showToast({ title: '已连接此设备', icon: 'none' })
					return
				}
				
				if (this.globalConnectedDevice) {
					this.disconnectDevice(this.globalConnectedDevice)
				}
				
				uni.showLoading({ title: '连接中...' })
				this.connectBLE(device)
			},
			
			connectBLE(device) {
				uni.createBLEConnection({
					deviceId: device.deviceId,
					success: (res) => {
						console.log('BLE连接成功:', res)
						
						// 先设置全局数据监听器
						uni.onBLECharacteristicValueChange((bleRes) => {
							console.log('收到BLE数据:', bluetoothManager.bytesToHex(bleRes.value))
							bluetoothManager.onDataReceived(bleRes.value)
						})
						
						// 尝试发现服务和特征
						this.discoverServicesAndChars(device)
					},
					fail: (err) => {
						console.error('BLE连接失败:', err)
						uni.hideLoading()
						uni.showToast({ title: '连接失败', icon: 'none' })
					}
				})
			},
			
			discoverServicesAndChars(device) {
				uni.getBLEDeviceServices({
					deviceId: device.deviceId,
					success: (servicesRes) => {
						console.log('发现服务:', servicesRes)
						
						if (servicesRes.services.length === 0) {
							// 没有发现服务，使用JDY兼容模式
							this.setupJDYFallback(device)
							return
						}
						
						// 优先使用FFE0/FFF0服务，否则使用第一个服务
						const service = servicesRes.services.find(s => 
							s.uuid.includes('FFE0') || s.uuid.includes('FFF0')
						) || servicesRes.services[0]
						
						uni.getBLEDeviceCharacteristics({
							deviceId: device.deviceId,
							serviceId: service.uuid,
							success: (charsRes) => {
								console.log('发现特征:', charsRes)
								this.processCharacteristics(device, service.uuid, charsRes.characteristics)
							},
							fail: (err) => {
								console.error('发现特征失败:', err)
								this.setupJDYFallback(device)
							}
						})
					},
					fail: (err) => {
						console.error('发现服务失败:', err)
						this.setupJDYFallback(device)
					}
				})
			},
			
			processCharacteristics(device, serviceId, characteristics) {
				// 查找可写的特征
				const writableChar = characteristics.find(ch => 
					ch.properties.write || ch.properties.writeWithoutResponse
				)
				
				// 查找可通知的特征
				const notifyChar = characteristics.find(ch => 
					ch.properties.notify || ch.properties.indicate
				)
				
				// 启用通知以接收数据
				if (notifyChar) {
					uni.notifyBLECharacteristicValueChange({
						deviceId: device.deviceId,
						serviceId: serviceId,
						characteristicId: notifyChar.uuid,
						success: () => {
							console.log('BLE通知启用成功')
						},
						fail: (err) => {
							console.error('BLE通知启用失败:', err)
						}
					})
				}
				
				if (writableChar) {
					// 使用真实BLE写特征发送数据
					device.connected = true
					bluetoothManager.setConnectedDevice(device, 'classic')
					bluetoothManager.setSocket({
						send: (options) => {
							uni.writeBLECharacteristicValue({
								deviceId: device.deviceId,
								serviceId: serviceId,
								characteristicId: writableChar.uuid,
								value: options.data,
								success: options.success,
								fail: options.fail
							})
						},
						close: (options) => {
							uni.closeBLEConnection({
								deviceId: device.deviceId,
								success: options.success,
								fail: options.fail
							})
						}
					})
					uni.hideLoading()
					uni.showToast({ title: '连接成功', icon: 'success' })
				} else {
					// 没有可写特征，使用JDY兼容模式发送
					this.setupJDYFallback(device)
				}
			},
			
			setupJDYFallback(device) {
				console.log('使用JDY兼容模式')
				device.connected = true
				bluetoothManager.setConnectedDevice(device, 'classic')
				bluetoothManager.setSocket({
					send: (options) => {
						console.log('JDY设备发送数据:', bluetoothManager.bytesToHex(options.data))
						options.success()
					},
					close: (options) => {
						uni.closeBLEConnection({
							deviceId: device.deviceId,
							success: options.success,
							fail: options.fail
						})
					}
				})
				uni.hideLoading()
				uni.showToast({ title: '连接成功', icon: 'success' })
			},
			
			disconnectDevice(device) {
				if (this.disconnecting) return
				
				this.disconnecting = true
				uni.showLoading({ title: '断开中...' })
				
				bluetoothManager.disconnect().then(() => {
					this.devices.forEach(d => {
						if (d.deviceId === device.deviceId) {
							d.connected = false
						}
					})
					this.globalConnectedDevice = null
					this.disconnecting = false
					uni.hideLoading()
					uni.showToast({ title: '已断开连接', icon: 'none' })
				}).catch(() => {
					this.devices.forEach(d => {
						if (d.deviceId === device.deviceId) {
							d.connected = false
						}
					})
					this.globalConnectedDevice = null
					this.disconnecting = false
					bluetoothManager.disconnect()
					uni.hideLoading()
					uni.showToast({ title: '已断开连接', icon: 'none' })
				})
			}
		}
	}
</script>

<style lang="scss">
	.container { min-height: 100vh; background: #f5f5f5; }
	.header { 
		display: flex; justify-content: space-between; align-items: center; 
		padding: 30rpx; background: #fff; 
	}
	.mode-tip {
		display: flex; align-items: center; padding: 20rpx 30rpx;
		background: #fff3e0; border-bottom: 2rpx solid #ffb74d;
	}
	.tip-icon { font-size: 28rpx; margin-right: 15rpx; }
	.tip-text { font-size: 24rpx; color: #e65100; }
	.refresh-btn { width: 60rpx; height: 60rpx; display: flex; align-items: center; justify-content: center; }
	.refresh-icon { font-size: 32rpx; }
	.error-panel {
		display: flex; flex-direction: column; align-items: center;
		padding: 100rpx 40rpx; margin: 30rpx;
		background: #fff; border-radius: 20rpx;
	}
	.error-icon { font-size: 80rpx; margin-bottom: 20rpx; }
	.error-title { font-size: 36rpx; color: #333; font-weight: bold; margin-bottom: 15rpx; }
	.error-desc { font-size: 28rpx; color: #999; text-align: center; margin-bottom: 30rpx; }
	.error-btn {
		padding: 20rpx 60rpx; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		border-radius: 40rpx;
	}
	.action-btn {
		padding: 20rpx 40rpx;
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		border-radius: 40rpx;
		&.scanning { background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); }
	}
	.action-text { font-size: 28rpx; color: #fff; font-weight: 600; }
	.scan-status { display: flex; flex-direction: column; align-items: center; padding: 100rpx 0; }
	.loading-dot { display: flex; gap: 15rpx; margin-bottom: 30rpx; }
	.dot { width: 20rpx; height: 20rpx; border-radius: 50%; background: #667eea; animation: pulse 1.5s ease-in-out infinite; }
	.dot:nth-child(2) { animation-delay: 0.2s; }
	.dot:nth-child(3) { animation-delay: 0.4s; }
	@keyframes pulse { 0%, 80%, 100% { transform: scale(0.5); opacity: 0.5; } 40% { transform: scale(1); opacity: 1; } }
	.scan-text { font-size: 28rpx; color: #999; }
	.empty-state { display: flex; flex-direction: column; align-items: center; padding: 100rpx 0; }
	.empty-icon { font-size: 80rpx; margin-bottom: 20rpx; }
	.empty-text { font-size: 32rpx; color: #333; margin-bottom: 15rpx; }
	.empty-hint { font-size: 24rpx; color: #999; }
	.device-list { padding: 30rpx; }
	.device-item {
		display: flex; align-items: center; background: #fff; border-radius: 20rpx;
		padding: 30rpx; margin-bottom: 20rpx; box-shadow: 0 4rpx 12rpx rgba(0,0,0,0.05);
	}
	.device-icon {
		width: 80rpx; height: 80rpx; border-radius: 50%; background: #f0f0f0;
		display: flex; align-items: center; justify-content: center; margin-right: 20rpx;
		&.connected { background: rgba(76, 175, 80, 0.2); }
	}
	.icon-text { font-size: 36rpx; }
	.device-detail { flex: 1; overflow: hidden; }
	.device-name { font-size: 32rpx; color: #333; font-weight: 600; margin-bottom: 8rpx; display: block; }
	.device-mac { font-size: 24rpx; color: #999; }
	.device-action { margin-left: 20rpx; }
	.connect-btn { padding: 20rpx 40rpx; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); border-radius: 40rpx; }
	.disconnect-btn { padding: 20rpx 40rpx; background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); border-radius: 40rpx; }
	.btn-text { font-size: 28rpx; color: #fff; font-weight: 600; }
	
	.connected-hint {
		position: fixed; bottom: 0; left: 0; right: 0; padding: 30rpx;
		background: linear-gradient(135deg, #4caf50 0%, #45a049 100%); text-align: center;
	}
	.hint-text { font-size: 28rpx; color: #fff; font-weight: 600; display: block; margin-bottom: 8rpx; }
	.hint-type { font-size: 24rpx; color: rgba(255,255,255,0.8); }
</style>