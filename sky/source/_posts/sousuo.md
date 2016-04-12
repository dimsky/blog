---
title: 更聪明的搜索
---

人们已经越来越离不开搜索，搜索最大的目的其实就是让人在短时间内得到想要的东西，简单的说就是提升效率。

使用App也是在提升效率，淘宝、京东提升购物效率，微信、陌陌提升射交效率，猎聘、拉钩提升招聘效率...

iOS中的Spotlight已经不新鲜了，个人使用习惯的不同可能让有些人不知道她的存在，在手机存储暴增的今天，手机里不装个几百个App可能有点对不住你128G的存储了，但如果还滑来滑去的找应用那就说不过去了，在最新的iOS9中，Spotlight的功能肯定不止应用搜索这么简单。

简单的计算：
![计算器](/images/sousuo/1.png)

货币转换：

![货币转换](/images/sousuo/2.png)

以前可能还需要你打开App才能做到的，现在只需要简单几步就搞定，可能还有一些小惊喜等着你去发掘，如果你还有其他的好玩的，记得告诉我。


iOS 9 中的Spotlight还能搜索包括第三应用程序提供的app内信息，让用户搜索到的信息更可靠、更丰富、更人性化、更具联动性，并且允许开发者设置任意内容让Spotlight索引到。
我们能做些什么？
你不需要打开App 就能快速定位到想要找到的内容。
而且可以把更多的快捷操作放入搜索中。
* 输入好友名字立即打开聊天窗口；
* 输入歌名立即播放；
* 输入患者信息进入至患者详情；
* 输入cpatient(create patient)立即进入创建患者页面；
* ...

Apple也给了一些[应用场景](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/AppSearch/Choosing.html)

#### 新的API
在iOS 9 中提供了三种API来帮助我们实现搜索：
###### NSUserActivity
`NSUserActivity` 从iOS 8开始就已经在Handoff中使用了，可以通过activity 中传输的数据在另一台设备上继续。

![Handoff](/images/sousuo/3.png)
[文档](https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/Handoff/HandoffFundamentals/HandoffFundamentals.html)也说的很详细，本文主要讲搜索，就不对Handoff做过多说明。
在iOS 9中 `NSUserActivity`提供了一些新的方法和属性来帮助我们索引到指定的某个导航页面来继续操作，可以是新建或者查看列表内容等等。
###### Core Spotlight
新的Core Spotlight框架允许所有应用程序中的任何内容添加索引，来用Spotlight方便访问，当然你也可以对索引进行添加、删除、更新。使用Core Spotlight可以索引到应用程序中的内容，即使应用程序没有运行。
>Using a search extension, you can index content in the background even if your app is not running. This allows you to keep the index up to date as content changes even if the user isn't actively using your app. This is great for things like messages and conversations stored on your server but accessed through your app, or for locally generated user content.

###### Web Markup
这个特性就是你如果在你的网站上打上了Apple规范的markup内容，Applebot爬虫就会爬你的网站，然后出现在Spotlight 和Safari的搜索结果中。也就是说用户不用安装你的应用，你的应用也有机会展示给潜在用户。

##### 写几行代码吧

接下来要做的是让应用可以在Spotlight中根据更多的关键字搜索到。
###### 搜索
先看看搜索结果是如何展示的。
![](/images/sousuo/4.png)

    import Foundation
    import CoreSpotlight
    import MobileCoreServices

    @available(iOS 9.0, *)
    class SpotlightManager: NSObject {

        class func setupOpenAppAction() {
            var searchableItems: [CSSearchableItem] = []

            let openAppSet = CSSearchableItemAttributeSet(itemContentType: kUTTypeItem as String)
            openAppSet.title = "MedClipper"
            openAppSet.keywords = ["patient", "diagnosis", "doctor", "病历夹"]
            let openAppSetItem = CSSearchableItem(uniqueIdentifier: "openApp", domainIdentifier: nil, attributeSet: openAppSet)
            searchableItems.append(openAppSetItem)

            CSSearchableIndex.defaultSearchableIndex().indexSearchableItems(searchableItems, completionHandler: { (error) -> Void in
                if let error = error {
                    print("SearchableIndex error: \(error.localizedDescription)")
                }
            })
        }
    }

我加入了"patient", "diagnosis", "doctor", "病历夹" 关键字，通过这些关键字都能搜索出App，在应用启动之后调用`setupOpenAppAction`,退出应用就可以搜索了。
然后我们只需要在Spotlight中输入定义好的关键字，你的App就会出现在搜索结果中了。
![doctor](/images/sousuo/5.png)

当然，除了这种方式，你还可以给你的App起一个很长的名字或者副标题。

![QQ音乐](/images/sousuo/6.png)

