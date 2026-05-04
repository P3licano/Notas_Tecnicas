

Vemos los servicios que corre



### Mediante ssh


***Necesitamos ser root***

ssh -R puerto_donde_queramos_llevar:IP_victima:puerto_que_queremos_redireccionar root@IP_victima



### Mediante ssh con LIGOLO

#!/bin/bash

if [ "$#" -ne 1 ]; then  
  echo "Uso: $0 <direccion_IP>"  
  exit 1  
fi

sudo ip tuntap add user kali mode tun ligolo

sudo ip link set ligolo up

ip_address="$1"  
sudo ip route add "$ip_address"/24 dev ligolo

./proxy -selfcert