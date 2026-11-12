# ADR-005: Event Sourcing - PROPUESTA

## Estado
**Propuesta** (No Implementado)

## Contexto
Actualmente, los eventos se usan para sincronización (Taller 8), pero no se guardan todos los eventos del sistema. Esto significa que no hay un historial completo de cambios.

¿Qué pasó con la ficha del investigador hace 3 meses? ¿Cuándo se le agregó la 10ª publicación?

## Solución Propuesta
**Event Sourcing**: Guardar TODOS los eventos como fuente de verdad

### Idea Central

En lugar de guardar solo el estado actual:
```
Investigador { id: 1, nombre: "Juan", publicaciones: 5 }
```

Guardar la secuencia de eventos que llevó a ese estado:
```
1. InvestigadorCreadoEvent { id: 1, nombre: "Juan" }
2. PublicacionRegistradaEvent { id: 1, pubId: 10 }
3. PublicacionRegistradaEvent { id: 1, pubId: 11 }
4. PublicacionRegistradaEvent { id: 1, pubId: 12 }
5. PublicacionRegistradaEvent { id: 1, pubId: 13 }
6. PublicacionRegistradaEvent { id: 1, pubId: 14 }
```

### Beneficios

- ✅ **Auditoría completa**: Saber exactamente qué cambió y cuándo
- ✅ **Time travel**: Reconstruir el estado en cualquier punto del tiempo
- ✅ **Replay**: Reproducir eventos para recalcular proyecciones (CQRS)
- ✅ **Debugging**: Entender la secuencia exacta de eventos

### Desafíos

- ❓ Almacenamiento: Guardar todos los eventos consume mucho espacio
- ❓ Consultas: Reconstruir estado requiere procesar todos los eventos
- ❓ Eliminación: ¿Cómo eliminar datos por GDPR si los eventos son inmutables?
- ❓ Versionamiento: ¿Qué pasa si la estructura del evento cambia?

## Implementación Posible

```java
@Document(collection = "event_store")
public class DomainEvent {
  String id;
  String aggregateId;
  String eventType;
  Object payload;
  LocalDateTime timestamp;
  int version;
}

// Al crear investigador
DomainEvent evento = new DomainEvent(
  aggregateId: "inv-1",
  eventType: "InvestigadorCreadoEvent",
  payload: { nombre: "Juan", ... },
  timestamp: now(),
  version: 1
);
eventStore.append(evento);
```

## Alternativas

1. **Solo proyecciones** (actual): Guardar solo estado actual
2. **Event Sourcing completo**: Guardar todos los eventos
3. **Hybrid**: Guardar eventos + snapshots periódicos

## Decisiones Relacionadas
- ADR-003: Persistencia y sincronización (CQRS)
- Taller 8: CQRS con proyecciones (es un paso hacia Event Sourcing)

## Referencias
- Greg Young, "Event Sourcing Pattern"
- Microsoft Docs, "Event Sourcing pattern"
