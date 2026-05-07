```mermaid
gantt
    title Plan de Trabajo ApuestaUCM (Marzo - Abril 2026)
    dateFormat  YYYY-MM-DD
    axisFormat  %d-%m
    
    section Fase Elaboracion
    Analisis y Diseno Arquitectonico       :active, task1, 2026-03-02, 14d
    Hito E2 Scripts DDL y Modelo BD        :milestone, 2026-03-15, 0d
    
    section Fase Construccion
    Iteracion C1 Registro y CRUDs          :task2, 2026-03-16, 14d
    Hito C1.1 Flujo de Acceso Operativo    :milestone, 2026-03-29, 0d
    Iteracion C2 Blackjack y Riesgo        :task3, 2026-03-30, 14d
    Hito C2.1 Logica Critica Estable       :milestone, 2026-04-12, 0d
    Iteracion C3 Historial y Alertas       :task4, 2026-04-13, 14d
    Hito C3.1 Pre-entrega Sistema Completo :milestone, 2026-04-26, 0d
    
    section Fase de Cierre
    Documentacion Final y Memoria          :task5, 2026-04-27, 4d
    Hito C3.2 Entrega Final Proyecto       :milestone, 2026-04-30, 0d
