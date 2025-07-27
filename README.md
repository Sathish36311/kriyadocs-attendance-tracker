# Kriyadocs Frontend Web Developer Case Study: Attendance Tracking System

## Overview

This repository contains the solution for the Kriyadocs Frontend Web Developer case study. It implements an attendance tracking system using ReactJS for the frontend and NodeJS with Fastify for the backend, supported by MongoDB for data storage. The application is designed to capture attendee images via the device camera, record entry/exit, and perform image matching for verification. The entire solution is containerized using Docker.

## Technical Outline and Architectural Design

### 1. Project Goal

To build a web-based attendance tracking system capable of recording attendee entry/exit with image verification.

### 2. Technology Stack Chosen

* **Frontend:** ReactJS (chosen for its component-based architecture and widespread adoption)
    * Key Libraries: `react-webcam` for camera access, `axios` for API calls, `styled-components` for styling.
* **Backend:** NodeJS with Fastify (chosen for its high performance and developer-friendly experience)
    * Key Libraries: `mongoose` (MongoDB ODM), `multer` (for file uploads), `face-api.js` (for image matching).
* **Database:** MongoDB (containerized with Docker)
* **Containerization:** Docker, Docker Compose
* **Testing:** Jest, React Testing Library, Supertest, Cypress

### 3. Application Architecture

The application follows a standard client-server architecture:

* **Frontend (ReactJS SPA):** Handles user interaction, camera access, and communicates with the backend via RESTful API calls.
* **Backend (NodeJS/Fastify API):** Manages data persistence (MongoDB), image processing, and facial recognition logic.
* **MongoDB:** Stores attendee profiles (name, email, facial descriptors) and attendance records.

### 4. Backend API Endpoints

#### `POST /api/attendees/checkin-checkout`

* **Purpose:** Records an attendance event (ENTRY or EXIT) for an attendee based on facial recognition.
* **Request Method:** `POST`
* **Content-Type:** `multipart/form-data`
* **Request Body:**
    * `image`: The image captured from the device camera (as a file/blob).
* **Logic:**
    1.  Receives the captured image.
    2.  Extracts facial descriptors from the incoming image using `face-api.js`.
    3.  Queries the MongoDB `attendees` collection to retrieve all stored attendee profiles and their facial descriptors.
    4.  Compares the incoming image's descriptors against all stored descriptors to find the closest match.
    5.  **If a match is found:**
        * Identifies the matched attendee.
        * Retrieves the attendee's last attendance record from `attendance_records`.
        * If the last record was `ENTRY`, a new `EXIT` record is created.
        * If the last record was `EXIT` (or no previous record), a new `ENTRY` record is created.
        * The new attendance record (type, timestamp, attendee ID) is saved to MongoDB.
    6.  **If no match is found:** Returns an error indicating "Person not recognized."
* **Success Response (200 OK):**
    ```json
    {
        "success": true,
        "message": "Welcome, Sathish! Entry recorded at 2025-07-26T09:30:00Z",
        "attendeeName": "Sathish",
        "type": "ENTRY",
        "timestamp": "2025-07-26T09:30:00Z"
    }
    ```
    or
    ```json
    {
        "success": true,
        "message": "Goodbye, Sathish! Exit recorded at 2025-07-26T17:00:00Z",
        "attendeeName": "Sathish",
        "type": "EXIT",
        "timestamp": "2025-07-26T17:00:00Z"
    }
    ```
* **Error Response (400 Bad Request / 404 Not Found):**
    ```json
    {
        "success": false,
        "message": "Error: Person not recognized. Please register or try again."
    }
    ```

### 5. Database Schema (MongoDB)

#### `attendees` collection

```json
{
    "_id": "<ObjectId>",
    "name": "<String>",           
    "email": "<String>",         
    "referenceImagePath": "<String>", 
    "facialDescriptors": [ <Number>, <Number>, ... ],
    "createdAt": "<ISODate>",
    "updatedAt": "<ISODate>"
}
