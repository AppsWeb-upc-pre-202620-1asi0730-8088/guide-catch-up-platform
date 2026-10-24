# CatchUp Platform Guide

## Table of Contents

- [Project Setup](#project-setup)
- [(TS001) Create a Favorite Source](#create-a-favorite-source-ts001)
- [(TS002) Retrieve a Favorite Source by Id](#retrieve-a-favorite-source-by-id-ts002)
- [(TS003) List Favorite Sources by NewsAPI Key](#list-favorite-sources-by-newsapi-key-ts003)
- [(TS004) Check Whether a Source Is Favorite](#check-whether-a-source-is-favorite-ts004)
- [(TS006) Remove a Favorite Source](#remove-a-favorite-source-ts006)
- [Add XML Doc Comments to Every Class](#add-xml-doc-comments-to-every-class)
- [Prepare the First Release](#prepare-the-first-release)
- [Release](#release)
- [Testing (optional, explore on your own)](#testing-optional-explore-on-your-own)
- [Appendix](#appendix)
  - [Continuing on another computer](#continuing-on-another-computer)
  - [Installing the GitHub CLI](#installing-the-github-cli)
  - [Signing in to GitHub with a token](#signing-in-to-github-with-a-token)
  - [Backing up unfinished work](#backing-up-unfinished-work)
  - [Feature Finish and pull requests](#feature-finish-and-pull-requests)
  - [Shipping a fix after a release (hotfix)](#shipping-a-fix-after-a-release-hotfix)
  - [Removing a stray .git folder](#removing-a-stray-git-folder)
  - [Creating the repo without the GitHub CLI](#creating-the-repo-without-the-github-cli)
  - [If the class diagram doesn't render](#if-the-class-diagram-doesnt-render)
  - [If MySQL refuses the connection](#if-mysql-refuses-the-connection)
  - [Free JetBrains license for students](#free-jetbrains-license-for-students)

## Project Setup

1. **Create the GitHub repo and clone it.**

   **Sign in first.** On a shared machine, sign out whoever used it last, then sign back in as yourself:

   ```
   gh auth logout
   gh auth login
   ```

   Answer its four prompts:
   - *Where do you use GitHub?* → `GitHub.com`
   - *What is your preferred protocol for Git operations on this host?* → `HTTPS`
   - *Authenticate Git with your GitHub credentials?* → `Yes`
   - *How would you like to authenticate GitHub CLI?* → `Login with a web browser`

   It shows a one-time code (like `3155-2B43`) and waits at `Press Enter to open https://github.com/login/device...`. Copy the code, press Enter, paste it into the page that opens, click **Authorize**.

   **Note:** `gh auth logout` prints "command not found", or this is a different machine than before? Install the CLI first: see [Appendix: Installing the GitHub CLI](#installing-the-github-cli).

   **Create your own GitHub organization first.** Everything below pushes to an organization that is yours, never the course's.
   - If you don't have one, go to [github.com/organizations/plan](https://github.com/organizations/plan), pick the **Free** plan, choose an account name.
   - That account name is your `<org>` in the command below.

   **Create the private repo, blank.** Replace `<org>` with your organization's name.

     ```
     gh repo create <org>/catch-up-platform --private --description "A REST API built with ASP.NET Core, Domain-Driven Design, and CQRS, tracking favorite news sources."
     ```

     This creates an empty repository on GitHub — no README, no license, no `.gitignore`, and nothing on disk yet.

   **Clone it.** `cd` into whatever folder should hold it — this creates a `catch-up-platform` folder right there, wired to `origin`.

     ```
     gh repo clone <org>/catch-up-platform
     ```

     **Note:** this is the one command to remember separately from the one above — in an exam, the repo is handed to you already created, so this `gh repo clone` is where your own work actually starts. There's nothing to build yet; step 2 scaffolds the solution inside this same folder.

   **Note:** no GitHub CLI? Create the repo on the website and clone it by hand: see [Appendix: Creating the repo without the GitHub CLI](#creating-the-repo-without-the-github-cli).

2. **Scaffold the solution with the .NET CLI.** Rider's own "New Solution" wizard refuses to target a folder that already exists — and the one from step 1 already does, `.git` and all. The `dotnet` CLI has no such restriction, so use it instead. In a terminal, move into the cloned folder:

   ```
   cd catch-up-platform
   ```

   Then run:

   ```
   dotnet new sln -n catch-up-platform --format sln
   dotnet new webapi -n Acme.CatchUp.Platform -o Acme.CatchUp.Platform -controllers -f net10.0
   dotnet sln catch-up-platform.sln add Acme.CatchUp.Platform/Acme.CatchUp.Platform.csproj
   ```

   - `--format sln` keeps the classic `.sln` format the rest of this course already uses; .NET 10's own default is the newer `.slnx`, which Rider can open too, but it isn't what you'll see in the other guides.
   - `-controllers` matches this project's MVC-style `[ApiController]` classes, not Minimal APIs — the one setting that needs a flag. OpenAPI generation and top-level statements are both already the template's default, so no flags for either.

   **Note:** this is the general technique for adding a .NET solution into a repo that already exists before you start — including one you're simply handed, such as in an exam. The CLI doesn't care whether the target folder is empty, already has a `.git`, or came from someone else; Rider's GUI wizard does.

   Now open it in Rider: Welcome screen → **Open**, or `File` → `Open...` if a solution is already open → pick the `catch-up-platform` folder (the one containing `catch-up-platform.sln`). Rider recognizes the solution and opens it directly — no wizard involved this time.

   **Note:** every IDE action in this guide comes with a menu path and a keyboard shortcut, and the shortcuts are the ones from Rider's **IntelliJ** keymap. Rider ships with a Visual Studio keymap by default, so switch it once: open `Settings` (`Rider → Settings` on macOS, `File → Settings` on Windows) → `Keymap` and pick **`IntelliJ`** from the dropdown at the top (on a Mac it may read `IntelliJ (macOS)`). If a shortcut ever does something unexpected, use the menu path instead.

3. **Add NuGet dependencies and set the project properties.** Open Rider's **Terminal** tool window (bottom toolbar); it opens at the solution root by default. Run:

   ```
   dotnet add Acme.CatchUp.Platform package Microsoft.EntityFrameworkCore --version 10.0.13
   dotnet add Acme.CatchUp.Platform package Microsoft.EntityFrameworkCore.Relational --version 10.0.13
   dotnet add Acme.CatchUp.Platform package Microsoft.EntityFrameworkCore.Design --version 10.0.13
   dotnet add Acme.CatchUp.Platform package MySql.EntityFrameworkCore --version 10.0.9
   dotnet add Acme.CatchUp.Platform package Humanizer --version 3.0.10
   dotnet add Acme.CatchUp.Platform package Scalar.AspNetCore --version 2.18.0
   ```

   This adds every `<PackageReference>` to `Acme.CatchUp.Platform.csproj` for you, no manual XML to paste. `Humanizer` is only used for its pluralization helper (`"FavoriteSource".Pluralize()` → `"FavoriteSources"`), part of the snake-case naming convention later in this section. `Scalar.AspNetCore` renders the OpenAPI document the `webapi` template already generates as an interactive page, the same role Swagger UI plays in this course's Java/Spring Boot projects.

   Now switch Solution Explorer to **File System** view (the dropdown at the top of the panel; **Solution** view, the default, doesn't show the `.csproj`). Open `Acme.CatchUp.Platform.csproj` and add three more properties inside the existing `<PropertyGroup>`:

   ```xml
   <LangVersion>14</LangVersion>
   <InvariantGlobalization>false</InvariantGlobalization>
   <Version>0.1.0-preview</Version>
   ```

   `LangVersion` is set explicitly because this project uses C# 14 features. `InvariantGlobalization` defaults to `true` in .NET 10 (it trims culture-specific data to shrink deployments), but this project localizes its responses into `en`, `en-US`, `es`, and `es-PE` with real `.resx` resources further down this section — that needs the real culture data the invariant mode strips out, so it's switched off here, once, before any localization code exists.

   **Note:** the dropdown at the top of Solution Explorer switches between two layouts:
   - **File System** view mirrors the folders on disk. Use it to add plain files: anything under `docs/`, `.gitignore`, `README.md`, `LICENSE.md`, `CHANGELOG.md`, `.resx` resource files, and to edit the `.csproj` directly.
   - **Solution** view shows the logical project. Use it to add C# classes/interfaces/records.

   Each step below says which view it needs.

   **Note:** a `-preview` suffix is NuGet's way of marking a version "prerelease". Starting below `1.0.0` signals early development. `## Release` later walks through dropping the suffix and jumping to `1.0.0`.

4. **Point the browser launch at Scalar.** Still in **File System** view, open `Properties/launchSettings.json`. Set `launchBrowser` to `true` and `launchUrl` to `scalar` in both the `http` and `https` profiles:

   ```json
   "launchBrowser": true,
   "launchUrl": "scalar"
   ```

   **Note:** don't paste a whole `launchSettings.json` from anywhere else, including this guide. The template assigns each project its own local ports (`applicationUrl`); overwriting the whole file would replace yours with someone else's. Only change the two lines above, in both profiles.

5. **Update `Program.cs` so the app boots, and verify it with the example endpoint still there.** Open `Program.cs` and replace the template-generated content — just enough to serve Scalar. No database, no localization; each of those gets wired in later in this section, next to the class that needs it. The example `WeatherForecastController` from the `webapi` template is still in the project at this point, so `AddControllers()`/`MapControllers()` stay in, same as the template left them.

   <details>
   <summary>Program.cs</summary>

   ```csharp
   using Scalar.AspNetCore;

   var builder = WebApplication.CreateBuilder(args);

   builder.Services.AddControllers();
   builder.Services.AddOpenApi();

   var app = builder.Build();

   app.UseHttpsRedirection();
   app.UseAuthorization();

   if (app.Environment.IsDevelopment())
   {
       app.MapOpenApi();
       app.MapScalarApiReference();
   }

   app.MapControllers();

   app.Run();
   ```
   </details>

   Run it (green ▷ next to `main` in the toolbar). The browser opens straight to `/scalar` (step 4) and renders the page listing one endpoint, `GET /WeatherForecast` — the template's own example, still here on purpose: seeing it listed confirms the host, Scalar, and the OpenAPI pipeline all boot correctly, with a real, known-good endpoint to prove it works before any of this project's own code exists. Stop the app once it renders.

   **Note:** `MapOpenApi`/`MapScalarApiReference` are guarded by `IsDevelopment()` — the generated document and its interactive page are a build-time aid, not something a production deployment should serve to the public internet. Rider's default run profile is `Development` (`Properties/launchSettings.json`), so this guard never gets in the way of anything in this guide.

6. **Delete the Controllers folder and the example class.** It did its job verifying the pipeline in step 5; nothing from here on reuses it. Still in **File System** view, delete:
   - `Controllers/WeatherForecastController.cs`, and the now-empty `Controllers/` folder with it
   - `WeatherForecast.cs`, at the project root

   Run the app again (optional): Scalar now renders empty, with no endpoints listed — expected, since nothing has been mapped to a route yet.

7. **Set up `.gitignore`.** Still in **File System** view: right-click the solution root → `Add` → `File` → type `.gitignore` → Enter. If Rider already left one at the solution root, open that one and replace its contents instead.

   <details>
   <summary>.gitignore</summary>

   ```
   bin/
   obj/
   /packages/
   riderModule.iml
   /_ReSharper.Caches/
   .DS_Store
   .idea/
   *.user
   .vs/
   ```
   </details>

   **Note:**
   - `.idea/` and `.vs/` have to be ignored **before** the first commit: the IDE constantly rewrites files inside them (indexing, installing plugins), which would otherwise leave the working tree dirty every time you go to commit.
   - `.DS_Store` is macOS Finder metadata, `*.user` is per-developer Rider settings; neither belongs in shared history.

8. **Make the first commit.** Open Rider's **Terminal** tool window (bottom toolbar); it opens at the solution root (`catch-up-platform/`, the folder with the `.sln` file) by default — the same folder step 1 cloned.

   **Note:** check the terminal prompt is at the solution root, not inside `Acme.CatchUp.Platform/`, before running anything below.

   ```
   git checkout -b main
   git config user.name "Your Name"
   git config user.email "your.email@example.com"
   git add .
   git commit -m "chore: initial commit."
   git push -u origin main
   ```

   **Note:**
   - No `git init` here: step 1's clone already set up `.git` and the `origin` remote. But cloning a repo with **no commits yet** doesn't give you a `main` branch to go with them — there's nothing to check out yet, so without `git checkout -b main`, your first commit would land on whatever branch name your own machine's git defaults to (often `master`, depending on `init.defaultBranch`), not GitHub's `main`. Running it now, before anything is staged, means `master` never gets created at all — `main` is the only branch this repo ever has, which is also the name **Git Flow Helper** expects when it runs `Init` next.
   - `git config` without `--global` scopes the identity to just this repo.
   - `git push` alone would fail here with "no upstream branch" the first time; `-u origin main` sets that tracking relationship so plain `git push` works from the next commit on.
   - Ran `git init` somewhere by mistake and ended up with two `.git` folders? See [Appendix: Removing a stray .git folder](#removing-a-stray-git-folder).

9. **Install the Git Flow Helper plugin.**
   - macOS: `Rider → Settings → Plugins → Marketplace → search "Git Flow Helper"` → `Install`
   - Windows: `File → Settings → Plugins → Marketplace → search "Git Flow Helper"` → `Install`

10. **Initialize Git Flow.** Git Flow Helper pushes through Rider's own GitHub connection, not the terminal's. Register **your** account there first, and make sure it's the only one.
    - Get your token, copy what it prints (this is the same token your terminal git already uses):

      ```
      gh auth token
      ```

    - Open `Settings` → `Version Control` → `GitHub` (macOS: `Rider → Settings`; Windows: `File → Settings`).
    - If any account is already listed (a shared machine may still have someone else's), select each one and click `−` to remove it. The list must be empty before you add yours.
    - Click `+` → `Log In with Token...` (not `Log In via GitHub...`, whose browser sign-in produces an OAuth token your organization blocks for third-party apps).
    - Paste the `gh auth token` value, click `Add Account`. Close `Settings`.
    - Click the **Git Flow Helper** widget in the status bar → `Init`.
    - The branch-prefix fields (`Main`, `Develop`, `Feature`, `Release`, `Hotfix`) are pre-filled with sensible defaults; click `OK`.

    This creates a `develop` branch from `main` and pushes it. From here on, `main` is only touched through a Release, never worked on directly.

11. **Set the connection string.** Still in **File System** view, open `appsettings.json` and replace its contents:

   <details>
   <summary>appsettings.json</summary>

   ```json
   {
     "ConnectionStrings": {
       "DefaultConnection": "server=localhost;user=root;password=123456789;database=catch-up-wa"
     },
     "Logging": {
       "LogLevel": {
         "Default": "Information",
         "Microsoft.AspNetCore": "Warning"
       }
     },
     "AllowedHosts": "*"
   }
   ```
   </details>

   Open `appsettings.Development.json` and replace its contents the same way, without the `AllowedHosts` line (not relevant for local development):

   <details>
   <summary>appsettings.Development.json</summary>

   ```json
   {
     "ConnectionStrings": {
       "DefaultConnection": "server=localhost;user=root;password=123456789;database=catch-up-wa"
     },
     "Logging": {
       "LogLevel": {
         "Default": "Information",
         "Microsoft.AspNetCore": "Warning"
       }
     }
   }
   ```
   </details>

   `root` / `123456789` are the lab machines' own MySQL credentials; `catch-up-wa` doesn't need to exist yet — nothing in this guide runs `CREATE DATABASE` by hand, the first EF Core migration creates the schema the first time the app runs (Feature 1). At home, install MySQL yourself and either match these same credentials or edit this connection string to match your own; see [Appendix: If MySQL refuses the connection](#if-mysql-refuses-the-connection) if the app can't reach it.

   Still in **File System** view, add one more file, `appsettings.Production.json`:

   <details>
   <summary>appsettings.Production.json</summary>

   ```json
   {
     "ConnectionStrings": {
       "DefaultConnection": "server=%DATABASE_URL%;user=%DATABASE_USER%;password=%DATABASE_PASSWORD%;database=%DATABASE_SCHEMA%"
     },
     "Logging": {
       "LogLevel": {
         "Default": "Information",
         "Microsoft.AspNetCore": "Warning"
       }
     }
   }
   ```
   </details>

   This profile isn't used anywhere in this guide (everything below runs with `ASPNETCORE_ENVIRONMENT=Development`, the default Rider run profile already sets); it's here so a real deployment has somewhere to read `DATABASE_URL`/`DATABASE_USER`/`DATABASE_PASSWORD`/`DATABASE_SCHEMA` from the environment, `%NAME%`-style, instead of a secret ever being committed.

   ```
   git add .
   git commit -m "chore: configure database connection strings per environment."
   ```

12. **Create `docs/user-stories.md`.** Still in **File System** view: right-click the solution root → `Add` → `File` → type `docs/user-stories.md` → Enter.

   **Tip:** typing the `docs/` prefix creates that folder too.

   <details>
   <summary>docs/user-stories.md</summary>

   ```markdown
   # CatchUp Platform REST API Technical Stories

   ## Overview
   This document contains API-focused technical stories intended for frontend or mobile developers integrating with the catch-up-platform REST API.

   Base resource route: `/api/v1/favorite-sources` (kebab-case route naming convention).

   ## Requirement Traceability Matrix

   | Technical Story | Scenario                        | Implementation                                                                 |
   |------------------|----------------------------------|---------------------------------------------------------------------------------|
   | TS001            | Successful create                | `FavoriteSourceCommandService.Handle(CreateFavoriteSourceCommand)`             |
   | TS001            | Validation error                 | `NewsApiKey`/`SourceId` constructors, `[Required]` on `CreateFavoriteSourceResource` |
   | TS001            | Duplicate favorite                | unique index + `CreateFavoriteSourceError.DuplicateFavoriteSource`             |
   | TS001            | Unexpected server error          | `CreateFavoriteSourceError.UnexpectedError`                                     |
   | TS002            | Found                            | `FavoriteSourcesController.GetFavoriteSourceById`                               |
   | TS002            | Not found                        | `IBaseRepository.FindByIdAsync` → `null` → `404`                               |
   | TS003            | Returns one or more favorites    | `FavoriteSourcesController.GetFavoriteSourceFromQuery`, `IFavoriteSourceQueryService` |
   | TS003            | Returns no favorites             | `FavoriteSourceRepository.FindByNewsApiKeyAsync` (empty list, not `404`)        |
   | TS004            | Favorite exists                  | `FavoriteSourceRepository.FindByNewsApiKeyAndSourceIdAsync`                     |
   | TS004            | Favorite does not exist          | same method → `null` → `404`                                                   |
   | TS005            | Supported language requested     | `LocalizedOpenApiAssembler`, `UseRequestLocalization`                          |
   | TS005            | Unsupported or missing language  | default culture fallback (`en`)                                                |
   | TS006            | Successful delete                | `FavoriteSourceCommandService.Handle(DeleteFavoriteSourceCommand)`             |
   | TS006            | Not found                        | `DeleteFavoriteSourceError.NotFound`                                            |
   | TS006            | Invalid id                       | `DeleteFavoriteSourceCommand` constructor                                      |

   ## Technical Stories

   ### TS001 — Create a favorite source
   As a frontend developer, I want to add a favorite source through the API so that I can implement the creation feature in my application.
   #### Acceptance criteria:
   - Scenario: Successful create
       - Given a request `POST /api/v1/favorite-sources` that includes required fields (`newsApiKey`, `sourceId`) is received
       - When the API validates and persists the new resource successfully
       - Then the API responds with `201 Created`, and returns the created resource representation (`id`, `newsApiKey`, `sourceId`).
   - Scenario: Validation error
       - Given a request `POST /api/v1/favorite-sources` with missing required fields or invalid values is received
       - When the API validation fails
       - Then the API responds with `400 Bad Request` and a localized error message describing the validation failure.
   - Scenario: Duplicate favorite
       - Given a request `POST /api/v1/favorite-sources` for a `newsApiKey` + `sourceId` combination that already exists is received
       - When the API detects the duplicate pair before persistence or during persistence confirmation
       - Then the API responds with `409 Conflict` and an explanatory error payload.
   - Scenario: Unexpected server error
       - Given a request `POST /api/v1/favorite-sources` is received
       - When an unexpected error occurs during persistence
       - Then the API responds with `500 Internal Server Error` and a `ProblemDetails` payload.
   ---
   ### TS002 — Retrieve a favorite source by id
   As a frontend developer, I want to request a favorite source by its `{id}` so that I can implement the details view in my application.
   #### Acceptance criteria:
   - Scenario: Found
       - Given a request `GET /api/v1/favorite-sources/{id}` is received
       - When the API finds the resource
       - Then the API responds with `200 OK` and returns the favorite source resource (`id`, `newsApiKey`, `sourceId`).
   - Scenario: Not found
       - Given a request `GET /api/v1/favorite-sources/{id}` for a non-existent `{id}` is received
       - When the API does not find the resource
       - Then the API responds with `404 Not Found`.
   ---
   ### TS003 — List favorite sources by NewsAPI key
   As a frontend developer, I want to request the API to list favorite sources for a given `{newsApiKey}` so that I can implement the listing feature in my application.
   #### Acceptance criteria:
   - Scenario: Returns one or more favorites
       - Given a request `GET /api/v1/favorite-sources?newsApiKey={newsApiKey}` is received
       - When the API finds one or more matching favorite resources
       - Then the API responds with `200 OK` and returns a non-empty array of favorite source resources, with each resource including `id`, `newsApiKey`, and `sourceId`.
   - Scenario: Returns no favorites
       - Given a request `GET /api/v1/favorite-sources?newsApiKey={newsApiKey}` is received
       - When the API finds no matching resources
       - Then the API responds with `200 OK` and returns an empty array.
   ---
   ### TS004 — Check whether a source is favorite for a NewsAPI key
   As a frontend developer, I want to query the API with `{newsApiKey}` and `{sourceId}` so that I can implement the UI state that indicates whether a source is favorite.
   #### Acceptance criteria:
   - Scenario: Favorite exists
       - Given a request `GET /api/v1/favorite-sources?newsApiKey={newsApiKey}&sourceId={sourceId}` is received
       - When the API finds a matching favorite resource
       - Then the API responds with `200 OK` and returns the matching favorite resource.
   - Scenario: Favorite does not exist
       - Given a request `GET /api/v1/favorite-sources?newsApiKey={newsApiKey}&sourceId={sourceId}` with no match is received
       - When the API does not find a matching resource
       - Then the API responds with `404 Not Found`.
   ---
   ### TS005 — Read the API description in my language
   As a frontend developer, I want the API description document to be served in my preferred language so that I can understand the endpoints and their responses without translating them.
   #### Acceptance criteria:
   - Scenario: Supported language requested
       - Given a request `GET /openapi/v1.json` with an `Accept-Language` header for a supported culture (`en`, `en-US`, `es`, `es-PE`) is received
       - When the API builds the description document
       - Then the API responds with `200 OK` and the operation summaries, descriptions, response descriptions and resource property descriptions are in that language.
   - Scenario: Unsupported or missing language
       - Given a request `GET /openapi/v1.json` without an `Accept-Language` header, or with an unsupported one, is received
       - When the API builds the description document
       - Then the API responds with `200 OK` and the texts are in the default language.
   ---
   ### TS006 — Remove a favorite source
   As a frontend developer, I want to remove a favorite source through the API so that I can implement the "unfavorite" feature in my application.
   #### Acceptance criteria:
   - Scenario: Successful delete
       - Given a request `DELETE /api/v1/favorite-sources/{id}` for an existing `{id}` is received
       - When the API removes the resource
       - Then the API responds with `204 No Content`.
   - Scenario: Not found
       - Given a request `DELETE /api/v1/favorite-sources/{id}` for a non-existent `{id}` is received
       - When the API does not find the resource
       - Then the API responds with `404 Not Found` and a localized error message.
   - Scenario: Invalid id
       - Given a request `DELETE /api/v1/favorite-sources/{id}` with `id` zero or negative is received
       - When the API validates the id
       - Then the API responds with `400 Bad Request`.
   ```
   </details>

   ```
   git add .
   git commit -m "docs: add user stories and requirement traceability matrix."
   ```

13. **Look at the architecture before writing any code.**
   - Real projects rarely start from a blank slate: the course already sets DDD as part of the Definition of Done. What's ahead is learning to read a given architecture and implement it well.
   - Install the **plantuml4idea** plugin so the diagram renders: `Settings` → `Plugins` → `Marketplace` → search `plantuml4idea` → `Install` (macOS: `Rider → Settings`; Windows: `File → Settings`).
   - Still in **File System** view: right-click the solution root → `Add` → `File` → type `docs/class-diagram.puml` → Enter.

   `News` is a single bounded context with one aggregate, `FavoriteSource`. `NewsApiKey` and `SourceId` are value objects validated on construction, never passed around as raw strings past the API boundary. `Shared` holds cross-cutting infrastructure every bounded context in this solution can reuse: `Result<TValue, TError>` for railway-style outcomes, `IBaseRepository`/`IUnitOfWork` for persistence, `IAuditableEntity` for automatic timestamps.

   <details>
   <summary>docs/class-diagram.puml</summary>

   ```
   @startuml

   package "Acme.CatchUp.Platform.Shared.Application.Patterns" {
     abstract class "Result<TValue, TError>" {
       +bool IsSuccess
       +bool IsFailure
     }
   }

   package "Acme.CatchUp.Platform.Shared.Domain.Repositories" {
     interface "IBaseRepository<TEntity>" {
       +Task AddAsync(TEntity entity, CancellationToken)
       +Task<TEntity?> FindByIdAsync(int id, CancellationToken)
       +void Update(TEntity entity)
       +void Remove(TEntity entity)
       +Task<IEnumerable<TEntity>> ListAsync(CancellationToken)
     }
     interface IUnitOfWork {
       +Task CompleteAsync(CancellationToken)
     }
   }

   package "Acme.CatchUp.Platform.Shared.Domain.Model" {
     interface IAuditableEntity {
       +DateTimeOffset? CreatedAt
       +DateTimeOffset? UpdatedAt
     }
   }

   package "Acme.CatchUp.Platform.News.Domain.Model.ValueObjects" {
     class NewsApiKey <<sealed record>> {
       +string Value
       +string MaskedForLogging()
     }
     class SourceId <<sealed record>> {
       +string Value
     }
   }

   package "Acme.CatchUp.Platform.News.Domain.Model.Aggregates" {
     class FavoriteSource {
       +int Id
       +NewsApiKey NewsApiKey
       +SourceId SourceId
       +DateTimeOffset? CreatedAt
       +DateTimeOffset? UpdatedAt
     }
   }

   package "Acme.CatchUp.Platform.News.Domain.Model.Commands" {
     class CreateFavoriteSourceCommand <<record>> {
       +NewsApiKey NewsApiKey
       +SourceId SourceId
     }
     class DeleteFavoriteSourceCommand <<sealed record>> {
       +int Id
     }
   }

   package "Acme.CatchUp.Platform.News.Domain.Model.Queries" {
     class GetFavoriteSourceByIdQuery <<record>> {
       +int Id
     }
     class GetAllFavoriteSourcesByNewsApiKeyQuery <<record>> {
       +NewsApiKey NewsApiKey
     }
     class GetFavoriteSourceByNewsApiKeyAndSourceIdQuery <<record>> {
       +NewsApiKey NewsApiKey
       +SourceId SourceId
     }
   }

   package "Acme.CatchUp.Platform.News.Domain.Repositories" {
     interface IFavoriteSourceRepository {
       +Task<IEnumerable<FavoriteSource>> FindByNewsApiKeyAsync(NewsApiKey, CancellationToken)
       +Task<FavoriteSource?> FindByNewsApiKeyAndSourceIdAsync(NewsApiKey, SourceId, CancellationToken)
     }
   }

   package "Acme.CatchUp.Platform.News.Application.Services" {
     interface IFavoriteSourceCommandService {
       +Task<Result<FavoriteSource, CreateFavoriteSourceError>> Handle(CreateFavoriteSourceCommand, CancellationToken)
       +Task<Result<int, DeleteFavoriteSourceError>> Handle(DeleteFavoriteSourceCommand, CancellationToken)
     }
     interface IFavoriteSourceQueryService {
       +Task<IEnumerable<FavoriteSource>> Handle(GetAllFavoriteSourcesByNewsApiKeyQuery, CancellationToken)
       +Task<FavoriteSource?> Handle(GetFavoriteSourceByNewsApiKeyAndSourceIdQuery, CancellationToken)
       +Task<FavoriteSource?> Handle(GetFavoriteSourceByIdQuery, CancellationToken)
     }
   }

   package "Acme.CatchUp.Platform.News.Interfaces.REST" {
     class FavoriteSourcesController {
       +Task<ActionResult> CreateFavoriteSource(CreateFavoriteSourceResource, CancellationToken)
       +Task<ActionResult> GetFavoriteSourceFromQuery(string, string?, CancellationToken)
       +Task<ActionResult> GetFavoriteSourceById(int, CancellationToken)
       +Task<ActionResult> DeleteFavoriteSource(int, CancellationToken)
     }
   }

   IBaseRepository <|.. IFavoriteSourceRepository
   FavoriteSource ..|> IAuditableEntity
   FavoriteSource --> "1" NewsApiKey : has
   FavoriteSource --> "1" SourceId : has
   FavoriteSourcesController ..> IFavoriteSourceCommandService : uses
   FavoriteSourcesController ..> IFavoriteSourceQueryService : uses
   IFavoriteSourceCommandService ..> "Result<TValue, TError>" : returns
   FavoriteSourcesController ..> CreateFavoriteSourceCommand : obtains
   IFavoriteSourceCommandService ..> CreateFavoriteSourceCommand : handles
   FavoriteSource ..> CreateFavoriteSourceCommand : constructed from
   FavoriteSourcesController ..> DeleteFavoriteSourceCommand : creates
   IFavoriteSourceCommandService ..> DeleteFavoriteSourceCommand : handles
   FavoriteSourcesController ..> GetFavoriteSourceByIdQuery : creates
   FavoriteSourcesController ..> GetAllFavoriteSourcesByNewsApiKeyQuery : creates
   FavoriteSourcesController ..> GetFavoriteSourceByNewsApiKeyAndSourceIdQuery : creates
   IFavoriteSourceQueryService ..> GetFavoriteSourceByIdQuery : handles
   IFavoriteSourceQueryService ..> GetAllFavoriteSourcesByNewsApiKeyQuery : handles
   IFavoriteSourceQueryService ..> GetFavoriteSourceByNewsApiKeyAndSourceIdQuery : handles

   @enduml
   ```
   </details>

   Rider shows a rendered preview beside the source. If it shows an error instead, see [Appendix: If the class diagram doesn't render](#if-the-class-diagram-doesnt-render).

   ```
   git add .
   git commit -m "docs: add class diagram."
   ```

14. **Create `Result<TValue, TError>`.** Switch to **Solution** view: right-click `Acme.CatchUp.Platform` → `Add` → `Class/Interface` → type `Shared/Application/Patterns/Result` → select `Class` → Enter. An abstract `record` with two nested `sealed record` cases, `Success(TValue Value)` and `Failure(TError Error)` — the first railway-style outcome type in this track's C# backend, used by every command from here on.

    <details>
    <summary>Shared/Application/Patterns/Result.cs</summary>

    ```csharp
    namespace Acme.CatchUp.Platform.Shared.Application.Patterns;

    public abstract record Result<TValue, TError>
    {
        public sealed record Success(TValue Value) : Result<TValue, TError>;

        public sealed record Failure(TError Error) : Result<TValue, TError>;

        public bool IsSuccess => this is Success;

        public bool IsFailure => this is Failure;

        public Result<TNext, TError> Map<TNext>(Func<TValue, TNext> onSuccess) =>
            this switch
            {
                Success s => new Result<TNext, TError>.Success(onSuccess(s.Value)),
                Failure f => new Result<TNext, TError>.Failure(f.Error),
                _ => throw new InvalidOperationException("Unknown Result type")
            };

        public TResult Fold<TResult>(Func<TValue, TResult> onSuccess, Func<TError, TResult> onFailure) =>
            this switch
            {
                Success s => onSuccess(s.Value),
                Failure f => onFailure(f.Error),
                _ => throw new InvalidOperationException("Unknown Result type")
            };

        public void Match(Action<TValue> onSuccess, Action<TError> onFailure)
        {
            if (this is Success s)
                onSuccess(s.Value);
            else if (this is Failure f)
                onFailure(f.Error);
        }
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(shared): add result pattern to standardize operation outcomes with success and failure representations."
    ```

15. **Create `IBaseRepository<TEntity>` and `IUnitOfWork`.** Right-click `Acme.CatchUp.Platform` → `Add` → `Class/Interface` → type `Shared/Domain/Repositories/IBaseRepository` → select `Interface` → Enter. Generic CRUD every repository in this course extends: `AddAsync`, `FindByIdAsync`, `Update`, `Remove`, `ListAsync`.

    <details>
    <summary>Shared/Domain/Repositories/IBaseRepository.cs</summary>

    ```csharp
    namespace Acme.CatchUp.Platform.Shared.Domain.Repositories;

    public interface IBaseRepository<TEntity>
    {
        Task AddAsync(TEntity entity, CancellationToken cancellationToken = default);

        Task<TEntity?> FindByIdAsync(int id, CancellationToken cancellationToken = default);

        void Update(TEntity entity);

        void Remove(TEntity entity);

        Task<IEnumerable<TEntity>> ListAsync(CancellationToken cancellationToken = default);
    }
    ```
    </details>

    Same way, create `Shared/Domain/Repositories/IUnitOfWork` as an `Interface`:

    <details>
    <summary>Shared/Domain/Repositories/IUnitOfWork.cs</summary>

    ```csharp
    namespace Acme.CatchUp.Platform.Shared.Domain.Repositories;

    public interface IUnitOfWork
    {
        Task CompleteAsync(CancellationToken cancellationToken = default);
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(shared): add base repository and unit of work contracts."
    ```

16. **Create `IAuditableEntity` and `AuditableEntityInterceptor`.** Create `Shared/Domain/Model/IAuditableEntity` as an `Interface`: any entity implementing it gets `CreatedAt` set once and `UpdatedAt` refreshed on every save, automatically.

    <details>
    <summary>Shared/Domain/Model/IAuditableEntity.cs</summary>

    ```csharp
    namespace Acme.CatchUp.Platform.Shared.Domain.Model;

    public interface IAuditableEntity
    {
        DateTimeOffset? CreatedAt { get; set; }

        DateTimeOffset? UpdatedAt { get; set; }
    }
    ```
    </details>

    Create `Shared/Infrastructure/Persistence/EFC/Interceptors/AuditableEntityInterceptor` as a `Class`, extending EF Core's `SaveChangesInterceptor`:

    <details>
    <summary>Shared/Infrastructure/Persistence/EFC/Interceptors/AuditableEntityInterceptor.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.Shared.Domain.Model;
    using Microsoft.EntityFrameworkCore;
    using Microsoft.EntityFrameworkCore.Diagnostics;

    namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Interceptors;

    public sealed class AuditableEntityInterceptor : SaveChangesInterceptor
    {
        public override InterceptionResult<int> SavingChanges(DbContextEventData eventData, InterceptionResult<int> result)
        {
            ApplyAuditTimestamps(eventData.Context);
            return base.SavingChanges(eventData, result);
        }

        public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
            DbContextEventData eventData,
            InterceptionResult<int> result,
            CancellationToken cancellationToken = default)
        {
            ApplyAuditTimestamps(eventData.Context);
            return base.SavingChangesAsync(eventData, result, cancellationToken);
        }

        private static void ApplyAuditTimestamps(DbContext? context)
        {
            if (context is null) return;

            var now = DateTimeOffset.UtcNow;

            foreach (var entry in context.ChangeTracker.Entries<IAuditableEntity>())
            {
                if (entry.State is EntityState.Added or EntityState.Modified) entry.Entity.UpdatedAt = now;
                if (entry.State == EntityState.Added) entry.Entity.CreatedAt ??= now;
            }
        }
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(shared): add audit timestamp interceptor and marker interface."
    ```

17. **Create the snake-case naming convention.** Create `Shared/Infrastructure/Persistence/EFC/Configuration/Extensions/StringExtensions` as a `Class` (`static`):

    <details>
    <summary>Shared/Infrastructure/Persistence/EFC/Configuration/Extensions/StringExtensions.cs</summary>

    ```csharp
    using Humanizer;

    namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration.Extensions;

    public static class StringExtensions
    {
        public static string ToSnakeCase(this string text)
        {
            return new string(Convert(text.GetEnumerator()).ToArray());

            static IEnumerable<char> Convert(CharEnumerator e)
            {
                if (!e.MoveNext()) yield break;

                yield return char.ToLower(e.Current);

                while (e.MoveNext())
                    if (char.IsUpper(e.Current))
                    {
                        yield return '_';
                        yield return char.ToLower(e.Current);
                    }
                    else
                    {
                        yield return e.Current;
                    }
            }
        }

        public static string ToPlural(this string text)
        {
            return text.Pluralize(false);
        }
    }
    ```
    </details>

    Same folder, create `ModelBuilderExtensions` as a `Class` (`static`): an extension method on `ModelBuilder` that renames every table, column, key, foreign key, and index to snake_case, and pluralizes table names (`FavoriteSource` → `favorite_sources`).

    <details>
    <summary>Shared/Infrastructure/Persistence/EFC/Configuration/Extensions/ModelBuilderExtensions.cs</summary>

    ```csharp
    using Microsoft.EntityFrameworkCore;

    namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration.Extensions;

    public static class ModelBuilderExtensions
    {
        public static void UseSnakeCaseNamingConvention(this ModelBuilder builder)
        {
            foreach (var entity in builder.Model.GetEntityTypes())
            {
                var tableName = entity.GetTableName();
                if (!string.IsNullOrEmpty(tableName)) entity.SetTableName(tableName.ToPlural().ToSnakeCase());

                foreach (var property in entity.GetProperties())
                    property.SetColumnName(property.GetColumnName().ToSnakeCase());

                foreach (var key in entity.GetKeys())
                {
                    var keyName = key.GetName();
                    if (!string.IsNullOrEmpty(keyName)) key.SetName(keyName.ToSnakeCase());
                }

                foreach (var foreignKey in entity.GetForeignKeys())
                {
                    var foreignKeyName = foreignKey.GetConstraintName();
                    if (!string.IsNullOrEmpty(foreignKeyName)) foreignKey.SetConstraintName(foreignKeyName.ToSnakeCase());
                }

                foreach (var index in entity.GetIndexes())
                {
                    var indexDatabaseName = index.GetDatabaseName();
                    if (!string.IsNullOrEmpty(indexDatabaseName)) index.SetDatabaseName(indexDatabaseName.ToSnakeCase());
                }
            }
        }
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(shared): add snake case naming convention for database schema."
    ```

18. **Create the `AppDbContext` skeleton.** Create `Shared/Infrastructure/Persistence/EFC/Configuration/AppDbContext` as a `Class`, extending `DbContext`. No aggregate exists yet, so `OnModelCreating` only wires the snake-case convention for now — Feature 1 adds the `FavoriteSource` mapping into this same method.

    <details>
    <summary>Shared/Infrastructure/Persistence/EFC/Configuration/AppDbContext.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration.Extensions;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Interceptors;
    using Microsoft.EntityFrameworkCore;

    namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;

    public class AppDbContext(DbContextOptions options) : DbContext(options)
    {
        protected override void OnConfiguring(DbContextOptionsBuilder builder)
        {
            builder.AddInterceptors(new AuditableEntityInterceptor());
            base.OnConfiguring(builder);
        }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            base.OnModelCreating(builder);

            builder.UseSnakeCaseNamingConvention();
        }
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(shared): add application database context skeleton."
    ```

19. **Implement `BaseRepository<TEntity>` and `UnitOfWork`.** Create `Shared/Infrastructure/Persistence/EFC/Repositories/BaseRepository` as a `Class`. Every repository in this course extends this one instead of re-implementing the same five methods against EF Core.

    <details>
    <summary>Shared/Infrastructure/Persistence/EFC/Repositories/BaseRepository.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.Shared.Domain.Repositories;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;
    using Microsoft.EntityFrameworkCore;

    namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;

    public class BaseRepository<TEntity>(AppDbContext context) : IBaseRepository<TEntity> where TEntity : class
    {
        protected readonly AppDbContext Context = context;

        public async Task AddAsync(TEntity entity, CancellationToken cancellationToken = default)
        {
            await Context.Set<TEntity>().AddAsync(entity, cancellationToken);
        }

        public async Task<TEntity?> FindByIdAsync(int id, CancellationToken cancellationToken = default)
        {
            return await Context.Set<TEntity>().FindAsync([id], cancellationToken);
        }

        public void Update(TEntity entity)
        {
            Context.Set<TEntity>().Update(entity);
        }

        public void Remove(TEntity entity)
        {
            Context.Set<TEntity>().Remove(entity);
        }

        public async Task<IEnumerable<TEntity>> ListAsync(CancellationToken cancellationToken = default)
        {
            return await Context.Set<TEntity>().ToListAsync(cancellationToken);
        }
    }
    ```
    </details>

    Same way, create `Shared/Infrastructure/Persistence/EFC/Repositories/UnitOfWork` as a `Class`:

    <details>
    <summary>Shared/Infrastructure/Persistence/EFC/Repositories/UnitOfWork.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.Shared.Domain.Repositories;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;

    namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;

    public class UnitOfWork(AppDbContext context) : IUnitOfWork
    {
        public async Task CompleteAsync(CancellationToken cancellationToken = default)
        {
            await context.SaveChangesAsync(cancellationToken);
        }
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(shared): add base repository and unit of work implementations."
    ```

20. **Create the kebab-case route naming convention.** Create `Shared/Infrastructure/Interfaces/ASP/Configuration/Extensions/StringExtensions` as a `Class` (`static`, `partial`):

    <details>
    <summary>Shared/Infrastructure/Interfaces/ASP/Configuration/Extensions/StringExtensions.cs</summary>

    ```csharp
    using System.Text.RegularExpressions;

    namespace Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration.Extensions;

    public static partial class StringExtensions
    {
        public static string ToKebabCase(this string text)
        {
            if (string.IsNullOrEmpty(text)) return text;

            return KebabCaseRegex().Replace(text, "-$1")
                .Trim()
                .ToLower();
        }

        [GeneratedRegex("(?<!^)([A-Z][a-z]|(?<=[a-z])[A-Z])", RegexOptions.Compiled)]
        private static partial Regex KebabCaseRegex();
    }
    ```
    </details>

    One folder up, create `Shared/Infrastructure/Interfaces/ASP/Configuration/KebabCaseRouteNamingConvention` as a `Class`, implementing `IControllerModelConvention`: rewrites `[controller]` route tokens so `FavoriteSourcesController` maps to `/api/v1/favorite-sources` instead of `/api/v1/FavoriteSources`.

    <details>
    <summary>Shared/Infrastructure/Interfaces/ASP/Configuration/KebabCaseRouteNamingConvention.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration.Extensions;
    using Microsoft.AspNetCore.Mvc.ApplicationModels;

    namespace Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration;

    public class KebabCaseRouteNamingConvention : IControllerModelConvention
    {
        public void Apply(ControllerModel controller)
        {
            foreach (var selector in controller.Selectors)
                selector.AttributeRouteModel = ReplaceControllerTemplate(selector, controller.ControllerName);

            foreach (var selector in controller.Actions.SelectMany(a => a.Selectors))
                selector.AttributeRouteModel = ReplaceControllerTemplate(selector, controller.ControllerName);
        }

        private static AttributeRouteModel? ReplaceControllerTemplate(SelectorModel selector, string name)
        {
            return selector.AttributeRouteModel != null
                ? new AttributeRouteModel
                {
                    Template = selector.AttributeRouteModel.Template?.Replace("[controller]", name.ToKebabCase())
                }
                : null;
        }
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(shared): add kebab case route naming convention."
    ```

21. **Create `Resources/SharedResource.cs` and the resource files.** Switch Solution Explorer to **Solution** view: right-click `Acme.CatchUp.Platform` → `Add` → `Class/Interface` → type `Resources/SharedResource` → select `Class` → Enter.

    <details>
    <summary>Resources/SharedResource.cs</summary>

    ```csharp
    namespace Acme.CatchUp.Platform.Resources;

    public class SharedResource
    {
    }
    ```
    </details>

    This class has no members; it only exists as a type `IStringLocalizer<SharedResource>` can point at to find the resource files below. Switch back to **File System** view:
    - Right-click `Resources` → `Add` → `Resources (.resx)` → name it `SharedResource.resx` (the neutral, default culture) → Enter.
    - Delete the `SharedResource.Designer.cs` Rider generates next to it.
    - Open `Acme.CatchUp.Platform.csproj`.
    - Remove the `<EmbeddedResource Update="Resources\SharedResource.resx">` entry Rider just added — the one wrapping `<Generator>` and `<LastGenOutput>`.
    - Delete the surrounding `<ItemGroup>` too, if that entry was the only thing inside it.
    - Repeat the file-creation bullet for `SharedResource.en.resx`, `SharedResource.en-us.resx`, `SharedResource.es.resx`, and `SharedResource.es-pe.resx`.
    - Check `Resources/` and `Acme.CatchUp.Platform.csproj` after each of those four for the same `Designer.cs`/`<EmbeddedResource>` pair. If one appears, remove it the same way as above.
    - Open each `.resx` with Rider's resource editor (or its XML source: right-click → `Open With` → `Text Editor`).
    - Add these three keys to each one — the only ones needed before any feature exists; every feature below adds its own as it goes.

    **Note:** for `SharedResource.resx`, the generated `Designer.cs` declares a class also named `SharedResource`, colliding with the one just created (`CS0101`, duplicate definition) if left in place. For the four culture files, a generated `Designer.cs` usually doesn't collide by name and won't fail the build — remove it anyway, so no file in this project defines a type nothing here uses. Either way, removing the `<EmbeddedResource>` entry stops the file from being regenerated on the next build; the `.resx` stays embedded regardless — SDK-style projects include it by folder/extension convention on their own.

    Open `SharedResource.resx`, `SharedResource.en.resx`, and `SharedResource.en-us.resx`, and add these three entries inside each file's `<root>` element — same English text in all three (the neutral file is the fallback when no `Accept-Language` matches a supported culture, and `en` is this API's default):

    <details>
    <summary>SharedResource.resx / SharedResource.en.resx / SharedResource.en-us.resx (same values)</summary>

    ```xml
    <data name="InvalidModelStateTitle" xml:space="preserve">
        <value>The request contains invalid values.</value>
    </data>
    <data name="UnexpectedServerError" xml:space="preserve">
        <value>Unexpected server error</value>
    </data>
    <data name="UnexpectedErrorProcessingRequest" xml:space="preserve">
        <value>An unexpected error occurred while processing your request.</value>
    </data>
    ```
    </details>

    Open `SharedResource.es.resx` and `SharedResource.es-pe.resx`, and add these three entries inside each file's `<root>` element — same Spanish text in both:

    <details>
    <summary>SharedResource.es.resx / SharedResource.es-pe.resx</summary>

    ```xml
    <data name="InvalidModelStateTitle" xml:space="preserve">
        <value>La solicitud contiene valores no válidos.</value>
    </data>
    <data name="UnexpectedServerError" xml:space="preserve">
        <value>Error inesperado del servidor</value>
    </data>
    <data name="UnexpectedErrorProcessingRequest" xml:space="preserve">
        <value>Ocurrió un error inesperado al procesar su solicitud.</value>
    </data>
    ```
    </details>

    ```
    git add .
    git commit -m "feat(shared): add shared resource localization infrastructure."
    ```

22. **Update `Program.cs`.** Open `Program.cs` and replace the minimal version from step 5 with the full configuration this project needs.

    <details>
    <summary>Program.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.Resources;
    using Acme.CatchUp.Platform.Shared.Domain.Repositories;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.AspNetCore.Mvc.Infrastructure;
    using Microsoft.EntityFrameworkCore;
    using Microsoft.Extensions.Localization;
    using Scalar.AspNetCore;

    var builder = WebApplication.CreateBuilder(args);

    // Configure Lower Case URLs
    builder.Services.AddRouting(options => options.LowercaseUrls = true);

    // Localization Configuration
    builder.Services.AddLocalization();

    // Configure Kebab Case Route Naming Convention
    builder.Services.AddControllers(options => options.Conventions.Add(new KebabCaseRouteNamingConvention()))
        .AddDataAnnotationsLocalization(options =>
            options.DataAnnotationLocalizerProvider = (_, factory) => factory.Create(typeof(SharedResource)))
        .ConfigureApiBehaviorOptions(options =>
        {
            options.InvalidModelStateResponseFactory = context =>
            {
                var localizer = context.HttpContext.RequestServices
                    .GetRequiredService<IStringLocalizer<SharedResource>>();
                var problemDetailsFactory = context.HttpContext.RequestServices
                    .GetRequiredService<ProblemDetailsFactory>();
                var problemDetails = problemDetailsFactory.CreateValidationProblemDetails(
                    context.HttpContext,
                    context.ModelState,
                    statusCode: StatusCodes.Status400BadRequest,
                    title: localizer["InvalidModelStateTitle"].Value);

                var result = new BadRequestObjectResult(problemDetails);
                result.ContentTypes.Add("application/problem+json");
                return result;
            };
        });

    // Register RFC 7807 ProblemDetails payloads for centralized exception handling.
    builder.Services.AddProblemDetails(options =>
    {
        options.CustomizeProblemDetails = context =>
        {
            if (context.ProblemDetails.Status is null or >= 500)
            {
                var localizer = context.HttpContext.RequestServices.GetRequiredService<IStringLocalizer<SharedResource>>();
                context.ProblemDetails.Title ??= localizer["UnexpectedServerError"].Value;
                context.ProblemDetails.Detail ??= localizer["UnexpectedErrorProcessingRequest"].Value;
            }
        };
    });

    // Generate the OpenAPI document with ASP.NET Core's built-in support.
    builder.Services.AddOpenApi();

    // Configure Database Context and route EF logs through the app logger pipeline.
    builder.Services.AddDbContext<AppDbContext>((serviceProvider, options) =>
    {
        var connectionStringTemplate = builder.Configuration.GetConnectionString("DefaultConnection");
        if (string.IsNullOrWhiteSpace(connectionStringTemplate))
            throw new InvalidOperationException("Database connection string is not set in the configuration.");

        var connectionString = Environment.ExpandEnvironmentVariables(connectionStringTemplate);

        options.UseMySQL(connectionString)
            .UseLoggerFactory(serviceProvider.GetRequiredService<ILoggerFactory>())
            .EnableDetailedErrors();

        if (builder.Environment.IsDevelopment())
            options.EnableSensitiveDataLogging();
    });

    // Shared Bounded Context Injection Configuration
    builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

    var app = builder.Build();

    // Configure the HTTP request pipeline.
    app.UseExceptionHandler();

    // Localization must run before Swagger so generated documentation follows the request culture.
    string[] supportedCultures = ["en", "en-US", "es", "es-PE"];
    var localizationOptions = new RequestLocalizationOptions()
        .SetDefaultCulture(supportedCultures[0])
        .AddSupportedCultures(supportedCultures)
        .AddSupportedUICultures(supportedCultures);
    localizationOptions.ApplyCurrentCultureToResponseHeaders = true;
    app.UseRequestLocalization(localizationOptions);

    app.UseHttpsRedirection();

    app.UseAuthorization();

    if (app.Environment.IsDevelopment())
    {
        app.MapOpenApi();
        app.MapScalarApiReference();
    }

    app.MapControllers();

    app.Run();
    ```
    </details>

    **Note:** `MapOpenApi`/`MapScalarApiReference` stay guarded by `IsDevelopment()`, same as the minimal version from step 5 — the document and its interactive page are a build-time aid, not something to serve to the public internet from a real deployment.

    ```
    git add .
    git commit -m "feat(api): configure localization, openapi, problem details, and database context."
    ```

23. **Add `LICENSE.md`.** Switch to **File System** view: right-click the solution root → `Add` → `File` → type `LICENSE.md` → Enter. README's badge links to it next, so it goes in first.

    <details>
    <summary>LICENSE.md</summary>

    ```markdown
    # MIT License

    Copyright © 2026 Web Applications Developer Team

    Permission is hereby granted, free of charge, to any person getting a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or significant portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE, AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES, OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT, OR OTHERWISE, ARISING FROM,
    OUT OF, OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
    ```
    </details>

    ```
    git add .
    git commit -m "docs: add license."
    ```

24. **Add these fourteen Architecture Decision Records to a single `docs/adrs.md` file.** Switch to **File System** view: right-click the `docs` folder → `Add` → `File` → `adrs.md`. The shared-kernel code built above already carries most of these decisions; the few tied to a feature not built yet (value objects, log masking, the delete error type) are specified ahead of writing that feature's code, the same way `docs/user-stories.md` (step 12) and the class diagram (step 13) already describe it — an ADR records a decision, not necessarily a decision already shipped.

   <details>
   <summary>docs/adrs.md</summary>

   ````markdown
   # Architecture Decision Records (ADRs)

   ## About this document

   This document records the significant architectural decisions of the Catch-Up Platform REST API. Each record follows the lightweight format proposed by Michael Nygard: **Context**, **Decision**, **Consequences**, plus a **Status** and **Date**.

   Guidelines for maintaining this log:

   - One decision per record; records are short and focused on the *why*.
   - Records are immutable once **Accepted**. To change a decision, add a new ADR that **Supersedes** the old one and update the old one's status.
   - Statuses: `Proposed`, `Accepted`, `Deprecated`, `Superseded by ADR-XXX`.
   - Number records sequentially and never reuse numbers.

   ## Index

   | ADR | Title | Status |
   |-----|-------|--------|
   | [ADR-001](#adr-001--organize-the-code-by-bounded-context-with-a-layered-ddd-structure) | Organize the code by bounded context with a layered DDD structure | Accepted |
   | [ADR-002](#adr-002--separate-command-and-query-services-cqrs) | Separate command and query services (CQRS) | Accepted |
   | [ADR-003](#adr-003--model-expected-failures-with-a-result-type) | Model expected failures with a `Result` type | Accepted |
   | [ADR-004](#adr-004--use-value-objects-for-newsapikey-and-sourceid) | Use value objects for `NewsApiKey` and `SourceId` | Accepted |
   | [ADR-005](#adr-005--map-rest-resources-with-static-assemblers) | Map REST resources with static assemblers | Accepted |
   | [ADR-006](#adr-006--persist-with-ef-core-on-mysql-using-repository-and-unit-of-work) | Persist with EF Core on MySQL using Repository and Unit of Work | Accepted |
   | [ADR-007](#adr-007--use-snake_case-database-naming-and-automatic-audit-timestamps) | Use `snake_case` database naming and automatic audit timestamps | Accepted |
   | [ADR-008](#adr-008--apply-database-migrations-on-startup) | Apply database migrations on startup | Accepted |
   | [ADR-009](#adr-009--use-lowercase-kebab-case-routes) | Use lowercase kebab-case routes | Accepted |
   | [ADR-010](#adr-010--report-errors-as-rfc-7807-problemdetails) | Report errors as RFC 7807 `ProblemDetails` | Accepted |
   | [ADR-011](#adr-011--localize-user-facing-text-with-resx-resources) | Localize user-facing text with `.resx` resources | Accepted |
   | [ADR-012](#adr-012--mask-news-api-keys-in-logs) | Mask News API keys in logs | Accepted |
   | [ADR-013](#adr-013--use-built-in-openapi-generation-with-scalar-for-api-documentation) | Use built-in OpenAPI generation with Scalar for API documentation | Accepted |
   | [ADR-014](#adr-014--add-a-dedicated-error-type-and-assembler-for-removing-a-favorite-source) | Add a dedicated error type and assembler for removing a favorite source | Accepted |

   ---

   ## ADR-001 — Organize the code by bounded context with a layered DDD structure

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   The API will grow beyond favorite sources. A structure is needed that keeps domain rules independent from frameworks and prevents unrelated features from becoming entangled.

   ### Decision
   Use a single deployable project organized first by bounded context (`News`, `Shared`) and then by layer:

   - `Domain` — aggregates, value objects, commands, queries, repository contracts.
   - `Application` — service contracts (`Services`), implementations (`Internal`), and application errors.
   - `Infrastructure` — persistence and other technical adapters.
   - `Interfaces` — REST controllers, resources, and assemblers.

   Dependencies point inward: `Interfaces` and `Infrastructure` depend on `Application` and `Domain`, never the reverse. Cross-context technical concerns live in `Shared`.

   ### Consequences
   - (+) Domain logic is testable without ASP.NET Core or EF Core.
   - (+) New bounded contexts can be added as sibling folders.
   - (−) More folders and types than a simple CRUD layout.
   - (−) The dependency rule is enforced by convention only; a future split into separate projects or architecture tests could enforce it.

   ---

   ## ADR-002 — Separate command and query services (CQRS)

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   Writes (creating or removing a favorite) have validation, uniqueness rules, and persistence side effects, while reads are simple lookups. Mixing both in one service hides these differences.

   ### Decision
   Expose two application services per aggregate: `IFavoriteSourceCommandService` (handles commands such as `CreateFavoriteSourceCommand` and `DeleteFavoriteSourceCommand`) and `IFavoriteSourceQueryService` (handles queries such as `GetFavoriteSourceByIdQuery`). Commands and queries are immutable records; every handler accepts a `CancellationToken`.

   ### Consequences
   - (+) Clear intent and smaller, focused classes.
   - (+) Read and write paths can evolve independently.
   - (−) Slightly more boilerplate per use case.
   - (−) Both sides still share one database and model — this is CQRS at the service level, not a fully separate read/write store.

   ---

   ## ADR-003 — Model expected failures with a `Result` type

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   Invalid keys, duplicate favorites, a missing id on delete, and similar outcomes are expected business results, not exceptional conditions. Using exceptions for them obscures control flow and is costly.

   ### Decision
   Command handlers return `Result<TValue, TError>` (`Shared.Application.Patterns`) with an error enum specific to the command, such as `CreateFavoriteSourceError` or `DeleteFavoriteSourceError`. The REST layer translates the result into an HTTP response through a dedicated assembler per command (ADR-005). Exceptions are reserved for unexpected failures.

   ### Consequences
   - (+) Failure modes are explicit in the method signature and exhaustively mappable to HTTP status codes.
   - (+) No exception-driven control flow for business rules.
   - (−) Callers must handle both branches; a custom type is maintained instead of using a library.

   ---

   ## ADR-004 — Use value objects for `NewsApiKey` and `SourceId`

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   Both identifiers are strings with validation rules. Passing raw strings around allows invalid values and parameter mix-ups (`newsApiKey` vs `sourceId`).

   ### Decision
   Model them as immutable value objects (`NewsApiKey`, `SourceId`) that validate on construction. EF Core maps them to string columns with value converters. Raw strings are only accepted at the API boundary.

   ### Consequences
   - (+) Invalid values cannot enter the domain; the compiler prevents argument mix-ups.
   - (+) Behavior such as log masking (ADR-012) lives with the data it protects.
   - (−) Requires value converters and mapping code.

   ---

   ## ADR-005 — Map REST resources with static assemblers

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   Domain objects must not be exposed directly through the API, and controllers should stay thin.

   ### Decision
   Use dedicated resources (`CreateFavoriteSourceResource`, `FavoriteSourceResource`) and static assembler classes in `Interfaces/REST/Transform`, named `<Target>From<Source>Assembler` (e.g. `FavoriteSourceResourceFromEntityAssembler`). Each command gets its own result-to-`ActionResult` assembler — `ActionResultFromCreateFavoriteSourceResultAssembler` and `ActionResultFromDeleteFavoriteSourceResultAssembler` are separate classes, not one class overloaded for both, because each maps a different `Result<TValue, TError>` shape.

   ### Consequences
   - (+) The public contract is decoupled from the domain model; controllers contain no mapping logic.
   - (+) Pure static functions are trivial to unit test.
   - (−) No automatic mapping; each new resource or command result needs its own assembler.

   ---

   ## ADR-006 — Persist with EF Core on MySQL using Repository and Unit of Work

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   The service needs relational persistence with a uniqueness constraint on `(news_api_key, source_id)` and a way to commit changes atomically per use case.

   ### Decision
   Use Entity Framework Core with the MySQL provider (`MySql.EntityFrameworkCore`). Define repository interfaces in the domain (`IBaseRepository<T>`, `IFavoriteSourceRepository`), implement them in infrastructure, and commit through `IUnitOfWork.CompleteAsync`. Repositories do not save changes themselves. The connection string comes from configuration and supports environment variable expansion in production so secrets are not committed.

   ### Consequences
   - (+) Domain and application code depend on abstractions, not EF Core.
   - (+) A single commit point per use case.
   - (+) The unique composite index backs the duplicate-favorite rule at the database level.
   - (−) Switching databases requires changing the provider and re-validating migrations.

   ---

   ## ADR-007 — Use `snake_case` database naming and automatic audit timestamps

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   MySQL conventions favor `snake_case`; C# favors `PascalCase`. Audit timestamps should not depend on each handler remembering to set them.

   ### Decision
   Apply a custom `UseSnakeCaseNamingConvention` in `AppDbContext` for tables, columns, and indexes. Entities implementing `IAuditableEntity` receive `CreatedAt`/`UpdatedAt` automatically via `AuditableEntityInterceptor` on `SavingChanges`/`SavingChangesAsync`.

   ### Consequences
   - (+) Idiomatic naming on both sides without per-property attributes.
   - (+) Audit data is consistent and cannot be forgotten.
   - (−) Custom convention code must be maintained; raw SQL must use the snake_case names.

   ---

   ## ADR-008 — Apply database migrations on startup

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   This is a learning/sandbox service that should run with minimal manual setup, matching the lab environment.

   ### Decision
   Call `Database.Migrate()` when the application starts. It is idempotent when the schema is up to date.

   ### Consequences
   - (+) Zero-step local setup: the schema exists after the first run.
   - (−) Not recommended for multi-instance production: concurrent startups can race, and the application needs schema-altering privileges.
   - Revisit (new ADR) before production use: run migrations as a separate deployment step.

   ---

   ## ADR-009 — Use lowercase kebab-case routes

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   Consistent, predictable URLs are part of the API contract (`/api/v1/favorite-sources`).

   ### Decision
   Enable `LowercaseUrls` and register `KebabCaseRouteNamingConvention` so controller names such as `FavoriteSources` become `favorite-sources`. Versioning is expressed in the path (`v1`).

   ### Consequences
   - (+) Uniform URLs without per-controller route boilerplate.
   - (−) Renaming a controller changes its public URL; treat controller names as contract.

   ---

   ## ADR-010 — Report errors as RFC 7807 `ProblemDetails`

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   Clients need a consistent, machine-readable error shape for validation errors and server failures, without leaking internals.

   ### Decision
   Use `AddProblemDetails` with `UseExceptionHandler` for unexpected errors (HTTP 500), and a custom `InvalidModelStateResponseFactory` returning a validation `ProblemDetails` (HTTP 400). Unexpected `Result` failures are mapped to `ProblemDetails` (HTTP 500) by the matching assembler, while duplicates and not-found results return a plain localized message with the matching status (`409`, `404`). Titles and details are localized (ADR-011).

   ### Consequences
   - (+) One error contract across the API, documented in the OpenAPI document via `[ProducesResponseType]`.
   - (+) Exception details are not exposed to clients.
   - (−) Handlers must not throw for expected conditions (see ADR-003).
   - (−) The `404`/`409` responses are plain localized messages rather than `ProblemDetails`; aligning them is a possible follow-up.

   ---

   ## ADR-011 — Localize user-facing text with `.resx` resources

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   Clients in different languages consume the API, and error messages and API documentation should follow the caller's language.

   ### Decision
   Use ASP.NET Core localization with a shared marker class (`SharedResource`) and `.resx` files for the neutral, `en`, `en-US`, `es`, and `es-PE` cultures; `en` is the default. `UseRequestLocalization` selects the culture from the request (e.g. `Accept-Language`) and runs before the OpenAPI document is mapped so generated documentation follows it too (TS005). Data annotation messages use keys (`[Required(ErrorMessage = "RequiredNewsApiKey")]`) resolved from `SharedResource`. All resource files keep the same set of keys.

   Internal log messages and exception messages are intentionally **not** localized, to keep operational logs searchable and consistent.

   ### Consequences
   - (+) Localized errors, validation messages, and API descriptions.
   - (+) Adding a language is a new `.resx` file plus a supported-culture entry.
   - (−) Every new user-facing string requires entries in all resource files.
   - (−) The generated OpenAPI document varies by request culture, so caches must vary on `Accept-Language`.

   ---

   ## ADR-012 — Mask News API keys in logs

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   `NewsApiKey` is a credential for a third-party service. Logging it as plain text would leak it into the logs. Some log statements run before validation, so the value may still be a raw string at that point.

   ### Decision
   Never log a News API key in clear text. Use `NewsApiKey.MaskedForLogging()` on validated instances and the static `NewsApiKey.MaskedForLogging(string?)` for raw input. This applies to all log statements in the command service and controller, and to any future ones.

   ### Consequences
   - (+) Credentials do not reach logs while correlation by partial value remains possible.
   - (−) Relies on developer discipline at each call site; a code review checklist item or analyzer could enforce it.

   ---

   ## ADR-013 — Use built-in OpenAPI generation with Scalar for API documentation

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   The API needs a machine-readable description document and an interactive page to exercise it by hand during development, without adding more third-party dependencies than necessary.

   ### Decision
   Generate the document with `AddOpenApi`/`MapOpenApi` (served at `/openapi/v1.json`), ASP.NET Core's own built-in OpenAPI support, and serve the interactive page with `Scalar.AspNetCore` (`/scalar`) instead of Swagger UI. Localized summaries, descriptions, and schema texts are applied through operation and schema transformers in `LocalizedOpenApiAssembler`. Response metadata uses `[ProducesResponseType]`.

   ### Consequences
   - (+) Fewer third-party dependencies for document generation; first-party support and updates.
   - (+) A modern interactive page with request samples.
   - (−) The transformer API (`AddOperationTransformer`/`AddSchemaTransformer`) is specific to this generator; porting to another one means rewriting `LocalizedOpenApiAssembler`.
   - (−) Exposing the explorer in production is a security and information-disclosure choice; restrict it to Development, or protect it, before a production release (new ADR).

   ---

   ## ADR-014 — Add a dedicated error type and assembler for removing a favorite source

   - **Status:** Accepted
   - **Date:** 2026-10-20

   ### Context
   TS006 (remove a favorite source) wasn't part of the original four technical stories, but it completes the resource's lifecycle: a caller that can mark a source as favorite needs a way to unmark it. Reusing `CreateFavoriteSourceError` for this would mix unrelated failure modes (`DuplicateFavoriteSource` means nothing on a delete) into one enum.

   ### Decision
   Model deletion as its own command (`DeleteFavoriteSourceCommand`, validating a positive id in its own constructor rather than a plain positional `record`), its own error enum (`DeleteFavoriteSourceError { NotFound, UnexpectedError }`), and its own result-to-`ActionResult` assembler (`ActionResultFromDeleteFavoriteSourceResultAssembler`), following the same patterns ADR-003 and ADR-005 already established for create. `IBaseRepository<T>.Remove(TEntity)` (already defined for every repository in this course) needed no change — the existing `FindByIdAsync` lookup doubles as the existence check before removing.

   ### Consequences
   - (+) The lifecycle (create, read, delete) is complete, and every command keeps its own, exhaustive set of failure modes instead of a shared enum with cases that don't apply to every operation.
   - (+) No change to `IBaseRepository<T>` or `IFavoriteSourceRepository` was needed; the feature only added application- and interface-layer code.
   - (−) One more error enum and one more assembler class to maintain, for a feature with fewer failure modes than create.
   ````
   </details>

   ```
   git add .
   git commit -m "docs(adr): add architecture decision records."
   ```

25. **Add `README.md`.** Switch to **File System** view: right-click the solution root → `Add` → `File` → type `README.md` → Enter. Cross-links the ADRs just created, `docs/user-stories.md` (step 12), and `docs/class-diagram.puml` (step 13).

   <details>
   <summary>README.md</summary>

   ````markdown
   # Catch-Up Platform (`catch-up-platform`)

   [![.NET](https://img.shields.io/badge/.NET-10-purple.svg)](https://dotnet.microsoft.com/)
   [![C#](https://img.shields.io/badge/C%23-14-blue.svg)](https://learn.microsoft.com/dotnet/csharp/)
   [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE.md)

   `catch-up-platform` is a small ASP.NET Core service that provides an API to manage users' favorite news sources. The project demonstrates a clean package structure separated into bounded contexts (`news` and `shared`) and follows simple command/query service patterns with Entity Framework Core persistence.

   This API does not proxy NewsAPI.org's content endpoints; it only stores which `(newsApiKey, sourceId)` pairs a caller marked as favorite, so a frontend can ask "is this source one of mine" without re-sending that question to NewsAPI.org itself.

   ---

   ## Domain-Driven Design approach

   The codebase applies a lightweight Domain-Driven Design style to keep the favorite-sources feature easy to evolve:

   - **Bounded contexts**
     - `News`: owns the favorite-source use cases, aggregate, commands, queries, and API contracts (see [ADR-001](docs/adrs.md#adr-001--organize-the-code-by-bounded-context-with-a-layered-ddd-structure)).
     - `Shared`: owns cross-cutting abstractions used by multiple contexts, such as repository contracts, unit of work, auditing, and reusable application patterns.
   - **Domain layer**
     - Contains the core business concepts: `FavoriteSource`, `NewsApiKey`, `SourceId` (see [ADR-004](docs/adrs.md#adr-004--use-value-objects-for-newsapikey-and-sourceid)), and command/query records (see [ADR-002](docs/adrs.md#adr-002--separate-command-and-query-services-cqrs)).
   - **Application layer**
     - Coordinates use cases through command/query services.
     - Returns explicit outcomes with a shared `Result<TValue, TError>` pattern (see [ADR-003](docs/adrs.md#adr-003--model-expected-failures-with-a-result-type) and [ADR-014](docs/adrs.md#adr-014--add-a-dedicated-error-type-and-assembler-for-removing-a-favorite-source)).
   - **Interfaces layer**
     - Exposes the REST API through controllers and ASP.NET Core's built-in OpenAPI metadata (see [ADR-013](docs/adrs.md#adr-013--use-built-in-openapi-generation-with-scalar-for-api-documentation)).
     - Uses static assemblers to translate request resources into commands and application results into HTTP `ActionResult` responses (see [ADR-005](docs/adrs.md#adr-005--map-rest-resources-with-static-assemblers)).
   - **Infrastructure layer**
     - Persists aggregates with Entity Framework Core and MySQL (see [ADR-006](docs/adrs.md#adr-006--persist-with-ef-core-on-mysql-using-repository-and-unit-of-work)).
     - Maps value objects to database columns, applies migrations on startup (see [ADR-008](docs/adrs.md#adr-008--apply-database-migrations-on-startup)), and manages audit timestamps through a save-changes interceptor (see [ADR-007](docs/adrs.md#adr-007--use-snake_case-database-naming-and-automatic-audit-timestamps)).

   ---

   ## Technologies

   - C# 14 and .NET 10 (`net10.0`)
   - ASP.NET Core Web API (REST controllers + built-in OpenAPI generation)
   - Scalar.AspNetCore for interactive API documentation
   - Entity Framework Core 10 + MySQL provider (`MySql.EntityFrameworkCore`)
   - Localization with `.resx` resources (`en`, `en-US`, `es`, `es-PE`) (see [ADR-011](docs/adrs.md#adr-011--localize-user-facing-text-with-resx-resources))
   - Structured logging with `ILogger<T>`
   - PlantUML (architecture diagrams in `docs/`)

   ---

   ## Features

   - API base route: `/api/v1/favorite-sources` (kebab-case route naming convention, see [ADR-009](docs/adrs.md#adr-009--use-lowercase-kebab-case-routes))
   - List favorite sources by `newsApiKey`
   - Retrieve a favorite source by `id`
   - Retrieve a favorite source by `newsApiKey` + `sourceId`
   - Create (persist) a new favorite source
   - Remove a favorite source by `id` (see [ADR-014](docs/adrs.md#adr-014--add-a-dedicated-error-type-and-assembler-for-removing-a-favorite-source))
   - Duplicate detection enforced at application and database level (returns `409 Conflict`)
   - Localized validation and error responses, and the OpenAPI document itself (see [ADR-011](docs/adrs.md#adr-011--localize-user-facing-text-with-resx-resources))
   - Entity Framework naming strategy for `snake_case` identifiers and plural table names (see [ADR-007](docs/adrs.md#adr-007--use-snake_case-database-naming-and-automatic-audit-timestamps))
   - News API keys are masked in all log statements (see [ADR-012](docs/adrs.md#adr-012--mask-news-api-keys-in-logs))

   ---

   ## Environment variables

   | Variable | Used by | Purpose |
   | :--- | :--- | :--- |
   | `DATABASE_URL` | `appsettings.Production.json` | MySQL host, expanded into the connection string. |
   | `DATABASE_USER` | `appsettings.Production.json` | MySQL user. |
   | `DATABASE_PASSWORD` | `appsettings.Production.json` | MySQL password. |
   | `DATABASE_SCHEMA` | `appsettings.Production.json` | MySQL schema/database name. |

   Development and the default profile (`appsettings.json`/`appsettings.Development.json`) read a plain, literal connection string instead — no environment variables needed locally.

   ---

   ## Project Documentation

   | Document | Description |
   | :--- | :--- |
   | [**Architecture Decision Records (ADRs)**](docs/adrs.md) | Fourteen architectural decisions (ADR-001 through ADR-014). |
   | [**User Stories & RTM**](docs/user-stories.md) | Technical stories (TS001-TS006) and Requirement Traceability Matrix. |
   | [**Class Diagram**](docs/class-diagram.puml) | PlantUML class diagram of the `News` and `Shared` bounded contexts. |
   | [**Changelog**](CHANGELOG.md) | Version history and release notes. |
   | [**License**](LICENSE.md) | Project licensing information (MIT). |

   ---

   ## Getting Started

   ### Prerequisites

   - [.NET 10 SDK](https://dotnet.microsoft.com/download) (or later)
   - A reachable MySQL instance

   ### Restore dependencies

   ```bash
   dotnet restore catch-up-platform.sln
   ```

   ### Run the API

   ```bash
   dotnet run --project Acme.CatchUp.Platform
   ```

   ### Database initialization

   The application automatically applies pending EF Core migrations on startup via `Database.Migrate()` (see [ADR-008](docs/adrs.md#adr-008--apply-database-migrations-on-startup)) — wired in Feature 1, once the first migration exists.

   ### Explore the API

   Port in the examples below is this project's own; check `Properties/launchSettings.json` for yours if it's different.

   - Scalar: `http://localhost:5128/scalar`
   - The generated OpenAPI document: `http://localhost:5128/openapi/v1.json`

   ---

   ## Error responses

   The API returns standardized error responses (see [ADR-010](docs/adrs.md#adr-010--report-errors-as-rfc-7807-problemdetails)):

   - **400 Bad Request**: Request validation failed and returns a localized error message.
   - **404 Not Found**: Resource not found.
   - **409 Conflict**: Duplicate `newsApiKey` + `sourceId` pair for known duplicate-pair violations.
   - **500 Internal Server Error**: Unexpected server error (RFC 7807 `ProblemDetails` format).

   ### Build and verify

   ```bash
   dotnet build catch-up-platform.sln
   ```
   ````
   </details>

   ```
   git add .
   git commit -m "docs: add readme with architecture decision links."
   git push
   ```

   **Note:** this is the only `git push` since step 8 — none of the steps after it push on their own, only commit locally; this one sends everything built since then to `origin` in one go.

---
## Create a Favorite Source ([TS001](./user-stories.md))

1. **Start the feature.** Git Flow Helper widget → `Feature` → `Feature Start` → **Feature description** `create-favorite-source` → `OK`.

2. **Create the `FavoriteSource` aggregate.** Switch to **Solution** view: right-click `Acme.CatchUp.Platform` → `Add` → `Class/Interface` → type `News/Domain/Model/Aggregates/FavoriteSource` → select `Class` → Enter.
   - A `protected` parameterless constructor for EF Core — it never runs validation, it just satisfies the ORM.
   - A `public` constructor from `CreateFavoriteSourceCommand`, null-checking the command.

   <details>
   <summary>News/Domain/Model/Aggregates/FavoriteSource.cs</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Domain.Model.Commands;
   using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

   namespace Acme.CatchUp.Platform.News.Domain.Model.Aggregates;

   public partial class FavoriteSource
   {
       protected FavoriteSource()
       {
           NewsApiKey = null!;
           SourceId = null!;
       }

       public FavoriteSource(CreateFavoriteSourceCommand command)
       {
           ArgumentNullException.ThrowIfNull(command);

           NewsApiKey = command.NewsApiKey;
           SourceId = command.SourceId;
       }

       public int Id { get; private set; }

       public NewsApiKey NewsApiKey { get; private set; }

       public SourceId SourceId { get; private set; }
   }
   ```
   </details>

   **Note:** this references `CreateFavoriteSourceCommand` and the `NewsApiKey`/`SourceId` value objects, created in the next four steps — Rider shows them unresolved until then; that's expected, keep going. Starting from the aggregate first (rather than its parts) works here because, unlike JPA, EF Core's mapping lives entirely in `AppDbContext` (step 7) — nothing on this class itself needs the value objects' converters to exist yet.

   ```
   git add .
   git commit -m "feat(favorite-sources): add favorite source aggregate."
   ```

3. **Create the `FavoriteSourceAudit` partial class.** Same folder, `News/Domain/Model/Aggregates/FavoriteSourceAudit` as a `Class`, same name `FavoriteSource` inside, marked `partial` like step 2's file: implements `IAuditableEntity`, populated automatically by `AuditableEntityInterceptor` (Project Setup, step 16). Splitting the file this way keeps the pure domain shape (step 2) separate from the audit concern.

   <details>
   <summary>News/Domain/Model/Aggregates/FavoriteSourceAudit.cs</summary>

   ```csharp
   using Acme.CatchUp.Platform.Shared.Domain.Model;

   namespace Acme.CatchUp.Platform.News.Domain.Model.Aggregates;

   public partial class FavoriteSource : IAuditableEntity
   {
       public DateTimeOffset? CreatedAt { get; set; }

       public DateTimeOffset? UpdatedAt { get; set; }
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(audit): add audit fields for creation and update dates."
   ```

4. **Create `NewsApiKey`.** Same way, `News/Domain/Model/ValueObjects/NewsApiKey` as a `Class`. `sealed record`, validates on construction (non-blank, at most 255 characters), and carries its own log-masking logic — a credential for a third-party service must never show up in a log as plain text.

   <details>
   <summary>News/Domain/Model/ValueObjects/NewsApiKey.cs</summary>

   ```csharp
   namespace Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

   public sealed record NewsApiKey
   {
       private const int MaxLength = 255;

       public NewsApiKey(string value)
       {
           if (string.IsNullOrWhiteSpace(value))
               throw new ArgumentException("NewsApiKey cannot be null, empty, or whitespace.", nameof(value));

           if (value.Length > MaxLength)
               throw new ArgumentException($"NewsApiKey cannot be longer than {MaxLength} characters.", nameof(value));

           Value = value;
       }

       public string Value { get; }

       public override string ToString() => Value;

       public string MaskedForLogging() => MaskedForLogging(Value);

       public static string MaskedForLogging(string? value) =>
           string.IsNullOrEmpty(value)
               ? string.Empty
               : value.Length <= 4
                   ? new string('*', value.Length)
                   : new string('*', value.Length - 4) + value[^4..];
   }
   ```
   </details>

   **Note:** `MaskedForLogging(string?)` (the `static` overload) exists for raw, not-yet-validated input — some log sites run before a `NewsApiKey` is ever constructed, so the value may still be a plain string at that point. ADR-012 in `## Release`.

   ```
   git add .
   git commit -m "feat(domain): add news api key value object with validation and masking."
   ```

5. **Create `SourceId`.** Same way, `News/Domain/Model/ValueObjects/SourceId` as a `Class`. Same shape as `NewsApiKey`, no masking (it isn't a credential).

   <details>
   <summary>News/Domain/Model/ValueObjects/SourceId.cs</summary>

   ```csharp
   namespace Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

   public sealed record SourceId
   {
       private const int MaxLength = 255;

       public SourceId(string value)
       {
           if (string.IsNullOrWhiteSpace(value))
               throw new ArgumentException("SourceId cannot be null, empty, or whitespace.", nameof(value));

           if (value.Length > MaxLength)
               throw new ArgumentException($"SourceId cannot be longer than {MaxLength} characters.", nameof(value));

           Value = value;
       }

       public string Value { get; }

       public override string ToString() => Value;
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(domain): add source id value object with validation."
   ```

6. **Create `CreateFavoriteSourceCommand`.** Same way, `News/Domain/Model/Commands/CreateFavoriteSourceCommand` as a `Class`, then pick `record` in the type dropdown if Rider offers it, or type it as a one-line positional record directly. This resolves the aggregate's constructor from step 2.

   <details>
   <summary>News/Domain/Model/Commands/CreateFavoriteSourceCommand.cs</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

   namespace Acme.CatchUp.Platform.News.Domain.Model.Commands;

   public record CreateFavoriteSourceCommand(NewsApiKey NewsApiKey, SourceId SourceId);
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add create favorite source command."
   ```

7. **Update `AppDbContext.OnModelCreating`.** Switch to **File System** view, open `Shared/Infrastructure/Persistence/EFC/Configuration/AppDbContext.cs` (created empty in Project Setup, step 18). Register `FavoriteSource`:
   - `HasKey`/`ValueGeneratedOnAdd` for `Id`.
   - `HasConversion` for `NewsApiKey`/`SourceId` — value object ↔ `string`, the first time a value object is mapped through `HasConversion` in this course; a short explanation is enough here.
   - A unique index on `(NewsApiKey, SourceId)`, the database-level half of duplicate detection.

   <details>
   <summary>Shared/Infrastructure/Persistence/EFC/Configuration/AppDbContext.cs (full file, OnModelCreating extended)</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
   using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
   using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration.Extensions;
   using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Interceptors;
   using Microsoft.EntityFrameworkCore;

   namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;

   public class AppDbContext(DbContextOptions options) : DbContext(options)
   {
       protected override void OnConfiguring(DbContextOptionsBuilder builder)
       {
           builder.AddInterceptors(new AuditableEntityInterceptor());
           base.OnConfiguring(builder);
       }

       protected override void OnModelCreating(ModelBuilder builder)
       {
           base.OnModelCreating(builder);

           builder.Entity<FavoriteSource>().HasKey(f => f.Id);
           builder.Entity<FavoriteSource>().Property(f => f.Id).IsRequired().ValueGeneratedOnAdd();
           builder.Entity<FavoriteSource>()
               .Property(f => f.SourceId)
               .HasConversion(valueObject => valueObject.Value, value => new SourceId(value))
               .IsRequired();
           builder.Entity<FavoriteSource>()
               .Property(f => f.NewsApiKey)
               .HasConversion(valueObject => valueObject.Value, value => new NewsApiKey(value))
               .IsRequired();
           builder.Entity<FavoriteSource>()
               .HasIndex(f => new { f.NewsApiKey, f.SourceId })
               .IsUnique();

           builder.UseSnakeCaseNamingConvention();
       }
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add unique index for news api key and source id fields."
   ```

8. **Generate the initial migration, and apply it on startup.** The domain model is mapped — nothing about the repository, the application layer, or the controller changes what EF Core generates here, so this is the earliest point a migration can exist. Open Rider's **Terminal** tool window (bottom toolbar); it opens at the solution root (`catch-up-platform/`, the folder with the `.sln` file) by default, same as Project Setup step 8 — not inside `Acme.CatchUp.Platform/`.

   The `dotnet ef` command below needs the EF Core CLI tool, which the .NET SDK doesn't include by default. This single command works whether or not it's already installed — it installs it if missing, or updates it to the latest version if it's there from a previous project, with no error either way:

   ```
   dotnet tool install --global dotnet-ef
   ```

   Now generate the migration:

   ```
   dotnet ef migrations add InitialCreate --project Acme.CatchUp.Platform --output-dir Shared/Infrastructure/Persistence/EFC/Migrations
   ```

   This creates `Shared/Infrastructure/Persistence/EFC/Migrations/*` — `--output-dir` is what puts it there; without it, the CLI's default is a `Migrations` folder at the project root instead. A migration sitting on disk doesn't apply itself — until now there was none to apply, so `Program.cs` had no reason to touch `Database` at all. This is the one new block, right after `var app = builder.Build();`:

   ```csharp
   // Apply pending migrations on startup (safe to call even when schema is up to date)
   using (var scope = app.Services.CreateScope())
   {
       var services = scope.ServiceProvider;
       var context = services.GetRequiredService<AppDbContext>();
       context.Database.Migrate();
   }
   ```

   Everything else is unchanged from Project Setup, step 22. Open `Program.cs` and replace it with the full version below (the block above already in place) rather than pasting it in by hand:

   <details>
   <summary>Program.cs (full file, migration applied on startup)</summary>

   ```csharp
   using Acme.CatchUp.Platform.Resources;
   using Acme.CatchUp.Platform.Shared.Domain.Repositories;
   using Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration;
   using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;
   using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.AspNetCore.Mvc.Infrastructure;
   using Microsoft.EntityFrameworkCore;
   using Microsoft.Extensions.Localization;
   using Scalar.AspNetCore;

   var builder = WebApplication.CreateBuilder(args);

   // Configure Lower Case URLs
   builder.Services.AddRouting(options => options.LowercaseUrls = true);

   // Localization Configuration
   builder.Services.AddLocalization();

   // Configure Kebab Case Route Naming Convention
   builder.Services.AddControllers(options => options.Conventions.Add(new KebabCaseRouteNamingConvention()))
       .AddDataAnnotationsLocalization(options =>
           options.DataAnnotationLocalizerProvider = (_, factory) => factory.Create(typeof(SharedResource)))
       .ConfigureApiBehaviorOptions(options =>
       {
           options.InvalidModelStateResponseFactory = context =>
           {
               var localizer = context.HttpContext.RequestServices
                   .GetRequiredService<IStringLocalizer<SharedResource>>();
               var problemDetailsFactory = context.HttpContext.RequestServices
                   .GetRequiredService<ProblemDetailsFactory>();
               var problemDetails = problemDetailsFactory.CreateValidationProblemDetails(
                   context.HttpContext,
                   context.ModelState,
                   statusCode: StatusCodes.Status400BadRequest,
                   title: localizer["InvalidModelStateTitle"].Value);

               var result = new BadRequestObjectResult(problemDetails);
               result.ContentTypes.Add("application/problem+json");
               return result;
           };
       });

   // Register RFC 7807 ProblemDetails payloads for centralized exception handling.
   builder.Services.AddProblemDetails(options =>
   {
       options.CustomizeProblemDetails = context =>
       {
           if (context.ProblemDetails.Status is null or >= 500)
           {
               var localizer = context.HttpContext.RequestServices.GetRequiredService<IStringLocalizer<SharedResource>>();
               context.ProblemDetails.Title ??= localizer["UnexpectedServerError"].Value;
               context.ProblemDetails.Detail ??= localizer["UnexpectedErrorProcessingRequest"].Value;
           }
       };
   });

   // Generate the OpenAPI document with ASP.NET Core's built-in support.
   builder.Services.AddOpenApi();

   // Configure Database Context and route EF logs through the app logger pipeline.
   builder.Services.AddDbContext<AppDbContext>((serviceProvider, options) =>
   {
       var connectionStringTemplate = builder.Configuration.GetConnectionString("DefaultConnection");
       if (string.IsNullOrWhiteSpace(connectionStringTemplate))
           throw new InvalidOperationException("Database connection string is not set in the configuration.");

       var connectionString = Environment.ExpandEnvironmentVariables(connectionStringTemplate);

       options.UseMySQL(connectionString)
           .UseLoggerFactory(serviceProvider.GetRequiredService<ILoggerFactory>())
           .EnableDetailedErrors();

       if (builder.Environment.IsDevelopment())
           options.EnableSensitiveDataLogging();
   });

   // Shared Bounded Context Injection Configuration
   builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

   var app = builder.Build();

   // Apply pending migrations on startup (safe to call even when schema is up to date)
   using (var scope = app.Services.CreateScope())
   {
       var services = scope.ServiceProvider;
       var context = services.GetRequiredService<AppDbContext>();
       context.Database.Migrate();
   }

   // Configure the HTTP request pipeline.
   app.UseExceptionHandler();

   // Localization must run before Swagger so generated documentation follows the request culture.
   string[] supportedCultures = ["en", "en-US", "es", "es-PE"];
   var localizationOptions = new RequestLocalizationOptions()
       .SetDefaultCulture(supportedCultures[0])
       .AddSupportedCultures(supportedCultures)
       .AddSupportedUICultures(supportedCultures);
   localizationOptions.ApplyCurrentCultureToResponseHeaders = true;
   app.UseRequestLocalization(localizationOptions);

   app.UseHttpsRedirection();

   app.UseAuthorization();

   if (app.Environment.IsDevelopment())
   {
       app.MapOpenApi();
       app.MapScalarApiReference();
   }

   app.MapControllers();

   app.Run();
   ```
   </details>

   Then run the app from the green ▷ next to `main` — `context.Database.Migrate()` applies the migration automatically, creating the `catch-up-wa` schema and the `favorite_sources` table against the connection from Project Setup, step 11. Check it in a MySQL client (or `SHOW TABLES;`/`DESCRIBE favorite_sources;` from a terminal): the table is there, with the right columns, before a single line of repository, application, or controller code exists. It runs on every startup from here on, even once there's nothing new to apply — a safe no-op in that case, not something to guard against.

   ```
   git add .
   git commit -m "feat(migrations): add initial migration with favorite sources table setup."
   ```

9. **Create `CreateFavoriteSourceError`.** Same way, `News/Application/Errors/CreateFavoriteSourceError` as an `Enum`.

   <details>
   <summary>News/Application/Errors/CreateFavoriteSourceError.cs</summary>

   ```csharp
   namespace Acme.CatchUp.Platform.News.Application.Errors;

   public enum CreateFavoriteSourceError
   {
       InvalidNewsApiKey,
       InvalidSourceId,
       DuplicateFavoriteSource,
       UnexpectedError
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add create favorite source error enum."
   ```

10. **Create `IFavoriteSourceRepository`.** Same way, `News/Domain/Repositories/IFavoriteSourceRepository` as an `Interface`, extending `IBaseRepository<FavoriteSource>`. Adds the one lookup every feature in this guide needs sooner or later: by the composite key.

    <details>
    <summary>News/Domain/Repositories/IFavoriteSourceRepository.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
    using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
    using Acme.CatchUp.Platform.Shared.Domain.Repositories;

    namespace Acme.CatchUp.Platform.News.Domain.Repositories;

    public interface IFavoriteSourceRepository : IBaseRepository<FavoriteSource>
    {
        Task<FavoriteSource?> FindByNewsApiKeyAndSourceIdAsync(NewsApiKey newsApiKey, SourceId sourceId,
            CancellationToken cancellationToken = default);
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(favorite-sources): add favorite source repository contract."
    ```

11. **Create `FavoriteSourceRepository`.** Same way, `News/Infrastructure/Persistence/EFC/Repositories/FavoriteSourceRepository` as a `Class`, extending `BaseRepository<FavoriteSource>` (Project Setup, step 19).

    <details>
    <summary>News/Infrastructure/Persistence/EFC/Repositories/FavoriteSourceRepository.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
    using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
    using Acme.CatchUp.Platform.News.Domain.Repositories;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;
    using Microsoft.EntityFrameworkCore;

    namespace Acme.CatchUp.Platform.News.Infrastructure.Persistence.EFC.Repositories;

    public class FavoriteSourceRepository(AppDbContext context)
        : BaseRepository<FavoriteSource>(context), IFavoriteSourceRepository
    {
        public async Task<FavoriteSource?> FindByNewsApiKeyAndSourceIdAsync(NewsApiKey newsApiKey, SourceId sourceId,
            CancellationToken cancellationToken = default)
        {
            return await Context.Set<FavoriteSource>()
                .FirstOrDefaultAsync(f => f.NewsApiKey == newsApiKey && f.SourceId == sourceId,
                    cancellationToken);
        }
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(favorite-sources): add favorite source repository implementation."
    ```

12. **Create `IFavoriteSourceCommandService`.** Switch to **Solution** view: `News/Application/Services/IFavoriteSourceCommandService` as an `Interface`.

    <details>
    <summary>News/Application/Services/IFavoriteSourceCommandService.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
    using Acme.CatchUp.Platform.News.Domain.Model.Commands;
    using Acme.CatchUp.Platform.News.Application.Errors;
    using Acme.CatchUp.Platform.Shared.Application.Patterns;

    namespace Acme.CatchUp.Platform.News.Application.Services;

    public interface IFavoriteSourceCommandService
    {
        Task<Result<FavoriteSource, CreateFavoriteSourceError>> Handle(CreateFavoriteSourceCommand command, CancellationToken cancellationToken = default);
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(favorite-sources): add favorite source command service contract."
    ```

13. **Implement `FavoriteSourceCommandService`.** Same way, `News/Application/Internal/CommandServices/FavoriteSourceCommandService` as a `Class`:
    - Pre-check `FindByNewsApiKeyAndSourceIdAsync` → `Failure(DuplicateFavoriteSource)`, logged as a warning.
    - Otherwise build the aggregate, `AddAsync`, `unitOfWork.CompleteAsync` — all wrapped in `try`/`catch`.
    - `ArgumentException` → `UnexpectedError`.
    - A `DbUpdateException` wrapping a MySQL duplicate-key error (code `1062`) → `DuplicateFavoriteSource` — a second line of defense against a race between the pre-check and the insert.
    - Any other `DbUpdateException`/`Exception` → `UnexpectedError`, logged as an error.

    <details>
    <summary>News/Application/Internal/CommandServices/FavoriteSourceCommandService.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
    using Acme.CatchUp.Platform.News.Application.Errors;
    using Acme.CatchUp.Platform.News.Application.Services;
    using Acme.CatchUp.Platform.News.Domain.Model.Commands;
    using Acme.CatchUp.Platform.News.Domain.Repositories;
    using Acme.CatchUp.Platform.Shared.Domain.Repositories;
    using Acme.CatchUp.Platform.Shared.Application.Patterns;
    using Microsoft.EntityFrameworkCore;

    namespace Acme.CatchUp.Platform.News.Application.Internal.CommandServices;

    public class FavoriteSourceCommandService(
        IFavoriteSourceRepository favoriteSourceRepository,
        IUnitOfWork unitOfWork,
        ILogger<FavoriteSourceCommandService> logger)
        : IFavoriteSourceCommandService
    {
        public async Task<Result<FavoriteSource, CreateFavoriteSourceError>> Handle(CreateFavoriteSourceCommand command,
            CancellationToken cancellationToken = default)
        {
            var existingSource =
                await favoriteSourceRepository.FindByNewsApiKeyAndSourceIdAsync(command.NewsApiKey, command.SourceId,
                    cancellationToken);
            if (existingSource != null)
            {
                logger.LogWarning(
                    "Duplicate favorite source rejected for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                    command.NewsApiKey.MaskedForLogging(),
                    command.SourceId);
                return new Result<FavoriteSource, CreateFavoriteSourceError>.Failure(
                    CreateFavoriteSourceError.DuplicateFavoriteSource);
            }

            try
            {
                var favoriteSource = new FavoriteSource(command);
                await favoriteSourceRepository.AddAsync(favoriteSource, cancellationToken);
                await unitOfWork.CompleteAsync(cancellationToken);
                return new Result<FavoriteSource, CreateFavoriteSourceError>.Success(favoriteSource);
            }
            catch (ArgumentException ex)
            {
                logger.LogWarning(ex,
                    "Invalid arguments while creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                    command.NewsApiKey.MaskedForLogging(),
                    command.SourceId);
                return new Result<FavoriteSource, CreateFavoriteSourceError>.Failure(
                    CreateFavoriteSourceError.UnexpectedError);
            }
            catch (DbUpdateException ex) when (IsDuplicateKeyViolation(ex))
            {
                logger.LogWarning(ex,
                    "Duplicate key violation creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                    command.NewsApiKey.MaskedForLogging(),
                    command.SourceId);
                return new Result<FavoriteSource, CreateFavoriteSourceError>.Failure(
                    CreateFavoriteSourceError.DuplicateFavoriteSource);
            }
            catch (DbUpdateException ex)
            {
                logger.LogError(ex,
                    "Database update failed creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                    command.NewsApiKey.MaskedForLogging(),
                    command.SourceId);
                return new Result<FavoriteSource, CreateFavoriteSourceError>.Failure(
                    CreateFavoriteSourceError.UnexpectedError);
            }
            catch (OperationCanceledException) when (cancellationToken.IsCancellationRequested)
            {
                throw;
            }
            catch (Exception ex)
            {
                logger.LogError(ex,
                    "Unexpected error creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                    command.NewsApiKey.MaskedForLogging(),
                    command.SourceId);
                return new Result<FavoriteSource, CreateFavoriteSourceError>.Failure(
                    CreateFavoriteSourceError.UnexpectedError);
            }
        }

        private static bool IsDuplicateKeyViolation(DbUpdateException exception)
        {
            for (Exception? current = exception; current is not null; current = current.InnerException)
            {
                if (!string.Equals(current.GetType().Name, "MySqlException", StringComparison.Ordinal)) continue;
                var numberProperty = current.GetType().GetProperty("Number");
                if (numberProperty?.PropertyType == typeof(int) &&
                    numberProperty.GetValue(current) is int errorCode &&
                    errorCode == 1062)
                    return true;
            }
            return false;
        }
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(favorite-sources): add conflict handling for duplicate favorite sources creation."
    ```

14. **Create `CreateFavoriteSourceResource`.** Same way, `News/Interfaces/REST/Resources/CreateFavoriteSourceResource` as a `Class`, as a `record`. The request shape; `[Required]` on both fields, each pointing at a message key resolved from `SharedResource` (wired in Project Setup, step 22's `AddDataAnnotationsLocalization`).

    <details>
    <summary>News/Interfaces/REST/Resources/CreateFavoriteSourceResource.cs</summary>

    ```csharp
    using System.ComponentModel.DataAnnotations;

    namespace Acme.CatchUp.Platform.News.Interfaces.REST.Resources;

    public record CreateFavoriteSourceResource(
        [Required(ErrorMessage = "RequiredNewsApiKey")]
        string NewsApiKey,
        [Required(ErrorMessage = "RequiredSourceId")]
        string SourceId);
    ```
    </details>

    ```
    git add .
    git commit -m "feat(favorite-sources): add data annotations for required fields in create favorite source resource."
    ```

15. **Create `FavoriteSourceResource`.** Same way, `News/Interfaces/REST/Resources/FavoriteSourceResource` as a `record`. The response shape, reused by every feature below.

    <details>
    <summary>News/Interfaces/REST/Resources/FavoriteSourceResource.cs</summary>

    ```csharp
    namespace Acme.CatchUp.Platform.News.Interfaces.REST.Resources;

    public record FavoriteSourceResource(
        int Id,
        string NewsApiKey,
        string SourceId);
    ```
    </details>

    ```
    git add .
    git commit -m "feat(favorite-sources): add favorite source response resource."
    ```

16. **Create `CreateFavoriteSourceCommandFromResourceAssembler`.** Same way, `News/Interfaces/REST/Transform/CreateFavoriteSourceCommandFromResourceAssembler` as a `Class` (`static`). This is where an invalid-format `newsApiKey`/`sourceId` throws `ArgumentException`, caught by the controller (step 19).

    <details>
    <summary>News/Interfaces/REST/Transform/CreateFavoriteSourceCommandFromResourceAssembler.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.News.Domain.Model.Commands;
    using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
    using Acme.CatchUp.Platform.News.Interfaces.REST.Resources;

    namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

    public static class CreateFavoriteSourceCommandFromResourceAssembler
    {
        public static CreateFavoriteSourceCommand ToCommandFromResource(CreateFavoriteSourceResource resource) =>
            new(new NewsApiKey(resource.NewsApiKey), new SourceId(resource.SourceId));
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(favorite-sources): add create favorite source command assembler."
    ```

17. **Create `FavoriteSourceResourceFromEntityAssembler`.** Same way, `News/Interfaces/REST/Transform/FavoriteSourceResourceFromEntityAssembler` as a `Class` (`static`). Reused by every feature below that returns a `FavoriteSourceResource`.

    <details>
    <summary>News/Interfaces/REST/Transform/FavoriteSourceResourceFromEntityAssembler.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
    using Acme.CatchUp.Platform.News.Interfaces.REST.Resources;

    namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

    public static class FavoriteSourceResourceFromEntityAssembler
    {
        public static FavoriteSourceResource ToResourceFromEntity(FavoriteSource entity) =>
            new(entity.Id, entity.NewsApiKey.Value, entity.SourceId.Value);
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(favorite-sources): add favorite source resource assembler."
    ```

18. **Create `ActionResultFromCreateFavoriteSourceResultAssembler`.** Same way, `News/Interfaces/REST/Transform/ActionResultFromCreateFavoriteSourceResultAssembler` as a `Class` (`static`):
    - `Success` → `201` via `Created`, building the new resource's location from a template string.
    - `Failure` → `409` for `DuplicateFavoriteSource`, `500` `ProblemDetails` otherwise.
    - `CreatedAtAction` isn't an option yet — it resolves the location through the route table at request time, and `GetFavoriteSourceById` (Feature 2) has no route to resolve until that action exists. Calling it this early throws `InvalidOperationException: No route matches the supplied values` on every successful create, not a compile error.
    - Feature 2 switches this to `CreatedAtAction` once that route exists.

    <details>
    <summary>News/Interfaces/REST/Transform/ActionResultFromCreateFavoriteSourceResultAssembler.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.News.Application.Errors;
    using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
    using Acme.CatchUp.Platform.Resources;
    using Acme.CatchUp.Platform.Shared.Application.Patterns;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Localization;

    namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

    public static class ActionResultFromCreateFavoriteSourceResultAssembler
    {
        public static ActionResult ToActionResultFromCreateFavoriteSourceResult(
            Result<FavoriteSource, CreateFavoriteSourceError> result,
            ControllerBase controller,
            IStringLocalizer<SharedResource> localizer,
            string createdLocationTemplate) =>
            result switch
            {
                Result<FavoriteSource, CreateFavoriteSourceError>.Success success =>
                    controller.Created(string.Format(createdLocationTemplate, success.Value.Id),
                        FavoriteSourceResourceFromEntityAssembler.ToResourceFromEntity(success.Value)),

                Result<FavoriteSource, CreateFavoriteSourceError>.Failure failure =>
                    failure.Error switch
                    {
                        CreateFavoriteSourceError.DuplicateFavoriteSource =>
                            controller.Conflict(localizer["NewsFavoriteSourceDuplicated"].Value),

                        CreateFavoriteSourceError.UnexpectedError =>
                            controller.Problem(
                                title: localizer["UnexpectedServerError"].Value,
                                detail: localizer["UnexpectedErrorCreatingFavoriteSource"].Value,
                                statusCode: 500),

                        _ => controller.Problem(
                            title: localizer["UnexpectedServerError"].Value,
                            detail: localizer["UnexpectedErrorProcessingRequest"].Value,
                            statusCode: 500)
                    },

                _ => controller.Problem(
                    title: localizer["UnexpectedServerError"].Value,
                    detail: localizer["UnexpectedErrorProcessingRequest"].Value,
                    statusCode: 500)
            };
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(favorite-sources): add action result assembler for create favorite source result."
    ```

19. **Create `FavoriteSourcesController`.** Same way, `News/Interfaces/REST/FavoriteSourcesController` as a `Class`, extending `ControllerBase`:
    - `[ApiController] [Route("api/v1/[controller]")]` — kebab-cased to `/api/v1/favorite-sources` by the convention from Project Setup, step 20.
    - `[HttpPost] CreateFavoriteSource(...)`: assemble the command → `favoriteSourceCommandService.Handle` → assemble the action result.
    - `catch (ArgumentException)` → `BadRequest(localizer["InvalidFavoriteSourceRequest"])`.
    - `catch (Exception)` → `Problem(..., 500)`.
    - `FavoriteSourceLocationTemplate` holds the route this controller already answers to (`[Route("api/v1/[controller]")]`, kebab-cased) plus an `{0}` placeholder for the new id — Feature 2 deletes it once `CreatedAtAction` can resolve the real route instead.

    Every action below also takes an `acceptLanguage` parameter, bound with `[FromHeader(Name = "Accept-Language")]`. `UseRequestLocalization` (Project Setup, step 22) already reads `Accept-Language` on its own to pick the response culture — this parameter doesn't change that — but declaring it explicitly makes Scalar's "Try it" panel show an editable field for it, so step 26 can test the Spanish messages without leaving Scalar for Postman.

    <details>
    <summary>News/Interfaces/REST/FavoriteSourcesController.cs</summary>

    ```csharp
    using System.Net.Mime;
    using Acme.CatchUp.Platform.News.Application.Services;
    using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
    using Acme.CatchUp.Platform.News.Interfaces.REST.Resources;
    using Acme.CatchUp.Platform.News.Interfaces.REST.Transform;
    using Acme.CatchUp.Platform.Resources;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Localization;

    namespace Acme.CatchUp.Platform.News.Interfaces.REST;

    [ApiController]
    [Route("api/v1/[controller]")]
    [Produces(MediaTypeNames.Application.Json)]
    [Tags("Favorite Sources")]
    public class FavoriteSourcesController(
        IFavoriteSourceCommandService favoriteSourceCommandService,
        IStringLocalizer<SharedResource> localizer,
        ILogger<FavoriteSourcesController> logger)
        : ControllerBase
    {
        private const string FavoriteSourceLocationTemplate = "/api/v1/favorite-sources/{0}";

        [HttpPost]
        [ProducesResponseType(typeof(FavoriteSourceResource), StatusCodes.Status201Created)]
        [ProducesResponseType(typeof(string), StatusCodes.Status400BadRequest)]
        [ProducesResponseType(typeof(string), StatusCodes.Status409Conflict)]
        [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status500InternalServerError)]
        public async Task<ActionResult> CreateFavoriteSource([FromBody] CreateFavoriteSourceResource resource,
            [FromHeader(Name = "Accept-Language")] string? acceptLanguage,
            CancellationToken cancellationToken)
        {
            try
            {
                var createFavoriteSourceCommand =
                    CreateFavoriteSourceCommandFromResourceAssembler.ToCommandFromResource(resource);
                var result = await favoriteSourceCommandService.Handle(createFavoriteSourceCommand, cancellationToken);
                return ActionResultFromCreateFavoriteSourceResultAssembler.ToActionResultFromCreateFavoriteSourceResult(
                    result,
                    this,
                    localizer,
                    FavoriteSourceLocationTemplate);
            }
            catch (ArgumentException ex)
            {
                logger.LogWarning(ex,
                    "Validation failed while creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                    NewsApiKey.MaskedForLogging(resource.NewsApiKey),
                    resource.SourceId);
                return BadRequest(localizer["InvalidFavoriteSourceRequest"].Value);
            }
            catch (Exception ex)
            {
                logger.LogError(ex,
                    "Unexpected error while creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                    NewsApiKey.MaskedForLogging(resource.NewsApiKey),
                    resource.SourceId);

                return Problem(
                    title: localizer["UnexpectedServerError"].Value,
                    detail: localizer["UnexpectedErrorCreatingFavoriteSource"].Value,
                    statusCode: 500);
            }
        }
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(favorite-sources): add favorite sources controller."
    ```

20. **Add this feature's resource keys to every `SharedResource` file.** Switch to **File System** view, open each of the five `.resx` files from Project Setup step 21 and add these entries, alongside the three already there.

    English (`SharedResource.resx`, `SharedResource.en.resx`, `SharedResource.en-us.resx`):

    <details>
    <summary>Feature 1 resx entries (English)</summary>

    ```xml
    <data name="RequiredNewsApiKey" xml:space="preserve">
        <value>News API key is required.</value>
    </data>
    <data name="RequiredSourceId" xml:space="preserve">
        <value>Source ID is required.</value>
    </data>
    <data name="NewsFavoriteSourceDuplicated" xml:space="preserve">
        <value>Favorite source with this SourceId already exists for the given NewsApiKey</value>
    </data>
    <data name="UnexpectedErrorCreatingFavoriteSource" xml:space="preserve">
        <value>An unexpected error occurred while creating the favorite source.</value>
    </data>
    <data name="InvalidFavoriteSourceRequest" xml:space="preserve">
        <value>The favorite source request contains invalid values.</value>
    </data>
    <data name="CreateFavoriteSourceSummary" xml:space="preserve">
        <value>Create a favorite source</value>
    </data>
    <data name="CreateFavoriteSourceDescription" xml:space="preserve">
        <value>Create a favorite source using a News API key and source ID.</value>
    </data>
    <data name="CreateFavoriteSourceCreated" xml:space="preserve">
        <value>The favorite source was created.</value>
    </data>
    <data name="CreateFavoriteSourceBadRequest" xml:space="preserve">
        <value>The request payload is invalid.</value>
    </data>
    <data name="CreateFavoriteSourceConflict" xml:space="preserve">
        <value>The favorite source already exists.</value>
    </data>
    <data name="CreateFavoriteSourceServerError" xml:space="preserve">
        <value>An unexpected server error occurred.</value>
    </data>
    <data name="CreateFavoriteSourceSchema" xml:space="preserve">
        <value>Request payload for creating a favorite source.</value>
    </data>
    <data name="CreateFavoriteSourceNewsApiKey" xml:space="preserve">
        <value>The News API key provided by NewsAPI.</value>
    </data>
    <data name="CreateFavoriteSourceSourceId" xml:space="preserve">
        <value>The source identifier provided by NewsAPI.</value>
    </data>
    <data name="FavoriteSourceSchema" xml:space="preserve">
        <value>A favorite source returned by the API.</value>
    </data>
    <data name="FavoriteSourceId" xml:space="preserve">
        <value>The server-generated favorite source identifier.</value>
    </data>
    <data name="FavoriteSourceNewsApiKey" xml:space="preserve">
        <value>The News API key associated with the favorite source.</value>
    </data>
    <data name="FavoriteSourceSourceId" xml:space="preserve">
        <value>The source identifier from NewsAPI.</value>
    </data>
    ```
    </details>

    Spanish (`SharedResource.es.resx`, `SharedResource.es-pe.resx`):

    <details>
    <summary>Feature 1 resx entries (Spanish)</summary>

    ```xml
    <data name="RequiredNewsApiKey" xml:space="preserve">
        <value>La clave de News API es obligatoria.</value>
    </data>
    <data name="RequiredSourceId" xml:space="preserve">
        <value>El ID de la fuente es obligatorio.</value>
    </data>
    <data name="NewsFavoriteSourceDuplicated" xml:space="preserve">
        <value>Favorite source con este SourceId ya existe para el NewsApiKey brindado</value>
    </data>
    <data name="UnexpectedErrorCreatingFavoriteSource" xml:space="preserve">
        <value>Ocurrió un error inesperado al crear la fuente favorita.</value>
    </data>
    <data name="InvalidFavoriteSourceRequest" xml:space="preserve">
        <value>La solicitud de fuente favorita contiene valores no válidos.</value>
    </data>
    <data name="CreateFavoriteSourceSummary" xml:space="preserve">
        <value>Crear una fuente favorita</value>
    </data>
    <data name="CreateFavoriteSourceDescription" xml:space="preserve">
        <value>Crea una fuente favorita usando una clave de News API y el ID de la fuente.</value>
    </data>
    <data name="CreateFavoriteSourceCreated" xml:space="preserve">
        <value>Se creó la fuente favorita.</value>
    </data>
    <data name="CreateFavoriteSourceBadRequest" xml:space="preserve">
        <value>El cuerpo de la solicitud no es válido.</value>
    </data>
    <data name="CreateFavoriteSourceConflict" xml:space="preserve">
        <value>La fuente favorita ya existe.</value>
    </data>
    <data name="CreateFavoriteSourceServerError" xml:space="preserve">
        <value>Ocurrió un error inesperado en el servidor.</value>
    </data>
    <data name="CreateFavoriteSourceSchema" xml:space="preserve">
        <value>Datos de la solicitud para crear una fuente favorita.</value>
    </data>
    <data name="CreateFavoriteSourceNewsApiKey" xml:space="preserve">
        <value>La clave de News API proporcionada por NewsAPI.</value>
    </data>
    <data name="CreateFavoriteSourceSourceId" xml:space="preserve">
        <value>El identificador de la fuente proporcionado por NewsAPI.</value>
    </data>
    <data name="FavoriteSourceSchema" xml:space="preserve">
        <value>Una fuente favorita devuelta por la API.</value>
    </data>
    <data name="FavoriteSourceId" xml:space="preserve">
        <value>El identificador de la fuente favorita generado por el servidor.</value>
    </data>
    <data name="FavoriteSourceNewsApiKey" xml:space="preserve">
        <value>La clave de News API asociada con la fuente favorita.</value>
    </data>
    <data name="FavoriteSourceSourceId" xml:space="preserve">
        <value>El identificador de la fuente de NewsAPI.</value>
    </data>
    ```
    </details>

    ```
    git add .
    git commit -m "feat(i18n): add localized messages for create favorite source."
    ```

21. **Create the `LocalizedOpenApiAssembler` skeleton.** Same way, `News/Interfaces/REST/Transform/LocalizedOpenApiAssembler` as a `Class` (`static`): `AddOpenApi`'s transformers call into this to localize summaries, descriptions, and response texts. No operation exists yet, so `OperationResources` starts empty — this step fills in the first entry next.

    <details>
    <summary>News/Interfaces/REST/Transform/LocalizedOpenApiAssembler.cs</summary>

    ```csharp
    using Acme.CatchUp.Platform.Resources;
    using Microsoft.AspNetCore.OpenApi;
    using Microsoft.AspNetCore.Mvc.Controllers;
    using Microsoft.Extensions.Localization;
    using Microsoft.OpenApi;

    namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

    public static class LocalizedOpenApiAssembler
    {
        private static readonly Dictionary<string, (string Summary, string Description,
            IReadOnlyDictionary<string, string> Responses)> OperationResources =
            new Dictionary<string, (string, string, IReadOnlyDictionary<string, string>)>();

        public static Task TransformOperationAsync(
            OpenApiOperation operation,
            OpenApiOperationTransformerContext context,
            CancellationToken cancellationToken)
        {
            if (context.Description.ActionDescriptor is not ControllerActionDescriptor action ||
                !OperationResources.TryGetValue(action.ActionName, out var resources))
                return Task.CompletedTask;

            var localizer = context.ApplicationServices.GetRequiredService<IStringLocalizer<SharedResource>>();
            operation.OperationId = action.ActionName;
            operation.Summary = localizer[resources.Summary].Value;
            operation.Description = localizer[resources.Description].Value;
            if (operation.Responses is null) return Task.CompletedTask;

            foreach (var (statusCode, resourceKey) in resources.Responses)
            {
                if (operation.Responses.TryGetValue(statusCode, out var response) &&
                    response is OpenApiResponse openApiResponse)
                    openApiResponse.Description = localizer[resourceKey].Value;
            }

            return Task.CompletedTask;
        }

        public static Task TransformSchemaAsync(
            OpenApiSchema schema,
            OpenApiSchemaTransformerContext context,
            CancellationToken cancellationToken)
        {
            var localizer = context.ApplicationServices.GetRequiredService<IStringLocalizer<SharedResource>>();
            return Task.CompletedTask;
        }
    }
    ```
    </details>

    **Note:** `TransformSchemaAsync` does nothing yet — it only resolves the localizer so the method compiles. Step 22 below adds the first `if (type == ...)` branch once `CreateFavoriteSourceResource`/`FavoriteSourceResource` exist.

    ```
    git add .
    git commit -m "feat(news): add localized openapi assembler skeleton."
    ```

22. **Wire the OpenAPI transformers into `Program.cs`.** The transformers created above do nothing until `AddOpenApi` is told to call them. One new `using` at the top of the file, and the bare `AddOpenApi()` call from step 8 changes to:

    ```csharp
    builder.Services.AddOpenApi(options =>
    {
        options.AddOperationTransformer(LocalizedOpenApiAssembler.TransformOperationAsync);
        options.AddSchemaTransformer(LocalizedOpenApiAssembler.TransformSchemaAsync);
    });
    ```

    Everything else is unchanged. Open `Program.cs` and replace it with the full file below (the change above already in place) rather than editing it by hand:

    <details>
    <summary>Program.cs (full file, OpenAPI transformers wired in)</summary>

    ```csharp
    using Acme.CatchUp.Platform.News.Interfaces.REST.Transform;
    using Acme.CatchUp.Platform.Resources;
    using Acme.CatchUp.Platform.Shared.Domain.Repositories;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.AspNetCore.Mvc.Infrastructure;
    using Microsoft.EntityFrameworkCore;
    using Microsoft.Extensions.Localization;
    using Scalar.AspNetCore;

    var builder = WebApplication.CreateBuilder(args);

    // Configure Lower Case URLs
    builder.Services.AddRouting(options => options.LowercaseUrls = true);

    // Localization Configuration
    builder.Services.AddLocalization();

    // Configure Kebab Case Route Naming Convention
    builder.Services.AddControllers(options => options.Conventions.Add(new KebabCaseRouteNamingConvention()))
        .AddDataAnnotationsLocalization(options =>
            options.DataAnnotationLocalizerProvider = (_, factory) => factory.Create(typeof(SharedResource)))
        .ConfigureApiBehaviorOptions(options =>
        {
            options.InvalidModelStateResponseFactory = context =>
            {
                var localizer = context.HttpContext.RequestServices
                    .GetRequiredService<IStringLocalizer<SharedResource>>();
                var problemDetailsFactory = context.HttpContext.RequestServices
                    .GetRequiredService<ProblemDetailsFactory>();
                var problemDetails = problemDetailsFactory.CreateValidationProblemDetails(
                    context.HttpContext,
                    context.ModelState,
                    statusCode: StatusCodes.Status400BadRequest,
                    title: localizer["InvalidModelStateTitle"].Value);

                var result = new BadRequestObjectResult(problemDetails);
                result.ContentTypes.Add("application/problem+json");
                return result;
            };
        });

    // Register RFC 7807 ProblemDetails payloads for centralized exception handling.
    builder.Services.AddProblemDetails(options =>
    {
        options.CustomizeProblemDetails = context =>
        {
            if (context.ProblemDetails.Status is null or >= 500)
            {
                var localizer = context.HttpContext.RequestServices.GetRequiredService<IStringLocalizer<SharedResource>>();
                context.ProblemDetails.Title ??= localizer["UnexpectedServerError"].Value;
                context.ProblemDetails.Detail ??= localizer["UnexpectedErrorProcessingRequest"].Value;
            }
        };
    });

    // Generate the OpenAPI document with ASP.NET Core's built-in support.
    builder.Services.AddOpenApi(options =>
    {
        options.AddOperationTransformer(LocalizedOpenApiAssembler.TransformOperationAsync);
        options.AddSchemaTransformer(LocalizedOpenApiAssembler.TransformSchemaAsync);
    });

    // Configure Database Context and route EF logs through the app logger pipeline.
    builder.Services.AddDbContext<AppDbContext>((serviceProvider, options) =>
    {
        var connectionStringTemplate = builder.Configuration.GetConnectionString("DefaultConnection");
        if (string.IsNullOrWhiteSpace(connectionStringTemplate))
            throw new InvalidOperationException("Database connection string is not set in the configuration.");

        var connectionString = Environment.ExpandEnvironmentVariables(connectionStringTemplate);

        options.UseMySQL(connectionString)
            .UseLoggerFactory(serviceProvider.GetRequiredService<ILoggerFactory>())
            .EnableDetailedErrors();

        if (builder.Environment.IsDevelopment())
            options.EnableSensitiveDataLogging();
    });

    // Shared Bounded Context Injection Configuration
    builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

    var app = builder.Build();

    // Apply pending migrations on startup (safe to call even when schema is up to date)
    using (var scope = app.Services.CreateScope())
    {
        var services = scope.ServiceProvider;
        var context = services.GetRequiredService<AppDbContext>();
        context.Database.Migrate();
    }

    // Configure the HTTP request pipeline.
    app.UseExceptionHandler();

    // Localization must run before Swagger so generated documentation follows the request culture.
    string[] supportedCultures = ["en", "en-US", "es", "es-PE"];
    var localizationOptions = new RequestLocalizationOptions()
        .SetDefaultCulture(supportedCultures[0])
        .AddSupportedCultures(supportedCultures)
        .AddSupportedUICultures(supportedCultures);
    localizationOptions.ApplyCurrentCultureToResponseHeaders = true;
    app.UseRequestLocalization(localizationOptions);

    app.UseHttpsRedirection();

    app.UseAuthorization();

    if (app.Environment.IsDevelopment())
    {
        app.MapOpenApi();
        app.MapScalarApiReference();
    }

    app.MapControllers();

    app.Run();
    ```
    </details>

    ```
    git add .
    git commit -m "feat(news): wire localized openapi transformers into the pipeline."
    ```

23. **Localize the create operation's OpenAPI entry.** Open `News/Interfaces/REST/Transform/LocalizedOpenApiAssembler.cs` (step 21 above) and fill in `OperationResources` and the schema switch, both empty until now.

    <details>
    <summary>LocalizedOpenApiAssembler.cs (OperationResources, first entry)</summary>

    ```csharp
    private static readonly Dictionary<string, (string Summary, string Description,
        IReadOnlyDictionary<string, string> Responses)> OperationResources =
        new Dictionary<string, (string, string, IReadOnlyDictionary<string, string>)>
        {
            ["CreateFavoriteSource"] = (
                "CreateFavoriteSourceSummary",
                "CreateFavoriteSourceDescription",
                new Dictionary<string, string>
                {
                    ["201"] = "CreateFavoriteSourceCreated",
                    ["400"] = "CreateFavoriteSourceBadRequest",
                    ["409"] = "CreateFavoriteSourceConflict",
                    ["500"] = "CreateFavoriteSourceServerError"
                })
        };
    ```
    </details>

    <details>
    <summary>LocalizedOpenApiAssembler.cs (TransformSchemaAsync, first branches)</summary>

    ```csharp
    public static Task TransformSchemaAsync(
        OpenApiSchema schema,
        OpenApiSchemaTransformerContext context,
        CancellationToken cancellationToken)
    {
        var localizer = context.ApplicationServices.GetRequiredService<IStringLocalizer<SharedResource>>();
        var type = context.JsonTypeInfo.Type;
        if (type == typeof(CreateFavoriteSourceResource))
            schema.Description = localizer["CreateFavoriteSourceSchema"].Value;
        else if (type == typeof(FavoriteSourceResource))
            schema.Description = localizer["FavoriteSourceSchema"].Value;

        var propertyResourceKey = context.JsonPropertyInfo?.Name switch
        {
            "id" => "FavoriteSourceId",
            "newsApiKey" => "FavoriteSourceNewsApiKey",
            "sourceId" => "FavoriteSourceSourceId",
            _ => null
        };

        if (propertyResourceKey is not null)
            schema.Description = localizer[propertyResourceKey].Value;

        return Task.CompletedTask;
    }
    ```
    </details>

    Plus one new `using` at the top of the file:

    ```csharp
    using Acme.CatchUp.Platform.News.Interfaces.REST.Resources;
    ```

    Everything else is unchanged from step 21's skeleton. Open `LocalizedOpenApiAssembler.cs` and replace it with the full file below (all three changes above already in place) rather than editing it by hand:

    <details>
    <summary>LocalizedOpenApiAssembler.cs (full file, create operation localized)</summary>

    ```csharp
    using Acme.CatchUp.Platform.News.Interfaces.REST.Resources;
    using Acme.CatchUp.Platform.Resources;
    using Microsoft.AspNetCore.OpenApi;
    using Microsoft.AspNetCore.Mvc.Controllers;
    using Microsoft.Extensions.Localization;
    using Microsoft.OpenApi;

    namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

    public static class LocalizedOpenApiAssembler
    {
        private static readonly Dictionary<string, (string Summary, string Description,
            IReadOnlyDictionary<string, string> Responses)> OperationResources =
            new Dictionary<string, (string, string, IReadOnlyDictionary<string, string>)>
            {
                ["CreateFavoriteSource"] = (
                    "CreateFavoriteSourceSummary",
                    "CreateFavoriteSourceDescription",
                    new Dictionary<string, string>
                    {
                        ["201"] = "CreateFavoriteSourceCreated",
                        ["400"] = "CreateFavoriteSourceBadRequest",
                        ["409"] = "CreateFavoriteSourceConflict",
                        ["500"] = "CreateFavoriteSourceServerError"
                    })
            };

        public static Task TransformOperationAsync(
            OpenApiOperation operation,
            OpenApiOperationTransformerContext context,
            CancellationToken cancellationToken)
        {
            if (context.Description.ActionDescriptor is not ControllerActionDescriptor action ||
                !OperationResources.TryGetValue(action.ActionName, out var resources))
                return Task.CompletedTask;

            var localizer = context.ApplicationServices.GetRequiredService<IStringLocalizer<SharedResource>>();
            operation.OperationId = action.ActionName;
            operation.Summary = localizer[resources.Summary].Value;
            operation.Description = localizer[resources.Description].Value;
            if (operation.Responses is null) return Task.CompletedTask;

            foreach (var (statusCode, resourceKey) in resources.Responses)
            {
                if (operation.Responses.TryGetValue(statusCode, out var response) &&
                    response is OpenApiResponse openApiResponse)
                    openApiResponse.Description = localizer[resourceKey].Value;
            }

            return Task.CompletedTask;
        }

        public static Task TransformSchemaAsync(
            OpenApiSchema schema,
            OpenApiSchemaTransformerContext context,
            CancellationToken cancellationToken)
        {
            var localizer = context.ApplicationServices.GetRequiredService<IStringLocalizer<SharedResource>>();
            var type = context.JsonTypeInfo.Type;
            if (type == typeof(CreateFavoriteSourceResource))
                schema.Description = localizer["CreateFavoriteSourceSchema"].Value;
            else if (type == typeof(FavoriteSourceResource))
                schema.Description = localizer["FavoriteSourceSchema"].Value;

            var propertyResourceKey = context.JsonPropertyInfo?.Name switch
            {
                "id" => "FavoriteSourceId",
                "newsApiKey" => "FavoriteSourceNewsApiKey",
                "sourceId" => "FavoriteSourceSourceId",
                _ => null
            };

            if (propertyResourceKey is not null)
                schema.Description = localizer[propertyResourceKey].Value;

            return Task.CompletedTask;
        }
    }
    ```
    </details>

    ```
    git add .
    git commit -m "feat(i18n): localize openapi metadata for create favorite source."
    ```

24. **Update `Program.cs` dependency injection.** Add the `News` bounded context registration, right after the `Shared` one from Project Setup:

    ```csharp
    // News Bounded Context Injection Configuration
    builder.Services.AddScoped<IFavoriteSourceRepository, FavoriteSourceRepository>();
    builder.Services.AddScoped<IFavoriteSourceCommandService, FavoriteSourceCommandService>();
    ```

    Plus four new `using` directives at the top of the file:

    ```csharp
    using Acme.CatchUp.Platform.News.Application.Internal.CommandServices;
    using Acme.CatchUp.Platform.News.Application.Services;
    using Acme.CatchUp.Platform.News.Domain.Repositories;
    using Acme.CatchUp.Platform.News.Infrastructure.Persistence.EFC.Repositories;
    ```

    Everything else is unchanged. Open `Program.cs` and replace it with the full file below (both changes above already in place) rather than editing it by hand:

    <details>
    <summary>Program.cs (full file, News dependency injection added)</summary>

    ```csharp
    using Acme.CatchUp.Platform.News.Application.Internal.CommandServices;
    using Acme.CatchUp.Platform.News.Application.Services;
    using Acme.CatchUp.Platform.News.Domain.Repositories;
    using Acme.CatchUp.Platform.News.Infrastructure.Persistence.EFC.Repositories;
    using Acme.CatchUp.Platform.News.Interfaces.REST.Transform;
    using Acme.CatchUp.Platform.Resources;
    using Acme.CatchUp.Platform.Shared.Domain.Repositories;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;
    using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.AspNetCore.Mvc.Infrastructure;
    using Microsoft.EntityFrameworkCore;
    using Microsoft.Extensions.Localization;
    using Scalar.AspNetCore;

    var builder = WebApplication.CreateBuilder(args);

    // Configure Lower Case URLs
    builder.Services.AddRouting(options => options.LowercaseUrls = true);

    // Localization Configuration
    builder.Services.AddLocalization();

    // Configure Kebab Case Route Naming Convention
    builder.Services.AddControllers(options => options.Conventions.Add(new KebabCaseRouteNamingConvention()))
        .AddDataAnnotationsLocalization(options =>
            options.DataAnnotationLocalizerProvider = (_, factory) => factory.Create(typeof(SharedResource)))
        .ConfigureApiBehaviorOptions(options =>
        {
            options.InvalidModelStateResponseFactory = context =>
            {
                var localizer = context.HttpContext.RequestServices
                    .GetRequiredService<IStringLocalizer<SharedResource>>();
                var problemDetailsFactory = context.HttpContext.RequestServices
                    .GetRequiredService<ProblemDetailsFactory>();
                var problemDetails = problemDetailsFactory.CreateValidationProblemDetails(
                    context.HttpContext,
                    context.ModelState,
                    statusCode: StatusCodes.Status400BadRequest,
                    title: localizer["InvalidModelStateTitle"].Value);

                var result = new BadRequestObjectResult(problemDetails);
                result.ContentTypes.Add("application/problem+json");
                return result;
            };
        });

    // Register RFC 7807 ProblemDetails payloads for centralized exception handling.
    builder.Services.AddProblemDetails(options =>
    {
        options.CustomizeProblemDetails = context =>
        {
            if (context.ProblemDetails.Status is null or >= 500)
            {
                var localizer = context.HttpContext.RequestServices.GetRequiredService<IStringLocalizer<SharedResource>>();
                context.ProblemDetails.Title ??= localizer["UnexpectedServerError"].Value;
                context.ProblemDetails.Detail ??= localizer["UnexpectedErrorProcessingRequest"].Value;
            }
        };
    });

    // Generate the OpenAPI document with ASP.NET Core's built-in support.
    builder.Services.AddOpenApi(options =>
    {
        options.AddOperationTransformer(LocalizedOpenApiAssembler.TransformOperationAsync);
        options.AddSchemaTransformer(LocalizedOpenApiAssembler.TransformSchemaAsync);
    });

    // Configure Database Context and route EF logs through the app logger pipeline.
    builder.Services.AddDbContext<AppDbContext>((serviceProvider, options) =>
    {
        var connectionStringTemplate = builder.Configuration.GetConnectionString("DefaultConnection");
        if (string.IsNullOrWhiteSpace(connectionStringTemplate))
            throw new InvalidOperationException("Database connection string is not set in the configuration.");

        var connectionString = Environment.ExpandEnvironmentVariables(connectionStringTemplate);

        options.UseMySQL(connectionString)
            .UseLoggerFactory(serviceProvider.GetRequiredService<ILoggerFactory>())
            .EnableDetailedErrors();

        if (builder.Environment.IsDevelopment())
            options.EnableSensitiveDataLogging();
    });

    // Shared Bounded Context Injection Configuration
    builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

    // News Bounded Context Injection Configuration
    builder.Services.AddScoped<IFavoriteSourceRepository, FavoriteSourceRepository>();
    builder.Services.AddScoped<IFavoriteSourceCommandService, FavoriteSourceCommandService>();

    var app = builder.Build();

    // Apply pending migrations on startup (safe to call even when schema is up to date)
    using (var scope = app.Services.CreateScope())
    {
        var services = scope.ServiceProvider;
        var context = services.GetRequiredService<AppDbContext>();
        context.Database.Migrate();
    }

    // Configure the HTTP request pipeline.
    app.UseExceptionHandler();

    // Localization must run before Swagger so generated documentation follows the request culture.
    string[] supportedCultures = ["en", "en-US", "es", "es-PE"];
    var localizationOptions = new RequestLocalizationOptions()
        .SetDefaultCulture(supportedCultures[0])
        .AddSupportedCultures(supportedCultures)
        .AddSupportedUICultures(supportedCultures);
    localizationOptions.ApplyCurrentCultureToResponseHeaders = true;
    app.UseRequestLocalization(localizationOptions);

    app.UseHttpsRedirection();

    app.UseAuthorization();

    if (app.Environment.IsDevelopment())
    {
        app.MapOpenApi();
        app.MapScalarApiReference();
    }

    app.MapControllers();

    app.Run();
    ```
    </details>

    Commit:

    ```
    git add .
    git commit -m "feat(api): add database context and dependency injection for favorite sources."
    ```

25. **Verify.** Open Scalar (it launches automatically; otherwise `http://localhost:<port>/scalar`), `POST /api/v1/favorite-sources` — a valid payload (`201`), a blank field (`400`), then repeat the valid payload (`409`).

    ```
    git add .
    git commit -m "test(favorite-sources): verify create favorite source endpoint via scalar."
    ```

26. **Verify the Spanish message and the localized OpenAPI document.**
    - **The validation message, in Scalar.** `POST /api/v1/favorite-sources` → "Try it" — the `acceptLanguage` field from step 19 now shows up alongside the request body.
      - `acceptLanguage` → type `es`.
      - Body → `{"newsApiKey": "", "sourceId": "bbc-news"}` (blank `newsApiKey`, to trigger `400`).
      - Send. The response comes back in Spanish.
    - **The OpenAPI document itself.** This one isn't a specific operation's "Try it" panel — it's the raw document at `/openapi/v1.json` — so it needs a header on a plain request. A terminal is enough, no extra tool to install:

      ```
      curl -s -H "Accept-Language: es" http://localhost:<port>/openapi/v1.json | grep -B2 '"CreateFavoriteSource"'
      ```

      `"CreateFavoriteSource"` is this operation's `operationId` (step 21's `LocalizedOpenApiAssembler` sets it from the action name); the two lines above it are `description` and `summary`. The summary reads `"Crear una fuente favorita"`, not the English text. Run it again without the header (or with `-H "Accept-Language: en"`) and the same field reads in English — confirming TS005's fallback to the default culture.

    Later features reuse the `acceptLanguage` field for their own Spanish-message checks, just changing the endpoint and request — no Postman needed anywhere in this guide.

    ```
    git add .
    git commit -m "test(i18n): verify spanish validation message and localized openapi document."
    ```

27. **Publish and finish the feature.** Git Flow Helper widget → `Feature` → `Feature Publish`, then → `Feature Finish` (`Integrate Immediately`, `Keep remote branch when finished` unchecked).

---
## Retrieve a Favorite Source by Id ([TS002](./user-stories.md))

1. **Start the feature.** Git Flow Helper widget → `Feature` → `Feature Start` → **Feature description** `get-favorite-source-by-id` → `OK`.

2. **Create `GetFavoriteSourceByIdQuery`.** Switch to **Solution** view: `News/Domain/Model/Queries/GetFavoriteSourceByIdQuery` as a `record`. No format to validate — a non-existent or non-positive id simply won't match anything and folds into `404`.

   <details>
   <summary>News/Domain/Model/Queries/GetFavoriteSourceByIdQuery.cs</summary>

   ```csharp
   namespace Acme.CatchUp.Platform.News.Domain.Model.Queries;

   public record GetFavoriteSourceByIdQuery(int Id);
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add get favorite source by id query."
   ```

3. **Create `IFavoriteSourceQueryService`.** Same way, `News/Application/Services/IFavoriteSourceQueryService` as an `Interface` — the first query service in this guide, so it starts with just this one method; Features 3 and 4 add the other two.

   <details>
   <summary>News/Application/Services/IFavoriteSourceQueryService.cs</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
   using Acme.CatchUp.Platform.News.Domain.Model.Queries;

   namespace Acme.CatchUp.Platform.News.Application.Services;

   public interface IFavoriteSourceQueryService
   {
       Task<FavoriteSource?> Handle(GetFavoriteSourceByIdQuery query, CancellationToken cancellationToken = default);
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add favorite source query service contract."
   ```

4. **Implement `FavoriteSourceQueryService`.** Same way, `News/Application/Internal/QueryServices/FavoriteSourceQueryService` as a `Class`. Uses `favoriteSourceRepository.FindByIdAsync` — inherited from `IBaseRepository<FavoriteSource>` (Project Setup, step 15), no new repository method needed.

   <details>
   <summary>News/Application/Internal/QueryServices/FavoriteSourceQueryService.cs</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
   using Acme.CatchUp.Platform.News.Application.Services;
   using Acme.CatchUp.Platform.News.Domain.Model.Queries;
   using Acme.CatchUp.Platform.News.Domain.Repositories;

   namespace Acme.CatchUp.Platform.News.Application.Internal.QueryServices;

   public class FavoriteSourceQueryService(IFavoriteSourceRepository favoriteSourceRepository)
       : IFavoriteSourceQueryService
   {
       public async Task<FavoriteSource?> Handle(GetFavoriteSourceByIdQuery query,
           CancellationToken cancellationToken = default) =>
           await favoriteSourceRepository.FindByIdAsync(query.Id, cancellationToken);
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add get favorite source by id query handling."
   ```

5. **Add the GET-by-id action to `FavoriteSourcesController`.**
   - Inject `IFavoriteSourceQueryService` alongside the command service already there.
   - Add `[HttpGet("{id}")] GetFavoriteSourceById(...)`: `null` → `NotFound()`, otherwise assemble and `Ok(resource)`.
   - This is the route `FavoriteSourceLocationTemplate` (Feature 1, step 18-19) stood in for with a hand-built path — now that the route exists, `CreatedAtAction` can resolve it directly, so both the constant and the manual path-building in the assembler go away.

   <details>
   <summary>FavoriteSourcesController.cs (constructor parameter, add it to the primary constructor)</summary>

   ```csharp
   IFavoriteSourceQueryService favoriteSourceQueryService,
   ```
   </details>

   <details>
   <summary>FavoriteSourcesController.cs (new method, add it inside the class; new import at the top)</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Domain.Model.Queries;
   ```

   ```csharp
   [HttpGet("{id}")]
   [ProducesResponseType(typeof(FavoriteSourceResource), StatusCodes.Status200OK)]
   [ProducesResponseType(StatusCodes.Status404NotFound)]
   public async Task<ActionResult> GetFavoriteSourceById(int id,
       [FromHeader(Name = "Accept-Language")] string? acceptLanguage,
       CancellationToken cancellationToken = default)
   {
       var getFavoriteSourceByIdQuery = new GetFavoriteSourceByIdQuery(id);
       var result = await favoriteSourceQueryService.Handle(getFavoriteSourceByIdQuery, cancellationToken);
       if (result is null) return NotFound();
       var resource = FavoriteSourceResourceFromEntityAssembler.ToResourceFromEntity(result);
       return Ok(resource);
   }
   ```
   </details>

   Delete the `FavoriteSourceLocationTemplate` constant, and update `CreateFavoriteSource`'s call into the action result assembler to pass `nameof(GetFavoriteSourceById)` directly in its place:

   <details>
   <summary>FavoriteSourcesController.cs (CreateFavoriteSource, last argument)</summary>

   ```csharp
   nameof(GetFavoriteSourceById));
   ```
   </details>

   `ActionResultFromCreateFavoriteSourceResultAssembler` goes back to `CreatedAtAction` now that the route it names actually exists — rename its last parameter back to `getFavoriteSourceByIdActionName` and replace the whole `Success` branch:

   <details>
   <summary>ActionResultFromCreateFavoriteSourceResultAssembler.cs (parameter and Success branch)</summary>

   ```csharp
   public static ActionResult ToActionResultFromCreateFavoriteSourceResult(
       Result<FavoriteSource, CreateFavoriteSourceError> result,
       ControllerBase controller,
       IStringLocalizer<SharedResource> localizer,
       string getFavoriteSourceByIdActionName) =>
       result switch
       {
           Result<FavoriteSource, CreateFavoriteSourceError>.Success success =>
               controller.CreatedAtAction(getFavoriteSourceByIdActionName, new { id = success.Value.Id },
                   FavoriteSourceResourceFromEntityAssembler.ToResourceFromEntity(success.Value)),
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add get favorite source by id endpoint."
   ```

6. **Add this feature's resource keys.** Same five `.resx` files as Feature 1. English:

   <details>
   <summary>Feature 2 resx entries (English)</summary>

   ```xml
   <data name="GetFavoriteSourceByIdSummary" xml:space="preserve">
       <value>Get a favorite source by ID</value>
   </data>
   <data name="GetFavoriteSourceByIdDescription" xml:space="preserve">
       <value>Retrieve a favorite source using its identifier.</value>
   </data>
   <data name="GetFavoriteSourceByIdSuccess" xml:space="preserve">
       <value>The favorite source was found.</value>
   </data>
   <data name="GetFavoriteSourceByIdNotFound" xml:space="preserve">
       <value>The favorite source was not found.</value>
   </data>
   ```
   </details>

   Spanish:

   <details>
   <summary>Feature 2 resx entries (Spanish)</summary>

   ```xml
   <data name="GetFavoriteSourceByIdSummary" xml:space="preserve">
       <value>Obtener una fuente favorita por ID</value>
   </data>
   <data name="GetFavoriteSourceByIdDescription" xml:space="preserve">
       <value>Obtiene una fuente favorita mediante su identificador.</value>
   </data>
   <data name="GetFavoriteSourceByIdSuccess" xml:space="preserve">
       <value>Se encontró la fuente favorita.</value>
   </data>
   <data name="GetFavoriteSourceByIdNotFound" xml:space="preserve">
       <value>No se encontró la fuente favorita.</value>
   </data>
   ```
   </details>

   ```
   git add .
   git commit -m "feat(i18n): add localized messages for get favorite source by id."
   ```

7. **Localize this operation's OpenAPI entry.** Add a second entry to `OperationResources` in `LocalizedOpenApiAssembler.cs`, next to `CreateFavoriteSource`:

   <details>
   <summary>LocalizedOpenApiAssembler.cs (OperationResources, new entry)</summary>

   ```csharp
   ["GetFavoriteSourceById"] = (
       "GetFavoriteSourceByIdSummary",
       "GetFavoriteSourceByIdDescription",
       new Dictionary<string, string>
       {
           ["200"] = "GetFavoriteSourceByIdSuccess",
           ["404"] = "GetFavoriteSourceByIdNotFound"
       }),
   ```
   </details>

   ```
   git add .
   git commit -m "feat(i18n): localize openapi metadata for get favorite source by id."
   ```

8. **Update `Program.cs` dependency injection.** Add one line next to the `News` registrations from Feature 1:

   ```csharp
   builder.Services.AddScoped<IFavoriteSourceQueryService, FavoriteSourceQueryService>();
   ```

   Plus one new `using` at the top of the file:

   ```csharp
   using Acme.CatchUp.Platform.News.Application.Internal.QueryServices;
   ```

   Everything else is unchanged. Open `Program.cs` and replace it with the full file below (both changes above already in place) rather than editing it by hand:

   <details>
   <summary>Program.cs (full file, query service dependency injection added)</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Application.Internal.CommandServices;
   using Acme.CatchUp.Platform.News.Application.Internal.QueryServices;
   using Acme.CatchUp.Platform.News.Application.Services;
   using Acme.CatchUp.Platform.News.Domain.Repositories;
   using Acme.CatchUp.Platform.News.Infrastructure.Persistence.EFC.Repositories;
   using Acme.CatchUp.Platform.News.Interfaces.REST.Transform;
   using Acme.CatchUp.Platform.Resources;
   using Acme.CatchUp.Platform.Shared.Domain.Repositories;
   using Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration;
   using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;
   using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.AspNetCore.Mvc.Infrastructure;
   using Microsoft.EntityFrameworkCore;
   using Microsoft.Extensions.Localization;
   using Scalar.AspNetCore;

   var builder = WebApplication.CreateBuilder(args);

   // Configure Lower Case URLs
   builder.Services.AddRouting(options => options.LowercaseUrls = true);

   // Localization Configuration
   builder.Services.AddLocalization();

   // Configure Kebab Case Route Naming Convention
   builder.Services.AddControllers(options => options.Conventions.Add(new KebabCaseRouteNamingConvention()))
       .AddDataAnnotationsLocalization(options =>
           options.DataAnnotationLocalizerProvider = (_, factory) => factory.Create(typeof(SharedResource)))
       .ConfigureApiBehaviorOptions(options =>
       {
           options.InvalidModelStateResponseFactory = context =>
           {
               var localizer = context.HttpContext.RequestServices
                   .GetRequiredService<IStringLocalizer<SharedResource>>();
               var problemDetailsFactory = context.HttpContext.RequestServices
                   .GetRequiredService<ProblemDetailsFactory>();
               var problemDetails = problemDetailsFactory.CreateValidationProblemDetails(
                   context.HttpContext,
                   context.ModelState,
                   statusCode: StatusCodes.Status400BadRequest,
                   title: localizer["InvalidModelStateTitle"].Value);

               var result = new BadRequestObjectResult(problemDetails);
               result.ContentTypes.Add("application/problem+json");
               return result;
           };
       });

   // Register RFC 7807 ProblemDetails payloads for centralized exception handling.
   builder.Services.AddProblemDetails(options =>
   {
       options.CustomizeProblemDetails = context =>
       {
           if (context.ProblemDetails.Status is null or >= 500)
           {
               var localizer = context.HttpContext.RequestServices.GetRequiredService<IStringLocalizer<SharedResource>>();
               context.ProblemDetails.Title ??= localizer["UnexpectedServerError"].Value;
               context.ProblemDetails.Detail ??= localizer["UnexpectedErrorProcessingRequest"].Value;
           }
       };
   });

   // Generate the OpenAPI document with ASP.NET Core's built-in support.
   builder.Services.AddOpenApi(options =>
   {
       options.AddOperationTransformer(LocalizedOpenApiAssembler.TransformOperationAsync);
       options.AddSchemaTransformer(LocalizedOpenApiAssembler.TransformSchemaAsync);
   });

   // Configure Database Context and route EF logs through the app logger pipeline.
   builder.Services.AddDbContext<AppDbContext>((serviceProvider, options) =>
   {
       var connectionStringTemplate = builder.Configuration.GetConnectionString("DefaultConnection");
       if (string.IsNullOrWhiteSpace(connectionStringTemplate))
           throw new InvalidOperationException("Database connection string is not set in the configuration.");

       var connectionString = Environment.ExpandEnvironmentVariables(connectionStringTemplate);

       options.UseMySQL(connectionString)
           .UseLoggerFactory(serviceProvider.GetRequiredService<ILoggerFactory>())
           .EnableDetailedErrors();

       if (builder.Environment.IsDevelopment())
           options.EnableSensitiveDataLogging();
   });

   // Shared Bounded Context Injection Configuration
   builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

   // News Bounded Context Injection Configuration
   builder.Services.AddScoped<IFavoriteSourceRepository, FavoriteSourceRepository>();
   builder.Services.AddScoped<IFavoriteSourceCommandService, FavoriteSourceCommandService>();
   builder.Services.AddScoped<IFavoriteSourceQueryService, FavoriteSourceQueryService>();

   var app = builder.Build();

   // Apply pending migrations on startup (safe to call even when schema is up to date)
   using (var scope = app.Services.CreateScope())
   {
       var services = scope.ServiceProvider;
       var context = services.GetRequiredService<AppDbContext>();
       context.Database.Migrate();
   }

   // Configure the HTTP request pipeline.
   app.UseExceptionHandler();

   // Localization must run before Swagger so generated documentation follows the request culture.
   string[] supportedCultures = ["en", "en-US", "es", "es-PE"];
   var localizationOptions = new RequestLocalizationOptions()
       .SetDefaultCulture(supportedCultures[0])
       .AddSupportedCultures(supportedCultures)
       .AddSupportedUICultures(supportedCultures);
   localizationOptions.ApplyCurrentCultureToResponseHeaders = true;
   app.UseRequestLocalization(localizationOptions);

   app.UseHttpsRedirection();

   app.UseAuthorization();

   if (app.Environment.IsDevelopment())
   {
       app.MapOpenApi();
       app.MapScalarApiReference();
   }

   app.MapControllers();

   app.Run();
   ```
   </details>

   Commit:

   ```
   git add .
   git commit -m "feat(api): add dependency injection for favorite source query service."
   ```

9. **Verify.** Scalar, `GET /api/v1/favorite-sources/{id}` for an existing id (reuse one created in Feature 1) and a non-existent id (`404`). For the Spanish message, repeat the `404` case with the `acceptLanguage` field set to `es` (same as Feature 1, step 26).

   ```
   git add .
   git commit -m "test(favorite-sources): verify get favorite source by id endpoint via scalar."
   ```

10. **Publish and finish the feature.** Git Flow Helper widget → `Feature` → `Feature Publish`, then → `Feature Finish` (`Integrate Immediately`, `Keep remote branch when finished` unchecked).

---
## List Favorite Sources by NewsAPI Key ([TS003](./user-stories.md))

1. **Start the feature.** Git Flow Helper widget → `Feature` → `Feature Start` → **Feature description** `list-favorite-sources-by-news-api-key` → `OK`.

2. **Create `GetAllFavoriteSourcesByNewsApiKeyQuery`.** Switch to **Solution** view: `News/Domain/Model/Queries/GetAllFavoriteSourcesByNewsApiKeyQuery` as a `record`.

   <details>
   <summary>News/Domain/Model/Queries/GetAllFavoriteSourcesByNewsApiKeyQuery.cs</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

   namespace Acme.CatchUp.Platform.News.Domain.Model.Queries;

   public record GetAllFavoriteSourcesByNewsApiKeyQuery(NewsApiKey NewsApiKey);
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add get all favorite sources by news api key query."
   ```

3. **Add `FindByNewsApiKeyAsync` to the repository.** `IFavoriteSourceRepository`:

   ```csharp
   Task<IEnumerable<FavoriteSource>> FindByNewsApiKeyAsync(NewsApiKey newsApiKey,
       CancellationToken cancellationToken = default);
   ```

   `FavoriteSourceRepository`, new `using`:

   ```csharp
   using Microsoft.EntityFrameworkCore;
   ```

   ```csharp
   public async Task<IEnumerable<FavoriteSource>> FindByNewsApiKeyAsync(NewsApiKey newsApiKey,
       CancellationToken cancellationToken = default)
   {
       return await Context.Set<FavoriteSource>()
           .Where(f => f.NewsApiKey == newsApiKey)
           .ToListAsync(cancellationToken);
   }
   ```

   ```
   git add .
   git commit -m "feat(favorite-sources): add find by news api key to repository."
   ```

4. **Add `Handle(GetAllFavoriteSourcesByNewsApiKeyQuery)` to the query service.** `IFavoriteSourceQueryService`:

   ```csharp
   Task<IEnumerable<FavoriteSource>> Handle(GetAllFavoriteSourcesByNewsApiKeyQuery query,
       CancellationToken cancellationToken = default);
   ```

   `FavoriteSourceQueryService`:

   ```csharp
   public async Task<IEnumerable<FavoriteSource>> Handle(GetAllFavoriteSourcesByNewsApiKeyQuery query,
       CancellationToken cancellationToken = default) =>
       await favoriteSourceRepository.FindByNewsApiKeyAsync(query.NewsApiKey, cancellationToken);
   ```

   ```
   git add .
   git commit -m "feat(favorite-sources): add get all favorite sources by news api key query handling."
   ```

5. **Add the list endpoint to `FavoriteSourcesController`.**
   - A private `GetAllFavoriteSourcesByNewsApiKey` helper: maps the list through the assembler, always `Ok`, empty array included — this is how TS003's "no favorites" scenario stays a `200` with `[]`, never a `404`.
   - The public entry point `GetFavoriteSourceFromQuery`: builds the `NewsApiKey` value object (an invalid format throws, caught below); when `sourceId` isn't present in the query string, delegates to the helper above.
   - Feature 4 extends this same method with the `sourceId` branch.

   <details>
   <summary>FavoriteSourcesController.cs (new methods, add them inside the class; new imports at the top)</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
   ```

   ```csharp
   private async Task<ActionResult> GetAllFavoriteSourcesByNewsApiKey(NewsApiKey newsApiKey,
       CancellationToken cancellationToken)
   {
       var getAllFavoriteSourcesByNewsApiKeyQuery = new GetAllFavoriteSourcesByNewsApiKeyQuery(newsApiKey);
       var result = await favoriteSourceQueryService.Handle(getAllFavoriteSourcesByNewsApiKeyQuery, cancellationToken);
       var resources = result.Select(FavoriteSourceResourceFromEntityAssembler.ToResourceFromEntity);
       return Ok(resources);
   }

   [HttpGet]
   [ProducesResponseType(typeof(IEnumerable<FavoriteSourceResource>), StatusCodes.Status200OK)]
   [ProducesResponseType(typeof(string), StatusCodes.Status400BadRequest)]
   public async Task<ActionResult> GetFavoriteSourceFromQuery([FromQuery] string newsApiKey,
       [FromHeader(Name = "Accept-Language")] string? acceptLanguage,
       CancellationToken cancellationToken = default)
   {
       try
       {
           var newsApiKeyValueObject = new NewsApiKey(newsApiKey);
           return await GetAllFavoriteSourcesByNewsApiKey(newsApiKeyValueObject, cancellationToken);
       }
       catch (ArgumentException ex)
       {
           logger.LogWarning(ex,
               "Validation failed while querying favorite source for NewsApiKey {NewsApiKey}",
               NewsApiKey.MaskedForLogging(newsApiKey));
           return BadRequest(localizer["InvalidFavoriteSourceQuery"].Value);
       }
   }
   ```
   </details>

   **Note:** `GetAllFavoriteSourcesByNewsApiKeyQuery` is already in scope (step 2), and `GetAllFavoriteSourcesByNewsApiKey` above uses it directly; no extra `using` needed for it since it lives in the same `Queries` namespace already imported in Feature 2.

   ```
   git add .
   git commit -m "feat(favorite-sources): add list favorite sources endpoint."
   ```

6. **Add this feature's resource keys.** English:

   <details>
   <summary>Feature 3 resx entries (English)</summary>

   ```xml
   <data name="InvalidFavoriteSourceQuery" xml:space="preserve">
       <value>The query parameters for favorite sources contain invalid values.</value>
   </data>
   <data name="GetFavoriteSourceQuerySummary" xml:space="preserve">
       <value>Get favorite sources by query</value>
   </data>
   <data name="GetFavoriteSourceQueryDescription" xml:space="preserve">
       <value>Provide only newsApiKey to list matching favorite sources (an empty list if none exist). Add sourceId to retrieve one matching favorite source.</value>
   </data>
   <data name="GetFavoriteSourceQuerySuccess" xml:space="preserve">
       <value>The matching favorite source or sources were found.</value>
   </data>
   <data name="GetFavoriteSourceQueryBadRequest" xml:space="preserve">
       <value>A required query parameter is missing or invalid.</value>
   </data>
   <data name="GetFavoriteSourceQueryNotFound" xml:space="preserve">
       <value>No favorite source matches the supplied newsApiKey and sourceId.</value>
   </data>
   ```
   </details>

   Spanish:

   <details>
   <summary>Feature 3 resx entries (Spanish)</summary>

   ```xml
   <data name="InvalidFavoriteSourceQuery" xml:space="preserve">
       <value>Los parámetros de consulta para fuentes favoritas contienen valores no válidos.</value>
   </data>
   <data name="GetFavoriteSourceQuerySummary" xml:space="preserve">
       <value>Obtener fuentes favoritas mediante parámetros de consulta</value>
   </data>
   <data name="GetFavoriteSourceQueryDescription" xml:space="preserve">
       <value>Proporcione solo newsApiKey para listar las fuentes favoritas coincidentes (una lista vacía si no hay ninguna). Agregue sourceId para obtener una fuente favorita específica.</value>
   </data>
   <data name="GetFavoriteSourceQuerySuccess" xml:space="preserve">
       <value>Se encontró la fuente o las fuentes favoritas coincidentes.</value>
   </data>
   <data name="GetFavoriteSourceQueryBadRequest" xml:space="preserve">
       <value>Falta un parámetro de consulta obligatorio o no es válido.</value>
   </data>
   <data name="GetFavoriteSourceQueryNotFound" xml:space="preserve">
       <value>No existe una fuente favorita que coincida con newsApiKey y sourceId.</value>
   </data>
   ```
   </details>

   **Note:** the description already mentions `sourceId`, even though this feature's endpoint doesn't accept it yet — Feature 4 extends the same action, and this text describes both branches together rather than needing a second, near-duplicate key.

   ```
   git add .
   git commit -m "feat(i18n): add localized messages for list favorite sources."
   ```

7. **Localize this operation's OpenAPI entry.**

   <details>
   <summary>LocalizedOpenApiAssembler.cs (OperationResources, new entry)</summary>

   ```csharp
   ["GetFavoriteSourceFromQuery"] = (
       "GetFavoriteSourceQuerySummary",
       "GetFavoriteSourceQueryDescription",
       new Dictionary<string, string>
       {
           ["200"] = "GetFavoriteSourceQuerySuccess",
           ["400"] = "GetFavoriteSourceQueryBadRequest",
           ["404"] = "GetFavoriteSourceQueryNotFound"
       }),
   ```
   </details>

   ```
   git add .
   git commit -m "feat(i18n): localize openapi metadata for list favorite sources."
   ```

8. **Verify.** Scalar, `GET /api/v1/favorite-sources?newsApiKey=...` with matches, with no matches (`200`, empty array), and with an invalid key (`400`).

   ```
   git add .
   git commit -m "test(favorite-sources): verify list favorite sources endpoint via scalar."
   ```

9. **Publish and finish the feature.** Git Flow Helper widget → `Feature` → `Feature Publish`, then → `Feature Finish` (`Integrate Immediately`, `Keep remote branch when finished` unchecked).

---
## Check Whether a Source Is Favorite ([TS004](./user-stories.md))

1. **Start the feature.** Git Flow Helper widget → `Feature` → `Feature Start` → **Feature description** `get-favorite-source-by-news-api-key-and-source-id` → `OK`.

2. **Create `GetFavoriteSourceByNewsApiKeyAndSourceIdQuery`.** Switch to **Solution** view: `News/Domain/Model/Queries/GetFavoriteSourceByNewsApiKeyAndSourceIdQuery` as a `record`.

   <details>
   <summary>News/Domain/Model/Queries/GetFavoriteSourceByNewsApiKeyAndSourceIdQuery.cs</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

   namespace Acme.CatchUp.Platform.News.Domain.Model.Queries;

   public record GetFavoriteSourceByNewsApiKeyAndSourceIdQuery(NewsApiKey NewsApiKey, SourceId SourceId);
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add get favorite source by news api key and source id query."
   ```

3. **Add `Handle(GetFavoriteSourceByNewsApiKeyAndSourceIdQuery)` to the query service.** Uses `FindByNewsApiKeyAndSourceIdAsync`, already on `IFavoriteSourceRepository` since Feature 1 (step 8) — no new repository method needed. `IFavoriteSourceQueryService`:

   ```csharp
   Task<FavoriteSource?> Handle(GetFavoriteSourceByNewsApiKeyAndSourceIdQuery query,
       CancellationToken cancellationToken = default);
   ```

   `FavoriteSourceQueryService`:

   ```csharp
   public async Task<FavoriteSource?> Handle(GetFavoriteSourceByNewsApiKeyAndSourceIdQuery query,
       CancellationToken cancellationToken = default) =>
       await favoriteSourceRepository.FindByNewsApiKeyAndSourceIdAsync(query.NewsApiKey, query.SourceId,
           cancellationToken);
   ```

   ```
   git add .
   git commit -m "feat(favorite-sources): add get favorite source by news api key and source id query handling."
   ```

4. **Extend `FavoriteSourcesController`.**
   - A private `GetFavoriteSourceByNewsApiKeyAndSourceId` helper: `null` → `NotFound()`, else `Ok(resource)`.
   - Extend `GetFavoriteSourceFromQuery` (Feature 3) so that when `sourceId` is present and non-empty, it builds the `SourceId` value object and delegates here instead of to the list helper.
   - Wrap the whole method body in `try`/`catch (ArgumentException)` → `BadRequest(...)`, covering both this feature's and Feature 3's validation.

   <details>
   <summary>FavoriteSourcesController.cs (new private method, add it inside the class)</summary>

   ```csharp
   private async Task<ActionResult> GetFavoriteSourceByNewsApiKeyAndSourceId(NewsApiKey newsApiKey, SourceId sourceId,
       CancellationToken cancellationToken)
   {
       var getFavoriteSourceByNewsApiKeyAndSourceIdQuery =
           new GetFavoriteSourceByNewsApiKeyAndSourceIdQuery(newsApiKey, sourceId);
       var result =
           await favoriteSourceQueryService.Handle(getFavoriteSourceByNewsApiKeyAndSourceIdQuery, cancellationToken);
       if (result is null) return NotFound();
       var resource = FavoriteSourceResourceFromEntityAssembler.ToResourceFromEntity(result);
       return Ok(resource);
   }
   ```
   </details>

   <details>
   <summary>FavoriteSourcesController.cs (GetFavoriteSourceFromQuery, replaces the Feature 3 version)</summary>

   ```csharp
   [HttpGet]
   [ProducesResponseType(typeof(IEnumerable<FavoriteSourceResource>), StatusCodes.Status200OK)]
   [ProducesResponseType(typeof(string), StatusCodes.Status400BadRequest)]
   [ProducesResponseType(StatusCodes.Status404NotFound)]
   public async Task<ActionResult> GetFavoriteSourceFromQuery([FromQuery] string newsApiKey,
       [FromQuery] string? sourceId = null,
       [FromHeader(Name = "Accept-Language")] string? acceptLanguage = null,
       CancellationToken cancellationToken = default)
   {
       try
       {
           var newsApiKeyValueObject = new NewsApiKey(newsApiKey);
           return !Request.Query.ContainsKey(nameof(sourceId))
               ? await GetAllFavoriteSourcesByNewsApiKey(newsApiKeyValueObject, cancellationToken)
               : await GetFavoriteSourceByNewsApiKeyAndSourceId(newsApiKeyValueObject,
                   new SourceId(sourceId ?? string.Empty), cancellationToken);
       }
       catch (ArgumentException ex)
       {
           logger.LogWarning(ex,
               "Validation failed while querying favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
               NewsApiKey.MaskedForLogging(newsApiKey),
               sourceId);
           return BadRequest(localizer["InvalidFavoriteSourceQuery"].Value);
       }
   }
   ```
   </details>

   **Note:** `Request.Query.ContainsKey(nameof(sourceId))`, not `sourceId is not null`, is what branches the two scenarios apart — `?sourceId=` (present, empty) still means "check this exact pair" and should validate as a `400`, not silently fall back to the list behavior.

   ```
   git add .
   git commit -m "feat(favorite-sources): add get favorite source by news api key and source id endpoint."
   ```

5. **Verify.** Scalar, `GET /api/v1/favorite-sources?newsApiKey=...&sourceId=...` for an existing pair, a non-existent pair (`404`), and an invalid `sourceId` (`400`). For the Spanish message, repeat the `404`/`400` cases with the `acceptLanguage` field set to `es` (same as Feature 1, step 26).

   ```
   git add .
   git commit -m "test(favorite-sources): verify get favorite source by news api key and source id endpoint via scalar."
   ```

6. **Publish and finish the feature.** Git Flow Helper widget → `Feature` → `Feature Publish`, then → `Feature Finish` (`Integrate Immediately`, `Keep remote branch when finished` unchecked).

---
## Remove a Favorite Source ([TS006](./user-stories.md))

1. **Start the feature.** Git Flow Helper widget → `Feature` → `Feature Start` → **Feature description** `delete-favorite-source` → `OK`.

2. **Create `DeleteFavoriteSourceCommand`.** Switch to **Solution** view: `News/Domain/Model/Commands/DeleteFavoriteSourceCommand` as a `Class`. Same validating shape as `NewsApiKey`/`SourceId` (Feature 1), not a plain positional `record` — a `DELETE` for `id=0` or a negative id should fail loudly with a `400`, not silently 404.

   <details>
   <summary>News/Domain/Model/Commands/DeleteFavoriteSourceCommand.cs</summary>

   ```csharp
   namespace Acme.CatchUp.Platform.News.Domain.Model.Commands;

   public sealed record DeleteFavoriteSourceCommand
   {
       public DeleteFavoriteSourceCommand(int id)
       {
           if (id <= 0)
               throw new ArgumentException("Id must be a positive integer.", nameof(id));

           Id = id;
       }

       public int Id { get; }
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add delete favorite source command with validation."
   ```

3. **Create `DeleteFavoriteSourceError`.** Same way, `News/Application/Errors/DeleteFavoriteSourceError` as an `Enum`.

   <details>
   <summary>News/Application/Errors/DeleteFavoriteSourceError.cs</summary>

   ```csharp
   namespace Acme.CatchUp.Platform.News.Application.Errors;

   public enum DeleteFavoriteSourceError
   {
       NotFound,
       UnexpectedError
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add delete favorite source error enum."
   ```

4. **Add `Handle(DeleteFavoriteSourceCommand)` to the command service contract.** `IFavoriteSourceCommandService`, new `using` at the top:

   ```csharp
   using Acme.CatchUp.Platform.News.Domain.Model.Commands;
   ```

   ```csharp
   Task<Result<int, DeleteFavoriteSourceError>> Handle(DeleteFavoriteSourceCommand command, CancellationToken cancellationToken = default);
   ```

   ```
   git add .
   git commit -m "feat(favorite-sources): add delete favorite source to command service contract."
   ```

5. **Implement it in `FavoriteSourceCommandService`.**
   - `FindByIdAsync` (inherited from `IBaseRepository<FavoriteSource>`) → `null` → `Failure(NotFound)`.
   - Otherwise `Remove` (also inherited — it takes the entity, not the id, so the lookup above doubles as the existence check) + `unitOfWork.CompleteAsync`.
   - → `Success(command.Id)`, the removed id, for symmetry with the other `Handle` methods.

   <details>
   <summary>FavoriteSourceCommandService.cs (new method, add it inside the class; new import at the top)</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Application.Errors;
   ```

   ```csharp
   public async Task<Result<int, DeleteFavoriteSourceError>> Handle(DeleteFavoriteSourceCommand command,
       CancellationToken cancellationToken = default)
   {
       var favoriteSource = await favoriteSourceRepository.FindByIdAsync(command.Id, cancellationToken);
       if (favoriteSource is null)
       {
           logger.LogDebug("Favorite source not found for deletion: id={Id}", command.Id);
           return new Result<int, DeleteFavoriteSourceError>.Failure(DeleteFavoriteSourceError.NotFound);
       }

       try
       {
           favoriteSourceRepository.Remove(favoriteSource);
           await unitOfWork.CompleteAsync(cancellationToken);
           logger.LogInformation("Favorite source removed: id={Id}", command.Id);
           return new Result<int, DeleteFavoriteSourceError>.Success(command.Id);
       }
       catch (Exception ex)
       {
           logger.LogError(ex, "Unexpected error removing favorite source: id={Id}", command.Id);
           return new Result<int, DeleteFavoriteSourceError>.Failure(DeleteFavoriteSourceError.UnexpectedError);
       }
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): implement delete favorite source in command service."
   ```

6. **Create `ActionResultFromDeleteFavoriteSourceResultAssembler`.** Switch to **Solution** view: `News/Interfaces/REST/Transform/ActionResultFromDeleteFavoriteSourceResultAssembler` as a `Class` (`static`) — a separate assembler from Feature 1's, same naming convention (`<Target>From<Source>Assembler`), since it maps a different `Result` type.
   - `Success` → `204 NoContent`.
   - `Failure` → `404` for `NotFound`, `500` `ProblemDetails` otherwise.

   <details>
   <summary>News/Interfaces/REST/Transform/ActionResultFromDeleteFavoriteSourceResultAssembler.cs</summary>

   ```csharp
   using Acme.CatchUp.Platform.News.Application.Errors;
   using Acme.CatchUp.Platform.Resources;
   using Acme.CatchUp.Platform.Shared.Application.Patterns;
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.Extensions.Localization;

   namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

   public static class ActionResultFromDeleteFavoriteSourceResultAssembler
   {
       public static ActionResult ToActionResultFromDeleteFavoriteSourceResult(
           Result<int, DeleteFavoriteSourceError> result,
           ControllerBase controller,
           IStringLocalizer<SharedResource> localizer) =>
           result switch
           {
               Result<int, DeleteFavoriteSourceError>.Success => controller.NoContent(),

               Result<int, DeleteFavoriteSourceError>.Failure failure =>
                   failure.Error switch
                   {
                       DeleteFavoriteSourceError.NotFound =>
                           controller.NotFound(localizer["FavoriteSourceNotFound"].Value),

                       _ => controller.Problem(
                           title: localizer["UnexpectedServerError"].Value,
                           detail: localizer["UnexpectedErrorProcessingRequest"].Value,
                           statusCode: 500)
                   },

               _ => controller.Problem(
                   title: localizer["UnexpectedServerError"].Value,
                   detail: localizer["UnexpectedErrorProcessingRequest"].Value,
                   statusCode: 500)
           };
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add action result assembler for delete favorite source result."
   ```

7. **Add the DELETE endpoint to `FavoriteSourcesController`.**

   <details>
   <summary>FavoriteSourcesController.cs (new method, add it inside the class)</summary>

   ```csharp
   [HttpDelete("{id}")]
   [ProducesResponseType(StatusCodes.Status204NoContent)]
   [ProducesResponseType(typeof(string), StatusCodes.Status404NotFound)]
   [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status500InternalServerError)]
   public async Task<ActionResult> DeleteFavoriteSource(int id,
       [FromHeader(Name = "Accept-Language")] string? acceptLanguage,
       CancellationToken cancellationToken = default)
   {
       var result = await favoriteSourceCommandService.Handle(new DeleteFavoriteSourceCommand(id), cancellationToken);
       return ActionResultFromDeleteFavoriteSourceResultAssembler.ToActionResultFromDeleteFavoriteSourceResult(
           result, this, localizer);
   }
   ```
   </details>

   An invalid `id` (zero or negative) throws from `DeleteFavoriteSourceCommand`'s own constructor (step 2) before this method's body runs any further — ASP.NET Core's model binding doesn't catch it, so a `400` here comes from the same `ArgumentException` → `BadRequest` pattern as the other actions. Wrap the body in `try`/`catch (ArgumentException)` → `BadRequest(...)`:

   <details>
   <summary>FavoriteSourcesController.cs (DeleteFavoriteSource, with validation handling)</summary>

   ```csharp
   [HttpDelete("{id}")]
   [ProducesResponseType(StatusCodes.Status204NoContent)]
   [ProducesResponseType(typeof(string), StatusCodes.Status400BadRequest)]
   [ProducesResponseType(typeof(string), StatusCodes.Status404NotFound)]
   [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status500InternalServerError)]
   public async Task<ActionResult> DeleteFavoriteSource(int id,
       [FromHeader(Name = "Accept-Language")] string? acceptLanguage,
       CancellationToken cancellationToken = default)
   {
       try
       {
           var result = await favoriteSourceCommandService.Handle(new DeleteFavoriteSourceCommand(id), cancellationToken);
           return ActionResultFromDeleteFavoriteSourceResultAssembler.ToActionResultFromDeleteFavoriteSourceResult(
               result, this, localizer);
       }
       catch (ArgumentException ex)
       {
           logger.LogWarning(ex, "Validation failed while deleting favorite source: id={Id}", id);
           return BadRequest(localizer["InvalidFavoriteSourceRequest"].Value);
       }
   }
   ```
   </details>

   ```
   git add .
   git commit -m "feat(favorite-sources): add delete favorite source endpoint."
   ```

8. **Add this feature's resource keys.** English:

   <details>
   <summary>Feature 5 resx entries (English)</summary>

   ```xml
   <data name="FavoriteSourceNotFound" xml:space="preserve">
       <value>The favorite source was not found.</value>
   </data>
   <data name="DeleteFavoriteSourceSummary" xml:space="preserve">
       <value>Remove a favorite source</value>
   </data>
   <data name="DeleteFavoriteSourceDescription" xml:space="preserve">
       <value>Removes a favorite source by its ID.</value>
   </data>
   <data name="DeleteFavoriteSourceNoContent" xml:space="preserve">
       <value>The favorite source was removed.</value>
   </data>
   <data name="DeleteFavoriteSourceNotFound" xml:space="preserve">
       <value>The favorite source was not found.</value>
   </data>
   ```
   </details>

   Spanish:

   <details>
   <summary>Feature 5 resx entries (Spanish)</summary>

   ```xml
   <data name="FavoriteSourceNotFound" xml:space="preserve">
       <value>No se encontró la fuente favorita.</value>
   </data>
   <data name="DeleteFavoriteSourceSummary" xml:space="preserve">
       <value>Eliminar una fuente favorita</value>
   </data>
   <data name="DeleteFavoriteSourceDescription" xml:space="preserve">
       <value>Elimina una fuente favorita mediante su ID.</value>
   </data>
   <data name="DeleteFavoriteSourceNoContent" xml:space="preserve">
       <value>Se eliminó la fuente favorita.</value>
   </data>
   <data name="DeleteFavoriteSourceNotFound" xml:space="preserve">
       <value>No se encontró la fuente favorita.</value>
   </data>
   ```
   </details>

   ```
   git add .
   git commit -m "feat(i18n): add localized messages for delete favorite source."
   ```

9. **Localize this operation's OpenAPI entry.**

   <details>
   <summary>LocalizedOpenApiAssembler.cs (OperationResources, new entry)</summary>

   ```csharp
   ["DeleteFavoriteSource"] = (
       "DeleteFavoriteSourceSummary",
       "DeleteFavoriteSourceDescription",
       new Dictionary<string, string>
       {
           ["204"] = "DeleteFavoriteSourceNoContent",
           ["404"] = "DeleteFavoriteSourceNotFound"
       }),
   ```
   </details>

   ```
   git add .
   git commit -m "feat(i18n): localize openapi metadata for delete favorite source."
   ```

10. **Verify.** Scalar, `DELETE /api/v1/favorite-sources/{id}` for an existing id (`204`), a non-existent id (`404`), and `id=0` (`400`).

    ```
    git add .
    git commit -m "test(favorite-sources): verify delete favorite source endpoint via scalar."
    ```

11. **Verify the Spanish message.** Repeat the `404` case with the `acceptLanguage` field set to `es` (same as Feature 1, step 26) — the response comes back as "No se encontró la fuente favorita."

    ```
    git add .
    git commit -m "test(i18n): verify spanish message for delete favorite source."
    ```

12. **Publish and finish the feature.** Git Flow Helper widget → `Feature` → `Feature Publish`, then → `Feature Finish` (`Integrate Immediately`, `Keep remote branch when finished` unchecked).

---
## Add XML Doc Comments to Every Class

All five features are done and merged into `develop`. This single step adds `///` XML doc comments to every class, interface, record, and enum across `News` and `Shared` — one pass, one commit, instead of documenting each file the moment it's created. Writing the comments after the behavior is settled avoids rewriting a `<summary>` every time a method's contract shifts mid-feature.

Still in **File System** view, add one property to `Acme.CatchUp.Platform.csproj`, in the same `<PropertyGroup>` as `<Version>`:

```xml
<GenerateDocumentationFile>true</GenerateDocumentationFile>
```

This turns on the compiler's `CS1591` warning ("missing XML comment for publicly visible member") for every public type and member — the comments below are what the Verify step at the end checks are all there, by that warning staying gone.

Switch Solution Explorer to **Solution** view and open each file below; replace it with the version shown.

<details>
<summary>Shared/Application/Patterns/Result.cs</summary>

```csharp
namespace Acme.CatchUp.Platform.Shared.Application.Patterns;

/// <summary>
/// Represents the outcome of an operation that can succeed with a value or fail with an error.
/// </summary>
/// <typeparam name="TValue">The type of the successful result value.</typeparam>
/// <typeparam name="TError">The type of the error information.</typeparam>
public abstract record Result<TValue, TError>
{
    /// <summary>
    /// Represents a successful operation outcome.
    /// </summary>
    /// <param name="Value">The resulting value from the successful operation.</param>
    public sealed record Success(TValue Value) : Result<TValue, TError>;

    /// <summary>
    /// Represents a failed operation outcome.
    /// </summary>
    /// <param name="Error">The error information from the failed operation.</param>
    public sealed record Failure(TError Error) : Result<TValue, TError>;

    /// <summary>
    /// Determines whether the result represents a successful outcome.
    /// </summary>
    public bool IsSuccess => this is Success;

    /// <summary>
    /// Determines whether the result represents a failed outcome.
    /// </summary>
    public bool IsFailure => this is Failure;

    /// <summary>
    /// Applies a transformation function to the success value if the result is successful.
    /// </summary>
    /// <typeparam name="TNext">The type of the next result value.</typeparam>
    /// <param name="onSuccess">Function to apply if successful.</param>
    /// <returns>A new Result with the transformed value, or the current Failure.</returns>
    public Result<TNext, TError> Map<TNext>(Func<TValue, TNext> onSuccess) =>
        this switch
        {
            Success s => new Result<TNext, TError>.Success(onSuccess(s.Value)),
            Failure f => new Result<TNext, TError>.Failure(f.Error),
            _ => throw new InvalidOperationException("Unknown Result type")
        };

    /// <summary>
    /// Applies a function to either the success or failure case.
    /// </summary>
    /// <typeparam name="TResult">The type of the final result.</typeparam>
    /// <param name="onSuccess">Function to apply if successful.</param>
    /// <param name="onFailure">Function to apply if failed.</param>
    /// <returns>The result of applying the appropriate function.</returns>
    public TResult Fold<TResult>(Func<TValue, TResult> onSuccess, Func<TError, TResult> onFailure) =>
        this switch
        {
            Success s => onSuccess(s.Value),
            Failure f => onFailure(f.Error),
            _ => throw new InvalidOperationException("Unknown Result type")
        };

    /// <summary>
    /// Executes an action based on the result type without transforming the value.
    /// </summary>
    /// <param name="onSuccess">Action to execute if successful.</param>
    /// <param name="onFailure">Action to execute if failed.</param>
    public void Match(Action<TValue> onSuccess, Action<TError> onFailure)
    {
        if (this is Success s)
            onSuccess(s.Value);
        else if (this is Failure f)
            onFailure(f.Error);
    }
}
```
</details>

<details>
<summary>Shared/Domain/Repositories/IBaseRepository.cs and IUnitOfWork.cs</summary>

```csharp
namespace Acme.CatchUp.Platform.Shared.Domain.Repositories;

/// <summary>
///     Base repository interface for all repositories
/// </summary>
/// <remarks>
///     This interface defines the basic CRUD operations for all repositories
/// </remarks>
/// <typeparam name="TEntity">The Entity Type</typeparam>
public interface IBaseRepository<TEntity>
{
    /// <summary>
    ///     Add entity to the repository
    /// </summary>
    /// <param name="entity">Entity object to add</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>A task representing the asynchronous add operation.</returns>
    Task AddAsync(TEntity entity, CancellationToken cancellationToken = default);

    /// <summary>
    ///     Find entity by id
    /// </summary>
    /// <param name="id">The Entity ID to Find</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>A task representing the asynchronous read operation, containing the entity if found, or null otherwise.</returns>
    Task<TEntity?> FindByIdAsync(int id, CancellationToken cancellationToken = default);

    /// <summary>
    ///     Update entity
    /// </summary>
    /// <param name="entity">The entity object to update</param>
    void Update(TEntity entity);

    /// <summary>
    ///     Remove and entity
    /// </summary>
    /// <param name="entity">The entity object to remove</param>
    void Remove(TEntity entity);

    /// <summary>
    ///     Get All entities
    /// </summary>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>An Enumerable containing all entity objects</returns>
    Task<IEnumerable<TEntity>> ListAsync(CancellationToken cancellationToken = default);
}
```

```csharp
namespace Acme.CatchUp.Platform.Shared.Domain.Repositories;

/// <summary>
///     Unit of work interface
/// </summary>
/// <remarks>
///     This interface defines the basic operations for a unit of work
/// </remarks>
public interface IUnitOfWork
{
    /// <summary>
    ///     Commit changes to the database
    /// </summary>
    Task CompleteAsync(CancellationToken cancellationToken = default);
}
```
</details>

<details>
<summary>Shared/Infrastructure/Persistence/EFC/Repositories/BaseRepository.cs and UnitOfWork.cs</summary>

```csharp
using Acme.CatchUp.Platform.Shared.Domain.Repositories;
using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;
using Microsoft.EntityFrameworkCore;

namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;

/// <summary>
///     Generic base repository providing CRUD operations for all entity types.
/// </summary>
/// <remarks>
///     This generic repository implements the IBaseRepository contract and provides
///     common CRUD operations against an EF Core DbContext. Derived repositories
///     can extend this class to add entity-specific query methods.
/// </remarks>
/// <typeparam name="TEntity">The entity type that this repository manages.</typeparam>
/// <param name="context">The EF Core database context.</param>
public class BaseRepository<TEntity>(AppDbContext context) : IBaseRepository<TEntity> where TEntity : class
{
    /// <summary>
    ///     The EF Core <see cref="AppDbContext"/> instance available to derived repositories.
    /// </summary>
    protected readonly AppDbContext Context = context;

    /// <inheritdoc />
    public async Task AddAsync(TEntity entity, CancellationToken cancellationToken = default)
    {
        await Context.Set<TEntity>().AddAsync(entity, cancellationToken);
    }

    /// <inheritdoc />
    public async Task<TEntity?> FindByIdAsync(int id, CancellationToken cancellationToken = default)
    {
        return await Context.Set<TEntity>().FindAsync([id], cancellationToken);
    }

    /// <inheritdoc />
    public void Update(TEntity entity)
    {
        Context.Set<TEntity>().Update(entity);
    }

    /// <inheritdoc />
    public void Remove(TEntity entity)
    {
        Context.Set<TEntity>().Remove(entity);
    }

    /// <inheritdoc />
    public async Task<IEnumerable<TEntity>> ListAsync(CancellationToken cancellationToken = default)
    {
        return await Context.Set<TEntity>().ToListAsync(cancellationToken);
    }
}
```

```csharp
using Acme.CatchUp.Platform.Shared.Domain.Repositories;
using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;

namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;

/// <summary>
///     Unit of work implementation for coordinating database transactions.
/// </summary>
/// <remarks>
///     This class implements the IUnitOfWork contract and encapsulates EF Core's
///     SaveChangesAsync method. It represents a single database transaction scope,
///     ensuring that all pending entity changes are persisted atomically.
/// </remarks>
/// <param name="context">The EF Core database context.</param>
public class UnitOfWork(AppDbContext context) : IUnitOfWork
{
    /// <inheritdoc />
    /// <remarks>
    ///     Persists all pending entity changes (Add, Modify, Remove) to the database
    ///     in a single atomic transaction.
    /// </remarks>
    public async Task CompleteAsync(CancellationToken cancellationToken = default)
    {
        await context.SaveChangesAsync(cancellationToken);
    }
}
```
</details>

<details>
<summary>Shared/Domain/Model/IAuditableEntity.cs and Shared/Infrastructure/Persistence/EFC/Interceptors/AuditableEntityInterceptor.cs</summary>

```csharp
namespace Acme.CatchUp.Platform.Shared.Domain.Model;

/// <summary>
///     Marks an entity as carrying audit timestamps managed by the persistence layer.
/// </summary>
/// <remarks>
///     Any entity in any bounded context that implements this interface will automatically
///     have <see cref="CreatedAt"/> set once on first persistence and <see cref="UpdatedAt"/>
///     refreshed on every subsequent save, via <c>AuditableEntityInterceptor</c>.
/// </remarks>
public interface IAuditableEntity
{
    /// <summary>
    ///     Gets or sets the UTC timestamp when the entity was first persisted.
    /// </summary>
    DateTimeOffset? CreatedAt { get; set; }

    /// <summary>
    ///     Gets or sets the UTC timestamp when the entity was last saved.
    /// </summary>
    DateTimeOffset? UpdatedAt { get; set; }
}
```

```csharp
using Acme.CatchUp.Platform.Shared.Domain.Model;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Diagnostics;

namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Interceptors;

/// <summary>
///     EF Core interceptor that automatically populates audit timestamps on any entity
///     that implements <see cref="IAuditableEntity"/>.
/// </summary>
/// <remarks>
///     <list type="bullet">
///         <item><description><c>CreatedAt</c> — set once when the entity is first added.</description></item>
///         <item><description><c>UpdatedAt</c> — refreshed on every add or update.</description></item>
///     </list>
///     Registered in <c>AppDbContext.OnConfiguring</c> so it applies to all bounded contexts
///     sharing the same <see cref="DbContext"/>.
/// </remarks>
public sealed class AuditableEntityInterceptor : SaveChangesInterceptor
{
    /// <inheritdoc />
    public override InterceptionResult<int> SavingChanges(DbContextEventData eventData, InterceptionResult<int> result)
    {
        ApplyAuditTimestamps(eventData.Context);
        return base.SavingChanges(eventData, result);
    }

    /// <inheritdoc />
    public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData eventData,
        InterceptionResult<int> result,
        CancellationToken cancellationToken = default)
    {
        ApplyAuditTimestamps(eventData.Context);
        return base.SavingChangesAsync(eventData, result, cancellationToken);
    }

    private static void ApplyAuditTimestamps(DbContext? context)
    {
        if (context is null) return;

        var now = DateTimeOffset.UtcNow;

        foreach (var entry in context.ChangeTracker.Entries<IAuditableEntity>())
        {
            if (entry.State is EntityState.Added or EntityState.Modified) entry.Entity.UpdatedAt = now;
            if (entry.State == EntityState.Added) entry.Entity.CreatedAt ??= now;
        }
    }
}
```
</details>

<details>
<summary>Shared/Infrastructure/Persistence/EFC/Configuration/Extensions/StringExtensions.cs and ModelBuilderExtensions.cs</summary>

```csharp
using Humanizer;

namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration.Extensions;

/// <summary>
///     String extensions
/// </summary>
/// <remarks>
///     This class contains extension methods for strings.
///     It includes methods to convert strings to snake case and pluralize them.
/// </remarks>
public static class StringExtensions
{
    /// <summary>
    ///     Convert a string to snake case
    /// </summary>
    /// <param name="text">The string to convert</param>
    /// <returns>The string converted to snake case</returns>
    public static string ToSnakeCase(this string text)
    {
        return new string(Convert(text.GetEnumerator()).ToArray());

        static IEnumerable<char> Convert(CharEnumerator e)
        {
            if (!e.MoveNext()) yield break;

            yield return char.ToLower(e.Current);

            while (e.MoveNext())
                if (char.IsUpper(e.Current))
                {
                    yield return '_';
                    yield return char.ToLower(e.Current);
                }
                else
                {
                    yield return e.Current;
                }
        }
    }

    /// <summary>
    ///     Pluralize a string
    /// </summary>
    /// <param name="text">The string to convert</param>
    /// <returns>The string converted to plural</returns>
    public static string ToPlural(this string text)
    {
        return text.Pluralize(false);
    }
}
```

```csharp
using Microsoft.EntityFrameworkCore;

namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration.Extensions;

/// <summary>
///     Model builder extensions
/// </summary>
/// <remarks>
///     This class contains extension methods for the model builder.
///     It includes a method to use the snake case and/or plural naming convention according to an object type.
///     It also pluralizes the table names.
/// </remarks>
public static class ModelBuilderExtensions
{
    /// <summary>
    ///     Use snake case naming convention
    /// </summary>
    /// <remarks>
    ///     This method sets the naming convention for the database tables, columns, keys, foreign keys and indexes to snake
    ///     case.
    /// </remarks>
    public static void UseSnakeCaseNamingConvention(this ModelBuilder builder)
    {
        foreach (var entity in builder.Model.GetEntityTypes())
        {
            var tableName = entity.GetTableName();
            if (!string.IsNullOrEmpty(tableName)) entity.SetTableName(tableName.ToPlural().ToSnakeCase());

            foreach (var property in entity.GetProperties())
                property.SetColumnName(property.GetColumnName().ToSnakeCase());

            foreach (var key in entity.GetKeys())
            {
                var keyName = key.GetName();
                if (!string.IsNullOrEmpty(keyName)) key.SetName(keyName.ToSnakeCase());
            }

            foreach (var foreignKey in entity.GetForeignKeys())
            {
                var foreignKeyName = foreignKey.GetConstraintName();
                if (!string.IsNullOrEmpty(foreignKeyName)) foreignKey.SetConstraintName(foreignKeyName.ToSnakeCase());
            }

            foreach (var index in entity.GetIndexes())
            {
                var indexDatabaseName = index.GetDatabaseName();
                if (!string.IsNullOrEmpty(indexDatabaseName)) index.SetDatabaseName(indexDatabaseName.ToSnakeCase());
            }
        }
    }
}
```
</details>

<details>
<summary>Shared/Infrastructure/Interfaces/ASP/Configuration/Extensions/StringExtensions.cs and KebabCaseRouteNamingConvention.cs</summary>

```csharp
using System.Text.RegularExpressions;

namespace Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration.Extensions;

/// <summary>
///     String utility extensions used by the ASP.NET Core infrastructure layer.
/// </summary>
public static partial class StringExtensions
{
    /// <summary>
    ///     Converts a PascalCase or camelCase string to kebab-case.
    /// </summary>
    /// <param name="text">The string to convert.</param>
    /// <returns>The kebab-case representation of <paramref name="text"/>.</returns>
    public static string ToKebabCase(this string text)
    {
        if (string.IsNullOrEmpty(text)) return text;

        return KebabCaseRegex().Replace(text, "-$1")
            .Trim()
            .ToLower();
    }

    [GeneratedRegex("(?<!^)([A-Z][a-z]|(?<=[a-z])[A-Z])", RegexOptions.Compiled)]
    private static partial Regex KebabCaseRegex();
}
```

```csharp
using Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration.Extensions;
using Microsoft.AspNetCore.Mvc.ApplicationModels;

namespace Acme.CatchUp.Platform.Shared.Infrastructure.Interfaces.ASP.Configuration;

/// <summary>
///     An MVC controller-model convention that rewrites <c>[controller]</c> route tokens
///     to their kebab-case equivalents (e.g. <c>FavoriteSources</c> → <c>favorite-sources</c>).
/// </summary>
public class KebabCaseRouteNamingConvention : IControllerModelConvention
{
    /// <summary>
    ///     Applies the kebab-case naming transformation to all selectors on the controller and its actions.
    /// </summary>
    /// <param name="controller">The controller model to transform.</param>
    public void Apply(ControllerModel controller)
    {
        foreach (var selector in controller.Selectors)
            selector.AttributeRouteModel = ReplaceControllerTemplate(selector, controller.ControllerName);

        foreach (var selector in controller.Actions.SelectMany(a => a.Selectors))
            selector.AttributeRouteModel = ReplaceControllerTemplate(selector, controller.ControllerName);
    }

    private static AttributeRouteModel? ReplaceControllerTemplate(SelectorModel selector, string name)
    {
        return selector.AttributeRouteModel != null
            ? new AttributeRouteModel
            {
                Template = selector.AttributeRouteModel.Template?.Replace("[controller]", name.ToKebabCase())
            }
            : null;
    }
}
```
</details>

<details>
<summary>Shared/Infrastructure/Persistence/EFC/Configuration/AppDbContext.cs</summary>

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration.Extensions;
using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Interceptors;
using Microsoft.EntityFrameworkCore;

namespace Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;

/// <summary>
///     Application database context
/// </summary>
public class AppDbContext(DbContextOptions options) : DbContext(options)
{
    /// <inheritdoc />
    protected override void OnConfiguring(DbContextOptionsBuilder builder)
    {
        // Apply audit timestamp interceptor for all IAuditableEntity implementations
        builder.AddInterceptors(new AuditableEntityInterceptor());
        base.OnConfiguring(builder);
    }

    /// <inheritdoc />
    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);

        builder.Entity<FavoriteSource>().HasKey(f => f.Id);
        builder.Entity<FavoriteSource>().Property(f => f.Id).IsRequired().ValueGeneratedOnAdd();
        builder.Entity<FavoriteSource>()
            .Property(f => f.SourceId)
            .HasConversion(valueObject => valueObject.Value, value => new SourceId(value))
            .IsRequired();

        builder.Entity<FavoriteSource>()
            .Property(f => f.NewsApiKey)
            .HasConversion(valueObject => valueObject.Value, value => new NewsApiKey(value))
            .IsRequired();
        builder.Entity<FavoriteSource>()
            .HasIndex(f => new { f.NewsApiKey, f.SourceId })
            .IsUnique();

        builder.UseSnakeCaseNamingConvention();
    }
}
```
</details>

<details>
<summary>News/Domain/Model/ValueObjects/NewsApiKey.cs and SourceId.cs</summary>

```csharp
namespace Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

/// <summary>
///     Value object representing a validated News API key.
/// </summary>
public sealed record NewsApiKey
{
    private const int MaxLength = 255;

    /// <summary>
    ///     Initializes a new <see cref="NewsApiKey"/> with the given raw string value.
    /// </summary>
    /// <param name="value">The raw News API key string. Must be non-null, non-whitespace, and at most 255 characters.</param>
    /// <exception cref="ArgumentException">Thrown when <paramref name="value"/> is null, empty, whitespace, or exceeds the maximum length.</exception>
    public NewsApiKey(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            throw new ArgumentException("NewsApiKey cannot be null, empty, or whitespace.", nameof(value));

        if (value.Length > MaxLength)
            throw new ArgumentException($"NewsApiKey cannot be longer than {MaxLength} characters.", nameof(value));

        Value = value;
    }

    /// <summary>
    ///     Gets the underlying primitive value.
    /// </summary>
    public string Value { get; }

    /// <inheritdoc />
    public override string ToString() => Value;

    /// <summary>
    ///    Returns a masked version of the NewsApiKey for logging purposes, showing only the last 4 characters.
    /// </summary>
    /// <returns>
    ///   A string with all but the last 4 characters replaced by asterisks. If the key is 4 characters or shorter, all characters are masked.
    /// </returns>
    public string MaskedForLogging() => MaskedForLogging(Value);

    /// <summary>
    ///     Masks a raw News API key for logging, including values that have not passed validation.
    /// </summary>
    /// <param name="value">The raw key value to mask.</param>
    /// <returns>The masked key, showing only its last four characters when available.</returns>
    public static string MaskedForLogging(string? value) =>
        string.IsNullOrEmpty(value)
            ? string.Empty
            : value.Length <= 4
                ? new string('*', value.Length)
                : new string('*', value.Length - 4) + value[^4..];
}
```

```csharp
namespace Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

/// <summary>
///     Value object representing a validated source identifier.
/// </summary>
public sealed record SourceId
{
    private const int MaxLength = 255;

    /// <summary>
    ///     Initializes a new <see cref="SourceId"/> with the given raw string value.
    /// </summary>
    /// <param name="value">The raw source identifier string. Must be non-null, non-whitespace, and at most 255 characters.</param>
    /// <exception cref="ArgumentException">Thrown when <paramref name="value"/> is null, empty, whitespace, or exceeds the maximum length.</exception>
    public SourceId(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            throw new ArgumentException("SourceId cannot be null, empty, or whitespace.", nameof(value));

        if (value.Length > MaxLength)
            throw new ArgumentException($"SourceId cannot be longer than {MaxLength} characters.", nameof(value));

        Value = value;
    }

    /// <summary>
    ///     Gets the underlying primitive value.
    /// </summary>
    public string Value { get; }

    /// <inheritdoc />
    public override string ToString() => Value;
}
```
</details>

<details>
<summary>News/Domain/Model/Commands/CreateFavoriteSourceCommand.cs and DeleteFavoriteSourceCommand.cs</summary>

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

namespace Acme.CatchUp.Platform.News.Domain.Model.Commands;

/// <summary>
///     Command to create a favorite news source
/// </summary>
/// <param name="NewsApiKey">The NewsApiKey obtained from provider</param>
/// <param name="SourceId">The SourceId of the news source</param>
public record CreateFavoriteSourceCommand(NewsApiKey NewsApiKey, SourceId SourceId);
```

```csharp
namespace Acme.CatchUp.Platform.News.Domain.Model.Commands;

/// <summary>
///     Command to remove a favorite news source by its server-generated id.
/// </summary>
public sealed record DeleteFavoriteSourceCommand
{
    /// <summary>
    ///     Initializes a new <see cref="DeleteFavoriteSourceCommand"/> with the given id.
    /// </summary>
    /// <param name="id">The favorite source id to remove. Must be a positive integer.</param>
    /// <exception cref="ArgumentException">Thrown when <paramref name="id"/> is zero or negative.</exception>
    public DeleteFavoriteSourceCommand(int id)
    {
        if (id <= 0)
            throw new ArgumentException("Id must be a positive integer.", nameof(id));

        Id = id;
    }

    /// <summary>
    ///     Gets the id of the favorite source to remove.
    /// </summary>
    public int Id { get; }
}
```
</details>

<details>
<summary>News/Domain/Model/Aggregates/FavoriteSource.cs and FavoriteSourceAudit.cs</summary>

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.Commands;
using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

namespace Acme.CatchUp.Platform.News.Domain.Model.Aggregates;

/// <summary>
///     Represents a favorite news source for a given News API key.
/// </summary>
/// <remarks>
///     This aggregate root represents a user's favorite news source.
///     The combination of (NewsApiKey, SourceId) must be unique within the system.
///     Instances are immutable after creation; updates are handled through commands.
///     Audit timestamps (CreatedAt, UpdatedAt) are managed by the persistence layer.
/// </remarks>
public partial class FavoriteSource
{
    /// <summary>
    ///     Protected parameterless constructor for EF Core.
    /// </summary>
    protected FavoriteSource()
    {
        NewsApiKey = null!;
        SourceId = null!;
    }

    /// <summary>
    ///     Creates a new FavoriteSource from a creation command.
    /// </summary>
    /// <remarks>
    ///     This constructor acts as the command handler for CreateFavoriteSourceCommand.
    ///     It initializes the aggregate with the required NewsApiKey and SourceId.
    /// </remarks>
    /// <param name="command">The CreateFavoriteSourceCommand command</param>
    /// <exception cref="ArgumentNullException">Thrown when command is null.</exception>
    public FavoriteSource(CreateFavoriteSourceCommand command)
    {
        ArgumentNullException.ThrowIfNull(command);

        NewsApiKey = command.NewsApiKey;
        SourceId = command.SourceId;
    }

    /// <summary>
    ///     Gets the server-generated identifier for this favorite source.
    /// </summary>
    public int Id { get; private set; }

    /// <summary>
    ///     Gets the News API key associated with this favorite source.
    /// </summary>
    public NewsApiKey NewsApiKey { get; private set; }

    /// <summary>
    ///     Gets the source identifier from the News API.
    /// </summary>
    public SourceId SourceId { get; private set; }
}
```

```csharp
using Acme.CatchUp.Platform.Shared.Domain.Model;

namespace Acme.CatchUp.Platform.News.Domain.Model.Aggregates;

/// <summary>
///     Audit extension for FavoriteSource aggregate.
/// </summary>
/// <remarks>
///     This partial class extends FavoriteSource with audit trail properties.
///     CreatedAt and UpdatedAt are automatically managed by the persistence layer
///     via the AuditableEntityInterceptor in the Shared bounded context.
///     Implements the IAuditableEntity interface to provide these properties.
/// </remarks>
public partial class FavoriteSource : IAuditableEntity
{
    /// <summary>
    ///     Gets the timestamp when this favorite source was created.
    /// </summary>
    /// <remarks>
    ///     Automatically set once by the persistence layer on first insert. Maps to column: created_at.
    /// </remarks>
    public DateTimeOffset? CreatedAt { get; set; }

    /// <summary>
    ///     Gets the timestamp when this favorite source was last updated.
    /// </summary>
    /// <remarks>
    ///     Automatically refreshed by the persistence layer on every save. Maps to column: updated_at.
    /// </remarks>
    public DateTimeOffset? UpdatedAt { get; set; }
}
```
</details>

<details>
<summary>News/Application/Errors/CreateFavoriteSourceError.cs and DeleteFavoriteSourceError.cs</summary>

```csharp
namespace Acme.CatchUp.Platform.News.Application.Errors;

/// <summary>
/// Represents errors that can occur when creating a favorite source.
/// </summary>
public enum CreateFavoriteSourceError
{
    /// <summary>
    /// The provided news API key is invalid.
    /// </summary>
    InvalidNewsApiKey,

    /// <summary>
    /// The provided source ID is invalid.
    /// </summary>
    InvalidSourceId,

    /// <summary>
    /// A favorite source with the same API key and source ID already exists.
    /// </summary>
    DuplicateFavoriteSource,

    /// <summary>
    /// An unexpected error occurred during the operation.
    /// </summary>
    UnexpectedError
}
```

```csharp
namespace Acme.CatchUp.Platform.News.Application.Errors;

/// <summary>
/// Represents errors that can occur when removing a favorite source.
/// </summary>
public enum DeleteFavoriteSourceError
{
    /// <summary>
    /// No favorite source matches the given id.
    /// </summary>
    NotFound,

    /// <summary>
    /// An unexpected error occurred during the operation.
    /// </summary>
    UnexpectedError
}
```
</details>

<details>
<summary>News/Domain/Repositories/IFavoriteSourceRepository.cs and Infrastructure/.../FavoriteSourceRepository.cs</summary>

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
using Acme.CatchUp.Platform.Shared.Domain.Repositories;

namespace Acme.CatchUp.Platform.News.Domain.Repositories;

/// <summary>
///     The Favorite source repository contract.
/// </summary>
public interface IFavoriteSourceRepository : IBaseRepository<FavoriteSource>
{
    /// <summary>
    ///     Find favorite sources by News API Key
    /// </summary>
    /// <param name="newsApiKey">The News API Key to search.</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     An Enumerable containing the favorite source objects if found, or empty otherwise.
    /// </returns>
    Task<IEnumerable<FavoriteSource>> FindByNewsApiKeyAsync(NewsApiKey newsApiKey,
        CancellationToken cancellationToken = default);

    /// <summary>
    ///     Find a favorite source by News API Key and Source ID
    /// </summary>
    /// <param name="newsApiKey">The News API Key</param>
    /// <param name="sourceId">The Source ID</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     The favorite source object if found, or null otherwise.
    /// </returns>
    Task<FavoriteSource?> FindByNewsApiKeyAndSourceIdAsync(NewsApiKey newsApiKey, SourceId sourceId,
        CancellationToken cancellationToken = default);
}
```

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
using Acme.CatchUp.Platform.News.Domain.Repositories;
using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Configuration;
using Acme.CatchUp.Platform.Shared.Infrastructure.Persistence.EFC.Repositories;
using Microsoft.EntityFrameworkCore;

namespace Acme.CatchUp.Platform.News.Infrastructure.Persistence.EFC.Repositories;

/// <summary>
///     Entity Framework repository for favorite source persistence.
/// </summary>
/// <remarks>
///     This repository extends the generic BaseRepository with favorite-source-specific query methods.
///     It provides methods to find favorites by NewsApiKey and by composite key (NewsApiKey + SourceId).
/// </remarks>
/// <param name="context">The EF Core database context.</param>
public class FavoriteSourceRepository(AppDbContext context)
    : BaseRepository<FavoriteSource>(context), IFavoriteSourceRepository
{
    /// <inheritdoc />
    public async Task<IEnumerable<FavoriteSource>> FindByNewsApiKeyAsync(NewsApiKey newsApiKey,
        CancellationToken cancellationToken = default)
    {
        return await Context.Set<FavoriteSource>()
            .Where(f => f.NewsApiKey == newsApiKey)
            .ToListAsync(cancellationToken);
    }

    /// <inheritdoc />
    public async Task<FavoriteSource?> FindByNewsApiKeyAndSourceIdAsync(NewsApiKey newsApiKey, SourceId sourceId,
        CancellationToken cancellationToken = default)
    {
        return await Context.Set<FavoriteSource>()
            .FirstOrDefaultAsync(f => f.NewsApiKey == newsApiKey && f.SourceId == sourceId,
                cancellationToken);
    }
}
```
</details>

<details>
<summary>News/Domain/Model/Queries/*.cs (all three queries)</summary>

```csharp
namespace Acme.CatchUp.Platform.News.Domain.Model.Queries;

/// <summary>
///     Query to get a favorite source by ID
/// </summary>
/// <param name="Id">The Source ID to search</param>
public record GetFavoriteSourceByIdQuery(int Id);
```

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

namespace Acme.CatchUp.Platform.News.Domain.Model.Queries;

/// <summary>
///     Query to get all favorite sources by NewsApiKey
/// </summary>
/// <param name="NewsApiKey">The NewsApiKey to search</param>
public record GetAllFavoriteSourcesByNewsApiKeyQuery(NewsApiKey NewsApiKey);
```

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;

namespace Acme.CatchUp.Platform.News.Domain.Model.Queries;

/// <summary>
///     Query to get a favorite source by NewsApiKey and SourceId
/// </summary>
/// <param name="NewsApiKey">The NewsApiKey to search</param>
/// <param name="SourceId">The Source ID to search</param>
public record GetFavoriteSourceByNewsApiKeyAndSourceIdQuery(NewsApiKey NewsApiKey, SourceId SourceId);
```
</details>

<details>
<summary>News/Application/Services/IFavoriteSourceCommandService.cs and IFavoriteSourceQueryService.cs</summary>

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
using Acme.CatchUp.Platform.News.Domain.Model.Commands;
using Acme.CatchUp.Platform.News.Application.Errors;
using Acme.CatchUp.Platform.Shared.Application.Patterns;

namespace Acme.CatchUp.Platform.News.Application.Services;

/// <summary>
///     Command service interface for favorite source operations.
/// </summary>
/// <remarks>
///     This interface defines the contract for handling favorite source commands:
///     creating a new favorite source, and removing an existing one by id.
/// </remarks>
public interface IFavoriteSourceCommandService
{
    /// <summary>
    ///     Handle the create favorite source command.
    /// </summary>
    /// <param name="command">CreateFavoriteSourceCommand command</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     A Result containing the created FavoriteSource on success, or the error reason on failure.
    /// </returns>
    Task<Result<FavoriteSource, CreateFavoriteSourceError>> Handle(CreateFavoriteSourceCommand command, CancellationToken cancellationToken = default);

    /// <summary>
    ///     Handle the delete favorite source command.
    /// </summary>
    /// <param name="command">DeleteFavoriteSourceCommand command</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     A Result containing the removed favorite source's id on success, or the error reason on failure.
    /// </returns>
    Task<Result<int, DeleteFavoriteSourceError>> Handle(DeleteFavoriteSourceCommand command, CancellationToken cancellationToken = default);
}
```

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
using Acme.CatchUp.Platform.News.Domain.Model.Queries;

namespace Acme.CatchUp.Platform.News.Application.Services;

/// <summary>
///     Query service interface for favorite source operations.
/// </summary>
/// <remarks>
///     This interface defines the contract for querying favorite sources by various filters.
///     It supports queries by NewsApiKey, ID, and composite key (NewsApiKey + SourceId).
/// </remarks>
public interface IFavoriteSourceQueryService
{
    /// <summary>
    ///     Handle the GetAllFavoriteSourcesByNewsApiKeyQuery.
    /// </summary>
    /// <param name="query">The GetAllFavoriteSourcesByNewsApiKeyQuery query</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>An IEnumerable containing the FavoriteSource objects</returns>
    Task<IEnumerable<FavoriteSource>> Handle(GetAllFavoriteSourcesByNewsApiKeyQuery query,
        CancellationToken cancellationToken = default);

    /// <summary>
    ///     Handle the GetFavoriteSourceByNewsApiKeyAndSourceIdQuery.
    /// </summary>
    /// <param name="query">The GetFavoriteSourceByNewsApiKeyAndSourceIdQuery query</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>The FavoriteSource object if found, or null otherwise</returns>
    Task<FavoriteSource?> Handle(GetFavoriteSourceByNewsApiKeyAndSourceIdQuery query,
        CancellationToken cancellationToken = default);

    /// <summary>
    ///     Handle the GetFavoriteSourceByIdQuery.
    /// </summary>
    /// <param name="query">The GetFavoriteSourceByIdQuery query</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     The FavoriteSource object if found, or null otherwise
    /// </returns>
    Task<FavoriteSource?> Handle(GetFavoriteSourceByIdQuery query, CancellationToken cancellationToken = default);
}
```
</details>

<details>
<summary>News/Application/Internal/CommandServices/FavoriteSourceCommandService.cs (full file)</summary>

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
using Acme.CatchUp.Platform.News.Application.Errors;
using Acme.CatchUp.Platform.News.Application.Services;
using Acme.CatchUp.Platform.News.Domain.Model.Commands;
using Acme.CatchUp.Platform.News.Domain.Repositories;
using Acme.CatchUp.Platform.Shared.Domain.Repositories;
using Acme.CatchUp.Platform.Shared.Application.Patterns;
using Microsoft.EntityFrameworkCore;

namespace Acme.CatchUp.Platform.News.Application.Internal.CommandServices;

/// <summary>
///     Application service for handling favorite source commands.
/// </summary>
/// <remarks>
///     This service acts as the command handler for creating and removing favorite sources.
///     It enforces duplicate detection through both application-level checks and database constraints,
///     then coordinates with the unit of work to persist changes.
///     Logs warnings for duplicate detections and errors for persistence failures.
/// </remarks>
/// <param name="favoriteSourceRepository">Repository for accessing favorite source data.</param>
/// <param name="unitOfWork">Unit of work for managing transaction scope.</param>
/// <param name="logger">Logger for diagnostic and error reporting.</param>
public class FavoriteSourceCommandService(
    IFavoriteSourceRepository favoriteSourceRepository,
    IUnitOfWork unitOfWork,
    ILogger<FavoriteSourceCommandService> logger)
    : IFavoriteSourceCommandService
{
    /// <inheritdoc />
    public async Task<Result<FavoriteSource, CreateFavoriteSourceError>> Handle(CreateFavoriteSourceCommand command,
        CancellationToken cancellationToken = default)
    {
        var existingSource =
            await favoriteSourceRepository.FindByNewsApiKeyAndSourceIdAsync(command.NewsApiKey, command.SourceId,
                cancellationToken);
        if (existingSource != null)
        {
            logger.LogWarning(
                "Duplicate favorite source rejected for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                command.NewsApiKey.MaskedForLogging(),
                command.SourceId);
            return new Result<FavoriteSource, CreateFavoriteSourceError>.Failure(
                CreateFavoriteSourceError.DuplicateFavoriteSource);
        }

        try
        {
            var favoriteSource = new FavoriteSource(command);
            await favoriteSourceRepository.AddAsync(favoriteSource, cancellationToken);
            await unitOfWork.CompleteAsync(cancellationToken);
            return new Result<FavoriteSource, CreateFavoriteSourceError>.Success(favoriteSource);
        }
        catch (ArgumentException ex)
        {
            logger.LogWarning(ex,
                "Invalid arguments while creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                command.NewsApiKey.MaskedForLogging(),
                command.SourceId);
            return new Result<FavoriteSource, CreateFavoriteSourceError>.Failure(
                CreateFavoriteSourceError.UnexpectedError);
        }
        catch (DbUpdateException ex) when (IsDuplicateKeyViolation(ex))
        {
            logger.LogWarning(ex,
                "Duplicate key violation creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                command.NewsApiKey.MaskedForLogging(),
                command.SourceId);
            return new Result<FavoriteSource, CreateFavoriteSourceError>.Failure(
                CreateFavoriteSourceError.DuplicateFavoriteSource);
        }
        catch (DbUpdateException ex)
        {
            logger.LogError(ex,
                "Database update failed creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                command.NewsApiKey.MaskedForLogging(),
                command.SourceId);
            return new Result<FavoriteSource, CreateFavoriteSourceError>.Failure(
                CreateFavoriteSourceError.UnexpectedError);
        }
        catch (OperationCanceledException) when (cancellationToken.IsCancellationRequested)
        {
            throw;
        }
        catch (Exception ex)
        {
            logger.LogError(ex,
                "Unexpected error creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                command.NewsApiKey.MaskedForLogging(),
                command.SourceId);
            return new Result<FavoriteSource, CreateFavoriteSourceError>.Failure(
                CreateFavoriteSourceError.UnexpectedError);
        }
    }

    /// <inheritdoc />
    public async Task<Result<int, DeleteFavoriteSourceError>> Handle(DeleteFavoriteSourceCommand command,
        CancellationToken cancellationToken = default)
    {
        var favoriteSource = await favoriteSourceRepository.FindByIdAsync(command.Id, cancellationToken);
        if (favoriteSource is null)
        {
            logger.LogDebug("Favorite source not found for deletion: id={Id}", command.Id);
            return new Result<int, DeleteFavoriteSourceError>.Failure(DeleteFavoriteSourceError.NotFound);
        }

        try
        {
            favoriteSourceRepository.Remove(favoriteSource);
            await unitOfWork.CompleteAsync(cancellationToken);
            logger.LogInformation("Favorite source removed: id={Id}", command.Id);
            return new Result<int, DeleteFavoriteSourceError>.Success(command.Id);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Unexpected error removing favorite source: id={Id}", command.Id);
            return new Result<int, DeleteFavoriteSourceError>.Failure(DeleteFavoriteSourceError.UnexpectedError);
        }
    }

    /// <summary>
    /// Determines whether a DbUpdateException represents a duplicate key constraint violation.
    /// </summary>
    /// <param name="exception">The exception to inspect.</param>
    /// <returns>True if the exception is due to a MySQL duplicate key error (code 1062), false otherwise.</returns>
    private static bool IsDuplicateKeyViolation(DbUpdateException exception)
    {
        for (Exception? current = exception; current is not null; current = current.InnerException)
        {
            if (!string.Equals(current.GetType().Name, "MySqlException", StringComparison.Ordinal)) continue;
            var numberProperty = current.GetType().GetProperty("Number");
            if (numberProperty?.PropertyType == typeof(int) &&
                numberProperty.GetValue(current) is int errorCode &&
                errorCode == 1062)
                return true;
        }
        return false;
    }
}
```
</details>

<details>
<summary>News/Application/Internal/QueryServices/FavoriteSourceQueryService.cs (full file)</summary>

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
using Acme.CatchUp.Platform.News.Application.Services;
using Acme.CatchUp.Platform.News.Domain.Model.Queries;
using Acme.CatchUp.Platform.News.Domain.Repositories;

namespace Acme.CatchUp.Platform.News.Application.Internal.QueryServices;

/// <summary>
///     Application service for querying favorite sources.
/// </summary>
/// <remarks>
///     This service acts as the query handler for retrieving favorite sources.
///     It supports queries by NewsApiKey, by ID, and by the composite key (NewsApiKey + SourceId).
///     All queries delegate to the repository layer for data access.
/// </remarks>
/// <param name="favoriteSourceRepository">Repository for accessing favorite source data.</param>
public class FavoriteSourceQueryService(IFavoriteSourceRepository favoriteSourceRepository)
    : IFavoriteSourceQueryService
{
    /// <inheritdoc />
    public async Task<IEnumerable<FavoriteSource>> Handle(GetAllFavoriteSourcesByNewsApiKeyQuery query,
        CancellationToken cancellationToken = default) =>
        await favoriteSourceRepository.FindByNewsApiKeyAsync(query.NewsApiKey, cancellationToken);

    /// <inheritdoc />
    public async Task<FavoriteSource?> Handle(GetFavoriteSourceByNewsApiKeyAndSourceIdQuery query,
        CancellationToken cancellationToken = default) =>
        await favoriteSourceRepository.FindByNewsApiKeyAndSourceIdAsync(query.NewsApiKey, query.SourceId,
            cancellationToken);

    /// <inheritdoc />
    public async Task<FavoriteSource?> Handle(GetFavoriteSourceByIdQuery query,
        CancellationToken cancellationToken = default) =>
        await favoriteSourceRepository.FindByIdAsync(query.Id, cancellationToken);
}
```
</details>

<details>
<summary>News/Interfaces/REST/Resources/CreateFavoriteSourceResource.cs and FavoriteSourceResource.cs</summary>

```csharp
using System.ComponentModel.DataAnnotations;

namespace Acme.CatchUp.Platform.News.Interfaces.REST.Resources;

/// <summary>
///     Represents the data required to create a favorite source.
/// </summary>
/// <param name="NewsApiKey">The News API Key provided by NewsAPI (required)</param>
/// <param name="SourceId">The source identifier from NewsAPI (required)</param>
public record CreateFavoriteSourceResource(
    [Required(ErrorMessage = "RequiredNewsApiKey")]
    string NewsApiKey,
    [Required(ErrorMessage = "RequiredSourceId")]
    string SourceId);
```

```csharp
namespace Acme.CatchUp.Platform.News.Interfaces.REST.Resources;

/// <summary>
///     Represents the data provided by the server about a favorite source.
/// </summary>
/// <param name="Id">The server-generated ID of the favorite source</param>
/// <param name="NewsApiKey">The News API Key provided by NewsAPI</param>
/// <param name="SourceId">The source identifier from NewsAPI</param>
public record FavoriteSourceResource(
    int Id,
    string NewsApiKey,
    string SourceId);
```
</details>

<details>
<summary>News/Interfaces/REST/Transform/CreateFavoriteSourceCommandFromResourceAssembler.cs and FavoriteSourceResourceFromEntityAssembler.cs</summary>

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.Commands;
using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
using Acme.CatchUp.Platform.News.Interfaces.REST.Resources;

namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

/// <summary>
///     Assembles a CreateFavoriteSourceCommand from a CreateFavoriteSourceResource.
/// </summary>
public static class CreateFavoriteSourceCommandFromResourceAssembler
{
    /// <summary>
    ///     Assembles a CreateFavoriteSourceCommand from a CreateFavoriteSourceResource.
    /// </summary>
    /// <param name="resource">The CreateFavoriteSourceResource resource</param>
    /// <returns>
    ///     A CreateFavoriteSourceCommand assembled from the CreateFavoriteSourceResource
    /// </returns>
    public static CreateFavoriteSourceCommand ToCommandFromResource(CreateFavoriteSourceResource resource) =>
        new(new NewsApiKey(resource.NewsApiKey), new SourceId(resource.SourceId));
}
```

```csharp
using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
using Acme.CatchUp.Platform.News.Interfaces.REST.Resources;

namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

/// <summary>
///     Assembles a FavoriteSourceResource from a FavoriteSource.
/// </summary>
public static class FavoriteSourceResourceFromEntityAssembler
{
    /// <summary>
    ///     Assembles a FavoriteSourceResource from a FavoriteSource.
    /// </summary>
    /// <param name="entity">The FavoriteSource entity</param>
    /// <returns>
    ///     A FavoriteSourceResource assembled from the FavoriteSource
    /// </returns>
    public static FavoriteSourceResource ToResourceFromEntity(FavoriteSource entity) =>
        new(entity.Id, entity.NewsApiKey.Value, entity.SourceId.Value);
}
```
</details>

<details>
<summary>News/Interfaces/REST/Transform/ActionResultFromCreateFavoriteSourceResultAssembler.cs and ActionResultFromDeleteFavoriteSourceResultAssembler.cs</summary>

```csharp
using Acme.CatchUp.Platform.News.Application.Errors;
using Acme.CatchUp.Platform.News.Domain.Model.Aggregates;
using Acme.CatchUp.Platform.Resources;
using Acme.CatchUp.Platform.Shared.Application.Patterns;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Localization;

namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

/// <summary>
///     Assembles an HTTP action result from the result of the command handling to create a favorite source in the application layer.
/// </summary>
public static class ActionResultFromCreateFavoriteSourceResultAssembler
{
    /// <summary>
    ///     Assembles an <see cref="ActionResult" /> from the result of a create favorite source operation.
    /// </summary>
    /// <param name="result">The application result produced by the application layer after handling the command to create a favorite source.</param>
    /// <param name="controller">The controller used to produce HTTP-specific action results.</param>
    /// <param name="localizer">The string localizer used for API response messages.</param>
    /// <param name="getFavoriteSourceByIdActionName">The name of the action used to generate the location header.</param>
    /// <returns>The HTTP action result corresponding to the application outcome.</returns>
    public static ActionResult ToActionResultFromCreateFavoriteSourceResult(
        Result<FavoriteSource, CreateFavoriteSourceError> result,
        ControllerBase controller,
        IStringLocalizer<SharedResource> localizer,
        string getFavoriteSourceByIdActionName) =>
        result switch
        {
            Result<FavoriteSource, CreateFavoriteSourceError>.Success success =>
                controller.CreatedAtAction(getFavoriteSourceByIdActionName, new { id = success.Value.Id },
                    FavoriteSourceResourceFromEntityAssembler.ToResourceFromEntity(success.Value)),

            Result<FavoriteSource, CreateFavoriteSourceError>.Failure failure =>
                failure.Error switch
                {
                    CreateFavoriteSourceError.DuplicateFavoriteSource =>
                        controller.Conflict(localizer["NewsFavoriteSourceDuplicated"].Value),

                    CreateFavoriteSourceError.UnexpectedError =>
                        controller.Problem(
                            title: localizer["UnexpectedServerError"].Value,
                            detail: localizer["UnexpectedErrorCreatingFavoriteSource"].Value,
                            statusCode: 500),

                    _ => controller.Problem(
                        title: localizer["UnexpectedServerError"].Value,
                        detail: localizer["UnexpectedErrorProcessingRequest"].Value,
                        statusCode: 500)
                },

            _ => controller.Problem(
                title: localizer["UnexpectedServerError"].Value,
                detail: localizer["UnexpectedErrorProcessingRequest"].Value,
                statusCode: 500)
        };
}
```

```csharp
using Acme.CatchUp.Platform.News.Application.Errors;
using Acme.CatchUp.Platform.Resources;
using Acme.CatchUp.Platform.Shared.Application.Patterns;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Localization;

namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

/// <summary>
///     Assembles an HTTP action result from the result of the command handling to remove a favorite source in the application layer.
/// </summary>
public static class ActionResultFromDeleteFavoriteSourceResultAssembler
{
    /// <summary>
    ///     Assembles an <see cref="ActionResult" /> from the result of a delete favorite source operation.
    /// </summary>
    /// <param name="result">The application result produced by the application layer after handling the command to remove a favorite source.</param>
    /// <param name="controller">The controller used to produce HTTP-specific action results.</param>
    /// <param name="localizer">The string localizer used for API response messages.</param>
    /// <returns>The HTTP action result corresponding to the application outcome.</returns>
    public static ActionResult ToActionResultFromDeleteFavoriteSourceResult(
        Result<int, DeleteFavoriteSourceError> result,
        ControllerBase controller,
        IStringLocalizer<SharedResource> localizer) =>
        result switch
        {
            Result<int, DeleteFavoriteSourceError>.Success => controller.NoContent(),

            Result<int, DeleteFavoriteSourceError>.Failure failure =>
                failure.Error switch
                {
                    DeleteFavoriteSourceError.NotFound =>
                        controller.NotFound(localizer["FavoriteSourceNotFound"].Value),

                    _ => controller.Problem(
                        title: localizer["UnexpectedServerError"].Value,
                        detail: localizer["UnexpectedErrorProcessingRequest"].Value,
                        statusCode: 500)
                },

            _ => controller.Problem(
                title: localizer["UnexpectedServerError"].Value,
                detail: localizer["UnexpectedErrorProcessingRequest"].Value,
                statusCode: 500)
        };
}
```
</details>

<details>
<summary>News/Interfaces/REST/Transform/LocalizedOpenApiAssembler.cs (full file, all four operations)</summary>

```csharp
using Acme.CatchUp.Platform.News.Interfaces.REST.Resources;
using Acme.CatchUp.Platform.Resources;
using Microsoft.AspNetCore.OpenApi;
using Microsoft.AspNetCore.Mvc.Controllers;
using Microsoft.Extensions.Localization;
using Microsoft.OpenApi;

namespace Acme.CatchUp.Platform.News.Interfaces.REST.Transform;

/// <summary>
///     Assembles localized OpenAPI metadata (summaries, descriptions, and response texts)
///     for the News REST endpoints and resources.
/// </summary>
public static class LocalizedOpenApiAssembler
{
    private static readonly Dictionary<string, (string Summary, string Description,
        IReadOnlyDictionary<string, string> Responses)> OperationResources =
        new Dictionary<string, (string, string, IReadOnlyDictionary<string, string>)>
        {
            ["CreateFavoriteSource"] = (
                "CreateFavoriteSourceSummary",
                "CreateFavoriteSourceDescription",
                new Dictionary<string, string>
                {
                    ["201"] = "CreateFavoriteSourceCreated",
                    ["400"] = "CreateFavoriteSourceBadRequest",
                    ["409"] = "CreateFavoriteSourceConflict",
                    ["500"] = "CreateFavoriteSourceServerError"
                }),
            ["GetFavoriteSourceById"] = (
                "GetFavoriteSourceByIdSummary",
                "GetFavoriteSourceByIdDescription",
                new Dictionary<string, string>
                {
                    ["200"] = "GetFavoriteSourceByIdSuccess",
                    ["404"] = "GetFavoriteSourceByIdNotFound"
                }),
            ["GetFavoriteSourceFromQuery"] = (
                "GetFavoriteSourceQuerySummary",
                "GetFavoriteSourceQueryDescription",
                new Dictionary<string, string>
                {
                    ["200"] = "GetFavoriteSourceQuerySuccess",
                    ["400"] = "GetFavoriteSourceQueryBadRequest",
                    ["404"] = "GetFavoriteSourceQueryNotFound"
                }),
            ["DeleteFavoriteSource"] = (
                "DeleteFavoriteSourceSummary",
                "DeleteFavoriteSourceDescription",
                new Dictionary<string, string>
                {
                    ["204"] = "DeleteFavoriteSourceNoContent",
                    ["404"] = "DeleteFavoriteSourceNotFound"
                })
        };

    /// <summary>
    ///     Localizes the summary, description, operation id and response descriptions of known operations.
    /// </summary>
    /// <param name="operation">The OpenAPI operation being transformed.</param>
    /// <param name="context">The transformer context, used to resolve the localizer and action descriptor.</param>
    /// <param name="cancellationToken">The cancellation token.</param>
    /// <returns>A completed task.</returns>
    public static Task TransformOperationAsync(
        OpenApiOperation operation,
        OpenApiOperationTransformerContext context,
        CancellationToken cancellationToken)
    {
        if (context.Description.ActionDescriptor is not ControllerActionDescriptor action ||
            !OperationResources.TryGetValue(action.ActionName, out var resources))
            return Task.CompletedTask;

        var localizer = context.ApplicationServices.GetRequiredService<IStringLocalizer<SharedResource>>();
        operation.OperationId = action.ActionName;
        operation.Summary = localizer[resources.Summary].Value;
        operation.Description = localizer[resources.Description].Value;
        if (operation.Responses is null) return Task.CompletedTask;

        foreach (var (statusCode, resourceKey) in resources.Responses)
        {
            if (operation.Responses.TryGetValue(statusCode, out var response) &&
                response is OpenApiResponse openApiResponse)
                openApiResponse.Description = localizer[resourceKey].Value;
        }

        return Task.CompletedTask;
    }

    /// <summary>
    ///     Localizes the description of known resource schemas and their properties.
    /// </summary>
    /// <param name="schema">The OpenAPI schema being transformed.</param>
    /// <param name="context">The transformer context, used to resolve the localizer and type information.</param>
    /// <param name="cancellationToken">The cancellation token.</param>
    /// <returns>A completed task.</returns>
    public static Task TransformSchemaAsync(
        OpenApiSchema schema,
        OpenApiSchemaTransformerContext context,
        CancellationToken cancellationToken)
    {
        var localizer = context.ApplicationServices.GetRequiredService<IStringLocalizer<SharedResource>>();
        var type = context.JsonTypeInfo.Type;
        if (type == typeof(CreateFavoriteSourceResource))
            schema.Description = localizer["CreateFavoriteSourceSchema"].Value;
        else if (type == typeof(FavoriteSourceResource))
            schema.Description = localizer["FavoriteSourceSchema"].Value;

        var propertyResourceKey = context.JsonPropertyInfo?.Name switch
        {
            "id" => "FavoriteSourceId",
            "newsApiKey" => "FavoriteSourceNewsApiKey",
            "sourceId" => "FavoriteSourceSourceId",
            _ => null
        };

        if (propertyResourceKey is not null)
            schema.Description = localizer[propertyResourceKey].Value;

        return Task.CompletedTask;
    }
}
```
</details>

<details>
<summary>News/Interfaces/REST/FavoriteSourcesController.cs (full file, all four endpoints)</summary>

```csharp
using System.Net.Mime;
using Acme.CatchUp.Platform.News.Application.Services;
using Acme.CatchUp.Platform.News.Domain.Model.Commands;
using Acme.CatchUp.Platform.News.Domain.Model.Queries;
using Acme.CatchUp.Platform.News.Domain.Model.ValueObjects;
using Acme.CatchUp.Platform.News.Interfaces.REST.Resources;
using Acme.CatchUp.Platform.News.Interfaces.REST.Transform;
using Acme.CatchUp.Platform.Resources;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Localization;

namespace Acme.CatchUp.Platform.News.Interfaces.REST;

/// <summary>
///     Favorite sources' controller.
/// </summary>
/// <param name="favoriteSourceCommandService">The Favorite Source Command Service</param>
/// <param name="favoriteSourceQueryService">The Favorite Source Query Service</param>
/// <param name="localizer">String localizer for response messages</param>
/// <param name="logger">Logger for diagnostics and error reporting</param>
[ApiController]
[Route("api/v1/[controller]")]
[Produces(MediaTypeNames.Application.Json)]
[Tags("Favorite Sources")]
public class FavoriteSourcesController(
    IFavoriteSourceCommandService favoriteSourceCommandService,
    IFavoriteSourceQueryService favoriteSourceQueryService,
    IStringLocalizer<SharedResource> localizer,
    ILogger<FavoriteSourcesController> logger)
    : ControllerBase
{
    /// <summary>
    ///     Creates a favorite source.
    /// </summary>
    /// <param name="resource">CreateFavoriteSourceResource resource</param>
    /// <param name="acceptLanguage">Preferred response language (e.g. en, es)</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     A response as an action result containing the created favorite source, or bad request if the favorite source was
    ///     not created.
    /// </returns>
    [HttpPost]
    [ProducesResponseType(typeof(FavoriteSourceResource), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(string), StatusCodes.Status400BadRequest)]
    [ProducesResponseType(typeof(string), StatusCodes.Status409Conflict)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status500InternalServerError)]
    public async Task<ActionResult> CreateFavoriteSource([FromBody] CreateFavoriteSourceResource resource,
        [FromHeader(Name = "Accept-Language")] string? acceptLanguage,
        CancellationToken cancellationToken)
    {
        try
        {
            var createFavoriteSourceCommand =
                CreateFavoriteSourceCommandFromResourceAssembler.ToCommandFromResource(resource);
            var result = await favoriteSourceCommandService.Handle(createFavoriteSourceCommand, cancellationToken);
            return ActionResultFromCreateFavoriteSourceResultAssembler.ToActionResultFromCreateFavoriteSourceResult(
                result,
                this,
                localizer,
                nameof(GetFavoriteSourceById));
        }
        catch (ArgumentException ex)
        {
            logger.LogWarning(ex,
                "Validation failed while creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                NewsApiKey.MaskedForLogging(resource.NewsApiKey),
                resource.SourceId);
            return BadRequest(localizer["InvalidFavoriteSourceRequest"].Value);
        }
        catch (Exception ex)
        {
            logger.LogError(ex,
                "Unexpected error while creating favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                NewsApiKey.MaskedForLogging(resource.NewsApiKey),
                resource.SourceId);

            return Problem(
                title: localizer["UnexpectedServerError"].Value,
                detail: localizer["UnexpectedErrorCreatingFavoriteSource"].Value,
                statusCode: 500);
        }
    }

    /// <summary>
    ///     Gets all favorite sources by news API key.
    /// </summary>
    /// <param name="newsApiKey">The News API Key generated by the service provider</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     A response as an action result containing the favorite sources, or an empty array if none were found.
    /// </returns>
    private async Task<ActionResult> GetAllFavoriteSourcesByNewsApiKey(NewsApiKey newsApiKey,
        CancellationToken cancellationToken)
    {
        var getAllFavoriteSourcesByNewsApiKeyQuery = new GetAllFavoriteSourcesByNewsApiKeyQuery(newsApiKey);
        var result = await favoriteSourceQueryService.Handle(getAllFavoriteSourcesByNewsApiKeyQuery, cancellationToken);
        var resources = result.Select(FavoriteSourceResourceFromEntityAssembler.ToResourceFromEntity);
        return Ok(resources);
    }

    /// <summary>
    ///     Gets a favorite source by news API key and source id.
    /// </summary>
    /// <param name="newsApiKey">The News API Key generated by the news service provider</param>
    /// <param name="sourceId">The Source ID from the news service provider</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     A response as an action result containing the favorite source, or not found if the favorite source was not found.
    /// </returns>
    private async Task<ActionResult> GetFavoriteSourceByNewsApiKeyAndSourceId(NewsApiKey newsApiKey, SourceId sourceId,
        CancellationToken cancellationToken)
    {
        var getFavoriteSourceByNewsApiKeyAndSourceIdQuery =
            new GetFavoriteSourceByNewsApiKeyAndSourceIdQuery(newsApiKey, sourceId);
        var result =
            await favoriteSourceQueryService.Handle(getFavoriteSourceByNewsApiKeyAndSourceIdQuery, cancellationToken);
        if (result is null) return NotFound();
        var resource = FavoriteSourceResourceFromEntityAssembler.ToResourceFromEntity(result);
        return Ok(resource);
    }

    /// <summary>
    ///     Gets a favorite source according to parameters.
    /// </summary>
    /// <param name="newsApiKey">The News API Key generated by the news service provider</param>
    /// <param name="sourceId">The Source ID from the news service provider</param>
    /// <param name="acceptLanguage">Preferred response language (e.g. en, es)</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     A response as an action result containing the favorite source if News API Key and Source ID are provided,
    ///     or a response with the favorite sources for the News API Key if Source ID is not present.
    /// </returns>
    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<FavoriteSourceResource>), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(string), StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult> GetFavoriteSourceFromQuery([FromQuery] string newsApiKey,
        [FromQuery] string? sourceId = null,
        [FromHeader(Name = "Accept-Language")] string? acceptLanguage = null,
        CancellationToken cancellationToken = default)
    {
        try
        {
            var newsApiKeyValueObject = new NewsApiKey(newsApiKey);
            return !Request.Query.ContainsKey(nameof(sourceId))
                ? await GetAllFavoriteSourcesByNewsApiKey(newsApiKeyValueObject, cancellationToken)
                : await GetFavoriteSourceByNewsApiKeyAndSourceId(newsApiKeyValueObject,
                    new SourceId(sourceId ?? string.Empty), cancellationToken);
        }
        catch (ArgumentException ex)
        {
            logger.LogWarning(ex,
                "Validation failed while querying favorite source for NewsApiKey {NewsApiKey} and SourceId {SourceId}",
                NewsApiKey.MaskedForLogging(newsApiKey),
                sourceId);
            return BadRequest(localizer["InvalidFavoriteSourceQuery"].Value);
        }
    }

    /// <summary>
    ///     Gets a favorite source by id.
    /// </summary>
    /// <param name="id">The Favorite Source ID</param>
    /// <param name="acceptLanguage">Preferred response language (e.g. en, es)</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     A response as an action result containing the favorite source, or not found if the favorite source was not found.
    /// </returns>
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(FavoriteSourceResource), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult> GetFavoriteSourceById(int id,
        [FromHeader(Name = "Accept-Language")] string? acceptLanguage,
        CancellationToken cancellationToken = default)
    {
        var getFavoriteSourceByIdQuery = new GetFavoriteSourceByIdQuery(id);
        var result = await favoriteSourceQueryService.Handle(getFavoriteSourceByIdQuery, cancellationToken);
        if (result is null) return NotFound();
        var resource = FavoriteSourceResourceFromEntityAssembler.ToResourceFromEntity(result);
        return Ok(resource);
    }

    /// <summary>
    ///     Removes a favorite source by id.
    /// </summary>
    /// <param name="id">The Favorite Source ID</param>
    /// <param name="acceptLanguage">Preferred response language (e.g. en, es)</param>
    /// <param name="cancellationToken">Token to cancel the asynchronous operation.</param>
    /// <returns>
    ///     A response as an action result with no content on success, or an error response otherwise.
    /// </returns>
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(typeof(string), StatusCodes.Status400BadRequest)]
    [ProducesResponseType(typeof(string), StatusCodes.Status404NotFound)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status500InternalServerError)]
    public async Task<ActionResult> DeleteFavoriteSource(int id,
        [FromHeader(Name = "Accept-Language")] string? acceptLanguage,
        CancellationToken cancellationToken = default)
    {
        try
        {
            var result = await favoriteSourceCommandService.Handle(new DeleteFavoriteSourceCommand(id), cancellationToken);
            return ActionResultFromDeleteFavoriteSourceResultAssembler.ToActionResultFromDeleteFavoriteSourceResult(
                result, this, localizer);
        }
        catch (ArgumentException ex)
        {
            logger.LogWarning(ex, "Validation failed while deleting favorite source: id={Id}", id);
            return BadRequest(localizer["InvalidFavoriteSourceRequest"].Value);
        }
    }
}
```
</details>

```
git add .
git commit -m "docs: add xml doc comments to every class."
```

**Verify:** `dotnet build`. No `CS1591` warnings — every public member has the `<summary>` the compiler started checking for the moment `GenerateDocumentationFile` went on, above.

---
## Prepare the First Release

**All of this happens on `develop`:** `Feature Finish` leaves you there. One last end-to-end check before tagging `v1.0.0`; `LICENSE.md`, `README.md`, and `docs/adrs.md` already shipped back in Project Setup.

1. **Run every scenario end to end.** Start the app and, from Scalar, run through every scenario in `docs/user-stories.md`: create (`201`, blank field `400`, duplicate `409`), get by id (found/not found), list by `newsApiKey` (matches/empty array), exact lookup by `newsApiKey`+`sourceId` (found/not found/invalid), delete (`204`/`404`/`400`), and the Scalar checks from Feature 1 step 26 for the Spanish message and the localized OpenAPI document. No automated test drives these HTTP scenarios end to end, so this manual run is the acceptance check.

---
## Release

**Still on `develop`, right where *Prepare the First Release* left off.** All five features are merged. `main` shouldn't stay permanently behind `develop`: close the loop with a release.

1. **Start the release.** Git Flow Helper widget → `Release` → `Release Start` → **Version description** `v1.0.0` → `OK`. Creates and switches you to `release/v1.0.0`.

   **Note:** the release name carries a `v` prefix (`v1.0.0`), matching the git tag it becomes on finish. The `.csproj` `<Version>` stays plain (`1.0.0`), and so does the `CHANGELOG.md` heading (`## [1.0.0]`).

2. **Bump the version.** A release branch needs at least one commit of its own, or the merge into `develop` is a no-op. Still in **File System** view, open `Acme.CatchUp.Platform.csproj`, change `<Version>0.1.0-preview</Version>` to `<Version>1.0.0</Version>`.

   ```
   git add .
   git commit -m "chore(release): bump version to 1.0.0."
   ```

3. **Add `CHANGELOG.md`.** Right-click the solution root → `Add` → `File` → type `CHANGELOG.md` → Enter.

   <details>
   <summary>CHANGELOG.md</summary>

   ```markdown
   # Changelog

   All notable changes to this project will be documented in this file.

   The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
   and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

   ## [1.0.0] - 2026-10-20

   ### Added
   - TS001: Create a Favorite Source, with duplicate detection at the application and database level.
   - TS002: Retrieve a Favorite Source by Id.
   - TS003: List Favorite Sources by NewsAPI Key.
   - TS004: Check Whether a Source Is Favorite (by NewsAPI key and source id).
   - TS005: Localized OpenAPI document, following the request's `Accept-Language`.
   - TS006: Remove a Favorite Source by Id.
   - `Result<TValue, TError>` railway-style outcome type, `IBaseRepository`/`IUnitOfWork` for persistence, `IAuditableEntity` for automatic audit timestamps.
   - `NewsApiKey`/`SourceId` value objects, validated on construction; `NewsApiKey` masks itself in logs.
   - Localization via `.resx` resources (`en`, `en-US`, `es`, `es-PE`) for validation messages, error responses, and the OpenAPI document.
   - Requirement Traceability Matrix in `docs/user-stories.md`, Architecture Decision Records in `docs/adrs.md`.
   ```
   </details>

   ```
   git add .
   git commit -m "docs: add changelog for 1.0.0."
   ```

   **Note:** don't push here. This commit rides to the remote with `Release Publish` in the next step, together with the version bump.

4. **Publish and finish the release.**
   - Git Flow Helper widget → `Release` → `Release Publish` (pushes `release/v1.0.0` with all commits).
   - Git Flow Helper widget → `Release` → `Release Finish`.

   `Release Finish` merges `release/v1.0.0` into `main` (tagging it `v1.0.0`), merges it into `develop`, pushes both, and deletes the release branch.

5. **Publish the GitHub Release.**
   - On GitHub: **Releases** → **Draft a new release**.
   - Tag: pick the existing `v1.0.0` (do not create a new one).
   - Release title: `Version 1.0.0`.
   - Description: the release notes below.
   - Click **Publish release**.

   <details>
   <summary>Release notes (1.0.0)</summary>

   ```markdown
   ## 🚀 Added

   - **TS001: Create a Favorite Source**: `POST /api/v1/favorite-sources` validates and persists a `(newsApiKey, sourceId)` pair, `201 Created`, `409 Conflict` on a duplicate.
   - **TS002: Retrieve a Favorite Source by Id**: `GET /api/v1/favorite-sources/{id}`.
   - **TS003: List Favorite Sources by NewsAPI Key**: `GET /api/v1/favorite-sources?newsApiKey=...`, always a `200` array, empty when there are no matches.
   - **TS004: Check Whether a Source Is Favorite**: `GET /api/v1/favorite-sources?newsApiKey=...&sourceId=...`, a single resource or `404`.
   - **TS005: Localized API description**: the OpenAPI document itself follows `Accept-Language`, not just error messages.
   - **TS006: Remove a Favorite Source**: `DELETE /api/v1/favorite-sources/{id}`, `204 No Content` or `404`.
   - `Result<TValue, TError>` railway-style outcomes, `IBaseRepository`/`IUnitOfWork`, `IAuditableEntity` with automatic audit timestamps.
   - `NewsApiKey`/`SourceId` value objects; `NewsApiKey` self-masks in logs.
   - Localization via `.resx` resources (`en`, `en-US`, `es`, `es-PE`).
   - Project `README.md` and MIT license; `docs/user-stories.md` with a Requirement Traceability Matrix, `docs/class-diagram.puml`, `docs/adrs.md`; `CHANGELOG.md`.
   ```
   </details>

   **Tip:** the same works from the command line: `gh release create v1.0.0 --title "Version 1.0.0" --notes-file CHANGELOG.md`.

6. **Back on `develop`, pick the `-preview` suffix back up.**
   - Still in **File System** view, in `Acme.CatchUp.Platform.csproj`: `<Version>1.0.0</Version>` → `<Version>1.0.1-preview</Version>`.
   - Then:

   ```
   git add .
   git commit -m "chore(dev): set development version to 1.0.1-preview."
   git push
   ```

---
## Testing (optional, explore on your own)

Everything above already shipped as `1.0.0`. This section is optional: it adds a unit test project covering the domain model, then leaves you a starting point to go further.

1. **Add the test project.** Switch Solution Explorer to **Solution** view. Right-click the solution root → `Add` → `New Project...` → `xUnit Test Project` → name it `Acme.CatchUp.Platform.Tests`, same solution directory, then add the reference to the main project:

   ```
   dotnet add Acme.CatchUp.Platform.Tests reference Acme.CatchUp.Platform
   ```

2. **Create each test file below**, in the matching folder under `Acme.CatchUp.Platform.Tests` (create it the same way as any other folder if it doesn't exist yet): `NewsApiKeyTests.cs`, `SourceIdTests.cs`, `FavoriteSourceTests.cs`, `DeleteFavoriteSourceCommandTests.cs`. They cover, between them:
   - `NewsApiKey`'s and `SourceId`'s validation and masking.
   - `FavoriteSource`'s two constructors, including the null-command guard.
   - `DeleteFavoriteSourceCommand`'s validation for zero and negative ids.

3. **Run the suite.**

   ```
   dotnet test
   ```

4. **From here, it's on you.** Add a test for `FavoriteSourceCommandService` (mock `IFavoriteSourceRepository`/`IUnitOfWork` and assert the duplicate/not-found/success paths), or break a validation rule on purpose and confirm a test catches it.

---
## Appendix

### Continuing on another computer

- Install [Rider](https://www.jetbrains.com/rider/) and the .NET 10 SDK.
- Install MySQL locally, or confirm you can reach the same instance the lab uses.
- Sign in to `gh` again: `gh auth login` (see [Project Setup step 1](#project-setup)).
- Clone: `gh repo clone <org>/catch-up-platform`, or from Rider's Welcome screen: `Clone Repository`, paste `https://github.com/<org>/catch-up-platform.git`.
- Open the solution; Rider restores NuGet packages automatically on first open.
- Plugins live in the IDE, not the repo, so reinstall Git Flow Helper (Project Setup step 9) and plantuml4idea (Project Setup step 13) if this machine doesn't have them.
- Register your GitHub account in the IDE (Project Setup step 10): get the token with

  ```
  gh auth token
  ```

  and add it under `Settings` → `Version Control` → `GitHub`.

### Installing the GitHub CLI

Already covered once, in `hello-asp-net-developer`'s Project Setup. Here again in case it's a new machine, or `gh` got uninstalled since.

- macOS: check Homebrew itself is installed first:
  ```
  brew --version
  ```
  No output, or `command not found: brew`? Install it:
  ```
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```
  The installer prints one or two `echo` commands near the end, under "Next steps", that add Homebrew to your `PATH`, they differ by chip (Apple Silicon vs Intel) and shell. Run exactly the ones it shows you, then close the terminal and open a new one. Confirm it worked:

  ```
  brew --version
  ```

  Now install the CLI:
  ```
  brew install gh
  ```
- Windows:
  ```
  winget install --id GitHub.cli
  ```
  No `winget`? Grab the installer from [cli.github.com](https://cli.github.com/).
- Check it installed:
  ```
  gh --version
  ```
  If you get "command not found", close and reopen the terminal (the installer only updates the PATH for new sessions).

### Signing in to GitHub with a token

The guide uses `gh auth login` (Project Setup step 1), which is the simplest way. If you can't install `gh`, GitHub also accepts a Personal Access Token.

1. GitHub → `Settings` → `Developer settings` → `Personal access tokens` → `Generate new token (classic)`. Use **classic**, not "Fine-grained tokens": fine-grained tokens need an organization owner's approval before they work.
2. Fill in:
   - **Note:** `UPC` (a label to recognize it later).
   - **Expiration:** the default is fine.
   - **Scopes:** check only the top-level `repo` checkbox.
3. Click **Generate token**, then copy it somewhere safe (a password manager) before navigating away. GitHub shows it **only once**. The IDE GitHub account (Project Setup step 10) needs it too.

### Backing up unfinished work

Mid-feature and need to stop? Commit what you have, even if incomplete, and push the feature branch:

```
git add .
git commit -m "wip: <short description of where you stopped>."
git push -u origin <branch-name>
```

`wip` (work in progress) isn't one of the conventional commit types this guide otherwise uses; it's a deliberate signal that this commit is a checkpoint, not a finished unit of work.

### Feature Finish and pull requests

This guide's `Feature Finish` merges straight into `develop` with no pull request, appropriate for a solo project. A team would instead:

1. `Feature Publish` (pushes the branch, no `Feature Finish`).
2. Open a pull request on GitHub, `feature/xxx` → `develop`.
3. Review, then merge through GitHub, not through Git Flow Helper.

### Shipping a fix after a release (hotfix)

A `Hotfix` is for a defect in a release that is already tagged and published, when the correction shouldn't wait for the next feature to ship. It branches from `main`, not `develop`, and merges back into both.

**Note:** Git Flow Helper's `Hotfix` menu is greyed out while you're on `develop`. Check out `main` first: branch widget in the status bar (bottom-right) → `main` → `Checkout`.

1. **Start the hotfix.** Git Flow Helper widget → `Hotfix` → `Hotfix Start` → **Version description** `v1.0.1` → `OK`.

2. **Make the fix, then record it.**
   - Correct whatever is wrong.
   - In `Acme.CatchUp.Platform.csproj`, bump `<Version>`, for example `1.0.0` → `1.0.1`.
   - In `CHANGELOG.md`, add an entry above the previous one:

     ```markdown
     ## [1.0.1] - YYYY-MM-DD

     ### Fixed
     - <one line describing the fix>
     ```

3. **Commit.** Don't push; `Hotfix Publish` does that next.

   ```
   git add .
   git commit -m "fix: <short description>"
   ```

4. **Publish and finish the hotfix.**
   - Git Flow Helper widget → `Hotfix` → `Hotfix Publish`.
   - Git Flow Helper widget → `Hotfix` → `Hotfix Finish` (`Integrate Immediately`, `Keep remote branch when finished` unchecked).

5. **Publish the GitHub Release**, same as the main release flow, tag `v1.0.1`.

**Note:** if the `Hotfix` menu stays greyed even from `main`, re-run `Init` from the widget or restart Rider. The plain-git equivalent: branch `hotfix/v1.0.1` from `main`, commit, `git merge --no-ff` it into both `main` (then `git tag -a v1.0.1 -m "Version 1.0.1"`) and `develop`, delete the branch, then `git push origin main develop v1.0.1`.

### Removing a stray .git folder

Ran `git init` inside `Acme.CatchUp.Platform/` by mistake, instead of using the one step 1 already cloned at the solution root? Remove the stray repository:

- macOS/Linux:
  ```
  rm -rf Acme.CatchUp.Platform/.git
  ```
- Windows (PowerShell):
  ```
  Remove-Item -Recurse -Force Acme.CatchUp.Platform\.git
  ```

### Creating the repo without the GitHub CLI

No `gh`? Do the whole thing through the GitHub website plus plain `git`.

1. Authenticate git first: follow [Signing in to GitHub with a token](#signing-in-to-github-with-a-token).
2. On GitHub, inside your organization, create an empty **private** repo named `catch-up-platform`, with no README, license, or `.gitignore`.
3. On the repo's "Quick setup" page, copy the **HTTPS** clone URL.
4. `cd` into whatever folder should hold the project, then clone it:

   ```
   git clone https://github.com/<org>/catch-up-platform.git
   ```

   This creates the `catch-up-platform` folder, empty except for `.git`. Project Setup step 2 scaffolds the solution inside this same folder.
5. On GitHub, set the repo's **About** description manually:

   ```
   A REST API built with ASP.NET Core, Domain-Driven Design, and CQRS, tracking favorite news sources.
   ```

### If the class diagram doesn't render

- Confirm the **plantuml4idea** plugin is installed and enabled (`Settings` → `Plugins` → `Installed`).
- The plugin needs a local Java runtime and Graphviz to render; if it reports either missing, install a JDK and Graphviz, then restart Rider.
- As a fallback, paste the diagram's content into the [PlantUML web server](https://www.plantuml.com/plantuml/uml/) to render it in a browser.

### If MySQL refuses the connection

- Confirm MySQL is actually running: on the lab machines it already is; at home, start it (e.g. `brew services start mysql` on macOS, or the MySQL service in Windows' Services panel).
- Check `appsettings.Development.json`'s `DefaultConnection` matches the credentials your MySQL instance actually has — `root`/`123456789` is what the lab machines use, but a home install may have a different root password, or none at all.
- The first run creates the `catch-up-wa` schema itself via the initial migration (Feature 1, step 8) — you don't need to run `CREATE DATABASE` by hand, only the connection itself needs to work.
- A connection error that mentions SSL usually means your MySQL server doesn't have certificates configured; this project's connection string doesn't request SSL, so this shouldn't come up, but if it does, add `;SslMode=none` to the connection string's end.

### Free JetBrains license for students

JetBrains gives free licenses for all its IDEs (Rider included) to students with an institutional email: [jetbrains.com/community/education/#students](https://www.jetbrains.com/community/education/#students). Apply once per academic year with your UPC email.
