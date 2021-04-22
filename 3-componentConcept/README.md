# 组件理念
## 一、组件的定义及复用性，局部组件和全局组件

### 1、组件具备复用性，而组件中的数据是独立的；
### 2、app.component()定义的**全局组件**
* 随时随地可以使用，但会一直占用内存，性能较低，使用简单；
* 建议名字为小写字母单词，中间横线间隔；
```html
<script>
    const app = Vue.createApp({
        template: `
            <div>
                <counter-parent/>
                <counter/>
                <counter/>
            </div>
        `
    });
    
    app.component("counter-parent", {
        template: `<counter/>`
    });

    app.component("counter", {
        data() {
            return {
                count: 1
            }
        },
        methods: {
            handelClick() {
                this.count += 1;
            }
        },
        template: `
            <div @click="handelClick">{{count}}</div>
        `
    });

    app.mount("#root");
</script>
```
### 3、const定义的**局部组件**
* 定义后，要注册才能用，之间需要做一个名字和组件间的映射对象，若不写映射，Vue底层会自动尝试映射；
* 使用起来比较麻烦，但性能较高；
* 建议大写字母开头，驼峰命名；
```html
<script>
    const ShowText = {
        template: `<div>hello word</div>`
    };

    const AddCounter = {
        data() {
            return {
                count: 99
            }
        },
        methods: {
            handleClick() {
                this.count += 1;
            }
        },
        template: `<div @click="handleClick">{{count}}</div>`
    }

    const app = Vue.createApp({
        components: {
            ShowText,
            'add-counter':AddCounter
        },
        template: `
            <div>
                <add-counter/>
                <show-text/>
            </div>
        `
    });

    app.mount("#root");
</script>
```

## 二、组件间传值及传值校验
### 1、组件间静态传参
* 会传递一个固定的写死的字符串；
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                num: 2222
            }
        },
        template: `
            <div>
                <counter value="hey你好！"/>
            </div>
        `
    });

    app.component("counter", {
        props: ["value"],
        template: `
            <div>{{value}}</div>
        `
    });

    app.mount("#root");
</script>
```
### 2、组件间动态传参
* 会传递data中的灵活的数据，可以是各种类型；
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                num: 2222
            }
        },
        template: `
            <div>
                <static v-bind:value="num" />
            </div>
        `
    });

    app.component("static", {
        props: ["value"],
        template: `
            <div>{{typeof value}}:{{value}}</div>
        `
    });

    app.mount("#root");
</script>
```
### 3、组件间传参校验
* 校验类型:String/Number/Boolean/Function/Object/Array/Symbol；
    - 子组件中通过`props:{value1: Function}`校验:
    ```js
    props: {
        value1: Function,
        value2: String
    },
    ```
* 校验必填：required
    ```js
    value3: {
        type: Number,
        required: true
    },
    ```
* 校验默认值：default
    ```js
    value4: {
        type: Number,
        required: false,
        default: 6666666
    },
    ```
* 校验定制规则
    ```js
    value5: {
        type: Number,
        required: true,
        default: 666,
        validator: function(data) {
            return data < 777
        }
    }
    ```
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                value1: () => {alert("456")},
                value2: "8888",
                value3: 9999999,
                // value4: 8888888
                value5: 8888
            }
        },
        template: `
            <div>
                <test :value1="value1" :value2="value2" :value5="value5"/>
            </div>
        `
    });

    app.component("test", {
        props: {
            value1: Function,
            value2: String,
            value3: {
                type: Number,
                required: true
            },
            value4: {
                type: Number,
                required: false,
                default: 6666666
            },
            value5: {
                type: Number,
                required: true,
                default: 666,
                validator: function(data) {
                    return data < 777
                }
            }
        },
        methods: {
            handleClick() {
                console.log("123");
                this.value1();
            }
        },
        template: `
            <div @click="handleClick">{{typeof value1}}:{{value1}}</div>
            <div>{{typeof value2}}:{{value2}}</div>
            <div>{{typeof value3}}:{{value3}}</div>
            <div>{{typeof value4}}:{{value4}}</div>
            <div>{{typeof value5}}:{{value5}}</div>
        `
    });

    app.mount("#root");
</script>

<!-- 
    由于上方有两个参数校验失败，故浏览器会产生两个警告⚠️：
    [Vue warn]: Missing required prop: "value3" 
    at <Test value1=fn<value1> value2="8888" value5=8888 > 
    at <App>
    [Vue warn]: Invalid prop: custom validator check failed for prop "value5". 
    at <Test value1=fn<value1> value2="8888" value5=8888 > 
    at <App>
