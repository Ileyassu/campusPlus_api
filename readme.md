# UM6P Campus API Documentation (Reverse Engineered)

This documentation covers the internal backend endpoints extracted from the `com.khalidbaba.wasl` Flutter client binary (`libapp.so`).

---

## 1. Overview & Network Constraints

* **Base URL:** `[https://campus-api.um6p.ma](https://campus-api.um6p.ma)`
* **Default Headers:**
* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer <TOKEN>` (required on all authenticated endpoints)


* **Network Scope:** Access requires either being connected to the internal UM6P campus network or routing through the campus VPN gateway. Requests from public/external IP ranges will drop or fail to resolve.

---

## 2. Authentication & Account Management

### 2.1 User Login

Authenticates user credentials and returns a Bearer access token.

* **Endpoint:** `/api/login`
* **Method:** `POST`
* **Auth Required:** No
* **Headers:**
```http
Content-Type: application/json
Accept: application/json

```


* **Request Body:**
```json
{
  "email": "user@um6p.ma",
  "password": "<PASSWORD>"
}

```


* **cURL Example:**
```bash
curl -s -X POST "https://campus-api.um6p.ma/api/login" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"email": "user@um6p.ma", "password": "your_password"}'

```



---

### 2.2 User Logout

Invalidates the current session token.

* **Endpoint:** `/api/logout`
* **Method:** `POST`
* **Auth Required:** Yes (`Bearer <TOKEN>`)

---

### 2.3 Other Account Endpoints

| Endpoint | Method | Description |
| --- | --- | --- |
| `/api/register` | `POST` | New user account registration |
| `/api/get_user` | `GET` | Fetches the current authenticated user's profile |
| `/api/profile/update` | `POST` | Updates profile metadata (name, contact, etc.) |
| `/api/photo` | `POST` | Profile picture upload |
| `/api/password/check` | `POST` | Validates password status |
| `/api/password/reset` | `POST` | Password reset trigger |
| `/api/sendcode` | `POST` | Sends 2FA / verification code to email or phone |
| `/api/verifycode` | `POST` | Validates submitted 2FA code |
| `/api/settings` | `GET` / `POST` | Application configuration and user settings |

---

## 3. Gym & Service Reservation Workflow

The reservation flow consists of fetching available time slots for a service, submitting a booking, and listing current reservations.

### 3.1 Fetch Available Time Slots

Retrieves available booking windows for campus prestations (gym, sports facilities, etc.).

* **Endpoint:** `/api/prestation/timeslotes/`
* **Method:** `GET`
* **Auth Required:** Yes (`Bearer <TOKEN>`)
* **Query Parameters / URI Structure:**
* Often accepts a prestation ID suffix or date query:
* `/api/prestation/timeslotes/{prestation_id}`
* `/api/prestation/timeslotes/?date=YYYY-MM-DD`




* **cURL Example:**
```bash
curl -s -X GET "https://campus-api.um6p.ma/api/prestation/timeslotes/" \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Accept: application/json"

```


* **Observed Response Schema Keys:**
* `reservation_time_id`: Unique integer identifier for the slot.
* `reservation_date`: Target date string (`YYYY-MM-DD`).
* `is_available`: Boolean status indicator.
* Capacity counters (`booked_count`, `capacity`).



---

### 3.2 Create / Reserve a Slot

Locks in an appointment for a specific slot.

* **Endpoint:** `/api/booking/put`
* **Method:** `POST`
* **Auth Required:** Yes (`Bearer <TOKEN>`)
* **Request Body:**
```json
{
  "reservation_time_id": 1234,
  "reservation_date": "2026-09-24"
}

```


* **cURL Example:**
```bash
curl -s -X POST "https://campus-api.um6p.ma/api/booking/put" \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "reservation_time_id": 1234,
    "reservation_date": "2026-09-24"
  }'

```



---

### 3.3 List Active & Past Appointments

Retrieves the user's booking history and upcoming tickets.

* **Endpoint:** `/api/appointment`
* **Method:** `GET`
* **Auth Required:** Yes (`Bearer <TOKEN>`)
* **cURL Example:**
```bash
curl -s -X GET "https://campus-api.um6p.ma/api/appointment" \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Accept: application/json"

```


* **Observed Response Schema Keys:**
* `booking_id`: Identifier for the confirmed reservation.
* `reservation_time`: Timestamp/slot detail.
* `reservation_date`: Date of the reservation.
* Verification tokens or QR payload strings for entrance validation.



---

### 3.4 Appointment Statistics

Summarizes usage metrics for sports and campus facilities.

* **Endpoint:** `/api/user/appointment_stats`
* **Method:** `GET`
* **Auth Required:** Yes (`Bearer <TOKEN>`)

---

## 4. Other Discovered Endpoints

| Endpoint | Method | Context / Purpose |
| --- | --- | --- |
| `/api/request` | `POST` | General campus helpdesk or administrative requests |
| `/api/assistant` | `POST` | Internal AI assistant route |
| `/api/chat` | `POST` | Chatbot messaging endpoint |
| `/api/embedding` | `POST` | Vector generation endpoint for campus search |
| `/api/getCareers` | `GET` | Campus job and internship postings |
| `/api/parsePdf` | `POST` | Document ingestion / CV parser |