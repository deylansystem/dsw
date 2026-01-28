# API REST en PHP con PDO y PostgreSQL

Este documento explica **por partes** el funcionamiento de una API REST escrita en PHP, pensada como **chuleta de examen** y material de referencia.

---

## 1. Cabecera de la respuesta

```php
header('Content-Type: application/json');
```

### Funcionalidad

* Indica que **todas las respuestas** de la API se devuelven en formato **JSON**.
* Es obligatorio en una API REST para que el cliente (Postman, frontend, etc.) interprete correctamente la respuesta.

---

## 2. Configuración de conexión a la base de datos

```php
$dsn = 'pgsql:host=localhost;dbname=tienda';
$user = 'postgres';
$password = '123456';
$token_valido = "mi_token_secreto_123";
```

### Funcionalidad

* `$dsn`: define el motor (PostgreSQL), el host y la base de datos.
* `$user` y `$password`: credenciales de acceso.
* `$token_valido`: token de seguridad que se exigirá en métodos sensibles (POST, PUT, DELETE).

---

## 3. Creación de la conexión con PDO

```php
$conn = new PDO($dsn, $user, $password);
$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
$conn->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
```

### Funcionalidad

* Se crea una conexión segura con **PDO**.
* `ERRMODE_EXCEPTION`: los errores se capturan con `try/catch`.
* `FETCH_ASSOC`: los resultados se devuelven como arrays asociativos.

### Control de errores

```php
catch (PDOException $e) {
    http_response_code(500);
    echo json_encode(['error' => 'Error de conexión']);
    exit;
}
```

* Si falla la conexión → **HTTP 500 (Error del servidor)**.

---

## 4. Detección del método HTTP

```php
$metodo = $_SERVER['REQUEST_METHOD'];
```

### Funcionalidad

* Detecta el verbo HTTP usado por el cliente:

  * GET
  * POST
  * PUT
  * DELETE

Se usa para aplicar la lógica REST correcta.

---

## 5. Estructura principal: `switch` por método HTTP

```php
switch ($metodo) {
```
```php
    case 'GET':
// (Tu código anterior de lectura aquí...)
        $id = $_GET['id'] ?? null;
        if ($id) {
            $stmt = $conn->prepare("SELECT * FROM producto WHERE id = ?");
            $stmt->execute([$id]);
            $response = $stmt->fetch() ?: ['error' => 'No encontrado'];
        } else {
            $response = $conn->query("SELECT * FROM producto")->fetchAll();
        }
        break;
```
```php
    case 'POST':
 // 1. Validar seguridad
        validarToken($token_valido);

        // 2. Recoger datos (siguiendo tu lógica de usar $_GET o parámetros de URL)
        $id = $_GET['id'] ?? null;
        $nombre = $_GET['nombre'] ?? null;
        $precio = $_GET['precio'] ?? null;
        $fabricante = $_GET['id_fabricante'] ?? null;

        // 3. Validar que no falten datos (campos NOT NULL en la BD)
        if (!$id || !$nombre || !$precio || !$fabricante) {
            enviarError(400, "Faltan datos para crear el producto (id, nombre, precio, id_fabricante)");
        }

        try {
            // 4. Preparar la sentencia de inserción
            $sql = "INSERT INTO producto (id, nombre, precio, id_fabricante) VALUES (:id, :nombre, :precio, :id_fabricante)";
            $stmt = $conn->prepare($sql);
            
            // 5. Ejecutar
            $stmt->execute([
                ':id' => $id,
                ':nombre' => $nombre,
                ':precio' => $precio,
                ':id_fabricante' => $fabricante
            ]);

            // 6. Responder con éxito y el código 201 (Created)
            http_response_code(201);
            $response = [
                'mensaje' => "Producto creado con éxito",
                'id_generado' => $id // Nos dice qué ID le puso la base de datos
            ];
        } catch (PDOException $e) {
            enviarError(500, "Error al insertar en la base de datos: " . $e->getMessage());
        }
        break;
```
```php
    case 'PUT':
// --- TAREA EVALUABLE: ACTUALIZACIÓN ---
        validarToken($token_valido);
        $id = $_GET['id'] ?? null;
        
        // Obtenemos los campos del cuerpo de la petición (Body)
        // En PUT, Postman enviará nombre, precio e id_fabricante
        $nombre = $_GET['nombre'] ?? null;
        $precio = $_GET['precio'] ?? null;
        $fabricante = $_GET['id_fabricante'] ?? null;

        if (!$id || !$nombre || !$precio || !$fabricante) {
            enviarError(400, "Faltan datos obligatorios (id, nombre, precio, id_fabricante)");
        }

        $sql = "UPDATE producto SET nombre = :nombre, precio = :precio, id_fabricante = :id_fabricante WHERE id = :id";
        $stmt = $conn->prepare($sql);
        $stmt->execute([
            ':nombre' => $nombre,
            ':precio' => $precio,
            ':id_fabricante' => $fabricante,
            ':id' => $id
        ]);

        if ($stmt->rowCount() > 0) {
            $response = ['mensaje' => "Producto $id actualizado correctamente"];
        } else {
            enviarError(404, "No se realizaron cambios o el ID no existe");
        }
        break;
```
```php
    case 'DELETE':
// --- TAREA EVALUABLE: ELIMINACIÓN ---
        validarToken($token_valido);
        $id = $_GET['id'] ?? null;

        if (!$id) {
            enviarError(400, "Falta el ID para eliminar");
        }

        $stmt = $conn->prepare("DELETE FROM producto WHERE id = ?");
        $stmt->execute([$id]);

        if ($stmt->rowCount() > 0) {
            $response = ['mensaje' => "Producto $id eliminado con éxito"];
        } else {
            enviarError(404, "No existe el producto con ID $id");
        }
        break;
}
```

