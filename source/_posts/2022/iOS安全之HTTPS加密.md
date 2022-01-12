---
title: iOS安全之请求客户端单向加密
date: 2022-01-11 17:04:36
tags: [Swift,安全]
category: [iOS]
---

先了解一波一般的抓包原理：一般抓包都是通过代理服务来冒充你的服务器，客户端真正交互的是这个假冒的代理服务，这个假冒的服务再和我们真正的服务交互，这个代理就是一个中间者，我们所有的数据都会通过这个中间者，所以我们的数据就会被抓取。HTTPS 也同样会被这个中间者伪造的证书来获取我们加密的数据。

那么，我们针对这个代理抓包可以做两个处理：
第一个，简单的，做数据请求前，判断有没有代理;
第二个，如果是HTTPS的话，就就是判断HTTPS证书的合法性;

这里主要是对第二个方法做说明，主要用方法叫SSL Pinning，即SSL证书绑定，
通过SSL证书绑定来验证服务器身份，防止应用被抓包。

针对Swift，可以依赖Alamofire库，来进行证书验证，OC的话可以用AFNetworking。

主要分为五步：
1.从服务器取的证书；
2.导入证书；
3.导入Alamofire库；
4.设置本地Session；
5.使用我们自己的Session做请求；

### 1.从服务器取的证书

这里的证书指的是服务端HTTPS的SSL证书，证书有很多后缀比如: .cer，.pem ，.crt，以及.p12等，当然不止这一些，但是最好是用后缀是cer的证书。

如果服务器给的是如果他们给个.pem文件，要使用命令行转换：

openssl x509 -inform PEM -in name.pem -outform DER -out name.cer


如果给了个.crt文件，请这样转换：

openssl x509 -in name.crt -out name.cer -outform der


一般我们拿到后台人员导出的证书，会用到两个，一个是后缀名为cer(或者其他)的包含公钥的CA证书，用来让客户端对服务器进行验证；一个是后缀名为p12的客户端证书，用来让服务器对客户端进行验证,但是，我们要做的只要客户端对服务器端进行验证就好了，也就是我们只需要使用后缀为.cer的CA证书，当然，最安全的肯定是双向验证。

### 2.导入证书

1.通过Xcode左上角"File" -->  Add File to "xxxx"
将证书文件导入到项目

![导入文件](/images/iOS安全之HTTPS加密/1.png)

在这里要注意下添加完之后，在项目的"Build Phases" -->   "Copy Bundle Resoures" 目录里面应该是能找到证书文件的，如果没有找到，就需要在这个目录中再手动添加下

![检查文件](/images/iOS安全之HTTPS加密/2.png)


### 3.导入Alamofire库

CocoaPods导入： 
```

pod 'Alamofire', '~> 5.5'

```

Carthage导入： 
```

github "Alamofire/Alamofire" ~> 5.5

```

Swift Package Manager导入： 
```

dependencies: [
    .package(url: "https://github.com/Alamofire/Alamofire.git", .upToNextMajor(from: "5.5.0"))
]

```


### 4.设置本地Session

Alamofire的每一次请求都是由一个Session来维护的，本身会有一个默认的Session，叫 “Session.default”,我们要做的就是写一个有我们自己设置的Session对象，来做后续的请求,最好这个Session对象是由一个单例持有来维护的。

```
struct NetworkClient {
  
  
  struct Certificates {
    
    //获取本地证书的方法
    static let URLCer = Certificates.certificate(filename: "MyChain")
    private static func certificate(filename: String) -> SecCertificate {
      let filePath = Bundle.main.path(forResource: filename, ofType: "cer")!
      let data = try! Data(contentsOf: URL(fileURLWithPath: filePath))
      return SecCertificateCreateWithData(nil, data as CFData)!
    }
  }
  
  static let shared = NetworkClient()

//我们自己的session
  let session: Session
    //设置正确的服务器地址以及证书
  let evaluators = [
    "xxxxx.xxxx.com": PinnedCertificatesTrustEvaluator(certificates: [Certificates.URLCer]),

  ]

  init() {
    //session的设置
    let manager = ServerTrustManager(evaluators: self.evaluators)
    self.session = Session(configuration:URLSessionConfiguration.default,serverTrustManager: manager)
    
  }
  
}

```

### 5.使用我们自己的Session做请求

在我们做数据请求的地方将之前写的Session.request替换成我们自己的xxxx.shared.session.request就好了.一般是替换"AF.request"


至此，其实我们已经实现了客户端对服务器的验证，这样在每次请求前，客户端这边都会验证服务器的证书，如果被代理抓包的话，就会显示警告，抓取不到数据的。


### 6.扩展

其实在设置证书的位置，我们可以添加不用的验证方式，Alamofire提供的有6种:
```

DefaultTrustEvaluator: Uses the default server trust evaluation while allowing you to control whether to validate the host provided by the challenge.
RevocationTrustEvaluator: Checks the status of the received certificate to ensure it hasn’t been revoked. This isn’t usually performed on every request due to the network request overhead it entails.
PinnedCertificatesTrustEvaluator: Uses the provided certificates to validate the server trust. The server trust is considered valid if one of the pinned certificates match one of the server certificates. This evaluator can also accept self-signed certificates.
PublicKeysTrustEvaluator: Uses the provided public keys to validate the server trust. The server trust is considered valid if one of the pinned public keys match one of the server certificate public keys.
CompositeTrustEvaluator: Evaluates an array of ServerTrustEvaluating values, only succeeding if all of them are successful. This type can be used to combine, for example, the RevocationTrustEvaluator and the PinnedCertificatesTrustEvaluator.
DisabledTrustEvaluator: This evaluator should only be used in debug scenarios as it disables all evaluation which in turn will always consider any server trust as valid. This evaluator should never be used in production environments!

```

使用如下:
```

let evaluators: [String: ServerTrustEvaluating] = [
    // By default, certificates included in the app bundle are pinned automatically.
    "cert.example.com": PinnedCertificatesTrustEvaluator(),
    // By default, public keys from certificates included in the app bundle are used automatically.
    "keys.example.com": PublicKeysTrustEvaluator(),
]

let manager = ServerTrustManager(evaluators: evaluators)

```

这是一个验证方式的键值对，key是服务器地址，value值是验证方式

参考：https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#security
