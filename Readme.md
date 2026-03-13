# UpNext App

A desktop Q&A application built with Java Swing. UpNext lets users sign in, browse and search questions, ask new questions with tags, read details, post answers, and vote on both questions and answers.

## Table of Contents

1. Highlights
2. Tech Stack
3. Getting Started (Windows PowerShell)
4. Configuration and Environments
5. Theme
6. Folder Structure and Source Layout
7. Database and Migrations
8. Key Screens & Flows
9. Architecture Overview
10. Components and Reuse
11. Logging and Diagnostics
12. Testing
13. Theming, Icons, and UI Customization
14. Troubleshooting
15. Contributing and Code Style

## Highlights

- Clean, modern Swing UI (FlatLaf) with a top hero bar for search and navigation
- Question feed with filters (New, Hot, Unanswered, Solved)
- Full question details page with tags, author, vote panel, answer list, and related questions
- Rich answer input with validation, character counter, and success feedback
- Voting with shared reusable component and consistent icons across the app
- Profile area and skill management (legacy feature retained)
- MySQL-backed repository layer with schema bootstrap on first run

## Tech Stack

- Language: Java (JDK 21)
- UI: Swing + FlatLaf
- Build: Maven
- Database: MySQL 8.x via MySQL Connector/J
- Testing: JUnit 5

## Getting Started

### 1) Configure the database

Copy the sample config and adjust values as needed (Windows PowerShell):

```powershell
Copy-Item config\database.properties.sample config\database.properties -Force
```

Edit `config/database.properties` if your credentials differ (host/port/db/user/password). The app will initialize the schema on first launch using the SQL files under `sql/` and `src/main/resources/db/`.

Create an empty database (if it doesn’t exist yet):

```sql
CREATE DATABASE upnex;
```

### 2) Run the app

```powershell
# build (skip tests while getting started)
mvn -q -DskipTests compile

# launch
mvn -q exec:java
```

The first run will connect to MySQL and apply schema migrations. Logs are written to the `logs/` folder.

### 3) Package a runnable JAR

```powershell
mvn -q -DskipTests package
```

The JAR will be in `target/`. You’ll need the `lib` folder on the classpath for third‑party dependencies.

## Configuration and Environments

- Configuration file: `config/database.properties` (don’t commit secrets).
- Use `config/database.properties.sample` as a starting point. Common keys:
  - `db.host`, `db.port`, `db.name`
  - `db.user`, `db.password`
  - Optional pool settings: `db.pool.min`, `db.pool.max`, `db.pool.timeout`
- The app reads this file on startup and will fail fast with a helpful error if it’s missing or malformed.

## Theme

The application uses a centralized theme in `com.upnext.app.ui.theme.AppTheme`.
See `docs/AppThemePalette.md` for palette and typography.

## Folder Structure (top level)

```
CurrentRoadmap.md
pom.xml
Readme.md
config/
docs/
lib/
logs/
scripts/
sql/
src/
```

### Source code layout

```
src/main/java/com/upnext/app/
  App.java                   # Application entry point
  config/                    # App configuration and constants
  core/                      # Logger and helpers
  data/                      # Repositories and DB bootstrap
    JdbcConnectionProvider.java
    SchemaInitializer.java
    question/                # Question/Answer/Tag repositories
    validation/
  domain/                    # Domain entities and DTOs
    question/                # Question, Answer, Vote, Subject, etc.
  service/                   # Business services (Auth, Search, etc.)
  ui/                        # UI components and screens
    components/              # Reusable widgets (VotePanel, AnswerInputPanel…)
    navigation/
    screens/                 # SignIn, Home, AddQuestion, QuestionDetail, Profile…
    theme/                   # AppTheme colors/typography

src/main/resources/ui/icons/ # Up/down vote icons and other assets
src/main/resources/db/       # Embedded schema resources
sql/                         # SQL migration scripts (executed on startup)
```

## Database and Migrations

Schema is created/updated on startup. The application runs the numbered SQL scripts in `/sql` and may load embedded resources from `src/main/resources/db/`.

- Example migration scripts:


  - `001_create_users_table.sql`
  - `002_create_skills_table.sql` (legacy)
  - `003_create_subjects_table.sql`
  - `004_create_tags_table.sql`
  - `005_create_questions_table.sql`
  - `006_create_question_tags_table.sql`
  - `007_create_answers_table.sql`
  - `008_add_question_context_and_constraints.sql`
