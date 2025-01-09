# M7-L7-Consultas
Educativo y de Aprendizaje Personal

---
## Tabla de Contenidos
- [Tecnologías](#Tecnologías)
- [Configuración Inicial](#configuración-Inicial)
- [Creación del Modelo](#creación-del-modelo)
---
# Tecnologías
- Django: Framework web en Python.
- SQLite:
--- 
# Configuración Inicial 
1. Entorno virtual 
    ```bash 
    python -m venv venv

2. Activar el entorno virtual
    ```bash 
    venv\Scripts\activate

3. Instalar Django
    ```bash 
    pip install django 
        
4. Actualizamos el pip 
    ```bash
    python.exe -m pip install --upgrade pip

5. Crear el proyecto de django consultas
    ```bash 
    django-admin startproject consultas

6. Guardamos dependencias
    ```bash
    pip freeze > requirements.txt

7. Ingresamos al proyecto consultas
    ```bash 
    cd migraciones

9. Creamos la aplicacion llamada consultas_orm
    ```bash     
    python manage.py startapp consultas_orm


10. Configuración de /settings.py 
    ```bash 
        INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'consultas_orm',
        ]
       



# Creación del Modelo 

11. en consultas_orm/models.py
    ```bash
    from django.db import models

    # Create your models here.
    from django.db import models

    class Cliente(models.Model):
        nombre = models.CharField(max_length=100)
        apellidos = models.CharField(max_length=100)
        fecha_nacimiento = models.DateField()
        activo = models.BooleanField(default=True)

12. Hacemos las migraciones correspondientes
    ```bash
    python manage.py makemigrations
    python manage.py migrate

13. consultas_orm/views.py
    ```bash
    from django.shortcuts import render
    from .models import Cliente
    import datetime

    def todos_clientes(request):
        clientes = Cliente.objects.all()
        return render(request, 'consulta_orm/todos_clientes.html', {'clientes': clientes})

    def clientes_activos(request):
        clientes = Cliente.objects.filter(activo=True)
        return render(request, 'consulta_orm/clientes_activos.html', {'clientes': clientes})

    def clientes_rango_fechas(request):
        inicio = datetime.date(1980, 1, 1)
        final = datetime.date(2000, 12, 31)
        clientes = Cliente.objects.filter(fecha_nacimiento__range=(inicio, final))
        return render(request, 'consulta_orm/clientes_rango_fechas.html', {'clientes': clientes})


14. en migraciones/urls.py
    ```bash
    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('modelos.urls')),
    ]

15. modelos/urls.py
    ```bash
    from django.urls import path
    from . import views

    urlpatterns = [
        path('', views.index, name='index'),
    ]
16. Nos Vamos http://127.0.0.1:8000/ 
    ```bash
    python manage.py runserver