# logotron

You can push it with the command cf push -m 128M -p bin/linux/ -b binary_buildpack -c './logotron' logotron

 if you want to test with larger log sizes like 100k, you can just bump up the padding size like this.
curl logotron.apps.deschutes.mini.pez.pivotal.io?pad=102400\&code=0x1F428

I did some testing with syslog today. To test this, I simply opened up rsyslog that's running on the Ops Manager VM to listen for requests on port 514. Here's the changes I made.
Uncomment these lines in /etc/rsyslog.conf.
# provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514

# provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514
Run sudo service rsyslog restart.
Then create a user provided service, cf cups my-logs -l syslog://om.deschutes.mini.pez.pivotal.io:514 and bind it to logotron.
From there, I tried running curl logotron.apps.deschutes.mini.pez.pivotal.io?pad=61439\&code=0x41 which is the control. This should work fine and it continued to for cf logs output. In /var/log/syslog, I saw an error though.
Jul 18 19:28:53 pivotal-ops-manager rsyslogd: received oversize message: size is 61541 bytes, max msg size is 8096, truncating...
To work around this, I bumped up the max message size of rsyslog to 100k. I did this by adding this line to /etc/rsyslog.conf before the network directives (above) and restarted rsyslog again.
$MaxMessagesize 100k
After doing this, I ran curl logotron.apps.deschutes.mini.pez.pivotal.io?pad=61439\&code=0x41 again and rsyslog got and logged the full message on one line. Next I ran the test where I expect it to split into two messages, curl logotron.apps.deschutes.mini.pez.pivotal.io?pad=61440\&code=0x41. That also worked as expected. I got one line with A and the second line with lots of a's.
So given the right rsyslog config, things seem to be working as expected.
