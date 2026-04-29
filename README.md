# 🏦 Guía Completa del Taller: Azure AI Agents con Microsoft Foundry

> **Nivel**: Desde principiante hasta experto  
> **Duración estimada**: 4-6 horas  
> **Idioma**: Español  

---

## 📋 Tabla de Contenidos

1. [Antes de Empezar: ¿Qué es un Agente de IA?](#1-antes-de-empezar-qué-es-un-agente-de-ia)
2. [Arquitectura General del Taller](#2-arquitectura-general-del-taller)
3. [Lab 1: Tu Primer Agente — Asesor Financiero Básico](#3-lab-1-tu-primer-agente--asesor-financiero-básico)
4. [Lab 2: Code Interpreter — Agente que Ejecuta Código Python](#4-lab-2-code-interpreter--agente-que-ejecuta-código-python)
5. [Lab 3: File Search — Agente que Busca en Documentos](#5-lab-3-file-search--agente-que-busca-en-documentos)
6. [Lab 4: Bing Grounding — Agente con Búsqueda Web en Tiempo Real](#6-lab-4-bing-grounding--agente-con-búsqueda-web-en-tiempo-real)
7. [Lab 5: AI Search — Agente con Búsqueda Vectorial Semántica](#7-lab-5-ai-search--agente-con-búsqueda-vectorial-semántica)
8. [Lab 6: Multi-Agent Workflows — Orquestación de Múltiples Agentes](#8-lab-6-multi-agent-workflows--orquestación-de-múltiples-agentes)
9. [Lab 7: MCP Tools — Agente con Foundry MCP Server](#9-lab-7-mcp-tools--agente-con-foundry-mcp-server)
10. [Lab 8: Foundry IQ — Agente con Múltiples Fuentes de Conocimiento](#10-lab-8-foundry-iq--agente-con-múltiples-fuentes-de-conocimiento)
11. [Lab 9: Memory Search — Agente con Memoria Persistente](#11-lab-9-memory-search--agente-con-memoria-persistente)
12. [Mapa de Progresión: De Principiante a Experto](#12-mapa-de-progresión-de-principiante-a-experto)
13. [Glosario de Términos Clave](#13-glosario-de-términos-clave)
14. [Troubleshooting Común](#14-troubleshooting-común)

---

## 1. Antes de Empezar: ¿Qué es un Agente de IA?

### Para quien nunca ha visto esto antes

Imagina que tienes un asistente virtual super inteligente. Le das instrucciones de cómo comportarse (por ejemplo: "eres un asesor financiero") y le das **herramientas** (acceso a documentos, calculadora, buscador web, etc.). El agente puede:

- **Entender** lo que el usuario pregunta
- **Decidir** qué herramienta usar para responder
- **Ejecutar** acciones (buscar documentos, hacer cálculos, buscar en internet)
- **Responder** con información útil y contextual

### El Patrón Fundamental

```
┌───────────┐     ┌──────────────────────┐     ┌───────────────────┐
│  Usuario  │────▶│  Agente de IA        │────▶│  Herramientas     │
│ (pregunta)│     │  (instrucciones +    │     │  • Code Interpreter│
│           │◀────│   modelo LLM)        │◀────│  • File Search    │
│ (respuesta│     │                      │     │  • Bing Search    │
└───────────┘     └──────────────────────┘     │  • AI Search      │
                                                │  • MCP Tools      │
                                                │  • Memory         │
                                                └───────────────────┘
```

### Componentes Clave que usaremos en TODO el taller

| Componente | ¿Qué es? | Analogía Simple |
|---|---|---|
| **AIProjectClient** | El "control remoto" para manejar todo en Microsoft Foundry | Es como la llave de tu casa: te da acceso a todo |
| **Agente (Agent)** | La "personalidad" + instrucciones + herramientas | Es el cerebro con superpoderes |
| **Conversación (Conversation)** | El hilo de chat donde usuario y agente se comunican | Es como un chat de WhatsApp |
| **Herramienta (Tool)** | Capacidad extra que le damos al agente | Son los superpoderes del cerebro |
| **Credencial** | La forma de autenticarnos con Azure | Es como tu contraseña del banco |

### Requisitos Previos

1. **Azure CLI** instalado y autenticado (`az login --use-device-code`)
2. **Un proyecto en Microsoft AI Foundry** con endpoint configurado
3. **Archivo `.env`** con las variables de entorno necesarias
4. **Python 3.10+** con las dependencias instaladas (`pip install -r requirements.txt`)

---

## 2. Arquitectura General del Taller

Este taller sigue una **progresión de complejidad creciente**. Cada lab agrega una capacidad nueva:

```
Lab 1: Agente Básico ──────────────────────────────── Nivel: ⭐
  └── Solo texto, sin herramientas

Lab 2: + Code Interpreter ─────────────────────────── Nivel: ⭐⭐
  └── El agente puede ejecutar código Python

Lab 3: + File Search ──────────────────────────────── Nivel: ⭐⭐
  └── El agente busca en documentos subidos (RAG)

Lab 4: + Bing Grounding ───────────────────────────── Nivel: ⭐⭐
  └── El agente busca información en tiempo real en internet

Lab 5: + AI Search (vectorial) ────────────────────── Nivel: ⭐⭐⭐
  └── Búsqueda semántica avanzada con embeddings

Lab 6: Multi-Agent Workflows ──────────────────────── Nivel: ⭐⭐⭐⭐
  └── Múltiples agentes trabajando juntos con YAML

Lab 7: MCP Tools ──────────────────────────────────── Nivel: ⭐⭐⭐
  └── Agente conectado a servicios externos vía MCP

Lab 8: Foundry IQ ────────────────────────────────── Nivel: ⭐⭐⭐⭐⭐
  └── Múltiples fuentes de conocimiento con enrutamiento inteligente

Lab 9: Memory Search ─────────────────────────────── Nivel: ⭐⭐⭐⭐
  └── Agente que recuerda preferencias entre conversaciones
```

### La API que usamos en TODOS los Labs

Todos los notebooks siguen el mismo patrón de 4 pasos:

```python
# 1. CONECTAR - Crear el cliente del proyecto
project_client = AIProjectClient(endpoint=..., credential=...)

# 2. CREAR AGENTE - Definir personalidad + herramientas
agent = project_client.agents.create_version(
    agent_name="mi-agente",
    definition=PromptAgentDefinition(
        model="gpt-4o",
        instructions="Eres un asesor financiero...",
        tools=[...]  # <-- herramientas opcionales
    )
)

# 3. CONVERSAR - Crear conversación y enviar mensajes
openai_client = project_client.get_openai_client()
conversation = openai_client.conversations.create()
response = openai_client.responses.create(
    conversation=conversation.id,
    input="Mi pregunta...",
    extra_body={"agent": {"name": agent.name, "version": agent.version, "type": "agent_reference"}}
)

# 4. LIMPIAR - Borrar el agente cuando termines
project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
```

> **💡 PUNTO CLAVE**: Este patrón de 4 pasos se repite en CADA lab. Lo que cambia son las **herramientas** y las **instrucciones** del agente.

---

## 3. Lab 1: Tu Primer Agente — Asesor Financiero Básico

📓 **Notebook**: `1-basics.ipynb`  
⭐ **Nivel**: Principiante  
⏱️ **Duración**: 20-30 min

### ¿Qué estamos construyendo?

Un agente de IA que actúa como **asesor financiero de un banco minorista**. No tiene herramientas extra — solo usa el conocimiento del modelo de lenguaje (GPT-4o) para responder preguntas sobre banca, préstamos e inversiones.

### ¿Por qué empezamos con esto?

Porque necesitas entender los **fundamentos** antes de agregar complejidad:
- Cómo conectarse a Azure
- Cómo crear un agente
- Cómo manejar conversaciones
- Cómo funciona el ciclo de vida de un agente

### Paso a paso: ¿Qué hace cada celda?

#### Celda 1-2: Autenticación 🔐

```
Usuario → az login → Azure AD → Token de acceso → AIProjectClient
```

**¿Qué está pasando?** Nos autenticamos con Azure usando `InteractiveBrowserCredential`. Esto abre tu navegador para que inicies sesión. El `AIProjectClient` es el objeto principal que usaremos para TODO.

> **⚠️ IMPORTANTE**: Si estás en una red corporativa, usa `az login --use-device-code` primero en la terminal.

#### Celda 3: Crear el Agente 🤖

**¿Qué está pasando?** Creamos un agente con `create_version()`. Lo más importante son las **instrucciones** (el `instructions`), que definen la personalidad y las reglas del agente.

**Concepto clave — Instructions (System Prompt):**
```
"Eres un asesor financiero amigable. Siempre:
1. Incluye disclaimers regulatorios
2. Anima a consultar con asesores profesionales
3. Explica conceptos como APR, tasas de interés en lenguaje simple"
```

Esto es lo que hace que el agente sea un "asesor financiero" y no un chatbot genérico. **Las instrucciones son el alma del agente.**

#### Celda 4: Crear Conversación 💬

**¿Qué está pasando?** Creamos un "hilo de chat" (`conversation`). Cada conversación tiene un ID único y mantiene el historial de mensajes.

> **Analogía**: Es como abrir un nuevo chat en WhatsApp. El agente recuerda todo lo que se dijo en ESA conversación.

#### Celda 5-6: Enviar Mensajes y Obtener Respuestas

**¿Qué está pasando?** Usamos `openai_client.responses.create()` para enviar preguntas al agente y recibir respuestas. El `extra_body` con `agent_reference` le dice al sistema QUÉ agente debe responder.

Se hacen 4 pruebas:
1. Pregunta sobre **hipotecas** (factores que afectan la tasa)
2. Pregunta sobre **score crediticio** (cómo se calcula y mejora)
3. Pregunta sobre **tipos de cuentas de ahorro** (savings vs CD vs money market)
4. Pregunta **compleja** (escenario de compra de vivienda con datos específicos)

#### Celda 7: Conversación con Contexto

**¿Qué está pasando?** Aquí se demuestra una conversación **multi-turno** donde se pasa el `conversation_id`. Esto permite que el agente recuerde lo que ya se habló.

### Lo más importante de este Lab

| Concepto | Por qué importa |
|---|---|
| **`AIProjectClient`** | Es tu punto de entrada a TODO en Microsoft Foundry |
| **`create_version()`** | Crea un agente versionado (puedes tener v1, v2, etc.) |
| **Instructions** | Definen la personalidad, reglas y comportamiento del agente |
| **`conversations.create()`** | Crea un hilo de chat que mantiene el historial |
| **`responses.create()`** | Envía un mensaje y recibe la respuesta del agente |
| **Cleanup** | SIEMPRE borra tus agentes al terminar para no generar costos |

---

## 4. Lab 2: Code Interpreter — Agente que Ejecuta Código Python

📓 **Notebook**: `2-code-interpreter.ipynb`  
⭐⭐ **Nivel**: Principiante-Intermedio  
⏱️ **Duración**: 30-40 min

### ¿Qué estamos construyendo?

Un **agente calculador de préstamos** que puede ejecutar código Python en un sandbox seguro. El agente puede hacer cálculos financieros complejos, generar tablas de amortización y analizar portafolios de préstamos.

### ¿Qué diferencia hay con el Lab 1?

En el Lab 1 el agente solo "hablaba". Ahora le damos la capacidad de **escribir y ejecutar código Python**. Es como darle una calculadora científica al asesor.

```
Lab 1: Agente ──▶ Responde con texto
Lab 2: Agente ──▶ Escribe código Python ──▶ Lo ejecuta ──▶ Responde con resultados
```

### La herramienta estrella: `CodeInterpreterTool`

```python
from azure.ai.projects.models import CodeInterpreterTool, CodeInterpreterToolAuto

code_interpreter_tool = CodeInterpreterTool(container=CodeInterpreterToolAuto())
```

**¿Qué hace esto?** Le da al agente un **entorno aislado (sandbox)** donde puede ejecutar código Python. El agente decide cuándo necesita hacer cálculos y escribe el código automáticamente.

> **🔒 Seguridad**: El código se ejecuta en un contenedor aislado, no en tu máquina. No puede acceder a tu sistema de archivos ni a internet.

### Paso a paso: ¿Qué hace cada celda?

#### Celda 1: Setup + Datos de Ejemplo

Se crean **datos sintéticos** de un portafolio de préstamos (7 préstamos con diferentes tipos, montos y estados). Esto simula datos reales de un banco.

#### Celda 2: Crear Agente con Code Interpreter

**Lo clave**: El agente se crea con `tools=[code_interpreter_tool]`. Sus instrucciones incluyen la **fórmula de amortización**:

$$M = P \times \frac{r(1+r)^n}{(1+r)^n - 1}$$

Donde: M = pago mensual, P = principal, r = tasa mensual, n = número de pagos.

> **💡 ¿Por qué incluimos la fórmula en las instrucciones?** Porque queremos que el agente use la fórmula correcta, no que improvise. Esto es una buena práctica: ser específico en lo que quieres que el agente calcule.

#### Celda 3: Cálculo de Préstamo

El agente recibe: principal=$350,000, tasa=6.5%, plazo=30 años. Él **escribe código Python** para:
1. Calcular el pago mensual
2. Calcular el total pagado a lo largo del préstamo
3. Calcular el total de intereses
4. Mostrar los primeros 12 meses del cronograma de amortización

#### Celda 4: Comparación de Opciones de Préstamo

Se comparan 3 opciones de hipoteca (30 años vs 15 años vs 20 años). El agente genera una **tabla comparativa** calculada con código.

#### Celda 5: Análisis de Riesgo del Portafolio

El agente analiza el portafolio de préstamos completo: valor total, tasa promedio ponderada, préstamos en riesgo (morosos), tasa de morosidad y recomendaciones.

### Lo más importante de este Lab

| Concepto | Por qué importa |
|---|---|
| **CodeInterpreterTool** | Permite al agente ejecutar código Python en un sandbox seguro |
| **Sandbox aislado** | El código se ejecuta de forma segura, sin acceso a tu sistema |
| **Fórmulas en instrucciones** | Ser específico en las instrucciones mejora la precisión |
| **Datos sintéticos** | En un taller usamos datos de ejemplo; en producción usarías datos reales |

> **🎯 PARA RECORDAR**: Code Interpreter es ideal cuando necesitas que el agente haga **cálculos matemáticos, análisis de datos o genere gráficos**. No es solo para chatear, es para resolver problemas computacionales.

---

## 5. Lab 3: File Search — Agente que Busca en Documentos

📓 **Notebook**: `3-file-search.ipynb`  
⭐⭐ **Nivel**: Principiante-Intermedio  
⏱️ **Duración**: 30-40 min

### ¿Qué estamos construyendo?

Un agente que puede **buscar información dentro de documentos** que le subimos. En este caso, documentos de políticas bancarias y guías de cumplimiento regulatorio.

### Concepto Fundamental: RAG (Retrieval-Augmented Generation)

```
┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│  Documentos  │────▶│ Vector Store  │────▶│   Agente     │
│  (archivos)  │     │ (embeddings)  │     │  busca aquí  │
└──────────────┘     └───────────────┘     │  cuando le   │
                                            │  preguntan   │
                                            └──────────────┘
```

**RAG** significa que el agente **no inventa respuestas** — primero busca en tus documentos y luego responde basándose en lo que encontró. Esto es FUNDAMENTAL para aplicaciones empresariales donde necesitas respuestas basadas en información real.

### Paso a paso: ¿Qué hace cada celda?

#### Celda 1: Setup y Autenticación

Se usa `DefaultAzureCredential` (en vez de `InteractiveBrowserCredential`). Esta es la opción recomendada para producción porque intenta múltiples métodos de autenticación automáticamente.

#### Celda 2: Crear Documentos de Ejemplo

Se crean 2 archivos markdown:
- **`loan_policies.md`**: Políticas de préstamos (requisitos de hipotecas, autos, personales, negocios)
- **`compliance_guidelines.md`**: Guías de cumplimiento regulatorio (TILA, Fair Lending, KYC, AML)

> **💡 En producción**: Estos serían documentos reales de tu organización: manuales, políticas internas, contratos, etc.

#### Celda 3: Crear Vector Store

**¿Qué es un Vector Store?** Es una base de datos especial que almacena tus documentos como **vectores numéricos** (embeddings). Esto permite buscar por **significado** en lugar de por palabras exactas.

```python
# 1. Crear el vector store vacío
vs = openai_client.vector_stores.create(name="banking_policies_store")

# 2. Subir archivos al vector store
file = openai_client.vector_stores.files.upload_and_poll(
    vector_store_id=vs.id,
    file=open("loan_policies.md", "rb")
)
```

> **⚠️ ERRORES COMUNES**: 
> - **403 Forbidden**: Necesitas el rol "Storage Blob Data Contributor" en la cuenta de almacenamiento del proyecto
> - **Timeout**: Los archivos grandes pueden tardar en procesarse

#### Celda 4: Crear Agente con FileSearchTool

```python
from azure.ai.projects.models import FileSearchTool

tool = FileSearchTool(vector_store_ids=[vector_store.id])
agent = project_client.agents.create_version(
    agent_name="banking-document-search-agent",
    definition=PromptAgentDefinition(
        model="gpt-4o-mini",
        instructions="Busca siempre en los documentos antes de responder...",
        tools=[tool]
    )
)
```

**La diferencia clave**: El agente ahora puede buscar dentro de tus documentos y citar las fuentes.

#### Celda 5: Hacer Preguntas de Búsqueda

Se hacen 3 preguntas:
1. "¿Cuál es el score crediticio mínimo para una hipoteca?" → Busca en loan_policies.md
2. "¿Cuáles son los requisitos de KYC?" → Busca en compliance_guidelines.md
3. "¿Qué documentación necesito para un préstamo comercial > $100,000?" → Busca en loan_policies.md

### Lo más importante de este Lab

| Concepto | Por qué importa |
|---|---|
| **Vector Store** | Almacena documentos como vectores para búsqueda semántica |
| **FileSearchTool** | La herramienta que le da al agente acceso a los documentos |
| **RAG** | El patrón de buscar primero y responder después |
| **upload_and_poll()** | Sube archivos y espera a que se procesen |
| **Permisos** | Necesitas "Storage Blob Data Contributor" en el storage account |

> **🎯 PARA RECORDAR**: File Search es ideal cuando tienes **documentos internos** (manuales, políticas, FAQs) y quieres que el agente responda basándose en ellos, NO en conocimiento general.

---

## 6. Lab 4: Bing Grounding — Agente con Búsqueda Web en Tiempo Real

📓 **Notebook**: `4-bing-grounding.ipynb`  
⭐⭐ **Nivel**: Intermedio  
⏱️ **Duración**: 20-30 min

### ¿Qué estamos construyendo?

Un agente de **investigación de mercados financieros** que puede buscar información **en tiempo real** en internet usando Bing. Puede encontrar tasas de interés actuales, noticias financieras y tendencias económicas.

### ¿Cuándo usar Bing Grounding vs File Search?

| Necesidad | Herramienta |
|---|---|
| Información de **tus documentos internos** | File Search (Lab 3) |
| Información **actualizada de internet** | Bing Grounding (Lab 4) |
| **Ambas** | Puedes combinar ambas en un mismo agente |

### Prerequisito Especial

Necesitas configurar una **conexión de Bing** en el portal de Microsoft AI Foundry:

1. Ir a Azure Portal → Crear recurso "Grounding with Bing Search"
2. En AI Foundry → Management Center → Connected Resources → Añadir la conexión de Bing
3. Poner el nombre de la conexión en tu `.env` como `GROUNDING_WITH_BING_CONNECTION_NAME`

### Paso a paso: ¿Qué hace cada celda?

#### Celda 1: Setup

Se cargan las variables de entorno y se inicializa el cliente.

#### Celda 2: Crear Agente con BingGroundingTool

```python
from azure.ai.projects.models import (
    BingGroundingAgentTool,
    BingGroundingSearchToolParameters,
    BingGroundingSearchConfiguration,
)

# 1. Obtener la conexión de Bing del proyecto
bing_connection = project_client.connections.get(name=bing_conn_name)

# 2. Crear la herramienta
bing_tool = BingGroundingAgentTool(
    bing_grounding=BingGroundingSearchToolParameters(
        search_configurations=[
            BingGroundingSearchConfiguration(
                project_connection_id=bing_connection.id
            )
        ]
    )
)

# 3. Crear agente con la herramienta
agent = project_client.agents.create_version(
    agent_name="financial-market-research-agent",
    definition=PromptAgentDefinition(
        model="gpt-4o-mini",
        instructions="Usa Bing para encontrar tasas actuales, noticias...",
        tools=[bing_tool]
    )
)
```

**¿Qué está pasando internamente?** Cuando le haces una pregunta al agente, él:
1. Detecta que necesita información actualizada
2. Formula una búsqueda en Bing
3. Lee los resultados
4. Responde citando las fuentes

#### Celda 3: Hacer Preguntas de Investigación

Se hacen 3 preguntas sobre mercados financieros en tiempo real:
1. Decisiones de la Reserva Federal sobre tasas de interés
2. Tendencias del sector bancario y movimientos de acciones
3. Pronósticos de inflación

### Lo más importante de este Lab

| Concepto | Por qué importa |
|---|---|
| **BingGroundingTool** | Permite al agente buscar en internet en tiempo real |
| **Conexión de Bing** | Debe configurarse en el portal de AI Foundry primero |
| **Citaciones** | El agente incluye URLs y fuentes de donde sacó la info |
| **Modelos compatibles** | No todos los modelos soportan Bing (ej: `gpt-4o-0513` sí) |

> **🎯 PARA RECORDAR**: Bing Grounding es ideal cuando necesitas **información actualizada** que cambia frecuentemente (precios, noticias, tasas). No lo uses para información estática interna.

---

## 7. Lab 5: AI Search — Agente con Búsqueda Vectorial Semántica

📓 **Notebook**: `5-agents-aisearch.ipynb`  
⭐⭐⭐ **Nivel**: Intermedio  
⏱️ **Duración**: 40-50 min

### ¿Qué estamos construyendo?

Un **asesor de productos bancarios** que busca en un catálogo de productos financieros usando **Azure AI Search con búsqueda vectorial**. Es como tener un buscador inteligente que entiende lo que quieres aunque no uses las palabras exactas.

### ¿Cuál es la diferencia con File Search (Lab 3)?

| Aspecto | File Search (Lab 3) | AI Search (Lab 5) |
|---|---|---|
| **Datos** | Archivos subidos (docs, PDFs) | Índice de búsqueda estructurado |
| **Escalabilidad** | Archivos pequeños/medianos | Millones de documentos |
| **Control** | Bajo (Azure lo maneja todo) | Alto (defines esquema, vectorización) |
| **Caso de uso** | Documentos internos simples | Catálogos, productos, datos empresariales |
| **Vectorización** | Automática | Manual (tú generas embeddings) |

### Concepto Clave: Embeddings y Búsqueda Vectorial

```
Texto: "tarjeta sin comisión anual" 
    ↓ (modelo de embeddings)
Vector: [0.023, -0.451, 0.178, ..., 0.892]  (1536 dimensiones)

Búsqueda: "quiero una tarjeta barata" 
    ↓ (se convierte a vector)
    ↓ (se buscan vectores similares)
Resultado: "No-Fee Cash Back Card" ✅ (alta similitud vectorial)
```

> **💡 ¿Por qué funciona?** Porque "sin comisión" y "barata" tienen significados similares en el espacio vectorial, aunque las palabras sean diferentes.

### Paso a paso: ¿Qué hace cada celda?

#### Celda 1: Setup y Conexiones

Se inicializan TRES clientes diferentes:
- `AIProjectClient` — para crear agentes
- `SearchIndexClient` — para crear índices de búsqueda
- `AzureOpenAI` — para generar embeddings

#### Celda 2: Crear Índice de Búsqueda

Se crea un índice llamado `bankingproductsindex` con:
- Campos tradicionales: `ProductID`, `Name`, `Category`, `APR`, `AnnualFee`, `Description`
- Campo vectorial: `DescriptionVector` (1536 dimensiones)
- Algoritmo HNSW para búsqueda rápida de vecinos cercanos
- Configuración semántica para mejorar relevancia

#### Celda 3: Subir Productos con Embeddings

Se suben 7 productos bancarios (tarjetas de crédito, hipotecas, cuentas de ahorro, préstamos). Para cada producto:
1. Se combina nombre + categoría + descripción en un texto
2. Se genera un embedding (vector de 1536 números)
3. Se sube al índice con el vector incluido

#### Celda 4: Crear Agente con AzureAISearchTool

```python
from azure.ai.projects.models import (
    AzureAISearchAgentTool,
    AzureAISearchToolResource,
    AISearchIndexResource,
    AzureAISearchQueryType,
)

tool = AzureAISearchAgentTool(
    azure_ai_search=AzureAISearchToolResource(
        indexes=[AISearchIndexResource(
            project_connection_id=ai_search_connection_name,
            index_name="bankingproductsindex",
            query_type=AzureAISearchQueryType.SEMANTIC,  # Búsqueda semántica
        )]
    )
)
```

> **⚠️ PREREQUISITO**: Necesitas crear una conexión de AI Search en el portal de AI Foundry con **autenticación por API Key** (no AAD/Entra ID) para evitar problemas de permisos.

#### Celda 5: Probar el Agente

Se hacen preguntas como "Busco una tarjeta de crédito sin comisión anual y con buen cashback". El agente busca en el índice y recomienda productos específicos con APR, comisiones y características.

### Lo más importante de este Lab

| Concepto | Por qué importa |
|---|---|
| **Embeddings** | Representaciones numéricas de texto que capturan significado |
| **HNSW** | Algoritmo eficiente para búsqueda de vecinos cercanos |
| **Búsqueda Semántica** | Encuentra resultados por significado, no solo por palabras |
| **AzureAISearchTool** | Herramienta oficial del SDK para integrar AI Search con agentes |
| **Conexión API Key** | Usar API Key (no AAD) en la conexión para evitar errores 403 |

---

## 8. Lab 6: Multi-Agent Workflows — Orquestación de Múltiples Agentes

📓 **Notebook**: `6-multi-agent-solution-with-workflows.ipynb`  
⭐⭐⭐⭐ **Nivel**: Avanzado  
⏱️ **Duración**: 40-50 min

### ¿Qué estamos construyendo?

Un **sistema de procesamiento de reclamos de seguros** con CUATRO agentes especializados que trabajan juntos, orquestados por un **workflow declarativo en YAML**.

### ¿Por qué múltiples agentes?

Porque los problemas complejos se resuelven mejor dividiéndolos en partes especializadas:

```
                    ┌──────────────────┐
                    │  Reclamo de      │
                    │  Seguro          │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
    ┌─────────────┐ ┌──────────────┐ ┌──────────────┐
    │  Agente de  │ │  Agente de   │ │  Agente de   │
    │  Validez    │ │  Departamento│ │  Estimación  │
    │  ¿Es válido?│ │  ¿A quién va?│ │  ¿Cuánto?    │
    └──────┬──────┘ └──────┬───────┘ └──────┬───────┘
           │               │               │
           └───────────────┼───────────────┘
                           ▼
                 ┌──────────────────┐
                 │  Orquestador     │
                 │  Reporte Final   │
                 └──────────────────┘
```

### Concepto Clave: WorkflowAgentDefinition + YAML

En vez de orquestar agentes con código Python (programáticamente), usamos un **archivo YAML declarativo** que define el flujo:

```yaml
kind: workflow
trigger:
  kind: OnConversationStart
  actions:
    - kind: InvokeAzureAgent    # 1. Evaluar validez
      agent:
        name: claim-validity-agent
    - kind: InvokeAzureAgent    # 2. Asignar departamento
      agent:
        name: department-assignment-agent
    - kind: InvokeAzureAgent    # 3. Estimar pago
      agent:
        name: payout-estimation-agent
    - kind: InvokeAzureAgent    # 4. Sintetizar todo
      agent:
        name: claims-orchestrator-agent
    - kind: EndConversation     # 5. Terminar
```

> **💡 ¿Por qué YAML y no código?** Porque el YAML es **declarativo** — describes QUÉ quieres, no CÓMO hacerlo. Es más fácil de leer, mantener y modificar. Además, permite que personas no-técnicas entiendan el flujo.

### Paso a paso: ¿Qué hace cada celda?

#### Celda 1-4: Definir Instrucciones de Cada Agente Especialista

Cada agente tiene un rol muy específico:

| Agente | Responsabilidad | Output |
|---|---|---|
| **Validez** | ¿El reclamo es válido según la póliza? | Valid / Requires Review / Denied |
| **Departamento** | ¿Qué departamento maneja esto? | Auto / Home / Life / Health / Commercial |
| **Estimación** | ¿Cuánto se pagaría? | Low (<$5K) / Medium ($5K-$25K) / High (>$25K) |
| **Orquestador** | Sintetiza todo en un reporte final | Reporte estructurado completo |

#### Celda 5: Crear los 4 Agentes

Se crean los 4 agentes con `create_version()`. Todos usan `PromptAgentDefinition` (son agentes basados en instrucciones, sin herramientas extra).

#### Celda 6: Definir el Workflow YAML

El YAML define la secuencia: `Validez → Departamento → Estimación → Orquestador → Fin`.

**Detalle clave**: Todos los agentes comparten la **misma conversación**, así que el orquestador puede ver todo lo que dijeron los demás.

#### Celda 7: Crear el Workflow Agent

```python
workflow_agent = project_client.agents.create_version(
    agent_name="claims-processing-workflow",
    definition=WorkflowAgentDefinition(workflow=workflow_yaml),
)
```

> **Nota**: `WorkflowAgentDefinition` es diferente de `PromptAgentDefinition`. No tiene `model` ni `instructions` — su comportamiento viene del YAML.

#### Celda 8: Ejecutar el Workflow con Streaming

Se procesan 2 reclamos de ejemplo:
1. **Reclamo de auto**: Colisión en intersección, $3,200 de daños
2. **Reclamo de casa**: Daños por tubería rota, $15,000

El streaming permite ver cada acción del workflow en tiempo real.

### Lo más importante de este Lab

| Concepto | Por qué importa |
|---|---|
| **WorkflowAgentDefinition** | Crea un agente-orquestador basado en YAML |
| **YAML declarativo** | Define flujos de trabajo sin código |
| **Agentes especializados** | Divide problemas complejos en roles específicos |
| **Streaming** | Permite ver el progreso del workflow en tiempo real |
| **Conversación compartida** | Todos los agentes ven el historial completo |

> **🎯 PARA RECORDAR**: Multi-Agent Workflows es ideal para **procesos empresariales complejos** donde diferentes pasos requieren diferentes expertise. Piensa en: aprobación de préstamos, onboarding de clientes, revisión de documentos.

---

## 9. Lab 7: MCP Tools — Agente con Foundry MCP Server

📓 **Notebook**: `7-mcp-tools.ipynb`  
⭐⭐⭐ **Nivel**: Intermedio  
⏱️ **Duración**: 30-40 min

### ¿Qué estamos construyendo?

Un agente que puede acceder a las **capacidades de Microsoft Foundry** a través del protocolo MCP (Model Context Protocol). El agente puede explorar modelos disponibles, gestionar deployments, ejecutar evaluaciones y trabajar con índices de búsqueda.

### ¿Qué es MCP?

**MCP (Model Context Protocol)** es un protocolo estándar que permite a agentes de IA conectarse a **servicios externos** de forma segura. Piensa en MCP como un "USB" universal para herramientas de IA.

```
┌─────────────┐     ┌──────────────┐     ┌────────────────────┐
│   Agente    │────▶│  MCP Tool    │────▶│  Foundry MCP Server│
│             │◀────│  (conector)  │◀────│  mcp.ai.azure.com  │
└─────────────┘     └──────────────┘     └────────────────────┘
                                                    │
                                         ┌──────────┼──────────┐
                                         ▼          ▼          ▼
                                    Catálogo    Evaluaciones  Agentes
                                    de Modelos                Existentes
```

### Prerequisito: Configurar Conexión MCP en el Portal

1. Ir a [Microsoft Foundry](https://ai.azure.com) → Tu proyecto → **Tools**
2. Click **Add tool** → **Catalog** → Buscar "Foundry MCP Server (preview)"
3. Click **Create** → Se crea con autenticación OAuth
4. Copiar el **Project Connection ID** y ponerlo en `.env` como `FOUNDRY_MCP_CONNECTION_ID`

### Paso a paso: ¿Qué hace cada celda?

#### Celda 1-2: Setup y Autenticación

Se usa `AzureCliCredential` para autenticación.

#### Celda 3: Configurar MCPTool

```python
from azure.ai.projects.models import MCPTool

mcp_tool = MCPTool(
    server_label="foundry-mcp",
    server_url="https://mcp.ai.azure.com/sse",  # Endpoint SSE
    require_approval="never",                     # Sin aprobación manual
    project_connection_id=foundry_mcp_connection_id,
)
```

**Parámetros importantes**:
- `server_url`: El endpoint del servidor MCP (siempre `https://mcp.ai.azure.com/sse`)
- `require_approval`: `"never"` para demos, `"always"` para producción
- `project_connection_id`: La conexión configurada en el portal

#### Celda 4-5: Crear Agente y Conversación

El agente tiene instrucciones para ser un "Asistente de Desarrollo de IA" que puede explorar modelos, ayudar con deployments y evaluaciones.

#### Celda 6: Flujo de Aprobación MCP

**Concepto importante**: Cuando `require_approval="always"`, el servidor MCP puede enviar **solicitudes de aprobación** antes de ejecutar una acción. El código maneja esto con `McpApprovalResponse`:

```python
# Si el agente pide usar una herramienta MCP, aprobamos automáticamente
for item in response.output:
    if item.type == "mcp_approval_request":
        approval_requests.append(McpApprovalResponse(
            type="mcp_approval_response",
            approve=True,
            approval_request_id=item.id,
        ))
```

#### Celda 7: Consultas al Agente

Se hacen preguntas como:
- "¿Qué modelos recomiendas para análisis de documentos?"
- "¿Qué capacidades tiene el Foundry MCP Server?"
- "¿Qué características de seguridad ofrece para uso empresarial?"

### Herramientas disponibles en Foundry MCP Server

| Categoría | Herramientas | Uso |
|---|---|---|
| **Modelos** | `model_catalog_list`, `model_deploy`, `model_quota_list` | Explorar y desplegar modelos |
| **Evaluaciones** | `evaluation_create`, `evaluation_get` | Ejecutar evaluaciones de calidad |
| **Agentes** | `agent_update`, `agent_list` | Gestionar agentes existentes |
| **Conocimiento** | `index_list`, `query_index` | Trabajar con Azure AI Search |

### Lo más importante de este Lab

| Concepto | Por qué importa |
|---|---|
| **MCP Protocol** | Estándar abierto para conectar agentes a servicios externos |
| **MCPTool** | La clase del SDK que configura la conexión MCP |
| **Approval Flow** | Mecanismo de seguridad para aprobar acciones del agente |
| **Cloud-hosted** | No necesitas instalar un servidor local |
| **Entra ID Auth** | Autenticación segura con identidad corporativa |

---

## 10. Lab 8: Foundry IQ — Agente con Múltiples Fuentes de Conocimiento

📓 **Notebook**: `8-foundry-IQ-agents.ipynb`  
⭐⭐⭐⭐⭐ **Nivel**: Experto  
⏱️ **Duración**: 50-60 min

### ¿Qué estamos construyendo?

Un **agente de investigación de fraude** que consulta automáticamente TRES fuentes de conocimiento especializadas a través de **Foundry IQ**, la capa unificada de conocimiento de Azure.

### ¿Qué es Foundry IQ?

**Foundry IQ** es la plataforma de RAG (Retrieval-Augmented Generation) de siguiente generación de Microsoft:

```
┌─────────────────────────────────────────────────────────────┐
│                      FOUNDRY IQ                              │
│              (Capa Unificada de Conocimiento)                │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Patrones de  │  │ Cumplimiento │  │ Procedimientos│      │
│  │ Fraude       │  │ Regulatorio  │  │ de Invest.   │      │
│  │ (Índice 1)   │  │ (Índice 2)   │  │ (Índice 3)   │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         └─────────────────┼─────────────────┘               │
│                           ▼                                  │
│              ┌─────────────────────┐                        │
│              │   Knowledge Base    │                        │
│              │ (Router Inteligente)│ ← Enruta automáticamente│
│              └─────────────────────┘   a la fuente correcta │
│                           │                                  │
│                           ▼                                  │
│              ┌─────────────────────┐                        │
│              │ Agente de Fraude    │                        │
│              └─────────────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

**La magia**: El agente NO necesita saber en cuál fuente buscar. Foundry IQ **enruta automáticamente** la consulta a la(s) fuente(s) correcta(s).

### Conceptos nuevos en este Lab

| Concepto | ¿Qué es? |
|---|---|
| **Knowledge Source** | Una fuente de datos (índice) expuesta al sistema |
| **Knowledge Base** | El router inteligente que conecta múltiples fuentes |
| **Agentic Retrieval** | La IA planifica, busca y sintetiza entre fuentes |
| **MCP Endpoint** | El Knowledge Base se expone como endpoint MCP |

### Paso a paso: ¿Qué hace cada celda?

#### Paso 1-2: Setup y Configuración

Se requieren más variables de entorno que en labs anteriores:
- `AZURE_AI_SEARCH_ENDPOINT` — endpoint de AI Search
- `AZURE_OPENAI_ENDPOINT` — endpoint de OpenAI para embeddings
- `PROJECT_RESOURCE_ID` — ID completo del recurso del proyecto
- `EMBEDDING_MODEL_DEPLOYMENT_NAME` — modelo de embeddings (text-embedding-3-large)

#### Paso 3: Generar Datos Sintéticos

Se crean **24 documentos** distribuidos en 3 categorías:
1. **Patrones de fraude** (8 docs): Fraude de identidad sintética, takeover de cuentas, redes de mulas, BEC, etc.
2. **Cumplimiento regulatorio** (8 docs): BSA, SARs, KYC, OFAC, CTR, protección al consumidor, etc.
3. **Procedimientos de investigación** (8 docs): Triage de alertas, congelamiento de cuentas, entrevistas, evidencia, etc.

> **💡 En producción**: Estos serían documentos reales de tu departamento de fraude.

#### Paso 4: Crear 3 Índices de Búsqueda

Se crean 3 índices vectoriales independientes, cada uno con campos `id`, `title`, `content` y `content_vector` (3072 dimensiones con text-embedding-3-large).

#### Paso 5: Generar Embeddings y Subir Documentos

Se generan embeddings para cada documento y se suben a sus respectivos índices.

#### Paso 6: Crear Knowledge Sources

```python
from azure.search.documents.indexes.models import (
    SearchIndexKnowledgeSource,
    SearchIndexKnowledgeSourceParameters,
)

ks = SearchIndexKnowledgeSource(
    name="fraud-patterns-ks",
    description="Patrones de fraude, banderas rojas, estrategias de detección",
    search_index_parameters=SearchIndexKnowledgeSourceParameters(
        search_index_name="fraud-patterns-index",
        source_data_fields=[...]
    )
)
```

**La descripción es crucial**: Foundry IQ usa la descripción para decidir a cuál fuente enrutar cada consulta.

#### Paso 7: Crear Knowledge Base

```python
from azure.search.documents.indexes.models import KnowledgeBase

knowledge_base = KnowledgeBase(
    name="fraud-investigation-kb",
    knowledge_sources=[ref1, ref2, ref3],  # Las 3 fuentes
    output_mode=KnowledgeRetrievalOutputMode.EXTRACTIVE_DATA,
)
```

#### Paso 8: Crear Conexión MCP

La Knowledge Base se expone como un **endpoint MCP** al que el agente se conecta. Se crea una conexión en el proyecto usando `ProjectManagedIdentity`.

#### Paso 9: Crear Agente de Fraude

El agente usa `MCPTool` para conectarse al Knowledge Base y tiene instrucciones detalladas para citar fuentes.

#### Paso 10: Probar con Consultas Multi-Fuente

Se hacen 4 preguntas que demuestran el enrutamiento inteligente:

| Pregunta | Fuente esperada |
|---|---|
| "¿Red flags de fraude de identidad sintética?" | Patrones de Fraude |
| "¿Cuándo debo reportar un SAR?" | Cumplimiento Regulatorio |
| "¿Procedimiento para congelar una cuenta sospechosa?" | Procedimientos |
| "Encontré una red de mulas — ¿patrones, regulaciones y pasos?" | **LAS TRES FUENTES** |

### Lo más importante de este Lab

| Concepto | Por qué importa |
|---|---|
| **Foundry IQ** | RAG de siguiente generación con enrutamiento inteligente |
| **Knowledge Source** | Abstracción sobre índices de búsqueda |
| **Knowledge Base** | Router inteligente entre múltiples fuentes |
| **Enrutamiento automático** | No necesitas especificar dónde buscar |
| **Agregación multi-fuente** | Una sola pregunta puede buscar en todas las fuentes |
| **Roles requeridos** | Necesitas Search Index Data Reader en la managed identity del proyecto |

> **🎯 PARA RECORDAR**: Foundry IQ es para escenarios donde tienes **múltiples fuentes de conocimiento** y quieres que el agente sea lo suficientemente inteligente para saber dónde buscar automáticamente.

---

## 11. Lab 9: Memory Search — Agente con Memoria Persistente

📓 **Notebook**: `9-agent-memory-search.ipynb`  
⭐⭐⭐⭐ **Nivel**: Avanzado  
⏱️ **Duración**: 40-50 min

### ¿Qué estamos construyendo?

Un **asesor bancario personalizado** que **recuerda las preferencias del cliente** entre conversaciones. Si en la sesión 1 dices "soy conservador con inversiones", en la sesión 2 el agente ya lo sabe.

### ¿Por qué importa la memoria?

| Sin Memoria | Con Memoria |
|---|---|
| "¿Cuál es su tolerancia al riesgo?" (cada vez) | "Basándome en su perfil conservador..." |
| Recomendaciones genéricas | Recomendaciones personalizadas |
| Sin contexto de sesiones anteriores | "La última vez hablamos de su fondo de emergencia..." |
| Experiencia impersonal | Experiencia como un asesor que te conoce |

### Arquitectura de la Memoria

```
Conversación 1                        Conversación 2
"Soy conservador,                     "¿En qué debería invertir?"
 ahorro para jubilación"                        │
         │                                      ▼
         ▼                              ┌──────────────┐
┌──────────────┐                        │ Memory Search│
│ Memory Store │ ──────────────────────▶│ (recupera    │
│ (extrae y    │   Datos persistentes   │  memorias)   │
│  guarda)     │                        └──────────────┘
└──────────────┘                                │
                                                ▼
                                   "Dado su perfil conservador
                                    y meta de jubilación en 20
                                    años, recomendaría bonos..."
```

### Paso a paso: ¿Qué hace cada celda?

#### Paso 1-2: Setup y Clientes

Se usan las mismas variables de entorno que otros labs, más `EMBEDDING_MODEL_DEPLOYMENT_NAME` para la memoria.

#### Paso 3: Crear Memory Store

```python
from azure.ai.projects.models import (
    MemoryStoreDefaultDefinition,
    MemoryStoreDefaultOptions,
)

memory_definition = MemoryStoreDefaultDefinition(
    chat_model=model_deployment,          # Modelo para extraer memorias
    embedding_model=embedding_deployment,  # Modelo para buscar memorias
    options=MemoryStoreDefaultOptions(
        user_profile_enabled=True,   # Extraer preferencias del usuario
        chat_summary_enabled=True    # Resumir conversaciones
    )
)

memory_store = project_client.memory_stores.create(
    name="banking-customer-memory-store",
    definition=memory_definition
)
```

**¿Qué hace el Memory Store?**
- **user_profile_enabled**: Extrae automáticamente preferencias y rasgos (ej: "tolerancia al riesgo: conservador")
- **chat_summary_enabled**: Crea resúmenes de conversaciones pasadas

#### Paso 4: Configurar Memory Search Tool

```python
from azure.ai.projects.models import MemorySearchTool

memory_tool = MemorySearchTool(
    memory_store_name=memory_store.name,
    scope=customer_id,      # Aísla memorias por cliente
    update_delay=5           # Segundos de inactividad antes de guardar
)
```

**`scope` es fundamental**: Cada cliente tiene sus propias memorias aisladas. En producción usarías el ID real del cliente.

#### Paso 5: Crear Agente con Memoria

El agente se crea con `tools=[memory_tool]`. Sus instrucciones le dicen que:
- Referencie memorias previas ("Basándome en su preferencia por inversiones de bajo riesgo...")
- Pregunte sobre preferencias si no las recuerda
- Personalice recomendaciones según el perfil guardado

#### Paso 6: Conversación 1 — Establecer Preferencias

El "cliente" comparte su perfil:
- 45 años, inversiones conservadoras
- Ahorrando para jubilación en 20 años
- Quiere fondo de emergencia de 6 meses
- Ingreso: $120,000, puede invertir $1,500/mes

#### Paso 7: Esperar Extracción de Memorias

Se espera ~30 segundos para que el sistema extraiga y guarde las memorias automáticamente.

#### Paso 8: Conversación 2 — Probar Recall de Memoria

Se crea una **conversación NUEVA** y se hacen preguntas como "¿Qué me recomiendas invertir?". El agente debería responder usando las preferencias recordadas:
- "Dado su perfil conservador..."
- "Considerando su timeline de jubilación de 20 años..."
- "Con su capacidad de $1,500/mes..."

#### Paso 9-10: Actualizar Preferencias y Verificar

El cliente actualiza su perfil: "Ahora puedo manejar riesgo moderado" y "Mi ingreso subió a $140,000". Se verifica que la memoria se actualice correctamente.

### Lo más importante de este Lab

| Concepto | Por qué importa |
|---|---|
| **Memory Store** | Almacenamiento persistente de preferencias y resúmenes |
| **MemorySearchTool** | Herramienta que busca y actualiza memorias automáticamente |
| **Scope** | Aísla memorias por usuario (privacidad) |
| **update_delay** | Tiempo de inactividad antes de extraer memorias |
| **User Profile** | Extracción automática de preferencias del texto de la conversación |
| **Chat Summary** | Resúmenes automáticos de conversaciones pasadas |

> **🎯 PARA RECORDAR**: Memory Search es ideal para **experiencias personalizadas** donde el contexto del usuario importa entre sesiones. Wealth management, servicio al cliente, procesamiento de préstamos, seguros.

### Consideraciones para Producción

| Aspecto | Recomendación |
|---|---|
| **update_delay** | Usar 300+ segundos (5 min) para no extraer durante conversaciones activas |
| **Scope** | Usar IDs de cliente reales o `{{$userId}}` para aislamiento automático |
| **Retención** | Considerar políticas de retención de datos y GDPR |
| **Datos sensibles** | Evitar almacenar PII o datos financieros sensibles en memorias |

---

## 12. Mapa de Progresión: De Principiante a Experto

### Nivel 1: Fundamentos (Labs 1-2)
- ✅ Conectarse a Azure y crear un agente básico
- ✅ Manejar conversaciones y respuestas
- ✅ Agregar Code Interpreter para cálculos

### Nivel 2: RAG y Búsqueda (Labs 3-5)
- ✅ Subir documentos y buscar en ellos (File Search)
- ✅ Conectar a búsqueda web en tiempo real (Bing)
- ✅ Crear índices vectoriales con embeddings (AI Search)

### Nivel 3: Orquestación (Labs 6-7)
- ✅ Coordinar múltiples agentes con YAML (Workflows)
- ✅ Conectar a servicios externos vía MCP

### Nivel 4: Maestría (Labs 8-9)
- ✅ Múltiples fuentes de conocimiento con enrutamiento inteligente (Foundry IQ)
- ✅ Memoria persistente entre conversaciones (Memory Search)

---

## 13. Glosario de Términos Clave

| Término | Definición |
|---|---|
| **Agente (Agent)** | Entidad de IA con instrucciones, modelo y herramientas que puede realizar tareas |
| **AIProjectClient** | Cliente principal del SDK para interactuar con Microsoft Foundry |
| **Conversation** | Hilo de chat que mantiene historial entre usuario y agente |
| **Code Interpreter** | Herramienta que permite al agente ejecutar código Python en sandbox |
| **Embedding** | Representación numérica (vector) de texto que captura su significado |
| **File Search** | Herramienta para buscar dentro de documentos subidos a un vector store |
| **Foundry IQ** | Plataforma de conocimiento unificada con enrutamiento inteligente |
| **Knowledge Base** | Colección de fuentes de conocimiento con router inteligente |
| **Knowledge Source** | Fuente de datos individual expuesta a una knowledge base |
| **MCP** | Model Context Protocol — estándar para conectar agentes a servicios |
| **Memory Store** | Almacenamiento persistente de preferencias y resúmenes de conversaciones |
| **PromptAgentDefinition** | Definición de agente basada en instrucciones (system prompt) |
| **RAG** | Retrieval-Augmented Generation — buscar primero, responder después |
| **Vector Store** | Base de datos que almacena documentos como vectores para búsqueda semántica |
| **WorkflowAgentDefinition** | Definición de agente orquestador basada en YAML |

---

## 14. Troubleshooting Común

### Error 403 Forbidden al subir archivos
**Causa**: Falta el rol "Storage Blob Data Contributor" en la cuenta de almacenamiento.  
**Solución**: Ir a Azure Portal → Storage Account → Access Control (IAM) → Agregar rol.

### Error de autenticación
**Causa**: Token expirado o no autenticado.  
**Solución**: Ejecutar `az login --use-device-code` en la terminal.

### Error al crear agente: "Model not found"
**Causa**: El deployment del modelo no existe o el nombre es incorrecto.  
**Solución**: Verificar `AZURE_AI_MODEL_DEPLOYMENT_NAME` en el `.env`.

### Bing Grounding no funciona
**Causa**: Conexión de Bing no configurada en AI Foundry.  
**Solución**: Ir a AI Foundry → Management Center → Connected Resources → Agregar Bing.

### AI Search Tool devuelve "Access denied"
**Causa**: La conexión usa AAD en vez de API Key, o faltan permisos.  
**Solución**: Crear conexión con API Key en AI Foundry. Asignar rol "Search Index Data Reader".

### Memory Search no recuerda nada
**Causa**: No se esperó suficiente tiempo para la extracción de memorias.  
**Solución**: Aumentar `update_delay` y esperar al menos 30 segundos después de la conversación.

### Foundry IQ: Knowledge Base no enruta correctamente
**Causa**: Descripciones de Knowledge Sources vagas o ambiguas.  
**Solución**: Usar descripciones claras y específicas que diferencien bien cada fuente.

### Variable de entorno no encontrada
**Causa**: El archivo `.env` no está en la ubicación correcta.  
**Solución**: El `.env` debe estar en la raíz del proyecto (2 niveles arriba de los notebooks).

---

## 🏁 Conclusión

En este taller, has construido **9 agentes de IA** con capacidades progresivamente más avanzadas:

1. **Conversación básica** → La base de todo
2. **Ejecución de código** → Cálculos y análisis
3. **Búsqueda en documentos** → RAG con tus propios datos
4. **Búsqueda web** → Información en tiempo real
5. **Búsqueda vectorial** → Semántica avanzada con embeddings
6. **Multi-agente** → Orquestación declarativa
7. **MCP** → Conexión a servicios externos
8. **Foundry IQ** → Múltiples fuentes con routing inteligente
9. **Memoria** → Personalización persistente

Cada lab agrega una pieza del rompecabezas. Juntas, forman un toolkit completo para construir aplicaciones de IA empresariales con Microsoft Foundry.

> **💡 Próximo paso**: Revisa el folder `agent-framework/` para ver cómo implementar estos mismos conceptos usando el **Microsoft Agent Framework SDK** con más control programático.

---

*Guía creada para el taller de Azure AI Agents — Microsoft Foundry*
