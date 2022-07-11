<script>
	var jpushModule = uni.requireNativePlugin("JG-JPush");
	export default {
		onLaunch: function() {
			console.log('App Launch')
			// #ifdef APP-PLUS
			jpushModule.setLoggerEnable(true);
			// 初始化函数
			jpushModule.initJPushService();
			jpushModule.addConnectEventListener(result=>{
				let connectEnable = result.connectEnable
				console.log("jpush连接", connectEnable)
			})
			jpushModule.getRegistrationID(result => {
				console.log("注册ID.....",result)
				this.registerID = result.registerID
				uni.showToast({
					title:result.registerID,
					icon:"success",
				})
			})
			jpushModule.isPushStopped(result=>{
				let code = result.code
				console.log('连接状态回调',result)
			});
			// 设置别名
			jpushModule.setAlias({
				'alias' : 'coder',
				'sequence': 1
			})
			jpushModule.addNotificationListener(result=>{
				console.log('result--', result);
				let notificationEventType = result.notificationEventType
				let messageID = result.messageID
				let title = result.title
				let content = result.content
				let extras = result.extras
				console.log('通知事件回调',result)
				// 推送一个本地通知
				jpushModule.addLocalNotification({
					messageID,
					title,
					content,
					extras
				})
				if (notificationEventType === 'notificationOpened') {
					console.log('点击跳转--');
					uni.navigateTo({
						url: `/pages/index/test/test?id=${extras.testId}`
					})
				}
			})//#endif
		},
		onShow: function() {
			console.log('App Show')
			this.getNotificationEnabled()
			// #ifdef APP-PLUS
			jpushModule.addNotificationListener(result=>{
				let extras = result.extras
				console.log('onshow通知事件回调',result)
				if (result.notificationEventType === 'notificationOpened') {
					console.log('点击跳转--');
					uni.navigateTo({
						url: `/pages/index/test/test?id=${extras.testId}`
					})
				}
			})
			// #endif
		},
		onHide: function() {
			console.log('App Hide')
		},
		methods: {
			/* 1. 首先要判断用户是否打开通知权限 */
			getNotificationEnabled(){ 
				if(uni.getSystemInfoSync().platform == "ios"){
					jpushModule.requestNotificationAuthorization((result)=>{
						let status = result.status
						if (status < 2) {
							this.noticMsgTool()
						}
					})
				}else{
					jpushModule.isNotificationEnabled((result)=>{ //判断android是否打开权限
						if(result.code == 0){//如果为0则表示 未打开通知权限 
							this.noticMsgTool()
						}
					})
				}
			},
			noticMsgTool(){
				if(uni.getSystemInfoSync().platform == "ios"){
					//苹果打开对应的通知栏
					uni.showModal({
						title: '通知权限开启提醒',
						content: '您还没有开启通知权限，无法接受到消息通知，请前往设置！',
						showCancel: false,
						confirmText: '去设置',
						success: function(res) {
							if (res.confirm) {
								var app = plus.ios.invoke('UIApplication', 'sharedApplication');
								var setting = plus.ios.invoke('NSURL', 'URLWithString:', 'app-settings:');
								plus.ios.invoke(app, 'openURL:', setting);
								plus.ios.deleteObject(setting);
								plus.ios.deleteObject(app);
							}
						}
					});
				}else{
					//android打开对应的通知栏
					var main = plus.android.runtimeMainActivity();
					var pkName = main.getPackageName();
					var uid = main.getApplicationInfo().plusGetAttribute("uid");
					uni.showModal({
						title: '通知权限开启提醒',
						content: '您还没有开启通知权限，无法接受到消息通知，请前往设置！',
						showCancel: false,
						confirmText: '去设置',
						success: function(res) {
							if (res.confirm) {
							    var Intent = plus.android.importClass('android.content.Intent');
								var Build = plus.android.importClass("android.os.Build");
								//android 8.0引导  
								if (Build.VERSION.SDK_INT >= 26) {
									var intent = new Intent('android.settings.APP_NOTIFICATION_SETTINGS');
									intent.putExtra('android.provider.extra.APP_PACKAGE', pkName);
								} else if (Build.VERSION.SDK_INT >= 21) { //android 5.0-7.0  
									var intent = new Intent('android.settings.APP_NOTIFICATION_SETTINGS');
								    intent.putExtra("app_package", pkName);
									intent.putExtra("app_uid", uid);
								} else { //(<21)其他--跳转到该应用管理的详情页  
									intent.setAction(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
									var uri = Uri.fromParts("package", mainActivity.getPackageName(), null);
									intent.setData(uri);
								}
							    // 跳转到该应用的系统通知设置页  
							    main.startActivity(intent);
							}
						}
					});
				}
			}

		}
	}
</script>

<style>
	/*每个页面公共css */
</style>
