# Solución Examen IS2: Gestión de Bebidas Energéticas
Este documento proporciona la solución completa a los dos ejercicios del examen para el caso de uso **Alta Sabor de Bebida**, aplicando estrictamente los patrones de arquitectura multicapa estudiados en la asignatura (Ingeniería de Software II).

---

## 1. Diagrama de Clases (Ejercicio 1)

El diseño arquitectónico sigue el patrón de diseño clásico de tres capas (Presentación, Negocio e Integración) requerido en el proyecto. 

### Patrones de Diseño Aplicados
1. **Multicapa (Multitier)**: Desacoplamiento total de responsabilidades.
2. **Fachada (Facade)** (`FachadaBebidas`): Punto de entrada único a la capa de Negocio desde la capa de Presentación. Evita dependencias directas con los servicios de aplicación.
3. **Application Service / Servicio de Aplicación** (`SABebidas`): Contiene la lógica transaccional y las reglas de negocio del caso de uso.
4. **Transfer Object (DTO)** (`TBebida`, `TSabor`): Estructuras de datos ligeras, serializables y sin comportamiento para transportar datos entre capas.
5. **Abstract Factory** (`FactoriaSA`, `FactoriaDAO`): Creación desacoplada de las implementaciones de la lógica de negocio y de integración.
6. **Data Access Object (DAO)** (`DAOBebida`, `DAOSabor`): Encapsulan el acceso a la base de datos física (SQL/JDBC).

### Diagrama de Clases UML (Mermaid)

```mermaid
classDiagram
    %% Capa de Presentación
    class VistaAltaSabor {
        +mostrar() void
        +capturarDatos() void
        +mostrarResultado(int result) void
    }
    
    class ControladorBebidas {
        -fachada: FachadaBebidas
        +altaSabor(TSabor sabor): int
    }

    %% Capa de Negocio (Fachada y Factoría)
    class FachadaBebidas {
        <<interface>>
        +altaSabor(TSabor sabor) int
    }
    
    class FachadaBebidasImp {
        -saBebidas: SABebidas
        +getInstance() FachadaBebidas$
        +altaSabor(TSabor sabor) int
    }

    class FactoriaSA {
        <<abstract>>
        +getInstance() FactoriaSA$
        +generarSABebidas() SABebidas*
    }

    class FactoriaSAImp {
        +generarSABebidas() SABebidas
    }

    %% Capa de Negocio (Servicios y DTOs)
    class SABebidas {
        <<interface>>
        +altaSabor(TSabor sabor) int
    }

    class SABebidasImp {
        +altaSabor(TSabor sabor) int
    }

    class TBebida {
        -idBebida: String
        -nombre: String
        -lema: String
        -activo: boolean
        +getIdBebida() String
        +setIdBebida(String id) void
        +getNombre() String
        +setNombre(String nom) void
        +getLema() String
        +setLema(String lem) void
        +isActivo() boolean
        +setActivo(boolean act) void
    }

    class TSabor {
        -idSabor: int
        -idBebida: String
        -nombre: String
        -kilocalorias: int
        -detalle: String
        -toxicidad: int
        -activo: boolean
        +getIdSabor() int
        +setIdSabor(int id) void
        +getIdBebida() String
        +setIdBebida(String id) void
        +getNombre() String
        +setNombre(String nom) void
        +getKilocalorias() int
        +setKilocalorias(int kcal) void
        +getDetalle() String
        +setDetalle(String det) void
        +getToxicidad() int
        +setToxicidad(int tox) void
        +isActivo() boolean
        +setActivo(boolean act) void
    }

    %% Capa de Integración
    class FactoriaDAO {
        <<abstract>>
        +getInstance() FactoriaDAO$
        +generarDAOBebida() DAOBebida*
        +generarDAOSabor() DAOSabor*
    }

    class FactoriaDAOImp {
        +generarDAOBebida() DAOBebida
        +generarDAOSabor() DAOSabor
    }

    class DAOBebida {
        <<interface>>
        +buscarBebida(String idBebida) TBebida
        +insertar(TBebida bebida) boolean
    }

    class DAOBebidaImp {
        +buscarBebida(String idBebida) TBebida
        +insertar(TBebida bebida) boolean
    }

    class DAOSabor {
        <<interface>>
        +buscarSaborPorNombre(String idBebida, String nombreSabor) TSabor
        +insertar(TSabor sabor) int
        +actualizar(TSabor sabor) boolean
    }

    class DAOSaborImp {
        +buscarSaborPorNombre(String idBebida, String nombreSabor) TSabor
        +insertar(TSabor sabor) int
        +actualizar(TSabor sabor) boolean
    }

    %% Relaciones de dependencia y herencia
    VistaAltaSabor --> ControladorBebidas : usa
    ControladorBebidas --> FachadaBebidas : usa
    ControladorBebidas ..> TSabor : usa
    
    FachadaBebidas <|.. FachadaBebidasImp : implementa
    FachadaBebidasImp --> SABebidas : usa
    FachadaBebidasImp ..> TSabor : usa
    
    FactoriaSA <|-- FactoriaSAImp : hereda
    FachadaBebidasImp ..> FactoriaSA : usa
    
    SABebidas <|.. SABebidasImp : implementa
    SABebidasImp ..> FactoriaDAO : usa
    SABebidasImp --> DAOBebida : usa
    SABebidasImp --> DAOSabor : usa
    SABebidasImp ..> TSabor : procesa
    SABebidasImp ..> TBebida : procesa

    FactoriaDAO <|-- FactoriaDAOImp : hereda
    DAOBebida <|.. DAOBebidaImp : implementa
    DAOSabor <|.. DAOSaborImp : implementa
    
    DAOBebidaImp ..> TBebida : transporta
    DAOSaborImp ..> TSabor : transporta
```

