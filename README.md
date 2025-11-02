# Guía para Construir una Aplicación de Login en PHP con Patrón MVC

Esta guía explica paso a paso cómo construir desde cero una aplicación web de login y registro en PHP utilizando el patrón de diseño MVC (Modelo-Vista-Controlador). El enfoque MVC separa la lógica de negocio, la presentación y el control de la aplicación para un mejor mantenimiento y escalabilidad. Este pequeño aplicativo seguirá ampliándose designándose por partes para ir viendo cómo evoluciona, la siguiente parte, la 2, reutiliza este aplicativo de inicio de sesión y registro de usuarios, para permitir la gestión de tutorías. 

## ¿Qué es MVC?

- **Modelo (Model)**: Maneja los datos y la lógica de negocio. Interactúa con la base de datos.
- **Vista (View)**: Se encarga de la presentación. Muestra la información al usuario.
- **Controlador (Controller)**: Gestiona las solicitudes del usuario, coordina entre modelo y vista.

## Paso 1: Preparar el Entorno

1. Crea una carpeta para tu proyecto, por ejemplo `loginApp`.
2. Dentro de la carpeta, crea las subcarpetas: `models`, `views`, `controllers`.
3. Asegúrate de tener un servidor web (como WAMP o XAMPP) y una base de datos MySQL.

## Paso 2: Crear la Base de Datos

Ejecuta esta consulta SQL en tu base de datos para crear la tabla de usuarios:

```sql
CREATE DATABASE bdnegocio;

USE bdnegocio;

CREATE TABLE usuarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE,
    pass VARCHAR(255) NOT NULL
);
```

## Paso 3: Crear el Archivo de Configuración (config.php)

Este archivo establece la conexión a la base de datos.

```php
<?php
// Configuración de la base de datos
$servername = "localhost";
$username = "root";
$password = "";
$database = "bdnegocio";

try {
    $con = new PDO("mysql:host=$servername;dbname=$database", $username, $password);
    $con->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch(PDOException $e) {
    die("Error de conexión: " . $e->getMessage());
}
?>
```

### Explicación del archivo config.php

- **Palabras reservadas y bloques:**
  - `try`: Inicia un bloque try-catch para manejar excepciones. El código dentro de try se ejecuta, y si ocurre una excepción, se captura en catch.
  - `catch(PDOException $e)`: Captura excepciones de tipo PDOException. PDOException es una clase que representa errores de PDO.
  - `die()`: Función que termina la ejecución del script y muestra un mensaje de error.
  - `$con = new PDO(...)`: Crea una nueva instancia de la clase PDO para conectar a la base de datos MySQL.
  - `$con->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION)`: Configura PDO para que lance excepciones en caso de error, en lugar de solo devolver códigos de error.

- **Por qué se usa PDO::FETCH_ASSOC en otros archivos (como UserModel.php):**
  - `PDO::FETCH_ASSOC` es una constante de PDO que especifica el formato de los datos devueltos por `fetch()`. Devuelve un array asociativo donde las claves son los nombres de las columnas de la base de datos, facilitando el acceso a los datos por nombre en lugar de índices numéricos.

### Explicación del archivo models/UserModel.php

- **Palabras reservadas y bloques:**
  - `class UserModel`: Define una clase llamada UserModel que maneja operaciones relacionadas con usuarios.
  - `global $con;`: Permite acceder a la variable global $con definida en config.php para usar la conexión PDO.
  - `public function __construct()`: Método constructor que inicializa la conexión a la base de datos.
  - `public function authenticate($nombre, $pass)`: Método que verifica si un usuario existe y si la contraseña es correcta.
  - `public function register($nombre, $pass)`: Método que registra un nuevo usuario en la base de datos.

