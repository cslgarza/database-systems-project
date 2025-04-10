# 🐳 Docker + MySQL Dev Setup

This project sets up a MySQL database using Docker. On first startup, it automatically creates the database, runs schema and seed scripts from `/db`, and persists data using a Docker volume.

---

## 🚀 Getting Started

### Pre-requisites

Docker
Docker Compose

If on Windows:

Search for and install Docker Desktop.

Afterwards, activating WSL is required:

Open `Windows Powershell` and type:

```
wsl --install
```

Next, in the Windows Start Menu, look for `WSL` and open it.

Click the dropdown and select Ubuntu.

Run the following to set up Docker's `apt` repository:

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Run to install Docker packages:

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

In the WSL terminal, try running `docker ps`. You might get a permission error. If you do:

Create the docker group. You might get group docker already exists. That's fine. Just run the next command.
```
sudo groupadd docker

sudo usermod -aG docker $USER
```

If using $USER doesn't work. use the User that you created when installing WSL, or just replace with what you get
for `echo $USER`.

In Docker Desktop, open settings on the top right and go to Resources.

Click on `WSL Integration`.

Enable Ubuntu intergration.

Apply and restart.

Close and re-open Docker Desktop if it doesn't restart on its own.

### Clone the repo

```
git clone <ssh url you see when clicking the green clone button in the repo root page>
```

Cd into the repo root and continue.


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
