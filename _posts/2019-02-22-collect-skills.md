---  
layout: post  
title: 编码常用知识  
description: 编码常用知识  
category: blog  
subtype: skill  
---  

### Ajax访问webservice
```
$.ajax({
        url: "http://localhost:8000/ApiService.asmx/ShowStudent",
        type: "post",
        contentType: "application/json;charset=utf-8",
        dataType: "json",
        data: "{'student':{'Name':'张三','Sex':'男','Age':18}}",
        success: function(result) { alert(result.d); },
        error: function(e) { alert("错误" + e.responseText) }
});
```

### Webservice支持跨域
```
<system.webServer>
    <httpProtocol>
      <customHeaders>
        <add name="Access-Control-Allow-Origin" value="*" />
        <add name="Access-Control-Allow-Methods" value="POST" />
        <add name="Access-Control-Allow-Headers" value="content-type" />
      </customHeaders>
    </httpProtocol>
  </system.webServer>
```
