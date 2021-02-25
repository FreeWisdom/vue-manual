# 1、hello word
* `{{}}`：插值表达式，在标签中使用数据的语法；
    - `{{ "a" + "b" }}`：之间可以插入 js 表达式；
    - `{{ if(true){console.log} }}`：之间不可以插入 js 语句；
* `Vue.createApp({})`：创建Vue实例；
* `Vue.createApp({}).mount('#root')`：在`#root`节点置入Vue实例；
* `template: <div>hello word</div>`：值为字符串，是在HTML中显示的内容，为Vue创建实例时传入对象的一部分；
```html
<body>
    <div id="root"></div>
    <script>
        Vue.createApp({
            template: `<div>hello word</div>`
        }).mount('#root')
    </script>
</body>
```

# 2、counter
* `data() {return {content: 1}}`：值为函数，为template提供数据变量，也为Vue创建实例时传入对象的一部分；
* `mounted() {}`：值为函数，当页面加载完成，执行该函数中的内容，可以按需写入js操作也为Vue创建实例时传入对象的一部分；
```html
<body>
    <div id="root"></div>
    <script>
        Vue.createApp({
            data() {
                return {
                    content: 1
                }
            },
            mounted() {
                setInterval(() => {
                    this.$data.content += 1;//$data可以省略
                }, 1000)
            },
            template: `<div>{{content}}</div>`
        }).mount('#root')
    </script>
</body>
```

# 3、字符串反转
* `v-on:click="handleBtnClick"`：在DOM上监听点击事件；
    - `v-on:`可以缩写为`@`：`@click="handleBtnClick"`；
    - 动态属性：`@[event]:"handleClick"`，其中 event 需在 data(){return {event: "click"}} 中定义；
    - `<button v-on:click="handleBtnClick">显示/隐藏</button>`；
* `methods: {}`：值为对象，DOM绑定事件的方法可写在methods对象中，
```html
<body>
    <div id="root"></div>
    <script>
        Vue.createApp({
            data() {
                return {
                    content: "hello word!"
                }
            },
            methods: {
                handleBtnClick() {
                    this.content = this.content.split("").reverse().join("");
                }
            },
            template: `
                <div>
                    {{content}}
                    <button v-on:click="handleBtnClick">显示/隐藏</button>    
                </div>
            `
        }).mount("#root")
    </script>
</body>
```

# 4、内容隐藏
* `v-if="show"`：根据当前“data”中"show"的值，在DOM上判断该DOM是否应该展示；
    - `<span v-if="show">{{content}}</span>`
```html
<body>
    <div id="root"></div>
    <script>
        Vue.createApp({
            data() {
                return {
                    show: true,
                    content: "hello word!"
                }
            },
            methods: {
                handleBtnClick() {
                    this.show = !this.show;
                }
            },
            template: `
                <div>
                    <span v-if="show">{{content}}</span>
                    <button v-on:click="handleBtnClick">隐藏/显示</button>    
                </div>
            `
        }).mount("#root")
    </script>
</body>
```

# 5、双向绑定实现todo list
* `v-for="(item, index) of contentList"`：针对contentList[]数组进行循环;
    - `<li v-for="(item, index) of contentList">{{index + 1}}：{{item}}</li>`
* `v-model="inputValue"`：数据和输入框进行双向绑定；
    - `<input v-model="inputValue"/>`
```html
<body>
    <div id="root"></div>
    <script>
        Vue.createApp({
            data() {
                return {
                    inputValue: "",
                    contentList: []
                }
            },
            methods: {
                handleBtnClick() {
                    this.contentList.push(this.inputValue);
                    this.inputValue = "";
                }
            },
            template: `
                <div>
                    <input v-model="inputValue"/>
                    <button v-on:click="handleBtnClick">增加</button>
                    <ul>
                        <li v-for="(item, index) of contentList">{{index + 1}}：{{item}}</li>
                    </ul>   
                </div>
            `
        }).mount("#root")
    </script>
</body>
```

# 6、TODO list组件化
* `v-bind:title="inputValue"`：在标签上某个属性绑定数据内容时使用；
    - `v-bind`可以省略：`:title="inputValue"`；
    - 动态属性：`:[name]:"message"`，其中 name 需在 data(){return {name: "title"}} 中定义；
    - `<button v-bind:title="inputValue">增加</button>`：为 title 属性绑定数据；
    - `<input v-bind:disabled="disable" />`：为 disabled 属性绑定数据；
* `v-bind="params"`:可以传递一个名为params的对象，以减少`:title="inputValue"`在标签中的罗列；
    ```js
    params: {
        xxx: aaa
        ccc: bbb
    }
    ```
* `app.component("xxx", {})`：注册一个名为“xxx”的组件；
* `props: ["content", "index"],`：
    - 在子组件中，通过父组件`v-bind:content="item"`，注册绑定属性“content”，并将该属性传入子组件;
    - 在子组件中再通过`props: ["content", "index"],`注册，组件中就可以使用`{{content}}`；
```js
<body>
    <div id="root"></div>
    <script>
        const app = Vue.createApp({
            data() {
                return {
                    inputValue: "",
                    contentList: []
                }
            },
            methods: {
                handleBtnClick() {
                    this.contentList.push(this.inputValue);
                    this.inputValue = "";
                }
            },
            template: `
                <div>
                    <input v-model="inputValue"/>
                    <button 
                        v-on:click="handleBtnClick"
                        v-bind:title="inputValue"
                    >增加</button>
                    <ul>
                        <todo-item
                            v-for="(item, index) of contentList"
                            v-bind:content="item"
                            v-bind:index="index"
                        />
                    </ul>   
                </div>
            `
        });
        
        app.component("todo-item", {
            props: ["content", "index"],
            template: `
                <li>
                    <span>{{index}}</span>
                    <span>:</span>
                    <span>{{content}}</span>
                </li>
            `
        });
        
        app.mount("#root");
    </script>
</body>
```