# truenas-influxdb-grafana
This is how to create your own customize TrueNAS dashboard using Grafana and influxDB

## Running on

    Grafana 7.1.4
    Influxdb 1.8.2
    
    ## Configuration

## Define your custom Ports, static local IP address, influxDb username and password.
To make things easier to remmeber, we will decide on what port and static local IP address to use for this logging server. Depends on your network settings, yours might be different from this. If defaults are used, ensure it doesn't conflit with other system ports you might have. Also, if firewall is enable on the system, we have to make sure these ports are allowed to pass through.

	Software        Port Number
	Grafana         9800
	InfluxDB        9600
	
Assign static local IP 192.168.11.50 (this IP should be within the range of TrueNAS local IP address so that TrueNAS can communicate with this logging server)
For influxDb username and password, please use your own imagination to create username and password here. I leave it here for reference purpose ONLY.

	username: influx_admin
	password: admin_Password!

## Grafana Installation and Configurations
Visit https://grafana.com/grafana/download to get instructions to install Grafana. But before Grafana installation, run this first

	$ sudo apt update && sudo apt upgrade

### Choose your Configuration Options
Edit Grafana configuration file located at /etc/grafana/grafana.ini to your liking. In our case
	
	[server]
	http_addr = 192.168.11.50
	http_port = 9800

#### Install common plug-ins for grafana
	$ sudo grafana-cli plugins install grafana-worldmap-panel
	$ sudo grafana-cli plugins install savantly-heatmap-panel
	$ sudo grafana-cli plugins install grafana-piechart-panel
	$ sudo grafana-cli plugins install grafana-clock-panel
	
#### Enable service, start grafana-server, and double checking to see it's currently running.
	$ sudo systemctl daemon-reload
	$ sudo systemctl enable grafana-server
	$ sudo systemctl start grafana-server
	$ sudo systemctl --type=service --state=active | grep grafana    
    
    
### InfluxDB Installation and Configurations
Visit this wbesite https://portal.influxdata.com/downloads/ and select InfluxDB version to install. At this point, we will determine graphite engpoint is enabled in InfluxDb by editing configuration file. Also we will activate the template for TrueNAS. Configuration file is located at /etc/influxdb/influxdb.conf 
	
	[http]
	# Determines whether HTTP endpoint is enabled.
    	enabled = true

  	# The bind address used by the HTTP service.
    	bind-address = "192.168.11.50:9600"
	
	# Determines whether user authentication is enabled over HTTP/HTTPS.
    	auth-enabled = true


	[[graphite]]
  	# Determines whether the graphite endpoint is enabled.
   		enabled = true
   		database = "graphitedb"
   		retention-policy = ""
   		bind-address = ":2003"
   		protocol = "tcp"
   		consistency-level = "one"
	
	separator = "_"

	templates = [
     		"*.app env.service.resource.measurement",
     		"servers.* .host.resource.measurement*",
  		#   # Default template
  		#   "server.*",
  	 ]
			
		
#### Enable service, start influxdb, and double checking to see it's currently running.		
	$ sudo systemctl enable influxdb.service
	$ sudo systemctl start influxdb
	$ sudo systemctl --type=service --state=active | grep influxdb

Visit https://docs.influxdata.com/influxdb/v1.8/introduction/get-started/ for more info to create user, password and database to accpet TrueNAS output. While there get to know how to assign user priviledge to database. But before doing that we need to get into InfluxDB shell command by issuing this command where -host/-port are said at the beginiing

    $ influx -host 192.168.11.35 -port 9600
    
Once everything is setup, run a few queries to see if the data you are looking for is being populated.

    bash-4.4# influx
    Connected to http://localhost:8086 version 1.7.10
    InfluxDB shell version: 1.7.10
    > show databases
    name: databases
    name
    ----
    pfsense
    _internal
    > use pfsense
    Using database pfsense
    > show measurements
    name: measurements
    name
    ----
    cpu
    disk
    diskio
    dnsbl_log
    gateways
    interface
    ip_block_log
    mem
    net
    pf
    processes
    swap
    system
    temperature
    > select * from system limit 20
    name: system
    time                host                     load1         load15        load5         n_cpus n_users uptime     uptime_format
    ----                ----                     -----         ------        -----         ------ ------- ------     -------------
    1585272640000000000 pfSense.home         0.0615234375  0.07861328125 0.0791015625  4      1       196870     2 days,  6:41
    1585272650000000000 pfSense.home         0.05126953125 0.07763671875 0.076171875   4      1       196880     2 days,  6:41
    1585272660000000000 pfSense.home         0.04296875    0.07666015625 0.0732421875  4      1       196890     2 days,  6:41
    1585272670000000000 pfSense.home         0.03564453125 0.07568359375 0.0703125     4      1       196900     2 days,  6:41
    1585272680000000000 pfSense.home         0.02978515625 0.07470703125 0.0673828125  4      1       196910     2 days,  6:41
    1585272690000000000 pfSense.home         0.02490234375 0.07373046875 0.064453125   4      1       196920     2 days,  6:42
    ...

## [Original Reddit thread](https://www.reddit.com/r/PFSENSE/comments/fsss8r/additional_grafana_dashboard/ "Originial Reddit thread")

I was going to post this in the thread made by [/u/seb6596](https://www.reddit.com/u/seb6596 "/u/seb6596") since this is based on [their dashboard](https://www.reddit.com/r/PFSENSE/comments/fsf7f7/my_pfsense_monitor_dashboard_in_grafana/ "their dashboard"), but I made quite a few changes and wanted to include information that would get lost in the thread.

What I updated:

- Created dashboard wide variables to make the dashboard more portable and easily configurable. You shouldn't need to update any of the queries.
- Took some inspiration and panels [from this dashboard](https://grafana.com/grafana/dashboards/9806 "from this dashboard")
- Included gateway RTT from dpinger thanks to [this integration](https://forum.netgate.com/topic/142093/can-telegraf-package-gather-latency-packet-loss-information/3 "this integration")
- Used[ telegraf configs](https://www.reddit.com/r/pfBlockerNG/comments/bu0ms0/pfblockerngtelegrafinfluxdb_ip_block_list/ " telegraf configs") from this post by [/u/PeskyWarrior](https://www.reddit.com/u/PeskyWarrior "/u/PeskyWarrior")
- Tag, templating - No need to specify all cpus or interfaces in the graph queries. These values are pulled in with queries.
- Added chart to show all adapters, IP, MAC and Status[ from here](https://github.com/influxdata/telegraf/issues/3756#issuecomment-485606025 " from here")
- Added Temperature data based on feedback from[ /u/tko1982](https://www.reddit.com/u/tko1982 " /u/tko1982") - CPU Temp and any other ACPI device that reports temp is now collected and reported
