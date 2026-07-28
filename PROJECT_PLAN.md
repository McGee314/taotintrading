# Project Plan — Taotin Trading Co., Ltd

**Company Profile & Content Management System (CMS)**

---

## 📋 Project Overview

| Field              | Details                                                            |
|--------------------|--------------------------------------------------------------------|
| **Company**        | Taotin Trading Co., Ltd                                            |
| **Location**       | Kiribati                                                           |
| **Divisions**      | Supermarket, Electronics, Hardware, Shipping, Hotels               |
| **Tech Stack**     | Laravel, Filament PHP, PostgreSQL, Blade + Tailwind CSS, Docker    |
| **Environment**    | Docker (Laravel Sail) with Apple Silicon/M1 compatibility         |
| **Methodology**    | Agile / Sprint-based                                               |
| **Version Control**| Git (Conventional Commits)                                         |

---

## 🔁 Sprint Breakdown

---

### Sprint 1: Project Initialization & Environment Setup

**Goal:** Bootstrap the Laravel project, configure the Docker environment with PostgreSQL, and install Filament PHP.

#### Tasks

1. **Initialize Laravel Project**
   - Use `curl -s https://laravel.build/taotin-cms | bash` (or equivalent) to scaffold a new Laravel application via Laravel Sail.
   - Ensure the project directory is named `taotin-cms` and sits under the current working directory.

2. **Configure Laravel Sail with PostgreSQL**
   - Modify `docker-compose.yml` to include a PostgreSQL service instead of (or alongside) MySQL.
   - Update `.env` to point `DB_CONNECTION` to `pgsql` with appropriate host, port, database, username, and password.

3. **Verify Docker Environment**
   - Run `./vendor/bin/sail up -d` to start all containers.
   - Run `./vendor/bin/sail ps` to confirm all services (app, pgsql, redis, etc.) are healthy.
   - Ensure the environment is compatible with Apple Silicon/M1 by using ARM-compatible Docker images.

4. **Install & Publish Filament PHP**
   - Require Filament via Composer: `composer require filament/filament`.
   - Publish Filament assets and configuration: `php artisan filament:install --panels`.
   - Create an admin user: `php artisan make:filament-user`.

5. **Initial Commit**
   - `git init` and `git add .`
   - `git commit -m "chore: initialize Laravel project with Sail, PostgreSQL, and Filament"`

#### Acceptance Criteria
- [x] Laravel app runs at `http://localhost` without errors.
- [x] PostgreSQL container is up and Laravel can connect to it.
- [x] Filament admin panel is accessible at `/admin`.
- [x] Admin user can log in to the Filament panel.

---

### Sprint 2: Database Architecture & Models

**Goal:** Define the core database schema, create Eloquent models with relationships, seed dummy data, and verify the database layer.

#### Tasks

1. **Create Migrations for Core Entities**
   - `divisions`
     - `id` (uuid/bigIncrements)
     - `name` (string)
     - `slug` (string, unique)
     - `description` (text, nullable)
     - `icon` (string, nullable) — stores icon class or path
     - `active_status` (boolean, default `true`)
     - `sort_order` (integer, default `0`)
     - `timestamps()`
   - `products`
     - `id` (uuid/bigIncrements)
     - `division_id` (foreign key → divisions)
     - `title` (string)
     - `description` (text, nullable)
     - `image_path` (string, nullable)
     - `is_highlighted` (boolean, default `false`)
     - `timestamps()`
   - `vacancies`
     - `id` (uuid/bigIncrements)
     - `title` (string)
     - `description` (text)
     - `requirements` (text, nullable)
     - `status` (enum: `open`, `closed`, default `open`)
     - `deadline` (date, nullable)
     - `timestamps()`
   - `company_settings`
     - `id` (uuid/bigIncrements)
     - `key` (string, unique)
     - `value` (text, nullable)
     - `timestamps()`
     > Note: Alternatively, use a single-row schema with explicit columns (`hq_address`, `phone`, `email`, `operating_hours`, `hero_image_path`) for simplicity.

2. **Create Eloquent Models**
   - `Division` — `hasMany` Products
   - `Product` — `belongsTo` Division
   - `Vacancy`
   - `CompanySetting` (or a dedicated key-value model)

