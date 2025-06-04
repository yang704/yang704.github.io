# 安卓通话录音APP开发爬坑指南

### 技术原理
- 使用h5plus以及native.js调用安卓的原生API进行开发。
- 依据不同的手机品牌以及型号记录其通话录音文件的储存路径，我这里是由后端通过接口返回给前端的：
  
  ```json
    [
        '/smartisan/Recorder',
        '/Sounds/CallRecord',
        '/MIUI/sound_recorder/call_rec',
        '/Recordings',
        '/record',
        '/Record/PhoneRecord',
        '/Record/Call',
        '/neoRecorderTemp',
        '/neoRecorder/CallRecorder',
        '/Music/Recordings/Call Recordings'
    ]
  ```
    > 此部分的路径可自行前去百度，或者在手机自行总结查看。
    
- 监听手机的通话状态，依据通话状态的变更去获取通话记录。
- 根据路径匹配去递归查找录音文件，规则如下：
  1. 文件名称通常包含手机号
  2. 文件格式为常见的音频格式
  3. 文件最后的更新时间和通话记录获取到挂断时间相近。
- 根据匹配到的录音文件的绝对路径，上传录音文件到服务器。

### 技术实现
- 登录后连接webScoket并依据返回的用户ID进行绑定,以便后台进行手机号推送，并自动拨打电话。
  ```javascript
  //连接websocket
  handleConnection() {
		let t = this;
		uni.showLoading({
			title: '正在连接中...'
		});
		uni.connectSocket({
			url: "ws://boeradmin.cn:8282",
		});
		uni.onSocketOpen(function(res) {
			console.log("WebSocket连接已打开！", res);
		});
		uni.onSocketError(function(res) {
			console.log("WebSocket连接打开失败，请检查！", res);
		});
		uni.onSocketMessage(function(res) {
			console.log("收到服务器内容：" + res.data);
			setTimeout(() => {
				uni.hideLoading()
			}, 1500)
			t.status = "握手成功..."
			let uid = uni.getStorageSync("uid");
			let type = JSON.parse(res.data).type;
			if (type == 'init') {
				let client_id = JSON.parse(res.data).client_id;
				//绑定用户id:uid
				t.handleBindUserId(uid, client_id);
			} else {
				let phone = JSON.parse(res.data).phone;
				
				//h5以及小程序使用
				uni.makePhoneCall({
	            phoneNumber: phone,//仅为示例
              });
            
				//APP自动拨打电话
				// #ifdef APP-PLUS
				 plus.device.dial(phone, false);
				// #endif
			}
		});
	},
  ```
- 监听通话状态

    > 在onReady中进行调用且监听注册成功后会在重复调用，需要在此处进行限制。
    ```javascript
        //监听用户通话状态
		  listenPhoneStatus() {
			 let t = this;
			 let status = 2;
				
			 // #ifdef APP-PLUS
				
			 let maintest = plus.android.runtimeMainActivity();
			 let Contexttest = plus.android.importClass("android.content.Context");
			 let telephonyManager = plus.android.importClass("android.telephony.TelephonyManager");
			 let telManager = plus.android.runtimeMainActivity().getSystemService(Contexttest.TELEPHONY_SERVICE);
			 let receiver = plus.android.implements('io.dcloud.android.content.BroadcastReceiver', {
			 onReceive: function(Contexttest, intent) {
				plus.android.importClass(intent);
				let phonetype = telManager.getCallState();
				let phoneNumber = intent.getStringExtra(telephonyManager.EXTRA_INCOMING_NUMBER);
				
				    //  处理数据重复问题（不然会进行多次提交）
				    
				    if (t.callStatus == phonetype) {
							t.callStatus = t.callStatus == 2 ? 0 : 2;
							switch (phonetype) {
								case 0:
									console.log('空闲状态')
									t.getCalllog();
									break;
								case 1:
									console.log('振铃状态')
									break;
								case 2:
									console.log('通话存在')
									break;
							}
							return;
						}
					}
				});

			 let IntentFilter = plus.android.importClass('android.content.IntentFilter');
			 let filter = new IntentFilter();
				filter.addAction(telephonyManager.ACTION_PHONE_STATE_CHANGED);
			 maintest.registerReceiver(receiver, filter);
			 
			 // #endif

			},
    ```
