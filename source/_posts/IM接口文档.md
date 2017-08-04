---
title: IM接口文档
date: 2017-08-03 11:00:24
tags: [IM]
---

## 通用

- 请求URL： http://IP:20001/api/im
- 签名：      md5(timestamp + md5(school_id + account_id + 盐巴) )
- 盐巴为：  Tk07EiJfCcrL
- 请求时，头部需带上：Content-Type:application/json
- 每次请求都需传入唯一的timestamp(小于32位的时间戮或随机数,保证每次不一样即可)
- code:为0表示成功，其它值表示错误，用msg显示
- 请求json中的expanded参数：传0时，返回的json是经过压缩的,传1时，适合用在ajax请求中，返回的json未经压缩，建议传0，此参数可以不传，后台默认压缩json
- json中必须传的参数：cmd、school_id、account_id
- 请求json中的值，如果要求传的是数值，必须传数值类型，不能加上双引号以字符传输
- http返回200才表示结果正常响应，非200的情况下，响应json串中msg表示错误的信息 400:服务器不理解请求的语法, 401:签名错误
- if http_status=200 then {正常解析数据}  else {showErrorMsg(json.s['msg']);} 
- 后台使用account_id值来判断每次传输的timestamp是否唯一
- 查看此文档建议安装Typora，下载地址: http://typora.io/ 

<!-- more -->

## 获取用户聊天群列表

- 请求

  ```json
  {
  	"cmd": "getChatRoomList",
  	"school_id": 1,
  	"account_id": 4541,
  	"timestamp": "1234560789101X",
  	"sign": "d383df6ff01e49c86c4ac976ec42fbff",
  	"expanded": 1
  }
  ```

- 响应

  ```json
  {"fieldCount":6,"values":["im_group_id","relation_id","group_name","group_type","easemob_group_id","isOwner",2,2,"德育处","teacher","",0,64,20000001,"高一年段任课老师","teacher","",0,68,30000002,"数学教研组","teacher","",0,83,40001002,"高一数学集备组","teacher","",0,145,50000000,"全校教职工","teacher","",0,332,60567002,"高一(10)班数学","class_course","",1,376,60571002,"高一(14)班数学","class_course","",1,747,304,"高一(10)班师生群","teacherAndStudent","",0,751,308,"高一(14)班师生群","teacherAndStudent","",0,873,304,"高一(10)班家校群","teacherAndGuardian","",0,877,308,"高一(14)班家校群","teacherAndGuardian","",0],"rowCount":11}
  ```

- 返回字段说明

  1. im_group_id：中心库中的组ID
  2. group_name：组名
  3. group_type：组类型，值范围：'teacherAndGuardian':家校群,'teacherAndStudent':师生群,'guardian':家长群,'teacher':老师群,'student':学生群,'other':其他群,'class_course':学科群,'corporation':社团群
  4. easemob_group_id：环信系统中的组ID,**如果easemob_group_id值为空，表示群还未创建(未激活)**
  5. isOwner：**是否是群主,如果为1，表示有权限激活该群**
  6. **如果easemob_group_id值为空并且isOwner=1,则需显示激活按钮，让用户点击按钮进行群激活**
  7. **所有群在用户点击群组时判断群ID如果为空则发起激活**

## 激活聊天群

- 请求

  ```json
  {"cmd":"activeGroup","school_id":1,"account_id":3419,"im_group_id":1496,"im_user_id":151345,"timestamp":"1234560789101X","sign":"1f7be318563a1ad7b9f4eaada9adfd7a"}	
  //im_group_id:待激活的组ID
  //im_user_id:激活用户用户ID(等价环信ID)
  ```

- 响应

  ```json
  {"code":0,"msg":"sucess","easemob_group_id":"19448358567937"}
  //因后台要调用环信接口，故此接口的响应时间有可能在3秒以上,app建议加个进度条，否则会使界面无响应
  ```

## 获取聊天群用户列表

- 请求

  ```json
  {"cmd":"getChatRoomUserList","school_id":6,"account_id":3419,"im_group_id":710,"timestamp":"1234560789101X","sign":"1f7be318563a1ad7b9f4eaada9adfd7a","expanded":0}
  ```

