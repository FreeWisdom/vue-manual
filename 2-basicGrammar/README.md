# 一、vue中应用和组件的基础概念
* createApp 表示，创建一个Vue应用，存储到app变量中；
* 传入的参数表示，这个应用最外层的组件应该如何展示；
* MVvM 设计模式：
    - m -> model 数据；
    - v -> view 视图；
    - vm -> viewModel 视图数据连接层；
* `const vm = app.mount("#root");`：vm代表的就是Vue应用的根组件，也是视图数据连接层；
* `console.log(vm.$data.message)`：在根组件上，调用$data，可以操作数据；

# 二、vue中的生命周期函数
* 生命周期函数：某一时刻自动执行的函数
    1. `beforeCreate()`：在实例生成之前，自动执行；
    2. `created()`：在实例生成之后，自动执行；
    - 若参数中含有 `template:'';` 
        - 则将 `template:'';` 编译成 `render()` 函数，再继续执行 `beforeMount()` ；
    - 若参数中没有 `template:'';` 
        - 则将 id="#root" 节点的 innerHTML 当做 `template:'';` 编译成 `render()` 函数，再继续执行 `beforeMount()` ；
    3. `beforeMount()`： template 模板被编译成 render 函数后（组件被渲染到页面前），自动执行；
    4. `mounted()`：template 模板背编译成 render 函数后，且将 render 函数渲染完成后（组件被渲染到页面后），自动执行；
    5. `beforeUpdate()`：数据发生改变时，自动执行；
    6. `updated()`：数据发生改变，且页面更新完，自动执行；
    - 执行`app.unmount()`，在 #id="root" 节点上，销毁Vue应用；
    7. `beforeUnmount()`：执行`app.unmount()`时，且DOM销毁前，自动执行；
    8. `unmounted()`：执行`app.unmount()`后，且DOM销毁后自动执行；

# 三、常用模板语法
* `v-html='message'`：在所属的标签上，以 HTML 方式展示 'message' 变量，避免了 {{message}} 产生的对 message 的转译；
    - `template: "<div v-html='message'></div>"`
* `v-once='message'`：标签使用 v-once 指令后，只在第一次使用该变量时，进行所属标签与该变量绑定，之后就不与该变量绑定了；
    - 降低无用渲染，提高渲染性能；
    - `template: "<div v-once='message'>{{message}}</div>"`
* 动态参数；
    - 动态参数：`@[event]:"handleClick"`，其中 event 需在 data(){return {event: "click"}} 中定义；
    - 动态参数：`:[name]:"message"`，其中 name 需在 data(){return {name: "title"}} 中定义；
* 修饰符：
    - `@click.prevent="handleClick"`：阻止点击的默认行为事件；
```html
template: `<form action="https://www.baidu.com" @click.prevent="handleClick">
    <button type="submit"></button>
</form>`
```

# 四、数据、方法、计算属性、监听器
1.  data ：
    - `vm.$data.message`=`vm.message`
        - 是因为 message 为 data(){return {message:"444"}} 直接返回的根数据；
2.  methods ：
    - ***只要页面重新渲染，就会重新执行里面的所有方法；***
    - methods 里面的函数中若要使用 this；
        - 则不能使用箭头函数；
            - 是因为箭头函数中的 this 不被自动绑定到Vue实例上，而是指向向上级寻找到的对象；
        - 若使用普通函数；
            - this 会被自动绑定到Vue的实例上；
    - methods 里面的方法可以在插值表达式中使用
        - `template: "<div>{{formatString(message)}}</div>"`
        - `methods:{formatString(string) {return string.toUpperCase()}}`
3.  computed：
    - ***只有在计算属性依赖的属性值发生变更时，才会重新计算；***
    - 计算属性依赖的属性值未发生改变，不会重新计算；
    - 故，计算属性有缓存机制，做页面渲染更高效；
    - computed 里面的方法可以在插值表达式中使用
        - `template: "<div>{{total}}</div>"`
        - `computed:{ total(){return this.count * this.price} }`
