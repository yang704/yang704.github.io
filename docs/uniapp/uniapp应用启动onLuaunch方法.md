# uniapp 应用启动onLuaunch方法，改为同步，执行后再执行页面加载<!-- {docsify-ignore-all} -->

## 问题描述
app.vue中onLuaunch中如果有异步方法，返回结果可能会在页面的onLoad之后，为了让onLoad在onLaunch之后执行，使用以下解决方案。

- main.js添加如下代码
  ```javascript
  Vue.prototype.$onLaunched = new Promise(resolve => {
      Vue.prototype.$isResolve = resolve
  } )
  ```
- 在App.vue的onLaunch中增加代码this.$isResolve(),这个方法必须在业务逻辑执行完成后在执行，例如：
  ```javascript
    uni.login({
      success(res) {
          let t = this;
        var wxCode = res.code;
        //获取token
        t.$u.api.getToken({ code: wxCode }).then((res) => {
          if (res.code == 200) {
            t.$store.commit("setToken", res.token);
            t.$isResolve();//发起ajax请求成功后在执行此方法
          }
        });
      },
    });
  ```
- 在页面onLoad中增加代码`await this.$onLaunched`,注意onLoad要添加`async`,否则编译不过去
  ```javascript
   async onLoad(options){
  
       //等待登陆成功
       await this.$onLaunched;
  
       //后续业务逻辑
       ...
   }
  ```