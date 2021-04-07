Python - Django
===============

1. Cel zajęć
   1. Prezentaacja rozbudowanego pakietu do tworzenia aplikacji/serwisów webowych
   2. Umiejętność tworzenia prostych serwisów webowych
2. Dlaczego django
   1. Zgodny z arhitekturą MVC
      1. Model View Controler

      ![mvc block diagram][mvc-block]

      2. różnica pomiędzy lokalną aplikacją a serwisem internetowym
   2. Prosty i rozbudowany jednocześnie
   3. Należy pamiętać o alternatywach
      - Flask
      - Pyramid

3. Instalacja
   1. Virtual envirinment
   ```
   python3 -m venv .venv
   source .venv/bin/activate
   ```
   2. Pip
   ```
   pip install django
   ```
   3. Sprawdzenie wersji django.
   ``` python
   import django
   django.get_version()
   ```
   4. django-admin
4. Tworzenie projektu
   1. startproject 
   ```
   django-admin startproject
   ```
   2. Wyjaśnienie powstałej struktury.
   ```
   tree -I __pycache__
   ```
   3. manage.py - od teraz to on zarządza naszą aplikacją
   4. project - app - różnice
   5. runserver - prezentacja działającego serwisu
   ```
   pytyhon manage.py runserver
   ```
5. Pierwszy widok
   1. Request - Response

   ![request - response][req-resp]

   2. prosty widok index - HttpResponse
   ``` python
   from django.shortcuts import render
   from django.http import HttpResponse

   from .models import Question

   # Create your views here.
   def index(request):
      return HttpResponse("Hello world!")
   ```
6. Pierwszy model
   1. Model Book
   ``` python
   from django.db import models

   # Create your models here.

   class Book(models.Model):
      title = model.CharField(max_length=200)
      author = model.CharField(max_length=200)
      release = model.DateField()
      rating = DecimalField()
   ```
   2. Migracja: 
      1. Tworzenie migracji 
      ```
      python manage.py makemigrations
      ```
      2. Wykonywanie migracji
      ```
      python manage.py migrate
      ```
   3. Podgląd sql'a
   ```
   python manage.py sqlmigrate book 0001
   ```
   ``` sql
   BEGIN;
   --
   -- Create model Book
   --
   CREATE TABLE "book_book" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "title" varchar(200) NOT NULL, "author" varchar(200) NOT NULL, "release" date NOT NULL, "rating" decimal NOT NULL);
   COMMIT;
   ```
   5. Uzupełnienie widoku o model
   ``` python
   from django.shortcuts import render

   from django.http import HttpResponse

   from .models import Book

   # Create your views here.
   def index(request):
      books = Book.objects.all()
      output = ["%d: %s\n" % (book.id,book.title) for book in books]
      return HttpResponse(output)
   ```
   6. admin
      1. Utworzenie superusera
      ```
      python manage.py createsuperuser
      ```
      2. Logowanie
      3. Dodanie aplikacji ankieta
      ``` python
      # admin.py
      from django.contrib import admin

      from .models import Book
      # Register your models here.

      admin.site.register(Book)
      ```
      4. Obsługa panelu administratora
7. Rozbudowa widoków
   1. Widok szczegółowy
   ``` python
   # views.py
   def detail(request, book_id):
      book = Book.objects.get(pk=book_id)
      output = "%d: %s by %s,<br> released in %s" % (
         book.id, 
         book.title, 
         book.author, 
         book.release)
      return HttpResponse(output)
   ```
   ``` python
   # urls.py
   from django.urls import path

   from . import views

   urlpatterns = [
      path('', views.index, name='index'),
      path('<int:book_id>/', views.detail, name='detail')
   ]
   ```
   2. System szablonów

   templates/book/index.html
   ``` html
   <!DOCTYPE html>
   <html>
   <head>
      <meta charset="utf-8">
      <title>Book index page</title>
   </head>
   <body>
      <p>Books index:</p>
      {% if books_list %}
         {% for book in books_list %}
               <li>{{book.title}} by {{book.author}}</li>
         {% endfor %}
      {% endif %}
   </body>
   </html>
   ```
   views.py
   ``` python
   def index(request):
      books = Book.objects.all()
      return render(request, 'book/index.html', {'books_list': books})
   ```
   3. Prosty formularz

   templates/book/detail.html
   ``` html
   <!DOCTYPE html>
   <html>
   <head>
      <meta charset="utf-8">
      <title>Book detail page</title>
   </head>
   <body>
      <h1>{{ book.title }}</h1>
      <p>Book details:</p>
      <p>{{book.title}} by {{book.author}} released in {{book.release}}</p>
      <p>Last rating: {{book.rating}}</p>
      

      {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

      <form action="/{{book.id}}/vote" method="post">
         {% csrf_token %}
         <label for="fname">Your vote:</label><br>
         <input type="text" id="rating0" name="rating"><br>
         <input type="submit" value="Vote">
      </form>
   </body>
   </html>
   ```
   urls.py
   ``` python
   from django.urls import path

   from . import views

   urlpatterns = [
      path('', views.index, name='index'),
      path('<int:book_id>/', views.detail, name='detail'),
      path('<int:book_id>/vote', views.vote, name='vote')
   ]
   ```
   views.py
   ``` python
   def vote(request, book_id):
      rating = request.POST['rating']
      book = Book.objects.get(pk=book_id)
      book.rating = rating
      book.save()
      return HttpResponseRedirect(reverse('detail', args=(book_id,)))
   ```
8. django REST framework

![rest][rest]

9. Lektura

   - [Django Project](https://www.djangoproject.com/start/)
   - [Django For Everybody](https://www.dj4e.com/)


[mvc-block]: ./mvc-block.drawio.png
[req-resp]: ./req-resp.drawio.png
[rest]: ./rest.drawio.png