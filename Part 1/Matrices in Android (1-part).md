# Матрицы в Android, первая часть
### Актуальность темы
Если вы разрабатываете приложения под ОС Android, то, время от времени, сталкиваетесь с нетривиальными задачами. Например, дизайнер выдал макет с нестандартными элементами представления. И вы, прикинув сложность реализации, начинаете искать библиотеки, которые уже делают то, что вам нужно. В лучшем случае, вы их находите, затем интегрируете с наименьшими усилиями – все довольны, и все счастливы.

Бывает и так, что у вас есть приблизительное решение вашей задачи, но, чтобы получить желаемый результат, нужно внести изменения в существующее решение. В этом случае есть вероятность фиксить баги в библиотеке.

Наихудший вариант: вы не находите ничего похожего на элемент представления из макета. Вы объясняете дизайнеру, что писать с нуля эту вьюшку затратно по времени и просите заменить на аналогичное более простое решение. В ответ дизайнер говорит вам, что это - ключевой элемент дизайна и заменить его нельзя, так как это нарушит общую концепцию и т.д. Итак, смирившись со своей участью, вы начинаете искать способы решения.

### Введение

На протяжении трёх статей я попытаюсь раскрыть и объяснить основные принципы трансформаций двухмерного пространства с помощью аффинных преобразований, а также рассмотреть примеры часто используемых трансформаций. Для демонстрации примеров я буду использовать виджет ImageView c установленным параметром ScaleType = MATRIX. Отметим, что все трансформации, применяемые к ImageView, так же могут быть применены к вашим кастомным view - элементам, если вдруг для отрисовки вы используете шейдеры (Shaders).

Хорошо, а теперь рассмотрим, какие темы будут подниматься на протяжении всей серии статей.

Первая статья:
  * Аффинные преобразования;
  * Матрицы в Android;
  * Post и Pre функции трансформаций в Android;
  * Перемещение (Transition);

Вторая статья:
  * Масштабирование (Scale);
  * Поворот (Rotate);

Третья статья:
  * Наклон (Skew);
  * Отражение (Reflection);


Каждый раздел преобразования включает в себя примеры, на которых подробно рассматриваются изменения плоскости объекта.

Обращаю ваше внимание: в данной статье вы не найдете информацию по работе с изображением в 3D-пространстве.

Прежде чем перейти к основной теме обсуждения, предлагаю вам перейти по [ссылке](http://math1.ru/education.html#matrix) и освежить в памяти или же познакомиться с матрицами и операциями над ними.

### Аффинные преобразования 

  Перед тем как мы начнем обсуждать аффинные преобразования, нужно понять несколько моментов. 
  
 До того, как изображение появится на экране, оно должно быть как-то описано в графической системе устройства. Первоначально свойства объекта отображаются в системе координат этого самого объекта. Таким образом, при изменении пространства объект будет оставаться неизменным с течением времени. Например, представим, что на экране находятся два разных объекта (изображения). Каждый из них будет описываться в своей системе координат, при этом преобразование плоскости одного изображения не повлечет изменение другого. Чтобы описать расположение объектов, нам необходимо поместить все наши объекты в независимую от них плоскость. Здесь, давайте введем новое понятие "мировая система координат", где располагаются все наши графические объекты. Пользователи, по своей сути, являются наблюдателями и, так же, живут в своей системе координат (система координат наблюдателя). Системы координат объектов и наблюдателя являются трехмерными. Получается, в процессе обработки изображения, система координат объекта преобразуется в двумерную координатную плоскость. Объекты могут эволюционировать относительно наблюдателя. То есть менять свое расположение, сжиматься, вращаться и т.д. 
  
  Есть два решения для описания процесcов преобразований. Первый подход предполагает, что система координат наблюдателя остается неподвижна, а мировая система координат и система координат объекта могут эволюционировать. Данный вариант более удобен для наблюдения за изменениями объекта. Второй подход предполагает, что неподвижна мировая система координат, а системы координат наблюдателя и объекта эволюционируют. В операционной системе Android используется первый подход.
  
Таким образом, мы выяснили, что имеется 2 плоскости:

1. Плоскость объекта, которую мы будем преобразовывать;
2. Плоскость наблюдателя, из которой мы можем наблюдать изменения объекта;

  Вся работа с объектами происходит в прямоугольной декартовой системе координат, где каждая точка описывается парой координат x и y. Процесс аффинного преобразования точки является переносом данной точки на другую (совмещенную) систему координат. Этот процесс описывается системой уравнений:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formuls.png?raw=true) 

