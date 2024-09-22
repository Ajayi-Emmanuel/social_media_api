Here’s a sample `README.md` file that details the setup process, user registration, authentication, and provides a brief overview of the custom user model.

---

# Social Media API

This is a simple Social Media API built with Django and Django REST Framework. It features user registration, login, and token-based authentication, along with a profile endpoint to manage user profiles.

## Features
- Custom user model
- User registration and login
- Token-based authentication
- Profile management

## Project Setup

### 1. Clone the Repository

First, clone the project repository to your local machine.

```bash
git clone https://github.com/yourusername/social_media_api.git
cd social_media_api
```

### 2. Create a Virtual Environment

Create and activate a virtual environment to manage dependencies.

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate
```

### 3. Install Dependencies

Install the necessary dependencies, including Django, Django REST Framework, and other required libraries.

```bash
pip install -r requirements.txt
```

If you haven't generated a `requirements.txt` file yet, you can install the following:

```bash
pip install django djangorestframework djangorestframework.authtoken
```

### 4. Apply Migrations

Create the necessary database tables by applying migrations.

```bash
python manage.py migrate
```

### 5. Create a Superuser (Optional)

Create a superuser for accessing the Django admin interface.

```bash
python manage.py createsuperuser
```

### 6. Run the Development Server

Start the Django development server.

```bash
python manage.py runserver
```

The API should now be running locally at `http://127.0.0.1:8000/`.

---

## User Registration and Authentication

### 1. User Registration

To register a new user, make a `POST` request to the `/api/register/` endpoint.

**URL:** `/api/register/`  
**Method:** `POST`  
**Request Body:**

```json
{
  "username": "your_username",
  "email": "your_email@example.com",
  "password": "your_password"
}
```

**Response:**

```json
{
  "id": 1,
  "username": "your_username",
  "email": "your_email@example.com",
  "token": "generated_token_here"
}
```

This will create a new user and return an authentication token for the user.

### 2. User Login

To log in and retrieve a token, make a `POST` request to the `/api/login/` endpoint.

**URL:** `/api/login/`  
**Method:** `POST`  
**Request Body:**

```json
{
  "username": "your_username",
  "password": "your_password"
}
```

**Response:**

```json
{
  "token": "generated_token_here"
}
```

### 3. Accessing the User Profile

To retrieve the profile information of the authenticated user, send a `GET` request to the `/api/profile/` endpoint, including the user's token in the request header.

**URL:** `/api/profile/`  
**Method:** `GET`  
**Headers:**

```http
Authorization: Token your_token_here
```

**Response:**

```json
{
  "id": 1,
  "username": "your_username",
  "email": "your_email@example.com",
  "bio": "User bio here",
  "profile_picture": "http://example.com/media/profile_picture.jpg",
  "followers": []
}
```

---

## Custom User Model

This project uses a custom user model that extends Django’s default `AbstractUser`. In addition to the default fields, the custom user model includes:

- **bio**: A short description of the user.
- **profile_picture**: An optional profile picture.
- **followers**: A ManyToMany field representing other users who follow this user.

Here is a brief overview of the custom user model:

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    bio = models.TextField(blank=True, null=True)
    profile_picture = models.ImageField(upload_to='profile_pictures/', blank=True, null=True)
    followers = models.ManyToManyField('self', symmetrical=False, related_name='following')
```

---

## Endpoints Overview

### Registration

- **URL**: `/api/register/`
- **Method**: `POST`
- **Description**: Registers a new user and returns an authentication token.

### Login

- **URL**: `/api/login/`
- **Method**: `POST`
- **Description**: Authenticates the user and returns a token.

### Profile

- **URL**: `/api/profile/`
- **Method**: `GET`
- **Description**: Returns the profile information of the authenticated user.

---

## Testing with Postman

To test the API with Postman, follow these steps:

1. **Register a User:**
   - Make a `POST` request to `http://127.0.0.1:8000/api/register/` with the registration payload.
   
2. **Login a User:**
   - Make a `POST` request to `http://127.0.0.1:8000/api/login/` with the login credentials.

3. **Access User Profile:**
   - Make a `GET` request to `http://127.0.0.1:8000/api/profile/` with the `Authorization: Token <token>` header.

---

Here’s an implementation guide for adding posts and comments functionality to your **Social Media API**, following the task description you've provided.

---

### Step 1: Create Post and Comment Models

Start by creating a new app for handling posts and comments.

```bash
python manage.py startapp posts
```

#### Post Model:
In the `posts/models.py` file, define the `Post` model with the following fields:
- `author`: A foreign key to the `User` model
- `title`: The title of the post
- `content`: The content of the post
- `created_at` and `updated_at`: Timestamps

```python
from django.db import models
from django.conf import settings

class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='posts')
    title = models.CharField(max_length=255)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
```

#### Comment Model:
In the `posts/models.py` file, define the `Comment` model, which references both the `Post` and the `User`.

```python
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='comments')
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return f'Comment by {self.author} on {self.post}'
```

#### Migrate Models:
To apply the changes to the database, run the following commands:

```bash
python manage.py makemigrations posts
python manage.py migrate
```

