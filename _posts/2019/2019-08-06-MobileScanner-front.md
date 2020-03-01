---
layout: post

title: MoblieScanner 一站式扫描平台[前端]

date: 2019-08-06 15:34:24.000000000 +09:00

---

![1](/assets/mobilescanner/mobilescanner_web.png)

## 前端历史演进

最开始 Web1.0阶段使用命令行看网页的，到后来有了窗口，广泛用于论文的查看，页面也是静态的，不能交互只是简单跳转，所以以前的前端工程师，就是模板工程师。直到后来Ajax的出现，才开始进入Web2.0. 页面可交互，数据读写。 

Ajax 是指 Asynchronous JavaScript and XML (异步的JavaScript和XML技术)，它之所以在这个时候出现是因为微软发布IE浏览器5.0版，才有了允许JavaScript脚本向服务器发起HTTP请求的能力。也是要依靠浏览器对它开放出能力才行。

Andriod开发还有机会接触系统内核。但作为iOS开发，我们处于苹果系统之上的应用层，只能用苹果提供的 Cocoa Touch，包含Foundation和UIKit框架等来开发iPhone应用。更可怜的是前端开发，只是浏览器应用的一个Tab页面而已。同样所拥有的能力仅限于浏览器应用内核给它开放出来的功能。

现在大部分网站都是 SPA单页面应用，就是整体页面不刷新，只要改变局部改变内容，实现的原理就是用JS中的window.onhashchange监听到了 fragment 的变化，从而通过JS改变document(Html)的内容，改变界面。document发生变化，浏览器就会重新刷新(局部)

SSR服务器端渲染, 是指有服务器渲染好Html，直接交给浏览器展示就好了，而不是通过传统的AJAX 在浏览器端去请求数据，再更新自己的document(Html)。 更新下面是传统Ajax和SSR服务器渲染页面的对比。

### 使用Ajax操作数据渲染到页面

1. 用户地址栏输入URL
2. 浏览器使用HTTP协议从后台获取资源
3. 浏览器解析并渲染HTML页面呈现到浏览器上，同时异步执行Ajax操作
4. 浏览器发送Ajax请求后台接口
5. 浏览器获取到数据后，执行回调函数，将内容动态追加到页面上

```
<!DOCTYPE html>
    <head>
        <script type="text/javascript" src="lib/jquery.min.js"></script>
        <script type="text/javascript">
            /**
             * 使用jQuery将后台接口返回的数据显示到页面上
             */
            function renderData(){
                $.post(url,param,function(result){
                    //假设返回的是是一个List，我们追加到页面的ul中
                    $.each(result,function(i,d){
                        $('#list').append('<li>' + d.name + '</li>');
                    })
                });  
            };
            renderData();
        </script>
    </head>
    <body>
        <ul id="list"></ul>
    </body>
</html>
```

### 使用SSR技术显示页面

1. 用户地址栏输入URL
2. 浏览器使用HTTP协议从后台获取资源
3. 浏览器解析并渲染HTML页面呈现到浏览器上

```
const Vue = require('vue')
const server = require('express')()
const renderer = require('vue-server-renderer').createRenderer()

server.get('*', (req, res) => {
  //vue对象包含了template+data
  const app = new Vue({
    data: {
        list: [{
            name : 'lilei'
        },{
            name : 'hanmeimei'
        }]
    },
    template: `<ul><li v-for="item in list">{{item.name}}</li></ul>`
  })

  //将vue对象传入最终返回output结果html
  //再将html通过reponse对象返回给前端浏览器
  renderer.renderToString(app, (err, html) => {
    res.end(`
      <!DOCTYPE html>
      <html>
        <body>${html}</body>
      </html>
    `)
  })
})

server.listen(8080)
```

有此可见，前端开发主要目的就是渲染document(Html)，就像iOS中搭建UIView视图框架一样。JS就相当于给页面元素增加了交互动作的能力。能运行JS也是浏览器支持的。