Где:
  * x, y – координаты точки в "старой" системе координат;
  * x', y' – координаты точки в "новой" системе координат;
  * x(0)',y(0)' – расстояние сдвига одной системы координат по оси абсцисс и ординат соответственно;
  * a(ij) – числа описывающие параметры преобразования. При этом они связаны неравенством:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_2d.png?raw=true) 

Справедлива будет следующая запись:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/%20matrix_2d_2.png?raw=true)

Однако матрица 2 x 2 имеет следующие проблемы:
  * Матрица имеет ограничения по количеству выполняемых операций (Она дает возможность совершать такие действия, как масштабирование, вращение, отражение и наклон. Но, к сожалению, мы не сможем выполнить перемещение объекта);
  * Трудоемкий процесс расчета композиции преобразований. Например, композиция из одного преобразования уже выглядит громоздко, а на практике выполняются десятки преобразований:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix2x2_multi.png?raw=true)

Помимо того, что такую формулу сложно считать, запись, когда композиция состоит из нескольких преобразований, также вызывает затруднение. 

Для упрощения данной формулы, давайте введем еще одну координату (z). Другими словами, выполним погружение двумерного пространства в трехмерное и попадем в однородную систему координат. 

С математической точки зрения, однородным представлением n-мерного объекта называют его представление в (n+1)-мерном пространстве, полученном путем добавления еще одного множителя, так называемого, скалярного множителя. Итак, чтобы найти однородные координаты точки, необходимо выполнить следующее. Предположим, что на плоскости лежит точка T с координатами x и y.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/image1.png?raw=true)

Однородными координатами данной точки называется любая тройка чисел, одновременно не равных нулю (w1, w2, w3), которые связаны с координатами точки T следующими соотношениями:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/x=w1w3.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/y=w2w3.png?raw=true)

Точку T можно представить тройкой своих координат w1,w2,w3: любая выбранная точка лежащая на прямой, которая соединяет начало координат O(0,0,0), c точкой T' (x,y,1). Очевидно, что точка T' однозначно определяет точку T, также как и любая точка T'' с координатами (xh,yh,h). Здесь h - является скалярным множителем. Мы сейчас описали точку в однородных координатах. Обычно точку представляют как (x: y: 1), то есть h = 1. Тем не менее, используется общая форма записи (w1 : w2 : w3). Такая форма записи не должна вас смущать, если, вдруг вы с ней где-нибудь столкнетесь. Как вы знаете, существует и декартова система пространства, где точки также описываются тройкой координат (x,y,z). Чтобы отличить однородное описание точки от привычного декартовского, в однородном описании координаты точки разделяют не запятой, а двоеточием. Итак, теперь запишем общую формулу аффинного преобразования. 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_af.png?raw=true)

При умножении этих матрицы, мы получаем два основных уравнения, которые описывают перенос точки от одной системы координат в совмещенную.

Аффинные преобразования являются линейными. Линейные преобразования подчиняются следующим правилам:
  * Прямые (плоскости) переходят в прямые (плоскости);
  * Пересекающиеся прямые (плоскости) в пересекающиеся;
  * Параллельные в параллельные.

Мы теперь знаем, что для двумерных однородных координат используется матрица линейного преобразования общего вида. 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_at_3x3.png?raw=true)

  Данная матрица содержит 4 подматрицы, которые составлены по принципу разнородности их влияния на результат линейного преобразования. 

1. Ниже изображена подматрица 2 х 2.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/submatrix_2x2_1.png?raw=true)

  Элементы данной подматрицы отвечают за следующие операции: масштабирование, вращения и сдвиг;
  
2. Следующая подматрица 1 х 2.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/submatrix_1x2_2.png?raw=true)

  Элементы данной подматрицы отвечают за операцию перемещение (смещение).
  
3. Подматрица 2 х 1.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/submatrix_2x1_3.png?raw=true)

Элементы данной подматрицы отвечают за задание проекции. 

4. И, наконец, подматрица размерностью 1 x 1.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/submatrix_1x1_4.png?raw=true)

  Элемент данной подматрицы отвечает за однородное изменение масштаба.
  
  Таким образом, мы рассмотрели элементарные преобразования. Более сложные преобразования представляют цепочку последовательно выполняемых элементарных преобразований.
  Все преобразования сводятся к поиску положения точки после выполнения операции преобразования над ней:
  
