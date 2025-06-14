# hydra-health-ai Architecture

## Overview
AI-powered personalized hydration assistant for wearable devices, promoting consistent hydration and improved health.

### Problem Statement
Individuals struggle to maintain consistent hydration, leading to decreased energy, headaches, and health issues. Existing reminders are often ignored.

### Target Audience
Wearable device users (smartwatches, fitness trackers) concerned about hydration and health, including busy professionals, remote workers, and students, who want personalized recommendations and gentle reminders.

### Core Entities


## Design System

### Typography
- **Font**: Inter
- **Font Stack**: `'Inter', sans-serif`
- **Google Fonts**: https://fonts.google.com/specimen/Inter
- **Rationale**: Inter is chosen for its excellent readability at various sizes and its modern, clean design. It's also optimized for digital displays, making it a great choice for a health-focused application. Its variable font option allows for fine-grained control over weight and style, enhancing the user experience.

### Color Palette
A calm and reliable palette, promoting trust and clarity. We are using shades of blue for brand recognition, and greens to evoke a sense of nature. Accessibility is a key consideration, ensuring sufficient contrast for all users. The palette follows the WCAG AA guidelines for color contrast.

- **Primary**: `#3490dc` - Main brand color, used for headers and primary actions. (e.g., main call to action button, app bar background)
- **Secondary**: `#6cb2eb` - Supporting color for secondary elements. (e.g., less important buttons)
- **Accent**: `#38c172` - Used for call-to-action buttons and highlights related to wellness and hydration. (e.g., Success messages, progress indicators)
- **Neutral Text**: `#222f3e` - Primary text color for high contrast and readability.
- **Neutral Background**: `#f8fafc` - Main background color for content areas.
- **Neutral Border**: `#dae1e7` - For card borders, dividers, and form inputs.
- **Success**: `#38c172` - For success messages and confirmation. (e.g., Hydration goal achieved)
- **Warning**: `#ffed4a` - For warnings and non-critical alerts. (e.g., Low battery on wearable)
- **Danger**: `#e3342f` - For error messages and destructive actions. (e.g., Data synchronization failure)

## System Architecture

### 1. Core Components & Rationale
The architecture leverages Laravel for the backend (API and business logic) and React/Inertia.js with TypeScript for the frontend.  This separation allows for a scalable and maintainable application. Redis and Laravel Echo Server are used for real-time functionality.

*   **Laravel (Backend):** Chosen for its robust features, ORM (Eloquent), security features, and excellent developer experience.
*   **React/Inertia.js (Frontend):** React offers a component-based architecture ideal for building dynamic UIs. Inertia.js simplifies the integration with Laravel, allowing the use of Laravel's routing and controllers directly from React.
*   **TypeScript (Frontend):** Provides static typing, enhancing code maintainability and reducing runtime errors.
*   **Redis (Real-time Data):** An in-memory data store that's ideal for handling real-time updates and caching frequently accessed data.  [https://redis.io/]
*   **Laravel Echo Server (Real-time Server):** A Node.js server that handles WebSocket connections and broadcasts events to connected clients.  It simplifies real-time implementation with Laravel's event broadcasting.
*   **Wearable Device Integration:** This component will consist of APIs responsible for receiving data from connected wearable devices (e.g., Fitbit, Apple Watch).


### 2. Database Schema Design
The database schema will include the following tables (simplified):

*   **users:** id, name, email, password, created_at, updated_at, wearable_id (nullable - links to wearable device)
*   **profiles:** id, user_id (FK), age, weight, activity_level, location (latitude, longitude), created_at, updated_at
*   **hydration_data:** id, user_id (FK), timestamp, water_intake_ml, created_at, updated_at
*   **notifications:** id, user_id (FK), message, timestamp, is_read, created_at, updated_at
*   **wearable_devices:** id, user_id (FK), device_type, device_id, last_sync_time, created_at, updated_at

Appropriate indexes will be added to frequently queried columns for performance. Foreign key constraints are in place to maintain data integrity.

### 3. API Design & Key Endpoints
The API will follow RESTful principles and utilize JSON for data exchange. Authentication will be handled using Laravel Sanctum for API authentication.  Rate limiting will be implemented to prevent abuse. All API requests will require authentication.

Key Endpoints:

