# Матрицы в Android, первая часть
### Актуальность темы
  Если вы разрабатываете приложения под ОС Android, то, время от времени, сталкиваетесь с нетривиальными задачами. Например, дизайнер выдал макет с нестандартными элементами представления. И вы, прикинув сложность реализации, начинаете искать библиотеки, которые уже делают то, что вам нужно. В лучшем случае, вы их находите, затем интегрируете с наименьшими усилиями – все довольны, и все счастливы. 
 
  Бывает и так, что вы находите приблизительное решение вашей задачи, но, чтобы получить желаемый результат, нужно внести изменения в существующее решение. В этом случае есть вероятность фиксить баги в библиотеке.

  Наихудший вариант: вы не находите ничего похожего на элемент представления из макета. Вы объясняете дизайнеру, что писать с нуля эту вьюшку затратно по времени и просите заменить на аналогичное более простое решение. В ответ от дизайнера вы слышите, что это - ключевой элемент дизайна и заменить его нельзя, так как это нарушит общую концепцию и т.д. Ну что-же, смирившись со своей участью, вы начинате искать способы решения. 

### Введение
На протяжении трёх статей я попытаюсь раскрыть и объяснить основные принципы трансформаций двухмерного пространства с помощью аффинных преобразований, а также рассмотрим примеры часто используемых трансформаций. Для демонстрации примеров я буду использовать компонент ImageView c установленым параметром ScaleType = MATRIX. Отметим, что все трансформации применяемые к ImageView, так же могут быть применены к вашим кастомным view - элементам, если вдруг для отрисовки вы используете шейдеры (Shaders).

Хорошо, а теперь расмотрим какие темы будут расматриваться на протяжении всей серии статей.

Первая статья:
  * Афинные преобразования;
  * Матрицы в Android;
  * Post и Pre функции трансформаций в Android;
  * Перемещение (Transition);

Вторая статья:
  * Масштабирование (Scale);
  * Поворот (Rotate);

Третья статья:
  * Наклон (Skew);
  * Отражение (Sharing);


Каждый раздел преобразования включает в себя примеры, на которых подробно рассматриваются изменения плоскости объекта.

Чего вы не найдете в данной статье, так это работу с изображением в 3D пространстве.

Прежде чем перейти к основной теме обсуждения, предлагаю вам перейти по [ссылке](http://math1.ru/education.html#matrix) и освежить в памяти (или ознакомиться) с матрицами и операциями над ними.

### Аффинные преобразования 

  Прежде чем перейдем к обсуждению аффиных преобразований. Нужно понять несколько моментов. Прежде чем изображение появится на экране, оно должно быть как-то описанно  в графической системе устройства. Первоначально свойства объекта описываются в системе координат этого самого объекта. Это приводит к тому, что, при изменении пространтсва, объект остается неизменным с течением времени. Например, на экране находится 2 разные картинки, и каждая из них будет описываться в своей системе координат, и преобразование плоскости одной картинки не влечет изменение другой. Таким образом, введем новое понятие: мировая система координат, где располагаются все объекты.
  Пользователи, по своей сути, являются наблюдателями и, так же, живут в своей системе координат (система координат наблюдателя). Системы координат объектов и наблюдателя являются трехмерными. Получается, в процессе обработки изображения, система координат объекта преобразуется в двумерную координатную плоскость. 
  Объекты могут эволюционировать относительно наблюдателя. То есть менять свое расположение, сжиматься, вращаться и т.д. Есть два решения для описания этого процесcа. Первый подход предпологает, что система координат наблюдателя неподвижена, а мировая система координат и система координат объекта могут эволюционировать. Таким образом, удобно наблюдать изменения объекта во времени. Второй подход предпологает, что неподвижна мировая система координат, а системы координат наблюдателя и объекта эволюционируют. В операционной системе андройд используется первый подход.
Сейчас важно было понять, что имеется 2 плоскости:

1. Плоскость объекта, над котороый мы будем производить преобразования;
2. Плоскость наблюдателя, благодаря которой мы можем наблюдать изменения объекта;

  Вся работа с объектами происходит в прямогульной декартовой системе координат. Каждая точка в данной системе описывается парой координат x и y. Процесс аффинного преобразования точки является переносом данной точки на другую (совмещенную) систему координат. Это процесс описывается системой уравнений:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formuls.png?raw=true) 

