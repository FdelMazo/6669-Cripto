# Ejercicio sobre firewall

## Enunciado

1. Desde Kali escanee con nmap 0wn3d con el fin de identificar cuáles son los puertos abiertos y los servicios brindados por esos puertos. Deje el resultado en un archivo llamado 0wn3d-[fecha]-[hora].nmap. Debe subir el archivo con extensión .nmap.

2. Configurar el firewall en 0wn3d para que:

    - El tráfico entrante debe tener la política restrictiva
    - Permita que el equipo responda los pings
    - Permita el acceso desde toda la red a los servicios de HTTP y HTTPS
    - Permita el acceso al servicio de SSH únicamente desde la máquina física.
    - Para el tráfico saliente su política debe ser restrictiva
    - Permita que el equipo pueda hacer ping a  todos los host de Internet. Por ej.: google.com

3. Debe subir todas las reglas en un archivo .txt.

4. Vuelva a escanear desde Kali con nmap hacia 0wn3d con el fin de identificar cuáles son los puertos abiertos y los servicios brindados por esos puertos. Deje el resultado en un archivo llamado 0wn3d-[fecha]-[hora].nmap. Debe subir el archivo resultante.

5. Analice los resultados y compárelos con el escaneo anterior.¿Qué pasa con el puerto 443/tcp? Emita sus conclusiones

6. Todos los archivos deberán estar comprimidos en formato ZIP y deberá tener el nombre y apellido con el siguiente formato: apellido-nombre-fw.zip

---

## Resolución

1. ```nmap 192.168.1.10 > 0wn3d-`date --iso-8601=minutes`.nmap```

2. 
```
# Limpiar el estado de mi firewall
iptables --flush
iptables --zero
# El tráfico entrante debe tener la política restrictiva
iptables --policy INPUT DROP
iptables --append INPUT --match state --state ESTABLISHED,RELATED --jump ACCEPT
# Para el tráfico saliente su política debe ser restrictiva
iptables --policy OUTPUT DROP
iptables --append OUTPUT --match state --state ESTABLISHED,RELATED --jump ACCEPT
# Permita que el equipo responda los pings
iptables --append INPUT --proto icmp --jump ACCEPT
# Permita el acceso desde toda la red a los servicios de HTTP y HTTPS
iptables --append INPUT --proto tcp --dport 80 --jump ACCEPT
iptables --append INPUT --proto tcp --dport 443 --jump ACCEPT
# Permita el acceso al servicio de SSH únicamente desde la máquina física.
iptables --append INPUT --proto tcp --dport 22 --source 192.168.1.8 --jump ACCEPT
# Permita que el equipo pueda hacer ping a  todos los host de Internet.
iptables --append OUTPUT --proto udp --dport 53 --destination `grep "nameserver" /etc/resolv.conf | cut --delimiter ' ' --field 2` --jump ACCEPT
iptables --append OUTPUT --proto icmp --jump ACCEPT
```

3. `iptables-save > 0wn3d-rules.txt`

4. ```nmap 192.168.1.10 > 0wn3d-`date --iso-8601=minutes`.nmap```

5. El escaneo nuevo nos muestra el impacto de las reglas que aplicamos: la única interfaz `tcp` que queda entre `0wn3d` y el mundo exterior (`kali`) es el puerto `http` y el puerto `https`.

    - No puedo ver que el puerto `ssh` este abierto (porque solo tiene acceso mi máquina física, no `kali`).
    - No puedo ver el puerto de ping abierto, aunque lo este, porque nuestro comando de `nmap` solo esta pidiendo los servicios que pasan por `tcp`.
    - Por más que el puerto `https` sea accesible, no hay aplicación escuchando en el.
