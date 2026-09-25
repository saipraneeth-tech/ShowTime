# ShowTime

ShowTime is a full-stack movie and event ticket-booking platform. Users can discover movies, browse events, select seats or event zones, complete a booking flow, view QR-code tickets, manage wishlists, and submit ratings. Venue owners can manage venues, schedule movies, create events, upload images, and review booking analytics.

The repository contains two applications:

- `FRONTEND`: React 19 and Vite single-page application.
- `BACKEND`: Java 17 and Spring Boot REST API backed by MySQL.

## Features

### Customer experience

- Public landing page, login, and registration.
- Firebase-assisted Google login, email/password authentication, email-link authentication, and password reset.
- Movie discovery using TMDB, including search, language filtering, now-playing and popular titles, recommendations, movie details, and cast information.
- Event browsing and event details.
- Movie booking flow with theatre, showtime, seat, and payment screens.
- Event booking flow with date, zone, capacity, and payment screens.
- Live blocked-seat and event-zone availability checks.
- QR-code ticket generation and ticket image export.
- Booking history, cancellation, public ticket lookup, wishlist management, profile image upload, ratings, reviews, settings, and help/support.

### Venue-owner experience

- Owner dashboard and access control.
- Venue creation and management.
- Movie scheduling and show management.
- Event creation and event image upload.
- Booking details and movie/event analytics.

## Technology Stack

### Frontend

- React 19
- Vite 7
- React Router 6
- Tailwind CSS 3
- Framer Motion
- Lucide React
- Firebase Authentication
- TMDB API
- `qrcode.react` for ticket QR codes
- `html-to-image` for ticket export
- React Toastify for notifications

### Backend

- Java 17
- Spring Boot 3.5
- Spring Web
- Spring Data JPA and Hibernate
- Spring Security
- JWT authentication using JJWT
- MySQL Connector/J
- Lombok
- Jackson
- Twilio SDK for SMS verification
- Maven Wrapper

## Repository Structure

```text
ShowTime/
|-- BACKEND/
|   |-- pom.xml
|   |-- mvnw / mvnw.cmd
|   |-- src/main/java/com/excelr/
|   |   |-- config/       Security and web configuration
|   |   |-- controller/   REST API controllers
|   |   |-- entity/       JPA entities and enums
|   |   |-- repository/   Spring Data repositories
|   |   |-- security/     JWT authentication and utilities
|   |   |-- service/      Application services
|   |-- src/main/resources/
|       |-- application.properties
|       |-- migration.sql
|-- FRONTEND/
|   |-- package.json
|   |-- vite.config.js
|   |-- src/
|       |-- components/   Booking, user, owner, and shared UI
|       |-- services/     TMDB integration
|       |-- Firebase/     Firebase client setup
|       |-- AppContent.jsx
|       |-- index.css
|-- .gitignore
|-- README.md
```

## Prerequisites

Install the following before running the project locally:

- Git
- Java Development Kit (JDK) 17
- Node.js 18 or newer and npm
- MySQL 8 or compatible MySQL server
- A Firebase project for the authentication flows used by the frontend
- TMDB API credentials for movie discovery
- Twilio credentials if SMS verification is required

## Backend Setup

1. Create a MySQL database, or allow the default local connection to create the `showtime` database automatically.
2. Open a terminal in `BACKEND`.
3. Set the environment variables required by your environment.
4. Start Spring Boot with the Maven Wrapper.

### Windows PowerShell

```powershell
cd BACKEND
$env:RAILWAY_DB_URL = "jdbc:mysql://localhost:3306/showtime?createDatabaseIfNotExist=true"
$env:RAILWAY_DB_USERNAME = "root"
$env:RAILWAY_DB_PASSWORD = "your-local-password"
.\mvnw.cmd spring-boot:run
```

The command above should contain `.\mvnw.cmd` in a normal PowerShell terminal:

```powershell
.\mvnw.cmd spring-boot:run
```

### macOS or Linux

```bash
cd BACKEND
export RAILWAY_DB_URL='jdbc:mysql://localhost:3306/showtime?createDatabaseIfNotExist=true'
export RAILWAY_DB_USERNAME='root'
export RAILWAY_DB_PASSWORD='your-local-password'
./mvnw spring-boot:run
```

The API starts on `http://localhost:8080`.

### Backend environment variables

The checked-in `application.properties` contains no database password. Configure secrets through the environment:

| Variable | Purpose |
| --- | --- |
| `RAILWAY_DB_URL` | JDBC connection URL. The default is a local `showtime` database. |
| `RAILWAY_DB_USERNAME` | Database username. |
| `RAILWAY_DB_PASSWORD` | Database password. |
| `TWILIO_ACCOUNT_SID` | Twilio account identifier. |
| `TWILIO_AUTH_TOKEN` | Twilio authentication token. |
| `TWILIO_VERIFY_SERVICE_SID` | Twilio Verify service identifier. |
| `TWILIO_PHONE_NUMBER` | Twilio phone number used by the SMS integration. |
| `JWT_SECRET` | Recommended production JWT signing secret if supported by the deployed configuration. |

SMS verification requires the Twilio variables. Gmail SMTP host and port are configured, but Gmail username and app-password settings are currently commented out.