Здесь:
  * x, y - координаты точки в "старой" системе координат;
  * x', y' - координаты точки в "новой" системе координат;
  * x(0)',y(0)' - расстояние сдвига одной системы координат по оси абцисс и ординат соответсвтенно;
  *  a(ij) - числа описывающие параметры преобразования, и они связаны неравенством:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_2d.png?raw=true) 

Справедлива будет следующая запись:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/%20matrix_2d_2.png?raw=true)

Но матрица 2 x 2 имеет следующие проблемы:
  * Ограничена в количестве выполняемых операций (Такая матрица может выполнять операции: масштабирование, вращение, отражение и наклон. Но, к сожалению, мы не сможем выполнить смещение);
  * Трудоемкий процесс расчета композиции преобразований. Например, композиция из одного преобразования уже выглядит не очень хорошо, а на практике выполняются десятки преобразований:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix2x2_multi.png?raw=true)

Помимо того, что это сложно считать, вызывает затруднение и запись, когда композиция состоит из нескольких преобразований. 

В таком случае, было принято решение ввести еще одну координату (z). Другими словами, выполняем погружение двумерного пространства в трехмерное. Так мы попадаем в однородную систему координат.
С математической точки зрения, однородным представлением n-мерного объекта называют его представление в (n+1)-мерном пространтсве, полученного добавлением еще одного множителя, так называемого, скалярного множителя.
  Итак, чтобы найти однородные координаты точки, необходимо выполнить следующее.
Предположим, что на плоскости лежит точка T с координатами  x и y.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/image1.png?raw=true)

Однородными координатами данной точки называется любая тройка одновременно не равных нулю чисел(w1, w2, w3), которые связаны с координатами точки T следующими соотношениями:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/x=w1w3.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/y=w2w3.png?raw=true)

Точку T можно представить тройкой своих координат w1,w2,w3: любая выбранная точка лежащая на прямой, котороя соединяет начало координат O(0,0,0), c точкой T' (x,y,1).
Очевидино, что точка T' однозначно определяет точку T, также как и любая точка T'' с координатами (xh,yh,h). Здесь h - является скалярным множителем. Мы сейчас описали точку в однородных координатах. Обычно точку представляют как (x: y: 1). То есть h = 1. Тем не менее, используется общая форма записи (w1 : w2 : w3). Так что, пусть вас такая запись не смущает, если, вдруг, где-нибудь встретите. 
Как вы знаете, существует и декартова система пространства, где точки также описываются тройкой координат (x,y,z). И, чтобы отличить однородное описание точки от привычного декартовского описания, в однородном описании координаты точки разделяют не запятой, а двоеточием. Итак, теперь запишем общую формулу аффиного преобразования. 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_af.png?raw=true)

При умножении этих матрицы, получаем два основных уравнения, которые описывают перенос точки от одной системы координат в совмещенную.

Аффиные преобразования являются линейными. Правила, которым подчиняются линейные преобразования:
  * Прямые (плоскости) переходят в прямые (плоскости);
  * Пересекающиеся прямые (плоскости) в пересекающиеся;
  * Параллельные в параллельные.

Мы теперь знаем, что для двумерных однородных координат используется матрица линнейного преобразования общего вида. 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_at_3x3.png?raw=true)

  Данная матрица содержит 4 подматрицы. Которые составлены по принципу разнородности их влияния на результат линейного преобразования. 

1. Ниже изображена подматрица 2 х 2.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/submatrix_2x2_1.png?raw=true)

  Элементы данной подматрицы отвечают за операции: масштабирование, вращения и сдвиг;
  
2. Следующая подматрица 1 х 2.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/submatrix_1x2_2.png?raw=true)

  Элементы данной подматрица отвечают за операцию перемещения (смещения).
  
3. Подматрица 2 х 1.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/submatrix_2x1_3.png?raw=true)

Элементы данной подматрицы отвечают за задание проекции. 

