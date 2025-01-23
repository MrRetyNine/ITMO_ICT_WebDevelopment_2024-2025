# Отчет по лабораторной работе 3

## Введение
В данной лабораторной работе была разработана система управления автобусами с использованием фреймворка Django. Система включает в себя модели для автобусов, водителей, маршрутов и расписаний, а также API для взаимодействия с этими моделями.

## Структура проекта
Проект состоит из следующих основных компонентов:

- **Модели**: Определяют структуру данных.
- **Сериализаторы**: Преобразуют данные моделей в формат JSON и обратно.
- **Представления (Views)**: Обрабатывают HTTP-запросы и возвращают ответы.
- **URL-адреса**: Определяют маршруты для API.
- **Админка**: Позволяет управлять данными через интерфейс администратора.

## Модели
### Bus
```python
class Bus(models.Model):
    class BusType(models.TextChoices):
        VERY_SMALL = 'VS', _('Особо малый')
        SMALL = 'S', _('Малый')
        MEDIUM = 'M', _('Средний')
        LARGE = 'L', _('Большой')
        VERY_LARGE = 'VL', _('Особо большой')

    state_num = models.CharField(max_length=20, unique=True)
    capacity = models.PositiveIntegerField()
    bus_type = models.CharField(max_length=10, choices=BusType.choices)

    def __str__(self):
        return f"{self.state_num} ({self.get_bus_type_display()})"
```

### BusDriver
```python
class BusDriver(models.Model):
    class DriverCategory(models.TextChoices):
        NORMAL_BUS = 'A', _('Обычный автобус')
        ELECTRIC_BUS = 'B', _('Электробус')
        TOURIST_BUS = 'C', _('Туристический автобус')
        CARGO_VEHICLE = 'D', _('Грузовой автобус')

    full_name = models.CharField(max_length=40, verbose_name="ФИО")
    passport = models.CharField(max_length=20, unique=True, verbose_name="Паспортные данные")
    experience = models.PositiveIntegerField(verbose_name="Стаж работы в годах")
    driver_category = models.CharField(max_length=10, choices=DriverCategory.choices, verbose_name="Категория")
    salary = models.DecimalField(max_digits=10, decimal_places=2, verbose_name="Зарплата")

    def __str__(self):
        return f"{self.full_name} ({self.get_driver_category_display()})"
```

### Route
```python
    route_num = models.CharField(max_length=20, unique=True)
    start_point = models.CharField(max_length=100)
    end_point = models.CharField(max_length=100)
    start_time = models.TimeField()
    end_time = models.TimeField()
    interval = models.PositiveIntegerField()  # Интервал в минутах
    duration = models.PositiveIntegerField()  # Протяженность в минутах
    is_active = models.BooleanField(default=True)

    def __str__(self):
        return f"Route {self.route_num} is {self.is_active}: {self.start_point} → {self.end_point}"
```

### WorkSchedule
```python
    class Status(models.TextChoices):
        ACTIVE = 'active', _('Active')
        INACTIVE = 'inactive', _('Inactive')
        ACCIDENT = 'accident', _('Accident')

    driver = models.ForeignKey(BusDriver, on_delete=models.CASCADE, related_name="schedules")
    bus = models.ForeignKey(Bus, on_delete=models.CASCADE, related_name="schedules")
    route = models.ForeignKey(Route, on_delete=models.CASCADE, related_name="schedules")
    status = models.CharField(max_length=10, choices=Status.choices, default=Status.ACTIVE)
    date = models.DateField()
    shift_start = models.TimeField()
    shift_end = models.TimeField()

    def __str__(self):
        return f"Schedule: {self.driver.passport} - {self.bus.state_num} on {self.route.route_num}"
```

## Сериализаторы
Сериализаторы используются для преобразования данных моделей в JSON-формат и обратно. Каждый сериализатор соответствует своей модели и включает все поля.

### Пример сериализатор для водителей
```python
class BusDriverSerializer(serializers.ModelSerializer):
    class Meta:
        model = BusDriver
        fields = '__all__'
```
## Представления (Views)
Представления реализуют логику обработки запросов. Ниже приведены примеры представлений и их описание.

**CRUD-запросы для автобусов**
```python
class BusesListAPIView(generics.ListAPIView):
    serializer_class = BusSerializer
    queryset = Bus.objects.all()


class BusCreateView(generics.CreateAPIView):
    serializer_class = BusSerializer
    queryset = Bus.objects.all()


class BusDetailView(generics.RetrieveAPIView):
    serializer_class = BusSerializer
    queryset = Bus.objects.all()
    lookup_field = 'state_num'
```

**CRUD-запросы для водителей**
```python
class BusDriversListAPIView(generics.ListAPIView):
    serializer_class = BusDriverSerializer
    queryset = BusDriver.objects.all()


class BusDriverCreateView(generics.CreateAPIView):
    serializer_class = BusDriverSerializer
    queryset = BusDriver.objects.all()


class BusDriverDetailView(generics.RetrieveAPIView):
    serializer_class = BusDriverSerializer
    queryset = BusDriver.objects.all()
    lookup_field = 'passport'
```

