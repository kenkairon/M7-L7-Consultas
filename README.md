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
    from django.utils.dateparse import parse_date

    def todos_clientes(request):
        clientes = Cliente.objects.all()
        return render(request, 'consultas_orm/todos_clientes.html', {'clientes': clientes})

    def clientes_activos(request):
        clientes = Cliente.objects.filter(activo=True)
        return render(request, 'consultas_orm/clientes_activos.html', {'clientes': clientes})


    def clientes_rango_fechas(request):
        inicio = request.GET.get('inicio', None)
        final = request.GET.get('final', None)

        inicio = parse_date(inicio) if inicio else None
        final = parse_date(final) if final else None

        if inicio and final:
            clientes = Cliente.objects.filter(fecha_nacimiento__range=(inicio, final))
        elif inicio:
            clientes = Cliente.objects.filter(fecha_nacimiento__gte=inicio)
        elif final:
            clientes = Cliente.objects.filter(fecha_nacimiento__lte=final)
        else:
            clientes = Cliente.objects.all()

        return render(request, 'consultas_orm/clientes_rango_fechas.html', {
            'clientes': clientes,
            'inicio': inicio,
            'final': final,
        })


14. en consultas_orm/urls.py
    ```bash
    from django.urls import path
    from . import views

    urlpatterns = [
        path('todos/', views.todos_clientes, name='todos_clientes'),
        path('activos/', views.clientes_activos, name='clientes_activos'),
        path('rango-fechas/', views.clientes_rango_fechas, name='clientes_rango_fechas'),
    ]

15. consultas/urls.py
    ```bash
    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('consultas_orm.urls')),
        
    ]
16. Agremamos el archivo consultas_orm.py
    ```bash
    import os
    import django

    # Configurar el entorno de Django
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'consultas.settings')  # Cambia "myproject" por el nombre de tu carpeta de configuración
    django.setup()

    from consultas_orm.models import Cliente
    from datetime import date

    # Poblar la tabla Cliente (consulta_orm)
    def poblar_clientes():
        clientes_data = [
            {"nombre": "Juan", "apellidos": "Pérez", "fecha_nacimiento": date(1985, 5, 20), "activo": True},
            {"nombre": "María", "apellidos": "Gómez", "fecha_nacimiento": date(1990, 8, 15), "activo": False},
            {"nombre": "Carlos", "apellidos": "Rodríguez", "fecha_nacimiento": date(1980, 3, 10), "activo": True},
            {"nombre": "Ana", "apellidos": "López", "fecha_nacimiento": date(1995, 12, 25), "activo": False},
            {"nombre": "Gabriela", "apellidos": "Hernández", "fecha_nacimiento": date(2000, 7, 5), "activo": True},
        ]
        
        for cliente in clientes_data:
            Cliente.objects.create(
                nombre=cliente["nombre"],
                apellidos=cliente["apellidos"],
                fecha_nacimiento=cliente["fecha_nacimiento"],
                activo=cliente["activo"]
            )
        print("Clientes creados con éxito.")

    # Ejecutar la función para poblar las tablas
    def main():
        print("Poblando la base de datos...")
        poblar_clientes()

    if __name__ == "__main__":
        main()

17. Poblamos  de información ejecutamos la consulta 
    ```bash
    python consultas_orm.py 

18. creamos una nueva aplicación llamada consulta_sql
    ```bash
    python manage.py consulta_sql

19. consultas/settings.py  agregamos la aplicación consulta_sql 
    ```bash
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'consultas_orm',
        'consulta_sql',
    ]
20. Generamos el modelo consulta_sql/models.py
    ```bash
    from django.db import models

    class Factura(models.Model):
        cliente_id = models.IntegerField()
        importe = models.DecimalField(max_digits=10, decimal_places=2)
        pagada = models.BooleanField(default=False)

21. Creamos la vista en consulta_sql/views.py
    ```bash	
    from django.shortcuts import render
    from django.db import connection

    def todas_facturas(request):
        query = "SELECT * FROM consulta_sql_factura"
        with connection.cursor() as cursor:
            cursor.execute(query)
            columnas = [col[0] for col in cursor.description]
            facturas = [dict(zip(columnas, row)) for row in cursor.fetchall()]
        return render(request, 'consultas_sql/todas_facturas.html', {'facturas': facturas})

    def facturas_pagadas(request):
        query = "SELECT * FROM consulta_sql_factura WHERE pagada = TRUE"
        with connection.cursor() as cursor:
            cursor.execute(query)
            columnas = [col[0] for col in cursor.description]
            facturas = [dict(zip(columnas, row)) for row in cursor.fetchall()]
        return render(request, 'consultas_sql/facturas_pagadas.html', {'facturas': facturas})


    def facturas_por_cliente(request, cliente_id):
        query = "SELECT * FROM consulta_sql_factura WHERE cliente_id = %s"
        with connection.cursor() as cursor:
            cursor.execute(query, [cliente_id])
            columnas = [col[0] for col in cursor.description]
            facturas = [dict(zip(columnas, row)) for row in cursor.fetchall()]

        return render(request, 'consultas_sql/facturas_por_cliente.html', {'facturas': facturas})

    def facturas_por_cliente_join(request, cliente_id):
        query = """
            SELECT f.*, c.nombre, c.apellidos 
            FROM consulta_sql_factura f
            JOIN consultas_orm_cliente c ON f.cliente_id = c.id
            WHERE f.cliente_id = %s
        """
        with connection.cursor() as cursor:
            cursor.execute(query, [cliente_id])
            columnas = [col[0] for col in cursor.description]
            facturas = [dict(zip(columnas, row)) for row in cursor.fetchall()]

        return render(request, 'consultas_sql/facturas_por_cliente_join.html', {'facturas': facturas})

22. Ingresamos el nuevo url en consultas/urls.py
    ```bash
    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('consultas_orm.urls')),
        path('', include('consulta_sql.urls')),
    ]

23. En consulta_sql/ulrs.py
   ```bash 
    from django.urls import path
    from . import views

    urlpatterns = [
        path('todas/', views.todas_facturas, name='todas_facturas'),
        path('pagadas/', views.facturas_pagadas, name='facturas_pagadas'),
        path('cliente/<int:cliente_id>/', views.facturas_por_cliente, name='facturas_por_cliente'),
        path('cliente/<int:cliente_id>/join/', views.facturas_por_cliente_join, name='facturas_por_cliente'),
    ]