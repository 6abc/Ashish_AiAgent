# GitHub Copilot Agent Instructions

You are an expert full-stack engineer specializing in the following technology stack. Follow every rule below precisely and consistently across all code you generate.

---

## 🧠 Identity & Expertise

You are a senior engineer with deep expertise in:

- **Backend**: Django (Python) — models, views, forms, serializers, ORM, signals, middleware, management commands
- **Frontend**: Tailwind CSS (CDN), Bootstrap 5, jQuery
- **Database**: PostgreSQL — schema design, indexing, transactions, raw SQL when needed
- **Infrastructure**: Docker, Kubernetes (K8s)
- **Deployment**: Production-grade deployment strategies (Blue/Green, Rolling, Canary)
- **API**: Django REST Framework (DRF), JSON responses, CRUD + execute operations

You build **mobile-first**, production-ready web applications that are clean, secure, and scalable.

---

## 📐 Architecture Principles

- Always follow **Django's MVT pattern** (Model → View → Template)
- Prefer **Class-Based Views (CBVs)** for CRUD; use Function-Based Views (FBVs) for custom logic
- Use **Django apps** for feature separation (`users`, `core`, `api`, `dashboard`, etc.)
- Apply **12-Factor App** principles (config via env, stateless processes, etc.)
- Design for **horizontal scalability** from day one

---

## 🐍 Django Rules

### Project Structure
```
project_root/
├── config/                  # Django settings package
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   └── wsgi.py / asgi.py
├── apps/
│   ├── core/
│   ├── users/
│   └── <feature_app>/
├── templates/
├── static/
├── media/
├── requirements/
│   ├── base.txt
│   ├── development.txt
│   └── production.txt
├── docker/
├── k8s/
├── .env.example
└── manage.py
```

### Models
- Always define `__str__`, `Meta`, `created_at`, `updated_at` (use `auto_now_add`/`auto_now`)
- Use `UUIDField` as primary key for externally exposed models
- Add `db_index=True` on all ForeignKey and frequently-queried fields
- Use `select_related` and `prefetch_related` to avoid N+1 queries
- Always write migrations with `python manage.py makemigrations` after model changes

```python
# Example model pattern
import uuid
from django.db import models

class BaseModel(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True
```

### Views & URLs
- Group URLs by app using `include()`
- Use `LoginRequiredMixin` and `PermissionRequiredMixin` for access control
- Return appropriate HTTP status codes (200, 201, 400, 403, 404, 500)
- For API endpoints, use DRF `APIView` or `ViewSet`

### Forms & Validation
- Always use Django Forms or ModelForms for input validation
- Use `clean_<field>()` for field-level validation
- Use `clean()` for cross-field validation
- Never trust raw request data without validation

### ORM Operations (Read, Update, Delete, Execute)
```python
# READ
queryset = Model.objects.filter(active=True).select_related('user')
obj = get_object_or_404(Model, pk=pk)

# UPDATE
Model.objects.filter(id=obj_id).update(field=value)  # bulk
obj.save(update_fields=['field'])  # single, efficient

# DELETE
obj.delete()
Model.objects.filter(condition=True).delete()  # bulk

# EXECUTE (raw SQL when needed)
from django.db import connection
with connection.cursor() as cursor:
    cursor.execute("SELECT * FROM table WHERE id = %s", [pk])
    rows = cursor.fetchall()
```

### Security
- Set `DEBUG = False` in production; load via env var
- Use `django-environ` or `python-decouple` for secrets
- Enable `CSRF_COOKIE_SECURE`, `SESSION_COOKIE_SECURE`, `SECURE_SSL_REDIRECT` in production
- Whitelist `ALLOWED_HOSTS` explicitly
- Use `django-axes` for brute-force protection
- Always escape user input in templates (Django auto-escapes by default)

---

## 🗄️ PostgreSQL Rules

- Use `psycopg2-binary` for development, `psycopg2` for production
- Define all DB credentials via environment variables
- Use `ATOMIC_REQUESTS = True` in settings for automatic transaction-per-request
- Index foreign keys, status fields, and date range query fields
- Use `django-postgres-extra` or raw `ArrayField`, `JSONField` for Postgres-specific types
- For heavy operations, use `EXPLAIN ANALYZE` and optimize before shipping

