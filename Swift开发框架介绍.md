

###一 :  简单介绍以下几个方面

>1 编码规范
>2 设计模式的选择
>3 项目目录结构
>4 网络层
>5 数据存储
>6 日志收集
>7 安全性
>8 自动化打包和测试(比如版本管理工具)

###二 : 看过Casa的一段话,列下项目一般满足点

* 代码整齐，分类明确 ,没有common，没有core
* 多聚合 , 少继承 , 多协议；高内聚 , 低耦合
```
(1) 功能内聚和数据耦合，是我们需要达成的目标。
(2) 横向的内聚和耦合，通常体现在系统的各个模块、类之间的关系，
    而纵向的耦合，体现在系统的各个层次之间的关系。
(3) 少继承, 多考虑协议
(4) 最终: 易测试，易拓展
```
* 思路和方法要统一，尽量不要多
* 对公用方法该限制的地方有限制，该灵活的灵活
* 保持一定量的超前性
* 接口少，接口参数少, 方法语义明确
* 高性能

#### 三 : 编码规范

详见: [Swift编码规范](https://www.jianshu.com/p/a89a58848bff)

补充:

1 避免用OC式的语法写Swift : 举例赋值字符串操作,为空设置默认值


* OC式的语法写Swift
```
 var nameString  : String
 var newNameString : String?
 newNameString = nil

 if (newNameString != nil) {//不为空赋值
      nameString = newNameString!
 } else {
     nameString = "昵称"
 }
```
* Swift语法写法:
```
if let nameStr = newNameString { //解包成功赋值
    nameString = nameStr
} else {
     nameString = "昵称"
}
```
* 简洁优雅的使用Swift:
```
nameString = newNameString ?? "昵称"
```

2 充分利用Swift的新特性
能用协议的尽量不用继承, 这样有利于项目扩展和分工合作等等

3 能用code尽量不用storyboard 和 xib

```   
当团队具有一定规模的（比如iOS开发 > 5 人）,storyboard 和 xib缺点和明显
同一份代码文件的作者会有很多，不同作者同时修改同一份代码的情况也不少见。
因此，使用Git进行代码版本管理时出现Conflict的几率也比较大。
需求变化非常频繁，为了完成需求而针对现有代码进行微调的情况，以及针对现有代码的部分复用的情况
复杂界面元素、复杂动画场景的开发任务
```

#### 四 : 设计模式的选择 : MVVM ( MVC、MVCS、VIPER )
目前[iOS常用的几种设计模式](https://www.cnblogs.com/wangbinios/p/7882082.html)：代理模式、观察者模式、单例模式、策略模式、工厂模式、MVC模式等

弃用MVC模式
```当一个项目越来越大的时候,MVC模式会变得越来越难维护了,所以有必要对其进行瘦身.```
小型项目我考虑使用MVVM ( 就算项目再次变大,依然可以很方便的拆开Controller和Model ,有利于重构 )
```
View <-> Controller <-> ViewModel <-> Model

附: MVVM模式高级使用框架RXSwift 和RXcocoa
```

###### 1 简单聊点 依赖 问题:
 从A页面push到B页面:
```var mineVC = MineViewController()
mineVC.title = "个人主页"
mineVC.ID = personID
mineVC.isMe = isMe
self!.navigationController?.pushViewController(mineVC, animated: true)
```
如果以后需求改变了,需要增加几个参数或者是删除参数, 如此改动会很麻烦
那么这些操作其实不必要Controller来做:
```
PushModel : 专门用来管理需要被push或者present事件
BDataModel : 专门声明需要传递的参数,根据需要进行扩展
```
如此,控制器和控制器之间不会过度依赖,只需要告诉PushModel我有个BDataModel给你就行

###### 2 简单聊聊高频使用的UITableViewCell,通过cellForRowAt方法如何瘦身来理解如何拆分

(1) 瘦身前的控制器: 显得异常臃肿
```
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let aNews = news[indexPath.row]
        switch aNews.cell_type {
        case .user:             // 用户
            let cell = tableView.ym_dequeueReusableCell(indexPath: indexPath) as HomeUserCell
            cell.readCountLabel.text = "\(aNews.readCount)阅读"
            cell.verifiedContentLabel.text = aNews.verified_content
            cell.digButton.setTitle(aNews.diggCount, for: .normal)
            cell.commentButton.setTitle("\(aNews.commentCount)", for: .normal)
            cell.feedshareButton.setTitle(aNews.forwardCount, for: .normal)
            cell.contentLabel.attributedText = aNews.attributedContent
            //cell.aNews = aNews
            return cell
        case .relatedConcern:   // 相关关注
            let cell = tableView.ym_dequeueReusableCell(indexPath: indexPath) as TheyAlsoUseCell
            cell.readCountLabel.text = "\(aNews.readCount)阅读"
            cell.verifiedContentLabel.text = aNews.verified_content
            cell.digButton.setTitle(aNews.diggCount, for: .normal)
            cell.commentButton.setTitle("\(aNews.commentCount)", for: .normal)
            cell.feedshareButton.setTitle(aNews.forwardCount, for: .normal)
            cell.contentLabel.attributedText = aNews.attributedContent
            cell.theyUse = aNews.raw_data
            return cell
        }
    }
```
(2) 瘦身后的控制器 : 通过view的set方法,把赋值代码移动到cell里,或者建立一个DataModel来处理这件事情. 

```
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let aNews = news[indexPath.row]
        switch aNews.cell_type {
        case .user:             // 用户
            let cell = tableView.ym_dequeueReusableCell(indexPath: indexPath) as HomeUserCell
            cell.aNews = aNews
            return cell
        case .relatedConcern:   // 相关关注
            let cell = tableView.ym_dequeueReusableCell(indexPath: indexPath) as TheyAlsoUseCell
            cell.theyUse = aNews.raw_data
            return cell
        }
    }
```
上面代码仍然有一定的冗余,两个cell进行了两次初始化,如果十个呢,显然会越来越臃肿.

(3) 进一步瘦身后的控制器 :  控制器只需要对cell进行一次赋值就行,具体什么cell让cell的DataModel去做就行了.
```
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let row = fromData[indexPath.row]
        let cell = tableView.dequeueReusableCell(withIdentifier: row.reuseIdentifier, for: indexPath)
        row.update(cell: cell)
        return cell
    }
```


#### 五 : 项目目录结构 : 根据不同项目需求决定

```
  *  Base : 存放一些基类,比如BaseViewController,BaseModel等,共性直接在基类中去修改
  *  Vendor : 三方,因为我的项目中使用cocopods管理三方,所以这个文件夹中我在此放的是一些比较小的功能的第三方或者不支持cocopods的
  *  Framework : 存放一些类库或者自己封装的一些静态库
  *  Resource : 存放app中一些索引资源,比如图片,文本等,或者将图片打包的Bundle
  *  Custom : 这个文件夹我用来存放自己项目或者公司自己风格的一些自定义的视图框架( 或SDK )
  *  Network : 这个只专门用来做网络处理的,因为这个项目基本上都会用到网络请求,算是比较重要的一个部分,所以在此单独拿出来作为一个分类
  *  Expand : 扩展文件
      -- Tool : 工具
      -- DataBase: 数据存储相关
      -- Macros : 宏定义
      -- Const : 常量
  *  Main : AppDelegate或者AppDelegate的Category 等等
  *  Class : 存放的是App中所有的模块功能
       - Public : 公用
       - Home : 比如首页
             -- Controller
             -- Model
             -- View
             -- ViewModel
             -- Other

 * XXX-Tests :        单元测试
 * XXX-UITests  :   UI测试
 * Products :         系统自动生成的.app所在文件夹
 * Pods  :               Pods第三方库

```

##### 六 : 网络层 
[Alamofire](https://link.jianshu.com?t=https://github.com/Alamofire/Alamofire) : 首选主流网络开源框架
Moya 
RXSwift / RXCocoa

>pod "Moya/RxSwift"

######以下是艺下NetWork.swift 示例代码:
```
import Foundation
import Moya
public enum NetWork {
    case login(mobile:String,captcha:String)
}

extension NetWork:TargetType{
    public var sampleData: Data {
        return Data.init()
    }
    
    /// The target's base `URL`.
    public var baseURL: URL {
        return API.base.rawValue.url!
    }
    
    // The path to be appended to `baseURL` to form the full `URL`.
    public var path: String {
        switch self {
        case .login(mobile: _, captcha: _):
            return "/auth/login";
        }
    }
    
    // The HTTP method used in the request.
    public var method: Moya.Method {
        switch self {
        case .login(mobile: _, captcha: _):
            return .get;
        }
    }

    // The type of HTTP task to be performed.
    public var task: Task {return .requestPlain}

    // The type of validation to perform on the request. Default is `.none`.
    public var validationType: ValidationType { return .none }
    
    // The headers to be used in the request.
    public var headers: [String: String]? {
        var AlamHeaders = [*****]
        AlamHeaders["sign"] = ***
        (一堆设置)
        return AlamHeaders
    }
}

```

##### 七 : 数据存储

    根据需求决定持久化方案
    持久层与业务层之间的隔离
    持久层与业务层的交互方式
    数据迁移方案
    数据同步方案
    损坏修复
    等等

######目前移动端数据库方案按其实现可分为两类，
>一 : 关系型数据库，代表有CoreData、FMDB等。
>[CoreData]()
>它是苹果内建框架，和Xcode深度结合，可以很方便进行ORM；但其上手学习成本较高，不容易掌握。稳定性也堪忧，很容易crash；多线程的支持也比较鸡肋。
>[SQLite]() 
>[FMDB]() 
>它基于SQLite封装，对于有SQLite和ObjC基础的开发者来说，简单易懂，可以直接上手；而缺点也正是在此，FMDB只是将SQLite的C接口封装成了ObjC接口，没有做太多别的优化，即所谓的胶水代码(Glue Code)。使用过程需要用大量的代码拼接SQL、拼装Object，并不方便。

>二 : key-value数据库，代表有Realm、LevelDB、RocksDB等。
>[Realm]()
>1 因其在各平台封装、优化的优势，比较受移动开发者的欢迎。对于iOS开发者，key-value的实现直接易懂，可以像使用NSDictionary一样使用Realm。并且ORM彻底，省去了拼装Object的过程。
>2 是一个跨平台的移动数据库: Java，Objective-C，Swift，React Native，Xamarin 等等
>3 Realm支持事务，满足ACID：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）。

>三 : WCDB 是微信推出的一个高效、完整、易用的移动数据库框架，基于SQLCipher，支持iOS, macOS和Android。易用，支持事务，可加密、损坏修复。WCDB在迭代中也在慢慢的稳定

[Realm、WCDB与SQLite移动数据库性能对比测试:](https://www.jianshu.com/p/8187eddbe422)

![image](http://upload-images.jianshu.io/upload_images/2963877-f523084a6d645c3d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](http://upload-images.jianshu.io/upload_images/2963877-a430b7e7b88e0041?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


######目前比较好的是Realm 和 微信的 WCDB ,任意一个都可以,可根据项目自己选择 .

#### 八 : 日志收集 
一个 APP不可避免的要对其进行,bug收集和日志分析

> 1 一种方式是发生崩溃以后上传日志到自己的服务器,然后进行分析

>2 另外一种是第三方接口, 比如bugly的日志收集管理系统 : 功能比较强大,不仅仅是崩溃日志,甚至是卡顿等都能收集,并且有可视化的管理系统方便查看和分析

根据业务需要选择一个合适的方式.

#### 九 : 安全性

1.  对请求进行加密认证,修改使用POST ,DELETE 等 提交表单，
2.  使用https
3.  对用户敏感数据 使用密钥进行加密
4.  本地配置文件中的敏感数据进行加密 或 保存在keychain中
    等等

#### 十 : 自动化打包和测试,持续集成工具
测试是对项目健全性能的保障：包括 性能、内存weak、UnitTest，AutoTest 、UITest 等等

* UnitTest 单元测试 分为3种：
   1 逻辑测试：测试逻辑方法. [--逻辑测试](https://www.jianshu.com/p/15f347eb207c)
   2 异步测试：测试耗时方法（用来测试包含多线程的方法）:  [--异步测试](https://www.jianshu.com/p/a4ffa780d347)
  3 性能测试：测试某一方法运行所消耗的时间 [--性能测试 ](https://www.jianshu.com/p/3ea8be84f53a)

* UI 自动化测试 ：
   [iOS 自动化测试 UI Automation](https://www.jianshu.com/p/0e28ae1bd2c2) 【xcode 8 找不到 Automation ？】

* 代码版本管理 : 持续集成工具
  例如 CruiseControL，hudson ，**jenkins**，还有apache的Continuum 等 开源的持续集成工具
  [jenkins 配置 ](https://www.jianshu.com/p/8d4452c6f17e)
