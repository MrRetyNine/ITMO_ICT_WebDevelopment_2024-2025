# Отчет по лабораторной работе 2

## Введение
"Доска домашних заданий" представляет собой веб-приложение, разработанное с использованием фреймворка Django. Основная цель приложения — предоставить платформу для управления домашними заданиями, их сдачи и оценки.

## Структура проекта
Проект состоит из нескольких ключевых компонентов:

- **Модели**: Определяют структуру данных.
- **Админка**: Позволяет управлять данными через интерфейс администратора.
- **Формы**: Обрабатывают ввод данных от пользователей.
- **Представления**: Логика обработки запросов и генерации ответов.
- **Шаблоны**: HTML-страницы для отображения данных пользователям.
- **URL-ы**: Определяют маршрутизацию запросов.

## Модели
В проекте определены следующие модели:

- **Class**: Модель для представления классов.
- **User **: Расширенная модель пользователя с ролями (студент, учитель).
- **Subject**: Модель для представления предметов.
- **Homework**: Модель для домашних заданий.
- **Submission**: Модель для представления сдачи домашних заданий.

### Пример модели User
```python
class User(AbstractUser ):
    ROLE_CHOICES = [
        ('student', 'Student'),
        ('teacher', 'Teacher'),
    ]
    role = models.CharField(max_length=10, choices=ROLE_CHOICES, default='student')
    student_class = models.ForeignKey(Class, on_delete=models.SET_NULL, null=True, blank=True, related_name='students')
    birth_date = models.DateField(null=True, blank=True)
    phone_number = models.CharField(max_length=15, blank=True, null=True)

    def __str__(self):
        return f"{self.last_name} {self.first_name} ({self.get_role_display()})"
```
## Админка
В админке реализованы следующие админские интерфейсы:

- **ClassAdmin**: Управление классами.
- **User  Admin**: Управление пользователями.
- **SubjectAdmin**: Управление предметами.
- **HomeworkAdmin**: Управление домашними заданиями.
- **SubmissionAdmin**: Управление сдачами домашних заданий.

## Формы
Формы используются для обработки данных, введенных пользователями:

- **User  RegistrationForm**: Форма регистрации пользователей.
- **User  LoginForm**: Форма для входа в систему.
- **HomeworkForm**: Форма для добавления и редактирования домашних заданий.
- **SubmissionForm**: Форма для сдачи домашних заданий.

### Код forms
```python
from django import forms
from django.contrib.auth.forms import UserCreationForm, AuthenticationForm
from blog.models import Submission, Homework, User


class UserRegistrationForm(UserCreationForm):
    class Meta:
        model = User
        fields = ['username', 'first_name', 'last_name', 'password1', 'password2', 'role', 'student_class']


class UserLoginForm(AuthenticationForm):
    class Meta:
        model = User
        fields = ['username', 'password']


class HomeworkForm(forms.ModelForm):
    class Meta:
        model = Homework
        fields = ['subject', 'title', 'assigned_date', 'due_date', 'description', 'penalties_info']


class SubmissionForm(forms.ModelForm):
    class Meta:
        model = Submission
        fields = ['submission_text']


class GradeSubmissionForm(forms.ModelForm):
    class Meta:
        model = Submission
        fields = ['grade']


class UserUpdateForm(forms.ModelForm):
    class Meta:
        model = User
        fields = ['first_name', 'last_name', 'phone_number', 'birth_date']

```

## Представления
Представления обрабатывают запросы и возвращают соответствующие ответы:

- **main**: Главная страница приложения.
- **register**: Страница регистрации.
- **user_login**: Страница входа.
- **account_view**: Личный кабинет пользователя.
- **add_homework**: Страница добавления домашнего задания.
- **homework_list**: Список домашних заданий.

## Шаблоны
Шаблоны определяют, как данные будут отображаться пользователям. Примеры шаблонов:

- **main.html**: Главная страница.
- **login.html**: Страница входа.
- **register.html**: Страница регистрации.
- **homework_list.html**: Список домашних заданий.

## URL-ы
Маршрутизация запросов осуществляется через файл `urls.py`, который определяет, какие представления будут вызываться для различных URL-адресов.

## Заключение
Проект "Доска домашних заданий" предоставляет удобный интерфейс для управления домашними заданиями, их сдачи и оценки. Использование Django позволяет легко расширять функциональность и поддерживать приложение.