4. И наконец подматрица размерностью 1 x 1.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/submatrix_1x1_4.png?raw=true)

  Элемент данной подматрицы отвечает за однородное изменение масштаба.
  
  По своей сути это - элементарные преобразования. Более сложные преобразования представляют цепочку последовательно выполняемых элементарных преобразований.
  В конечном итоге, мы ищем положение точки после выполнения операции преобразования над ней:
  
![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_at_basic.png?raw=true)

Здесь:
  * T - применяемая матрица преобразования;
  * x, y - координаты точки в исходной вспомогательном снимке;
  * x', y' - координаты точки на совмещеном вспомогательном снимке;
  * z, z' - координаты точки на исходном и совмещеном вспомогательном снимке;
  
### Матрицы в Android 

В документации SDK для Android, содержится довольно скудная информация про класс Matrix. И не содержится абсолютно никакой информации про аффинные преобразования. То есть разработчики изначально предполагают, что если вы пишите приложения для Android, то должны знать как выполняются аффинные преобразования. К сожалению, чаще всего это не так.
Если взглянуть на класс матрицы в Android [android.graphics.Matrix](https://android.googlesource.com/platform/frameworks/base/+/master/graphics/java/android/graphics/Matrix.java), то можно увидеть много oops(); и вызовов методов native_. В общем, никакой полезной информации там нет. Это потому, что реальные операции выполняются классом Matrix, написанном на C++. То есть, Matrix.java является прослойкой между пользовательским кодом и нативными вызовами. 
Полагаю, что кого-нибудь заинтересует реализция класса Matrix.cpp. 
[Здесь](https://android.googlesource.com/platform/frameworks/base/+/master/core/jni/android/graphics/Matrix.h) вы найдете интерфейс Matrix.h, а [здесь](https://android.googlesource.com/platform/frameworks/base/+/master/core/jni/android/graphics/Matrix.cpp) сообственно реализацию Matrix.cpp. 
Но и это не все. Matrix.cpp также, в свою очередь, является посредником, так как все операции над матрицами он делегирует классу SkMatrix. Интерфейс этого класса можете глянуть [здесь](https://android.googlesource.com/platform/external/skia/+/master/include/core/SkMatrix.h) (SkMatrix.h) и его реализацию [здесь](https://android.googlesource.com/platform/external/skia/+/master/src/core/SkMatrix.cpp) (SkMatrix.cpp). 
    
  Если мы начнем изучать класс матриц, со страницы  документации от Google ([ссылка](https://developer.android.com/reference/android/graphics/Matrix)). Первое, что нам показывают - это список констант этого класса. Каждая константа является индексом в массиве который описывает матрицу. Так как матрицы 3 x 3, то размер оперируемого массива = 9, что соответствует количеству констант в классе. 
  Смысл параметров массива рассмотрим чуть позже, а сейчас предлагаю ознакомиться с основными методами класса Matirx.java.  
    
### Post и Pre функции трансформаций в Android

Предполагаю, что вы кратенько изучили документацию от Google по классу матриц. То скорее всего вы заметили, что мы можем выполнить различные операции преобразования несколькими способами. А именно, используются методы post и pre:

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

Из документации методов видим порядок умножения матриц меняется. В случае с post-методами: 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_post.png?raw=true)

И pre-методами:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_pre.png?raw=true)

Где:
  * M' - новая матрица;
  * T - матрица преобразования;
  * M - старая матрица;

Известно, что одно из свойств умножения матриц это не коммутативность, т.е.:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/AB.png?raw=true)

А это означает, что при выполнении pre- и post- операций, получим различные результаты.

Из коробки нам предлагается  богатый арсенал для управления матрицами. С помощью методов postConcate и preConcate вы можете задать свое преобразование, которого не предусмотрели разработчики (Например, отражение). 

 > В стандартном JDK тоже присутствуют классы для управления матрицами и выполнения аффинных преобразований. Также там имеются аналогичные методы для умножения матриц. Один из таких методов preConcatenate(...), который является аналогом preConcat(...). Но важно знать что в JDK порядок умножения матриц отличается от версии которую предлагает Google. К примеру, при выполнении  pre - операции, формула умножения такая: M' = T * M. Мне этот момент кажется довольно странным. 


### Подготовка ресурсов

Прежде чем перейти в обсуждению преобразований, подготовим рабочую среду, в которой будем выполнять преобразования. Первым делом, предлагаю скачать картинку в разрешении 240x320 пикселей.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/image.jpg?raw=true)

  Создаём новый пустой проект, в котором описываем activity_main (Layout):
  
~~~ xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
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

</LinearLayout>

~~~

Размещаем ImageView по всей доступной родительской площади. И, чтобы получить возможность выполнять аффинные операции над выбранным изображением, устанавливаем аттрибут scaleType в значение matrix. 

  В своих преобразованиях я буду работать с размерами изображений (не во всех). Ширину и высоту картинки получаем с помощью методов getIntrinsicWidth() и getIntrinsicHeight() соответственно у экземпляра класса Drawable. Важно понимать, что эти методы иногда возвращают неправильные  размеры изображения. Это происходит потому, что растровое изображение лежит в неправильной папке drawable. Если хотите получить точные размеры изображения, то необходимо позаботиться о наличии изображений, подходящих для различных плотностей пикселей, и размещении их в соответствующих папках drawable. Подробнее предлагаю ознакомиться в данной [статье](https://developer.android.com/training/multiscreen/screendensities?hl=ru). 
  Я буду тестировать на эмуляторе устройства Nexus 4. Учитывая плотность пикселей, я разместил изображение в папке drawable-xhdpi с именем image. Так я буду получать точные размеры изображения.

В исходном состоянии левый верхний угол изображения соответствует центру координатных осей, матрица координат картинки получается следующая:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matrix_input_coords.png?raw=true)

Вот так выглядит начальное изображение без преобразований:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/sample_1_before.png?raw=true)

 Чтобы получить исходную матрицу изображения, напишем метод `printMatrixValues`, который будет выводит в лог параметры матрицы переданной в параметрах метода.
 
