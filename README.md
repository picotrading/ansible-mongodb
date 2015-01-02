mongodb
=======

Ansible Role which installs MongoDB from RPM package and configures it with the
YAML format of the config file.

This role doesn't download the RPM package. In order to install it, it needs to
be put into a repo (see
[`yumrepo`](https://github.com/picotrading/ansible-yumrepo) role for RedHat-based
distros). You can encapsulate this role to manage the installation of the RPM
package automatically. The configuraton of the role is done in such way that it's
not necessary to change the role for any kind of modification. All can be changed
via parameters used by the role. That makes this role absolutely universal. See
examples bellow.

Please report any issues or send PR.


Examples
--------

```
---

# Example of how to use the role with default parameters
- hosts: myhost1
  vars:
    # This is needed to make the init.d script working
    mongodb_processManagement_fork: true
  roles:
    - role: yumrepo
      yumrepo_repos:
        mongodb:
          name: MongoDB repository
          baseurl: http://downloads-distro.mongodb.org/repo/redhat/os/$basearch/
          gpgcheck: 0
    - mongodb

# The above example is using mongodb_config_simple config template which you can
# modify like this:
- hosts: myhost2
  roles:
    - role: yumrepo
      yumrepo_repos:
        mongodb:
          name: MongoDB repository
          baseurl: http://downloads-distro.mongodb.org/repo/redhat/os/$basearch/
          gpgcheck: 0
    - role: mongodb
      mongodb_net_bindIp: 127.0.0.1
      mongodb_net_wireObjectCheck: false
      mongodb_net_unixDomainSocket_enabled: true
      mongodb_processManagement_fork: true
      mongodb_systemLog_logAppend: true
      mongodb_systemLog_timeStampFormat: iso8601-utc

# You can also use the mongodb_config_replica, mongodb_config_mongos or
# mongodb_config_mongos_server by assigning one of them to the mongodb_config:
- hosts: myhost3
  roles:
    - role: yumrepo
      yumrepo_repos:
        mongodb:
          name: MongoDB repository
          baseurl: http://downloads-distro.mongodb.org/repo/redhat/os/$basearch/
          gpgcheck: 0
    - role: mongodb
      mongodb_config: "{{ mongodb_config_replica }}"
      # Set other parameters of this template:
      mongodb_processManagement_fork: true

# Or you can create completely your own config template:
- hosts: myhost4
  roles:
    - role: yumrepo
      yumrepo_repos:
        mongodb:
          name: MongoDB repository
          baseurl: http://downloads-distro.mongodb.org/repo/redhat/os/$basearch/
          gpgcheck: 0
    - role: mongodb
      mongodb_config:
        storage:
          # This uses the default value defined in defaults/main.yaml
          dbPath: "{{ mongodb_storage_dbPath }}
          directoryPerDB: "{{ mongodb_storage_directoryPerDB }}
          journal:
            enabled: "{{ mongodb_storage_journal_enabled }}"
        systemLog:
          destination: "{{ mongodb_systemLog_destination }}"
          path: "{{ mongodb_systemLog_path }}"
          # This sets a custom value
          logAppend: true
          timeStampFormat: iso8601-utc
        processManagement:
          fork: true
        net:
          bindIp: "{{ mongodb_net_bindIp }}"
          port: "{{ mongodb_net_port }}"
          wireObjectCheck: false
          unixDomainSocket:
            enabled: true
```


Role variables
--------------

List of variables used by the role:

```
# Default list of RPM packages to install
mongodb_packages:
  - mongodb-org

# Default config file location
mongodb_config_file: /etc/mongod.conf

# Default sysconfig file location
mongodb_sysconfig_file: /etc/sysconfig/mongod

# Default service name
mongodb_service_name: mongod

# Default UNIX mongo user and group
mongodb_user: mongod
mongodb_group: mongod

# Default NUMA arguments
mongodb_numactl_args: "--interleave=all"


### Get all options from the doc on the web (not perfect but it helps)
# wget -q -O - http://docs.mongodb.org/manual/reference/configuration-options/ | \
# egrep '(<dt id="[a-zA-Z0-9\.]+[^"]"|<p><em>Default)' | \
# sed -r -e 's/.*="/#/' -e 's/".*//' -e 's/.*: /: /' -e 's/<\/p>//' | \
# egrep -v '^#pre$' | sed 's/^#/mongodb_/' | \
# sed -n -r 'h; /mongodb_/{s/\./_/g}; p' | tr '\n' ' ' | \
# sed -r 's/mongodb_/\nmongodb_/g' | sed 's/ : /: /'


mongodb_auditLog_destination: file
mongodb_auditLog_filter: {}
mongodb_auditLog_format: JSON
mongodb_auditLog_path: /var/log/mongodb/audit.log

mongodb_auditLog:
  destination: "{{ mongodb_auditLog_destination }}"
  filter: "{{ mongodb_auditLog_filter }}"
  format: "{{ mongodb_auditLog_format }}"
  path: "{{ mongodb_auditLog_path }}"


mongodb_net_http_enabled: false
mongodb_net_http_JSONPEnabled: false
mongodb_net_http_RESTInterfaceEnabled: false

mongodb_net_http:
  enabled: "{{ mongodb_net_http_enabled }}"
  JSONPEnabled: "{{ mongodb_net_http_JSONPEnabled }}"
  RESTInterfaceEnabled: "{{ mongodb_net_http_RESTInterfaceEnabled }}"

mongodb_net_ssl_allowInvalidCertificates: false
mongodb_net_ssl_CAFile: ''
mongodb_net_ssl_clusterFile: ''
mongodb_net_ssl_clusterPassword: ''
mongodb_net_ssl_CRLFile: ''
mongodb_net_ssl_FIPSMode: false
mongodb_net_ssl_mode: disabled
mongodb_net_ssl_PEMKeyFile: ''
mongodb_net_ssl_PEMKeyPassword: ''
mongodb_net_ssl_sslOnNormalPorts: true
mongodb_net_ssl_weakCertificateValidation: false

mongodb_net_ssl:
  allowInvalidCertificates: "{{ mongodb_net_ssl_allowInvalidCertificates }}"
  CAFile: "{{ mongodb_net_ssl_CAFile }}"
  clusterFile: "{{ mongodb_net_ssl_clusterFile }}"
  clusterPassword: "{{ mongodb_net_ssl_clusterPassword }}"
  CRLFile: "{{ mongodb_net_ssl_CRLFile }}"
  FIPSMode: "{{ mongodb_net_ssl_FIPSMode }}"
  mode: "{{ mongodb_net_ssl_mode }}"
  PEMKeyFile: "{{ mongodb_net_ssl_PEMKeyFile }}"
  PEMKeyPassword: "{{ mongodb_net_ssl_PEMKeyPassword }}"
  sslOnNormalPorts: "{{ mongodb_net_ssl_sslOnNormalPorts }}"
  weakCertificateValidation: "{{ mongodb_net_ssl_weakCertificateValidation }}"

mongodb_net_unixDomainSocket_enabled: false
mongodb_net_unixDomainSocket_pathPrefix: /tmp

mongodb_net_unixDomainSocket:
  enabled: "{{ mongodb_net_unixDomainSocket_enabled }}"
  pathPrefix: "{{ mongodb_net_unixDomainSocket_pathPrefix }}"

mongodb_net_bindIp: 0.0.0.0
mongodb_net_ipv6: false
mongodb_net_maxIncomingConnections: 1000000
mongodb_net_port: 27017
mongodb_net_wireObjectCheck: true

mongodb_net:
  bindIp: "{{ mongodb_net_bindIp }}"
  http: "{{ mongodb_net_http }}"
  ipv6: "{{ mongodb_net_ipv6 }}"
  maxIncomingConnections: "{{ mongodb_net_maxIncomingConnections }}"
  port: "{{ mongodb_net_port }}"
  ssl: "{{ mongodb_net_ssl }}"
  unixDomainSocket: "{{ mongodb_net_unixDomainSocket }}"
  wireObjectCheck: "{{ mongodb_net_wireObjectCheck }}"


mongodb_operationProfiling_mode: 'off'
mongodb_operationProfiling_slowOpThresholdMs: 100

mongodb_operationProfiling:
  mode: "{{ mongodb_operationProfiling_mode }}"
  slowOpThresholdMs: "{{ mongodb_operationProfiling_slowOpThresholdMs }}"


mongodb_processManagement_pidFilePath: /var/run/mongodb/mongod.pid
mongodb_processManagement_fork: false

mongodb_processManagement:
  pidFilePath: "{{ mongodb_processManagement_pidFilePath }}"
  fork: "{{ mongodb_processManagement_fork }}"


mongodb_replication_localPingThresholdMs: 15
mongodb_replication_oplogSizeMB: 10240
mongodb_replication_replSetName: rs1
mongodb_replication_secondaryIndexPrefetch: all

mongodb_replication:
  localPingThresholdMs: "{{ mongodb_replication_localPingThresholdMs }}"
  oplogSizeMB: "{{ mongodb_replication_oplogSizeMB }}"
  replSetName: "{{ mongodb_replication_replSetName }}"
  secondaryIndexPrefetch: "{{ mongodb_replication_secondaryIndexPrefetch }}"


mongodb_security_sasl_hostName: localhost
mongodb_security_sasl_saslauthdSocketPath: /var/run/saslauthd
mongodb_security_sasl_serviceName: mongodb

mongodb_security_sasl:
  hostName: "{{ mongodb_security_sasl_hostName }}"
  saslauthdSocketPath: "{{ mongodb_security_sasl_saslauthdSocketPath }}"
  serviceName: "{{ mongodb_security_sasl_serviceName }}"

mongodb_security_authorization: disabled
mongodb_security_clusterAuthMode: keyFile
mongodb_security_javascriptEnabled: true
mongodb_security_keyFile: ''

mongodb_security:
  authorization: "{{ mongodb_security_authorization }}"
  clusterAuthMode: "{{ mongodb_security_clusterAuthMode }}"
  javascriptEnabled: "{{ mongodb_security_javascriptEnabled }}"
  keyFile: "{{ mongodb_security_keyFile }}"
  mongodb_security_sasl: "{{ mongodb_security_sasl }}"


mongodb_setParameter: {}


mongodb_sharding_archiveMovedChunks: false
mongodb_sharding_autoSplit: false
mongodb_sharding_chunkSize: 64
mongodb_sharding_clusterRole: shardsvr
mongodb_sharding_configDB: []

mongodb_sharding:
  archiveMovedChunks: "{{ mongodb_sharding_archiveMovedChunks }}"
  autoSplit: "{{ mongodb_sharding_autoSplit }}"
  chunkSize: "{{ mongodb_sharding_chunkSize }}"
  clusterRole: "{{ mongodb_sharding_clusterRole }}"
  configDB: "{{ mongodb_sharding_configDB }}"


mongodb_snmp_master: false
mongodb_snmp_subagent: true
mongodb_snmp:
  master: "{{ mongodb_snmp_master }}"
  subagent: "{{ mongodb_snmp_subagent }}"


mongodb_storage_journal_commitIntervalMs: 100
mongodb_storage_journal_debugFlags: 0
mongodb_storage_journal_enabled: true

mongodb_storage_journal:
  commitIntervalMs: "{{ mongodb_storage_journal_commitIntervalMs }}"
  debugFlags: "{{ mongodb_storage_journal_debugFlags }}"
  enabled: "{{ mongodb_storage_journal_enabled }}"

mongodb_storage_quota_enforced: false
mongodb_storage_quota_maxFilesPerDB: 8

mongodb_storage_quota:
  enforced: "{{ mongodb_storage_quota_enforced }}"
  maxFilesPerDB: "{{ mongodb_storage_quota_maxFilesPerDB }}"

mongodb_storage_dbPath: /var/lib/mongo
mongodb_storage_directoryPerDB: true
mongodb_storage_indexBuildRetry: true
mongodb_storage_nsSize: 16
mongodb_storage_preallocDataFiles: true
mongodb_storage_repairPath: /data/db/_tmp
mongodb_storage_smallFiles: false
mongodb_storage_syncPeriodSecs: 60

mongodb_storage:
  dbPath: "{{ mongodb_storage_dbPath }}"
  directoryPerDB: "{{ mongodb_storage_directoryPerDB }}"
  indexBuildRetry: "{{ mongodb_storage_indexBuildRetry }}"
  nsSize: "{{ mongodb_storage_nsSize }}"
  preallocDataFiles: "{{ mongodb_storage_preallocDataFiles }}"
  repairPath: "{{ mongodb_storage_repairPath }}"
  smallFiles: "{{ mongodb_storage_smallFiles }}"
  syncPeriodSecs: "{{ mongodb_storage_syncPeriodSecs }}"
  journal: "{{ mongodb_storage_journal }}"
  quota: "{{ mongodb_storage_quota }}"

mongodb_systemLog_destination: file
mongodb_systemLog_logAppend: false
mongodb_systemLog_path: /var/log/mongodb/mongod.log
mongodb_systemLog_quiet: false
mongodb_systemLog_syslogFacility: user
mongodb_systemLog_timeStampFormat: iso8601-local
mongodb_systemLog_traceAllExceptions: false
mongodb_systemLog_verbosity: 0

mongodb_systemLog:
  destination: "{{ mongodb_systemLog_destination }}"
  logAppend: "{{ mongodb_systemLog_logAppend }}"
  path: "{{ mongodb_systemLog_path }}"
  quiet: "{{ mongodb_systemLog_quiet }}"
  syslogFacility: "{{ mongodb_systemLog_syslogFacility }}"
  timeStampFormat: "{{ mongodb_systemLog_timeStampFormat }}"
  traceAllExceptions: "{{ mongodb_systemLog_traceAllExceptions }}"
  verbosity: "{{ mongodb_systemLog_verbosity }}"


# Complete configuration template (mainly for the Ansible role testing)
mongodb_config_complete:
  auditLog: "{{ mongodb_auditLog }}"
  net: "{{ mongodb_net }}"
  operationProfiling: "{{ mongodb_operationProfiling }}"
  processManagement: "{{ mongodb_processManagement }}"
  replication: "{{ mongodb_replication }}"
  security: "{{ mongodb_security }}"
  setParameter: "{{ mongodb_setParameter }}"
  sharding: "{{ mongodb_sharding }}"
  snmp: "{{ mongodb_snmp }}"
  storage: "{{ mongodb_storage }}"
  systemLog: "{{ mongodb_systemLog }}"


###
# The configuration bellow was taken from this post:
# http://dba.stackexchange.com/a/82592
###


# Standalone mongod template with the default port, path, journal settings
mongodb_config_simple:
  storage:
    dbPath: "{{ mongodb_storage_dbPath }}"
    directoryPerDB: "{{ mongodb_storage_directoryPerDB }}"
    journal:
      enabled: "{{ mongodb_storage_journal_enabled }}"
  systemLog:
    destination: "{{ mongodb_systemLog_destination }}"
    path: "{{ mongodb_systemLog_path }}"
    logAppend: "{{ mongodb_systemLog_logAppend }}"
    timeStampFormat: "{{ mongodb_systemLog_timeStampFormat }}"
  processManagement:
    fork: "{{ mongodb_processManagement_fork }}"
  net:
    bindIp: "{{ mongodb_net_bindIp }}"
    port: "{{ mongodb_net_port }}"
    wireObjectCheck: "{{ mongodb_net_wireObjectCheck }}"
    unixDomainSocket:
      enabled: "{{ mongodb_net_unixDomainSocket_enabled }}"


# Sample config file for a typical production replica set member with
# authentication enabled and running as part of a sharded cluster
mongodb_config_replica:
  storage:
    dbPath: "{{ mongodb_storage_dbPath }}"
    directoryPerDB: "{{ mongodb_storage_directoryPerDB }}"
    journal:
      enabled: "{{ mongodb_storage_journal_enabled }}"
  systemLog:
    destination: "{{ mongodb_systemLog_destination }}"
    path: "{{ mongodb_systemLog_path }}"
    logAppend: "{{ mongodb_systemLog_logAppend }}"
    timeStampFormat: "{{ mongodb_systemLog_timeStampFormat }}"
  replication:
    oplogSizeMB: "{{ mongodb_replication_oplogSizeMB }}"
    replSetName: "{{ mongodb_replication_replSetName }}"
  processManagement:
    fork: "{{ mongodb_processManagement_fork }}"
  net:
    bindIp: "{{ mongodb_net_bindIp }}"
    port: "{{ mongodb_net_port }}"
  security:
    keyFile: "{{ mongodb_security_keyFile }}"
    authorization: "{{ mongodb_security_authorization }}"
  sharding:
    clusterRole: "{{ mongodb_sharding_clusterRole }}"


# Sample of mongos configuration
mongodb_config_mongos:
  sharding:
    configDB: "{{ mongodb_sharding_configDB }}"
    autoSplit: "{{ mongodb_sharding_autoSplit }}"
  systemLog:
    destination: "{{ mongodb_systemLog_destination }}"
    path: "{{ mongodb_systemLog_path }}"
    logAppend: "{{ mongodb_systemLog_logAppend }}"
    timeStampFormat: "{{ mongodb_systemLog_timeStampFormat }}"
  processManagement:
    fork: "{{ mongodb_processManagement_fork }}"
  net:
    bindIp: "{{ mongodb_net_bindIp }}"
    port: "{{ mongodb_net_port }}"
    maxIncomingConnections: "{{ mongodb_net_maxIncomingConnections }}"
  security:
    keyFile: "{{ mongodb_security_keyFile }}"
    authorization: "{{ mongodb_security_authorization }}"


# Sample of configuration of the sharded cluster server
mongodb_config_mongos_server:
  storage:
    dbPath: "{{ mongodb_storage_dbPath }}"
    journal:
      enabled: "{{ mongodb_storage_journal_enabled }}"
  systemLog:
    destination: "{{ mongodb_systemLog_destination }}"
    path: "{{ mongodb_systemLog_path }}"
    logAppend: "{{ mongodb_systemLog_logAppend }}"
    timeStampFormat: "{{ mongodb_systemLog_timeStampFormat }}"
  processManagement:
    fork: "{{ mongodb_processManagement_fork }}"
  net:
    bindIp: "{{ mongodb_net_bindIp }}"
    port: "{{ mongodb_net_port }}"
  security:
    keyFile: "{{ mongodb_security_keyFile }}"
    authorization: "{{ mongodb_security_authorization }}"
  sharding:
    clusterRole: "{{ mongodb_sharding_clusterRole }}"


# Default MongoDB config
mongodb_config: "{{ mongodb_config_simple }}"
```


Dependencies
------------

* [`yumrepo`](https://github.com/picotrading/ansible-yumrepo) role (optional)


License
-------

MIT


Author
------

Jiri Tyr