-->
```

## 三、单向数据流的理解
* 单项数据流中，子组件可以使用父组件传递过来的数据，但绝对不可以修改传递过来的数据；
    - 若修改传递过来的数据，会在子组件复用中造成耦合，子组件的复用就不会独立了；
* 若要达到业务效果，使每个子组件独立不耦合；
    - 可以在子组件中设置子组件的data，将父组件传递过来的值对子组件中data初始化，此时子组件可以修改自身data中被父组件数据初始化的值，以达到业务效果；
```html
<script>
    const app = Vue.createApp({
        data() {
            return {
                num: 0,
            }
        },
        template: `
            <div>
                <counter :value="num" />
            </div>
        `
    });

    app.component("counter", {
        props: ["value"],
        data() {
            return {
                myValue: this.value
            }
        },
        methods: {
            handleClick() {
                this.myValue += 1;
            }
        },
        template: `
            <div @click="handleClick">{{myValue}}</div>
        `
    });

    app.mount("#root");
</script>
```
## 四、Non-Props 属性是什么
1. Attribute 继承
    - 父组件给子组件传递参数，子组件不使用props属性接收参数；
    - 底层会把父组件传递给子组件的参数，放在子组件最外层的DOM标签上，参数变成该标签的属性；
```js
app.component('date-picker', {
  template: `
    <div class="date-picker">
      <input type="datetime" />
    </div>
  `
})
```
```html
<!-- 具有非prop attribute的Date-picker组件-->
<date-picker data-status="activated"></date-picker>

<!-- 渲染 date-picker 组件 -->
<div class="date-picker" data-status="activated">
  <input type="datetime" />
</div>
```
2. 禁用 Attribute 继承
    - `inheritAttrs: false,`如果不希望组件的根元素继承 attribute，你可以在组件的选项中设置 inheritAttrs: false；
        - 底层不会将该参数作为子组件最外层标签的属性；
    - `v-bind="$attrs"`如果需要将所有非 prop attribute 应用于 input 元素而不是根 div 元素，则可以使用 v-bind 缩写来完成；
```js
app.component('date-picker', {
  inheritAttrs: false,
  template: `
    <div class="date-picker">
      <input type="datetime" v-bind="$attrs" />
    </div>
  `
})
```
```html
<!-- Date-picker 组件 使用非 prop attribute -->
<date-picker data-status="activated"></date-picker>

<!-- 渲染 date-picker 组件 ---- data status attribute 将应用于 input 元素-->
<div class="date-picker">
  <input type="datetime" data-status="activated" />
</div>
```
3. 多个根节点上的 Attribute 继承
    - 与单个根节点组件不同，具有多个根节点的组件不具有自动 attribute 回退行为。如果未显式绑定 $attrs，将发出运行时警告;
    - `:message1="$attrs.message1"`如需要对父组件的某个属性单独使用在子组件的某个元素中，则可以使用`:message1="$attrs.message1"`；
    - 生命周期created()中可以用`this.$attrs`监听父组件传递给子组件的参数；
```html
<script>
    const app = Vue.createApp({
        template: `
            <div>
                <counter message="hello" message1="hello1" />
            </div>
        `
    });

    app.component("counter", {
        created() {
            console.log(this.$attrs) // { onChange: () => {}  }
        },
        inheritAttrs: false,
        template: `
            <div :message1="$attrs.message1">Counter</div>
            <div v-bind="$attrs">Counter</div>
            <div>Counter</div>
        `
    });

    app.mount("#root");
