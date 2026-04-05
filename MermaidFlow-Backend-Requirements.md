# MermaidFlow — Application Name & .NET Core Backend Requirements

---

## 🏷️ Application Name Suggestions

| Name | Why It Works |
|------|-------------|
| **MermaidFlow** | Clear, catchy — highlights the Mermaid focus |
| **GraphDown** | Blend of "Graph" + "Markdown" |
| **MarkGraph** | Short, memorable, says what it does |
| **FlowDown** | Flowcharts + Markdown |
| **Diagramify** | Action-oriented, modern feel |
| **MermView** | Mermaid + Viewer, concise |
| **VisualMD** | Visual Markdown — straightforward |
| **ChartDown** | Charts + Markdown |

**Top recommendation: `MermaidFlow`** — it's distinctive, instantly communicates the core feature (Mermaid diagrams), and sounds professional for a portfolio project.

---

## 🔧 .NET Core Backend Requirements

### Tech Stack

| Component | Technology |
|-----------|-----------|
| **Framework** | ASP.NET Core 8+ (Web API) |
| **ORM** | Entity Framework Core |
| **Database** | SQL Server or PostgreSQL |
| **Authentication** | ASP.NET Core Identity + JWT Bearer |
| **Caching** | IMemoryCache or Redis (for rendered diagram caching) |
| **File Storage** | Local disk or Azure Blob Storage |
| **API Documentation** | Swagger / Swashbuckle |
| **Logging** | Serilog |
| **Testing** | xUnit + Moq |

---

### API Endpoints

#### 1. Documents API — CRUD for Markdown Documents

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/documents` | List all user documents (paginated) |
| `GET` | `/api/documents/{id}` | Get a single document |
| `POST` | `/api/documents` | Create a new document |
| `PUT` | `/api/documents/{id}` | Update a document |
| `DELETE` | `/api/documents/{id}` | Delete a document |
| `GET` | `/api/documents/{id}/export?format=html\|pdf` | Export document |

#### 2. Mermaid API — Diagram Rendering & Validation

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/mermaid/render` | Accept Mermaid code → return rendered SVG |
| `POST` | `/api/mermaid/validate` | Validate Mermaid syntax, return errors |
| `POST` | `/api/mermaid/export` | Export diagram as SVG or PNG |
| `GET` | `/api/mermaid/themes` | List available Mermaid themes |

