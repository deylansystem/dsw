Aquí tienes la guía formateada de forma profesional utilizando **Markdown**. He organizado los comandos en bloques de código, utilizado una jerarquía de títulos clara y añadido notas importantes para que el proceso sea fácil de seguir.

---

# Guía de Configuración: Laravel + PostgreSQL

## Tarea 1: Descarga e Instalación de PostgreSQL

### Instalación del sistema

```bash
sudo apt update
sudo apt install postgresql

```

### Administración de PostgreSQL

Accede a la consola de administración de PostgreSQL:

```bash
sudo -u postgres psql

```

Cambia la contraseña del usuario `postgres`:

```sql
ALTER USER postgres PASSWORD '123456';

```

---

## Tarea 2: Creación de la Base de Datos y Carga de Datos

### Preparación de la Base de Datos

```sql
CREATE DATABASE tienda OWNER postgres;
\c tienda

```

### Creación de Tablas

```sql
CREATE TABLE fabricante (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE producto (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio FLOAT NOT NULL,
    id_fabricante INTEGER NOT NULL,
    FOREIGN KEY (id_fabricante) REFERENCES fabricante(id)
);

```

### Inserción de Datos

```sql
INSERT INTO fabricante VALUES(1, 'Asus'), (2, 'Lenovo'), (3, 'Hewlett-Packard'), (4, 'Samsung'), (5, 'Seagate'), (6, 'Crucial'), (7, 'Gigabyte'), (8, 'Huawei'), (9, 'Xiaomi');

INSERT INTO producto VALUES(1, 'Disco duro SATA3 1TB', 86.99, 5), (2, 'Memoria RAM DDR4 8GB', 120, 6), (3, 'Disco SSD 1 TB', 150.99, 4), (4, 'GeForce GTX 1050Ti', 185, 7), (5, 'GeForce GTX 1080 Xtreme', 755, 6), (6, 'Monitor 24 LED Full HD', 202, 1), (7, 'Monitor 27 LED Full HD', 245.99, 1), (8, 'Portátil Yoga 520', 559, 2), (9, 'Portátil Ideapd 320', 444, 2), (10, 'Impresora HP Deskjet 3720', 59.99, 3), (11, 'Impresora HP Laserjet Pro M26nw', 180, 3);

```

### Verificación y Salida

```sql
SELECT * FROM producto;
\q

```

---

## Tarea 3: Configuración del Entorno PHP y Composer

### Extensión de PostgreSQL para PHP

```bash
sudo apt-get install php8.3-pdo-pgsql

```

Edita el archivo de configuración para habilitar la extensión:

```bash
sudo nano /etc/php/8.3/fpm/php.ini

```

*Busca la línea `;extension=pdo_pgsql` y quita el punto y coma (`;`).*

Reinicia el servicio:

```bash
sudo systemctl restart nginx

```

### Instalación de Composer y Laravel

```bash
sudo apt install composer
composer --version

# Instalador de Laravel
composer global require laravel/installer

```

### Configuración del Path (Variable de Entorno)

```bash
nano ~/.bashrc

```

Añade al final: `export PATH="$PATH:$HOME/.config/composer/vendor/bin"`

```bash
source ~/.bashrc
laravel --version

```

---

## Tarea 4: Creación y Configuración del Proyecto Laravel

### Nuevo Proyecto y Dependencias

```bash
laravel new GestorProductos
cd GestorProductos

# Extensiones necesarias
sudo apt update
sudo apt install php8.3-xml php8.3-curl php8.3-mbstring php8.3-sqlite3 php8.3-mysql php8.3-bcmath

# Instalar dependencias del proyecto
composer install

```

### Configuración del archivo `.env`

Edita el archivo `.env` en la raíz del proyecto:

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=tienda
DB_USERNAME=postgres
DB_PASSWORD=123456

```

---

## Tarea 5: Desarrollo de Componentes (MVC)

### 1. El Modelo

**Comando:** `php artisan make:model ModeloProductos`

**Ubicación:** `app/Models/ModeloProductos.php`

```php
protected $table = 'producto';

```

### 2. El Controlador

**Comando:** `php artisan make:controller ControladorProductos`

**Ubicación:** `app/Http/Controllers/ControladorProductos.php`

```php
use App\Models\ModeloProductos;

public function MuestraProductos() {
    $productos = ModeloProductos::all();
    return view('VistaProductos', ['productos' => $productos]);
}

```

### 3. La Vista

**Comando:** `php artisan make:view VistaProductos`

**Ubicación:** `resources/views/VistaProductos.blade.php`

```html
<html>
    <body style="font-family: sans-serif; padding: 20px;">
        <h1 style="font-size: 30px;">Listado de productos</h1>
        <ul>
            @if($productos->isEmpty())
                <p>No hay productos.</p>
            @else
                @foreach ($productos as $producto)
                    <li>Nombre: {{ $producto->nombre }}, Precio: ${{ number_format($producto->precio, 2) }}</li>
                @endforeach
            @endif
        </ul>
    </body>
</html>

```

### 4. Rutas

**Archivo:** `routes/web.php`

```php
use App\Http\Controllers\ControladorProductos;

Route::get('/', [ControladorProductos::class, 'MuestraProductos'])->name('Productos');

```

---

## Tarea 6: Ejecución

```bash
php artisan migrate
php artisan serve

```

¿Necesitas ayuda para personalizar el diseño de la vista o para añadir alguna funcionalidad de búsqueda a este listado?