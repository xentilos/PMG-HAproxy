# PMG-HAproxy
Proxmox Mail Gateway behind HAProxy


#### HAProxy config:
```
global
	maxconn			1000
	stats socket /tmp/haproxy.socket level admin  expose-fd listeners
	uid			80
	gid			80
	nbproc			1
	nbthread			1
	hard-stop-after		15m
	chroot				/tmp/haproxy_chroot
	daemon
	server-state-file /tmp/haproxy_server_state

frontend email
	bind			1.2.3.4:25 name 1.2.3.4:25   
	mode			tcp
	log			global
	timeout client		30000
	default_backend emailPMG_ipvANY

backend emailPMG_ipvANY
	mode			tcp
	id			102
	log			global
	timeout connect		30000
	timeout server		30000
	retries			3
	server			pmgmailserver 1.2.3.4:25 id 103  send-proxy
```

#### on PMG:
```bash
mkdir /etc/pmg/templates
cp /var/lib/pmg/templates/main.cf.in /etc/pmg/templates/original_main.cf.in
echo '[% INCLUDE original_main.cf.in %]'            >  /etc/pmg/templates/main.cf.in
echo '# find me in /etc/pmg/templates/main.cf.in'   >> /etc/pmg/templates/main.cf.in
echo 'postscreen_upstream_proxy_protocol = haproxy' >> /etc/pmg/templates/main.cf.in
pmgconfig sync --restart 1
```
