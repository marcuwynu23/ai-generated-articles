# 🚀 How to Create a Django Application Using Poetry

> Modern Python development encourages using tools that handle dependency management and environment isolation effectively. [Poetry](https://python-poetry.org/) is one of the most popular solutions today. In this guide, you’ll learn how to create a Django application from scratch using Poetry.

---

## 📦 What Is Poetry?

[Poetry](https://python-poetry.org/) is a modern dependency and packaging manager for Python. It simplifies managing virtual environments, installing dependencies, and building your Python projects.

---

## 🛠 Prerequisites

- Python 3.7 or higher  
- Poetry installed  
- Basic command line knowledge

---

## 📍 Step 1: Install Poetry

If you haven’t installed Poetry yet, run:

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

After installation, restart your terminal and check:

```bash
poetry --version
```

---

## 🧱 Step 2: Create a New Poetry Project

Create your project folder:

```bash
poetry new mysite
cd mysite
```

This creates a new directory `mysite/` with the following structure:

```
mysite/
├── mysite/
│   └── __init__.py
├── tests/
│   └── __init__.py
├── pyproject.toml
```

---

## 📥 Step 3: Add Django as a Dependency

Tell Poetry to install Django:

```bash
poetry add django
```

Poetry updates your `pyproject.toml` file and installs Django inside a virtual environment.

---

## 🔁 Step 4: Activate the Virtual Environment

To activate Poetry’s virtual environment, run:

```bash
poetry shell
```

You are now inside an isolated environment with Django available.

---

## ⚙️ Step 5: Create Your Django Project

From inside the Poetry shell, create a Django project:

```bash
django-admin startproject config .
```

> The `.` tells Django to use the current directory rather than creating a subfolder.

You now have:

```
mysite/
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── manage.py
├── pyproject.toml
```

---

## ▶️ Step 6: Run the Development Server

Run the initial migration and start the server:

```bash
python manage.py migrate
python manage.py runserver
```

Now visit [http://127.0.0.1:8000](http://127.0.0.1:8000) in your browser—you’re up and running!

---

## 🧪 Step 7 (Optional): Add Development Tools

Want code formatting and linting? Add dev dependencies like so:

```bash
poetry add --group dev black flake8 isort
```

These tools help enforce clean code standards.

---

## ✅ Summary of Commands

```bash
curl -sSL https://install.python-poetry.org | python3 -
poetry new mysite
cd mysite
poetry add django
poetry shell
django-admin startproject config .
python manage.py migrate
python manage.py runserver
```

---

## 📌 Final Tips

- Use `poetry install` to install all dependencies from `pyproject.toml` on a new machine.
- Use `poetry add package_name` to add new dependencies.
- Manage Python version with `poetry env use python3.x` if needed.

---

By combining the power of Django with the modern tooling of Poetry, you get a cleaner, more maintainable development experience. This setup is ideal for solo projects, startups, and even enterprise applications.
