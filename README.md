# cassandra2

---

- name: "Install and Configure Cassandra"

  hosts: all

  vars:
    cassandra_configure_repo: true


  roles:
    - role: cassandra

