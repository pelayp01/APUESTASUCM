```mermaid
sequenceDiagram
    autonumber
    participant V as VistaLogin (UI)
    participant C as ControladorJugador
    participant F as FachadaEcosistema (Imp)
    participant SA as SAJugador (Imp)
    participant DAO as DAOJugador (Imp)
    participant DB as Base de Datos (MySQL)

    V->>C: pulsarBotonLogin(correo, pass)
    C->>C: procesarLogin(correo, pass)
    C->>F: loginJugador(correo, pass)
    F->>SA: login(correo, pass)
    SA->>DAO: buscarPorCorreoYPassword(correo, pass)
    DAO->>DB: getConnection()
    DAO->>DB: SELECT...
    DB-->>DAO: ResultSet
    DAO-->>SA: TJugador
    SA-->>F: TJugador
    F-->>C: TJugador
    C-->>V: TJugador
