# Матрицы в Android, вторая часть

В первой статье, мы познакомились с аффинным преобразованием, в каких целях оно используется и рассмотрели преобразование = "Сдвиг".В первой статье, большая часть строк была отведена теории, то во второй части поговорим о практическом применений, где подробно рассмотрим преобразования масштабирование и вращения, с учетом опорной точки для этих преобразований. Эти преобразования, широко используется в обработке изображений, по этому думаю будет интересно. Поехали!

P.S.: Напоминаю, что информацию о подготовке проекта и ресурсов вы найдете в первой статье!

### Масштабирование изображения (Scale)

Преобразование масштабирования увеличивает или уменьшает размер изображения объекта в системе координат наблюдателя по-сравнению с исходным размером в системе координат этого самого объекта.
  Так по какому принципу происходит масштабирование изображений?

  Для начала, вы подумаете, и скажите. "Хм..., а можно ведь взять координату точки и умножить на коэффициент масштабирования, и так с другой координатой". И будете правы. Итак, для начала проименуем коэффициенты для каждой оси Sx и Sy, для осей абсцисс и ординат соответственно.

Получаем следующие равенства:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_scale.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_scale_final.png?raw=true)

Матрица 3 х 3 для применения масштабирования принимает следующий вид: 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrice_scale.png?raw=true)

Чтобы применить матрицу масштабирования, умножаем ее на матрицу координат:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_scale_example.png?raw=true)

Теперь рассмотрим применение масштабирвания в Android.

Будем растягивать исходное изображение в 2 раза по двум осям. Первым делом вычислим координаты углов изображения после преобразования:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/sample_scale_results.png?raw=true)
 
Посмотрите на реализацию:

~~~ java

package mercuriy94.com.matrix.affinetransformations;

//Импорты 
...

public class MainActivity extends AppCompatActivity {

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
                scale();
            }
        });
    }

  private void scale() {

        Matrix transformMatrix = new Matrix();

        float sx = 2f;
        float sy = 2f;

        float[] matrixValues = new float[]{
                sx, 0f, 0f,
                0f, sy, 0f,
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

Результат:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/image_sample_scale_result.png?raw=true)

Убеждаемся в паравильности расчетов:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matix_values_log_input_scale_coords.png?raw=true)

Скорее всего, вы уже догадались, что нам необязательно создавать матрицу преобразования. Для этого достаточно знать о методах postScale(...) и preScale(...). Эти методы включают в себя обязательные параметры: sx и sy. А также опциональные (необязательные) параметры: px и py. 
Вот так выглядит использование метода scale(), без матрицы преобразования:

~~~ java

...

  private void scale() {

        float sx = 2f;
        float sy = 2f;

        Matrix imageMatrix = imageView.getImageMatrix();

        imageMatrix.postScale(sx, sy);
        imageView.setImageMatrix(imageMatrix);
    }
  
...
~~~

Хорошо, теперь рассмотрим масштабирования вокруг опорной точки. Тут нам необходимо выполнить комбинацию преобразований:
1. Сместить плоскость изображение так, чтобы опорная точка совпалдала с началом координат плоскости наблюдателя. Простыми словами, выполнить сдвиг на обратные велечины опорной точки.
2. Применить масштабирование;
3. Выполнить сдвиг плоскости изображения на расстояние координат опорной точки в соответствующих осях.

Тогда матрица 3 х 3 для масштабирования вокруг опорной точки принимает вид:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_scale_anchor_point_formula.png?raw=true)

Пример:

Чтобы все было красиво, давайте перед этим выполним известную нам операцию переноса в центр контейнера. Таким образом, получим комбинацию преобразований. 
Как и прежде, сначала рассчитаем финальное положение углов изображения. Но, чтобы это выполнить, нам необходимо получить опорную точку для масштабирования. Так как изображение будет расположено в центре контейнера, то опорной точкой будет центр контейнера, координаты которой равны половине его ширины и высоты. В моем случае получилось:
  * px = 384;
  * py = 512;

Параметры  tx и ty равны 264 и 352 соответственно, их вычисляли в первой статье, когда выполняли сдвиг изображения в центр. Теперь когда известны все переменные, перейдем к вычислениям:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/image_formula_scale_sample_anchor_point.png?raw=true)
 
А теперь, перейдем к реализации на Android:

~~~ java

package mercuriy94.com.matrix.affinetransformations;

