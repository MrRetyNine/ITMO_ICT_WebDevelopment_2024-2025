# Отчет по практической 3ьей работе

## Практическая работа 3.1

### Запрос на создание 6-7 новых автовладельцев и 5-6 автомобилей

```python
owners = [
Owner.objects.create(last_name="Иванов", first_name="Иван", date_of_birth="1985-06-15"),
Owner.objects.create(last_name="Петров", first_name="Петр", date_of_birth="1990-09-20"),
Owner.objects.create(last_name="Грин", first_name="Георгий", date_of_birth="2004-09-07"),
Owner.objects.create(last_name="Кузнецов", first_name="Илья", date_of_birth="1978-04-10"),
Owner.objects.create(last_name="Смирнов", first_name="Сергей", date_of_birth="1995-02-18"),
Owner.objects.create(last_name="Федоров", first_name="Федор", date_of_birth="1988-07-30"),
]
cars = [
Car.objects.create(car_number="777", brand="Toyota", car_model="Camry", colour="Белый"),
Car.objects.create(car_number="222", brand="Hyundai", car_model="Solaris", colour="Черный"),
Car.objects.create(car_number="111", brand="Kia", car_model="Rio", colour="Белый"),
Car.objects.create(car_number="555", brand="BMW", car_model="X5", colour="Серый"),
Car.objects.create(car_number="111", brand="Mercedes", car_model="E-Class", colour="Красный"),
]
```

Каждому автовладельцу назначьте удостоверение
```python
for owner in owners:
    License.objects.create(
    owner=owner,
    license_number="123",
    license_type="B",
    receiving_date=date(2015, 5, 20)
    )
```

и от 1 до 3 автомобилей
```python
Ownership.objects.create(owner=owners[0], car=cars[0], beginning="2020-01-01", ending="2024-01-01")
Ownership.objects.create(owner=owners[0], car=cars[1], beginning="2021-03-01")
Ownership.objects.create(owner=owners[1], car=cars[2], beginning="2019-05-15", ending="2023-12-31")
Ownership.objects.create(owner=owners[2], car=cars[3], beginning="2022-06-01")
Ownership.objects.create(owner=owners[2], car=cars[4], beginning="2023-07-15")
Ownership.objects.create(owner=owners[3], car=cars[0], beginning="2018-08-01", ending="2022-08-01")
Ownership.objects.create(owner=owners[3], car=cars[2], beginning="2023-09-01")
Ownership.objects.create(owner=owners[4], car=cars[1], beginning="2021-11-01")
Ownership.objects.create(owner=owners[4], car=cars[4], beginning="2022-12-01")
Ownership.objects.create(owner=owners[5], car=cars[3], beginning="2020-10-01")
```
### Выведете все машины марки “Toyota” (или любой другой марки, которая у вас есть)

```python
Car.objects.filter(brand="Toyota")
<QuerySet [<Car: Toyota Camry (777)>]> 
```

### Найти всех водителей с именем “Олег” (или любым другим именем на ваше усмотрение)

```python
>>> Owner.objects.filter(first_name="Федор")
<QuerySet [<Owner: Федоров Федор>]>
```

### Взяв любого случайного владельца получить его id, и по этому id получить экземпляр удостоверения в виде объекта модели (можно в 2 запроса)

```python
>>> import random
>>> owner_id = random.choice(Owner.objects.all()).id
>>> License.objects.get(owner_id=owner_id)
<License: Удостоверение 123 (B) - Иванов Иван>
>>>
```

### Вывести всех владельцев красных машин (или любого другого цвета, который у вас присутствует)

```python
>>>Owner.objects.filter(ownership__car__colour="Белый").distinct()
<QuerySet [<Owner: Иванов Иван>, <Owner: Петров Петр>, <Owner: Кузнецов Илья>]>
```

### Найти всех владельцев, чей год владения машиной начинается с 2010 (или любой другой год, который присутствует у вас в базе)

```python
>>>Owner.objects.filter(ownership__beginning__year=2021).distinct()
<QuerySet [<Owner: Иванов Иван>, <Owner: Смирнов Сергей>]>
```

### Вывод даты выдачи самого старшего водительского удостоверения

```python
>>> from django.db.models import Min
>>> License.objects.aggregate(Min('receiving_date'))
{'receiving_date__min': datetime.date(2015, 5, 20)}
```

### Укажите самую позднюю дату владения машиной, имеющую какую-то из существующих моделей в вашей базе

```python
>>> Ownership.objects.aggregate(Max('ending'))
{'ending__max': datetime.date(2024, 10, 5)}
```

### Выведите количество машин для каждого водителя

```python
>>> Owner.objects.values('first_name', 'last_name').annotate(Count('ownership__car'))
<QuerySet [{'first_name': 'Георгий', 'last_name': 'Грин', 'ownership__car__count': 2}, {'first_name': 'Иван', 'last_name': 'Иванов', 'ownership__car__count': 2}, {'first_name': 'Илья', 'last_name': 'Кузнецов', 'ownership__car__count': 2}, {'first_name': 'Петр', 'last_name': 'Петров', 'ownership__car__count': 1}, {'first_name': 'Сергей', 'last_name': 'Смирнов', 'ownership__car__count': 2}, {'first_name': 'Федор', 'last_name': 'Федоров', 'ownership__car__count': 1}]>
```

### Подсчитайте количество машин каждой марки

```python
>>> Car.objects.values('brand').annotate(Count('id'))
<QuerySet [{'brand': 'BMW', 'id__count': 1}, {'brand': 'Hyundai', 'id__count': 1}, {'brand': 'Kia', 'id__count': 1}, {'brand': 'Mercedes', 'id__count': 1}, {'brand': 'Toyota', 'id__count': 1}]>
```

### Отсортируйте всех автовладельцев по дате выдачи удостоверения

```python
>>>Owner.objects.distinct().order_by('license__receiving_date')
<QuerySet [<Owner: Иванов Иван>, <Owner: Петров Петр>, <Owner: Грин Георгий>, <Owner: Кузнецов Илья>, <Owner: Смирнов Сергей>, <Owner: Федоров Федор>]>
```

## Практическая работа 3.2
Реализовать ендпоинты для добавления и просмотра скилов методом, описанным в пункте выше.

### Добавление скилов

POST http://127.0.0.1:8000/war/skills/create/

```json
{
    "title": "write"
}
```
```json
{
    "Success": "Skill 'write' created successfully."
}
```

### Просмотр скилов

GET http://127.0.0.1:8000/war/warriors/

```json
{
    "Skills": [
        {
            "id": 1,
            "title": "read"
        }
    ]
}
```
### Вывод полной информации о всех войнах и их профессиях
GET http://127.0.0.1:8000//war/warriors/professions/
```json
[
    {
        "id": 4,
        "race": "d",
        "name": "Updated Warrior Name",
        "level": 5,
        "profession": {
            "id": 3,
            "title": "студент",
            "description": "обучается в университете"
        }
    }
]
```
### Вывод полной информации о войне (по id), его профессиях и скилах. GET />
```json
{
    "id": 4,
    "race": "d",
    "name": "Updated Warrior Name",
    "level": 5,
    "profession": {
        "id": 3,
        "title": "студент",
        "description": "обучается в университете"
    },
    "skill": []
}
```
### Удаление война по id. DELETE /delete>
```json
{
    "success": "Warrior deleted successfully"
}
```
### Редактирование информации о войне. PUT /update>
```json
{
    "level": 10
}
```
```json
{
    "success": "Warrior updated successfully",
    "data": {
        "name": "Alex",
        "level": 10,
        "race": "s",
        "profession": 3
    }
}
```
