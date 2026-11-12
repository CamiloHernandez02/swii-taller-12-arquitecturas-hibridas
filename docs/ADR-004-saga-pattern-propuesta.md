# ADR-004: Transacciones Distribuidas (Saga Pattern) - PROPUESTA

## Estado
**Propuesta** (No Implementado)

## Contexto
En un sistema con múltiples microservicios, una transacción puede requerir cambios en varios servicios. Ejemplo:
- Registrar una publicación + Actualizar la ficha del investigador + Notificar

¿Cómo garantizar consistencia si uno de estos pasos falla?

## Solución Propuesta
**Saga Pattern**: Transacción distribuida coordinada con compensaciones

### Tipos de Saga

1. **Orquestrada** (Centralized)
   - Un coordinador dirige todos los pasos
   - Cada paso es una llamada a otro servicio
   - Si falla un paso, el coordinador ejecuta compensaciones
   - Ejemplo: ObservabilidadController con SagaOrquestador

```
ObservabilidadService (Coordinador)
  ├─ Paso 1: Crear investigador
  ├─ Paso 2: Registrar publicación
  └─ Paso 3: Actualizar ficha
    └─ Si falla paso 2:
       ├─ Compensar paso 1 (eliminar investigador)
       └─ Compensar paso 2 (etc)
```

2. **Coreografía** (Decentralized)
   - Los servicios se comunican por eventos
   - Cada servicio sabe qué hacer al recibir un evento
   - No hay coordinador central
   - Más resiliente pero harder to understand

## Implementación Propuesta

```java
SagaOrquestador saga = new SagaOrquestador();

saga.agregarPaso("Crear investigador", 
  () -> investigadorService.crear(investigador),
  () -> investigadorService.eliminar(investigador.id)
);

saga.agregarPaso("Registrar publicación",
  () -> publicacionService.crear(publicacion),
  () -> publicacionService.eliminar(publicacion.id)
);

saga.agregarPaso("Actualizar ficha",
  () -> consultasService.actualizar(ficha),
  () -> consultasService.revert(ficha)
);

try {
  saga.ejecutarSaga();
} catch (SagaFallidaException e) {
  // Las compensaciones se ejecutaron automáticamente
}
```

## Desafíos No Resueltos

- ❓ **Idempotencia**: ¿Cómo garantizar que un paso es idempotente?
- ❓ **Orden de compensaciones**: Ejecutar en orden inverso (LIFO)
- ❓ **Fallos en compensaciones**: ¿Qué pasa si una compensación falla?
- ❓ **Timeouts**: ¿Cuánto esperar por un paso que demora?
- ❓ **Persistencia de Sagas**: ¿Guardar el estado de la saga para recovery?

## Alternativas No Exploradas

1. **Event Sourcing**: Guardar todos los eventos para reproducir estado
2. **Compensating Transactions**: Transacciones explícitas que revierten cambios
3. **Outbox Pattern**: Garantizar que eventos se publican junto con cambios de BD

## Decisiones Relacionadas
- ADR-002: Comunicación entre servicios
- ADR-003: Persistencia y sincronización

## Referencias
- Chris Richardson, "Pattern: Saga", https://microservices.io/patterns/data/saga.html
- Taller 10: SagaOrquestador implementado con éxito
