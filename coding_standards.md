# Coding Standards & Guidelines

## 1. Introduction

This document provides a unified set of coding standards, style guides, and best practices for this project. Adhering to these standards helps us:

-   **Improve Code Consistency and Readability**: Enables any team member to easily understand and maintain the codebase.
-   **Enhance Code Quality and Robustness**: Reduces common errors and potential bugs.
-   **Boost Team Collaboration and Efficiency**: A unified standard minimizes debates over style and allows code reviews to focus on logic and architecture.
-   **Empower AI-Assisted Development**: Provides clear, structured context for AI coding assistants to generate compliant code.

This is a **living document** and will be updated as the project evolves and new technologies are adopted.

---

## 2. General Principles

-   **KISS (Keep It Simple, Stupid)**: Prefer simple, straightforward solutions over overly engineered ones.
-   **DRY (Don't Repeat Yourself)**: Reuse code whenever possible; avoid copy-pasting.
-   **YAGNI (You Ain't Gonna Need It)**: Do not build functionality that is not required for the current task.
-   **Comments**: Code should be self-documenting. Add comments only for complex business logic, algorithms, or to explain the "why," not the "what."
-   **Language**: All code, comments, documentation, and Git commit messages **must** be written in **English**.

---

## 3. Git Workflow

### 3.1. Branching Strategy

-   `main`: The primary branch, which should always be in a deployable state. It only accepts merges from `develop` or `hotfix` branches.
-   `develop`: The main development branch, integrating all completed features. New feature branches are created from `develop`.
-   **Feature Branches**: `feat/<feature-name>` (e.g., `feat/user-authentication`)
-   **Fix Branches**: `fix/<issue-description>` (e.g., `fix/login-button-bug`)
-   **Release Branches**: `release/vX.X.X`
-   **Hotfix Branches**: `hotfix/<issue-description>`

### 3.2. Commit Messages

We follow the **[Conventional Commits](https://www.conventionalcommits.org/)** specification. This helps automate changelog generation and semantic versioning.

Format: `<type>(<scope>): <subject>`

-   **Types**: `feat`, `fix`, `build`, `chore`, `ci`, `docs`, `style`, `refactor`, `perf`, `test`.
-   **Scope (optional)**: The section of the codebase affected, e.g., `api`, `ui`, `auth`, `db`.
-   **Subject**: A concise description of the change, starting with a verb, max 50 characters.

**Examples:**
-   `feat(auth): add JWT token generation`
-   `fix(ui): correct button alignment on login page`
-   `docs(readme): update setup instructions`

### 3.3. Pull Requests

-   The PR title must follow the Conventional Commits specification.
-   The PR description should clearly explain the "what," "why," and "how to test."
-   A PR must be reviewed and approved by at least **one** other team member before merging.
-   All CI checks (Linting, Testing) must pass.

---

## 4. Python (Backend)

We use **Ruff** as our primary linter and formatter, and **Mypy** for type checking. All configurations should be defined in the `pyproject.toml` file.

### 4.1. Formatting

-   **Tool**: `ruff format`
-   **Style**: Follows the Black style with a line width of `88` characters.

### 4.2. Linting

-   **Tool**: `ruff check --fix`
-   **Rules**: Enable core rule sets including `flake8`, `isort`, `pyupgrade`, etc.

### 4.3. Naming Conventions (PEP 8)

-   **Variables/Functions/Methods**: `snake_case`, e.g., `user_name`, `get_user_profile`.
-   **Classes/Types**: `PascalCase`, e.g., `UserProfile`, `DatabaseConnection`.
-   **Constants**: `UPPER_SNAKE_CASE`, e.g., `MAX_RETRIES = 3`.
-   **Modules**: `snake_case`, e.g., `database_utils.py`.

### 4.4. Type Hinting

-   All functions and methods **must** have explicit type hints for parameters and return values.
-   Use `from __future__ import annotations` for a cleaner syntax.
-   **Tool**: `mypy`

```python
# GOOD
from __future__ import annotations

def get_user(user_id: int) -> User | None:
    # ...
    return user
```

### 4.5. Docstrings

  - All public modules, classes, functions, and methods **must** have a docstring.
  - **Style**: Follow the **[Google Python Style Guide](https://www.google.com/search?q=https://google.github.io/styleguide/pyguide.html%2338-comments-and-docstrings)**.

### 4.6. FastAPI Best Practices

  - **Dependency Injection**: Use `Depends` extensively for shared logic like database sessions and authentication.
  - **Data Models**: Use Pydantic models to define all request bodies, response models, and configurations.
  - **Path Operations**: Keep path operation functions simple and focused. Move complex business logic to a Service or CRUD layer.
  - **Project Structure**:
    ```
    /app
    ├── api/          # API Routers
    ├── core/         # Configuration, core settings
    ├── crud/         # Database interaction functions
    ├── models/       # Pydantic models (schemas)
    ├── services/     # Business logic
    └── main.py
    ```

### 4.7. Testing

  - **Framework**: `pytest`
  - **Directory**: All test files are located in the `tests/` directory, mirroring the `app/` structure.
  - **Naming**: Test files must be named `test_*.py`, and test functions `test_*`.

-----

## 5. JavaScript / TypeScript (Frontend)

We strongly recommend using **TypeScript** over plain JavaScript to enhance code robustness.

### 5.1. Formatting

  - **Tool**: **Prettier**. Configuration is stored in a `.prettierrc` file.
  - **Integration**: Configure your editor to format on save.

### 5.2. Linting

  - **Tool**: **ESLint**. Configuration is stored in an `.eslintrc.js` file.
  - **Rules**: Use a recommended rule set (e.g., `plugin:vue/vue3-recommended`, `eslint-config-airbnb`) and customize as needed.

### 5.3. Naming Conventions

  - **Variables/Functions**: `camelCase`, e.g., `userName`, `getUserProfile`.
  - **Classes/Components/Types**: `PascalCase`, e.g., `UserProfile`, `LoginButton.vue`.
  - **Constants**: `UPPER_SNAKE_CASE`, e.g., `const MAX_RETRIES = 3`.

### 5.4. React / Vue Component Design

  - **Single Responsibility**: Components should be small and focused on a single task.
  - **File Naming**: Use `PascalCase` for component filenames (e.g., `UserProfile.jsx`, `BaseModal.vue`).
  - **Props**:
      - Use `camelCase` for prop names.
      - In TypeScript, define a clear `interface` or `type` for props.
      - In Vue, use typed and validated prop definitions.
  - **State Management**:
      - Use local component state (`useState`, `ref`) for simple, non-shared state.
      - For cross-component or global state, choose one solution and use it consistently (e.g., `Context API`, `Redux Toolkit`, `Pinia`, `Zustand`).

### 5.5. API Communication

  - **Abstraction Layer**: Create a unified API service layer (e.g., `services/api.js` or `api/client.ts`) to handle all HTTP requests. Do not use `fetch` or `axios` directly in components.
  - **Environment Variables**: The API base URL and other configurations must be injected via environment variables (`.env`).

### 5.6. Testing

  - **Framework**: **Vitest** or **Jest**.
  - **Library**: **React Testing Library** or **Vue Test Utils**.
  - **Naming**: Test files should be named `*.test.ts(x)` or `*.spec.ts(x)`.

-----

## 6. API Design

  - **Style**: Follow RESTful principles.
  - **Naming**:
      - Use `kebab-case` and plural nouns for endpoint paths (e.g., `/api/v1/user-profiles`).
      - Use `camelCase` for JSON field names in request/response bodies.
  - **Versioning**: Include the API version in the URL (e.g., `/api/v1/...`).
  - **Error Responses**: Use a consistent error response structure.
    ```json
    {
      "error": {
        "code": "AUTH_ERROR",
        "message": "Invalid credentials provided."
      }
    }
    ```

-----

## 7. Tooling & Configuration

  - All configurations for linters, formatters, and type checkers must be committed to the repository (`pyproject.toml`, `.prettierrc`, `tsconfig.json`, etc.).
  - **CI/CD**: These checks must be automated and run as part of the CI pipeline, acting as a quality gate before code can be merged.

