```mermaid
sequenceDiagram
    autonumber
    participant V as VistaRecursos (Panel Historial)
    participant C as ControladorRecursoEducativo
    participant F as FachadaEcosistemaImp
    participant FactSA as FactoriaSA
    participant SA as SARecursoEducativoImp
    participant FactDAO as FactoriaDAO
    participant DAO as DAORecursoEducativoImp
    participant DB as BDConexion (MySQL)

    V->>C: solicitarHistorial(jugadorId)
    C->>F: listarRecursosVistos(jugadorId)
    
    F->>FactSA: generarSARecursoEducativo()
    FactSA-->>F: saRecurso
    F->>SA: listarRecursosVistos(jugadorId)
    
    SA->>FactDAO: generarDAORecursoEducativo()
    FactDAO-->>SA: daoRecurso
    SA->>DAO: leerVistosPorJugador(jugadorId)
    
    DAO->>DB: getConnection()
    DAO->>DB: SELECT R.* FROM Recurso_Educativo ...
    DB-->>DAO: ResultSet
    
    loop rs.next()
        DAO->>DAO: instanciarRecurso(rs)
    end
    
    DAO-->>SA: List<TRecursoEducativo>
    SA-->>F: List<TRecursoEducativo>
    F-->>C: List<TRecursoEducativo>
    C-->>V: List<TRecursoEducativo> (Actualizar JTable)


