

Remeber to set unprivileged ports either system wide or at per process basis
```sh
sysctl -w net.ipv4.ip_unprivileged_port_start=<lowest needed port>
good practice to clear ansible cache with 'rm -r ~/.ansible/'
```
// Use capabilities module with path: /foo
    capability:        cap_net_bind_service+ep
    state: present


This repo uses Ansible to deploys Envoy Proxy as a front proxy with certbot generated TLS certificate and defult set of preconfigured services each ran by system-user with no login shell.
It deploys systemd service units and provides jinja2 template for adding new 'simple' service not requiering self compiling or installing custom libraries, eg. using only precombiled binaraies from URL archive source.
Jinja template is updated using user defined enviroment variables set in inventory file for playpabook which then populates tasks files with new services



Default set of services consists of:

- Guacamole for remote desktop access via browser
- ExCalidraw for simple drawing sharing
- Pastebin for code snippets sharing
- VSCode code-server for acces via browser

All client's communication by default supposed to be mTLS autheticated with self generated certificate and CA (?) execpt sharing services( and optioanl OAuth / OpenID connect?)
Requirements to run this playbook:

What it does? (step by step)

Setups firewall to allow only ssh, http, https for envoy host and LAN communication for every hosts
Deploy and confiure envoy proxy for defined services and obtain SSL certs
Install guacamole(dockerized), vnc server, wastebin, excalidraw and vscode-server
Install any user definder services with variables



Having own domain name, with wildcard subdomains(neccessary for certbot SSL validation and services access, currently configure for porkbun, easly changable)
Few hosts deployed inside same LAN network(either locally or in cloud)
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
- ⚠️  Generate and setup CA for mTLS authentication(use ECDSA P-256 at least)
- ✅  Create some dasboards and presserve the config across deployments of this playbook
- ⚠️  Tidy up README and make it comprehensive
- ⚠️  Describe variables for hosts setting in inventory
- ⚠️  Minor fixes in naming and redundant expressions
- ✅  Fix downstream ssl error 'error:0A000126:SSL routines::unexpected eof while reading', maybe something with libs?
- ⚠️  Seperate services for mTLS enabled and just TLS enabled
- ⚠️  Fix vscode-server issue with websocket and tls, rather related to proxy as it works flawless without TLS





