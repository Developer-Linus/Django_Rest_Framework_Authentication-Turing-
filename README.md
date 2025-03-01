

````markdown
# Django REST Framework Authentication Guide

This guide covers various authentication methods available in Django REST Framework (DRF) to secure your APIs. Each method is explained with examples and best practices.

---

## 1. Basic Authentication

Basic Authentication uses standard HTTP Basic Authentication. The username and password are passed in the `Authorization` header when making requests.

### Example:

```http
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```
````

### Configuration:

Add `BasicAuthentication` to the `DEFAULT_AUTHENTICATION_CLASSES` list in your DRF settings:

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.BasicAuthentication',
        # Other authentication classes...
    ]
}
```

### Use Case:

- Suitable for quick prototyping and testing.
- **Avoid using in production** as credentials are passed in plain text.

---

## 2. Token Authentication

Token Authentication uses an arbitrary token instead of a username and password. Tokens are typically randomly generated strings stored with the user profile.

### Setup:

1. Install the `django-rest-framework-simplejwt` package:

   ```bash
   pip install djangorestframework-simplejwt
   ```

2. Add it to your installed apps:

   ```python
   # settings.py
   INSTALLED_APPS = [
       # ...
       'rest_framework_simplejwt',
   ]
   ```

3. Include `TokenAuthentication` in your DRF settings:

   ```python
   # settings.py
   REST_FRAMEWORK = {
       'DEFAULT_AUTHENTICATION_CLASSES': [
           'rest_framework.authentication.TokenAuthentication',
           # Other authentication classes...
       ]
   }
   ```

4. Add JWT views to your `urls.py`:

   ```python
   # urls.py
   from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

   urlpatterns = [
       # ...
       path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
       path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
   ]
   ```

### Usage:

- Users obtain a token by posting their username and password to the `token_obtain_pair` endpoint.
- The response includes an **access token** and a **refresh token**.
- Use the refresh token to get new access tokens when they expire.

### Example Header:

```http
Authorization: Bearer <token>
```

### Use Case:

- Ideal for production APIs.
- Avoids sending credentials with each request.
- Tokens can be revoked individually on the server side.

---

## 3. Session Authentication

Session Authentication leverages Django's session framework. Users are logged in, and session data is stored server-side.

### Configuration:

1. Enable sessions in Django:

   ```python
   # settings.py
   INSTALLED_APPS = [
       # ...
       'django.contrib.sessions',
   ]

   MIDDLEWARE = [
       # ...
       'django.contrib.sessions.middleware.SessionMiddleware',
       # ...
   ]
   ```

2. Add `SessionAuthentication` to your DRF settings:

   ```python
   # settings.py
   REST_FRAMEWORK = {
       'DEFAULT_AUTHENTICATION_CLASSES': [
           'rest_framework.authentication.SessionAuthentication',
           # Other authentication classes...
       ]
   }
   ```

3. Add login and logout views:

   ```python
   # views.py
   from rest_framework.response import Response
   from rest_framework.views import APIView

   class LoginView(APIView):
       def post(self, request):
           username = request.data.get('username')
           password = request.data.get('password')
           # Validate credentials and log in user
           return Response("User logged in")

   class LogoutView(APIView):
       def post(self, request):
           # Log out user
           return Response("User logged out")
   ```

### Use Case:

- Suitable for web APIs that need to maintain user state.
- Requires server-side session storage, which may not scale as well as token authentication.

---

## 4. JSON Web Token (JWT) Authentication

JWT is an open standard for securely transmitting information between parties. DRF provides a package for JWT authentication.

### Setup:

1. Install the `djangorestframework-jwt` package:

   ```bash
   pip install djangorestframework-jwt
   ```

2. Add JWT authentication to your DRF settings:

   ```python
   # settings.py
   REST_FRAMEWORK = {
       'DEFAULT_AUTHENTICATION_CLASSES': [
           'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
           # Other authentication classes...
       ]
   }
   ```

3. Add JWT views to your `urls.py`:

   ```python
   # urls.py
   from rest_framework_jwt.views import obtain_jwt_token, refresh_jwt_token

   urlpatterns = [
       # ...
       path('api/token/', obtain_jwt_token),
       path('api/token/refresh/', refresh_jwt_token),
   ]
   ```

### Usage:

- Users obtain a JWT by posting their username and password to the `obtain_jwt_token` endpoint.
- Use the JWT in the `Authorization` header:
  ```http
  Authorization: JWT <token>
  ```
- Refresh tokens can be used to obtain new access tokens when they expire.

### Use Case:

- Stateless authentication mechanism for APIs.
- Contains all user information encrypted in the token, eliminating the need for database lookups.

---

## 5. OAuth 2.0 Authentication

OAuth 2.0 is an authorization standard that allows users to grant third-party access to their data. DRF integrates with OAuth 2.0 for robust authentication.

### Setup:

1. Install the `django-rest-framework-social-oauth2` package:

   ```bash
   pip install djangorestframework-social-oauth2
   ```

2. Add it to your installed apps:

   ```python
   # settings.py
   INSTALLED_APPS = [
       # ...
       'rest_framework_social_oauth2',
   ]
   ```

3. Include OAuth2 authentication in your DRF settings:

   ```python
   # settings.py
   REST_FRAMEWORK = {
       'DEFAULT_AUTHENTICATION_CLASSES': [
           'rest_framework_social_oauth2.authentication.OAuth2Authentication',
       ]
   }
   ```

4. Configure OAuth providers (e.g., Google, Facebook):

   ```python
   # settings.py
   SOCIAL_AUTH_GOOGLE_OAUTH2_KEY = 'your-client-id'
   SOCIAL_AUTH_GOOGLE_OAUTH2_SECRET = 'your-client-secret'
   ```

5. Add OAuth login views to your `urls.py`:

   ```python
   # urls.py
   from rest_framework_social_oauth2.views import OAuth2LoginView

   urlpatterns = [
       # ...
       path('api/auth/login/google-oauth2/', OAuth2LoginView.as_view(provider='google-oauth2')),
   ]
   ```

### Use Case:

- Ideal for securely delegating authentication to trusted external providers like Google and Facebook.
- Prevents the need to store usernames and passwords yourself.

---

## 6. Custom Authentication

If the built-in authentication methods don't meet your needs, you can create custom authentication classes by subclassing `BaseAuthentication`.

### Example:

```python
# custom_auth.py
from rest_framework.authentication import BaseAuthentication

