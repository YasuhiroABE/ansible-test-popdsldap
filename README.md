YasuhiroABE.test-popdsldap
==========================

An example of openldap to populate dataset from given csv file.

This module configures the server1 openldap servers with sample data as like as follows:

[server1] YasuhiroABE.test-popdsldap

* Top-level (Example) directory (DN: dc=example,dc=com)
    * ou=people,dc=example,dc=com
    * ou=group,dc=example,dc=com

[server2] YasuhiroABE.test-transldap

* Translucent directory against server1 (override some attributes on ou=people,dc=example,dc=com of serve1)

[server3] YasuhiroABE.test-metaldap

* Unified directory: ou=proxy,dc=example,dc=com
    * ou=people,ou=proxy,dc=example,dc=com
    * ou=group,ou=proxy,dc=example,dc=com

Requirements
------------

This role is tested on the following platforms.

### Ansible
- Version 2.5

### Distributions
- Ubuntu 16.04

Role Variables
--------------

### Default
    popdsldap_init: true
    popdsldap_basedn_dbdirectory: "/var/lib/ldap/popds"
    popdsldap_removal_filepath:
      - '/etc/ldap/slapd.d/cn=config/olcDatabase={1}mdb.ldif'
      - '/etc/ldap/slapd.d/cn=config/olcDatabase={1}mdb'
      - '/var/lib/ldap/data.mdb'
      - '/var/lib/ldap/lock.mdb'
      - "{{ popdsldap_basedn_dbdirectory }}"
      
    popdsldap_add_dirinfo:
      - { path: "{{ popdsldap_basedn_dbdirectory }}", owner: 'openldap', group: 'openldap', mode: '0750' }

    popdsldap_additional_schema:
      - ppolicy

    popdsldap_ldapadd_cn_config_files:
      - popdsldap_01_create_backenddb.ldif

    popdsldap_ldapadd_backenddb_files:
      - popdsldap_02_add_subentries.ldif
      - popdsldap_03_popup_people_data.ldif
      - popdsldap_04_popup_group_data.ldif
    
    popdsldap_copy_ldif_files:
      - popdsldap_01_create_backenddb.ldif
      - popdsldap_02_add_subentries.ldif
      - popdsldap_03_popup_people_data.ldif
      - popdsldap_04_popup_group_data.ldif

    popdsldap_copy_ldif_perms:
      owner: root
      group: root
      mode: '0600'

    popdsldap_metadb_index: 1
    popdsldap_basedn: "dc=example,dc=com"
    popdsldap_basedn_top_object: "dc"
    popdsldap_basedn_top_objecttype: "dcObject"
    popdsldap_basedn_top_entry: "example"
    popdsldap_cn_admin_dnprefix: "cn=admin"
    popdsldap_cn_admin_password: "transpasswd"

    popdsldap_sub_entry:
      - { suffix_object: "ou", suffix_name: "people", suffix_objecttype: "organizationalUnit" }
      - { suffix_object: "ou", suffix_name: "group", suffix_objecttype: "organizationalUnit" }
    
    ## To populate database, load files/popds_{{ subentry_name }}_data.yml with specified objectClasses.
    popdsldap_populate_data:
      people: "{{ lookup('file', 'files/popds_people_data.yml')|from_yaml }}"
      group:  "{{ lookup('file', 'files/popds_group_data.yml')|from_yaml }}"

Dependencies
------------

* This role removes existing directory files specified with the popdsldap_removal_filepath variable.

Example Playbook
----------------

Following vars entries need to be configured for your environment.

    - name: ansible-test-transldap role test
      hosts: all
      vars:
        mfts_additional_packages:
          - slapd
          - ldap-utils
      roles:
        - YasuhiroABE.myfavorite-setting
        - ansible-test-popdsldap

License
-------

Apache License 2.0

Author Information
------------------

[Yasuhiro ABE](http://www.yasundial.org/foaf.xml)