~~~ java

package mercuriy94.com.matrix.affinetransformations;

  //Импорты
  ...

public class MainActivity extends AppCompatActivity {

    public static final String TAG = "MainActivity";

    ImageView imageView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageView = findViewById(R.id.imageView);
        imageView.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                imageView.getViewTreeObserver().removeOnGlobalLayoutListener(this);
                printMatrixValues(imageView.getImageMatrix());
            }
        });
    }

    private void printMatrixValues(Matrix matrix) {
        float[] values = new float[9];
        matrix.getValues(values);
        Log.i(TAG, String.format("Matrix values:\n" +
                        "%f %f %f\n" +
                        "%f %f %f\n" +
                        "%f %f %f",
                values[Matrix.MSCALE_X], values[Matrix.MSKEW_X], values[Matrix.MTRANS_X],
                values[Matrix.MSKEW_Y], values[Matrix.MSCALE_Y], values[Matrix.MTRANS_Y],
                values[Matrix.MPERSP_0], values[Matrix.MPERSP_1], values[Matrix.MPERSP_2]));
    }
}

~~~

Вывод:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matix_values_log_input.png?raw=true)

  Исходная матрица является единичной, а это означает, что её можно не учитывать в вычислениях, так как она не оказывает влияние на результат:
  
![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/android_input_matrix.png?raw=true)

  Метод `printMatrixValues(Matrix matrix) ` нам еще пригодится для вывода в лог окончательной матрицы преобразований.
  
  Обратите внимание, что преобразования сводятся к умножению точки лежащей на плоскости на матрицу преобразования.  Результатом являются координаты совмещенной точки. В таком случае необходимо добавить функционал, который будет подтверждать или опровергать правильность вычислений, добиться этого можно, печатая в лог координаты точек совмещенного изображения. Класс Matrix из SDK Android для этой цели не подходит, так как он работает только с матрицами 3x3. Нам как минимум требуется матрица размерностью 3x4, для расчета координат углов изображения. Подключаем эффективную библиотеку для работы с матрицами ([Ссылка на главную страницу](http://ejml.org/wiki/index.php?title=Main_Page)). 
Предполагаю, что процесс подключения библиотеки в проект, не вызовет затруднений.

  Теперь напишем метод, который будет показывать координаты изображения после применения преобразования:
 
~~~ java

...

import org.ejml.simple.SimpleMatrix;

...

      private void printImageCoords(Matrix matrix) {

        float imageWidth = imageView.getDrawable().getIntrinsicWidth();
        float imageHeight = imageView.getDrawable().getIntrinsicHeight();

        float[] inputCoords = new float[]{
                0f, imageWidth, imageWidth, 0f,
                imageHeight, imageHeight, 0f, 0f,
                1f, 1f, 1f, 1f};

        SimpleMatrix matrixCoords = new SimpleMatrix(3, 4, true, inputCoords);
        matrixCoords.print();

        float[] values = new float[9];
        matrix.getValues(values);
        SimpleMatrix imageImatrix = new SimpleMatrix(3, 3, true, values);
        SimpleMatrix resultCoords = imageImatrix.mult(matrixCoords);
        resultCoords.print();
    }
    
...

~~~
 
 Полгаю, что данный метод треубет комментариев, по этому давайте его разберем.
 
 В качестве параметра передается окончательная матрица преобразования. Далее получаем размеры изображения и на их основе создаем исходную матрицу координат. С этим нам помогает класс SimpleMatrix. У данного класса имеется несколько конструкторов. Для подробного изучения этого класса, предлагаю перейти по [ссылке](http://ejml.org/javadoc/). А мы остановимся на выбранном конструкторе. В качестве первых двух параметров мы передаем кол-во строк и столбцов соответственно, из которых будет состоять наша матрица. Третьим параметром передаем true, так как наша матрица является закодированной в строчный массив, то есть одномерный массив. И последним четвертым параметром сам массив значений матрицы.
Далее выполняем перевод матрицы переданной в параметрах, в удобный для расчета класс SimpleMatrix. 
Умножение матрицы выполняется  методом  `mult` из класса SimpleMatrix, результатом которого является новая результирующая матрица. 
Данный класс умеет самостоятельно выводить значения с помощью методов `print()`. Вывод попадает в Logcat c тегом = "System.out", что добавляет удобство в чтении координат углов изображения.
Ниже представлен результат координат исходной матрицы:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/sample_result_coords_img.png?raw=true)
 
Наконец-то все готово! 
В заключении первой статьи, предлагаю рассмотреть несложное преобразование.

### Перемещение (Сдвиг) (Transition)

  По моему мнению, это - самый простой вид преобразования, его также называют параллельный перенос. Преобразование сдвига  устанавливает соответствие между координатами точки в двух координатных системах, одна из которых сдвинута относительно другой на расстояние дельта X по горизонтали и дельта Y по вертикали.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans.png?raw=true)

Если начать раскрывать скобки, то получим следующую систему уравнений:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_2.png?raw=true)