- **Explicación de métodos clave:**

  - `prepare()`: Prepara una sentencia SQL para ejecución segura con parámetros, previniendo inyección SQL.
  - `bindParam()`: Vincula un parámetro SQL a una variable PHP para sanitizar la entrada.
  - `execute()`: Ejecuta la sentencia preparada.
  - `fetch(PDO::FETCH_ASSOC)`: Obtiene una fila del conjunto de resultados como un array asociativo.
  - `rowCount()`: Devuelve el número de filas afectadas por la última sentencia SQL.
  - `password_hash()`: Crea un hash seguro para almacenar contraseñas.
  - `password_verify()`: Verifica que una contraseña coincida con un hash almacenado.

¿Qué significa "hashear" en el contexto de UserModel.php?
"Hashear" (o "hashing") es el proceso de transformar una contraseña en una cadena de caracteres irreversible mediante un algoritmo matemático. Esto significa que la contraseña original no puede ser recuperada del hash, pero se puede verificar si una contraseña dada coincide con el hash almacenado usando funciones como `password_verify()`. En UserModel.php, se usa `password_hash()` para hashear la contraseña antes de almacenarla en la base de datos, y `password_verify()` para verificarla durante el login. Esto mejora la seguridad al evitar almacenar contraseñas en texto plano, protegiendo contra ataques si la base de datos es comprometida.

- **Flujo en authenticate($nombre, $pass):**
  1. Prepara y ejecuta una consulta para obtener el usuario por nombre.
  2. Obtiene el resultado como array asociativo.
  3. Verifica que el usuario exista y que la contraseña coincida usando password_verify.
  4. Retorna true si la autenticación es exitosa, false en caso contrario.

- **Flujo en register($nombre, $pass):**
  1. Verifica si el usuario ya existe en la base de datos.
  2. Si existe, retorna false.
  3. Si no, hashea la contraseña con password_hash.
  4. Inserta el nuevo usuario en la base de datos.
  5. Retorna true si el registro es exitoso.

### Explicación del archivo controllers/AuthController.php

- **Palabras reservadas y bloques:**
  - `class AuthController`: Define una clase llamada AuthController que maneja la lógica de autenticación.
  - `private $userModel;`: Propiedad privada que almacena una instancia de UserModel.
  - `public function __construct()`: Método constructor que crea una instancia de UserModel.
  - `public function login()`: Método que maneja el proceso de inicio de sesión.
  - `public function register()`: Método que maneja el proceso de registro de usuarios.
  - `public function logout()`: Método que maneja el cierre de sesión.

- **Explicación de métodos clave:**

  - `session_start()`: Inicia o reanuda una sesión PHP para manejar variables de sesión.
  - `$_SERVER["REQUEST_METHOD"] == "POST"`: Verifica si la solicitud HTTP es de tipo POST.
  - `$_POST['nombre']` y `$_POST['pass']`: Obtiene los datos enviados desde formularios.
  - `$this->userModel->authenticate($nombre, $pass)`: Llama al método authenticate para validar credenciales.
  - `$_SESSION['nombre'] = $nombre;`: Almacena el nombre de usuario en la sesión.
  - `header("Location: index.php?action=home")`: Redirige al usuario a la página principal.
  - `exit()`: Termina la ejecución del script para evitar código adicional.
  - `include('views/login.php')` y `include('views/register.php')`: Incluye las vistas correspondientes para mostrar formularios o mensajes.
  - `$this->userModel->register($nombre, $pass)`: Llama al método register para crear un nuevo usuario.
  - `session_destroy()`: Destruye la sesión actual para cerrar sesión.

- **Flujo en login():**
  1. Inicia la sesión.
  2. Si la solicitud es POST, obtiene nombre y contraseña del formulario.
  3. Llama a authenticate para validar credenciales.
  4. Si es válido, guarda el nombre en la sesión y redirige a home.
  5. Si no, muestra el formulario de login con un mensaje de error.
  6. Si la solicitud no es POST, muestra el formulario de login.

- **Flujo en register():**
  1. Inicia la sesión.
  2. Si la solicitud es POST, obtiene nombre y contraseña del formulario.
  3. Llama a register para crear un nuevo usuario.
  4. Si es exitoso, muestra mensaje de éxito en la vista de registro.
  5. Si no, muestra mensaje de error en la vista de registro.
  6. Si la solicitud no es POST, muestra el formulario de registro.

