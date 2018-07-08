# 软件设计文档
## 前端web页面部分
### 1. 概述
该点餐系统是使用vue框架开发的移动web页面，利用vuex组件进行状态管理，axios组件进行前后端交互，使用`mint-ui`组件库实现UI。整个web应用主要包括`MainPage`、`StoreInfoPage`、`OrderPage`和`PayPage`四个主要的页面组成，每个页面又由多个vue组件组成。
### 2. 技术选型理由
本次开发主要是在vue框架进行的前端开发，选择该框架主要有以下几个理由：
- vue上手容易。本次负责开发前端的组员之前并没有接触过前端开发，而vue让初学者也能快速上手
- vue是一个基于`MVVM模式`的数据驱动页面框架，具有声明式渲染和条件与循环两大特征，让我们能以非常简单的方法将数据与页面中的组件绑定在一起，减少了开发任务
- vue还有一个很重要的特征就是组件化应用构建，使我们能够将一个大的页面分解成多个不同的组件来进行开发。且这种组件化的构建方法更加有利于团队开发，提高了开发效率
- 本次实现的点餐系统主要是面向移动端的，vue更加适合用于开发移动端web应用
- vue能够很方便的引入第三方组件，利用这些组件我们可以很轻松的实现各种功能（比如用vuex实现不同页面不同组件之间的数据传递）
### 3. 架构设计
该点餐系统从总体上看是一个客户端-服务器模式的系统，客户端从服务器获取商家信息、菜品信息等数据并显示，然后向服务器提交订单数据。如果仅仅针对客户端来说，因为该系统使用的是vue框架，所以实际上是一个`MVVM架构`的系统。
![](https://github.com/CZXHenry/Documents/raw/master/软件架构图.png)
- 从总体上说，当用户打开点餐系统时，应用会向服务器发送请求，获取所有跟商家有关的信息并显示到界面上。与此同时，应用还会将这些数据写入到一个全局的数据对象，利用这个全局对象实现组件间的数据传递。而该系统实际上是一个单页面应用，页面之间的跳转利用路由来实现（这个其实就是vue框架的一个特性）。最后当用户选好菜品，确认付款之后，应用就会向服务器发送订单信息。
- 该应用不同页面之间的数据传递是用`vuex`实现的。当应用从服务器拿到包含商家信息的`json`文件之后，就会把解析该文件，把文件中的所有数据写到一个全局对象`store`（位于`src/store/index.js`)中。然后页面上的组件都是绑定了这个全局对象里面的数据，比如菜名、图片之类都是从这里获取数据然后进行显示。而页面发生数据改变，比如用户选择了某些菜，也会同步修改全局变量中的数据。而如何实现该全局对象让系统下面所有的组件都能够对他进行访问，则是利用了vuex这个第三方库来实现。
- 具体的数据绑定，也是交给vue框架来解决。而vue是一个`mvvm模式`的框架，`view`负责显示用户界面。`ViewModel`负责将数据对象与用户界面中的控件进行绑定，这样的绑定是双向的，在控件中改变数据会导致数据对象改变，而数据对象的改变也会导致控件中显示的数据的改变。`Model`就是数据对象。该系统中所有数据其实都储存在一个全局变量中。
### 4. 模块划分
该点餐系统主要分为四个主页面，然后每个页面主要是由多个不同的组件构成的

  ![](https://github.com/CZXHenry/Documents/raw/master//模块划分.png)
  
### 5. 软件设计技术
-该系统主要涉及`mvvm设计模式`，因为vue是一个基于`mvvc`的前端框架。以`MainPage.vue`为例，每个vue组件主要包含了`template`、`script`和`style`三大部分。
- `template`和`style`部分主要负责的就是用户界面的显示，对应的就是`view`
  ```html
  <template>
    <div class="mainPage" style="position:relative">
      <shopHeader style="z-index:2"></shopHeader>
      <dishesList style="z-index:1;"></dishesList>
      <div class="under-menu">
        <shoppingCart class="shoppingcart" v-show="$store.state.isShowShoppingCart"></shoppingCart>
        <under style="z-index:4;width:100%"></under>
      </div>
    </div>
  </template>
  ```
  ```html
  <style>
  .mainPage {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
  }
  .under-menu {
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  }
  .shoppingcart {
  z-index: 3;
  position: fixed;
  width: 100%;
  position: fixed;
  bottom: 8vh;
  }
  </style>
  ```
  - 可以看到代码中利用了 `v-show` 命令实现了数据的绑定，将全局数据对象中的数据绑定到了`shoppingCart`控件上。当购物车被点击唤出时，该数据对象就会变为1；当购物车不被唤出时，该数据对象就会变为0。而改变该数据对象也会影响购物车是否显示
- `script`部分包含了绑定到页面控件的数据以及操作这些数据的相关方法，其中`data`部分对应的就是`model`，其他部分对应的就是`ViewModel`
  ```html
    <script>
    import shopHeader from '../components/mainPage/shopHeader.vue'
    import under from '../components/mainPage/under.vue'
    import dishesList from '../components/mainPage/dishesList.vue'
    import shoppingCart from '../components/mainPage/shoppingCart.vue'
    import axios from 'axios'
    export default {
      name: 'mainPage',
      components: {
        'shopHeader': shopHeader,
        'dishesList': dishesList,
        'under': under,
        'shoppingCart': shoppingCart
      },
      data () {
        return {
        }
      },
      created () {
        this.getStoreInfo()
      },
      methods: {
        // 本地json获取商家信息
        getStoreInfo () {
          axios.get('http://172.18.218.192:9090/v1/rt/?rtname=RT1')
          .then((res) => {
            console.log(res)
            // this.storeInfo = res.data.store.info
          })
          .catch(function (error) {
            console.log(error)
          })
        }
      }
    }
  ```
  - `data`部分为空，是因为在这里直接绑定了全局的数据对象，这里的是储存的是局部的数据对象
  - `created`、`methods`中都定义了该如何操作数据对象。比如这里getStoreInfoPage函数就是表明从服务器中获取全部商家信息并将这些数据写入到全局数据对象中。
