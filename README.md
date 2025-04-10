# 🐳 Docker + MySQL Dev Setup

This project sets up a MySQL database using Docker. On first startup, it automatically creates the database, runs schema and seed scripts from `/db`, and persists data using a Docker volume.

---

## 🚀 Getting Started

### Start the Database

```bash
docker-compose -f docker-compose/docker-compose.dev.yml up --build
```

This will:
- Create a database called `my_database`
- Execute any `.sql` files in `/db` (e.g. `001_initial.sql`)
- Persist data in the `mysql_data` Docker volume

---

### Reset the Database (Clean Start)

```bash
docker-compose -f docker-compose/docker-compose.dev.yml down -v
docker-compose -f docker-compose/docker-compose.dev.yml up --build
```

This removes all data and re-applies your initialization SQL.

---

## 🧱 Configuration

**docker-compose/docker-compose.dev.yml:**

```yaml
services:
  mysql-db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: temp1234!
      MYSQL_DATABASE: my_database
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ../db:/docker-entrypoint-initdb.d:ro

volumes:
  mysql_data:
```

---

## 🧪 SQL Dump + Restore

**Create a database dump:**

```bash
docker exec -i mysql-db mysqldump -u root -ptemp1234! my_database > my_database_dump.sql
```

**Restore from a dump:**

```bash
docker exec -i mysql-db mysql -u root -ptemp1234! my_database < my_database_dump.sql
```

---

## 📝 Example Init File: `001_initial.sql`

```sql
CREATE DATABASE IF NOT EXISTS my_database;
USE my_database;

DROP TABLE IF EXISTS posts;
DROP TABLE IF EXISTS users;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

INSERT INTO users (name, email) VALUES
('Alice', 'alice@example.com'),
('Bob', 'bob@example.com');

INSERT INTO posts (user_id, title, content) VALUES
(1, 'Hello World', 'Alice''s first post.'),
(2, 'Docker and MySQL', 'Bob''s thoughts.');
```

---

## ✅ Notes

- SQL files in `/db` only run once on a fresh volume.
- Files must end in `.sql`, `.sql.gz`, or `.sh` to be executed.
- Use numbered filenames like `001__init.sql`, `002__more_seed_data.sql` for ordering.