- **Flujo en logout():**
  1. Inicia la sesión.
  2. Destruye la sesión.
  3. Redirige a la página de login.

### Explicación del archivo controllers/HomeController.php

- **Palabras reservadas y bloques:**
  - `class HomeController`: Define una clase llamada HomeController que maneja la lógica de la página principal.
  - `public function index()`: Método que maneja la solicitud para la página de inicio.

- **Explicación de métodos clave:**

  - `session_start()`: Inicia o reanuda una sesión PHP para acceder a variables de sesión.
  - `isset($_SESSION['nombre'])`: Verifica si la variable 'nombre' existe en el array superglobal $_SESSION.
  - `header("Location: index.php?action=login")`: Redirige al usuario a la página de login si no está autenticado.
  - `exit()`: Termina la ejecución del script para evitar código adicional.
  - `include('views/home.php')`: Incluye la vista de la página principal para mostrar el contenido.

- **Flujo en index():**
  1. Inicia la sesión.
  2. Verifica si el usuario está autenticado (si 'nombre' existe en la sesión).
  3. Si no está autenticado, redirige a la página de login.
  4. Si está autenticado, incluye la vista home.php para mostrar la página principal.

### Explicación del archivo views/login.php

- **Palabras reservadas y bloques:**
  - `<!DOCTYPE html>`: Declara el tipo de documento HTML5.
  - `<html>`: Elemento raíz del documento HTML.
  - `<head>`: Contiene metadatos del documento, como el título.
  - `<title>`: Define el título de la página que se muestra en la pestaña del navegador.
  - `<body>`: Contiene el contenido visible de la página.
  - `<h2>`: Encabezado de nivel 2 para el título de la página.
  - `<form>`: Define un formulario HTML para enviar datos al servidor.
  - `<label>`: Etiqueta asociada a un campo de entrada para describirlo.
  - `<input>`: Campo de entrada para texto o contraseña.
  - `<br>`: Salto de línea.
  - `<p>`: Párrafo de texto.
  - `<?php if (isset($error)) { echo "<p>$error</p>"; } ?>`: Bloque PHP que verifica si existe una variable $error y la muestra si es así.

- **Explicación de elementos clave:**

  - `method="post"`: Especifica que el formulario envía datos mediante el método POST.
  - `action="index.php?action=login"`: Define la URL a la que se envían los datos del formulario.
  - `type="text"` y `type="password"`: Tipos de entrada para nombre de usuario y contraseña.
  - `required`: Atributo que hace obligatorio el campo.
  - `type="submit"`: Botón para enviar el formulario.
  - `isset($error)`: Función PHP que verifica si la variable $error está definida.
  - `echo`: Función PHP que imprime el contenido de la variable.

- **Flujo en login.php:**
  1. Muestra la estructura básica HTML de la página de login.
  2. Renderiza el formulario con campos para nombre de usuario y contraseña.
  3. Incluye un botón para enviar el formulario.
  4. Si existe un mensaje de error (variable $error), lo muestra en un párrafo.
  5. Proporciona un enlace para ir a la página de registro.

### Explicación del archivo views/register.php

- **Palabras reservadas y bloques:**
  - `<!DOCTYPE html>`: Declara el tipo de documento HTML5.
  - `<html>`: Elemento raíz del documento HTML.
  - `<head>`: Contiene metadatos del documento, como el título.
  - `<title>`: Define el título de la página que se muestra en la pestaña del navegador.
  - `<body>`: Contiene el contenido visible de la página.
  - `<h2>`: Encabezado de nivel 2 para el título de la página.
  - `<form>`: Define un formulario HTML para enviar datos al servidor.
  - `<label>`: Etiqueta asociada a un campo de entrada para describirlo.
  - `<input>`: Campo de entrada para texto o contraseña.
  - `<br>`: Salto de línea.
  - `<p>`: Párrafo de texto.
  - `<?php if (isset($error)) { echo "<p>$error</p>"; } ?>`: Bloque PHP que verifica si existe una variable $error y la muestra si es así.
  - `<?php if (isset($success)) { echo "<p>$success</p>"; } ?>`: Bloque PHP que verifica si existe una variable $success y la muestra si es así.

