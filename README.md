# Shell Implementation
 - 

- common.sh file will containe **shared functions**
- each microservice has its own shell script (like `user.sh`, `cart.sh`, etc.) that **sources** this `common.sh` and calls relevant functions.

---

## OVERVIEW OF `common.sh`

This script defines:

* **Global variables**
* **Utility functions** (`stat_check`, `app_presetup`, etc.)
* **Technology-specific setup functions** (`nodejs`, `maven`, `python`, etc.)
* **Schema setup functions** for MongoDB and MySQL
* It’s designed to be **sourced** or **imported** from other scripts (`source common.sh`)

---

##  DETAILED EXPLANATION

### 🔹 1. **Global Variables**

```bash
color="\e[35m"
nocolor="\e[0m"
log_file="/tmp/roboshop.log"
app_path="/app"
user_id=$(id -u)
```

* `color` / `nocolor`: Adds purple color to logs for better visibility.
* `log_file`: All logs are redirected to this file.
* `app_path`: The directory where application code will be extracted.
* `user_id`: Gets the current user ID to check for root privileges.

---

###  2. **Root Check**

```bash
if [ $user_id -ne 0 ]; then
  echo Script should be running with sudo
  exit 1
fi
```

Ensures the script is executed as root or with `sudo`. If not, exits early.

---

###  3. **Function: `stat_check()`**

```bash
stat_check() {
  if [ $1 -eq 0 ]; then
    echo SUCCESS
  else
    echo FAILURE
    exit 1
  fi
}
```

* Used after every command.
* If the command's exit status is non-zero (i.e., failure), the script exits immediately.

---

### 🔹 4. **Function: `app_presetup()`**

```bash
app_presetup() {
  ...
}
```

Handles basic setup:

1. **Creates user `roboshop`** (if not exists).
2. **Creates `/app` directory**.
3. **Downloads component's zip** from AWS S3.
4. **Unzips it** into `/app`.

- Uses variable `${component}` which must be set **before** calling this function.

---

###  5. **Function: `systemd_setup()`**

```bash
systemd_setup() {
  ...
}
```

Handles setting up the systemd service:

1. Copies the respective service file for the component.
2. Replaces placeholder `roboshop_app_password` with the actual value.
3. Reloads systemd and starts the service.

**Important**:

* Assumes service file is available in `~/roboshop-shell/`.
* Uses variable `${roboshop_app_password}` — must be set externally.

---

###  6. **Function: `nodejs()`**

```bash
nodejs() {
  ...
}
```

Used for Node.js-based services:

1. Adds Node.js repo.
2. Installs Node.js.
3. Calls `app_presetup()`.
4. Installs dependencies via `npm`.
5. Calls `systemd_setup()`.

---

###  7. **Function: `mongo_schema_setup()`**

```bash
mongo_schema_setup() {
  ...
}
```

Used for services that need MongoDB schemas:

1. Copies MongoDB repo config file.
2. Installs MongoDB shell.
3. Runs the `.js` schema file using `mongo` CLI.

📝 Schema file: `${component}.js` expected in `/app/schema/`.

---

###  8. **Function: `mysql_schema_setup()`**

```bash
mysql_schema_setup() {
  ...
}
```

Used for MySQL-based services:

1. Installs MySQL client.
2. Runs `.sql` schema file using `mysql` CLI.

- Requires:

* `${mysql_root_password}` to be set.
* SQL file at `/app/schema/${component}.sql`.

---

###  9. **Function: `maven()`**

```bash
maven() {
  ...
}
```

Used for Java-based services:

1. Installs Maven.
2. Runs `app_presetup()`.
3. Builds `.jar` file.
4. Moves `.jar` to a friendly name (`<component>.jar`).
5. Calls `mysql_schema_setup()` if needed.
6. Sets up `systemd`.

---

###  10. **Function: `python()`**

```bash
python() {
  ...
}
```

Used for Python-based services:

1. Installs Python and required dev tools.
2. Calls `app_presetup()`.
3. Installs dependencies from `requirements.txt`.
4. Sets up systemd service.

---

##  HOW TO TRIGGER FROM OTHER SHELL FILES

Each microservice will have a shell file like `user.sh`:

###  Example `user.sh`:

```bash
#!/bin/bash

component=user
source common.sh
nodejs
mongo_schema_setup
```

* It sets `component=user`.
* Sources `common.sh` to load all functions.
* Calls `nodejs` to install and setup.
* Calls `mongo_schema_setup` to load schema.

---

##  Summary

| Function             | Purpose                                  |
| -------------------- | ---------------------------------------- |
| `stat_check`         | Checks command success                   |
| `app_presetup`       | Prepares app dir, user, downloads source |
| `systemd_setup`      | Sets up systemd service                  |
| `nodejs`             | Installs Node.js services                |
| `maven`              | Installs Maven-based (Java) services     |
| `python`             | Installs Python services                 |
| `mongo_schema_setup` | Loads MongoDB schema                     |
| `mysql_schema_setup` | Loads MySQL schema                       |

---

