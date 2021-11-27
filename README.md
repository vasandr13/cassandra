# Role for Installing and Configuring Cassandra Cluster.

https://github.com/locp/ansible-role-cassandra

https://galaxy.ansible.com/locp/cassandra


---

# https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/configuration/configCassandra_yaml.html#configCassandra_yaml__asterisks


- name: "Install and Configure Cassandra Cluster"

  hosts: all
  
  vars:
    cassandra_configuration:
      commitlog_directory: /commitlog/cassandra/commitlog
      data_file_directories: 
        - /data/cassandra/data
      hints_directory: /data/cassandra/hints
      saved_caches_directory: /data/cassandra/saved_caches
      
      cluster_name: columbia
      commitlog_sync: periodic
      commitlog_sync_period_in_ms: 10000
      listen_address: "{{ ansible_default_ipv4.address }}"
      num_tokens: 16
      partitioner: org.apache.cassandra.dht.Murmur3Partitioner
      endpoint_snitch: SimpleSnitch
      
      
      seed_provider:
        - class_name: "org.apache.cassandra.locator.SimpleSeedProvider"
          parameters:
            # This sets the seed to the first node in the Ansible group called
            # cassandra.
            - seeds: "{{ hostvars[groups['cassandra'][0]]['ansible_default_ipv4'].address }}, {{ hostvars[groups['cassandra'][1]]['ansible_default_ipv4'].address }}"
      start_native_transport: true
      
      # copy-paste from original cassandra.yaml file
      hinted_handoff_enabled: true
      max_hint_window_in_ms: 108000 # 3 hours
      hinted_handoff_throttle_in_kb: 1024
      max_hints_delivery_threads: 2
      hints_flush_period_in_ms: 10000
      max_hints_file_size_in_mb: 128
      batchlog_replay_throttle_in_kb: 1024
      authenticator: AllowAllAuthenticator
      authorizer: AllowAllAuthorizer
      role_manager: CassandraRoleManager
     
      
    cassandra_directories:
      root:
        group: root
        mode: "0755"
        owner: root
        paths:
          - /data
          - /commitlog
      data:
        paths:
          - /data/cassandra
          - /data/cassandra/data
          - /data/cassandra/hints
          - /data/cassandra/saved_caches
      commitlog:
        paths:
          - /commitlog/cassandra
          - /commitlog/cassandra/commitlog
        

  roles:
    - {role: "cassandra", cassandra__action: "configure_repofile"}
    - {role: "cassandra", cassandra__action: "install_packages"}
    - {role: "cassandra", cassandra__action: "create_directories"}
    - {role: "cassandra", cassandra__action: "configure_cluster"}
    - {role: "cassandra", cassandra__action: "service"}    
    

