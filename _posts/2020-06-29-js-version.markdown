---
layout:     post
title:      "aiotc视频云平台开发记录"
subtitle:   "测试步骤"
date:       2020-06-29
author:     "debugger999"
header-img: "img/post-bg-desk.jpg"
tags:
    - C/C++
    - 视频编解码
    - 流媒体
    - 深度学习算法
    - AIoTCloud
---

## 测试环境
	处理器: x86-64bit
	操作系统: ubuntu 16.04

## 下载
	git clone https://github.com/debugger999/aiotc.git

## 编译
	cd aiotc
	make

## 准备工作
	安装启动nginx
	cd tools/nginx
	./install.sh
	/usr/local/nginx/sbin/nginx

	配置mongodb(用于存储用户命令参数，初级测试可省略)
	需要一个启动好的mongodb服务器，然后配置src/config.json中如下mongodb参数：
	"db": {
      "type": "mongodb",
      "mongodb": {
          "host": "192.168.0.99",
          "port": 27017,
          "user": "null",
          "password": "null",
          "dbName": "null"
      }
	}

	需要一个启动好的rabbitMQ服务器，用于结构化信息输出(初级测试可省略，可直接查看函数output_thread里面的json)

## 运行
	cd src
	./aiotcProc

## 初始化(以下命令皆可参考"RESTAPI接口")
	方法: http post，推荐用postman，也可以用其他任何http client工具，如curl、python、java等等
	url: http://ip:11706/system/init
	参考参数(ip地址替换为自己的真实地址)：
	{
      "masterIp": "192.168.0.100",
      "msgOutParams":[
        {
            "type":"mq",
            "host":"192.168.0.10",
            "port":5672,
            "userName":"guest",
            "passWord":"guest",
            "exchange":"aiotc.exchange.message",
            "routingKey":""
        }
      ]
	}

## 添加slave
	url: http://ip:11706/system/slave/add
	参考参数：
	{
      "slaveIp":"192.168.0.100",
      "restPort":11762,
      "streamPort":11764,
      "internetIp":"互联网部署时的公网IP地址，无则删除此字段"
	}

## rtsp测试
	添加设备
	url: http://ip:11706/obj/add/rtsp
	参考参数：
	{
      "name":"test",
      "type":"camera",
      "id":3,
      "data":{
          "subType":"rtsp",
          "tcpEnable":0,
          "url":"rtsp://admin:12345@192.168.0.64/h264/ch1/main/av_stream"
      }
	}

	开始拉流
	url: http://ip:11706/obj/stream/start
	参考参数：
	{
      "id":3
	}

	开始视频预览
	url: http://ip:11706/obj/preview/start
	参考参数：
	{
      "id":3,
      "type": "hls"
	}
	返回：
	{
      "code":0,
      "msg":"success",
      "data":{
          "url":"http://192.168.0.100:8080/m3u8/stream3/play.m3u8"
      }
	}

	此时即可用vlc或浏览器hls插件播放url视频

## GAT1400测试
	上面初始化步骤时需要添加gat1400参数，详见"RESTAPI接口"
	参考参数：
	{
      "gat1400Params": {
          "localGatHostIp": "34010000002000000001",
          "localGatServerId": "32020200002000000002",
          "localGatPort": 7100,
          "platform": [
              {
                  "id": "32020200002000000001",
                  "ip": "192.168.0.99",
                  "port": 7200,
                  "username": "admin",
                  "password": "admin"
              }
          ]
      }
	}

	添加设备
	url: http://ip:11706/obj/add/gat1400
	参考参数：
	{
      "name":"test",
      "type":"camera",
      "id":3,
      "data":{
          "subType":"gat1400",
          "deviceId": "32020200002000000003",
          "mode": 0,
          "slaveIp": "192.168.0.100",
          "platformId": "32020200002000000001"
      }
	}

	开始抓拍
	url: http://ip:11706/obj/capture/start
	参考参数：
	{
      "id":3
	}

	此时消费rabbitMQ或打开函数output_thread里面的json调试信息，即可看到如下结果：
	{
      "msgType": "common",
      "data":{
          "id":3,
          "timeStamp": 99999999,
          "sceneImg": {
              "url": "http://xxxx:xxxx/scene.jpg"
          },
          "person":[{
              "face":{
                  "url": "http://xxxx:xxxx/face.jpg",
                  "rect": {"x":0.001,"y":0.001,"w":0.012,"h":0.033},
                  "quality": 0.9
              },
              "body":{
                  "url": "http://xxxx:xxxx/body.jpg",
                  "rect": {"x":0.001,"y":0.001,"w":0.012,"h":0.033}
              },
              "property":{
                  "gender":{"value":"man","confidence":99.99},
                  "age":{"value":33,"confidence":99.99}
              }
          }],
          "veh":[{
              "plate":{
                  "color":"blue",
                  "plateNo":"京A99999",
                  "url": "http://xxxx:xxxx/plate.jpg",
                  "rect": {"x":0.001,"y":0.001,"w":0.012,"h":0.033}
              },
              "body":{
                  "url": "http://xxxx:xxxx/body.jpg",
                  "rect": {"x":0.001,"y":0.001,"w":0.012,"h":0.033}
              },
              "property":{
                  "color":{"value":"blue","confidence":99.99},
                  "brand":{"value":"宝马-宝马X5-2014","confidence":99.99}
              }
          }]
      }
	}

## streamsdk互联网接入测试
	开发中

## 海康相机Ehome互联网接入测试
	开发中

## GB28181测试
	开发中

## 算法测试
	开发中

## 其他
	无

