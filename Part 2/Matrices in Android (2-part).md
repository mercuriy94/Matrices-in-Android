# Матрицы в Android, вторая часть

В первой части, мы познакомились с таким понятием как аффинное преобразования и в каких целях оно испольузется. Рссмотрели такой вид преобразования как "Сдвиг". Если в первой части в основном обсуждалась теория, то эта часть полностью посвещена практике, где подробно рассмотрим такие преобразования как масштабирование и вращения, с учетом опорной точки для этих преобразований.

### Масштабирование изображения (Scale)

Преобразование масштабирования увеличивает или уменьшает размер изображения объекта в системе координат наблюдателя по-сравнению с исходным размером в системе координат этого самого объекта.
  Так по какому принципу происходит масштабирование изображений? И какие тут могут возникать сложности?
  Для начала, вы подумаете, и скажите. "Хм..., а можно ведь взять координату точки и умножить на коэффициент масштабирования, и так с другой координатой". И будете правы. Итак, для начала нам нужно различать коэффициент для каждой оси Sx и Sy, для осей абцисс и ординат соответственно.

В итоге получаем следующее:

![Alt Text](https://bitbucket.org/mercury94/articles/raw/badafb3ce85837b42e4840870d977d18ed72a063/matrix_scale.png)

![Alt Text](https://bitbucket.org/mercury94/articles/raw/18c664ba4f9bb86d7d1ca7c0c680e3a1ba6a5854/matrix_scale_final.png)

Получим матрицу 3х3 для применения масштабирования:

![Alt Text](https://bitbucket.org/mercury94/articles/raw/46cecd569b0d78ff09c89b7872ead8170826f4a2/matrice_scale.png)

Чтобы применить матрицу масштабирования, ее надо умножить на матрицу координат:

![Alt Text](https://bitbucket.org/mercury94/articles/raw/fa9de303cd85f41e9a4ecbb43a2a493dbc537490/matrix_scale_example.png)

Теперь рассмотрим применение масштабирвания в Android.

В качестве примера растянем исходное изображение в 2 раза по двум осям.

Первым делом давайте вычислим координаты углов изображения после преобразования:

![Alt Text](https://bitbucket.org/mercury94/articles/raw/c5a4821acb200f5ebae32ad1ac01655fa2a2de69/sample_scale_results.png)
 
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

![](https://bitbucket.org/mercury94/articles/raw/926bee3b2a5674da6ddb3f3dc03cb69b851bb3a2/image_sample_scale_result.png)

Убеждаемся в паравильности расчетов:

![](https://bitbucket.org/mercury94/articles/raw/648289e11fd7e7ea90a53cd82a08c99c3ec45fe5/matix_values_log_input_scale_coords.png)

Скорее всего, вы уже догадались, что нам необязательно создавать матрицу преобразования. Для этого нам достаточно знать о методах postScale(...) и preScale(...). Эти методы включают в себя обязательные параметры: sx и sy. А также опциональные (необезательные) параметры: px и py. 
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

Хорошо, а что насчет масштабирования вокруг опорной точки? Тут нам необходимо выполнить комбинацию преобразований:
1. Сместить плоскость изображение так, чтобы опорная точка совпалдала с началом координат плоскости наблюдателя. Простыми словами, выполнить сдвиг на обратные велечины опорной точки.
2. Применить обычное машстабирование;
3. Выполнить сдвиг плоскости изображения на расстояние координат опорной точки в соответствующих осях.

Тогда матрица 3 х 3 для масштабирваония вокруг опорной точки имеет вид:

![Alt Text](https://bitbucket.org/mercury94/articles/raw/f5f4a398335fa13ad5a7f640b17f200dcaab93fb/matrix_scale_anchor_point_formula.png)

Пример:

Чтобы все было красиво, давайте перед этим выполним известную нам операцию переноса в центр контейнера. Таким образом, у нас получится комбинация преобразований. 
Как и прежде, сначала рассчитаем финальное положение углов изображения. Но, чтобы это выполнить, нам необходимо получить опорную точку для масштабирования. Так как изображение будет находиться в центре контейнера, то опорной точкой будет центр контейнера, координаты которой равны половине его размеров. В моем случае получилось: 
px = 384;
py = 512;

Параметры  tx и ty равны 264 и 352 соответственно, мы их вычисляли в первой части статьи. Теперь когда нам известны все переменные, можем перейти к вычислениям:

 ![](https://bitbucket.org/mercury94/articles/raw/cdb15790f15294bdcee25755ad767d1e66c3dbbf/image_formula_scale_sample_anchor_point.png)
 
А теперь напишем реализацию в Android:

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

Для удобства чтения я разбил метод на 2 региона, чтобы вы могли с легкостью наблюдать последовательное применение преобразования. Результат получился следующим:

 ![](https://bitbucket.org/mercury94/articles/raw/926bee3b2a5674da6ddb3f3dc03cb69b851bb3a2/image_sample_scale_2_result.png)
 
 Вывод лога можно разбить на 2 части:
 1 -  Вывод получившейся матрицы преобразования:
 
  ![](https://bitbucket.org/mercury94/articles/raw/74f0391589c3453964a5c42498ec84b9e36ec461/matix_values_log_input_scale_trans.png)
 
 2 - Вывод коордиант изображения после преобразования:
 
  ![](https://bitbucket.org/mercury94/articles/raw/74f0391589c3453964a5c42498ec84b9e36ec461/matix_values_log_input_scale_trans_coords.png)

Выводы, позволяют нам судить, что наши расчеты верны!

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

Преобразование поворота устаналивает соответствие между координатами точки, объекта и экраном системы координат наблюдателя при вращении объекта (без сдвига) отностиельно начала координат.
То есть, мы будем вращать плоскость, на которой лежит наше изображение.
Прежде чем начать обсуждать вращение, договоримся, что принято считать вращение против часовй стрелки положительным. При таком раскладе, удобно считать, что угол поворота (альфа) лежит в интервале [-П;П].

Чтобы найти формулу преобразования координат, выберем произвольную точку(x,y), вектор которой обозначим r. Тут уже требуется освежить в памяти тригонометрию.

![](https://bitbucket.org/mercury94/articles/raw/c6692dc1ac6de6224e2bb2ff59ef8d1297c4f4bd/x%3Drcos.png)

![](https://bitbucket.org/mercury94/articles/raw/c6692dc1ac6de6224e2bb2ff59ef8d1297c4f4bd/y%3Drsin.png)

Если повернем на угол тета:

![](https://bitbucket.org/mercury94/articles/raw/2782d406809d3841b70a180ba865aded99819975/x'%3Drcos.png)

![](https://bitbucket.org/mercury94/articles/raw/2782d406809d3841b70a180ba865aded99819975/y'%3Drsin.png)

Зная формулы суммы углов для синуса и косинуса:

![](https://bitbucket.org/mercury94/articles/raw/359494d16437cda621b58d6f3e58d967ee007768/sin(a%2Bq).png)

![](https://bitbucket.org/mercury94/articles/raw/359494d16437cda621b58d6f3e58d967ee007768/cos(a%2Bq).png)

Тогда:

![](https://bitbucket.org/mercury94/articles/raw/19e09c403a2a2da89975c9786c6bd9ffc4360d9f/x'%3Drcos(a%2Bq).png)

![](https://bitbucket.org/mercury94/articles/raw/19e09c403a2a2da89975c9786c6bd9ffc4360d9f/y'%3Drsin(a%2Bq).png)

Раскроем скобки и выполним замены, получим:

![](https://bitbucket.org/mercury94/articles/raw/02a2a049b28933fec9b6215bed9e2d88b5fc50f6/x'%3Dmatrix_rotate_formula.png)

![](https://bitbucket.org/mercury94/articles/raw/02a2a049b28933fec9b6215bed9e2d88b5fc50f6/y'%3Dmatix_rotate_formula.png)

Теперь осталось найти матрицу преобразования:

![](https://bitbucket.org/mercury94/articles/raw/7b57c4dcb0ca1106bcfad262b029b591c90d4c83/matrix_rotate.png)

![](https://bitbucket.org/mercury94/articles/raw/1b1773bc57459bcb559622d895739956293f9ebb/matrix_rotate_final.png)


И получим матрицу 3 х 3, применяемую для преобразования вращения: 

![Alt Text](https://bitbucket.org/mercury94/articles/raw/7b9979eddf5b845a2a12b9c4fa4fb00a7b0b601a/matrix_rotate_final_formula.png)

  Чтобы вращать плоскость вокруг опорной точки (например центра изображения), необходимо выполнить следующие операции преобразования:
  1: Сместить на обратные координаты опорной точки;
  2: Выполнить вращение;
  3: Сместить обратно на координаты опорной точки;

Таким способом мы получим необходимую матрицу преобразования вокруг опорной точки:

![Alt Text](https://bitbucket.org/mercury94/articles/raw/07175207dc95257399568f7932332e641c7b7e09/matrix_rotate_anchor_point_final.png)

Так как мы испольузем несколько разных типов преобразований, то этот вид вращения можно считать комбинацией преобразований.  

Теперь рассмотрим применение вращения в Android.
Предлагаю опустить базовый пример состоящего из одного преобразования, вместо этого предлагаю дополнить пример из раздела "Масштабирование". Следовательно в качестве исходных данных мы имем отцентрированное и растянутое по двух осям изображение, тогда в качестве исходной матрицы мы возьмем результат матрицы из раздела "Машстабирование". Вращать будем на 90 градусов.
Как обычно, давайте рассчитаем координаты на которые лягут углы изображения.

P.S: Предвижу ваши ужасания, но вам необязательно прослеживать весь путь решения, достаточно понимать выполняемые действия :=) 
 
 ![](https://bitbucket.org/mercury94/articles/raw/1351698066b8974389943da139749c5ae450779b/formua_sample_image_rotate_anchor_point%20%D0%BA%D0%BE%D0%BF%D0%B8%D1%8F.png)
 
 Теперь, наконец-то, давайте перейдем к коду:
 
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

Я переименовал метод в transScaleRotate(). Операции сдвига и масштабирования я оставил не изменными, но добавил регион кода выполняющего вращение (rotate). 

 ![](https://bitbucket.org/mercury94/articles/raw/4c868fdfd3f39d25ab798453ea8b673d24c4a111/image_sample_rotate.png)
 
 Выводы в лог подтвреждают правильность наших расчетов:
 
  ![](https://bitbucket.org/mercury94/articles/raw/6e725d4f24d5b9fbdd118e2d42f25b3eb4591f98/matix_values_log_input_transScaleRotate.png)

  ![](https://bitbucket.org/mercury94/articles/raw/6e725d4f24d5b9fbdd118e2d42f25b3eb4591f98/matix_values_log_input_transScaleRotate_coords.png)
 
  Конечно, класс Matrix включает в себя уже готовыe методы для выполнения вращения: postRotate и preRotate. У данных методов есть обязательный параметр degrees, который определяет угол поворота плоскости изображения в градусах. А также опциональные параметры px и py, также как и в масштабирование эти параметры определяют координату опорной точки. А теперь, по традиции, перепишем метод transScaleRotate с использованием метода postRotate(). 
Обратите внимание, что в операциях масштабирования и вращения мы используем одни и те же значения px и py, так как координаты опорных точек для этих опреаций вы выбрали одинаковыми, что является центром контейнера ImageView.

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

Теперь вы можете применять на практике такие преобразования как: сдвиг, масштабирование и вращение. И это уже очень хорошо!
В следующей заключетиельной части серии статей, мы поговорим о таких преобразованиях как наклон и отражение. 

Всем спасибо!