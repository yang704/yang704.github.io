# uniapp 小程序开发环境搭建（使用vue-cli搭建项目uniapp + uview）<!-- {docsify-ignore-all} -->
- 安装node/npm/cnpm
- 安装vue-cli:`npm install -g @vue-cli`;
- 创建uni-app:
  ```javascript
    //my-project为项目名称
    vue create -p dcloudio/uni-preset-vue my-project
  ```
  此时会提示选择模板，选择默认模板
- uView依赖scss,安装scss:
    ```javascript
    npm i node-sass@5 -D
    //报错：npm err! path 
    //解决：
    //删掉package.lock.json
    //清除cache：npm cache clean --force
    //进入下面这个文件夹清除cache
    //路径：C:Users PCAppDataRoamingnpm-cache
    //执行：npm cache clean --force
    //不要用淘宝镜像：npm set registry https://registry.npmjs.org/
    ```
- 安装scss加载器
  ```javascript
  npm i sass-loader@10 -D
  ```
- 安装uView
  ```javascript
  npm install uview-ui
  ```
- 配置
  - main.js
    ```javascript
    //main.js
    import uView from 'uview-ui';
    Vue.use(uView);
    ```
  - 再引入uView的全局Scss主题文件，在项目根目录的uni.scss中引入此文件
    ```css
    //uni.scss
    @import 'uview-ui/theme.scss';
    ```
  - 引入uView基础样式：在App.vue中首行的位置引入，注意给style标签加入lang='scss'
    ```css
    <style lang='scss'>
        @import "uview-ui/index.scss";
    </style>
    ```
  - 配置easycom组件模式
    ```json
    // pages.json
    {
        "easycom": {
            "^u-(.*)": "uview-ui/components/u-$1/u-$1.vue"
        },
        
        // 此为本身已有的内容
        "pages": [
            // ......
        ]
    }    
    ```
  -  sfc.d.ts 文件里添加declare module 'uview-ui'; //选择默认模板（TypeScript）时修改
  
- 安装vuex:
  ```
  npm install vuex --save
  ```
  - main.js添加如下代码
  ```javascript
  //main.js
  import store from './store'
  new Vue({
      el:"#app",
      store
  })
  ```
  - src文件夹下新建文件store/index.js
  ```javascript
  import Vue form 'vue';
  import Vuex from 'vuex';
  Vue.use(Vuex)
  const store = new Vuex.Store({
	state: {
		count: 0
	},
	getters: {},
	mutations: {
		increment(state) {
			state.count++
		}
	},
  	actions: {
    	// 异步方式
  	}
  })
  
  export default store
  ```