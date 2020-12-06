# Матрицы в Android, вторая часть

В первой статье, мы познакомились с аффинным преобразованием, в каких целях оно используется, а также рассмотрели преобразование = "Сдвиг". Если в первой статье большая часть строк была отведена теории, то во второй части мы поговорим о практическом применении, где подробно рассмотрим преобразования масштабирования и вращения, с учетом опорной точки для этих трансформаций. Эти преобразования, широко используется в обработке изображений, поэтому думаю будет интересно. Поехали!

P.S.: Информацию о подготовке проекта и ресурсов вы найдете в первой статье!

### Масштабирование изображения (Scale)

Преобразование масштабирования увеличивает или уменьшает размер изображения объекта в системе координат наблюдателя по сравнению с исходным размером в системе координат этого самого объекта.
 
 Так по какому принципу происходит масштабирование изображений? В основе процесса масштабирования лежит произведение координат на коэффициент масштабирования, конечно, это если не учитывать опорную точку, но о ней мы поговорим чуть позже. Итак, для начала проименуем коэффициенты для каждой оси Sx и Sy, для осей абсцисс и ординат соответственно.

Получаем следующие равенства:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_scale.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_scale_final.png?raw=true)

Матрица 3 х 3 для применения масштабирования принимает следующий вид: 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrice_scale.png?raw=true)

Чтобы применить матрицу масштабирования, умножаем ее на матрицу координат:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_scale_example.png?raw=true)

Теперь рассмотрим применение масштабирования в Android.

Будем растягивать исходное изображение в 2 раза по двум осям. Первым делом вычислим координаты углов изображения после преобразования:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/sample_scale_results.png?raw=true)
 
Посмотрите на реализацию:

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
        imageView.doOnLayout { scale() }
    }

    private fun scale() {
        val transformMatrix = Matrix()
        val sx = 2f
        val sy = 2f
        val matrixValues = floatArrayOf(
            sx, 0f, 0f,
            0f, sy, 0f,
            0f, 0f, 1f
        )
        transformMatrix.setValues(matrixValues)
        val imageMatrix = imageView.imageMatrix
        imageMatrix.postConcat(transformMatrix)
        imageView.imageMatrix = imageMatrix
        printImageCoords(transformMatrix)
    }
    ...
}

~~~

Результат:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/image_sample_scale_result.png?raw=true)

Посмотрим логи для убеждения в правильности наших расчетов:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matix_values_log_input_scale_coords.png?raw=true)

Скорее всего, вы уже догадались, что нам необязательно каждый раз создавать матрицу преобразования. Для этого достаточно знать о методах postScale(...) и preScale(...). Эти методы включают в себя обязательные параметры: sx и sy. А также опциональные (необязательные) параметры: px и py. 
Вот так выглядит использование метода scale(), без матрицы преобразования:

~~~ kotlin

...

  private fun scale() {
        val sx = 2f
        val sy = 2f
        val imageMatrix = imageView.imageMatrix
        imageMatrix.postScale(sx, sy)
        imageView.imageMatrix = imageMatrix
    }
  
...
~~~

Хорошо, теперь рассмотрим масштабирование вокруг опорной точки. Тут нам необходимо выполнить комбинацию преобразований:
1. Сместить плоскость изображения так, чтобы опорная точка совпадала с началом координат плоскости наблюдателя. Простыми словами, выполнить сдвиг на обратные величины опорной точки.
2. Применить масштабирование;
3. Выполнить сдвиг плоскости изображения на расстояние координат опорной точки в соответствующих осях.

Тогда матрица 3 х 3 для масштабирования вокруг опорной точки принимает вид:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_scale_anchor_point_formula.png?raw=true)

Пример:

Перед масштабированием выполним известную нам операцию переноса(из первой статьи) в центр контейнера. Таким образом, получим комбинацию преобразований. 
Как и прежде, сначала рассчитаем финальное положение углов изображения. Но, чтобы это выполнить, нам необходимо получить опорную точку для масштабирования. Так как изображение будет расположено в центре контейнера, то опорной точкой будет центр контейнера, а координаты данной точки будут равны половине его ширины и высоты. В моем случае получилось:

  * px = 384;
  * py = 512;

