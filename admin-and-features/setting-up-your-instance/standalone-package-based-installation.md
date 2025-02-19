# Standalone package based installation

## Introduction

This guide will walk you through the process of installing and running Debian packages (.deb) for **core, gateway, proxy** services on one server - as a **simple example**.

{% hint style="warning" %}
For production deployment we would recommend to divide services to multiple servers, eg.:

* proxy (used for remote enrollment, onboarding and configuring desktop clients) should be on a DMZ node that is exposed in the Internet
* gateway should be on your firewall/router
* core (the main control plain panel) - should be in internal network (intranet) and available only by intranet or VPN itself.
{% endhint %}

We will cover system requirements, additional dependencies, installation steps, and examples of configuration files and step by step running all services. In this example we will use nginx for a web server (proxy) exposing and securing web based services.

Examples will be made by using [**Debian 12**](https://www.debian.org/releases/stable/releasenotes) **and Ubuntu based system.**

{% hint style="info" %}
We also provide **RPM packages** - the procedure is similar to the one for installing DEB packages. If you need help installing RPM packages[ this guide offers help.](https://phoenixnap.com/kb/how-to-install-rpm-file-centos-linux)
{% endhint %}

### Hardware Requirements

All defguard components are **very low resource-consuming**. All of them are written in [Rust](https://www.rust-lang.org) and are single binaries. As minimum setup as follows should be more then enough:

| Resource     | Minimum requirements         |
| ------------ | ---------------------------- |
| CPU          | 1 GHz                        |
| RAM          | 2 GB (mostly for PostgreSQL) |
| Disk         | 2 GB                         |
| Architecture | x86\_64, ARM64               |

### System Requirements

Before proceeding with the installation, ensure your system meets the following requirements:

* Debian-based operating system (Debian, Ubuntu, etc.).
* Administrative (sudo) privileges.
* A server with a public IP (and you know what that IP address is and to which interface it's assigned) - in this example we use: 185.33.37.51.
* You have a domain name and know how to assign IP and manage subdomains, in our example: defguard main url will be _my-server.defguard.net_ (and the subdomain is pointed to 185.33.37.51).
* defguard [enrollment service](https://defguard.gitbook.io/defguard/help/enrollment) (run by proxy) that will enable [remote onboarding, enrollment](https://defguard.gitbook.io/defguard/help/enrollment) and  [easy configuration for our Desktop Clients (by adding defguard instances)](https://defguard.gitbook.io/defguard/help/configuring-vpn/add-new-instance)  with instance URL and one simple token - in this tutorial we use: _enroll.defguard.net_ (this subdomain also points to 185.33.37.51).
* If you have a **firewall**, we assume you have **open port 443** in order to expose both defguard and enrollment service, but also to automatically issue for these domains SSL Certificates. Port 444 (used for internal GRPC communication) **should not be exposed public.**

### Prequesities

#### PostgreSQL

Defguard core uses PostgreSQL database, so if you do not have installed and configured yet, you can do it in this section. For this tutorial we need to create **a user with superuser privileges and database**.

First of all, install postgresql:

```
apt install postgresql 
```

Now you can launch a default user and create a new superuser for your database. We create user, password and database with name `defguard`, beacuse this is by default in `/etc/defguard/core.conf`, you can change whatever you want.

```
# su -c /usr/bin/psql postgres
postgres=# CREATE USER defgaurd WITH SUPERUSER PASSWORD 'defguad';
postgres=# CREATE DATABASE defguard;
```

After creating a user and database we can connect our new user to this database. To make it easier to connect now and then, we could try to add auth file

```
# echo 'localhost:5432:defguard:defguard:defguard' >> ~/.pgpass
# chmod 600 ~/.pgpass
# psql -d defguard -h localhost -U defguard
defguard=# exit
```

- we created `.pgpass` file that consist of `<hostname>:<port>:<database>:<user>:<password>`
- we connected into the `defguard` database to verify `defguard` user can communicate with the database

#### NGINX

To expose our services in the server we need to configure a reverse proxy server. For this we will use nginx web server with ssl certificates for enabling https protocol.

To get started, we need to install:

```
apt install nginx certbot
```

Enable nginx service

```
systemctl enable nginx.service
systemctl start nginx.service
```

Disable all default domains:

```
unlink /etc/nginx/sites-enabled/default
```

## Installing packages

### Core service

Navigate to [core repository release](https://github.com/DefGuard/defguard/releases) and choose version of core package that you want to obtain that has debian package and then swap `<version>` in the following command:

```
wget https://github.com/DefGuard/defguard/releases/download/<version>/defguard-<version>-x86_64-unknown-linux-gnu.deb
```

Example:

```
wget https://github.com/DefGuard/defguard/releases/download/v0.11.0/defguard-0.11.0-x86_64-unknown-linux-gnu.deb
```

You can also download directly from the Github realse page, but please note that you should know the path where this could be storead after downloading. Once the package is downloaded, install it using dpkg:

```
dpkg -i <path_to_package>/defguard-<version>-x86_64-unknown-linux-gnu.deb
```

Example:

```
dpkg -i defguard-0.11.0-x86_64-unknown-linux-gnu.deb
```

You can check is core installed properly:

```
# defguard -V
defguard 0.11.0
```

### Gateway service

Navigate to [gateway repository release](https://github.com/DefGuard/gateway/releases) and choose version of core package that you want to obtain that has debian package and then swap `<version>` in the following command:

```
# wget https://github.com/DefGuard/gateway/releases/download/<version>/defguard-gateway_<version>_x86_64-unknown-linux-gnu.deb
```

Example:

```
# wget https://github.com/DefGuard/gateway/releases/download/v0.7.0/defguard-gateway_0.7.0_x86_64-unknown-linux-gnu.deb
```

You can also download directly from the Github realse page, but please note that you should know the path where this could be storead after downloading. Once the package is downloaded, install it using dpkg:

```
dpkg -i <path_to_package>/defguard-gateway_<version>_x86_64-unknown-linux-gnu.deb
```

Example:

```
dpkg -i defguard-gateway_0.7.0_x86_64-unknown-linux-gnu.deb
```

You can check is core installed properly:

```
# defguard-gateway -V
defguard-gateway 0.7.0
```

### Proxy service

Navigate to [proxy repository release](https://github.com/DefGuard/proxy/releases) and choose version of core package that you want to obtain that has debian package and then swap `<version>` in the following command:

```
wget https://github.com/DefGuard/proxy/releases/download/<version>>/defguard-proxy-<version>-x86_64-unknown-linux-gnu.deb
```

Example:

```
wget https://github.com/DefGuard/proxy/releases/download/v0.5.0/defguard-proxy-0.5.0-x86_64-unknown-linux-gnu.deb
```

You can also download directly from the Github realse page, but please note that you should know the path where this could be storead after downloading. Once the package is downloaded, install it using dpkg:

```
dpkg -i <path_to_package>/defguard-proxy-<version>-x86_64-unknown-linux-gnu.deb
```

Example:

```
dpkg -i defguard-proxy-0.5.0-x86_64-unknown-linux-gnu.deb
```

You can check is core installed properly:

```
# defguard-proxy -V
defguard-proxy 0.5.0
```

## Running defguard

### Generating SSL Certificates

Before we run defguard and configure the reverse proxy, first let's prepare SSL certificates that will be used by the NGINX service. We will generate a certificate for two domains we use in this example: _my-service.defguard.net_ and _enroll.defguard.net_:

```
certbot certonly --non-interactive --agree-tos --standalone --email admin@teonite.com -d my-server.defguard.net -d enroll.defgurd.net
```

Certbot will generate certificate in fullchain.pem and privkey.pem in path:

&#x20;`/etc/letsencrypt/live/my-server.defguard.net`

&#x20;`/etc/letsencrypt/live/enrolldefguard.net`

### Core - the control plain

To run core service we need to configure `/etc/defguard/core.conf`.&#x20;

{% hint style="info" %}
To generate any secret (which **we recommend to be 64 chars)**, use the following command:

`openssl rand -base64 55 | tr -d "=+/" | tr -d '\n' | cut -c1-64`
{% endhint %}

As previously mentioned, in this tutorial we wil use server domain `my-server.defguard.net`.

Example `/etc/defguard/core.conf`:

```
### Core configuration ###

#
# Generate secrets
# 
DEFGUARD_AUTH_SECRET=defguard-auth-secret
DEFGUARD_GATEWAY_SECRET=defguard-gateway-secret
DEFGUARD_YUBIBRIDGE_SECRET=defguard-yubibridge-secret
DEFGUARD_SECRET_KEY=9oZqdHRCN0TWIyMhjYOAYwgzVz9IfOqz62PzUvjvyMzqLICGSM3b0pRMdDH300CQ

# Define the URL under which defguard is running:
DEFGUARD_URL=https://my-server.defguard.net

# How long auth session lives in seconds
DEFGUARD_AUTH_SESSION_LIFETIME=604800

# Optional. Generated based on DEFGUARD_URL if not provided.
# DEFGUARD_WEBAUTHN_RP_ID=localhost

DEFGUARD_ADMIN_GROUPNAME=admin
DEFGUARD_DEFAULT_ADMIN_PASSWORD=pass123

# This will be displayed in the network settings when editing/adding a new location:
DEFGUARD_GRPC_URL=https://my-server.defguard.net:444

### Proxy configuration ###
# Proxy is optional - if you would like to use the remote enrollment
# and onboarding service, as well as easy desktop client configuration
# proxy must be enabled.
# For now we leave it uncofigured, will configure it in next step.
# DEFGUARD_PROXY_URL=http://localhost:50051

### LDAP configuration ###
# DEFGUARD_LDAP_URL=ldap://localhost:389
# DEFGUARD_LDAP_SERVICE_PASSWORD=adminpassword
# DEFGUARD_LDAP_USER_SEARCH_BASE="ou=users,dc=example,dc=org"
# DEFGUARD_LDAP_GROUP_SEARCH_BASE="ou=groups,dc=example,dc=org"
# DEFGUARD_LDAP_DEVICE_SEARCH_BASE="ou=devices,dc=example,dc=org"

### DB configuration ###
DEFGUARD_DB_HOST="localhost"
DEFGUARD_DB_PORT=5432
DEFGUARD_DB_NAME="defguard"
DEFGUARD_DB_USER="defguard"
DEFGUARD_DB_PASSWORD="defguard"
# for SQLX CLI
DATABASE_URL="postgresql://defguard:defguard@localhost/defguard"
```

**If you have configured your postgres with different names than in** [**PostgreSQL guide**](standalone-package-based-installation.md#postgresql)**, you can change it in DB configuration part. LDAP configuration is not part of this tutorial, you can also commented those lines.**

**We will back to this configuration to connect defguard core with proxy in the** [**Run proxy**](standalone-package-based-installation.md#run-proxy) **section. For now `DEFGUARD_PROXY_URL` is commented.**

After changes, you can simply enable and start your defguard core service:

```
systemctl enable defguard.service
systemctl start defguard.service
```

To see logs, type journalctl command:

```
# journalctl -u defguard.service | tail -n 50
Jul 29 13:57:15 defguard-testing systemd[1]: Started defguard.service - defguard core service.
Jul 29 13:57:15 defguard-testing defguard[2776504]: 2024-07-29T11:57:15.738420Z  INFO defguard: Starting defguard
Jul 29 13:57:15 defguard-testing defguard[2776504]: 2024-07-29T11:57:15.743079Z  INFO defguard::db: Initializing DB pool
Jul 29 13:57:16 defguard-testing defguard[2776504]: 2024-07-29T11:57:16.297407Z  INFO defguard: Using HMAC OpenID signing key
Jul 29 13:57:19 defguard-testing defguard[2776504]: 2024-07-29T11:57:19.156559Z  INFO defguard::db::models::user: Initializing admin user
Jul 29 13:57:19 defguard-testing defguard[2776504]: 2024-07-29T11:57:19.595218Z  INFO defguard::db::models::user: New admin user has been created, adding to Admin group...
Jul 29 13:57:19 defguard-testing defguard[2776504]: 2024-07-29T11:57:19.747717Z  INFO defguard::db::models::settings: Initializing default settings
Jul 29 13:57:19 defguard-testing defguard[2776504]: 2024-07-29T11:57:19.780563Z  INFO defguard: Started web services
```

#### Configuring NGINX reverse proxy with SSL

Now, we are able to create our first nginx config for defguard core service with _my-server.defguard.net_.

Create config file `/etc/nginx/site-available/my-server.defguard.net.conf`, example config file for _my-server.defguard.ent_ should look like this:

```
upstream defguard {
	server 127.0.0.1:8000;
}

upstream defguard-grpc {
	server 127.0.0.1:50055;
}

server {
	listen 443 ssl http2;
	server_name my-server.defguard.net;
	access_log /var/log/nginx/defguard.log;
	error_log /var/log/nginx/defguard.e.log;

	ssl_certificate /etc/letsencrypt/live/my-server.defguard.net/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/my-server.defguard.net/privkey.pem;
	ssl_trusted_certificate /etc/letsencrypt/live/my-server.defguard.net/fullchain.pem;

	client_max_body_size 128M;

	location / {
		proxy_pass		http://defguard;
		proxy_set_header	Host		$host;
		proxy_set_header	X-Real-IP	$remote_addr;
		proxy_set_header	X-Forwarded-For	$proxy_add_x_forwarded_for;
		proxy_http_version	1.1;
		proxy_set_header	Upgrade		$http_upgrade;
		proxy_set_header	Connection	"upgrade";
	}
}

server {
	listen 444 ssl http2;
	server_name my-server.defguard.net;
	access_log /var/log/nginx/defguard-grpc.log;
	error_log /var/log/nginx/defguard-grpc.e.log;

	ssl_certificate /etc/letsencrypt/live/my-server.defguard.net/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/my-server.defguard.net/privkey.pem;

	client_max_body_size 200m;

	location / {
		grpc_pass grpc://defguard-grpc;
	}
}
```

Link it to `/etc/nginx/site-available/`

```
ln -s /etc/nginx/sites-available/my-server.defguard.net.conf /etc/nginx/sites-enabled/my-server.defguard.net.conf
```

Restart nginx.service to activated changes:

```
systemctl reload nginx.service
```

Test your domain on another terminal tab

```
$ curl https://my-server.defguard.net/api/v1/health
alive
```

Success! We can move on to the next service.

{% hint style="danger" %}
If you will use this simple setup and run all services on one server, you can use [NGINX access restrictions](https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-proxied-tcp/) for securing core and allowing to access the _my-server.defguard.net_ only to selected networks - blocking the direct access from the Internet.
{% endhint %}

### Gateway - the WireGuard VPN service

To run gateway, we should do two things:

* setup our first location on https://my-server.defguard.net page to get `token` and `grpc_url` for gateway service,
* configure `/etc/defguard/gateway.toml`.

#### Setup location for gateway

Now, after setting up core service you should go to the website that you set on `DEFGUARD_URL`. The link should redirect you to login page. To log in type this credentials from `/etc/defguard/core.conf`

* login: admin
* password: `DEFGUARD_DEFAULT_ADMIN_PASSWORD` (by default: pass123)

Now we can configure our first location. Depends on what is more convenient fo you, choose configuration from Wireguard file or do it manualy.

<figure><img src="../../.gitbook/assets/choose_location_setup.png" alt=""><figcaption><p>Location wizard</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/location_configuration.png" alt=""><figcaption><p>Location configuration</p></figcaption></figure>

After saving configuration for location you should be redirect to Location overview page, where at the top right corner is `Edit Locations Settings` button, click on it.

<figure><img src="../../.gitbook/assets/edit_locations_settings.png" alt=""><figcaption><p>Manual configuration</p></figcaption></figure>

In `Gateway server setup` copy two variables: `DEFGUARD_TOKEN` and `DEFGUARD_GRPC_URL`

<figure><img src="../../.gitbook/assets/gateway_server_setup.png" alt=""><figcaption><p>Gateway server setup</p></figcaption></figure>

#### Create config file

After getting `DEFGUARD_TOKEN` and `DEFGUARD_GRPC_URL` variables, we can configure our gateway service. Create config.toml file and swap `<your_gateway_token>` and `<defguard_grpc_url>` with your values that you copied.

Template for configure gateway service looks like below:

```
# This is an example config file for defguard VPN gateway
# To use it fill in actual values for your deployment below

# Required: secret token generated by defguard
# NOTE: must replace default with actual value
token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJEZWZHdWFyZCIsInN1YiI6IkRFRkdVQVJELU5FVFdPUkstMSIsImNsaWVudF9pZCI6IjEiLCJleHAiOjYwMTczODM0MjQsIm5iZiI6MTcyMjQxNjEyOX0.HP-9ArvdXuyeBxRdQ6S_wJb3rBTq73J0sVyfwuPM-vY"
# Required: defguard server gRPC endpoint URL
# NOTE: must replace default with actual value
grpc_url = "https://my-server.defguard.net:444/"
# Optional: gateway name which will be displayed in defguard web UI
name = "Gateway A"
# Required: use userspace WireGuard implementation (e.g. wireguard-go)
userspace = false
# Optional: path to TLS cert file
# grpc_ca = cert.pem
# Required: how often should interface stat updates be sent to defguard server (in seconds)
stats_period = 60
# Required: name of WireGuard interface
ifname = "wg0"
# Optional: write PID to this file
# pidfile = defguard-gateway.pid
# Required: enable logging to syslog
use_syslog = false
# Required: which syslog facility to use
syslog_facility = "LOG_USER"
# Required: which socket to use for logging
syslog_socket = "/var/run/log"

# Optional: Command which will be run before bringing interface up
# Example: Allow all traffic through WireGuard interface:
#pre_up = "/path/to/iptables -A INPUT -i wg0 -j ACCEPT
# example with multiple commands - add them to a shell script
#pre_up = "/path/to/shell /path/to/script"

# Optional: Command which will be run after bringing interface up
# Example: Add a default route after WireGuard interface is up:
#post_up = "/path/to/ip route add default via 192.168.1.1 dev wg0"


# Optional: Command which will be run before bringing interface down
# Example: Remove WireGuard-related firewall rules before interface is taken down:
#pre_down = "/path/to/iptables -D INPUT -i wg0 -j ACCEPT"

# Optional: Command which will be run after bringing interface down
# Example: Remove the default route after WireGuard interface is down:
#post_down = "/pat/to/ip route del default via 192.168.1.1 dev wg0"

# A HTTP port that will expose the REST HTTP gateway health status
# STATUS CODES:
# 200 - Gateway is working and is connected to CORE
# 503 - gateway works but is not connected to CORE
#health_port = 55003
```

Now we can run gateway service with configuration above:

```
# systemctl enable defguard-gateway.service
# systemctl start defguard-gateway.service
# journalctl -u defguard-gateway.service | tail -n 50
[2024-07-27T16:37:56Z INFO  defguard_gateway::gateway] Starting defguard gateway version 0.7.0 with configuration: Config { token: "***", name: Some("Gateway on server X"), grpc_url: "https://my-server.defguard.net:444/", userspace: false, grpc_ca: None, stats_period: 60, ifname: "wg0", pidfile: None, use_syslog: false, syslog_facility: "LOG_USER", syslog_socket: "/var/run/log", config_path: None, pre_up: None, post_up: None, pre_down: None, post_down: None, health_port: None }
[2024-07-27T16:37:56Z INFO  defguard_gateway::gateway] gRPC server connection setup done.
[2024-07-27T16:37:56Z INFO  defguard_wireguard_rs::wgapi_linux] Creating interface wg0
[2024-07-27T16:37:56Z INFO  defguard_wireguard_rs::wgapi_linux] Configuring interface wg0 with config: InterfaceConfiguration { name: "Szczecin", address: "10.22.33.1/24", port: 50051, peers: [], mtu: None, .. }
[2024-07-27T16:37:56Z WARN  netlink_packet_route::link::buffer_tool] Specified IFLA_INET6_STATS NLA attribute holds more(most likely new kernel) data which is unknown to netlink-packet-route crate, expecting 288, got 296
[2024-07-27T16:37:56Z WARN  netlink_packet_route::link::buffer_tool] Specified IFLA_INET6_STATS NLA attribute holds more(most likely new kernel) data which is unknown to netlink-packet-route crate, expecting 288, got 296
[2024-07-27T16:37:56Z INFO  defguard_gateway::gateway] Reconfigured WireGuard interface Szczecin (address: 10.0.0.1/24)
[2024-07-27T16:37:56Z INFO  defguard_gateway::gateway] Stats thread spawned.
[2024-07-27T16:37:56Z INFO  defguard_gateway::gateway] Connected to defguard gRPC endpoint: https://my-server.defguard.net:444/
```

On the other side, core service should print those informations:

```
2024-07-27T16:37:56.379227Z  INFO defguard::grpc: Adding gateway user with to gateway map for network 1
2024-07-27T16:37:56.385951Z  INFO defguard::grpc::gateway: Configuration sent to gateway client, network [ID 1] Szczecin.
2024-07-27T16:37:56.388651Z  INFO defguard::grpc::gateway: New client connected to updates stream: user, network [ID 1] Szczecin
2024-07-27T16:37:56.388695Z  INFO defguard::grpc: Gateway user connected in network 1
2024-07-27T16:37:56.388810Z  INFO defguard::grpc::gateway: Starting update stream to gateway: user, network [ID 1] Szczecin
```

### Proxy - enrollment, onboardin and desktop configuration service

To run proxy service (for [remote onboarding & enrollment](../../help/enrollment.md)), we can do it by:

```
# systemctl enable defguard-proxy.service
# systemctl start defguard-proxy.service
# journalctl -u defguard-proxy.service | tail -n 50
2024-07-27T16:53:58.584154Z INFO defguard_proxy::tracing: Tracing initialized
2024-07-27T16:53:58.584233Z INFO defguard_proxy::http: Starting Defguard proxy server
2024-07-27T16:53:58.584371Z INFO defguard_proxy::http: Skipping rate limiter setup
2024-07-27T16:53:58.584438Z INFO defguard_proxy::http: gRPC server is listening on 0.0.0.0:50051
2024-07-27T16:53:58.585125Z INFO defguard_proxy::http: Defguard proxy server initialization complete
2024-07-27T16:53:58.585262Z INFO defguard_proxy::http: API web server is listening on 0.0.0.0:8080
```

#### Configuring NGiNX reverse proxy for enrollment

{% hint style="info" %}
Please note that [we already have issued the enrollemnt domain SSL certificate](standalone-package-based-installation.md#generating-ssl-certificates).
{% endhint %}

Create config file `/etc/nginx/sites-available/enroll.defguard.net.conf`, example config file for _enroll.defguard.net_ should look like this:

```
upstream defguard-proxy {
	server 127.0.0.1:8080;
}

upstream proxy-grpc {
	server 127.0.0.1:50051;
}

server {
	listen 443 ssl http2;
	server_name enroll.defguard.net;
	access_log /var/log/nginx/enroll.log;
	error_log /var/log/nginx/enroll.e.log;

	ssl_certificate /etc/letsencrypt/live/my-server.defguard.net/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/my-server.defguard.net/privkey.pem;

	client_max_body_size 200m;

	location / {
		proxy_pass 		http://defguard-proxy;
		proxy_set_header	Host		$host;
		proxy_set_header	X-Real-IP	$remote_addr;
		proxy_set_header	X-Forwarded-For	$proxy_add_x_forwarded_for;
	}
}

server {
	listen 444 ssl http2;
	server_name enroll.defguard.net;
	access_log /var/log/nginx/enroll.log;
	error_log /var/log/nginx/enroll.e.log;

	ssl_certificate /etc/letsencrypt/live/my-server.defguard.net/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/my-server.defguard.net/privkey.pem;

	client_max_body_size 200m;

	location / {
		grpc_pass grpc://proxy-grpc;
		grpc_socket_keepalive on;
		grpc_read_timeout 3000s;
		grpc_send_timeout 3000s;
		grpc_next_upstream_timeout 0;

		proxy_request_buffering off;
		proxy_buffering off;
		proxy_connect_timeout 3000s;
		proxy_send_timeout 3000s;
		proxy_read_timeout 3000s;
		proxy_socket_keepalive on;

		keepalive_timeout 90s;
		send_timeout 90s;

		client_body_timeout 3000s;
	}
}
```

Enable configuration and restart nginx:

```
ln -s /etc/nginx/sites-available/enroll.defguard.conf /etc/nginx/sites-enabled/enroll.defguard.conf
systemctl restart nginx.service
```

#### Enabling Proxy service in the Core

Now, we can update our **core configuration** in `/etc/defguard/core.conf` by uncommenting `DEFGUARD_PROXY_URL`

```
# Proxy connection configuration
DEFGUARD_PROXY_URL=https://enroll.defguard.net:444
```

Full `/etc/defguard/core.conf`:

```
### Core configuration ###

#
# Generate secrets
# 
DEFGUARD_AUTH_SECRET=defguard-auth-secret
DEFGUARD_GATEWAY_SECRET=defguard-gateway-secret
DEFGUARD_YUBIBRIDGE_SECRET=defguard-yubibridge-secret
DEFGUARD_SECRET_KEY=9oZqdHRCN0TWIyMhjYOAYwgzVz9IfOqz62PzUvjvyMzqLICGSM3b0pRMdDH300CQ

# Define the URL under which defguard is running:
DEFGUARD_URL=https://my-server.defguard.net

# How long auth session lives in seconds
DEFGUARD_AUTH_SESSION_LIFETIME=604800

# Optional. Generated based on DEFGUARD_URL if not provided.
# DEFGUARD_WEBAUTHN_RP_ID=localhost

DEFGUARD_ADMIN_GROUPNAME=admin
DEFGUARD_DEFAULT_ADMIN_PASSWORD=pass123

# This will be displayed in the network settings when editing/adding a new location:
DEFGUARD_GRPC_URL=https://my-server.defguard.net:444

### Proxy configuration ###
# Proxy is optional - if you would like to use the remote enrollment
# and onboarding service, as well as easy desktop client configuration
# proxy must be enabled.

# PROXY configuration:
DEFGUARD_GRPC_ULR=https://enroll.defguard.net:444 # add this line to your config file

### LDAP configuration ###
# DEFGUARD_LDAP_URL=ldap://localhost:389
# DEFGUARD_LDAP_SERVICE_PASSWORD=adminpassword
# DEFGUARD_LDAP_USER_SEARCH_BASE="ou=users,dc=example,dc=org"
# DEFGUARD_LDAP_GROUP_SEARCH_BASE="ou=groups,dc=example,dc=org"
# DEFGUARD_LDAP_DEVICE_SEARCH_BASE="ou=devices,dc=example,dc=org"

### DB configuration ###
DEFGUARD_DB_HOST="localhost"
DEFGUARD_DB_PORT=5432
DEFGUARD_DB_NAME="defguard"
DEFGUARD_DB_USER="defguard"
DEFGUARD_DB_PASSWORD="defguard"
# for SQLX CLI
DATABASE_URL="postgresql://defguard:defguard@localhost/defguard"
```

Reload changes in `/etc/defguarc/core.conf`

```
systemctl restart defguard.service
```

{% hint style="success" %}
Now you have full working defguard services 🥳
{% endhint %}

You can [configure your desktop client using the enrollment](../../help/configuring-vpn/add-new-instance/) service and use your VPN.

If you would like to use the feature in the desktop client to route **All traffic** through the VPN please configure your firewall to enable Internet access through your VPN - [here you can find exaples how to do it](https://defguard.gitbook.io/defguard/tutorials/step-by-step-setting-up-a-vpn-server#enabling-to-access-internet-through-your-vpn).
