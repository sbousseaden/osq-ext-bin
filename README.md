## 1. PolyLogyx osquery Extension for Windows

PolyLogyx OSQuery Extension (plgx_win_extension.ext.exe) for Windows platform extends the core [osquery](https://osquery.io/) on Windows by adding real time event collection capabilities to osquery on Windows platform. The capabilities are built using the kernel services library of PolyLogyx. The current release of the extension is a 'community-only' release It is a step in the direction aimed at increasing osquery footprint and adoption on Windows platform. With the extension acting as a proxy into Windows kernel for osquery, the possibilities can be enormous.

# 1.1 What it does:
The extension bridges the feature gap of osquery on Windows in comparison to MacOS and Linux by adding the following into the osquery:

1) File Integrity Monitoring (FIM)
2) Process Auditing
3) MSR (Model Specific Register) details
4) Removable Media Events
5) A way to track all the PE files on the system
6) Http requests being generated from the system
7) Socket (listen, accept, connect and close) events
8) DNS request and response events
9) A way to track all the executables that get loaded in memory and their certificate
10) An embedded YARA engine to scan files with YARA rules
11) Open Handles in a Process.
12) Ability to query the current status of security products installed on the system
13) Integration with an intelligent PowerShell script to analyze PowerShell script logs
14) Ability to track Registry Events in real time
15) Query the state of the endpoint security solution (e.g. AV)
16) Sysmon style events for RemoteThread and OpenProcess

This additional state of the Windows endpoint is exported by means of following additional tables created by the PolyLogyx Extension

- win_dns_events
- win_dns_response_events 
- win_epp_table
- win_file_events   
- win_file_timestomp_events
- win_http_events 
- win_image_load_events 
- win_msr
- win_obfuscated_ps
- win_pefile_events 
- win_process_events 
- win_process_handles
- win_process_open_events 
- win_registry_events 
- win_remote_thread_events 
- win_removable_media_events 
- win_socket_events 
- win_yara_events

The detailed schema for these tables is available 

## 2 Applying Filters

By default, PolyLogyx client is designed to capture the system events in real time over a wide variety of system activities and make that telemetry available via the flexible SQL syntax of osquery. Given the most of the system activity may be benign, and can cause additional burden of skimming thru a larger volume of data while searching for incidents of interest, we provide a way of filtering the events on most tables.

# 2.1 Types of Filters

Using filters, you can configure the PolyLogyx client to capture only data relevant to you. You can choose to include relevant data and exclude non-meaningful data. In effect you can define these type of filters:
- Include filters to receive information about events matching the specified filtering criteria.
- Exclude filters to ignore information about events matching the specified filtering criteria.

Note: Exclude filters take precedence over include filters when processing the defined filters. So, if an include and exclude filter match the same event, information is not captured. In the absence of any filters, all events are captured.
These filters operate on the tables and are defined in the osquery.conf file. Use the json syntax to define filters. Here is the syntax used to define a filter.

	“table name”: {
		“column name” : {
			“filter type” : {
				“values”: [
					“value 1”,
					“value 2”
					]


In the syntax:
- **table name** - Represents the name of the table for which to define filters. You must include the table names in quotes (“”). You can apply filters only on a selected set of tables . For more information, see Supported Tables.
- **column name** - Indicates the name of the column within the table on which to filter information. You must include the column names in quotes (“”). You can define filters on selected columns in a set of tables. For more information, see Supported Tables.
- **filter type** - Specifies the filter type. Possible values are include and exclude. You must include the values in quotes (“”).
- **value 1** and **value 2** - List the values to match for the specified filter. Each entry represents a value that you want to store or ignore data for (based on the filter type). You must include the values in quotes (“”). Specified values are case insensitive. You can also use following wild cards in the values.
 
 - \*  Represents one or more characters
 - ?  Represents a single character

Here is an example of exclude filters.

	"win_process_events": {	
		"cmdline": {
			"exclude" : {
				"values": 
				[
				"C:\\Windows\\system32\\DllHost.exe /Processid*",
				"C:\\Windows\\system32\\SearchIndexer.exe /Embedding",
				"C:\\windows\\system32\\wermgr.exe -queuereporting",
				]
     			}
    		}
	}
	
Here is an example of include filters.

	"win_registry_events": {
		"target_name": {
			"include": {
				"values": 
				[
				"*CurrentVersion\\Run*",
				"*Policies\\Explorer\\Run*",
				"*Group Policy\\Scripts*",
				"*Windows\\System\\Scripts*",
				]
			   }
		 }
	}


# 2.2 Event filtering support

Event filters are supported on following tables and columns:

| Table Name | Column Names |
|------------|--------------|
|win_process_events|cmdline, path, parent_path|
|win_registry_events|target_name|
|win_socket_events|process_name, remote_port, remote_address|
|win_file_events|target_path, process_name|
|win_remote_thread_events|src_path, target_path|
|win_process_open_events|src_path, target_path|
|win_dns_events|domain_name|
|win_dns_response_events|domain_name|


# 2.3 Credit for filters

The event filters are inspired from the filters on the popular IR tool [sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon). The filtering conditions in osquery.conf file provided with the extension are derived from the high fidelity sysmon filters built by [SwiftOnSecurity](https://github.com/SwiftOnSecurity/sysmon-config) and its fork by [ion-storm](https://github.com/ion-storm/sysmon-config). Many other configurations can be created. 

## 3 FAQ

Q) What is extension version?

A) It is 1.0.22.2. It is digitally signed by PolyLogyx

Q) What osquery version to use?

A) It has been built and tested with 3.2.6.

Q) I have installed osquery using the MSI from osquery website. Now what?

A) Stop the osquery service, replace the osquery.flags and osquery.conf with the ones provided here. Feel free to edit them to bring the configurations from previous files. Restart osqueryd/osqueryi

Q) Extension is loaded by osqueryd. Can I also see the extension tables by running osqueryi?

A) Unfortunately no. There are multiple reasons for it, one of them being the communication pipe between osquery core and extension is taken by osqueryd, so osqueryi won't load the extension. 

Q) Does it depend on any kernel component?

A) It does.

Q) Do we need to install the kernel component seperately?

A) No. The extension executable is self sufficient. The kernel component is automatically installed/uninstalled with the load and unlaod of extension. There are however situations when osquery doesn't install the extension very cleanly and the drivers may reamin loaded. 

Q) Is there a cleanup utility in such a case?

A) Yes. You can use _\_cleanup.bat._ It would need to be launched from an admin console

Q) What if something )
A) You get to keep both the pieces. Isn't that great?

Q) I want to report an issue.

A) You can log it here, mail to open@polylogyx.com or find us on [osquery slack](https://osquery.slack.com/) at channel # polylogyx-extension

Q) Any known issues?

A) There is a small race between application of filters and the event collection, so for a short windows events that are supposed to be fitered get captured.
