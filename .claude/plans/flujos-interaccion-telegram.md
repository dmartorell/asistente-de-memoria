# Flujos de interacción Telegram

Ejemplos concretos de interacción usuario-bot para desarrollo MVP.

## Decisiones de diseño

- Comandos opcionales, texto libre preferido
- Bot confirma solo si hay ambigüedad
- Solo español MVP

---

## 1. Casos de uso listas

### 1.1 Crear lista nueva

**Usuario:** `Lista compra Mercadona`

**Bot:** `✓ Lista "Compra Mercadona" creada`

**BD:**

```sql
INSERT INTO lists (id, user_id, title)
VALUES (uuid, user_uuid, 'Compra Mercadona');
```

---

### 1.2 Añadir item a lista existente

**Usuario:** `Añade leche a compra Mercadona`

**Bot:** `✓ Añadido "leche" a Compra Mercadona`

**BD:**

```sql
INSERT INTO list_items (id, list_id, text)
VALUES (uuid, list_uuid, 'leche');
```

---

### 1.3 Añadir item - lista implícita (1 lista)

Usuario tiene 1 sola lista.

**Usuario:** `Leche`

**Bot:** `✓ Añadido "leche" a Compra Mercadona`

**BD:** igual que 1.2

---

### 1.4 Añadir item - ambigüedad (varias listas)

Usuario tiene: "Compra Mercadona", "Compra Lidl", "Tareas casa"

**Usuario:** `Leche`

**Bot:**

```
¿A qué lista?
1. Compra Mercadona
2. Compra Lidl
```

**Usuario:** `1`

**Bot:** `✓ Añadido "leche" a Compra Mercadona`

---

### 1.5 Añadir múltiples items

**Usuario:** `Añade a compra: leche, pan, huevos, aceite`

**Bot:**

```
✓ Añadidos 4 items a Compra Mercadona:
- leche
- pan
- huevos
- aceite
```

**BD:**

```sql
INSERT INTO list_items (id, list_id, text) VALUES
(uuid1, list_uuid, 'leche'),
(uuid2, list_uuid, 'pan'),
(uuid3, list_uuid, 'huevos'),
(uuid4, list_uuid, 'aceite');
```

---

### 1.6 Ver items de lista

**Usuario:** `¿Qué tengo en la lista de compra?`

**Bot:**

```
📋 Compra Mercadona (4 items):
- leche
- pan
- huevos
- aceite
```

**BD:** `SELECT text FROM list_items WHERE list_id = ?`

---

### 1.7 Eliminar item

**Usuario:** `Quita leche de compra`

**Bot:** `✓ Eliminado "leche" de Compra Mercadona`

**BD:**

```sql
DELETE FROM list_items
WHERE list_id = ? AND text ILIKE 'leche';
```

---

### 1.8 Eliminar lista completa

**Usuario:** `Borra lista compra Mercadona`

**Bot:** `¿Seguro? Lista tiene 3 items. Responde "sí" para confirmar.`

**Usuario:** `sí`

**Bot:** `✓ Lista "Compra Mercadona" eliminada`

**BD:**

```sql
DELETE FROM lists WHERE id = ?;
-- list_items se borran por CASCADE
```

---

## 2. Casos de uso links

### 2.1 Guardar link con categoría explícita

**Usuario:** `Guarda este coche: https://wallapop.com/item/bmw-320`

**Bot:** `✓ Link guardado en "vehículo"`

**BD:**

```sql
INSERT INTO links (id, user_id, category, url, title)
VALUES (uuid, user_uuid, 'vehiculo', 'https://wallapop.com/item/bmw-320', 'BMW 320');
```

---

### 2.2 Guardar link - bot infiere categoría

**Usuario:** `https://youtube.com/watch?v=abc123`

**Bot:** `✓ Link guardado en "video"`

Classifier detecta dominio YouTube → categoría "video"

---

### 2.3 Guardar link - categoría ambigua

**Usuario:** `https://example.com/articulo`

