```mermaid
gantt
    title Plan de Trabajo: Proyecto ApuestaUCM (Marzo 2026)
    dateFormat  YYYY-MM-DD
    axisFormat  %d-%m
    tickInterval 1w

    section Elaboración
    Análisis y Diseño Arquitectónico (E1) :done, e1, 2026-03-02, 2026-03-08
    Hito E2: Línea Base de la Arquitectura  :milestone, 2026-03-08, 0d

    section Construcción (I1)
    Registro, Login y CRUDs (C1)           :active, c1, 2026-03-09, 7d
    Hito C1.1: Acceso y Persistencia OK    :milestone, 2026-03-15, 0d

    section Construcción (I2)
    Lógica Blackjack y Sistema de Riesgo (C2) :c2, 2026-03-16, 7d
    Hito C2.1: Núcleo del Juego Operativo    :milestone, 2026-03-22, 0d

    section Construcción (I3)
    Historial (TOA) y Alertas (C3)         :c3, 2026-03-23, 7d
    Hito C3.1: Sistema Completo Integrado    :milestone, 2026-03-29, 0d

    section Transición
    Documentación y Memoria Técnica        :task5, 2026-03-30, 5d
    Hito Final: Entrega del Proyecto       :milestone, 2026-04-03, 0d
