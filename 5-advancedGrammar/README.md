# 第5章 Vue 中的高级语法
## 5-1 Mixin 混入的基础语法
1. 组件中 data、methods 优先级高于mixin中 data、methods 优先级；
    - > 底层：数据对象在内部会进行递归合并，并在发生冲突时以组件数据优先；
    - > 底层：同名钩子函数将合并为一个数组，因此都将被调用。另外，混入对象的钩子将在组件自身钩子之前调用；
2. 生命周期函数，先执行 mixin 里面的，再执行组件中的；
3. 局部 mixin 使用时，需在每一层组件中注册`mixins: [myMixin],`
```js
const myMixin = {
    data() {
        return {
            num: 999,
            count: 666
        }
    },
    created() {
        console.log("mixin created")
    },
    methods: {
        handleClick() {
            console.log("mixin handleClick")
        }
    },
};

const app = Vue.createApp({
    data() {
        return {
            // num: 2,
        }
    },
    created() {
        console.log("created")
    },
    mixins: [myMixin],
    methods: {
        // handleClick() {
        //     console.log("handleClick")
        // }
    },
    template: `
        <div>
            <div>{{num}}</div>
            <div>{{count}}</div>
            <button @click="handleClick">切换</button>    
        </div>
    `
})

app.mount("#root")
```
4. 全局 mixin 使用时，不需要再在每一层组件注册，只需要全局注册`Vue.mixin({})`
    - 请谨慎使用全局混入，因为它会影响每个单独创建的 Vue 实例 (包括第三方组件)；
    - 大多数情况下，全局 mixin，只应当应用于自定义选项，就像上面示例一样。推荐将其作为插件发布，以避免重复应用混入；
5. 自定义属性：不存在于Vue定义的任何属性中的属性
    - 组件中的自定义属性  优先级高于  mixin中的自定义属性；
    - 无论在组建或者mixin中都可以使用`this.$options.myOption`调用自定义属性;
6. `app.config.optionMergeStrategies.myOption`可以自定义自定义属性的优先级；
```js
// 为自定义的选项 'myOption' 注入一个处理器。
const app = Vue.createApp({
    //myOption 不存在于Vue定义的任何属性中，为自定义属性
    myOption: 'hello!',
    template: `<div>{{this.$options.myOption}}</div>`
})

app.mixin({
    myOption: 'bye!',
    created: function () {
        //this.$options.myOption 调用自定义属性
        var myOption = this.$options.myOption
        if (myOption) {
            console.log(myOption)
        }
    }
})

app.config.optionMergeStrategies.myOption = (mixinValue, appValue) => {
    return mixinValue || appValue;
}

app.mount("#root")
```

> Vue3版本之后，不建议使用mixin，可以用compositionAPI、插件来代替；

## 5-2 开发实现 Vue 中的自定义指令
1. `Vue.directive("xxx", {})`:注册一个全局自定义指令;
2. `const directives = {xxx:{……}}`:注册一个局部自定义指令;
3. `<input v-xxx/>`:在template中使用该指令;
```js
//局部自定义指令
const directives = {
    focus2: {
        mounted(el) {
            el.focus()
        }
    }
}

const app = Vue.createApp ({
    //组件中注册局部自定义指令
    directives: directives,
    template: `
        <div>
            <input v-focus2/>
        </div>
    `
})

// 全局自定义指令定义
// app.directive("focus1", {
//     mounted(el) {
//         el.focus();
//     }
// })

app.mount("#root")
```
4. 当自定义指令中的mounted、updated两个生命周期函数中的内容相同时可以进行合并简写;
```js
app.directive("position", {
    mounted(el, binding){
        el.style.position = "absolute";
        el.style.top = (binding.value + "px");
    },
    updated(el, binding){
        el.style.position = "absolute";
        el.style.top = (binding.value + "px");
    }
})
```

可以简写为：

```js
app.directive("position", (el, binding) => {
    el.style.position= "absolute";
    el.style.top = (binding.value + "px");
});
```
5. 自定义指令的`binding`中，可以找到一切template中传入的属性或值或参数
```js
const app = Vue.createApp({
    data() {
        return {
            distance: 100
        }
    },
    template: `
        <div>
            <div v-position:right="distance" class="header">
                <input />
            </div>
        </div>
    `
});

app.directive("position", (el, binding) => {
    console.log(binding);
    el.style.position= "absolute";
    el.style[binding.arg] = binding.value + "px";
});

const vm = app.mount("#root");
```
## 5-3 Vue3之Teleport 传送门功能
1. `<teleport to="body">目标元素</teleport>`将组件中的某个元素或组件，挂载到其它的DOM上面；
2. 目标元素可挂载的其他DOM，但其他的数据还在该组件中；
```js
const app = Vue.createApp({
    data() {
        return {
            distance: 100,
            show: false,
            message: "我在这"
        }
    },
    methods: {
        handleClick() {
            this.show = !this.show;
        }
    },
    template: `
            <div class="area">
                <button @click="handleClick">蒙住</button>
                <teleport to="body">
                    <div v-show="show" class="mask">{{message}}</div>
                </teleport>
            </div>
    `
});

const vm = app.mount("#root");
```
## 5-4 更加底层的 render 函数
1. 若在组件中实现如下标题展示，其中标题按需展示的层级按照父组件模板`:level="5"`传入的参数不同而动态展示。
```js
const app = Vue.createApp({
    template: `
        <my-title :level="5">
            hello
        </my-title>
    `
});

app.component("my-title", {
    props:["level"],
    template: `
        <h1 v-if="level === 1"><slot/></h1>
        <h2 v-if="level === 2"><slot/></h2>
        <h3 v-if="level === 3"><slot/></h3>
        <h4 v-if="level === 4"><slot/></h4>
        <h5 v-if="level === 5"><slot/></h5>
    `
})

const vm = app.mount("#root");
```
    
