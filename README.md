# AndroidTips
android 日常开发总结

### 工具使用

1. 

2. 

3.



### RecyclerView中多Type性能优化原则  

1. 减少`inflate`时间，因为通过xml文件inflate，必然要经过文件的打开关闭，IO操作耗时，尤其当item的复用几率很低的情况下，随着type的增多，这种inflate带来的损耗是相当大的，不仅包括layout中xml，还包括drawable中的xml（getResource().getDrawable()）；

2. 减少View对象的创建，事实上如果只是简单的减少inflate时间，采用系统常规方法new view，一个稍微复杂的item会包含大量的view....而事实上大量的view的new也会消耗大量时间，一个item里面有大量的view初始化耗时，然后又有多种type的item...依然会造成性能问题；

3. 一定要减少过渡绘制和无意义的界面刷新，大量的item过渡绘制和无意义刷新都会造成卡顿；（invalidate）

4. 无论是带有交互或者复杂布局，都应减少重复的onMeasure和onlayout，例如类似weight这些属性一定要少用，不要过多嵌套布局层次，避免relativeLayout相互嵌套等；（requestLayout），必要的时候自己去写对应的onMeasure，onlayout，ondraw

5. 对系统TextView的优化，列表中的item就不应该出现系统的TextView，这货必然是最容易引起卡顿的，你可以自定义TextView，优化它的onmeasure等很多耗时的操作，具体根据场景而定，但一定要自己实现一个textView，不要用系统的，除非你自己实现不了或者有些场景满足不了；

6. 利用android studio自带的性能工具分析定位性能问题，并持续优化；

综上，**做到以上准则**之后，基本上大量的问题能被优化掉，而且已经能PK掉市面上**绝大部分APP**；

> 如果想进一步达到无论不管多少种type，不论各个item多复杂，基本上不会出现复用的情况下仍要性能最佳：

就要牺牲一些代码维护性，走自定义View，如果你对自定义View相当熟悉。那接下来的优化也很好办，只是前期耗点时间；

其实就是**扁平化**，把item从viewGroup—>view降维到view—->drawable;这样就避免了大量的view对象创建的耗时和addView的耗时，而且当item中包含大量view的时候，这种优化效果是显而易见的；n个View—>1个View，就会(n - 1)  * (view.initTime + view.onMeasureTime + addTime); 当然这个过程一定要合理设计，不要过度导致代码难以维护....

暂时以上！

最后，性能优化是个**持续**的过程，首先你要**重视**他，并坚信只要卡顿就必然有优化的空间，这样才能不断校正日常开发中带来性能问题的写法，另外你也要相信我以上所说的.

你也可以认为我所说的都是错的，自己去验证这里面的性能差距，相信你会有所收获的！


### 自定义View相关

1.`onMeasure()`标准写法

if (widthMode == MeasureSpec.EXACTLY) {
            width = widthSize;
        } else {
            //相关
            }


> 在itemType很多的情况下，仅靠RecyclerView自身的复用机制靠谱嘛（item类型越多复用几率越低）？为什么我按正常的开发流程开发完，很难达到满帧(item类型越多越卡出翔)?

以下性能优化都是基于你已经走投无路的情况下，确保代码层面没有人为的代码bug导致的性能问题，否则那就另当别论了；

对RecyclerView的性能优化主要集中在`onCreateViewHolder()`和`onBindViewHolder()`，即尽量减少相应的ViewHolder的创建和绑定时间，不要超过16ms，理论上就可以达到满帧，也就是不再卡顿；


#### `onCreateViewHolder()`: 主要是优化`itemView`的创建和它的`onMeasure()`耗时，方案如下(优化前后效果排序)：

1. 减少过渡绘制：
一定一定一定要减少过渡绘制，优化掉一个list里面每个item都存在的过渡绘制，这是性能优化的第一步，一定要重视这个问题。

2. 减少ixml文件`inflate`时间:
一般情况下itemView都是我们通过xml文件inflate出来的，但是实践发现通过xml文件inflate非常耗时，因为xml的读取是耗时的IO操作，尤其当item的复用几率很低的情况下，随着type的增多，这种inflate带来的损耗是相当大的。所以如果itemType的确足够多，item复用几率很低的情况下，那就必须要优化掉xml的inflate耗时，这里的xml文件不仅包括layout中的xml。还包括`drawable中`的xml；
- 具体做法就是将对应的xml布局转换成java代码，事实上就是去掉了系统层解析xml的过程，即：`new View()`的方式，如果对view比较熟悉的话，这部分也不是很难，只要搞清楚xml中每个节点的属性对应的api即可；

3. 减少View对象的创建:
完成上一步的优化之后，相比于之前的性能已经提升了一大截。但如果你对itemView的设计不够合理，即使采用系统常规方法new view，一个稍微复杂的item会包含大量的view....而大量的view的创建也会消耗大量时间;
- 正确设计你的itemView，对多ViewType能够共用的部分尽量设计成自定义View，减少view的构造和嵌套；
- 采用**预加载**的方式，不要在一开始创建view的时候就将数据绑定可能要出现的View全部初始化，然后再gone掉，而是在bindViewHolder的时候根据所需去初始化；
- 尽量`简化`你的itemView，一是为了提高复用几率，另外一个itemView过于复杂或者子view过多会导致view在布局或者重绘时更加耗时

4. 减少`onMeasure()`耗时：
多次`onMeasure()`也会导致性能问题，所以要尽量减少`RelativeLayout`的相互嵌套，少用`weight`等属性；必要的时候可以用自定义ViewGroup的方式实现；


####  `onBinderViewHolder()`：这里主要是绑定数据， 需要注意要避免一些无意义的界面刷新（invalidate）或者requestLayout;

> 另外
1. 对系统TextView的优化
系统的TextView也是列表中的item性能卡顿的一个主要原因，包括他的创建和onMeasure，(android越高的版本textview的性能也在慢慢变好);所以业务场景不太复杂（例如不用支持富文本）的情况下**自己动手实现一个textview**至少性能是不会低于系统的。类似的优化方案有很多，可参考一些源项目；
2. 利用android studio自带的性能工具分析定位性能问题，并持续优化；


综上，基本上大量的问题能被优化掉，而且已经能PK掉市面上**绝大部分APP**； 如果想进一步达到无论不管多少种type，不论各个item多复杂，基本上不会出现复用的情况下仍要性能最佳：

可能就要牺牲一些代码维护性，走自定义View，如果你对自定义View相当熟悉。那接下来的优化也很好办，只是前期耗点时间。其实就是**扁平化**，把item从(viewGroup—>view)降维到(view—->drawable);这样就避免了大量的view对象创建的耗时和addView的耗时，而且当item中包含大量view的时候，这种优化效果是显而易见的；n个View—>1个View，就会优化掉`(n - 1)  * (view.initTime + view.onMeasureTime + addTime)`; 当然这个过程一定要合理设计，不要过度导致代码难以维护....

暂时以上！

> 最后，性能优化是个**持续**的过程，首先你要**重视**他，并坚信只要卡顿就必然有优化的空间，这样才能不断校正日常开发中带来性能问题的写法。最重要的是用类似的思路找到其他性能问题的症结所在；

彩蛋：猜猜各大厂app性能优化做得怎么样😂...算了，不说整个应用的性能优化了，就猜猜他们的应用首页性能优化做得怎么样😂...算了就只猜猜他们解决过过渡绘制问题嘛....算了，不猜了自己去看吧😂

@题图采集自Chat Interaction

[image:A6385B87-847C-4CFC-A03E-C1B5D7485146-12747-00019843DA8B0731/weixin.jpg]


