# MMNNetwork

## INTRODUCE

简易的网络请求框架

## 接入环境

iOS版本：9.0+

## 接入指南

### Cocoapods接入

* 在工程Podfile文件中添加如下代码：

```
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/cosmos33/MMSpecs.git'

pod 'MMNNetwork'
```

* 在终端窗口项目根目录中运行以下命令：

```
 $ pod install
```

### 构建URLSessionManager

用于发起网络请求

```
    self.sessionManager = [[MNURLSessionManager alloc] initWithBaseURL:nil];
    self.sessionManager.requestSerializer.timeoutInterval = 15; // 设置超时时间
	// 设置json格式的网络响应解析类
    MNJSONResponseSerializer *responseSerializer = [MNJSONResponseSerializer serializer];
    responseSerializer.removesKeysWithNullValues = YES; // 移出Null值的数据
    self.sessionManager.responseSerializer = responseSerializer; 
```

### 如果用MNBaseRequest包装请求及解析请求，需遵循 MNNetworkProtocol 协议

```
    - (void)executeRequest:(__kindof MNBaseRequest *)request {
        // HTTP request method
        MNRequestMethod method = request.requestMethod;
        // request url
        NSString *url = request.serviceUrl;
        // params
        id params = request.requestParameters;
        // HTTPClient start request
        if (method ==  MNRequestPost) {
	    // 发起文件上传请求
            if (request.uploadDatas.count) {
                NSArray *uploadDatas = request.uploadDatas;
                request.sessionTask = [_sessionManager POST:url parameters:params constructingBodyWithBlock:^(id<MNMultipartFormData>  _Nonnull formData) {
                    [uploadDatas enumerateObjectsUsingBlock:^(MNUploadData *obj, NSUInteger idx, BOOL * _Nonnull stop) {
                        [formData appendPartWithFileData:obj.data name:obj.name fileName:obj.fileName mimeType:obj.contentType];
                    }];
                } completionHandler:^(id  _Nullable responseObject, NSError * _Nullable error) {
                    [request handleResponse:responseObject error:error];
                }];
            } else { // 发起普通post请求
                request.sessionTask = [_sessionManager POST:url parameters:params completionHandler:^(id  _Nullable responseObject, NSError * _Nullable error) {
                    [request handleResponse:responseObject error:error];
                }];
            }
        / 发起下载请求
        } else if (method == MNRequestDownload) {
            NSMutableURLRequest *urlRequest = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:url]
                                                                    cachePolicy:NSURLRequestReloadIgnoringCacheData
                                                                timeoutInterval:60]; // 可单独设置下载请求的超时时间
            request.sessionTask = [_sessionManager download:urlRequest
                                                destination:request.downloadDestinationBlock
                                                progress:request.progressBlock
                                       completionHandler:^(NSURL * _Nullable filePath, NSError * _Nullable error) {
                [request handleDowloadFilePath:filePath error:error];
            }];
        }
    }
```

### 可继承MNBaseRequest包装接口

- 子类包含拓展头文件
```
    #import "MNBaseRequest+Extension.h"
```

- 自定义初始化方法完成赋值

```
    - (instancetype)initWithURL:(NSString *)url requestMethod:(MNRequestMethod)requestMethod parameters:(NSDictionary *)parameters {
        self = [super init];
        if (self) {
            _serviceUrl = url.copy;
            _requestMethod = requestMethod;
            _requestParameters = parameters.copy;
        }
        return self;
    }  
```

- 重载解析方法解析回调数据

```
    - (void)handleResponse:(id _Nullable)responseObject error:(NSError * _Nullable)error {
        // 解析post请求响应数据
    }

    - (void)handleDowloadFilePath:(NSURL * _Nullable)filePath error:(NSError * _Nullable)error {
        // 执行下载完成操作
    }
```

## Author

zhu.xi, zhu.xi@immomo.com

## License

MMNNetwork is available under the MIT license. See the LICENSE file for more info.