</script>
```

## 五、父子组件间如何通过事件进行通信
1. 子组件使用`this.$emit("add", 2)`，父组件template中使用`@add="handleAdd`，实现子组件向父组件通信
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    count: 1
                }
            },
            methods: {
                handleAdd(param) {
                    this.count += param;
                }
            },
            template: `
                <div>
                    <counter :count="count" @add="handleAdd"/>
                </div>
            `
          
    });
    
        app.component("counter", {
            props:["count"],
            // emits: ["add"],
            emits: {
                add: (count) => {
                    if(count < 0) {
                        return true;
                };
    
                    return false;
                }
            },
            methods: {
                handleClick() {
                    this.$emit("add", 2)
                }
            },
            template: `
                <div @click="handleClick">{{count}}</div>
            `
    });
    
        app.mount("#root");
    </script>
    ```
    
2. `emits`的使用
    - 子组件中使用`emits:["add"]`记录组件发出的事件；
    - 子组件中使用`emits:{add: (value) => {}}`针对组件抛出的事件进行检验，若检验不通过返回false会触发警告；
    ```js
    app.component("counter", {
        props:["count"],
        // emits: ["add"],
        emits: {
            add: (count) => {
                if(count < 0) {
                    return true;
                };
                //若返回false会在控制台warning警告⚠️，事件参数校验不通过
                return false;
            }
        },
        methods: {
            handleClick() {
                this.$emit("add", 2)
            }
        },
        template: `
            <div @click="handleClick">{{count}}</div>
        `
    });
    ```
    
3. `v-model`在父子通信中的使用（简化代码）

    * 一定要在子组件使用props接收；

    ```html
    <script>
        // 使用v-model简化上述代码
        const app = Vue.createApp({
            data() {
                return {
                    count: 0,
                    count1: 0
                }
            },
            template: `
                <div>
                    <counter v-model:count="count" v-model:count1="count1"/>
                </div>
            `
        });
    
        app.component("counter", {
            props:["count", "count1"],
            methods: {
                handleClick() {
                    this.$emit("update:count", this.count + 1)
                },
                handleClick1() {
                    this.$emit("update:count1", this.count1 + 9)
                }
            },
            template: `
                <div @click="handleClick">{{count}}</div>
                <div @click="handleClick1">{{count1}}</div>
            `
        });
    
        app.mount("#root");
    </script>
    ```
## 六、组件间双向绑定高级内容
1. `v-model`中修饰符的使用

    ```html
    <script>
        // 使用v-model修饰符的使用
        const app = Vue.createApp({
            data() {
                return {
                    count: "a",
                }
            },
            template: `
                <div>
                    <counter v-model.uppercase="count"/>
                </div>
            `
        });
    
        app.component("counter", {
            props: {
                "modelValue": String,
                // 记住默认写法
                "modelModifiers": {
                    default: () => ({})
                }
            },
            methods: {
                handleClick() {
                    let newValue = this.modelValue + "b";
                    // modelModifiers 中接收 v-model.uppercas 中的 uppercas； 
                    if(this.modelModifiers.uppercase) {
                        newValue = newValue.toUpperCase()
                    }
                    this.$emit("update:modelValue", newValue)
                },
            },
            template: `
                <div @click="handleClick">{{modelValue}}</div>
            `
        });
    
        app.mount("#root");
    </script>
    ```

## 七、使用插槽和具名插槽解决组件内容传递问题
>插槽存在的意义:若组件中传递标签、DOM，使用props传递较为麻烦，故产生了插槽；

1. 插槽中使用数据的作用域的问题
    - `<slot></slot>`标签上不可以绑定事件，需要在外层包裹`<span></span>`辅助绑定事件；
    - `<slot>默认值设定</slot>`该标签中间的内容为插槽的默认值；
    - 父模板中调用的数据属性，使用父模板的数据；
    - 子模板中调用的数据属性，使用子模板的数据；
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    text: "submit"
                }
            },
            template: `
                <my-form>
                    <div>{{text}}</div>   
                </my-form>
                <my-form>
                    <button>{{text}}</button>   
                </my-form>
                <my-form>
                </my-form>
            `
        });

        app.component("my-form", {
            methods: {
                handelClick() {
                    console.log(99999)
                }
            },
            template: `
                <div>
                    <input />
                    <span @click="handelClick">
                        <slot>默认值设定</slot>
                    </span>
                </div>
            `
        });

        app.mount("#root");
    </script>
    ```
2. 具名插槽的使用方法
    - 父组件中的`<template v-slot:xxx>head</template>`，
      - 必须写在 `<template/>`标签中；
      - 必须作用在子组件的 template 中对应的 `<slot name="xxx"></slot>`上；
    - `v-slot:xxx`可以简写为`#xxx`;
    ```html
    <script>
        // 具名插槽的使用
        const app = Vue.createApp({
            template: `
                <layout>
                    <template v-slot:header>head</template>
                    <template #footer>footer</template>
                </layout>
        `
        });
    
        app.component("layout", {
            template: `
                <div>
                    <slot name="header"></slot>
                    <div>content</div>
                    <slot name="footer"></slot>
                </div>
        `
        });
    
        app.mount("#root");
    </script>
    ```

## 八、作用域插槽
1. 解决什么问题：
    
    - 当子组件渲染的内容由父组件决定的时候，通过作用域插槽实现，让父组件调用子组件item数据；
