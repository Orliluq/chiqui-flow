# 🤖 ChiquiFlow 3 — Agente IA con Telegram, n8n, Memoria y Bases de Datos

---

## 📌 Descripción del proyecto

ChiquiFlow 3 es un sistema de automatización inteligente construido en n8n, que conecta un agente conversacional con IA a Telegram, integrando:

- 🧠 Memoria por usuario (session ID)  
- 🗄️ Base de datos PostgreSQL (Neon)  
- 📊 Google Sheets como alternativa de persistencia  
- 🧰 Herramientas tipo RAG (Vector Store + Embeddings)  
- 💬 Telegram como canal de interacción  
- 🤖 LLM Cohere como motor de lenguaje  

El objetivo es crear un agente real en producción, capaz de mantener contexto, consultar datos estructurados y responder en lenguaje natural.

---

## 🧱 Arquitectura general

### Diagrama de flujo

<div style="text-align: center;">

```text
    Telegram Trigger
    ↓
    AI Agent (LangChain n8n)
    ↓        ↓         ↓
    Memory   Tools    Model (Cohere)
    ↓        ↓
    Postgres / Sheets / Vector Store
    ↓
    Telegram Response
```

</div>

---
## ⚙️ Tecnologías utilizadas

- 🧩 n8n (Orquestación de workflows)
- 🤖 Cohere (LLM + Embeddings)
- 💾 Neon (PostgreSQL cloud)
- 📊 Google Sheets API
- 🧠 LangChain Nodes en n8n
- 💬 Telegram Bot API

---
## 📡 Flujo 1 — ChiquiFlow con Neon (PostgreSQL)

### 🔹 Objetivo

Agente con memoria + consulta de empleados en base de datos.

### Componentes del flujo

#### 1. Telegram Trigger

Captura mensajes del usuario:

```json
{{ $json.message.text }}
```

#### 2. AI Agent (LangChain)

Nodo central del sistema:

- Recibe el input del usuario
- Decide si usar memoria o herramientas
- Genera respuesta final

#### 3. Cohere Chat Model

Modelo de lenguaje principal:

- Generación de respuestas
- Razonamiento del agente

#### 4. Simple Memory (Buffer Window)

```json
sessionKey: {{ $json.message.from.id }}
```

📌 **Función:**

- Mantener contexto por usuario
- Separar conversaciones por Telegram ID

#### 5. Vector Store (RAG interno)

Motor de búsqueda semántica

Base de conocimiento tipo RH:

- vacaciones
- horarios
- políticas internas

#### 6. Embeddings Cohere

```text
embed-multilingual-light-v3.0
```

📌 Convierte texto en vectores para búsqueda semántica.

#### 7. PostgreSQL Tool (Neon)

Consulta empleados:

```sql
SELECT * FROM empleados
WHERE nombre LIKE '%valor%'
```

📌 **Configuración clave:**

```json
sessionKey: {{ $json.message.from.id }}
```

#### 8. Telegram Response

```json
chatId: {{ $('Telegram Trigger').item.json.message.chat.id }}
text: {{ $json.output }}
```

---
## 📡 Flujo 2 — ChiquiFlow con Google Sheets

### 🔹 Objetivo

Reemplazo de PostgreSQL usando Google Sheets como base de datos ligera.

### Componentes del flujo

#### 1. Telegram Trigger

Igual que en flujo anterior.

#### 2. AI Agent + Cohere

Mismo motor de razonamiento.

#### 3. Simple Memory

```json
sessionKey: {{ $json.message.chat.id }}
```

📌 **Diferencia:**

aquí se usa chat.id en vez de user.id

#### 4. Google Sheets Tool

📊 **Documento:**

```text
ID del documento
```

📌 **Hoja:**

```text
tabla_empleados (gid=0)
```

📌 **Filtro:**

```text
nombre LIKE %valor%
```

#### 5. Vector Store + Embeddings

Igual que flujo Neon:

- búsqueda semántica
- contexto RAG

#### 6. Telegram Response

Entrega respuesta final al usuario.

---

## ⚠️ Problemas encontrados y soluciones

### ❌ Railway fallando (DB issues)

**✔ Solución:**

- migración a Neon PostgreSQL
- conexión más estable en cloud

---

## 🚀 Resultado final

El sistema permite:

- 💬 Conversación natural en Telegram
- 🧠 Memoria por usuario
- 📊 Consulta de empleados (Postgres o Sheets)
- 🔍 Búsqueda semántica (Vector Store)
- 🤖 IA con Cohere
- ⚙️ Arquitectura modular y escalable