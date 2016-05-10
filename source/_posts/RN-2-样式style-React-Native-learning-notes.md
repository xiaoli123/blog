title: '[RN]2.样式style--React-Native learning notes'
date: 2015-11-22 16:27:03
tags:
- React-Native
- Note
categories: [React-Native]
---
# tip:
这是一篇学习 React-Native for Android 的学习笔记，主要上其实是看官方的文档来自己学习 RN，目前的主要是学习android的开发。

😏😏😏😏😏😏😏😏😏😏😏😏😏😏😏😏
# React-Native for Android 样式的使用。
React-Native 编写的应用的样式不是靠css来实现的，而是依赖javascript来为你的应用来添加样式，先不讨论这样做的好处与坏处，因为这个实现方法本身就存在着很多争议，我们主要关注他的样式的语法和特性。
# 声明样式:
```
var styles = StyleSheet.create({
     base: {
          width: 38,
          height: 38,
     },
     background: {
          backgroundColor: '#222222',
     },
     active: {
          borderWidth: 2,
          borderColor: ‘#ff00ff',
     },
});
```
从语法来看：调用了React-Native的一个构造方法，传入一个对象生成style，如果你写过React就应该很熟悉这种写法，和React的React.createCladd()语法是一样的，传入对象的key就相当于类名（我是这么理解），每个类也是一个对象，可以配置各种样式参数，总体来说和CSS的写法差不多，差别上把CSS的命名又“-”连字符改成驼峰写法，然后长度不用加单位“px”，字符串比如色值需要加引号写成字符串。
其实也是和React的行内样式写法的语法一样。

# 样式的使用:
所有的核心组件都支持样式属性
``<View style={style.base} />``
当你需要设置多个属性类的时候，可以传入一个数组
``<View style={[style.base,style.backgroundColor]} />``
在两个样式又冲突的情况下，以右边的值优先，有些情况下可以加一些条件判断样式是否加载，比如，
``<View style={[style.base,this.state.active&&style.active]} />``
你也可以在组件中render样式，然而这种做法不推荐，其实就像一般html页面中行内样式不推荐一样，
```
<View
  style={[styles.base,{width:this.state.width, height:this.state.width*this.state.aspectRatio}]}
/>
```

# 布局 -- flexbox
React-Native 采用flexbox布局方式，flexbox是css3引入的布局模型－－弹性盒子模型，旨在通过弹性的方式对齐和分布容器中的item，使其适应不同的宽度和高度。
在 React-Native 中的flexbox 是css3中flexbox的一个子集，并不支持所有的flexbox属性。
flexbox  布局分为flexbox container 和 flexbox item ：如下图
![](/img/RN_img_4.png)
flexbox 是一个属性的集合，有些是属于container的有些湿属于item的。
可以看下面这幅图：
![](/img/RN_img_5.png)
对于 container 有 main axis（主轴）和cross axis（交叉轴）。main size 和 cross size 分别是container主轴方向的交叉轴方向的宽度，main start 和 main end 分别是主轴的起始和结点，其他同理，container里面包含items。
下面介绍一下属性：
## container的属性：
- flexDirection: ' row ' | ' column '
主轴的方向，水平 | 垂直，默认是 column ，item会按照主轴方向排列。

- flexWrap: ‘ warp ' | ‘ nowrap ’
flexbox 会默认将所有元素基于一行，这个属性表示是否折行。

- alignItems: ‘ flex-start ’ | ’ flex-end ’ | ’ center ’ | ‘ stretch '
表示item在 cross axis 上的对齐方式，基于cross axis的顶部｜基于cross axis的底部｜基于cross axis的中部｜布满整个。

- justifyContent: ‘ flex-start ’ | ‘ flex-end ‘ | ‘ center ‘ | ‘ space-between ‘ | ’space-around'
表示item在 main axis 上的对齐方式，基于主轴开始｜基于主轴结束｜居中｜左右两边对齐，item间隔相等｜每个item两端间隔相等。

## item的属性

- flex: num
item 所占的比例大小。

- alignSelf:‘ flex-start ’ | ’ flex-end ’ | ’ center ’ | ‘ stretch '
它允许项目中当个item和其他itemsyou不一样的对齐方式，会覆盖alignitems的属性。
