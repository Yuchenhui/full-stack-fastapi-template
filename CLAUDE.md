# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack FastAPI template with React frontend, PostgreSQL database, and Docker containerization. The project follows modern development practices with comprehensive type safety, authentication, and production-ready deployment.

**Tech Stack:**
- **Backend**: FastAPI + SQLModel + PostgreSQL + JWT authentication
- **Frontend**: React 18 + TypeScript + TanStack Router/Query + Chakra UI v3
- **Infrastructure**: Docker Compose + Traefik + PostgreSQL
- **Testing**: Pytest (backend) + Playwright (frontend E2E)

## Key Architecture Patterns

### Backend Architecture
- **Clean Architecture**: Separation of concerns with API routes, CRUD operations, and models
- **Dependency Injection**: FastAPI's dependency system for database sessions and authentication
- **Type Safety**: Full SQLModel integration with Pydantic validation
- **Authentication**: JWT tokens with role-based access control (superuser/regular users)

### Frontend Architecture
- **File-based Routing**: TanStack Router with type-safe route generation
- **Server State Management**: TanStack Query for API state with automatic caching
- **Auto-generated Client**: TypeScript API client generated from OpenAPI spec
- **Component Organization**: Domain-driven structure (Admin/, Items/, Common/, ui/)

### Database Design
- **SQLModel ORM**: Combines SQLAlchemy with Pydantic for type safety
- **UUID Primary Keys**: Security through non-enumerable identifiers
- **Migration System**: Alembic for version-controlled schema changes
- **Relationships**: Proper cascade deletes and foreign key constraints

## Common Development Commands

### Full Stack Development
```bash
# Start entire stack with hot reload
docker compose watch

# View logs for specific service
docker compose logs backend
docker compose logs frontend

# Stop specific service to run locally
docker compose stop backend
docker compose stop frontend
```

### Backend Development
```bash
# Install dependencies
cd backend && uv sync

# Activate virtual environment
source backend/.venv/bin/activate

# Run backend locally (after stopping Docker service)
cd backend && fastapi dev app/main.py

# Run tests
bash ./scripts/test.sh

# Run tests on running stack
docker compose exec backend bash scripts/tests-start.sh

# Run specific test with extra args
docker compose exec backend bash scripts/tests-start.sh -x  # stop on first error

# Database migrations
docker compose exec backend bash
alembic revision --autogenerate -m "Add column to User model"
alembic upgrade head
```

### Frontend Development
```bash
# Install dependencies
cd frontend && npm install

# Run frontend locally (after stopping Docker service)
cd frontend && npm run dev

# Generate API client after backend changes
./scripts/generate-client.sh

# E2E tests
docker compose up -d --wait backend
cd frontend && npx playwright test
cd frontend && npx playwright test --ui

# Linting and formatting
npm run lint
npm run format
```

### Testing Commands
```bash
# Backend tests
bash ./scripts/test.sh                    # Full test suite with Docker
docker compose exec backend bash scripts/tests-start.sh  # Quick tests on running stack

# Frontend E2E tests
docker compose up -d --wait backend
cd frontend && npx playwright test

# Test coverage
# Backend: check htmlcov/index.html after running tests
# Frontend: E2E tests with visual regression detection
```

## Critical Security Considerations

### ⚠️ Before Production Deployment
1. **Change Default Secrets**: Update all "changethis" values in `.env`
   - `SECRET_KEY`
   - `FIRST_SUPERUSER_PASSWORD`
   - `POSTGRES_PASSWORD`

2. **Generate Secure Keys**:
   ```bash
   python -c "import secrets; print(secrets.token_urlsafe(32))"
   ```

3. **Frontend Vulnerabilities**: Run `npm audit fix` in frontend/ directory

### Security Implementation Details
- **JWT Authentication**: 8-day token expiration with proper validation
- **Password Security**: BCrypt hashing with minimum 8-character requirement
- **Input Validation**: Comprehensive Pydantic validation on all endpoints
- **CORS Configuration**: Environment-aware CORS origins
- **SQL Injection Protection**: SQLModel ORM prevents direct SQL injection

