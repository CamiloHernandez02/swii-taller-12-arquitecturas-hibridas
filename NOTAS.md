# Notas de Documentación - Taller Lección 12

## ADRs Obligatorios (Implementados)

1. **Descomposición en microservicios** (Taller 5-6)
   - Por qué 2 servicios en lugar de 1
   - Límites de dominio
   - Ventajas vs. complejidad

2. **Comunicación asincrónica** (Taller 6)
   - Por qué eventos
   - RabbitMQ vs. alternativas
   - Tolerancia a fallos

3. **Persistencia políglota** (Taller 11)
   - PostgreSQL para investigadores
   - MongoDB para publicaciones
   - Trade-offs

4. **CQRS** (Taller 8)
   - Cuando se aplicó
   - Niveles: modelos separados, stores separados
   - Alternativas consideradas

## ADRs de Propuestas (No Implementados)

- **Service Mesh (Istio/Linkerd)**: Por qué no se usó
- **Data Mesh**: Arquitectura de datos descentralizada
- **Event Sourcing avanzado**: Por qué parado en nivel básico
- Otros que el grupo identificó

## Plantilla ADR

Ver template en repo.

## Entrega

- Branch: `documentación-adr-1`
- Carpeta: `docs/adr/`
