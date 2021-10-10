# Краснодар: анализ аттракторов

Алгоритм:
1. Поиск всех мест GP
2. Назначение местам своих категорий
3. Отбрасывание ненужных мест

## 1 этап (07.10)
<img src="https://i.ibb.co/mb15cnn/Drawing-sketchpad.jpg" alt="Drawing-sketchpad" border="0">

1. Границы города Краснодара были взяты из Яндекс.Карт: https://yandex.ru/maps/geo/krasnodar/53166138/?ll=38.985679%2C45.066115&z=11.71
2. Построена сетка точек на территории города Краснодар через каждые 1000 метров: черные линии - расстояние от центров окружностей.
3. По ним осуществлен поиск мест в округе 710 метров. 710 рассчитано по теореме Пифагора: изображено красной линией - радиус окружности.
4. Удалены дубликаты мест: те места, которые попадают в оранжевую зону, могли записаться несколько раз.

**upd**
Собрано 16329 мест (датасет `raw_data.csv`). По ним 99 уникальных категорий. Мест туристической привлекательности: 14. 

Мест с наличием отзывов: 9456. По ним 93 уникальных категории. Категории, которые по которым не найдено ни 1 места с отзывами (рейтингом):

Задачи: 
1. Сырые данные преобразовать в нормальный вид (парсинг GP).
2. Построить классификацию мест в соответствии с типологией.
3. Проверить корректность классификации (категорий).

## 1.2 этап (08.10)

В папку `data` добавлен файл с местами `places_GP_temp.csv`. Данные имеют координаты: geometry. 
|поле|обозначение|
|:--|:--|
|rating| рейтинг
|user_ratings_total| количество отзывов
|types| тип места в соответствии с Google API
|category| категория ru

## 1.3 (10.10)
Возможные категории (тот же текст представлен в файле `распределение категорий.txt`). В скобках количество мест. Через двоеточие `types` Google Places: https://developers.google.com/maps/documentation/places/web-service/supported_types?hl=ru.

Магазин (4000) = гипермаркет, супермаркет, одежды, продукты:
'home_goods_store, furniture_store, clothing_store, grocery_or_supermarket, electronics_store, supermarket, liquor_store, jewelry_store, shoe_store, pet_store, shopping_mall, book_store, convenience_store, department_store, bicycle_store, movie_rental, hardware_store'

Услуга = ремонт, салон, агенство (2354):
'car_repair, beauty_salon, hair_care, real_estate_agency, travel_agency, car_dealer, spa, florist, car_rental, car_wash, plumber, electrician, moving_company, laundry, lawyer, funeral_home, post_office, locksmith, painter, insurance_agency, gas_station, bank'

Образовательное учереждение (397) = школа, детский сад, университет:
'school, university, secondary_school, primary_school'

Здоровье (560) = аптека, клиника, больница:
'pharmacy, dentist, hospital, drugstore, veterinary_care, doctor'

Спорт (115) = спортзал, стадион:
'gym, stadium'

Открытые общественные пространства = 
Парк+Сквер+Бульвар(40), Сад (6)
площадь(10)
набережная (1)
парк развлечений, 
зоопарк,
Озеро (32)
пляж (4):
'tourist_attraction, park, natural_feature, town_square, amusement_park, zoo'

еда (1098) = ресторан, кафе, бар, пекарня
'restaurant, cafe, bar, bakery, meal_delivery, meal_takeaway'

ЖК (160)

Помещения культурные и развлекательные (47)
музей+галерея+церковь, собор+театр+кинотеатр + боулинг+библиотека:
'bowling_alley, art_gallery, movie_theater, place_of_worship, library'

Локальности(32) - почти нет отзывов
'locality, political, neighborhood, sublocality, sublocality_level_2, sublocality_level_1, airport, cemetery'

Остановки (235):
'bus_station, light_rail_station, transit_station'

Остальное - куда отнести? будем ли включать в анализ?:
'accounting, storage, local_government_office, finance, lodging, general_contractor, atm, night_club, parking, courthouse, premise, fire_station, police,  roofing_contractor, embassy'
