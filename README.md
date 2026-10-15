# Crestview Realty — `real-estate-company`

A realistic real estate listings web application built as a **DevOps training project** for
JumpToTech DevOps School. Students use this repo to practice CI pipelines, static analysis,
container security scanning, Docker, and Kubernetes deployment on a real (if fictional)
full-stack application.

## Architecture

Two independently deployable services, matching a typical microservice CI/CD exercise:

```
real-estate-company/
├── backend/     Node.js + Express REST API (property data, search & filters)
├── frontend/    React + Vite + Tailwind CSS single-page app
├── k8s/         Kubernetes manifests (Deployments, Services, Namespace)
├── .github/workflows/ci.yml   GitHub Actions pipeline
├── docker-compose.yml         Local multi-container dev environment
└── sonar-project.properties   SonarQube scanner configuration
```

- **Frontend → Backend**: the React app calls `/api/*`, which is proxied to the Express API
  (via Vite's dev proxy locally, or Nginx's `location /api/` block in the production container).
- **Backend**: in-memory data store (`backend/src/data/properties.js`) — no database required,
  keeping the stack simple for CI/CD focused lessons.

## Features

- Modern hero section with a city search box
- 12 sample property listings with images, price, beds, baths, and square footage
- Search by city, filter by min/max price and minimum bedrooms
- Property details page with photo gallery, specs, description, and features
- `/health` endpoints on both services for container/Kubernetes health checks

## Running locally (without Docker)

**Backend:**
```bash
cd backend
npm install
npm run dev        # http://localhost:4000
```

**Frontend** (in a separate terminal):
```bash
cd frontend
npm install
npm run dev         # http://localhost:5173, proxies /api to localhost:4000
```

## Running with Docker Compose

```bash
docker compose up --build
```

- Frontend: http://localhost:8080
- Backend API: http://localhost:4000

## Running tests & lint

```bash
cd backend && npm install && npm test && npm run lint
cd frontend && npm install && npm run lint && npm run build
```

## CI/CD pipeline (`.github/workflows/ci.yml`)

The pipeline demonstrates a typical DevOps flow:

1. **Backend job** — install, lint (ESLint), test with coverage (Jest + Supertest)
2. **Frontend job** — install, lint, build (Vite)
3. **SonarQube job** — static analysis scan using `sonar-project.properties`
   (runs only if the `SONAR_HOST_URL` repository variable is set; requires a `SONAR_TOKEN` secret)
4. **Docker job** — builds both Docker images, then scans each with **Trivy** for
   CRITICAL/HIGH vulnerabilities (fails the build on findings)

### Setting up SonarQube

1. Run a SonarQube server (e.g. `docker run -d -p 9000:9000 sonarqube:community`)
2. Create a project and generate a token
3. In the GitHub repo: add repository **variable** `SONAR_HOST_URL` and **secret** `SONAR_TOKEN`

### Setting up Trivy locally

```bash
trivy image real-estate-company-backend:local
trivy image real-estate-company-frontend:local
```

### About `.trivyignore`

The root `.trivyignore` suppresses a set of CVEs that live inside **npm's own
bundled dependencies** in the official `node:20-alpine` base image (not in
this app's code or its production `node_modules`) — Trivy flags them, but
there is nothing the application Dockerfile can do about them beyond waiting
for a new base image release. Each entry is commented with its origin and a
"last reviewed" date.

This is a deliberate, teachable example of a real vulnerability-management
decision. For class exercises, try: removing `.trivyignore` to show the
pipeline failing on real findings, triaging a finding, bumping the base image
tag, or switching to a distroless runtime image.

## Kubernetes (`k8s/`)

Basic manifests are included for the next stage of training:

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/backend-deployment.yaml -f k8s/backend-service.yaml
kubectl apply -f k8s/frontend-deployment.yaml -f k8s/frontend-service.yaml
```

Build and push images first (or load them into your local cluster, e.g.
`minikube image load real-estate-company-backend:latest`), and update the `image:` fields
in the deployment manifests to point at your registry.

## API Reference

| Method | Path                      | Description                                  |
|--------|---------------------------|-----------------------------------------------|
| GET    | `/health`                 | Health check                                   |
| GET    | `/api/properties`         | List properties (query: `city`, `minPrice`, `maxPrice`, `bedrooms`, `type`) |
| GET    | `/api/properties/:id`     | Single property details                        |
| GET    | `/api/properties/cities`  | Distinct list of cities                         |

## Notes for instructors

- All property data is fictional and generated for this project.
- Images are placeholder photos served via picsum.photos — swap for real assets if
  the training environment has no outbound internet access.
- The app has no database and no auth layer by design, so students can focus on the
  CI/CD/container/Kubernetes mechanics without extra infrastructure to stand up first.
# real-estate-company
