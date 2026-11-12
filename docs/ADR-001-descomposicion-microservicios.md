# ADR-001: Descomposición en Microservicios

## Estado
Aceptado

## Contexto
Se requiere construir una plataforma para gestionar investigadores y publicaciones científicas. El sistema necesita ser:
- Escalable: Cada componente debe escalar independientemente
- Resiliente: El fallo de un servicio no debería derribar toda la plataforma
- Mantenible: Equipos independientes pueden desarrollar cada microservicio

## Decisión
Descomponer la aplicación en microservicios basados en bounded contexts del dominio:

### Servicios Identificados

1. **investigadores-service** (Spring Boot + PostgreSQL)
   - Responsabilidad: Gestión de datos del investigador (CRUD)
   - Modelo: Investigador, Institución
   - Persistencia: PostgreSQL
   - Puerto: 8081

2. **publicaciones-service** (Spring Boot + MongoDB)
   - Responsabilidad: Gestión de publicaciones científicas
   - Modelo: Publicación, Autor
   - Persistencia: MongoDB
   - Puerto: 8082

3. **consultas-service** (Spring Boot + MongoDB - CQRS)
   - Responsabilidad: Modelo de lectura optimizado
   - Patrón: CQRS (Command Query Responsibility Segregation)
   - Persistencia: MongoDB (Read Model)
   - Puerto: 8083

4. **observabilidad-service** (Spring Boot)
   - Responsabilidad: Tracing, Métricas, Saga pattern
   - Herramientas: Micrometer, Prometheus, Brave
   - Puerto: 8084

5. **polyglot-service** (Spring Boot)
   - Responsabilidad: Demostración de persistencia políglota
   - Persistencia: PostgreSQL + MongoDB + Redis
   - Puerto: 8085

### Criterios de Descomposición

- **Cambios independientes**: Los servicios que cambian juntos pueden cambiar sin afectar otros
- **Datos independientes**: Cada servicio tiene su propia base de datos (Database per Service)
- **Ciclo de vida independiente**: Los servicios se despliegan independientemente
- **Escalabilidad selectiva**: Se puede escalar solamente el servicio que lo necesita

## Consecuencias

### Positivas
- ✅ Escalabilidad horizontal independiente
- ✅ Despliegue continuo sin necesidad de coordinar con otros servicios
- ✅ Libertad tecnológica: cada equipo puede elegir tecnologías apropiadas
- ✅ Fallos aislados: El error en un servicio no afecta los demás

### Negativas
- ❌ Complejidad operacional aumentada (necesita observabilidad, service discovery)
- ❌ Consistencia distribuida (necesita sagas, eventos para transacciones)
- ❌ Latencia de red entre servicios (vs. llamadas en-proceso)
- ❌ Debugging distribuido es más complejo

## Alternativas Consideradas
1. **Monolítico modular**: Más simple pero menos escalable
2. **Serverless Functions**: Mayor escalabilidad pero menos control sobre estado
3. **Arquitectura por capas horizontal**: Acoplamiento temporal entre equipos

## Decisiones Relacionadas
- ADR-002: Comunicación entre servicios
- ADR-003: Persistencia y sincronización de datos