3. **Create Factories**
   - `DivisionFactory` — generates realistic division names and descriptions.
   - `ProductFactory` — generates product titles and randomly assigns divisions.
   - `VacancyFactory` — generates job listings with varied statuses and deadlines.

4. **Create Seeders**
   - `DivisionSeeder` — seeds the five Taotin divisions: Supermarket, Electronics, Hardware, Shipping, Hotels.
   - `ProductSeeder` — seeds ~20 sample products distributed across divisions.
   - `VacancySeeder` — seeds ~5 sample job vacancies.
   - `CompanySettingSeeder` — seeds default company info (address, phone, email, hours).

5. **Run Migrations & Seed**
   - `php artisan migrate --seed`
   - Verify data exists in all tables via Tinker or database client.

6. **Commit**
   - `git add .`
   - `git commit -m "feat: add database schema, models, factories, and seeders for core entities"`

#### Acceptance Criteria
- [x] All five migrations run without errors.
- [x] Foreign key constraints are in place (products → divisions).
- [x] Database is seeded with realistic dummy data.
- [x] Models can be queried via Laravel Tinker with correct relationships.

---

### Sprint 3: Admin CMS Development (Filament)

**Goal:** Build out the admin panel using Filament Resources for all core entities so non-technical staff can manage content.

#### Tasks

1. **Generate Filament Resources**
   - `DivisionResource`
   - `ProductResource`
   - `VacancyResource`
   - `CompanySettingResource`
   > Use `php artisan make:filament-resource <EntityName>` for each.

2. **Configure Forms**
   - **DivisionResource form:**
     - `name` → `TextInput::make()->required()`
     - `slug` → `TextInput::make()` (auto-generated from name)
     - `description` → `RichEditor::make()`
     - `icon` → `TextInput::make()` (or `FileUpload::make()` if storing icons)
     - `active_status` → `Toggle::make()`
     - `sort_order` → `TextInput::make()->numeric()`
   - **ProductResource form:**
     - `title` → `TextInput::make()->required()`
     - `division_id` → `Select::make()->relationship('division', 'name')`
     - `description` → `RichEditor::make()`
     - `image_path` → `FileUpload::make()->image()->directory('products')`
     - `is_highlighted` → `Toggle::make()`
   - **VacancyResource form:**
     - `title` → `TextInput::make()->required()`
     - `description` → `RichEditor::make()`
     - `requirements` → `RichEditor::make()`
     - `status` → `Select::make()->options(['open' => 'Open', 'closed' => 'Closed'])`
     - `deadline` → `DatePicker::make()`
   - **CompanySettingResource form:**
     - `hq_address` → `Textarea::make()`
     - `phone` → `TextInput::make()->tel()`
     - `email` → `TextInput::make()->email()`
     - `operating_hours` → `TextInput::make()`
     - `hero_image_path` → `FileUpload::make()->image()`

3. **Configure Tables**
   - Add searchable columns (`TextColumn::make()->searchable()`).
   - Add sortable columns.
   - Add filters (e.g., `Filter::make('active')` for divisions, status for vacancies).
   - Add bulk actions where appropriate (delete, toggle active).

4. **Configure Navigation & Branding**
   - Set the Filament brand name to "Taotin CMS".
   - Group resources under logical navigation groups (e.g., "Content", "Settings").

5. **Test Admin Panel**
   - Navigate to `/admin` and verify all resources appear.
   - Create, Read, Update, and Delete records for each entity.
   - Test file uploads (images) and rich text editing.

6. **Commit**
   - `git add .`
   - `git commit -m "feat: add Filament admin resources for divisions, products, vacancies, and company settings"`

#### Acceptance Criteria
- [x] All four Filament Resources are accessible from the admin sidebar.
- [x] Forms include rich text editors and file uploads where specified.
- [x] Tables support search, sort, and filter functionality.
- [x] CRUD operations work end-to-end for each resource.

---

### Sprint 4: Public Frontend Slicing & Integration

**Goal:** Build the public-facing company profile website with dynamic data fetched from the CMS, styled with Tailwind CSS.

#### Tasks

1. **Setup Tailwind CSS**
   - Ensure Tailwind CSS is properly configured in the Laravel project (Laravel Breeze or manual Vite setup).
   - Configure `tailwind.config.js` with custom brand colors/fonts if needed.
   - Verify hot-reloading works with `npm run dev` (Vite).

