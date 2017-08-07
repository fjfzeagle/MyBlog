---
title: 使用第三方MQTT Client连接云巴的MQTT Broker
date: 2017-08-07 13:27:02
tags: [MQTT,云巴]
---

## 准备工作

- 下载arduino_sdk：https://github.com/yunba/yunba-arduino-sdk
- 下载第三方MQTT Client：http://www.jensd.de/apps/mqttfx/1.5.0/
- 云巴应用管理中的APPKey：563c4afef085fc471efdf803，需替换成自己应用的appKey
- TCP调试工具：网上随便下载一个

<!-- more -->

## 获取可用的MQTT Broker

1. 打开arduino_sdk下的YunbaWIFI.ino，找到get_host_v2函数源码​

```c
bool get_host_v2(const char *appkey, char *url) {
  uint8_t buf[256];
  bool rc = false;
  LWiFiClient net_client;
  while (0 == net_client.connect("tick-t.yunba.io", 9977)) {
    Serial.println("Re-connect to tick");
    delay(1000);
  }
  delay(100);

  String data = "{\"a\":\"" + String(appkey) + "\",\"n\":\"1\",\"v\":\"v1.0\",\"o\":\"1\"}";
  int json_len = data.length();
  int len;

  buf[0] = 1;
  buf[1] = (uint8_t)((json_len >> 8) & 0xff);
  buf[2] = (uint8_t)(json_len & 0xff);
  len = 3 + json_len;
  memcpy(buf + 3, data.c_str(), json_len);
  net_client.flush();
  net_client.write(buf, len);

  while (!net_client.available()) {
    Serial.println(json_len, len);
    Serial.println(len);
    Serial.println("wailt data");
    delay(100);
  }

  memset(buf, 0, 256);
  int v = net_client.read(buf, 256);
  if (v > 0) {
    len = (uint16_t)(((uint8_t)buf[1] << 8) | (uint8_t)buf[2]);
    if (len == strlen((char *)(buf + 3))) {
      Serial.println((char *)(&buf[3]));
      JsonObject& root = jsonBuffer.parseObject((char *)&buf[3]);
      if (!root.success()) {
        Serial.println("parseObject() failed");
        goto exit;
      }
      strcpy(url, root["c"]);
      Serial.println(url);
      rc = true;
    }
  }
exit:
  net_client.stop();
  return rc;
}
```

1. 分析以上的代码，生成自己appKey对应的请求串

   ```json
   {"a":"563c4afef085fc471efdf803","n":"1","v":"v1.0","o":"1"}
   ```

2. 把第2点生成的请求串转成16进制

   ```
   7B2261223A22353633633461666566303835666334373165666466383033222C226E223A2231222C2276223A2276312E30222C226F223A2231227D
   ```

3. 生成最终请求串,往第3点生成的请求串前面加三个字节01 00 3B ,其中3B是第2点请求串的长度59

   ```
   01003B7B2261223A22353633633461666566303835666334373165666466383033222C226E223A2231222C2276223A2276312E30222C226F223A2231227D
   ```

4. 使用TCP调试工具,连接tick-t.yunba.io:9977,以16进制模式发送第4点生成的内容，服务器返回的内容去掉前3字节，取得的内容如下:

   ```json
   {"c":"tcp://182.92.154.3:1883"}
   ```

5. 从第5点返回的内容，得到Broker：182.92.154.3:1883

## 获取client_id、username、password

1. 打开arduino_sdk下的YunbaWIFI.ino，找到setup_with_appkey_and_devid函数源码

   ```c
   bool setup_with_appkey_and_devid(const char* appkey, const char *deviceid) {
     uint8_t buf[256];
     bool rc = false;
     LWiFiClient net_client;

     if (appkey == NULL) return false;

     while (0 == net_client.connect("reg-t.yunba.io", 9944)) {
       Serial.println("Re-connect to tick");
       delay(1000);
     }
     delay(100);

     String data;
     if (deviceid == NULL)
       data = "{\"a\": \"" + String(appkey) + "\", \"p\":4}";
     else
       data = "{\"a\": \"" + String(appkey) + "\", \"p\":4, \"d\": \"" + String(deviceid) + "\"}";
     int json_len = data.length();
     int len;

     buf[0] = 1;
     buf[1] = (uint8_t)((json_len >> 8) & 0xff);
     buf[2] = (uint8_t)(json_len & 0xff);
     len = 3 + json_len;
     memcpy(buf + 3, data.c_str(), json_len);
     net_client.flush();
     net_client.write(buf, len);

     while (!net_client.available()) {
       Serial.println(json_len, len);
       Serial.println(len);
       Serial.println(data);
       Serial.println("wailt data");
       delay(100);
     }

     memset(buf, 0, 256);
     int v = net_client.read(buf, 256);
     if (v > 0) {
       len = (uint16_t)(((uint8_t)buf[1] << 8) | (uint8_t)buf[2]);
       if (len == strlen((char *)(buf + 3))) {
         Serial.println((char *)(&buf[3]));
         JsonObject& root = jsonBuffer.parseObject((char *)&buf[3]);
         if (!root.success()) {
           Serial.println("parseObject() failed");
           net_client.stop();
           return false;
         }
         strcpy(username, root["u"]);
         strcpy(password, root["p"]);
         strcpy(client_id, root["c"]);
         Serial.println(username);
         rc = true;
       }
     }

     net_client.stop();
     return rc;
   }
   ```

2. 分析以上的代码，生成自己appKey对应的请求串

   ```json
   {"a": "563c4afef085fc471efdf803", "p":4, "d": "deviceid"}
   ```

3. 把第2点生成的请求串转成16进制

   ```
   7B2261223A2022353633633461666566303835666334373165666466383033222C202270223A342C202264223A20226465766963656964227D
   ```

4. 生成最终请求串,往第3点生成的请求串前面加三个字节01 00 39 ,其中39是第2点请求串的长度57

   ```
   0100397B2261223A2022353633633461666566303835666334373165666466383033222C202270223A342C202264223A20226465766963656964227D
   ```

5. 使用TCP调试工具,连接reg-t.yunba.io:9944,以16进制模式发送第4点生成的内容，服务器返回的内容去掉前3字节，取得的内容如下:

   ```json
   {
     "u": "3012425797112621824",
     "p": "47cd938dfc1c5",
     "c": "0000002823-000000057954",
     "d": "deviceid"
   }
   ```

6. 从第5点返回的内容，解析得出三个参数

   ```
   username: 3012425797112621824
   password: 47cd938dfc1c5
   client_id: 0000002823-000000057954
   ```

## 使用第三方mqtt client工具mqttfx连接云巴的Broker

使用前面获取到的broker 与clientid,username,password连接云巴的broker连接成功，可以进行正常的订阅与发布，遗憾的是云巴的MQTT是基于3.1协议的，无法用delphi的三方控件TMS MQTT进行连接(tms mqtt 是基于3.1.1协议的)