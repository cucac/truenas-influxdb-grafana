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
	password: admin_Password

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

Visit https://docs.influxdata.com/influxdb/v1.8/introduction/get-started/ for more info to create admin user and password to accpet TrueNAS output. While there get to know how to assign user priviledge to. But before doing that we need to get into InfluxDB shell command by issuing this command where -host/-port are mentioned at the beginning

    $ influx -host 192.168.11.35 -port 9600
    
Once everything is setup with admin user, we can now tell TrueNAS to output time series data to influxDB


We can also run a few queries to see if the data you are looking for is being populated.

	$ influx -host 192.168.11.50 -port 9600 -username "influx_admin" -password "admin_Password"
	Connected to http://192.168.11.50:9600 version 1.8.2
	InfluxDB shell version: 1.8.2
	> SHOW DATABASES
	name: databases
	name
	----
	_internal
	graphitedb
	> USE graphitedb
	Using database graphitedb
	> SHOW MEASUREMENTS
	name: measurements
	name
	----
	.
	.
	da1.temperature
	.
	.

	> SELECT * FROM "da1.temperature"
	name: da1.temperature
	time                host          resource value
	----                ----          -------- -----
	1598032699000000000 truenas_local disktemp 38
	1598032999000000000 truenas_local disktemp 38
	1598033299000000000 truenas_local disktemp 38


Now you can access Grafana and customize your own dashboard. Default username and password for Grafana is admin/admin. Once you log in you can change your own password for admin user.

## Future impoving:
I am still learning thru the process how to pull time series data from TrueNAS and display it with Grafana. A lot to learn. Hope this little readme file helps you to create your own meaningfull TrueNAS dashboard. If you have questions please post on my YouTube video comment section.