//imports
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
                transAndScale();
            }
        });
    }

 private void transAndScale() {

        Matrix transformImageMatrix = imageView.getImageMatrix();

        //region translate to center

        Matrix translateToCenterMatrix = new Matrix();

        float tx = (imageView.getMeasuredWidth() - imageView.getDrawable().getIntrinsicWidth()) / 2f;
        float ty = (imageView.getMeasuredHeight() - imageView.getDrawable().getIntrinsicHeight()) / 2f;

        float[] translateMatrixValues = new float[]{
                1f, 0f, tx,
                0f, 1f, ty,
                0f, 0f, 1f};

        translateToCenterMatrix.setValues(translateMatrixValues);

        transformImageMatrix.postConcat(translateToCenterMatrix);

        //endregion translate to center

        //region scale

        float sx = 2f;
        float sy = 2f;

        float px = imageView.getMeasuredWidth() / 2f;
        float py = imageView.getMeasuredHeight() / 2f;

        float[] scaleMatrixValues = new float[]{
                sx, 0f, -px * sx + px,
                0f, sy, -py * sy + py,
                0f, 0f, 1f};

        Matrix scaleMatrix = new Matrix();
        scaleMatrix.setValues(scaleMatrixValues);

        transformImageMatrix.postConcat(scaleMatrix);

        //endregion scale

        imageView.setImageMatrix(transformImageMatrix);
   
        printMatrixValues(transformImageMatrix);
        printImageCoords(transformImageMatrix);
    }
  
  ....

}

~~~

Для удобства чтения разбиваем метод на 2 региона, чтобы с легкостью наблюдать последовательное применение преобразований. Результат получился следующим:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/image_sample_scale_2_result.png?raw=true)
 
 Вывод лога разбиваем на 2 части:

1. Вывод получившейся матрицы преобразования:
 
  ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matix_values_log_input_scale_trans.png?raw=true)
 
2. Вывод коордиант изображения после преобразования:
 
  ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matix_values_log_input_scale_trans_coords.png?raw=true)

Текущие выводы, позволяют нам судить, что наши расчеты верны!

Теперь перепишем метод transAndScale() с использованием методов postTranlate() и postScale():

~~~ java

...
  
  private void transAndScale() {

        Matrix transformImageMatrix = imageView.getImageMatrix();

        //region translate to center

        float tx = (imageView.getMeasuredWidth() - imageView.getDrawable().getIntrinsicWidth()) / 2f;
        float ty = (imageView.getMeasuredHeight() - imageView.getDrawable().getIntrinsicHeight()) / 2f;

        transformImageMatrix.postTranslate(tx, ty);

        //endregion translate to center

        //region scale

        float sx = 2f;
        float sy = 2f;

        float px = imageView.getMeasuredWidth() / 2f;
        float py = imageView.getMeasuredHeight() / 2f;

        transformImageMatrix.postScale(sx, sy, px, py);

        //endregion scale

        imageView.setImageMatrix(transformImageMatrix);
    }

...
  
~~~


### Поворот (вращение) изображения  (Rotate)

Преобразование поворота устанавливает соответствие между координатами точки, объекта и экраном системы координат наблюдателя при вращении объекта (без сдвига) относительно начала координат.
Другими словами, мы будем вращать плоскость, на которой лежит наше изображение.
Прежде чем начать обсуждать вращение, договоримся, что принято считать вращение против часовой стрелки положительным. При таком раскладе, удобно считать, что угол поворота (альфа) лежит в интервале [-П;П].

Чтобы найти формулу преобразования координат, выберем произвольную точку(x,y), вектор которой обозначим r. Тут уже требуется освежить в памяти тригонометрию.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/x=rcos.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/y=rsin.png?raw=true)

Если повернем на угол тета:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/x'=rcos.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/y'=rsin.png?raw=true)

Зная формулы суммы углов для синуса и косинуса:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/sin(a+q).png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/cos(a+q).png?raw=true)

Тогда:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/x'=rcos(a+q).png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/y'=rsin(a+q).png?raw=true)

Раскроем скобки и выполним замены, получим:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/x'=matrix_rotate_formula.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/y'=matix_rotate_formula.png?raw=true)

Теперь осталось найти матрицу преобразования:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_rotate.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_rotate_final.png?raw=true)


И получим матрицу 3 х 3, применяемую для преобразования вращения: 

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_rotate_final_formula.png?raw=true)

  Чтобы вращать плоскость вокруг опорной точки (например центра изображения), необходимо выполнить следующие операции преобразования:
1. Сместить на обратные координаты опорной точки;
2. Выполнить вращение;
3. Сместить обратно на координаты опорной точки;

Таким способом получаем необходимую матрицу преобразования вокруг опорной точки:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matrix_rotate_anchor_point_final.png?raw=true)

Так как используется несколько типов преобразований, то этот вид вращения является комбинацией преобразований.    

