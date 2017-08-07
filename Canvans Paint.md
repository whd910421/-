自定义绘制的方式是重写绘制方法，其中最常用的是 onDraw()
绘制的关键是 Canvas 的使用 
Canvas 的绘制类方法： drawXXX() （关键参数：Paint）
Canvas 的辅助类方法：范围裁切和几何变换
可以使用不同的绘制方法来控制遮盖关系

在onDraw()方法中进行复写，唯一需要注意的是别漏写了 super.onDraw()。
	Canvas 类下的所有 draw- 打头的方法，例如 drawCircle() drawBitmap()。

	Paint 类的几个最常用的方法。具体是： 
	Paint.setStyle(Style style) 设置绘制模式 
	ps：style 具体来说有三种： FILL,  STROKE 和 FILL_AND_STROKE 。FILL 是填充模式，STROKE 是画线模式（即勾边模式），FILL_AND_STROKE 是两种模式一并使用：既画线又填充。
	Paint.setColor(int color) 设置颜色
	Paint.setStrokeWidth(float width) 设置线条宽度
	Paint.setTextSize(float textSize) 设置文字大小
	Paint.setAntiAlias(boolean aa) 设置抗锯齿开关

Canvas
Canvas.drawColor(@ColorInt int color) 颜色填充 这类颜色填充方法一般用于在绘制之前设置底色，或者在绘制之后为界面设置半透明蒙版。
类似的方法还有 drawRGB(int r, int g, int b) 和 drawARGB(int a, int r, int g, int b)

drawCircle(float centerX, float centerY, float radius, Paint paint) 画圆
drawRect(float left, float top, float right, float bottom, Paint paint) 画矩形
drawPoint(float x, float y, Paint paint) 画点
点的大小可以通过 paint.setStrokeWidth(width) 来设置；点的形状可以通过 paint.setStrokeCap(cap) 来设置：ROUND 画出来是圆形的点，SQUARE 或 BUTT 画出来是方形的点。
drawOval(float left, float top, float right, float bottom, Paint paint) 画椭圆
drawLine(float startX, float startY, float stopX, float stopY, Paint paint) 画线
drawLines(float[] pts, int offset, int count, Paint paint) / drawLines(float[] pts, Paint paint) 画线（批量）
drawRoundRect(float left, float top, float right, float bottom, float rx, float ry, Paint paint) 画圆角矩形
drawArc(float left, float top, float right, float bottom, float startAngle, float sweepAngle, boolean useCenter, Paint paint) 绘制弧形或扇形 useCenter 表示是否连接到圆心，如果不连接到圆心，就是弧形，如果连接到圆心，就是扇形。

drawBitmap(Bitmap bitmap, float left, float top, Paint paint) 画 Bitmap
drawText(String text, float x, float y, Paint paint) 绘制文字
drawPath(Path path, Paint paint) 画自定义图形

Path
Path 方法第一类：直接描述路径。

第一组： addXxx() ——添加子图形
addCircle(float x, float y, float radius, Direction dir) 添加圆最后一个参数 dir 是画圆的路径的方向。
顺时针 (CW clockwise) 和逆时针 (CCW counter-clockwise) 。它只是在需要填充图形 (Paint.Style 为 FILL 或 FILL_AND_STROKE) ，并且图形出现自相交时，用于判断填充范围的

addOval(float left, float top, float right, float bottom, Direction dir) / addOval(RectF oval, Direction dir) 添加椭圆

addRect(float left, float top, float right, float bottom, Direction dir) / addRect(RectF rect, Direction dir) 添加矩形

addRoundRect(RectF rect, float rx, float ry, Direction dir) / addRoundRect(float left, float top, float right, float bottom, float rx, float ry, Direction dir) / addRoundRect(RectF rect, float[] radii, Direction dir) / addRoundRect(float left, float top, float right, float bottom, float[] radii, Direction dir) 添加圆角矩形

addPath(Path path) 添加另一个 Path

