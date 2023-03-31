#!/bin/bash

# Esta es una IP de CANTV que esta el listas negras y la utilizamos de prueba
#190.205.217.43

IPs="200.109.231.197
200.109.231.217
200.109.231.200
200.109.231.197
200.109.231.238
200.109.231.239
200.109.231.202
200.109.231.210"

BLACKLIST="bl.spamcop.net
zen.spamhaus.org
b.barracudacentral.org
ix.dnsbl.manitu.net
psbl.surriel.comx
dnsbl.sorbs.net"

function blacklist_check {
        IP_REVERSE=$1
        VALUE=$(for i in $BLACKLIST; do dig $IP_REVERSE.$i +short ; done | grep 127 | wc -l)

        if [ $VALUE -ge 1 ] ; then
                zabbix_sender  -z 10.133.0.54 -s srv-vccs-zabbix  -k "Lista_Negras_MX" -o 0 2&>1 /dev/null
                echo "La IP $IP_REVERSE esta en listas negras"
        else
                zabbix_sender  -z 10.133.0.54 -s srv-vccs-zabbix  -k "Lista_Negras_MX" -o 1 2&>1 /dev/null
                echo "La IP $IP_REVERSE NO esta en listas negras"
        fi
}

for i in $(echo $IPs)
do
        echo $i | awk -F"." '{print $4"."$3"."$2"."$1}'
        IP_REVERSE=$(echo $i | awk -F"." '{print $4"."$3"."$2"."$1}')
        blacklist_check $IP_REVERSE
done

exit


#IP_REVERSE=43.217.205.190
IP_REVERSE=217.231.109.200

LIST_BLACKLIST="bl.spamcop.net
zen.spamhaus.org
b.barracudacentral.org
ix.dnsbl.manitu.net
psbl.surriel.comx
dnsbl.sorbs.net"


VALUE=$(for i in $LIST_BLACKLIST; do echo "$i" ; dig $IP_REVERSE.$i +short ; done | grep 127 | wc -l)

if [ $VALUE -ge 1 ] ; then
        zabbix_sender  -z 10.133.0.54 -s srv-vccs-zabbix  -k "Lista_Negras_MX" -o 0
        echo "Estamos en listas negras"
else
        zabbix_sender  -z 10.133.0.54 -s srv-vccs-zabbix  -k "Lista_Negras_MX" -o 1
        echo "NO Estamos en listas negras"
fi

