# Arquitectura y Flujo de Operación: Simulador de Criticidad CHEC

Este documento describe la arquitectura multi-agente y el flujo de información del Simulador de Criticidad, basado en los conceptos del contexto del proyecto (`ContextoProyectoSimuladorCHEC.md`). El sistema emplea agentes de IA de diferentes capacidades (Low/High) trabajando en conjunto con un modelo predictivo pre-entrenado para analizar causas de fallas y simular escenarios de mejora en redes de distribución eléctrica de Nivel de Tensión 2.

---

## 1. Flujo de Información y Análisis

El proceso de simulación y análisis se divide en una secuencia lógica que va desde la observación de un evento pasado hasta la simulación predictiva de intervenciones:

1. **Selección del Evento Inicial:** El usuario interactúa con la interfaz del sistema y selecciona una fila específica de la Base de Datos de Eventos.
2. **Traducción Semántica:** El sistema toma los datos tabulares crudos de esa fila y, utilizando el conocimiento en `ContextoProyectoSimuladorCHEC.md`, genera una primera descripción en lenguaje natural del evento (ej. entorno, infraestructura afectada, impacto demográfico).
3. **Análisis Retrospectivo (12 Meses):** El sistema consulta el historial de la base de datos para recuperar todos los eventos asociados al mismo **vano** (y circuito) durante el año anterior a la fecha de la falla seleccionada.
4. **Estudio de Criticidad Histórica:** Se evalúan los puntos críticos en esa ventana temporal enfocándose en los indicadores **UiTI, SAIDI y SAIFI**. Se identifican fluctuaciones y cambios en las variables del entorno, clima y operativas registradas.
5. **Diagnóstico y Patrones:** Se compara el evento actual con el historial para identificar las causas más probables de la falla, revelando patrones de comportamiento o debilidades estructurales del circuito estudiado.
6. **Inferencia del Modelo Predictivo:** En paralelo, un modelo de IA predictiva (previamente entrenado) evalúa el evento actual. Este modelo entrega:
   - La predicción de las variables de interés (SAIDI, SAIFI, UiTi).
   - **Máscaras de relevancia (Feature Importance):** Valores entre 0 y 1 para cada variable, indicando cuáles características tienen mayor peso estadístico en el resultado de la falla.
7. **Cotejo Analítico (Razonamiento Cruzado):** El sistema cruza y contrasta el diagnóstico histórico elaborado en el paso 5, con las máscaras de relevancia matemáticas emitidas por el modelo en el paso 6, validando las hipótesis bajo las reglas del contexto del proyecto.
8. **Identificación de Variables Candidatas:** Basado en el cotejo analítico, se presenta al usuario una lista filtrada de los **Modos** (ej. Físicas, Entorno, Activos) y **Variables** específicas que tienen más potencial de mitigar la falla si fuesen intervenidas.
9. **Simulación "*What-If*":** El usuario modifica los valores de las variables sugeridas (simulando, por ejemplo, podas de vegetación, cambio de conductor a semiaislado, instalación de reconectadores).
10. **Reevaluación Predictiva:** El modelo predictivo procesa el nuevo escenario y proyecta los nuevos valores de UiTi, SAIDI y SAIFI. El sistema concluye si la intervención sugerida presenta una mejora estadísticamente significativa de los indicadores.

---

## 2. Arquitectura de Agentes de IA

Para materializar este flujo, la arquitectura se divide en agentes especializados, optimizando el consumo computacional usando modelos veloces ("Low") para tareas de ruteo/traducción, y modelos con alta capacidad de razonamiento ("High") para inferencia profunda.

### Agente 1: Descriptor de Contexto (Modelo *Low/Fast*)
- **Rol:** Traductor Semántico.
- **Función:** Recibe la fila seleccionada por el usuario y los diccionarios de `ContextoProyectoSimuladorCHEC.md`. Convierte la taxonomía técnica en un resumen narrativo de fácil lectura sobre el estado del circuito en el "instante cero" del fallo.

### Agente 2: Analista Histórico y Topológico (Modelo *High/Reasoning*)
- **Rol:** Investigador Forense de Datos.
- **Función:** Genera las consultas (Queries) para extraer el año de historial del vano. Analiza las series de tiempo, variaciones climáticas y topológicas. Su objetivo es encontrar la historia detrás de la degradación (ej. acumulación de lluvias, reiteradas fallas menores previas).