第二组：xxxTo() ——画线（直线或曲线） 这一组和第一组 addXxx() 方法的区别在于，第一组是添加的完整封闭图形（除了 addPath() ），而这一组添加的只是一条线。
lineTo(float x, float y) / rLineTo(float x, float y) 画直线 lineTo(x, y) 的参数是绝对坐标，而 rLineTo(x, y) 的参数是相对当前位置的相对坐标
quadTo(float x1, float y1, float x2, float y2) / rQuadTo(float dx1, float dy1, float dx2, float dy2) 画二次贝塞尔曲线
moveTo(float x, float y) / rMoveTo(float x, float y) 移动到目标位置

arcTo(RectF oval, float startAngle, float sweepAngle, boolean forceMoveTo) /
arcTo(float left, float top, float right, float bottom, float startAngle, float sweepAngle, boolean forceMoveTo) / 
arcTo(RectF oval, float startAngle, float sweepAngle)
画弧形它们也是用来画线的，但并不使用当前位置作为弧线的起点。forceMoveTo 参数的意思是，绘制是要「抬一下笔移动过去」，还是「直接拖着笔过去」，区别在于是否留下移动的痕迹。
addArc(float left, float top, float right, float bottom, float startAngle, float sweepAngle) / addArc(RectF oval, float startAngle, float sweepAngle) addArc() 只是一个直接使用了 forceMoveTo = true 的简化版 arcTo() 。

close() 封闭当前子图形.不是所有的子图形都需要使用 close() 来封闭。当需要填充图形时（即  Paint.Style 为 FILL 或 FILL_AND_STROKE），Path 会自动封闭子图形。

Path 方法第二类：辅助的设置或计算
Path.setFillType(Path.FillType ft) 设置填充方式

FillType 的取值有四个：
EVEN_ODD
WINDING （默认值）
INVERSE_EVEN_ODD
INVERSE_WINDING

EVEN_ODD
即 even-odd rule （奇偶原则）：对于平面中的任意一点，向任意方向射出一条射线，这条射线和图形相交的次数（相交才算，相切不算哦）如果是奇数，则这个点被认为在图形内部，是要被涂色的区域；如果是偶数，则这个点被认为在图形外部，是不被涂色的区域。

WINDING
即 non-zero winding rule （非零环绕数原则）：首先，它需要你图形中的所有线条都是有绘制方向的：
然后，同样是从平面中的点向任意方向射出一条射线，但计算规则不一样：以 0 为初始值，对于射线和图形的所有交点，遇到每个顺时针的交点（图形从射线的左边向右穿过）把结果加 1，遇到每个逆时针的交点（图形从射线的右边向左穿过）把结果减 1，最终把所有的交点都算上，得到的结果如果不是 0，则认为这个点在图形内部，是要被涂色的区域；如果是 0，则认为这个点在图形外部，是不被涂色的区域。


Paint
Paint 的 API 大致可以分为 4 类：

颜色
效果
drawText() 相关
初始化

颜色：
一，基本着色
Paint的着色方案：1.直接用setColor/ARGB（）来设置颜色  2.使用Shader来设置颜色
eg：paint.setColor(Color.parseColor("#FFFFFFF"));
paint.setShader(Shader shader);
shader 是一套着色规则。当设置了 Shader 之后，Paint 在绘制图形和文字时就不使用 setColor/ARGB() 设置的颜色了，而是使用 Shader 的方案中的颜色。
在 Android 的绘制里使用 Shader ，并不直接用 Shader 这个类，而是用它的几个子类。具体来讲有 LinearGradient RadialGradient SweepGradient BitmapShader ComposeShader
eg: Shader shader = new LinearGradient(100, 100, 500, 500, Color.parseColor("#E91E63"),  
        Color.parseColor("#2196F3"), Shader.TileMode.CLAMP);
//LinearGradient(float x0, float y0, float x1, float y1, int color0, int color1, Shader.TileMode tile)
Shader shader = new RadialGradient(300, 300, 200, Color.parseColor("#E91E63"),  
        Color.parseColor("#2196F3"), Shader.TileMode.CLAMP);		
