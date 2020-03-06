plugin:babel-plugin-module-resolver

```
yarn add babel-plugin-module-resolver -D
```
配置babel
```
plugins: [
    [
      require.resolve('babel-plugin-module-resolver'),
      {
        root: ['./src'],
        extensions: ['.tsx', '.ts', '.js'],
        alias: {
          '@': './',
        },
      },
    ],
  ],
```

tsconfig配置
```
    "baseUrl": "./",
    "paths": {
        "@/*":["./src/*"]
    }
```

