# Implementación CRUD para el Modelo Servicio
# 1. Aquí el modelo models.py que ya está creado
El modelo Servicio ya lo tienes en app_Taller/models.py:
code
Python
# app_Taller/models.py
    from django.db import models

# MODELO: CLIENTE 
# ==========================================
    class Cliente(models.Model):
      nombre = models.CharField(max_length=100)
      apellido = models.CharField(max_length=100)
      telefono = models.CharField(max_length=30, blank=True, null=True)
      email = models.EmailField(max_length=254, blank=True, null=True)
      rfc = models.CharField(max_length=20, blank=True, null=True)
      direccion = models.TextField(blank=True, null=True)
      fecha_registro = models.DateTimeField(auto_now_add=True)
  
      def __str__(self):
          return f"{self.nombre} {self.apellido}"

# MODELO: SERVICIO
# ==========================================
    class Servicio(models.Model):
      nombre = models.CharField(max_length=150)
      descripcion = models.TextField(blank=True, null=True)
      precio_base = models.DecimalField(max_digits=10, decimal_places=2, default=0)
      tiempo_est = models.IntegerField(help_text='Tiempo estimado en minutos', default=0)
      aplica_garantia = models.BooleanField(default=False)
      fecha_actualizacion = models.DateTimeField(auto_now=True)

      def __str__(self):
          return self.nombre

# 2. Procedimiento para realizar las migraciones (makemigrations y migrate).
Si ya habías ejecutado las migraciones en la primera parte, el modelo Servicio ya debería estar presente. Sin embargo, para asegurarnos de que cualquier cambio o nuevo modelo sea reconocido:
Bash
    
    python manage.py makemigrations
    
    python manage.py migrate
    
