# Dentwise – Dental Platform with AI Voice Agent

**Course:** Internet Application Development
**Teacher:** Sir Zaeem Tariq

**Group Members**

| Name | Student ID |
|---|---|
| Muhammad Mujtaba Khan Suri | B22110006103 |
| Wajahat Ali | B22110006176 |
| Raza Jaun | B22110006135 |
| Muhammad Tashfeen Malik | B22110006118 |

---

## Table of Contents

1. [Introduction / Project Overview](#1-introduction--project-overview)
2. [Target Users](#2-target-users)
   - [2.1 Patients / General Users](#21-patients--general-users)
   - [2.2 Dental Practice Administrators](#22-dental-practice-administrators)
3. [System Architecture](#3-system-architecture)
   - [3.1 Application Components](#31-application-components)
   - [3.2 External Services](#32-external-services)
   - [3.3 Database](#33-database)
   - [3.4 Deployment and Hosting](#34-deployment-and-hosting)
4. [Actors and Use Cases](#4-actors-and-use-cases)
5. [Database Design](#5-database-design)
   - [5.1 Entities and Attributes](#51-entities-and-attributes)
   - [5.2 Relationships](#52-relationships)
   - [5.3 Enumerations](#53-enumerations)
   - [5.4 Table Mappings and Identifiers](#54-table-mappings-and-identifiers)
6. [Appointment Booking Workflow](#6-appointment-booking-workflow)
7. [AI Voice Assistant (Vapi Integration)](#7-ai-voice-assistant-vapi-integration)
8. [Authentication and Subscription Management](#8-authentication-and-subscription-management)
9. [Admin Functionality](#9-admin-functionality)
10. [Email Notification System](#10-email-notification-system)
11. [UI Screenshots](#11-UI-screenshots) 
12. [Technology Stack Summary](#12-technology-stack-summary)
13. [Conclusion](#13-conclusion)

---

## 1. Introduction / Project Overview

Dentwise is a dental clinic management platform that allows patients to discover dentists, view their details, and book appointments through a web application, with an optional voice-based booking channel powered by an AI voice agent. The platform also provides a dedicated administrative interface through which a dental practice can manage doctors, oversee appointments, and review practice statistics.

The application is built as a single Next.js project that serves both the frontend user interface and the backend logic through server actions and API routes. Application data — users, doctors, and appointments — is persisted in a PostgreSQL database that is accessed through the Prisma ORM. User authentication and subscription/billing management are delegated to Clerk, an external identity and billing service. Email communication, including booking confirmations and invoices, is handled by Resend, and the AI voice booking experience is powered by Vapi through its Web SDK. The application is deployed and hosted on Sevalla.

This document presents the technical documentation of Dentwise as it is actually implemented: its system architecture, actors and use cases, database schema, appointment booking workflow, AI voice assistant integration, authentication and subscription handling, administrative functionality, and email notification system. The explanations are written to be consistent with the five diagrams accompanying it — the System Architecture Diagram, the Use Case Diagram, the Entity Relationship Diagram, and the two Major System Workflow (sequence) diagrams for standard and voice-assisted appointment booking.

## 2. Target Users

Dentwise is designed around two primary categories of users, each with a distinct set of responsibilities and permissions within the system: patients (general users) who consume the platform to find and book dental care, and dental practice administrators who operate and maintain the platform on behalf of the clinic.

### 2.1 Patients / General Users

After registering or signing in, a patient is able to:

- Register for an account or sign in using the supported authentication methods.
- Browse the list of available dentists and view individual dentist details.
- Book an appointment by selecting a dentist, a service or appointment reason, and an available date and time.
- View and manage their own booked appointments.
- Use the AI voice assistant to book an appointment through natural conversation, when the patient's account is eligible (Pro plan).
- Manage their subscription and billing, including viewing available plans, upgrading to the Pro plan, and reviewing invoices.

### 2.2 Dental Practice Administrators

Through the Admin Dashboard, an administrator is able to:

- Access the Admin Dashboard, separate from the patient-facing interface.
- Manage doctors registered on the platform, including adding new doctors and editing existing doctor information.
- Activate or deactivate a doctor, controlling whether that doctor is currently available for booking.
- View and manage patient appointments across the practice.
- View practice statistics, such as the total number of doctors, the total number of appointments, and the number of completed appointments.

> These responsibilities correspond directly to the actors and use cases shown in the Use Case Diagram (Figure 2), which identifies the Patient/User and the Admin as the two primary actors of the Dentwise platform.

## 3. System Architecture

Dentwise follows a modern web application architecture in which a single Next.js codebase serves both the client-facing interface and the server-side application logic, backed by a PostgreSQL database and integrated with a small set of external, purpose-specific services. The overall architecture is illustrated in Figure 1.

### 3.1 Application Components

The core of the system is the **Next.js application**, which combines the frontend and backend within one project:

- **UI / Pages** — built using React components, providing the patient-facing pages for browsing dentists and booking appointments.
- **Dashboard / Admin Interface** — a separate set of pages used exclusively by administrators to manage doctors, appointments, and statistics.
- **Server Actions / API Routes** — implement the booking logic, business logic, and authentication checks that connect the UI to the database and external services.
- **Integrations** — supporting libraries used within the application layer: TanStack Query for data fetching, React Hook Form for form handling, Zod for schema validation, and Recharts for rendering statistics on the admin dashboard.

Data access from the application layer is performed through **Prisma ORM**, which is responsible for querying and managing the data stored in the underlying database.

### 3.2 External Services

Dentwise integrates with three external, third-party services, each responsible for a well-defined concern outside of the application's own database:

- **Clerk** — handles authentication and user management (Google, GitHub, and email/password sign-in) as well as subscription and billing for Pro plans. Clerk user data is synchronized into the application through the user's `clerkId`.
- **Vapi** — provides the AI voice agent used for voice-based appointment booking, accessed in the browser through the Vapi Web SDK. Access to Vapi is restricted to accounts on a Pro subscription plan.
- **Resend** — the email service used to send booking confirmations, invoices, and other notifications to patients.

### 3.3 Database

Application data is persisted in a **PostgreSQL** database. All queries and data modifications are performed through Prisma ORM rather than direct SQL calls from the application code. The structure of the database is described in Section 5 and illustrated in the Entity Relationship Diagram (Figure 3).

### 3.4 Deployment and Hosting

The application is deployed and hosted on **Sevalla**, which provides Git-based deployment, auto-scaling, and the production environment in which the Next.js application runs.

![System Architecture Diagram of Dentwise](images/fig1_architecture.png)

*Figure 1: System Architecture Diagram of Dentwise*

## 4. Actors and Use Cases

Dentwise defines two primary human actors — the **Patient/User** and the **Admin** — along with a set of external systems that the application interacts with to fulfil its use cases. These actors, their use cases, and the relationships between them are shown in the Use Case Diagram (Figure 2).

**Patient / User**

- **Account & Authentication** — Register / Sign In, Verify Email using a six-digit code, and Manage Profile.
- **Browse & Book Appointment** — Browse Dentists and View Dentist Details, leading into the Book Appointment use case, which follows the sequence Dentist → Service/Reason → Date/Time → Confirm. This use case includes Select Dentist, Select Service/Appointment Reason, Check Booked Time Slots, View Available Time Slots, Select Date & Time, and Confirm Appointment.
- **Subscription Management** — View Plans, Upgrade Plan, and Manage Billing/Invoices. An active Pro Plan is required («requires») before the patient can use the AI Voice Assistant use case.
- **AI Voice Assistant** — Use AI Voice Assistant, available only once the Pro Plan requirement is satisfied.

**Admin**

- **Admin Dashboard** — View Appointments and Manage Appointments; Manage Doctors, which includes Add Doctor, Edit Doctor, and Activate/Deactivate Doctor; and View Statistics, which includes View Total Doctors, View Total Appointments, and View Completed Appointments.
- **System Support** — Handle Errors, Log Activity, and Manage Notifications, supporting the reliable operation of the platform.

**External Systems**

Clerk (authentication and subscription/billing), Vapi (AI voice agent, Pro plans only), Resend (booking confirmations, invoices, and notifications), and Sevalla (deployment). These are external services consumed by the application; they are not actors that initiate use cases themselves.

![Use Case Diagram of Dentwise](images/fig2_usecase.png)

*Figure 2: Use Case Diagram of Dentwise*

## 5. Database Design

Dentwise persists its data in a PostgreSQL database, defined and accessed through a Prisma schema consisting of three models: **User**, **Doctor**, and **Appointment**. The structure below reflects the actual schema, reproduced from the Entity Relationship Diagram (Figure 3), which is the source of truth for the database design.

### 5.1 Entities and Attributes

**User (table: `users`)**

| Field | Type | Constraints |
|---|---|---|
| id | String (CUID) | Primary Key |
| clerkId | String | Unique |
| email | String | Unique |
| firstName | String | Nullable |
| lastName | String | Nullable |
| phone | String | Nullable |
| createdAt | DateTime | – |
| updatedAt | DateTime | – |

**Doctor (table: `doctors`)**

| Field | Type | Constraints |
|---|---|---|
| id | String (CUID) | Primary Key |
| name | String | – |
| email | String | Unique |
| phone | String | – |
| speciality | String | – |
| bio | String | Nullable |
| imageUrl | String | – |
| gender | Gender | Enum |
| isActive | Boolean | – |
| createdAt | DateTime | – |
| updatedAt | DateTime | – |

**Appointment (table: `appointments`)**

| Field | Type | Constraints |
|---|---|---|
| id | String (CUID) | Primary Key |
| date | DateTime | – |
| time | String (HH:mm) | – |
| duration | Int | – |
| status | AppointmentStatus | Enum |
| notes | String | Nullable |
| reason | String | Nullable |
| createdAt | DateTime | – |
| updatedAt | DateTime | – |
| userId | String (CUID) | Foreign Key → users.id |
| doctorId | String (CUID) | Foreign Key → doctors.id |

### 5.2 Relationships

- **User → Appointment**: one-to-many («has»). A single user may have many appointments; each appointment belongs to exactly one user.
- **Doctor → Appointment**: one-to-many («belongs to»). A single doctor may be associated with many appointments; each appointment belongs to exactly one doctor.

### 5.3 Enumerations

| Enum | Values | Used In |
|---|---|---|
| Gender | MALE, FEMALE | doctors.gender |
| AppointmentStatus | CONFIRMED, COMPLETED | appointments.status |

### 5.4 Table Mappings and Identifiers

Each Prisma model maps to a PostgreSQL table of the same conceptual entity: the `User` model maps to the `users` table, the `Doctor` model maps to the `doctors` table, and the `Appointment` model maps to the `appointments` table. All primary key identifiers across the three tables are of type String, generated as a **CUID** (Collision-resistant Unique Identifier) rather than a UUID. No additional database entities exist in the current schema beyond User, Doctor, and Appointment.

![Entity Relationship Diagram of Dentwise](images/fig3_er.png)

*Figure 3: Entity Relationship Diagram of Dentwise*

## 6. Appointment Booking Workflow

### 6.1 Booking Process Overview

The standard appointment booking workflow, as implemented in Dentwise, consists of five major stages:

1. **Login / Register** — the patient signs in or creates a new account; Clerk handles authentication and establishes a user session.
2. **Browse Doctors & Services** — the patient views the list of available doctors, their details, and the services or appointment reasons offered.
3. **Select Date & Time** — the patient selects a preferred date and is shown the time slots available for the chosen doctor.
4. **Confirm Booking** — the system validates the selected slot and creates the appointment record in the database.
5. **Receive Confirmation** — the patient sees a confirmation in the UI, receives a confirmation email through Resend, and may optionally receive confirmation via the AI voice assistant (Vapi).

### 6.2 Detailed Sequence of Operations

The detailed interaction between the Patient, the Next.js Frontend, the Backend (Server Actions), the PostgreSQL Database, Clerk, and Resend is shown in the sequence diagram in Figure 4. The key steps are as follows:

- The patient opens the application and signs in; the frontend forwards the authentication request to Clerk, which returns an authenticated session.
- The frontend requests the list of doctors and services from the backend, which queries the database and returns the matching data for display.
- After the patient selects a date and time, the frontend asks the backend to check slot availability. At this point, the system checks the existing appointments already stored for the selected doctor and date, so that only time slots that are not already booked are presented to the patient as available.
- Once a valid, available slot is selected and the booking is confirmed, the backend validates the request and creates the appointment record in PostgreSQL through Prisma. The newly created record is reflected immediately in subsequent availability checks for that doctor and date.
- The backend returns the confirmation data to the frontend, which displays the booking confirmation to the patient.
- The backend sends a confirmation email through Resend, and, where applicable, an optional voice call confirmation may be triggered through Vapi.

This flow corresponds directly to the detailed booking steps shown in the System Architecture Diagram (Figure 1): Select Dentist, Select Date, Get Booked Slots, View Available Slots, Select Time, and Confirm Booking, after which the appointment is stored in PostgreSQL with a status of `CONFIRMED` and the associated reason or notes.

![Major System Workflow – Appointment Booking Sequence Diagram of Dentwise](images/fig4_workflow_booking.png)

*Figure 4: Major System Workflow – Appointment Booking Sequence Diagram of Dentwise*

## 7. AI Voice Assistant (Vapi Integration)

### 7.1 Eligibility and Access

Dentwise offers an alternative, voice-based booking channel using the **Vapi** AI voice agent, accessed through the Vapi Web SDK within the browser. Access to the AI Voice Assistant is gated behind subscription eligibility: only patients on a Pro subscription plan, managed through Clerk's subscription and billing functionality, are able to use this feature. This is reflected in the Use Case Diagram (Figure 2), where the Use AI Voice Assistant use case explicitly requires the Pro Plan.

### 7.2 Voice Booking Sequence

The interaction flow for voice-based booking, shown in the sequence diagram in Figure 5, proceeds as follows:

- The patient calls or activates the AI voice assistant and speaks their request.
- Vapi converts the speech to text and, through natural conversation, asks the patient for any additional details needed to complete the booking.
- Once sufficient information has been gathered, Vapi sends the extracted intent and data (such as service, date, and time) to the Dentwise frontend, which forwards the request to the backend through an API route.
- The backend validates and processes the request, querying the database to check doctor and time-slot availability, and returns the result (whether the slot is available or already booked).
- If the request is valid, the backend creates the appointment record in PostgreSQL, the same way as in the standard booking flow.
- Confirmation details are sent back through the frontend to Vapi, which converts the response to speech and confirms the booking to the patient as a voice message.
- In parallel, the backend sends a confirmation email to the patient through Resend.

Vapi is responsible for the real-time audio interaction — including call-start/call-end events, speech-start/speech-end events, transcript messages, and error handling — but it does not itself store appointment or patient data. All appointment records created through the voice channel are persisted exclusively in the same PostgreSQL `appointments` table used by the standard booking flow; there is no separate voice-session database table.

![Major System Workflow – AI Voice Assistant Booking Sequence Diagram of Dentwise](images/fig5_workflow_voice.png)

*Figure 5: Major System Workflow – AI Voice Assistant Booking Sequence Diagram of Dentwise*

## 8. Authentication and Subscription Management

Authentication and subscription/billing functionality in Dentwise is delegated entirely to **Clerk**, an external identity and billing service. Clerk is not part of the application's own PostgreSQL schema; it is an external service that the Next.js application communicates with over its own auth and subscription APIs.

Clerk provides the following capabilities within the platform:

- **Authentication** — account registration and sign-in through Google, GitHub, or email and password, along with email verification using a six-digit code, and profile management.
- **Subscription & Billing** — management of the platform's subscription plans, including viewing available plans, upgrading to the Pro plan, and managing billing and invoices. The Pro plan is what determines a patient's eligibility to use the AI Voice Assistant, as described in Section 7.

Because the application still needs to associate appointments with a local application user, the authenticated Clerk identity is synchronized with the application's own `User` record in PostgreSQL. Each `User` row stores a unique `clerkId` field, which links that Clerk account to its corresponding local user, allowing the `Appointment` model to reference the correct user through the standard `userId` foreign key. Subscription, billing, and plan data itself remain managed by Clerk and are not represented as separate Prisma models or tables within the Dentwise database.

## 9. Admin Functionality

Administrators operate the platform through a dedicated Admin Dashboard, distinct from the patient-facing interface, built as part of the same Next.js application. The following functionality is available to the Admin actor, consistent with the Admin Dashboard use cases shown in Figure 2:

- **Access the Admin Dashboard** — a separate administrative interface for operating the platform.
- **Manage Doctors** — add new doctors to the platform, edit existing doctor information (such as name, speciality, contact details, and bio), and activate or deactivate a doctor using the `isActive` field on the Doctor model, controlling whether that doctor currently appears as available for booking.
- **View and Manage Appointments** — review appointments booked across the practice and manage their status.
- **View Statistics** — view practice-level statistics derived from the stored data, specifically the total number of doctors, the total number of appointments, and the number of appointments with a `COMPLETED` status.

The Use Case Diagram additionally identifies a System Support group of use cases — Handle Errors, Log Activity, and Manage Notifications — which support the reliable day-to-day operation of the platform alongside the core administrative tasks described above.

## 10. Email Notification System

Dentwise uses **Resend** as its external email delivery service. Resend is invoked by the backend (server actions / API routes) after a relevant event has been persisted to the database; it is not represented as a database entity within the Dentwise schema.

Based on the system architecture, Resend is responsible for the following categories of outgoing email:

- **Booking confirmations** — sent to the patient after an appointment has been successfully created, whether through the standard booking flow (Section 6) or the AI voice assistant booking flow (Section 7).
- **Invoices** — sent in connection with the subscription and billing functionality managed through Clerk.
- **Notifications** — general notification emails sent to patients as required by the application.

In both the standard and voice-assisted booking sequences, the email step occurs after the appointment has already been created in PostgreSQL, so that the confirmation reflects a successfully stored booking rather than a pending or unvalidated request.

## 11. UI Screenshots

This section presents the main user-interface screens of the Dentwise platform.

![fig6](images/fig6.png)

*fig6*

![fig7](images/fig7.png)

*fig7*

![fig8](images/fig8.png)

*fig8*

![fig9](images/fig9.png)

*fig9*

![fig10](images/fig10.png)

*fig10*

![fig11](images/fig11.png)

*fig11*

![fig12](images/fig12.png)

*fig12*

![fig13](images/fig13.png)

*fig13*

![fig14](images/fig14.png)

*fig14*

![fig15](images/fig15.png)

*fig15*

![fig16](images/fig16.png)

*fig16*

![fig17](images/fig17.png)

*fig17*

![fig18](images/fig18.png)

*fig18*

![fig19](images/fig19.png)

*fig19*

![fig20](images/fig20.png)

*fig20*

![fig21](images/fig21.png)

*fig21*

![fig22](images/fig22.png)

*fig22*

![fig23](images/fig23.png)

*fig23*

![fig24](images/fig24.png)

*fig24*

![fig25](images/fig25.png)

*fig25*

![fig26](images/fig26.png)

*fig26*

![fig27](images/fig27.png)

*fig27*

## 12. Technology Stack Summary

| Layer / Category | Technology / Service | Role |
|---|---|---|
| Frontend | Next.js (React, TypeScript) | UI / Pages and Admin Dashboard interface |
| Backend | Next.js Server Actions / API Routes | Booking logic, business logic, authentication checks |
| ORM | Prisma | Querying and managing application data |
| Database | PostgreSQL | Persistent storage for Users, Doctors, and Appointments |
| Authentication & Billing | Clerk (external service) | User authentication and subscription/billing management |
| AI Voice Agent | Vapi (external service, Web SDK) | Voice-based appointment booking for Pro plan users |
| Email Service | Resend (external service) | Booking confirmations, invoices, and notifications |
| Deployment / Hosting | Sevalla | Git-based deployment, auto-scaling, production hosting |
| Supporting Libraries | TanStack Query, React Hook Form, Zod, Recharts | Data fetching, form handling, validation, statistics charts |

## 13. Conclusion

This documentation has presented the Dentwise dental platform as it is implemented: a Next.js application combining a patient-facing interface and an administrative dashboard, backed by a PostgreSQL database accessed through Prisma, and integrated with Clerk for authentication and subscription management, Vapi for AI voice-based appointment booking, and Resend for email notifications, with the application deployed on Sevalla.

The database design centers on three entities — User, Doctor, and Appointment — related through two one-to-many relationships, with identifiers generated as CUIDs and two supporting enumerations, Gender and AppointmentStatus.

The appointment booking workflow, in both its standard and voice-assisted forms, follows a consistent path from doctor and slot selection through availability checking, appointment creation, and confirmation via email and, where applicable, voice.

Administrative functionality allows the practice to manage doctors and appointments and to monitor overall activity through basic statistics.

The UI screenshots from `fig6` through `fig27` provide a visual representation of the application's implemented patient-facing and administrative interfaces.

Together with the five accompanying diagrams — the System Architecture Diagram, the Use Case Diagram, the Entity Relationship Diagram, and the two Major System Workflow sequence diagrams — this document provides a complete technical description of the Dentwise platform as submitted for the Internet Application Development course.