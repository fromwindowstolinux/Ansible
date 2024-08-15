 # Ansible Configuration

This section talks about the Ansible playbook designed for the deployment and configuration of the Airflow environment, along with related services such as Redis and PostgreSQL. Each block in the playbook has a specific purpose, and this section explains why each block is structured the way it is.

## Play 1: SELinux in Permissive Mode

![SELinux in Permissive Mode](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-33-45.png)

- Task 1: Put SELinux in permissive mode

This play block is designed to configure SELinux (Security-Enhanced Linux), a security feature in Linux, to run in permissive mode on specified hosts. The play targets the hosts named `airflow_master` and `airflow_worker`, which are parts of the Apache Airflow deployment. The `become: yes` directive allows the tasks to be executed as the root user. The play uses a `vars.yml` file to import any necessary variables. The specific task within this block sets SELinux to "permissive" mode, which means that while SELinux policies are still enforced and violations are logged, they do not prevent operations from being carried out. This is often done to troubleshoot SELinux-related issues or to allow software that is not fully compatible with SELinux to run without being blocked.

## Play 2: Redis Installation, Start, and Enable

![Install, start, and enable Redis](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-34-00.png)

- Task 1: Install Redis
- Task 2: Start and enable Redis service

This play block is responsible for installing, starting, and enabling the Redis service on hosts identified as airflow_redis. The first task installs the Redis package using the dnf package manager. The package installation is ensured by setting the `state: present`, meaning the Redis package will be installed if it is not already present. The second task starts the Redis service and ensures it is enabled to start automatically on boot. The service module is used to manage the Redis service, with `state: started` ensuring that Redis is running, and `enabled: yes` ensuring that the Redis service will start automatically on system boot. This setup made Redis available as a backend service for managing task queues or as a cache.

## Play 3: PostgreSQL and Psycopg2 Installation and Configuration

![Install Postgresql and Psycopg2 packages](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-36-50.png)


- Task 1: Install Postgresql and Psycopg2 packages
- Task 2: Initialize PostgreSQL

This play block prepares a PostgreSQL environment by installing the necessary software and initializing the database, making it ready for use by Apache Airflow for managing metadata and other data storage needs. The play block is designed to set up a PostgreSQL environment on a hosts identified as `airflow_postgresql`. The first task installs the PostgreSQL client and server packages, along with psycopg2, a PostgreSQL adapter for Python, and the Python 3.11 development package. The installation is managed by the dnf module. The second task initializes the PostgreSQL database using the `postgresql-setup initdb` command. This task will only run if the PostgreSQL initialization has not already been performed, as indicated by the absence of the `pg_hba.conf` file. The creates argument ensures that the task is only executed if the specified file does not exist, preventing reinitialization. 

![Install Postgresql and Psycopg2 packages](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-37-24.png)

- Task 3: Ensure pg_hba.conf has md5 authentication (ipv4)
- Task 4: Ensure pg_hba.conf has md5 authentication (ipv6)
- Task 5: Start and enable PostgreSQL service

This play block is designed to configure PostgreSQL to use md5 password-based authentication for both IPv4 and IPv6 connections and to ensure that the PostgreSQL service is running and enabled to start at boot. The third task modifies the pg_hba.conf file, which controls client authentication in PostgreSQL, to replace any existing ident authentication for IPv4 localhost connections (127.0.0.1/32) with md5 authentication. The fourth task performs a similar operation for IPv6 localhost connections (::1/128). Both tasks use the lineinfile module to ensure that these changes are present in the configuration file, enhancing security by requiring a username and password for connections. The fifth task ensures that the PostgreSQL service is restarted to apply these changes and is set to start automatically on boot, ensuring the database server is always available when needed.

Configuring md5 authentication ensures that Airflow can connect to the PostgreSQL database using a username and password. This setup is more secure and appropriate for production environments where there are multiple services or applications accessing the database.
- `path: /var/lib/pgsql/data/pg_hba.conf` specifies the file to be edited, which is the PostgreSQL `pg_hba.conf` file. This file controls the client authentication for PostgreSQL.
- `regexp: '^host.*all.*all.*127.0.0.1..*ident$'` regular expression searches for lines in the `pg_hba.conf` file that match the pattern of using ident authentication for connections from `127.0.0.1`. ident authentication is based on the operating system user’s identity.
- `line: 'host all all 127.0.0.1/32 md5'` specifies the line to replace the matched pattern with. It changes the authentication method for connections from `127.0.0.1` to `md5`, which is password-based authentication.
- `line: 'host all all ::1/128 md5'` replaces the matching line with one that uses `md5` authentication instead of `ident` for all users and all databases connecting from `::1/128`.

![Install Postgresql and Psycopg2 packages](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-37-39.png)

- Task 6: Create app database
- Task 7: Create db user
- Task 8: Grant db user access to app db