4.  watch：
    - 可以做异步操作，同步操作时，computed更简洁；
    - 当 data 中的 ‘xxx: yyy’ 数据放生变化时，watch 中相应的 xxx(current, prev){} 函数就会执行；
5. computed分别于methods/watch对比
    - computed对比methods
        - ***当 methods 、 computed 同时可以用的时候，优先选用 computed ，因为有缓存；***
    - computed对比watch
        - ***当 watch 、 computed 同时可以用的时候，优先选用 computed ，因为更加简洁；***

# 五、样式绑定语法
```css
/* 该 css 为以下三个样式举例的通用 css ; */
.red {
    color: red;
}
.green {
    color: green;
}
.yellow {
    color: yellow;
}
.boldest {
    font-weight: 900;
}
.bolder {
    font-weight: 500;
    font-size: x-large;
}
```
## 1、分别用字符串、对象、数组控制样式
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                //字符串控制样式
                classString: "green",
                //对象控制样式
                classObject: {
                    red: true,
                    boldest: true
                },
                //数组控制样式
                classArray: ["red", {"bolder": false}],
            }
        },
        template: `
            <div :class="classString">hello word</div>
            <div :class="classObject">hello word</div>
            <div :class="classArray">hello word</div>
        `
    });
    const vm = app.mount("#root");
</script>
```
## 2、子组件继承父组件样式的2种情况
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                classObject: {
                    red: true,
                    boldest: true
                },
            }
        },
        template: `
            <demo1 :class="classObject"/>
            <demo2 :class="classObject"/>
        `
    });

    app.component("demo1", {
        //若子组件中有2个或2个以上的并列 div ，父组件传递给子组件的样式需要用`:class="$attrs.class"`继承，其他 div 样式不受父组件影响；
        template: `
            <div :class="$attrs.class">子组件A</div>
            <div >子组件B</div>
        `
    });

    app.component("demo2", {
        //若子组件中仅有一个 div ，子组件默认接受父组件样式的继承；
        template: `
            <div>子组件1</div>
        `
    });

    const vm = app.mount("#root");
</script>
```
## 3、行内样式的几种写法
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                styleString: "color: pink",
                styleObject: {
                    color: "pink",
                    fontSize: "60px",
                }
            }
        },
        template: `
            <div style="color: pink">hello word</div>
            <div :style="styleString">hello word</div>
            <div :style="styleObject">hello word</div>
        `
    });
    const vm = app.mount("#root");
</script>
```

# 六、条件渲染
1. `v-if`和`v-show`区别：
    - `v-if`：若为 false ，将 DOM 移除；
    - `v-show`: 若为 false ，将 DOM 的`style="display: none"`；
2. `v-if`、`v-else-if`、`v-else`的使用：
```js
const app = Vue.createApp({
    data() {
        return {
            show: false,
            conditionOne: false,
            conditionTwo: false,
        }
    },
    template: `
        <div v-if="show">hello word</div>
        <div v-show="show">BB word</div>

        <div v-if="conditionOne">if</div>
        <div v-else-if="conditionTwo">else if</div>
        <div v-else>else</div>
    `
});
```

# 七、列表循环渲染
## 1、`v-for`的循环中使用`:key={{xxx}}`提高渲染效率
* 如下例，`v-for`的循环中使用`:key={{item}}`，提高循环渲染的效率
    1. 点击按钮事件后，数据中的数组改变，页面重新渲染，`v-for`遍历的样式跟着改变；
    2. 为循环遍历的每一项都加上`:key="item"`（注：item是唯一的），为每一个经过循环渲染的元素做区分；
    3. Vue会在底层判断相同的 key 值的元素可以复用，不用重新渲染，从而提高了循环渲染的效率；
* 如下例，`v-for`可以直接循环数字，`<div v-for="item in 10">{{item}}</div>`；
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                listArray: ["小明", "男", "19岁" ],
                listObject: {
                    name: "小明",
                    gender: "男",
                    age: "19岁"
                }
            }
        },
        methods: {
            handleClick() {
                this.listArray.push("凡尔赛人")
                this.listObject.job = "student";//注：新版本Vue3才支持改变对象后添加展示内容，旧版本Vue3之前这么展示有问题；
            }
        },
        template: `
            <div>
                <div v-for="(item, index) in listArray" :key="index">{{index}}：{{item}}</div>
                <div v-for="(value, key, index) in listObject">{{index}}-{{key}}：{{value}}</div>
                <button @click="handleClick">哪里人？</button>
                <div v-for="item in 10">{{item}}</div>
            </div>
        `
    });
    const vm = app.mount("#root");
</script>
```
## 2、`v-if`和`v-for`不能在同一标签内使用 及 `<template></template>`占位符的使用
* 若 template 如下，`v-if`与`v-for`在同一标签使用，`v-if`失效，页面仍会展示 `gender: "男"`，这一项
```html
template: `
    <div>
        <div 
            v-for="(value, key, index) in listObject" 
            :key="index" 
            v-if="key !== 'gender'"
        >
            {{index}}-{{key}}：{{value}}
        </div>
    </div>