### Funcionalidad

* Cada **verbo HTTP** representa una operación CRUD:

| Método | Operación | Acción              |
| ------ | --------- | ------------------- |
| GET    | READ      | Leer productos      |
| POST   | CREATE    | Crear producto      |
| PUT    | UPDATE    | Actualizar producto |
| DELETE | DELETE    | Eliminar producto   |

---

## 6. Método GET – Lectura de datos

```php
$id = $_GET['id'] ?? null;
```

### Funcionalidad

* Si se pasa un `id` → devuelve **un solo producto**.
* Si no → devuelve **todos los productos**.

```php
if ($id) {
    SELECT * FROM producto WHERE id = ?
} else {
    SELECT * FROM producto
}
```

### Características

* No requiere token (lectura pública).
* Usa **consultas preparadas** → más seguridad.

---

## 7. Método DELETE – Eliminación de un producto

```php
validarToken($token_valido);
```

### Funcionalidad

* Protege la operación con **token**.

```php
$id = $_GET['id'] ?? null;
```

* El ID es obligatorio.
* Si no existe → error 404.

```php
DELETE FROM producto WHERE id = ?
```

### Respuestas posibles

* ✅ 200 → producto eliminado.
* ❌ 400 → falta el ID.
* ❌ 404 → producto inexistente.
* ❌ 401 → token inválido.

---

## 8. Método POST – Creación de un producto

### Flujo completo

1. Validar token
2. Recoger datos
3. Validar campos obligatorios
4. Insertar en la base de datos
5. Responder con código 201

```php
INSERT INTO producto (id, nombre, precio, id_fabricante)
```

### Validaciones

* Todos los campos son obligatorios (NOT NULL).
* Si falta alguno → **HTTP 400**.

### Respuesta correcta

```json
{
  "mensaje": "Producto creado con éxito",
  "id_generado": 5
}
```

---

## 9. Método PUT – Actualización de un producto

### Funcionalidad

* Requiere token.
* Actualiza un producto existente.

```php
UPDATE producto SET nombre, precio, id_fabricante WHERE id
```

### Comprobaciones

* Si no hay cambios o el ID no existe → **404**.
* Si todo va bien → mensaje de éxito.

---

## 10. Respuesta final de la API

```php
echo json_encode($response, JSON_PRETTY_PRINT);
```

### Funcionalidad

* Devuelve la respuesta en JSON bien formateado.
* Facilita la lectura en Postman y en exámenes.

---

## 11. Función `validarToken()`

```php
$headers = getallheaders();
$token_recibido = $headers['Authorization'] ?? '';
```

### Funcionalidad

* Busca el token en el **header Authorization**.
* Si no coincide → **HTTP 401 No autorizado**.

---

## 12. Función `enviarError()`

```php
http_response_code($codigo);
echo json_encode(['error' => $mensaje]);
exit;
```

### Funcionalidad

* Centraliza los errores.
* Detiene la ejecución.
* Devuelve siempre JSON.

---

## 13. Resumen para examen

* API REST basada en **CRUD**
* PDO + PostgreSQL
* Seguridad con token
* Uso correcto de códigos HTTP
* Consultas preparadas
* Respuestas JSON

---

✔ Documento ideal como **chuleta de examen** y referencia rápida.
