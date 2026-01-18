# Plan completo – Asistente de memoria personal estructurada

## 1. Visión y problema

Las personas usan chats consigo mismas (especialmente en WhatsApp) como un repositorio informal: listas, enlaces, imágenes, documentos, notas, recordatorios. Este hábito está completamente validado, pero la información queda:
- No estructurada
- Difícil de recuperar
- No consultable de forma inteligente

El producto convierte ese flujo caótico en una **memoria personal estructurada**, sin cambiar el hábito del usuario.

## 2. Propuesta de valor

- El usuario escribe mensajes en lenguaje natural
- El sistema interpreta, estructura y almacena
- La información se puede consultar aunque el usuario no recuerde el origen
- La app propia permite gestión total (CRUD)

**Mensajería = entrada y consulta**
**App = sistema de verdad**

## 3. Alcance funcional

### 3.1 Entrada vía mensajería

- Crear y ampliar listas
- Guardar enlaces (YouTube, Instagram, TikTok, web)
- Guardar notas
- Guardar documentos e imágenes con metadatos
- Consultas en lenguaje natural

### 3.2 App (Expo)

- Visualización estructurada
- CRUD completo
- Edición y borrado
- Gestión de reglas aprendidas
- No usa IA para escribir datos

## 4. Restricciones y cumplimiento legal

- Prohibido: contenido sexual, ilegal, drogas, pornografía infantil
- Avisos claros sobre datos sensibles (DNI, menores)
- Cumplimiento GDPR desde el diseño
- No venta de datos a terceros

## 5. Arquitectura técnica

### 5.1 Principio rector

El core del sistema **no depende de ningún proveedor externo de mensajería**.

Arquitectura hexagonal (Ports & Adapters) con inversión de dependencias.

### 5.2 Diagrama lógico

```
App Expo
   │
REST / GraphQL
   │
Application Core (Use Cases)
   │
Domain (Entidades + Reglas)
   │
Ports (IA / Repos / Canales)
   │
Adapters:
- WhatsApp
- Telegram
- Web Chat
- PostgreSQL
- IA Provider
```

## 6. Normalización de mensajes

Todos los canales generan el mismo objeto:

```json
{
  "userId": "uuid",
  "content": "texto libre",
  "attachments": [],
  "language": "es",
  "timestamp": "..."
}
```

## 7. Diseño del cerebro IA

### 7.1 Principios

- La IA no decide, propone
- El sistema aprende reglas explícitas
- La app es la fuente de verdad

### 7.2 Agentes lógicos

- **Classifier**: detecta intención
- **Structurer**: genera datos estructurados
- **Query Builder**: traduce preguntas a consultas
- **Clarifier**: pregunta cuando hay ambigüedad
- **Embeddings**: búsqueda semántica

Embeddings encapsulados, sin necesidad de conocimientos de ML.

### 7.3 Flujo de ingesta

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

### 7.4 Aprendizaje por usuario

Reglas persistidas y versionadas.

**Tabla user_rules**: user_id, trigger, entity_type, prioridad

Editable desde la app.

## 8. Modelo de datos

- PostgreSQL
- Relacional como base
- JSONB para flexibilidad
- Embeddings opcionales por item

La app gestiona el CRUD sin pasar por IA.

## 9. Costes

### 9.1 Desarrollo inicial

| Concepto | Coste |
|----------|-------|
| Backend + IA (6 meses) | 24.000 € |
| Frontend Expo (6 meses) | 24.000 € |
| UX/UI básico | 3.000 € |
| **Total** | **~51.000 €** |

### 9.2 ALPHA – 1er año (600 usuarios)

| Categoría | Coste anual |
|-----------|-------------|
| Infraestructura | 960 € |
| WhatsApp API | 1.800 € |
| IA | 2.400 € |
| Otros | 180 € |
| **Total** | **~5.340 €** |

### 9.3 Salida a mercado

| Categoría | Coste anual |
|-----------|-------------|
| Infraestructura | 3.000 € |
| WhatsApp API | 6.000–8.000 € |
| IA | 8.000–12.000 € |
| Soporte | 2.000 € |

## 10. Legal y privacidad

### Europa (GDPR)

- Consultoría: 2.000–4.000 €
- TOS + Privacy: 1.000–1.500 €
- Auditoría básica: 1.500–3.000 €

### USA / LATAM

- 2.500–4.500 €

## 11. Estado del proyecto

- Definición cerrada
- Arquitectura clara
- Riesgos identificados
- Listo para modelo de datos detallado, UI o roadmap

## 12. Prompt de recuperación

> Estoy desarrollando un producto de memoria personal estructurada basado en mensajería y una app propia.
>
> - Backend Node.js
> - App Expo
> - PostgreSQL (relacional + JSONB)
> - Arquitectura hexagonal (ports & adapters)
> - Canales desacoplados (WhatsApp inicial)
> - IA como intérprete, no decisora
> - Aprendizaje por usuario mediante reglas persistidas
> - Embeddings encapsulados
> - App como sistema de verdad (CRUD sin IA)
>
> Ya existe un plan completo de producto, arquitectura técnica, diseño del cerebro IA y costes.
>
> Quiero continuar exactamente desde este punto, sin redefinir la idea, y avanzar a la siguiente fase.
