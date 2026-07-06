# SmartCityProject

Java Swing desktop app simulating a city's digital services under one login: employment, education, entertainment (movies/theatres), government admin, credit cards, and system administration. Backed by MySQL.

## Features

- **User accounts** — signup/login, role-based landing pages
- **Employment** — companies post jobs, users browse/apply, track applications
- **Education** — universities/courses, course registration and admin approval
- **Entertainment** — movies, theatres, shows, bookings, censor-board approval workflow
- **Government admin** — city planning/maps, censor board, city commissioner, course registration admin
- **Credit cards** — credit card applications, approval, balance/purchase tracking
- **System admin** — create admins, companies, courses, jobs, movies, theatres, universities

Each role logs in through the same screen and is routed to its own dashboard based on its `role` value in the database (`User`, `SystemAdmin`, `gadmin`, `censoradmin`, `mcreator`, `citycomm`, `ccadmin`, `compadmin`, `jobscreator`, `tcreator`, `ucreator`, `coursecreator`, `courseregadmin`).

## Tech stack

- Java 11, Swing (NetBeans GUI builder), Ant/NetBeans project (no Maven/Gradle)
- MySQL (via `mysql-connector-java`)
- [FlatLaf](https://www.formdev.com/flatlaf/) for the UI theme
- [JxBrowser](https://www.teamdev.com/jxbrowser) (commercial, TeamDev) for embedded maps/video — requires your own license/jars, not vendored

## Prerequisites

- JDK 11+
- Apache Ant (`brew install ant` on macOS)
- MySQL server
- JxBrowser 9.x jars + license (only needed to compile/run the map and video-embed panels — `GovernmentAdmin/CityMapsFrame`, `GovernmentAdmin/VideoPanel`, `SystemAdmin/MapsFrame`)

## Setup

1. Start MySQL and create a database named `test`.
2. Load the schema: `mysql -u root -p test < SQL_Script.sql` (also seeds one login per role — see `SQL_Script.sql` for usernames, all with password `password`).
3. The app connects with `jdbc:mysql://localhost:3306/test`, user `root`, password `root` (hardcoded in `MainJFrame.connectDatabase()` — no config file). Adjust your local MySQL user to match, or edit that method.
4. Point `nbproject/project.properties`'s `file.reference.jxbrowser-*` entries at your local JxBrowser jars (core, platform-native, swing-bindings).

## Build & run

```
cd SmartCityProject
ant jar     # build
ant run     # build + launch
```

There is no test suite in this project.

## Diagrams

**Class diagram:**

![Classdiagram](https://user-images.githubusercontent.com/113465932/206961620-07bc43c0-4fb6-47ba-a8e7-29fb22125b9d.jpeg)

**Sequence diagram:**

![smart city (1)](https://user-images.githubusercontent.com/114713947/206962207-a8dfa3ca-6845-45e0-bcb9-133000de418a.jpg)

**Use case diagram:**

![Usecase](https://user-images.githubusercontent.com/113465932/206961645-53f43460-b979-46c0-a7cf-874e6ab7f9d2.jpeg)
