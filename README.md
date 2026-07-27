# Infraestructura - El Almacén de Películas Online

Este repositorio contiene la configuración de la infraestructura y orquestación de los microservicios (verticales) del sistema "El Almacén de Películas Online", dando cumplimiento al requerimiento **RT-9** sobre la documentación de los servicios, sus puertos HTTP y los eventos que se consumen/publican.

## Propósito

El propósito de este repositorio es proveer el entorno de ejecución centralizado utilizando **Docker Compose**. Aquí se levantan y conectan todos los microservicios, bases de datos (PostgreSQL y MongoDB), el gestor de identidad (Keycloak) y el broker de mensajería (RabbitMQ) necesarios para el funcionamiento de la aplicación.

## Servicios Expuestos (HTTP)

La infraestructura orquesta los siguientes microservicios y expone los siguientes puertos:

| Servicio / Vertical | Contenedor | Puerto Expuesto | Base de Datos |
|---------------------|----------------------|-----------------|---------------------|
| **Películas** | `peliculas-app` | `8080` | PostgreSQL (5432) |
| **Carrito** | `carrito-app` | `8082` | PostgreSQL (5433) |
| **Historial** | `historial-app` | `8083` | MongoDB (27017) |
| **Descuentos** | `descuentos-app` | `8084` | PostgreSQL (5436) |
| **Notificaciones** | `notificaciones-app` | `8085` | N/A |
| **Rating** | `rating-app` | `8086` | PostgreSQL (5435) |
| **Keycloak (Auth)** | `keycloak` | `9090` | PostgreSQL |
| **RabbitMQ** | `rabbitmq` | `15672` (Admin) | N/A |

## Eventos (Publicación y Consumo)

La comunicación asíncrona entre los microservicios se realiza a través de **RabbitMQ**, manejando los siguientes flujos de eventos:

- **Notificaciones:** 
  - **Consume:** Eventos de compra para el envío de correos.
  - **Exchange:** `compra.exchange`
  - **Routing Key:** `compra.event`
  - **Queue:** `compras.notificaciones.queue`

- **Descuentos:**
  - **Publica/Consume:** Eventos relacionados a la gestión de descuentos.
  - **Exchange:** `descuento_exchange`
  - **Routing Key:** `descuento_routing_key`

## Diagrama C4 (Contexto)

A continuación se presenta un diagrama de contenedor (Nivel 2 de C4) que ilustra la arquitectura de la infraestructura orquestada:

```mermaid
C4Container
    title Diagrama de Contenedores - Infraestructura de El Almacén de Películas Online

    Person(cliente, "Cliente", "Usuario autenticado que compra películas y deja ratings.")
    Person(admin, "Administrador", "Gestiona el catálogo de películas y descuentos.")

    System_Boundary(cinecloud, "El Almacén de Películas Online") {
        Container(api_gateway, "API Gateway / Frontend", "Monolito", "Punto de entrada de la aplicación.")
        
        Container(peliculas, "Películas", "Spring Boot", "Catálogo y detalle de películas. Puerto 8080.")
        ContainerDb(peliculas_db, "Películas DB", "PostgreSQL", "Almacena datos del catálogo.")
        
        Container(carrito, "Carrito de Compras", "Spring Boot", "Gestión del carrito y checkout. Puerto 8082.")
        ContainerDb(carrito_db, "Carrito DB", "PostgreSQL", "Almacena los carritos activos.")
        
        Container(historial, "Historial de Compras", "Spring Boot", "Registro de compras. Puerto 8083.")
        ContainerDb(historial_db, "Historial DB", "MongoDB", "Almacena historial (NoSQL).")
        
        Container(descuentos, "Descuentos", "Spring Boot", "Gestión de descuentos. Puerto 8084.")
        ContainerDb(descuentos_db, "Descuentos DB", "PostgreSQL", "Almacena los descuentos.")
        
        Container(notificaciones, "Notificaciones", "Spring Boot", "Envío de emails. Puerto 8085.")
        
        Container(rating, "Rating", "Spring Boot", "Votos y comentarios de películas. Puerto 8086.")
        ContainerDb(rating_db, "Rating DB", "PostgreSQL", "Almacena los ratings.")
        
        Container(keycloak, "Keycloak", "IAM", "Gestión de identidad y seguridad. Puerto 9090.")
        
        ContainerQueue(rabbitmq, "RabbitMQ", "Message Broker", "Comunicación asíncrona entre microservicios.")
    }

    Rel(cliente, api_gateway, "Navega y realiza compras")
    Rel(admin, api_gateway, "Administra catálogo")
    
    Rel(api_gateway, keycloak, "Autenticación (OAuth2/OIDC)")
    Rel(api_gateway, peliculas, "Consulta catálogo HTTP")
    Rel(api_gateway, carrito, "Gestiona carrito HTTP")
    Rel(api_gateway, historial, "Consulta compras HTTP")
    Rel(api_gateway, rating, "Deja rating HTTP")
    
    Rel(peliculas, peliculas_db, "Lee/Escribe")
    Rel(carrito, carrito_db, "Lee/Escribe")
    Rel(historial, historial_db, "Lee/Escribe")
    Rel(descuentos, descuentos_db, "Lee/Escribe")
    Rel(rating, rating_db, "Lee/Escribe")
    
    Rel(carrito, rabbitmq, "Publica evento de compra", "AMQP")
    Rel(rabbitmq, notificaciones, "Consume evento de compra", "AMQP")
    Rel(rabbitmq, historial, "Consume evento de compra", "AMQP")
```

## Instrucciones de Uso

Para levantar toda la infraestructura de la aplicación en modo desarrollo (incluyendo bases de datos, RabbitMQ y Keycloak), ejecutar el siguiente comando desde la raíz de este directorio:

```bash
docker-compose up -d --build
```

Para detener y eliminar los contenedores:

```bash
docker-compose down
```
