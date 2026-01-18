# Plan completo – Asistente de memoria personal estructurada (v2)

## 1. Visión y problema

Usuarios usan chats consigo mismos como repositorio informal: listas, enlaces, notas. Info queda no estructurada, difícil de recuperar, no consultable inteligentemente.

Producto convierte flujo caótico en **memoria personal estructurada**, sin cambiar hábito.

## 2. Propuesta de valor

- Usuario escribe en lenguaje natural
- Sistema interpreta, estructura, almacena
- Info consultable aunque no recuerde origen
- App permite gestión total (CRUD)

**Mensajería = entrada y consulta**
**App = sistema de verdad**

## 3. Alcance funcional

### 3.1 MVP - Entrada vía mensajería

- Crear/ampliar listas
- Guardar enlaces (YouTube, Instagram, TikTok, web)
- Guardar notas
- Consultas en lenguaje natural

**Excluido MVP**: imágenes, documentos

### 3.2 App (Expo - iOS/Android)

- Visualización estructurada
- CRUD completo
- Edición y borrado
- Gestión de reglas aprendidas
- No usa IA para escribir datos

## 4. Autenticación

- Login en app: teléfono (campo único) + username + password
- Usuario = 1 número teléfono
- Vinculación con canal mensajería via mismo teléfono

## 5. Restricciones y cumplimiento legal

- Prohibido: contenido sexual, ilegal, drogas, CSAM
- Avisos claros datos sensibles (DNI, menores)
- GDPR desde diseño
- No venta datos terceros

## 6. Arquitectura técnica

### 6.1 Principio rector

Core **no depende de proveedor mensajería**.

Arquitectura hexagonal (Ports & Adapters).

### 6.2 Canal inicial

**Telegram** (más fácil, barato, sin approval process)

Diseño permite switch fácil a WhatsApp Business API futuro.

### 6.3 Diagrama lógico

```
                    ┌─────────────────┐
                    │   App Expo      │
                    │  (iOS/Android)  │
                    └────────┬────────┘
                             │ REST/GraphQL
                             ▼
┌──────────────┐    ┌─────────────────────────────┐    ┌──────────────┐
│   Telegram   │───▶│                             │◀───│  PostgreSQL  │
│    Bot       │    │     APPLICATION CORE        │    │              │
└──────────────┘    │                             │    └──────────────┘
                    │  ┌───────────────────────┐  │
┌──────────────┐    │  │       DOMAIN          │  │    ┌──────────────┐
│  WhatsApp    │───▶│  │  Entidades + Reglas   │  │◀───│  Vector DB   │
│  (futuro)    │    │  └───────────────────────┘  │    │ (embeddings) │
└──────────────┘    │                             │    └──────────────┘
                    │         Use Cases           │
                    │   Classifier, Structurer,    │   ┌──────────────┐
                    │   QueryBuilder, Clarifier    │◀──│  IA Provider │
                    └─────────────────────────────┘   │ (LLM API)    │
                                                      └──────────────┘
```

## 7. Normalización de mensajes

Todos canales generan mismo objeto:

```json
{
  "userId": "uuid",
  "content": "texto libre",
  "attachments": [],
  "language": "es",
  "timestamp": "..."
}
```

## 8. Diseño del cerebro IA

### 8.1 Principios

- IA no decide, propone
- Sistema aprende reglas explícitas
- App es fuente de verdad

### 8.2 Agentes lógicos

- **Classifier**: detecta intención
- **Structurer**: genera datos estructurados
- **Query Builder**: traduce preguntas a consultas
- **Clarifier**: pregunta cuando ambigüedad
- **Embeddings**: búsqueda semántica

### 8.3 Flujo ingesta

```
Mensaje
   ↓
Classifier
   ↓
¿Existe regla?
 ├─ Sí → Structurer
 └─ No → Clarifier → Guardar regla
   ↓
Persistir
   ↓
Generar embedding
```

### 8.4 Aprendizaje por usuario

Reglas persistidas y versionadas.

**Tabla user_rules**: user_id, trigger, entity_type, prioridad

Editable desde app.

## 9. Modelo de datos

- PostgreSQL
- Relacional como base
- JSONB para flexibilidad
- Embeddings opcionales por item

App gestiona CRUD sin pasar por IA.

## 10. Costes

### 10.1 Desarrollo inicial

| Concepto      | Coste         |
| ------------- | ------------- |
| Backend + IA  | 24.000 €      |
| Frontend Expo | 24.000 €      |
| UX/UI básico  | 3.000 €       |
| **Total**     | **~51.000 €** |

### 10.2 ALPHA – 1er año (600 usuarios)

| Categoría       | Coste anual  |
| --------------- | ------------ |
| Infraestructura | 960 €        |
| Telegram Bot    | ~0 €         |
| IA              | 2.400 €      |
| Otros           | 180 €        |
| **Total**       | **~3.540 €** |

### 10.3 Salida a mercado (WhatsApp)

| Categoría       | Coste anual    |
| --------------- | -------------- |
| Infraestructura | 3.000 €        |
| WhatsApp API    | 6.000–8.000 €  |
| IA              | 8.000–12.000 € |
| Soporte         | 2.000 €        |

## 11. Legal y privacidad

### Europa (GDPR)

- Consultoría: 2.000–4.000 €
- TOS + Privacy: 1.000–1.500 €
- Auditoría básica: 1.500–3.000 €

### USA / LATAM

- 2.500–4.500 €

## 12. Equipo

- **Enric**: creador idea, producto
- **Dani**: desarrollo

## 13. Estado del proyecto

- Definición cerrada
- Arquitectura clara
- Canal inicial: Telegram
- MVP: listas, notas, enlaces (sin imágenes/docs)
- UI/UX: solo concepto

## 14. Pendiente definir

- Modelo de negocio / pricing
- Schema entidades (fijo vs extensible)
- Roadmap / fases MVP
- Análisis competencia
- Modelo embeddings (OpenAI, Cohere, local)
- Funcionalidades futuras (compartir, offline, export)