**CRUD-запросы для маршрута**
```python
class RoutesListAPIView(generics.ListAPIView):
    serializer_class = RouteSerializer
    queryset = Route.objects.all()


class RouteCreateView(generics.CreateAPIView):
    serializer_class = RouteSerializer
    queryset = Route.objects.all()


class RouteDetailView(generics.RetrieveAPIView):
    serializer_class = RouteSerializer
    queryset = Route.objects.all()
    lookup_field = 'route_num'
```

**CRUD-запросы для расписания**
```python
class WorkSchedulesListAPIView(generics.ListAPIView):
    serializer_class = WorkScheduleSerializer
    queryset = WorkSchedule.objects.all()


class WorkScheduleCreateView(generics.CreateAPIView):
    serializer_class = WorkScheduleSerializer
    queryset = WorkSchedule.objects.all()


class WorkScheduleDetailView(generics.RetrieveAPIView):
    serializer_class = WorkScheduleSerializer
    queryset = WorkSchedule.objects.all()
```

**Аналитические запросы**
```python
class DriversByRouteView(APIView):
    """Список водителей, работающих на определенном маршруте с указанием графика их работы?"""

    def get(self, request, route_num):
        schedules = WorkSchedule.objects.filter(route__route_num=route_num).select_related('driver', 'route')
        serialized_data = WorkScheduleSerializer(schedules, many=True).data
        return Response(serialized_data)


class RouteTimingView(APIView):
    """Когда начинается и заканчивается движение автобусов на каждом маршруте?"""

    def get(self, request):
        routes = Route.objects.all()
        data = routes.values('route_num', 'start_time', 'end_time')
        return Response(data)


class TotalRouteDurationView(APIView):
    """Какова общая протяженность маршрутов, обслуживаемых автопарком?"""

    def get(self, request):
        total_duration = Route.objects.aggregate(total_duration=Sum('duration'))
        return Response({'total_duration': total_duration['total_duration']})


class AbsentBusesView(APIView):
    """Какие автобусы не вышли на линию в заданный день и по какой причине?"""

    def get(self, request, date):
        absent_schedules = WorkSchedule.objects.filter(date=date, status=WorkSchedule.Status.INACTIVE)
        serialized_data = WorkScheduleSerializer(absent_schedules, many=True).data
        return Response(serialized_data)


class DriverCountByCategoryView(APIView):
    """Сколько водителей каждого класса работает в автопарке?"""

    def get(self, request):
        driver_counts = BusDriver.objects.values('driver_category').annotate(count=Count('id'))
        return Response(driver_counts)


class ReportView(APIView):
    """Каково общее состояние автопарка, включая количество автобусов по типам, количество маршрутов, которые они обслуживают, количество водителей, а также средний стаж работы водителей?"""

    def get(self, request):
        bus_types = Bus.objects.values('bus_type').annotate(
            bus_count=Count('id'),
            route_count=Count('schedules__route', distinct=True),
            driver_count=Count('schedules__driver', distinct=True),
        )
        total_duration = Route.objects.aggregate(total_duration=Sum('duration'))
        driver_stats = BusDriver.objects.aggregate(
            avg_experience=Avg('experience'), total_count=Count('id')
        )
        report = {
            'bus_types': list(bus_types),
            'total_route_duration': total_duration['total_duration'],
            'driver_stats': driver_stats,
        }
        return Response(report)
```

## URL-адреса
URL-адреса определяют маршруты для API. Например:
```python
urlpatterns = [
    path('drivers/', BusDriversListAPIView.as_view(), name='driver-list'),
    path('drivers/create/', BusDriverCreateView.as_view(), name='driver-create'),
    path('drivers/<str:passport>/', BusDriverDetailView.as_view(), name='driver-detail'),
    path('buses/', BusesListAPIView.as_view(), name='bus-list'),
    path('buses/create/', BusCreateView.as_view(), name='bus-create'),
    path('buses/<str:state_num>/', BusDetailView.as_view(), name='bus-detail'),
    path('routes/', RoutesListAPIView.as_view(), name='route-list'),
    path('routes/create/', RouteCreateView.as_view(), name='route-create'),
    path('work-schedules/', WorkSchedulesListAPIView.as_view(), name='work-schedule-list'),
    path('work-schedules/create/', WorkScheduleCreateView.as_view(), name='work-schedule-create'),
]
```

## Админка
Система включает в себя административный интерфейс для управления моделями. Все модели зарегистрированы в админке, что позволяет легко добавлять, редактировать и удалять записи.

```python
from django.contrib import admin
from .models import Bus, Route, BusDriver, WorkSchedule

admin.site.register(Bus)
admin.site.register(Route)
admin.site.register(BusDriver)
admin.site.register(WorkSchedule)
```

## Заключение
В результате выполнения лабораторной работы была создана полноценная система управления автобусами с использованием фреймворка Django. 

### Основные достижения:
- Реализованы модели для автобусов, водителей, маршрутов и расписаний, что позволяет эффективно управлять данными.
- Созданы сериализаторы для преобразования данных в формат JSON и обратно, что упрощает взаимодействие с API.
- Разработаны представления (views) для обработки различных HTTP-запросов, включая получение, создание и обновление данных.
- Настроены URL-адреса для маршрутизации запросов к соответствующим представлениям.
- Реализован административный интерфейс, который позволяет легко управлять данными через графический интерфейс.

Таким образом, данная лабораторная работа продемонстрировала основные принципы разработки веб-приложений на Django и предоставила базу для дальнейшего изучения и развития в этой области.