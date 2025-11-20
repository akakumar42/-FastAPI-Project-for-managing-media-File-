# Media File Management API

A modern, async-first FastAPI application for managing and sharing media files (images and videos). Users can upload, view, and manage their media content with a secure authentication system powered by JWT tokens.

## Features

### 🔐 User Authentication & Management
- **User Registration & Login**: Create accounts and authenticate using JWT-based bearer tokens
- **Email Verification**: Optional email verification system for registered users
- **Password Reset**: Secure password recovery mechanism
- **User Profiles**: Update user information and manage your account
- **Session Management**: Automatic token expiration with configurable lifetime (default: 1 hour)

### 📤 Media Upload
- **Multi-format Support**: Upload both images and videos
- **Cloud Integration**: Seamlessly integrate with ImageKit for reliable media storage
- **Unique File Names**: Automatic unique naming to prevent file conflicts
- **File Metadata**: Capture captions and file information with each upload
- **Temporary File Handling**: Efficient temporary file management during upload

### 📸 Media Feed
- **Chronological Feed**: View all uploaded media sorted by most recent first
- **Ownership Tracking**: Easily identify which posts you own
- **File Type Detection**: Automatically categorize uploads as images or videos
- **Rich Metadata**: Access file URLs, names, and timestamps for each post

### 🗑️ Content Management
- **Delete Posts**: Remove your own media from the platform
- **Ownership Protection**: Only post owners can delete their content
- **Permanent Deletion**: Clean removal of posts from the database

## Tech Stack

- **Framework**: FastAPI (modern async Python web framework)
- **Database**: SQLite with async support via SQLAlchemy
- **Authentication**: FastAPI-Users with JWT tokens
- **Media Storage**: ImageKit (cloud-based media management)
- **Server**: Uvicorn (ASGI server)
- **ORM**: SQLAlchemy with async sessions

## Project Structure

```
.
├── main.py                 # Application entry point
├── pyproject.toml         # Project configuration and dependencies
├── README.md              # This file
└── app/
    ├── app.py             # FastAPI application and route handlers
    ├── db.py              # Database models and configuration
    ├── schemas.py         # Pydantic models for request/response validation
    ├── users.py           # User authentication and management
    └── images.py          # ImageKit integration setup
```

## Installation

### Prerequisites
- Python 3.13 or higher
- pip (Python package manager)

### Setup

1. **Clone the repository**
```bash
git clone <repository-url>
cd -FastAPI-Project-for-managing-media-File-
```

2. **Create a virtual environment**
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. **Install dependencies**
```bash
pip install -e .
```

4. **Configure environment variables**

Create a `.env` file in the project root:
```
IMAGEKIT_PRIVATE_KEY=your_imagekit_private_key
IMAGEKIT_PUBLIC_KEY=your_imagekit_public_key
IMAGEKIT_URL=your_imagekit_url_endpoint
```

Get your ImageKit credentials from the [ImageKit Dashboard](https://imagekit.io/).

## Running the Application

Start the development server:

```bash
python main.py
```

The API will be available at `http://localhost:8000`

Interactive API documentation is available at:
- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`

## API Endpoints

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/register` | Create a new user account |
| POST | `/auth/jwt/login` | Login with email and password |
| POST | `/auth/jwt/logout` | Logout (invalidate token) |
| POST | `/auth/request-verify-token` | Request email verification |
| POST | `/auth/verify` | Verify email address |
| POST | `/auth/forgot-password` | Request password reset |
| POST | `/auth/reset-password` | Reset password with token |

### User Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/users/me` | Get current user profile |
| PATCH | `/users/{user_id}` | Update user information |
| GET | `/users/{user_id}` | Get specific user details |

### Media Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/upload` | Upload an image or video |
| GET | `/feed` | Get all media posts (chronological) |
| DELETE | `/post/{post_id}` | Delete a specific post |

## API Usage Examples

### Register a New User

```bash
curl -X POST "http://localhost:8000/auth/register" \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "securepassword"}'
```

### Login

```bash
curl -X POST "http://localhost:8000/auth/jwt/login" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=user@example.com&password=securepassword"
```

### Upload Media

```bash
curl -X POST "http://localhost:8000/upload" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -F "file=@/path/to/image.jpg" \
  -F "caption=My awesome photo"
```

### Get Feed

```bash
curl -X GET "http://localhost:8000/feed" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Delete a Post

```bash
curl -X DELETE "http://localhost:8000/post/post-id-here" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Database Models

### User
- Extends FastAPI-Users base model with UUID primary key
- Fields: id, email, hashed_password, is_active, is_superuser, is_verified
- Relationships: One-to-many with Posts

### Post
- Stores media metadata and references
- Fields:
  - `id`: Unique post identifier (UUID)
  - `user_id`: Foreign key to user
  - `caption`: Optional text description
  - `url`: Cloud storage URL (ImageKit)
  - `file_type`: "image" or "video"
  - `file_name`: Original uploaded filename
  - `created_at`: Timestamp of upload
- Relationships: Many-to-one with User

## Key Implementation Details

### Secure File Handling
- Files are temporarily stored during upload to ImageKit
- Temporary files are automatically cleaned up after upload completes
- Unique filenames prevent conflicts on the storage service

### Async Operations
- All database queries use async/await for non-blocking I/O
- Concurrent request handling for improved performance
- Async session management with proper cleanup

### Authentication Flow
- JWT tokens are issued upon successful login
- Bearer token required for protected endpoints
- Tokens expire after 1 hour (configurable)
- Only active, verified users can access protected resources

### File Type Detection
- Automatically detects if uploaded content is video or image
- Based on MIME type inspection
- Supports multiple formats (JPEG, PNG, MP4, WebM, etc.)

## Development Notes

- **Database**: Currently uses SQLite for simplicity; easily scalable to PostgreSQL
- **Secret Key**: Change the `SECRET` in `app/users.py` for production
- **CORS**: Consider adding CORS middleware for frontend integration
- **Environment**: Uses `python-dotenv` for environment variable management

## Dependencies

- **fastapi** (>=0.121.2): Web framework
- **uvicorn[standard]** (>=0.38.0): ASGI server
- **fastapi-users[sqlalchemy]** (>=15.0.1): User management system
- **sqlalchemy** + **aiosqlite** (>=0.21.0): ORM and async database support
- **imagekitio** (>=4.2.0): ImageKit SDK for media management
- **python-dotenv** (>=1.2.1): Environment configuration

## Future Enhancements

- Add image resizing and optimization
- Implement tagging and filtering system
- Add like/comment functionality
- Implement user follow system
- Add media search capabilities
- Rate limiting for uploads
- File size and type validation

## Troubleshooting

### 500 Error on Upload
- Verify ImageKit credentials in `.env`
- Check file permissions for temporary directory
- Ensure file isn't corrupted

### 401 Unauthorized
- Token may have expired, re-login required
- Verify bearer token format: `Authorization: Bearer <token>`

### 404 Post Not Found
- Post may have been deleted
- Verify post ID is correct

## Getting Help

For issues or questions, please check the code comments and API documentation at `/docs` endpoint.

---

