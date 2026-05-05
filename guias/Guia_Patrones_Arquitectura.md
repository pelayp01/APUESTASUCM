# Guía de Patrones de Diseño y Arquitectura - ApuestaUCM

Esta guía detalla los patrones de diseño y arquitectónicos utilizados en el proyecto **ApuestaUCM** para la asignatura de Ingeniería de Software II (IS2). Entender estos patrones es fundamental para defender el proyecto y comprender cómo interactúan las distintas capas (Presentación, Negocio e Integración).

---

## 1. Patrón Singleton (Instancia Única)

### 📖 Teoría
El patrón Singleton garantiza que una clase tenga **una única instancia** y proporciona un punto de acceso global a ella. Es muy útil cuando exactamente un objeto es necesario para coordinar acciones en todo el sistema, y crear múltiples instancias causaría problemas (ej. desincronización) o gastaría memoria innecesariamente.

### 💻 Uso en ApuestaUCM
Lo usamos en el "corazón" de la aplicación para asegurarnos de que solo exista una fábrica o una fachada:

1.  **`FactoriaSA.getInstance()`**: Solo necesitamos una fábrica que construya los Servicios de Aplicación.
2.  **`FactoriaDAO.getInstance()`**: Solo necesitamos una fábrica que gestione la creación de los accesos a datos.
3.  **`FachadaEcosistemaImp.getInstance()`**: Asegura que todos los controladores de la vista hablen con la misma y única fachada del sistema.

### 🔍 Ejemplo de Implementación
```java
public abstract class FactoriaSA {
    private static FactoriaSA instancia; 

    public static FactoriaSA getInstance() {
        if (instancia == null) {
            instancia = new FactoriaSAImp();
        }
        return instancia;
    }
    // ... métodos de factoría ...
}
```

---

## 2. Patrón Fachada (Facade)

### 📖 Teoría
El patrón Fachada proporciona una interfaz unificada y de alto nivel que oculta la complejidad de un subsistema subyacente. Simplifica el uso del sistema para el cliente (en nuestro caso, la capa de Presentación), exponiendo solo las operaciones que realmente importan.

### 💻 Uso en ApuestaUCM
La capa de **Presentación** (los Controladores de las vistas) no debe conocer los entresijos de cómo funciona el negocio. Por ello, toda la comunicación de la vista hacia la lógica pasa por la **Fachada**.

*   **Dónde se encuentra:** Interfaz `FachadaEcosistema` y su implementación `FachadaEcosistemaImp`.
*   **Quién la llama:** Clases como `ControladorJugador`, `ControladorJuego`, `ControladorSimulacion`, etc. Ellos hacen `FachadaEcosistemaImp.getInstance().loginAdministrador(...)` sin saber qué Servicios de Aplicación (SA) se ejecutan por debajo.

---

## 3. Patrón Factoría Abstracta (Abstract Factory)

### 📖 Teoría
Proporciona una interfaz para crear familias de objetos relacionados o dependientes sin especificar sus clases concretas. Permite cambiar fácilmente entre distintas familias de productos (ej. cambiar de persistencia en Ficheros a persistencia en MySQL) sin tocar el código que usa dichos productos.

### 💻 Uso en ApuestaUCM
Lo usamos para desacoplar el Negocio de la Integración, y la Fachada del Negocio.

1.  **`FactoriaSA` / `FactoriaSAImp`**: La Fachada no hace un `new SAJugadorImp()`, sino que pide a la `FactoriaSA` que genere uno (`FactoriaSA.getInstance().generarSAJugador()`).
2.  **`FactoriaDAO` / `FactoriaDAOImp`**: El Servicio de Aplicación (`SAJugadorImp`) no sabe qué base de datos se usa. Pide a la factoría que le dé un `DAOJugador` genérico (`FactoriaDAO.getInstance().generarDAOJugador()`).

---

## 4. Patrón Data Access Object (DAO)

### 📖 Teoría
Es un patrón de diseño arquitectónico que separa la lógica de acceso a datos (bases de datos, ficheros, etc.) de la lógica de negocio de la aplicación. Centraliza todos los accesos a datos en clases específicas.

### 💻 Uso en ApuestaUCM
Se encuentra en la capa de **Integración**. Los DAOs son los únicos que saben hablar con la base de datos (mediante JDBC) o con los archivos.

*   **Dónde se encuentra:** Interfaces como `DAOJugador`, `DAOJuego`, `DAOCartera`, `DAOSimulacion`, `DAOMensajeAlerta` y sus respectivas implementaciones (`DAOJugadorImp`, etc.).
*   **Ejemplo de uso:** En `SAJugadorImp.java`, cuando un jugador quiere registrarse, el SA verifica las reglas de negocio (ej. que el email no esté repetido) y luego llama a `daoJugador.crear(tJugador)`.

---

## 5. Transfer Object (o Data Transfer Object - DTO)

### 📖 Teoría
Se utiliza para encapsular varios datos y enviarlos de golpe a través de las diferentes capas del sistema. Son objetos simples, generalmente sin lógica de negocio, que solo contienen atributos, constructores, *getters* y *setters*.

### 💻 Uso en ApuestaUCM
Se utilizan para mover información de la Vista al Negocio, y del Negocio a la Integración. Las clases suelen empezar por la letra **T**.

*   **Dónde se encuentra:** Clases como `TJugador`, `TAdministrador`, `TJuego`, `TCartera`.
*   **Por qué se usa:** Evita tener que pasar muchísimos parámetros a una función. En lugar de hacer `registrarJugador(String nombre, String email, String password, double saldo)`, se hace `registrarJugador(TJugador jugador)`.

---

## 6. Patrón Builder (Constructor)

### 📖 Teoría
El patrón Builder separa la construcción de un objeto complejo de su representación, de forma que el mismo proceso de construcción puede crear diferentes representaciones. Evita tener constructores gigantes (Anti-patrón Telescoping Constructor).

### 💻 Uso en ApuestaUCM
Se emplea mayoritariamente a la hora de crear los **Transfer Objects (DTOs)** en la capa de Presentación o Integración, especialmente si esos objetos tienen muchos atributos.

*   **Ejemplo de uso (Teórico/Práctico):**
```java
TJugador nuevoJugador = new TJugadorBuilder()
                            .withNombre("Usuario1")
                            .withCorreo("usuario@ucm.es")
                            .withPassword("12345")
                            .withNivel(1)
                            .build();
```
*   **Beneficio principal:** El código es mucho más legible, sobre todo cuando hay atributos opcionales que no siempre hace falta rellenar en el momento de creación.

---

## Resumen del Flujo Arquitectónico Completo

Para visualizar cómo todos estos patrones trabajan juntos en vuestro código, imagina el proceso de **"Registrar un nuevo Jugador"**:

1.  **Vista/Controlador:** Recoge los datos de los campos de texto, usa un **Builder** para crear un **TJugador (Transfer)** y se lo pasa a la **Fachada** (que obtiene mediante el patrón **Singleton**).
2.  **FachadaEcosistemaImp:** Obtiene de la **FactoriaSA (Abstract Factory)** el `SAJugador` correspondiente y llama a `saJugador.registrarJugador(tJugador)`.
3.  **SAJugadorImp:** Realiza las comprobaciones lógicas. Luego, pide a la **FactoriaDAO (Abstract Factory)** un `DAOJugador`.
4.  **DAOJugadorImp:** Recibe el **TJugador (Transfer)**, ejecuta la consulta SQL `INSERT` en la base de datos y devuelve el resultado (el ID generado) hacia arriba.