- **Explicación de elementos clave:**

  - `method="post"`: Especifica que el formulario envía datos mediante el método POST.
  - `action="index.php?action=register"`: Define la URL a la que se envían los datos del formulario.
  - `type="text"` y `type="password"`: Tipos de entrada para nombre de usuario y contraseña.
  - `required`: Atributo que hace obligatorio el campo.
  - `type="submit"`: Botón para enviar el formulario.
  - `isset($error)` y `isset($success)`: Funciones PHP que verifican si las variables $error o $success están definidas.
  - `echo`: Función PHP que imprime el contenido de la variable.

- **Flujo en register.php:**
  1. Muestra la estructura básica HTML de la página de registro.
  2. Renderiza el formulario con campos para nombre de usuario y contraseña.
  3. Incluye un botón para enviar el formulario.
  4. Si existe un mensaje de error (variable $error), lo muestra en un párrafo.
  5. Si existe un mensaje de éxito (variable $success), lo muestra en un párrafo.

### Explicación del archivo views/home.php

- **Palabras reservadas y bloques:**
  - `<!DOCTYPE html>`: Declara el tipo de documento HTML5.
  - `<html>`: Elemento raíz del documento HTML.
  - `<head>`: Contiene metadatos del documento, como el título.
  - `<title>`: Define el título de la página que se muestra en la pestaña del navegador.
  - `<body>`: Contiene el contenido visible de la página.
  - `<h2>`: Encabezado de nivel 2 para el título de la página.
  - `<a>`: Elemento de enlace para navegar a otra página.
  - `<?php echo $nombre; ?>`: Bloque PHP que imprime el valor de la variable $nombre.

- **Explicación de elementos clave:**

  - `href="index.php?action=logout"`: Atributo que especifica la URL a la que se dirige el enlace.
  - `echo`: Función PHP que imprime el contenido de la variable.
  - `$nombre`: Variable que contiene el nombre del usuario autenticado, obtenida de la sesión.

- **Flujo en home.php:**
  1. Muestra la estructura básica HTML de la página principal.
  2. Renderiza un mensaje de bienvenida personalizado con el nombre del usuario.
  3. Proporciona un enlace para cerrar sesión.

### Explicación del archivo index.php

- **Palabras reservadas y bloques:**
  - `<?php`: Etiqueta de apertura para código PHP.
  - `require_once()`: Función que incluye un archivo PHP solo una vez para evitar inclusiones múltiples.
  - `$action = isset($_GET['action']) ? $_GET['action'] : 'home';`: Operador ternario que asigna 'home' si no se especifica 'action'.
  - `switch ($action)`: Estructura de control switch que evalúa la variable $action.
  - `case 'login':`: Caso específico para la acción 'login'.
  - `$controller = new AuthController();`: Crea una nueva instancia de la clase AuthController.
  - `$controller->login();`: Llama al método login() del controlador.
  - `break;`: Termina la ejecución del caso actual.
  - `default:`: Caso por defecto si no coincide ningún case.
  - `?>`: Etiqueta de cierre para código PHP.

- **Explicación de elementos clave:**

  - `$_GET['action']`: Array superglobal que contiene parámetros enviados por método GET.
  - `isset()`: Función que verifica si una variable está definida.
  - `new`: Palabra clave para crear una instancia de una clase.
  - `AuthController()` y `HomeController()`: Constructores de las clases de controladores.
  - `login()`, `register()`, `logout()`, `index()`: Métodos de los controladores que manejan las acciones específicas.

- **Flujo en index.php:**
  1. Incluye los archivos de controladores necesarios.
  2. Obtiene el parámetro 'action' de la URL o asigna 'home' por defecto.
  3. Usa un switch para determinar qué controlador instanciar y qué método llamar basado en la acción.
  4. Instancia el controlador apropiado y ejecuta el método correspondiente.

## Paso 4: Crear el Modelo (models/UserModel.php)

