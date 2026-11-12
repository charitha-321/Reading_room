# Reading Room: Library Management System

A web-based library management system built with **Java Servlets, JSP (JSTL), JDBC and Oracle Database**, following the **MVC** pattern and deployed on **Apache Tomcat 9**.

Librarians manage books, loans, fines and members. Members browse the catalogue, borrow books and track due dates.

> The project source lives in the [`lib-oracle/`](lib-oracle/) folder. A detailed setup guide, troubleshooting table and change log are in [`lib-oracle/README.md`](lib-oracle/README.md).

---

## Features

**Librarian (admin)**
- Dashboard with library statistics
- Add, edit and delete books (title, author, category, ISBN, copies)
- Issue and return books, with automatic fine calculation (₹5 per day late)
- View and manage members

**Member**
- Register and log in
- Search the catalogue and borrow books (14-day loan, up to 5 books at once)
- **My books** page with due dates
- Change password
- Blocked from borrowing while any book is overdue

**UI**
- Responsive layout, split-screen login, book cards
- Colour palette picker (Ocean, Emerald, Violet, Rose, Mono) and Dark mode

## Security

- Salted **PBKDF2** password hashing (65,536 iterations); hashes are never kept in the session
- **PreparedStatement** everywhere (SQL injection protection)
- Output escaped with `<c:out>`; **Content-Security-Policy** blocks inline scripts
- **CSRF tokens** on every POST, including logout
- Login lockout: 5 failed attempts lock the email + IP for 5 minutes
- New session on login and password change (session fixation protection)
- `/admin/*` protected by a filter; JSPs sit under `WEB-INF`
- Transactions with `SELECT ... FOR UPDATE` so the last copy cannot be issued twice
- No-cache headers, no-framing and no-sniffing headers

## Tech stack

| Layer | Technology |
|---|---|
| Language | Java 11 |
| Web | Servlet 4.0, JSP 2.3, JSTL 1.2 |
| Server | Apache Tomcat 9 |
| Database | Oracle Database (10g and newer, including XE) |
| Build | Maven (WAR packaging) |
| IDE | Eclipse (Dynamic Web Project / Maven) |

## Project structure

```
library-management-oracle-v3/
├── README.md
├── .gitignore
└── lib-oracle/
    ├── pom.xml
    ├── sql/
    │   ├── 1_create_user.sql        # run once as SYSTEM
    │   └── 2_schema.sql             # tables, sequences, sample books
    └── src/main/
        ├── java/com/library/
        │   ├── model/               # Book, Issue, User
        │   ├── dao/                 # BookDAO, IssueDAO, UserDAO
        │   ├── web/                 # Servlets, SecurityFilter, LoginGuard, AppInit
        │   └── util/                # DB, Passwords
        ├── resources/db.properties  # DB connection and library rules
        └── webapp/
            ├── index.jsp
            ├── css/  js/
            └── WEB-INF/{web.xml, lib/, jsp/}
```

## Prerequisites

- JDK 11 or newer
- Apache Tomcat **9** (not 10 or 11; they use the `jakarta.*` namespace and will give 404 errors)
- Oracle Database (e.g. Express Edition) with SQL*Plus
- Eclipse IDE for Enterprise Java Developers (or any IDE with Tomcat support)
- Oracle JDBC driver (`ojdbc*.jar`) matching your database version

## Getting started

### 1. Create the database objects

In SQL*Plus (use your own folder path):

```
connect system
@path\to\lib-oracle\sql\1_create_user.sql
connect library/library123
@path\to\lib-oracle\sql\2_schema.sql
select count(*) from books;
```

The last query should print `8`.

On Oracle 12c+ (multitenant), connect to the pluggable database first, e.g. `connect system@localhost:1521/XEPDB1`.

### 2. Configure the connection

Edit `lib-oracle/src/main/resources/db.properties`. The defaults (`library` / `library123`, SID `XE`) match the SQL scripts. For Oracle 18c/21c Express, switch to the `XEPDB1` URL provided in the file.

| Property | Default | Meaning |
|---|---|---|
| `db.url` | `jdbc:oracle:thin:@localhost:1521:XE` | JDBC URL |
| `db.user` / `db.password` | `library` / `library123` | Schema owner |
| `loan.days` | `14` | Loan period |
| `fine.per.day` | `5` | Fine per late day (₹) |
| `max.loans` | `5` | Max books per member |
| `show.errors` | `true` | Show technical errors (set `false` in production) |

### 3. Add the Oracle JDBC driver

The driver jar is not distributed with this repository (it is git-ignored). Copy your own `ojdbc*.jar` (from your Oracle install, or `ojdbc8` from Maven Central) into:

```
lib-oracle/src/main/webapp/WEB-INF/lib/
```

### 4. Run

1. Import `lib-oracle` into Eclipse as an **Existing Maven Project**.
2. Add a **Tomcat v9.0** runtime under Window → Preferences → Server → Runtime Environments.
3. Right-click the project → **Run As → Run on Server**.
4. Open **http://localhost:8080/library/**

## Default login

| Role | Email | Password |
|---|---|---|
| Librarian | `admin@library.com` | `admin123` |

The admin account is created automatically on first start. **Change this password after your first login** (click your name in the top bar → Change password). Members can sign up through **Create account**.

## Main URLs

`/login` `/register` `/home` `/books` `/my-books` `/account` `/admin/dashboard` `/admin/books` `/admin/loans` `/admin/members`

## Troubleshooting (quick reference)

| Problem | Fix |
|---|---|
| `ClassNotFoundException: oracle.jdbc.OracleDriver` | `ojdbc*.jar` is missing from `WEB-INF/lib`; add it, then Project → Clean and restart the server |
| `ORA-12505` / `ORA-12514` | Wrong SID or service name in `db.url` (XE vs XEPDB1) |
| `ORA-01017` | Wrong user or password in `db.properties` |
| `ORA-01882: timezone region not found` | Add `-Doracle.jdbc.timezoneAsRegion=false` to the Tomcat VM arguments |
| `ORA-28040` / `ORA-28000` | Driver and database versions don't match; use the jar from your own Oracle install |
| `ORA-65096` | Connect to the pluggable database before creating the user |
| 404 on every page | Use Tomcat 9 |

More detail is in [`lib-oracle/README.md`](lib-oracle/README.md).

## Resetting the data

Run the `DROP` statements listed at the top of `2_schema.sql`, then run the file again.

## Notes for contributors

- `db.properties` contains default development credentials. For any shared or deployed setup, change the database password and set `show.errors=false`.
- Build output (`target/`), IDE files and the Oracle driver jar are listed in `.gitignore`.