## Code Quality Standards

### Type Safety
- **Backend**: 100% type hints with SQLModel integration
- **Frontend**: Strict TypeScript with auto-generated API client
- **Validation**: Pydantic models for all API inputs/outputs

### Linting and Formatting
- **Backend**: Ruff for linting and formatting
- **Frontend**: Biome for linting and formatting
- **Pre-commit Hooks**: Automatic code quality checks

```bash
# Run pre-commit manually
uv run pre-commit run --all-files
```

## Database Operations

### Model Development Pattern
1. **Modify Models**: Edit `backend/app/models.py`
2. **Generate Migration**: `alembic revision --autogenerate -m "Description"`
3. **Apply Migration**: `alembic upgrade head`
4. **Update API**: Modify routes in `backend/app/api/routes/`
5. **Generate Client**: Run `./scripts/generate-client.sh`

### Database Access Patterns
- **Session Management**: Use `SessionDep` dependency injection
- **CRUD Operations**: Located in `backend/app/crud.py` (partially implemented)
- **Relationships**: Proper foreign key constraints with cascade deletes
- **Validation**: SQLModel provides automatic validation

## Environment Configuration

### Development URLs
- **Frontend**: http://localhost:5173
- **Backend**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs
- **Database Admin**: http://localhost:8080 (Adminer)
- **Traefik Dashboard**: http://localhost:8090

### Environment Variables
Key variables in `.env` file:
- `ENVIRONMENT`: local/staging/production
- `SECRET_KEY`: JWT signing key
- `POSTGRES_PASSWORD`: Database password
- `FIRST_SUPERUSER_PASSWORD`: Initial admin password
- `FRONTEND_HOST`: Frontend URL for CORS
- `BACKEND_CORS_ORIGINS`: Comma-separated CORS origins

## Production Deployment

### Docker Images
```bash
# Build and push images
bash ./scripts/build.sh
bash ./scripts/build-push.sh

# Deploy to production
bash ./scripts/deploy.sh
```

### Deployment Configuration
- **Traefik**: Automatic HTTPS with Let's Encrypt
- **PostgreSQL**: Production database with proper credentials
- **Environment**: Set `ENVIRONMENT=production`
- **Secrets**: Use environment variables, not .env file

## Known Architecture Issues

Based on comprehensive code analysis, be aware of these patterns:

### Backend Issues
- **Inconsistent CRUD**: Partial CRUD implementation in `crud.py` - some routes use it, others don't
- **Mixed Business Logic**: Some business logic in route handlers instead of service layer
- **Error Handling**: Non-standardized error response formats

### Performance Considerations
- **Database**: No connection pooling configured
- **Caching**: No Redis/caching layer implemented
- **Frontend**: Missing code splitting and virtualization for large lists

### Recommended Improvements
1. **Standardize CRUD Operations**: Use consistent pattern across all entities
2. **Add Service Layer**: Extract business logic from route handlers
3. **Implement Caching**: Add Redis for session and query caching
4. **Add Rate Limiting**: Protect against brute force attacks
5. **Security Headers**: Add CSP, HSTS, and other security headers

## Development Workflow

### Typical Development Cycle
1. **Start Stack**: `docker compose watch`
2. **Make Changes**: Edit backend/frontend code
3. **Test Changes**: Automatic reload with Docker watch
4. **Run Tests**: Backend and frontend test suites
5. **Generate Client**: After backend API changes
6. **Commit**: Pre-commit hooks ensure code quality

### Working with Both Services
- **Backend Changes**: Hot reload with uvicorn
- **Frontend Changes**: Hot reload with Vite
- **Database Changes**: Requires migration + client regeneration
- **Full Stack Features**: Coordinate backend API + frontend integration

This template provides a solid foundation for modern full-stack applications with production-ready defaults and comprehensive development tooling.