El modelo maneja las operaciones de base de datos para usuarios.

```php
<?php
require_once('config.php');

class UserModel {
    private $conn;

    public function __construct() {
        global $con;
        $this->conn = $con;
    }

    public function authenticate($nombre, $pass) {
        $stmt = $this->conn->prepare("SELECT nombre, pass FROM usuarios WHERE nombre = :nombre");
        $stmt->bindParam(':nombre', $nombre);
        $stmt->execute();
        $user = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($user && password_verify($pass, $user['pass'])) {
            return true;
        }
        return false;
    }

    public function register($nombre, $pass) {
        $stmt = $this->conn->prepare("SELECT id FROM usuarios WHERE nombre = :nombre");
        $stmt->bindParam(':nombre', $nombre);
        $stmt->execute();

        if ($stmt->rowCount() > 0) {
            return false;
        }

        $hashedPass = password_hash($pass, PASSWORD_DEFAULT);
        $stmt = $this->conn->prepare("INSERT INTO usuarios (nombre, pass) VALUES (:nombre, :pass)");
        $stmt->bindParam(':nombre', $nombre);
        $stmt->bindParam(':pass', $hashedPass);
        $stmt->execute();

        return true;
    }
}
?>
```

## Paso 5: Crear los Controladores

### controllers/AuthController.php

Maneja login, registro y logout.

```php
<?php
require_once('models/UserModel.php');

class AuthController {
    private $userModel;

    public function __construct() {
        $this->userModel = new UserModel();
    }

    public function login() {
        session_start();

        if ($_SERVER["REQUEST_METHOD"] == "POST") {
            $nombre = $_POST['nombre'];
            $pass = $_POST['pass'];

            if ($this->userModel->authenticate($nombre, $pass)) {
                $_SESSION['nombre'] = $nombre;
                header("Location: index.php?action=home");
                exit();
            } else {
                $error = "Usuario o contraseña incorrectos.";
                include('views/login.php');
            }
        } else {
            include('views/login.php');
        }
    }

    public function register() {
        session_start();

        if ($_SERVER["REQUEST_METHOD"] == "POST") {
            $nombre = $_POST['nombre'];
            $pass = $_POST['pass'];

            if ($this->userModel->register($nombre, $pass)) {
                $success = "Registro exitoso. Puede iniciar sesión <a href='index.php?action=login'>aquí</a>.";
                include('views/register.php');
            } else {
                $error = "El usuario ya existe.";
                include('views/register.php');
            }
        } else {
            include('views/register.php');
        }
    }

    public function logout() {
        session_start();
        session_destroy();
        header("Location: index.php?action=login");
        exit();
    }
}
?>
```

### controllers/HomeController.php

Maneja la página principal.

```php
<?php
class HomeController {
    public function index() {
        session_start();

        if (!isset($_SESSION['nombre'])) {
            header("Location: index.php?action=login");
            exit();
        }

        $nombre = $_SESSION['nombre'];
        include('views/home.php');
    }
}
?>
```

## Paso 6: Crear las Vistas

### views/login.php

Formulario de login.

```php
<!DOCTYPE html>
<html>
<head>
    <title>Iniciar Sesión</title>
</head>
<body>
    <h2>Iniciar Sesión</h2>
    <form method="post" action="index.php?action=login">
        <label for="nombre">Nombre de Usuario:</label>
        <input type="text" name="nombre" required><br>

        <label for="pass">Contraseña:</label>
        <input type="password" name="pass" required><br>
        <input type="submit" value="Iniciar Sesión">
    </form>
    <?php if (isset($error)) { echo "<p>$error</p>"; } ?>

    <p>¿No tienes una cuenta? <a href="index.php?action=register">Regístrate</a></p>
</body>
</html>
```

### views/register.php

Formulario de registro.