- 响应

  ```json
  {"fieldCount":6,"values":["im_user_id","account_id","real_name","user_type","remark","isOwner",162077,3179,"沈贞辉","teacher","段长",0,162317,3419,"竹子","teacher","初一(1)班",0,162157,3259,"黄良材","teacher","初一(2)班",0,162239,3341,"钟电昇","teacher","初一(3)班",0,162320,3422,"陈科锦","teacher","初一(4)班",0],"rowCount":5}
  ```

- 返回字段说明

  1. im_user_id：中心库中的用户ID
  2. real_name：用户姓名
  3. user_type：用户类型，值范围：'guardian':家长,'teacher':老师,'student':学生
  4. remark：备注信息
  5. isOwner：是否是群主

## 获取老师的聊天ID(环信ID)

- 请求

```json
{"cmd":"getTeacherIMUserID","school_id":6,"account_id":3419,"teacher_id":290,"timestamp":"1234560789101X","sign":"1f7be318563a1ad7b9f4eaada9adfd7a"}
```

- 响应

```json
{"fieldCount":1,"values":["im_user_id",162317],"rowCount":1}
```

- 返回的字段说明
  1. im_user_id：聊天ID
  2. 如果响应JSON中rowCount为0，表示获取失败，可能原因：用户还未上传到环信系统中、老师ID非法、数据还未同步到中心库

## 获取学生的聊天ID(环信ID)

- 请求

  ```json
  {"cmd":"getStudentIMUserID","school_id":6,"account_id":3419,"student_id":290,"timestamp":"1234560789101X","sign":"1f7be318563a1ad7b9f4eaada9adfd7a"}
  ```

- 响应

  ```json
  {"fieldCount":1,"values":["im_user_id",159189],"rowCount":1}
  ```

- 返回的字段说明

  1. im_user_id：聊天ID
  2. 如果响应JSON中rowCount为0，表示获取失败，可能原因：用户还未上传到环信系统中、学生ID非法、数据还未同步到中心库

## 获取学生家长的聊天ID(环信ID)

- 请求

  ```json
  {"cmd":"getParentIMUserID","school_id":6,"account_id":3419,"student_id":1,"timestamp":"1234560789101X","sign":"1f7be318563a1ad7b9f4eaada9adfd7a"}
  ```

- 响应

  ```json
  {"fieldCount":1,"values":["im_user_id",254671,254672,254673,254677,254678,296637,296638,341757],"rowCount":8}
  ```

- 返回字段说明

  1. im_user_id：聊天ID
  2. 如果响应JSON中rowCount为0，表示获取失败，可能原因：用户还未上传到环信系统中、学生ID非法、数据还未同步到中心库
  3. 一个学生有可能有多个家长，需注意

## 获取用户的聊天ID(学生、老师、家长)

- 请求

  ```json
  {"cmd":"getIMUserID","school_id":6,"account_id":3419,"account_id_list":[3419],"timestamp":"1234560789101X","sign":"1f7be318563a1ad7b9f4eaada9adfd7a"}
  //account_id_list:数组，可以传多个account_id
  ```

- 响应

  ```json
  {"fieldCount":2,"values":["account_id","im_user_id",3419,162317],"rowCount":1}
  ```

- 返回字段说明

  1. im_user_id：聊天ID
  2. 如果响应JSON中rowCount为0，表示获取失败，可能原因：用户还未上传到环信系统中、account_id非法、数据还未同步到中心库

## 获取学生或老师的照片

- 签名使用的字段：school_id,account_id,timestamp,sign
- 如果有指定teacher_id则是获取老师照片
- 如果有指定student_id则是获取学生照片


- 通过account_id获取照片(学生或老师)

  ```html
  http://IP:20001/api/userpic?school_id=6&account_id=3640&timestamp=1234560789101X&sign=206de30cd2680ce61aece8ab34ff21e0
  ```

- 通过teacher_id获取老师照片

  ```html
  http://IP:20001/api/userpic?school_id=6&account_id=3640&teacher_id=290&timestamp=1234560789101X&sign=206de30cd2680ce61aece8ab34ff21e0
  <!-- account_id是用来签名使用的，此时可以传登录用户对应的account_id或者随便传一个也行 -->
  ```

- 通过student_id获取学生照片

  ```html
  http://IP:20001/api/userpic?school_id=6&account_id=3640&student_id=290&timestamp=1234560789101X&sign=206de30cd2680ce61aece8ab34ff21e0
  <!-- account_id是用来签名使用的，此时可以传登录用户对应的account_id或者随便传一个也行 -->
  ```

## IM集备组聊天记录获取与发送

