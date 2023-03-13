## 問題概述

.net 6已經設定了Cors但Angular再連接時仍有出現以下錯誤訊息
        
<font color=#FF0000>Access to XMLHttpRequest at'</font><font color=#0000FF>https://localhost:8080/API/Control</font><font color=#FF0000>' from origin '</font><font color=#0000FF>http://localhost:4200</font><font color=#FF0000>' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. 
</font>

## 相關設定
    1. .net6已做了Cors相關設定
        builder.Services.AddCors(options =>
        {
            options.AddPolicy("AllowAngularOrigins",
            builder =>
            {
          builder
            .AllowAnyHeader()
            .AllowAnyMethod()
            .SetIsOriginAllowed((host) => true)
            .AllowCredentials();
            });
        }
        並於Services.AddControllers()後
        加入此Cors設定
        app.UseCors("AllowAngularOrigins");
    
    2. Angular透過HttpClient呼叫此API
        this.http.post<any>('https://localhost:8080/API/Controllers', {}, httpOptions).subscribe(x=>{
        ##todo
      })
    
## 處理方法

於Angular Src目錄底下增加檔案proxy.conf.json
並填寫內容
    
    {
        "/api": {
        "target": "https://localhost:8080",
        "secure": false,
        "pathRewrite": {
          "^/api": ""
        },
        "changeOrigin": true,
        "logLevel": "debug"
          }
    }
此設定再連接到https://localhost:8080的API時自動進行Proxy的動作解決同源問題，另外再使用API時，如果url以API為開頭，將自動替換為https://localhost:8080，以下為範例：

    this.http.post<any>('https://localhost:8080/API/Controllers', {}, httpOptions).subscribe(x=>{
        //todo
      })
      
    ##上下語法等價
    
    this.http.post<any>('/API/Controllers', {}, httpOptions).subscribe(x=>{
        //todo
      })

可以使API的呼叫寫起來更整齊

再添加Proxy後，要去 angular.json 將Proxy Config指向此文件
    
      "serve": {
          "options": {
            "proxyConfig": "./src/proxy.conf.json"
          },
          "builder": "@angular-devkit/build-angular:dev-server",
          "configurations": {
            "production": {
              "browserTarget": "AngularWebApp:build:production"
            },
            "development": {
              "browserTarget": "AngularWebApp:build:development"
            }
          },
          "defaultConfiguration": "development"
        },
   
之後再重新ng serve一次，大概就可以解決問題了。

