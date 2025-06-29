# 📄 Backend Requirements Documentation

This document outlines the **technical and functional requirements** for five key backend features of the Airbnb Clone system.

---

## 🔑 Core Functionalities

---

## 🔐 1. User Authentication

### Functional Requirements
- Allow users to register and log in as guests or hosts.
- Secure user sessions using JWT.
- Support OAuth (Google/Facebook) login.

### API Endpoints
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/oauth/google`
- `POST /api/auth/oauth/facebook`
- `GET /api/auth/profile`
- `PUT /api/auth/profile`

### Input/Output Specifications
- **Input**: `{ email, password, full_name, role }`
- **Output**: `{ access_token, refresh_token, user_info }`

### Validation Rules
- Email must be unique and valid.
- Password must meet complexity requirements (min 8 characters, alphanumeric).
- Role must be either `"guest"` or `"host"`.

### Performance Criteria
- Auth response time < 300ms.
- Stateless JWT-based authentication.

---

## 🏠 2. Property Management

### Functional Requirements
- Hosts can add, update, and delete listings.
- Listings include title, description, price, amenities, images, and availability.

### API Endpoints
- `POST /api/properties/`
- `GET /api/properties/:id/`
- `PUT /api/properties/:id/`
- `DELETE /api/properties/:id/`

### Input/Output Specifications
- **Input**: `{ title, description, price, location, images, amenities, availability }`
- **Output**: Full property object including `property_id`, `host_id`, timestamps.

### Validation Rules
- Required: title, description, price, location.
- Only the owner can edit/delete their listing.
- Price must be a positive float.

### Performance Criteria
- Property creation < 500ms.
- Asynchronous image upload or pre-signed URL usage.

---

## 🔎 3. Search and Filtering

### Functional Requirements
- Users can search and filter properties.
- Supports location, price range, guest count, and amenities.
- Pagination and sorting supported.

### API Endpoints
- `GET /api/properties/search?location=&min_price=&max_price=&guests=&amenities=`

### Input/Output Specifications
- **Input**: Query params for filters.
- **Output**: Paginated list of matching properties.

### Validation Rules
- Default 10 results/page, max 50.
- Amenities must be array or comma-separated.

### Performance Criteria
- Search must respond in < 800ms under normal load.
- Indexes and caching for optimized queries.

---

## 💳 4. Payment Integration

### Functional Requirements
- Guests pay via Stripe or PayPal.
- Hosts receive payouts post-checkout.
- Link payments to booking IDs.

### API Endpoints
- `POST /api/payments/charge`
- `POST /api/payments/payout`
- `GET /api/payments/:booking_id/`

### Input/Output Specifications
- **Input**: `{ booking_id, payment_method, amount }`
- **Output**: `{ status, transaction_id, confirmation_url }`

### Validation Rules
- Amount must match booking total.
- Valid booking ID required.
- Secure payment method.

### Performance Criteria
- Resilient to timeouts, retries enabled.
- Compliant with PCI-DSS and HTTPS required.

---

## 📆 5. Booking System

### Functional Requirements
- Guests can book properties.
- Validate availability and prevent overlap.
- Support cancellations and status tracking.

### API Endpoints
- `POST /api/bookings/`
- `GET /api/bookings/:id/`
- `PUT /api/bookings/:id/cancel`
- `GET /api/bookings/user/:user_id/`

### Input/Output Specifications
- **Input**: `{ property_id, check_in, check_out, guests }`
- **Output**: `{ booking_id, status, total_price, confirmation }`

### Validation Rules
- Check-in must be before check-out.
- Booking dates must not overlap.
- Guest count ≤ max capacity.

### Performance Criteria
- Confirmation response < 700ms.
- Safe concurrent operations using DB transactions.

---

## 🌟 6. Reviews and Ratings

### Functional Requirements
- Guests can leave reviews and ratings for properties.
- Hosts can respond to reviews.
- Reviews must be linked to completed bookings.

### API Endpoints
- `POST /api/reviews/`
- `GET /api/reviews/property/:property_id/`
- `GET /api/reviews/user/:user_id/`
- `POST /api/reviews/:review_id/reply`

### Input/Output Specifications
- **Input**: `{ rating, comment, booking_id }`
- **Output**: Review object with user info, rating, comment, and timestamps.

### Validation Rules
- Reviews only allowed on completed bookings.
- Rating must be between 1 and 5.

### Performance Criteria
- Review submission < 400ms.
- Reviews fetchable via indexed queries per property.

---

## 🔔 7. Notifications System

### Functional Requirements
- Send notifications via email and in-app for events like:
  - Booking confirmations
  - Cancellations
  - Payment status updates

### API Endpoints
- `POST /api/notifications/`
- `GET /api/notifications/user/:user_id/`

### Input/Output Specifications
- **Input**: `{ type, user_id, content }`
- **Output**: `{ message_id, status }`

### Validation Rules
- Must include a valid `type` and target user.
- Avoid duplicate notifications for the same event.

### Performance Criteria
- Notification delivery within 5 seconds of triggering event.
- Asynchronous background processing recommended.

---

## 🛠 8. Admin Dashboard

### Functional Requirements
- Admins can:
  - View and manage users
  - View and manage property listings
  - View all bookings and transactions
  - Delete inappropriate content

### API Endpoints
- `GET /api/admin/users/`
- `PUT /api/admin/users/:id/status`
- `DELETE /api/admin/properties/:id`
- `GET /api/admin/bookings/`
- `GET /api/admin/payments/`

### Input/Output Specifications
- Admin access token required for all endpoints.
- Output: Paginated and filterable datasets.

### Validation Rules
- Admin-only access (RBAC required).
- Logs all admin actions for audit purposes.

### Performance Criteria
- Admin queries < 1000ms for large datasets.
- Uses optimized queries with pagination.

---




## 🛠️ Technical Requirement

---

### 1. 📦 Database Management

- **Database**: PostgreSQL (preferred) or MySQL  
- **Tables**:
  - `users` – Guest and host data
  - `properties` – Property listings
  - `bookings` – Reservation records
  - `payments` – Payment and payout details
  - `reviews` – Guest reviews tied to bookings
  - `notifications` – Email/in-app notifications

- **Constraints & Optimization**:
  - Foreign key relationships
  - Unique constraint on `email`, `property_id`
  - Indexed columns: `location`, `user_id`
  - Normalized structure (3NF)
  - Use connection pooling (e.g., PgBouncer)

---

### 2. 🌐 API Development

- **Architecture**: RESTful API  
- **Versioning**: `/api/v1/`  
- **HTTP Status Codes**: Proper usage of 200, 201, 401, 403, 404, 422, 500

#### 🔑 Auth Endpoints

- `POST /api/v1/auth/register/`  
  - Input: `{ name, email, password, role }`  
  - Output: `{ token, user_id }`  
  - Validation: Unique email, strong password

- `POST /api/v1/auth/login/`  
  - Input: `{ email, password }`  
  - Output: `{ token }`  
  - Errors: 401 Unauthorized

#### 🏠 Property Endpoints

- `GET /api/v1/properties/`  
  - Filters: `?location=&guests=&price_min=&price_max=`  
  - Output: Paginated property list

- `POST /api/v1/properties/`  
  - Input: Property details + media  
  - Role: Host only

#### 📅 Booking Endpoints

- `POST /api/v1/bookings/`  
  - Input: `{ property_id, check_in, check_out }`  
  - Output: `{ booking_id, status }`  
  - Validation: Prevent date overlaps

- `DELETE /api/v1/bookings/{id}/cancel/`  
  - Role: Guest or host

---

### 3. 🔐 Authentication & Authorization

- **Auth Method**: JWT (JSON Web Token)  
  - Access Token: 15 minutes  
  - Refresh Token: 7 days

- **Roles**: Guest, Host, Admin  
- **Access Control**:
  - Guests: Book, Review  
  - Hosts: Create/Edit Listings, View Bookings  
  - Admins: Manage all resources

---

### 4. 🗂️ File Storage (Scenario Based)

- **Storage Option**: AWS S3 or Cloudinary  
- **Usage**:
  - Profile Photos → `/media/users/`  
  - Property Images → `/media/properties/`

- **Validation**:
  - Types: JPG, PNG  
  - Size: Max 5MB

---

### 5. 🔌 Third-Party Services

#### 📧 Email Notifications
- **Provider**: SendGrid or Mailgun  
- **Triggers**:
  - Booking Confirmations  
  - Booking Cancellations  
  - Payment Updates

#### 💳 Payment Processing
- **Provider**: Stripe or PayPal  
- **Flow**:
  - Guests make payment at booking
  - Hosts receive payout post-stay
- **Features**:
  - Tokenized card payments  
  - Multi-currency support

---

### 6. ⚠️ Error Handling & Logging

- **Error Format**:  
  ```json
  {
    "status": "error",
    "message": "Validation failed",
    "details": {
      "email": ["This field must be unique."]
    }
  }
