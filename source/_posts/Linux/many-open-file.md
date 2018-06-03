---
title: linux many open file问题排查
categories:
  - Linux
tags: []
abbrlink: 5618a78c
date: 2017-07-07 08:51:38
---
# lsof查询many open file等问题

### 1、netstat显示的tcp连接数正常
```sh
[imhtp@im2-wx-kf2 ~]$ netstat -n | awk '/^tcp/ {++state[$NF]} END {for(key in state) print key,"\t",state[key]}'
TIME_WAIT    8481
CLOSE_WAIT   322
FIN_WAIT2    43
ESTABLISHED      144
```
<!-- more -->
### 2、ss -s显示大量的closed连接
```sh
[imhtp@im2-wx-kf2 ~]$ ss -s
Total: 707 (kernel 766)
TCP:   8483 (estab 130, closed 7910, orphaned 0, synrecv 0, timewait 7896/0), ports 685

Transport Total     IP        IPv6
*     766       -         -
RAW   0         0         0
UDP   10        6         4
TCP   573       13        560
INET      583       19        564
FRAG      0         0         0
```
上面信息说明存在socket fd泄漏，那么用lsof命令检查系统sock的文件句柄。closed 7910，很多socket是处于closed状态。

### 3、lsof | grep sock
```sh
[imhtp@im2-wx-kf2 ~]$ lsof | grep sock
java       1029   imhtp   36u     sock                0,6        0t0   7121949 can't identify protocol
java       1773   imhtp   36u     sock                0,6        0t0     13282 can't identify protocol
java       2747   imhtp   36u     sock                0,6        0t0 129765023 can't identify protocol
java       7070   imhtp   36u     sock                0,6        0t0   9465828 can't identify protocol
java       7681   imhtp   36u     sock                0,6        0t0 268478565 can't identify protocol
java       7681   imhtp  192u     unix 0xffff88004ac89080        0t0 268479492 socket
java      12545   imhtp   38u     sock                0,6        0t0 187396984 can't identify protocol
java      12545   imhtp  190u     unix 0xffff880233f7b3c0        0t0 187400250 socket
java      14216   imhtp   36u     sock                0,6        0t0 106323858 can't identify protocol
java      14216   imhtp  130u     unix 0xffff880235e539c0        0t0 106324660 socket
java      16615   imhtp   36u     sock                0,6        0t0  34219267 can't identify protocol
java      16615   imhtp  145u     unix 0xffff88023924e080        0t0  34219975 socket
java      16870   imhtp   38u     sock                0,6        0t0 198357475 can't identify protocol
java      16870   imhtp   56u     unix 0xffff8802392c7c80        0t0 198358088 socket
java      16870   imhtp   65u     unix 0xffff8802392c7c80        0t0 198358088 socket
java      16870   imhtp   66u     unix 0xffff8802392c7c80        0t0 198358088 socket
java      17021   imhtp   36u     sock                0,6        0t0 213679956 can't identify protocol
java      19405   imhtp   36u     sock                0,6        0t0 177291674 can't identify protocol
java      22165   imhtp   36u     sock                0,6        0t0 271476844 can't identify protocol
java      22165   imhtp   40u     unix 0xffff88004aea26c0        0t0 271477354 socket
java      30738   imhtp   36u     sock                0,6        0t0 269955359 can't identify protocol
java      30738   imhtp   41u     sock                0,6        0t0 271839241 can't identify protocol
java      30738   imhtp   42u     sock                0,6        0t0 271839255 can't identify protocol
java      30738   imhtp   43u     sock                0,6        0t0 271839242 can't identify protocol
java      30738   imhtp   44u     sock                0,6        0t0 271839253 can't identify protocol
java      30738   imhtp   46u     sock                0,6        0t0 271839257 can't identify protocol
java      30738   imhtp   47u     sock                0,6        0t0 271839256 can't identify protocol
java      30738   imhtp   51u     sock                0,6        0t0 271839258 can't identify protocol
java      30738   imhtp   53u     sock                0,6        0t0 271839259 can't identify protocol
java      30738   imhtp   60u     sock                0,6        0t0 271839248 can't identify protocol
java      30738   imhtp   61u     sock                0,6        0t0 271839260 can't identify protocol
java      30738   imhtp  124u     unix 0xffff88004aea2cc0        0t0 270139880 socket
java      31781   imhtp   36u     sock                0,6        0t0 207958180 can't identify protocol
```
可以发现，Name列的值为“an’t identify protocol”，socket找不到打开的文件。


