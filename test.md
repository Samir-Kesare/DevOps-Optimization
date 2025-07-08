+ scp -P 922 /app/jenkins/pgpool_connection/pgpool_connections_script.sh esbuser@172.23.36.206:/tmp/pgpool_connections_script.sh
***********************************************************************************************************************
WARNING: Unauthorized access to this system is strictly prohibited.
All activity is monitored and logged.
***********************************************************************************************************************

  ____          _ _           _     _____       _                       _            _     _                     ___
 |  _ \ ___  __| | |__   __ _| |_  | ____|_ __ | |_ ___ _ __ _ __  _ __(_)___  ___  | |   (_)_ __  _   ___  __  / _ \
 | |_) / _ \/ _` | '_ \ / _` | __| |  _| | '_ \| __/ _ \ '__| '_ \| '__| / __|/ _ \ | |   | | '_ \| | | \ \/ / | (_) |
 |  _ <  __/ (_| | | | | (_| | |_  | |___| | | | ||  __/ |  | |_) | |  | \__ \  __/ | |___| | | | | |_| |>  <   \__, |
 |_| \_\___|\__,_|_| |_|\__,_|\__| |_____|_| |_|\__\___|_|  | .__/|_|  |_|___/\___| |_____|_|_| |_|\__,_/_/\_\    /_/
                                                            |_|

Kernel: 
PROJECT:
Application:
------------------------------------------------------------------------------------------------------------------------
+ ssh -p 922 esbuser@172.23.36.206 'chmod +x /tmp/pgpool_connections_script.sh && bash /tmp/pgpool_connections_script.sh'
***********************************************************************************************************************
WARNING: Unauthorized access to this system is strictly prohibited.
All activity is monitored and logged.
***********************************************************************************************************************

  ____          _ _           _     _____       _                       _            _     _                     ___
 |  _ \ ___  __| | |__   __ _| |_  | ____|_ __ | |_ ___ _ __ _ __  _ __(_)___  ___  | |   (_)_ __  _   ___  __  / _ \
 | |_) / _ \/ _` | '_ \ / _` | __| |  _| | '_ \| __/ _ \ '__| '_ \| '__| / __|/ _ \ | |   | | '_ \| | | \ \/ / | (_) |
 |  _ <  __/ (_| | | | | (_| | |_  | |___| | | | ||  __/ |  | |_) | |  | \__ \  __/ | |___| | | | | |_| |>  <   \__, |
 |_| \_\___|\__,_|_| |_|\__,_|\__| |_____|_| |_|\__\___|_|  | .__/|_|  |_|___/\___| |_____|_|_| |_|\__,_/_/\_\    /_/
                                                            |_|

Kernel: 
PROJECT:
Application:
------------------------------------------------------------------------------------------------------------------------
rm: cannot remove '/tmp/Default_conn.txt': Operation not permitted
Unable to use a TTY - input is not a terminal or the right kind of file
Unable to use a TTY - input is not a terminal or the right kind of file
Unable to use a TTY - input is not a terminal or the right kind of file
Unable to use a TTY - input is not a terminal or the right kind of file
Unable to use a TTY - input is not a terminal or the right kind of file
Unable to use a TTY - input is not a terminal or the right kind of file
+--------------------+---------------------+---------------------+
| Namespace          | Total Connections   | Available           |
+--------------------+---------------------+---------------------+
| default            |                 488 |                 197 |
| am-finrep          |                 108 |                  63 |
+--------------------+---------------------+---------------------+

Default Namespace Connection Details
Count#ServiceName
195#wait
51#rubik_services
37#pnpbulk_services
29#workflow_services
27#abgateway_services
22#ppo_services
15#casemgmt_services
10#churn_services
8#auc_services
7#notify_services
6#in_services
6#empapp_services
6#charging_services
6#canvas_services
5#user_services
5#provision_services
5#finrep_services
5#cms_services
5#bigben_services
4#qna_services
3#usermgmt_services
3#targetmgmt_services
3#salesprofile_services
3#epprofile_services
3#codegenerator_services
3#cockpit_services
3#air_services
2#cp_services
1#postgres
1#bulkupload_services

am-finrep Namespace Connection Details
Count#ServiceName
60#wait
39#amfinrep_services
tee: /tmp/Default_conn.txt: Permission denied
