# 创建一个package.json文件
npm init

# 查看当前包的安装路径
npm root

# 安装包
npm install ModuleName

# 全局安装
npm install ModuleName -g

# 安装包的同时，将信息写入到package.json中的 dependencies 配置中
npm install ModuleName --save

# 安装包的同时，将信息写入到package.json中的 devDependencies 配置中
npm install ModuleName --save-dev

# 配置安装模式
npm set global=true # 全局模式
npm set global=false # 本地模式

# 查看当前安装模式，将会得到一个布尔值
npm get global

# 查看npm的版本
npm -v

# 查看所有全局安装的包
npm ls -g

# 查看本地项目中安装的包
npm ls

# 查看包的 package.json文件
npm view ModuleName

# 查看包的依赖关系
npm view ModuleName dependencies

# 查看包的源文件地址
npm view ModuleName repository.url

# 查看包所依赖的node版本
npm view ModuleName engines

# 查看npm所使用的文件夹
npm help folders

# 更改包内容后进行重建
npm rebuild ModuleName

# 检查包是否已经过时，此命令会列出所有已经过时的包，可以及时进行包的更新
npm outdated

# 更新当前目录下node_modules子目录里的包
npm update ModuleName

# 全局更新
npm update ModuleName

# 卸载包
npm uninstall ModuleName

# 访问npm的json文件，此命令将会打开一个网页
npm help json

# 发布一个包的时候，需要检验某个包名是否存在
npm search ModuleName

# 清空npm缓存
npm cache clear

# 撤销自己发布过的某个版本代码
npm unpublish <package> <version>

# 使用淘宝镜像
npm install -g cnpm --registry=https://registry.npm.taobao.org

