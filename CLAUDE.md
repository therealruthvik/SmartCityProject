# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Java Swing desktop app ("SmartCity") + MySQL backend. NetBeans/Ant project, no Maven/Gradle. Single monolithic GUI app simulating a city's services: user login/signup, employment, education, entertainment (movies/theatres), government admin, credit cards, system admin.

## Build / run / test

NetBeans-generated Ant project (`SmartCityProject/build.xml` imports `nbproject/build-impl.xml`). No standalone Maven/Gradle wrapper.

- Build: `cd SmartCityProject && ant jar` (or open in NetBeans, Build)
- Run: `cd SmartCityProject && ant run` — launches `smartcityproject.MainJFrame` (entry point / `main.class` in `nbproject/project.properties`)
- Clean: `ant clean`
- No test directory/suite exists (`test.src.dir=test` in project.properties points to a dir that isn't populated) — there is no test command to run.

**Classpath is machine-specific and will likely NOT build as-is on a new machine.** `nbproject/project.properties` hardcodes `file.reference.*` jar paths to specific developers' local filesystems (e.g. `C:\Users\ksara\Downloads\...`, `/Users/rajkumarkumaravelu/Downloads/libraries/...`). jxbrowser and `license.jar` are referenced only via these absolute paths and are NOT vendored in the repo — jxbrowser is actively used (`GovernmentAdmin/CityMapsFrame.java`, `GovernmentAdmin/VideoPanel.java`, `SystemAdmin/MapsFrame.java` embed a browser/video view via `com.teamdev.jxbrowser.*`), so these paths must be fixed/re-pointed or the project reopened+reconfigured in NetBeans before those files will compile. (JavaFX `file.reference.*` entries were removed 2026-07-06 — dead, no `javafx.*` import existed anywhere in `src/`.) Local jars that ARE vendored live in `SmartCityProject/src/Libraries/` (mysql-connector, javax.mail, activation) and one level up as `jcalendar-1.4.jar`.

## Database

- MySQL. Schema lives in root-level `.sql` files: `SQL_Script.sql` (fullest, ~15 tables: users, theatres, movies, shows, bookings, companies, jobs, applications, censor_applications, city_applications, credit_applications, credit_cards, my_purchases, course_applications, company_jobs), plus `sql latest.sql` and `sql latest 2.sql` (older/alternate dumps — check which one matches current code before assuming schema).
- Connection is hardcoded in `MainJFrame.connectDatabase()`: `jdbc:mysql://localhost:3306/test`, user/pass `root`/`root`. No config file, no env vars — a local MySQL instance named `test` with the schema loaded is required to run the app.
- Driver: `mysql-connector-java-8.0.30.jar` in `src/Libraries/`.

## Architecture

**Layering:** `Directories/*` classes (package `Directories`) are the entire data-access layer — one class per domain (`UserDirectory`, `CityDirectory`, `CompanyDirectory`, `CensorDirectory`, `EducationDirectory`, `TheatreDirectory`, `BuildingsDirectory`, `UserCoordinatesDirectory`). Each wraps a shared `java.sql.Connection` passed in via constructor and exposes plain methods that build `PreparedStatement`s inline and return raw `ResultSet`s (no ORM, no DAO interfaces, no DTOs beyond the two classes in `model/`). Callers iterate `ResultSet`s directly and pull columns by both name and 1-based index in the same codebase (e.g. `rs.getString("role")` next to `rs.getString(9)`) — column-index lookups are brittle to schema changes, check the actual `SELECT *` column order in the relevant `Directories` method before changing a table.

**Wiring/composition root:** `smartcityproject/MainJFrame` is the single entry point. It opens the one shared `Connection`, constructs every `Directories` instance once, and passes them down by constructor injection into whichever role-specific frame it opens next. There is no DI framework — it's manual, and the constructor argument lists for role frames (e.g. `SysAdminJFrame`, `GAdminLandingPage`, `CourseCreatorFrame`) are the map of "what data access this feature needs."

**Login → role dispatch:** `MainJFrame.LoginButtonActionPerformed` queries `users`, checks `can_login` (email-verified flag) and then does a long if/else chain on `role` string values (`User`, `SystemAdmin`, `gadmin`, `censoradmin`, `mcreator`, `citycomm`, `ccadmin`, `compadmin`, `jobscreator`, `tcreator`, `ucreator`, `coursecreator`, `courseregadmin`) to decide which top-level `JFrame`/`JPanel` to open next. Each role string maps 1:1 to a specific admin/creator frame — adding a new role means adding both a DB role value and a branch here.

**Feature packages** (each roughly: a landing/admin frame + creator panels + `.form` NetBeans GUI layout files paired 1:1 with `.java` controllers):
- `UI/` — end-user-facing: signup, user landing/profile/dashboard, credit card admin
- `Education/`, `Employment/`, `Entertainment/`, `GovernmentAdmin/`, `SystemAdmin/` — admin/creator panels for each vertical
- `model/` — plain data holders (`UserCoordinates`, `BuildingCoordinates`), used for map/coordinate features only; most other data flows as raw `ResultSet`s, not model objects

**GUI files:** every NetBeans form has a paired `.form` (XML layout, edited by NetBeans' GUI builder) and `.java` (generated `initComponents()` plus hand-written event handlers). Don't hand-edit the generated `initComponents()` block or the `.form` XML will drift from the `.java` — use NetBeans' builder for layout changes, or edit only the hand-written event-handler methods directly.

**Images/assets:** `src/Images/` holds icons/screenshots referenced via classpath (`getClass().getResource("/Images/...")`) — these ship inside the jar, not loaded from disk paths.
