# ALX Airbnb Clone Backend - Requirement Specifications

This document details the functional and non-functional requirements for key backend features of the Airbnb Clone application. These specifications include API endpoints, input/output structures, validation rules, and performance criteria.

---

## 1. Feature: User Authentication (Registration & Login)

### 1.1. User Registration

**Description:** Allows new users to create an account on the platform, specifying their role (guest or host).

* **API Endpoint:** `POST /api/v1/auth/register`
* **Input Specification (Request Body - JSON):**
    ```json
    {
      "first_name": "string",      // Required, min 2 chars, max 50
      "last_name": "string",       // Required, min 2 chars, max 50
      "email": "string",           // Required, valid email format, unique
      "password": "string",        // Required, min 8 chars, strong (e.g., 1 uppercase, 1 lowercase, 1 number, 1 special char)
      "phone_number": "string",    // Optional, valid phone number format, unique
      "role": "string"             // Required, enum: "guest" or "host"
    }
    ```
* **Output Specification (Response Body - JSON):**
    * **Success (201 Created):**
        ```json
        {
          "message": "User registered successfully.",
          "user_id": "uuid",
          "email": "string",
          "role": "string",
          "token": "string" // JWT token for immediate login
        }
        ```
    * **Error (400 Bad Request):** Invalid input (e.g., missing fields, invalid email, weak password).
        ```json
        {
          "error": "Invalid input data",
          "details": {
            "email": "Invalid format",
            "password": "Password too weak"
          }
        }
        ```
    * **Error (409 Conflict):** Email or phone number already exists.
        ```json
        {
          "error": "User already exists",
          "details": "Email or phone number already registered."
        }
        ```
* **Validation Rules:**
    * `first_name`, `last_name`: Must be non-empty strings, alphanumeric, 2-50 characters.
    * `email`: Must be a valid email format, unique in the database.
    * `password`: Must be at least 8 characters long, contain at least one uppercase letter, one lowercase letter, one digit, and one special character.
    * `phone_number`: Must be a valid international phone number format (optional, but if provided, must be unique).
    * `role`: Must be either "guest" or "host".
* **Performance Criteria:**
    * Response time: Max 500ms for 95% of requests.
    * Throughput: Support 100 concurrent registration requests per second.

### 1.2. User Login

**Description:** Authenticates an existing user and provides an authentication token.

* **API Endpoint:** `POST /api/v1/auth/login`
* **Input Specification (Request Body - JSON):**
    ```json
    {
      "email": "string",           // Required, valid email format
      "password": "string"         // Required
    }
    ```
* **Output Specification (Response Body - JSON):**
    * **Success (200 OK):**
        ```json
        {
          "message": "Login successful.",
          "user_id": "uuid",
          "email": "string",
          "role": "string",
          "token": "string" // JWT token
        }
        ```
    * **Error (400 Bad Request):** Invalid input (e.g., missing fields, invalid email format).
        ```json
        {
          "error": "Invalid input data",
          "details": "Email format is incorrect."
        }
        ```
    * **Error (401 Unauthorized):** Invalid credentials (email/password mismatch).
        ```json
        {
          "error": "Unauthorized",
          "details": "Invalid email or password."
        }
        ```
* **Validation Rules:**
    * `email`: Must be a valid email format.
    * `password`: Must be non-empty.
* **Performance Criteria:**
    * Response time: Max 300ms for 95% of requests.
    * Throughput: Support 200 concurrent login requests per second.

---

## 2. Feature: Property Management (Create Property)

**Description:** Allows an authenticated host to create and list a new property on the platform.

* **API Endpoint:** `POST /api/v1/properties`
* **Input Specification (Request Body - JSON):**
    ```json
    {
      "name": "string",          // Required, min 5 chars, max 255
      "description": "string",   // Required, min 20 chars
      "location": "string",      // Required, descriptive location (e.g., "Paris, France", "Downtown Apartment")
      "pricepernight": 150.00,   // Required, decimal, > 0
      "images": [                // Optional, array of image URLs or base64 (for upload)
        "string_url_or_base64"
      ],
      "num_bedrooms": 2,         // Optional, integer, >= 0, default 1
      "num_bathrooms": 1.5,      // Optional, decimal, >= 0, default 1.0
      "max_guests": 4,           // Optional, integer, >= 1, default 1
      "amenities": [             // Optional, array of amenity IDs or names
        "string_amenity_id_or_name"
      ]
    }
    ```
* **Output Specification (Response Body - JSON):**
    * **Success (201 Created):**
        ```json
        {
          "message": "Property listed successfully.",
          "property_id": "uuid",
          "name": "string",
          "location": "string",
          "pricepernight": 150.00
        }
        ```
    * **Error (400 Bad Request):** Invalid input data.
    * **Error (401 Unauthorized):** User not authenticated.
    * **Error (403 Forbidden):** Authenticated user is not a 'host' role.
* **Validation Rules:**
    * Request must be from an authenticated user with `host` role.
    * `name`: Must be a non-empty string, 5-255 characters.
    * `description`: Must be a non-empty string, at least 20 characters.
    * `location`: Must be a non-empty string.
    * `pricepernight`: Must be a positive decimal number.
    * `num_bedrooms`: Must be a non-negative integer.
    * `num_bathrooms`: Must be a non-negative decimal.
    * `max_guests`: Must be an integer greater than or equal to 1.
    * `images`: If provided, each item must be a valid URL or base64 encoded image string (backend handles storage).
    * `amenities`: If provided, each item must correspond to an existing amenity.
* **Performance Criteria:**
    * Response time: Max 800ms for 95% of requests (including image processing if direct upload).
    * Throughput: Support 50 concurrent property creation requests per second.

---

## 3. Feature: Booking System (Create Booking)

**Description:** Allows an authenticated guest to request a booking for an available property.

* **API Endpoint:** `POST /api/v1/bookings`
* **Input Specification (Request Body - JSON):**
    ```json
    {
      "property_id": "uuid",     // Required, ID of the property to book
      "start_date": "YYYY-MM-DD",// Required, future date
      "end_date": "YYYY-MM-DD",  // Required, future date, must be after start_date
      "num_guests": 2            // Required, integer, >= 1, <= property.max_guests
    }
    ```
* **Output Specification (Response Body - JSON):**
    * **Success (201 Created):**
        ```json
        {
          "message": "Booking request submitted successfully.",
          "booking_id": "uuid",
          "property_id": "uuid",
          "status": "pending",
          "total_price": 750.00
        }
        ```
    * **Error (400 Bad Request):** Invalid input (e.g., dates in past, end date before start date, num_guests exceeds max).
    * **Error (401 Unauthorized):** User not authenticated.
    * **Error (403 Forbidden):** Authenticated user is not a 'guest' role or is the host of the property.
    * **Error (404 Not Found):** Property ID does not exist.
    * **Error (409 Conflict):** Property not available for the requested dates, or already booked.
* **Validation Rules:**
    * Request must be from an authenticated user with `guest` role.
    * `property_id`: Must exist in the database.
    * `start_date`, `end_date`: Must be valid future dates. `end_date` must be strictly after `start_date`.
    * `num_guests`: Must be an integer >= 1 and <= the `max_guests` allowed for the specified `property_id`.
    * **Availability Check:** The backend must verify that the property is available for the entire `start_date` to `end_date` range (no existing bookings or host-blocked dates).
    * User making the booking cannot be the host of the property.
* **Performance Criteria:**
    * Response time: Max 400ms for 95% of requests.
    * Throughput: Support 150 concurrent booking requests per second.