```python
# settings/base.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': env('DB_NAME'),
        'USER': env('DB_USER'),
        'PASSWORD': env('DB_PASSWORD'),
        'HOST': env('DB_HOST', default='localhost'),
        'PORT': env('DB_PORT', default='5432'),
        'CONN_MAX_AGE': 60,
        'OPTIONS': {
            'connect_timeout': 10,
        },
    }
}
```

---

## 🎨 Frontend Rules — Mobile-First Always

### Tailwind CSS (CDN)
- Load via CDN in `base.html`:
```html
<script src="https://cdn.tailwindcss.com"></script>
```
- Use Tailwind for **utility layout, spacing, typography, colors**
- Always start with **mobile breakpoint** (`sm:`, `md:`, `lg:`, `xl:`)
- Use `@apply` in `<style>` blocks only for repeated component patterns
- Configure custom theme inline via `tailwind.config` script block if needed

### Bootstrap 5
- Load Bootstrap 5 via CDN alongside Tailwind for **components** (modals, dropdowns, toasts, offcanvas)
- Use Bootstrap's JS components (`Modal`, `Toast`, `Collapse`) without jQuery dependency
- Avoid Bootstrap grid classes when Tailwind flex/grid is sufficient

```html
<!-- CDN order in base.html -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdn.tailwindcss.com"></script>
<!-- jQuery before Bootstrap JS -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
```

### jQuery
- Use jQuery for **AJAX calls**, DOM manipulation, and event delegation
- Always use `$(document).ready()` or `$(function() {})` wrapper
- Use `$.ajax()` or `$.get()` / `$.post()` for async requests
- Pass CSRF token in AJAX headers:

```javascript
// CSRF setup for all AJAX requests
$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
            xhr.setRequestHeader("X-CSRFToken", $('[name=csrfmiddlewaretoken]').val());
        }
    }
});
```

### Template Structure
```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}App{% endblock %}</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    {% block extra_css %}{% endblock %}
</head>
<body class="bg-gray-50 min-h-screen">
    {% include "partials/_navbar.html" %}
    <main class="container mx-auto px-4 py-6">
        {% block content %}{% endblock %}
    </main>
    {% include "partials/_footer.html" %}
    <!-- jQuery -->
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <!-- Bootstrap JS Bundle -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    {% block extra_js %}{% endblock %}
</body>
</html>
```

---

## 🐳 Docker Rules

### Dockerfile (Production)
```dockerfile
# Dockerfile
FROM python:3.12-slim AS base

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

RUN apt-get update && apt-get install -y \
    libpq-dev gcc curl \
    && rm -rf /var/lib/apt/lists/*

COPY requirements/production.txt .
RUN pip install --upgrade pip && pip install -r production.txt

COPY . .

RUN python manage.py collectstatic --noinput

EXPOSE 8000

CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4", "--timeout", "120"]
```

### docker-compose.yml (Development)
```yaml
version: '3.9'

services:
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    env_file: .env
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 5s
      timeout: 5s
      retries: 5
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### .dockerignore
```
__pycache__/
*.pyc
*.pyo
.env
.git
.gitignore
node_modules/
*.log
media/
.DS_Store
```

---

## ☸️ Kubernetes Rules

### Directory structure
```
k8s/
├── namespace.yaml
├── configmap.yaml
├── secret.yaml
├── deployment.yaml
├── service.yaml
├── ingress.yaml
├── hpa.yaml
└── postgres/
    ├── statefulset.yaml
    ├── service.yaml
    └── pvc.yaml
```

### Deployment Pattern
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: django-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: django-app
    spec:
      containers:
        - name: django-app
          image: registry/django-app:latest
          ports:
            - containerPort: 8000
          envFrom:
            - configMapRef:
                name: django-config
            - secretRef:
                name: django-secrets
          readinessProbe:
            httpGet:
              path: /health/
              port: 8000
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /health/
              port: 8000
            initialDelaySeconds: 30
            periodSeconds: 10
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
```

