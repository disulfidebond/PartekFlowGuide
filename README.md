# PartekFlowGuide

### Guide to Installation and Maintenance of Partek Flow

An Installation PDF [can be found here](https://github.com/disulfidebond/PartekFlowGuide/blob/master/Partek_Flow_setupAndInstallation.pdf). 
Briefly, it provides instructions on how to setup and install on a single node RedHat server.

Partek Flow server provides a Graphical User Interface (GUI) for running Bioinformatics analyses.  More information can be found on the [Partek website](http://www.partek.com)

The Partek server at LSU is currently setup using a url that is only accessible on the LSU domain, or using a VPN.  Partek itself is completely self-contained within a java web server, and runs on top of a RedHat Linux installation.  Most troubleshooting for Partek can be accomplished by either restarting Partek Flow:

       # login to redhat using SSH with sudo-enabled account
       sudo service partekflowd restart

Installing updates:

       # login to redhat using SSH with sudo-enabled account
       sudo yum update partekflow
       
Or by contacting the support department for Partek.

Logs for Partek mainly are useful during startup, but are found at /opt/partek_flow/logs

The most recent log is stored in catalina.out and is usually what you want.

Despite the fact that an institution may only have a limited number of licenses, Partek Flow is configured with a POSIX framework, meaning one root account that is desicned to set up additional accounts.  Logging for accounts is minimal, beyond technical information, and generally assumes each account will use the GUI as a log.  However, the apache access_log inside the partek_flow/logs directory can be used to derive information, and potentially create alerts.  For example, the following line with a fake IP address, will indicate that someone is looking at an analysis:

>192.168.1.5 [01/Mar/2018:01:01:01 -0600] "POST /flow/ProjectThumbnail HTTP/1.1" 200 size: 20 bytes time: 2 millis

This line indicates that someone is considering deleting a project:

>192.168.1.5 [01/Mar/2018:01:01:01 -0600] "GET /flow/resources/images/delete.png HTTP/1.1" 304 size: - bytes time: 1 millis

While this GET indicates that someone deleted a project:

>192.168.1.5 [01/Mar/2018:01:01:01 -0600] "GET /flow/projects/project-deletion.xhtml?p=107&redirectToProjectManagement=true HTTP/1.1" 200 size: 12720 bytes time: 128 millis

Something similar to the following could set up an alert:

      tail -f localhost_access.log | grep --line-buffered 'project-deletion' | cut -d\  -f1 ; echo 'Warning, project deleted by above address'


### Licensing

Licensing for Partek uses a pool of available concurrent licenses, and each user account and collaborator on a project deducts from the available pool.  Either the same account, or different accounts can be used, but each additional account that is created deducts from the available pool of licenses.  

For example, if a University has 2 concurrent licenses, then this means a pool of 2 available licenses for use.  Within the Partek Flow User interface, 1 user account can use the software (2 - 1 == 1 available), and also 1 collaborator can be assigned to a project (1 - 1 == 0 available).  **However**, at this point, no one except the original user and the collaborator could login, because from the available pool of 2 licenses, 1 is deducted from being assigned to a collaborator, and 1 is "tied" to the researcher from the project.  

On the other hand, if there are 2 concurrent licenses, and no collaborators, then 2 people can use Partek Flow freely (2 - 1 == 1 for the first login, then 1 - 1 == 0 after the second login).   When a researcher logs out in this scenario, a license is freed for someone else to use.  Be aware that if someone logs in when only two licenses are available, one user that is logged in will get immediately kicked offline.

### Libraries and Data

Libraries can be setup inside the Partek Flow System settings.  This interface can also be used to reconnect the internal Partek database with an orphaned or unlinked file, such as from a network drive that was unmounted.

Partek fully supports mounted drives, and will usually immediately recognize any drives that are mounted by the RedHat OS.  There are a few important caveats:

* From experience, if a mounted network drive suddenly goes offline, this can cause Partek Flow and RedHat to completely crash; use discretion when deciding to add a network drive to fstab or manually mount it.

* Restarting Partek Flow may be required after mounting a network drive.

* Do not use a mounted network drive as output from analyses due to problems with I/O and network bandwidth.  Use the internal Partek Flow RAID instead.