2. 代码如上，不但代码冗长，而且在每一个级别的标题中重复书写了 `<slot></slot>`，在要插入锚点元素时还要再次重复。虽然模板在大多数组件中都非常好用，但是显然在这里它就不合适了。那么，我们来尝试使用 render 函数重写上面的例子：
    - `h()`函数中的参数可见文档：https://cn.vuejs.org/v2/guide/render-function.html#createElement-%E5%8F%82%E6%95%B0
        - `this.$slots.default()`位置所代表的的第三个参数可以为：
            - 字符串
            ```js
            render() {
                const { h } = Vue;
                return h("h"+ this.level, {}, this.$slots.default())
            }
            ```
            - 数组
            ```js
            render() {
                const { h } = Vue;
                return h("h"+ this.level, {}, [
                    this.$slots.default(),
                    h("h3", {}, "子元素")
                ])
            }
            ```
```js
const app = Vue.createApp({
    template: `
        <my-title :level="2">
            hello
        </my-title>
    `
});

app.component("my-title", {
    props:["level"],
    render() {
        const { h } = Vue;
        return h("h"+ this.level, {}, this.$slots.default())
    }
})

const vm = app.mount("#root");
```
3. `render(){}`和`template`有什么关系，`template`如何渲染到页面的：
    1. `template`在底层被编译会生成`render(){}`;
    2. `render(){}`调用底层 Vue 中的 h 方法(即`return h()`)，生成了虚拟DOM(即js对象表示DOM节点)；
        - 示例中的虚拟DOM为
        ```js
        tagName: "h2",
        text: "hello",
        attributes: {}
        ```
    3. 产生虚拟DOM后，Vue再在底层做映射生成真正的DOM，展示到页面上；
    
    >`template`渲染到页面流程图：
    >*** `template`--编译-->`render(){}`--调用-->`h()`--生成-->`虚拟DOM(JS对象)`--映射-->`真实DOM`--渲染-->`浏览器展示` ***

## 5-5 Vue插件的开发和使用
1. 插件通常用来为 Vue 添加全局功能;
    - 也是Vue-router等插件实现的原理
```js
const myPlugin = {
    install(app, options) {
        // 拓展依赖注入
        app.provide("name", options.name);
        
        //拓展指令
        app.directive("focus", {
            mounted(el) {
                el.focus()
            }
        });

        //拓展mixin
        app.mixin({
                mounted() {
                    //打印了两次，因为是app调用的，故app下的每一个组件调用都执行一次，共两个组件，故打印了两次；
                    console.log("mixin!");
                }
        });

        //针对Vue底层做全局的属性做拓展；
        app.config.globalProperties.$myMethod = "hey!!!";

    }
};

const app = Vue.createApp({
    template: `
        <my-title>
        </my-title>
    `
});

app.component("my-title", {
    inject: ["name"],
    mounted() {
        console.log(this.$myMethod);
    },
    template: `<div>hello {{name}}<input v-focus /></div>`
})

app.use(myPlugin, { name: "zhz" })
const vm = app.mount("#root");
```
## 5-6 validatorPlugin 数据校验插件开发实例
```js
const validatorPlugin = {
    install(app, options) {
        app.mixin({
            mounted(){
                //this.$options可以选择出app上定义的一切
                let rules = this.$options.rules;

                for (const key in rules) {
                    const item = rules[key];

                    this.$watch(key, (newVal, oldVal) => {
                        const result = item.validate(newVal);
                        if(!result) console.log(item.message);
                    })
                }

            }
        })
    }
};

const app = Vue.createApp({
    data() {
        return {
            name: "zhz",
            age: 22
        }
    },
    rules: {
        age: {
            validate: age => age > 18,
            message: "年轻人，太简单！"
        },
        name: {
            validate: name => name.length < 4,
            message: "老滑头，ID太长！"
        }
    },
    template: `
        <div>
            name:{{name}}; age:{{age}};
        </div>
    `
});

app.use(validatorPlugin);

const vm = app.mount("#root");
```