# 3. Ahora trabajamos con el MODELO: SERVICIOS
# 4. En views.py de app_Taller crear las funciones con sus códigos correspondientes (agregar_servicio, actualizar_servicio, realizar_actualizacion_servicio, borrar_servicio)
Abre el archivo app_Taller/views.py y añade las siguientes funciones para el CRUD de servicios:
Python
app_Taller/views.py
    
    from django.shortcuts import render, redirect, get_object_or_404
    from .models import Cliente, Servicio
    from django.urls import reverse

    # ... (tus funciones de Cliente e inicio_taller aquí) ...
    
    def inicio_taller(request):
      # página principal de la app (taller)
      return render(request, 'inicio.html', {})
    
    def agregar_clientes(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre', '').strip()
        apellido = request.POST.get('apellido', '').strip()
        email = request.POST.get('email', '').strip() or None
        rfc = request.POST.get('rfc', '').strip() or None
        telefono = request.POST.get('telefono', '').strip() or None
        direccion = request.POST.get('direccion', '').strip() or None
        cliente = Cliente(
            nombre=nombre,
            apellido=apellido,
            email=email,
            rfc=rfc,
            telefono=telefono,
            direccion=direccion,
        )
        cliente.save()
        return redirect('ver_clientes')
    return render(request, 'clientes/agregar_clientes.html', {})

    def ver_clientes(request):
        clientes = Cliente.objects.all().order_by('apellido', 'nombre')
        return render(request, 'clientes/ver_clientes.html', {'clientes': clientes})
    
    def actualizar_clientes(request, cliente_id):
        cliente = get_object_or_404(Cliente, id=cliente_id)
        return render(request, 'clientes/actualizar_clientes.html', {'cliente': cliente})
    
    def realizar_actualizacion_clientes(request, cliente_id):
        cliente = get_object_or_404(Cliente, id=cliente_id)
        if request.method == 'POST':
            cliente.nombre = request.POST.get('nombre', cliente.nombre).strip()
            cliente.apellido = request.POST.get('apellido', cliente.apellido).strip()
            email = request.POST.get('email', '').strip()
            cliente.email = email or None
            rfc = request.POST.get('rfc', '').strip()
            cliente.rfc = rfc or None
            telefono = request.POST.get('telefono', '').strip()
            cliente.telefono = telefono or None
            direccion = request.POST.get('direccion', '').strip()
            cliente.direccion = direccion or None
            cliente.save()
            return redirect('ver_clientes')
        # si llama GET, redirigir a formulario de edición
        return redirect('actualizar_clientes', cliente_id=cliente.id)
    
    def borrar_clientes(request, cliente_id):
        cliente = get_object_or_404(Cliente, id=cliente_id)
        if request.method == 'POST':
            cliente.delete()
            return redirect('ver_clientes')
        return render(request, 'clientes/borrar_clientes.html', {'cliente': cliente})


# ==========================================
# FUNCIONES CRUD PARA SERVICIOS
# ==========================================

    def agregar_servicio(request):
        if request.method == 'POST':
            nombre = request.POST['nombre']
            descripcion = request.POST.get('descripcion', '') # Permite que sea opcional
            precio_base = request.POST['precio_base']
            # el modelo define el campo como tiempo_est (minutos)
            tiempo_est = request.POST.get('tiempo_est', '0')
            aplica_garantia = 'aplica_garantia' in request.POST # True si está marcado, False si no
    
            Servicio.objects.create(
                nombre=nombre,
                descripcion=descripcion,
                precio_base=precio_base,
                tiempo_est=tiempo_est,
                aplica_garantia=aplica_garantia
            )
            return redirect('ver_servicio')
        return render(request, 'servicio/agregar_servicio.html')
    
    def ver_servicio(request):
        servicios = Servicio.objects.all()
        return render(request, 'servicio/ver_servicio.html', {'servicios': servicios})
    
    def actualizar_servicio(request, servicio_id):
        servicio = get_object_or_404(Servicio, pk=servicio_id)
        return render(request, 'servicio/actualizar_servicio.html', {'servicio': servicio})
    
    def realizar_actualizacion_servicio(request, servicio_id):
        servicio = get_object_or_404(Servicio, pk=servicio_id)
        if request.method == 'POST':
            servicio.nombre = request.POST.get('nombre', servicio.nombre)
            servicio.descripcion = request.POST.get('descripcion', servicio.descripcion)
            servicio.precio_base = request.POST.get('precio_base', servicio.precio_base)
            servicio.tiempo_est = request.POST.get('tiempo_est', servicio.tiempo_est)
            servicio.aplica_garantia = 'aplica_garantia' in request.POST
            servicio.save()
            return redirect('ver_servicio')
        return redirect('ver_servicio')
    
    def borrar_servicio(request, servicio_id):
        servicio = get_object_or_404(Servicio, pk=servicio_id)
        if request.method == 'POST':
            servicio.delete()
            return redirect('ver_servicio')
        return render(request, 'servicio/borrar_servicio.html', {'servicio': servicio})

# 5. Modificar el archivo navbar.html que está dentro de templates, para actualizar la opción “Servicios” en submenu de Servicios(Agregar servicios, ver servicios).

    <nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm">
    <div class="container">
    <a class="navbar-brand" href="{% url 'inicio_taller' %}">🔧 Sistema de Administración Taller</a>
          <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMenu">
          <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navMenu">
              <ul class="navbar-nav ms-auto">
                  <li class="nav-item"><a class="nav-link" href="{% url 'inicio_taller' %}">Inicio</a></li>
                  <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" id="clientesDrop" role="button" data-bs-toggle="dropdown">
                  <i class="bi bi-people-fill"></i> Clientes
              </a>
              <ul class="dropdown-menu" aria-labelledby="clientesDrop">
                <li><a class="dropdown-item" href="{% url 'agregar_clientes' %}"><i class="bi bi-person-plus"></i> Agregar clientes</a></li>
                <li><a class="dropdown-item" href="{% url 'ver_clientes' %}"><i class="bi bi-eye"></i> Ver clientes</a></li>
              </ul>
            </li>
  
          <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="#" id="servDrop" role="button" data-bs-toggle="dropdown">
              <i class="bi bi-gear-fill"></i> Servicios
            </a>
            <ul class="dropdown-menu" aria-labelledby="servDrop">
              <li><a class="dropdown-item" href="{% url 'agregar_servicio' %}"><i class="bi bi-gear-fill"></i> Agregar servicio</a></li>
              <li><a class="dropdown-item" href="{% url 'ver_servicio' %}"><i class="bi bi-eye"></i> Ver servicios</a></li>
            </ul>
          </li>

          <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="#" id="vehDrop" role="button" data-bs-toggle="dropdown">
              <i class="bi bi-truck"></i> Vehículos
            </a>
            <ul class="dropdown-menu" aria-labelledby="vehDrop">
              <li><a class="dropdown-item" href="#"><i class="bi bi-truck"></i> Agregar vehículo</a></li>
              <li><a class="dropdown-item" href="#"><i class="bi bi-eye"></i> Ver vehículos</a></li>
            </ul>
          </li>
        </ul>
      </div>
    </div>
    </nav>
  
  <!-- Bootstrap Icons CDN (opcional para iconos) -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css">
    
# 6. Crear la subcarpeta carpeta servicio dentro de app_Taller\templates.
Dentro de app_Taller/templates, crea una nueva carpeta llamada servicio.
# 7. Crear los archivos html con su código correspondientes de (agregar_servicio.html, ver_servicio.html mostrar en tabla con los botones ver, editar y borrar, actualizar_servicio.html, borrar_servicio.html) dentro de app_Taller\templates\servicio.
# agregar_servicio.html
Html

        {% extends 'base.html' %}
    {% block title %}Agregar Servicio{% endblock %}
    {% block content %}
    <h2>Agregar Servicio</h2>
    
    <form method="post">
      {% csrf_token %}
    
      <div class="mb-3">
        <label class="form-label">Nombre del Servicio</label>
        <input type="text" name="nombre" class="form-control" required>
      </div>
    
      <div class="mb-3">
        <label class="form-label">Descripción</label>
        <textarea name="descripcion" class="form-control"></textarea>
      </div>
    
      <div class="mb-3">
        <label class="form-label">Precio Base ($)</label>
        <input type="number" step="0.01" name="precio_base" class="form-control" required>
      </div>
    
      <div class="mb-3">
        <label class="form-label">Tiempo Estimado (minutos)</label>
        <input type="number" name="tiempo_est" class="form-control" required>
      </div>
    
      <div class="form-check mb-3">
        <input class="form-check-input" type="checkbox" name="aplica_garantia" id="aplica_garantia">
        <label class="form-check-label" for="aplica_garantia">
          Aplica Garantía
        </label>
      </div>
    
      <button class="btn btn-primary">
        <i class="bi bi-save"></i> Guardar
      </button>
      <a href="{% url 'ver_servicio' %}" class="btn btn-secondary">
        <i class="bi bi-x-lg"></i> Cancelar
      </a>
    
    </form>
    {% endblock %}

# ver_servicio.html
Html

        {% extends 'base.html' %}
    {% block title %}Ver Servicios{% endblock %}
    {% block content %}
    <h2><i class="bi bi-gear-fill"></i> Lista de Servicios</h2>
    
    <a href="{% url 'agregar_servicio' %}" class="btn btn-success mb-3">
      <i class="bi bi-plus-lg"></i> Agregar servicio
    </a>
    
    <table class="table table-striped">
      <thead>
        <tr>
          <th>Nombre del Servicio</th>
          <th>Descripción</th>
          <th>Precio Base ($)</th>
          <th>Tiempo Estimado (min)</th>
          <th>Aplica Garantía</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        {% for s in servicios %}
        <tr>
          <td>{{ s.nombre }}</td>
          <td>{{ s.descripcion|truncatewords:10 }}</td>
          <td>{{ s.precio_base }}</td>
          <td>{{ s.tiempo_est }}</td>
          <td>{% if s.aplica_garantia %}Sí{% else %}No{% endif %}</td>
          <td>
            <a class="btn btn-sm btn-info" href="{% url 'actualizar_servicio' s.id %}">
              <i class="bi bi-pencil"></i> Editar
            </a>
            <a class="btn btn-sm btn-danger" href="{% url 'borrar_servicio' s.id %}">
              <i class="bi bi-trash"></i> Borrar
            </a>
          </td>
        </tr>
        {% empty %}
        <tr>
          <td colspan="6">No hay servicios registrados.</td>
        </tr>
        {% endfor %}
      </tbody>
    </table>
    {% endblock %}

# actualizar_servicio.html
Html

       {% extends 'base.html' %}
    {% block title %}Actualizar Servicio{% endblock %}
    {% block content %}
    <h2>Editar Servicio</h2>
    
    <form method="post" action="{% url 'realizar_actualizacion_servicio' servicio.id %}">
      {% csrf_token %}
      
      <div class="mb-3">
        <label class="form-label">Nombre del Servicio</label>
        <input type="text" name="nombre" value="{{ servicio.nombre }}" class="form-control" required>
      </div>
    
      <div class="mb-3">
        <label class="form-label">Descripción</label>
        <textarea name="descripcion" class="form-control">{{ servicio.descripcion }}</textarea>
      </div>
    
      <div class="mb-3">
        <label class="form-label">Precio Base ($)</label>
        <input type="number" step="0.01" name="precio_base" value="{{ servicio.precio_base }}" class="form-control" required>
      </div>
    
      <div class="mb-3">
        <label class="form-label">Tiempo Estimado (minutos)</label>
        <input type="number" name="tiempo_est" value="{{ servicio.tiempo_est }}" class="form-control" required>
      </div>
    
      <div class="form-check mb-3">
        <input class="form-check-input" type="checkbox" name="aplica_garantia" id="aplica_garantia"
               {% if servicio.aplica_garantia %}checked{% endif %}>
        <label class="form-check-label" for="aplica_garantia">
          Aplica Garantía
        </label>
      </div>
    
      <button class="btn btn-primary">
        <i class="bi bi-pencil-square"></i> Actualizar
      </button>
      <a href="{% url 'ver_servicio' %}" class="btn btn-secondary">
        <i class="bi bi-x-lg"></i> Cancelar
      </a>
    </form>
    
    {% endblock %}

# borrar_servicio.html
Html

        {% extends 'base.html' %}
    {% block title %}Borrar Servicio{% endblock %}
    {% block content %}
    <h2>Confirmar borrado</h2>
    <p>¿Deseas eliminar el servicio <strong>{{ servicio.nombre }}</strong>?</p>
    <form method="post">
      {% csrf_token %}
      <button class="btn btn-danger">
        <i class="bi bi-trash-fill"></i> Sí, borrar
      </button>
      <a href="{% url 'ver_servicio' %}" class="btn btn-secondary">
        <i class="bi bi-x-lg"></i> Cancelar
      </a>
    </form>
    {% endblock %}
    
# 8. No utilizar forms.py.
Hemos mantenido la implementación de formularios directamente en las vistas con request.POST, sin usar el módulo forms de Django.
# 9. Procedimiento para agregar en urls.py en app_Taller con el código correspondiente para acceder a las funciones de views.py para operaciones de crud en servicios.
Abre el archivo app_Taller/urls.py y añade las URLs para los servicios:
Python
# app_Taller/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
    path('', views.inicio_taller, name='inicio_taller'),
    path('clientes/agregar/', views.agregar_clientes, name='agregar_clientes'),
    path('clientes/', views.ver_clientes, name='ver_clientes'),
    path('clientes/editar/<int:cliente_id>/', views.actualizar_clientes, name='actualizar_clientes'),
    path('clientes/editar/guardar/<int:cliente_id>/', views.realizar_actualizacion_clientes, name='realizar_actualizacion_clientes'),
    path('clientes/borrar/<int:cliente_id>/', views.borrar_clientes, name='borrar_clientes'),

    # URLs para Servicios
    path('servicios/', views.ver_servicio, name='ver_servicio'),
    path('servicios/agregar/', views.agregar_servicio, name='agregar_servicio'),
    path('servicios/actualizar/<int:servicio_id>/', views.actualizar_servicio, name='actualizar_servicio'),
    path('servicios/actualizar/confirmar/<int:servicio_id>/', views.realizar_actualizacion_servicio, name='realizar_actualizacion_servicio'),
    path('servicios/borrar/<int:servicio_id>/', views.borrar_servicio, name='borrar_servicio'),
    ]
