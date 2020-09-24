# [移动端Rem适配（基于vue-cli3 ，ui框架用的是vant-ui）](https://www.cnblogs.com/wuqun/p/12084304.html)

介绍
postcss-pxtorem 是一款 postcss 插件，用于将单位转化为 rem
lib-flexible 用于设置 rem 基准值

1、安装lib-flexible（用于设置 rem 基准值）

```
npm i -S amfe-flexible
```

2、在main.js文件中引入lib-flexible

```
import` `'amfe-flexible'
```

3、安装postcss-pxtorem（postcss-pxtorem是一款 postcss 插件，用于将单位转化为 rem）

```
npm install postcss-pxtorem --save-dev
```

4、在public/index.html加入meta标签

```html
<meta name=``"viewport"` `content=``"width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no"``>
```

5、项目配置postcss，有两种方式（注意package.json 和 vue.config.js只选一种配置即可，不要同时配置）

（1）、在package.json中配置

```json
"postcss"``: {``  ``"plugins"``: {``    ``"autoprefixer"``: {},``  ``"postcss-pxtorem"``: {``     ``"rootValue"``: 37.5,``     ``"propList"``: [ ``"*"``]
} } }
```

（2）、在vue.config.js中配置

```javascript
const autoprefixer = require(``'autoprefixer'``);``const pxtorem = require(``'postcss-pxtorem'``);` `module.exports = {``  ``css: {``    ``loaderOptions: {``    ``postcss: {``      ``plugins: [``      ``autoprefixer(),``      ``pxtorem({``        ``rootValue: 37.5,``        ``propList: [``'*'``]``      ``})``      ``]``    ``}``    ``}``  ``}``};
```

这样就完成了适配，需要注意的是，css里面我们就只需要写px，然后会自动转换成rem在浏览器显示，rootValue设置为37.5，之所以设为37.5，是为了引用像vant这样的第三方UI框架，因为第三方框架没有兼容rem，用的是px单位，将rootValue的值设置为设计图宽度（这里为750px）75的一半，即可以1:1还原vant、mint-ui的组件，否则会样式会有变化，例如按钮会变小。既然设置成了37.5 那么我们必须在写样式时，也将值改为设计图（设计图的宽为750px）的一半。

由于在写样式的时候将所有的大小都改成一半这样写起来不方便，所以通常在设置rootValue的时候更多人愿意将其设置为75，这样就可以按设计稿的大小来写样式了，设计稿设计出来是多少像素，写样式的时候就写成多样式。可以这样的话对于vant、mint-ui等第三方框架的组件，他们渲染出来就会出现变小的情况，因为他们是基于375设计的。下面提供了两种方法解决这种第三方框架的组件出现缩小的问题，下面均以vant-ui为例：

1、在配置中设置排除vant-ui相关的组件，让该框架的组件不转为rem，这样的话框架的组件是px作为单位均不自适应了。（这种方式个人不怎么建议）

这里以vue.config.js中的配置为例（package.json的配置类似）

```js
const autoprefixer = require(``'autoprefixer'``);``const pxtorem = require(``'postcss-pxtorem'``);` `module.exports = {``  ``css: {``    ``loaderOptions: {``    ``postcss: {``      ``plugins: [``      ``autoprefixer(),``      ``pxtorem({``        ``rootValue: 75,``        ``propList: [``'*'``],``        ``"selectorBlackList"``:[``"van-"``]  ``//排除vant框架相关组件``      ``})``      ``]``    ``}``    ``}``  ``}``};
```

 

2、动态设置rootValue的大小，即对于vant-ui框架的组件 将rootValue设置成37.5，对于我们自己的750宽度的设计稿的将rootValue设置成75。这样就可以达到所有的均可自适应了

在根目录下新建postcss.config.js文件，同时将vue.config.js 及 package.json文件中的postcss的配置删除。然后在postcss.config.js文字中添加入下内容：

```js
const autoprefixer = require(``'autoprefixer'``);``const pxtorem = require(``'postcss-pxtorem'``);``module.exports = ({ file }) => {``  ``let` `remUnit``  ``if` `(file && file.dirname && file.dirname.indexOf(``'vant'``) > -1) {``    ``remUnit = 37.5``  ``} ``else` `{``    ``remUnit = 75``  ``}``  ``return` `{``    ``plugins: [``      ``autoprefixer(),``      ``pxtorem({``        ``rootValue: remUnit,``        ``propList: [``'*'``],``        ``selectorBlackList: [``'van-circle__layer'``]``      ``})``    ``]``  ``}``}　　
```

好了，到目前为止就算解决了 第三方框架的组件出现缩小 的问题。

最后啰嗦一句：由于`viewport`单位得到众多浏览器的兼容，`lib-flexible`这个过渡方案已经可以放弃使用，不管是现在的版本还是以前的版本，都存有一定的问题。建议大家开始使用`viewport`来替代此方案