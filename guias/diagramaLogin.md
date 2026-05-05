```mermaid
sequenceDiagram
    autonumber
    participant V as VistaLogin (ActionListener)
    participant C as ControladorJugador
    participant F as FachadaEcosistemaImp
    participant SA as SAJugadorImp
    participant Fact as FactoriaDAO
    participant DAO as DAOJugadorImp
    participant DB as BDConexion (MySQL)

    V->>C: procesarLogin(correo, password)
    C->>F: loginJugador(correo, password)
    F->>SA: login(correo, password)
    
    Note over SA: El SA pide el DAO a la Factoría
    SA->>Fact: generarDAOJugador()
    Fact-->>SA: daoJugador (Instancia)
    
    SA->>DAO: buscarPorCorreoYPassword(correo, password)
    DAO->>DB: getConnection()
    DAO->>DB: PreparedStatement.executeQuery(sql)
    DB-->>DAO: ResultSet
    
    alt Usuario encontrado
        DAO->>DAO: instanciarJugador(rs)
        DAO-->>SA: jugador (TJugador)
        SA-->>F: jugador (TJugador)
        F-->>C: jugador (TJugador)
        C-->>V: jugador (TJugador)
    else Usuario NO encontrado
        DAO-->>SA: null
        SA-->>C: throw new Exception("...")
        C-->>V: return null
    end