# 10. Procedimiento para registrar los modelos en admin.py y volver a realizar las migraciones.
Abre el archivo app_Taller/admin.py y ahora registra también el modelo Servicio:
Python
    app_Taller/admin.py
    
    from django.contrib import admin
    from .models import Cliente, Servicio, Vehiculo
    
    
    @admin.register(Cliente)
    class ClienteAdmin(admin.ModelAdmin):
        list_display = ('nombre', 'apellido', 'email', 'telefono', 'rfc', 'fecha_registro')
        search_fields = ('nombre', 'apellido', 'email', 'rfc')
    
    
    @admin.register(Servicio)
    class ServicioAdmin(admin.ModelAdmin):
        list_display = ('nombre', 'precio_base', 'tiempo_est', 'aplica_garantia', 'fecha_actualizacion')
        search_fields = ('nombre',)
    
    # Dejamos comentados por ahora, según la instrucción 11
    @admin.register(Vehiculo)
    class VehiculoAdmin(admin.ModelAdmin):
        list_display = ('matricula', 'marca', 'modelo', 'anio', 'cliente')
        search_fields = ('matricula', 'marca', 'modelo')
    
Luego, ejecuta las migraciones para asegurarte de que admin.py se actualice (aunque Servicio ya debería estar migrado, esto es para que Django reconozca el registro en el panel de administración):
Bash

    python manage.py makemigrations
    
    python manage.py migrate
    
