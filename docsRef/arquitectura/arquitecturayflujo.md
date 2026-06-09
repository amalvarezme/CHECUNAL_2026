# Arquitectura y Flujo de Operación: Simulador de Criticidad CHEC

Este documento describe la arquitectura multi-agente y el flujo de información del Simulador de Criticidad, basado en los conceptos del contexto del proyecto (`ContextoProyectoSimuladorCHEC.md`). El sistema emplea agentes de IA de diferentes capacidades (Low/High), técnicas de Recuperación de Información (RAG) para análisis documental, y un modelo predictivo pre-entrenado para analizar causas de fallas y simular escenarios de mejora en redes de distribución eléctrica de Nivel de Tensión 2.

---

## 1. Flujo de Información y Análisis Multimodal

El proceso integra tanto datos estructurados (tabulares) como datos no estructurados (texto de normativas y bitácoras), asegurando un análisis causal profundo y fundamentado:

1. **Selección del Evento Inicial:** El usuario selecciona una fila específica de la Base de Datos de Eventos tabulares.
2. **Traducción Semántica:** El sistema toma los datos crudos y, utilizando los diccionarios, genera una primera descripción en lenguaje natural del evento (entorno, infraestructura, impacto demográfico).
3. **Análisis Retrospectivo Tabular (12 Meses):** El sistema consulta el historial de la base de datos para recuperar todos los eventos asociados al mismo **vano** (y circuito) durante el año anterior a la falla, analizando fluctuaciones en los indicadores UiTI, SAIDI y SAIFI.
4. **Análisis Documental y Normativo (Nuevo):** En paralelo al paso 3, el sistema consulta un repositorio de documentos (PDFs, textos, docs). Revisa:
   - **Bitácoras de Intervenciones:** Extrae el registro de mantenimientos programados (ej. podas, reposiciones) e intervenciones de mitigación no programadas asociadas a la zona en el último año.
   - **Normativa de Sistemas de Potencia:** Extrae los lineamientos técnicos aplicables a la infraestructura afectada en la falla seleccionada.
5. **Diagnóstico Preliminar:** Se consolida el evento actual con el historial tabular y el registro documental para plantear hipótesis. Por ejemplo: *¿Hubo una falla por vegetación a pesar de que la bitácora reporta una poda hace 3 meses?*
6. **Inferencia del Modelo Predictivo:** Un modelo de IA predictiva procesa las variables tabulares del evento actual y entrega:
   - La predicción matemática de los indicadores (SAIDI, SAIFI, UiTi).
   - **Máscaras de relevancia (Feature Importance):** Valores de 0 a 1 indicando qué características tienen mayor peso estadístico en la falla.
7. **Cotejo Analítico a Tres Vías (Razonamiento Cruzado):** El sistema realiza un cruce crítico y justifica las causas de falla combinando:
   - Los patrones matemáticos/temporales (paso 3).
   - Las justificaciones cualitativas de las bitácoras y normas (paso 4).
   - Las máscaras matemáticas del modelo ML (paso 6).
8. **Identificación de Escenarios Guiados por Normativa:** Basado en el cotejo, se presenta una lista filtrada de las variables candidatas a intervenir. **Las opciones son guiadas por las bitácoras y la norma** (ej. si la norma exige conductor semiaislado en zonas de alta vegetación, se sugiere cambiar el conductor en el simulador).
9. **Simulación "*What-If*":** El usuario modifica los valores de las variables sugeridas en la interfaz.
10. **Reevaluación Predictiva:** El modelo predictivo procesa el nuevo escenario y proyecta los nuevos valores de UiTi, SAIDI y SAIFI, confirmando si la intervención cumple técnica y estadísticamente con la mejora esperada.

---

## 2. Arquitectura de Agentes de IA

La arquitectura se expande para manejar capacidades multimodales (RAG + Datos Estructurados + ML Tradicional), utilizando agentes especializados con modelos acordes a su carga cognitiva.

### Agente 1: Descriptor de Contexto (Modelo *Low/Fast*)
- **Rol:** Traductor Semántico.
- **Función:** Convierte la fila de la base de datos seleccionada en un resumen narrativo de estado y condiciones iniciales.

### Agente 2: Analista Histórico Estructurado (Modelo *High/Reasoning*)
- **Rol:** Analista Forense de Datos.
- **Función:** Analiza el historial de datos numéricos/tabulares del último año. Identifica estacionalidad, acumulación de estrés ambiental y patrones recurrentes de degradación en UiTi, SAIDI y SAIFI.

### Agente 3: Analista Documental y RAG (Modelo *High/Context-Heavy*)
- **Rol:** Especialista en Normativa y Operaciones.
- **Función:** Conectado a una base de datos vectorial (Vector Store). Ingresa palabras clave de la falla (circuito, vano, fecha, causa) para "leer" bitácoras de mantenimiento, reportes de poda e incidencias no programadas. Coteja si el mantenimiento se hizo según la norma técnica vigente.

