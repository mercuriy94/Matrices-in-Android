# Матрицы в Android, третья (заключительная) часть

Во второй статье мы рассмотрели масштабирование и вращение изображений с подробными примерами. В текущей заключительной статье мы познакомимся с такими преобразованиями, как наклон (Skew) и отражение (Reflexion). Не будем затягивать, приступим!

P.S.: Напоминаю, что информацию о подготовке проекта и ресурсов вы найдете в первой статье!

### Наклон (Skew)

К сожалению, я не нашел понятного объяснения этого преобразования. Так что, будем получать матрицу преобразования опытным путем. Для начала, давайте посмотрим на следующее изображение.

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_image.png?raw=true)

Здесь параллелограмм с зеленым цветом периметра является квадратом, расположенным под углом. Проведем касательную левой-верхней границы наклоненного квадрата. Таким образом, получаем угол наклона. 

А также, наблюдаем прямоугольный треугольник  (B'CC'). Вспоминаем, что в прямоугольном треугольнике:
 - Гипотенуза — самая длинная сторона;
 - Катеты — стороны, лежащие напротив острых углов;
 - Противолежащий катет — сторона треугольника, которая лежит напротив заданного острого угла (СС');
 - Прилежащий катет — сторона треугольника которая прилегает к заданному острому углу (B'C).

Основываясь на курсе тригонометрии, воспользуемся следующими тождествами:

Синус острого угла в прямоугольном треугольнике — отношение противолежащего катета к гипотенузе:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/sinA.png?raw=true)

Косинус острого угла в прямоугольном треугольнике — отношение прилежащего катета к гипотенузе:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/cosA.png?raw=true)

Тангенс острого угла в прямоугольном треугольнике — отношение противолежащего катета к прилежащему:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/tanA.png?raw=true)

Так как, противолежащая сторона неизвестна, но мы знаем угол наклона и прилежащий катет, то с легкостью воспользуемся функцией тангенса:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/CC'.png?raw=true)

Теперь остается решить систему уравнений, чтобы найти матрицу преобразования для наклона:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/y=y'.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_x_skew.png?raw=true)

Напоминаю, что общая формула преобразования для оси абсцисс выглядит следующим образом (мы выводили ее в первой статье): 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/x'=ax+by+cz.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/a=1.png?raw=true)

Аналогичная ситуация и с осью ординат.
В итоге матрица преобразования 3 x 3 для наклона по оси абсцисс (X), принимает вид:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_X.png?raw=true)

А для оси ординат (Y):

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_Y.png?raw=true) 

Если требуется одновременно наклонять по двум осям, то используем полную матрицу:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_formula.png?raw=true) 

Отлично, но ведь можно еще наклонять вокруг опорной точки, тут, как и в предыдущих разделах не обойтись без сдвигов:

1. Сместить на обратные координаты опорной точки;
2. Выполнить вращение;
3. Сместить обратно на координаты опорной точки;

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_anchor_point_formula.png?raw=true) 

Не перепутайте: угол альфа используется для наклона по оси абсцисс, а угол бета по оси ординат.

В качестве примера, давайте выполним наклон отцентрированного и растянутого изображения. В качестве исходной матрицы предлагаю взять результат матрицы из второй статьи раздела "Масштабирование". Наклонять будем вокруг опорной точки, и, как обычно, в качестве координат опорной точки возьмем центр контейнера ImageView. Выполним наклон по оси абсцисс на 15 градусов, а по оси ординат на 30 градусов. 

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/%20image_skew_sample.png?raw=true)
  
  Перепишем наш пример:
  
~~~ kotlin
package com.mercuriy94.matrix.affinetransformations

import android.graphics.Matrix
import android.os.Bundle
import android.util.Log
import android.widget.ImageView
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.doOnLayout
import org.ejml.simple.SimpleMatrix


class MainActivity : AppCompatActivity() {

    companion object {
        private const val TAG = "MainActivity"
    }

