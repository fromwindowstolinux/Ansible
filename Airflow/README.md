 # Ansible Configuration

This section talks about the Ansible playbook designed for the deployment and configuration of the Airflow environment, along with related services such as Redis and PostgreSQL. Each block in the playbook has a specific purpose, and this section explains why each block is structured the way it is.

## Play 1: SELinux in Permissive Mode

![SELinux in Permissive Mode]()

This play block is designed to configure SELinux (Security-Enhanced Linux), a security feature in Linux, to run in permissive mode on specified hosts. The play targets the hosts named `airflow_master` and `airflow_worker`, which are parts of the Apache Airflow deployment. The `become: yes` directive allows the tasks to be executed as the root user. The play uses a `vars.yml` file to import any necessary variables. The specific task within this block sets SELinux to "permissive" mode, which means that while SELinux policies are still enforced and violations are logged, they do not prevent operations from being carried out. This is often done to troubleshoot SELinux-related issues or to allow software that is not fully compatible with SELinux to run without being blocked.