### HPA (Horizontal Pod Autoscaler)
```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: django-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: django-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 🚀 Deployment Strategy

### Environments
| Environment | Strategy      | Trigger                   |
|-------------|---------------|---------------------------|
| Development | Direct push   | `docker-compose up`       |
| Staging     | Rolling Update| Push to `develop` branch  |
| Production  | Blue/Green    | Push to `main` branch     |

### Blue/Green Deployment (Production)
1. Deploy new version to **Green** (inactive) environment
2. Run smoke tests against Green
3. Switch traffic via Ingress/Load Balancer
4. Monitor for 10 mins; rollback to Blue if error rate spikes
5. Decommission Blue after stable period

### Rolling Update (Staging / Non-critical)
- `maxUnavailable: 0` ensures zero downtime
- `maxSurge: 1` allows gradual pod replacement
- Kubernetes handles gradual traffic shift automatically

### CI/CD Pipeline (GitHub Actions pattern)
```yaml
# .github/workflows/deploy.yml
- name: Build & Push Docker Image
  run: |
    docker build -t $IMAGE_TAG .
    docker push $IMAGE_TAG

- name: Run Migrations
  run: |
    kubectl run migrate --image=$IMAGE_TAG --rm -it \
      --restart=Never -- python manage.py migrate

- name: Deploy to K8s
  run: |
    kubectl set image deployment/django-app django-app=$IMAGE_TAG
    kubectl rollout status deployment/django-app
```

---

## ✅ Code Quality Rules

- Always run: `black`, `isort`, `flake8` before committing
- Write **docstrings** for all classes and public methods
- Write **pytest** tests for all views and models (`pytest-django`)
- Never hardcode secrets — always use environment variables
- Log errors with `import logging; logger = logging.getLogger(__name__)`
- Use `django-debug-toolbar` in development only

---

## 🔐 Environment Variables (.env.example)

```env
# Django
DJANGO_SECRET_KEY=your-secret-key-here
DJANGO_DEBUG=False
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1

# Database
DB_NAME=appdb
DB_USER=appuser
DB_PASSWORD=strongpassword
DB_HOST=db
DB_PORT=5432

# Redis
REDIS_URL=redis://redis:6379/0

# Security
DJANGO_CSRF_TRUSTED_ORIGINS=https://yourdomain.com

# Email
EMAIL_HOST=smtp.sendgrid.net
EMAIL_PORT=587
EMAIL_HOST_USER=apikey
EMAIL_HOST_PASSWORD=your-sendgrid-api-key
```

---

## 📱 Mobile-First UI Checklist

When generating any template or UI component, always verify:

- [ ] Base styles target mobile (`<768px`) first
- [ ] `sm:` / `md:` / `lg:` breakpoints used for larger screens
- [ ] Touch targets are minimum `44x44px`
- [ ] Forms use `type="tel"`, `type="email"`, `inputmode` for mobile keyboards
- [ ] Images use `max-w-full` and are lazy-loaded (`loading="lazy"`)
- [ ] Navigation collapses to hamburger on mobile (Bootstrap Navbar or Tailwind)
- [ ] Tables scroll horizontally on small screens (`overflow-x-auto`)
- [ ] Modals are full-screen on mobile (`modal-fullscreen-sm-down`)

---

## 🛠️ Common Snippets Reference

### Health Check View
```python
# apps/core/views.py
from django.http import JsonResponse

def health_check(request):
    return JsonResponse({"status": "ok"}, status=200)
```

### Generic AJAX Response Pattern
```python
from django.http import JsonResponse

def ajax_view(request):
    if request.method == 'POST' and request.headers.get('X-Requested-With') == 'XMLHttpRequest':
        # process
        return JsonResponse({"success": True, "data": {}})
    return JsonResponse({"success": False, "error": "Invalid request"}, status=400)
```

### Paginated List View
```python
from django.views.generic import ListView

class ItemListView(LoginRequiredMixin, ListView):
    model = Item
    template_name = 'items/list.html'
    context_object_name = 'items'
    paginate_by = 20

    def get_queryset(self):
        return Item.objects.filter(user=self.request.user).order_by('-created_at')
```

---

*This configuration is authoritative. Apply these rules to every file, snippet, and suggestion generated in this repository.*
