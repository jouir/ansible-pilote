# Variables

Senstivie data should be encrypted using
[ansible-vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html).

## bacula_catalog_name

Name of the Bacula catalog.

```yaml
bacula_catalog_name: HomeCatalog
```

## bacula_clients

List of Bacula Clients.

```yaml
bacula_clients:
  - name: pilote-fd
    address: localhost
    catalog: HomeCatalog
    password: ***
    file_retention: 60 days
    job_retention: 6 months
    autoprune: 'yes'
  - name: vps-fd
    address: 192.168.0.1
    catalog: HomeCatalog
    password: ***
    file_retention: 60 days
    job_retention: 6 months
    autoprune: 'yes'
  - name: storage1-fd
    address: 192.168.0.2
    catalog: HomeCatalog
    password: ***
    file_retention: 60 days
    job_retention: 6 months
    autoprune: 'yes'
```

## bacula_device_archive_device

Directory of the Device where to store Bacula backups.

```yaml
bacula_device_archive_device: /storage/bacula/backup
```

## bacula_device_name

Name of the Bacula Device.

```yaml
bacula_device_name: FileStorage
```

## bacula_director_address

Address of the Bacula director.

```yaml
bacula_director_address: 127.0.0.1
```

## bacula_director_name

Name of the Bacula director.

```yaml
bacula_director_name: pilote-dir
```

## bacula_director_password

Password of the Bacula director.

```yaml
bacula_director_password: ***
```

## bacula_filedaemon_address

Address of the Bacula Client (File Daemon).

```yaml
bacula_filedaemon_address: 127.0.0.1
```

## bacula_filedaemon_name

Name of the Bacula Client (File Daemon).

```yaml
bacula_filedaemon_name: pilote-fd
```

## bacula_filedaemon_password

Password of the Bacula Client (File Daemon).

```yaml
bacula_filedaemon_password: ***
```

## bacula_filesets

List of Bacula File Sets.

```yaml
bacula_filesets:
  - name: DebianFileSet
    include:
      options:
        signature: MD5
        compression: GZIP
      files:
        - /etc
        - /var/log
        - /root
        - /home
    exclude:
      files:
        - '*~'
  - name: CatalogFileSet
    include:
      options:
        signature: MD5
        compression: GZIP
      files:
        - /var/lib/bacula/bacula.sql
  - name: InfluxDBFileSet
    include:
      options:
        signature: MD5
      files:
        - /var/lib/bacula/influxdb
  - name: GrafanaFileSet
    include:
      options:
        signature: MD5
      files:
        - /var/lib/bacula/grafana
```

## bacula_jobs

List of Bacula Jobs.

```yaml
bacula_jobs:
  - name: BackupPilote
    client: pilote-fd
    fileset: DebianFileSet
  - name: BackupStorage1
    client: storage1-fd
    fileset: DebianFileSet
  - name: BackupStorage2
    client: storage2-fd
    fileset: DebianFileSet
  - name: BackupStorage3
    client: storage3-fd
    fileset: DebianFileSet
  - name: BackupCatalog
    client: pilote-fd
    level: Full
    fileset: CatalogFileSet
    schedule: DefaultScheduleAfterBackup
    run_before_job: /etc/bacula/scripts/make_catalog_backup.pl HomeCatalog
    run_after_job: /etc/bacula/scripts/delete_catalog_backup
    priority: 11  # run after main backup
  - name: BackupInfluxDB
    client: storage1-fd
    fileset: InfluxDBFileSet
    schedule: DefaultScheduleAfterBackup
    client_run_before_job: /etc/bacula/scripts/influxdb-backup %l
    client_run_after_job: /etc/bacula/scripts/influxdb-cleanup
    priority: 11  # run after main backup
  - name: BackupGrafana
    client: storage1-fd
    level: Full
    fileset: GrafanaFileSet
    schedule: DefaultScheduleAfterBackup
    client_run_before_job: /etc/bacula/scripts/grafana-backup
    client_run_after_job: /etc/bacula/scripts/grafana-cleanup
    priority: 11  # run after main backup
  - name: RestoreFiles
    type: Restore
    client: storage1-fd
    storage: storage1-sd
    fileset: DebianFileSet  # required but not used
    pool: FullFile  # required but not used
    messages: Standard
    where: /storage/bacula/restore
```

