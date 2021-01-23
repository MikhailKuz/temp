<!-- python -m readme2tex --output papers.md --nocdn --rerender papers_raw.md -->
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
  - Raw:
    - Our  motivation  is  that  Rashomon  sets  (defined  formally  below)  summarize  therange of effective prediction strategies that an analyst might choose
    - (MR). This measure is based on permutationimportance measures for Random Forests and can be  expanded  to  describe  conditional  importance
    - beyond describing variable importance, these tools can describe the range of riskpredictions that well-fitting models assign to a particular covariate profile, or the variance ofpredictions made by well-fitting models
    - To this end, we apply a broad class of flexible, kernel-basedprediction models to predict COMPAS score
    - Equipped with MCR, wecan relax the common assumption of being able to correctly specify the unknown model ofinterest (here, COMPAS) up to a parametric form
    - //We assume that observations ofZareiid, thatn≥2, and thatsolutions to arg min and arg max operations exist whenever optimizing over sets mentionedin this paper (for example, in Theorem 4, below)//
    - R()  :={f∈F:EL(f,Z)≤EL(fref,Z) +}
    - efwfew wefw ![alt text](./equation.svg "Title") efwefewf wefwefew
    - feewfwefw weeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee
    - efwfew wefw ![alt text](./1.svg "Title") efwefewf wefwefe
    - feewfwefw weeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee
  - Основные формулы:  
  ![alt text](./1.svg "Title")![alt text](./1.svg "Title")![alt text](./1.svg "Title")![alt text](./1.svg "Title")
  
  *Итоги экспериментов*:  
    - NRFE и RFE в целом имеют одинаковое качество