```php
<!DOCTYPE html>
<html>
<head>
    <title>Registro</title>
</head>
<body>
    <h2>Registro de Usuario</h2>
    <form method="post" action="index.php?action=register">
        <label for="nombre">Nombre de Usuario:</label>
        <input type="text" name="nombre" required><br>

        <label for="pass">Contraseña:</label>
        <input type="password" name="pass" required><br>

        <input type="submit" value="Registrarse">
    </form>
    <?php if (isset($error)) { echo "<p>$error</p>"; } ?>
    <?php if (isset($success)) { echo "<p>$success</p>"; } ?>
</body>
</html>
```

### views/home.php

Página principal.

```php
<!DOCTYPE html>
<html>
<head>
    <title>Página de inicio</title>
</head>
<body>
    <h2>Bienvenido, <?php echo $nombre; ?>!</h2>
    <a href="index.php?action=logout">Cerrar Sesión</a>
</body>
</html>
```

## Paso 7: Crear el Router (index.php)

Este archivo dirige las solicitudes a los controladores apropiados.

```php
<?php
require_once('controllers/AuthController.php');
require_once('controllers/HomeController.php');

// La variable $action obtiene el parámetro 'action' de la URL usando $_GET, que es un array superglobal que contiene datos enviados por método GET.
// Si no se especifica 'action', se asigna 'home' por defecto.
$action = isset($_GET['action']) ? $_GET['action'] : 'home';

// El switch evalúa la acción y crea una instancia del controlador correspondiente.
// Luego llama al método adecuado para manejar la solicitud.
switch ($action) {
    case 'login':
        // Instancia AuthController y llama al método login()
        $controller = new AuthController();
        $controller->login();
        break;
    case 'register':
        // Instancia AuthController y llama al método register()
        $controller = new AuthController();
        $controller->register();
        break;
    case 'logout':
        // Instancia AuthController y llama al método logout()
        $controller = new AuthController();
        $controller->logout();
        break;
    case 'home':
    default:
        // Instancia HomeController y llama al método index()
        $controller = new HomeController();
        $controller->index();
        break;
}
?>
```
?>

## Paso 8: Probar la Aplicación

1. Coloca todos los archivos en la carpeta raíz de tu servidor web.
2. Accede a `http://localhost/tu_proyecto/index.php`.
3. Registra un usuario.
4. Inicia sesión.
5. Verifica la página principal y el logout.

## Explicación del Flujo MVC

1. El usuario accede a una URL como `index.php?action=login`.
2. El router (index.php) determina que la acción es 'login' y crea una instancia de AuthController.
3. AuthController llama al método login(), que puede mostrar el formulario o procesar el envío.
4. Si es un POST, usa UserModel para verificar credenciales en la base de datos.
5. Si es exitoso, redirige a home; si no, muestra la vista login con error.
6. La vista solo contiene HTML y PHP simple para mostrar datos.

---

## Explicación de la Sintaxis PHP en Funciones Clave

### Uso de Variables Superglobales

- `$_POST`: Array asociativo que contiene datos enviados mediante el método POST, usado para obtener datos de formularios.
- `$_GET`: Array asociativo que contiene datos enviados mediante el método GET, usado para obtener parámetros de URL.
- `$_SESSION`: Array asociativo que almacena variables de sesión para mantener estado entre solicitudes.

### Manejo de Sesiones

- `session_start()`: Inicia una sesión o reanuda la actual para almacenar y acceder a variables de sesión.
- `session_destroy()`: Destruye todos los datos registrados en la sesión actual.

### Control de Flujo

- `if ($_SERVER["REQUEST_METHOD"] == "POST")`: Verifica si la solicitud HTTP es de tipo POST, comúnmente para procesar formularios.
- `header("Location: url")`: Envía una cabecera HTTP para redirigir al navegador a otra URL.
- `exit()`: Termina la ejecución del script inmediatamente para evitar que se ejecute código adicional.

### Uso de Clases y Métodos

- `new ClassName()`: Crea una nueva instancia de una clase.
- `$this->method()`: Llama a un método dentro de la misma clase.
- `require_once()`: Incluye un archivo PHP solo una vez para evitar inclusiones múltiples.

### Operaciones con Base de Datos (PDO)