`
```
* 若要使`v-if`恢复作用，不展示 `gender: "男"`这一项，需要在内层再加一层 div ，将`v-if="key !== gender"`从外层移到内层；
    - 此时，循环渲染出的内容都会多出一层 <div><!--v-if--></div> 的包裹；
    - 可在使用`v-for`的标签上将 <div></div> 改为 <template></template> 占位符；
    - 循环渲染出的内容都会多出一层就会消失 <!--v-if-->
```html
template: `
    <div>
        <template 
            v-for="(value, key, index) in listObject" 
            :key="index" 
        >
            <div v-if="key !== 'gender'">
                {{index}}-{{key}}：{{value}}
            </div>
        </template>
    </div>
`
```

# 八、事件绑定
1. vue中传递参数与事件的方法
2. vue中点击一个按钮绑定多个事件方法
3. 事件修饰符：stop、prevent、self、once、capture、passive
    * @click.self 修饰符，只点击本事件才触发
    * @click.once 修饰符，事件仅触发一次
    * @click.stop 修饰符，阻止事件冒泡
```html
<body>
    <div id="root">
    </div>
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    counter: 0
                }
            },
            methods: {
                handleClick(num, event) {
                    console.log(event.target, 0);
                    this.counter += num;
                },
                handleClick1(num, event) {
                    console.log("click1")
                },
                handleDivClick() {
                    alert("div")
                }
            },
            template: `
                <div @click.self.once="handleDivClick">
                    {{counter}}
                    <button @click.stop="handleClick(2, $event), handleClick1()">button</button>
                </div>
            `
        });
        
        app.mount("#root");
    </script>
</body>
```

4. 按键修饰符：enter、tab、delete、esc、up、down、left、right……
```html
<body>
    <div id="root">
    </div>
    <script>
        const app = Vue.createApp({
            methods: {
                handelKeyDown() {
                    console.log("keydown");
                },
                handelLeft() {
                    console.log("left");
                }
            },
            template: `
                <div>
                    <input @keydown.delete="handelKeyDown"/>
                </div>
            `
        });
        
        app.mount("#root");
    </script>
</body>
```

5. 鼠标修饰符：middle、left、right……
```html
<body>
    <div id="root">
    </div>
    <script>
        const app = Vue.createApp({
            methods: {
                handelLeft() {
                    console.log("left");
                }
            },
            template: `
                <div>
                    <div @click.left="handelLeft">11111</div>
                </div>
            `
        });
        
        app.mount("#root");
    </script>
</body>
```

6. 精确修饰符：exact……
```html
<body>
    <div id="root">
    </div>
    <script>
        const app = Vue.createApp({
            methods: {
                handelLeft() {
                    console.log("left");
                }
            },
            template: `
                <!-- vue中精确修饰符：exact……-->
                <div>
                    <div @click.ctrl.exact="handelLeft">11111</div>
                </div>
            `
        });
        
        app.mount("#root");
    </script>
</body>
```
7. 双向绑定修饰符：lazy、number、trim
* lazy:失焦后进行双向绑定;
* number:将绑定之后的数据转换成数字;
* trim:将绑定之后的string前后的空格清除，字符串中间的空格不能清除;
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                message: "",
                numberMessage: 123,
                spaceMessage: "hey"
            }
        },
        template: `
            <div>
                {{message}}
                <input v-model.lazy="message"/>
            </div>
            <div>
                {{numberMessage}}
                {{typeof numberMessage}}
                <input v-model.number="numberMessage" type="number"/>
            </div>
            <div>
                {{spaceMessage}}
                <input v-model.trim="spaceMessage"/>
            </div>
        `
    });
    
    app.mount("#root");
