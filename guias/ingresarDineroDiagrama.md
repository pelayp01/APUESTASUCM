```mermaid
sequenceDiagram
    autonumber
    participant V as VistaCartera (UI)
    participant C as ControladorCartera
    participant F as FachadaEcosistemaImp
    participant SA as SACarteraImp
    participant FactDAO as FactoriaDAO
    participant DAO as DAOCarteraImp
    participant DB as BDConexion (MySQL)
    
    V->>C: ingresarDinero(jugadorId, cantidad)
    C->>F: ingresarDinero(jugadorId, cantidad)
    
    Note over F: FachadaEcosistemaImp ya instanció<br/>SACarteraImp en su constructor
    F->>SA: ingresarDinero(jugadorId, cantidad)
    
    rect rgb(240, 240, 240)
        Note over SA: Validar cantidad > 0
    end
    
    SA->>FactDAO: generarDAOCartera()
    FactDAO-->>SA: daoCartera
    
    %% Paso 1: Leer la cartera actual
    SA->>DAO: leerPorJugador(jugadorId)
    DAO->>DB: getConnection()
    DAO->>DB: SELECT * FROM Cartera_Virtual WHERE jugador_id = ?
    DB-->>DAO: ResultSet (datos de la cartera)
    DAO-->>SA: TCartera (cartera)
    
    rect rgb(240, 240, 240)
        Note over SA: Validar cartera != null<br/>Calcular: nuevoSaldo = saldoActual + cantidad
    end
    
    %% Paso 2: Actualizar con el nuevo saldo calculado
    SA->>DAO: actualizarSaldo(jugadorId, nuevoSaldo)
    DAO->>DB: getConnection()
    DAO->>DB: UPDATE Cartera_Virtual SET saldoActual = ? WHERE jugador_id = ?
    DB-->>DAO: Filas afectadas (1)
    
    DAO-->>SA: true
    SA-->>F: true
    F-->>C: true
    C-->>V: (Muestra mensaje de éxito)