### Orquestador del Modelo Predictivo (API / No LLM)
- **Rol:** Puente de Inferencia ML.
- **Función:** Llama al modelo pre-entrenado y retorna las predicciones y el tensor de máscaras de relevancia de las variables.

### Agente 4: Consultor de Criticidad y Causalidad (Modelo *High/Reasoning*)
- **Rol:** Investigador Principal.
- **Función:** El núcleo de síntesis. Recibe el análisis matemático (Agente 2), el contexto cualitativo (Agente 3) y las máscaras (Orquestador). Construye la narrativa causal justificando el fallo. *Ejemplo:* "El modelo predictivo da peso de 0.92 al viento. El Agente 2 nota historial de fallas en invierno. El Agente 3 encontró una bitácora que indica que la poda programada fue pospuesta. Causa: Caída de rama por viento debido a retraso en mantenimiento."

### Agente 5: Simulador y Evaluador de Escenarios (Modelo *Low/Fast*)
- **Rol:** Asistente Interactiva "What-If".
- **Función:** Gestiona la simulación con el usuario, guiando los cambios de parámetros basado en las recomendaciones normativas del Agente 4, y ejecutando la reevaluación con el modelo predictivo.

---

## 3. Diagrama de Arquitectura Multimodal y Flujo

El siguiente gráfico ilustra la interacción ampliada, integrando el flujo de repositorios documentales y razonamiento cruzado.

```mermaid
graph TD
    %% Definición de Estilos
    classDef user fill:#f9f2f4,stroke:#333,stroke-width:2px;
    classDef agentLow fill:#d9edf7,stroke:#31708f,stroke-width:2px;
    classDef agentHigh fill:#dff0d8,stroke:#3c763d,stroke-width:2px;
    classDef db fill:#fcf8e3,stroke:#8a6d3b,stroke-width:2px;
    classDef model fill:#e2e2ea,stroke:#666,stroke-width:2px,stroke-dasharray: 5 5;
    classDef doc fill:#e2eedd,stroke:#5cb85c,stroke-width:2px;

    %% Nodos
    User([Usuario / Operador]):::user
    DB[(Base de Datos Tabular)]:::db
    DocStore[(Repositorio Documental RAG\nNormas, PDFs, Bitácoras)]:::doc
    Contexto[Contexto.md KB]:::db
    
    A1[Agente 1: Descriptor\nModelo Low]:::agentLow
    A2[Agente 2: Analista Tabular\nModelo High]:::agentHigh
    A3[Agente 3: Analista Documental\nModelo High]:::agentHigh
    A4[Agente 4: Consultor Causalidad\nModelo High]:::agentHigh
    A5[Agente 5: Simulador Escenarios\nModelo Low]:::agentLow
    
    MP{{Modelo Predictivo\nPre-entrenado}}:::model

    %% Flujo Inicial
    User -->|1. Selecciona Evento| A1
    Contexto -.->|Diccionarios y Modos| A1
    Contexto -.->|Reglas Sistema| A4
    
    A1 -->|2. Descripción| A2
    A1 -->|2. Filtros Búsqueda| A3
    
    %% Flujo Histórico y Documental (Paralelo)
    A2 -->|3a. Consulta 12 Meses| DB
    DB -->|Historial SAIDI/SAIFI/UiTi| A2
    
    A3 -->|3b. Búsqueda Textos| DocStore
    DocStore -->|Normativa y Bitácoras| A3
    
    %% Inferencia ML
    A1 -->|Fila Actual| MP
    MP -->|4. Máscaras Relevancia (0-1)| A4
    
    %% Cotejo Centralizado
    A2 -->|Patrones Cuantitativos| A4
    A3 -->|Justificación Cualitativa| A4
    
    A4 -->|5. Cotejo a 3 Vías: Causas| A4
    A4 -->|6. Opciones Guiadas por Norma| A5
    A5 -->|Muestra Opciones Intervención| User
    
    %% Ciclo de Simulación
    User -->|7. Modifica Valores (What-If)| A5
    A5 -->|Envía Nuevo Escenario| MP
    MP -->|Retorna Nuevos Indicadores| A5
    A5 -->|8. Reporte Impacto vs Norma| User
```

## Resumen del Ciclo de Valor Ampliado
1. **Comprender:** Desde la taxonomía numérica (`Agente 1`).
2. **Contextualizar Multimodalmente:** Cruzando el comportamiento histórico numérico (`Agente 2`) con la "historia operativa" en las bitácoras y las reglas de ingeniería en las normas (`Agente 3`).
3. **Validar Causas:** Contrastando el raciocinio humano/LLM cualitativo con las inferencias estadísticas y matemáticas (`Agente 4` + `Modelo Predictivo`).
4. **Actuar Inteligentemente:** Diseñando escenarios de simulación que no solo busquen un mejor indicador numérico, sino que estén fundamentados y respaldados por la normativa técnica y las realidades operativas descritas en campo (`Agente 5`).
