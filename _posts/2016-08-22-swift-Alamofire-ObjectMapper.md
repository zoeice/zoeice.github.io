---
layout:     post
title:      Swift下封装Alamofire
subtitle:   Alamofire+ObjectMapper+RealmSwift
description: Alamofire算是Swift下AFNetworking的比较好的替代方案.
date:       2016-08-14 09:42:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-desktop.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-desktop.jpg?x-oss-process=image/resize,w_380
catalog: true
category: iOS
tags:
    - iOS
    - Swift
---

>Alamofire算是Swift下AFNetworking的比较好的替代方案

由于项目数据结构，需要加一层BaseModel来统一管理，再把业务数据分发下去，这里用`ObjectMapper`处理Json解析   
用`RealmSwift`来缓存请求数据.

Json文件如下：   

```json
{
    "success":false,
    "code":"ERR0008",
    "msg":"受限请求",
    "data":{
        "flag":"Student",
        "items":[
            {
                "name":"Tom",
                "age":12
            },
            {
                "name":"Jack",
                "age":18
            }
        ]
    }
}
```

在`podfile`里加入：  

```Shell
# 网络请求库
pod 'Alamofire', '~> 3.4'
# 本地存储
pod 'RealmSwift'
# JSON 解析
pod 'ObjectMapper', '~> 1.3'
```

统一管理的Model基类 `BaseModel.swift`：  

```swift
import ObjectMapper

class BaseModel : Mappable{
    /*
     *  请求状态
     */
    var success : Bool!
    /*
     *  错误码
     */
    var code : String?
    /*
     *  错误消息
     */
    var msg : String?
    /*
     *  响应内容
     */
    var data : AnyObject?

    required init(_ map: Map){
    }

    func mapping(map: Map) {
        success <- map["success"]
        code <- map["code"]
        msg <- map["msg"]
        data <- map["data"]
    }
}
```

>Objectmapper似乎不支持Realm的列表类型，解决办法是把JSON映射到一个 ignored array property，然后把item遍历添加进Realm的列表.    

Array转换类 [ArrayTransform.swift](https://gist.github.com/Jerrot/fe233a94c5427a4ec29b)：  

```swift
import RealmSwift
import ObjectMapper

class ArrayTransform<T:RealmSwift.Object where T:Mappable> : TransformType {
    typealias Object = List<T>
    typealias JSON = Array<AnyObject>

    func transformFromJSON(value: AnyObject?) -> List<T>? {
        let result = List<T>()
        if let tempArr = value as! Array<AnyObject>? {
            for entry in tempArr {
                let mapper = Mapper<T>()
                let model : T = mapper.map(entry)!
                result.append(model)
            }
        }
        return result
    }

    func transformToJSON(value: List<T>?) -> Array<AnyObject>? {
        if (value?.count > 0)
        {
            var result = Array<T>()
            for entry in value! {
                result.append(entry)
            }
            return result
        }
        return nil
    }
}
```

下面是学生Model类：`StudentModel.swift`, 就是通过这种方式处理的Realm的列表.  

```swift
import ObjectMapper
import RealmSwift

class StudentModel : Object, Mappable {
    dynamic var flag : String? = ""
    var items = List<StudentItem>()

    required convenience init?(_ map: Map){
        self.init()
    }

    func mapping(map: Map) {
        flag <- map["flag"]
        items <- (map["items"], ArrayTransform<StudentItem>())
    }
}

class StudentItem : Object, Mappable{
    dynamic var name : String = ""
    dynamic var age : Int32 = 0

    required convenience init?(_ map: Map){
        self.init()
    }

    func mapping(map: Map) {
        name <- map["name"]
        age <- map["age"]
    }
}
```

用Alamofire封装网络请求 `HttpClient.swift`   

```swift
import Alamofire
import ObjectMapper
import AlamofireObjectMapper

class HttpClient {

    static let instance = HttpClient()
    private init() {}

    /**
     *  网络请求成功闭包:
     */
    typealias HttpSuccess = (response:AnyObject) -> ()

    /**
     *  网络请求失败闭包:
     */
    typealias HttpFailure = (error:NSError) -> ()

}

extension HttpClient {
    /**
     * POST 请求, <注：GET请求类似>
     * parameter urlString:  请求URL
     * parameter parameters: 请求参数
     * parameter success:    成功回调
     * parameter failure:    失败回调
     */
    func post(path: String ,parameters: [String : String]?, success: HttpSuccess, failure : HttpFailure) {
        Alamofire.request(.POST, urlString, parameters: parameters, headers: configHeaders())
            .responseJSON { (response: Response<AnyObject, NSError>) in
                //处理逻辑
                switch response.result {
                case .Success(_):
                    if let value = response.result.value {
                        let info = Mapper<BaseModel>().map(value)
                        success(response:(info?.response)!)
                    }
                    break
                case .Failure(let error):
                    failure(error:error)
                    debugPrint(error)
                    break
                }
        }
    }

    /**
     * 配置headers, 可以自定义
     */
    func configHeaders() -> [String : String]? {
        let headers = [
            "content": "application/x-www-form-urlencoded; charset=utf-8",
            "Accept": "application/json",
            "token": "AOS51ADKH7881391"
        ]
        return headers
    }
}

```

ViewController里发起请求：

```swift
func queryStudent() {
    let params = [
        "page": "1"
    ]
    HttpClient.instance.post(URL_Student, parameters: params, success: { (response) in
        let data = Mapper<StudentModel>().map(response)
        print(data?.toJSONString())
        self.mTableView!.reloadData()
    }) { (error) in
        print(error)
    }
}
```

至此，成功取到数据并解析成Model