- 获取用户通话记录
    > h5plus获取用户通话记录的时候，首先要定义一个空的数组，取其中的50条数据存放于数组中，数组第一条数据就是最新的通话记录，此处有一个小bug，如果单独获取一条通话记录或者获取的通话记录数量不够多的话是无法获取到最新的一条数据的。具体原因未知。
       
    ```javascript
        // 获取用户通话记录列表
        getCalllog() {
			    let t = this;
				//#ifdef APP-PLUS
				// 获取通话记录的主体代码 顺序不能够乱
				var CallLog = plus.android.importClass('android.provider.CallLog');
				var Activity = plus.android.runtimeMainActivity();
				var ContentResolver = plus.android.importClass('android.content.ContentResolver');
				var resolver = Activity.getContentResolver();
				plus.android.importClass(resolver);
				var String = plus.android.importClass("java.lang.String");

				var cs = resolver.query(CallLog.Calls.CONTENT_URI, null, null, null, CallLog.Calls.DEFAULT_SORT_ORDER);
				plus.android.importClass(cs);
				let callLogListArr = [];
				var count = 0; // 记录多少条 用于处理循环跳出
				while (cs.moveToNext()) {
					count++;
					//号码
					var number = cs.getString(cs.getColumnIndex(CallLog.Calls.NUMBER));
					//呼叫类型
					var type;
					switch (parseInt(cs.getString(cs.getColumnIndex(CallLog.Calls.TYPE))))
					// 判断通话类型
					{
						case CallLog.Calls.INCOMING_TYPE:
							type = "呼入";

							break;
						case CallLog.Calls.OUTGOING_TYPE:
							type = "呼出";

							break;
						case CallLog.Calls.MISSED_TYPE:
							type = "未接";

							break;
						case CallLog.Calls.REJECTED_TYPE:
							type = "拒绝";

							break;
						case CallLog.Calls.BLOCKED_TYPE:
							type = "阻止";

							break;
						case CallLog.Calls.ANSWERED_EXTERNALLY_TYPE:
							type = "接听";

							break;
						default:
							type = "挂断";

							break;
					}
					// 获取时间
					var date = new Date(parseInt(
						cs.getString(cs.getColumnIndexOrThrow(CallLog.Calls.DATE))));
					// 联系人
					var Name_Col = cs.getColumnIndexOrThrow(CallLog.Calls.CACHED_NAME);
					var name = cs.getString(Name_Col);
					// 号码归属地 返回：北京 联通
					var numberLocation = cs.getString(
						cs.getColumnIndex(CallLog.Calls.GEOCODED_LOCATION)
					);
					//通话时间,单位:s
					var Duration_Col = cs.getColumnIndexOrThrow(CallLog.Calls.DURATION);
					var duration = cs.getString(Duration_Col);
					// 存入数组 
					let obj = {
						name: name, // 联系人的姓名
						mobile: number, // 联系人电话
						numberLocation: numberLocation, // 号码的归属地
						callTimeStr: date,
						callTime: t.$u.timeFormat(new Date(date), 'yyyy-mm-dd hh:MM:ss'), // 呼入或呼出时间
						talkTime: duration, // 通话时长
						type: type
					}
					callLogListArr.push(obj)

					if (count > 50) {
						t.callLog = callLogListArr[0];
						t.endTimeStr = Date.parse(new Date(callLogListArr[0].callTimeStr)) + (duration * 1000);
						t.handleGetCallDir()
						break;
					}
				}
				//#endif
			},

    ```
    