*   `POST /api/register`: User registration.
*   `POST /api/login`: User login (returns Sanctum API token).
*   `GET /api/user`: Get authenticated user information.
*   `GET /api/profile`: Get authenticated user's profile.
*   `PUT /api/profile`: Update authenticated user's profile.
*   `GET /api/hydration-data`: Get hydration data for the authenticated user.
*   `POST /api/hydration-data`: Add new hydration data for the authenticated user.
*   `GET /api/notifications`: Get notifications for the authenticated user.
*   `GET /api/wearable-data`: Get wearable device data for the authenticated user. (authenticated + wearable device connection required).
*   `POST /api/wearable-data`: Sync wearable device data (authenticated + wearable device connection required).


### 4. Frontend Structure (TypeScript)
The frontend will be structured using a component-based architecture with React and TypeScript. The project will leverage the 'laravel/react-starter-kit' structure with components organized into directories:

*   `src/`: Main source code directory.
    *   `Components/`: Reusable React components.
        *   `Dashboard/`: Dashboard-specific components.
        *   `Notification/`: Notification-related components.
        *   `Profile/`: Profile-related components.
        *   `UI/`: Generic UI components (buttons, inputs, etc.).
    *   `Pages/`: Inertia.js pages, representing different routes.
        *   `Dashboard.tsx`
        *   `Profile.tsx`
        *   `Notifications.tsx`
    *   `types/`: TypeScript type definitions for data models and API responses. Types for User, Profile, HydrationData, Notification. (Using generics and discriminated unions for type safety).
    *   `services/`: API service to handle HTTP requests to the backend using axios with type safety. (e.g., `UserService.ts`, `HydrationService.ts`).
    *   `hooks/`: Custom React hooks for state management and side effects. (e.g., `useHydrationData.ts`, `useNotifications.ts`).
    *   `context/` : React context to manage global application state such as user authentication information or hydration goals.

Global state management will leverage React Context API or a lightweight library like Zustand.  Form management will utilize a library like Formik or React Hook Form. This will enable faster form implementation and easier validation handling.

### 5. Real-time & Events Architecture
Real-time functionality will be implemented using Redis and Laravel Echo Server. Laravel events will be broadcast to Redis channels. Laravel Echo Server will subscribe to these channels and push updates to connected clients (browsers, wearable devices) via WebSockets.

Example Flow (New Hydration Data):

