# ADR-003: Persistencia y Sincronización de Datos

## Estado
Aceptado

## Contexto
Cada microservicio maneja su propio estado persistido. Sin embargo, los datos pueden replicarse o sincronizarse entre servicios para optimizar consultas o resiliencia. Se deben decidir:
- Qué datos se replican
- Cómo se mantiene la coherencia
- Qué pasa en caso de inconsistencias

## Decisión
**Database per Service + Event-Driven Replication**

### Estrategia de Persistencia

1. **investigadores-service: PostgreSQL**
   - Almacén de verdad (source of truth)
   - Transacciones ACID
   - Modelos: Investigador, Institución
   - Acceso: REST API, gRPC

2. **publicaciones-service: MongoDB**
   - Modelos: Publicación, Autor
   - Flexible schema (permite cambios sin migrations)
   - Indexado para búsquedas complejas

3. **consultas-service: MongoDB (Read Model)**
   - No es fuente de verdad, es proyección
   - Denormalizado para optimizar lecturas
   - Se actualiza mediante eventos Kafka
   - Modelo: FichaInvestigador (nombre + todas sus publicaciones)

4. **polyglot-service: PostgreSQL + MongoDB + Redis**
   - Demostración de múltiples motores
   - Write-through: PostgreSQL es fuente de verdad
   - Propagación: MongoDB para análisis, Redis para caché
   - TTL en caché: 1 hora

### Flujo de Replicación

```
PublicacionesService registra publicación en MongoDB
    ↓
Publica evento: publicaciones.registradas en Kafka
    ↓
ConsultasService escucha evento
    ↓
ProyectorFichas actualiza FichaInvestigador en MongoDB
    ↓
GET /consultas/fichas/{investigador} obtiene dato actualizado
```

### Manejo de Inconsistencias

**Problema**: ¿Qué pasa si un evento se pierde o llega tarde?

**Soluciones**:

1. **Idempotencia**: Cada listener guarda el ID del evento procesado
2. **Reconciliación**: Endpoint `/sync` que reconstruye el modelo desde cero
3. **Dead Letter Topic (DLT)**: Eventos que fallan se envían a un tema especial
4. **Eventual Consistency**: Aceptar que hay un delay entre write y read

```java
// Manejo de idempotencia en ProyectorFichas
synchronized method registrarPublicacion(id, ...) {
  if (fichaPublicaciones.contains(id)) return; // Ya procesado
  fichaPublicaciones.add(id);
  save();
}
```

## Consecuencias

### Positivas
- ✅ Independencia: Cada servicio controla su BD
- ✅ Flexibilidad: Elegir la BD apropiada para cada caso (SQL, NoSQL)
- ✅ Escalabilidad: Pueden tenerse diferentes instancias/shards
- ✅ Resiliencia: El fallo de una BD no afecta los otros

### Negativas
- ❌ Consistencia eventual: Hay un delay entre writes y reads
- ❌ Complejidad en sincronización: Posibles inconsistencias
- ❌ Debugging: Verificar si datos están sincronizados es complejo
- ❌ Requerimientos de almacenamiento: Datos replicados en múltiples BD

## Alternativas Consideradas
1. **Base de datos compartida**: Más simple pero alto acoplamiento
2. **Shared Database + Event Log**: Comparte DB pero log de eventos desacoplado
3. **Eventual Consistency + Batch Reconciliation**: Reconciliación programada

## Implementaciones

### Taller 8: CQRS
- Write Model: investigadores-service + publicaciones-service (escrituras normales)
- Read Model: consultas-service (lecturas optimizadas)
- Sincronización: Eventos Kafka → MongoDB

### Taller 11: Polyglot Persistence
- PostgreSQL: Fuente de verdad
- MongoDB: Análisis y búsquedas
- Redis: Caché de alta velocidad
- Sincronización: SincronizadorPolyglot (write-through)

## Decisiones Relacionadas
- ADR-002: Comunicación entre servicios
- ADR-004: Transacciones distribuidas (Saga)