//RadialGradient(float centerX, float centerY, float radius, int centerColor, int edgeColor, TileMode tileMode)
Shader shader = new SweepGradient(300, 300, Color.parseColor("#E91E63"),  
        Color.parseColor("#2196F3"));   	//SweepGradient(float cx, float cy, int color0, int color1)
BitmapShader(Bitmap bitmap, Shader.TileMode tileX, Shader.TileMode tileY)
TileMode 总共三个模式 CLAMP,  MIRROR 和 REPEAT

ComposeShader 混合着色器,就是把两个 Shader 一起使用。
// 第一个 Shader：头像的 Bitmap
Bitmap bitmap1 = BitmapFactory.decodeResource(getResources(), R.drawable.batman);  
Shader shader1 = new BitmapShader(bitmap1, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);

// 第二个 Shader：从上到下的线性渐变（由透明到黑色）
Bitmap bitmap2 = BitmapFactory.decodeResource(getResources(), R.drawable.batman_logo);  
Shader shader2 = new BitmapShader(bitmap2, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);

// ComposeShader：结合两个 Shader
Shader shader = new ComposeShader(shader1, shader2, PorterDuff.Mode.SRC_OVER);  
paint.setShader(shader);

//其构造函数为：ComposeShader(Shader shaderA, Shader shaderB, PorterDuff.Mode mode)
PorterDuff.Mode 是用来指定两个图像共同绘制时的颜色策略的。
PorterDuff.Mode 一共有 17 个，可以分为两类：

Alpha 合成 (Alpha Compositing)
混合 (Blending)

所以对于这些 Mode，正确的做法是：对于 Alpha 合成类的操作，掌握他们，并在实际开发中灵活运用；而对于混合类的，你只要把它们的名字记住就好了，这样当某一天设计师告诉你「我要做这种混合效果」的时候，你可以马上知道自己能不能做，怎么做。

二，setColorFilter(ColorFilter colorFilter)
为绘制设置颜色过滤。
在 Paint 里设置 ColorFilter ，使用的是 Paint.setColorFilter(ColorFilter filter) 方法。  ColorFilter 并不直接使用，而是使用它的子类。它共有三个子类：LightingColorFilter PorterDuffColorFilter 和 ColorMatrixColorFilter。

LightingColorFilter 是用来模拟简单的光照效果的。其构造方法是 LightingColorFilter(int mul, int add) ，参数里的 mul 和  add 都是和颜色值格式相同的 int 值，其中 mul 用来和目标像素相乘，add 用来和目标像素相加
一个「保持原样」的「基本 LightingColorFilter 」，mul 为 0xffffff，add 为 0x000000
可以把它的 mul 改为 0x00ffff，那么像素里的红色则会被移除

PorterDuffColorFilter 是使用一个指定的颜色和一种指定的 PorterDuff.Mode 来与绘制对象进行合成。它的构造方法是 PorterDuffColorFilter(int color, PorterDuff.Mode mode) 其中的 color 参数是指定的颜色， mode 参数是指定的 Mode。

ColorMatrixColorFilter 使用一个 ColorMatrix 来对颜色进行处理。 

三。setXfermode(Xfermode xfermode)
Xfermode 指的是你要绘制的内容和 Canvas 的目标位置的内容应该怎样结合计算出最终的颜色。注意之前的一和二的使用都是针对颜色的 ，这里是直接针对两个View的，还是有差别的
eg:
Xfermode xfermode = new PorterDuffXfermode(PorterDuff.Mode.DST_IN);

canvas.drawBitmap(rectBitmap, 0, 0, paint); // 画方  
paint.setXfermode(xfermode); // 设置 Xfermode  
canvas.drawBitmap(circleBitmap, 0, 0, paint); // 画圆  
paint.setXfermode(null); // 用完及时清除 Xfermode 

通常在绘制最前面加上 int saved = canvas.saveLayer(null, null, Canvas.ALL_SAVE_FLAG);
在绘制完后加上canvas.restoreToCount(saved);来进行离屏缓冲，提高效率