# 11. Por lo pronto solo trabajar con “servicios” dejar pendiente # MODELO: VEHICULO
Hemos seguido esta instrucción, dejando el modelo Vehiculo y sus funcionalidades para más adelante.
# 12. Utilizar colores suaves, atractivos y modernos, el código de las páginas web sencillas para servicios.
Se ha mantenido el uso de Bootstrap 5 para el diseño de las páginas de servicios, utilizando clases como bg-success para "Agregar Servicio" y bg-primary para la lista, para una interfaz coherente y visualmente agradable.
# 13. No validar entrada de datos.
Al igual que con los clientes, no se ha implementado validación de datos a nivel de Django para los servicios, más allá de los atributos required en el HTML.
# 14. Al inicio crear la estructura completa de carpetas y archivos actualizados donde se muestra la carpeta servicios dentro de templates con los archivos html correspondientes a las operaciones crud servicios.
La estructura de tu proyecto ahora debería verse así (lo nuevo resaltado):
Code

    UIII_Taller_Mecanico_0446/
    ├── .venv/
    ├── backend_Taller/
    │   ├── __init__.py
    │   ├── asgi.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    ├── app_Taller/
    │   ├── migrations/
    │   ├── __init__.py
    │   ├── admin.py
    │   ├── apps.py
    │   ├── models.py
    │   ├── tests.py
    │   ├── views.py
    │   ├── urls.py
    │   └── templates/
    │       ├── base.html
    │       ├── header.html
    │       ├── navbar.html  <-- Modificado
    │       ├── footer.html
    │       ├── inicio.html
    │       ├── cliente/
    │       │   ├── agregar_cliente.html
    │       │   ├── ver_cliente.html
    │       │   ├── actualizar_cliente.html
    │       │   └── borrar_cliente.html
    │       └── servicio/  <-- Nueva carpeta
    │           ├── agregar_servicio.html
    │           ├── ver_servicio.html
    │           ├── actualizar_servicio.html
    │           └── borrar_servicio.html
    ├── manage.py
    └── db.sqlite3
# 15. Proyecto totalmente funcional.
Ahora el proyecto es totalmente funcional para la gestión CRUD de Clientes y Servicios.
# 16. Finalmente ejecutar servidor en el puerto puerto 8446.
Para ejecutar el servidor:
Abre la terminal en VS Code.
Activa tu entorno virtual si no está activo.
Navega a la raíz del proyecto (donde está manage.py).
Ejecuta:
Bash

    python manage.py runserver 8446
    
Ahora puedes acceder a la aplicación en http://127.0.0.1:8446/ y probar las funcionalidades de Clientes y Servicios a través de la barra de navegación.
