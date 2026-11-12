# Reading Room: Library Management System

A web-based library management system built with **Java Servlets, JSP (JSTL), JDBC and Oracle Database**, following the **MVC** pattern and deployed on **Apache Tomcat 9**.

Librarians manage books, loans, fines and members. Members browse the catalogue, borrow books and track due dates.

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
- Cannot borrow while any book is overdue

**UI**
- Clean top bar, split-screen login, book cards with coloured covers, status badges
- Palette picker (Ocean, Emerald, Violet, Rose, Mono) and Dark mode

## Tech stack

| Layer | Technology |
|---|---|
| Language | Java 11 |
| Web | Servlet 4.0, JSP 2.3, JSTL 1.2 |
| Server | Apache Tomcat 9 |
| Database | Oracle Database (10g and newer, including Express Edition) |
| Build | Maven (WAR packaging) |
| IDE | Eclipse or VS Code |

## Project structure

```
Reading_room/
├── pom.xml
├── README.md
├── .gitignore
├── sql/
│   ├── 1_create_user.sql            # run once as SYSTEM
│   └── 2_schema.sql                 # tables, sequences, sample books
└── src/main/
    ├── java/com/library/
    │   ├── model/                   # Book, Issue, User
    │   ├── dao/                     # BookDAO, IssueDAO, UserDAO
    │   ├── web/                     # Servlets, SecurityFilter, LoginGuard, AppInit
    │   └── util/                    # DB, Passwords
    ├── resources/db.properties      # DB connection and library rules
    └── webapp/
        ├── index.jsp
        ├── css/  js/
        └── WEB-INF/{web.xml, lib/, jsp/}
```

## Prerequisites

- JDK 11 or newer
- Apache Tomcat **9** (not 10 or 11; they use the `jakarta.*` namespace and give 404 errors)
- Oracle Database (for example Express Edition) with SQL*Plus
- Oracle JDBC driver (`ojdbc*.jar`) matching your database version
- Eclipse IDE for Enterprise Java Developers, or any IDE with Tomcat support

## Getting started

### 1. Get the code

```
git clone https://github.com/charitha-321/Reading_room.git
```

### 2. Create the database objects

Open **Run SQL Command Line** (SQL*Plus) and enter these one at a time, using your own folder path:

```
connect system
@D:\path\to\Reading_room\sql\1_create_user.sql
connect library/library123
@D:\path\to\Reading_room\sql\2_schema.sql
select count(*) from books;
```

The last command should print `8`. If the path has spaces, wrap it in quotes: `@"D:\my folder\file.sql"`.

On Oracle 12c and newer (multitenant), connect to the pluggable database first, for example `connect system@localhost:1521/XEPDB1`.

### 3. Check the connection settings

Open `src/main/resources/db.properties`. The defaults (`library` / `library123`, SID `XE`) match the SQL scripts. If you use Oracle 18c/21c Express, switch to the `XEPDB1` URL given in the file.

| Property | Default | Meaning |
|---|---|---|
| `db.url` | `jdbc:oracle:thin:@localhost:1521:XE` | JDBC URL |
| `db.user` / `db.password` | `library` / `library123` | Schema owner |
| `loan.days` | `14` | Loan period in days |
| `fine.per.day` | `5` | Fine per late day (₹) |
| `max.loans` | `5` | Maximum books per member |
| `show.errors` | `true` | Show technical errors on the error page (set `false` for production) |

### 4. Add the Oracle JDBC driver (one time)

The driver jar is **not** included in this repository (it is git-ignored). Add your own:

1. Search your Oracle install folder (for example `C:\oraclexe`) for `ojdbc*.jar`.
2. Copy it into `src/main/webapp/WEB-INF/lib/`.
3. If you cannot find one, download `ojdbc8` from Maven Central (search "ojdbc8 19.3.0.0").

### 5. Run in Eclipse

1. Window → Preferences → Server → Runtime Environments → Add → **Apache Tomcat v9.0** → select your Tomcat folder.
2. File → Import → Maven → **Existing Maven Projects** → select the project folder → Finish.
3. Right-click the project → **Run As → Run on Server** → Tomcat v9.0.
4. Open **http://localhost:8080/library/**

## Default login

| Role | Email | Password |
|---|---|---|
| Librarian | `admin@library.com` | `admin123` |

The admin account is created automatically on first start. **Change this password after your first login** (click your name in the top bar, then Change password). Members can sign up through **Create account**.