Чтобы решить такую систему, нам необходимо рассмотреть каждое равенство в отдельности. Начнем с первого:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_x.png?raw=true)

Нам необходимо выбрать такие значение коэффициентов a,b и с, чтобы данное равенство было верно.
Мы с вами уже знаем, что z всегда равна 1, это обсуждалось в разделе "Аффинные преобразования". 
Так как левая часть уравнения не содержит параметра y, следовательно b = 0. Давайте взглянем на получившееся уравнение:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_x_2.png?raw=true)

Теперь остается подобрать коэффициенты a и с. Следовательно:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/a=1%20c=tx.png?raw=true)

Теперь перейдем ко второму уравнению:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_y.png?raw=true)

Здесь, как и в первом уравнении выполняем поиск коэффициентоф d,e и f для выполнения данного равенства.
Знаем, что z = 1. Левая часть не содержит параметра x, значит d = 0.
Получаем уравнение:
 
 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_y_2.png?raw=true)
 
Из этого уравнения получаем:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/e=1%20f=ty.png?raw=true)
 
C третьим уравннием все еще легче. 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formula_trans_z.png?raw=true)

Чтобы удовлитврорялось данное равенство, принимаем g и h равные нулю. 
Так как z = 1. То получаем i = 1.

В итоге, для выполнения смещения получаем матрицу размерностью 3 x 3:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/trans_matrix_final.png?raw=true) 

Скорее всего, если вы до сих пор читаете эту статью, то вам действительно интересно, и все понятно.  Следовательно, я не вижу смысла расписывать каждое преобразование с такой подробностью :=). 