</script>
```

# 九、表单中双向绑定指令的使用
1. input、textarea的双向绑定
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                message: "hey"
            }
        },
        template: `
            <div>
                {{message}}
                <input v-model="message"/>
                <textarea v-model="message"/>
            </div>
        `
    });
    
    app.mount("#root");
</script>
```
2. checkbox的双向绑定
* 单个checkbox
* 多个checkbox
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    singleCheckBox: false,
                    checkBoxes: []
                }
            },
            template: `
                <div>
                    {{singleCheckBox}}
                    <input type="checkbox" v-model="singleCheckBox"/>
                    
                    {{checkBoxes}}
                    红<input type="checkbox" v-model="checkBoxes" value="红"/>
                    黄<input type="checkbox" v-model="checkBoxes" value="黄"/>
                    蓝<input type="checkbox" v-model="checkBoxes" value="蓝"/>
                    绿<input type="checkbox" v-model="checkBoxes" value="绿"/>
                </div>
            `
        });
    
        app.mount("#root");
    </script>
    ```
* 单个checkbox时自定义 true-value & false-value
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    checkBoxValue: "假",
                }
            },
            template: `
                <div>
                    {{checkBoxValue}}
                    <input type="checkbox" v-model="checkBoxValue" true-value="真" false-value="假"/>
                </div>
            `
        });
        
        app.mount("#root");
    </script>
    ```

3. radio的双向绑定
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                radioData: "中"
            }
        },
        template: `
            <div>
                {{radioData}}
                大<input type="radio" v-model="radioData" value="大">
                中<input type="radio" v-model="radioData" value="中">
                小<input type="radio" v-model="radioData" value="小">
            </div>
        `
    });
    
    app.mount("#root");
</script>
```
4. select的双向绑定
* select单选
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    selectValue: ""
                }
            },
            template: `
                <div>
                    <select v-model="selectValue">
                        <option value="" disabled>请选择</option>
                        <option value="a">a</option>
                        <option value="b">b</option>
                        <option value="c">c</option>
                    </select>
                </div>
            `
        });
        
        app.mount("#root");
    </script>
    ```
* select多选
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    selectValues: [],
                }
            },
            template: `
                <div>
                    {{selectValues}}
                    <select v-model="selectValues" multiple>
                        <option value="a">a</option>
                        <option value="b">b</option>
                        <option value="c">c</option>
                    </select>
                </div>
            `
        });
        
        app.mount("#root");
    </script>
    ```
* v-for使option循环
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    option: [{
                        text: "a",
                        value: "a"
                    },{
                        text: "b",
                        value: "b"
                    },{
                        text: "c",
                        value: "c"
                    }],
                    optionA: [],
                }
            },
            template: `
                <div>
                    {{optionA}}
                    <select v-model="optionA" multiple>
                        <option v-for="item in option" :value="item.value">{{item.text}}</option>
                    </select>
                </div>
            `
        });
        
        app.mount("#root");
    </script>
    ```
* 利用select进行数据转化
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    options: [{
                        text: "a",
                        value: {value: "a"}
                    },{
                        text: "b",
                        value: {value: "b"}
                    },{
                        text: "c",
                        value: {value: "c"}
                    }],
                    optionsA: [],
                }
            },
            template: `
                <div>
                    {{optionsA}}
                    <select v-model="optionsA" multiple>
                        <option v-for="item in options" :value="item.value">{{item.text}}</option>
                    </select>
                </div>
            `
        });
        
        app.mount("#root");
    </script>
    ```

