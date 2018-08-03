# Матрицы в Android, третья (заключительная) часть

Во второй части мы обсуждали масштабирование и вращение изображение, с подробными примерами. В текущей заключительной части мы рассмотрим такие пробразования как наклон (Skew) и отражение. Не будем затягивать, приступим!

### Наклон (Skew)

К сожалению, я не нашел понятного объяснения данного преобразования. Так что, будем получать матрицу преобразования опытным путем. Для начала, взгялнем на следующее изображение.

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_image.png?raw=true)

Здесь параллелограмм с зеленым цветом периметра является квадратом, подвергнутым наклону. Мы можем провести касательную левой-верхней точки границы наклоненного квадрата. Таким образом, мы получим угол наклона. 

А также, наблюдаем прямоугольный трегоульник (B'CC'). Вспоминаем, что в прямоуголном треугольнике:
 - Гипотенуза — самая длиная сторона;
 - Катеты — стороны, лежащие напротив острых углов;
 - Противолежащий катет — сторона треугольник которая лежит напротив заданного острого угла (СС');
 - Прилежащий катет — сторона треуголника которая прилегает к заданному острому углу (B'C);

Основываясь на курсе тригонометрии, мы можем воспользоваться следующими тождествами:

Синус острого угла в прямоугольном треугольнике — это отношение противолежащего катета к гипотенузе:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/sinA.png?raw=true)

Косинус острого угла в прямоугольном треугольнике — отношение прилежащего катета к гипотенузе:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/cosA.png?raw=true)

Тангенс острого угла в прямоугольном треугольнике — отношение противолежащего катета к прилежащему:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/tanA.png?raw=true)

Так как, противолежащая сторона нам неизвестна, но мы знаем угол наклона и прилежащий катет, то мы с легкостью можем воспользоваться функцией тангенса:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/CC'.png?raw=true)

Теперь нам остается решить систему уравнений, чтобы найти матрицу преобразования  для наклона:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/y=y'.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_x_skew.png?raw=true)

Мы также помним общую формулу преобразования для оси абцисс.

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/x'=ax+by+cz.png?raw=true)

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/a=1.png?raw=true)

Аналогичная ситуация и с осью ординат. В итоге мы получаем матрицу преобразования 3x3 для наклона по оси абцисc (X):

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_X.png?raw=true)

А для оси ординат (Y):

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_Y.png?raw=true) 

А если вам понядобиться одновременно наклонять по двум осям, то используем полную матрицу:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_formula.png?raw=true) 

Отлично, но ведь можно еще наклонять вокруг опорной точки, тут как и в предыдущих разделах не обойтись без сдвигов:

  1: Сместить на обратные координаты опорной точки;
  2: Выполнить вращение;
  3: Сместить обратно на координаты опорной точки;

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_skew_anchor_point_formula.png?raw=true) 

Не перепутайте, что угол альфа используется для наклона по оси абцисс, а бета по оси ординат.

В качестве примера, давайте выполним наклон отцентрированного и растянутого изображения. Исходной матрицей предлагаю взять результат матрицы из второй части статьи раздела "Масштабирование". Наклонять будем вокруг опорной точки, как обычно, в качестве координаты опорной точки возьмем центр контейнера ImageView. Выполним наклон по абцисс на 15 градусов, а по оси ординат на 30 градусов. 

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/%20image_skew_sample.png?raw=true)
  
  Перепишем наш пример:
  
~~~ java

package mercuriy94.com.matrix.affinetransformations;

//  Импорты
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
                transScaleSkew();
            }
        });
    }

  private void transScaleSkew() {

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

        //region skew

        float[] skewMatrixValues = new float[]{
                1f, (float) Math.tan(Math.toRadians(15d)), -(float) Math.tan(Math.toRadians(15d)) * py,
                (float) Math.tan(Math.toRadians(30d)), 1f, -(float) Math.tan(Math.toRadians(30d)) * px,
                0f, 0f, 1f};

        Matrix skewMatrix = new Matrix();
        skewMatrix.setValues(skewMatrixValues);

        transformImageMatrix.postConcat(skewMatrix);

        //endregion skew

        imageView.setImageMatrix(transformImageMatrix);

        printMatrixValues(transformImageMatrix);
        printImageCoords(transformImageMatrix);
    }

  ...
    
}

~~~

Результат получился следующим:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/image_skew_sample_result.png?raw=true)
 
 Выводы в лог позволяют нам заключить, что наши расчеты вполне сносны:
 
  ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matix_values_log_input_transScaleSkew.png?raw=true)

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matix_values_log_input_transScaleSkew_coords.png?raw=true)

