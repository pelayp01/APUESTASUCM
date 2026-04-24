# Guía de Implementación de Patrones de Diseño (GoF)

Esta guía propone cómo podrías aplicar varios de los 23 patrones de diseño clásicos de la "Gang of Four" (GoF) en tu proyecto específico (`proyecto-de-is2-equipo-08-apuestasucm`). No es necesario implementar todos los patrones, el objetivo es aplicar **solo aquellos que resuelvan un problema real** en tu código, aumentando su mantenibilidad y escalabilidad.

---

## 1. Patrones Creacionales (Creational)

Estos patrones te ayudarán a independizar tu sistema de cómo se crean sus objetos, haciéndolo más flexible.

### 🌟 Abstract Factory
**Estado: IMPLEMENTADO**
**Problema resuelto:** En tus controladores y servicios se instanciaban directamente las implementaciones usando `new` (ej. `new SAJugadorImpl()`, `new DAOCarteraImp()`). Esto acoplaba el código fuertemente a las implementaciones.
**Implementación actual:**
Se han creado `FactoriaSA` y `FactoriaDAO` como Singletons.
*Uso en Controlador:* `FactoriaSA.getInstance().generarSAJugador();`
*Uso en Servicio:* `FactoriaDAO.getInstance().generarDAOJugador();`
De esta forma se cumple el principio de Inversión de Dependencias.

### 🌟 Builder (Constructor)
**Problema actual:** Los objetos `TJugador` y `Usuario` pueden tener muchos atributos, lo que obliga a tener constructores gigantes o usar muchos `.setX()`.
**Implementación:**
Usa Builder para construir transfers de forma fluida.
```java
TJugador jugador = new TJugador.Builder()
                        .conId("USER-1")
                        .conDni("12345678A")
                        .conNombre("Pepe")
                        .conEdad(25)
                        .build();
```

### 🌟 Singleton (Singleton)
**Implementación:** Ya lo tienes perfectamente implementado en `BDConexion.java`. Garantiza una sola conexión. Deberías replicar el patrón en las Factorías (como en el ejemplo de Abstract Factory arriba).

---

## 2. Patrones de Estructura (Structural)

Se enfocan en cómo las clases y objetos se componen para formar estructuras más grandes.

### 🌟 Fachada / Business Delegate (Facade)
**Estado: IMPLEMENTADO**
**Implementación actual:** Ya utilizas el patrón Fachada con `FachadaEcosistema` y su implementación `FachadaEcosistemaImpl`. Funciona también como un *Business Delegate* al centralizar y ocultar a la capa de presentación toda la complejidad de orquestar los Servicios de Aplicación (`SAJugador`, `SACartera`, etc.). Se accede a ella a través del patrón Singleton (`FachadaEcosistemaImpl.getInstance()`).


### 🌟 Proxy
**Caso de uso:** Carga perezosa (Lazy Loading) de las imágenes de Recursos Educativos o el Historial del jugador.
**Implementación:** En lugar de traer todo el perfil del jugador y sus transacciones de la BD, traes un `ProxyPerfil`. Solo cuando la `VistaHistorial` llame a `jugador.getHistorial()`, el Proxy ejecutará internamente la llamada SQL.

---

## 3. Patrones de Comportamiento (Behavioral)

Se encargan de las responsabilidades y la comunicación entre los objetos.

### 🌟 Strategy (Estrategia)
**Caso de uso:** Tienes varios juegos distintos (Blackjack, Apuestas regulares, etc.) o distintos sistemas de evaluación de riesgo (`EvaluacionRiesgo`).
**Implementación:**
Si la forma de calcular la ganancia varía según el juego:
```java
public interface EstrategiaPremio {
    double calcular(double apuesta);
}

public class PremioBlackjackEstrategia implements EstrategiaPremio {
    public double calcular(double apuesta) { return apuesta * 2.5; }
}

public class PremioRuletaEstrategia implements EstrategiaPremio {
    public double calcular(double apuesta) { return apuesta * 36; }
}
```
En el momento del juego, seteas la estrategia activa y la ejecutas, compartiendo el código de pagos.

### 🌟 Observer (Observador)
**Problema actual:** Si alguien actualiza su cartera (`VistaCartera`), ¿cómo se entera automáticamente la barra de estado superior (`VistaPrincipal`) de que el saldo de las monedas ha bajado?
**Implementación:**
La `CarteraVirtual` o el Modelo es el "Observable", y las diferentes ventanas de UI son los "Observers".
1. Cuando invocas cambiar el saldo, el Modelo dispara `notificarObservadores()`.
2. Las Vistas, que están suscritas, reciben el evento y actualizan sus `JLabels` sin intervención manual del usuario.

### 🌟 State (Estado)
**Caso de uso:** En el minijuego de `Blackjack`.
**Implementación:** El juego de blackjack transita por estados lógicos precisos (Apostando -> Repartiendo -> Turno Jugador -> Turno Croupier -> Pago).
Implementar un `EstadoBlackjack` evita que la interfaz tenga decenas de `if (juegoIniciado == true)` habilitando o deshabilitando botones. Al cambiar de clase estado, los botones se habilitan automáticamente según el estado actual.

### 🌟 Chain of Responsibility (Cadena de Responsabilidad)
**Caso de uso:** El control de validaciones al registrar o apostar (Ej: Validar NIF -> Validar Edad -> Validar Dinero Suficiente -> Validar Riesgo Ludopatía).
**Implementación:** Configuras una cadena de objetos validadores. Si todos pasan a su sucesor, la acción procede. Si uno falla, se quiebra la cadena y devuelve un error concreto sin llenar los SAs de `if-else` gigantes.

---
### Resumen Práctico: Próximos Pasos
Te recomiendo implementar **los siguientes 2 patrones** para dar un gran salto arquitectónico fácil de demostrar y explicar en clase/presentación:
1. **Observer**: Agrégalo en tu Panel de Jugador para que la UI se actualice sola si el saldo cambia en un minijuego sin pedir "refrescar" la pantalla manualmente.
2. **Strategy**: Ideal para aplicar diferentes algoritmos de evaluación de factor de riesgo o fórmulas de premios en tus juegos.
