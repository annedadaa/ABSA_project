# Aspect Based Sentiment Analysis
### Файлы:
1) aspect_extraction.ipynb -- выделение аспектов
2) sentiment_extraction.ipynb -- оценка тональности аспектов
3) categories_sentiment_with_NN.ipynb -- тональности категорий
4) category_sentiment_with_sklearn.ipynb -- улучшенная версия тональности категорий (ее и использовали в итоге)
5) use_all_models.ipynb -- файл, чтобы запустить модели обученные на всем корпусе и получить полную разметку для всех трех пунктов
6) dev_pred_aspects_ours.txt -- разметка эксплицитных аспектов
7) dev_pred_cats_ours.txt --разметка тональности по категориям 
8) task1.sav -- модель для первого задания (обучена на всем корпусе)
9) task2.sav -- модель для второго задания (обучена на всем корпусе)
10) task3.sav -- модель для третьего задания (обучна на всем корпусе)
11) tfidf.pickle -- тфидф, нужен, чтобы запустить модель из второго задания
12) encoder.pickle -- энкодер для y. нужен, чтобы запустить модель из третьего задания

Участники проекта: Анна Смирнова, Кирилл Конча, Михаил Сонькин, Антон Арцишевский

## Корпус

Мы не меняли и не дополняли корпус. Использовали для обучения и проверки скоров моделей train и dev-файлы из папки [проекта](https://github.com/named-entity/hse-nlp/tree/master/4th_year/Project).

## Выделение эксплицитных аспектов

Мы использовали метод sequence labelling, разметили тексты в формате BIO, для того, чтобы можно было учитывать контекст. Разаметили на основе готовых данных n-граммы упоминания аспектов, а затем обучили модель CRF. Так как данных в принципе было немного (284 текста), а распределение по аспектам очень не равномерным, то модель обучилась плохо и особенно плохо она распознает малочисленные аспекты (например, I-Price). 

В целом по сравнению с размеченными данными, CRF показала хорошие результаты, поэтому оставили ее. Кроме того, наша модель выделила больше аспектов, чем есть на самом деле, и эти аспекты вполне осмысленные и могут считаться верными.

Распределение аспектов:

![alt text](https://sun9-67.userapi.com/impg/zZGVO7XSKFiplvAgxGTpsjlCh7WYMi63vyP7HA/D0fWPsuQkwM.jpg?size=704x483&quality=96&sign=20712793d33f84196cd343484532de19&type=album![image](https://user-images.githubusercontent.com/42929201/147552556-22256d93-7404-4d87-a6a0-8fec2aaa5d7b.png))

| Aspect        |     Number      |   
|:------------- |:---------------:| 
| B-Food        |  1428           |   
| I-Food        |  908            |   
| B-Whole       |  686            |   
| I-Whole       |  610            |    
| B-Service     |  510           |     
| I-Service     |  209            |     
| B-Price       |  160            |     
| I-Price       |  139             |     
| B-Interior    |  100            |     
| I-Interior    |  19            |     

Результаты:

| Matrics       | Baseline        | CRF          |
|:------------- |:---------------:| -------------:|
| Precision (FM)|  0.48.          |     0.77      |
| Recall (FM)   |  0.71           |     0.62      |
| PM Ratio      |  0.61           |     0.88      |
| Accuracy (PM) |  0.46           |     0.74      |
| Accuracy (FM) |  0.60           |     0.87      |

## Оценка тональности упоминания аспекта (эксплицитного аспекта)

Для определения тональности аспектов, мы решили для каждого упоминания взять окно в тексте отзыва и прогнать корпус полученных окон через векторизатор. Таким же образом мы должны поступить и с тестовыми данными.
Окна векторизуются при помощи TF-IDF. Странным образом, эмбеддинги этих окон при помощи BERT дали результаты хуже, а увеличение окна оценку не улучшило (~0.66 accuracy).

В качестве модели была выбрана логистическая регрессия, ее recall и f1 оказались выше, чем у наивного байеса (Gaussian и Multinomial). Precision значительно лучше оказался у Multinomial, однако ее f1-score ниже чем у лог.рег. на 0.08.

Несмотря на то, что мы выбрали лучшую модель с точки зрения f1-score, она все равно большинство отзывов считает positive. negative определяется плохо. Это объясняет низкий f1 при высокой accuracy.

Evaluation-scores:

| Model     | Accuracy       | 
|:------------- |:---------------:|
| Baseline (PM) |  0.57         |    
| Baseline (FM)  |   0.67      | 
| Logreg (PM)    |  0.46           |    
| Logreg (FM)    |  0.45           |




## Оценка тональности всего отзыва по категориям

Для оценки тональности всего отзыва по категориям мы решили использовать количественный подход. Мы посчитали сколько аспектов положительные, нейтральные или отрицательные в каждом отзыве и к какой категории они относятся. Сопаставили эту таблицу с тональностью всей категории в отзыве и обучал разные модели предсказывать тональность по этим данным.

Обучили 4 модели:
* 4 слойная feed-forward нейросеть
* SVM
* Decision Tree Classifier
* KNN Neigbours Classifier

Нейросеть имела следущую архитектуру:
* 4 слоя
* нормирование
* везде ReLU как функция активации
* dropout = 0.4

Остальные модели выбирались с целью охватить наибольшее разнообразие (линейная модель, tree, knn). Ко всем ним гридсерчем подбирались гиперпараметры. Лучший результат получился у Decision Tree Classifier. Модель лучше всего предсказывает положительные отзывы. Это неудивительно: в разметке полученной в результате выполнение предыдущего задания эта категория отзывов доминирует (negative и neutral намного меньше).

| Category      |        Number   | 
|:------------- |:---------------:|
| positive|  1005          |
| negative |  89          |
| neutral    |  43          |

Evaluation-scores:

| Model     | Accuracy       | 
|:------------- |:---------------:|
| Baseline |  0.52         |    
| Decision Tree   |   0.73        | 


P.S.
Результаты теста на золотом стандарте:

| Model     | Accuracy       | 
|:------------- |:---------------:|
| Baseline |  0.52         |    
| 4 layer NN   |   0.78        | 
| Decision Tree      |  0.83           |    
| SVM |             0.80 |
| KNN |  0.82         | 
