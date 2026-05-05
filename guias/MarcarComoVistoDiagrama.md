```mermaid
sequenceDiagram
    autonumber
    participant V as VistaLecturaRecurso (ActionListener)
    participant C as ControladorRecursoEducativo
    participant F as FachadaEcosistemaImp
    participant SA as SARecursoEducativoImp
    participant Fact as FactoriaDAO
    participant DAO as DAORecursoEducativoImp
    participant DB as BDConexion (MySQL)

    V->>C: marcarComoVisto(jugadorId, recursoId)
    C->>F: marcarRecursoComoVisto(jugadorId, recursoId)
    F->>SA: marcarComoVisto(jugadorId, recursoId)
    
    Note over SA: Obtener el DAO de recursos
    SA->>Fact: generarDAORecursoEducativo()
    Fact-->>SA: daoRecurso (Instancia)
    
    SA->>DAO: yaVisualizado(jugadorId, recursoId)
    DAO->>DB: getConnection()
    DB-->>DAO: ResultSet (count)
    
    alt yaVisualizado == false
        SA->>DAO: registrarVisualizacion(jugadorId, recursoId)
        DAO->>DB: PreparedStatement.executeUpdate(sqlInsert)
        DB-->>DAO: void
        DAO-->>SA: true
    else yaVisualizado == true
        DAO-->>SA: true
    end
    
    SA-->>F: true
    F-->>C: true
    C-->>V: true
