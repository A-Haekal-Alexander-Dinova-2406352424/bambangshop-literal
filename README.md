# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [x] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [x] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [x] Commit: `Implement publish function in Program service and Program controller.`
    -   [x] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [x] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
Using `DashMap<String, DashMap<String, Subscriber>>` makes the repository fit the access pattern of this tutorial. The outer `DashMap` groups subscribers by product type, so notification lookup stays focused on one topic instead of scanning every subscriber. The inner `DashMap` uses subscriber URL as a key, which makes insertion and deletion straightforward because each receiver is uniquely identified by its callback URL. This structure also reduces manual synchronization work because `DashMap` already provides concurrent access semantics.

If we only used a `Vec<Subscriber>` for all subscribers, the code would become less efficient and less clear. Every subscribe, list, and unsubscribe operation would need extra iteration logic to filter by product type and find the matching subscriber URL. That would mix grouping logic into many functions and make the notification flow more error-prone. With the current nested map approach, the repository expresses the domain model directly: one product type can have many subscribers.

`DashMap` is also appropriate here because the publisher can receive multiple requests and later notify subscribers from multiple threads. Compared with a plain `HashMap`, we would otherwise need to add our own synchronization wrapper to avoid data races. With `DashMap`, the repository stays simpler while still being safe for concurrent reads and writes.

#### Reflection Publisher-2
In MVC, the Model is often described broadly, but separating repository and service layers keeps each unit focused. The repository layer concentrates on data access and storage rules, while the service layer concentrates on business logic and application flow. This follows separation of concerns and makes the code easier to maintain because storage details do not leak into controller code or into the data structs themselves.

If we only used the Model layer for everything, each model would need to know too much. `Product` would need to understand persistence and notification flow, `Subscriber` would need to understand storage and delivery orchestration, and `Notification` could become entangled with request handling. That would increase coupling between models, make changes riskier, and blur the boundary between plain data and business behavior. The result would be heavier models that are harder to test and harder to extend.

Postman helps a lot in this tutorial because the system is driven by HTTP interactions across two applications. It makes it easy to repeat requests for subscribe, unsubscribe, create, publish, and delete without manually rebuilding each request. Features that are especially useful are saved collections, configurable request bodies and query parameters, and the ability to observe response payloads quickly while iterating on endpoint behavior.

#### Reflection Publisher-3
This tutorial uses the Push variation of the Observer pattern. The publisher prepares the notification payload and actively sends it to every subscribed receiver through each subscriber's `update()` method. The receivers do not ask the publisher for updates on their own; they only receive notifications when the publisher decides to push them.

If we used the Pull variation instead, receivers would need to contact the publisher again whenever they wanted the latest information. One advantage is that the receiver could control when it fetches data and potentially request more details only when needed. The disadvantages for this tutorial are higher receiver complexity, additional API design for fetching notification data, and weaker immediacy because the subscriber has to initiate the follow-up request instead of receiving the full payload directly.

Without multithreading in the notification process, the publisher would send updates to subscribers one by one on the same execution flow. That means a slow or unreachable receiver could delay the whole request, including the response to the shop owner's create, publish, or delete action. With separate threads, the publisher can trigger each notification independently, so one slow subscriber does not block the others as much and the notification process scales better when more subscribers are registered.
