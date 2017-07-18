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