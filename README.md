# n8n-workflows-b-valeria
Automatizaciones con n8n: Google Sheets, Gmail, WhatsApp, Asana y más.
# n8n-workflows

Colección de workflows (export .json) para n8n: automatizaciones con Google Workspace, Asana/Teams, WhatsApp + OpenAI, y un generador de contenido con IA.


---

## Workflows incluidos

### 1) AgenteWS (`AgenteWS.json`)
**Qué hace:** Webhook que recibe mensajes (tipo WhatsApp via proveedor/API), detecta si el mensaje es **texto / audio / imagen**, procesa según el tipo y responde con un **AI Agent (OpenAI)**.  
- Trigger: **Webhook (POST)** con `path` definido en el workflow.  
- Routing: **Switch** por `Tipo_mensaje` (text/audio/image).  
- Imagen: descarga media → analiza con **OpenAI (gpt-4o)**.  
- Audio: descarga/convierte → transcribe → pasa al agente.  
- Respuesta: envía el texto de salida a una API HTTP para responder al usuario (número + texto).  
Referencias: Webhook path y Switch por tipo:contentReference[oaicite:0]{index=0}, descarga/analiza imagen + agente:contentReference[oaicite:1]{index=1}:contentReference[oaicite:2]{index=2}, envío final por HTTP Request:contentReference[oaicite:3]{index=3}.

---

### 2) Google Calendar desde Google Sheets (duplicado en 2 archivos)
#### a) `Automate Event Creation in Google Calendar from Google Sheets.json`
#### b) `calendar-event-automation.json`
**Qué hace:** Cuando se agrega una fila en un Google Sheet, formatea la fecha y crea un evento en Google Calendar.  
- Trigger: **Google Sheets Trigger** (rowAdded, polling).  
- Transformación: **Code** (“Event Date Formatter”).  
- Acción: **Google Calendar** → crear evento (all-day, summary, description, location, color).  
Referencias: trigger + formatter + calendar creator:contentReference[oaicite:4]{index=4}:contentReference[oaicite:5]{index=5}.  
Descripción equivalente en el archivo alternativo:contentReference[oaicite:6]{index=6}.

> Nota: ambos `.json` representan el mismo workflow (mismo nombre/idea). Podés quedarte con uno solo para evitar duplicados.

---

### 3) Cumpleaños (`Cumpleaños.json`)
**Qué hace:** Todos los días a las 08:00, lee una Google Sheet y envía un email a quienes cumplen años hoy.  
- Trigger: **Cron** diario 08:00.  
- Data: **Google Sheets** (Get Many Rows).  
- Lógica: **Function** filtra por `Fecha Cumpleaños` (MM-DD).  
- Acción: **Email Send (SMTP)** con asunto “Feliz cumpleaños…”.  
Referencias: nodos y lógica:contentReference[oaicite:7]{index=7}.

---

### 4) Tareas vencidas Asana → Teams (`daily-overdue-asana-to-teams.json`)
**Qué hace:** Lun–Vie 09:00 consulta tareas de Asana, detecta vencidas, arma un resumen agrupado por responsable y publica en un canal de Microsoft Teams si hay vencidas.  
- Trigger: **Cron** (weekday 1–5, 09:00).  
- Data: **HTTP Request** a Asana `/tasks` usando `ASANA_TOKEN` y `ASANA_PROJECT_ID`.  
- Lógica: **Code** arma mensaje + cuenta.  
- Condición: **IF** `totalOverdue > 0`.  
- Acción: **Microsoft Teams** post a channel (`TEAMS_TEAM_ID`, `TEAMS_CHANNEL_ID`).  
Referencias: cron + Asana HTTP + env vars:contentReference[oaicite:8]{index=8}, IF + Teams post + timezone settings:contentReference[oaicite:9]{index=9}:contentReference[oaicite:10]{index=10}.

---

### 5) Digestor de Datos (clínica) (`Digestor_de_Datos(clinica).json`)
**Qué hace:** Flujo con **Chat Trigger + AI Agent** con memoria en Postgres y búsqueda/ingesta en **Supabase Vector Store**. Incluye pipeline para descargar un PDF desde Google Drive y cargarlo a un vector store con embeddings.  
- Trigger: **When chat message received**.  
- IA: **AI Agent** + **OpenAI Chat Model (gpt-4.1-mini)**.  
- Memoria: **Postgres Chat Memory**.  
- RAG: **Supabase Vector Store** como herramienta (retrieve-as-tool).  
- Ingesta: descarga archivo Drive → embeddings → loader → insert en vector store.  
Referencias: nodos principales:contentReference[oaicite:11]{index=11}, conexiones del pipeline:contentReference[oaicite:12]{index=12}.

---

### 6) Contenido / Generador de videos cortos con IA (`contenido.json`)
**Qué hace:** Template “AI-Powered Short-Form Video Generator” usando OpenAI + servicios externos (ej. PiAPI, ElevenLabs, Creatomate), Google Sheets/Drive y notificación por webhook (ej. Discord).  
Referencias: requerimientos/stack en notas del workflow:contentReference[oaicite:13]{index=13} y pasos del render/upload/notificación:contentReference[oaicite:14]{index=14}.

---

## Requisitos (según workflow)

- n8n (self-hosted o cloud).
- Credenciales en n8n (OAuth/API Keys), según corresponda:
  - Google Sheets / Google Calendar / Google Drive
  - OpenAI
  - Asana
  - Microsoft Teams
  - SMTP (emails)
  - Postgres + Supabase (para RAG/Vector Store)
  - Proveedor/API para WhatsApp (en AgenteWS)

---

## Importar un workflow (.json) en n8n

1. n8n → **Workflows**
2. **Import from File**
3. Seleccioná el archivo `.json`
4. Abrí el workflow importado y configurá:
   - Credenciales (Credentials)
   - IDs (Sheet ID, Calendar ID, Team/Channel IDs, etc.)
   - Variables de entorno (si aplica)
5. Guardá y activá (si corresponde)

---

## Variables de entorno sugeridas

> Algunos workflows ya las usan como `$env.*` (ej. Asana/Teams/Timezone).

- `ASANA_TOKEN`
- `ASANA_PROJECT_ID`
- `TEAMS_TEAM_ID`
- `TEAMS_CHANNEL_ID`
- `TIMEZONE` (ej. `America/Argentina/Buenos_Aires`)

---

## Estructura recomendada del repo

