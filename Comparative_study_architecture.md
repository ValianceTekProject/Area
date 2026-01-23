# Comparative Study: Microservices vs Client–Server (Monolithic) Architecture

## Comparison Criteria

| Criterion                         | Client–Server (Monolithic)                                              | Microservices                                                                                 |
| --------------------------------- | ----------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Definition                        | A single server application provides all functionalities to the client. | The application is divided into independent services communicating via APIs.                  |
| Setup Complexity                  | Easy to deploy, development is centralized.                             | Complex: each service requires deployment and orchestration (Docker/K8s).                     |
| Scalability                       | Vertical scalability (increase server power).                           | Horizontal scalability: each microservice can be scaled independently.                        |
| Maintenance                       | Changes affect the entire system, higher risk of side effects.          | Easier maintenance: each service is independent, isolated testing possible.                   |
| Performance                       | Very performant for small projects, low overhead.                       | May have network overhead (inter-service API calls), but performant for distributed projects. |
| Deployment                        | Simple: single server/single application.                               | Complex: each microservice must be deployed and monitored.                                    |
| Fault Tolerance                   | A failure affects the entire system.                                    | A failure affects only the concerned microservice (isolated).                                 |
| Extensibility / Flexibility       | Limited: adding new technology or module can be difficult.              | Highly flexible: each service can use a different stack (Go, Node, etc.).                     |
| Logging / Debugging               | Centralized: easy to monitor.                                           | Distributed: requires centralized monitoring tools (ELK, Prometheus).                         |
| Development / Infrastructure Cost | Low initially, but can become complicated as the application grows.     | High initially (infrastructure + orchestration), but better suited for large projects.        |

## Advantages and Disadvantages in the Context of the AREA Project

AREA currently consists of:

* Web Frontend: Next.js
* Mobile: React Native
* Backend: Go + PostgreSQL
* Deployment: Docker Compose

| Option                     | Relevance for AREA                                                                                                             |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Client–Server / Monolithic | Suitable for a small project with a single backend and internal APIs. Easy to manage with Docker Compose.                    |
| Microservices              |  Complex for the project size. More relevant for hundreds of services with high load. Deployment and monitoring are heavier. |

## Conclusion

For AREA, the client-server (monolithic) architecture is appropriate: quick to implement, easy to maintain, and sufficient for the expected load.

Microservices would be overkill at this stage but could be considered if AREA evolves into a large-scale multi-service platform.
