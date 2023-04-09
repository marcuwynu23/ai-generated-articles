# Using Authentication in Django

Django is a popular Python web framework that provides built-in support for authentication. In this article, we will discuss how to use authentication in Django and protect your web application from unauthorized access.

### Creating User Models
The first step in using authentication in Django is to create user models. Django provides a built-in User model, but you can also create your own custom user model by inheriting from the AbstractBaseUser and BaseUserManager classes.

```python
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager

class CustomUserManager(BaseUserManager):
    def create_user(self, email, password=None):
        if not email:
            raise ValueError('Users must have an email address')

        user = self.model(
            email=self.normalize_email(email),
        )

        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password):
        user = self.create_user(
            email=email,
            password=password,
        )
        user.is_admin = True
        user.save(using=self._db)
        return user


class CustomUser(AbstractBaseUser):
    email = models.EmailField(
        verbose_name='email address',
        max_length=255,
        unique=True,
    )
    is_active = models.BooleanField(default=True)
    is_admin = models.BooleanField(default=False)

    objects = CustomUserManager()

    USERNAME_FIELD = 'email'

    def __str__(self):
        return self.email

    def has_perm(self, perm, obj=None):
        return True

    def has_module_perms(self, app_label):
        return True

    @property
    def is_staff(self):
        return self.is_admin
```
### Authenticating Users
Once you have created user models, you can authenticate users in your views. Django provides several authentication backends, including the ModelBackend, which authenticates users against your database.

```python
from django.contrib.auth import authenticate, login, logout

def my_view(request):
    if request.method == 'POST':
        email = request.POST['email']
        password = request.POST['password']
        user = authenticate(request, email=email, password=password)
        if user is not None:
            login(request, user)
            return redirect('my_dashboard')
        else:
            return HttpResponse('Invalid login credentials')
    else:
        return render(request, 'login.html')
```
###  Protecting Views
You can protect views from unauthorized access using the @login_required decorator, which will redirect users to the login page if they are not authenticated.

```python
from django.contrib.auth.decorators import login_required

@login_required
def my_dashboard(request):
    return render(request, 'dashboard.html')
```
### Conclusion
In this article, we have discussed how to use authentication in Django to protect your web application from unauthorized access. We have created user models, authenticated users in views, and protected views using the @login_required decorator. With these tools, you can create secure web applications with Django.