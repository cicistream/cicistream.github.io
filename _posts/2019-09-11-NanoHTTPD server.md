---
layout: post
title: "React-Native局域网内跨平台互传文件"
subtitle: "同一个WiFi下,手机与PC实现互传"
date:  2019-09-11
author: "cici"
header-img: "img/post-bg-rwd.jpg"
tags:
  - mobile
  - service
  - React-Native
  - NanoHTTPD
---

我使用的技术栈是：react-native（0.56.0）+ [react-navigation](https://reactnavigation.org/zh-Hans/) + react-redux + ant-design + axios

-------

我在做的ReactNative项目需要实现同一WIFI下跨平台互传文件功能，涉及原生模块的功能

我实现这两个功能使用的库有
- [@react-native-community/netinfo](http://npm.taobao.org/package/@react-native-community/netinfo)
- [NanoHTTPD](https://github.com/NanoHttpd/nanohttpd)

-------

首先需要明白限制条件是在同一个局域网内（连接同一wifi），前期已经用umi开发了一组传输文件时准备在PC上展示的相关网页并打包出来。

## 核心技术：

通过使用NanoHTTPD在手机上建立本地服务器实现数据传输。
为了保证安全性，需要PC端输入APP生成的验证码予以验证。

## 整体流程：

### 1. 判断是否连接WIFI并获取ip地址

 WifiManager是 Android 暴露给开发者使用的一个系统服务管理类, 其中包含对WiFi的响应的操作函数。

```java
    //获取wifi服务
    WifiManager wifiManager = (WifiManager) context.getApplicationContext().getSystemService(Context.WIFI_SERVICE);
    //判断wifi是否开启
    if (!wifiManager.isWifiEnabled()) {
        return null;
    }
    //判断wifi是否连接
    if (wifiManager.getConnectionInfo() == null) {
        return null;
    }
    // ip地址获取
int ipAddress = wifiManager.getConnectionInfo().getIpAddress();
    return (ipAddress & 0xFF) + "." +
            ((ipAddress >> 8) & 0xFF) + "." +
            ((ipAddress >> 16) & 0xFF) + "." +
            (ipAddress >> 24 & 0xFF);

```

### 2. 启动服务

主要逻辑继承NanoHTTPD，根据你的需求重写serve方法就可以了

在我写的this.openHttp(port,code,mainKey,toPage)中，code是验证码，mainkey是自定义需要传输的部分内容，toPage是判断是由手机向PC还是PC向手机传输文件。

```java
// 随机数防止端口被占用，如果被占用catch后stopServer
int port = (int)((Math.random()*9+1)*1000);
final String host = ip + ":" + port;
this.openHttp(port,code,mainKey,toPage);
promise.resolve("http://" + host);
// 
private void openHttp(int port,String code,ReadableMap mainKey, String toPage) throws IOException {
        httpServer = new HttpServer(getReactApplicationContext(), port,code,mainKey,toPage);
        httpServer.start();
    }
```

### 3. 自定义的serve，返回网页或者响应请求

根据不同的路径返回不同的数据

将文件转化的字符串或者数组作为响应内容返回return  Response.newFixedLengthResponse(字符串)
或者return  Response.newFixedLengthResponse(状态码，mime类型，字节数组)

```java
public  Response serve(IHTTPSession session){
        String uri = session.getUri();
        // 根据html的路径返回相应的操作
       if (uri.equals("/postValidatePhoneCode")) {
            return validatecode(session);
        }else if (uri.equals("/upload")){
            return renderUpload(session);
        }else if (uri.equals("/getPhoneKey")){
            return getKey(session);
        }else if (uri.equals("/device")){
            return download(session);
        }else if (uri.indexOf(".json")!=-1){
            return downloadKey(session);
        } else {
            return renderRes(session);
        }
    }

```

###  renderRes

不带路径的url根据toPage参数返回html，保证PC访问到初始页面

```java
private Response renderRes(IHTTPSession session) {
        String uri = session.getUri();
        String filename;
        if (uri.equals("/")) filename = "index.html";
        else filename = uri.substring(1);     
// 定义mineType
        String mimeType = null; 
        if (filename.endsWith(".html") || filename.endsWith(".htm")) {
            mimeType = "text/html";
        } else if (filename.endsWith(".js")) {
            mimeType = "text/javascript";
        } else if (filename.endsWith(".css")) {
            mimeType = "text/css";
        }
        else if (filename.endsWith(".ico")) {
            mimeType = "image/x-icon";
        }
        if (mimeType == null) {
            return newFixedLengthResponse(Response.Status.BAD_REQUEST, "text/plain", "未知文件类型[" + filename + "]");
        }
        InputStreamReader inputStreamReader = null;
        BufferedReader reader = null;

        try {
//            filename = "Resources"+filename;
            inputStreamReader = new InputStreamReader(context.getAssets().open(filename));
            reader = new BufferedReader(inputStreamReader);

            StringBuilder returnMsg = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                returnMsg.append(line);
            }
            return newFixedLengthResponse(Response.Status.OK, mimeType, returnMsg.toString());

        } catch (IOException e) {
            Utils.closeSilent(reader);
            Utils.closeSilent(inputStreamReader);
            e.printStackTrace();
            return newFixedLengthResponse(Response.Status.INTERNAL_ERROR, "text/plain", e.getMessage());
        }
    }

```

###  validatecode 验证验证码

```java
    session.parseBody(files);
    String param=files.get("postData");
    JSONObject json = new JSONObject(param);
    String code = json.getString("code");
    if (code.equals(this.code+"")){
        Map map = new HashMap();
        map.put("validate",true);
        map.put("toPage",this.toPage);
        JSONObject jsonObj=new JSONObject(map);
        return newFixedLengthResponse(Response.Status.OK, "text/plain", jsonObj.toString());
    }else {
        Map map = new HashMap();
        map.put("validate",false);
        JSONObject jsonObj=new JSONObject(map);
        return newFixedLengthResponse(Response.Status.REDIRECT_SEE_OTHER, "text/plain", jsonObj.toString());
    }

```

验证码正确则跳转到相应页面，进行下载或上传

###  download  上传文件到PC

```java
private Response downloadKey(IHTTPSession session) {
    try {
        File distFile = new File(this.context.getFilesDir().getAbsolutePath() + File.separatorChar + "wifi" + File.separatorChar + this.mainKey.getString("key_name") + ".json");
        //判断该文件的所属文件夹存不存在，不存在则创建文件夹
        if(!distFile.getParentFile().exists()){
            distFile.getParentFile().mkdirs();
        }
        //创建文件
        if (!distFile.exists()){
            //如果不存在file文件，则创建
            distFile.createNewFile();
        }
        //创建字符流（使用字节流比较麻烦）
        FileWriter fw = new FileWriter(distFile);
        Map map = new HashMap();
        map.put("address",this.mainKey.getString("pubaddress"));
        map.put("algo","0x03");
        map.put("encrypted",this.mainKey.getString("privateKeystr").substring(2,this.mainKey.getString("privateKeystr").length()));
        map.put("version","2.0");
        map.put("privateKeyEncrypted",false);
        JSONObject jsonObj=new JSONObject(map);

        fw.write(jsonObj.toString());
        //这里要说明一下，write方法是写入缓存区，并没有写进file文件里面，要使用flush方法才写进去
        fw.flush();
        //关闭资源
        fw.close();
        FileInputStream fis = new FileInputStream(distFile.getPath());
        return newFixedLengthResponse(Response.Status.OK, "application/octet-stream", fis,fis.available());
    } catch (FileNotFoundException e) {
        e.printStackTrace();
        return newFixedLengthResponse(Response.Status.BAD_REQUEST, "text/plain", e.getMessage());
    } catch (IOException e) {
        e.printStackTrace();
        return newFixedLengthResponse(Response.Status.BAD_REQUEST, "text/plain", e.getMessage());
    } catch (Exception e) {
        e.printStackTrace();
        return newFixedLengthResponse(Response.Status.BAD_REQUEST, "text/plain", e.getMessage());
    }

}
```

###  upload  从PC端下载文件

```java
private void sendUploadFileToClient(IHTTPSession session) throws IOException, ResponseException {

    Map<String, String> files = new HashMap<>();
    session.parseBody(files);

    Map<String, List<String>> parameters = session.getParameters();

    WritableArray params = Arguments.createArray();
    String str = "";
    for (String key : files.keySet()) {
        List<String> nameStr = parameters.get(key);
        if (nameStr == null) continue;
        File tempFile = new File(files.get(key));
        File distFile = new File(this.context.getFilesDir().getAbsolutePath() + File.separatorChar + "wifi" + File.separatorChar + tempFile.getName());

        Utils.copyFile(tempFile, distFile);//NanoHttpd 会自动在请求完毕删除掉文件 所以克隆一份

        WritableMap map = Arguments.createMap();
        map.putString("name", nameStr.get(0));
        map.putString("path", distFile.getPath());
        params.pushMap(map);
        str = this.readTxt(distFile.getPath());
    }
    // 页面监听WifiServer.FILE_UPLOAD_NEW事件，接收从PC导入的数据
    this.context.getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class).emit(WifiServer.FILE_UPLOAD_NEW, str);
    }
    private  String readTxt(String filePath) {
        StringBuilder result = new StringBuilder();
        try {
            BufferedReader bfr = new BufferedReader(new InputStreamReader(new FileInputStream(new File(filePath)), "UTF-8"));
            String lineTxt = null;
            while ((lineTxt = bfr.readLine()) != null) {
                result.append(lineTxt).append("\n");
            }
            bfr.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result.toString();
    }

```

## 4. 关闭本地服务

```java
this.httpServer.stop();
```

以上就是手机端起简易服务器与PC端实现互传的流程，涉及到很多java文件读取的方法，我本来想使用nanohttpd的webserver解决，但是发现他只能展示页面，不好修改（它本身就是依赖nanohttpd完成的，更多信息在nanohttpd的官方文档）。读了源码也有些一知半解，最后在公司前辈的帮助下完成的该功能。

感谢阅读，有疑问请留言，我会尽我所能解答的。（要源码的就别留了）



