# Online Classroom Booking System

## Project Overview
This project aims to solve the problem of offline classroom booking, which often leads to duplication and difficulty in verification. We are building an online system to streamline this process.

## Core Features
-   **User Authentication**: Login and account management with roles for Student/Teacher (ROLE_USER) and Admin (ROLE_ADMIN).
-   **Room Management**: View room listings and details.
-   **Booking System**: Book rooms by selecting time slots, with the system checking for time conflicts.
-   **Booking Approval**: Bookings will be in a "PENDING" status, and Admins can approve or reject them.
-   **Booking History**: Users can view their own booking history, and Admins can view a dashboard.

## Core Architecture (Strictly Enforced)
-   **Framework**: Spring Boot 3 (or newer)
-   **Language**: Java 17 (or newer)
-   **Architecture**: 3-Tier Architecture (Controller, Service, Repository)
    -   **Controller (API Layer)**: Uses `@RestController` for RESTful APIs.
    -   **Service (Business Logic Layer)**: Contains core business logic (e.g., conflict detection) using `@Service`.
    -   **Repository (Data Access Layer)**: Uses Spring Data JPA (Hibernate) and `@Repository`.
-   **Database**: PostgreSQL (or MySQL)
-   **Authentication**: Spring Security 6 (or newer) + JWT (JSON Web Tokens)
-   **Dependency Injection**: Always use Constructor Injection (with `final` dependencies).

## Package Structure
All code resides under `com.project.booking` and is divided by feature:
```
com.project.booking
├── auth          (User, Authentication, Security)
│   ├── controller
│   ├── dto
│   ├── service
│   ├── repository
│   └── model (User Entity)
├── room          (Room Management)
│   ├── controller
│   ├── dto
│   ├── service
│   ├── repository
│   └── model (Room Entity)
├── booking       (Booking, Approval, History)
│   ├── controller
│   ├── dto
│   ├── service
│   ├── repository
│   └── model (Booking Entity)
├── dashboard     (Admin Dashboard)
│   ├── controller
│   ├── dto
│   └── service
├── config        (SecurityConfig, AppConfig)
└── common        (Global Exception Handling, Enums)
```

## Core Data Entities
The following are the main JPA Entities. Their core fields and relationships must not be changed.

### User (`auth.model`)
-   `Long id` (PK)
-   `String username` (Unique)
-   `String password` (Hashed)
-   `String email` (Unique)
-   `UserRole role` (Enum: `ROLE_USER`, `ROLE_ADMIN`)

### Room (`room.model`)
-   `Long id` (PK)
-   `String roomCode` (e.g., "IT-404")
-   `String name`
-   `int capacity`
-   `String equipment`

### Booking (`booking.model`)
-   `Long id` (PK)
-   `LocalDateTime startTime`
-   `LocalDateTime endTime`
-   `String purpose`
-   `BookingStatus status` (Enum: `PENDING`, `APPROVED`, `REJECTED`, `CANCELLED`)
-   **Relationships**:
    -   `@ManyToOne User user` (Booker)
    -   `@ManyToOne Room room` (Booked Room)

## API Contract
The following endpoints are agreed upon. Controllers must be created to match these (refer to previous documentation for permissions).

-   `POST /api/v1/auth/register`
-   `POST /api/v1/auth/login`
-   `GET /api/v1/auth/me`
-   `POST /api/v1/rooms` (Admin)
-   `GET /api/v1/rooms` (User/Admin)
-   `GET /api/v1/rooms/{roomId}` (User/Admin)
-   `POST /api/v1/bookings` (User creates booking - status PENDING)
-   `GET /api/v1/bookings/my-history` (User views history)
-   `PUT /api/v1/bookings/{bookingId}/cancel` (User)
-   `GET /api/v1/bookings` (Admin views all, Filter PENDING)
-   `PUT /api/v1/bookings/{bookingId}/approve` (Admin approves)
-   `PUT /api/v1/bookings/{bookingId}/reject` (Admin rejects)
-   `GET /api/v1/dashboard/summary` (Admin)

## DTO Strategy
-   **Rule**: JPA Entities (User, Room, Booking) must NOT be sent directly to the API Layer.
-   **Controller**: Must only receive Request DTOs (e.g., `CreateBookingRequest`) and return Response DTOs (e.g., `BookingResponse`).
-   **Service**: Responsible for converting DTOs to Entities (for saving) and Entities to DTOs (for returning).
-   **Java Records**: Use Java Records for DTOs for conciseness and immutability.
-   **Example DTOs**:
    -   `CreateBookingRequest` (receives `roomId`, `startTime`, `endTime`, `purpose`)
    -   `BookingResponse` (sends `id`, `roomName`, `startTime`, `endTime`, `status`, `username`)
    -   `RoomResponse` (sends `id`, `roomCode`, `name`, `capacity`)

## Coding Standards
-   **Error Handling**: Use `@RestControllerAdvice` and Custom Exceptions (e.g., `ResourceNotFoundException`, `BookingConflictException`).
-   **Business Logic**: Complex logic, such as "booking time conflict detection," must reside ONLY in the Service Layer, not in the Controller.
-   **Security**:
    -   Use `@PreAuthorize` (e.g., `@PreAuthorize("hasRole('ADMIN')")`) on Controller/Service methods to enforce permissions.
    -   Retrieve logged-in user information via `SecurityContextHolder` or `AuthenticationPrincipal`.
-   **Immutability**: Strive to use Immutable objects and DTOs (Java Records).

## Setup and Running the Application

### Prerequisites
-   Java 17 or higher
-   Maven
-   PostgreSQL (or MySQL) database

### Database Configuration
1.  Open the `src/main/resources/application.properties` file.
2.  Update the database connection details (`spring.datasource.url`, `username`, `password`) to match your PostgreSQL (or MySQL) setup.
    ```properties
    # Database Configuration
    spring.datasource.url=jdbc:postgresql://localhost:5432/your_database_name
    spring.datasource.username=your_username
    spring.datasource.password=your_password
    ```
3.  **Important**: Change the `application.security.jwt.secret-key` to a long, random, and secure string. You can generate one from a tool like [AllKeysGenerator](https://www.allkeysgenerator.com/Random/Security-Encryption-Key-Generator.aspx).
    ```properties
    application.security.jwt.secret-key=YOUR_VERY_LONG_AND_SECURE_JWT_SECRET_KEY
    ```

### Build and Run
1.  Navigate to the root directory of the project in your terminal.
2.  Build the project using Maven:
    ```bash
    mvn clean install
    ```
3.  Run the Spring Boot application:
    ```bash
    mvn spring-boot:run
    ```

The application should start on `http://localhost:8080` (default port).