[大前端的技术原理和变迁史](https://juejin.im/post/5b5adc9b6fb9a04f9244555d)
[图解浏览器的基本工作原理](https://zhuanlan.zhihu.com/p/47407398)

## 主流框架 - Vue 

当前有了请求数据的能力后，也就有了分离数据和UI的需求，就会有MVC到MVVM的演进。Vue 框架该登场了。 有点类似于 Rx 里的 RxCocoa 和 RxSwift，能讲 View 和 ViewModel 之间双向绑定。

### 双向绑定-页面数据发生变化如何通知到JS ？
通过给页面元素添加 onchange 或者 oninput 事件，在事件中获取表单的值，然后赋值给Js对应的对象上即可。 
onchange 源于HTML 4 的新特性之一是可以使 HTML 事件触发浏览器中的行为，比方说当用户点击某个 HTML 元素时启动一段 JavaScript。
oninput 是HTML DOM 事件，允许Javascript在HTML文档元素中注册不同事件处理程序。

比如：示例中的输入框就可以添加oninput事件

```
<input type="text" oninput="evtInput" />
```
然后在js中定义这个函数执行相关赋值操作就可以：

```
function evtInput(){
    vue.name = this.value;
}
```

### 双向绑定-JS数据变化如何通知到页面？
JavaScript原生有个方法 Object.defineProperty() ，这个方法可以重新设置一个js对象中某个元素的一些属性，也就是重写了Set与Get方法，达到监听的目的

```
Object.defineProperty(data,'name',{
    set : function(v){
		document.getElementById('input').value = v;
	}
});
```

## 选择模板   

![1](/assets/mobilescanner/moban.png)

[vue-element-admin](https://github.com/PanJiaChen/vue-element-admin/blob/master/README.zh-CN.md) 是一个后台前端解决方案，它基于 vue 和 element-ui实现。如图所示他基本满足场景的需要，所以直接在上面修改成自己的业务再好不过了。业务数据获取也比较容易，剩下的塞数据就可以了。只是单点登录这里花了点时间。
 
### 单点登录

首选前端检测自己当前是否有username和ticket，他俩都是在单点登录后，登录服务器直接回调给后台并设置到cookies里的。如果他俩无效，说明登录过去或者未登录，直接去发起登录请求 ‘/sso/login’, 后台收到登录请求后直接返回203，前台可以通过http拦截器监听203这个错误，如果监听到直接跳转到单点登录服务网站去登录，企业的同学登录成功后，登录后台就会给咱们的后台发送有效的Code，拿着code，再去换一次ticket和name，成功后设置到cookies里。

```
/// 路由拦截
router.beforeEach(async(to, from, next) => {
  const username = getUserName()
  console.info('登录权限路由器在查看 username: ',username);
  if (username ) {//已经登录
    var routesNeedAdd = store.state.permission.routesNeedAdd;
    console.info('新登录或者强制刷新了，的需要重新添加动态路由 : ' + routesNeedAdd);
    if (routesNeedAdd){
      const accessRoutes = await store.dispatch('permission/generateRoutes');
      console.info('根据 User 动态配置的路由:');
      console.table(accessRoutes);
      router.addRoutes(accessRoutes)
      store.commit('permission/SET_ROUTE_NEED_ADD',false);
      next({ ...to, replace: true })  
    }else{
      next();
    }
  } else { // 未登录状态
    if (window.location.host.startsWith('ms.intra.didichuxing.com') &&
        window.location.pathname !== '/sso/callback'){
      login(); // 去申请登录 
    }
    next();
  }
})

// 响应拦截器，拦截到203错误后，跳转到登录网址去登录
httpInstance.interceptors.response.use(
    // 请求成功
    res => {
        if (res.status === 203) {
            window.location = 'http://mis.diditaxi.com.cn/auth/sso/login?app_id=2852&version=1.0&jumpto=http%3a%2f%2flocalhost%3a9528%2fdashboard';
        }
        return res.status === 200 ? Promise.resolve(res) : Promise.reject(res)
    },
    // 请求失败
    error => {
        return Promise.reject(error.response);
    }
);

/// 申请登录
function login () {
  console.info('请求登录');
  http.post('/sso/login')
  .then(response => {
      if (response.data.code === 0) {
          console.info('登录请求发送-成功');
      } else {
          console.info('登录请求发送-失败');
      }
  })
  .catch(function(error) {
      console.log(error);
  })
}
```

## 参考

[模板使用文档](https://panjiachen.github.io/vue-admin-template-site/)

[Vue&&TypeSprict--基础总结](https://www.kancloud.cn/cyyspring/vuejs/941801)

[Vue官方文档](https://cn.vuejs.org/v2/guide/)

[Vue Router](http://www.shouce.ren/api/view/a/11755)

[Vue2.0 探索之路——vue-router入门教程和总结](https://segmentfault.com/a/1190000009651628)
