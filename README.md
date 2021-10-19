# Краснодар: анализ аттракторов

Алгоритм:
1. Поиск всех мест GP
2. Назначение местам своих категорий

## Поиск мест ([notebooks/1 - Collecting places from GP.ipynb](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/notebooks/1%20-%20Collecting%20places%20from%20GP.ipynb))
<img src="https://i.ibb.co/mb15cnn/Drawing-sketchpad.jpg" alt="Drawing-sketchpad" border="0">

1. Границы города Краснодара были взяты из Яндекс.Карт: https://yandex.ru/maps/geo/krasnodar/53166138/?ll=38.985679%2C45.066115&z=11.71
2. Построена сетка точек на территории города Краснодар через каждые 1000 метров: черные линии - расстояние от центров окружностей.
3. По ним осуществлен поиск мест в округе 710 метров. 710 рассчитано по теореме Пифагора: изображено красной линией - радиус окружности.
4. Удалены дубликаты мест: те места, которые попадают в оранжевую зону, могли записаться несколько раз.

**Результат:**
Датасет [raw_data.csv](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/data/raw_data.csv)содержит сырые данные первого сбора через GP API (в последствии локально обновлялся для сбора доп мест). По ним 99 уникальных категорий. Мест туристической привлекательности: 14. 

## Cбор метаданных ([notebooks/2 - Collecting metadata.ipynb](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/notebooks/2%20-%20Collecting%20metadata.ipynb) )
Сырые данные преобразованы в нормальный вид (парсинг GP с помощью библиотеки `selenium`): собраны русскоязычные названия и категории мест от GP.

**Результат:**
Датасет [places_GP_temp.csv](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/data/places_GP_temp.csv.csv) 
|поле|обозначение|
|:--|:--|
|geometry|точечные данные с широтой и долготой
|rating| рейтинг
|user_ratings_total| количество отзывов
|types| тип места в соответствии с Google API
|category| категория ru

На этом же этапе были сформированы возможные категории ([распределение категорий.txt](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/data/распределение%20категорий.txt)). В скобках указано количество мест. Через двоеточие `types` Google Places: https://developers.google.com/maps/documentation/places/web-service/supported_types?hl=ru.

## Сбор количества фото ([notebooks/3 - Adding count photos.ipynb](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/notebooks/3%20-%20Adding%20count%20photos.ipynb))
С помощью отправки запроса метода GET HTTP и регулярных выражений - были извлечены числовые показатели количества фотографий на странице.

## Категоризация ([notebooks/4 - Categorization.ipynb](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/notebooks/4%20-%20Categorization.ipynb))
1. В соответствии с заданной типлогией, был сформирован перечень категорийю Ниже представлена категория из поля `category` и количество мест по ним:
```
                         6993
Магазин                  4279
Услуга                   1968
Здоровье                  600
Общепит                   466
Образование               379
Жилой комплекс            206
Остановка ОТ              205
Спорт                     108
Достопримечательность      93
Культура                   64
Зеленые зоны               40
Развлечения                33
Озеро                      32
Локальность                31
Мост                       14
Площадь                     9
Набережная                  4
Пляж                        3
```
Категории были идентифицированы по вхождению ключевых слов, например как в файле [распределение категорий.txt](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/data/распределение%20категорий.txt)

2. Далее с помощью метода MinMaxScaler() были нормированы показатели: количество фото, количество отзывов, рейтинг и найдена целевая функция в виде суммы таких показателей.

**Результат:**
Датасет [places_GP.csv](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/data/places_GP.csv) 
|поле|обозначение|
|:--|:--|
|geometry|точечные данные с широтой и долготой
|rating| рейтинг
|count_review| количество отзывов
|count_photo| количество фото
|types| тип места в соответствии с Google API
|type| категория ru
|category_2| наши категории (агрегированы по категориям GP) - имеют уровень ниже, чем `category`
|category| наши категории (агрегированы по категориям GP)
|normalized_rating| сумма по нормированным показателям 
|rating_by_category| сумма по нормированным по категории показателям

`category`: 
|категория|какие значения `category_2` включают|
|:--|:--|
|Магазины|Магазины, Магазины одежды
|Культура| Религия, Музей, Библиотека, Театр, Галерея
|Зеленые зоны| Сад, Парк, Бульвар, Сквер
|Достопримечательность| Достопримечательность, Памятник
|Развлечения| Кинотеатр, Развлечение, Аттракционы, Зоопарк, Боулинг

Остальные категории имеют такие же значения, как и в `category_2`.

## Визуализация ([notebooks/5 - Visualization.ipynb](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/notebooks/5%20-%20Visualization.ipynb))

## Сентимент-анализ ([notebooks/7 - Text analysis.ipynb](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/notebooks/7%20-%20Text%20analysis.ipynb))

**Результат:**
Датасет [places_GP_sent.csv](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/data/places_GP_sent.csv) 

(описано только то, что добавлено в `places_GP`)

|поле|обозначение|
|:--|:--|
|count_downloaded| количество загруженных отзывов
|count_filled| количество заполненных отзывов
|count_pos| количество позитивных отзывов
|count_neg| количествонегативных отзывов
|count_neutral| количество нейтральных отзывов
|median_score| медиана сентиментов отзывов
|median_negative| медиана сентиментов отзывов среди негативных
|median_positive| медиана сентиментов отзывов среди позитивных

Датасет [places_GP_attractors.csv](https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/data/places_GP_attractors.csv) 

Срез по аттракторам:
```
252              Городской сад, парк культуры и отдыха
1609                                  Парк «Краснодар»
1616                            Стадион ФК "Краснодар"
15196    Ботанический сад им. И.С. Косенко (Дендрарий)
15198                                Чистяковская Роща
15207                            Сквер им. Г.К. Жукова
15233                                    Мост Поцелуев
15248                             Екатерининский сквер
15267     Парк культуры и отдыха имени 30-летия Победы
15297                               ЦПКиО им. Горького
```

Просмотреть визуализацию по сентиментам всех мест и аттракторов: https://github.com/yupest/Novoznamenskiy_Krasnodar/blob/main/notebooks/9%20-%20Visualisation%20sentiments/9%20-%20Analysis%20and%20Visualization%20of%20reviews.md#анализ-сентиментов 