Чтобы понимать, как эту формулу использовать, давайте рассмотрим простой пример:
На плоскости размещен квадрат, координаты которого описывает матрица 4 x 2. 

По всей логике, мы должны умножить матрицу координат на матрицу смещения. Но так как размерности матрицы различаются, то мы добавляем к матрице координат строку заполненную единицами. И получаем такую формулу:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/formala_matrtix_trans_multiplay.png?raw=true)

А теперь давайте рассмотрим данный тип преобразования на практике.
В качестве примера, сделаем перенос изображения в центр контейнера ImageView, чтобы центр изображения и экрана совпадали. Для того, чтобы вычислить расстояние сдвига по оси абсцисс, нам необходимо от ширины контейнера отнять ширину картинки и поделить на 2. Аналогично поступаем и с высотой, чтобы вычислить сдвиг по оси ординат. В моем случае ширина контейнера равна 768, а высота 1024. Теперь рассчитаем сдвиги: 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/trans_sample_tx_ty_calc.png?raw=true)

Получившаяся матрица однозначно описывает координаты углов изображения в контейнере ImageView.

Теперь вычислим конечные координаты углов изображения:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/sample_trans_results.png?raw=true)

А теперь реализуем в Android (все внимание на метод `translateToCenter`):

~~~ java

package mercuriy94.com.matrix.affinetransformations;

  //Импорты
  ...
    
public class MainActivity extends AppCompatActivity {

    public static final String TAG = "MainActivity";
  
    ImageView imageView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageView = findViewById(R.id.imageView);
        imageView.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {


            @Override
            public void onGlobalLayout() {
                imageView.getViewTreeObserver().removeOnGlobalLayoutListener(this);
                translateToCenter();
            }
            
        });
    }


    private void translateToCenter() {
        Matrix transformMatrix = new Matrix();

        float tx = (imageView.getMeasuredWidth() - imageView.getDrawable().getIntrinsicWidth()) / 2f;
        float ty = (imageView.getMeasuredHeight() - imageView.getDrawable().getIntrinsicHeight()) / 2f;

        float[] matrixValues = new float[]{
                1f, 0f, tx,
                0f, 1f, ty,
                0f, 0f, 1f};

        transformMatrix.setValues(matrixValues);

        Matrix imageMatrix = imageView.getImageMatrix();

        imageMatrix.postConcat(transformMatrix);
        imageView.setImageMatrix(imageMatrix);
        printImageCoords(transformMatrix);
    }
  
  ...
  
}

~~~

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/image_sample_trans_result.png?raw=true)

Вывод в Logcate:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%201/Resources/Images/matix_values_log_input_trans.png?raw=true)

Все верно! Рассчитанные и получившиеся коордианты изображения после преобразования совпадают.

Вся соль в методе translateToCenter(), который и выполняет сдвиг в центр контейнера. В данном примере я в ручную заполняю значения матрицы преобразования (matrixValues). Это достаточно утомительное занятие, поэтому в классе matrix есть два метода postTranslate и preTranslate. Различие между post и pre методах обсуждалось выше в разделе "Post и Pre функции трансформаций в андройде". В качестве параметров, данные методы принимают расстояние сдвига tx и ty. Давайте перепишем метод translateToCenter(), как это выглядело бы в реальной задаче центрирования.

~~~ java

  ...

    private void translateToCenter() {

        float tx = (imageView.getMeasuredWidth() - imageView.getDrawable().getIntrinsicWidth()) / 2f;
        float ty = (imageView.getMeasuredHeight() - imageView.getDrawable().getIntrinsicHeight()) / 2f;

        Matrix imageMatrix = imageView.getImageMatrix();

        imageMatrix.postTranslate(tx, ty);
        imageView.setImageMatrix(imageMatrix);
    }

...

~~~

Можете поменять postTranslate на preTranslate, но результат останется таким же. Потому что базовая матрица является единичной. Собственно, по этому мы не учли ее в вычислениях. Но если бы до этого применялось какое-нибудь преобразование, то результаты бы различались.

Во второй статье рассматривются такие виды преобразования как масштабирование и вращение!

Спасибо, за внимание!
