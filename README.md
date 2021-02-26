# Дневник
_Формулы корректно отображаются на [этом сайте](https://upmath.me/)_

#### 27/09/2020 - повторил 2 первые главы [курса](https://stepik.org/course/54098/promo)
##### Глава 1
- Генерация текста:
   - через поиск похожих
   - по шаблону
   - с помощью нейросетей
- При векторном разряженном представлении документа теряется зависимость слов
- Предиктивные модели (BERT, Transformer и т.п.) не требуют размеченной выборки
- Сходство текстов можно определить как долю совпадающих путей, проходимых в графовых представлениях текстов   
- В классификации с текстами:
   - большой длины линейные модели дают основное качество
   - короткими, в зависимости от объема gold_labels:
     - малый объем - ядерные методы
     - совсем нет - системы правил
- В эксплоративном анализе применяются методы тематического регулирования: LDA, ARTM

##### Глава 2
- В подходе с TF-IDF не используется информация о метках документов => теряем часть информация, если она есть


#### 04/10/2020 - прочитал обзорную [статью](https://arxiv.org/pdf/1802.02871.pdf) про online learning
- В большенстве случаев рассматривается бинарная классификация и задача оптимизации  
  <p align="center"><img src="svgs/6ed423a922093bd5a8b5788f7f9d4fea.svg?invert_in_darkmode" align=middle width=234.8915679pt height=47.60747145pt/></p>  
  
  причём <img src="svgs/b3520dc7da5f9731724eb6e1768a45a7.svg?invert_in_darkmode" align=middle width=29.96320305pt height=21.6830097pt/> ищется для <img src="svgs/31fae8b8b78ebe01cbfbe2fe53832624.svg?invert_in_darkmode" align=middle width=12.21084645pt height=14.1552444pt/>, не зависящего от <img src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg?invert_in_darkmode" align=middle width=5.93609775pt height=20.2218027pt/>
- В Contextual Bandits минимизируется
  <p align="center"><img src="svgs/fb344e07a8a3044adcd4414dfb922ec9.svg?invert_in_darkmode" align=middle width=196.4236923pt height=47.60747145pt/></p>  
  
  где <img src="svgs/62c1f9e0fcba81bb95c93229199b6d48.svg?invert_in_darkmode" align=middle width=157.2094953pt height=24.657534pt/>, &nbsp; <img src="svgs/b96bec0403cfbeb85791aa7870b5a828.svg?invert_in_darkmode" align=middle width=57.7053378pt height=28.0910388pt/> - действие выбранное на t шаге
- Есть ссылка на потенциально интересную [статью](https://arxiv.org/pdf/1711.03705.pdf) про online deep learning


#### 12/10/2020 - прочитал [статью](https://papers.nips.cc/paper/4928-understanding-variable-importances-in-forests-of-randomized-trees.pdf) про variable importances in forests of randomized trees
- Ограничения в работе: неповторение в детях признаков родителей, выборка полностью описывающая распределение <img src="svgs/d7b23b36b8b2a061477fd7c9c649476f.svg?invert_in_darkmode" align=middle width=116.6452452pt height=24.657534pt/>, бесконечное кол-во полных рандомизированных деревьев
- Если выбираем на этапе деления рандомно один признак и глубина дерева >= кол-во рел. признаков -> важность признака == 0 <-> он нерелевантный
- Если выбираем > 1 признака и из них максимизирующий уменьшение энтропии -> появляется маскирующий эффект: некоторые релевантные признаки могут иметь сильно меньшую важность по сравнению с похожими рел. признаками
    - Добавление нерелевантных может сказаться на важности релевантных

#### 26/10/2020 - прочитал [пример](https://scikit-learn.org/stable/modules/permutation_importance.html), [пример](https://scikit-learn.org/stable/auto_examples/inspection/plot_permutation_importance.html#sphx-glr-auto-examples-inspection-plot-permutation-importance-py), [пример](https://scikit-learn.org/stable/auto_examples/inspection/plot_permutation_importance_multicollinear.html#sphx-glr-auto-examples-inspection-plot-permutation-importance-multicollinear-py) с sklearn про permutation feature importance и из [источника](https://machinelearningmastery.com/calculate-feature-importance-with-python/#:~:text=Feature%20importance%20refers%20to%20a,feature%20when%20making%20a%20prediction.), [источника](https://towardsdatascience.com/interpretable-machine-learning-1dec0f2f3e6b) про важность признаков
- impurity-based feature importance for trees are strongly biased and favor high cardinality features
- если в датасете есть скоррелированные признаки, то в подходе permutation importance таким признакам будет даваться малый вес
    - решение проблемы: иерархическая кластеризация по корреляциям рангового порядка Спирмена, выбор порога и сохранение одного объекта из каждого кластера
- в случае random forest если сложность модели велика по сравнению с данными, алгоритм может переобучиться и даже рандомные признаки будут играть большую роль
- drop column метод вычислительно трудозатратный, но точный
- в методе lime особую роль играет подбор возбуждений экземпляра выборки

**25/12/2020** - читаю статью «Fisher A, Rudin C, Dominici F (2018) All models are wrong but many are useful». В последнее время разбирал и конспектировал статьи.

**23/01/2021** - начал ввести описание библиотек в [таблице](libs.md). Добавил rfpimp, eli5, cxplain.

**08/02/2021** - доделываю [слайды](https://github.com/MikhailKuz/3_course_diary/blob/master/seminar/speech_1/speech_1.pdf) на семинар.

**22/02/2021** - законспектировал [две статьи](https://github.com/MikhailKuz/3_course_diary/commit/5719baf75c4a5bb0c8c95010a2ef770e1ec1e755).

