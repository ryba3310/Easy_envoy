
# Intro

Ease_envoy is an Ansible role which enables easy deployment of front edge reverse proxy which forwad request to upstream services securing connection with at least TLS and for chosen services with mTLS.
It's easy  to deploy becouse it only requiers a single host with domain name pointing to it and open TCP port 443.

By defult it deploys these services:
```
Envoy - serving as a front edge proxy
Grafana - for metrics monitoring and log proccessing
Loki - for log aggregation
Prometheus - for metrics polling and aggregation
Promtail - for log pushing to Loki instance
Node exporter - for system metrics polling by Prometheus
Excalidraw - a virutal collaboratie white board
Wastebin - a pastebin for sharing code snippets
Vscode-server - a self-hosted in-browser remotely accesible VSCode instance
Guacamole - a web-based remote desktop gateway supporting multiple remote access protocols
Vnc server - a remote desktop server using VNC protocol allowing to access desktop remotely
One-Time-Secret - a secret sharing service to demonstrate a user defined service
```

You can check Grafna monitoring Dashboard at my own instance at grafna.justalab.xyz with login: user and password: 12345678

## How to step-by-step

Clone this repo
```sh
git clone git@github.com:ryba3310/Easy_envoy.git
```
Define host variables inside inventory.ini with your favoite text editor.
Run generate_tasks.yml to generate custom services tasks:
```sh
ansible-playbook generate_tasks.yml
```
Run the main playbook.yml
```sh
ansible-playbook playbook.yml
```
Obtain client certificate from envoy host in /root/ directory with either scp or and preferably method
```sh
scp your-host:/root/client_bundle.pfx /path/on/local/machine
```
Connect to your service uri <service_name>.<domain_name>:
```sh
grafana.example.com
```
And login to admin panel with login: admin and password: 12345678 set by default on deployment

## How it works

This role utilizes Ansible definition of variables per host in inventory file. Each host if it supposed to host a service is described by variable name corresponding. 
to the service eg. ```grafana=True```. Default services consit of just simple ```name=True``` scheme and are predefined, whereas custom user defined services
 requier a bit more complex scheme. <br />To define a custom service a host in inventory file should have at least 4 variables: <br />```service<###>=<service_name>``` representing service name,<br />
 ```service<###>_port=<service_listeing portj>``` representing service listening port,<br /> ```service<###>_uri=<uri_link```representing a uri to do download a service package,
 typically a tar archive from Github or other service,<br /> ```service<###>_path=</path/to/exec/after/unpack>``` representing path to executable after unpacking
 the archive and optionally a 5th,<br /> ```service<###>_log_path=</path/to/log/file>``` representing a path to log file of the service.<br /> A ```<###>``` characters represent a
  service number which should be uniqe to indetifie the service and its required values among others services. An example inventory.ini file is supplied in this repository
   and looks like this:
```
[cloud1]
cloud-1
[cloud2]
cloud-2
[cloud3]
cloud-3
[hosts:children]
cloud1
cloud2
cloud3
[cloud1:vars]
email_address=<email_address>
domain_name=<domain_name>
porkbun_api_key=<prokbun_api_key>
porkbun_secret=<porkpun_secret_api_key>
envoy=True
prometheus=True
guacamole=True
vnc_server=True
grafana=True
loki=True
[cloud2:vars]
excalidraw=True
wastebin=True
[cloud3:vars]
service1=ots
service1_port=3000
service1_path=/opt/ots/ots
service1_url=https://github.com/Luzifer/ots/releases/download/v1.13.0/ots_linux_amd64.tgz
vscode=True
```

This inventory consists of 3 hosts and each first grouped into its own group for ease of adding varibles to it with [<group>:vars] syntax.
Also dont forget to fillout neccessary information for Lets's Encrypt, e-mail address, domain name(without wildcard) and API keys.

## Info

Whole setup tries to maintain a good security practices but doesn't adhere to any particular standard therefore isin't suited for production envioremnt and might have security vurneabilities. I will try to outline some basic applied protections. All services run as non-root user, on each host firewall is deployed and allows only http, https and ssh for Envoy host and ssh port for all other hosts. All external communication is encrypted with TLS and for services like VSCode which gives access to its local host's shell a user authentication with mTLS is required. A proxy's certificate is valid and is acquired with Let's Encrypt whereas mTLS certificates are generated with self-signed CA. All services should run on port > 1024 except Envoy. To bind to ports < 1024 without root priviliges Envoy executable is set with capabilites or permission might be set system-wide with:
```sh
sysctl -w net.ipv4.ip_unprivileged_port_start=<lowest needed port>
```

