# Grafana

```yaml
version: '3.9'

services:
  grafana:
    image: grafana/grafana
    restart: always
    env_file: .grafana.env
    #volumes:
    #  - "/srv/main/grafana/lib:/var/lib/grafana"
    #  - "/srv/main/grafana/etc:/etc/grafana"
    ports:
      - "[::1]:8000:3000"
```

Unfortunately you need to copy some file out of the container before you can use grafana:
```shell
sudo mkdir -p /srv/main/grafana

sudo docker-compose up -d grafana

sudo docker cp grafana-grafana-1:/var/lib/grafana \
  /srv/main/grafana/lib

sudo docker cp grafana-grafana-1:/etc/grafana \
  /srv/main/grafana/etc

sudo chown -R 472:472 /srv/main/grafana/
```

Next you can remove the comments in front of the volumes and start up the container.  
The default credentials are `admin`:`admin`.

### Data sources and Dashboards
Finally, you can add data sources and create dashboards:

!!! info ""
    Checkout the [officially supported datasources](https://grafana.com/docs/grafana/latest/datasources/#supported-data-sources)

![Datasources](../img/services/grafana_datasources.png?raw=true){: loading=lazy }

![Dashboard](../img/services/grafana_dashboard.png?raw=true){: loading=lazy }


### Plugins
You may specify a list of plugins which you would like to install in the `.grafana.env`:
```
GF_INSTALL_PLUGINS=grafana-piechart-panel
```

### LDAP Auth
You can configure ldap auth in `/srv/main/grafana/etc/grafana.ini` and `/srv/main/grafana/etc/ldap.toml`:
```ini
#################################### Auth LDAP ##########################
[auth.ldap]
enabled = true
;config_file = /etc/grafana/ldap.toml
;allow_sign_up = true
```

```toml
[log]
filters = "ldap:debug"

[[servers]]
host = "ldap"
port = 389
use_ssl = false
start_tls = false
ssl_skip_verify = false

bind_dn = "cn=admin,dc=domain,dc=de"
bind_password = 'S3cr3T'

search_filter = "(&(objectclass=person)(&(memberof=cn=grafana,ou=groups,dc=domain,dc=de))(uid=%s))"
search_base_dns = ["dc=domain,dc=de"]

[servers.attributes]
name = "givenName"
surname = "sn"
username = "cn"
member_of = "memberOf"
email =  "email"

[[servers.group_mappings]]
group_dn = "cn=admin,cn=grafana,ou=groups,dc=domain,dc=de"
org_role = "Admin"

[[servers.group_mappings]]
group_dn = "cn=editor,cn=grafana,ou=groups,dc=domain,dc=de"
org_role = "Editor"

[[servers.group_mappings]]
group_dn = "*"
org_role = "Viewer"
```

All members of `cn=grafana,ou=groups,dc=domain,dc=de` get the Viewer role, members that are also in `cn=editor,cn=grafana,ou=groups,dc=domain,dc=de` get the Editor role...

## OpenID / KeyCloak
Checkout [janikvonrotz.ch/2020/08/27/grafana-oauth-with-keycloak-and-how-to-validate-a-jwt-token](https://janikvonrotz.ch/2020/08/27/grafana-oauth-with-keycloak-and-how-to-validate-a-jwt-token/)
