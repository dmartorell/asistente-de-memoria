# Decisiones pendientes por orden de importancia

Puntos que bloquean o afectan el inicio del desarrollo.

---

## 1. Schema de entidades (CRÍTICO)

Afecta: modelo datos, API, app, IA. Bloquea inicio desarrollo.

### Alternativas

**A) Schema fijo predefinido** (más simple)
- Tipos hardcoded: `list`, `note`, `link`
- Cada tipo tiene campos fijos
- Fácil de implementar, UI predecible
- Limitado si usuario quiere tipos custom

```sql
-- Listas (compra, tareas, etc.)
CREATE TABLE lists (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  title TEXT NOT NULL,  -- "Compra Mercadona", "Compra semanal"
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE list_items (
  id UUID PRIMARY KEY,
  list_id UUID REFERENCES lists(id) ON DELETE CASCADE,
  text TEXT NOT NULL  -- "Leche", "Pan", "Huevos"
);

-- Links con categoría fija
CREATE TABLE links (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  category TEXT NOT NULL CHECK (category IN ('vehiculo', 'pelicula', 'serie', 'otro')),
  url TEXT NOT NULL,
  image_url TEXT,
  title TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**B) Schema semi-flexible** (equilibrado)
- Tipos base predefinidos + campos JSONB para metadata custom
- Categorías de links flexibles (no hardcoded)
- Balance entre estructura y flexibilidad

```sql
-- Listas con items
CREATE TABLE lists (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  title TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE list_items (
  id UUID PRIMARY KEY,
  list_id UUID REFERENCES lists(id) ON DELETE CASCADE,
  text TEXT NOT NULL
);

-- Links con categoría flexible + metadata
CREATE TABLE links (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  category TEXT NOT NULL,  -- cualquier valor: "vehiculo", "peli", "receta", etc.
  url TEXT NOT NULL,
  image_url TEXT,
  title TEXT,
  metadata JSONB DEFAULT '{}',  -- {"precio": 15000, "km": 80000} para vehículos
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Ejemplo: usuario puede crear categorías nuevas sin cambiar schema
```

**C) Schema totalmente dinámico** (más complejo)
- Usuario define sus propios tipos de colección con campos custom
- Máxima flexibilidad, más complejidad en UI y validación
- Riesgo: over-engineering para MVP

```sql
-- Tipos de colección definidos por usuario
CREATE TABLE collection_types (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  name TEXT NOT NULL,           -- "vehículos", "películas", "lista compra"
  item_schema JSONB NOT NULL    -- define campos de cada item
);

-- Ejemplo item_schema para "vehículos": {
--   "fields": [
--     {"name": "precio", "type": "number"},
--     {"name": "km", "type": "number"},
--     {"name": "marca", "type": "text"}
--   ]
-- }

-- Colecciones
CREATE TABLE collections (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  type_id UUID REFERENCES collection_types(id),
  title TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Items genéricos (listas compra, links, todo)
CREATE TABLE collection_items (
  id UUID PRIMARY KEY,
  collection_id UUID REFERENCES collections(id) ON DELETE CASCADE,
  data JSONB NOT NULL  -- campos según schema del tipo
);
```

---

## 2. Modelo de embeddings (ALTO)

Afecta: búsqueda semántica, costes operativos, latencia.

### Alternativas

**A) OpenAI text-embedding-3-small** (más simple)
- API lista, sin infraestructura
- ~$0.02/1M tokens
- Dependencia externa
- Fácil de implementar

**B) Cohere embed-v3** (equilibrado)
- Mejor para multilingüe (español)
- API similar a OpenAI
- Tier gratuito generoso para MVP

**C) Self-hosted (sentence-transformers)** (más complejo)
- Sin costes por uso
- Requiere GPU o CPU potente
- Mayor control, más infraestructura
- Modelos: `paraphrase-multilingual-MiniLM-L12-v2`

---

## 3. API: REST vs GraphQL (MEDIO-ALTO)

Diagrama menciona ambos. Hay que decidir.

### Alternativas

**A) REST puro** (más simple)
- Endpoints claros: `/lists`, `/notes`, `/links`
- Tooling maduro, fácil debug
- Puede requerir múltiples requests

**B) GraphQL** (equilibrado)
- Un endpoint, queries flexibles
- Ideal si app necesita datos anidados
- Más setup inicial, curva aprendizaje

**C) REST + GraphQL parcial** (más complejo)
- REST para CRUD simple
- GraphQL para queries complejas/reportes
- Dos sistemas a mantener

---

## 4. Roadmap / fases MVP (MEDIO)

Sin fases claras, riesgo de scope creep.

### Alternativas

**A) MVP mínimo vertical** (más simple)
- Fase 1: Solo listas via Telegram + app read-only
- Fase 2: Añadir notas
- Fase 3: Añadir enlaces + búsqueda
- Validar cada fase con usuarios reales

**B) MVP horizontal básico** (equilibrado)
- Fase 1: Listas + notas + enlaces via Telegram (sin búsqueda semántica)
- Fase 2: App CRUD completa
- Fase 3: Búsqueda semántica + reglas usuario

**C) MVP completo** (más complejo)
- Todo el alcance definido de golpe
- Mayor riesgo, más tiempo hasta feedback
- Solo si hay certeza del producto

---

## 5. Proveedor LLM para clasificación (MEDIO)

Classifier, Structurer, Clarifier necesitan LLM.

### Alternativas

**A) OpenAI GPT-4o-mini** (más simple)
- Barato (~$0.15/1M input tokens)
- Muy capaz para clasificación
- API estable

**B) Anthropic Claude Haiku** (equilibrado)
- Similar coste
- Mejor en español según algunos benchmarks
- Buena opción si ya usas Claude

**C) Self-hosted (Llama 3, Mistral)** (más complejo)
- Sin costes por uso
- Requiere infraestructura GPU
- Más control, más mantenimiento

---

## 6. Vector DB para embeddings (MEDIO)

Si se usan embeddings, dónde almacenarlos.

### Alternativas

**A) pgvector en PostgreSQL** (más simple)
- Mismo DB, sin infra adicional
- Suficiente para <100k vectores
- Queries SQL normales

**B) Supabase (pgvector managed)** (equilibrado)
- PostgreSQL + pgvector hosted
- Auth incluido
- Tier gratuito generoso

**C) Pinecone / Qdrant dedicado** (más complejo)
- Optimizado para vectores
- Mejor performance a escala
- Otro servicio a gestionar

---

## Resumen recomendación rápida

| Decisión | Recomendación MVP |
|----------|-------------------|
| Schema entidades | B) Semi-flexible |
| Embeddings | A) OpenAI o B) Cohere |
| API | A) REST puro |
| Roadmap | A) MVP mínimo vertical |
| LLM | A) GPT-4o-mini |
| Vector DB | A) pgvector |

---

## Preguntas para decidir

1. ¿Cuánta flexibilidad necesita el usuario en tipos de entidad?
2. ¿Presupuesto mensual máximo para APIs externas en MVP?
3. ¿Preferencia por vendor (OpenAI vs Anthropic vs open source)?
4. ¿Primera funcionalidad a validar con usuarios: listas, notas o enlaces?
