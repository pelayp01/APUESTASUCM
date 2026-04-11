# Guía de Funcionalidad: Control de Edad y Riesgo (Ludopatía)

En esta guía se detalla cómo funcionan las dos principales expansiones realizadas al proyecto, enfocadas a cumplir con regulaciones y fomentar el juego responsable.

## 1. Validación de Edad Mínima
Para utilizar la aplicación e interactuar con divisas ficticias o reales simuladas, es estrictamente obligatorio demostrar mayoría de edad (+18 años).
- Al momento de registrar una nueva cuenta (`VistaRegistroJugador`), se le pide la edad al usuario.
- **Flujo:** Si el usuario es menor de 18, la lógica de negocio detiene la inserción en el modelo de dominio utilizando un control de Seguridad `throw new Exception("ERROR_EDAD");`.
- Una vez interceptado por el controlador, el sistema despliega un pop-up de alerta de prevención, forzando la visualización directa del *Recurso Educativo* `REC-MENOR` traído directamente de la base de datos para explicarle al menor por qué no debería intentar apostar.

## 2. Juego de Blackjack Virtual
Hemos incluído un entorno interactivo en la pestaña del *Panel Jugador* ("Jugar Blackjack").
- Este juego tiene estipulado un **Winrate (probabilidad de victoria) de la casa implícita**: El usuario solo tiene un 48% de posibilidad de ganar.
- El saldo con la `CarteraVirtual` se sincroniza de inmediato a nivel base de datos retirando dinero, procesando probabilidad con `Math.random()`, y devolviendo el saldo con premio o bloqueando la divisa.

## 3. Control de Riesgos Basado en Pérdidas
Jugar constantemente puede conllevar a una ludopatía si no es detectado a tiempo. Por ello, la simulación lleva un escaneo completo al instante de cada jugada que modifique el historial.
El algoritmo funciona según las incidencias:
- **Nivel Medio:** Ocurre si el usuario ha superado 100 UCM Coins de perdidas **O** si ha encadenado al menos 2 derrotas en línea. En este estado, el jugador entra en etapa de concientización y el sistema lanza cuadros de confirmación severos antes de dejarle abrir la ventana de BlackJack ("¿Estás seguro que deseas continuar?").
- **Nivel Alto:** Ocurre cuando el entorno se sale de control: Si el jugador ha perdido ya más de 200 UCM Coins en toda su vida virtual **O** lleva 4 o más derrotas sin éxito consecutivas. En este momento, el botón de blackjack sufre una restricción a nivel vista. En el intento de entrar, la aplicación lo bloquea por completo (`btnPlayBlackjack` detecta el Riesgo ALTO y deniega acceso), protegiendo permanentemente al jugador y recordándole que visite la sección de Recursos Educativos.
