## h5项目模板-vue

### todos

1. 工具类

   - [ ] 天天基金获取用户信息的js

   - [ ] 统一封装axios

   - [ ] ipx 下方预留空间

2. 配置类

   - [ ] hash/history
   - [x] sass/less->sass,包含less功能，node-sass安装较慢
   - [x] prettier:继承airbnb方案，遵循组内开发手册
   - [x] 响应式布局方案，px2rem，flexible
   - [x] 打包时去除注释，代码压缩

3. 其他

   - [x] 1px问题

     flexible.js中对支持0.5像素的设备，在document上加了 hairlines 的类名，可以根据该类名自行处理。

     postcss-adaptive在px2rem的基础上提供了将hairlines 下的1px边框转为0.5px。

     ios8以上支持0.5px

      http://dieulot.net/css-retina-hairline

     通过调用 border-top-1px等mixin方法兼容1px边框的问题，见src/assets/styles/_mixins.scss
     
     搬砖自: https://juejin.im/post/5d70a030f265da03a715f3fd 



### 问题

1. rem还是viewport单位

    https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html  flexible



2. 路由钩子，登陆等



### 在vue-cli基础上做了哪些事情

1. browserslist配置：目标浏览器版本

2. eslint，prettier：先shift+alt+f格式化，再保存自动修复eslint格式化错误，再自行修复其他错误

3. vue.config.js: 代码压缩，预处理器等

4. 引入jsconfig.json,添加对alias别名目录的自动提示，防止手动引入出错

   参考 https://code.visualstudio.com/docs/languages/jsconfig

5. css:一些通用的scss的mixin/重置css等

6. 1px问题

7. 响应式方案：flexible+px2rem









#### vue风格指南

[链接](https://cn.vuejs.org/v2/style-guide/index.html#%E4%BC%98%E5%85%88%E7%BA%A7-A-%E7%9A%84%E8%A7%84%E5%88%99%EF%BC%9A%E5%BF%85%E8%A6%81%E7%9A%84-%E8%A7%84%E9%81%BF%E9%94%99%E8%AF%AF](https://cn.vuejs.org/v2/style-guide/index.html#优先级-A-的规则：必要的-规避错误) )

## 代码检查和格式化(eslint,prettier)

> 为了eslint与prettier的格式配置不发生冲突，使用eslint-plugin-prettier（把prettier规则融入eslint中）和eslint-config-prettier(取消所有eslint的格式化报错，只用prettier/prettier，解决冲突)。在eslintrc的extends加上plugin:prettier/recommended。[参考链接]( https://github.com/prettier/eslint-plugin-prettier )
>
> - `eslint-config-prettier` 关闭 `Eslint` 中与 `Prettier` 冲突的选项，只会关闭冲突的选项，不会启用`Prettier`的规则
> - `eslint-plugin-prettier` 启用 `Prettier` 的规则
>
> 【由于冲突以及prettier-eslint有问题，这里不引入prettier规则。用prettier格式化，eslint保存自动修复即可】

#### eslint的配置

见`.eslintrc.js`
[eslint配置详解]( https://eslint.bootcss.com/docs/rules/ )

- 第三方配置：

  - eslint-plugin-vue [参考]( https://eslint.vuejs.org/rules/v-on-style.html )

    使用vue风格指南的essential部分

  - eslint-standard

    参考 https://github.com/standard/eslint-config-standard/blob/master/eslintrc.json






#### prettier配置：
见`.prettierrc`

#### editorconfig

对于非vscode的编辑器，可以用editorconfig进行格式化

[文档]( https://editorconfig.org/ )

参考配置

```
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 4
end_of_line = crlf
insert_final_newline = true
trim_trailing_whitespace = true

```



#### 批量格式化

> 在已有项目/大量文件未格式化的项目中，进行批量的文件格式化。

vue-cli项目中直接`vue-cli-service lint`即可

不用该命令则参考eslint和prettier的文档

## vscode配置

#### 插件

- prettier
- vetur
- eslint

#### settings.json

eslint/vetur/prettier的一些相关配置，可以参考一些，并不一定要全盘照抄，但是这几个插件都要进行配置（目标：eslint不报错即可）

需要注意的是`eslint.validate`中一定要设置vue的autofix，否则自动修复不会生效。

```json
 "eslint.validate": [
    "javascript",
    {
      "language": "vue",
      "autoFix": true
    },
    "javascriptreact",
    "typescript"
  ],
  "eslint.autoFixOnSave": true,
  "eslint.enable": true,
  "vetur.validation.template": false,
  "vetur.format.defaultFormatter.html": "prettier",
  "vetur.format.defaultFormatter.js": "prettier",
  "vetur.validation.script": false,
  "prettier.requireConfig": true,
  "[vue]": {
    "editor.defaultFormatter": "octref.vetur"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "vscode.json-language-features"
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },
  "[less]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
```

