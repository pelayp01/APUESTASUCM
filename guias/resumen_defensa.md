# 🛡️ Guía para la Defensa del Proyecto: ApuestasUCM

Este documento contiene los puntos clave para realizar una defensa sólida del proyecto **ApuestasUCM (Ludopatía UCM)**, centrándose en las decisiones técnicas y de diseño solicitadas por la profesora.

---

## 🏗️ 1. Arquitectura del Sistema (N-Tier)

El proyecto se basa en una **Arquitectura en 3 Capas** (Presentación, Negocio e Integración). Esta separación garantiza que los cambios en la interfaz no afecten a la lógica y que la lógica sea independiente del motor de base de datos.

1.  **Capa de Presentación (Swing + MVC):**
    *   **Vistas:** Interfaces gráficas construidas con Java Swing.
    *   **Controladores:** Actúan como intermediarios. Capturan los eventos de la vista y delegan en la Fachada. **Importante:** Las vistas nunca acceden directamente al Negocio.
2.  **Capa de Negocio (Lógica de Aplicación):**
    *   **Fachada (FachadaEcosistema):** El "cerebro" central. Es el único punto de entrada para la Presentación.
    *   **Servicios de Aplicación (SA):** Clases que contienen las reglas de negocio (ej. validación de edad, cálculo de riesgo de ludopatía).
    *   **Transfer Objects (TO):** Objetos simples que transportan datos entre capas para evitar el acoplamiento excesivo.
3.  **Capa de Integración (Persistencia):**
    *   **DAO (Data Access Object):** Encapsulan el acceso a MySQL mediante JDBC. Cada DAO se encarga de una entidad (Jugador, Cartera, etc.).
    *   **BDConexion:** Gestiona la conexión única con la base de datos.

---

## 🛠️ 2. Patrones de Diseño Implementados

Hemos implementado patrones de diseño **GoF** para asegurar un código robusto y mantenible:

| Patrón | Por qué se ha implementado |
| :--- | :--- |
| **Fachada (Facade)** | Proporciona una interfaz unificada y simplificada. Evita que la UI tenga que conocer todos los servicios internos. |
| **DAO (Data Access Object)** | Abstrae el almacenamiento. Si mañana cambiamos MySQL por un fichero XML, solo cambiamos el DAO, no el resto del programa. |
| **Abstract Factory** | (FactoriaSA / FactoriaDAO) Centraliza la creación de implementaciones. Permite desacoplar las interfaces de sus clases concretas (`SAJugadorImp`). |
| **Singleton** | Se usa en `BDConexion` y las Factorías. Asegura que solo exista una instancia global, ahorrando memoria y conexiones abiertas. |
| **Builder** | Se aplica en los Transfer Objects (ej. `TJugador`). Permite crear objetos complejos paso a paso de forma legible y segura. |
| **Transfer Object (TO)** | Crucial para la comunicación entre capas sin pasar múltiples parámetros primitivos. |

---

## 🔄 3. Casos de Uso Críticos y Flujos

Los casos de uso que definen la esencia del proyecto son:

### 1️⃣ Registro de Jugador con Transacción
*   **Importancia:** Involucra tres tablas de forma atómica.
*   **Flujo:**
    1.  El SA valida que el usuario sea **mayor de 18 años**.
    2.  El DAO inicia una **transacción SQL**.
    3.  Se inserta en `Usuario` -> `Jugador` -> `Cartera_Virtual` (con saldo inicial).
    4.  Si algo falla, se hace `rollback` para no dejar datos inconsistentes.

### 2️⃣ Simulación de Blackjack y Evaluación de Riesgo
*   **Importancia:** Es el núcleo del sistema de prevención.
*   **Flujo:**
    1.  El jugador apuesta. El sistema descuenta el saldo.
    2.  Se simula la partida. Si gana, se ingresa el premio.
    3.  **Evaluación Automática:** Inmediatamente después, el SA analiza el historial. Si hay rachas perdedoras o pérdidas totales elevadas, el riesgo del jugador sube a **ALTO** o **MEDIO**.

### 3️⃣ Prevención Activa (Alertas)
*   **Importancia:** Cumple el objetivo social del proyecto.
*   **Flujo:**
    1.  Al iniciar sesión, el sistema carga las alertas basadas en el umbral de pérdida del jugador.
    2.  Si el jugador intenta jugar siendo menor, se le muestra un recurso educativo obligatorio.

---

## 🗄️ 4. Modelo de Datos y Relaciones

La base de datos MySQL está diseñada para cumplir los requisitos de herencia y multiplicidad de IS2:

*   **Herencia (Table-per-Class):** `Usuario` es la tabla padre. `Jugador` y `Administrador` tienen sus propias tablas que heredan el ID (FK y PK) de Usuario.
*   **Relación 1:1:** Cada `Jugador` tiene exactamente una `Cartera_Virtual`.
*   **Relación 1:N:** Un `Jugador` puede tener múltiples `Simulaciones` (historial).
*   **Relación N:M:** Un `Jugador` puede ver múltiples `Recursos_Educativos`, y un recurso puede ser visto por muchos jugadores (registrado en la tabla intermedia `Jugador_Recurso`).

---

## 🏁 5. Conclusión para la Defensa

"Hemos construido un ecosistema digital que no solo simula un casino, sino que aplica técnicas de **Ingeniería de Software** para proteger al usuario. Gracias al uso de patrones como la **Fachada** y el **DAO**, el sistema es escalable. La lógica de **evaluación de riesgo** integrada en el Negocio demuestra cómo la tecnología puede servir para la prevención social."
