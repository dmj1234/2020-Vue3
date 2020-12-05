---
title: Vue(第三部分:组件化开发)
tags: --！油加满，向前冲！学习使我快乐--！#
---

https://cn.vuejs.org/v2/guide/
####  全局组件注册语法
Vue.component(组件名称，{
    data: 组件数据  是一个函数
    template: 组件模板内容必须是单个跟元素，可以是模板字符串   
})
<!-- more -->
- 组件命名方式：
  短横线方式：
  Vue.component('my-component', {/*       */  })
  驼峰方式：
  Vue.component('MyComponent' , { /*   */})  
  驼峰方式注意事项：如果使用驼峰命名组件，那么再使用组件的时候，只能在字符串模板中用驼峰的方式使用组件，但是在普通标签模板中，必须使用短横线的方式使用组件。hello-world

- 局部组件注册：
  只能在注册它的父组件中使用

   var ComponentA = { /*  */};
   var ComponentB = { /*  */};
   var ComponentC = { /*  */};
       var vm = new Vue({
      el: '#app',
      data: {
        
      },
    components: {
      'component-a' :ComponentA,
      'component-b' :ComponentB,
      'component-c' :ComponentC,
      }
    })


#### Vue调试工具大法
- 调试工具安装     https://github.com/vuejs/vue-devtools
  克隆仓库
  安装依赖包     yarn install
  构建        yarn run build
  打开Chrome拓展页面
  选中开发者模式
  加载已解压的拓展，选择shell/chrome

#### 组件间数据交互
- 组件内部通过props接收传递过来的值
  Vue.component('menu-item' , {
    props:['title'],
    template: '<div> {{title}}</div>'
  })
- 父组件通过属性将值传递给子组件
  <menu-item title="来自父组件的数据"> </menu-item>
  <menu-item :title="title"></menu-item>
  - props属性名规则
     在props中使用驼峰形式，模板中需要使用短横线的形式
     字符串形式的模板中没有这个限制
- prop属性值类型：字符串String、数值Number、 布尔值Boolean、数组Array、对象Object     
- 都是可以通过属性的方式进行传递，在子组件中获取对应的值，布尔和数值通过v-bind进行绑定，在子组件中就可以得到对应的类型，不用则是字符串。

```js
  <div id="app">
    <div>{{pmsg}}</div>
    <menu-item :pstr='pstr' :pnum='12' pboo='true' :parr='parr' :pobj='pobj'></menu-item>
  </div>
  <script type="text/javascript" src="js/vue.js"></script>
  <script type="text/javascript">
    /*
      父组件向子组件传值-props属性值类型
    */
    
    Vue.component('menu-item', {
      props: ['pstr','pnum','pboo','parr','pobj'],
      template: `
        <div>
          <div>{{pstr}}</div>
          <div>{{12 + pnum}}</div>
          <div>{{typeof pboo}}</div>
          <ul>
            <li :key='index' v-for='(item,index) in parr'>{{item}}</li>
          </ul>
            <span>{{pobj.name}}</span>
            <span>{{pobj.age}}</span>
          </div>
        </div>
      `
    });
    var vm = new Vue({
      el: '#app',
      data: {
        pmsg: '父组件中内容',
        pstr: 'hello',
        parr: ['apple','orange','banana'],
        pobj: {
          name: 'lisi',
          age: 12
        }
      }
    });
```

- 子组件向父组件传值：通过自定义事件向父组件传递信息
  当你点击按钮button就触发一个特定的方法$emit 事件
  父组件监听子组件的事件 ：子组件被点击了，**父组件就响应事件**


#### 组件插槽
```js
    Vue.component('alert-box', {
      template: `
        <div>
          <strong>ERROR:</strong>
          <slot>默认内容</slot>
        </div>
      `
    });
```
插槽内容：<alert-box>Something bad happened <alert-box>


#### 案列时间：购物车
- 按照组件化方式实现业务员需求：
  根据业务功能进行组件化划分：
         标题插件：展示文本
         列表组件：列表展示、商品数量变更、商品删除
         结算组件：计算商品总额