Теперь рассмотрим применение вращения в Android.
Предлагаю опустить базовый пример состоящего из одного преобразования, вместо этого предлагаю дополнить пример из раздела "Масштабирование". Следовательно исходными данными будет отцентрированное и растянутое по двух осям изображение, тогда входной матрицей будет результат матрицы из раздела "Масштабирование". Вращать будем на 90 градусов.
Как обычно, давайте рассчитаем координаты на которые лягут углы изображения.

P.S: Предвижу ваши ужасания, но вам необязательно прослеживать путь решения, достаточно понимать выполняемые действия :=) 
 
 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/formua_sample_image_rotate_anchor_point.png?raw=true)
 
После долгих вычислений, давайте перейдем к коду:
 
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
                transScaleRotate();
            }
        });
    }

     private void transScaleRotate() {

        Matrix transformImageMatrix = imageView.getImageMatrix();

        //region translate to center

        float tx = (imageView.getMeasuredWidth() - imageView.getDrawable().getIntrinsicWidth()) / 2f;
        float ty = (imageView.getMeasuredHeight() - imageView.getDrawable().getIntrinsicHeight()) / 2f;

        transformImageMatrix.postTranslate(tx, ty);

        //endregion translate to center

        //region scale

        float sx = 2f;
        float sy = 2f;

        float px = imageView.getMeasuredWidth() / 2f;
        float py = imageView.getMeasuredHeight() / 2f;

        transformImageMatrix.postScale(sx, sy, px, py);

        //endregion scale

        //region rotate

        Matrix matrixRotate = new Matrix();

        double degrees = Math.toRadians(90d);

        float[] rotateMatrixValues = new float[]{
                (float) Math.cos(degrees), -(float) Math.sin(degrees), -(float) Math.cos(degrees) * px + (float) Math.sin(degrees) * py + px,
                (float) Math.sin(degrees), (float) Math.cos(degrees), -(float) Math.sin(degrees) * px - (float) Math.cos(degrees) * py + py,
                0f, 0f, 1f};


        matrixRotate.setValues(rotateMatrixValues);
        transformImageMatrix.postConcat(matrixRotate);

        //endregion rotate

        imageView.setImageMatrix(transformImageMatrix);
       
        printMatrixValues(transformImageMatrix);
        printImageCoords(transformImageMatrix);
    }
  
  ... 
    
}

~~~

Я переименовал метод в transScaleRotate(). Операции сдвига и масштабирования оставили не изменными, но добавили регион кода выполняющего вращение (rotate). 

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/image_sample_rotate.png?raw=true)
 
Выводы в лог подтвреждают правильность наших расчетов:
 
  ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matix_values_log_input_transScaleRotate.png?raw=true)

  ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%202/Resources/Images/matix_values_log_input_transScaleRotate_coords.png?raw=true)
 
  Конечно, класс Matrix включает в себя уже содержит методы для выполнения вращения: postRotate и preRotate. У данных методов есть обязательный параметр degrees, который определяет угол поворота плоскости изображения в градусах. А также опциональные параметры px и py, также как и в масштабирование эти параметры определяют координату опорной точки. А теперь, по традиции, перепишем метод transScaleRotate с использованием метода postRotate(). 
Обратите внимание, что в операциях масштабирования и вращения, используются одни и те же значения px и py, так как координаты опорных точек для этих операций выбрали одинаковыми, что является центром контейнера ImageView.

~~~ java

...

  private void transScaleRotate() {

        Matrix imageMatrix = imageView.getImageMatrix();

        //region translate to center

        float tx = (imageView.getMeasuredWidth() - imageView.getDrawable().getIntrinsicWidth()) / 2f;
        float ty = (imageView.getMeasuredHeight() - imageView.getDrawable().getIntrinsicHeight()) / 2f;

        imageMatrix.postTranslate(tx, ty);

        //endregion translate to center

        //region scale

        float sx = 2f;
        float sy = 2f;

        float px = imageView.getMeasuredWidth() / 2f;
        float py = imageView.getMeasuredHeight() / 2f;

        imageMatrix.postScale(sx, sy, px, py);

        //endregion scale

        //region rotate

        float degrees = 90f;

        imageMatrix.postRotate(degrees, px, py);

        //endregion rotate

        printMatrixValues(imageMatrix);

        imageView.setImageMatrix(imageMatrix);
    }

...

~~~

Теперь, вы знаете, как применять на практике преобразования: сдвига, масштабирование и вращение. И это уже очень хорошо!
В следующей заключительной статье, поговорим о том, как выполнять наклон и отражение изображений. 

Всем спасибо!