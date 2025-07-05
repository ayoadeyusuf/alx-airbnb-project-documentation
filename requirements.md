# Backend Features Requirement Specifications

Below are detailed requirement specifications for three key backend features: **User Authentication**, **Property Management**, and **Booking System**. Each section includes API endpoints, input/output specifications, validation rules, and performance criteria.

---

## 1. User Authentication

### API Endpoints

| Method | Endpoint           | Description                 |
|--------|--------------------|----------------------------|
| POST   | /api/auth/register | Register user (guest/host) |
| POST   | /api/auth/login    | User login                 |
| GET    | /api/users/me      | Get current user profile   |
| PATCH  | /api/users/me      | Update user profile        |

### Input/Output Specifications

#### Register (`POST /api/auth/register`)
- **Input (JSON):**
  - `email`: string (required, valid email)
  - `password`: string (required, min 8 chars)
  - `role`: string (required, 'guest' or 'host')
  - `name`: string (required)
- **Output (JSON):**
  - `id`: string
  - `email`: string
  - `role`: string
  - `token`: string (JWT)

#### Login (`POST /api/auth/login`)
- **Input (JSON):**
  - `email`: string (required)
  - `password`: string (required)
- **Output (JSON):**
  - `id`: string
  - `email`: string
  - `role`: string
  - `token`: string (JWT)

#### Get/Update Profile (`GET`/`PATCH /api/users/me`)
- **Input (PATCH, JSON):**
  - `name`: string (optional)
  - `profile_photo`: string (URL, optional)
  - `contact_info`: string (optional)
- **Output (JSON):**
  - User profile object

### Validation Rules
- Email must be unique and valid format.
- Password must be at least 8 characters.
- Role must be 'guest' or 'host'.
- All endpoints require JWT authentication except register/login.

### Performance Criteria
- Registration and login should complete in <500ms under normal load.
- Profile fetch/update should complete in <300ms.

---

## 2. Property Management

### API Endpoints

| Method | Endpoint                 | Description                       |
|--------|--------------------------|-----------------------------------|
| POST   | /api/properties          | Create new property (host only)   |
| GET    | /api/properties          | List/search properties            |
| GET    | /api/properties/:id      | Get property details              |
| PATCH  | /api/properties/:id      | Update property (host only)       |
| DELETE | /api/properties/:id      | Delete property (host only)       |

### Input/Output Specifications

#### Create Property (`POST /api/properties`)
- **Input (JSON):**
  - `title`: string (required)
  - `description`: string (required)
  - `location`: string/object (required)
  - `price`: number (required, >0)
  - `amenities`: array of strings (optional)
  - `images`: array of strings (URLs, optional)
  - `availability`: array of date ranges (optional)
- **Output (JSON):**
  - Property object with unique `id`

#### List/Search Properties (`GET /api/properties`)
- **Input (Query Params):**
  - `location`, `price_min`, `price_max`, `guests`, `amenities`, `page`, `limit`
- **Output (JSON):**
  - Array of property objects (paginated)

#### Update/Delete Property
- **Input:** Property fields to update (PATCH), property ID (DELETE)
- **Output:** Updated property object or success message

### Validation Rules
- Only authenticated hosts can create, update, or delete their own properties.
- Title, description, location, and price are required.
- Images must be valid URLs.
- Price must be a positive number.

### Performance Criteria
- Search/list endpoint should return results in <700ms for up to 10,000 properties (with proper indexing).
- Create/update/delete should complete in <500ms.

---

## 3. Booking System

### API Endpoints

| Method | Endpoint                | Description                          |
|--------|-------------------------|--------------------------------------|
| POST   | /api/bookings           | Create new booking (guest only)      |
| GET    | /api/bookings           | List user bookings (guest/host)      |
| GET    | /api/bookings/:id       | Get booking details                  |
| PATCH  | /api/bookings/:id/cancel| Cancel booking (guest/host)          |

### Input/Output Specifications

#### Create Booking (`POST /api/bookings`)
- **Input (JSON):**
  - `property_id`: string (required)
  - `check_in`: date (required)
  - `check_out`: date (required)
  - `guests`: number (required)
- **Output (JSON):**
  - Booking object with unique `id`, `status` (pending/confirmed)

#### List Bookings (`GET /api/bookings`)
- **Input (Query Params):**
  - `user_id` (optional, for admin/host)
  - `status` (optional)
- **Output (JSON):**
  - Array of booking objects

#### Cancel Booking (`PATCH /api/bookings/:id/cancel`)
- **Input:** None
- **Output (JSON):**
  - Booking object with updated `status`

### Validation Rules
- Bookings cannot overlap existing confirmed bookings for the same property.
- Check-in date must be before check-out date.
- Guests must not exceed property’s max allowed.
- Only guests can create bookings; only booking owner or host can cancel.

### Performance Criteria
- Booking creation (including availability check) must complete in <1s.
- List and fetch booking details in <500ms.
- Concurrency handling to prevent double bookings.
