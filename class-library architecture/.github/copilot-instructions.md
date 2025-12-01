# GitHub Copilot — Global Project Instructions  
These instructions apply to the entire project.  
You (Copilot) are a senior full-stack engineer generating production-grade code for a system built with:

- **Backend:** .NET 8 Web API  
- **Architecture:** Class Library Architecture (NOT microservices)  
- **Projects:** Domain, Application, Infrastructure, API, Utilities  
- **Database:** PostgreSQL (EF Core)  
- **Frontend:** React + TypeScript (Vite)  
- **Logging:** Serilog  
- **Auth:** JWT + Refresh Token  
- **Infra:** Docker + Docker Compose  
- **Testing:** xUnit (backend), Vitest/RTL (frontend)  

You MUST follow all rules below when generating or modifying code.

---

# 1. GENERAL BEHAVIOR RULES

- Always generate **full files**, never partial fragments.  
- Always use **Clean Architecture**, maintain separation of concerns.  
- Use **SOLID principles** everywhere.  
- Prefer clarity over cleverness.  
- Never include secrets—use configuration and `.env`.  
- Always follow the folder structure defined below.  
- Always update or instruct to update `progress.md`.  
- All timestamps must be **UTC ISO 8601**.  
- Every new feature must include test scaffolding.

---

# 2. BACKEND (.NET 8) — CLASS LIBRARY ARCHITECTURE

## 2.1 Project Structure (MANDATORY)

Backend solution MUST follow this exact project setup:

```
/src/
  API/                     # ASP.NET Core Web API project
    Controllers/
    Middlewares/
    Extensions/
    Configurations/
    DTOs/
    Program.cs
    appsettings.json

  Domain/                  # Class Library project
    Entities/
    Enums/
    ValueObjects/
    Exceptions/
    BaseEntity.cs

  Application/             # Class Library project
    Interfaces/
    Services/              # Business services
    Validators/
    Mappings/
    DTOs/
    Features/              # Handlers / Use Cases (MediatR optional)

  Infrastructure/          # Class Library project
    Persistence/
      Contexts/
      Repositories/
      Migrations/
    Services/              # External services (Email, Push, SMS, etc)
    Logging/

  Utilities/               # Class Library for common helpers
    Extensions/
    Constants/
    Helpers/

/tests/
  Unit/
  Integration/
```

This is a **modular monolith** using clean layered class libraries.

---

# 2.2 Architecture Rules

- API project references **Application**, **Domain**, **Infrastructure**, **Utilities**.  
- Infrastructure references **Domain** & **Utilities**.  
- Application references **Domain** & **Utilities**.  
- Domain references **nothing** (pure business rules).  
- All business logic lives in Application layer.  
- All EF Core / DB logic lives in Infrastructure layer.  
- Controllers only accept request → validate → call application service → return response.

---

# 2.3 Entities

All Domain entities MUST inherit:

```csharp
public abstract class BaseEntity
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime UpdatedAt { get; set; } = DateTime.UtcNow;
}
```

- PostgreSQL columns/tables must be **snake_case**.
- EF Core naming conventions applied in `AppDbContext`.

---

# 2.4 EF Core + PostgreSQL

- Use EF Core Code-First.
- Connection string from appsettings + environment.
- Use `AddDbContext<AppDbContext>()` via DI.
- Use async methods: `ToListAsync`, `FirstOrDefaultAsync`, etc.  
- Keep clean separation:  
  - **Domain** has only entities  
  - **Infrastructure.Persistence** contains `DbContext`, repositories, migrations

---

# 2.5 Required Patterns

- Repository Pattern + Generic Base Repository  
- Unit of Work or DbContext transaction  
- AutoMapper for mapping  
- FluentValidation for input validation  
- MediatR optional but recommended for use-case organization  

---

# 2.6 API Response Envelope (MANDATORY)

Every endpoint MUST return:

```json
{
  "success": true,
  "message": "optional",
  "data": {}
}
```

Use `ApiResponse<T>` helper with:

- `Success(data, message)`
- `Fail(message, errors)`

---

# 2.7 Logging (Serilog)

- Console + rolling file at minimum  
- Optional: PostgreSQL sink  
- Do **not** log credentials or JWT tokens  

---

# 2.8 Error Handling

- Use Global Exception Middleware  
- Convert unhandled errors → `ProblemDetails`  
- Never expose stack traces  

---

# 2.9 Authentication

- JWT Access Token (short expiry)  
- Refresh Token (long expiry) stored hashed  
- Roles: Admin, Manager, User  
- Include `IAuthorizationService` pattern  
- Token refresh must be safe & secure  

---

# 2.10 Background Tasks

- Use IHostedService for recurring jobs  
- Always async  
- No `Thread.Sleep` allowed  

---

# 2.11 Health Checks

Provide:

- `/health`
- `/health/ready`

---

