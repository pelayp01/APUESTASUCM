```mermaid
sequenceDiagram
    autonumber
    participant V as VistaLecturaRecurso
    participant C as ControladorRecursoEducativo
    participant F as FachadaEcosistemaImp
    participant SA as SARecursoEducativoImp
    participant Fact as FactoriaDAO
    participant DAO as DAORecursoEducativoImp
    participant DB as BDConexion

    V->>C: marcarComoVisto(jugadorId, recursoId)
    C->>F: marcarRecursoComoVisto(jugadorId, recursoId)
    F->>SA: marcarComoVisto(jugadorId, recursoId)
    
    SA->>Fact: generarDAORecursoEducativo()
    Fact-->>SA: daoRecurso
    
    Note over SA, DAO: El SA solo pide registrar la acción
    SA->>DAO: registrarVisualizacion(jugadorId, recursoId)
    
    rect rgb(240, 240, 240)
        Note over DAO: Lógica interna del DAO para evitar duplicados
        DAO->>DAO: yaVisualizado(jugadorId, recursoId)
        DAO->>DB: SELECT COUNT(*) ...
        DB-->>DAO: rs (0 o 1)
    end
    
    alt yaVisualizado == false
        DAO->>DB: INSERT INTO Jugador_Recurso ...
        DB-->>DAO: ok
    end
    
    DAO-->>SA: true
    SA-->>F: true
    F-->>C: true
    C-->>V: true

