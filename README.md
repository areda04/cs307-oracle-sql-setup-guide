# Oracle/SQL Setup Guide on Windows/Linux (For CS307 Course)

---

## Disclaimer

- This guide is for **educational purposes only** (i.e., studying/training) — do not use it in production.

```
#include <std_disclaimer.h>
/*
 * I am not responsible for bricked software, dead SSDs, non-genuine OSes,
 * thermonuclear war, or you getting fired because the alarm app failed. Please
 * do some research if you have any concerns about applying this to your project.
 * YOU are choosing to make these modifications, and if
 * you point the finger at me for messing up your project, I will ignore it.
 */
```

---

## Requirements

- Java 17 till 21 (after that may fail, before that will make errors)
- just **you** and an **internet connection**

---

> - This guide is used for Windows/Linux only (as there is no **Oracle Database XE** for macOS)

---

### Step 1a (Ubuntu/Debian-Based OS): Install **Docker** to Use **Oracle Database XE** Inside It

Before anything, we need Docker to install **Oracle Database XE**.
To install it, open a terminal (or press `Ctrl+Alt+T`) and run:

```
sudo apt update
```

Then, install certificates & required software for configuring **Docker** below as follows:

```
sudo apt install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Then run:

```
sudo apt update
```

Then, install Docker & its components as follows:

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Finally, start it by running:

```
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```

---

### Step 1b (Ubuntu/Debian-Based OS): Restart Your Device

---

### Step 2 (Ubuntu/Debian-Based OS): Install **Oracle Database XE** Container Inside **Docker**

Open a new terminal (or press `Ctrl+Alt+T`) and run:

```
docker pull gvenzl/oracle-xe
```

This will download our Oracle Database XE (latest version).

---

### Step 3 (Ubuntu/Debian-Based OS): Define **Oracle Database XE** Container Credentials Inside **Docker**

Run in the same terminal the following:

```
docker run -d \
  --name oracle-xe \
  -p 1521:1521 \
  -p 5500:5500 \
  -e ORACLE_PASSWORD=admin \
  gvenzl/oracle-xe
```

These are our container credentials, that means:

- **Container Name** : `oracle-xe`
- **Port 1** : `1521`
- **Port 2** : `5500`
- **Oracle Username** : `system` (can't be changed & won't be used)
- **Oracle Password** : `admin` (won't be used)

> **Note:** You can change your container credential values (except **Oracle Username**), but this is on your own responsibility and I don't recommend that.

---

### Step 4 (Ubuntu/Debian-Based OS): Check That Your **Oracle Database XE** Container Is Configured Inside **Docker**

Run in the same terminal the following:

```
docker ps
```

- If it appears like this:

```
CONTAINER ID   IMAGE              COMMAND                  CREATED        STATUS       PORTS                                                                                      NAMES
a32ebda9c557   gvenzl/oracle-xe   "container-entrypoin…"   2 months ago   Up 3 hours   0.0.0.0:1521->1521/tcp, [::]:1521->1521/tcp, 0.0.0.0:5500->5500/tcp, [::]:5500->5500/tcp   oracle-xe
```

Go to Step 5.

- If it appears like this (i.e., no containers in Docker):

```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

You need to open your container by running: `docker start oracle-xe` and repeat Step 4.

> **Note:** If you want to close/stop your container at any time, run: `docker stop oracle-xe` in a terminal.

---

### Step 5 (Ubuntu/Debian-Based OS): Enter Your **Oracle Database XE** Container

Run in the same terminal the following:

```
docker exec -it oracle-xe bash
```

Then:

```
sqlplus / as sysdba
```

It must take you inside **Oracle SQL** (i.e., to `SQL>`) like this:

```
SQL*Plus: Release 21.0.0.0.0 - Production on Sat May 9 16:54:10 2026
Version 21.3.0.0.0
Copyright (c) 1982, 2021, Oracle.  All rights reserved.
Connected to:
Oracle Database 21c Express Edition Release 21.0.0.0.0 - Production
Version 21.3.0.0.0
SQL> 
```

---

### Step 6 (Ubuntu/Debian-Based OS): Create Your User for Connecting It to Our Database

> **Note:** You can name your username & password as you like.

