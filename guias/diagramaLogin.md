```mermaid
sequenceDiagram
    autonumber
    participant V as VistaLogin (ActionListener)
    participant C as ControladorJugador
    participant F as FachadaEcosistemaImp
    participant SA as SAJugadorImp
    participant DAO as DAOJugadorImp
    participant DB as BDConexion (MySQL)

    V->>C: procesarLogin(correo, password)
    C->>F: loginJugador(correo, password)
    F->>SA: login(correo, password)
    
    SA->>DAO: buscarPorCorreoYPassword(correo, password)
    DAO->>DB: getConnection()
    DAO->>DB: PreparedStatement.executeQuery(sql)
    DB-->>DAO: ResultSet
    
    alt rs.next() es true
        DAO->>DAO: instanciarJugador(rs)
        DAO-->>SA: jugador (TJugador)
        SA-->>F: jugador (TJugador)
        F-->>C: jugador (TJugador)
        C-->>V: jugador (TJugador)
    else rs.next() es false
        DAO-->>SA: null
        SA-->>C: throw new Exception("Correo o contraseña incorrectos.")
        C-->>V: return null
    end