    private lateinit var imageView: ImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        imageView = findViewById(R.id.imageView)
        imageView.doOnLayout { transScaleSkew() }
    }

    private fun transScaleSkew() {
        val transformImageMatrix = imageView.imageMatrix

        //region translate to center
        val tx = (imageView.measuredWidth - imageView.drawable.intrinsicWidth) / 2f
        val ty = (imageView.measuredHeight - imageView.drawable.intrinsicHeight) / 2f
        transformImageMatrix.postTranslate(tx, ty)
        //endregion translate to center

        //region scale
        val sx = 2f
        val sy = 2f
        val px = imageView.measuredWidth / 2f
        val py = imageView.measuredHeight / 2f
        transformImageMatrix.postScale(sx, sy, px, py)
        //endregion scale

        //region skew
        val skewMatrixValues = floatArrayOf(
            1f, Math.tan(Math.toRadians(15.0)).toFloat(), (-Math.tan(Math.toRadians(15.0))).toFloat() * py,
            Math.tan(Math.toRadians(30.0)).toFloat(), 1f, (-Math.tan(Math.toRadians(30.0))).toFloat() * px,
            0f, 0f, 1f
        )
        val skewMatrix = Matrix()
        skewMatrix.setValues(skewMatrixValues)
        transformImageMatrix.postConcat(skewMatrix)
        //endregion skew

        imageView.imageMatrix = transformImageMatrix
        printMatrixValues(transformImageMatrix)
        printImageCoords(transformImageMatrix)
    }
    ...

}
~~~

Результат получился следующим:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/image_skew_sample_result.png?raw=true)
 
Выводы в лог позволяют заключить, что наши расчеты верны:
 
  ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matix_values_log_input_transScaleSkew.png?raw=true)

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matix_values_log_input_transScaleSkew_coords.png?raw=true)

Конечно, класс Matrix содержит готовые методы для выполнения наклона postSkew и preSkew. У этих методов есть два обязательных параметра kx и ky, которые являются значениями тангенса угла. Если вам нужно наклонять только по одной из осей, то для второй передаем параметр 0f, так как тангенс нуля равен нулю. А также, имеется два опциональных параметра px и py, смысл которых нам уже известен из второй статьи. Теперь перепишем наш метод transScaleSkew с использованием метода postSkew.

~~~kotlin
    ...

    private fun transScaleSkew() {
        val imageMatrix = imageView.imageMatrix

        //region translate to center
        val tx = (imageView.measuredWidth - imageView.drawable.intrinsicWidth) / 2f
        val ty = (imageView.measuredHeight - imageView.drawable.intrinsicHeight) / 2f
        imageMatrix.postTranslate(tx, ty)
        //endregion translate to center

        //region scale
        val sx = 2f
        val sy = 2f
        val px = imageView.measuredWidth / 2f
        val py = imageView.measuredHeight / 2f
        imageMatrix.postScale(sx, sy, px, py)

        //endregion scale

        //region skew
        imageMatrix.postSkew(
            Math.tan(Math.toRadians(15.0)).toFloat(),
            Math.tan(Math.toRadians(30.0)).toFloat(),
            px,
            py
        )
        //endregion skew
        printMatrixValues(imageMatrix)
        imageView.imageMatrix = imageMatrix
    }
    
    ...
~~~

### Отражение (Reflexion)

Различают два вида отражения:

 * Отражение относительно оси;
 * Отражение относительно точки;

Для начала разберемся с отражением вокруг начала координат. Рассмотрим отражение вокруг осей по отдельности.

Предлагаю, отталкиваться от мысли, что принцип отражения относительно начала координат заключается в смене знака соседней координаты, относительно которой выполняется отражение, на противоположный. Предположим, имеется точка с координатами P(x,y). Чтобы выполнить отражение относительно оси абсцисс, нам необходимо сменить координату на противоположную, и мы получим P' (x, -y). Тогда:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflect_around_x_1.png?raw=true)
 
 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflect_around_x.png?raw=true)
 