2. **Create Public Layout**
   - Create a Blade layout component: `resources/views/layouts/public.blade.php`.
   - Include partials:
     - `partials/navbar.blade.php` — responsive navigation with links to Home, Divisions, Careers, Contact.
     - `partials/footer.blade.php` — company info, quick links, copyright.
   - Use Tailwind utility classes for styling.

3. **Build Components**
   - **Hero Section** (`partials/hero.blade.php`):
     - Displays `hero_image_path` from `CompanySettings` as background.
     - Shows company name and tagline.
     - Call-to-action button(s).
   - **Divisions Section** (`partials/divisions.blade.php`):
     - Fetches active divisions from the database.
     - Displays division cards with icon, name, and description.
   - **Highlighted Products Section** (`partials/highlighted-products.blade.php`):
     - Fetches products where `is_highlighted = true`.
     - Displays product cards with image, title, and division name.
   - **Contact Section** (`partials/contact.blade.php`):
     - Displays HQ address, phone, email, and operating hours from `CompanySettings`.

4. **Build Pages**
   - **Home Page** (`routes/web.php` → `HomeController@index`):
     - Assembles Hero, Divisions, Highlighted Products, and Contact sections.
     - Passes data from controller to view.
   - **Careers/Jobs Page** (`routes/web.php` → `CareerController@index`):
     - Fetches open vacancies (`status = 'open'`), ordered by newest first.
     - Displays job cards with title, description excerpt, requirements, and deadline.
     - Shows "No open positions" message when no vacancies are available.

5. **Responsive & Accessibility Review**
   - Ensure all pages are fully responsive (mobile, tablet, desktop).
   - Check color contrast and keyboard navigation.

6. **Commit**
   - `git add .`
   - `git commit -m "feat: build public frontend with dynamic data integration and Tailwind CSS"`

#### Acceptance Criteria
- [x] Homepage loads without errors and displays dynamic division/product data.
- [x] Careers page lists all open vacancies with relevant details.
- [x] Site is responsive on mobile, tablet, and desktop viewports.
- [x] All text content is pulled from the database (CMS-managed), not hardcoded.

---

## 📐 Architecture Diagram (Conceptual)

```
┌──────────────────────────────────────────────────────┐
│                     Docker (Sail)                     │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐  │
│  │  App     │  │ PostgreSQL│  │  Redis (optional) │  │
│  │ (PHP 8.x)│  │  (DB)    │  │  (Cache/Session)  │  │
│  └────┬─────┘  └────┬─────┘  └───────────────────┘  │
│       │             │                                 │
│  ┌────┴─────────────┴─────┐                          │
│  │      Laravel App        │                          │
│  │  ┌───────────────────┐  │                          │
│  │  │  Admin (/admin)    │  │  ← Filament Panel       │
│  │  │  Public (/)        │  │  ← Blade + Tailwind     │
│  │  └───────────────────┘  │                          │
│  └─────────────────────────┘                          │
└──────────────────────────────────────────────────────┘
```

## 🌿 Branching Strategy

| Branch          | Purpose                                        |
|-----------------|------------------------------------------------|
| `main`          | Production-ready code                          |
| `develop`       | Integration branch for features                |
| `feature/*`     | Individual feature branches (per sprint/task)  |

---

## 📝 Conventional Commits Cheat Sheet

| Type       | Usage                                  |
|------------|----------------------------------------|
| `feat:`    | New feature (e.g., new page, resource) |
| `fix:`     | Bug fix                                |
| `chore:`   | Tooling, config, dependencies          |
| `docs:`    | Documentation                          |
| `refactor:`| Code restructuring (no feature change) |
| `style:`   | Formatting, CSS changes                |
| `test:`    | Adding or updating tests               |

---

## ✅ Sprint Sign-off

| Sprint | Description                              | Status      | Date Completed |
|--------|------------------------------------------|-------------|----------------|
| 1      | Project Initialization & Environment     | ⬜ Pending  | —              |
| 2      | Database Architecture & Models           | ⬜ Pending  | —              |
| 3      | Admin CMS Development (Filament)         | ⬜ Pending  | —              |
| 4      | Public Frontend Slicing & Integration    | ⬜ Pending  | —              |

---

*This document serves as the living project plan. Each sprint will be executed sequentially with approval gates between phases.*