- `prepare()`: Prepara una sentencia SQL para ejecución segura con parámetros.
- `bindParam()`: Vincula un parámetro SQL a una variable PHP para evitar inyección SQL.
- `execute()`: Ejecuta la sentencia preparada.
- `fetch()`: Obtiene una fila del conjunto de resultados.
- `rowCount()`: Devuelve el número de filas afectadas por la última sentencia SQL.

### Seguridad

- `password_hash()`: Crea un hash seguro para almacenar contraseñas.
- `password_verify()`: Verifica que una contraseña coincida con un hash almacenado.

#### ¿Qué significa "hashear" en el contexto de UserModel.php?

"Hashear" (o "hashing") es el proceso de transformar una contraseña en una cadena de caracteres irreversible mediante un algoritmo matemático. Esto significa que la contraseña original no puede ser recuperada del hash, pero se puede verificar si una contraseña dada coincide con el hash almacenado usando funciones como `password_verify()`. En UserModel.php, se usa `password_hash()` para hashear la contraseña antes de almacenarla en la base de datos, y `password_verify()` para verificarla durante el login. Esto mejora la seguridad al evitar almacenar contraseñas en texto plano, protegiendo contra ataques si la base de datos es comprometida.

#### ¿Qué son las variables de sesión (SESSION) en PHP?

Las variables de sesión en PHP permiten almacenar información del usuario de manera persistente entre diferentes solicitudes HTTP. Una sesión se inicia con `session_start()` y se identifica mediante un ID único enviado al navegador del usuario (generalmente a través de cookies). Esto permite mantener el estado de la aplicación, como recordar si un usuario ha iniciado sesión. En esta aplicación, se usa `$_SESSION['nombre']` para guardar el nombre del usuario después del login y verificar en páginas protegidas si el usuario está autenticado. Las sesiones se destruyen con `session_destroy()` al cerrar sesión, eliminando la información almacenada.

Este resumen ayuda a comprender cómo se aplican conceptos básicos y avanzados de PHP en la aplicación para manejar autenticación y sesiones de usuario de forma segura y organizada.

## Explicación Detallada de la Sintaxis PHP Aplicada en Cada Bloque de Funciones

### controllers/AuthController.php

#### __construct()
- `$this->userModel = new UserModel();`: Crea una nueva instancia de la clase UserModel y la asigna a la propiedad privada $userModel. Esto permite acceder a métodos de base de datos en esta clase.

#### login()
- `session_start();`: Función PHP que inicia una sesión HTTP para almacenar variables de estado del usuario entre solicitudes.
- `if ($_SERVER["REQUEST_METHOD"] == "POST")`: Verifica si el método de la solicitud HTTP es POST, lo que indica que se envió un formulario.
- `$nombre = $_POST['nombre'];`: Array superglobal $_POST que contiene datos enviados vía POST; accede al campo 'nombre' del formulario.
- `$pass = $_POST['pass'];`: Similar, obtiene la contraseña del formulario.
- `if ($this->userModel->authenticate($nombre, $pass))`: Llama al método authenticate del modelo; usa $this para acceder a la propiedad de instancia.
- `$_SESSION['nombre'] = $nombre;`: Array superglobal $_SESSION para almacenar datos en la sesión; asigna el nombre del usuario.
- `header("Location: index.php?action=home");`: Función header() envía encabezados HTTP; aquí redirige al navegador a la URL especificada.
- `exit();`: Termina la ejecución del script inmediatamente para evitar código adicional.
- `include('views/login.php');`: Incluye y ejecuta el archivo de vista login.php, permitiendo que el código HTML se muestre.

#### register()
- `session_start();`: Inicia la sesión para posibles usos futuros, aunque no se modifica aquí.
- `if ($_SERVER["REQUEST_METHOD"] == "POST")`: Verifica si la solicitud es POST para procesar el formulario de registro.
- `$nombre = $_POST['nombre'];`: Recupera el nombre de usuario del formulario enviado.
- `$pass = $_POST['pass'];`: Recupera la contraseña del formulario.
- `if ($this->userModel->register($nombre, $pass))`: Llama al método register del modelo para intentar insertar el usuario.
- `include('views/register.php');`: Incluye la vista de registro para mostrar el mensaje de éxito o error.

