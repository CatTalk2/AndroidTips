# AndroidTips
android 日常开发总结

### 工具使用

1. 

2. 

3.



### RecyclerView中多Type性能优化原则  

1. 减少`inflate`时间，因为通过xml文件inflate，必然要经过文件的打开关闭，IO操作耗时，尤其当item的复用几率很低的情况下，随着type的增多，这种inflate带来的损耗是相当大的；

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
