# ARCHITECTURE.md - Proyecto Integrador B1 2026-2 G2

---

## 1. Visión y Principios Arquitecturales

El sistema **`jfinfotest/proyecto-integrador-b1-2026-2-g2`** está diseñado bajo los más altos estándares de la ingeniería de software empresarial moderna. La arquitectura combina los principios de **Clean Architecture** (Arquitectura Limpia) con el patrón de **Modular Monolith** (Monolito Modular), adaptado específicamente para el ecosistema **Java y Spring Boot**.

### Principios Fundamentales:
1. **Independencia de Frameworks y Librerías:** El núcleo de la lógica de negocio (Dominio) es completamente agnóstico a Spring Boot, bases de datos o librerías externas. Los frameworks son detalles de implementación que se ubican en las capas externas.
2. **Inversión de Dependencias (Dependency Inversion Principle - DIP):** Las dependencias apuntan exclusivamente hacia adentro. Las capas externas conocen a las internas, pero las internas jamás tienen conocimiento de las externas.
3. **Alto Acoplamiento Cohesivo por Módulo:** Cada dominio de negocio encapsula su propio modelo, lógica de aplicación y adaptadores de persistencia, facilitando una futura migración transparente hacia una arquitectura de microservicios si la escala del negocio lo demanda.
4. **Testabilidad Extrema:** Al separar los efectos secundarios (I/O, bases de datos, redes) de la lógica pura de negocio, la cobertura de pruebas unitarias y de integración se vuelve determinista y de ejecución ultra rápida.

---

## 2. Diagrama de Arquitectura Detallado

El siguiente diagrama en formato Mermaid detalla el flujo de control, las capas y la separación de responsabilidades basada en Puertos y Adaptadores (Arquitectura Hexagonal dentro del Monolito Modular).

```mermaid
graph TD
    subgraph Client Layer
        UI[Cliente / API Consumer]
    end

    subgraph Presentation Layer [Adaptadores de Entrada / Inbound Adapters]
        REST[Controladores REST / Spring MVC]
        DTO[Data Transfer Objects & Mappers]
    end

    subgraph Core Domain / Application Layer [Puertos de Entrada / Inbound Ports]
        UseCases[Casos de Uso / Servicios de Aplicación]
    end

    subgraph Domain Layer [Núcleo de Negocio / Domain]
        Entities[Entidades de Dominio / Value Objects]
        DomainServices[Servicios de Dominio]
    end

    subgraph Infrastructure Layer [Adaptadores de Salida / Outbound Adapters]
        Persistence[Repositorios JPA / Spring Data]
        DB[(Base de Datos Relacional / NoSQL)]
        ExternalAPI[Clientes HTTP / Servicios Externos]
    end

    %% Flujo de interacciones
    UI -->|HTTP Request| REST
    REST -->|Invoca DTO Mapping| DTO
    DTO -->|Puerto de Entrada| UseCases
    UseCases -->|Orquesta| Entities
    UseCases -->|Ejecuta lógica compleja| DomainServices
    UseCases -->|Puerto de Salida (Interface)| Persistence
    Persistence -->|Implementa Adaptador| DB
    UseCases -->|Llamada externa| ExternalAPI

    style Domain Layer fill:#f9f,stroke:#333,stroke-width:2px
    style Core Domain / Application Layer fill:#bbf,stroke:#333,stroke-width:2px
    style Presentation Layer fill:#dfd,stroke:#333,stroke-width:2px
    style Infrastructure Layer fill:#ff9,stroke:#333,stroke-width:2px
```

---

## 3. Desglose de Componentes y Módulos Clave

El árbol de directorios `src` sigue una estricta organización basada en Clean Architecture y Modular Monolith. 

### Árbol de Directorios Objetivo (`src/`)

```text
src/
├── main/
│   └── java/
│       └── com/
│           └── jfinfotest/
│               └── proyecto/
│                   ├── Application.java                 # Punto de entrada de Spring Boot
│                   ├── shared/                          # Componentes transversales (Excepciones globales, seguridad, configs)
│                   │   ├── config/
│                   │   ├── exception/
│                   │   └── utils/
│                   └── [modulo_negocio]/                # Ejemplo: module (ej. user, order, catalog)
│                       ├── domain/                      # Capa 1: Núcleo de Dominio (Puro Java)
│                       │   ├── model/                   # Entidades de negocio y Value Objects
│                       │   ├── exception/               # Excepciones de negocio
│                       │   └── repository/              # Puertos de salida (Interfaces de repositorio)
│                       ├── application/                 # Capa 2: Casos de Uso / Aplicación
│                       │   ├── usecase/                 # Implementaciones de los casos de uso
│                       │   └── service/                 # Servicios de aplicación
│                       └── infrastructure/              # Capa 3: Adaptadores e Infraestructura (Spring Boot / JPA)
│                           ├── inbound/                 # Adaptadores de Entrada (REST Controllers, Global Handlers)
│                           │   ├── rest/
│                           │   └── dto/
│                           └── outbound/                # Adaptadores de Salida (Persistence JPA, Mappers)
│                               ├── persistence/         # Spring Data JPA Entities y Repositories
│                               └── adapter/             # Implementación de los puertos de dominio
└── test/
    └── java/
        └── com/
            └── jfinfotest/
                └── proyecto/
                    ├── unit/                            # Pruebas unitarias de dominio y casos de uso
                    └── integration/                     # Pruebas de integración con Spring Boot Context y Testcontainers
```