你除了可以通过QQ音乐搜索到之外，还能通过听歌、K歌、FM、电台等关键字搜索到QQ音乐。


![小众音乐](/images/sousuo/7.png)

通过我们、音乐、倾听、彼此都能在Spotlight中搜索到小众音乐。

这样的命名方式无论是Spotlight 还是 App store 都可以让你的App在用户面前有更多的展示机会。


######搜索结果处理
接下来我们来处理搜索结果，点击之后的跳转，让用户可以通过搜索结果跳转到创建页面，或者是详情页面。

我们需要再定义一个用来执行创建操作的`CSSearchableItem`和多个可以被搜索用来查看详情的`CSSearchableItem`。

    import Foundation
    import CoreSpotlight
    import MobileCoreServices

    @available(iOS 9.0, *)
    class SpotlightManager: NSObject {
        class func setupOpenAppAction() {
            var searchableItems: [CSSearchableItem] = []

            let openAppSet = CSSearchableItemAttributeSet(itemContentType: kUTTypeItem as String)
            openAppSet.title = "MedClipper"
            openAppSet.keywords = ["patient", "diagnosis", "doctor", "病历夹"]
            let openAppSetItem = CSSearchableItem(uniqueIdentifier: "openApp", domainIdentifier: nil, attributeSet: openAppSet)
            searchableItems.append(openAppSetItem)

            let createPatientSet = CSSearchableItemAttributeSet(itemContentType: kUTTypeItem as String)
            createPatientSet.title = "Create Patient Info"
            createPatientSet.contentDescription = "创建患者信息"
            createPatientSet.keywords = ["cp", "create patient", "patient", "sick", "create patient", "cs"]
            let createPatientItem = CSSearchableItem(uniqueIdentifier: "create-Patient", domainIdentifier:nil, attributeSet:createPatientSet)
            searchableItems.append(createPatientItem)
            CSSearchableIndex.defaultSearchableIndex().indexSearchableItems(searchableItems, completionHandler: { (error) -> Void in
                if let error = error {
                    print("SearchableIndex error: \(error.localizedDescription)")
                }
            })
        }

        class func setupUserData() {
            CSSearchableIndex.defaultSearchableIndex().deleteSearchableItemsWithDomainIdentifiers(["patient"]) { (error) -> Void in
                if let error = error {
                    print("SearchableIndex delete error: \(error.localizedDescription)")
                } else {
                    var searchableItems: [CSSearchableItem] = []
                    let patients = Patient.findAll()
                    for (index, patient) in patients.enumerate() {
                        let patientSet = CSSearchableItemAttributeSet(itemContentType: kUTTypeItem as String)
                        patientSet.title = "\(patient.firstName) \(patient.lastName)"
                        patientSet.contentDescription = patient.medicalHistory
                        patientSet.keywords = [patient.firstName, patient.lastName]
                        let patientItem = CSSearchableItem(uniqueIdentifier: "patient-\(index)", domainIdentifier:"patient", attributeSet:patientSet)
                        searchableItems.append(patientItem)
                    }
                    CSSearchableIndex.defaultSearchableIndex().indexSearchableItems(searchableItems, completionHandler: { (error) -> Void in
                        if let error = error {
                            print("SearchableIndex error: \(error.localizedDescription)")
                        }
                    })

                }
            }
        }
    }

`setupOpenAppAction()` 定义快捷打开方式的索引。
`setupUserData()` 定义可搜索的数据索引。
由于用户在App中会对数据进行修改新增，所以把`setupUserData()`在用户按Home退出之后再更新数据索引。

    func applicationDidEnterBackground(application: UIApplication) {
        if #available(iOS 9.0, *) {
            SpotlightManager.setupUserData()
        }
    }

然后通过代理方法来处理用户点击搜索结果之后的回调，如果不做处理默认只会打开App。

    func application(application: UIApplication, continueUserActivity userActivity: NSUserActivity, restorationHandler: ([AnyObject]?) -> Void) -> Bool {
        if #available(iOS 9.0, *) {
            if userActivity.activityType == CSSearchableItemActionType {
                guard let identifier = userActivity.userInfo?[CSSearchableItemActivityIdentifier] as? String else {
                    return true
                }
                if identifier == "create-Patient" {
                    //跳转到新建的页面
                } else if identifier.hasPrefix("patient-") {
                    let patientId = identifier.substringFromIndex(identifier.startIndex.advancedBy(8))
                    //通过patientId 跳转到详情页面
                }
            }
        }
        return true
    }

我们通过`userInfo`中的identifier来确定用户的动作来执行下一步操作
好的，完成了，我们来看下效果：


![Activity Result](/images/sousuo/8.gif)

是的，就是这么简单，让搜索变得更聪明。

>据说聪明的人都懂得用搜索。

参考：
[App Search Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/AppSearch/index.html)
