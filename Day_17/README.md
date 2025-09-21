### Issue;

The Nautilus application development team has shared that they are planning to deploy one newly developed application on Nautilus infra in Stratos DC. The application uses PostgreSQL database, so as a pre-requisite we need to set up PostgreSQL database server as per requirements shared below:

### Tasks;

PostgreSQL database server is already installed on the Nautilus database server.

a. Create a database user kodekloud_pop and set its password to Rc5C9EyvbU.

b. Create a database kodekloud_db2 and grant full permissions to user kodekloud_pop on this database.

Note: Please do not try to restart PostgreSQL server service.

---

#### Step 1: SSH into the Database Server

```
ssh peter@stdb01
# password: Sp!dy
```

Explanation:

- Connects to the PostgreSQL database server stdb01 using SSH.

- All database setup will be done here.

#### Step 2: Switch to the postgres System User

```
sudo su - postgres
```

Explanation:

- The postgres user is the default administrative account for PostgreSQL.

- We need this account to create roles and databases.

#### Step 3: Access the PostgreSQL CLI

```
psql
```

Explanation:

- Launches the PostgreSQL interactive terminal where we can run SQL commands.

#### Step 4: Create the Database User

```
CREATE USER kodekloud_pop WITH PASSWORD 'Rc5C9EyvbU';
```

Explanation:

- Creates a new database role (user) named kodekloud_pop.

- Assigns the password Rc5C9EyvbU for authentication.

#### Step 5: Create the Database

```
CREATE DATABASE kodekloud_db2;
```

Explanation:

- Creates a new database named kodekloud_db2.

- This will be used by the Nautilus application.

#### Step 6: Grant Permissions to the User

```
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db2 TO kodekloud_pop;
```

Explanation:

- Grants full privileges (CONNECT, CREATE, SELECT, INSERT, UPDATE, DELETE, etc.) on kodekloud_db2 to the kodekloud_pop user.

- Ensures the application user can fully interact with the database.

#### Step 7: Verify the Setup

List users:

```
\du
```

List databases:

```
\l
```

Expected Output:

- kodekloud_pop should appear in the user list.

- kodekloud_db2 should appear in the database list with privileges granted.

### Result:

PostgreSQL is now configured with:

User: `kodekloud_pop (password: Rc5C9EyvbU)`

Database: `kodekloud_db2` with full access granted to the user.

This completes the pre-requisite setup for the Nautilus application.