2. 执行流程：
    - 父组件调用`list`子组件；
    - `<div>{{item}}</div>`作为模板传递给子组件的slot；
    - 子组件中通过`v-for`将数据`item`传递给父组件；
    - 父组件通过`v-slot="{item}"`接收到item；
    - 在父组件的模板中就可以使用子组件传递过来的`item`值；
    ```html
    <script>
        // 作用域插槽的使用
        const app = Vue.createApp({
            template: `
                <!-- 使用解构可以省略slotProps
                <list v-slot="slotProps">
                     <div>{{slotProps.item}}</div>
                </list>
                -->
                <list v-slot="{item}">
                    <!-- 作为slot -->
                    <div>{{item}}</div>
                </list>
                
            `
        });

        app.component("list", {
            data() {
                return {
                    list: [1, 2, 3]
                }
            },
            template: `
                <div>
                    <slot v-for="item in list" :item="item"></slot>
                </div>
            `
        });

        app.mount("#root");
    </script>
    ```
## 九、动态组件和异步组件
1. 动态组件
    - 根据数据的变化，结合`<component :is="currentItem"></component>`标签，实现动态切换组件；
    - 使用`<keep-alive></keep-alive>`包裹的`<component :is="currentItem"></component>`，失活的组件将会被缓存；
    - data中`currentItem: "checkbox-item"`的`"checkbox-item"`必须与组件同名；
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    currentItem: "checkbox-item"
                }
            },
            methods: {
                handleClick() {
                    this.currentItem === "checkbox-item" ? this.currentItem = "text-item" : this.currentItem = "checkbox-item"
                }
            },
            template: `
            <!--
                <checkbox-item v-show="currentItem === 'checkbox-item'"/>
                <text-item v-show="currentItem === 'text-item'"/>
            -->
                <keep-alive>
                    <component :is="currentItem"></component>
                </keep-alive>
                <button @click="handleClick">切换</button>
            `
        });

        app.component("checkbox-item", {
            template: `<input type="checkbox"/>`
        });

        app.component("text-item", {
            template: `<input type="text"/>`
        });

        app.mount("#root");
    </script>
    ```
2. 异步组件
    - 异步执行某些组件
    ```html
    <script>
        const app = Vue.createApp({
            template: `
                <async-item/>
            `
        });

        app.component("async-item", Vue.defineAsyncComponent(() => {
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    resolve({
                        template: `4 second later！`
                    })
                }, 4000);
            })
        }))

        app.mount("#root");
    </script>
    ```

## 十、基础语法知识点查缺补漏 
1. 属性传递使用 `content-abc`命名，属性接收使用 `contentAbc`命名
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    num: 2222,
                }
            },
            template: `
                <div>
                    <static :num-value="num" />
                </div>
            `
        });

        app.component("static", {
            props: ["numValue"],
            // props: ["num-value"],
            template: `
                <div>{{typeof num-value}}:{{num-value}}</div>
                <div>{{typeof numValue}}:{{numValue}}</div>
            `
        });

        app.mount("#root");
    </script>
    <!--
        <div>{{typeof num-value}}:{{num-value}}</div>:显示NaN
        <div>{{typeof numValue}}:{{numValue}}</div>:正确
    -->
    ```
2. `v-once`让某个元素标签只渲染一次
    - 但背后真实数据可能跟渲染的这一次不一样了；
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    count: 0,
                }
            },
            template: `
                <div @click="count += 1" v-once>
                    {{count}}
                </div>
            `
        });

        const vm = app.mount("#root");
    </script>
    ```
3. `ref`的使用
    - 可以获取DOM节点的引用；
    - 可以获取组件的引用,从而可以调用子组件中的方法；
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    count: 0,
                }
            },
            mounted() {
                this.$refs.count.innerHTML = "随后添加";
                this.$refs.common.sayHello();
            },
            template: `
                <div ref="count">
                    {{count}}
                </div>
                <common-item ref="common"/>
            `
        });

        app.component("common-item", {
            methods: {
                sayHello() {
                    alert("hello")
                }
            },
            template: `
                <div>hello word</div>
            `
        })

        const vm = app.mount("#root");
    </script>
    ```
4. `Provide / Inject`的使用
    - 父组件有一个 provide 选项来提供数据，子组件有一个 inject 选项来开始使用这些数据；
    - 但默认情况下，provide/inject 绑定是一次性的，并不是响应式的，即：例中点击事件不会改变页面中的数字，但实际相关变量count早已改变；
    - 注：在Vue3中有该问题的解决方案；
    ```html
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    count: 0,
                }
            },
            provide() {
                return {
                    count: 1
                }
            },
            template: `
                <div>
                    <child :count="count"/>
                    <button @click="count += 1">add</button>
                </div>
            `
        });

        app.component("child", {
            template: `
                <child-child />
            `
        })

        app.component("child-child", {
            inject:["count"],
            template: `<div>{{count}}</div>`
        })

        const vm = app.mount("#root");
    </script>
    ```

