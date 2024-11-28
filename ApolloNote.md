# ApolloNote



## 前端相关

### react

​		windows的环境

```shell

## 安装node.js，直接在官网下载即可。内部带有了npm
# 安装好后，在终端进行测试
node -v
npm -v
# 出现如下错误
# npm : 无法加载文件 D:\Program Files\nodejs\npm.ps1，因为在此系统上禁止运行脚本
# 在管理员终端运行
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

## 使用npm安装包
# 使用npm安装create-react-app和yarn
npm install -g create-react-app
npm install -g yarn
# 可能出现网络不稳定的情况，可以直接换源
npm config set registry https://registry.npmmirror.com/
```

​		然后构建第一个react脚手架示例

```shell
npx create-react-app my-app
cd my-app
yarn start
```



