## [Blackstring](http://www.maravento.com/p/blackstring.html)

**Blackstring** is an experimental project, aimed at blocking connections of [circumvention](https://en.wikipedia.org/wiki/Internet_censorship_circumvention) type applications, also called 'Anonymous Proxy' or 'Anonymizer' (such as Ultrasurf, Psiphon3, TOR, Hotspot shield, Freegate, Your-Freedom, Gtunnel, etc.), which use a combination of secure communications with VPN obfuscation technologies, SSH, and HTTP Proxy and make retransmission and reassembly of their connections, detection and blocking being very difficult. To achieve this, we use the Wireshark and tcpdump tools, which allow the capture and analysis of the data flow, both arrival and departure, to extract the strings in hexadecimal format linked to these applications and block them with the [Iptables](http://www.netfilter.org/documentation/HOWTO/es/packet-filtering-HOWTO-7.html) firewall

**Blackstring** es un proyecto experimental, orientado a bloquear las conexiones de los aplicativos tipo [circumvention](https://en.wikipedia.org/wiki/Internet_censorship_circumvention), también llamados 'Anonymous Proxy' o 'Anonimizador' (tales como Ultrasurf, Psiphon3, TOR, Hotspot shield, Freegate, Your-Freedom, Gtunnel, etc.), los cuales utilizan una combinación de comunicaciones seguras con tecnologías de ofuscación VPN, SSH, y HTTP Proxy y hacen retransmisión y re-ensamblado de sus conexiones, siendo muy difícil su detección y bloqueo. Para lograrlo utilizamos las herramientas Wireshark y tcpdump, que permiten la captura y análisis del flujo de datos, tanto de llegada como de salida, para extraer las cadenas (strings) en formato hexadecimal vinculadas a estas aplicaciones y bloquearlas con el firewall [Iptables](http://www.netfilter.org/documentation/HOWTO/es/packet-filtering-HOWTO-7.html)

### DEPENDENCIES
---

```
iptables ulogd2
```

### ⚠️ WARNING: BEFORE YOU CONTINUE!
---

- This project contains hexadecimal chains that are not exclusive to anonymizers, so they can eventually generate false positives (FP). Additionally [Iptables](http://www.netfilter.org/documentation/HOWTO/es/packet-filtering-HOWTO-7.html) rule can slow down your system. Keep in mind that string matching is intensive, unreliable and is the last resort, since to block anonymizers there are other more effective solutions, such as VPNs, Web Filters, domain-wide security policies, Firewalls, Proxy ([Squid](http://www.squid-cache.org/) non-transparent proxy mode with [Blackweb](https://github.com/maravento/blackweb) and [Blackip](https://github.com/maravento/blackip) projects [advanced rules](https://github.com/maravento/blackip#squid-cache-advanced-rules)), etc. Use it at your own risk / Este proyecto contiene cadenas hexadecimales que no son exclusivas de los anonimizadores, por tanto eventualmente pueden generar falsos positivos (FP). Adicionalmente la regla [Iptables](http://www.netfilter.org/documentation/HOWTO/es/packet-filtering-HOWTO-7.html) puede ralentizar su sistema. Tenga en cuenta que la coincidencia de cadenas es intensiva, poco confiable y es el último recurso, ya que para bloquear los anonimizadores existen otras soluciones más efectivas, como VPNs, Web Filters, Firewalls, domain-wide security policies, Proxy ([Squid](http://www.squid-cache.org/) modo proxy no-transparente con los proyectos [Blackweb](https://github.com/maravento/blackweb) y [Blackip](https://github.com/maravento/blackip) [reglas avanzadas](https://github.com/maravento/blackip#squid-cache-advanced-rules)), etc. Úselo bajo su propio riesgo
- At the moment, only hex-strings are included for [Ultrasurf](https://ultrasurf.us/) / Por el momento, solo se incluye hex-strings para [Ultrasurf](https://ultrasurf.us/)

### HOW TO USE
---

####  [Iptables](http://www.netfilter.org/documentation/HOWTO/es/packet-filtering-HOWTO-7.html) Rules

Edit your Iptables bash script and add the following rule: / Edite su Iptables bash script y agregue la siguiente regla:
```
# BLACKSTRING (BS)
# Global Variables
iptables=/sbin/iptables
lan=eth1
# Blackstring Variables
BSG=$(curl -s https://raw.githubusercontent.com/maravento/blackstring/master/bsgen)
BSH=$(curl -s https://raw.githubusercontent.com/maravento/blackstring/master/bshex)
# Blackstring Rule
$iptables -N blackstring
for string in `echo -e "$BSG" | sed -e '/^#/d' -e 's:#.*::g'`; do
    $iptables -A FORWARD -i $lan -p tcp --sport 49152:65535 --dport 443 -m string --hex-string "|$string|" --algo bm -j blackstring
done
for string in `echo -e "$BSH" | sed -e '/^#/d' -e 's:#.*::g'`; do
    $iptables -A blackstring -m string --hex-string "|$string|" --algo kmp -j NFLOG --nflog-prefix 'Blackstring: '
    $iptables -A blackstring -m string --hex-string "|$string|" --algo kmp -j DROP
done
```

#####  Replace LAN (eth1)

```
ip -o link | awk '$2 != "lo:" {print $2, $(NF-2)}'
enp2s1: 08:00:27:XX:XX:XX
enp2s0: 94:18:82:XX:XX:XX
```

#####  NFLOG (/var/log/ulog/syslogemu.log)

```
Sep 30 19:09:10 user Blackstring: IN=enp2s1 OUT=enp2s0 MAC=94:18:82:XX:XX:XX:08:00:27:XX:XX:XX:08:00 SRC=192.16.48.200 DST=192.168.1.198 LEN=1440 TOS=00 PREC=0x00 TTL=46 ID=2570 PROTO=TCP SPT=443 DPT=52335 SEQ=3803412614 ACK=2610496290 WINDOW=288 ACK URGP=0 MARK=0
Sep 30 19:09:10 user Blackstring: IN=enp2s1 OUT=enp2s0 MAC=94:18:82:XX:XX:XX:08:00:27:XX:XX:XX:08:00 SRC=192.16.48.200 DST=192.168.1.198 LEN=1440 TOS=00 PREC=0x00 TTL=46 ID=2598 PROTO=TCP SPT=443 DPT=52335 SEQ=3803412614 ACK=2610496290 WINDOW=288 ACK URGP=0 MARK=0
Sep 30 19:09:10 user Blackstring: IN=enp2s1 OUT=enp2s0 MAC=94:18:82:XX:XX:XX:08:00:27:XX:XX:XX:08:00 SRC=13.224.106.59 DST=192.168.1.198 LEN=1440 TOS=00 PREC=0x00 TTL=229 ID=53683 DF PROTO=TCP SPT=443 DPT=52334 SEQ=3460251850 ACK=197824253 WINDOW=119 ACK URGP=0 MARK=0
Sep 30 19:09:10 user Blackstring: IN=enp2s1 OUT=enp2s0 MAC=94:18:82:XX:XX:XX:08:00:27:XX:XX:XX:08:00 SRC=192.16.48.200 DST=192.168.1.198 LEN=1440 TOS=00 PREC=0x00 TTL=46 ID=2644 PROTO=TCP SPT=443 DPT=52335 SEQ=3803412614 ACK=2610496290 WINDOW=288 ACK URGP=0 MARK=0
Sep 30 19:09:11 user Blackstring: IN=enp2s1 OUT=enp2s0 MAC=94:18:82:XX:XX:XX:08:00:27:XX:XX:XX:08:00 SRC=192.16.48.200 DST=192.168.1.198 LEN=1440 TOS=00 PREC=0x00 TTL=46 ID=2672 PROTO=TCP SPT=443 DPT=52335 SEQ=3803412614 ACK=2610496290 WINDOW=288 ACK URGP=0 MARK=0
Sep 30 19:09:12 user Blackstring: IN=enp2s1 OUT=enp2s0 MAC=94:18:82:XX:XX:XX:08:00:27:XX:XX:XX:08:00 SRC=13.224.106.59 DST=192.168.1.198 LEN=552 TOS=00 PREC=0x00 TTL=229 ID=55082 DF PROTO=TCP SPT=443 DPT=52331 SEQ=2591131959 ACK=3496229244 WINDOW=119 ACK URGP=0 MARK=0
```

##### Algorithms Used

-  [bm](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_string-search_algorithm)
-  [kmp](https://en.wikipedia.org/wiki/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm)

### CONTRIBUTIONS
---

We thank all those who contributed to this project / Agradecemos a todos los que han contribuido con este proyecto

Special thanks to: [Jhonatan Sneider](https://github.com/sney2002)

### DONATE
---

BTC: 3M84UKpz8AwwPADiYGQjT9spPKCvbqm4Bc

### LICENCES
---

[![GPL-3.0](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl.txt)

[![CreativeCommons](https://licensebuttons.net/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)
[maravento.com](http://www.maravento.com) is licensed under a [Creative Commons Reconocimiento-CompartirIgual 4.0 Internacional License](http://creativecommons.org/licenses/by-sa/4.0/).

© 2019 [Maravento Studio](http://www.maravento.com)

### DISCLAIMER
---

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