This play block is responsible for setting up a PostgreSQL database environment, specifically for an application. The sixth task creates a new PostgreSQL database with a name defined by the `db_name` variable, ensuring that it is present. The seventh task creates a new PostgreSQL user, using the `db_user` and `db_password` variables to define the username and password, respectively. The eighth task grants the newly created user all necessary privileges on the newly created database, allowing the user to fully manage and interact with the database. Each task is executed as the postgres user, who has the necessary permissions to manage databases, users, and privileges in PostgreSQL.

## Play 4: Airflow Installation in Virtual Environment

![Install Airflow in virtual environment](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-38-22.png)

- Task 1: Ensure Python3 and pip3 are installed
- Task 2: Create a virtual environment for Airflow
- Task 3: Send constraint file
- Task 4: pip upgrade
- Task 5: Install Airflow using pip in the virtual environment
- Task 6: Verify Airflow installation
- Task 7: Print Airflow version

This play block automates the installation of Apache Airflow in a Python virtual environment. The block begins by ensuring that Python 3 and pip (Python's package installer) are installed on the target hosts using the dnf package manager. It then creates a virtual environment in a specified directory (`airflow_install_dir`), ensuring isolation for the Airflow installation. A constraints file is copied to the virtual environment, which helps manage specific versions of Python packages. The script upgrades pip within the virtual environment to the latest version before installing Airflow using pip with the specified version and dependencies, including the Celery component. The installation process is made idempotent by using the creates argument to ensure tasks like virtual environment creation, pip upgrade, and Airflow installation are only executed if necessary. After the installation, the play verifies the installation by checking the Airflow version and then prints the installed version to confirm successful setup.

![Install Airflow in virtual environment](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-38-58.png)

- Task 8: Create Airflow user if not exists
- Task 9: Ensure airflow_home, airflow_install_dir, and airflow_home/dags directory exists
- Task 10: Template a file to /etc/file.conf

This play block is designed to set up the necessary user, directories, and configuration file for Apache Airflow on the target system. The eighth task ensures that a system user named `airflow` exists, creating it if necessary, with no login shell (`/bin/false`) and no home directory by default. This user is essential for running Airflow services securely under its own user account. The ninth task ensures that specific directories required by Airflow exist. It iterates over a list of directory paths (`airflow_home`, `airflow_install_dir`, and `airflow_home/dags`) and ensures that each directory is present, owned by the airflow user and group, and created if it does not already exist. These directories are crucial for storing Airflow configurations, the virtual environment, and DAG (Directed Acyclic Graph) files. The tenth task copies an Airflow configuration file (`airflow.cfg`) to the `airflow_home` directory, ensuring it is owned by the airflow user and group, and that its permissions are set to `0600`, meaning only the owner can read and write the file. This task uses the template module, which allows the configuration file to be dynamically generated from a template, possibly incorporating variables or logic within the `airflow.cfg` file.

![Install Airflow in virtual environment](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-39-15.png)

- Task 11: Initialize Airflow database
- Task 12: Create Airflow admin user

This play block handles the initialization of the Airflow database and the creation of an Airflow admin user, both executed under the airflow user account. The first task initializes the Airflow database by running the airflow `db init` command within the Airflow installation directory. It uses the creates argument to ensure that the initialization is only performed if a specific file (`init_success`) does not already exist, making the task idempotent. The environment variable `AIRFLOW_HOME` is set to the designated Airflow home directory to ensure the command runs in the correct context. The second task creates an Airflow admin user by running the airflow users create command with specified parameters for the username, password, first name, last name, role, and email, all of which are provided via variables. This task is also idempotent, using the creates argument to check for a file (`admin_created`) that indicates the user creation has already been completed. Like the first task, this command is executed with the `AIRFLOW_HOME` environment variable set appropriately.

## Play 5: Systemd Service Files Creation for Airflow Components

![Install Airflow in virtual environment](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-39-37.png)

- Task: Create systemd service file for Airflow webserver

![Install Airflow in virtual environment](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-39-53.png)

- Task: Create systemd service file for Airflow celery worker

![Install Airflow in virtual environment](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-40-15.png)

- Task: Create systemd service file for Airflow scheduler

This play block is responsible for creating a systemd service file to manage the Apache Airflow webserver, celery worker and scheduler as a service on the target hosts, identified as airflow_master and airflow_worker. It starts by defining the content of the service file using the copy module. The service file configures the Airflow webserver to run as a systemd-managed service, setting it to start automatically after the network is available (`After=network.target`). The service is defined with Restart=always to ensure it is restarted automatically if it crashes, with a minimal delay (`RestartSec=1`). The ExecStart command specifies the command to start the Airflow webserver, using the paths defined by variables `airflow_install_dir` and `airflow_home`. The `WorkingDirectory` and `Environment` directives ensure that the service runs in the correct directory and with the appropriate environment variables set. The service file is then copied to the `/etc/systemd/system/airflow-webserver.service` location on the target hosts, with ownership set to root and permissions set to `0644`, allowing the service to be managed by systemd.