---

### Step 2: Implement Serializers for Posts and Comments

In `posts/serializers.py`, create serializers to handle data validation and transformation for `Post` and `Comment`.

```python
from rest_framework import serializers
from .models import Post, Comment

class PostSerializer(serializers.ModelSerializer):
    author = serializers.ReadOnlyField(source='author.username')

    class Meta:
        model = Post
        fields = ['id', 'author', 'title', 'content', 'created_at', 'updated_at']

class CommentSerializer(serializers.ModelSerializer):
    author = serializers.ReadOnlyField(source='author.username')

    class Meta:
        model = Comment
        fields = ['id', 'post', 'author', 'content', 'created_at', 'updated_at']
```

---

### Step 3: Create Views for CRUD Operations

In `posts/views.py`, create views for managing `Post` and `Comment` using Django REST Framework’s `ModelViewSet`.

```python
from rest_framework import viewsets, permissions
from .models import Post, Comment
from .serializers import PostSerializer, CommentSerializer

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

class CommentViewSet(viewsets.ModelViewSet):
    queryset = Comment.objects.all()
    serializer_class = CommentSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
```

---

### Step 4: Configure URL Routing

In `posts/urls.py`, set up URL routing for `Post` and `Comment` using DRF’s routers.

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import PostViewSet, CommentViewSet