Получаем матрицу 3 х 3, для выполнения отражения относительно оси абсцисс:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflect_arround_x_final.png?raw=true)

И для оси ординат:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflecty_around_y_1.png?raw=true)
 
 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_reflect_around_y_2.png?raw=true)
 
И матрица 3 х 3 получается:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_reflect_around_y_final.png?raw=true)
 
Отлично! Но, как часто нам приходится отражать точки относительно начала координат? Ответом скорее всего будет «очень редко». Поэтому попробуем вывести матрицу для отражения относительно опорной точки. Для этого выполним комбинацию преобразований:

1. Сместить на обратные координаты опорной точки;
2. Выполнить отражение;
3. Сместить обратно на координаты опорной точки;

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_reflect_anchor_point.png?raw=true)

Рассмотрим пример отражения в Android, относительно точки. Отражать будем отцентрированное изображение в сторону верхней границы контейнера (представьте, что выполняем разворот изображения снизу вверх). Следовательно, координаты  опорной точки будут:

 * px = 384 (половина ширины контейнера ImageView);
 * py = 352 (используемый сдвиг по оси y для центрирования);

Как положено, рассчитаем конечные координаты изображения:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_reflect_anchor_point_sample.png?raw=true)
 
Перейдем к реализации на Android:

~~~kotlin
package com.mercuriy94.matrix.affinetransformations

import android.graphics.Matrix
import android.os.Bundle
import android.util.Log
import android.widget.ImageView
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.doOnLayout
import org.ejml.simple.SimpleMatrix


class MainActivity : AppCompatActivity() {

    companion object {
        private const val TAG = "MainActivity"
    }

    private lateinit var imageView: ImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        imageView = findViewById(R.id.imageView)
        imageView.doOnLayout { translateToCenterAndReflect() }
    }

    private fun translateToCenterAndReflect() {
        val transformMatrix = Matrix()

        //region trans
        val tx = (imageView.measuredWidth - imageView.drawable.intrinsicWidth) / 2f
        val ty = (imageView.measuredHeight - imageView.drawable.intrinsicHeight) / 2f
        val imageMatrix = imageView.imageMatrix
        imageMatrix.postTranslate(tx, ty)
        //endregion trans

        //region reflect
        val px = imageView.measuredWidth / 2f
        val py = ty
        val matrixValues = floatArrayOf(
            -1f, 0f, 2 * px,
            0f, -1f, 2 * py,
            0f, 0f, 1f
        )
        transformMatrix.setValues(matrixValues)
        imageMatrix.postConcat(transformMatrix)

        //endregion reflect
        imageView.imageMatrix = imageMatrix
        printImageCoords(imageView.imageMatrix)
    }
    
    ...
}
~~~

Результат:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/image_matrix_reflect_sample_result.png?raw=true)

Теперь заглянем в лог:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_reflect_sample_result_coords.png?raw=true)

Да, все правильно! 

### Итоги

Пришло время подвести итоги и вспомнить, какие матрицы преобразования были рассмотрены на протяжении трех статей:

Параллельный перенос (сдвиг):

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_translation.png?raw=true)

Масштабирование:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_scale%202.png?raw=true)

Масштабирование с опорной точкой:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_scale_anchor_point.png?raw=true)

Вращение:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_rotate%202.png?raw=true)

Вращение с опорной точкой:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_rotate_anchor_point.png?raw=true)

Наклон:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew.png?raw=true)

Наклон с опорной точкой:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_anchor_point.png?raw=true)
 
Отражение:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflect_final.png?raw=true)

Отражение относительно точки, лежащей на плоскости:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflect_anhor_point_final.png?raw=true)

Теперь мы знаем, как применять аффинные преобразования на практике и рассчитывать координату, на которую ляжет та или иная точка в результате преобразований. Конечно, чаще всего вам не придется выполнять все эти операции, но знать, по каким правилам изменяется положение точек, необходимо.

Спасибо за внимание! 

Всем чистого кода!