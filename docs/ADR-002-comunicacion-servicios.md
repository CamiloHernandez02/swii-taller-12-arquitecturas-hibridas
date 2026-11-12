# ADR-002: Comunicación entre Servicios

## Estado
Aceptado

## Contexto
Los microservicios necesitan comunicarse para ciertas operaciones (e.g., cuando se crea una publicación, notificar al servicio de consultas). Se deben considerar dos patrones principales:
- **Síncrono (Request/Response)**: REST, gRPC
- **Asíncrono (Publish/Subscribe)**: Message Broker (Kafka, RabbitMQ)

## Decisión
**Estrategia Híbrida**:

### 1. Comunicación Síncrona (gRPC para servicio a servicio)
- **Use cases**: Consultas que requieren respuesta inmediata
- **Implementación**: OpenFeign con circuit breaker (Resilience4j)
- **Ejemplo**: consultas-service pregunta a investigadores-service por detalles del investigador
- **Ventaja**: Latencia baja, modelo request/response claro
- **Desventaja**: Acoplamiento temporal, el servicio debe estar disponible

### 2. Comunicación Asíncrona (Kafka para eventos)
- **Use cases**: Notificaciones de cambios, replicación de datos
- **Patrones**:
  - **Outbox Pattern**: Garantiza que el evento se publica si la BD se actualiza
  - **Dead Letter Topic (DLT)**: Manejo de mensajes que fallan
- **Topics**:
  - `publicaciones.registradas`: Cuando se crea una publicación
  - `investigadores.eliminados`: Cuando se elimina un investigador
  - `eventos.investigador.creado`: Nuevo investigador
  - `eventos.investigador.alta`: Alta en el sistema
- **Ejemplo**: publicaciones-service publica evento, consultas-service se suscribe
- **Ventaja**: Desacoplamiento temporal, resiliencia a fallos
- **Desventaja**: Consistencia eventual, complejidad de debugging

### 3. Service Discovery (Eureka)
- **Problema**: ¿Cómo sabe un servicio la dirección IP/puerto de otro?
- **Solución**: Eureka registry
- Cada servicio se registra al iniciar
- Los clientes consultan Eureka para descubrir servicios dinámicamente

## Configuración

```yaml
# investigadores-service publica evento
kafka:
  topic: publicaciones.registradas
  bootstrap-servers: localhost:9092

# consultas-service se suscribe
@KafkaListener(topics = "publicaciones.registradas", groupId = "consultas-group")
public void alPublicacionRegistrada(PublicacionRegistradaEvent evento) {
  proyectorFichas.proyectarPublicacionRegistrada(evento);
}
```

## Consecuencias

### Positivas
- ✅ Flexibilidad: Elegir síncrono o asíncrono según la necesidad
- ✅ Resiliencia: Kafka permite desacoplamiento de fallos
- ✅ Escalabilidad: Los servicios no espera respuestas síncronas
- ✅ Consistencia eventual: Aceptable para la mayoría de casos

### Negativas
- ❌ Debugging más complejo (eventos pueden perderse o retrasarse)
- ❌ Gestión de orden de mensajes (Kafka no garantiza orden global)
- ❌ Necesidad de compensaciones en sagas distribuidas

## Alternativas Consideradas
1. **Solo síncrono (REST)**: Más simple pero acoplamiento temporal
2. **Solo asíncrono (Kafka)**: Máxima resiliencia pero latencia variable
3. **gRPC + Protocol Buffers**: Mayor eficiencia de red pero más complejo de debuggear

## Decisiones Relacionadas
- ADR-003: Persistencia y sincronización
- ADR-004: Manejo de transacciones distribuidas (Saga Pattern)