**Bot:** `¿Cómo categorizo este link?`

**Usuario:** `recetas`

**Bot:** `✓ Link guardado en "recetas"`

Sistema puede aprender: links de example.com → recetas (si frecuente)

---

### 2.4 Ver links por categoría

**Usuario:** `¿Qué coches tengo guardados?`

**Bot:**

```
🚗 Vehículos (3 links):
1. BMW 320 - wallapop.com
2. Seat León - milanuncios.com
3. VW Golf - coches.net
```

**BD:**

```sql
SELECT title, url FROM links
WHERE user_id = ? AND category = 'vehiculo';
```

---

### 2.5 Ver todos los links

**Usuario:** `¿Qué links tengo guardados?`

**Bot:**

```
📎 Tus links:

🚗 Vehículos (3)
🎬 Videos (5)
🍳 Recetas (2)

Escribe categoría para ver detalle.
```

---

## 3. Manejo de ambigüedad (Clarifier)

### 3.1 Lista con nombre similar

Usuario tiene: "Compra semanal", "Compra mensual"

**Usuario:** `Añade arroz a compra`

**Bot:**

```
¿Cuál lista?
1. Compra semanal
2. Compra mensual
```

---

### 3.2 Item podría ir a varias listas

Usuario tiene: "Regalos Ana", "Regalos Pedro"

**Usuario:** `Libro de cocina`

**Bot:**

```
¿A qué lista añado "libro de cocina"?
1. Regalos Ana
2. Regalos Pedro
3. Nueva lista
```

---

### 3.3 Intención ambigua

**Usuario:** `Pan`

Bot no sabe si es:

- Añadir a lista de compra
- Crear nueva lista "Pan"
- Otra cosa

**Bot:** `¿Qué quieres hacer con "pan"?`

Usuario tiene contexto previo (última acción fue añadir a compra):

**Bot:** `✓ Añadido "pan" a Compra Mercadona`

---

### 3.4 Crear regla implícita

Patrón repetido: usuario siempre añade a "Compra semanal" cuando escribe alimentos sueltos.

Sistema aprende regla:

```sql
INSERT INTO user_rules (user_id, trigger, entity_type, target_id)
VALUES (user_uuid, 'alimento_suelto', 'list_item', list_compra_uuid);
```

Próxima vez:
**Usuario:** `Tomates`
**Bot:** `✓ Añadido "tomates" a Compra semanal`

Sin preguntar.

---

## 4. Comandos opcionales

Para usuarios que prefieren comandos explícitos:

| Comando             | Equivalente natural      |
| ------------------- | ------------------------ |
| `/lista Compra`     | "Crea lista compra"      |
| `/add leche`        | "Añade leche"            |
| `/ver compra`       | "¿Qué tengo en compra?"  |
| `/link https://...` | Enviar link directamente |
| `/links`            | "¿Qué links tengo?"      |

Bot acepta ambos, no hay diferencia funcional.

---

## 5. Flujo completo ejemplo

```
Usuario: Lista para el viaje a Roma

Bot: ✓ Lista "Viaje a Roma" creada

Usuario: Pasaporte, cargador, adaptador enchufe

Bot: ✓ Añadidos 3 items a Viaje a Roma:
- pasaporte
- cargador
- adaptador enchufe

Usuario: https://booking.com/hotel-roma-123

Bot: ¿Cómo categorizo este link?

Usuario: viaje roma

Bot: ✓ Link guardado en "viaje roma"

Usuario: ¿Qué me falta para Roma?

Bot: 📋 Viaje a Roma (3 items):
- pasaporte
- cargador
- adaptador enchufe

📎 Links "viaje roma" (1):
- Hotel Roma Centro - booking.com
```

---

## Preguntas sin resolver

- ¿Máximo items por mensaje? (rendimiento)
- ¿Persistir contexto conversación cuánto tiempo?
- ¿Notificaciones proactivas? (ej: "¿Fuiste a comprar?")