router = DefaultRouter()
router.register(r'posts', PostViewSet)
router.register(r'comments', CommentViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

In your main project `urls.py`, include the `posts` app URLs.

```python
from django.urls import path, include

urlpatterns = [
    path('api/', include('posts.urls')),
    # other URLs
]
```

---

### Step 5: Implement Pagination and Filtering

To manage large datasets, implement pagination in the `settings.py` file.

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}
```

For filtering posts by title or content, install `django-filter` and add it to your project.

```bash
pip install django-filter
```

In `views.py`, add the filtering capability:

```python
from rest_framework import filters

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    filter_backends = [filters.SearchFilter]
    search_fields = ['title', 'content']

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
```

---

### Step 6: Test and Validate Functionality

Use Postman or Django's built-in testing tools to test all the CRUD operations for posts and comments. Here are the endpoints:

- **Posts**: 
  - `GET /api/posts/` – List all posts (paginated)
  - `POST /api/posts/` – Create a new post (authenticated users only)
  - `GET /api/posts/{id}/` – Retrieve a single post by ID
  - `PUT /api/posts/{id}/` – Update a post (only the author)
  - `DELETE /api/posts/{id}/` – Delete a post (only the author)

- **Comments**: 
  - `GET /api/comments/` – List all comments (paginated)
  - `POST /api/comments/` – Create a new comment (authenticated users only)
  - `GET /api/comments/{id}/` – Retrieve a single comment by ID
  - `PUT /api/comments/{id}/` – Update a comment (only the author)
  - `DELETE /api/comments/{id}/` – Delete a comment (only the author)

---

### Step 7: Document API Endpoints

Provide API documentation with examples of how to interact with the posts and comments endpoints.

#### Example: Creating a Post

**Endpoint**: `POST /api/posts/`  
**Request**:

```json
{
  "title": "My First Post",
  "content": "This is the content of the first post."
}
```

**Response**:

```json
{
  "id": 1,
  "author": "user1",
  "title": "My First Post",
  "content": "This is the content of the first post.",
  "created_at": "2024-09-22T18:00:00Z",
  "updated_at": "2024-09-22T18:00:00Z"
}
```

#### Example: Creating a Comment

**Endpoint**: `POST /api/comments/`  
**Request**:

```json
{
  "post": 1,
  "content": "This is a comment on the first post."
}
```

**Response**:

```json
{
  "id": 1,
  "post": 1,
  "author": "user1",
  "content": "This is a comment on the first post.",
  "created_at": "2024-09-22T18:10:00Z",
  "updated_at": "2024-09-22T18:10:00Z"
}
```

---

## Deliverables

1. **Code Files**: Include models, serializers, views, and URLs for `Post` and `Comment` models.
2. **API Documentation**: Document all the endpoints with sample request/response payloads.
3. **Testing**: Provide evidence of testing the API using Postman or automated tests.

---

This completes the post and comment functionality for the Social Media API!

Here's a detailed API documentation for the follow/unfollow and feed features of your Social Media API, including examples of requests and responses.

---

# API Documentation

## User Follow/Unfollow and Feed Features

### 1. Follow a User

- **Endpoint**: `POST /api/follow/<int:user_id>/`
- **Description**: Allows the authenticated user to follow another user by their user ID.

#### Request

- **Headers**:
  - `Authorization: Token <your_token>`

- **URL Parameters**:
  - `user_id`: The ID of the user to follow.

- **Example**:
```http
POST /api/follow/2/
Authorization: Token abc123def456
```

#### Response

- **Success (200 OK)**:
```json
{
    "success": "You are now following user_name"
}
```

- **Error (400 Bad Request)**:
```json
{
    "error": "You cannot follow yourself."
}
```

### 2. Unfollow a User

- **Endpoint**: `POST /api/unfollow/<int:user_id>/`
- **Description**: Allows the authenticated user to unfollow a user by their user ID.

#### Request

- **Headers**:
  - `Authorization: Token <your_token>`

- **URL Parameters**:
  - `user_id`: The ID of the user to unfollow.

- **Example**:
```http
POST /api/unfollow/2/
Authorization: Token abc123def456
```

#### Response

- **Success (200 OK)**:
```json
{
    "success": "You have unfollowed user_name"
}
```

- **Error (400 Bad Request)**:
```json
{
    "error": "You cannot unfollow yourself."
}
```

### 3. Get User Feed

- **Endpoint**: `GET /api/feed/`
- **Description**: Returns a list of posts from users that the authenticated user follows, ordered by creation date (most recent first).

#### Request

- **Headers**:
  - `Authorization: Token <your_token>`

- **Example**:
```http
GET /api/feed/
Authorization: Token abc123def456
```

#### Response

- **Success (200 OK)**:
```json
[
    {
        "id": 1,
        "author": "author_username",
        "title": "Post Title 1",
        "content": "Content of the first post.",
        "created_at": "2024-09-22T18:00:00Z",
        "updated_at": "2024-09-22T19:00:00Z"
    },
    {
        "id": 2,
        "author": "author_username",
        "title": "Post Title 2",
        "content": "Content of the second post.",
        "created_at": "2024-09-21T15:30:00Z",
        "updated_at": "2024-09-21T16:00:00Z"
    }
]
```

- **Error (401 Unauthorized)**:
```json
{
    "detail": "Authentication credentials were not provided."
}
```

---

## Notes

- **Authentication**: All endpoints require token-based authentication. Ensure you include the `Authorization` header in your requests.
- **User Feedback**: The API returns user-friendly messages for successful actions as well as informative error messages for unsuccessful attempts.

---

This documentation provides a clear overview of the new functionalities related to user follows and the feed feature, complete with examples for practical usage.

Here's the documentation for the likes and notifications systems in your Social Media API, including detailed examples of requests, expected responses, and an explanation of how users can interact with these features.

---

# Likes and Notifications API Documentation

## Overview

The Likes and Notifications features enhance user engagement within the Social Media API by allowing users to like posts and receive notifications about interactions related to their posts and followers. This creates a more interactive experience, encouraging users to engage with content and each other.

## Endpoints

### Likes

#### 1. Like a Post

- **URL:** `/posts/<int:pk>/like/`
- **Method:** `POST`
- **Description:** Allows a user to like a specific post.

**Request Example:**

```http
POST /posts/1/like/
Authorization: Token <your_token>
```

**Expected Response:**

```json
{
    "message": "Post liked successfully."
}
```

#### 2. Unlike a Post

- **URL:** `/posts/<int:pk>/unlike/`
- **Method:** `DELETE`
- **Description:** Allows a user to unlike a specific post.

**Request Example:**

```http
DELETE /posts/1/unlike/
Authorization: Token <your_token>
```

**Expected Response:**

```json
{
    "message": "Post unliked successfully."
}
```

### Notifications

#### 3. Fetch Notifications

- **URL:** `/notifications/`
- **Method:** `GET`
- **Description:** Retrieves a list of notifications for the authenticated user.

**Request Example:**

```http
GET /notifications/
Authorization: Token <your_token>
```

**Expected Response:**

```json
[
    {
        "actor": "User1",
        "verb": "liked your post.",
        "timestamp": "2024-09-22T15:30:00Z",
        "read": false
    },
    {
        "actor": "User2",
        "verb": "started following you.",
        "timestamp": "2024-09-22T15:00:00Z",
        "read": true
    }
]
```

### Interaction with Features

1. **Liking Posts:**
   - Users can like posts to express their appreciation or agreement with the content.
   - The API ensures that a user cannot like the same post multiple times, providing immediate feedback on their action.

2. **Unliking Posts:**
   - Users can also remove their like from a post, allowing them to change their minds about content.
   - The response confirms the action, making it clear that the like has been successfully removed.

3. **Notifications:**
   - Notifications keep users informed about interactions with their content, such as likes on their posts or new followers.
   - Users can fetch their notifications to see recent activities, enhancing their awareness of engagement on the platform.

### Benefits of Features

- **Enhanced User Engagement:** By allowing likes and notifications, users are encouraged to interact more with the content and with each other, increasing overall platform activity.
- **Real-time Feedback:** Users receive immediate notifications about interactions, fostering a sense of community and responsiveness.
- **Increased Visibility:** Likes and comments can lead to increased visibility for posts, helping users discover more content.
- **User Retention:** Engaging features like these can lead to higher user retention rates as users are more likely to return to see new interactions and updates.

---

This documentation should help users understand how to use the likes and notifications functionalities in your Social Media API effectively. Make sure to test these endpoints thoroughly to ensure they work as intended!

## Project Structure

```bash
social_media_api/
│
├── accounts/
│   ├── migrations/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   └── urls.py
│
├── social_media_api/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
├── manage.py
└── README.md
```

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## Author

- **Emmanuel Ajayi**