![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_at_basic.png?raw=true)

Здесь:
  * T - применяемая матрица преобразования;
  * x, y - координаты точки в исходном вспомогательном снимке;
  * x', y' - координаты точки на совмещенном вспомогательном снимке;
  * z, z' - координаты точки на исходном и совмещенном вспомогательном снимке;
  
### Матрицы в Android 

В документации SDK для Android довольно скудно представлена информация про класс Matrix и нет никаких упоминаний про аффинные преобразования. То есть разработчики изначально предполагают, что если вы пишите приложения для Android, то должны знать как выполняются аффинные преобразования.
К сожалению, чаще всего это не так. Если взглянуть на класс матрицы в Android [android.graphics.Matrix](https://android.googlesource.com/platform/frameworks/base/+/master/graphics/java/android/graphics/Matrix.java), то можно увидеть много oops(); и вызовов методов native_. В общем, никакой полезной информации там нет. Это связано с тем, что реальные операции выполняются классом Matrix, написанном на C++. То есть, Matrix.java является прослойкой между пользовательским кодом и нативными вызовами. 

Полагаю, что кого-нибудь заинтересует реализация класса Matrix.cpp. 
[Здесь](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/libs/hwui/jni/android_graphics_Matrix.h) вы найдете интерфейс Matrix.h, а [здесь](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/libs/hwui/jni/android_graphics_Matrix.cpp) собственно реализацию Matrix.cpp. 
Но и это еще не все. Matrix.cpp так же, в свою очередь, является посредником, так как все операции над матрицами он делегирует классу SkMatrix. Интерфейс этого класса можете глянуть [здесь](https://android.googlesource.com/platform/external/skia/+/master/include/core/SkMatrix.h) (SkMatrix.h) и его реализацию [здесь](https://android.googlesource.com/platform/external/skia/+/master/src/core/SkMatrix.cpp) (SkMatrix.cpp). 
    
  Если мы, например, начнем изучать класс матриц, со страницы  документации от Google ([ссылка](https://developer.android.com/reference/android/graphics/Matrix)), то первое, что нам показывают — это список констант этого класса. Каждая константа является индексом в массиве который описывает матрицу. Так как размер матрицы 3 x 3, то размер оперируемого массива = 9, что соответствует количеству констант в классе. 
  Смысл параметров массива рассмотрим чуть позже, а сейчас предлагаю ознакомиться с основными методами класса Matirx.java.  
    
### Post и Pre функции трансформаций в Android

Предполагаю, что вы познакомились с документацией от Google по классу матриц. Скорее всего вы заметили, что мы можем выполнить различные операции преобразования несколькими способами. А именно, используя методы post и pre:

Post-методы:

~~~ java

 ...
   
  postTranslate(float dx, float dy) 
  postScale(float sx, float sy)
  postScale(float sx, float sy, float px, float py)
  postRotate(float degrees)
  postRotate(float degrees, float px, float py)
  postSkew(float kx, float ky)
  postSkew(float kx, float ky, float px, float py)
  postConcat(Matrix other)
   
  ...

~~~

Pre-методы:

~~~ java

  ...
  
  preTranslate(float dx, float dy) 
  preScale(float sx, float sy)
  preScale(float sx, float sy, float px, float py)
  preRotate(float degrees)
  preRotate(float degrees, float px, float py)
  preSkew(float kx, float ky)
  preSkew(float kx, float ky, float px, float py)
  preConcat(Matrix other)

  ...

~~~

Из документации методов мы видим, что порядок умножения матриц меняется. В случае с post-методами: 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_post.png?raw=true)

И pre-методами:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_pre.png?raw=true)

Где:
  * M' - новая матрица;
  * T - матрица преобразования;
  * M - старая матрица;

Известно, что одно из свойств умножения матриц это некоммутативность, т.е.:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/AB.png?raw=true)

А это означает, что при выполнении pre- и post- операций, мы получим разные результаты.

Решения из коробки предлагают нам богатый арсенал для управления матрицами. С помощью методов postConcate и preConcate вы можете задать свое преобразование, которое не предусмотрели разработчики (например, отражение). 

 > В стандартном JDK тоже присутствуют классы для управления матрицами и для выполнения аффинных преобразований. Также там имеются аналогичные методы для умножения матриц. Один из таких методов preConcatenate(...), который является аналогом preConcat(...). Важно знать, что в JDK порядок умножения матриц отличается от версии, которую предлагает Google. К примеру, при выполнении  pre - операции, формула умножения будет такой: M' = T * M. 

### Подготовка ресурсов

Прежде чем перейти к обсуждению преобразований, давайте подготовим рабочую среду, в которой мы будем выполнять преобразования. Первым делом, предлагаю скачать картинку в разрешении 240x320 пикселей.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/image.jpg?raw=true)

  Создаём новый пустой проект, в котором описываем activity_main (Layout):
  