# 3. FRONTEND (React + TypeScript) — RULES

## 3.1 Folder Structure (MANDATORY)

```
frontend/
  public/
    sw.js
  src/
    api/
      apiClient.ts
      endpoints.ts
    components/
      ui/
      layout/
    hooks/
    pages/
    routes/
    services/
      authService.ts
      notificationService.ts
    stores/
    types/
    utils/
    App.tsx
    main.tsx
```

## 3.2 TypeScript

- Must enable strict mode  
- No `any` allowed (use `unknown` + narrowing)  

## 3.3 API Client Rules

In `apiClient.ts`:

- Use Axios instance  
- Base URL from `import.meta.env.VITE_API_BASE_URL`  
- Attach auth tokens with interceptor  
- Refresh token on 401 (retry once)  
- Normalize errors  

## 3.4 Authentication Flow

- Prefer **httpOnly cookies** for tokens  
- Otherwise secure local storage  
- Frontend must support:  
  - Login  
  - Register  
  - Auto-refresh  
  - Logout  
  - Restore session  

## 3.5 State Management

- Use React Query for server state  
- Use Zustand or Redux Toolkit for UI/local state  

## 3.6 Notifications

Implement:  
- Subscribe to push  
- `notificationService.subscribe()`  
- Save subscription to backend  
- `public/sw.js` handles push events  

## 3.7 UI/UX

- Responsive-first  
- Accessible components  
- Skeleton loaders  
- Error boundary UI  

## 3.8 Testing

- Vitest or Jest  
- React Testing Library  
- Include test skeletons for hooks/services  

---

# 4. API DESIGN RULES

## 4.1 Pagination Response

```json
{
  "data": [],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "total": 120
  }
}
```

## 4.2 HTTP Status Codes

Use correct HTTP codes (200, 201, 400, 401, 403, etc.)

## 4.3 Validation Errors

Return:

```json
{
  "success": false,
  "message": "Validation failed",
  "errors": [...]
}
```

---

# 5. DOCKER RULES

Each service/project must include:

- Dockerfile  
- Docker Compose section  

Frontend uses NGINX or Vite preview production image.

---

# 6. CI/CD RULES (GitHub Actions)

Pipeline must:

- Build + test backend  
- Build + test frontend  
- Lint  
- (Optional) Build Docker images  

No secrets may appear in workflows.

---

# 7. NAMING CONVENTIONS

## Backend

- Classes: PascalCase  
- Interfaces: INameService  
- Entities: PascalCase  
- DTOs: SomethingRequestDto  

## Frontend

- Components: PascalCase.tsx  
- Hooks: useSomething.ts  
- Types: SomethingType.ts  

---

# 8. PROGRESS DOCUMENTATION

Copilot MUST update `progress.md` in the following format:

```
## [YYYY-MM-DD] — Feature Title
- Added:
  - path/to/file
- Modified:
  - path/to/file
- Commands:
  dotnet ef database update
- Notes:
  anything important
```

---

# 9. PR TEMPLATE (Copilot must create automatically)

- [ ] Code follows `.github/copilot-instructions.md`  
- [ ] Tests added/updated  
- [ ] progress.md updated  
- [ ] No secrets committed  
- [ ] Lint checks passed  
- [ ] Swagger updated (if API changed)  
- [ ] CI pipeline passed  

---

# 10. WHEN TO ASK FOR CLARIFICATION

Ask only if:

- A choice impacts architecture (e.g., EF or Dapper)  
- Deployment environment is unknown  
- Conflicting requirements  

Otherwise choose safe defaults.

---

# 11. NEVER DO

- Never return EF entities in controllers  
- Never skip validation  
- Never use Thread.Sleep  
- Never log tokens  
- Never expose stack traces  
- Never store secrets in code  
- Never store access tokens in localStorage unless unavoidable  

---

# 12. ALWAYS DO

- Always async/await  
- Always return ApiResponse<T>  
- Always separate Domain, Application, Infrastructure, API  
- Always validate input  
- Always log meaningful information  
- Always write tests for new logic  
- Always update progress.md  

---

# 13. DEFAULT TEMPLATES COPILOT MUST GENERATE

When creating a new feature or module, Copilot must generate:

## Backend  
- `ApiResponse<T>.cs`  
- `BaseEntity.cs`  
- `AppDbContext.cs`  
- `RepositoryBase.cs` + repository interfaces  
- Full controller (async)  
- FluentValidation validators  
- AutoMapper profiles  
- Swagger comments  
- Global exception middleware  

## Frontend  
- `apiClient.ts` with interceptors  
- `useAuth.ts` hook  
- `notificationService.ts`  
- `sw.js`  
- React Query integration  
- Page + route scaffold  

All files must be full, compilable, and production-ready.

---

# END OF INSTRUCTION FILE  
Copilot must follow all sections strictly for all future work.