Generally, in the same terminal that shows `SQL>`, run the following **Line by Line** (i.e., copy and paste the line, then press `Enter` and go to the next line until it is finished):

And replace each `(Username)` with your username and `(Password)` with your password.

1. `ALTER SESSION SET CONTAINER = XEPDB1;` (this means your `service name` will be `XEPDB1`).
2. `DROP USER (Username) CASCADE;` (if you previously created a user with your new username, skip it if it's your first time to create users or something else).
3. `CREATE USER (Username) IDENTIFIED BY (Password);`
4. `ALTER USER (Username) DEFAULT TABLESPACE users QUOTA UNLIMITED ON users;`
5. `ALTER USER (Username) TEMPORARY TABLESPACE TEMP;`
6. `GRANT CONNECT TO (Username);`
7. `GRANT CREATE SESSION, CREATE VIEW, CREATE TABLE, ALTER SESSION, CREATE SEQUENCE TO (Username);`
8. `GRANT CREATE SYNONYM, CREATE DATABASE LINK, RESOURCE, UNLIMITED TABLESPACE TO (Username);`
9. `GRANT CREATE TRIGGER TO (Username);`

It must output in each line something like `USER CREATED`, `USER ALTERED`, `SESSION ALTERED`, or something that says it works correctly.

> **Note:** If you face errors in any line, just repeat Step 5 and complete Step 6 from where you stopped and what you faced an error in.

We will make a user with **Username =** `hr` and **Password =** `hr` (for the first time).

i.e., run these commands inside `SQL>`:

1. `ALTER SESSION SET CONTAINER = XEPDB1;`
2. `DROP USER hr CASCADE;` (if you previously created a user with `hr` username, skip it if it's your first time to create users or something else).
3. `CREATE USER hr IDENTIFIED BY hr;`
4. `ALTER USER hr DEFAULT TABLESPACE users QUOTA UNLIMITED ON users;`
5. `ALTER USER hr TEMPORARY TABLESPACE TEMP;`
6. `GRANT CONNECT TO hr;`
7. `GRANT CREATE SESSION, CREATE VIEW, CREATE TABLE, ALTER SESSION, CREATE SEQUENCE TO hr;`
8. `GRANT CREATE SYNONYM, CREATE DATABASE LINK, RESOURCE, UNLIMITED TABLESPACE TO hr;`
9. `GRANT CREATE TRIGGER TO hr;`

---

### Step 7 (Ubuntu/Debian-Based OS): Save Your User Credentials for Using It in **SQL Developer**

We need now `username`, `password`, `host`, `port`, and `service name`.

From **Step 4**:

After running `docker ps` in our terminal (outside `SQL>`), it appears:

```
CONTAINER ID   IMAGE              COMMAND                  CREATED        STATUS       PORTS                                                                                      NAMES
a32ebda9c557   gvenzl/oracle-xe   "container-entrypoin…"   2 months ago   Up 3 hours   0.0.0.0:1521->1521/tcp, [::]:1521->1521/tcp, 0.0.0.0:5500->5500/tcp, [::]:5500->5500/tcp   oracle-xe
```

We need only the `PORTS` section:

```
PORTS
0.0.0.0:1521->1521/tcp, [::]:1521->1521/tcp, 0.0.0.0:5500->5500/tcp, [::]:5500->5500/tcp
```

From the first line `0.0.0.0:1521->1521/tcp`, it represents `host:port->port/protocol`.

So, `host`: `0.0.0.0` (equivalent to `localhost`) and `port`: `1521`.

From **Step 6**:

After creating user (`hr`) with password (`hr`) in service name `XEPDB1`:

So, `username`: `hr`, `password`: `hr`, and `service name`: `XEPDB1`.

In summary:

| Placeholder    |     Value     |
|----------------|---------------|
| `host`         | `localhost`   |
| `port`         | `1521`        |
| `username`     | `hr`          |
| `password`     | `hr`          |
| `service name` | `XEPDB1`      |

---

### Step 8 (Ubuntu/Debian-Based OS): Install **SQL Developer & JDK 17**

- Download **SQL Developer** from this link: [SQL Developer Download](https://download.oracle.com/otn_software/java/sqldeveloper/sqldeveloper-24.3.1.347.1826-no-jre.zip)
- Extract our `.zip` file.
- Check that your Java version is 17 till 21 by running `java --version` in a terminal.
- If it is less than 17, download JDK 17 by running `sudo apt install openjdk-17-jdk -y` in a terminal.
- Give the Java path (as we need it in SQL Developer) by running `update-alternatives --config java` in a terminal.

It appears like this:

```
There is 1 choice for the alternative java (providing /usr/bin/java).
  Selection    Path                                        Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/temurin-21-jdk-amd64/bin/java   2111      auto mode
  1            /usr/lib/jvm/temurin-21-jdk-amd64/bin/java   2111      manual mode
Press <enter> to keep the current choice[*], or type selection number: 
```

- Copy only `/usr/lib/jvm/temurin-21-jdk-amd64/` or similar and press `Ctrl+C`.

Run `dirname "$(find ~ -name 'sqldeveloper.sh' 2>/dev/null)"` and copy the output.

It outputs like:

```
/home/$USER/Downloads/sqldeveloper-24.3.1.347.1826-no-jre/sqldeveloper
```

Follow these steps:

1. Run `sudo mv (SQL Developer Path) /opt/`
> replace `(SQL Developer Path)` with result of `dirname "$(find ~ -name 'sqldeveloper.sh' 2>/dev/null)"`
i.e. make it `sudo mv /home/$USER/Downloads/sqldeveloper-24.3.1.347.1826-no-jre/sqldeveloper /opt/`

2. Then run `sudo chmod -R 755 /opt/sqldeveloper`
3. Then run `sudo rm /usr/local/bin/sqldeveloper`
4. Then run `sudo ln -s /opt/sqldeveloper/sqldeveloper.sh /usr/local/bin/sqldeveloper`
5. Then run `sudo nano /usr/share/applications/sqldeveloper.desktop`
    - It will open a text file; paste this script in it:

```
[Desktop Entry]
Version=1.0
Type=Application
Name=Oracle SQL Developer
Comment=Oracle Database IDE
Exec=/opt/sqldeveloper/sqldeveloper.sh
Icon=/opt/sqldeveloper/icon.png
Terminal=false
Categories=Development;Database;
StartupNotify=true
StartupWMClass=
```

5. Then run `sudo chmod +x /usr/share/applications/sqldeveloper.desktop`

It will make an icon `Oracle SQL Developer` in your apps menu.

---

### Step 9 (Ubuntu/Debian-Based OS): Open **SQL Developer** & Configure It

When you open it, it will need your JDK Path.

From **Step 8** (we got the Java path `/usr/lib/jvm/temurin-21-jdk-amd64/`):

Paste it and go to configure the user connection.

Then, follow the photos:

- Click the **+** button in the top-left corner.

![Step_1_Photo_3.png](readme-photos/Step_1_Photo_3.png)

- From Step 7 (we have our data to enter in this window):

![Step_1_Photo_4.png](readme-photos/Step_1_Photo_4.png)

- Write in Name anything.
- Database Type: `Oracle`
- Authentication Type: `Default`
- Username: `hr`
- Password: `hr`
- Role: `default`
- Connection Type: `Basic`
- Hostname: `localhost`
- Port: `1521`
- SID (make it empty & choose Service Name)
- Service Name: `XEPDB1`

Press "Connect".

---

### Step 10 (Ubuntu/Debian-Based OS): Add Your "HR Schema"

Add the schema [hr_schema.txt](DBSchema/hr_schema.txt) in SQL Developer:

Paste the content of our schema into SQL Developer, then press `F5`.

- If the Script Output appears like this, your schema has been added, and you can go to Step 11:

![Step_1_Photo_9.png](readme-photos/Step_1_Photo_9.png)

---

### Step 11 (Ubuntu/Debian-Based OS): Let's Test Our Database

Write `SELECT * FROM employees` like this:

![Step_1_Photo_6.png](readme-photos/Step_1_Photo_6.png)

Press `Ctrl + Enter`, If the Query Result appears like this, your schema is in your database and your Oracle Setup is done:

![Step_1_Photo_7.png](readme-photos/Step_1_Photo_7.png)

---

*Made by Ahmed R. Ibrahim **([@areda04](https://github.com/areda04))** — Good luck!*