class CustomAuth(BaseAuthentication):
    def authenticate(self, request):
        # Check credentials and return (user, token)
        return None

    def authenticate_header(self, request):
        # Return custom auth header, e.g.
        return 'Custom realm="API"'
```

### Configuration:

Add your custom authentication class to the DRF settings:

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'path.to.CustomAuth',
        # Other authentication classes...
    ]
}
```

### Use Case:

- Useful for integrating with external authentication systems or implementing custom validation logic.

---

## Best Practices for API Authentication

1. **Use HTTPS** to encrypt all authentication credentials and traffic.
2. **Short-lived access tokens** with rotation for enhanced security.
3. **Restrict permissions** based on authentication to limit data exposure.
4. **Adopt stateless authentication** (e.g., JWTs) for scalability.
5. **Offload authentication** to trusted external providers (e.g., OAuth) when possible.
6. **Log and monitor authentication failures** to detect attacks.
7. **Practice defense in depth** with multiple authentication layers if needed.

---

## Conclusion

Django REST Framework provides a variety of authentication methods to secure your APIs. Whether you're using Basic Authentication, Token Authentication, Session Authentication, JWT, OAuth 2.0, or custom authentication, it's crucial to choose the right method based on your project's requirements.

By following best practices and leveraging DRF's built-in tools, you can build secure and reliable APIs that protect sensitive data and resources.

---

This guide is a starting point for implementing API authentication in Django REST Framework. For more details, refer to the [official DRF documentation](https://www.django-rest-framework.org/).

```

```