## bacula_pools

List of Bacula Pools.

```yaml
bacula_pools:
  - name: FullFile
    pool_type: Backup
    recycle: 'yes'
    auto_prune: 'yes'
    volume_retention: 10 years
    storage: storage1-sd
    maximum_volume_bytes: 1G
    maximum_volumes: 100
    labelformat: Full-
  - name: DiffFile
    pool_type: Backup
    recycle: 'yes'
    auto_prune: 'yes'
    volume_retention: 6 weeks
    storage: storage1-sd
    maximum_volume_bytes: 1G
    maximum_volumes: 100
    labelformat: Diff-
  - name: IncrFile
    pool_type: Backup
    recycle: 'yes'
    auto_prune: 'yes'
    volume_retention: 3 weeks
    storage: storage1-sd
    maximum_volume_bytes: 1G
    maximum_volumes: 100
    labelformat: Incr-
```

## bacula_schedules

List of Bacula Schedules.

```yaml
bacula_schedules:
  - name: DefaultSchedule
    runs:
      - datetime: 1st sun at 0:00
        job_overrides:
          level: Full
      - datetime: 2nd-5th sun at 0:00
        job_overrides:
          level: Differential
      - datetime: mon-sat at 0:00
        job_overrides:
          level: Incremental
  - name: DefaultScheduleAfterBackup
    runs:
      - datetime: sun-sat at 0:00
        job_overrides:
          level: Full
```

## bacula_storage_address

Address of the Bacula Storage.

```yaml
bacula_storage_address: 127.0.0.1
```

## bacula_storage_name

Name of the Bacula Storage.

```yaml
bacula_storage_name: storage1-sd
```

## bacula_storage_password

Password of the Bacula Storage.

```yaml
bacula_storage_password: ***
```

## bacula_storages

List of Bacula Storages.

```yaml
bacula_storages:
  - name: storage1-sd
    address: 192.168.0.2
    password: ***
    device: FileStorage
    media_type: File
```

## easyrsa_ca_dir

Path to the CA directory to create.

```yaml
easyrsa_ca_dir: /var/lib/easyrsa
```

## easyrsa_clients

List of client hostnames that will have RSA certificates.

```yaml
easyrsa_clients:
  - pilote
  - storage1
  - storage2
  - storage3
  - vps
```

## hostname

Name of the remote host.

```yaml
hostname: pilote
```

## local_subnet

Local subnet where the remote host lives.

```yaml
local_subnet: 192.168.0.0/24
```

## mosquitto_passwords

List of usernames and passwords to defined mosquitto users.

```yaml
mosquitto_passwords:
  - user: telegraf
    hash: '$***'
  - user: nagios
    hash: '$***'
```

