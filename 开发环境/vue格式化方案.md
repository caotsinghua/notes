## vue格式化方案

#### vue风格指南

[链接]( [https://cn.vuejs.org/v2/style-guide/index.html#%E4%BC%98%E5%85%88%E7%BA%A7-A-%E7%9A%84%E8%A7%84%E5%88%99%EF%BC%9A%E5%BF%85%E8%A6%81%E7%9A%84-%E8%A7%84%E9%81%BF%E9%94%99%E8%AF%AF](https://cn.vuejs.org/v2/style-guide/index.html#优先级-A-的规则：必要的-规避错误) )

## 代码检查和格式化(eslint,prettier)

> 为了eslint与prettier的格式配置不发生冲突，使用eslint-plugin-prettier（把prettier规则融入eslint中）和eslint-config-prettier(取消冲突)。在eslintrc的extends加上plugin:prettier/recommended。[参考链接]( https://github.com/prettier/eslint-plugin-prettier ) 
>
> 【由于brace-style冲突，这里不引入，用prettier格式化，eslint保存自动修复即可】

#### eslint的配置

[eslint配置详解]( https://eslint.bootcss.com/docs/rules/ )

- 第三方配置：

  - eslint-plugin-vue [参考]( https://eslint.vuejs.org/rules/v-on-style.html )

    使用vue风格指南的essential部分
    
  - eslint-standard

    参考 https://github.com/standard/eslint-config-standard/blob/master/eslintrc.json 

- 自定义配置项

  > brace-style  强制在代码块中使用一致的大括号风格 

  ```js
module.exports = {
      root: true,
      extends: ['plugin:vue/essential', '@vue/standard'],
      env: {
          browser: true,
          es6: true
      },
      plugins: ['vue'],
      rules: {
          'vue/no-use-v-if-with-v-for': 'off',
          'generator-star-spacing': 'off',
          // allow debugger during development
          'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off',
          'vue/no-parsing-error': [
              2,
              {
                  'x-invalid-end-tag': false
              }
          ],
          'no-undef': 'off',
          camelcase: 'off',
          eqeqeq: 'off',
          'brace-style': ['error', 'stroustrup'],
          indent: [
              'error',
              4,
              {
                  SwitchCase: 1
              }
          ],
          'no-mixed-spaces-and-tabs': 'error',
          semi: ['error', 'always'],
          'space-before-function-paren': 'off',
          'require-await': 'error',
          'space-before-blocks': 'error'
      },
      parserOptions: {
          parser: 'babel-eslint'
      }
  };
  
  
  ```
  
  

#### prettier配置：

```json
{
    "printWidth": 100, // 每行100字符时换行
    "tabWidth": 4, // 缩进4空格
    "useTabs": false, // 不用tab缩进
    "singleQuote": true, // 使用单引号
    "semi": true, // 每行带分号;
    "quoteProps": "as-needed", // 对象的key带''号时按需采用（如{'a-1':1,b:2}）
    "jsxSingleQuote": false, // jsx不使用单引号
    "trailingComma": "none", // 数组或对象元素末尾不带逗号
    "bracketSpacing": true, // 对象中大括号前后带空格{ a: 1 }
    "jsxBracketSameLine": false, // jsx中多行属性时，第一个>号另起一行。参考文档
    "arrowParens": "avoid", // 箭头函数单参数不带括号。a=>a,不是(a)=>a
    "htmlWhitespaceSensitivity": "css", // html中的空格处理，以html元素dispay形式为准。参考文档。
    "vueIndentScriptAndStyle": false, // vue中的script和style中代码是否缩进
    "endOfLine": "lf", // 末尾行时是 \n \r \n\r auto ，使用\n，具体见文档。
    "overrides": [ // 配置解析器
        {
            "files": ".prettierrc",
            "options": {
                "parser": "json"
            }
        },
        {
            "files": "*.vue",
            "options": {
                "parser": "vue"
            }
        },
        {
            "files": "*.html",
            "options": {
                "parser": "html"
            }
        }
    ]
}

```

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
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

```



#### 批量格式化

> 在已有项目/大量文件未格式化的项目中，进行批量的文件格式化。

vue-cli项目中直接`vue-cli-service lint`即可

## vscode配置

#### 插件

- prettier
- vetur
- eslint

#### settings.json

eslint/vetur/prettier的一些相关配置，可以参考一些，并不一定要全盘照抄，但是这几个插件都要进行响应的配置（目标：eslint不报错即可）

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

