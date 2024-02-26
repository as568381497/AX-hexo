---
title: SwiftUI之登陆界面
date: 2021-01-19 14:23:08
tags: [Swift,SwiftUI]
category: [iOS]
---

昨天捣鼓了下新出的SwiftUI的使用，绘制了一个登陆页面，但是在做登陆完成跳转的时候犯了难，网上介绍的都是利用NavigationView或者NavigationLink进行页面的跳转，但是，在登陆页面的时候，将登陆页面做成根页面进行跳转，总觉得很奇怪，虽然按着方法push跳转后返回按钮能隐藏掉，但是，在栈里面，根页面还是存在了，并且是登陆页面，所以想用一下其他的方法，于是找到了一个利用ObservableObject协议来实现重绘View的一个方法。

那，我们的目标就来了，需求就是：

1.在登陆页面登陆成功时，直接跳转到首页，不用push的方式去跳转；
2.后续进行push跳转时，根页面是首页；
3.用户已经登陆，再进入APP时，直接到首页；


方法是：

在首页和登陆页面之前有一个父View，App显示的根View也是他，他来根据某一个值，来判断是现实登陆页面还是首页，也或者在登陆页面实现跳转。


步骤是：

1.利用ObservableObject协议，定义一个控制绘制页面的类

```

class CenterLogin: ObservableObject {
    //是否跳转到首页
    @Published var isLogin = false
    
    func toLogin()  {
        self.isLogin = true
    }
}


```

在这个类里面有一个控制是绘制登陆页面还是首页的属性“isLogin”，当为false的时候，到登陆页面，当为true的时候到首页。

2.定义登陆页面的View和首页的View

```

//登陆页
struct LoginPage: View {
    
    //用户账号
    @State private var userName: String = ""
    //用户米密码
    @State private var userPassWord: String = ""
    
    //持有一个跳转控制器
    @ObservedObject var loginCenter : CenterLogin
    
    @State var isActive = false
    let detail = HomePage()
    
    var body: some View {
        
        VStack {
            HStack{
                
                Label("账号：", systemImage: "")
                TextField("请输入账号：", text: $userName)
                
            }
            .padding(EdgeInsets(top: 10, leading: 20, bottom: 10, trailing: 20))
            
            
            HStack{
                
                Label("密码：", systemImage: "")
                TextField("请输入密码：", text: $userPassWord)
                
            }
            .padding(EdgeInsets(top: 10, leading: 20, bottom: 10, trailing: 20))
            
            Button(action: ({
                //前往首页
                self.loginCenter.toLogin()
                
            }), label: {
                Text("登陆")
                    .accentColor(.blue)
                
            })
            .padding(.top)
            
            
            
        }
        
        
    }
    
}

//首页
struct HomePage: View {
    
    var body: some View {
        
        TabView{
            
            Text("Tab1")
                
                .tabItem {
                    
                    Image(systemName: "1.square.fill")
                    
                    Text("First")
                    
                }
            
            Text("Tab2")
                
                .tabItem {
                    
                    Image(systemName: "2.square.fill")
                    
                    Text("Second")
                    
                }
            
            Text("Tab3")
                
                .tabItem {
                    
                    Image(systemName: "3.square.fill")
                    
                    Text("Third")
                    
                }
            
            Text("Tab4")
                
                .tabItem {
                    
                    Image(systemName: "4.square.fill")
                    
                    Text("four")
                    
                }
            
        }
        
    }
    
}


```

在登陆页面，需要持有一个父View传进来的绘制控制器，也就是我们定义的“CenterLogin”类，因为我们需要在用户登陆成功的时候，调用这个类的方法，给"isLogin"重新赋值，同时调用父View重新绘制页面。

3.定义现实的父View

```

//显示的父类
struct UserRootPage: View {
    //持有跳转控制器
    @ObservedObject var login = CenterLogin()
    
    
    init() {
        //init时切换页面
        self.login.isLogin = true
    }
    
    var body: some View {
        VStack{
            if self.login.isLogin == false{
                LoginPage(loginCenter: login)
            }else{
                HomePage()
            }
            
        }
    }
}


```

APP显示的也是这个View，由他来判断是显示那个页面，同时他持有一个“CenterLogin”的类，来交给登陆页，告诉自己什么时候重新绘制页面，在init的方法里面，可以进行判断，用户是否是已经登陆过的，如果已经登陆过直接绘制首页。


到此，我们的功能也就全部实现完成了，指的注意的是"ObservableObject"这个类，在这个类是SwiftUI框架所包含的，所有的类都可以调用，实现了这个协议的类，在类里面所用“Published”关键字声明的属性，在属性改变时，可使拥有此类实例的视图重新调用body属性，也就是可以重新绘制界面。