### Justificación de la Estructura:
* **`shared/`**: Contiene configuraciones transversales que no pertenecen a ningún dominio específico (manejo global de excepciones, configuraciones de seguridad corporativa, interceptores).
* **`domain/`**: Cero dependencias externas. Contiene reglas de negocio puras, invariantes y contratos (puertos) que la infraestructura debe cumplir.
* **`application/`**: Orquestador de transacciones y flujos de trabajo de usuario. Implementa los casos de uso del sistema.
* **`infrastructure/`**: Contiene los detalles tecnológicos. Aquí residen las anotaciones de Spring (`@RestController`, `@Entity`, `@Repository`), frameworks de persistencia y adaptadores de red.

---

## 4. Flujo de Datos y Manejo del Estado

El flujo de datos se rige por una transferencia estrictamente tipada y unidireccional a través de fronteras arquitectónicas bien definidas:

1. **Recepción (Inbound):** El cliente envía un payload JSON que es capturado por el `@RestController` en la capa de infraestructura. Los datos se mapean a objetos de transferencia (`DTOs`) y se validan mediante anotaciones JSR-380 (`@NotNull`, `@Valid`, etc.).
2. **Traducción:** El Adaptador de Entrada convierte el DTO en un objeto de comando o directamente en un parámetro comprensible para el **Caso de Uso** (`Application Layer`).
3. **Procesamiento de Negocio:** El Caso de Uso invoca las **Entidades de Dominio**. Las entidades validan sus propias invariantes de negocio. Si una regla se viola, se lanza una excepción de negocio (`DomainException`).
4. **Persistencia (Outbound):** Si se requiere persistencia, el Caso de Uso interactúa con una interfaz de puerto (`Repository Interface`). El adaptador de infraestructura implementa dicha interfaz, mapeando la entidad de dominio a una entidad JPA (`@Entity`) y delegando en Spring Data.
5. **Retorno:** El resultado fluye de regreso convirtiendo las entidades de infraestructura a entidades de dominio, luego a DTOs de salida, y finalmente serializándose a JSON hacia el cliente HTTP.

---

## 5. Registro de Decisiones de Arquitectura (ADR)

### ADR 1: Adopción de Modular Monolith sobre Microservicios Distribuidos
* **Estado:** Aceptado
* **Contexto:** El proyecto actual requiere alta velocidad de entrega, simplicidad operacional y un acotado dominio inicial. Una arquitectura de microservicios introduciría complejidad innecesaria en red, despliegue y consistencia eventual (Sagas).
* **Decisión:** Desarrollar un Monolito Modular estrictamente particionado por paquetes de dominio.
* **Consecuencias:** Permite mantener la separación lógica propia de los microservicios, facilitando la refactorización futura y extrayendo módulos independientes hacia servicios de red aislados cuando el crecimiento del negocio lo justifique.

### ADR 2: Uso de Clean Architecture con Puertos y Adaptadores
* **Estado:** Aceptado
* **Contexto:** Necesidad de aislar la lógica de negocio de los cambios tecnológicos del framework Spring Boot o de la base de datos subyacente.
* **Decisión:** Implementar Clean Architecture estricta, aplicando inversión de dependencias mediante interfaces en el dominio.
* **Consecuencias:** Aumento inicial de boilerplate (creación de mappers y DTOs adicionales), compensado con creces por una alta testabilidad, mantenibilidad a largo plazo y desacoplamiento tecnológico.

### ADR 3: Gestión de Transacciones en la Capa de Aplicación
* **Estado:** Aceptado
* **Contexto:** ¿Dónde deben colocarse las anotaciones `@Transactional` de Spring?
* **Decisión:** Ubicar la gestión transaccional exclusivamente en los servicios de la capa de aplicación (Casos de Uso).
* **Consecuencias:** Se garantiza que el dominio permanezca libre de anotaciones del framework de persistencia, manteniendo la transaccionalidad acoplada al caso de uso de negocio correspondiente.

---

## 6. Estrategia de Seguridad, Escalabilidad y Resiliencia

### Seguridad
* **Autenticación y Autorización:** Stateless mediante **JSON Web Tokens (JWT)** o OAuth2 Resource Server, asegurando que los servicios no mantengan sesiones en memoria.
* **Validación y Sanitización:** Validación estricta en la frontera de entrada (DTOs) para prevenir ataques de inyección (SQL Injection, XSS). Las entidades de dominio actúan como última línea de defensa validando invariantes.
* **Principio de Mínimo Privilegio:** Aislamiento de endpoints mediante control de acceso basado en roles (RBAC) a nivel de Spring Security Filter Chain.

### Escalabilidad
* **Statelessness (Ausencia de Estado):** Las instancias del backend en Spring Boot no almacenan estado de sesión, permitiendo un escalado horizontal elástico detrás de un balanceador de carga (Load Balancer).
* **Optimización de Base de Datos:** Uso de índices adecuados en columnas de búsqueda frecuente, paginación obligatoria en consultas masivas (`Pageable`) para evitar desbordamientos de memoria (OOM), y conexiones gestionadas mediante HikariCP.

### Resiliencia
* **Manejo Centralizado de Excepciones:** Uso de `@ControllerAdvice` para capturar excepciones de negocio, técnicas e infraestructura, retornando contratos de error estandarizados (RFC 7807 - Problem Details for HTTP APIs).
* **Timeouts y Circuit Breakers:** Preparado para integrar mecanismos de tolerancia a fallos (Resilience4j) en los adaptadores de salida que consumen APIs o recursos externos de red.