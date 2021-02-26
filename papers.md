<!--- 
activate p2
python -m readme2tex --nocdn --output papers.md --rerender papers_raw.md 
-->
# Конспекты статей
1. [Thiagarajan JJ at el (2020) Accurate and Robust Feature Importance Estimation under Distribution Shifts](#6)

## <a name="1" /> Thiagarajan JJ at el (2020) [Accurate and Robust Feature Importance Estimation under Distribution Shifts](https://arxiv.org/pdf/2009.14454.pdf)
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

## <a name="2" /> Wojtas M, Chen K (2020) [Feature Importance Ranking for Deep Learning](https://arxiv.org/pdf/2010.08973.pdf)
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

## <a name="3" /> Gimenez JR, Ghorbani A, Zou J (2019) [Knockoffs for the mass: new feature importance statistics with false discovery guarantees](https://arxiv.org/pdf/1807.06214.pdf)
- аппроксимируется распределение данных (только признаки) байесовскими сетями
- используется аугментация выборки определённым образом (чтобы не выходить за исходное распределение)
- вместо того, чтобы перемешивать значения признака (в permutation importance), берется взвешенная сумма исходного признака и соответствующего признака из аугментированной выборки
- важность признака - площадь под кривой (y - доля правильно отобранных признаков, x - параметр взвешенной суммы) для некоторого диапазона (например, [0, 10])
- FDR в реальности нельзя оценить, предполагается, что мы хорошо моделируем распределение выборки

## <a name="4" /> Lundberg S, Lee S (2017) [A Unified Approach to Interpreting Model Predictions](https://proceedings.neurips.cc/paper/2017/file/8a20a8621978632d76c43dfd28b67767-Paper.pdf)
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

## <a name="w" /> Shrikumar A, Greenside P, Kundaje A (2017) [Learning Important Features Through Propagating Activation Differences](https://arxiv.org/pdf/1704.02685.pdf)
- метод основан на разнице значений нейронов между начальным значением (reference) и конечным
- разделяются положительный и отрицательный вклады в целевую переменную
- важность признака линейно зависит от разницы x - x_reference
- важность - shapley value с количеством разбиений 2
- x_reference выбирается в зависимости от задачи  
*Итог*:
- использование разности x - x_reference позволяет информации распространяться когда градиент равен нулю
- gradients, gradients*input, guided backprop, rescale rule теряют зависимости в ходе вычисления важности в некоторых случаях в отличии от предложенного reveal_cancel rule

## <a name="1" /> Schwab P, Karlen W (2019) [CXPlain: Causal Explanations for Model Interpretation under Uncertainty](https://arxiv.org/pdf/1910.12336.pdf)
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

## <a name="1" /> Strobl C, Boulesteix A, Zeileis A & Hothorn T (2017) [Bias in random forest variable importance measures](https://link.springer.com/content/pdf/10.1186/1471-2105-8-25.pdf)
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

## <a name="6"/> Gregorutti B, Michel B, Saint-Pierre P (2015) [Grouped variable importance with random forests and application to multiple functional data analysis](https://arxiv.org/pdf/1411.4170.pdf)
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

## <a name="1" /> Gregorutti B, Michel B, Saint-Pierre P (2017) [Correlation and variable importance in random forests](https://arxiv.org/pdf/1310.5726.pdf)**
- продолжение вышеописанной работы 
- эмпирическая важность признака при использовании purely rf для независимых признаков сходится экспоненциально к теоретической при стремлении количества итераций разбиения узла дерева и мощности тренировочной выборки так, чтобы отношение первого ко второму стремилось к 0 
- даже сильно коррелирующие признаки с целевой переменной могут получить малую важность из-за корреляции между собой  

*Итоги экспериментов*:
- NRFE и RFE в целом имеют одинаковое качество
  

## <a name="1" /> Kononenko I et al (2010) [An efficient explanation of individual classifications using game theory](https://www.jmlr.org/papers/volume11/strumbelj10a/strumbelj10a.pdf)
We present *a general method* for explaining *individual predictions* of *classification models*.  
**Example of the disadvantage of methods related to masking only 1 feature** Let model is 1 or 1, then importance of left and right 1s will be 0  
**Theorem** Let:  
<p align="center"><img src="svgs/b3927fd163eeae8c0c35d1224f9cf937.svg?invert_in_darkmode" align=middle width=432.5447907pt height=66.65529255pt/></p>  
<p align="center"><img src="svgs/85b71bfeed397922c6f3aba9128cfdfe.svg?invert_in_darkmode" align=middle width=206.0751033pt height=37.77510825pt/></p>  
<p align="center"><img src="svgs/8b125fec7ab2723e9bd4c36fca95ade6.svg?invert_in_darkmode" align=middle width=252.98866725pt height=36.90432735pt/></p>  
<p align="center"><img src="svgs/8b13b38feb58f021f2bd1072e8b2652f.svg?invert_in_darkmode" align=middle width=325.4694069pt height=46.7371938pt/></p>  

Then: <img src="svgs/7c08dd7b47762244a86b430482f32b92.svg?invert_in_darkmode" align=middle width=157.28660145pt height=24.657534pt/> is a coalitional form game and <img src="svgs/e613f9a343b62108a626dab1bbb87312.svg?invert_in_darkmode" align=middle width=167.89749735pt height=24.657534pt/> corresponds to the game's Shapley value <img src="svgs/b6e82acdaa64f61189e1da6233ae4ea6.svg?invert_in_darkmode" align=middle width=44.74900155pt height=24.657534pt/>.  
**Main idea of the paper**  To use bootstrap sampling (with replacement) + the folowing definition:
<p align="center"><img src="svgs/56db687dd5d2514559bc51c7c69df228.svg?invert_in_darkmode" align=middle width=580.50676695pt height=45.00203565pt/></p>  

The key to minimizing the number of samples is *to estimate the sample variance* and draw the appropriate number of samples.  
The optimal (*minimal*) number of samples we need for the entire explanation is  <img src="svgs/a37bb58dc5d1a115b19d5c755b4e1e85.svg?invert_in_darkmode" align=middle width=212.21608155pt height=41.1466572pt/>  

### Experiments
The greater the variance in the model responses, the more samples will be required. For example, MLP have higher error/samples rate then logreg, dt, nb.

  
## <a name="1" /> Datta A, Sen S, Zick Y (2016) [Algorithmic transparency via quantitative input influence: theory and experiments with learning systems](http://www.andrew.cmu.edu/user/danupam/datta-sen-zick-oakland16.pdf) [[code]](https://github.com/hovinh/QII)
**Важность** - влияние input на интересующую функцию от <img src="svgs/c11cd2e3029f26a6a31c3a9ac4a5c75b.svg?invert_in_darkmode" align=middle width=9.97711605pt height=14.6118786pt/>  

Consider **expanded  probability  space** on <img src="svgs/3d2ab5621d0e3b8ba2a24770cb64eb7c.svg?invert_in_darkmode" align=middle width=48.35611935pt height=22.4657235pt/>, with distribution <img src="svgs/743e458cb82e29385967a897f829f72c.svg?invert_in_darkmode" align=middle width=138.4187607pt height=24.657534pt/>. Вариацию элемента из датасета будет делать с помощью 2-го вероятностного пространства.  
For a quantity  of interes <img src="svgs/4a9bf1922de1393a571127c8bf3b114a.svg?invert_in_darkmode" align=middle width=41.7277146pt height=24.657534pt/>, and an input <img src="svgs/77a3b857d53fb44e33b53e4c8b68351a.svg?invert_in_darkmode" align=middle width=5.6632257pt height=21.6830097pt/>, **the Quantitative Input Influence** of <img src="svgs/77a3b857d53fb44e33b53e4c8b68351a.svg?invert_in_darkmode" align=middle width=5.6632257pt height=21.6830097pt/> on <img src="svgs/4a9bf1922de1393a571127c8bf3b114a.svg?invert_in_darkmode" align=middle width=41.7277146pt height=24.657534pt/> is defined to be  
<p align="center"><img src="svgs/165b14f3f1928fc05d5033d72aab9887.svg?invert_in_darkmode" align=middle width=226.0421493pt height=18.7598829pt/></p>  

*QII for Individual Outcomes*  
<p align="center"><img src="svgs/7c26eeade99c63335a9b5c26962019d9.svg?invert_in_darkmode" align=middle width=424.78349595pt height=16.438356pt/></p>  

*QII for Group Outcomes*  
<p align="center"><img src="svgs/4f552a0189d3b5cf332e08010515704c.svg?invert_in_darkmode" align=middle width=422.13070515pt height=20.95157625pt/></p>  
<p align="center"><img src="svgs/881d5f1752d720149616182e5337ff88.svg?invert_in_darkmode" align=middle width=428.30357625pt height=51.3856596pt/></p>  

*QII for Group Disparity*  
<p align="center"><img src="svgs/02316401afc24f0b6eb83c2febf22757.svg?invert_in_darkmode" align=middle width=384.38888895pt height=21.8400138pt/></p>  
<p align="center"><img src="svgs/c3f5b34c233bbfffb967c9a8c5faa840.svg?invert_in_darkmode" align=middle width=267.65091045pt height=21.8400138pt/></p>  
<p align="center"><img src="svgs/3620a0f76d361fd4a88877ca5d3f70db.svg?invert_in_darkmode" align=middle width=354.0841755pt height=59.1786591pt/></p>  

*Set  QII*  
<p align="center"><img src="svgs/441e6e259a63ecc504c7025036c1caa1.svg?invert_in_darkmode" align=middle width=206.1864618pt height=18.7598829pt/></p>  

*Marginal QII*  
<p align="center"><img src="svgs/9a288a77c3bf932e9114c2821cab745f.svg?invert_in_darkmode" align=middle width=317.39127585pt height=20.49507075pt/></p>  

:small_orange_diamond: для QII выполняются аксиомы, как для Shapley value  

### Transparency Schemas
It consists of the following elements:
- *A quantity of interest*, which captures the aspect of the system we wish to gain transparency into
- *An intervention distribution*, which defines how a counterfactual distribution is constructed from the true distribution
- *A difference measure*, which quantifies the difference between two quantities of interest
- *An aggregation technique*, which combines marginal QII measures across different subsets of inputs (features)
  
### Experiments  
В реальности сложность полного алгоритма большая -> сэмплируем выборку и на ней применяем алгоритм + добавляем шум <img src="svgs/a39614f21b3b5d05cb2247844922df04.svg?invert_in_darkmode" align=middle width=104.74126575pt height=24.657534pt/> для приватности. Доказано, что при достаточном n можно приблизить оценки истинных параметров сколь угодно близко (по вероятности).  
Для определения влияния признака не достаточно его одного изменить. Нужно другие тоже изменить. Это обусловлено наличием скоррелированных признаков.  
Features correlated with the sensitive attribute are the most influential for group disparity according to the sensitive attribute instead of the sensitive attribute itself. It is in this sense that **QII measures can identify proxy variables** that cause associations between outcomes and sensitive attributes.  

Filter methods for computing the importance of a feature:
- Mutual Information
- Jaccard Index
- Pearson Correlation
- Disparate Impact Ratio  

Embedded importance:
- Bayesian Rule Lists
- Supersparse Linear IntegerModels
- Probabilistic  Scaling  
  
### Appendix
**Shapley value** - <img src="svgs/9bdbff647fbb9cabbd799700c4b77575.svg?invert_in_darkmode" align=middle width=204.56316045pt height=24.6577353pt/>, where <img src="svgs/ecb3735a7e249535bca17707c6df48c4.svg?invert_in_darkmode" align=middle width=142.03270125pt height=76.6033587pt/>  
**The Banzhaf Index** defines as <img src="svgs/819f38106c0afccdcb86fe0c2685d3e9.svg?invert_in_darkmode" align=middle width=235.3303524pt height=27.7756545pt/>
- index  is  not  guaranteed  to  be  *efficient*, i.e. <img src="svgs/54db9a2e914365bceb7c3eb42f96db26.svg?invert_in_darkmode" align=middle width=103.575351pt height=24.6577353pt/> is not necessarily equal to <img src="svgs/fc6d302183dd80c822ba12c907941e10.svg?invert_in_darkmode" align=middle width=36.3432432pt height=24.657534pt/>. But satisfies *2-efficiency* property  
- the only function to satisfy (*sym*), (*d*), (*mono*) and (*2-eff*)
- tend to select sets with the cardinality equals to <img src="svgs/d6d54860f3796e33548482099695dec5.svg?invert_in_darkmode" align=middle width=26.30529495pt height=24.657534pt/>  

**The Deegan-Packel index** - <img src="svgs/b7d7d23521edb069027f70eeb0e1773e.svg?invert_in_darkmode" align=middle width=240.23633535pt height=27.7756545pt/>
- work with binary "outcome" (<img src="svgs/6c4adbc36120d62b98deef2a20d5d303.svg?invert_in_darkmode" align=middle width=8.5578603pt height=14.1552444pt/>) of subsets in <img src="svgs/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode" align=middle width=14.99998995pt height=22.4657235pt/>
  
  
## <a name="1" /> Fisher A, Rudin C, Dominici F (2018) [All models are wrong but many are useful: Variable importance for black-box, proprietary, or misspecified prediction models, using model class reliance](https://arxiv.org/pdf/1801.01489.pdf) [[code]](https://github.com/aaronjfisher/mcr-supplement)
**Идея** - будем искать важность группы признаков <img src="svgs/9efd0126287224eeab878b4d0b47b73c.svg?invert_in_darkmode" align=middle width=20.17129785pt height=22.4657235pt/> не для одной хорошей модели (reference model), а для класса моделей  
**Датасет** - iid  

### Введём несколько определений:  
*a population ε-Rashomon set*:  <img src="svgs/9342112aaa03b638367e4057db8fb58c.svg?invert_in_darkmode" align=middle width=331.3169772pt height=24.657534pt/>  
*model relience*: <img src="svgs/246d2949518f0e072a94de9eb5a00789.svg?invert_in_darkmode" align=middle width=277.7566704pt height=30.648288pt/>  
*a population-level model class reliance (MCR) range*: <img src="svgs/ffa3ac10030479bc13de83a17339abe9.svg?invert_in_darkmode" align=middle width=469.3592046pt height=27.9453933pt/>  
*the set of functions <img src="svgs/ff02df9d466f24ac28a9a6651c5544b7.svg?invert_in_darkmode" align=middle width=16.23606435pt height=22.4657235pt/> as an <img src="svgs/89f2e0d2d24bcf44db73aab8fc03252c.svg?invert_in_darkmode" align=middle width=7.8729552pt height=14.1552444pt/> -margin-expectation-cover*: if for any <img src="svgs/83f4b767e68d7919b1dfbb786c10d92a.svg?invert_in_darkmode" align=middle width=43.3560171pt height=22.8310566pt/> and any distribution <img src="svgs/e03ebcf17dcf2d3f40dc9021b39a1cde.svg?invert_in_darkmode" align=middle width=18.63245505pt height=22.4657235pt/> there exists <img src="svgs/999c928b27f4389347a61bf0a4e60c1e.svg?invert_in_darkmode" align=middle width=44.7575601pt height=22.4657235pt/> such that <img src="svgs/31e50e1924b0db53c3284ae64fb533f9.svg?invert_in_darkmode" align=middle width=176.8293186pt height=24.657534pt/>  
*the covering number <img src="svgs/76c49413e779522d8543e9095fa14ade.svg?invert_in_darkmode" align=middle width=57.3215874pt height=24.657534pt/>*: to be the size of the smallest <img src="svgs/89f2e0d2d24bcf44db73aab8fc03252c.svg?invert_in_darkmode" align=middle width=7.8729552pt height=14.1552444pt/> -margin-expectationcover for <img src="svgs/84cc939597f3eec200843a2fc8830732.svg?invert_in_darkmode" align=middle width=13.447467pt height=22.4657235pt/>  
> **_NOTE:_** If <img src="svgs/221ba46e23343b21cce257f1ad15c790.svg?invert_in_darkmode" align=middle width=73.51693965pt height=24.657534pt/> is low, then no well-performing model in <img src="svgs/84cc939597f3eec200843a2fc8830732.svg?invert_in_darkmode" align=middle width=13.447467pt height=22.4657235pt/> places high importance on <img src="svgs/ef94a731f1a699515d87dca8d785af8e.svg?invert_in_darkmode" align=middle width=25.55943555pt height=22.4657235pt/> and <img src="svgs/9efd0126287224eeab878b4d0b47b73c.svg?invert_in_darkmode" align=middle width=20.17129785pt height=22.4657235pt/> can be discarded at low cost regardless of future modeling decisions. Similarly for <img src="svgs/aec420cc680f5f9a2f41f3c8f47d7c4f.svg?invert_in_darkmode" align=middle width=73.69956825pt height=24.657534pt/>.  

### Возможны следующие вариации empirical MR:
<img src="svgs/e6e15a434fb7ff91803b90f86f96bac0.svg?invert_in_darkmode" align=middle width=141.37212045pt height=33.2053986pt/>  
<img src="svgs/908f1d9e35f3d768d48e741f931f6914.svg?invert_in_darkmode" align=middle width=478.23323955pt height=84.6738387pt/>  
<img src="svgs/2a0f84fec0172ae51bfe0b983b3edf64.svg?invert_in_darkmode" align=middle width=326.00016075pt height=27.9453933pt/>  
<img src="svgs/1061f8fca3bf89f6d24b9d1c8575eae5.svg?invert_in_darkmode" align=middle width=419.4236574pt height=27.9453933pt/>  

> **_NOTE:_** The estimators <img src="svgs/c0b1011abbb7fe0877418710b9a042f0.svg?invert_in_darkmode" align=middle width=136.4868285pt height=24.657534pt/> and <img src="svgs/f958022a491e85342a86708a4aea6f40.svg?invert_in_darkmode" align=middle width=70.22293575pt height=24.657534pt/> all belong to the well-studied class of U-statistics. Thus, under fairly minor conditions, *these estimators are unbiased, asymptotically normal, and have finite-sample probabilistic bounds*  
> Можно заменить сочетание всех пар объектов на группы и внутри них делать такое сочетание  

  

 ### Illustrative Toy Example with Simulated Data    
**Building an empirical <img src="svgs/1526dd9e7c063f7c98ae001d3ce6e202.svg?invert_in_darkmode" align=middle width=32.0662947pt height=24.657534pt/>:**  
Обучаются reference model/models. <img src="svgs/1526dd9e7c063f7c98ae001d3ce6e202.svg?invert_in_darkmode" align=middle width=32.0662947pt height=24.657534pt/> получается нормальным зашумлением координат одной выбранной reference model с дисперсиями:
- если дисперсия хотя бы одной координаты из соотвествующей колонки матрицы весов reference model/models == 0: все 1
- если дисперсия хотя бы одной координаты из соотвествующей колонки матрицы весов reference model/models != 0: соотвествующей дисперсией столбца  

Причём, если нам необходимо <img src="svgs/a1440dd74ba1e92365207181e8af0963.svg?invert_in_darkmode" align=middle width=33.66241065pt height=15.2968299pt/> (p-доля хороших моделей, n-количество всего моделей) в <img src="svgs/1526dd9e7c063f7c98ae001d3ce6e202.svg?invert_in_darkmode" align=middle width=32.0662947pt height=24.657534pt/>, то процедура поправки "плохой" модели осуществляется линейным приближением к reference model.  

### Simulations of Bootstrap Confidence Intervals  
**идея**:  
* 1 подход: возьмём ориг. датасет (20k записей), посчитаем на нем MCR, разделим весь датасет на 2 части training subset and analysis subset
  - на training subset: обучаем reference model
  - на analysis subset: сэмплируем выборку (500 times) и считаем <img src="svgs/6e4554423d2a03580ec0c895b2e96968.svg?invert_in_darkmode" align=middle width=62.73063885pt height=34.3683945pt/>, а после CI
- 2 подход (проще): cэмплируем выборку с ориг. датасета (500 times), делим его на 2 части:
  - на 1ой части обучаем модель
  - на 2ой оцениваем её <img src="svgs/6e4554423d2a03580ec0c895b2e96968.svg?invert_in_darkmode" align=middle width=62.73063885pt height=34.3683945pt/>, получаем CI  

**Итог**: 1 подход more robust to the misspecification of the models used to approximate Y and the model of Y itself  
    
### COMPAS score  
**Эксперимент**:  
Как влияют половые и подобные необъективные признаки на скор того, что подсудимый вновь разбоем.  

**Используемая модель:** регрессия + RBF ядро   
**Используемый датасет:** 500 train, 2873 test  
**Используемые признаки:**  

<img src="./images/compas.png" alt="drawing" width="400"/>  
  

Итог:  
- 1000 бустреэп сэмплирований -> 12 часов  

The bootstrap 95% CI for <img src="svgs/6e4554423d2a03580ec0c895b2e96968.svg?invert_in_darkmode" align=middle width=62.73063885pt height=34.3683945pt/> on “inadmissible variables” is **[1.00, 1.73]**  
For “admissible variables” the <img src="svgs/6e4554423d2a03580ec0c895b2e96968.svg?invert_in_darkmode" align=middle width=62.73063885pt height=34.3683945pt/> range with a 95% bootstrap CI is equal to **[1.62, 3.96]**  


<img src="./images/MCR_range.png" alt="drawing" width="400"/>  


Идея работы алгоритма:  
- Считаем необходимые матрицы и значения (расстояния, коэффициент регуляризации и т.д.)  
- Считам MRC




### We introduce three bounded loss assumptions:  
**Assumption 1** (*Bounded individual loss*) For a given model <img src="svgs/395cf4bf7dfc7f2e2da51b7a9a84d77c.svg?invert_in_darkmode" align=middle width=47.92223535pt height=22.8310566pt/> assume that <img src="svgs/d00be88766456732b157a8a360587de6.svg?invert_in_darkmode" align=middle width=25.57074135pt height=21.1872144pt/> <img src="svgs/0362b866c8b100f031c67717d70a820c.svg?invert_in_darkmode" align=middle width=165.13601775pt height=24.657534pt/> for any <img src="svgs/0b87ec9fbf75c80286c35c55797da12c.svg?invert_in_darkmode" align=middle width=193.18192575pt height=24.657534pt/>  
**Assumption 2** (*Bounded relative loss*) For a given model <img src="svgs/395cf4bf7dfc7f2e2da51b7a9a84d77c.svg?invert_in_darkmode" align=middle width=47.92223535pt height=22.8310566pt/> assume that <img src="svgs/c0ed02762c6b0ac304747f5bf91ffe75.svg?invert_in_darkmode" align=middle width=137.1650412pt height=24.657534pt/> <img src="svgs/63d9f5512ad2b947f85f05497d90b219.svg?invert_in_darkmode" align=middle width=187.79617395pt height=24.657534pt/> for any <img src="svgs/64b09dfb5f3a9ec25bf5d7c35d66cbb6.svg?invert_in_darkmode" align=middle width=102.8955939pt height=24.657534pt/>  
**Assumption 3** (*Bounded aggregate loss*) For a given model <img src="svgs/395cf4bf7dfc7f2e2da51b7a9a84d77c.svg?invert_in_darkmode" align=middle width=47.92223535pt height=22.8310566pt/> assume that <img src="svgs/bb133ec4145b6d217554879fbcb0294e.svg?invert_in_darkmode" align=middle width=43.83563745pt height=24.657534pt/> <img src="svgs/5e52e9438c61cb4a30a51bb74ec1b3ab.svg?invert_in_darkmode" align=middle width=350.7039987pt height=24.657534pt/>
> **_NOTE:_** some constants can be derived from others  

**Theorem 4** (*"Outer" MCR Bounds*) Given a constant <img src="svgs/434f6d79cf3e7605879c945f0bcce27a.svg?invert_in_darkmode" align=middle width=36.80923125pt height=21.1872144pt/>, let <img src="svgs/41cbe9d9685c9fe8cad27d3d509034af.svg?invert_in_darkmode" align=middle width=188.42796225pt height=24.657534pt/> and <img src="svgs/7c5035652a2f41d75f3e01dd3df70735.svg?invert_in_darkmode" align=middle width=185.4142224pt height=24.657534pt/> be prediction models that attain the highest and lowest model reliance among models in <img src="svgs/d3818fe9bff73052412280a7dd7dbd92.svg?invert_in_darkmode" align=middle width=33.38939175pt height=24.657534pt/>. If <img src="svgs/c1775ebbacd300a3b1ef7df2941ffdff.svg?invert_in_darkmode" align=middle width=27.5224092pt height=22.8310566pt/> and <img src="svgs/ff8ee454f3798d20024ce335ac8d0cd5.svg?invert_in_darkmode" align=middle width=27.7050576pt height=22.8310566pt/> satisfy Assumptions 1, 2, 3 , then
<p align="center"><img src="svgs/ebcebe13dcb38ae5f6e293c269c81a0f.svg?invert_in_darkmode" align=middle width=583.70409405pt height=103.69392825pt/></p>  

> **_NOTE:_**    
> - As <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.86687625pt height=14.1552444pt/> increases, <img src="svgs/237bd810324c5f192a7fbd8bd326009a.svg?invert_in_darkmode" align=middle width=25.8989379pt height=14.1552444pt/> approaches <img src="svgs/7ccca27b5ccc533a2dd72dc6fa28ed84.svg?invert_in_darkmode" align=middle width=6.6723921pt height=14.1552444pt/> and <img src="svgs/38834d7d013b79060f2d086645a17c25.svg?invert_in_darkmode" align=middle width=32.6512659pt height=22.4657235pt/> approaches zero
> - with high probability, the largest possible estimation error for <img src="svgs/90799cd0f3f2624e7fb9c1a986dba025.svg?invert_in_darkmode" align=middle width=52.95103275pt height=24.657534pt/> across all models in <img src="svgs/84cc939597f3eec200843a2fc8830732.svg?invert_in_darkmode" align=middle width=13.447467pt height=22.4657235pt/> is bounded by <img src="svgs/9cec0385363b3dee00c10f8cbbf0dd5f.svg?invert_in_darkmode" align=middle width=59.16670155pt height=24.657534pt/>, which can be made arbitrarily small by increasing <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.86687625pt height=14.1552444pt/> and decreasing <img src="svgs/89f2e0d2d24bcf44db73aab8fc03252c.svg?invert_in_darkmode" align=middle width=7.8729552pt height=14.1552444pt/>
> - The  existence  of  this  uniform  bound  implies  that  it  is  feasible  to  train  a  model  and  to evaluate its importance using the *same data*  

**Theorem 6** (*"Inner" MCR Bounds*) Given constants <img src="svgs/434f6d79cf3e7605879c945f0bcce27a.svg?invert_in_darkmode" align=middle width=36.80923125pt height=21.1872144pt/> and <img src="svgs/efa3244ceb1894cbd7e062da6bb9ebf0.svg?invert_in_darkmode" align=middle width=42.5760192pt height=21.1872144pt/> if Assumptions 1 , 2 and 3 hold for all <img src="svgs/395cf4bf7dfc7f2e2da51b7a9a84d77c.svg?invert_in_darkmode" align=middle width=47.92223535pt height=22.8310566pt/> and then
<p align="center"><img src="svgs/000e14dd720dd2e79a9b10e37abefcab.svg?invert_in_darkmode" align=middle width=333.1536153pt height=59.1786591pt/></p>  

### Calculating Empirical Estimates of MCR  
- computing <img src="svgs/17c03d0793e9a6170c1cf7020c514308.svg?invert_in_darkmode" align=middle width=73.6439253pt height=34.3683945pt/> however will require that we are able to minimize arbitrary linear combinations of <img src="svgs/2b98820d751e478539757714225e28c0.svg?invert_in_darkmode" align=middle width=57.4717209pt height=24.657534pt/> and <img src="svgs/daf800b1b76c20be39f70242e874487e.svg?invert_in_darkmode" align=middle width=71.7092244pt height=24.657534pt/>
- we present bound functions <img src="svgs/49e648991ffdfcbccd3b3fb2005ebe4b.svg?invert_in_darkmode" align=middle width=17.3288181pt height=22.8310566pt/> and <img src="svgs/706a470865330c718598b5dc21ed291e.svg?invert_in_darkmode" align=middle width=17.1461697pt height=22.8310566pt/> satisfying <img src="svgs/4ea8c210394ae0fbc804a501d7f4f746.svg?invert_in_darkmode" align=middle width=77.5422813pt height=24.657534pt/> <img src="svgs/6cf76cf8bcf20ee06c13946ba1c7bc3f.svg?invert_in_darkmode" align=middle width=139.30602675pt height=32.9680131pt/> simultaneously for all <img src="svgs/33a26985ed3f74c6c328d961bdb4040f.svg?invert_in_darkmode" align=middle width=306.673455pt height=24.657534pt/>
- almost all of the results shown in this section, and those in Section 6.2 . also hold if we replace <img src="svgs/d758b434ea5969b8eb5cf4ea54c47c3e.svg?invert_in_darkmode" align=middle width=43.85524275pt height=22.8310566pt/> with <img src="svgs/c96dc920c4b6fd8b88f1e068eff8fe4f.svg?invert_in_darkmode" align=middle width=42.3689541pt height=22.8310566pt/> throughout (see Eq 3.5), including in the definition of <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> and <img src="svgs/59e22f4aaefc5ca42e8f5a653371b456.svg?invert_in_darkmode" align=middle width=59.25291075pt height=31.5068952pt/>  

### Binary Search for Empirical MR Lower Bound  
**Condition 8** (*Criteria to continue search for <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> lower bound*) <img src="svgs/f78966e0991a9e03d6ee41a14a557d3f.svg?invert_in_darkmode" align=middle width=108.19909155pt height=31.5068952pt/> and <img src="svgs/98e16b9bf83e07cded0253295956f7a3.svg?invert_in_darkmode" align=middle width=128.55289755pt height=24.657534pt/>  
**Lemma 9** (*Lower bound*) If <img src="svgs/6fd73a861ca99d39a24a5cc49413f9e5.svg?invert_in_darkmode" align=middle width=41.38717935pt height=22.6483917pt/> satisfies <img src="svgs/546ffd587a2a4c3c2eeb3f7482ce8f31.svg?invert_in_darkmode" align=middle width=112.76531475pt height=31.5068952pt/> then <img src="svgs/83f4b767e68d7919b1dfbb786c10d92a.svg?invert_in_darkmode" align=middle width=43.3560171pt height=22.8310566pt/> satisfying <img src="svgs/53b35323e4d819003c03929275df6ae0.svg?invert_in_darkmode" align=middle width=110.565444pt height=24.657534pt/>
<p align="center"><img src="svgs/d35418f16d6b41defffadc5a3a374ff9.svg?invert_in_darkmode" align=middle width=184.41855465pt height=40.6157334pt/></p>  

for all <img src="svgs/83f4b767e68d7919b1dfbb786c10d92a.svg?invert_in_darkmode" align=middle width=43.3560171pt height=22.8310566pt/> satisfying <img src="svgs/53b35323e4d819003c03929275df6ae0.svg?invert_in_darkmode" align=middle width=110.565444pt height=24.657534pt/> It also follows that  

<p align="center"><img src="svgs/464324796aabc4c18339fe10ec82d0c4.svg?invert_in_darkmode" align=middle width=210.3427557pt height=20.59359555pt/></p>  

> **_NOTE:_**
> - Additionally, if <img src="svgs/9179443fc63bac0f71cbe3dd5070aa80.svg?invert_in_darkmode" align=middle width=61.3665261pt height=22.8310566pt/> and at least one of the inequalities in Condition 8 holds with equality, then top Eq holds with equality.
> - It remains to determine which value of <img src="svgs/11c596de17c342edeed29f489aa4b274.svg?invert_in_darkmode" align=middle width=9.42388095pt height=14.1552444pt/> should be used in top Eq. The following lemma implies that this value can be determined by a binary search, given a particular value of interest for <img src="svgs/8ea213afa93e648ee39ecc7d61fcf6c9.svg?invert_in_darkmode" align=middle width=25.6930476pt height=14.1552444pt/>  

**Lemma 10** (*Monotonicity for <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> lower bound binary search*) The following monotonicity results hold:  
1. <img src="svgs/2aa4bec95a231fe99a3160e90db869a7.svg?invert_in_darkmode" align=middle width=302.2298763pt height=31.5068952pt/>.  
2. <img src="svgs/3eb27bff2087f482a0a664ac9ddc2df0.svg?invert_in_darkmode" align=middle width=80.84728905pt height=24.657534pt/> is monotonically decreasing in <img src="svgs/11c596de17c342edeed29f489aa4b274.svg?invert_in_darkmode" align=middle width=9.42388095pt height=14.1552444pt/>.  
3. Given <img src="svgs/909d5560f3f72e4f5fb2ec4279bb8e9a.svg?invert_in_darkmode" align=middle width=31.17609165pt height=14.1552444pt/> the lower bound from <img src="svgs/875268283e5ccc3e94031b6ccd69aac4.svg?invert_in_darkmode" align=middle width=176.7938766pt height=38.7304599pt/> is monotonically decreasing in <img src="svgs/11c596de17c342edeed29f489aa4b274.svg?invert_in_darkmode" align=middle width=9.42388095pt height=14.1552444pt/> in the range where <img src="svgs/414071c344e4a2c47c41c3cf1bd5de47.svg?invert_in_darkmode" align=middle width=133.9410072pt height=24.657534pt/> and increasing otherwise.  
> **_NOTE:_** the value of <img src="svgs/11c596de17c342edeed29f489aa4b274.svg?invert_in_darkmode" align=middle width=9.42388095pt height=14.1552444pt/> resulting in the tightest lower bound from Eq 6.1 occurs when <img src="svgs/11c596de17c342edeed29f489aa4b274.svg?invert_in_darkmode" align=middle width=9.42388095pt height=14.1552444pt/> is as low as possible while still satisfying Condition 8  

**Proposition 11** (*Nonnegative weights for <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> lower bound binary search*) Assume that <img src="svgs/ddcb483302ed36a59286424aa5e0be17.svg?invert_in_darkmode" align=middle width=11.18724255pt height=22.4657235pt/> and <img src="svgs/84cc939597f3eec200843a2fc8830732.svg?invert_in_darkmode" align=middle width=13.447467pt height=22.4657235pt/> satisfy the following conditions.
1. (Predictions are sufficient for computing the loss) The loss <img src="svgs/6225945d71c19385d6974d89fef070c8.svg?invert_in_darkmode" align=middle width=126.41557005pt height=24.657534pt/> depends on the covariates <img src="svgs/71446c7042f29be8b69cdce4486843df.svg?invert_in_darkmode" align=middle width=62.0777355pt height=24.657534pt/> only via the prediction function <img src="svgs/6f60e7714a307b2095b24945797a63b4.svg?invert_in_darkmode" align=middle width=13.4703921pt height=22.8310566pt/> that is, <img src="svgs/e662b59c69bde0122d0f3c14733dad40.svg?invert_in_darkmode" align=middle width=167.5446201pt height=37.8085059pt/>  
<p align="center"><img src="svgs/ab5d39333dc0b4d38e3971f66a8c87b3.svg?invert_in_darkmode" align=middle width=436.60400025pt height=29.58934275pt/></p>  

2. (Irrelevant information does not improve predictions) For any distribution <img src="svgs/78ec2b7008296ce0561cf83393cb746d.svg?invert_in_darkmode" align=middle width=14.06623185pt height=22.4657235pt/> satisfying <img src="svgs/854174f37909fe339f9951113939cff2.svg?invert_in_darkmode" align=middle width=109.1157507pt height=24.657534pt/>, there exists a function <img src="svgs/59594839feb5edee4a012442140a945b.svg?invert_in_darkmode" align=middle width=19.1501013pt height=22.8310566pt/> satisfying  

<p align="center"><img src="svgs/b885a87b87979c28f2c3e022e8f30e4c.svg?invert_in_darkmode" align=middle width=391.5711261pt height=25.2967704pt/></p>
<p align="center"><img src="svgs/bf4d8b08bd0afa9be3eec875a9c8f7f4.svg?invert_in_darkmode" align=middle width=466.0547826pt height=29.58934275pt/></p>  

Let <img src="svgs/7eaedc1b9d7a4b11f78f1c63edf34f3a.svg?invert_in_darkmode" align=middle width=39.56070195pt height=21.1872144pt/>. Under the above assumptions, it follows that either (i) there exists a function <img src="svgs/48895a24eb707d255dbd416b85f7b261.svg?invert_in_darkmode" align=middle width=28.57128285pt height=22.8310566pt/> minimizing <img src="svgs/c3b4c414757b2e9c91044161729689c9.svg?invert_in_darkmode" align=middle width=30.20181615pt height=31.5068952pt/> that does not satisfy Condition <img src="svgs/52c9334aa06111e5beb9aa65bed392b2.svg?invert_in_darkmode" align=middle width=12.7854342pt height=21.1872144pt/> or <img src="svgs/52a305dd460bf66af073157eec768642.svg?invert_in_darkmode" align=middle width=24.1118856pt height=24.657534pt/> <img src="svgs/89d08930b8881b020ab47848fc693385.svg?invert_in_darkmode" align=middle width=127.49271645pt height=24.657534pt/> and <img src="svgs/028ee8af7f6f58b11fb9bf9c04f57b5f.svg?invert_in_darkmode" align=middle width=92.6179386pt height=32.9680131pt/> 1 for any function <img src="svgs/48895a24eb707d255dbd416b85f7b261.svg?invert_in_darkmode" align=middle width=28.57128285pt height=22.8310566pt/> minimizing <img src="svgs/c3b4c414757b2e9c91044161729689c9.svg?invert_in_darkmode" align=middle width=30.20181615pt height=31.5068952pt/>.  

> **_NOTE:_** tractability of our approach, as minimizing <img src="svgs/e10250bcb78a35a11ba7fe1bc60bafe0.svg?invert_in_darkmode" align=middle width=31.2620154pt height=31.5068952pt/> for <img src="svgs/9176cc553740c30fb1eb46ab627ba3a2.svg?invert_in_darkmode" align=middle width=39.56070195pt height=21.1872144pt/> is equivalent to minimizing reweighted empirical loss over an expanded sample of size <img src="svgs/671384664426f82966adbde77d983140.svg?invert_in_darkmode" align=middle width=16.4194239pt height=26.7617526pt/>:  
> 
> <p align="center"><img src="svgs/282b3c938429af6b9f2a729bf3c95e79.svg?invert_in_darkmode" align=middle width=639.23738505pt height=47.1348339pt/></p>  
> 
> <p align="center"><img src="svgs/a742f1228e4becff1028d59ebd959863.svg?invert_in_darkmode" align=middle width=294.95653275pt height=38.8349148pt/></p>  

### Binary Search for Empirical MR Upper Bound  
<img src="svgs/0e966d02453e06d54e13be3a54b050da.svg?invert_in_darkmode" align=middle width=244.25013525pt height=31.5068952pt/> and <img src="svgs/371d351a36ba28cf9b36f9e86d04fcaa.svg?invert_in_darkmode" align=middle width=172.5885018pt height=32.3286942pt/>  
Given an observed sample, we define the following condition for a pair of values <img src="svgs/b4635f8bf548f2dce0e282964bd75129.svg?invert_in_darkmode" align=middle width=75.2081418pt height=24.657534pt/> <img src="svgs/bb9800eb58b40dd0ed5704c9b5310a59.svg?invert_in_darkmode" align=middle width=83.6986887pt height=22.6483917pt/> and argmin function <img src="svgs/527755e80d779eb423ee66bb3decc25c.svg?invert_in_darkmode" align=middle width=39.40301805pt height=22.8310566pt/>  
**Condition 12** (*Criteria to continue search for <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> upper bound*) <img src="svgs/f29ef9fa70fdd5d92c6556934e00cb2a.svg?invert_in_darkmode" align=middle width=107.8338327pt height=31.5068952pt/> and <img src="svgs/fe3321fe2a8418bcf480637a0eaf2ef4.svg?invert_in_darkmode" align=middle width=128.37026895pt height=24.657534pt/>  
> **_NOTE:_** We can now develop a procedure to upper bound <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/>, as shown in the next lemma.  

**Lemma 13** (*Upper bound for <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/>*) If <img src="svgs/6fd73a861ca99d39a24a5cc49413f9e5.svg?invert_in_darkmode" align=middle width=41.38717935pt height=22.6483917pt/> satisfies <img src="svgs/f89d27203caad87817dd570f6ea88dae.svg?invert_in_darkmode" align=middle width=39.56070195pt height=21.1872144pt/> and <img src="svgs/55cf07331b803837adc8725504ab8a84.svg?invert_in_darkmode" align=middle width=112.40005755pt height=31.5068952pt/> then  
<p align="center"><img src="svgs/36fd86945d154a9a7b28f29b293e07f1.svg?invert_in_darkmode" align=middle width=240.2954433pt height=49.3155696pt/></p>  

for all <img src="svgs/83f4b767e68d7919b1dfbb786c10d92a.svg?invert_in_darkmode" align=middle width=43.3560171pt height=22.8310566pt/> satisfying <img src="svgs/53b35323e4d819003c03929275df6ae0.svg?invert_in_darkmode" align=middle width=110.565444pt height=24.657534pt/> It also follows that  

<p align="center"><img src="svgs/6d36695c0c6fd02402a3340bc2106b52.svg?invert_in_darkmode" align=middle width=228.90432675pt height=22.2375318pt/></p>  

Additionally, if <img src="svgs/692bc89af45dda02a0ba212a04b4d344.svg?invert_in_darkmode" align=middle width=61.18387935pt height=22.8310566pt/> and at least one of the inequalities in Condition 12 holds with equality, then Eq 6.4 holds with equality.  
**Lemma 14** (*Monotonicity for <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> upper bound binary search*) The following monotonicity results hold:  
1. <img src="svgs/6c4caf37d69f25f06527615acc8a3af9.svg?invert_in_darkmode" align=middle width=77.69699685pt height=31.5068952pt/> is monotonically increasing in <img src="svgs/11c596de17c342edeed29f489aa4b274.svg?invert_in_darkmode" align=middle width=9.42388095pt height=14.1552444pt/>.  
2. <img src="svgs/2ead0012ddc2757a440b834d302d12db.svg?invert_in_darkmode" align=middle width=80.6646588pt height=24.657534pt/> is monotonically decreasing in <img src="svgs/11c596de17c342edeed29f489aa4b274.svg?invert_in_darkmode" align=middle width=9.42388095pt height=14.1552444pt/> for <img src="svgs/941aeee7a2af301370e4bd2b845a266c.svg?invert_in_darkmode" align=middle width=44.12692515pt height=21.1872144pt/> and Condition 12 holds for <img src="svgs/7eaedc1b9d7a4b11f78f1c63edf34f3a.svg?invert_in_darkmode" align=middle width=39.56070195pt height=21.1872144pt/> and <img src="svgs/16aef5d14baf6d317c969f434771c61c.svg?invert_in_darkmode" align=middle width=164.0929158pt height=24.657534pt/>  
3. Given <img src="svgs/909d5560f3f72e4f5fb2ec4279bb8e9a.svg?invert_in_darkmode" align=middle width=31.17609165pt height=14.1552444pt/> the upper boundary <img src="svgs/bedf1387f1d8ea60dc20aba0df3b351a.svg?invert_in_darkmode" align=middle width=147.0394629pt height=38.7304599pt/> is monotonically increasing in in the range where <img src="svgs/fe3321fe2a8418bcf480637a0eaf2ef4.svg?invert_in_darkmode" align=middle width=128.37026895pt height=24.657534pt/> and <img src="svgs/322e3f3c4226f78afced05d4a1469ccb.svg?invert_in_darkmode" align=middle width=44.12692515pt height=21.1872144pt/> and decreasing in the range where <img src="svgs/9797c7cad099a2ff0815db17daca3708.svg?invert_in_darkmode" align=middle width=128.37026895pt height=24.657534pt/> and <img src="svgs/3f570e5f36cf91c64cee6df1059362c0.svg?invert_in_darkmode" align=middle width=39.56070195pt height=21.1872144pt/>  
> **_NOTE:_** Together, the results from Lemma 14 imply that we can use a binary search across <img src="svgs/6fd73a861ca99d39a24a5cc49413f9e5.svg?invert_in_darkmode" align=middle width=41.38717935pt height=22.6483917pt/> to tighten the boundary on <img src="svgs/3ad1b4f7e9adbad3bbb7704a6ee71e46.svg?invert_in_darkmode" align=middle width=30.3481893pt height=32.9680131pt/> from Lemma <img src="svgs/47d3617445b54b7c6cb9fb47355578d6.svg?invert_in_darkmode" align=middle width=21.00464355pt height=21.1872144pt/>  
> Как подбирается gamma: сначала ищем интервал вида [-a, a] на границах которого не будет выполнятся необходимое условие, потом индуктивно делим отрезок попалам и "сжимаем" границы  

### Convex Models
- **идея**: пусть функции параметризуются некоторым вектором переменных, разобьём это пространство на симплексы, на них h совпадает в вершинах с некоторой гиперплоскостью, заменяем h её, получаем нижнию оценку, так для всех подвыборок из пространства и индуктивно повторяем процедуру  

### MR & MCR for Linear Models, Additive Models  
Throughout this section, we assume that <img src="svgs/525d2b68f96a33db3c9d6903b457a4cc.svg?invert_in_darkmode" align=middle width=54.6987507pt height=22.6483917pt/> for <img src="svgs/6f1cc81a5cf62297d38807247fd66323.svg?invert_in_darkmode" align=middle width=54.8001564pt height=26.1773094pt/> that <img src="svgs/df0525dd40363d25686f8908e12835aa.svg?invert_in_darkmode" align=middle width=58.0684467pt height=26.7617526pt/> and that <img src="svgs/ddcb483302ed36a59286424aa5e0be17.svg?invert_in_darkmode" align=middle width=11.18724255pt height=22.4657235pt/> is the squared error loss function <img src="svgs/923d1ef8595c8ce2325a43ac73658a77.svg?invert_in_darkmode" align=middle width=245.722785pt height=37.8085059pt/>.  
**Proposition 15** (*Interpreting <img src="svgs/a31992ee5753d0c3a9729a5e32257c37.svg?invert_in_darkmode" align=middle width=34.9144092pt height=22.4657235pt/> and computing empirical MR for linear models*) For any prediction model <img src="svgs/6f60e7714a307b2095b24945797a63b4.svg?invert_in_darkmode" align=middle width=13.4703921pt height=22.8310566pt/> let <img src="svgs/f76c52abf47fa74538b1bc013b6119a6.svg?invert_in_darkmode" align=middle width=205.83066405pt height=24.657534pt/> and <img src="svgs/daf800b1b76c20be39f70242e874487e.svg?invert_in_darkmode" align=middle width=71.7092244pt height=24.657534pt/> be defined based on the squared error loss <img src="svgs/6c90a58890f7ff2a437bb89bf09b2e32.svg?invert_in_darkmode" align=middle width=253.25707275pt height=31.3608075pt/> for <img src="svgs/e4fe82e8dcbe27f21551c668ef1b30ad.svg?invert_in_darkmode" align=middle width=115.23138495pt height=22.6483917pt/> and <img src="svgs/de660c07e8f4febe3f8c5cb400a4dd55.svg?invert_in_darkmode" align=middle width=61.1029419pt height=22.6483917pt/>  

where <img src="svgs/f467c5bf149d12e7bb28489e6164ebf9.svg?invert_in_darkmode" align=middle width=14.82311325pt height=14.1552444pt/> and <img src="svgs/d882a6b325cb3b31b5866882721cf8de.svg?invert_in_darkmode" align=middle width=14.82311325pt height=14.1552444pt/> are positive integers. Let <img src="svgs/68c7990caff502dcc6b82b5c20cd1423.svg?invert_in_darkmode" align=middle width=85.5193251pt height=24.657534pt/> and <img src="svgs/3410e8b77a8601e7618820f2558e3a50.svg?invert_in_darkmode" align=middle width=16.09314465pt height=22.8310566pt/> satisfy <img src="svgs/9207b882cde6a3c8348e7b908a96ada7.svg?invert_in_darkmode" align=middle width=130.9615329pt height=22.8310566pt/> and <img src="svgs/c4d33f1e2243b813f0f048aa9e6af3ae.svg?invert_in_darkmode" align=middle width=198.64429365pt height=24.7161288pt/> Then  

<p align="center"><img src="svgs/82c1692dfe28e39c7cf6212f043b2a51.svg?invert_in_darkmode" align=middle width=457.9519164pt height=37.6933392pt/></p>  

and, for finite samples,  

<p align="center"><img src="svgs/be714082c545f51d061d0247c89542d8.svg?invert_in_darkmode" align=middle width=555.1804962pt height=41.7812637pt/></p>  

where <img src="svgs/4782371ddecbf6bb53080a79ca3d442c.svg?invert_in_darkmode" align=middle width=191.34402705pt height=27.7756545pt/> is the <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.86687625pt height=14.1552444pt/> -length vector of ones, and <img src="svgs/54a937784e79b8adddb37d9b2263e27c.svg?invert_in_darkmode" align=middle width=15.29494725pt height=22.5570873pt/> is the <img src="svgs/3add1221abfa79cb14021bc2dacd5725.svg?invert_in_darkmode" align=middle width=39.8249445pt height=19.1781018pt/>
identity matrix.  

> **_NOTE:_** сложность вычисления растёт линейно (можно расписать)  

**Remark 16** (*Tractability of empirical MCR for linear model classes*) For any <img src="svgs/0b3224020b22b8293953c6417cc644b2.svg?invert_in_darkmode" align=middle width=64.70986995pt height=22.8310566pt/> and any fixed coefficients <img src="svgs/0409c0ab67c9122543d5824c2a06cdb9.svg?invert_in_darkmode" align=middle width=126.8860329pt height=22.8310566pt/> the linear combination  
<p align="center"><img src="svgs/796ba6fd697e42f02cac98a866f078f6.svg?invert_in_darkmode" align=middle width=251.9973423pt height=17.0319402pt/></p>  

is proportional in <img src="svgs/8217ed3c32a785f0b5aad4055f432ad8.svg?invert_in_darkmode" align=middle width=10.165551pt height=22.8310566pt/> to the quadratic function <img src="svgs/a7fe98af31abe4ec0f2bcf89897e9162.svg?invert_in_darkmode" align=middle width=109.56035475pt height=24.7161288pt/> where  

<p align="center"><img src="svgs/3762cef6fe75f3c2cf65ed00324eb916.svg?invert_in_darkmode" align=middle width=673.38272595pt height=51.6443763pt/></p>  

<p align="center"><img src="svgs/5c5511dec0dc056c46fd62faf70175bf.svg?invert_in_darkmode" align=middle width=171.02633955pt height=27.7756545pt/></p>  

> **_NOTE:_**  
> 
> - Thus, minimizing <img src="svgs/a612f2a8a19baad1e773c87b9e75576f.svg?invert_in_darkmode" align=middle width=251.99734395pt height=24.657534pt/> is equivalent to an unconstrained (possibly non-convex) quadratic program.
> - <img src="svgs/b37e1c7739e7d50605fe8a0fd4e1eb9a.svg?invert_in_darkmode" align=middle width=409.1601789pt height=24.7161288pt/>
> - The resulting optimization problem is a (possibly non-convex) quadratic program withone  quadratic  constraint  

**Lemma 17** (*Loss upper bound for linear models*) If <img src="svgs/100b67cef52c22ef5bf59da78336d102.svg?invert_in_darkmode" align=middle width=33.83375655pt height=22.5570873pt/> is positive definite, <img src="svgs/91aac9730317276af725abd8cef04ca9.svg?invert_in_darkmode" align=middle width=13.1963865pt height=22.4657235pt/> is bounded within a known range, and there exists a known constant <img src="svgs/6940e84f35678b3aff684d787e5ef463.svg?invert_in_darkmode" align=middle width=18.5237019pt height=14.1552444pt/> such that <img src="svgs/6c698ef3df94efbf508a6c4c59849479.svg?invert_in_darkmode" align=middle width=99.43675995pt height=28.8949551pt/> for all <img src="svgs/061d1720a5d427021651f245afb95c39.svg?invert_in_darkmode" align=middle width=107.86838535pt height=24.657534pt/> then Assumption 1 holds for the model class <img src="svgs/e6a5639c07265f2bb3ea3c3691189a1e.svg?invert_in_darkmode" align=middle width=57.9626322pt height=22.4657235pt/> the squared error loss function, and the constant  
<p align="center"><img src="svgs/77a0819beceab655eb7fcd723d52d3b3.svg?invert_in_darkmode" align=middle width=439.88553345pt height=49.3155696pt/></p>  

### Regression Models in a Reproducing Kernel Hilbert Space  
<p align="center"><img src="svgs/f152422d3cf308e8074964937d68cf28.svg?invert_in_darkmode" align=middle width=523.8295458pt height=49.3155696pt/></p>  

Above, the norm <img src="svgs/22cfa2de4616bd3325e7675d8a3080ca.svg?invert_in_darkmode" align=middle width=41.12024895pt height=24.657534pt/> is defined as  

<p align="center"><img src="svgs/f439d9dda12151905456d8654480e0cb.svg?invert_in_darkmode" align=middle width=349.7462727pt height=50.04352485pt/></p>  

### Calculating MCR  
For any two constants <img src="svgs/0409c0ab67c9122543d5824c2a06cdb9.svg?invert_in_darkmode" align=middle width=126.8860329pt height=22.8310566pt/> we can show that minimizing the linear combination <img src="svgs/b068a4fd3be4b75e042fc286083c2f05.svg?invert_in_darkmode" align=middle width=252.99885435pt height=24.657534pt/> over <img src="svgs/3d8eb5d1919b6d0e2d2306f0219f4006.svg?invert_in_darkmode" align=middle width=39.5225523pt height=22.4657235pt/> is equivalent to the minimization problem  
<p align="center"><img src="svgs/d8a7c6054e9bfc0b36e3ab7f0ead8485.svg?invert_in_darkmode" align=middle width=521.3086296pt height=41.87855595pt/></p>  

### Upper Bounding the Loss  
**Lemma 18** (*Loss upper bound for regression in a RKHS*) Assume that <img src="svgs/91aac9730317276af725abd8cef04ca9.svg?invert_in_darkmode" align=middle width=13.1963865pt height=22.4657235pt/> is bounded within a known range, and there exists a known constant <img src="svgs/a34893f936d403e4c7359da7d0f1c543.svg?invert_in_darkmode" align=middle width=18.7519233pt height=14.1552444pt/> such that <img src="svgs/00f6d1fd9af3305908e15bd35bbd1b4c.svg?invert_in_darkmode" align=middle width=139.2236901pt height=28.8949551pt/> for all <img src="svgs/061d1720a5d427021651f245afb95c39.svg?invert_in_darkmode" align=middle width=107.86838535pt height=24.657534pt/> where <img src="svgs/a9ef3251a5017c4616da0900c628b14d.svg?invert_in_darkmode" align=middle width=89.13141435pt height=27.6567522pt/> is the function satisfying <img src="svgs/969ba4e57956c0e0fd6463b65af971d8.svg?invert_in_darkmode" align=middle width=151.6835991pt height=27.9453933pt/> Under these  
conditions, Assumption 1 holds for the model class <img src="svgs/e685076412a653c02d66142cc1e0ed5c.svg?invert_in_darkmode" align=middle width=44.2942599pt height=22.4657235pt/> the squared error loss function,  
and the constant  
<p align="center"><img src="svgs/ae60df13655e43238ca8f03632db4e1b.svg?invert_in_darkmode" align=middle width=502.7319198pt height=49.3155696pt/></p>  

### Model Reliance and Causal Effects  
- Let <img src="svgs/43eaeea5accf88ac13066ee1ab6b9987.svg?invert_in_darkmode" align=middle width=214.69168875pt height=24.657534pt/>, <img src="svgs/de918639f648bb439f957cb95a016797.svg?invert_in_darkmode" align=middle width=228.33899715pt height=24.657534pt/>    
  Proposition 19 (Causal interpretations of MR) For any prediction model <img src="svgs/6f60e7714a307b2095b24945797a63b4.svg?invert_in_darkmode" align=middle width=13.4703921pt height=22.8310566pt/> let <img src="svgs/aab3629f27d1dd612675e41cb2c01b89.svg?invert_in_darkmode" align=middle width=57.4717242pt height=24.657534pt/> and <img src="svgs/b8e7e1e4676cdf4d56668f68fc33fa71.svg?invert_in_darkmode" align=middle width=71.7092277pt height=24.657534pt/> be defined based on the squared error loss <img src="svgs/6c266fd1b1212004666a94b52921d4e4.svg?invert_in_darkmode" align=middle width=206.7998988pt height=26.7617526pt/> If <img src="svgs/3675b601c02d116f8e8a850460297267.svg?invert_in_darkmode" align=middle width=114.35700045pt height=24.657534pt/> (conditional ignorability) and <img src="svgs/b30ff4acda4ba75b66d09e7c2e4d942f.svg?invert_in_darkmode" align=middle width=180.7854444pt height=24.657534pt/> for all values of c (positivity), then <img src="svgs/6e12165d0a4eb03d896e215ef3078a51.svg?invert_in_darkmode" align=middle width=61.2957213pt height=24.657534pt/> is equal to  
  <p align="center"><img src="svgs/34ff3e72777eddf6bcd7caf78cfc7c37.svg?invert_in_darkmode" align=middle width=562.7067171pt height=46.7371938pt/></p>  
  where <img src="svgs/98d9c567d581efe4fc60e397e4297dc1.svg?invert_in_darkmode" align=middle width=50.2912245pt height=24.657534pt/> is the marginal variance of the treatment assignment.

### Conditional Importance:  Adjusting for Dependence Between <img src="svgs/0f40e6e5e62540b00031284316b128ba.svg?invert_in_darkmode" align=middle width=23.12787675pt height=22.4657235pt/> and <img src="svgs/13dd556b083841259b42bf0b978ccb52.svg?invert_in_darkmode" align=middle width=23.12787675pt height=22.4657235pt/>  
<img src="svgs/9418b1e57f8320d5f6ed575832b1cc28.svg?invert_in_darkmode" align=middle width=576.29246895pt height=37.8085059pt/>  
<img src="svgs/9213402d780ee7e4bfb14d8058894981.svg?invert_in_darkmode" align=middle width=141.6255126pt height=33.2053986pt/>  

> **_NOTE:_** This means that CMR will not be influenced by impossible combinations of <img src="svgs/7e0dded6496a3ed3f3c0db74604087ac.svg?invert_in_darkmode" align=middle width=15.94753545pt height=14.1552444pt/> and <img src="svgs/345508ce4e933b712fe803f442f74d63.svg?invert_in_darkmode" align=middle width=15.94753545pt height=14.1552444pt/>, while MR may be influenced by them

### Estimation of CMR by Weighting, Matching, or Imputation  
<img src="svgs/66a53a6f2a16676d89ee543e74e396f2.svg?invert_in_darkmode" align=middle width=561.7138626pt height=27.9453933pt/>  

> **_NOTE:_**  
> - <img src="svgs/e6ab88bcb3a29eee07efa0d274334010.svg?invert_in_darkmode" align=middle width=72.7000758pt height=24.657534pt/> is unbiased for <img src="svgs/1082da229d899ac1ef48990348a4e30a.svg?invert_in_darkmode" align=middle width=62.42608185pt height=24.657534pt/> <img src="svgs/d4e48e19eb2277145137d98de3d2cfd1.svg?invert_in_darkmode" align=middle width=544.4781012pt height=43.0684122pt/>
> - if the inverse probability weight <img src="svgs/83852ec7f50accfa6271cbd81caa41fb.svg?invert_in_darkmode" align=middle width=125.43850605pt height=34.6486305pt/> is known, then <img src="svgs/f54935d1e5f95b43279b0ee9e6b03d71.svg?invert_in_darkmode" align=middle width=70.74802185pt height=24.657534pt/> is unbiased for <img src="svgs/1082da229d899ac1ef48990348a4e30a.svg?invert_in_darkmode" align=middle width=62.42608185pt height=24.657534pt/> (see Appendix A.7).
> - However, when the covariate space is continuous or high dimensional, we typically cannotestimate  CMR  nonparametrically.
> - When the covariate space is continuous or high dimensional we define <img src="svgs/3118e416dde61fa174ee23f070c8b8e3.svg?invert_in_darkmode" align=middle width=16.45747125pt height=14.1552444pt/> to be the conditional expectation function <img src="svgs/bdbfe60ca01dc1f0e00b7569dfc4f8f6.svg?invert_in_darkmode" align=middle width=199.6532934pt height=24.657534pt/> and assume that the random residual <img src="svgs/cbfb1b2a33b28eab8a3e59464768e810.svg?invert_in_darkmode" align=middle width=14.90868885pt height=22.4657235pt/> <img src="svgs/6dfccda44725dbea4f6d308182ce766c.svg?invert_in_darkmode" align=middle width=53.79768405pt height=24.657534pt/> is independent of <img src="svgs/3930949a56ee806512c58ed9903c1033.svg?invert_in_darkmode" align=middle width=20.17129785pt height=22.4657235pt/>. Under this assumption, it can be shown that  
> 
>   <p align="center"><img src="svgs/0fa25d3952dac8f38e7a89cc1211f583.svg?invert_in_darkmode" align=middle width=513.97555275pt height=29.58934275pt/></p>  