---

## 2. Diagrama de Secuencia Único (Ejercicio 2)

A continuación, se detalla el flujo completo de interacciones del caso de uso **Alta Sabor de Bebida** en un **único diagrama de secuencia**. 

Para representar todos los posibles flujos y códigos de retorno (`> 0`, `0`, `-1`, `-2`) en un solo modelo, se utilizan fragmentos condicionales **`alt` / `else`** de UML (representados mediante bloques `alt` en Mermaid), permitiendo visualizar claramente las bifurcaciones de la lógica de negocio.

### Diagrama de Secuencia Unificado (Mermaid)

```mermaid
sequenceDiagram
    autonumber
    actor Admin as Administrador
    participant Vista as VistaAltaSabor
    participant Ctrl as ControladorBebidas
    participant Fachada as FachadaBebidasImp
    participant SA as SABebidasImp
    participant FactDAO as FactoriaDAOImp
    participant DAOB as DAOBebidaImp
    participant DAOS as DAOSaborImp
    participant BD as Base de Datos (SQL)

    Admin ->> Vista: Rellena formulario y pulsa "Guardar"
    Note over Vista: Datos: idBebida, nombre, kcal, detalle, toxicidad
    Vista ->> Ctrl: altaSabor(tSabor)
    Ctrl ->> Fachada: altaSabor(tSabor)
    Fachada ->> SA: altaSabor(tSabor)
    
    Note over SA: Inicio de transacción y obtención de DAOs
    SA ->> FactDAO: generarDAOBebida()
    FactDAO -->> SA: DAOBebida (Instancia)
    SA ->> DAOB: buscarBebida(idBebida)
    
    %% Consulta de Bebida a la BD
    DAOB ->> BD: SELECT * FROM Bebida WHERE idBebida = ?
    BD -->> DAOB: Datos de la bebida / ResultSet
    DAOB -->> SA: tBebida (Resultado de búsqueda)

    alt [Bebida no existe en DB O está inactiva (tBebida == null || !tBebida.isActivo())]
        Note over SA: -- RETORNO -2: Bebida inválida o inactiva --
        SA -->> Fachada: -2
        Fachada -->> Ctrl: -2
        Ctrl -->> Vista: -2
        Vista -->> Admin: Muestra error: "La bebida no existe o no está activa (-2)"

    else [Bebida es válida y está activa (tBebida != null && tBebida.isActivo())]
        SA ->> FactDAO: generarDAOSabor()
        FactDAO -->> SA: DAOSabor (Instancia)
        SA ->> DAOS: buscarSaborPorNombre(idBebida, nombre)
        
        %% Consulta de Sabor a la BD
        DAOS ->> BD: SELECT * FROM Sabor WHERE idBebida = ? AND nombre = ?
        BD -->> DAOS: Datos del sabor / ResultSet
        DAOS -->> SA: tSaborExistente (Resultado de búsqueda)

        alt [Sabor ya existe y está activo (tSaborExistente != null && tSaborExistente.isActivo())]
            Note over SA: -- RETORNO -1: Sabor duplicado y activo --
            SA -->> Fachada: -1
            Fachada -->> Ctrl: -1
            Ctrl -->> Vista: -1
            Vista -->> Admin: Muestra error: "El sabor ya existe y está activo (-1)"

        else [Sabor existe pero está inactivo (tSaborExistente != null && !tSaborExistente.isActivo())]
            Note over SA: -- RETORNO 0: Reactivación y actualización de datos --
            Note over SA: Se actualizan kilocalorías, detalle y toxicidad.<br/>Se cambia estado a activo: tSaborExistente.setActivo(true)
            SA ->> DAOS: actualizar(tSaborExistente)
            
            %% UPDATE en BD
            DAOS ->> BD: UPDATE Sabor SET activo = 1, kilocalorias = ?, detalle = ?, toxicidad = ? WHERE idSabor = ?
            BD -->> DAOS: 1 (Número de filas afectadas)
            DAOS -->> SA: true (UPDATE SQL correcto)
            
            SA -->> Fachada: 0
            Fachada -->> Ctrl: 0
            Ctrl -->> Vista: 0
            Vista -->> Admin: Muestra: "El sabor ya existía inactivo. Ha sido reactivado y actualizado (0)"

        else [Sabor no existe (tSaborExistente == null)]
            Note over SA: -- RETORNO > 0: Creación de nuevo sabor (Alta limpia) --
            Note over SA: Se establece activo = true
            SA ->> DAOS: insertar(tSabor)
            
            %% INSERT en BD
            DAOS ->> BD: INSERT INTO Sabor (idBebida, nombre, kilocalorias, detalle, toxicidad, activo) VALUES (?, ?, ?, ?, ?, 1)
            Note over BD: Genera ID único autoincrementado (p.ej. 104)
            BD -->> DAOS: idSabor generado (104)
            DAOS -->> SA: 104 (idSabor autogenerado)
            
            SA -->> Fachada: 104
            Fachada -->> Ctrl: 104
            Ctrl -->> Vista: 104
            Vista -->> Admin: Muestra: "Sabor creado con éxito. ID: 104"
        end
    end
```

---

> [!TIP]
> **Detalle clave de Implementación (Control de Excepciones para Retorno -2):**
> Para garantizar que el sistema siempre devuelva `-2` ante cualquier error inesperado de base de datos (p. ej. fallo de conexión JDBC, caída del servidor, violación inesperada de claves foráneas), toda la lógica dentro de `SABebidasImp.altaSabor` debe estar envuelta en un bloque `try-catch (Exception e)`. Si se produce una excepción, esta es capturada en el catch, se registra el error (log) y se devuelve `-2` de forma limpia y controlada a las capas superiores.

