# Eventify 🎉

A full-featured **Event Management Web Application** built with Django. Eventify lets organisers create and manage events, handle attendee registration requests, generate QR codes for confirmed attendees, and scan those QR codes at the event entrance for attendance tracking.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Data Models](#data-models)
- [URL Reference](#url-reference)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Variables](#environment-variables)
  - [Running the Development Server](#running-the-development-server)
- [Key Implementation Details](#key-implementation-details)

---

## Features

| Category | Details |
|---|---|
| **Authentication** | Email-based login/signup using a custom user model; OTP-based forgot-password flow via email |
| **Event Management** | Create, edit, delete, and view upcoming & past events with image upload support |
| **Attendee Registration** | Public registration form (no login required); requests are queued for organiser review |
| **Request Approval** | Organiser can approve or reject each attendee request |
| **QR Code Generation** | On approval, a unique QR code is generated and emailed to the attendee |
| **QR Code Scanning** | Organiser uploads a QR image at the entrance; the app marks the attendee as attending |
| **Email Notifications** | Approval confirmation with QR attachment; bulk reminder emails to all confirmed attendees |
| **Profile Management** | Update name, phone number, profile picture, date of birth, and password |
| **Tailwind CSS UI** | Responsive, dark-themed UI styled with Tailwind CSS via `django-tailwind` |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python 3, Django 5.1.5 |
| Database | SQLite (default) |
| Styling | Tailwind CSS (`django-tailwind`) |
| QR Code | `qrcode` (generation), OpenCV + NumPy + Pillow (scanning) |
| Email | Django SMTP backend (Gmail) |
| Hot Reload | `django-browser-reload` |
| Config | `python-dotenv` |

---

## Project Structure

```
TeamEventify-main/
├── eventify/               # Django project settings & root URL config
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── event/                  # Core event app
│   ├── models.py           # Event, Attendee, AttendeeRequest
│   ├── views.py            # All event-related view functions
│   ├── forms.py            # EventForm with validation
│   ├── urls.py             # Event URL patterns
│   ├── templates/event/    # HTML templates for events
│   └── static/             # App-level static files
├── user1/                  # Custom user / authentication app
│   ├── models.py           # CustomUser (email-based auth)
│   ├── views.py            # Login, signup, profile, OTP reset
│   ├── forms.py            # CustomUserCreationForm, CustomUserUpdateForm
│   ├── urls.py             # Auth URL patterns
│   └── templates/user1/    # Login, signup, profile templates
├── theme/                  # Tailwind CSS theme app
│   └── static_src/         # Tailwind source files
├── templates/
│   └── layout.html         # Base HTML layout
├── static/
│   └── style.css           # Global custom styles
├── media/                  # User-uploaded files (event images, profile pics)
├── manage.py
└── db.sqlite3
```

---

## Data Models

### `CustomUser` (`user1` app)
Extends Django's `AbstractBaseUser` with **email** as the primary identifier.

| Field | Type | Notes |
|---|---|---|
| `email` | EmailField | Unique; used as `USERNAME_FIELD` |
| `name` | CharField | Required |
| `phone_number` | CharField | Optional |
| `profile_picture` | ImageField | Defaults to `default_profile.png` |
| `date_of_birth` | DateField | Optional |
| `is_active` / `is_staff` | BooleanField | Standard Django flags |

### `Event` (`event` app)

| Field | Type | Notes |
|---|---|---|
| `title` | CharField | |
| `description` | TextField | |
| `location` | CharField | |
| `date_time` | DateTimeField | Must be ≥ 5 hours from now (form-level validation) |
| `image` | ImageField | Optional; stored in `event_images/` |
| `max_attendees` | PositiveIntegerField | Default: 100 |
| `attendees` | ManyToManyField → `Attendee` | Confirmed attendees |
| `organizer` | ForeignKey → `CustomUser` | Set automatically on creation |

### `Attendee`

| Field | Type | Notes |
|---|---|---|
| `name` | CharField | |
| `email` | EmailField | Unique |
| `phone_no` | CharField | Optional |

### `AttendeeRequest`

| Field | Type | Notes |
|---|---|---|
| `event` | ForeignKey → `Event` | |
| `attendee` | ForeignKey → `Attendee` | |
| `requested_at` | DateTimeField | Auto-set on creation |
| `is_approved` | BooleanField | `False` = pending |
| `is_attending` | BooleanField | Set to `True` after QR scan |

---

## URL Reference

### Auth (`/user1/` prefix or `/`)

| URL | View | Description |
|---|---|---|
| `/` | `login_view` | Login page (default landing page) |
| `/user1/signup/` | `signup_view` | New user registration |
| `/user1/profile/update/` | `update_profile` | Edit profile & change password |
| `/user1/logout/` | `logout_view` | Logout and redirect to login |
| `/user1/forgot_password/` | `send_otp` | Enter email → receive OTP |
| `/user1/verify_otp/` | `verify_otp` | Enter OTP to verify identity |
| `/user1/reset_password/` | `reset_password` | Set new password after OTP verification |

### Events (`/event/` prefix)

| URL | View | Description |
|---|---|---|
| `/event/home/` | `home` | Landing / home page |
| `/event/` | `event_list` | List of upcoming events |
| `/event/events/past/` | `past_events` | List of past events |
| `/event/create_event/` | `create_event` | Create a new event |
| `/event/<id>/` | `event_details` | View event details |
| `/event/<id>/edit_event/` | `edit_event` | Edit an existing event |
| `/event/<id>/delete_event/` | `delete_event` | Delete an event |
| `/event/event/<id>/register/` | `register_user_page` | Public registration form |
| `/event/event/<id>/register/submit/` | `register_user` | Handle registration form POST |
| `/event/event/<id>/manage/` | `manage_attendee_requests` | Organiser: view pending requests |
| `/event/approve-attendee/<event_id>/<attendee_id>/` | `approve_attendee` | Approve request + email QR code |
| `/event/reject-attendee/<event_id>/<attendee_id>/` | `reject_attendee` | Reject an attendee request |
| `/event/event/<event_id>/remove-attendee/<attendee_id>/` | `remove_attendee` | Move attendee back to pending |
| `/event/event/<id>/send-email/` | `send_email_to_attendees` | Send reminder email to all confirmed attendees |
| `/event/profile/` | `profile` | View organiser profile & upcoming organised events |
| `/event/profile/past/` | `profile_past` | View past organised events |
| `/event/past-event/<id>/` | `past_event_details` | Details of a past event |
| `/event/scan_qr/` | `scan_qr_code` | Upload & scan a QR code to mark attendance |

---

## Getting Started

### Prerequisites

- Python 3.10+
- Node.js & npm (required by `django-tailwind` to compile CSS)
- A Gmail account (or any SMTP server) for email features

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/kashyap-1811/Eventify.git
cd Eventify/TeamEventify-main

# 2. Create and activate a virtual environment
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

# 3. Install Python dependencies
pip install django pillow qrcode opencv-python-headless numpy python-dotenv django-tailwind django-browser-reload

# 4. Install Tailwind CSS dependencies
python manage.py tailwind install

# 5. Apply database migrations
python manage.py migrate

# 6. Create a superuser (optional, for admin access)
python manage.py createsuperuser
```

### Environment Variables

Create a `.env` file inside `TeamEventify-main/` (same directory as `manage.py`):

```env
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-gmail-app-password
```

> **Note:** Use a [Gmail App Password](https://support.google.com/accounts/answer/185833) rather than your actual Gmail password.

### Running the Development Server

You need **two terminals** running simultaneously — one for Django and one for Tailwind's live CSS compilation:

**Terminal 1 – Tailwind watcher:**
```bash
cd TeamEventify-main
python manage.py tailwind start
```

**Terminal 2 – Django dev server:**
```bash
cd TeamEventify-main
python manage.py runserver
```

The app will be available at **http://127.0.0.1:8000/**.

---

## Key Implementation Details

### Email-Based Authentication
The project replaces Django's default username-based auth with a custom user model (`CustomUser`) that uses **email as the login identifier**. This is configured via `AUTH_USER_MODEL = 'user1.CustomUser'` in `settings.py`.

### OTP Password Reset
Forgot-password flow does **not** use Django's built-in password-reset emails. Instead it:
1. Generates a random 6-digit OTP and stores it in a server-side in-memory dictionary (`otp_storage` in `user1/views.py`).
2. Emails the OTP to the user via SMTP.
3. Verifies the OTP in a second step before allowing the password reset.

> **⚠️ Development note:** The in-memory `otp_storage` dict does not survive server restarts and won't work correctly with multiple server processes. For production, replace it with a database or cache-backed store (e.g. Django's cache framework with Redis).

### Attendee Registration Workflow
1. Any visitor can fill in the public registration form (`/event/event/<id>/register/`).
2. An `AttendeeRequest` record is created with `is_approved=False`.
3. The event organiser reviews requests at `/event/event/<id>/manage/`.
4. On approval, a unique QR code (encoding event title, attendee name and email) is generated with the `qrcode` library and emailed as a PNG attachment.
5. At the entrance, the organiser uploads the attendee's QR image via the scan page. OpenCV's `QRCodeDetector` decodes it and sets `is_attending=True` on the matching `AttendeeRequest`.

### Media Files
Uploaded event images are stored in `media/event_images/` and profile pictures in `media/profile_pics/`. Django serves these in development via the `MEDIA_URL`/`MEDIA_ROOT` settings plus the `static()` helper in `urls.py`.

### Tailwind CSS Integration
UI styling is handled by `django-tailwind` with a dedicated `theme` app. The Tailwind source lives in `theme/static_src/`. Run `python manage.py tailwind start` during development to watch for changes and recompile CSS automatically.
