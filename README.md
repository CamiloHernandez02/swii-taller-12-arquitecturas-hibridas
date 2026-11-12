# Taller 12: Arquitecturas Híbridas - Documentación

## Descripción
Este taller documenta las decisiones arquitectónicas del proyecto RICA (Plataforma de Gestión de Investigadores y Publicaciones) mediante Architecture Decision Records (ADRs).

Los ADRs registran decisiones importantes del sistema de forma estructurada, incluyendo:
- El contexto y problema a resolver
- La decisión tomada y por qué
- Las consecuencias (positivas y negativas)
- Las alternativas consideradas

## ADRs Implementados

### [ADR-001: Descomposición en Microservicios](docs/ADR-001-descomposicion-microservicios.md)
**Estado**: ✅ Aceptado e Implementado

Descompone el sistema en 5 microservicios independientes:
- investigadores-service (PostgreSQL)
- publicaciones-service (MongoDB)
- consultas-service (MongoDB + CQRS)
- observabilidad-service (Micrometer, Prometheus)
- polyglot-service (PostgreSQL + MongoDB + Redis)

### [ADR-002: Comunicación entre Servicios](docs/ADR-002-comunicacion-servicios.md)
**Estado**: ✅ Aceptado e Implementado (Parcialmente)

Estrategia híbrida:
- **Síncrono**: REST con circuit breaker (Resilience4j), gRPC
- **Asíncrono**: Kafka con Dead Letter Topic (DLT)
- **Discovery**: Eureka para service discovery dinámico

Implementado en Taller 8 (CQRS) y Taller 9 (EDA).

### [ADR-003: Persistencia y Sincronización de Datos](docs/ADR-003-persistencia-sincronizacion.md)
**Estado**: ✅ Aceptado e Implementado

Database per Service con replicación basada en eventos:
- **Write Model** (fuente de verdad): investigadores-service + publicaciones-service
- **Read Model** (CQRS): consultas-service
- **Polyglot**: PostgreSQL → MongoDB → Redis
- **Sincronización**: Eventos Kafka + SincronizadorPolyglot

Implementado en Taller 8 (CQRS) y Taller 11 (Polyglot).

## ADRs Propuestos (No Implementados)

### [ADR-004: Transacciones Distribuidas (Saga Pattern)](docs/ADR-004-saga-pattern-propuesta.md)
**Estado**: 📋 Propuesta

Propuesta de implementar el Saga Pattern para coordinar transacciones distribuidas con compensaciones. Prototipo en Taller 10 (SagaOrquestador).

**Desafíos pendientes**:
- Idempotencia de pasos
- Manejo de fallos en compensaciones
- Timeouts y retries
- Persistencia de estado de saga

### [ADR-005: Event Sourcing](docs/ADR-005-event-sourcing-propuesta.md)
**Estado**: 📋 Propuesta

Guardar todos los eventos del sistema como fuente de verdad, permitiendo:
- Auditoría completa
- Time travel (reconstruir estado en cualquier momento)
- Replay de eventos

Requeriría cambios significativos en la arquitectura actual de CQRS.

## Matriz de Decisiones

| ADR | Tema | Estado | Tallers | Tecnologías |
|-----|------|--------|--------|-------------|
| 001 | Descomposición | ✅ Implementado | T5-T11 | Microservicios, Spring Boot |
| 002 | Comunicación | ✅ Implementado | T8, T9 | Kafka, REST, gRPC, Eureka |
| 003 | Persistencia | ✅ Implementado | T8, T11 | PostgreSQL, MongoDB, Redis |
| 004 | Saga Pattern | 📋 Propuesta | T10 | Orquestación distribuida |
| 005 | Event Sourcing | 📋 Propuesta | - | EventStore, Domain Events |

## Consecuencias Arquitectónicas

### Ventajas del Sistema Actual
- ✅ **Escalabilidad**: Cada servicio escala independientemente
- ✅ **Resiliencia**: Fallos aislados no derriban todo
- ✅ **Flexibilidad tecnológica**: Elegir BD apropiada para cada caso
- ✅ **Despliegue continuo**: Equipos independientes
- ✅ **Observabilidad**: Distributed tracing, métricas, correlation IDs

### Desafíos Actuales
- ❌ **Complejidad operacional**: Necesita orquestación (Docker, K8s)
- ❌ **Consistencia eventual**: Datos desincronizados temporalmente
- ❌ **Debugging distribuido**: Más difícil seguir una transacción
- ❌ **Latencia de red**: Mayor overhead vs. monolítico
- ❌ **Transacciones distribuidas**: Sin ACID completo

## Cómo Usar Este Repositorio

1. **Leer ADRs** en orden (001-005) para entender decisiones
2. **Implementar nuevas features**: Seguir los patrones en los ADRs
3. **Cambiar decisiones**: Crear nuevo ADR con estado "Superceded" y referencia
4. **Proponer cambios**: Crear ADR con estado "Propuesta" para discusión

## Próximos Pasos

1. **Implementar ADR-004**: Event-Driven Saga Orchestration
2. **Explorar ADR-005**: Event Sourcing con CQRS
3. **Agregar ADR-006**: API Gateway y enrutamiento
4. **Agregar ADR-007**: Seguridad (OAuth2, JWT, gRPC TLS)
5. **Agregar ADR-008**: CI/CD y deployment en Kubernetes

## Referencias

- Microservices.io: https://microservices.io/patterns/index.html
- AWS Well-Architected Framework
- Martin Fowler, "Microservices"
- Chris Richardson, "Building Microservices with Event Sourcing"

---
**Taller 12**: Documentación de Arquitectura  
**Fecha**: 2026-11-11  
**Autor**: Equipo RICA
