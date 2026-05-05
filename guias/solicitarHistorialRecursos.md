```mermaid
sequenceDiagram
    autonumber
    participant V as VistaHistorial (UI)
    participant C as ControladorRecursoEducativo
    participant F as FachadaEcosistemaImp
    participant SA as SARecursoEducativoImp
    participant Fact as FactoriaDAO
    participant DAO as DAORecursoEducativoImp
    participant DB as BDConexion (MySQL)

    V->>C: solicitarHistorial(jugadorId)
    C->>F: listarRecursosVistos(jugadorId)
    F->>SA: listarRecursosVistos(jugadorId)
    
    Note over SA: Validación: jugadorId != null
    SA->>Fact: generarDAORecursoEducativo()
    Fact-->>SA: daoRecurso
    
    SA->>DAO: leerVistosPorJugador(jugadorId)
    DAO->>DB: getConnection()
    
    Note over DAO, DB: Consulta con JOIN para traer los datos del recurso
    DAO->>DB: SELECT R.* FROM Recurso_Educativo R INNER JOIN Jugador_Recurso JR...
    DB-->>DAO: ResultSet (Varias filas)
    
    rect rgb(240, 240, 240)
        Note over DAO: Bucle while (rs.next())
        loop Para cada fila en rs
            DAO->>DAO: instanciarRecurso(rs)
            Note right of DAO: Crea TArticulo o TInfografia
        end
    end
    
    DAO-->>SA: List<TRecursoEducativo>
    SA-->>F: List<TRecursoEducativo>
    F-->>C: List<TRecursoEducativo>
    C-->>V: List<TRecursoEducativo>