## Using the site

**Add books (librarian only)**
1. Log in as the librarian.
2. Click **Manage books** in the top bar.
3. Fill in the **Add a book** form (title, author, category, ISBN, copies) and click **Add book**.
4. Use **Edit** or **Delete** in the table to change or remove a book.

If the top bar only shows *Catalogue* and *My books*, you are logged in as a member. Log out and log in as the librarian.

**Borrow a book (member)**
1. Log in with a member account, or create one with **Create account**.
2. Open **Catalogue**, find a book and click **Borrow**.
3. The book appears under **My books** with its due date (14 days).
4. A member can hold up to 5 books and cannot borrow while any book is overdue.

**Return a book and collect fines (librarian)**

Open **Loans**, find the loan and click **Return**. A late return shows the fine to collect. The librarian can also issue a book to a member from the **Loans** page.

## Main URLs

`/login` `/register` `/home` `/books` `/my-books` `/account` `/admin/dashboard` `/admin/books` `/admin/loans` `/admin/members`

## Security

| Area | Protection |
|---|---|
| Passwords | Salted PBKDF2 (65,536 iterations); hashes are never kept in the session |
| SQL injection | Every query uses a `PreparedStatement` |
| XSS | All output goes through `<c:out>`; Content-Security-Policy blocks inline scripts |
| CSRF | Every POST, including logout, needs a session token |
| Brute force | 5 wrong logins lock that email + IP for 5 minutes |
| Session fixation | New session on login and password change |
| Access control | A filter protects `/admin/*`; JSPs live under `WEB-INF` |
| Concurrency | Transactions with `SELECT ... FOR UPDATE` so the last copy cannot be issued twice |
| Browser headers | No-cache pages, no framing, no sniffing |
| Input checks | Length and range limits with friendly messages (title 200, author 150, ISBN 20, copies up to 9999) |
| Password rules | Minimum 8 characters with a letter and a number |

## Troubleshooting

| Problem | Fix |
|---|---|
| "The request failed" page | Oracle service not running, jar missing from `WEB-INF/lib`, or wrong `db.url`. Read the Eclipse Console or `catalina.log` for the `ORA-` or `ClassNotFound` message |
| `ClassNotFoundException: oracle.jdbc.OracleDriver` | The jar is not in `WEB-INF/lib` (Step 4). Run Project → Clean and restart the server |
| `ORA-12505` / `ORA-12514` | Wrong SID or service name in `db.url` (XE vs XEPDB1) |
| `ORA-01017` | Wrong user or password in `db.properties` |
| `ORA-01882: timezone region not found` | Server → Open launch configuration → Arguments → VM arguments, add `-Doracle.jdbc.timezoneAsRegion=false` |
| `ORA-28040` / `ORA-28000` | Driver and database versions don't match. Use the jar from your own Oracle install |
| `ORA-65096` when creating the user | Oracle 12c+ multitenant: connect to the pluggable database first |
| 404 error | Use Tomcat **9**, not 10 or 11 |

## Resetting the data

Run the `DROP` lines at the top of `sql/2_schema.sql`, then run the file again. Do not run the SQL scripts again if your tables already exist (the `select count(*) from books` check printed 8).

## Version history

**v3**
- Borrow button in the catalogue for members
- Redesigned look: white top bar, split-screen login, book cards, status badges, dashboard icons, palette picker and Dark mode

**v2**
- Security fixes: POST logout with token, login lockout, no-cache pages, security headers, stronger password rules and a Change password page, input length checks, no password hash in the session
- Members with overdue books are blocked from borrowing; maximum 5 loans per member (`max.loans`)
- Error page shows the technical cause while `show.errors=true`
- Admin account is created at the next login attempt if Oracle started after Tomcat

**From the MySQL version**
- `AUTO_INCREMENT` replaced by sequences (`users_seq`, `books_seq`, `issues_seq`), so it also works on Oracle 10g
- `ENUM` replaced by `VARCHAR2` with a `CHECK` constraint
- Search is case-insensitive using `UPPER()`
- Oracle error codes (ORA-00001 duplicate, ORA-02292 child record found) are checked instead of MySQL exceptions
- The JDBC driver is supplied as a jar in `WEB-INF/lib`

## Notes for contributors

- `db.properties` contains default development credentials. For any shared or deployed setup, change the database password and set `show.errors=false`.
- Build output (`target/`), Eclipse files and the Oracle driver jar are listed in `.gitignore`.