# vue与webpack踩坑

项目使用vue来作为后台管理页面的框架，确实方便了许多，不过其中还是踩了些坑。


## webpack-dev-server不会自动刷新

按照之前使用webpack-dev-server，先安装全局的，再安装开发依赖的：

```
npm install webpack-dev-server -g//全局安装，作为工具使用
npm install webpack-dev-server --save-dev//局部安装，作为开发依赖的

```

然后再webpack.config.js中配置devServer:

```
devServer: {
        inline:true
}
```

然后使用命令行`webpack-dev-server --inline`，发现单文件组件修改之后，虽然会自动打包，但是页面不会自动刷新了。

检查官方的(文档)[https://webpack.js.org/configuration/dev-server/#devserver-publicpath-]，才知道。默认情况下，打包的文件是设置在根目录下，我们需要使用publicpath选项来配置打包文件的位置。

由于项目的打包文件是配置在/dist/目录下的，因此修改为：

```
devServer: {
        publicPath: "/dist/",
        inline:true
    }
```

再次运行`webpack-dev-server --inline`，之后修改vue的单文件组件，发现成功刷新页面。

## vue父子组件的数据流

在vue中，父组件与子组件的数据流是单向的。也就是说，父组件的数据通过props传给子组件，而子组件不能修改父组件的数据，也就是说不能修改props的数据。如果要修改，只能通过emit事件通知父组件，让父组件自己修改。

(官文文档有更详细的介绍)[https://cn.vuejs.org/v2/guide/components.html#单向数据流]

不过由于我在项目中使用的是数组中存储对象，而数组使用索引的话，则无法进行数据绑定与视图刷新，需要使用官方订制的数组方法。

因此也就无法使用.sync这种方法，因为这种方法实际上还是一层语法糖，用来简化props+emit的过程。

但是.sync扩展之后的事件响应方法是：`@update:foo="val => bar = val"`，这种方法不会使用数组的方法，因此还得使用最原始的方法，在事件监听函数中使用数组的方法改变响应的数据。