### 聊天记录列表获取

- 请求

  ```json
  {"cmd":"getJiBeiChatLog","school_id":6,"account_id":3419,
   "grade_no":1,"course_id":2,"plan_id":1,"begin_time":"2017-06-09 00:00:00","end_time":"2017-06-09 23:59:59",
   "timestamp":"1234560789101X","sign":"1f7be318563a1ad7b9f4eaada9adfd7a","expanded":0}
  ```

- 响应

  ```json
  {"fieldCount":13,"values":["im_message_id","sender_id","account_id","user_id_fk","real_name","easemob_type","content","easemob_filename","easemob_length","easemob_file_length","easemob_width","easemob_height","msg_time",2006,162317,3419,290,"竹子","TXT","文本",null,null,null,null,null,"2017-06-09T14:33:26",2007,162317,3419,290,"竹子","VOICE",null,"1496990117711.amr",null,null,null,null,"2017-06-09T14:35:32",2008,162317,3419,290,"竹子","IMAGE",null,"1496990136165.jpeg",null,null,null,null,"2017-06-09T14:35:50",2009,162317,3419,290,"竹子","FILE",null,"1496990168149.jpg",null,null,null,null,"2017-06-09T14:36:22"],"rowCount":4}
  ```

- 手机端聊天界面
  ![集备组聊天界面](/upload_image/集备组聊天界面.png)

### 下载文件(公司服务器先从环信服务器下载然后再回传)

- 请求

  ```html
  http://
  :20001/api/easemob/file?school_id=6&account_id=3640&im_message_id=2007&timestamp=1234560789101X&sign=206de30cd2680ce61aece8ab34ff21e0
  <!-- account_id是用来签名使用的，此时可以传登录用户对应的account_id或者随便传一个也行 -->
  ```



### 发送文本消息

- 请求

  ```json
  {"cmd":"sendMsgToJiBeiChat","school_id":6,"account_id":3940,
   "grade_no":1,"course_id":2,"msg":{"type":"txt","msg":"消息文本内容"},
   "ext":{"plan_id":1,"plan_name":"测试"},
   "timestamp":"1234560789101X","sign":"d14990fdd2d35fa8801d3f9eb81032b3"}
  //account_id:消息发送者
  //[school_id_id,grade_no,course_id] 指定集备组
  //ext:扩展字段(沿用环信接口),可以自行添加其他内容
  //plan_id：集备中的研究课题ID
  ```

- 响应

  ```json
  "code":0,"im_message_id":2013,"msg":"success"}
  ```

## 通过im_group_id获取relation_id

- 请求

  ```json
  {"cmd":"getGroupRelationId","school_id":6,"account_id":3419,"im_group_id":2433,"timestamp":"1234560789101X","sign":"1f7be318563a1ad7b9f4eaada9adfd7a"}
  ```

- 响应

  ```json
  {"relation_id":58}
  //如果值为0表示没有找到
  ```



## 创建自定义聊天群

- 请求

  ```json
  {"cmd":"createCustomGroup","school_id":1,"account_id":3419,
   "groupname":"群组名称",
   "owner_account_id":0,
   "owner_teacher_id":0,
   "owner_student_id":0,
   "members":{
     "account_id":[1,2,3]
     "teacher_id":[1,2,3]
     "student_id":[1,2,3]
   },
   "timestamp":"1234560789101X","sign":"1f7be318563a1ad7b9f4eaada9adfd7a"
  }	
  //群的创建者:owner_account_id,owner_teacher_id,owner_student_id,三个值只能传一个
  //members:可以同时传account_id、teacher_id、student_id键值
  ```

- 响应

  ```json

  ```

## 修改自定义聊天群

- 请求

  ```json
  {"cmd":"modifyCustomGroup","school_id":1,"account_id":3419,
   "im_group_id":0
   "groupname":"群组名称",
   "addmembers":{
     "account_id":[1,2,3]
     "teacher_id":[1,2,3]
     "student_id":[1,2,3]
   },
    "deletemembers":{
     "account_id":[1,2,3]
     "teacher_id":[1,2,3]
     "student_id":[1,2,3]
   },
   "timestamp":"1234560789101X","sign":"1f7be318563a1ad7b9f4eaada9adfd7a"
  }
  //im_group_id:要修改的群ID
  //addmembers:新增群员
  //deletemembers:删除群员
  ```

- 响应

  ```json

  ```