## Frontend Setup

Open a second terminal:

```bash
cd FRONTEND
npm install
npm run dev
```

Vite prints the local development URL, normally `http://localhost:5173`.

Available scripts:

```bash
npm run dev      # Start the Vite development server
npm run build    # Create a production build
npm run lint     # Run ESLint
npm run preview  # Preview the production build locally
```

The current frontend uses a deployed backend URL in application source rather than a Vite proxy or documented frontend `.env` file. When developing against the local backend, update the API base URL references to `http://localhost:8080` or move that value into a frontend environment variable before deployment.

The TMDB and Firebase client configuration is also currently stored in frontend source files. Client-side Firebase configuration is designed to be visible, but access should be restricted in Firebase project settings. TMDB credentials should be moved to environment variables or a protected backend proxy before production hardening.

## API Overview

The backend exposes these main route groups:

| Route group | Responsibility |
| --- | --- |
| `/api/auth` | Registration, login, Google login, user profile, and SMS OTP operations |
| `/api/events` | Event creation, listing, retrieval, and owner event queries |
| `/api/venues` | Venue CRUD and owner venue listing |
| `/api/schedules` | Movie schedule creation and venue schedules |
| `/api/shows` | Shows by venue/date, show summaries, and movie deletion |
| `/api/bookings` | Movie/event bookings, cancellation, user bookings, blocked seats, availability, and public ticket lookup |
| `/api/ratings` | Ratings, reviews, and rating summaries |
| `/api/wishlist` | Wishlist retrieval, toggling, and deletion |
| `/api/analytics` | Movie and event booking analytics |
| `/api/upload` | Profile/event image upload and TMDB image proxy operations |

For exact request and response contracts, see the controller classes in `BACKEND/src/main/java/com/excelr/controller` and the entity classes in `BACKEND/src/main/java/com/excelr/entity`.

## Database

The application uses MySQL with Hibernate. The normal local database name is `showtime`. Hibernate is configured with `spring.jpa.hibernate.ddl-auto=update`, so the schema is updated from the entity model when the backend starts.

The main domain entities cover:

- Users and authentication data
- Venues and venue types
- Events
- Movie schedules and shows
- Movie and event bookings
- Ratings and reviews
- Wishlists

### Migration warning

`BACKEND/src/main/resources/migration.sql` is a manual, destructive migration. It drops and recreates tables related to shows, movie schedules, and bookings for the current seat-pricing model. Do not run it against production data without a backup and a deliberate migration plan.

`BACKEND/fix_db_constraints.sql` contains manual foreign-key cleanup and cascading-delete statements. Review it against the live schema before executing it.

There is currently no Flyway or Liquibase migration pipeline.

## Authentication and Authorization

The backend uses Spring Security and JWT authentication. The frontend stores the authenticated session token in browser storage and sends it with protected API requests. Owner-only screens and endpoints require the appropriate owner role.

For production deployments:

- Provide a strong, unique JWT secret through deployment configuration.
- Set a deliberate token expiration time.
- Restrict CORS to the deployed frontend origin.
- Rotate any credentials that may previously have been exposed.
- Keep database, Twilio, and other service credentials out of Git.

## Deployment Notes

The frontend contains `FRONTEND/vercel.json`, which rewrites SPA routes to `index.html` for Vercel deployments. The frontend currently targets a deployed Railway backend URL.

The backend does not include a Dockerfile or a provider-specific deployment manifest. Configure the deployment platform with:

- Java 17
- The Maven Wrapper or Maven build command
- Port `8080`, or the platform-provided port if the application is updated to read it
- MySQL connection variables
- Twilio variables when SMS is enabled
- A persistent storage strategy for uploaded files

Uploaded images are currently stored in a local `uploads/` directory and served by the backend. Ephemeral cloud instances can lose local files after redeployment, so production should use object storage or a persistent volume.

## Current Limitations

- The payment screens are currently a frontend simulation; they do not connect to a payment gateway.
- Bookings are stored as completed/confirmed by the current backend flow.
- The frontend backend URL is hardcoded in multiple source files.
- TMDB and Firebase configuration are present in frontend source files.
- CORS is permissive and should be restricted for production.
- Local filesystem uploads are not durable on many cloud platforms.
- SMS service behavior may fall back to development logging when Twilio fails; this should not be used as a production verification mechanism.
- Automated integration and end-to-end test coverage is limited.

## Verification Checklist

Before opening a pull request or deploying:

```bash
cd FRONTEND
npm run lint
npm run build
```

Then start the backend and verify:

1. The backend connects to the intended MySQL database.
2. Registration and login return a token.
3. Movie and event lists load in the frontend.
4. Seat and event-zone availability is updated after a booking.
5. Protected user and owner routes reject unauthenticated requests.
6. Image upload storage works on the target deployment platform.

## Contributing

1. Create a feature branch from `main`.
2. Keep secrets in local environment configuration, never in committed files.
3. Run frontend lint and build checks.
4. Run the backend test suite with `mvnw.cmd test` or `./mvnw test`.
5. Describe database or environment changes in the pull request.

## License

No project license has been defined in this repository yet.