- 功能实现步骤：
  实现整体布局和样式效果
  划分独立的功能组件
  组合所有的子组件形成整体结构
  逐个实现各个组件功能：标题组件、列表组件、结算组件

```js
<div id="app">
    <div class="container">
      <my-cart></my-cart>
    </div>
  </div>
  <script type="text/javascript" src="js/vue.js"></script>
  <script type="text/javascript">
    
    var CartTitle = {
      props: ['uname'],
      template: `
        <div class="title">{{uname}}的商品</div>
      `
    }
    var CartList = {
      props: ['list'],
      template: `
        <div>
          <div :key='item.id' v-for='item in list' class="item">
            <img :src="item.img"/>
            <div class="name">{{item.name}}</div>
            <div class="change">
              <a href="" @click.prevent='sub(item.id)'>－</a>
              <input type="text" class="num" :value='item.num' @blur='changeNum(item.id, $event)'/>
              <a href="" @click.prevent='add(item.id)'>＋</a>
            </div>
            <div class="del" @click='del(item.id)'>×</div>
          </div>
        </div>
      `,
      methods: {
        changeNum: function(id, event){
          this.$emit('change-num', {
            id: id,
            type: 'change',
            num: event.target.value
          });
        },
        sub: function(id){
          this.$emit('change-num', {
            id: id,
            type: 'sub'
          });
        },
        add: function(id){
          this.$emit('change-num', {
            id: id,
            type: 'add'
          });
        },
        del: function(id){
          // 把id传递给父组件
          this.$emit('cart-del', id);
        }
      }
    }
    var CartTotal = {
      props: ['list'],
      template: `
        <div class="total">
          <span>总价：{{total}}</span>
          <button>结算</button>
        </div>
      `,
      computed: {
        total: function() {
          // 计算商品的总价
          var t = 0;
          this.list.forEach(item => {
            t += item.price * item.num;
          });
          return t;
        }
      }
    }
    Vue.component('my-cart',{
      data: function() {
        return {
          uname: '张三',
          list: [{
            id: 1,
            name: 'TCL彩电',
            price: 1000,
            num: 1,
            img: 'img/a.jpg'
          },{
            id: 2,
            name: '机顶盒',
            price: 1000,
            num: 1,
            img: 'img/b.jpg'
          },{
            id: 3,
            name: '海尔冰箱',
            price: 1000,
            num: 1,
            img: 'img/c.jpg'
          },{
            id: 4,
            name: '小米手机',
            price: 1000,
            num: 1,
            img: 'img/d.jpg'
          },{
            id: 5,
            name: 'PPTV电视',
            price: 1000,
            num: 2,
            img: 'img/e.jpg'
          }]
        }
      },
      template: `
        <div class='cart'>
          <cart-title :uname='uname'></cart-title>
          <cart-list :list='list' @change-num='changeNum($event)' @cart-del='delCart($event)'></cart-list>
          <cart-total :list='list'></cart-total>
        </div>
      `,
      components: {
        'cart-title': CartTitle,
        'cart-list': CartList,
        'cart-total': CartTotal
      },
      methods: {
        changeNum: function(val) {
          // 分为三种情况：输入域变更、加号变更、减号变更
          if(val.type=='change') {
            // 根据子组件传递过来的数据，跟新list中对应的数据
            this.list.some(item=>{
              if(item.id == val.id) {
                item.num = val.num;
                // 终止遍历
                return true;
              }
            });
          }else if(val.type=='sub'){
            // 减一操作
            this.list.some(item=>{
              if(item.id == val.id) {
                item.num -= 1;
                // 终止遍历
                return true;
              }
            });
          }else if(val.type=='add'){
            // 加一操作
            this.list.some(item=>{
              if(item.id == val.id) {
                item.num += 1;
                // 终止遍历
                return true;
              }
            });
          }
        },
        delCart: function(id) {
          // 根据id删除list中对应的数据
          // 1、找到id所对应数据的索引
          var index = this.list.findIndex(item=>{
            return item.id == id;
          });
          // 2、根据索引删除对应数据
          this.list.splice(index, 1);
        }
      }
    });
    var vm = new Vue({
      el: '#app',
      data: {

      }
    });
 ```