## Quirky limitations

A Layer 5 - session layer problem with mTLS due to HTTP2 long living connection. When the connection is first established as normal TLS without mTLS, subsequent requests are therefore forward through existing HTTP2 stream which doesn't know about mTLS and doesnt grant access to protected resource. Temporal fix is to run ```sudo ss -K dst <ip> dport = 443``` to kill the TLS session and then establish mTLS session by connecting to mTLS protected resource.

Ansible proably does static analysis of whole YAML playbook looking for syntax error and after that runs the playbook therefore, can't dynamically at runtime reload its playbook, due to it before running main playbook it is neccessary to run ```generate_tasks.yml''' to generate tasks for installing user defiend services and then run the main plabook.
Also in case of some wired behavior with ansible eg. rong tasks genereation or SSH connection hanging, clear ansible might help cache with 'rm -r ~/.ansible/'

It deploys systemd service units adding new 'simple' service not requiering self compiling or installing custom libraries so only precombiled binaraies from URL archive sources are supported

Also mTLS and TLS seperation in envoy works as a kinda hack, which requries to seperate the naming scheme of hosts in uri, so being a mTLS client one should avoid using normal naming for hosts in FQDN for not mTLS protected services, rather add 'adm' to host's name in FQDN eg. admgrafana.example.com. Wastebin, ExCalidraw and Grafana are not mTLS enabled.

Currently supports only Debain based distros and Fedora distro and Envoy host MUST be Debian based becouse Envoy doesn't provide yum repository.

## Reqierements

Asnible
Having own domain name, with wildcard subdomains, neccessary for certbot SSL validation and services access and registrar API keys(currently configured for porkbun, easly changable)
At least one host with SSH access, preferably, few hosts deployed inside same LAN network(either locally or in cloud)
Open ports 443 for accessing services
Root acces to hosts thorugh sudo with provided password in .passwordfile or playbook enviroment variable

# TODO

- ✅ Some basic outline of purpose

- ✅  Finish all default services
- ✅  Try guacamole in docker instead(going with all the dependencies is crazy)
- ✅  Finish and tidy up Envoy config
- ✅  Install Fail2ban for all instances
- ✅  Distribute all tasks from monolithic file among seperate role's tasks
- ✅  Make varible defined setting in inventory file for jinja template generation
- ✅  Prepare jinja2 template for envoy config
- ❌  Employ jinja2 into playbook for new service integration
- ❌  Find out way to dynamaically generate tasks files from variables with jinja tamplate
- ✅  Generate systemd service units for each new service with j2 template
- ✅  Get default services to be generated from env vars (execpt guacamole?)
- ✅  Setup node exporteers/prometheus scrape/ loki for each node
- ✅  Setup centralized Grafana in some point of the network
- ✅  Setup firewall for each host
- ✅  Setup certbot and wildcard domain name
- ✅  Generate and setup CA for mTLS authentication(use ECDSA P-256 at least)
- ✅  Create some dasboards and presserve the config across deployments of this playbook
- ✅️  Tidy up README and make it comprehensive
- ✅️  Describe variables for hosts setting in inventory
- ⚠️   Minor fixes in naming and redundant expressions
- ✅  Fix issue with guacamole path redirection
- ✅  Fix downstream ssl error 'error:0A000126:SSL routines::unexpected eof while reading', maybe something with libs?
- ✅  Seperate services for mTLS enabled and just TLS enabled
- ✅  Fix vscode-server issue with websocket and TLS, rather related to proxy as it works flawless without TLS
- ❌  Maybe apply chroot jail for vscode-server or host it in docker instead - mTLS will do
- ❌  Make use of Filter chains in envoy instead of authrity match feature in routes - seems like to much hussle
- ✅  Build Grafana dashboards
- ⚠️   Make logs for everything and push it with promtail
- ⚠️   Moar DASHBOARDS
- ⚠️   Emply SDS in Envoy to avoid additional restrt after deployment to load generated certificate, SDS dynamically looks for new secrets
- ⚠️   Add add and use Envoy ability to scale and discover services as needed under workload
- ⚠️   Find solution to L5 problem





