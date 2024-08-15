 # Ansible Configuration

This section talks about the Ansible playbook designed for the deployment and configuration of the Airflow environment, along with related services such as Redis and PostgreSQL. Each block in the playbook has a specific purpose, and this section explains why each block is structured the way it is.

## Play 1: SELinux in Permissive Mode

![SELinux in Permissive Mode](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-33-45.png)

Task 1: Put SELinux in permissive mode

This play block is designed to configure SELinux (Security-Enhanced Linux), a security feature in Linux, to run in permissive mode on specified hosts. The play targets the hosts named `airflow_master` and `airflow_worker`, which are parts of the Apache Airflow deployment. The `become: yes` directive allows the tasks to be executed as the root user. The play uses a `vars.yml` file to import any necessary variables. The specific task within this block sets SELinux to "permissive" mode, which means that while SELinux policies are still enforced and violations are logged, they do not prevent operations from being carried out. This is often done to troubleshoot SELinux-related issues or to allow software that is not fully compatible with SELinux to run without being blocked.

## Play 2: Redis Installation, Start, and Enable

![Install, start, and enable Redis](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-34-00.png)

Task 1: Install Redis

Task 2: Start and enable Redis service

This play block is responsible for installing, starting, and enabling the Redis service on hosts identified as airflow_redis. The first task installs the Redis package using the dnf package manager. The package installation is ensured by setting the `state: present`, meaning the Redis package will be installed if it is not already present. The second task starts the Redis service and ensures it is enabled to start automatically on boot. The service module is used to manage the Redis service, with `state: started` ensuring that Redis is running, and `enabled: yes` ensuring that the Redis service will start automatically on system boot. This setup made Redis available as a backend service for managing task queues or as a cache.

## Play 3: PostgreSQL and Psycopg2 Installation and Configuration

![Install Postgresql and Psycopg2 packages](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-36-50.png)


Task 1: Install Postgresql and Psycopg2 packages

Task 2: Initialize PostgreSQL

This play block prepares a PostgreSQL environment by installing the necessary software and initializing the database, making it ready for use by Apache Airflow for managing metadata and other data storage needs. The play block is designed to set up a PostgreSQL environment on a hosts identified as `airflow_postgresql`. The first task installs the PostgreSQL client and server packages, along with psycopg2, a PostgreSQL adapter for Python, and the Python 3.11 development package. The installation is managed by the dnf module. The second task initializes the PostgreSQL database using the `postgresql-setup initdb` command. This task will only run if the PostgreSQL initialization has not already been performed, as indicated by the absence of the `pg_hba.conf` file. The creates argument ensures that the task is only executed if the specified file does not exist, preventing reinitialization. 

![Install Postgresql and Psycopg2 packages](https://github.com/fromwindowstolinux/Ansible/blob/main/Airflow/images/Screenshot%20from%202024-08-14%2009-37-24.png)

Task 3: Ensure pg_hba.conf has md5 authentication (ipv4)

Task 4: Ensure pg_hba.conf has md5 authentication (ipv6)

Task 5: Start and enable PostgreSQL service

This play block is designed to configure PostgreSQL to use md5 password-based authentication for both IPv4 and IPv6 connections and to ensure that the PostgreSQL service is running and enabled to start at boot. The third task modifies the pg_hba.conf file, which controls client authentication in PostgreSQL, to replace any existing ident authentication for IPv4 localhost connections (127.0.0.1/32) with md5 authentication. The fourth task performs a similar operation for IPv6 localhost connections (::1/128). Both tasks use the lineinfile module to ensure that these changes are present in the configuration file, enhancing security by requiring a username and password for connections. The fifth task ensures that the PostgreSQL service is restarted to apply these changes and is set to start automatically on boot, ensuring the database server is always available when needed.

Configuring md5 authentication ensures that Airflow can connect to the PostgreSQL database using a username and password. This setup is more secure and appropriate for production environments where there are multiple services or applications accessing the database.
- path: /var/lib/pgsql/data/pg_hba.conf specifies the file to be edited, which is the PostgreSQL pg_hba.conf file. This file controls the client authentication for PostgreSQL.
- regexp: '^host.*all.*all.*127.0.0.1..*ident$' regular expression searches for lines in the pg_hba.conf file that match the pattern of using ident authentication for connections from 127.0.0.1. ident authentication is based on the operating system user’s identity.
- line: 'host all all 127.0.0.1/32 md5' specifies the line to replace the matched pattern with. It changes the authentication method for connections from 127.0.0.1 to md5, which is password-based authentication.
- line: 'host all all ::1/128 md5' replaces the matching line with one that uses md5 authentication instead of ident for all users and all databases connecting from ::1/128.