Параметры  tx и ty равны 264 и 352 соответственно, их мы вычисляли в первой статье, когда выполняли сдвиг изображения в центр. Теперь, когда известны все входные данные, перейдем к вычислениям окончательных координат углов изображения после выполнения комбинации преобразований

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/image_formula_scale_sample_anchor_point.png?raw=true)
 
Давайте приступим к реализации на Android:

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
        imageView.doOnLayout { transAndScale() }
    }

    private fun transAndScale() {
        val transformImageMatrix = imageView.imageMatrix

        //region translate to center
        val translateToCenterMatrix = Matrix()
        val tx = (imageView.measuredWidth - imageView.drawable.intrinsicWidth) / 2f
        val ty = (imageView.measuredHeight - imageView.drawable.intrinsicHeight) / 2f
        val translateMatrixValues = floatArrayOf(
            1f, 0f, tx,
            0f, 1f, ty,
            0f, 0f, 1f
        )
        translateToCenterMatrix.setValues(translateMatrixValues)
        transformImageMatrix.postConcat(translateToCenterMatrix)
        //endregion translate to center

        //region scale
        val sx = 2f
        val sy = 2f
        val px = imageView.measuredWidth / 2f
        val py = imageView.measuredHeight / 2f
        val scaleMatrixValues = floatArrayOf(
            sx, 0f, -px * sx + px,
            0f, sy, -py * sy + py,
            0f, 0f, 1f
        )
        val scaleMatrix = Matrix()
        scaleMatrix.setValues(scaleMatrixValues)
        transformImageMatrix.postConcat(scaleMatrix)
        //endregion scale

        imageView.imageMatrix = transformImageMatrix
        printMatrixValues(transformImageMatrix)
        printImageCoords(transformImageMatrix)
    }

    ...

}
~~~

Для удобства чтения разбиваем метод на 2 региона, чтобы с легкостью наблюдать последовательное применение преобразований. Результат получился следующим:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/image_sample_scale_2_result.png?raw=true)
 
 Вывод лога разбиваем на 2 части:

1. Вывод получившейся матрицы преобразования:
 
  ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matix_values_log_input_scale_trans.png?raw=true)
 
2. Вывод координат изображения после преобразования:
 
  ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matix_values_log_input_scale_trans_coords.png?raw=true)

Текущие выводы позволяют нам судить, что наши расчеты верны!

Теперь перепишем метод transAndScale() с использованием методов postTranlate() и postScale():

~~~ kotlin
...
  
    private fun transAndScale() {
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
        imageView.imageMatrix = transformImageMatrix
    }

...
~~~

### Поворот (вращение) изображения  (Rotate)

Преобразование поворота устанавливает соответствие между координатами точки, объекта и экраном системы координат наблюдателя при вращении объекта (без сдвига) относительно начала координат.
Другими словами, мы будем вращать плоскость, на которой лежит наше изображение.
Прежде чем начать обсуждать вращение, договоримся, что принято считать вращение против часовой стрелки положительным. В таком случае, удобно считать, что угол поворота (альфа) лежит в интервале [-П;П].

Чтобы найти формулу преобразования координат, выберем произвольную точку(x,y), вектор которой обозначим r. Тут уже требуется освежить в памяти тригонометрию.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/x=rcos.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/y=rsin.png?raw=true)

Если повернем на угол тета, то получим:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/x'=rcos.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/y'=rsin.png?raw=true)

Зная формулы суммы углов для синуса и косинуса:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/sin(a+q).png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/cos(a+q).png?raw=true)

Тогда:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/x'=rcos(a+q).png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/y'=rsin(a+q).png?raw=true)

Раскроем скобки и выполним замены:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/x'=matrix_rotate_formula.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/y'=matix_rotate_formula.png?raw=true)

Теперь осталось найти матрицу преобразования:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_rotate.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_rotate_final.png?raw=true)