~~~ xml

<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scaleType="matrix"
        android:src="@drawable/image" />

</FrameLayout>

~~~

Обратите внимание ImageView размещается по всей доступной родительской площади. Чтобы получить возможность выполнять аффинные операции над выбранным изображением, устанавливаем атрибут scaleType в значение matrix. 

  В своих преобразованиях я буду работать с размерами изображений. Ширину и высоту картинки получаем с помощью методов getIntrinsicWidth() и getIntrinsicHeight() соответственно, у экземпляра класса Drawable. Важно понимать, что эти методы иногда возвращают неправильные  размеры изображения. Это происходит потому, что растровое изображение лежит в неправильной папке drawable. Если хотите получить точные размеры изображения, то необходимо позаботиться о наличии изображений, подходящих для различных плотностей пикселей, и размещении их в соответствующих папках drawable. Подробнее предлагаю ознакомиться в данной [статье](https://developer.android.com/training/multiscreen/screendensities?hl=ru). 
  
  Я буду тестировать преобразования на эмуляторе устройства Nexus 4. Учитывая плотность пикселей, я разместил изображение в папке drawable-xhdpi с именем image, чтобы буду получать точные размеры изображения.

В исходном состоянии левый верхний угол изображения соответствует центру координатных осей – матрица координат картинки получается следующая:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_input_coords.png?raw=true)

Вот так выглядит начальное изображение без преобразований:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/sample_1_before.png?raw=true)

 Чтобы получить исходную матрицу изображения, напишем метод `printMatrixValues`, который будет выводить в лог параметры матрицы, переданной в параметрах метода.
 
~~~ kotlin

package com.mercuriy94.matrix.affinetransformations

import android.graphics.Matrix
import android.os.Bundle
import android.util.Log
import android.widget.ImageView
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.doOnLayout

class MainActivity : AppCompatActivity() {

    companion object {
        private const val TAG = "MainActivity"
    }

    private lateinit var imageView: ImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        imageView = findViewById(R.id.imageView)
        imageView.doOnLayout { printMatrixValues(imageView.imageMatrix) }

    }

    private fun printMatrixValues(matrix: Matrix) {
        val values = FloatArray(9)
        matrix.getValues(values)
        Log.i(
            TAG,
            "Matrix values:\n" +
                    "${values[Matrix.MSCALE_X]} ${values[Matrix.MSKEW_X]} ${values[Matrix.MTRANS_X]}\n" +
                    "${values[Matrix.MSKEW_Y]} ${values[Matrix.MSCALE_Y]} ${values[Matrix.MTRANS_Y]}\n" +
                    "${values[Matrix.MPERSP_0]} ${values[Matrix.MPERSP_1]} ${values[Matrix.MPERSP_2]}\n"
        )
    }
}

~~~

Вывод:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matix_values_log_input.png?raw=true)

  Исходная матрица является единичной, а это означает, что её можно не учитывать в вычислениях, так как она не оказывает влияние на результат:
  
