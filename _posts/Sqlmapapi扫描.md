title: Sqlmapapi扫描
date: 1912-12
categories: [sqlmap]
tags: [sqlmapapi,批量]
---

Usage: sqlmapapi.py [options]

Options:
  -h, --help            show this help message and exit
  -s, --server          Act as a REST-JSON API server
  -c, --client          Act as a REST-JSON API client
  -H HOST, --host=HOST  Host of the REST-JSON API server
  -p PORT, --port=PORT  Port of the the REST-JSON API server
  
<!--more-->
从sqlmapapi.py文件可以看出来,我们利用的文件的调用关系是

进入到lib/utils/api.py的server类,可以发现通过向server提交数据进行与服务的交互。 一共分为3种类型。

Users' methods 用户方法
Admin function 管理函数
sqlmap core interact functions 核心交互函数
可以提交数据的种类如下。

用户方法

@get("/task/new")
@get("/task//delete")
管理函数

@get("/admin//list")
@get("/admin//flush")
核心交互函数

@get("/option//list")
@post("/option//get")
@post("/option//set")
@post("/scan//start")
@get("/scan//stop")
@get("/scan//kill")
@get("/scan//status")
@get("/scan//data")
@get("/scan//log//")
@get("/scan//log")
@get("/download///")
不难发现这些操作可以完全满足我们的测试需求,因此利用这些就可以批量了。当然每一种请求都会有不同的返回值,这些返回值是json的形式传回, 解析就好了。其实这些我已经替大家做好了,调用AutoSqli类就可以了,但是还是要挑一些讲下。

task/new 任务建立

GET /task/new Response:
{
    "taskid": "1d47d7f046df1504"
}
 
    /scan/<task_id>/status 扫描任务状态
GET /scan/<task_id>/status Response:
{
    "status": "terminated",
    "returncode": 0 
}


http://drops.wooyun.org/tips/6653?utm_source=tuicool
