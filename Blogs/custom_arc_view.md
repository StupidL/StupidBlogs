# Arc View  
今天用一个简单的自定义View来演示如何创建一个自定义View。这个自定义View—— **ArcView** 的效果如下：  
![arc_view](http://123.206.221.113:80/upload/2017/04/0fe0gcpc50g7oqpbdff4ehoh64.png)  

看上去非常简单。但是对于自定义View，常常会发现，看起来相同的一个效果，代码的完整程度可能差别很大。就上述这个效果来说，肯定有人会忽略以下几点：  
* 是否支持padding属性  
* 是否支持wrap_content属性  
* View的状态保存与恢复     

自定义View分为以下几个步骤：  
* 构思  
* 测量  
* 绘制  

接下来，会结合例子对每个步骤逐一讲解。  

## 构思  
这个过程就是把想要实现的效果，分解成一个一个独立的绘制单元，然后逐一绘制组合起来，最终达成想要的效果。  

以ArcView为例，有以下几个单元：  
* 蓝色背景的大圆  
* 红色背景的弧形  
* 深蓝色背景的小圆  
* 小圆中间的文字  

分解完毕之后，在onDraw(Canvas canvas)方法中，逐一绘制就行了。这个会在后面的代码中体现。  

## 测量大小  
接下来就要确定这个View的大小，即长度和宽度。这个步骤，在onMeasure(int, int)方法中实现。  
这个步骤，有两个比较重要的点——wrap_content属性和padding属性的支持。  
说到这里，不得不提MeasureSpec这个东西。在我们的onMeasure(int, int)方法中，有两个整型参数。先贴代码助于理解：  

```java  
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
            int w = (int) (mOuterCircleRadius * 2 + getPaddingLeft() + getPaddingRight());
            setMeasuredDimension(w, w);
        } else if (widthMode == MeasureSpec.AT_MOST) {
            int w = (int) (mOuterCircleRadius * 2 + getPaddingLeft() + getPaddingRight());
            setMeasuredDimension(w, heightSize);
        } else if (heightMode == MeasureSpec.AT_MOST) {
            int h = (int) (mOuterCircleRadius * 2 + getPaddingTop() + getPaddingBottom());
            setMeasuredDimension(widthSize, h);
        }
    }  
```  
MeasureSpec通过SpecMode和SpecSize两个值打包成一个int类型的数据，也就是上述方法中的两个参数。  
int类型一共32位，高2位的值即SpecMode，低30位即SpecSize。  
SpecMode一共有三种：  

|Mode|描述|  
|:---|:---|  
|UNSPECIFIED|要多大给多大，一般用于系统内部调用|  
|EXACTLY|精确值，对应LayoutParams里match_parent或者直接制定数值的大小|  
|AT_MOST|最大的可用大小，对应LayoutParams里wrap_content|  
|  

回到上述方法的实现里面。我们首先获取的是宽度和高度的SpecMode和SpecSize。这两个属性与wrap_content的处理有关。  
**如果长度和宽度都是AT_MOST类型，即都指定为wrap_content,那么我们直接制定一个默认的大小就行。在本例中，指定的默认值就是 大圆直径加上左右两边的内边距。**  
**如果宽度是wrap_content、高度是精确值，那么我们的View的宽度就应该是大圆直径加上两边外边距，View的高度就是match_parent或者指定数值的精确值。**  
**如果宽度是精确值，高度是wrap_content，那么我们的View的宽度就是match_parent或者指定数值的精确值，View的高度就是大圆的直径加上上下两边的外边距。**  

由此，我们的View的大小就已经确定了。  

## 绘制  
按照上述的构思过程，我们分别绘制就行。代码如下：  

```java  

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        mCircleX = getWidth() / 2;
        mCircleY = getHeight() / 2;

        drawOuterCircle(canvas);
        drawArc(canvas);
        drawCircle(canvas);
        drawText(canvas);
        canvas.restore();
    }  

```  

在OnDraw方法中，我们获取了View的宽度和高度，并且计算除了View的中心位置。注意，在onMeasure确定了之后，才能通过getWidth()和getHeight()获取宽度和高度值。  
然后，就是绘制各个单元了。代码如下：  

```java  
 private void drawOuterCircle(Canvas canvas) {

        mOuterCirclePaint.setColor(mOuterCircleColor);
        canvas.drawCircle(mCircleX, mCircleY, mOuterCircleRadius, mOuterCirclePaint);

        mOuterCircleRectF.left = mCircleX - mOuterCircleRadius;
        mOuterCircleRectF.top = mCircleY - mOuterCircleRadius;
        mOuterCircleRectF.right = mCircleX + mOuterCircleRadius;
        mOuterCircleRectF.bottom = mCircleY + mOuterCircleRadius;

    }

    private void drawCircle(Canvas canvas) {
        mInnerCirclePaint.setColor(mInnerCircleColor);
        canvas.drawCircle(mCircleX, mCircleY, mInnerCircleRadius, mInnerCirclePaint);

        mInnerCircleRectF.left = mCircleX - mInnerCircleRadius;
        mInnerCircleRectF.top = mCircleY - mInnerCircleRadius;
        mInnerCircleRectF.right = mCircleX + mInnerCircleRadius;
        mInnerCircleRectF.bottom = mCircleY + mInnerCircleRadius;

    }

    private void drawArc(final Canvas canvas) {
        mArcPaint.setColor(mArcColor);
        mArcPaint.setStyle(Paint.Style.STROKE);
        mArcPaint.setStrokeWidth(mOuterCircleRadius - mInnerCircleRadius);

        mArcRectF.left = (mInnerCircleRectF.left + mOuterCircleRectF.left) / 2;
        mArcRectF.top = (mInnerCircleRectF.top + mOuterCircleRectF.top) / 2;
        mArcRectF.right = (mInnerCircleRectF.right + mOuterCircleRectF.right) / 2;
        mArcRectF.bottom = (mInnerCircleRectF.bottom + mOuterCircleRectF.bottom) / 2;

        canvas.drawArc(mArcRectF.left, mArcRectF.top, mArcRectF.right, mArcRectF.bottom,
                mArcAngleStart, mArcAngleSweep, false, mArcPaint);

    }

    private void drawText(Canvas canvas) {
        mTextPaint.setColor(Color.WHITE);
        mTextPaint.setTextSize(ArchViewUtil.dip2px(getContext(), 18));
        Rect bounds = new Rect();
        mTextPaint.getTextBounds(mCircleText, 0, mCircleText.length(), bounds);
        canvas.drawText(mCircleText, (mInnerCircleRectF.centerX() - mTextPaint.measureText(mCircleText) / 2),
                (mInnerCircleRectF.centerY() + bounds.height() / 2), mTextPaint);
    }
```  

## 自定义属性  
为了方便在xml布局文件中设置属性，我们应该添加自定义属性的支持。  
以ArcView为例，首先在项目的本module下面的res文件夹内，找到values目录，新建一个文件**attrs.xml**，然后在文件内声明支持的属性：  

```xml  
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="ArcView">
        <attr name="innerCircleRadius" format="dimension|reference" />
        <attr name="innerCircleColor" format="color|reference" />
        <attr name="circleText" format="string|reference" />
        <attr name="circleTextColor" format="color|reference" />
        <attr name="circleTextSize" format="dimension|reference" />
        <attr name="outerCircleRadius" format="dimension|reference" />
        <attr name="outerCircleColor" format="color|reference" />
        <attr name="arcColor" format="color|reference" />
        <attr name="arcAngleStart" format="float|reference" />
        <attr name="arcAngleSweep" format="float|reference" />
    </declare-styleable>
</resources>  
```  

然后，你需要在自定义View——**ArcView**的Java文件中解析这些支持的属性：  

```java  
private void init(AttributeSet attrs, int defStyle) {
        // Load attributes
        final TypedArray a = getContext().obtainStyledAttributes(attrs, R.styleable.ArcView, defStyle, 0);
        mInnerCircleRadius = (int) a.getDimension(R.styleable.ArcView_innerCircleRadius, ArchViewUtil.dip2px(getContext(), 50));
        mInnerCircleColor = a.getColor(R.styleable.ArcView_innerCircleColor, Color.parseColor("#303F9F"));
        mCircleText = a.getString(R.styleable.ArcView_circleText);
        mCircleTextColor = a.getColor(R.styleable.ArcView_circleTextColor, Color.WHITE);
        mCircleTextSize = a.getDimension(R.styleable.ArcView_circleTextSize, ArchViewUtil.sp2px(getContext(), 18));
        mOuterCircleRadius = (int) a.getDimension(R.styleable.ArcView_outerCircleRadius, ArchViewUtil.dip2px(getContext(), 100));
        mOuterCircleColor = a.getColor(R.styleable.ArcView_outerCircleColor, Color.parseColor("#3F51B5"));
        mArcColor = a.getColor(R.styleable.ArcView_arcColor, Color.parseColor("#FF4081"));
        mArcAngleStart = a.getFloat(R.styleable.ArcView_arcAngleStart, -90);
        mArcAngleSweep = a.getFloat(R.styleable.ArcView_arcAngleSweep, 90);
        a.recycle();

        mOuterCirclePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mOuterCirclePaint.setColor(mOuterCircleColor);
        mInnerCirclePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mInnerCirclePaint.setColor(mInnerCircleColor);
        mArcPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mArcPaint.setColor(mArcColor);
        mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mTextPaint.setTextSize(mCircleTextSize);
        mTextPaint.setColor(mCircleTextColor);

        mOuterCircleRectF = new RectF();
        mInnerCircleRectF = new RectF();
        mArcRectF = new RectF();

        mRectFPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mRectFPaint.setStyle(Paint.Style.STROKE);
        mRectFPaint.setStrokeWidth(5);
        mRectFPaint.setColor(Color.BLACK);

    }  
```  

解析属性的过程中，大部分都提供有一个默认值，当在xml布局文件未制定属性的时候，不至于属性值为空。  

这样就完成了自定义属性。  

## 关于    
我的GitHub主页：  
[StupidL](https://github.com/StupidL)  

该demo的完整代码已经放在GitHub了：  
[ArcView](https://github.com/StupidL/CustomView)  


至此，一个简单的View就完成了。但是目前为止，还有两个问题没有解决：  
* 常见的自定义动画是如何实现的  
* 状态的保存与获取是如何实现的  

这两个问题会在接下来的文章中说明。敬请期待。