![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/android_input_matrix.png?raw=true)

  Метод `printMatrixValues(Matrix matrix) ` нам еще пригодится для вывода в лог окончательной матрицы преобразований.
  
  Обратите внимание, что преобразования сводятся к умножению точки, лежащей на плоскости, на матрицу преобразования. Результатом являются координаты совмещенной точки. В таком случае, необходимо добавить функционал, который будет подтверждать или опровергать правильность вычислений. Добиться этого можно, печатая в лог координаты точек совмещенного изображения. Класс Matrix из SDK Android для этой цели не подходит, так как он работает только с матрицами 3x3. Нам, как минимум, требуется матрица размерностью 3x4 для расчета координат углов изображения. Подключаем эффективную библиотеку для работы с матрицами ([Ссылка на главную страницу библиотеки](http://ejml.org/wiki/index.php?title=Main_Page)). 
    Чтобы подключить бибилотеку к проекту, нужно в файле build.gradle модуля вашего проекта добавить строчку в блоке dependencies рядом с остальными вашими зависимостями.
    
~~~ groovy

...

dependencies {

    ...
    implementation 'org.ejml:ejml-all:0.40'
    ...
}
    
...

~~~     
    
  Теперь напишем метод, который будет показывать координаты изображения после применения преобразования:
 
~~~ kotlin

...

import org.ejml.simple.SimpleMatrix

...

     /**
     * Метод распечатывает координаты углов изобаржения в исходном состоянии и
     * после применения матрицы преобразования [matrix].
     *
     * @param matrix - матрица преобразования
     * */
    private fun printImageCoords(matrix: Matrix) {
        val imageWidth = imageView.drawable.intrinsicWidth.toFloat()
        val imageHeight = imageView.drawable.intrinsicHeight.toFloat()
        //Создаем матрицу 3 х 4 с исходными координатами углов картинки
        val inputCoords = arrayOf(
            floatArrayOf(0f, imageWidth, imageWidth, 0f),
            floatArrayOf(imageHeight, imageHeight, 0f, 0f),
            floatArrayOf(1f, 1f, 1f, 1f)
        )
        val matrixCoords = SimpleMatrix(inputCoords)
        println("Исходные координаты углов изображения:\n")
        matrixCoords.print()
        val values = FloatArray(9)
        imageView.imageMatrix.getValues(values)
        //Создаем двухмерную матрицу преборазования в SimpleMatrix
        val matrixValues = arrayOf(
            floatArrayOf(values[Matrix.MSCALE_X], values[Matrix.MSKEW_X], values[Matrix.MTRANS_X]),
            floatArrayOf(values[Matrix.MSKEW_Y], values[Matrix.MSCALE_Y], values[Matrix.MTRANS_Y]),
            floatArrayOf(values[Matrix.MPERSP_0], values[Matrix.MPERSP_1], values[Matrix.MPERSP_2])
        )
        val imageMatrix = SimpleMatrix(matrixValues)
        //Выполним умножение матриц чтобы получить искомый результат
        val resultCoords = imageMatrix.mult(matrixCoords)
        println("Координаты углов изображения после применения матрицы трансформации:")
        resultCoords.print()
    }
    
...

~~~
В качестве параметра передается матрица преобразования для который мы хотим расчитать координаты изображения после ее применения. Далее мы получаем исходные размеры изображения и на их основе создаем первончальную двумерную матрицу координат `inputCoords`. С помощью данной двумерной матрицы создаем объект класса SimpleMatrix передав `inputCoords` в конструктор SimpleMatrix. Вообще у данного класса имеется несколько конструкторов. Для подробного изучения этого класса, можно перейти по [ссылке](http://ejml.org/javadoc/org/ejml/simple/SimpleMatrix.html). Далее подобным образом выполняем перевод, переданной в параметрах матрицы преобразования `matrix` в класс SimpleMatrix. Умножение матрицы выполняется  методом `mult` из класса SimpleMatrix, результатом которого является новая результирующая матрица которая описывает новые координаты углов изображения. 
    
Данный класс умеет самостоятельно выводить значения с помощью методов `print()`. Вывод попадает в Logcat c тегом = "System.out".
Ниже представлен результат координат исходной матрицы:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/sample_result_coords_img.png?raw=true)
 
Наконец-то все готово! 
    В заключение первой статьи, предлагаю рассмотреть несложное преобразование.

### Перемещение (Сдвиг) (Transition)

  По моему мнению, сдвиг - самый простой вид преобразования, его также называют параллельный перенос. Преобразование сдвига  устанавливает соответствие между координатами точки в двух координатных системах, одна из которых сдвинута относительно другой на расстояние дельта X по горизонтали и дельта Y по вертикали.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans.png?raw=true)

Если начать раскрывать скобки, то получим следующую систему уравнений:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_2.png?raw=true)

Чтобы решить такую систему, нам необходимо рассмотреть каждое равенство в отдельности. Начнем с первого:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_x.png?raw=true)

Нам нужно выбрать такие значение коэффициентов a,b и с, чтобы данное равенство было верно.
Мы с вами уже знаем, что z всегда равна 1, это обсуждалось в разделе "Аффинные преобразования". 
Так как левая часть уравнения не содержит параметра y, следовательно, b = 0. Давайте взглянем на получившееся уравнение:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_x_2.png?raw=true)

Теперь остается подобрать коэффициенты a и с. Следовательно:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/a=1%20c=tx.png?raw=true)