1.  User adds new hydration data (POST `/api/hydration-data`).
2.  Backend creates the `HydrationData` record and fires a `HydrationDataCreated` event.
3.  The `HydrationDataCreated` event is broadcast to a Redis channel (e.g., `hydration.user.{user_id}`).
4.  Laravel Echo Server receives the event.
5.  Laravel Echo Server pushes the event data to the connected client (browser). React component subscribes to this event and updates the UI in real-time. [https://laravel.com/docs/10.x/broadcasting] + [https://github.com/tlaverdure/laravel-echo-server]


### 6. Authentication & Authorization Flow
Authentication will be handled using Laravel Sanctum. Sanctum provides a lightweight API authentication system using tokens. For the web frontend, Inertia.js handles session management seamlessly.  For the mobile app/wearable device, API tokens will be used.

*   **Registration:** User provides credentials and a new user record is created. An API token is generated for the user.
*   **Login:** User provides credentials, and the system authenticates the user. An API token is returned.
*   **Protected Routes:** API endpoints are protected using Sanctum's `auth:sanctum` middleware.  This middleware verifies the presence and validity of the API token in the request header.
*   **Frontend:** Inertia.js persists the user session using cookies. After login, the user is redirected to the dashboard. If a user is not authenticated, they are redirected to the login page. [https://laravel.com/docs/10.x/sanctum]


### 7. Deployment & Scalability Plan
The application will be deployed using a containerized approach (Docker). The backend (Laravel), frontend (React), Redis, and Laravel Echo Server will each be deployed in separate containers. The containers will be orchestrated using Docker Compose for local development and Kubernetes for production deployment.

*   **Infrastructure:** AWS, Google Cloud Platform (GCP) or DigitalOcean are suitable options.
*   **Load Balancing:** A load balancer (e.g., Nginx or HAProxy) will distribute traffic across multiple backend instances.
*   **Database:** A managed database service (e.g., AWS RDS, Google Cloud SQL) will be used to simplify database management and scaling.
*   **Caching:** Redis will be used for caching frequently accessed data. Implement HTTP caching for static assets using a CDN.
*   **Scaling:** Horizontal scaling will be used to handle increased traffic.  Backend and frontend instances can be scaled independently based on resource utilization.


### 8. Testing Strategy
A comprehensive testing strategy will be implemented to ensure the quality and reliability of the application.

*   **Backend (Laravel):**
    *   **Unit Tests (Pest):** Test individual components and functions in isolation.  Focus on testing business logic, models, and controllers. [https://pestphp.com/]
    *   **Feature Tests (Pest):** Test API endpoints and user interactions. Simulate real user scenarios.
*   **Frontend (React/TypeScript):**
    *   **Unit Tests (Vitest):** Test individual React components and utility functions. [https://vitest.dev/]
    *   **Component Tests (React Testing Library):** Test React components in a more realistic environment, focusing on user interactions and rendering. [https://testing-library.com/docs/react-testing-library/intro/]
    *   **End-to-End Tests (Cypress or Playwright):** Test the entire application flow from the frontend to the backend.  Simulate user interactions and verify that the application behaves as expected. [https://www.cypress.io/] or [https://playwright.dev/]

Continuous Integration (CI) will be implemented using GitHub Actions or GitLab CI.  All tests will be run automatically on every code commit and pull request.

### 9. Security & Hardening Plan
A 'security-first' approach is mandatory, addressing OWASP Top 10 vulnerabilities. [https://owasp.org/Top10/]

*   **Input Validation:** Validate all user inputs on both the frontend and the backend. Sanitize data to prevent Cross-Site Scripting (XSS) attacks.
*   **Output Encoding:** Encode data before rendering it on the frontend to prevent XSS attacks.
*   **Authentication & Authorization:** Use Laravel Sanctum for authentication. Implement role-based access control (RBAC) to restrict access to sensitive data and functionality.  Use strong password policies and multi-factor authentication (MFA).
*   **SQL Injection Prevention:** Use Laravel's Eloquent ORM to prevent SQL injection attacks.  Avoid using raw SQL queries.
*   **Cross-Site Request Forgery (CSRF) Protection:** Laravel provides built-in CSRF protection. Ensure that CSRF protection is enabled for all forms.
*   **Security Headers:** Configure web server to send security headers (e.g., Content Security Policy (CSP), Strict-Transport-Security (HSTS), X-Frame-Options) to mitigate various attacks.
*   **Dependency Management:** Regularly update dependencies to patch security vulnerabilities. Use tools like `npm audit` or `composer audit` to identify and fix vulnerabilities.
*   **Regular Security Audits:** Conduct regular security audits to identify and address potential vulnerabilities. Consider using static analysis tools to identify security flaws in the code.
*   **Rate Limiting:** Implement rate limiting to prevent brute-force attacks and denial-of-service attacks.
*   **Data Encryption:** Encrypt sensitive data at rest and in transit.  Use HTTPS for all communication. Encrypt sensitive data in the database.


### 10. Logging & Observability
Comprehensive logging and monitoring are crucial for identifying and resolving issues in production.

*   **Logging:** Use Laravel's built-in logging system to log application events. Configure the `stderr` channel for sending logs to standard error.  Include relevant information in logs, such as timestamps, user IDs, and request details. [https://laravel.com/docs/10.x/logging]
*   **Monitoring:** Monitor key metrics such as CPU usage, memory usage, database performance, and API response times.  Use a monitoring tool like Prometheus or Grafana to visualize metrics. [https://prometheus.io/] or [https://grafana.com/]
*   **Error Tracking:** Use an error tracking service like Sentry to capture and track exceptions and errors. [https://sentry.io/]
*   **Application Performance Monitoring (APM):** Integrate an APM tool like New Relic or Datadog to monitor application performance and identify bottlenecks. [https://newrelic.com/] or [https://www.datadoghq.com/]
*   **Alerting:** Configure alerts to notify developers when critical issues occur (e.g., high error rate, slow response times). This includes setup for alerting on Redis and Laravel Echo Server errors.

## Feature Implementation Roadmap

### 1. Wearable device connection & data synchronization (activity level, location)
Allows users to connect their wearable devices (e.g., Fitbit, Apple Watch) to the application and synchronize data such as activity level and location.

**Database Changes:**
- Add `wearable_devices` table (id, user_id, device_type, device_id, last_sync_time, created_at, updated_at)
- Add `wearable_data` table (id, user_id, timestamp, step_count, location_latitude, location_longitude, created_at, updated_at)

**Backend Components:**
- UserController (for managing user profiles)
- WearableDeviceController (for managing device connections)
- WearableDataController (for handling data synchronization)
- WearableDevice model
- WearableData model
- Jobs for asynchronous data synchronization

**Frontend Components:**
- Profile page (for managing device connections)
- Dashboard page (for displaying wearable data)
- WearableDeviceModal.tsx (React component for handling device connections)

**API Endpoints:**
- GET /api/wearable-devices
- POST /api/wearable-devices
- DELETE /api/wearable-devices/{id}
- POST /api/wearable-data/sync

**Testing Requirements:**
- Unit tests for WearableDevice model and WearableData model
- Feature tests for WearableDeviceController and WearableDataController
- Component tests for WearableDeviceModal.tsx
- End-to-end tests for device connection and data synchronization


### 2. Personalized hydration recommendations based on user profile and environment
Provides personalized hydration recommendations based on user profile data (age, weight, activity level) and environmental factors (location, temperature).

**Database Changes:**
- Add `daily_hydration_goals` table (id, user_id, date, recommended_intake_ml, created_at, updated_at)
- Add `temperature_readings` table (id, location_latitude, location_longitude, temperature_celsius, timestamp, created_at, updated_at)

**Backend Components:**
- ProfileController (for retrieving user profile data)
- HydrationRecommendationService (for calculating recommendations)
- TemperatureService (for fetching temperature data from external API)
- DailyHydrationGoal model

**Frontend Components:**
- Dashboard page (for displaying hydration recommendations)

**API Endpoints:**
- GET /api/hydration-recommendations

**Testing Requirements:**
- Unit tests for HydrationRecommendationService and TemperatureService
- Feature tests for hydration recommendations API endpoint
- Component tests for the hydration recommendation display component on the dashboard


### 3. Contextual reminders delivered via wearable device notifications
Delivers contextual reminders to the user via wearable device notifications based on their activity level and location (e.g., 'You're at the gym, hydrate!').

**Backend Components:**
- NotificationService (for sending notifications)
- ActivityRecognitionService (for detecting user activity)
- LocationService (for determining user location)
- HydrationReminderScheduler (for scheduling reminders)

**Testing Requirements:**
- Unit tests for NotificationService, ActivityRecognitionService, and LocationService
- Integration tests for the HydrationReminderScheduler
- End-to-end tests for reminder delivery via wearable device


### 4. Automated tracking of water intake via voice input or wearable sensor data
Allows users to automatically track their water intake via voice input or wearable sensor data (if available).

**Backend Components:**
- VoiceInputService (for processing voice input)
- WearableSensorService (for reading sensor data)
- HydrationDataController (for adding hydration data)
- HydrationData model

**API Endpoints:**
- POST /api/voice-input/process
- POST /api/wearable-sensor/data

**Real-time Events:**
- HydrationDataCreated (broadcast when new hydration data is added)

**Testing Requirements:**
- Unit tests for VoiceInputService and WearableSensorService
- Feature tests for hydration data creation via voice input or wearable sensor data


### 5. Dashboard to visualize hydration progress and trends over time
Provides a dashboard for users to visualize their hydration progress and trends over time.

**Backend Components:**
- HydrationDataController (for retrieving hydration data)
- DashboardService (for calculating statistics and generating charts)

**Frontend Components:**
- Dashboard.tsx (main dashboard component)
- HydrationChart.tsx (React component for displaying hydration data charts)
- HydrationProgress.tsx (React component for displaying hydration progress)

**API Endpoints:**
- GET /api/dashboard-data

**Real-time Events:**
- HydrationDataCreated (to update the dashboard in real-time)

**Testing Requirements:**
- Unit tests for DashboardService
- Feature tests for dashboard data API endpoint
- Component tests for HydrationChart.tsx and HydrationProgress.tsx
- End-to-end tests for the dashboard functionality


## Metadata
- **Generated**: 2025-06-14T12:15:07.907652+00:00
- **Project Type**: Laravel React Starter Kit
- **Architecture Version**: 1.0.0

---
*This architecture document is maintained by the CodeWebMobile AI system and should be the source of truth for all development decisions.*
