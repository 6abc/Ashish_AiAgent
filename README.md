# GitHub Copilot Instructions

Place this file at:

```text
.github/copilot-instructions.md
```

GitHub Copilot will automatically use these instructions when generating code for this repository.

## Included Standards

### Django

- Enforce Django MVT (Model-View-Template) architecture.
- Prefer Class-Based Views (CBVs) where appropriate.
- Use Function-Based Views (FBVs) only when they provide clearer implementation.
- Use UUID primary keys for all new models.
- Prevent N+1 query issues using:
  - `select_related()`
  - `prefetch_related()`
- Split settings into:
  - `settings/base.py`
  - `settings/development.py`
  - `settings/production.py`
- Store secrets and environment-specific values in environment variables.

### PostgreSQL

- Configure database connections using environment variables.
- Enable `ATOMIC_REQUESTS=True` where transaction consistency is required.
- Apply proper indexing strategies for frequently queried fields.
- Support:
  - Create operations
  - Read operations
  - Update operations
  - Delete operations
- Use raw SQL execution only when ORM performance or functionality requires it.

### Frontend (Mobile-First)

#### Required Asset Load Order

```html
<!-- Bootstrap CSS -->
<link>

<!-- Tailwind CSS -->
<script>

<!-- jQuery -->
<script>

<!-- Bootstrap JS -->
<script>
```

#### AJAX Requirements

- Configure jQuery AJAX requests with CSRF protection.
- Use Django CSRF tokens in all POST, PUT, PATCH, and DELETE requests.

#### Base Template

Generate a complete `base.html` that includes:

- Mobile-first responsive layout
- Bootstrap
- Tailwind
- jQuery
- CSRF support
- Reusable blocks for:
  - title
  - content
  - css
  - javascript

### Docker

Provide:

#### Dockerfile

- Production-ready image
- Gunicorn as application server
- Minimal image size
- Non-root user execution

#### docker-compose.yml

Include:

- Django service
- PostgreSQL service
- Health checks
- Environment variables
- Persistent database volumes

#### .dockerignore

Exclude:

- `.git`
- `__pycache__`
- `.venv`
- `node_modules`
- local development files
- temporary files

### Kubernetes

Use the following structure:

```text
k8s/
├── namespace.yaml
├── deployment.yaml
├── service.yaml
├── ingress.yaml
├── hpa.yaml
├── configmap.yaml
└── secret.yaml
```

#### Deployment

- RollingUpdate strategy
- Readiness probes
- Liveness probes
- Resource requests and limits

#### Horizontal Pod Autoscaler

Requirements:

- Minimum replicas: 2
- Maximum replicas: 10
- Target CPU utilization: 70%

### Deployment Strategy

#### Production

Use Blue/Green deployment strategy.

Requirements:

- Zero-downtime deployments
- Database migration execution before traffic switch
- Rollback capability

#### Staging

Use Rolling deployments.

### CI/CD

Use GitHub Actions.

Pipeline should include:

1. Linting
2. Unit tests
3. Build container image
4. Push image to registry
5. Run database migrations
6. Deploy to Kubernetes
7. Verify deployment health

### Mobile-First Checklist

Apply this checklist to every generated template:

- Mobile-first responsive design
- Touch targets minimum 44x44 pixels
- Proper Bootstrap breakpoints
- Responsive navigation
- Horizontal scrolling for wide tables
- Full-screen mobile modals
- Responsive images
- Avoid horizontal page scrolling