Теперь перейдем ко второму уравнению:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_y.png?raw=true)

Здесь, как и в первом уравнении, выполняем поиск коэффициентов d,e и f для выполнения данного равенства.
Знаем, что z = 1. Левая часть не содержит параметра x, значит d = 0.
Получаем уравнение:
 
 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_y_2.png?raw=true)
 
Из этого уравнения получаем:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/e=1%20f=ty.png?raw=true)
 
C третьим уравнением все еще легче. 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_z.png?raw=true)

Чтобы удовлетворялось данное равенство, принимаем g и h равные нулю. 
Так как z = 1. То получаем i = 1.

В итоге, для выполнения смещения получаем матрицу размерностью 3 x 3:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/trans_matrix_final.png?raw=true) 

Скорее всего, если вы до сих пор читаете эту статью, то вам действительно интересно, и все понятно.  Следовательно, я не вижу смысла расписывать каждое преобразование с такой подробностью =). 

Чтобы понимать, как эту формулу использовать, давайте рассмотрим простой пример. На плоскости размещен квадрат, координаты которого описывает матрица 4 x 2. 

По всей логике, мы должны умножить матрицу координат на матрицу смещения. Но так как размерности матрицы различаются, то мы добавляем к матрице координат строку, заполненную единицами. И получаем такую формулу:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formala_matrtix_trans_multiplay.png?raw=true)

А теперь давайте рассмотрим данный тип преобразования на практике.
В качестве примера, сделаем перенос изображения в центр контейнера ImageView, чтобы центр изображения и экрана совпадали. Для того, чтобы вычислить расстояние сдвига по оси абсцисс, нам необходимо от ширины контейнера отнять ширину картинки и поделить на 2. Аналогично поступаем и с высотой, чтобы вычислить сдвиг по оси ординат. В моем случае ширина контейнера равна 768, а высота 1024. Теперь рассчитаем сдвиги: 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/trans_sample_tx_ty_calc.png?raw=true)

Получившаяся матрица однозначно описывает координаты углов изображения в контейнере ImageView.

Теперь вычислим конечные координаты углов изображения:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/sample_trans_results.png?raw=true)

А теперь реализуем в Android (все внимание на метод `translateToCenter`):

~~~ kotlin

package com.mercuriy94.matrix.affinetransformations

    ...

class MainActivity : AppCompatActivity() {

    companion object {
        private const val TAG = "MainActivity"
    }

    private lateinit var imageView: ImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        imageView = findViewById(R.id.imageView)
        imageView.doOnLayout { translateToCenter() }
    }

    private fun translateToCenter() {
        val transformMatrix = Matrix()
        val tx = (imageView.measuredWidth - imageView.drawable.intrinsicWidth) / 2f
        val ty = (imageView.measuredHeight - imageView.drawable.intrinsicHeight) / 2f
        val matrixValues = floatArrayOf(
            1f, 0f, tx,
            0f, 1f, ty,
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

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/image_sample_trans_result.png?raw=true)

Вывод в Logcate:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matix_values_log_input_trans.png?raw=true)

Все верно! Рассчитанные и получившиеся координаты изображения после преобразования совпадают.

Вся соль в методе translateToCenter(), который и выполняет сдвиг в центр контейнера. В данном примере я вручную заполняю значения матрицы преобразования (matrixValues). Это достаточно утомительное занятие, поэтому в классе matrix есть два метода postTranslate и preTranslate. Различие между post и pre методах обсуждалось выше в разделе «Post и Pre функции трансформаций в андройде». В качестве параметров, данные методы принимают расстояние сдвига tx и ty. Давайте перепишем метод translateToCenter(), как это выглядело бы в реальной задаче центрирования.

~~~ kotlin

  ...

    private fun translateToCenter() {
        val tx = (imageView.measuredWidth - imageView.drawable.intrinsicWidth) / 2f
        val ty = (imageView.measuredHeight - imageView.drawable.intrinsicHeight) / 2f
        val imageMatrix = imageView.imageMatrix
        imageMatrix.postTranslate(tx, ty)
        imageView.imageMatrix = imageMatrix
    }

...

~~~

Можете поменять postTranslate на preTranslate, но результат останется таким же. Потому что базовая матрица является единичной. Собственно, поэтому мы не учли ее в вычислениях. Но если бы до этого применялось какое-нибудь преобразование, то результаты бы различались.

Во второй статье рассматриваются такие виды преобразования как масштабирование и вращение!

Спасибо, за внимание!