Конечно, класс Matrix содержит готовые методы для выполнения наклона postSkew и preSkew. У этих методов есть два обязательных параметра kx и ky, которые являются значениями тангенса угла. Если вам нужно наклонять только по одной из осей, то во второй параметр передаем 0f, так как тангенс нуля равен нулю. А также, имеется два опциональных параметра px и py, смысл которых нам уже известен. Теперь перепишем наш метод transScaleSkew с использованием метода postSkew.

~~~java

...

  private void transScaleSkew() {

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

        //region skew

        imageMatrix.postSkew((float) Math.tan(Math.toRadians(15)), (float) Math.tan(Math.toRadians(30)), px, py);

        //endregion skew

        printMatrixValues(imageMatrix);

        imageView.setImageMatrix(imageMatrix);
    }
...

~~~

### Отражение (Reflexion)

Чтобы облегчить понимаение, отражение можно рассмтаривать как вращение изображения.
  Различают два вида отражения:
  - Отражение относительно оси;
  - Отражение относительно точки;

Для началао разберемся с отражением вокруг начала координат.
Для понимания, удобно рассматривать отражение вокруг осей по отдельности.

Предлагаю, отталкиваться от мысли, что принцип отражения относительно начала координат - в смене знака соседней координаты на противоположный, относительно которой выполняется отражение. То есть, предположим, имеется точка с координатами P(x,y), чтобы выполнить отражение относительно jси абсцисс, необходимо сменить координату y на противположный, и получим P' (x, -y). Тогда:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflect_around_x_1.png?raw=true)
 
 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflect_around_x.png?raw=true)
 
Получаем матрицу 3 х 3, для выполнения отражения относительно оси абсцисс:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflect_arround_x_final.png?raw=true)

И для оси ординат:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflecty_around_y_1.png?raw=true)
 
 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_reflect_around_y_2.png?raw=true)
 
И матрица 3 х 3 получается:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_reflect_around_y_final.png?raw=true)
 
Отлично! Но, как часто приходится отражать относительно начала координат? Ответом наверно будет - скорее всего очень редко. По этому попробуем вывести матрицу для отражения опорной точки. 

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_reflect_anchor_point.png?raw=true)

Теперь рассмотрим пример отражения в Android, отночительно точки. Отражать будем отценрованное изображение в сторону верхней границы контейнера. Следовательно коориданты опорной точки будут: 
  * px = 384 (половина ширины контейнера ImageView);
  * py = 352 (используемый сдвиг по оси y для центрирования);

Как положено, рассчитаем конечные координаты изображения:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_reflect_anchor_point_sample.png?raw=true)
 
Перейдем к реализации на Android:

~~~java

package mercuriy94.com.matrix.affinetransformations;

public class MainActivity extends AppCompatActivity {

  //  Импорты
  ...
  
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
                translateToCenterAndReflect();
            }

        });
    }

    private void translateToCenterAndReflect() {

        Matrix transformMatrix = new Matrix();

        //region trans

        float tx = (imageView.getMeasuredWidth() - imageView.getDrawable().getIntrinsicWidth()) / 2f;
        float ty = (imageView.getMeasuredHeight() - imageView.getDrawable().getIntrinsicHeight()) / 2f;

        Matrix imageMatrix = imageView.getImageMatrix();
        imageMatrix.postTranslate(tx, ty);

        //endregion trans

        //region reflect

        float px = imageView.getMeasuredWidth() / 2f;
        float py = ty;

        float[] matrixValues = new float[]{
                -1f, 0f, 2 * px,
                0f, -1f, 2 * py,
                0f, 0f, 1f};

        transformMatrix.setValues(matrixValues);
        imageMatrix.postConcat(transformMatrix);

        //endregion reflect

        imageView.setImageMatrix(imageMatrix);

        printImageCoords(imageView.getImageMatrix());
    }

    ...

}

~~~

Результат:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/image_matrix_reflect_sample_result.png?raw=true)

Отлично! Теперь заглянем в лог:

 ![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/matrix_reflect_sample_result_coords.png?raw=true)

Да, все правильно! 

### Итоги

Пришло время подвести итоги и вспомнить, какие матрицы преобразования мы рассмотрели на протежении трех статей:

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

Отражение относительно точки лежащей на плоскости:

![](https://github.com/mercuriy94/Matrices-in-Android/blob/master/Part%203/Resources/Images/reflect_anhor_point_final.png?raw=true)

Теперь мы знаем, как применять аффинные преобразования на практике и рассчитывать координату на которую ляжет та или иная точка в результате преобразований. Конечно, чаще всего вам не придется выполнять все эти операции, но знать, по каким правилам изменяется положение точек, необходимо.

Всем чистого кода!