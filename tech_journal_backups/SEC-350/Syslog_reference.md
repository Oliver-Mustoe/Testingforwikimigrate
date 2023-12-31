# Syslog reference

This page contains configurations/tips on working with the rsyslog.

**Table of contents**

1. [Configuring syslog service on logging server](#configuring-syslog-service-on-logging-server)
2. [Configuring syslog service on logging client](#configuring-syslog-service-on-logging-client)
3. [Custom log organization](#custom-log-organization)

## Configuring syslog service on logging server

First, on the logging server, install "rsyslog" from your repository. Then, enable port 514 tcp/udp. For example on a Redhat server, the following commands would be used to install, setup firewall, and check the firewall:

```bash
sudo dnf install rsyslog  
firewall-cmd --permanent --zone=public --add-port=514/tcp  
firewall-cmd --permanent --zone=public --add-port=514/udp  
firewall-cmd --reload  
firewall-cmd --list-all
```

Then the following lines must be uncommented in `/etc/rsyslog.conf`:
![image](https://user-images.githubusercontent.com/71083461/212767775-7454ce85-6b1d-4ec7-b525-7fed245863f7.png)

Afterwards, restart the rsyslog service with the following command:

```bash
sudo systemctl restart rsyslog
```

And then check if rsyslog is listening on the appropriate ports with the following command (image is desired output from a test server):

```bash
netstat -tupan | grep 514
```

![image](https://user-images.githubusercontent.com/71083461/212768235-a94d145f-2ec0-40ef-804b-935fe4082250.png)

## Configuring syslog service on logging client

First, on the logging client, install rsyslog. For example, on a Redhat based client the following would be run:

```bash
sudo dnf install rsyslog 
```

Then add the following line to a .conf file in `/etc/rsyslog.d`. For example, this tutorial will use a file called "sec350.conf". Replace `{LOG_server_IP}` with the IP of the logging server.

```bash
user.notice @{LOG_server_IP}
```

**IN THE ABOVE LINE:**

* user=syslog facility
* notice=syslog priority
* @=UDP (@@=TCP)

Example configuration (file should be edited with proper permissions!):
![image](https://user-images.githubusercontent.com/71083461/212943446-58e2143f-b867-4f8a-9c89-1c4801f127c9.png)

To test Syslog, run the command `logger -t test SOMETHING` from your logging client. This should appear in the logging server's `/var/log/messages` file. Below is a test showing the logging server "log01-oliver" receiving the test "TESTFROMWEB01TOLOG01" from "web01-oliver", a logging client:
![image](https://user-images.githubusercontent.com/71083461/212769825-b12dfe85-7b96-46f6-9c12-1355314d2d61.png)

## Custom log organization

First, on the logging server, comment out the following lines inside "/etc/rsyslog.conf":

![image](https://user-images.githubusercontent.com/71083461/214687950-57827681-960f-41fd-83ef-268275c863fe.png)  

Then create a .conf file, such as “03-sec350.conf”, in /etc/rsyslog.d **AS ROOT**. Give it the following [instructor provided content](https://raw.githubusercontent.com/gmcyber/sec350-share/main/03-sec350.conf):

```
module(load="imudp")
input(type="imudp" port="514" ruleset="RemoteDevice")
template(name="DynFile" type="string"
    string="/var/log/remote-syslog/%HOSTNAME%/%$YEAR%.%$MONTH%.%$DAY%.%PROGRAMNAME%.log"
)
ruleset(name="RemoteDevice"){
    action(type="omfile" dynaFile="DynFile")
}
```

![image](https://user-images.githubusercontent.com/71083461/214688465-af6b3636-97a5-4d9d-8ef6-efa03aa3e69e.png)  

Then, restart syslog, and from the logging client, execute the test seen in the screenshot below:

![image](https://user-images.githubusercontent.com/71083461/214688669-8fbfa865-8083-4908-af01-05c1712ce1cf.png)

Which will result in the message logged (shown in second command output) in a folder for the clients hostname (shown in first command output):

![image](https://user-images.githubusercontent.com/71083461/214689060-4af191b9-72d0-4120-b709-09409e925229.png)  

## Sources:

- https://firewalld.org/documentation/howto/open-a-port-or-service.html
- https://firewalld.org/documentation/howto/reload-firewalld.html
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-viewing_current_status_and_settings_of_firewalld
