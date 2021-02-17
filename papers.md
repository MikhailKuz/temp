## Конспекты статей 

- **Accurate and Robust Feature Importance Estimation under Distribution Shifts, 2020**  [[paper]](https://arxiv.org/pdf/2009.14454.pdf)
  - описывается подход к оценке важности признаков для нейросетей: основная сеть обучается совместно с дополнительной (second net), у которой:
    - цель - научиться предсказывать loss основной сети
    - input - латентные представления после некоторых слоёв основной сети
    - loss
      - contrastive training - сохраняем правильное упорядочивание пар скоров
      - dropout calibration - hinge loss + доверительные интервалы
  - используется Granger определение причинности (связь между признаком и целевой переменной сущетсвует, если качество только ухудшится при отбрасывании данного признака)
  - важность признака - разница предсказаний вспомогательной сети с его маскированием и без  
  *Итог*:
    - на 15-30 % лучше качество, чем у Shap
    - при увеличении различия распределения x_test, по сравнению с x_train, loss second net монотонно растёт
    - подход устойчивей при сильных изменениях x_test Deep_Shap'а в 2 раза

- **Feature Importance Ranking for Deep Learning, 2020** [[paper]](https://arxiv.org/pdf/2010.08973.pdf)
  - рассматривается две сети operator net и selector net, маски для признаков - бинарные вектора (1 - берем признак, 0 - нет), оптимальное кол-во признаков - гиперпараметр
  - обучение происходит поочередно
  - operator net:
    - цель - обучение с учителем конкретной задачи
    - input - x и маска признаков
    - loss - соответствующий задаче
  - selector net:
    - цель - предсказать loss operator net
    - input - маска признаков
    - loss - l2 с loss'ом, переданным от operator net
  - важность признака - соответствующая компонента градиента loss'а selector net'а в точке оптимального набора признаков
  - процесс построения оптимального набора очень долгий  
  *Итог*:
    - в среднем лучше качество на синтетических данных
    - лучшее RFE, BAHSIC, mRMR, CCM на 4-ёх benchmark датасетах

- **Knockoffs for the mass: new feature importance statistics with false discovery guarantees, 2019** [[paper]](https://arxiv.org/pdf/1807.06214.pdf)
  - аппроксимируется распределение данных (только признаки) байесовскими сетями
  - используется аугментация выборки определённым образом (чтобы не выходить за исходное распределение)
  - вместо того, чтобы перемешивать значения признака (в permutation importance), берется взвешенная сумма исходного признака и соответствующего признака из аугментированной выборки
  - важность признака - площадь под кривой (y - доля правильно отобранных признаков, x - параметр взвешенной суммы) для некоторого диапазона (например, [0, 10])
  - FDR в реальности нельзя оценить, предполагается, что мы хорошо моделируем распределение выборки

- **A Unified Approach to Interpreting Model Predictions, 2017** [[paper]](https://proceedings.neurips.cc/paper/2017/file/8a20a8621978632d76c43dfd28b67767-Paper.pdf)
  - рассматривается семейство аддитивных explanation models
  - в данном классе существует единственная explanation model, удовлетворяющая свойствам:
    - local accuracy - совпадение значений f(x) и exp_model(x')
    - missingness - признак, не присутствующий в x, будет иметь нулевую важность
    - consistency - признак во всевозможных комбинациях остальных имеет не меньшее значение на изменение выхода f, чем на f' -> его важность для f >= важность для f'
  - считать такую explanation model дорого
  - Linear LIME + Kernel SHAP дают истинные значения SHAP values
  - в случае f = max, можно за квадрат размерности признаков SHAP посчитать (пользуемся свойствами max)
  - Deep SHAP - вместо важности в DeepLIFT подставляем SHAP важность для промежуточных расчётах  
  *Итог*:
    - обобщили предыдущие методы
    - на реальной задаче SHAP важность совпала с человеческой
    - для конкретных моделей улучшили время расчёта

- **Learning Important Features Through Propagating Activation Differences, 2017** [[paper]](https://arxiv.org/pdf/1704.02685.pdf)
  - метод основан на разнице значений нейронов между начальным значением (reference) и конечным
  - разделяются положительный и отрицательный вклады в целевую переменную
  - важность признака линейно зависит от разницы x - x_reference
  - важность - shapley value с количеством разбиений 2
  - x_reference выбирается в зависимости от задачи  
  *Итог*:
    - использование разности x - x_reference позволяет информации распространяться когда градиент равен нулю
    - gradients, gradients*input, guided backprop, rescale rule теряют зависимости в ходе вычисления важности в некоторых случаях в отличии от предложенного reveal_cancel rule

- **CXPlain: Causal Explanations for Model Interpretation under Uncertainty, 2019** [[paper]](https://arxiv.org/pdf/1910.12336.pdf)
  - используется Granger's definition of causality (в реальности, исходя только из данных, нельзя проверить)
    - все признаки релевантные
    - признак временно предшествует метке (для того, чтобы получить метку, нужна информация о признаке)
  - истинная важность признака - нормированная разница ошибок объясняемой модели на x_mask и x_reference
  - обучается explanation model (подходящая решаемой задаче)
    - цель - предсказать важность признаков
    - input - маскированный элемент x_train
    - loss - расстояние Кульбака — Лейблера между истинным и предсказанным распределениями важностей признаков
  - для устойчивости обучаем ансамбль explanation_models (на сэмплированных выборках), предсказанная важность - медиана предсказаний ансамбля, а точность - интерквантильный размах  
  *Итог*:
    - точность оценки важности коррелирует с ошибкой ранжирования важности признаков
    - при небольшой мощности ансамбля (5) хорошо оценивается точность explanation_model
    - качество лучше на 20%, быстрее x100, чем Shap, Lime на Mnist и ImageNet
    - качество сильно зависит от устройства explanation_model

- **Bias in random forest variable importance measures, 2017** [[paper]](https://link.springer.com/content/pdf/10.1186/1471-2105-8-25.pdf)
  - имплементирован метод построения дерева (ctree), где выбор переменной осуществляется путем минимизации значения p критерия независимости условного вывода, сравнимого, например, с тестом χ2 со степенью свободы, равной числу категорий признака
  - лучше себя показывает, чем rf в синтетических экспериментах (с/без бутстрэпом, способ сэмплинга)
  - bias в важности признаков в rf возникает из-за того, что признаки с большим количество уникальных значений располагаются ближе к корню дерева
  - рассматривается две оценки важности признака
    - количество узлов, в которых используется признак для разделения выборки (selection frequency)
    - permutation importance
    - Gini importance (большой bias)  
  *Итоги экспериментов*:
    - сэмплинг с возвратом сильно смещает selection frequency в сторону признаков с большим числом уникальных значений
    - permutation importance более устойчив

- **Grouped variable importance with random forests and application to multiple functional data analysis, 2015** [[paper]](https://arxiv.org/pdf/1411.4170.pdf)
  - рассматривается оценка важности группы признаков с теоретической и практической стороны
  - теоретическая сторона
    - (признаки, целевая переменная) - случайный вектор
    - важность признака - разница квадратичного риска с заменой/без замены признака на одинакового распределенный признак, но не зависящий от остальных и целевой переменной
    - в определённых условиях важность группы признаков пропорциональна дисперсии функции (=модель) от этой группы
  - практическая сторона
    - важность группы признаков - oob + случайная перестановка строк для столбцов из группы
    - используется RFE
  - с помощью вейвлет декомпозиции можно получить различные разбиения коэффициентов на группы
  - для отбора некоторых групп не применим алгоритм RFE, т.к. существует общая составляющая, вносящая большой вклад -> делим интересующий параметр на сетку и вычисляем важность в конкретных точках  
  *Итоги экспериментов*:  
    - оценка важности согласовывается как с синтетическими экспериментами, так и с реальными

- **Correlation and variable importance in random forests, 2017** [[paper]](https://arxiv.org/pdf/1310.5726.pdf)
  
  - продолжение вышеописанной работы 
  - эмпирическая важность признака при использовании purely rf для независимых признаков сходится экспоненциально к теоретической при стремлении количества итераций разбиения узла дерева и мощности тренировочной выборки так, чтобы отношение первого ко второму стремилось к 0 
  - даже сильно коррелирующие признаки с целевой переменной могут получить малую важность из-за корреляции между собой  
  *Итоги экспериментов*:  
    - NRFE и RFE в целом имеют одинаковое качество
  
- **All Models are Wrong, but Many are Useful, 2018** [[paper]](https://arxiv.org/pdf/1801.01489.pdf)
  Идея - будем искать важность группы признаков X_1 не для одной хорошей модели (reference model), а для класса моделей
  Датасет - iid
  Введём несколько определений:
  a population ε-Rashomon  set:  <img src="svgs/9342112aaa03b638367e4057db8fb58c.svg?invert_in_darkmode" align=middle width=331.3169772pt height=24.657534pt/>
  model relience: <img src="svgs/246d2949518f0e072a94de9eb5a00789.svg?invert_in_darkmode" align=middle width=277.7566704pt height=30.648288pt/>
  a population-level model class reliance (MCR) range: <img src="svgs/ffa3ac10030479bc13de83a17339abe9.svg?invert_in_darkmode" align=middle width=469.3592046pt height=27.9453933pt/>
  - If <img src="svgs/221ba46e23343b21cce257f1ad15c790.svg?invert_in_darkmode" align=middle width=73.51693965pt height=24.657534pt/> is low, then <img src="svgs/73f8e771d39b7b5b9273602159843676.svg?invert_in_darkmode" align=middle width=17.83492755pt height=14.1552444pt/> well-performing model in <img src="svgs/84cc939597f3eec200843a2fc8830732.svg?invert_in_darkmode" align=middle width=13.447467pt height=22.4657235pt/> places high importance on <img src="svgs/ef94a731f1a699515d87dca8d785af8e.svg?invert_in_darkmode" align=middle width=25.55943555pt height=22.4657235pt/> and <img src="svgs/9efd0126287224eeab878b4d0b47b73c.svg?invert_in_darkmode" align=middle width=20.17129785pt height=22.4657235pt/> can be discarded at low cost regardless of future modeling decisions. Similarly for <img src="svgs/aec420cc680f5f9a2f41f3c8f47d7c4f.svg?invert_in_darkmode" align=middle width=73.69956825pt height=24.657534pt/>
  the set of functions <img src="svgs/ff02df9d466f24ac28a9a6651c5544b7.svg?invert_in_darkmode" align=middle width=16.23606435pt height=22.4657235pt/> as an <img src="svgs/89f2e0d2d24bcf44db73aab8fc03252c.svg?invert_in_darkmode" align=middle width=7.8729552pt height=14.1552444pt/> -margin-expectation-cover if for any <img src="svgs/83f4b767e68d7919b1dfbb786c10d92a.svg?invert_in_darkmode" align=middle width=43.3560171pt height=22.8310566pt/> and any distribution <img src="svgs/e03ebcf17dcf2d3f40dc9021b39a1cde.svg?invert_in_darkmode" align=middle width=18.63245505pt height=22.4657235pt/> there exists <img src="svgs/999c928b27f4389347a61bf0a4e60c1e.svg?invert_in_darkmode" align=middle width=44.7575601pt height=22.4657235pt/> such that <img src="svgs/14a9d4a60f0054370c550fc4cb835c28.svg?invert_in_darkmode" align=middle width=176.8293186pt height=24.657534pt/>
  the covering number <img src="svgs/76c49413e779522d8543e9095fa14ade.svg?invert_in_darkmode" align=middle width=57.3215874pt height=24.657534pt/> to be the size of the smallest <img src="svgs/89f2e0d2d24bcf44db73aab8fc03252c.svg?invert_in_darkmode" align=middle width=7.8729552pt height=14.1552444pt/> -margin-expectationcover for <img src="svgs/84cc939597f3eec200843a2fc8830732.svg?invert_in_darkmode" align=middle width=13.447467pt height=22.4657235pt/>
  
  Возможны следующие вариации empirical MR:
  <img src="svgs/e6e15a434fb7ff91803b90f86f96bac0.svg?invert_in_darkmode" align=middle width=141.37212045pt height=33.2053986pt/>
  <img src="svgs/908f1d9e35f3d768d48e741f931f6914.svg?invert_in_darkmode" align=middle width=478.23323955pt height=84.6738387pt/>
  <img src="svgs/2a0f84fec0172ae51bfe0b983b3edf64.svg?invert_in_darkmode" align=middle width=326.00016075pt height=27.9453933pt/>
  <img src="svgs/1061f8fca3bf89f6d24b9d1c8575eae5.svg?invert_in_darkmode" align=middle width=419.4236574pt height=27.9453933pt/>
  - The estimators <img src="svgs/c0b1011abbb7fe0877418710b9a042f0.svg?invert_in_darkmode" align=middle width=136.4868285pt height=24.657534pt/> and <img src="svgs/f958022a491e85342a86708a4aea6f40.svg?invert_in_darkmode" align=middle width=70.22293575pt height=24.657534pt/> all belong to the well-studied class of U-statistics. Thus, under fairly minor conditions, *these estimators are unbiased, asymptotically normal, and have finite-sample probabilistic bounds*
  
  We introduce three bounded loss assumptions:
  **Assumption 1** (*Bounded individual loss*) For a given model <img src="svgs/395cf4bf7dfc7f2e2da51b7a9a84d77c.svg?invert_in_darkmode" align=middle width=47.92223535pt height=22.8310566pt/> assume that <img src="svgs/d00be88766456732b157a8a360587de6.svg?invert_in_darkmode" align=middle width=25.57074135pt height=21.1872144pt/> <img src="svgs/0362b866c8b100f031c67717d70a820c.svg?invert_in_darkmode" align=middle width=165.13601775pt height=24.657534pt/> for any <img src="svgs/0b87ec9fbf75c80286c35c55797da12c.svg?invert_in_darkmode" align=middle width=193.18192575pt height=24.657534pt/>
  **Assumption 2** (*Bounded relative loss*) For a given model <img src="svgs/395cf4bf7dfc7f2e2da51b7a9a84d77c.svg?invert_in_darkmode" align=middle width=47.92223535pt height=22.8310566pt/> assume that <img src="svgs/c0ed02762c6b0ac304747f5bf91ffe75.svg?invert_in_darkmode" align=middle width=137.1650412pt height=24.657534pt/> <img src="svgs/63d9f5512ad2b947f85f05497d90b219.svg?invert_in_darkmode" align=middle width=187.79617395pt height=24.657534pt/> for any <img src="svgs/64b09dfb5f3a9ec25bf5d7c35d66cbb6.svg?invert_in_darkmode" align=middle width=102.8955939pt height=24.657534pt/>
  **Assumption 3** (*Bounded aggregate loss*) For a given model <img src="svgs/395cf4bf7dfc7f2e2da51b7a9a84d77c.svg?invert_in_darkmode" align=middle width=47.92223535pt height=22.8310566pt/> assume that <img src="svgs/bb133ec4145b6d217554879fbcb0294e.svg?invert_in_darkmode" align=middle width=43.83563745pt height=24.657534pt/> <img src="svgs/5e52e9438c61cb4a30a51bb74ec1b3ab.svg?invert_in_darkmode" align=middle width=350.7039987pt height=24.657534pt/>
  - some constants can be derived from others
  
  <img src="svgs/1123b97c563df95763ebf1210a150e12.svg?invert_in_darkmode" align=middle width=644.17306305pt height=71.232843pt/>
  reliance among models in <img src="svgs/9eaf794bfc810492a9f1455999b1baf9.svg?invert_in_darkmode" align=middle width=37.9556166pt height=24.657534pt/> If <img src="svgs/c1775ebbacd300a3b1ef7df2941ffdff.svg?invert_in_darkmode" align=middle width=27.5224092pt height=22.8310566pt/> and <img src="svgs/ff8ee454f3798d20024ce335ac8d0cd5.svg?invert_in_darkmode" align=middle width=27.7050576pt height=22.8310566pt/> satisfy Assumptions 1,2 \& 3 , then
  <p align="center"><img src="svgs/f38ac706243053d55bc490f029b80a83.svg?invert_in_darkmode" align=middle width=583.70409405pt height=103.69392825pt/></p>
  - <img src="svgs/60c3857632a60c4b1c15ad6522e07d80.svg?invert_in_darkmode" align=middle width=434.99335605pt height=22.8310566pt/>![img_13.png](img_13.png)
  - with high probability, the largest possible estimation error for <img src="svgs/90799cd0f3f2624e7fb9c1a986dba025.svg?invert_in_darkmode" align=middle width=52.95103275pt height=24.657534pt/> across all models in <img src="svgs/84cc939597f3eec200843a2fc8830732.svg?invert_in_darkmode" align=middle width=13.447467pt height=22.4657235pt/> is bounded by <img src="svgs/9cec0385363b3dee00c10f8cbbf0dd5f.svg?invert_in_darkmode" align=middle width=59.16670155pt height=24.657534pt/>, which can be made arbitrarily small by increasing <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.86687625pt height=14.1552444pt/> and decreasing <img src="svgs/89f2e0d2d24bcf44db73aab8fc03252c.svg?invert_in_darkmode" align=middle width=7.8729552pt height=14.1552444pt/> 
  - The  existence  of  this  uniform  bound  implies  that  it  is  feasible  to  train  a  model  and  to evaluate its importance using the *same data*
  **Theorem 6** (*"Inner" MCR Bounds*) Given constants <img src="svgs/434f6d79cf3e7605879c945f0bcce27a.svg?invert_in_darkmode" align=middle width=36.80923125pt height=21.1872144pt/> and <img src="svgs/efa3244ceb1894cbd7e062da6bb9ebf0.svg?invert_in_darkmode" align=middle width=42.5760192pt height=21.1872144pt/> if Assumptions 1 , 2 and 3 hold for all <img src="svgs/395cf4bf7dfc7f2e2da51b7a9a84d77c.svg?invert_in_darkmode" align=middle width=47.92223535pt height=22.8310566pt/> and then
  <p align="center"><img src="svgs/fdf07c3044bc695a82c01119039fab76.svg?invert_in_darkmode" align=middle width=333.1536153pt height=59.1786591pt/></p>
  
  Calculating Empirical Estimates of MCR
  - computing <img src="svgs/17c03d0793e9a6170c1cf7020c514308.svg?invert_in_darkmode" align=middle width=73.6439253pt height=34.3683945pt/> however will require that we are able to minimize arbitrary linear combinations of <img src="svgs/2b98820d751e478539757714225e28c0.svg?invert_in_darkmode" align=middle width=57.4717209pt height=24.657534pt/> and <img src="svgs/daf800b1b76c20be39f70242e874487e.svg?invert_in_darkmode" align=middle width=71.7092244pt height=24.657534pt/>
  - we present bound functions <img src="svgs/49e648991ffdfcbccd3b3fb2005ebe4b.svg?invert_in_darkmode" align=middle width=17.3288181pt height=22.8310566pt/> and <img src="svgs/706a470865330c718598b5dc21ed291e.svg?invert_in_darkmode" align=middle width=17.1461697pt height=22.8310566pt/> satisfying <img src="svgs/4ea8c210394ae0fbc804a501d7f4f746.svg?invert_in_darkmode" align=middle width=77.5422813pt height=24.657534pt/> <img src="svgs/6cf76cf8bcf20ee06c13946ba1c7bc3f.svg?invert_in_darkmode" align=middle width=139.30602675pt height=32.9680131pt/> simultaneously for all <img src="svgs/33a26985ed3f74c6c328d961bdb4040f.svg?invert_in_darkmode" align=middle width=306.673455pt height=24.657534pt/>
  - almost all of the results shown in this section, and those in Section 6.2 . also hold if we replace <img src="svgs/d758b434ea5969b8eb5cf4ea54c47c3e.svg?invert_in_darkmode" align=middle width=43.85524275pt height=22.8310566pt/> with <img src="svgs/c96dc920c4b6fd8b88f1e068eff8fe4f.svg?invert_in_darkmode" align=middle width=42.3689541pt height=22.8310566pt/> throughout (see Eq 3.5), including in the definition of <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> and <img src="svgs/59e22f4aaefc5ca42e8f5a653371b456.svg?invert_in_darkmode" align=middle width=59.25291075pt height=31.5068952pt/> 
    
  Binary Search for Empirical MR Lower Bound
  - Condition 8 (Criteria to continue search for <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> lower bound) <img src="svgs/f78966e0991a9e03d6ee41a14a557d3f.svg?invert_in_darkmode" align=middle width=108.19909155pt height=31.5068952pt/> and <img src="svgs/98e16b9bf83e07cded0253295956f7a3.svg?invert_in_darkmode" align=middle width=128.55289755pt height=24.657534pt/>
  - **Lemma 9** (*Lower bound for <img src="svgs/d698d55cafb480794e1b57edc9f05574.svg?invert_in_darkmode" align=middle width=44.96010255pt height=32.9680131pt/> If <img src="svgs/6fd73a861ca99d39a24a5cc49413f9e5.svg?invert_in_darkmode" align=middle width=41.38717935pt height=22.6483917pt/> satisfies <img src="svgs/546ffd587a2a4c3c2eeb3f7482ce8f31.svg?invert_in_darkmode" align=middle width=112.76531475pt height=31.5068952pt/> then
    $$
    \frac{\hat{h}_{-, \gamma}\left(\hat{g}_{-, \gamma}\right)}{\epsilon_{a b s}}-\gamma \leq \widehat{M R}(f)
    $$
    for all $f \in \mathcal{F}$ satisfying $\hat{e}_{\text {orig }}(f) \leq \epsilon_{a b s} .$ It also follows that
    $$
    -\gamma \leq \widehat{M R}(f) \quad \text { for all } f \in \mathcal{F}
    $$
    Additionally, if $f=\hat{g}_{-, \gamma}$ and at least one of the inequalities in Condition 8 holds with equality, then top Eq holds with equality.
  - It remains to determine which value of <img src="svgs/11c596de17c342edeed29f489aa4b274.svg?invert_in_darkmode" align=middle width=9.42388095pt height=14.1552444pt/> should be used in top Eq. The following lemma implies that this value can be determined by a binary search, given a particular value of interest for <img src="svgs/8ea213afa93e648ee39ecc7d61fcf6c9.svg?invert_in_darkmode" align=middle width=25.6930476pt height=14.1552444pt/>
  - **Lemma 10** (*Monotonicity for <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> lower bound binary search*) The following monotonicity results hold:
    1. $\hat{h}_{-, \gamma}\left(\hat{g}_{-, \gamma}\right)$ is monotonically increasing in $\gamma$.
    2. $\hat{e}_{\text {orig }}\left(\hat{g}_{-, \gamma}\right)$ is monotonically decreasing in $\gamma$.
    3. Given $\epsilon_{a b s},$ the lower bound from $E q 6.1,\left\{\frac{\hat{h}_{-, \gamma}\left(\hat{g}_{-, \gamma}\right)}{\epsilon_{a b s}}-\gamma\right\},$ is monotonically decreasing in $\gamma$ in the range where $\hat{e}_{\text {oria }}\left(\hat{g}_{-, \gamma}\right) \leq \epsilon_{a b s},$ and increasing otherwise.
    - the value of $\gamma$ resulting in the tightest lower bound from Eq 6.1 occurs when $\gamma$ is as low as possible while still satisfying Condition 8
  - **Proposition 11** (*Nonnegative weights for <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> lower bound binary search*) Assume that <img src="svgs/ddcb483302ed36a59286424aa5e0be17.svg?invert_in_darkmode" align=middle width=11.18724255pt height=22.4657235pt/> and <img src="svgs/84cc939597f3eec200843a2fc8830732.svg?invert_in_darkmode" align=middle width=13.447467pt height=22.4657235pt/> satisfy the following conditions.
    1. (Predictions are sufficient for computing the loss) The loss $L\left\{f,\left(Y, X_{1}, X_{2}\right)\right\}$ depends on the covariates $\left(X_{1}, X_{2}\right)$ only via the prediction function $f,$ that is, $L\left\{f,\left(y, x_{1}^{(a)}, x_{2}^{(a)}\right)\right\}=$
    $$
    L\left\{f,\left(y, x_{1}^{(b)}, x_{2}^{(b)}\right)\right\} \text { whenever } f\left(x_{1}^{(a)}, x_{2}^{(a)}\right)=f\left(x_{1}^{(b)}, x_{2}^{(b)}\right)
    $$
    2. (Irrelevant information does not improve predictions) For any distribution $D$ satisfying $X_{1} \perp_{D}\left(X_{2}, Y\right)$, there exists a function $f_{D}$ satisfying
    $$
    \mathbb{E}_{D} L\left\{f_{D},\left(Y, X_{1}, X_{2}\right)\right\}=\min _{f \in \mathcal{F}} \mathbb{E}_{D} L\left\{f,\left(Y, X_{1}, X_{2}\right)\right\} 
    $$
    and
    $$
    f_{D}\left(x_{1}^{(a)}, x_{2}\right)=f_{D}\left(x_{1}^{(b)}, x_{2}\right) \text { for any } x_{1}^{(a)}, x_{1}^{(b)} \in \mathcal{X}_{1} \text { and } x_{2} \in \mathcal{X}_{2}
    $$ 
    Let $\gamma=0$. Under the above assumptions, it follows that either (i) there exists a function $\hat{g}_{-, 0}$ minimizing $\hat{h}_{-, 0}$ that does not satisfy Condition $8,$ or $($ ii $) \hat{e}_{\text {orig }}\left(\hat{g}_{-, 0}\right) \leq \epsilon_{a b s}$ and $\widehat{M R}\left(g_{-, 0}\right) \leq$ 1 for any function $\hat{g}_{-, 0}$ minimizing $\hat{h}_{-, 0}$.
    - tractability of our approach, as minimizing $\hat{h}_{-, \gamma}$ for $\gamma \geq 0$ is equivalent to minimizing reweighted empirical loss over an expanded sample of size $n^{2}$ :
      $$
      \hat{h}_{-, \gamma}(f)=\gamma \hat{e}_{\text {orig }}(f)+\hat{e}_{\text {switch }}(f)=\sum_{i=1}^{n} \sum_{j=1}^{n} w_{\gamma}(i, j) L\left\{f,\left(\mathbf{y}_{[i]}, \mathbf{X}_{1[j, \cdot]}, \mathbf{X}_{2[i, \cdot]}\right)\right\}
      $$
      where $w_{\gamma}(i, j)=\frac{\gamma 1(i=j)}{n}+\frac{1(i \neq j)}{n(n-1)} \geq 0$
      
  Binary Search for Empirical MR Upper Bound
  - <img src="svgs/0e966d02453e06d54e13be3a54b050da.svg?invert_in_darkmode" align=middle width=244.25013525pt height=31.5068952pt/> and <img src="svgs/371d351a36ba28cf9b36f9e86d04fcaa.svg?invert_in_darkmode" align=middle width=172.5885018pt height=32.3286942pt/>
  - Given an observed sample, we define the following condition for a pair of values <img src="svgs/b4635f8bf548f2dce0e282964bd75129.svg?invert_in_darkmode" align=middle width=75.2081418pt height=24.657534pt/> <img src="svgs/bb9800eb58b40dd0ed5704c9b5310a59.svg?invert_in_darkmode" align=middle width=83.6986887pt height=22.6483917pt/> and argmin function <img src="svgs/527755e80d779eb423ee66bb3decc25c.svg?invert_in_darkmode" align=middle width=39.40301805pt height=22.8310566pt/>
    Condition $12 \quad$ (Criteria to continue search for $\widehat{M R}$ upper bound) $\hat{h}_{+, \gamma}\left(\hat{g}_{+, \gamma}\right) \geq 0$ and $\hat{e}_{\text {orig }}\left(\hat{g}_{+, \gamma}\right) \leq \epsilon_{a b s}$
    We can now develop a procedure to upper bound $\widehat{M R}$, as shown in the next lemma.
    Lemma 13 (Upper bound for $\widehat{M R}$ ) If $\gamma \in \mathbb{R}$ satisfies $\gamma \leq 0$ and $\hat{h}_{+, \gamma}\left(\hat{g}_{+, \gamma}\right) \geq 0,$ then
    $$
    \widehat{M R}(f) \leq\left\{\frac{\hat{h}_{+, \gamma}\left(\hat{g}_{+, \gamma}\right)}{\epsilon_{a b s}}-1\right\} \gamma^{-1}
    $$
    for all $f \in \mathcal{F}$ satisfying $\hat{e}_{\text {orig }}(f) \leq \epsilon_{a b s} .$ It also follows that
    $$
    \widehat{M R}(f) \leq\left|\gamma^{-1}\right| \quad \text { for all } f \in \mathcal{F}
    $$
    Additionally, if $f=\hat{g}_{+, \gamma}$ and at least one of the inequalities in Condition 12 holds with equality, then Eq 6.4 holds with equality. 
    3. Given $\epsilon_{a b s},$ the upper boundary $\left\{\frac{\hat{h}_{+, \gamma}\left(\hat{g}_{+, \gamma}\right)}{\epsilon_{a b s}}-1\right\} \gamma^{-1}$ is monotonically increasing in in the range where $\hat{e}_{\text {orig }}\left(\hat{g}_{+, \gamma}\right) \leq \epsilon_{a b s}$ and $\gamma<0,$ and decreasing in the range where $\hat{e}_{\text {orig }}\left(\hat{g}_{+, \gamma}\right)>\epsilon_{a b s}$ and $\gamma<0$
    Together, the results from Lemma 14 imply that we can use a binary search across $\gamma \in \mathbb{R}$ to tighten the boundary on $\widehat{M R}$ from Lemma $13 .$
  
  Convex Models
  - идея: пусть функции параметризуются некоторым вектором переменных, разобьём это пространство на симплексы, на них h совпадает в вершинах с некоторой гиперплоскостью, заменяем h её, получаем нижнию оценку, так для всех подвыборок из пространства и индуктивно повторяем процедуру
  
  MR & MCR for Linear Models, Additive Models
  - Throughout this section, we assume that <img src="svgs/525d2b68f96a33db3c9d6903b457a4cc.svg?invert_in_darkmode" align=middle width=54.6987507pt height=22.6483917pt/> for <img src="svgs/6f1cc81a5cf62297d38807247fd66323.svg?invert_in_darkmode" align=middle width=54.8001564pt height=26.1773094pt/> that <img src="svgs/df0525dd40363d25686f8908e12835aa.svg?invert_in_darkmode" align=middle width=58.0684467pt height=26.7617526pt/> and that <img src="svgs/ddcb483302ed36a59286424aa5e0be17.svg?invert_in_darkmode" align=middle width=11.18724255pt height=22.4657235pt/> is the squared error loss function <img src="svgs/923d1ef8595c8ce2325a43ac73658a77.svg?invert_in_darkmode" align=middle width=245.722785pt height=37.8085059pt/>.
  - **Proposition 15** (*Interpreting <img src="svgs/a31992ee5753d0c3a9729a5e32257c37.svg?invert_in_darkmode" align=middle width=34.9144092pt height=22.4657235pt/> and computing empirical MR for linear models*) For any prediction model <img src="svgs/6f60e7714a307b2095b24945797a63b4.svg?invert_in_darkmode" align=middle width=13.4703921pt height=22.8310566pt/> let <img src="svgs/f76c52abf47fa74538b1bc013b6119a6.svg?invert_in_darkmode" align=middle width=205.83066405pt height=24.657534pt/> and <img src="svgs/daf800b1b76c20be39f70242e874487e.svg?invert_in_darkmode" align=middle width=71.7092244pt height=24.657534pt/> be defined based on the squared error loss <img src="svgs/6c90a58890f7ff2a437bb89bf09b2e32.svg?invert_in_darkmode" align=middle width=253.25707275pt height=31.3608075pt/> for <img src="svgs/e4fe82e8dcbe27f21551c668ef1b30ad.svg?invert_in_darkmode" align=middle width=115.23138495pt height=22.6483917pt/> and <img src="svgs/de660c07e8f4febe3f8c5cb400a4dd55.svg?invert_in_darkmode" align=middle width=61.1029419pt height=22.6483917pt/>
    where $p_{1}$ and $p_{2}$ are positive integers. Let $\beta=\left(\beta_{1}, \beta_{2}\right)$ and $f_{\beta}$ satisfy $\beta_{1} \in \mathbb{R}^{p_{1}}, \beta_{2} \in \mathbb{R}^{p_{2}}$ and $f_{\beta}(x)=x^{\prime} \beta=x_{1}^{\prime} \beta_{1}+x_{2}^{\prime} \beta_{2} .$ Then
    $$
    M R\left(f_{\beta}\right)=1+\frac{2}{e_{\text {orig }}\left(f_{\beta}\right)}\left\{\operatorname{Cov}\left(Y, X_{1}\right) \beta_{1}-\beta_{2}^{\prime} \operatorname{Cov}\left(X_{2}, X_{1}\right) \beta_{1}\right\}
    $$
    and, for finite samples,
    $$
    \left.\hat{e}_{\text {switch }}\left(f_{\beta}\right)=\frac{1}{n}\left\{\begin{array}{l}
    \mathbf{y}^{\prime} \mathbf{y}-2\left[\begin{array}{c}
    \mathbf{X}_{1}^{\prime} \mathbf{W} \mathbf{y} \\
    \mathbf{X}_{2}^{\prime} \mathbf{y}
    \end{array}\right]^{\prime}
    \end{array}\right] \beta+\beta^{\prime}\left[\begin{array}{cc}
    \mathbf{X}_{1}^{\prime} \mathbf{X}_{1} & \mathbf{X}_{1}^{\prime} \mathbf{W} \mathbf{X}_{2} \\
    \mathbf{X}_{2}^{\prime} \mathbf{W} \mathbf{X}_{1} & \mathbf{X}_{2}^{\prime} \mathbf{X}_{2}
    \end{array}\right] \beta\right\}
    $$
    where $\mathbf{W}:=\frac{1}{n-1}\left(\mathbf{1}_{n} \mathbf{1}_{n}^{\prime}-\mathbf{I}_{n}\right), \mathbf{1}_{n}$ is the $n$ -length vector of ones, and $\mathbf{I}_{n}$ is the $n \times n$
    identity matrix.
    - сложность вычисления растёт линейно (можно расписать)
  - **Remark 16** (*Tractability of empirical MCR for linear model classes*) For any <img src="svgs/0b3224020b22b8293953c6417cc644b2.svg?invert_in_darkmode" align=middle width=64.70986995pt height=22.8310566pt/> and any fixed coefficients <img src="svgs/0409c0ab67c9122543d5824c2a06cdb9.svg?invert_in_darkmode" align=middle width=126.8860329pt height=22.8310566pt/> the linear combination
    $$
    \xi_{\text {orig }} \hat{e}_{\text {orig }}\left(f_{\beta}\right)+\xi_{\text {switch }} \hat{e}_{\text {switch }}\left(f_{\beta}\right)
    $$
    is proportional in $\beta$ to the quadratic function $-2 \mathbf{q}^{\prime} \beta+\beta^{\prime} \mathbf{Q} \beta,$ where
    $$
    \mathbf{Q}:=\xi_{\text {orig }} \mathbf{X}^{\prime} \mathbf{X}+\xi_{\text {switch }}\left[\begin{array}{cc}
    \mathbf{X}_{1}^{\prime} \mathbf{X}_{1} & \mathbf{X}_{1}^{\prime} \mathbf{W} \mathbf{X}_{2} \\
    \mathbf{X}_{2}^{\prime} \mathbf{W} \mathbf{X}_{1} & \mathbf{X}_{2}^{\prime} \mathbf{X}_{2}
    \end{array}\right], \quad \mathbf{q}:=\left(\xi_{\text {orig }} \mathbf{y}^{\prime} \mathbf{X}+\xi_{\text {switch }}\left[\begin{array}{c}
    \mathbf{X}_{1}^{\prime} \mathbf{W} \mathbf{y} \\
    \mathbf{X}_{2}^{\prime} \mathbf{y}
    \end{array}\right]^{\prime}\right)^{\prime}
    $$
    and $\mathbf{W}:=\frac{1}{n-1}\left(\mathbf{1}_{n} \mathbf{1}_{n}^{\prime}-\mathbf{I}_{n}\right) .$ Thus, minimizing $\xi_{\text {orig }} \hat{e}_{\text {orig }}\left(f_{\beta}\right)+\xi_{\text {switch }} \hat{e}_{\text {switch }}\left(f_{\beta}\right)$ is equivalent to an unconstrained (possibly non-convex) quadratic program.
  - <img src="svgs/b37e1c7739e7d50605fe8a0fd4e1eb9a.svg?invert_in_darkmode" align=middle width=409.1601789pt height=24.7161288pt/>
  - The resulting optimization problem is a (possibly non-convex) quadratic program withone  quadratic  constraint 
  - **Lemma 17** (*Loss upper bound for linear models*) If <img src="svgs/100b67cef52c22ef5bf59da78336d102.svg?invert_in_darkmode" align=middle width=33.83375655pt height=22.5570873pt/> is positive definite, <img src="svgs/91aac9730317276af725abd8cef04ca9.svg?invert_in_darkmode" align=middle width=13.1963865pt height=22.4657235pt/> is bounded within a known range, and there exists a known constant <img src="svgs/6940e84f35678b3aff684d787e5ef463.svg?invert_in_darkmode" align=middle width=18.5237019pt height=14.1552444pt/> such that <img src="svgs/6c698ef3df94efbf508a6c4c59849479.svg?invert_in_darkmode" align=middle width=99.43675995pt height=28.8949551pt/> for all <img src="svgs/061d1720a5d427021651f245afb95c39.svg?invert_in_darkmode" align=middle width=107.86838535pt height=24.657534pt/> then Assumption 1 holds for the model class <img src="svgs/e6a5639c07265f2bb3ea3c3691189a1e.svg?invert_in_darkmode" align=middle width=57.9626322pt height=22.4657235pt/> the squared error loss function, and the constant
    $$
    B_{\text {ind }}=\max \left[\left\{\min _{y \in \mathcal{Y}}(y)-\sqrt{r_{\mathcal{X}} r_{l m}}\right\}^{2},\left\{\max _{y \in \mathcal{Y}}(y)+\sqrt{r_{\mathcal{X}} r_{l m}}\right\}^{2}\right]
    $$
    
  Regression Models in a Reproducing Kernel Hilbert Space
  - <p align="center"><img src="svgs/8d42e24e03d916a3ccad8e93538da248.svg?invert_in_darkmode" align=middle width=523.8295458pt height=49.3155696pt/></p>
    Above, the norm $\left\|f_{\alpha}\right\|_{k}$ is defined as
    $$
    \left\|f_{\alpha}\right\|_{k}:=\sum_{i=1}^{R} \sum_{j=1}^{R} \alpha_{[i]} \alpha_{[j]} k\left(\mathbf{D}_{[i, \cdot]}, \mathbf{D}_{[j, \cdot]}\right)=\alpha^{\prime} \mathbf{K}_{\mathbf{D}} \alpha
    $$
  
  Calculating MCR
  - For any two constants <img src="svgs/0409c0ab67c9122543d5824c2a06cdb9.svg?invert_in_darkmode" align=middle width=126.8860329pt height=22.8310566pt/> we can show that minimizing the linear combination <img src="svgs/b068a4fd3be4b75e042fc286083c2f05.svg?invert_in_darkmode" align=middle width=252.99885435pt height=24.657534pt/> over <img src="svgs/3d8eb5d1919b6d0e2d2306f0219f4006.svg?invert_in_darkmode" align=middle width=39.5225523pt height=22.4657235pt/> is equivalent to the minimization problem
    $$
    \begin{array}{l}
    \text { minimize } \quad \frac{\xi_{\text {orig }}}{n}\left\|\mathbf{y}-\mu-\mathbf{K}_{\text {orig }} \alpha\right\|_{2}^{2}+\frac{\xi_{\text {switch }}}{n(n-1)}\left\|\mathbf{y}_{\text {switch }}-\mu-\mathbf{K}_{\text {switch }} \alpha\right\|_{2}^{2} \\
    \text { subject to } \alpha^{\prime} \mathbf{K}_{\mathbf{D}} \alpha<r_{k} .
    \end{array}
    $$
  
  Upper Bounding the Loss
  - **Lemma 18** (*Loss upper bound for regression in a RKHS*) Assume that <img src="svgs/91aac9730317276af725abd8cef04ca9.svg?invert_in_darkmode" align=middle width=13.1963865pt height=22.4657235pt/> is bounded within a known range, and there exists a known constant <img src="svgs/a34893f936d403e4c7359da7d0f1c543.svg?invert_in_darkmode" align=middle width=18.7519233pt height=14.1552444pt/> such that <img src="svgs/00f6d1fd9af3305908e15bd35bbd1b4c.svg?invert_in_darkmode" align=middle width=139.2236901pt height=28.8949551pt/> for all <img src="svgs/061d1720a5d427021651f245afb95c39.svg?invert_in_darkmode" align=middle width=107.86838535pt height=24.657534pt/> where <img src="svgs/a9ef3251a5017c4616da0900c628b14d.svg?invert_in_darkmode" align=middle width=89.13141435pt height=27.6567522pt/> is the function satisfying <img src="svgs/969ba4e57956c0e0fd6463b65af971d8.svg?invert_in_darkmode" align=middle width=151.6835991pt height=27.9453933pt/> Under these
    conditions, Assumption 1 holds for the model class $\mathcal{F}_{\mathrm{D}, r_{k}},$ the squared error loss function,
    and the constant
    $$
    B_{i n d}=\max \left[\left\{\min _{y \in \mathcal{Y}}(y)-\left(\mu+\sqrt{r_{\mathrm{D}} r_{k}}\right)\right\}^{2},\left\{\max _{y \in \mathcal{Y}}(y)+\left(\mu+\sqrt{r_{\mathrm{D}} r_{k}}\right)\right\}^{2}\right]
    $$
  
  Model Reliance and Causal Effects
  - Let <img src="svgs/43eaeea5accf88ac13066ee1ab6b9987.svg?invert_in_darkmode" align=middle width=214.69168875pt height=24.657534pt/>, <img src="svgs/de918639f648bb439f957cb95a016797.svg?invert_in_darkmode" align=middle width=228.33899715pt height=24.657534pt/>  
    Proposition 19 (Causal interpretations of MR) For any prediction model $f,$ let $e_{\text {orig }}(f)$ and $e_{\text {switch }}(f)$ be defined based on the squared error loss $L(f,(y, t, c)):=(y-f(t, c))^{2}$ If $\left(Y_{1}, Y_{0}\right) \perp T \mid C$ (conditional ignorability) and $0<\mathbb{P}(T=1 \mid C=c)<1$ for all values of c (positivity), then $M R\left(f_{0}\right)$ is equal to
    $$
    1+\frac{\operatorname{Var}(T)}{\mathbb{E}_{T, C} \operatorname{Var}(Y \mid T, C)} \sum_{t \in\{0,1\}}\left\{\mathbb{E}\left(Y_{1}-Y_{0} \mid T=t\right)^{2}+\operatorname{Var}(\operatorname{CATE}(C) \mid T=t)\right\}
    $$
    where $\operatorname{Var}(T)$ is the marginal variance of the treatment assignment.
  
  Conditional Importance:  Adjusting for Dependence BetweenX1andX2
  - <img src="svgs/9418b1e57f8320d5f6ed575832b1cc28.svg?invert_in_darkmode" align=middle width=576.29246895pt height=37.8085059pt/>
    $C M R(f)=\frac{e_{\text {cond }}(f)}{e_{\text {orig }}(f)}$
  - This means that CMR will not be influenced by impossible combinations of <img src="svgs/7e0dded6496a3ed3f3c0db74604087ac.svg?invert_in_darkmode" align=middle width=15.94753545pt height=14.1552444pt/> and <img src="svgs/345508ce4e933b712fe803f442f74d63.svg?invert_in_darkmode" align=middle width=15.94753545pt height=14.1552444pt/>, while MR may be influenced by them
  
  Estimation of CMR by Weighting, Matching, or Imputation
  - <img src="svgs/66a53a6f2a16676d89ee543e74e396f2.svg?invert_in_darkmode" align=middle width=561.7138626pt height=27.9453933pt/> ![img_38.png](img_38.png)
    - $\hat{e}_{\text {weight }}(f)$ is unbiased for $e_{\text {cond }}(f)$
  - <img src="svgs/d4e48e19eb2277145137d98de3d2cfd1.svg?invert_in_darkmode" align=middle width=544.4781012pt height=43.0684122pt/>
    - if the inverse probability weight $\mathbb{P}\left(X_{2}=\mathbf{X}_{2[i, \cdot}\right)^{-1}$ is known, then $\hat{e}_{\text {match }}(f)$ is unbiased for $e_{\text {cond }}(f)$ (see Appendix A.7).
  - However, when the covariate space is continuous or high dimensional, we typically cannotestimate  CMR  nonparametrically.
  - When the covariate space is continuous or high dimensional  
    Specifically, we define $\mu_{1}$ to be the conditional expectation function $\mu_{1}\left(x_{2}\right)=\mathbb{E}\left(X_{1} \mid X_{2}=x_{2}\right),$ and assume that the random residual $X$ $\mu_{1}\left(X_{2}\right)$ is independent of $X_{2}$. Under this assumption, it can be shown that
    $$
    e_{\text {cond }}(f)=\mathbb{E} L\left[f,\left(Y^{(b)},\left\{X_{1}^{(a)}-\mu_{1}\left(X_{2}^{(a)}\right)\right\}+\mu_{1}\left(X_{2}^{(b)}\right), X_{2}^{(b)}\right)\right]
    $$
  
  Simulations of Bootstrap Confidence Intervals  
  - идея: 
    1 подход
    возьмём ориг. датасет (20k записей), посчитаем на нем MCR, разделим весь датасет на 2 части
    training subset and analysis subset
    - на training subset: обучаем reference model
    - на analysis subset: сэмплируем выборку (500 times) и считаем emp_MCR_- и emp_MCR_+, а после CI
    
    2 подход (проще)
    cэмплируем выборку с ориг. датасета (500 times), делим его на 2 части
    - на 1ой части обучаем модель
    - на 2ой оцениваем её emp_MR
    получаем CI
  Итог: 1 подход more robust to the misspecification of the models used to approximate Y and the model of Y itself
  
  COMPAS score
  The bootstrap 95% CI for MCR on “inadmissiblevariables” is [1.00, 1.73]
  For “admissible variables” the MCR range with a 95% bootstrapCI is equal to [1.62, 3.96]