#### logout()
- `session_start();`: Inicia la sesión para poder acceder y destruirla.
- `session_destroy();`: Función que destruye todas las variables de sesión y termina la sesión actual.
- `header("Location: index.php?action=login");`: Redirige al usuario a la página de login después de cerrar sesión.
- `exit();`: Asegura que el script termine aquí.

### controllers/HomeController.php

#### index()
- `session_start();`: Inicia la sesión para poder leer las variables de sesión almacenadas.
- `if (!isset($_SESSION['nombre']))`: Función isset() verifica si la variable 'nombre' existe en el array superglobal $_SESSION; el operador ! niega la condición.
- `header("Location: index.php?action=login");`: Envía un encabezado de redirección HTTP para enviar al usuario a la página de login.
- `exit();`: Termina la ejecución del script para evitar que se ejecute código posterior.
- `$nombre = $_SESSION['nombre'];`: Asigna el valor de la variable de sesión 'nombre' a la variable local $nombre.
- `include('views/home.php');`: Incluye el archivo de vista home.php, ejecutando su código PHP y HTML.

### models/UserModel.php

#### __construct()
- `global $con;`: Palabra clave global permite acceder a la variable $con definida en el ámbito global (config.php).
- `$this->conn = $con;`: Asigna la conexión PDO global a la propiedad privada $conn de la instancia.

#### authenticate($nombre, $pass)
- `$stmt = $this->conn->prepare("SELECT nombre, pass FROM usuarios WHERE nombre = :nombre");`: Método prepare() de PDO crea una declaración preparada para evitar inyección SQL; usa marcadores de posición (:nombre).
- `$stmt->bindParam(':nombre', $nombre);`: Vincula el parámetro :nombre con la variable $nombre para sanitizar la entrada.
- `$stmt->execute();`: Ejecuta la consulta preparada.
- `$user = $stmt->fetch(PDO::FETCH_ASSOC);`: Método fetch() obtiene la fila siguiente como array asociativo; PDO::FETCH_ASSOC especifica el formato.
- `if ($user && password_verify($pass, $user['pass']))`: Verifica si $user no es null (usuario encontrado) y si password_verify() confirma que la contraseña coincide con el hash almacenado.

#### register($nombre, $pass)
- `$stmt = $this->conn->prepare("SELECT id FROM usuarios WHERE nombre = :nombre");`: Prepara una consulta SELECT para verificar existencia del usuario.
- `$stmt->bindParam(':nombre', $nombre);`: Vincula el parámetro con la variable.
- `$stmt->execute();`: Ejecuta la consulta.
- `if ($stmt->rowCount() > 0)`: Método rowCount() devuelve el número de filas afectadas; si >0, el usuario existe.
- `$hashedPass = password_hash($pass, PASSWORD_DEFAULT);`: Función password_hash() crea un hash seguro de la contraseña usando el algoritmo por defecto. "Hashear" significa transformar la contraseña en una cadena de caracteres irreversible que no puede ser revertida a la contraseña original, pero que puede verificarse con password_verify(). Esto asegura que las contraseñas no se almacenen en texto plano, mejorando la seguridad.
- `$stmt = $this->conn->prepare("INSERT INTO usuarios (nombre, pass) VALUES (:nombre, :pass)");`: Prepara una consulta INSERT con marcadores de posición.
- `$stmt->bindParam(':nombre', $nombre);`: Vincula el nombre.
- `$stmt->bindParam(':pass', $hashedPass);`: Vincula la contraseña hasheada.
- `$stmt->execute();`: Ejecuta la inserción.

### index.php

#### Routing Logic
- `$action = isset($_GET['action']) ? $_GET['action'] : 'home';`: Usa el operador ternario para asignar 'home' si no se especifica acción.
- `switch ($action)`: Estructura de control switch para determinar qué controlador instanciar basado en la acción.
- `$controller = new AuthController();`: Instancia la clase AuthController.
- `$controller->login();`: Llama al método login para manejar la acción.