### Orquestador del Modelo Predictivo (API / No LLM)
- **Rol:** Puente de Inferencia.
- **Función:** No es un LLM, sino un componente de software que empaqueta las entradas, llama al modelo de Machine Learning pre-entrenado, y retorna las predicciones y el tensor de máscaras de relevancia [0-1].

### Agente 3: Consultor de Criticidad y Causalidad (Modelo *High/Reasoning*)
- **Rol:** Analista Experto / Juez.
- **Función:** Es el núcleo del sistema. Recibe el reporte del Agente 2 y las máscaras del Orquestador Predictivo. Realiza el "Cotejo Analítico". Si el modelo predictivo le da un valor de `0.9` al `WIND_GUST` (viento), y el Agente 2 notó un incremento de ráfagas en el historial, el Agente 3 concluye causalidad y recomienda al usuario intervenir variables relacionadas a resistencia estructural o podas (`NR_T`).

### Agente 4: Simulador y Evaluador de Escenarios (Modelo *Low/Fast*)
- **Rol:** Asistente Interactiva "What-If".
- **Función:** Gestiona la interfaz con el usuario para la simulación. Toma los valores modificados por el usuario, los envía al Orquestador Predictivo, y genera una comparativa entre el "Antes" y el "Después" de la intervención sobre los indicadores UiTi, SAIDI y SAIFI, emitiendo una recomendación final de rentabilidad o viabilidad.

---

## 3. Diagrama de Arquitectura y Flujo

El siguiente gráfico ilustra la interacción entre el usuario, los agentes de IA, la Base de Datos y el Modelo Predictivo.

```mermaid
graph TD
    %% Definición de Estilos
    classDef user fill:#f9f2f4,stroke:#333,stroke-width:2px;
    classDef agentLow fill:#d9edf7,stroke:#31708f,stroke-width:2px;
    classDef agentHigh fill:#dff0d8,stroke:#3c763d,stroke-width:2px;
    classDef db fill:#fcf8e3,stroke:#8a6d3b,stroke-width:2px;
    classDef model fill:#e2e2ea,stroke:#666,stroke-width:2px,stroke-dasharray: 5 5;

    %% Nodos
    User([Usuario / Operador]):::user
    DB[(Base de Datos Eventos)]:::db
    Contexto[Contexto.md KB]:::db
    
    A1[Agente 1: Descriptor\nModelo Low]:::agentLow
    A2[Agente 2: Analista Histórico\nModelo High]:::agentHigh
    A3[Agente 3: Consultor Causalidad\nModelo High]:::agentHigh
    A4[Agente 4: Simulador Escenarios\nModelo Low]:::agentLow
    
    MP{{Modelo Predictivo\nPre-entrenado}}:::model

    %% Flujo Inicial
    User -->|1. Selecciona Fila Evento| A1
    Contexto -.->|Diccionarios y Modos| A1
    Contexto -.->|Reglas Físico/Eléctricas| A3
    
    A1 -->|2. Descripción Semántica| A2
    A2 -->|3. Consulta 12 Meses (Vano)| DB
    DB -->|Historial UiTI/SAIDI/SAIFI| A2
    
    %% Flujo Analítico
    A2 -->|4. Análisis Patrones Históricos| A3
    A1 -->|Fila Actual| MP
    MP -->|5. Predicción + Máscaras Relevancia (0-1)| A3
    
    %% Cotejo y What If
    A3 -->|6. Cotejo: Histórico vs Máscaras| A3
    A3 -->|7. Sugiere Lista Variables Candidatas| A4
    A4 -->|Muestra Opciones Intervención| User
    
    %% Ciclo de Simulación
    User -->|8. Modifica Valores (What-If)| A4
    A4 -->|Envía Nuevo Escenario| MP
    MP -->|Retorna Nuevos UiTi/SAIDI/SAIFI| A4
    A4 -->|9. Reporte Comparativo Impacto| User
```

## Resumen del Ciclo de Valor
1. **Comprender:** A partir de datos crudos (`Agente 1`).
2. **Contextualizar:** Revisando la historia temporal de los activos y el entorno (`Agente 2`).
3. **Validar:** Cruzando el razonamiento humano/LLM con las matemáticas puras de inferencia predictiva (`Agente 3` + `Modelo Predictivo`).
4. **Actuar:** Otorgando al tomador de decisiones la capacidad de simular soluciones y probar su impacto en la calidad de la red antes de ejecutarlas físicamente (`Agente 4`).