See [mosquitto_passwd](https://mosquitto.org/man/mosquitto_passwd-1.html)
command to generate the hash file.

## nagios_commands

List of Nagios commands.

```yaml
nagios_commands:
  - command_name: check_https_vhost_certificate
    command_line: /usr/lib/nagios/plugins/check_http --ssl --sni -I '$HOSTADDRESS$' -H '$ARG1$' -C '$ARG2$'
```

## nagios_contact_groups

List of Nagios contact groups.

```yaml
nagios_contact_groups:
  - contactgroup_name: admins
    alias: Nagios Administrators
    members:
      - admin
      - telegram
```

## nagios_contacts

List of Nagios contacts.

```yaml
nagios_contacts:
  - contact_name: admin
    use: generic-contact
    alias: Nagios Admin
    email: noreply@nonexistant.com
    host_notifications_enabled: 0
    service_notifications_enabled: 0
  - contact_name: telegram
    use: generic-contact
    alias: Telegram notifications
    pager: 000000000
    email: noreply@nonexistant.com
    service_notification_commands: notify-service-by-telegram
    host_notification_commands: notify-host-by-telegram
```

## nagios_hostgroups

List of Nagios host groups.

```yaml
nagios_hostgroups:
  - hostgroup_name: linux-servers
    alias: Linux servers
    members:
      - pilote
      - vps
      - storage1
      - storage2
      - storage3
  - hostgroup_name: web-servers
    alias: Web servers
    members:
      - vps
```

## nagios_hosts

List of Nagios hosts.

```yaml
nagios_hosts:
  - use: home-host
    host_name: pilote
    alias: pilote
    address: 127.0.0.1
  - use: home-host
    host_name: vps
    alias: vps
    address: 10.8.0.1
```

## nagios_host_templates

List of Nagios host templates.

```yaml
nagios_host_templates:
  - name: home-host
    use: generic-host
    check_command: check-host-alive
    contact_groups: admins
    notification_options:
      - d
      - u
      - r
    check_interval: 5
    retry_interval: 5  # retry every 5 minutes
    max_check_attempts: 12  # alert at 1 hour (12x5 minutes)
    notification_interval: 720  # resend notifications every 12 hours
```

## nagios_htdigest_users

List of users for basic authentication.

```yaml
nagios_htdigest_users:
  - name: admin
    hash: '...'
```

## nagios_service_dependencies

List of Nagios service dependencies.

```yaml
nagios_service_dependencies:
  - host_name: pilote
    service_description: ovhcloud_voip
    dependent_host_name: pilote
    dependent_service_description: ovhcloud_ping
    execution_failure_criteria: u
    notification_failure_criteria: u
```

## nagios_services

List of Nagios services.

```yaml
nagios_services:
  - use: home-service
    hostgroup_name: linux-servers
    service_description: load
    check_command: check_nrpe_nossl!check_load
  - use: home-service
    hostgroup_name: web-servers
    service_description: https_monitoring_tld_certificate
    check_command: check_https_vhost_certificate!monitoring.tld!1
```

## nagios_service_templates

List of Nagios service templates.

```yaml
nagios_service_templates:
  - name: home-service
    use: generic-service
    contact_groups: admins
    check_interval: 5
    retry_interval: 5  # retry every 5 minutes
    max_check_attempts: 12  # alert at 1 hour (12x5 minutes)
    notification_interval: 720  # 12 hours
  - name: public-service
    use: generic-service
    contact_groups: admins
    check_interval: 1
    retry_interval: 1  # retry every minute
    max_check_attempts: 3  # alert after 3 minutes
    notification_interval: 60  # 1 hour
```

## nagios_telegram_auth_key

Key used to authenticate to the Telegram API. See [how to create a
bot](https://core.telegram.org/bots#3-how-do-i-create-a-bot).

```yaml
nagios_telegram_auth_key: '***'
```

## nagios_telegram_chat_id

Unique identifier for the target chat or username of the target channel (in the
format `@channelusername`). See [API
specifications](https://core.telegram.org/bots/api#sendmessage).

```yaml
nagios_telegram_chat_id: 000000000
```

## nrpe_allowed_hosts

List of IP addresses or ranges allowed to talk to the NRPE daemon.

```yaml
nrpe_allowed_hosts:
  - 10.8.0.0/24
  - 127.0.0.1
```

## nrpe_commands

List of NRPE commands.

```yaml
nrpe_commands:
  - name: check_load
    line: /usr/lib/nagios/plugins/check_load -r -w 1,1,1 -c 4,4,4
  - name: check_openvpn
    line: '/usr/lib/nagios/plugins/check_procs -c 1: -C openvpn'
  - name: check_openvpn_cert
    line: >-
      /opt/check_ssl_cert/check_ssl_cert -f /etc/openvpn/client.crt --ignore-maximum-validity
      --ignore-incomplete-chain  --allow-empty-san --ignore-sct --warning 15 --critical 1
```

## nrpe_opts

Options for the NRPE daemon.

```yaml
nrpe_opts: '-n'  # Disable TLS
```

## openvpn_ca

Content of the certificate of the Certificate Authority (CA) used to certify
VPN connections.

```yaml
openvpn_ca: |
  -----BEGIN CERTIFICATE-----
```

## openvpn_cert

Content of the certificate used to authenticate to the VPN server.

```yaml
openvpn_cert: |
  -----BEGIN CERTIFICATE-----
```

## openvpn_key

Content of the private key used to authenticate to the VPN server.

```yaml
openvpn_key:
```

## openvpn_remote_host

Hostname or IP address of the remote VPN server.

```yaml
openvpn_remote_host: vpn.fqdn
```

## openvpn_subnet

Subnet used by OpenVPN to group clients.

```yaml
openvpn_subnet: 10.8.0.0/24
```

## openvpn_ta

Content of the OpenVPN static key used for TLS authentication.

```yaml
openvpn_ta:
```

## ovh_application_key

Application key used to authenticate to the OVH API.

```yaml
ovh_application_key: deadbeef
```

See [first steps with the OVHcloud
APIs](https://help.ovhcloud.com/csm/en-gb-api-getting-started-ovhcloud-api?id=kb_article_view&sysparm_article=KB0042784).

## ovh_application_secret

Application secret used to authenticate to the OVH API.

```yaml
ovh_application_secret: deadbeef
```

See [first steps with the OVHcloud
APIs](https://help.ovhcloud.com/csm/en-gb-api-getting-started-ovhcloud-api?id=kb_article_view&sysparm_article=KB0042784).

## ovh_consumer_key

Consumer key used to authenticate to the OVH API.

```yaml
ovh_consumer_key: deadbeef
```

See [first steps with the OVHcloud
APIs](https://help.ovhcloud.com/csm/en-gb-api-getting-started-ovhcloud-api?id=kb_article_view&sysparm_article=KB0042784).

## ovh_endpoint

Endpoint of the OVH API.

```yaml
ovh_endpoint: ovh-eu
```

See [first steps with the OVHcloud
APIs](https://help.ovhcloud.com/csm/en-gb-api-getting-started-ovhcloud-api?id=kb_article_view&sysparm_article=KB0042784).

## serial2mqtt_host

Hostname or IP address used by serial2mqtt to send messages to the MQTT broker.

```yaml
serial2mqtt_host: localhost
```

## serial2mqtt_interface

Name of the serial interface name used by serial2mqtt to gather metrics
produced by the Arduino board.

```yaml
serial2mqtt_interface: /dev/ttyACM0
```

## serial2mqtt_password

Password used by serial2mqtt to send messages to the MQTT broker.

```yaml
serial2mqtt_password: ***
```
## serial2mqtt_port

Port used by serial2mqtt to send messages to the MQTT broker.

```yaml
serial2mqtt_port: 1883
```

## serial2mqtt_topic_prefix

Add this prefix to topic names on the MQTT broker for serial2mqtt messages.

```yaml
serial2mqtt_topic_prefix: sensors
```

## serial2mqtt_username

Username used by serial2mqtt to send messages to the MQTT broker.

```yaml
serial2mqtt_username: telegraf
```

## ssh_authorized_keys

List of SSH authorized keys.

```yaml
ssh_authorized_keys:
  - user: root
    key: ssh-ed25519 hash
    comment: desktop
```

Used by
[ansible.posix.authorized_keys](https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html)
module.

## telegraf_influxdb_database

Name of the InfluxDB database used by telegraf to send metrics.

```yaml
telegraf_influxdb_database: metrics
```

## telegraf_influxdb_password

Password of the InfluxDB user used by telegraf to send metrics.

```yaml
telegraf_influxdb_password: ***
```

## telegraf_influxdb_urls

List of InfluxDB endpoints used by telegraf to send metrics.

```yaml
telegraf_influxdb_urls:
  - https://192.168.0.1:8088
```

## telegraf_influxdb_username

Name of the InfluxDB user used by telegraf to send metrics.

```yaml
telegraf_influxdb_username: telegraf
```

## telegraf_mqtt_consumer_password

Password used to authenticate to the MQTT broker for telegraf.

```yaml
telegraf_mqtt_consumer_password: ***
```

## telegraf_mqtt_consumer_servers

List of MQTT brokers for telegraf.

```yaml
telegraf_mqtt_consumer_servers:
  - tcp://localhost:1883
```

## telegraf_mqtt_consumer_topics

List of MQTT topics to consume for telegraf.

```yaml
telegraf_mqtt_consumer_topics:
  - sensors/humidity
  - sensors/temperature
```

## telegraf_mqtt_consumer_username

Name used to authenticate to the MQTT broker for telegraf.

```yaml
telegraf_mqtt_consumer_username: telegraf
```

## telegraf_ping_ip

IP address of the host to ping for latency metrics.

```yaml
telegraf_ping_ip: 192.168.0.1
```

## timezone

Alias of the time zone.

```yaml
timezone: Europe/Brussels
```

## users

List of users to configure on the remote host.

```yaml
users:
  - name: root
    password: hash
```

Used by
[ansible.builtin.user](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)
module.
