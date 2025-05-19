#### JavaScript操作

[参考资料](https://blog.csdn.net/qq_42842786/article/details/107641415)

!!! warning "JavaScript过滤器不应处理大批量json数据,否则导致引擎GC Pause"

!!! warning "为避免平台出现乱码问题,通讯点及过滤器编码统一设置为UTF-8"

##### 常用

```javascript
//js解析消息体节点请尽量使用input[0].getField(),不使用next.getField()
//next一般用于设置属性,设置字段;防止内部死循环;
             
// 动态路由目标动态设置
next.setProperty("router:Destination", "@com.alsoapp.esb.rhapsody.route."+_json.event.eventCode);
// 或者使用过滤器Property Population设置属性
next.setProperty("rhapsody:consumerOperation", "对方web service接口方法名");
// 禁用消息超时
next.setProperty("rhapsody:TimeToLive", null)

//移除属性
next.setProperty("data", null);

//生成uuid
generateUuid()
```

!!! success "Rhapsody解析大json文本推荐用法:<br>1.使用javascript封装成xml;{==var data = "&lt;xml&gt;<![CDATA["+input[0].text+"]]>&lt;/xml&gt;";==}<br>2.使用freemarker解析json;需配置:{==Escape Characters Mode设为None;==}"

##### XML操作

更多xml操请参考: [点击此链接](https://www.alsoapp.com/docs-rhapsody-7.1/en/xml-object.html)

!!! success "温馨提示:<br>1.Rhapsody 7.1 JavaScript使用较新脚本引擎,请尽快升级至Rhapsody7以上"

###### xml添加子节点

```javascript
// 此处不能从根节点开始遍历
//去除不是出院的就诊信息
var data = new XML(input[0].xml);
var tmp_data = new XML(<AdmInfoList></AdmInfoList>);
for(var i = 0; i < data.AdmInfo.length(); i++){
	if(data.AdmInfo[i].AdmStatusCode == 'D'){
		tmp_data.appendChild(data.AdmInfo[i]);
		}
	}
next.setText(tmp_data, 'UTF-8');
```

###### 构造xml节点

```javascript
// 构造xml数据
var data = <xml>
	<code>200</code>
	<msg>消息接收成功!</msg>
	<gmt_rcv>{new Date(parseInt(next.getProperty('InputTime')))}</gmt_rcv>
	<gmt_created>{new Date().toString()}</gmt_created>
</xml>;
next.setText(data, 'UTF-8');
```

###### 获取指定子节点内容

```javascript
// _xml内容为<message><patient></ENTER_DATE_TIME></patient></message>
for(var i = 0 ; i < (pat_json.PATIENT).length ;i++){
	var DATA_ELEMENT_EN_NAME = pat_json.PATIENT[i].DATA_ELEMENT_EN_NAME;	
	var DATA_ELEMENT_VALUE = "";
	
	if(DATA_ELEMENT_EN_NAME == "ENTER_DATE_TIME" || DATA_ELEMENT_EN_NAME == "MODIFY_DATE_TIME"){
	if(_xml.patient.child(DATA_ELEMENT_EN_NAME) != null && _xml.patient.child(DATA_ELEMENT_EN_NAME) != ""){
		var d = _xml.patient.child(DATA_ELEMENT_EN_NAME);
        // 获取xml节点值的长度,先用获取节点信息,转字符串,取长度
		if((d.toString()).length == 15){
			DATA_ELEMENT_VALUE = _xml.patient.child(DATA_ELEMENT_EN_NAME).toString();
					}else{
DATA_ELEMENT_VALUE = library.DateStrToGBTime(_xml.patient.child(DATA_ELEMENT_EN_NAME));
						}
			}else {
				DATA_ELEMENT_VALUE="";
			}
		}else {
			DATA_ELEMENT_VALUE = _xml.patient.child(DATA_ELEMENT_EN_NAME).toString();
		}
		pat_json.PATIENT[i].DATA_ELEMENT_VALUE = DATA_ELEMENT_VALUE;
	}
```

###### XML节点动态获取节点信息

```javascript
// 根据xml节点信息获取节点信息和值
// 暂时仅适用于简单xml
// 1.获取节点
const items = data.getElements('//data');
let result = [];
// 2.遍历
item.forEach(function(item, index){
    let nodes = item.getElments('*');
    let node = {};
    nodes.forEach(function(item){
        node[item.domNode.toString().split(':')[0].replace('[', '')] = item.text;
    });
    result.push(node);
}
```

###### XML节点循环解析

```javascript
// 适用于hl7 v3
// 循环获取诊断信息,以检验申请单为例
let data_diags = [];
// 1.获取需要循环的节点
const diags = content.getElements('//pertinentInformation1');
// 2.遍历诊断数组;
diags.forEach(function(diag){
    temp_diag = {};
    // 注意xpath路径写法,前面不使用//,并且父节点为pertinentInformation1
    temp_diag['diag_code'] = diag.getValue('observationDx/value/@code');
    data_diags.push(temp_diag);
});
```

!!! success "温馨提示:<br>Rhapsody 7.1 已测试通过<br>forEach用法:<br>arr.forEach(function(currentValue, index, arr){})"

###### 其它

```javascript
// 遍历xml节点示例;注意操作xml,路径少一层根节点
var data = new XML(input[0].xml);
data.DML_TYPE.@id; //取xml节点的属性内容
data.DML_TYPE = 'D'; // 也适用于json消息字段设置

// or 非xml处理时,须考虑根节点
next.setField('/CACHE/DML_TYPE', 'D');  //也适用于HL7 v2版本消息字段设置

// 若不指定item索引值,则返回所以item的节点用,分割
var patient_id = next.getField('//controlActProcess/subject/procedureRequest/componentOf1/encounter/subject/patient/id/item[@root="2.16.156.10011.2.5.1.4"]/@extension')
//设置xpath节点数据
next.setField('//observationRequest/componentOf1/encounter/id/item[@root="2.16.156.10011.2.5.1.8"]/@extension', next.getProperty('visit_times'));   

// 获取xml节点重复个数
// eg-1;注意少一层根节点;返回为浮点数
input[0].xml.OrderRowIDList.length();
// eg-2;/Response/OrderRowIDList为xml路径;推荐
input[0].getRepeatCount('/Response/OrderRowIDList');
```

##### JSON操作

```javascript
// 删除json节点
//删除门诊就诊节点信息
delete data.queryAck.ENCOUNTER_OUTPATIENTS;

// forEach用法:
// arr.forEach(function(currentValue, index, arr), thisValue)
data.message.PATIENT.forEach(function(item) {
if(item['DATA_ELEMENT_EN_NAME'].indexOf('DATE') > -1 || item['DATA_ELEMENT_EN_NAME'].indexOf('TIME') > -1 || item['DATA_ELEMENT_EN_NAME'].indexOf('GMT') > -1){
		content[item['DATA_ELEMENT_EN_NAME']] = lib.get_datetime_format(lib.str2date(item.DATA_ELEMENT_VALUE.replace('T', '').valueOf()), "yyyy-MM-dd HH:mm:ss");
		}else{
		content[item['DATA_ELEMENT_EN_NAME']] = item.DATA_ELEMENT_VALUE;
        }
    });

// 方法二
for(let item of PATIENT){
	next.setProperty(PATIENT[item].DATA_ELEMENT_EN_NAME, PATIENT[item].DATA_ELEMENT_VALUE);
}

// 方式三
for(var i=0; i < data.message.PATHOLOGY_APPLY.length; i++){
	log.info(item in data.message.PATHOLOGY_APPLY[i]);
}

//json节点判断存在及过滤
if(data.hasOwnProperty('status_code') && data.status_code != 200){
	throw "返回内容异常";
	}

// 动态json遍历key-value
Object.entries(data).forEach(function([key, value]){
    next.setProperty(key, value);
});
```
##### 正则表达式

```javascript
// 更多的正则表达式匹配示例请参考regexp.md文件
const tmp_data = input[0].text;
const reg = new RegExp("http://17.{1,128}.pdf");
//如果匹配到则执行一下语句
if(reg.test(input[0].text)){
	let tmp_url = reg.exec(input[0].text);
    //或使用str.match(reg)
	next.setProperty("tmp_url", tmp_url);
	let tmp_data = input[0].text.replace(reg, 'url因为有特殊字符而被替换');
	}

// 除去换行符号和空格;不适合>50k的文本处理                                                     
const data = input[0].text.replace(/\r\n/g,"").replace(/\s+/g,"");
next.setText(data, "utf-8");
// eg-1: 全部替换,如果不加/g,则替换第一个
replace(/-/g, '')
// eg-2: 替换\"成空字符串
replace(/\\"/g, '')

// xml特殊符号去除
// 存数据库之前
value.replace(/\&/g,"&amp;").
replace(/\"/g,"&quot;").
replace(/\'/g,"&#39;").
replace(/\</g,"&lt;").
replace(/\>/g,"&gt;")

#json.parse解析之前
var data = input[0].text.
replace(/</g, "&lt;").
replace(/>/g, "&gt;").
replace(/\'/g, "&#39;").
replace(/("")+/g, "&quot;").
replace(/\r/g, "\\r").
replace(/\n/g, "\\n").
replace(/\t/g, "\\t").
replace(/\\/g, "\\\\").
replace(/ /g, "&nbsp;");
next.text = '<xml><![CDATA['+data+']]></xml>';
```
##### 自定义索引

```javascript
/*
为输入属性编制索引
*/
next.indexInputProperty("hip_id");
/*
为输出属性编制索引
*/
next.indexOutputProperty("hip_id");
/*
删除指定属性的所有索引
*/
removeIndexingForProperty(string propertyName)
```

##### Base64加密解密操作

```javascript
//编码,第二个参数为是否多行
encodeBase64(input[0].text, false, "UTF-8")
//解码
decodeBase64(value[, encoding])
```

##### 瞬态对象

```javascript
// 设置状态
// eg1: 简单设置一个状态
transientState.setState('key', 'val');
// eg2: 设置一个带有过期时间的状态
transientState.setState('key', 'val', {
expires: '2023-05-23T19:00:00z'
});

// 获取状态
// eg1: 简单获取一个状态
next.setProperty('Prop' , transientState.getState('key'));

// 同时获取和设置状态
//eg1: 获取旧值后设置新值
var value = transientState.getAndSetState('key','val');
```

##### JSON简单转xml

```javascript
//通过getErrors()方法获取错误路由返回的异常数据
var body = input[0].getErrors()

//循环解析具体的异常信息
var errxml = new XML(<err></err>);
for(var i=0; i<body.length; i++){
    var errdata = "<err"+i+">"+body[i]+"</err+"+i+">";
    errxml.appendChild(errdata);
}
//将异常信息输出
next.setText("<message>"+errxml+"<message>")
```

##### xml转json

```python
// 动态xml节点转json
// 请参考:公共JavaScript库目录下->动态xml转json
```

##### 查找表操作

```javascript
// 查找表使用
if(tmp_dig_code != null && tmp_dig_code.length > 0){
    var data = lookup("mapping_icd_10",{"value_dh": tmp_dig_code});
    if(data != null){
    	var dig_code = data.value;
    	var dig_name = data.comment;
    }
}
// 查找表返回多条数据情况示例
var items = lookupAll("fans", [{columnName: "service_code", value: next.getProperty("action")}], {returnClolumns: ["firm_id"]});
items.forEach(function(item) {
  // 动态路由目标添加
  next.addPropertyValue("router:Destination", "@com.alsoapp.esb.rhapsody.route."+item.firm_id);
});

//获取环境变量
var env = lookup("configs", {"key": "env"});
next.setProperty("env", env.value);
```

##### 随机数生成

```javascript
//需要产生个人信息注册的消息厂商列表
var list_sender = ['H0001', 'H0002'];
var sender = list_sender[Math.floor(Math.random()*list_sender.length)];
```

##### 系统保留关键字取值

```javascript
// extends为系统保留关键字,请使用字典取值法进行取值
// eg-1
log.info(data.message.message['extends'])
```

##### 其他操作

```python
generateUuid() // 生成随机uuid
// 日期格式化转换
var data = dateChangeFormat('2012/01/02', 'YYYY/MM/dd', 'dd/MM/YYYY');
// 一个属性添加多个值
next.addPropertyValue('router:Destination', '@hl7v3')
// 设置消息体内容
next.text = ''
或: next.setText('', 'UTF-8');
getBodySize() //获取body消息体大小
getFieldAsList(string fieldPath) // 得到字段为重复的数组
getRepeatCount(string fieldPath) // 得到字段重复次数
getPropertyNames() // 得到所有属性内容
getErrors() // 得到错误信息
// 迭代消息属性遍历 
var properties = readOnlyMessage.getPropertyAsList("PropertyList");
// Loop through the properties, 
var iterator = properties.iterator();
while (iterator.hasNext()) {
    var oneProperty = iterator.next();
    log.info("One property is: " + oneProperty);
}
// 获取rhapsody全局变量
getVariable(variableName[, decryptVariable])
                                                                              
//将两个或多个字符的文本组合起来，返回一个新的字符串
if(body.lastIndexOf('}') == -1){
	body = body.concat('}');
}
//返回字符串中一个子串最后一处出现的索引（从右到左搜索），如果没有匹配项，返回 -1 
body.lastIndexOf('}')

// 判断列表是否存在某个指定元素
var list_append = ['E280401', 'E280402'];
list_append.indexOf(data.event.eventCode) >= 0 //返回true或false

// switch操作
switch(service_code){
   case "S0001": 
   case "S0002":
        break;
   default:
        break;
}
```

##### JavaScript计数器

!!! warning "仅限于数值类型"

```javascript
// 递增全局计数器以获取新的唯一ID
var myUniqueID = incCounter("MyUID", 1);
getCounter(counterName)
setCounter(counterName, int)
// 将新ID设置为消息的属性
next.setProperty("UniqueID", myUniqueID);
```

##### 异常处理

```javascript
try {
  //运行代码
} catch(err) {
   log.info(err)
   log.info(err.message);
   next.addError(err.message);
   // 抛出异常
   // throw '错误提示'
}finally{    //可选
	}
// 获取js抛出的异常
next.getErrors() //得到的为list,可取第一个错误提示
```
##### 公共JavaScript库

###### 使用方法

```javascript
// 引入js库名称
var lib = require("com_alsoapp_esb_utils");
// 使用该js库下的对应的方法
var result = lib.get_datetime_format(new Date(), "yyyy-MM-dd HH:mm:ss");
```

###### 解析URL参数

??? ":material-file: get_url_key.js"

    ```javascript
    function get_url_key(url) {
      // 解析url携带的参数信息
      var params = {};
      try{
        var urls = url.split("?");           
        var arr = urls[1].split("&");              
        for (var i = 0, l = arr.length; i < l; i++) {
          var a = arr[i].split("=");              
          params[a[0]] = a[1];                  
        }
                                               
      }catch(e){}
      return params;
    }
    
    // eg-1
    var tmp_url = next.getProperty('http:request-url');
    var data = lib.get_url_key(tmp_url);
    next.setProperty("patient_id", data["patient_id"]);
    next.setProperty("source_system_code", data["source_system_code"]);
    ```

###### 时间格式化

??? ":material-file: get_datetime_format.js"

    ```javascript
    function get_datetime_format(date, format) {
      var get_datetime_format = function (obj_date, fmt) {
      var dateTime = obj_date;
      var o = {
          "M+": dateTime.getMonth() + 1, //月份 
          "d+": dateTime.getDate(), //日 
          "H+": dateTime.getHours(), //小时 
          "m+": dateTime.getMinutes(), //分 
          "s+": dateTime.getSeconds(), //秒 
          "q+": Math.floor((dateTime.getMonth() + 3) / 3), //季度 
          "S": dateTime.getMilliseconds() //毫秒 
        };
      if (/(y+)/.test(fmt)) {
         fmt = fmt.replace(RegExp.$1, (dateTime.getFullYear() + "").substr(4 -  RegExp.$1.length));
      }
      for (var k in o){
          if (new RegExp("(" + k + ")").test(fmt)) 
          {
            fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
          }
      }
      return fmt;
    }
    
    return get_datetime_format(date, format);
    }
    
    // eg-1:
    log.info(lib.get_datetime_format(new Date(), "yyyy-MM-dd HH:mm:ss.S"));
    // eg-2:
    log.info(lib.get_datetime_format(new Date(parseInt(next.getProperty("InputTime"))), "yyyy-MM-dd HH:mm:ss.S"));
    // eg-3: 标准时间格式: ISO 8601
    log.info(lib.get_datetime_format(new Date(), "yyyy-MM-ddTHH:mm:ss+08:00"));
    ```

###### 字符串转时间对象

??? ":material-file: str2date.js"

    ```javascript
    function str2date(str) {
        /*
        字符串转时间对象
        str: 字符串,形如: 20221024010101
        */
        var y = Number(str.substring(0, 4));      
        var m = Number(str.substring(4, 6))-1;      
        var d = Number(str.substring(6, 8));      
        var h = Number(str.substring(8, 10));      
        var mm = Number(str.substring(10, 12)); 
        var ss = Number(str.substring(12, 14));      
        var time = new Date(y, m, d, h, mm, ss, 0);      
        return time;      
    }   
    // eg-1: 得到时间对象
    lib.str2date("20221024010101")
    // eg-2: 得到时间戳
    lib.str2date("20221024010101").valueOf() 
    ```

###### 动态xml转json

??? ":material-file: Dxml2json.js"

~~~javascript
```javascript

function Dxml2json(xml) {
    /*
    动态xml节点转json键值对
    Rhpaosdy 7.1版本以上不支持,请参考版本2
    形如: <message>
	<data>
    	<d1>data</d1>
    </data>
    </message>
    */
    var tmp_data = new XML(xml);
    var content = tmp_data.data.children();
    var data = {};
    for(var i = 0; i < content.length(); i++){
        // 也可使用content[j].toXMLString()得到在字符串节点,然后使用字符串截取即可;
        if(content[i].name() !== null){
        var str_data = content[i].name().toString()+':'+content[i].text()
        var arr = str_data.split(":");  
        data[arr[0].toLowerCase()] = arr[1];  
        }
        };
    return data;
}
// eg-1
lib.Dxml2json(input[0].xml)
```
~~~

###### 动态xml转json v2版

Rhapsody7  JavaScript （v.2） 筛选器基于 GraalVM JavaScript 脚本引擎，而 JavaScript （v.1） 筛选器基于 JavaScript 的 Mozilla Rhino 实现

??? ":material-file: Dxml2jsonV2.js"

```javascript
function Dxml2jsonV2(xml){
	/*
	xml: XML元素对象
	
	return：
		得到json对象
	
	*/
	let nodes = xml.getElements('*');
	let data = {};
	
	nodes.forEach(function(item, index){
		data[item.domNode.toString().replace('[', '').replace(': null]', '').toLowerCase()] = item.text;
		})
	return data;
}

// eg-1
lib.Dxml2jsonV2(input[0].xml)
```

!!! success "温馨提示:<br>1.Rhapsody 7.1 已测试通过"

###### 基于HMAC-SHA1的消息签名

??? ":material-file: hmac_sha1.js"

    ```javascript
    function hmac_sha1(key, message){
        /*
        key: 用于签名的密钥
        message: 签名的数据
        return: 返回基于key的生成的签名
        */
        var strInstance = new java.lang.String(key);
        var messageInstance = new java.lang.String(message);
        var keybytes = strInstance.getBytes("UTF-8");
        var messagebytes = messageInstance.getBytes("UTF-8");
        var secretKey = new javax.crypto.spec.SecretKeySpec(keybytes, "HmacSHA1");
        var mac = javax.crypto.Mac.getInstance("HmacSHA1");
            mac.init(secretKey);
        var hmacSha1_bytes = mac.doFinal(messagebytes);
    
         // 将结果转换为十六进制字符串
        var result = new java.lang.StringBuilder();
    
        for (var i = 0; i < hmacSha1_bytes.length; i++) {
            var hex = java.lang.Integer.toHexString(hmacSha1_bytes[i] & 0xFF);
                    if (hex.length() == 1) {
                        result.append('0');
                    }
                    result.append(hex);
          }
    
        return result.toString();
    }
    
    // eg-1: 
    log.ingo("test的签名内容为:" + lib.hmac_sha1("test", "test"))
    ```

###### MD5

??? ":material-file: md5_hexdigest.js"

    ```javascript
    # eg-1
    
    log.info(lib.md5_hexdigest("abc"))
    
    /*  
     * 实现一个简单的自我测试功能,检查是否正常执行  
     */  
    function test_md5_hexdigest()  
    {  
      return md5_hexdigest("abc") == "900150983cd24fb0d6963f7d28e17f72";  
    }  
    
    /*  
     * 接受一个字符串作为输入数据,将其编码为字节（bytes）并计算MD5哈希值,最后通过返回获取十六进制表示的哈希值;
     */ 
    function md5_hexdigest(string) {
      function md5_RotateLeft(lValue, iShiftBits) {
        return (lValue << iShiftBits) | (lValue >>> (32 - iShiftBits));
      }
      function md5_AddUnsigned(lX, lY) {
        var lX4, lY4, lX8, lY8, lResult;
        lX8 = (lX & 0x80000000);
        lY8 = (lY & 0x80000000);
        lX4 = (lX & 0x40000000);
        lY4 = (lY & 0x40000000);
        lResult = (lX & 0x3FFFFFFF) + (lY & 0x3FFFFFFF);
        if (lX4 & lY4) {
          return (lResult ^ 0x80000000 ^ lX8 ^ lY8);
        }
        if (lX4 | lY4) {
          if (lResult & 0x40000000) {
            return (lResult ^ 0xC0000000 ^ lX8 ^ lY8);
          } else {
            return (lResult ^ 0x40000000 ^ lX8 ^ lY8);
          }
        } else {
          return (lResult ^ lX8 ^ lY8);
        }
      }
      function md5_F(x, y, z) {
        return (x & y) | ((~x) & z);
      }
      function md5_G(x, y, z) {
        return (x & z) | (y & (~z));
      }
      function md5_H(x, y, z) {
        return (x ^ y ^ z);
      }
      function md5_I(x, y, z) {
        return (y ^ (x | (~z)));
      }
      function md5_FF(a, b, c, d, x, s, ac) {
        a = md5_AddUnsigned(a, md5_AddUnsigned(md5_AddUnsigned(md5_F(b, c, d), x), ac));
        return md5_AddUnsigned(md5_RotateLeft(a, s), b);
      };
      function md5_GG(a, b, c, d, x, s, ac) {
        a = md5_AddUnsigned(a, md5_AddUnsigned(md5_AddUnsigned(md5_G(b, c, d), x), ac));
        return md5_AddUnsigned(md5_RotateLeft(a, s), b);
      };
      function md5_HH(a, b, c, d, x, s, ac) {
        a = md5_AddUnsigned(a, md5_AddUnsigned(md5_AddUnsigned(md5_H(b, c, d), x), ac));
        return md5_AddUnsigned(md5_RotateLeft(a, s), b);
      };
      function md5_II(a, b, c, d, x, s, ac) {
        a = md5_AddUnsigned(a, md5_AddUnsigned(md5_AddUnsigned(md5_I(b, c, d), x), ac));
        return md5_AddUnsigned(md5_RotateLeft(a, s), b);
      };
      function md5_ConvertToWordArray(string) {
        var lWordCount;
        var lMessageLength = string.length;
        var lNumberOfWords_temp1 = lMessageLength + 8;
        var lNumberOfWords_temp2 = (lNumberOfWords_temp1 - (lNumberOfWords_temp1 % 64)) / 64;
        var lNumberOfWords = (lNumberOfWords_temp2 + 1) * 16;
        var lWordArray = Array(lNumberOfWords - 1);
        var lBytePosition = 0;
        var lByteCount = 0;
        while (lByteCount < lMessageLength) {
          lWordCount = (lByteCount - (lByteCount % 4)) / 4;
          lBytePosition = (lByteCount % 4) * 8;
          lWordArray[lWordCount] = (lWordArray[lWordCount] | (string.charCodeAt(lByteCount) << lBytePosition));
          lByteCount++;
        }
        lWordCount = (lByteCount - (lByteCount % 4)) / 4;
        lBytePosition = (lByteCount % 4) * 8;
        lWordArray[lWordCount] = lWordArray[lWordCount] | (0x80 << lBytePosition);
        lWordArray[lNumberOfWords - 2] = lMessageLength << 3;
        lWordArray[lNumberOfWords - 1] = lMessageLength >>> 29;
        return lWordArray;
      };
      function md5_WordToHex(lValue) {
        var WordToHexValue = "", WordToHexValue_temp = "", lByte, lCount;
        for (lCount = 0; lCount <= 3; lCount++) {
          lByte = (lValue >>> (lCount * 8)) & 255;
          WordToHexValue_temp = "0" + lByte.toString(16);
          WordToHexValue = WordToHexValue + WordToHexValue_temp.substr(WordToHexValue_temp.length - 2, 2);
        }
        return WordToHexValue;
      };
      function md5_Utf8Encode(string) {
        string = string.replace(/\r\n/g, "\n");
        var utftext = "";
        for (var n = 0; n < string.length; n++) {
          var c = string.charCodeAt(n);
          if (c < 128) {
            utftext += String.fromCharCode(c);
          } else if ((c > 127) && (c < 2048)) {
            utftext += String.fromCharCode((c >> 6) | 192);
            utftext += String.fromCharCode((c & 63) | 128);
          } else {
            utftext += String.fromCharCode((c >> 12) | 224);
            utftext += String.fromCharCode(((c >> 6) & 63) | 128);
            utftext += String.fromCharCode((c & 63) | 128);
          }
        }
        return utftext;
      };
      var x = Array();
      var k, AA, BB, CC, DD, a, b, c, d;
      var S11 = 7, S12 = 12, S13 = 17, S14 = 22;
      var S21 = 5, S22 = 9, S23 = 14, S24 = 20;
      var S31 = 4, S32 = 11, S33 = 16, S34 = 23;
      var S41 = 6, S42 = 10, S43 = 15, S44 = 21;
      string = md5_Utf8Encode(string);
      x = md5_ConvertToWordArray(string);
      a = 0x67452301; b = 0xEFCDAB89; c = 0x98BADCFE; d = 0x10325476;
      for (k = 0; k < x.length; k += 16) {
        AA = a; BB = b; CC = c; DD = d;
        a = md5_FF(a, b, c, d, x[k + 0], S11, 0xD76AA478);
        d = md5_FF(d, a, b, c, x[k + 1], S12, 0xE8C7B756);
        c = md5_FF(c, d, a, b, x[k + 2], S13, 0x242070DB);
        b = md5_FF(b, c, d, a, x[k + 3], S14, 0xC1BDCEEE);
        a = md5_FF(a, b, c, d, x[k + 4], S11, 0xF57C0FAF);
        d = md5_FF(d, a, b, c, x[k + 5], S12, 0x4787C62A);
        c = md5_FF(c, d, a, b, x[k + 6], S13, 0xA8304613);
        b = md5_FF(b, c, d, a, x[k + 7], S14, 0xFD469501);
        a = md5_FF(a, b, c, d, x[k + 8], S11, 0x698098D8);
        d = md5_FF(d, a, b, c, x[k + 9], S12, 0x8B44F7AF);
        c = md5_FF(c, d, a, b, x[k + 10], S13, 0xFFFF5BB1);
        b = md5_FF(b, c, d, a, x[k + 11], S14, 0x895CD7BE);
        a = md5_FF(a, b, c, d, x[k + 12], S11, 0x6B901122);
        d = md5_FF(d, a, b, c, x[k + 13], S12, 0xFD987193);
        c = md5_FF(c, d, a, b, x[k + 14], S13, 0xA679438E);
        b = md5_FF(b, c, d, a, x[k + 15], S14, 0x49B40821);
        a = md5_GG(a, b, c, d, x[k + 1], S21, 0xF61E2562);
        d = md5_GG(d, a, b, c, x[k + 6], S22, 0xC040B340);
        c = md5_GG(c, d, a, b, x[k + 11], S23, 0x265E5A51);
        b = md5_GG(b, c, d, a, x[k + 0], S24, 0xE9B6C7AA);
        a = md5_GG(a, b, c, d, x[k + 5], S21, 0xD62F105D);
        d = md5_GG(d, a, b, c, x[k + 10], S22, 0x2441453);
        c = md5_GG(c, d, a, b, x[k + 15], S23, 0xD8A1E681);
        b = md5_GG(b, c, d, a, x[k + 4], S24, 0xE7D3FBC8);
        a = md5_GG(a, b, c, d, x[k + 9], S21, 0x21E1CDE6);
        d = md5_GG(d, a, b, c, x[k + 14], S22, 0xC33707D6);
        c = md5_GG(c, d, a, b, x[k + 3], S23, 0xF4D50D87);
        b = md5_GG(b, c, d, a, x[k + 8], S24, 0x455A14ED);
        a = md5_GG(a, b, c, d, x[k + 13], S21, 0xA9E3E905);
        d = md5_GG(d, a, b, c, x[k + 2], S22, 0xFCEFA3F8);
        c = md5_GG(c, d, a, b, x[k + 7], S23, 0x676F02D9);
        b = md5_GG(b, c, d, a, x[k + 12], S24, 0x8D2A4C8A);
        a = md5_HH(a, b, c, d, x[k + 5], S31, 0xFFFA3942);
        d = md5_HH(d, a, b, c, x[k + 8], S32, 0x8771F681);
        c = md5_HH(c, d, a, b, x[k + 11], S33, 0x6D9D6122);
        b = md5_HH(b, c, d, a, x[k + 14], S34, 0xFDE5380C);
        a = md5_HH(a, b, c, d, x[k + 1], S31, 0xA4BEEA44);
        d = md5_HH(d, a, b, c, x[k + 4], S32, 0x4BDECFA9);
        c = md5_HH(c, d, a, b, x[k + 7], S33, 0xF6BB4B60);
        b = md5_HH(b, c, d, a, x[k + 10], S34, 0xBEBFBC70);
        a = md5_HH(a, b, c, d, x[k + 13], S31, 0x289B7EC6);
        d = md5_HH(d, a, b, c, x[k + 0], S32, 0xEAA127FA);
        c = md5_HH(c, d, a, b, x[k + 3], S33, 0xD4EF3085);
        b = md5_HH(b, c, d, a, x[k + 6], S34, 0x4881D05);
        a = md5_HH(a, b, c, d, x[k + 9], S31, 0xD9D4D039);
        d = md5_HH(d, a, b, c, x[k + 12], S32, 0xE6DB99E5);
        c = md5_HH(c, d, a, b, x[k + 15], S33, 0x1FA27CF8);
        b = md5_HH(b, c, d, a, x[k + 2], S34, 0xC4AC5665);
        a = md5_II(a, b, c, d, x[k + 0], S41, 0xF4292244);
        d = md5_II(d, a, b, c, x[k + 7], S42, 0x432AFF97);
        c = md5_II(c, d, a, b, x[k + 14], S43, 0xAB9423A7);
        b = md5_II(b, c, d, a, x[k + 5], S44, 0xFC93A039);
        a = md5_II(a, b, c, d, x[k + 12], S41, 0x655B59C3);
        d = md5_II(d, a, b, c, x[k + 3], S42, 0x8F0CCC92);
        c = md5_II(c, d, a, b, x[k + 10], S43, 0xFFEFF47D);
        b = md5_II(b, c, d, a, x[k + 1], S44, 0x85845DD1);
        a = md5_II(a, b, c, d, x[k + 8], S41, 0x6FA87E4F);
        d = md5_II(d, a, b, c, x[k + 15], S42, 0xFE2CE6E0);
        c = md5_II(c, d, a, b, x[k + 6], S43, 0xA3014314);
        b = md5_II(b, c, d, a, x[k + 13], S44, 0x4E0811A1);
        a = md5_II(a, b, c, d, x[k + 4], S41, 0xF7537E82);
        d = md5_II(d, a, b, c, x[k + 11], S42, 0xBD3AF235);
        c = md5_II(c, d, a, b, x[k + 2], S43, 0x2AD7D2BB);
        b = md5_II(b, c, d, a, x[k + 9], S44, 0xEB86D391);
        a = md5_AddUnsigned(a, AA);
        b = md5_AddUnsigned(b, BB);
        c = md5_AddUnsigned(c, CC);
        d = md5_AddUnsigned(d, DD);
      }
      return (md5_WordToHex(a) + md5_WordToHex(b) + md5_WordToHex(c) + md5_WordToHex(d)).toLowerCase();
    }
    ```

