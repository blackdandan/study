# View的绘制

Android 自定义 View 的 onDraw 方法中，常用的绘制类 API 包括 Canvas、Paint、Path、Rect、Bitmap 等。下面结合源码和常见图形举例说明。

---

## 1. Canvas

Canvas 是 Android 提供的画布类，onDraw 方法的参数就是 Canvas。它提供了丰富的绘制方法。

- 直线：
  ```java
  canvas.drawLine(float startX, float startY, float stopX, float stopY, Paint paint);
  ```
- 矩形：
  ```java
  canvas.drawRect(float left, float top, float right, float bottom, Paint paint);
  // 或
  Rect rect = new Rect(left, top, right, bottom);
  canvas.drawRect(rect, paint);
  ```
- 圆形/椭圆：
  ```java
  canvas.drawCircle(float cx, float cy, float radius, Paint paint);
  canvas.drawOval(RectF oval, Paint paint);
  ```
- 文本：
  ```java
  canvas.drawText(String text, float x, float y, Paint paint);
  ```
- 位图：
  ```java
  canvas.drawBitmap(Bitmap bitmap, float left, float top, Paint paint);
  ```

---

## 2. Paint

Paint 用于描述颜色、样式、抗锯齿、线宽、字体等属性。

- 设置颜色：
  ```java
  Paint paint = new Paint();
  paint.setColor(Color.RED);
  ```
- 抗锯齿：
  ```java
  paint.setAntiAlias(true);
  ```
- 填充/描边：
  ```java
  paint.setStyle(Paint.Style.FILL); // 填充
  paint.setStyle(Paint.Style.STROKE); // 描边
  ```
- 线宽：
  ```java
  paint.setStrokeWidth(5f);
  ```

---

## 3. Path

Path 用于描述复杂的几何路径（如多边形、贝塞尔曲线等）。

- 示例：绘制三角形
  ```java
  Path path = new Path();
  path.moveTo(100, 100);
  path.lineTo(200, 100);
  path.lineTo(150, 200);
  path.close();
  canvas.drawPath(path, paint);
  ```

---

## 4. Rect / RectF

Rect（整型）和 RectF（浮点型）用于描述矩形区域。

- 示例：
  ```java
  Rect rect = new Rect(50, 50, 200, 200);
  canvas.drawRect(rect, paint);
  ```

---

## 5. Bitmap

Bitmap 用于处理和绘制位图图片。

- 示例：
  ```java
  Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.example);
  canvas.drawBitmap(bitmap, 0, 0, paint);
  ```

---

## 6. 图形举例

- 画一条红色直线：
  ```java
  Paint paint = new Paint();
  paint.setColor(Color.RED);
  paint.setStrokeWidth(5f);
  canvas.drawLine(50, 50, 250, 50, paint);
  ```
- 画一个蓝色实心圆：
  ```java
  Paint paint = new Paint();
  paint.setColor(Color.BLUE);
  paint.setStyle(Paint.Style.FILL);
  canvas.drawCircle(150, 150, 100, paint);
  ```
- 画一个绿色三角形：
  ```java
  Paint paint = new Paint();
  paint.setColor(Color.GREEN);
  paint.setStyle(Paint.Style.FILL);
  Path path = new Path();
  path.moveTo(100, 100);
  path.lineTo(200, 100);
  path.lineTo(150, 200);
  path.close();
  canvas.drawPath(path, paint);
  ```

---

## 7. 参考源码

- [Canvas.java (AOSP)](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/graphics/java/android/graphics/Canvas.java)
- [Paint.java (AOSP)](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/graphics/java/android/graphics/Paint.java)
- [Path.java (AOSP)](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/graphics/java/android/graphics/Path.java)

---

如需更详细的自定义绘制案例或源码分析，欢迎继续提问！ 