前面讲的就是 Paint 的第一类 API——关于颜色的三层设置：直接设置颜色的 API 用来给图形和文字设置颜色； setColorFilter() 用来基于颜色进行过滤处理；  setXfermode() 用来处理源图像和 View 已有内容的关系

效果：
setAntiAlias (boolean aa) 设置抗锯齿
setStyle(Paint.Style style)
线条形状:
	setStrokeWidth(float width), 
	setStrokeCap(Paint.Cap cap),  
	setStrokeJoin(Paint.Join join),  设置拐角的形状。有三个值可以选择：MITER 尖角、 BEVEL 平角和 ROUND 圆角。默认为  MITER
	setStrokeMiter(float miter) 这个方法是对于 setStrokeJoin() 的一个补充，它用于设置 MITER 型拐角的延长线的最大值。仅在setStrokeJoin(MITER)时管用
色彩优化：
	setDither(boolean dither) 增加抖动使画面看起来更真实
	setFilterBitmap(boolean filter) 设置是否使用双线性过滤来绘制 Bitmap
设置轮廓效果：setPathEffect(PathEffect effect)
	PathEffect 分为两类，单一效果的  CornerPathEffect DiscretePathEffect DashPathEffect PathDashPathEffect ，和组合效果的  SumPathEffect ComposePathEffect。
 CornerPathEffect(float radius) 把所有拐角变成圆角
 
 DiscretePathEffect(float segmentLength, float deviation) 把线条进行随机的偏离，让轮廓变得乱七八糟
 	segmentLength 是用来拼接的每个线段的长度， deviation 是偏离量
 
 DashPathEffect(float[] intervals, float phase) 使用虚线来绘制线条 第一个参数 intervals 是一个数组，它指定了虚线的格式：数组中元素必须为偶数（最少是 2 个），按照「画线长度、空白长度】；第二个参数  phase 是虚线的偏移量。

 PathDashPathEffect(Path shape, float advance, float phase, PathDashPathEffect.Style style)它是使用一个 Path 来绘制「虚线」style 的类型为 PathDashPathEffect.Style
 TRANSLATE：位移
 ROTATE：旋转
 MORPH：变体

 SumPathEffect(PathEffect , PathEffect ) 分别按照两种 PathEffect 分别对目标进行绘制。
 ComposePathEffect(PathEffect , PathEffect ) 不过它是先对目标 Path 使用一个 PathEffect，然后再对这个改变后的 Path 使用另一个 PathEffect

在之后的绘制内容下面加一层阴影:
setShadowLayer(float radius, float dx, float dy, int shadowColor) 
clearShadowLayer()用来清除阴影层

绘制层上方的附加效果
setMaskFilter（MaskFilter maskfilter ） BlurMaskFilter EmbossMaskFilter

Text相关这次不涉及

初始化：
reset() 相当于重置
set(Paint src) 将src的所有属性全部复制过来


文字绘制
canvas.drawText()
canvas.drawTextOnPath("Hello HeCoder", path, 0, 0, paint)
参数里，需要解释的只有两个： hOffset 和 vOffset。它们是文字相对于 Path 的水平偏移量和竖直偏移量，利用它们可以调整文字的位置。
StaticLayout staticLayout2 = new StaticLayout(text2, paint, 600,  
        Layout.Alignment.ALIGN_NORMAL, 1, 0, true);

width 是文字区域的宽度，文字到达这个宽度后就会自动换行； 
align 是文字的对齐方向； 
spacingmult 是行间距的倍数，通常情况下填 1 就好； 
spacingadd 是行间距的额外增加值，通常情况下填 0 就好； 
includeadd 是指是否在文字上下添加额外的空间，来避免某些过高的字符的绘制出现越界

然后就paint的各种Text的set方法，用的时候再说

测量文字尺寸类
float getFontSpacing() 获取推荐的行距。
getTextBounds(String text, int start, int end, Rect bounds) 获取文字的显示范围,放到bounds。
float measureText(String text) 测量文字的宽度并返回。
这期实用性东西不多