#### 3. Auth API

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/auth/register` | User registration |
| `POST` | `/api/auth/login` | Login → return JWT |
| `POST` | `/api/auth/refresh` | Refresh JWT token |
| `POST` | `/api/auth/logout` | Revoke refresh token |

---

### Data Models

#### `Document`

```
- Id (Guid)
- Title (string, required, max 200)
- Content (string, required)  // Raw markdown
- UserId (Guid, FK)
- CreatedAt (DateTime)
- UpdatedAt (DateTime)
- IsPublic (bool)
- Tags (List<string>)
```

#### `User`

```
- Id (Guid)
- Email (string, unique)
- PasswordHash (string)
- DisplayName (string)
- CreatedAt (DateTime)
```

#### `DiagramCache` (optional — for performance)

```
- Id (Guid)
- MermaidHash (string, indexed)  // SHA256 of mermaid code
- RenderedSvg (string)           // Cached SVG output
- Theme (string)
- CreatedAt (DateTime)
- ExpiresAt (DateTime)
```

---

### Project Structure

```
MermaidFlow.Backend/
├── MermaidFlow.API/                    # Web API project
│   ├── Controllers/
│   │   ├── DocumentsController.cs
│   │   ├── MermaidController.cs
│   │   └── AuthController.cs
│   ├── Middleware/
│   │   ├── ExceptionHandlingMiddleware.cs
│   │   └── RequestLoggingMiddleware.cs
│   ├── Filters/
│   │   └── ValidationFilter.cs
│   ├── Program.cs
│   └── appsettings.json
│
├── MermaidFlow.Core/                   # Domain layer
│   ├── Entities/
│   │   ├── Document.cs
│   │   ├── User.cs
│   │   └── DiagramCache.cs
│   ├── Interfaces/
│   │   ├── IDocumentRepository.cs
│   │   ├── IMermaidService.cs
│   │   └── IAuthService.cs
│   └── DTOs/
│       ├── DocumentDto.cs
│       ├── MermaidRenderRequest.cs
│       └── MermaidRenderResponse.cs
│
├── MermaidFlow.Infrastructure/         # Data access & external services
│   ├── Data/
│   │   ├── AppDbContext.cs
│   │   └── Migrations/
│   ├── Repositories/
│   │   └── DocumentRepository.cs
│   └── Services/
│       ├── MermaidRenderService.cs     # Server-side Mermaid rendering
│       ├── ExportService.cs
│       └── DiagramCacheService.cs
│
├── MermaidFlow.Tests/                  # Unit & integration tests
│   ├── Controllers/
│   ├── Services/
│   └── Repositories/
│
└── MermaidFlow.sln
```

---

### Key Backend Features to Implement

#### 1. Server-Side Mermaid Rendering (Most Important)

- Use **Puppeteer Sharp** (headless Chromium for .NET) or **Playwright for .NET** to render Mermaid diagrams server-side
- Alternative: Use **Mermaid CLI (`mmdc`)** via `Process.Start()` — simpler but requires Node.js installed on the server
- Cache rendered SVGs by hashing the Mermaid code input (avoid re-rendering identical diagrams)

#### 2. Diagram Caching Strategy

- Hash Mermaid code + theme → check cache before rendering
- Use `IMemoryCache` for simple deployments, Redis for distributed
- Set expiration (e.g., 24 hours) to limit memory usage

#### 3. Input Validation & Security

- Sanitize Markdown input to prevent XSS
- Limit Mermaid code size (e.g., max 50KB per diagram)
- Rate limit the `/api/mermaid/render` endpoint (rendering is CPU-intensive)
- Use `FluentValidation` for request validation

#### 4. Export Service

- HTML export: Render full Markdown with embedded SVG diagrams
- PDF export: Use a library like **QuestPDF** or **DinkToPdf**
- SVG/PNG export: Extract individual diagrams

#### 5. Real-Time Preview (Optional)

- Add **SignalR** hub for real-time document collaboration
- Push rendered diagram updates via WebSocket instead of polling

---

### NuGet Packages

| Package | Purpose |
|---------|---------|
| `Microsoft.EntityFrameworkCore` | ORM |
| `Npgsql.EntityFrameworkCore.PostgreSQL` | PostgreSQL provider |
| `Microsoft.AspNetCore.Authentication.JwtBearer` | JWT auth |
| `FluentValidation.AspNetCore` | Request validation |
| `Serilog.AspNetCore` | Structured logging |
| `Swashbuckle.AspNetCore` | Swagger/OpenAPI docs |
| `PuppeteerSharp` | Headless Chrome for server-side Mermaid rendering |
| `QuestPDF` | PDF export |
| `Markdig` | Server-side Markdown parsing (.NET native) |
| `AspNetCoreRateLimit` | API rate limiting |

---

### Mermaid Rendering Approach (Server-Side)

The critical backend piece — two options:

**Option A: Puppeteer Sharp (Recommended)**
- Spin up headless Chromium, load Mermaid.js, render to SVG
- Pro: Full Mermaid.js compatibility, renders exactly like the browser
- Con: Heavier resource usage, needs Chromium on the server

**Option B: Mermaid CLI Wrapper**
- Shell out to `npx mmdc` (Mermaid CLI) to render diagrams
- Pro: Simple implementation
- Con: Requires Node.js on the server, slower per-request

**Option C: Hybrid (Best for Production)**
- Client renders diagrams in real-time (for the live preview)
- Backend renders only for export (PDF, HTML) and caching
- This minimizes server load while still supporting export features