- Voting tables and indices are initialized by repositories if needed; see startup logs.

Minimal schema (conceptual):
- users(id, name, email, password_hash, ...)
- subjects(id, name, ...)
- tags(id, name, ...)
- questions(id, user_id, subject_id, title, content, created_at, upvotes, downvotes, ...)
- question_tags(question_id, tag_id)
- answers(id, question_id, user_id, content, created_at, upvotes, downvotes, ...)
- votes tables (question/answer specific)

## Key Screens & Flows

- SignInScreen – Email/password sign‑in and navigation to Home on success.
- HomeScreen – Three‑column layout (subjects/tags • question feed • profile), with filters.
- AddQuestionScreen – Title, description, tags; input validation and persistence.
- QuestionDetailScreen – Full question view, tags, author, VotePanel, answers list, related questions.
- ProfileScreen / SkillsetScreen – User profile and legacy skill management.

## Architecture Overview

- Presentation (Swing UI):
  - Screens in `ui/screens` (Home, AddQuestion, QuestionDetail, SignIn, Profile).
  - Reusable components in `ui/components` (VotePanel, AnswerInputPanel, HeroBar, QuestionDetailsCard).
- Services (`service/*`): business logic such as search, filtering, and authentication.
- Data (`data/*`): repositories wrap JDBC access via `JdbcConnectionProvider`; `SchemaInitializer` orchestrates migrations.
- Domain (`domain/*`): POJOs representing Questions, Answers, Tags, Subjects, Users, Votes.

Typical flow (post answer):
1) User types in `AnswerInputPanel` → clicks Post.
2) Validation and auth check.
3) Repository persists answer → service updates view model.
4) UI refreshes answers list and resets editor.

Typical flow (vote):
1) Click up/down in `VotePanel`.
2) Repository records/cancels vote and recomputes counts.
3) UI updates net score; state is persisted.

## Components and Reuse

- VotePanel
  - Shared voting widget used in feed cards and detail pages.
  - Uses PNG icons from `src/main/resources/ui/icons/` to ensure consistent rendering.
- AnswerInputPanel
  - Width-constrained editor with counter, disabled/enabled states, and right-aligned Post button.
- QuestionDetailsCard
  - Sidebar metadata (subject, tags, author/date).
- HeroBar
  - Global search, breadcrumbs, and session actions.

## Logging and Diagnostics

- Logs are written to `logs/` (e.g., `logs/upnext.log`).
- On startup you’ll see DB connection info, applied migrations, and repository initialization.
- Include the last 100–200 lines of `logs/upnext.log` in bug reports.

## Testing

```powershell
mvn -q test
```

See `docs/TestPlan.md` for additional notes.

Outputs:
- Unit and integration test reports: `target/surefire-reports/`
- Compiled test classes: `target/test-classes/`

## Theming, Icons, and UI Customization

- Theme palette and typography live in `ui/theme/AppTheme` (see `docs/AppThemePalette.md`).
- Up/down vote icons are in `src/main/resources/ui/icons/` (`upvote.png`, `downvote.png`).
  - Icon sizing constants exist in `VotePanel` and `QuestionCard` for HiDPI tuning.
- Standard spacing and paddings are centralized via AppTheme constants and BoxLayout helpers.

## Documentation

- `docs/TestPlan.md` – Test notes and procedures
- `docs/AppThemePalette.md` – Theme palette and typography
- `docs/SkillsUserGuide.md` – Legacy skills feature
- `docs/*Step*_Summary.md` – Iterative design and engineering notes

## Troubleshooting

- DB connection errors: verify `config/database.properties` and that the `upnex` DB exists.
- Tables missing: check logs; the schema is created at startup from `sql/`.
- Icons look blurry on HiDPI: adjust icon size constants in `VotePanel`/`QuestionCard`.

---

If you’re setting this up on Windows PowerShell, the commands above are copy‑paste ready.

## Contributing and Code Style

- Branch naming: `feature/<short-topic>`, `fix/<short-issue>`, `docs/<short-topic>`.
- Keep PRs focused and include:
  - What changed (UI/DB/services), screenshots if UI changes.
  - Any migration impacts and rollback notes.
- Java style:
  - Prefer meaningful names, avoid long methods, keep Swing EDT rules in mind.
  - Use `final` for fields where practical, and encapsulate JDBC access in repositories.
Done.