```java
[2017-03-02 22:19:18 ERROR]-[TcpServer - LogicWork0] XiaoZCallback.getXiaoZView(292) | openid: oBBmBjh7sOne_Ay19Lkym1-L307s
java.net.MalformedURLException: Illegal character in URL
    at sun.net.www.http.HttpClient.getURLFile(HttpClient.java:588)
    at sun.net.www.protocol.http.HttpURLConnection.writeRequests(HttpURLConnection.java:464)
    at sun.net.www.protocol.http.HttpURLConnection.getInputStream(HttpURLConnection.java:1216)
    at cn.network.http.HttpTool.httpContentGet(HttpTool.java:269)
    at cn.network.http.HttpTool.httpContentGet(HttpTool.java:252)
    at com.chinanet.callback.XiaoZCallback.getXiaoZView(XiaoZCallback.java:213)
    at com.chinanet.callback.XiaoZCallback.getXiaoZhiContent(XiaoZCallback.java:65)
    at com.chinanet.mng.XiaoIMng.process(XiaoIMng.java:93)


[2017-03-02 22:20:03 ERROR]-[TcpServer - LogicWork0] XiaoZCallback.getXiaoZView(292) | openid: oBBmBjueL7mCFFyDdZkNchqqgvX4
net.sf.json.JSONException: Unterminated string at character 736 of {"startTime":"2017-03-02 22:15:00","errCode":0,"answers":[{"menuItemsIDs":["CUSTOMER_KBDATAID_1565330","CUSTOMER_KBDATAID_1473994","CUSTOMER_KBDATAID_1567944"],"menuItemsAnswers":["您好！感谢您使用中国电信业务。中国电信天翼手机、固定电话和小灵通用户开通七彩铃音业务后，可以为呼叫自己电话的其他主叫用户预先设置时尚歌曲、幽默言语、佳节问候或企业欢迎语等个性化回铃音，不再是单调的“嘟…嘟…”提示音，为拨打电话的人带来独特的音乐和语音感受。功能包括：基础功能、赠送功能、铃音盒功能、铃音群组功能、指定铃音功能、111120自听铃音功能、七彩铃音“1#复制”功能、短信一键订购功能。适用于中国电信天翼手机、固定电话和小灵通用户。月租费：七彩铃音5元/月，铃音盒0元/月-5元/月，111120自听铃音0元/月，七彩铃音“1#复制”0元/月。可以通过短信、爱音乐官方网站、118100/118101电话自助办理或通过10000号、营业厅办理。七彩铃音短信指令：1、开通方式：发送“6042”或“KTQCLY”至10001开通七彩铃音。2、取消方式：发送“6043”或“QXQCLY”至10001取消七彩铃音。铃音盒短信指令：1、订购铃音盒：编辑“M”发送至118100或10659100。2、退订铃音盒：编辑“YYH”发送至118100或10659100退。了解更多点击：http://gd.zhidao.189.cn/ckb/knowledge_detail/jy/20130326/29691
"],"menuItems":["想知道你的彩铃业务是什么样的,给我介绍一下呗.","作为一个联通用户,我要办理彩铃,你们有什么好介绍的吗?","我是老联通用户,我要办理彩铃,你们有什么好介绍的吗?"],"answerPat":"您好,为您找到以下信息：<@菜单选项>谢谢！","answerType":"FAQ菜单","channel":"WX","userNumberType":"","userNumber":"waJjIHP2qVkf_Z_h","userID":"QricyzdVQtVF_z_Hjw8ovjEYcrF_z_HsGeu0Zgjw5PLG/0cj0D4f_Z_h","date":"2017-03-02"}],"answersCount":1,"endTime":"2017-03-02 22:15:01"}

    at net.sf.json.util.JSONTokener.syntaxError(JSONTokener.java:512)
    at net.sf.json.util.JSONTokener.nextString(JSONTokener.java:244)
    at net.sf.json.util.JSONTokener.nextValue(JSONTokener.java:352)
    at net.sf.json.JSONArray._fromJSONTokener(JSONArray.java:1158)
    at net.sf.json.JSONArray.fromObject(JSONArray.java:147)
    at net.sf.json.util.JSONTokener.nextValue(JSONTokener.java:358)
    at net.sf.json.JSONObject._fromJSONTokener(JSONObject.java:1128)
    at net.sf.json.JSONObject.fromObject(JSONObject.java:179)
    at net.sf.json.util.JSONTokener.nextValue(JSONTokener.java:355)
    at net.sf.json.JSONArray._fromJSONTokener(JSONArray.java:1158)
    at net.sf.json.JSONArray.fromObject(JSONArray.java:147)
    at net.sf.json.util.JSONTokener.nextValue(JSONTokener.java:358)
    at net.sf.json.JSONObject._fromJSONTokener(JSONObject.java:1128)
    at net.sf.json.JSONObject._fromString(JSONObject.java:1317)
    at net.sf.json.JSONObject.fromObject(JSONObject.java:185)
    at net.sf.json.JSONObject.fromObject(JSONObject.java:154)
    at com.chinanet.callback.XiaoZCallback.getXiaoZView(XiaoZCallback.java:216)
    at com.chinanet.callback.XiaoZCallback.getXiaoZhiContent(XiaoZCallback.java:67)
    at com.chinanet.mng.XiaoIMng.process(XiaoIMng.java:93)
    at com.chinanet.handle.ServerHandler.defaultProcess(ServerHandler.java:700)
    at com.chinanet.handle.ServerHandler.internalHandle(ServerHandler.java:322)
    at com.chinanet.handle.ServerHandler.handle(ServerHandler.java:139)
```
