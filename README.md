# policyd-geoip
Postfix Policy for filtering emails/servers by GeoIP

 Install instructions for policyd-geoip2 Postfix policy to filter by country code.
 By Malac inspired by and utilising some code from policyd-spf filter program.
 
 This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License
 as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.
 This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
 without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.
 You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.

[Installation]<br>
You will need the maxminddb python module, install instructions if not in your package manager; https://pypi.org/project/maxminddb/<br>
Using the available data from https://www.maxmind.com/en/home<br>
On Debian, Ubuntu and derivatives you may use (sudo apt install python3-maxminddb) the module may be listed as just "python-maxminddb" depending on distribution or your package manager.<br>
OR :<br>
You can use the old style .dat file, available using the Python-Geoip module (sudo apt install python3-geoip). This will require some code changes to policyd-geoip2 file in /usr/bin.<br>
The changes are to the import operations at the top of the file, uncomment the GeoIP import and comment out the maxminddb import.<br>
Uncomment the "gi = GeoIP.open...." line and comment out the "reader = maxminddb.open_database....." line.<br>
Uncomment the "result, instance_dict = dat_query(data, instance_dict, path_to_data)" and comment out the "result, instance_dict = query(data, instance_dict, path_to_data)".<br>
Place the policyd-geoip2 file into /usr/bin directory.<br>
Change owner/group to root (chown root:root /usr/bin/policyd-geoip2).<br>
Give it execute permissions (chmod 755 /usr/bin/policyd-geoip2).<br>
<br>
Create Directory /etc/postfix-policyd-geoip <br>
Change owner/group to root (chown root:root /etc/postfix-policyd-geoip).<br>
Set this to execute for owner/group/others (chmod 755 /etc/postfix-policyd-geoip).<br>

Copy the following files into it:<br>
accept_list.conf<br>
accept_list_exceptions.conf<br>
COPYING (this file if you wish)<br>
policyd-geoip2.conf<br>
policyd-geoip2.sql<br>
README.md (this file if you wish)<br>
recipient_bypasses.conf<br>
reject_list.conf<br>
reject_list_exceptions.conf<br>
sender_bypasses.conf<br>

Set owner/group to root on all files. (chown root:root /etc/postfix-policyd-geoip/*).<br>
Set permissions on all files. (chmod 644 /etc/postfix-policyd-geoip/*).<br>

The file /etc/postfix-policyd-geoip/policyd-geoip2.conf contains explanations of all options, how to format and what they do.

[Database]<br>
If you wish you can set up a MySQL database to log rejected servers.<br>
The file /etc/postfix-policyd-geoip/policyd-geoip2.sql contains structure data which can be imported via phpmyadmin or mysql cli interface.<br>
You will then need to create a user to access this database from policyd-geoip2.<br>
The default name for host is: localhost<br>
The default name for database is: policyd-geoip<br>
The default name for user is: policyd-geoip (obviously don't connect as root use a dedicated user of your choice).<br>
YOU MUST create the user password yourself in phpmyadmin or mysql cli and give them access permissions to the database.<br>

Set log_reject_to_db to "true" (no quotes) in /etc/postfix-policy-geoip/policyd-geoip2.conf to log entries.

[Postfix]<br>
Add the following to the bottom of /etc/postfix/master.cf (you may add -d after /usr/bin/policyd-geoip2 to display debug messages), you can skip the # comments if you wish.<br>
\# ==========================================================================<br>
\# service type  private unpriv  chroot  wakeup  maxproc command + args      <br>
\#               (yes)   (yes)   (yes)   (never)  (100)                       <br>
\# ==========================================================================<br>
\# Added for geoip policy:                                                   <br>
policy-geoip unix    -       n       n       -       0     spawn           <br>
        user=nobody argv=/usr/bin/policyd-geoip2                           <br>

Then in /etc/postfix/main.cf add "check_policy_service unix:private/policy-geoip" (no quotes) to smtpd_recipient_restrictions setting at the place you wish in the chain of checking.<br>
Create the variable policy-geoip_time_limit and set it to 3600s (e.g. policy-geoip_time_limit = 3600s).<br>
After the process is started it deals with several messages it is left open to deal with successive messages but this is the maximum time the process should run before being closed.<br>
This saves on resources or having to start a separate process for every message. The minimum recommended time is 1000s<br>
Then postmap /etc/postfix/main.cf<br>
Reload your postfix.<br>
Messages are logged to /var/log/mail.log.<br>

[CLI options]<br>
If you run /usr/bin/policyd-geoip2 --help from the command line you will get a list of cli switches and options and what they do.<br>
None of these should EVER be used in the master.cf entry (except -d switch for debugging), they are for database maintainance and reporting only.<br>
