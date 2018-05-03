<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name='viewport' content='width=device-width,initial-scale=1.0,user-scalable=no'>
	<title>websocket测试平台</title>
	<link rel="stylesheet" href="./resource/amazeui/css/amazeui.min.css">
	<link rel="stylesheet" type="text/css" href="./resource/iconfont/iconfont.css">
	<link rel="stylesheet" href="./resource/stylesheets/common.css">
	<link rel="stylesheet" href="./resource/stylesheets/websocket.css">
	<script src="./resource/javascripts/vue.min.js"></script>
</head>
<body>
	<div id='websocket'>
		<div class='h1 tcenter'><label for="">芯智云websocket测试平台</label></div>
		<div class='wTop am-g'>
			<span class='am-u-md-4 am-u-sm-12 am-u-xs-12'>
				<label for="">域&emsp;名: </label>
				<input type="text" placeholder="请输入域名" v-model='url'>
			</span>
			<span class='am-u-md-4 am-u-sm-12 am-u-xs-12'>
				<label for="">端口号: </label>
				<input type="text" placeholder="请输入端口号" v-model='port'>
			</span>
			<span class='am-u-md-4 am-u-sm-12 am-u-xs-12'>
				<label for="">设备id: </label>
				<input type="text" placeholder="请输入设备id" v-model='deviceid'>
			</span>
		</div>
		<ul class='am-g content'>
			<li class='am-u-md-6 am-u-sm-12 am-u-xs-12'>
				<button @click='link'>连接</button>
				<button @click='offline'>断开</button>
				<ul>
					<label for="">状态:</label>
					<li v-text='status'></li>
				</ul>
			</li>
			<li class='am-u-md-6 am-u-sm-12 am-u-xs-12 set'>
				<div>
					<label for="">水机状态</label>
					<input type="text" v-model='setObj.DeviceStause'>
				</div>
				<ul>
					<li>
						<!-- <button>冲洗</button>
						<button>开机</button>
						<button>关机</button> -->
					</li>
					<li class='cfix'>
						<span class='am-u-md-6 am-u-sm-6 am-u-xs-12'>
							<label for="">剩余流量：</label>
							<input type="text" v-model='setObj.reFlow'>
						</span>
						<span class='am-u-md-6 am-u-sm-6 am-u-xs-12'>
							<label for="">剩余天数：</label>
							<input type="text" v-model='setObj.reDay'>
						</span>
					</li>
					<li class='cfix'>
						<span class='am-u-md-6 am-u-sm-6 am-u-xs-12'>
							<label for="">总&nbsp;&nbsp;流&nbsp;&nbsp;量：</label>
							<input type="text" v-model='setObj.allFlow'>
						</span>
						<span class='am-u-md-6 am-u-sm-6 am-u-xs-12'>
							<label for="">总&nbsp;&nbsp;天&nbsp;&nbsp;数：</label>
							<input type="text" v-model='setObj.allDay'>
						</span>
					</li>
				</ul>
				<div>
					<p><label for="">滤芯：</label></p>
					<span>剩余流量&nbsp;<span v-text='showObj.reFlow'></span>&nbsp;</span>
					<span>剩余天数&nbsp;<span v-text='showObj.reDay'></span>&nbsp;</span>
					<span>总流量&nbsp;<span v-text='showObj.allFlow'></span>&nbsp;</span>
					<span>总流量&nbsp;<span v-text='showObj.allDay'></span>&nbsp;</span>
				</div>
				<div style='margin: 2vmin 0;'>
					<button @click='setDataFn'>设置</button>
					<button @click='clearSetObj'>清空</button>
				</div>
			</li>
		</ul>
		<div id='receive'>
			<p>接收:</p>
			<table>
				<thead>
					<tr>
						<th>数据包类型</th>
						<th>设备状态</th>
						<th>剩余流量</th>
						<th>剩余天数</th>
						<th>总流量</th>
						<th>总天数</th>
						<th>时间</th>
					</tr>
				</thead>
				<tbody>
					<tr v-for='s in receive'>
						<td v-text='s.PackType'>数据包类型</td>
						<td v-text='s.DeviceStause'>设备状态</td>
						<td v-text='s.reFlow'>剩余流量</td>
						<td v-text='s.reDay'>剩余天数</td>
						<td v-text='s.allFlow'>总流量</td>
						<td v-text='s.allDay'>总天数</td>
						<td v-text='s.time'>时间</td>
					</tr>
				</tbody>
			</table>
		</div>
	</div>
	<script src="./resource/javascripts/websocket.js"></script>
	<script src="./resource/javascripts/public.js"></script>
	<script>
		var websocket = new Vue({
			el: '#websocket',
			data(){
				return {
					wsurl: '',		// websocket 地址
					url: '',
					port: '',
					deviceid: '',
					receive: [],	// 接收到的数据
					status: '',
					// 设备状态集合
					dstauseList: {'0':'制水','1':'冲洗','2':'水满','3':'缺水','4':'漏水','5':'检修','6':'欠费停机','7':'关机','8':'开机(仅命令)'},
					//登录包
					login: {
					    "DeviceID": '',
					    "PackType": "login",
					    "Vison": 0,
					},
					setData: {
						DeviceID: '',		// 设备id
						DeviceStause: '',	//设备状态码
						PackType: 'SetData',	// 数据包类型
					},
					showObj: {
						reFlow: '',		// 剩余流量
						reDay: '',		// 剩余天数
						allFlow: '',		// 总流量
						allDay: '',		// 总天数
					},
					setObj: {
						reFlow: '',		// 剩余流量
						reDay: '',		// 剩余天数
						allFlow: '',		// 总流量
						allDay: '',		// 总天数
					}
				}
			},
			mounted(){
				this.url = '120.78.184.0';
				this.port = '6001';

				// this.receive = [{
				// 	PackType: 'login',
				// 	DeviceStause: '1',
				// 	reFlow: '10',
				// 	reDay: '20',		
				// 	allFlow: '50',		
				// 	allDay: '60',		
				// 	time: '20180503',		
				// }]
			},
			methods: {
				link: function(){
					websocket.wsurl = 'ws://' + websocket.url + ':' + websocket.port;
					websocket.login['DeviceID'] = websocket.deviceid;	// 登录包
					if(!websocket.url){
						noticeFn({text:'请输入域名'});
						return
					}else if(!websocket.port){
						noticeFn({text:'请输入端口号'});
						return
					}else if(!websocket.deviceid){
						noticeFn({text:'请输入设备id'});
						return
					}
					// 连接websocket
					/*
						调用封装的websocket函数
						函数在 websocket.js中
					*/ 
					wsFun(websocket.wsurl, function(){
						// 连接成功
						// 发送登录包
						websocket.sendMessage(websocket.login);

					}, function(str, res){
						// str: 数据类型（接收，关闭， 错误），
						// 在websocket.js中给出，不需要传
						console.log(str, res);
						
						// 接收数据更新到页面
						if(str == 'message'){
							var data = JSON.parse(res);
							console.log(data);
							websocket.status = websocket.dstauseList[data.DeviceStause];
							websocket.showObj.reFlow = data.reFlow;
							websocket.showObj.reDay = data.reDay;		
							websocket.showObj.allFlow = data.allFlow;		
							websocket.showObj.allDay = data.allDay;
							
							websocket.receive.push(data);
							// 多于20条则清空
							if(websocket.receive.length >= 20){
								websocket.receive.splice(0,20);
							}
						}
						// console.log(websocket.receive)
					});
				},
				offline: function(){
					// 断开websocket
					ws.close();
				},
				sendMessage: function(data){
					// 发送json格式数据
					if(Object.prototype.toString.call(data) === "[object Object]"){
						data = JSON.stringify(data);
					}
					// 发送数据
					sendmsg(data);
				},
				// 发送设置包
				setDataFn: function(){

					websocket.setData['DeviceID'] = websocket.deviceid;	
					websocket.setData['DeviceStause'] = websocket.setObj['DeviceStause'];	
					websocket.setData['reFlow'] = websocket.setObj['reFlow'];
					websocket.setData['reDay'] = websocket.setObj['reDay'];
					websocket.setData['allFlow'] = websocket.setObj['allFlow'];
					websocket.setData['allDay'] = websocket.setObj['allDay'];
					console.log(websocket.setData);
					// 发送数据
					websocket.sendMessage(websocket.setData);
				},
				// 清空设置的数据
				clearSetObj: function(){
					websocket.setObj['DeviceStause'] = '';	
					websocket.setObj['reFlow'] = '';
					websocket.setObj['reDay'] = '';
					websocket.setObj['allFlow'] = '';
					websocket.setObj['allDay'] = '';
				}
			}
		})
	</script>
</body>
</html>