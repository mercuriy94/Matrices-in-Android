# Матрицы в Android, третья (заключительная) часть

Во второй части мы обсуждали масштабирование и вращение изображение, с подробными примерами. В текущей заключительной части мы рассмотрим такие пробразования как наклон (Skew) и отражение. Не будем затягивать, приступим!

### Наклон (Skew)

К сожалению, я не нашел понятного объяснения данного преобразования. Так что, будем получать матрицу преобразования опытным путем. Для начала, взгялнем на следующее изображение.

 ![](https://bitbucket.org/mercury94/articles/raw/3e94b24cdfc27b6da93cdd1a5cb7d96a976d85a6/matrix_skew_image.png)

Здесь параллелограмм с зеленым цветом периметра является квадратом, подвергнутым наклону. Мы можем провести касательную левой-верхней точки границы наклоненного квадрата. Таким образом, мы получим угол наклона. 

А также, наблюдаем прямоугольный трегоульник (B'CC'). Вспоминаем, что в прямоуголном треугольнике:
 - Гипотенуза — самая длиная сторона;
 - Катеты — стороны, лежащие напротив острых углов;
 - Противолежащий катет — сторона треугольник которая лежит напротив заданного острого угла (СС');
 - Прилежащий катет — сторона треуголника которая прилегает к заданному острому углу (B'C);

Основываясь на курсе тригонометрии, мы можем воспользоваться следующими тождествами:

Синус острого угла в прямоугольном треугольнике — это отношение противолежащего катета к гипотенузе:

 ![](https://bitbucket.org/mercury94/articles/raw/c2382b4b566c5b2022d077f47590c337fbe2ecc6/sinA.png)

Косинус острого угла в прямоугольном треугольнике — отношение прилежащего катета к гипотенузе:

 ![](https://bitbucket.org/mercury94/articles/raw/c2382b4b566c5b2022d077f47590c337fbe2ecc6/cosA.png)

Тангенс острого угла в прямоугольном треугольнике — отношение противолежащего катета к прилежащему:

 ![Alt Text](https://bitbucket.org/mercury94/articles/raw/c2382b4b566c5b2022d077f47590c337fbe2ecc6/tanA.png)

Так как, противолежащая сторона нам неизвестна, но мы знаем угол наклона и прилежащий катет, то мы с легкостью можем воспользоваться функцией тангенса:

 ![](https://bitbucket.org/mercury94/articles/raw/c2382b4b566c5b2022d077f47590c337fbe2ecc6/CC'.png)

Теперь нам остается решить систему уравнений, чтобы найти матрицу преобразования  для наклона:

![](https://bitbucket.org/mercury94/articles/raw/8a963f0a811757c28dc21f429690d24a78e65c72/y%3Dy'.png)

![](https://bitbucket.org/mercury94/articles/raw/0f15c1c72e5f38e8f95f8e8c1fadf3f2e52c3512/matrix_x_skew.png)

Мы также помним общую формулу преобразования для оси абцисс.

![](https://bitbucket.org/mercury94/articles/raw/ab9cf66a69e9f6191d432875dd97707453bdca51/x'%3Dax%2Bby%2Bcz.png)

![](https://bitbucket.org/mercury94/articles/raw/a8a45e424fca1ae44930547dd622eabf50eb4256/a%3D1.png)

Аналогичная ситуация и с осью ординат. В итоге мы получаем матрицу преобразования 3x3 для наклона по оси абцисc (X):

![Alt Text](https://bitbucket.org/mercury94/articles/raw/90c0d7c2dc3aa0574490bc3fc10c385a0ce6a5fb/matrix_skew_X.png)

А для оси ординат (Y):

![](https://bitbucket.org/mercury94/articles/raw/90c0d7c2dc3aa0574490bc3fc10c385a0ce6a5fb/matrix_skew_Y.png) 

А если вам понядобиться одновременно наклонять по двум осям, то используем полную матрицу:

![](https://bitbucket.org/mercury94/articles/raw/be7b53252ebd3fe27e7cd198f88149ba3ebf88de/matrix_skew_formula.png) 

Отлично, но ведь можно еще наклонять вокруг опорной точки, тут как и в предыдущих разделах не обойтись без сдвигов:

  1: Сместить на обратные координаты опорной точки;
  2: Выполнить вращение;
  3: Сместить обратно на координаты опорной точки;

![](https://bitbucket.org/mercury94/articles/raw/3db7802c34ed6961e1301bba631282766957614a/matrix_skew_anchor_point_formula.png) 

Не перепутайте, что угол альфа используется для наклона по оси абцисс, а бета по оси ординат.

В качестве примера, давайте выполним наклон отцентрированного и растянутого изображения. Исходной матрицей предлагаю взять результат матрицы из второй части статьи раздела "Масштабирование". Наклонять будем вокруг опорной точки, как обычно, в качестве координаты опорной точки возьмем центр контейнера ImageView. Выполним наклон по абцисс на 15 градусов, а по оси ординат на 30 градусов. 

 ![](https://bitbucket.org/mercury94/articles/raw/0dfa91b51b628a2d86a5d06e7b9abc4357dcdfc3/%20image_skew_sample.png)
  
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

 ![](https://bitbucket.org/mercury94/articles/raw/4f30819f0a30e60f2532ba2584a8513537317c24/image_skew_sample_result.png)
 
 Выводы в лог позволяют нам заключить, что наши расчеты вполне сносны:
 
  ![](https://bitbucket.org/mercury94/articles/raw/207e58ab7ce1a823142a1d8a0c81d03d939ccb03/matix_values_log_input_transScaleSkew.png)


 ![](https://bitbucket.org/mercury94/articles/raw/207e58ab7ce1a823142a1d8a0c81d03d939ccb03/matix_values_log_input_transScaleSkew_coords.png)

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

 ![](https://bitbucket.org/mercury94/articles/raw/b7f72b41c490f6f9830523bbcd00ebe3fbdeae03/reflect_around_x_1.png)
 
 ![](https://bitbucket.org/mercury94/articles/raw/b7f72b41c490f6f9830523bbcd00ebe3fbdeae03/reflect_around_x.png)
 
Получаем матрицу 3 х 3, для выполнения отражения относительно оси абсцисс:

 ![](https://bitbucket.org/mercury94/articles/raw/b7f72b41c490f6f9830523bbcd00ebe3fbdeae03/reflect_arround_x_final.png)

И для оси ординат:

 ![](https://bitbucket.org/mercury94/articles/raw/b7f72b41c490f6f9830523bbcd00ebe3fbdeae03/reflect_arround_x_final.png)
 
 ![](https://bitbucket.org/mercury94/articles/raw/b7f72b41c490f6f9830523bbcd00ebe3fbdeae03/reflect_around_x.png)
 
И матрица 3 х 3 получается:

 ![](https://bitbucket.org/mercury94/articles/raw/b7f72b41c490f6f9830523bbcd00ebe3fbdeae03/reflect_arround_x_final.png)

### Итоги

Давайте подведем итог и вспомним, какие матрицы преобразования мы рассмотрели на протежении трех статей:

Параллельный перенос (сдвиг):

 ![](https://bitbucket.org/mercury94/articles/raw/a8b098cc442c2502660f34362044ce5983b00b55/matrix_translation.png)

Масштабирование:

 ![](https://bitbucket.org/mercury94/articles/raw/a8b098cc442c2502660f34362044ce5983b00b55/matrix_scale%202.png)

Масштабирование с опорной точкой:

 ![](https://bitbucket.org/mercury94/articles/raw/a8b098cc442c2502660f34362044ce5983b00b55/matrix_scale_anchor_point.png)

Вращение:

 ![](https://bitbucket.org/mercury94/articles/raw/a8b098cc442c2502660f34362044ce5983b00b55/matrix_rotate%202.png)

Вращение с опорной точкой:

 ![](https://bitbucket.org/mercury94/articles/raw/a8b098cc442c2502660f34362044ce5983b00b55/matrix_rotate_anchor_point.png)

Наклон:

 ![](https://bitbucket.org/mercury94/articles/raw/a8b098cc442c2502660f34362044ce5983b00b55/matrix_skew.png)

Наклон с опорной точкой:

 ![](https://bitbucket.org/mercury94/articles/raw/a8b098cc442c2502660f34362044ce5983b00b55/matrix_skew_anchor_point.png)


Теперь мы знаем, как применять аффинные преобразования на практике и рассчитывать координату на которую ляжет та или иная точка в результате преобразований. Конечно, чаще всего вам не придется выполнять все эти операции, но знать, по каким правилам изменяется положение точек, необходимо.

Всем чистого кода!