- 获取用户通话录音文件(递归)
    > 重点来了！！！
    - 获取手机根目录  
    ```javascript
        // 进入系统即获取系统根目录
			getRootPath() {
				let t = this;
				let environment = plus.android.importClass("android.os.Environment");
				let sdRoot = environment.getExternalStorageDirectory(); //文件夹根目录 
				t.sdRoot = sdRoot;
			},
    ```
    - 获取通话录音储存文件目录
    
    > addFile 与 addData的所有参数均要写成字符串，否则无法成功上传，h5plus文档中给的示例键不是字符串
    ```javascript
        //获取录音文件夹路径
        getVoice(path) {
				// let path = path;
				let pathStr = this.sdRoot + path;
				let resFilePath = this.dealFilesTree(pathStr); // 获取录音文件绝对路径
			},
			
			//递归获取录音文件路径
		   dealFilesTree(path) {
				let t = this;
				let File = plus.android.importClass("java.io.File");

				let readFr = new File(path);
				// 获取指定路径下所有的文件及目录
				let fileArr = readFr.list();
				// 判断是否空目录
				if (fileArr) {
					for (let i = 0; i < fileArr.length; i++) {
						let pathStr = path + "/" + fileArr[i]
						let tmpFile = new File(pathStr);
						if (tmpFile.isFile()) {
					
					      //获取文件信息
							let name = tmpFile.getName();//文件名称
							let size = tmpFile.length();//文件大小
							let time = tmpFile.lastModified();//文件创建时间
							
							//正则匹配音频文件
							if (/\.(mp3|m4a|wav|amr|awb|aac|flac|mid|midi|xmf|rtttl|rtx|wma|ra|mka|m3u|pls)$/.test(name) &&
								size > 0) {
								
								//依据通话时间匹配筛选通话录音文件：匹配规则两者时间相差不超过10s的文件。
								let time2 = t.endTimeStr / 1000
								if (Math.abs(time2 - (time / 1000)) < 10) {
								
								  // 使用native.js调用原生接口上传文件
									var task = plus.uploader.createUpload(t.$store.state.apiBaseUrl + "/upload/index", {
											method: "POST",
											priority: 1000
										},
										function(res, status) {
											// 上传完成
											let getAbsolutePath = tmpFile.getAbsolutePath();
											if (status == 200) {
											
											  //获取文件网络地址，并上传通话记录
												let url = JSON.parse(res.responseText).data;
												t.uploadCallRecord(url, getAbsolutePath);

											}
											return;
										})
										
									//注意：此处巨坑，addFile 与 addData的所有参数均要写成字符串，否则无法成功上传，h5plus文档中给的示例键不是字符串
									task.addFile(tmpFile.getPath(), {
										'key': 'file'
									});
									task.addData('type', "2")
									task.addData('path', 'crm/call_log_app/');
									task.start();
								}
							}
						} else {
							let resFile = t.dealFilesTree(tmpFile.getPath());
							if (resFile) {
								return resFile;
							}
						}
					}
				}
				return '';
			},
			
    ```
    
    - app后台运行

    > 此处代码要放在main.js中
    ```javascript
        // #ifdef APP-PLUS
            let main = plus.android.runtimeMainActivity();
            //为了防止快速点按返回键导致程序退出重写quit方法改为隐藏至后台  
            plus.runtime.quit = function() {
            	main.moveTaskToBack(false);
            };
            //重写toast方法如果内容为 ‘再次返回退出应用’ 就隐藏应用，其他正常toast 
            plus.nativeUI.toast = (function(str) {
            	if (str == '再次返回退出应用') {
            		plus.runtime.quit();
            	} else {
            		uni.showToast({
            			title: '再次返回退出应用', //可以自定义其他弹出显示的内容
            			icon: 'none'
            		})
            	}
            });
        // #endif
    ```
    - 检测是否开启`允许在其他APP上层使用`
    > `允许在其他APP上层使用`是APP后台运行时或者开启悬浮窗以及安卓保活的关键，确保app在后台运行时也能收到websocket推送的电话号码并拨打电话。
    ```javascript
        //检测是否开启允许在其他APP上层使用
			checkAndroid_overlays() {
				const isIos = uni.getSystemInfoSync().platform == 'ios'
				const android_overlays = () => {
					var main = plus.android.runtimeMainActivity()
					var pkName = main.getPackageName()
					var Settings = plus.android.importClass('android.provider.Settings')
					var Uri = plus.android.importClass('android.net.Uri')
					var Build = plus.android.importClass('android.os.Build')
					var Intent = plus.android.importClass('android.content.Intent')
					var intent = new Intent(
						'android.settings.action.MANAGE_OVERLAY_PERMISSION',
						Uri.parse('package:' + pkName)
					)
					// main.startActivityForResult(intent, 5004);  
					if (!Settings.canDrawOverlays(main)) {
						// 检测悬浮窗  
						uni.showModal({
							title: '温馨提示',
							content: '为了更方便的使用,请先打开系统悬浮窗权限',
							showCancel: true,
							success: function(res) {
								if (res.confirm) {
									main.startActivityForResult(intent, 5004) // 转跳到悬浮窗设置  
								}
							}
						})
					}
				}
				const ios_overlays = () => {
					// IOS不支持,无需判断 
				}
				return !isIos ? android_overlays() : ios_overlays()
			},
    ```
    
   
    - 悬浮窗功能
    > 悬浮窗功能使用了uniapp插件库中的`Ba-FloatWindow` 插件，具体使用方法以及文档请前往查看。
    
    - 安卓保活
    > 安卓保活功能使用了uniapp插件库中的`Ba-KeepAlive` 插件，具体使用方法以及文档请前往查看。安卓保活功能可以确保APP在手机锁屏的状态下也能自动拨打电话。
    

### 技术待优化
- 目前自动通话录音功能还需要用户去进行手动开启。
- 在上传完录音文件后使用File类的delete()方法无法删除文件。

### 总结

因本人是纯粹的web前端，在做此项目前没有接触过安卓的一些技术知识点，上述的技术待优化确实是我现在无法解决的问题，之后会在进行更深入的学习后进行完善优化。

### 参考文档
- [HTML5+规范](https://www.html5plus.org/doc/h5p.html)
- [基于native.js的文件系统管理功能实现](https://ask.dcloud.net.cn/article/809)
- [Native.js示例汇总](https://ask.dcloud.net.cn/article/114)

 
    