Получим матрицу 3 х 3, применяемую для преобразования вращения: 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_rotate_final_formula.png?raw=true)

  Предлагаю не останавливаться и сразу перейти к вращению плоскости вокруг опорной точки (например центра изображения). Для этого как и с масштабированием необходимо выполнить следующие операции преобразования:
1. Сместить на обратные координаты опорной точки;
2. Выполнить вращение;
3. Сместить обратно на координаты опорной точки;

Таким способом получаем необходимую матрицу преобразования вокруг опорной точки:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_rotate_anchor_point_final.png?raw=true)

Так как используется несколько типов преобразований, то этот вид вращения является комбинацией преобразований.    

Теперь рассмотрим применение вращения в Android.
Пропустим базовый пример, состоящий из одного преобразования, вместо этого предлагаю дополнить пример из раздела "Масштабирование". Следовательно, исходными данными будет отцентрированное и растянутое по двух осям изображение, а входной матрицей будет результат матрицы из раздела "Масштабирование". Вращать изображение мы будем на 90 градусов.
Как обычно, давайте рассчитаем координаты, на которые лягут углы изображения на координатной плоскости.

P.S: Не пугайтесь, вам необязательно прослеживать весь путь решения, достаточно понимать выполняемые действия. =) 
 
 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/formua_sample_image_rotate_anchor_point.png?raw=true)
 
Теперь когда у нас есть ожидаемый результат, давайте перейдем к реализации:
 
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
        imageView.doOnLayout { transScaleRotate() }
    }

    private fun transScaleRotate() {
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

        //region rotate
        val matrixRotate = Matrix()
        val degrees = Math.toRadians(90.0)
        val rotateMatrixValues = floatArrayOf(
            Math.cos(degrees).toFloat(), (-Math.sin(degrees)).toFloat(), (-Math.cos(degrees)).toFloat() * px + Math.sin(degrees).toFloat() * py + px,
            Math.sin(degrees).toFloat(), Math.cos(degrees).toFloat(), (-Math.sin(degrees)).toFloat() * px - Math.cos(degrees).toFloat() * py + py,
            0f, 0f, 1f
        )
        matrixRotate.setValues(rotateMatrixValues)
        transformImageMatrix.postConcat(matrixRotate)
        //endregion rotate

        imageView.imageMatrix = transformImageMatrix
        printMatrixValues(transformImageMatrix)
        printImageCoords(transformImageMatrix)
    }

    ...
}
~~~

Я переименовал метод в transScaleRotate(). Операции сдвига и масштабирования мы оставили неизменными, но добавили регион кода выполняющего вращение (rotate). 

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/image_sample_rotate.png?raw=true)
 
Выводы в лог подтверждают правильность наших расчетов:
 
  ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matix_values_log_input_transScaleRotate.png?raw=true)

  ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matix_values_log_input_transScaleRotate_coords.png?raw=true)
 
  Конечно, класс Matrix уже содержит методы для выполнения вращения: postRotate и preRotate. У данных методов есть обязательный параметр degrees, который определяет угол поворота плоскости изображения в градусах, а также опциональные параметры px и py – также как и в масштабировании эти параметры определяют координату опорной точки. А теперь, по традиции, перепишем метод transScaleRotate с использованием метода postRotate(). 
Обратите внимание, что в операциях масштабирования и вращения, используются одни и те же значения px и py, так как координаты опорных точек для этих операций мы сделали одинаковыми. Напоминаю, что px и py в нашем случае описывают координаты центра контейнера ImageView.

~~~ kotlin

...
    private fun transScaleRotate() {
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

        //region rotate
        val degrees = 90f
        imageMatrix.postRotate(degrees, px, py)
        //endregion rotate
        
        printMatrixValues(imageMatrix)
        imageView.imageMatrix = imageMatrix
    }
...

~~~

Теперь, вы знаете, как применять на практике преобразования: сдвиг, масштабирование и вращение. И это уже очень хорошо!
В следующей заключительной статье, мы поговорим о том, как выполнять наклон и отражение изображений. 

Всем спасибо!