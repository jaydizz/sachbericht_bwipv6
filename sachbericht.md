# Sachbericht bwIPv6@academia 2020

Dieser Sachbericht bietet einen Einblick in die Tätigkeiten die im Rahmen des Projekts ausgeführt wurden. Einmal soll hierbei auf die Umsetzung der Maßnahmen im Integrationsplan eingegangen werden. Danach wird noch einmal auf die Projektmeilensteine eingegangen. 

## Rückblick auf Integrationsplan 

Der Integrationsplan stellte den Rahmen dar, in dem die Projektziele an der Universität Ulm erfüllt werden sollten. Dabei ist nicht nur der Roll-Out von IPv6 in den Institutsnetzen im Fokus, sondern vor allem die Schaffung von zukunftssicheren Strukturen in Bezug auf Firewalling und Netzsegmentierung. 
Die Key-Punkte aus dem Integrationsplan waren:
* Renumbering (`2001:7c0:900::/48 -> 2001:7c0:3100::/40`)
* IPAM
* Stateful-Firewalling
* Policy-Klassen
    * Dummy-Netze im "Fritz-Box-Mode"
    * Offene Netze
    * Mischnetze
* Konfigurationsmethodik
* Dienste mit Dual-Stack ausstatten. 
* Hardware
* Management 


### Renumbering

Das Renumbering stellte eine der größeren Hürden dar, da wir nicht nur IPv6 ausrollen, sondern zeitgleich auch bestehende Deployments anpassen mussten. Das Renumbering von `2001:7c0:900::/48` nach `2001:7c0:3100::/40` ist jedoch abgeschlossen. 
Lediglich das BGP-Announcement muss, nach Absprache mit BelWue, abgestellt werden und die Delegation im DNS und im whois an BelWüe zurück gegeben werden. 

### IPAM

Das IPAM war schon zum Zeitpunkt des Integrationsplans weitestgehend fertig und im produktiven Einsatz. 
Jedoch sind noch Features hinzugekommen:
Es ist nun möglich, per Keyword Netze als "PLANNED" zu deklarieren. Das sind Prefixes, die zwar für den jeweiligen Einsatz vorgesehen, aber aus diversen Gründen noch nicht konfiguriert sind. 

#### IPAM und Firewalling
Im Zusammenspiel mit der CI-Pipeline des Firewallings können Prefixes nun auch mit dem Key-Value-Paar `FIREWALL = <PROFILE>` belegt werden. Dies sorgt dafür, dass das jeweilige Prefix mit einem gewissen Firewall-Profil belegt wird. 
Beispielheader einer Vlan-Datei:
```
###############################################################
VLAN_ID     = 2
VLAN_NAME   = o26-kiz
DESCRIPTION = kiz-staff-ost-old
SUBNET_V4   = 10.100.2.0/24
GATEWAY_V4  = 10.100.2.2
PREFIX_V6   = 2001:7c0:3101:4005::/64
FIREWALL    = std
NAT_ROUTER  = 134.60.112.188
###############################################################
```

#### IPAM und DHCPv6
Das IPAM kann nun auch DHCPv6 Leases erstellen und direkt und automatisisert dem isc-dhcp-server zuführen. 

#### Zusammenfassung:

Das IPAM war von Anfang an für Feature-Parity zwischen IPv4 und IPv6 ausgelegt. Über die Zeit kamen noch weitere spezifische Features hinzu die aber natürlich für IPv4 und IPv6 implementiert wurden. 


### Policy-Klassen und Firewalling

Im Integrationsplan wird das Konzept der Netzklassen in Bezug auf das Firewalling vorgestellt. Dabei ging es darum, Nutzern und Instituten die Möglichkeit zu bieten, offene Netzbereiche und Bereiche hinter einer stateful-Firewall zu nutzen. 
Das wird im ersten Schritt dadurch abgebildet, dass pro Vlan mehrere Prefixe verwendet werden: eines ist "OPEN", sprich nicht hinter einer Firewall. Weitere Netze sind eventuell hinter der Firewall.
Zum Rollout von IPv6 in den Instituten erachteten wir im Sachbericht die Ermöglichung von SLAAC für essentiell. Diese koppelten wir aber an Firewalling, da sichergestellt werden sollte, dass Rechner nicht aus Versehen im Internet erreichbar sein sollen. Analog dazu wollen wir für diese gefirewallten Netzen auch IPv4 via DHCP anbieten. 


#### Verfügbare Policies

Zum Zeitpunkt des Statusberichtes sind zwei Firewall-Policies zusätzlich zum Bypass verfügbar.
Für alle Klassen gilt: IPv6 wird native angeboten, während IPv4 hinter NAT hängt und private Adressen genutzt werden. 

**STD (Standard)**: "Fritz-Box-Mode", überallhin stateful nach der Regel `established in`. 
**STD_VPN**:        Wie STD, aber VPN-Clients dürfen auch von aussen nach innnen. 

Folgende Policies werden noch implementiert:

**GRP**: Gruppen-Policy: Alle Netze aus einer Gruppe können aufeinander zugreifen, alles andere ist stateful und `established in`

**CUSTOM**: Kunde kann selber via gitlab Profilregeln festlegen und pull-requests stellen. 



#### Zusammenfassung

Das Firewalling und Policy-Konzept wurde weitestgehend umgesetzt, wie im Integrationsplan beschrieben. 
Die ersten Institute laufen schon in diesem Modus und sind bisher sehr zufrieden. 



### Konfigurationsmethoden

Im Integrationsplan stellte sich noch die Frage, welche Konfigurationsmethode wann verwendet werden soll. 
Mit unserem Konzept, dass mehrere Netzbereiche verwendet werden können, hat sich folgende Herangehensweise als die Sinnvollste herausgestellt:

#### OPEN:
**Konfigurationsmethode:** statisch. 

Wenn eine offene Anbindung ans Internet gewünscht ist, dann wollen wir, dass sich die Nutzer _davor_ mit uns in Verbindung setzen. 
Wir teilen dann auch einen DNS-Namen zu. Zusätzlich werden die Neighbor-Caches überprüft, ob jeder Adresse aus einem statischen Bereich auch ein Hostname zugeordnet ist. Ansonsten wird die jeweilige Adresse am Edge per ACL gesperrt. 


#### FIREWALLED/FRITZBOX:
**Konfigurationsmethode:** SLAAC

Grundsätzlich ist uns wichtig, dass niemand ohne sein Wissen auf einmal offen im Internet erreichbar ist. Da wir nun in der Lage sind, einfache Netze zu firewallen, wollen wir jedem, der sich ansteckt zunächst einmal Konnektivität bieten. 
Deswegen setzen wir in den FRITZBOX-Netzen auf SLAAC.

#### AUF ANFORDERUNG:

Nutzern, die spezielle Anforderungen haben (z.B DHCPv6), können wir diese erfüllen. 
Je nach Wunsch, kann ein offenes oder firewalled Netz mit einem DHCP Relay auf unseren Routern kombiniert werden. 


### Dienste mit Dual-Stack ausstatten

Einige Dienste der Universität waren noch nicht per IPv6 erreichbar. Wir wollen erreichen, dass alle bestehenden Dienste dual-stack-fähig werden. 
Im Folgenden soll erneut ein Blick auf die im Integrationsplan aufgeführten Dienste geworfen werden. 

#### UPDATE: Mail:
**Status:** Dual-Stack.

Mail ist mittlerweile dual-stack-fähig und im produktiven Einsatz. 

#### UPDATE: VPN:
**Status:** Dual-Stack.

Das VPN ist renumbered und im produktiven Einsatz.

#### RADIUS: 
**Status:** IPv4 Only.

Der RADIUS-Server ist immer noch nur via IPv4 erreichbar. Hier gilt es nach wie vor zu klären, ob und wann DFN den RadSecProxy auch via IPv6 anbietet. 

#### UPDATE: Active Directory

Dei Windows-Clients haben alle via DHCPv6 eine gültige IPv6-Adresse. Die Domänencontroller haben auch schon IPv6. Jedoch haben die Domänencontroller noch keinen AAAA-Record. Hier muss noch evaluiert werden, bevor das deployment live geschaltet werden kann. 


#### UPDATE: Zentrale Universitätsverwaltung (ZUV)
**Status:** Clients können v6. Server müssen noch auf Dual-Stack umgestellt werden.

Die Firewall vor der ZUV kann nun IPv6. Die meisten Clients haben nun auch eine IPv6-Adresse und benutzen Dual-Stack. 
Jedoch fehlen noch einige Server-Dienste: Unter anderem auch HIS. 

#### IDM

Nachwievor haben die LDAP-Server AAAA-Einträge. Hier sind noch Abhängigkeiten zum IPv6 Rollout in der Zentralen Universitätsverwaltung vorhanden. Erst danach können alle IDM-Dienste voll dual-stack-fähig gemacht werde.  
Das Self-Service-Portal für Studierende wird auch erst nach Umstellung aller ZUV-Serverdienste umgestellt.

#### Sonstige Dienste: 
**Status:** gemischt

Bei SOGO herrscht noch eine Abhängigkeit zur Uni Konstanz, da diese den Service für uns erbringt. 

#### WWW-Proxy:

Der WWW-Proxy im Verwaltungsnetz ist nun dual-stack-fähig.


### Hardware 

####  Firewalling

Zur Bereitstellung des Firewallings wurden zwei Mikrotik CCR1072 beschafft und konfiguriert. Eine CI-Pipeline existiert. 
Mit diesen Geräten können nun Firwalling-Dienste für die automatisch konfigurierten Netzbereiche zur Verfügung gestellt werden.

#### Sonstige:

Wir haben keine Hardware außerhalb der regulären Anschaffung gekauft. 

### Management

In diesem Abschnitt wird vor allem das Management-Netz und dessen Services betrachtet. 

#### Monitoring

Die neue zentrale Monitoringplattform ist mittlerweile in der Lage, Metriken via IPv6 abzuholen. 
Dies passiert auch schon bei Geräten, deren Management auch IPv6 unterstützt. 
Dazu zählen vor allem die Core-Router, Nat-Router/Firewalls, oVirt-Knoten und viele mehr. 

##### Probleme bei der Umstellung:
Bei Mikrotik-Geräten mit mehreren IP-Adressen muss eine Source-Adresse für SNMP konfiguriert werden. Leider kann hier nur _entweder_ eine IPv4 _oder_ eine IPv6-Adresse angegeben werden. Abhängig von der Address-Family bindet der snmp-daemon dann nur auf diese Addr-family. `::/0` sorgt zwar dafür, dass der snmpd auf alle dst-adressen hört, jedoch antwortet RouterOS beim Vorhandensein von mehreren IPv6 (Loopback-)Adressen mit der _niedrigsten_ Adresse. 
Das ist für IPv4 egal, bei IPv6 dropt Go-SNMP jedoch das Antwortpaket, was dazu führt, dass Telegraf keine Metriken mehr sammeln kann. 
Ein Patch für GO-SNMP ist bereits eingereicht (nicht von uns), ein Workaround ist, die src-addr des snmpd auf eine feste Loopback-Adresse zu konfigurieren. Damit verlieren wir aber leider (_sarcasm_) die Möglichkeit Metriken via IPv4 zu sammeln. 

Bei den von uns eingesetzten Cisco ASAs konnte keine ACL für snmp mit IPv6 festgelegt werden. Das wird erst in ASA 9.9 unterstüzt. jedoch können zwei unserer ASAs diese Version nicht mehr bekommen. Hier werden die Metriken weiter über IPv4 gesammelt. 

#### Management-Netz
Alle unsere Geräte unterstützen prinzipiell ssh via IPv6. Allerdings sind wir momentan noch vorsichtig bei der Umstellung des Management-Netzes auf IPv6-only. Das Monitoring hat gezeigt, dass doch nicht überall Feature-Parity vorherrscht.
Deswegen gilt es hier, zunächst einzelne Inseln zu schaffen. 
Als Zwischenschritt werden wir auch hier vorerst den Dual-Stack-Betrieb anstreben. 

#### Management-Tooling

Die Management-Knoten im Netzwerkbereich sind alle im Dual-Stack Betrieb. Alle eingesetzten Tools sind mittlerweile bei feature-parity angelangt: 
So werden beispielsweise bei einem Lookup, an welchem Switchport eine IPv4-Adresse hängt, automatisch alle v6-Adressen mit ausgegeben. 
Alle Neighbor-Caches werden eingesammelt und verarbeitet. 
Beispiel:
```
mad-portfinder -a 134.60.2.40 
------------------------------------------------------------------------------------------------------------------------------------------------------------
SWITCH:PORT      | MAC               | VLAN | P-TYPE | IP                                   | DNS                  | FDB-TIMESTAMP       | ARP-DRIFT | PATCH
------------------------------------------------------------------------------------------------------------------------------------------------------------
o26-5003-p-gs1:4 | 54:05:DB:76:06:BF |    2 | ACCESS | 134.60.2.40                          | jaydi-new.rz         | 2021-02-01 18:52:01 |   +1006 s |      
o26-5003-p-gs1:4 | 54:05:DB:76:06:BF |    2 | ACCESS | 2001:7c0:3101:4005:39:e31f:3b9c:5b0b | vlan-2-firewalled.rz | 2021-02-01 18:52:01 |   +1015 s |      
o26-5003-p-gs1:4 | 54:05:DB:76:06:BF |    2 | ACCESS | fe80::9b05:c2db:e548:f804            | -                    | 2021-02-01 18:52:01 |   +1015 s |    
```





## Blick auf die Meilensteine im Projekt

### Meilenstein 1: Statusübersicht der IPv6-Fähigkeit der eingesetzen Kompononenten
Die Statusübersicht liegt im Integrationsplan vor. Im L2 _sollten_ einige Switches ersetzt werden, die keine IPv6-Features haben (RA-Guard o.ä.). Diese hindern allerdings nicht am Rollout. 
Vereinzelt zeigen sich noch Probleme bei einigen Appliances, die via IPv6 diverse Management-Features nicht unterstützen (Cisco ASA kann kein SNMP via v6). Bisher gab es hier jedoch keine Show-Stopper. 
Es werden bei uns überwiegend folgende Geräte eingesetzt:

In der Tabelle ist eine Übersicht der von uns verwendeten Hardware zu sehen. 
Hinweise zu den Bemerkungen: 
-> Vollumfänglich heißt: Alle von uns benötigten Features werden unterstützt 
* RA-Guards
* IGMP für IPv6
* ACLs für IPv6
* SNMP/SSH/Management via IPv6 

| Geräteklasse| Modell |IPv6 Fähigkeiten | Bemerkung |
| -------- | ------- |-------- | -------- |
| Access-Switch                     | Aruba/Procurve 2530/2540 | Voll umfänglich.      |
| Access-Switch                     | Procuve 2510/2650/2810   | teilw. unzureichend. z.B kein RA-Guard. | Werden sowieso ersetzt. 
| Aggregation/Distribution-Switch   | Cisco Nexus 3xxx         | Vollumfänglich | teilweise EoL, aber "good enough" 
| Aggregation/Distribution-Switch   | Cisco Catalyst 68xx      | Vollumfänglich | Alte Core-Router
| Core/Edge                         | Cisco Nexus 9k           | Vollumfänglich | 
| Firewalling/Nat-GWs               | Mikrotiks (Verschiedene mit RouterOS) | Vollumfänglich (Ausnahme: Keine /127 Linknetze) | Stateful-Firewalls, Wohnheims-FWs, DMZ, Wifi, etc. 
| VPN                               | Cisco ASA                | In aktueller Version kein SNMP via IPv6. | Kein Show-Stopper. Workaround für Monitoring. 
| Wifi-APs                          | Aruba (verschiedene)     | Keine Client-Isolation für v6 ohne Controller. | Ersetzen die alten Cisco-APs
| Wifi-APs                          | (verschiedene)           | Teilweise keine ACLs für IPv6, kein Multicast Snooping | werden durch Aruba turnusgemäß ersetzt. 



### Meilenstein 2: Aufwands und Investitionsschätzung

Die notwendingen Investitionen können weitestgehend im normalen Haushalt abgefangen werden: Veraltete Switches werden turnusgemäß getauscht. 
Speziell für das IPv6-Projekt wurden zwei stateful-firewalls angeschafft. Die Investition belief sich auf < 10.000 Euro. 
Bei der Entwiclungs des notwendigen Toolings (IPAM, CI-Pipelines, etc.) sind  zwei Personen eingebunden. Diese Tätigkeiten befinden sich aber teils auch in den normalen Aufgabengebieten dieser Personen.

### Meilenstein 3: IPv6-Allokationsplan und Zielvorgaben

Die Uni Ulm entwickelte selbst ein IPAM. Ein Addressierungsplan wurde gemäß des Integrationsplans erstellt und ist aktiv in Verwendung.
Ein Auszug aus dem Adress-Plan (ohne Vlans, nur grobe Delegationen) ist im Folgenden zu sehen.

```
├─ 2001:07c0:233~~/44 ................. CONTAINER: bwCloud external users IPv6
├─ 2001:07c0:31~~/40 .................. CONTAINER: second official ulm university block for IPv6
│  ├─ 2001:07c0:3100::/48 ................ CONTAINER: kiz IT services
│  │  ├─ 2001:07c0:3100:0~~/52 .............. CONTAINER: kiz basic IT services, AAa=000
│  │  └─ 2001:07c0:3100:1~~/52 .............. CONTAINER: Netzwerk Infrastruktur
│  ├─ 2001:07c0:3101::/48 ................ CONTAINER: AAa=01x
│  │  ├─ 2001:07c0:3101:0~~/52 .............. CONTAINER: OPEN: dualstack VLANs, static only, no autoconfig AAa=010 BBB=number
│  │  ├─ 2001:07c0:3101:1~~/52 .............. CONTAINER: OPEN: dualstack VLANs, Slaac, AAa=011 BBB=number
│  │  ├─ 2001:07c0:3101:2~~/52 .............. CONTAINER: OPEN: dualstack VLANs, DHCP, AAa=012 BBB=number
│  │  ├─ 2001:07c0:3101:3~~/52 .............. CONTAINER: FIREWALLED: dualstack VLANs, Static, AAa=013 BBB=number
│  │  └─ 2001:07c0:3101:4~~/52 .............. CONTAINER: FIREWALLED: dualstack VLANs, Slaac, AAa=014 BBB=number
│  ├─ 2001:07c0:3102::/48 ................ CONTAINER: AAa=02x, Firewalled by Customer.
│  │  └─ 2001:07c0:3102:00~~/56 ............. CONTAINER: AAa=02x, Kiz: ZUV, DMZ, etc.
│  ├─ 2001:07c0:31fe::/48 ................ CONTAINER: Testnetze für alles Mögliche
│  └─ 2001:07c0:31ff::/48 ................ CONTAINER: extern, Aninstitute, Wohnheime, ...
│     ├─ 2001:07c0:31ff:0~~/52 .............. CONTAINER: Wohnheime SLAAC
│     ├─ 2001:07c0:31ff:1~~/52 .............. CONTAINER: Wohnheime Prefix Delegation
│     ├─ 2001:07c0:31ff:2~~/52 .............. CONTAINER: OPEN: Externe Institute
│     │  ├─ 2001:07c0:31ff:20~~/56 ............. CONTAINER: OPEN: Externe Institute, Static
│     │  ├─ 2001:07c0:31ff:21~~/56 ............. CONTAINER: OPEN: Externe Institute, SLAAC
│     │  └─ 2001:07c0:31ff:22~~/56 ............. CONTAINER: OPEN: Externe Institute, DHCP
│     └─ 2001:07c0:31ff:3~~/52 .............. CONTAINER: FIREWALLED: Externe Institute
│        ├─ 2001:07c0:31ff:30~~/56 ............. CONTAINER: FIREWALLED: Externe Institute, Static
│        ├─ 2001:07c0:31ff:31~~/56 ............. CONTAINER: FIREWALLED: Externe Institute, SLAAC
│        └─ 2001:07c0:31ff:32~~/56 ............. CONTAINER: FIREWALLED: Externe Institute, DHCP
└─ fc<</7 ............................. CONTAINER: IANA ULA fc00::/7
   └─ fdcd:aa59:8bce::/48 ................ CONTAINER: ULA Uni-Ulm
      └─ fdcd:aa59:8bce:00~~/56 ............. CONTAINER: Router Loopbacks, Linknetze, ...
```

Die Aufteilung der Netze erfolgt zuerst nach der Art der Institution. Innerhalb dieser Aufteilung wird nach Firewalling Policy und Konfigurationsart unterschieden (SLAAC, DHCP, STATIC).

### Meilenstein 4: Integrationsplan 

Der Integrationsplan liegt vor und kann unter (https://hackmd.io/GlY5nPoEQ0iNyrXR7mdBkA?view) eingesehen werden.

### Meilenstein 5: Aussenanbindung mit IPv6 ausgestattet

Die Universität Ulm verfügte schon vor Beginn des Projekts über ein IPv6-Prefix, das auch aktiv genutzt wurde und an BelWue announced wurde. 

### Meilenstein 6: Aktivierung von IPv6 im Kernnetz

Bereits vor Beginn war der Core der Uni Ulm voll IPv6-fähig. 

### Meilenstein 7: Anbindung zentraler Dienste per IPv6 (NTP, DNS, Mail und Web)

Die vier genannten Dienste sind via IPv6 erreichbar und nutzbar.


### Meilenstein 8: Wlan und VPN via IPv6

Das VPN unterstützt IPv6 in vollem Umfang. 
Im Wlan steht dem Roll-Out technisch nichts im Wege. Jedoch stellten wir fest, dass wir ohne Wlan-Controller auf unseren Aruba-APs keine IPv6 Client-Isolation konfigurieren können. 
Hier haben wir nun die politische Frage zu klären, ob dies ein Show-Stopper ist, oder nicht.
Momentan ist daher der Roll-Out von IPv6 im Wlan-Netz noch n8cht erfolgt und bis zur Klärung dieser Frage verschoben.

### Meilenstein 9: IPv6-Anbindung von mindestens fünf Instituten. 

Der IPv6-Rollout erfolgt graduell. 
Bei bestehenden Netzen erfolgt ein Roll-Out auf Anfrage. Neue Netze bekommen IPv6 per-se mitgeliefert. 
Nachdem die CI-Pipeline für das Stateful-Firewalling fertig ist, werden wir mit dem flächendeckenden Roll-Out beginnen. 
Allerdings ist Meileinstein 9 auch so bereits erfüllt: Es sind mehr als 5 Institute mit IPv6 versorgt. Die Akzeptanz der Administratoren war dabei sehr hoch und in den meisten Fällen mit viel Freude verbunden. 
Unten ist ein Auszug aus unserem Vergabeplan zu sehen, aus dem hervorgeht, dass bereits einige Einrichtungen der Universität Ulm IPv6-fähig sind. 

```
  ├─ 2001:07c0:3101::/48 ................ CONTAINER: AAa=01x
│  │  ├─ 2001:07c0:3101:0~~/52 .............. CONTAINER: OPEN: dualstack VLANs, static only, no autoconfig AAa=010 BBB=number
│  │  │  ├─ 2001:07c0:3101:010b::/64 ........... vid(  -1) pool-omi-vms-1
│  │  │  ├─ 2001:07c0:3101:0a00::/64 ........... vid(  40) Magazin/Housing
│  │  │  ├─ 2001:07c0:3101:0a01::/64 ........... vid( 313) Justus2
│  │  │  ├─ 2001:07c0:3101:0a02::/64 ........... vid(  58) Uni Institute in der Lise Meitner Straße 16
│  │  │  ├─ 2001:07c0:3101:0a03::/64 ........... vid(  51) Institute im Pavillon-1, AEA-5
│  │  │  ├─ 2001:07c0:3101:0a04::/64 ........... vid(   2) kiz-staff-ost-old
│  │  │  ├─ 2001:07c0:3101:0a05::/64 ........... vid(  -1) bwnet-100g-projekt
│  │  │  ├─ 2001:07c0:3101:0a06::/64 ........... vid( 320) welcome static routed
│  │  │  ├─ 2001:07c0:3101:0a07::/64 ........... vid( 310) bwCloud-uni-ulm
│  │  │  ├─ 2001:07c0:3101:0a08::/64 ........... vid(  89) parkstrasse-mixed
│  │  │  ├─ 2001:07c0:3101:0a09::/64 ........... vid(  68) SGI-Server
│  │  │  ├─ 2001:07c0:3101:0a0a::/64 ........... vid(  47) Magazin/Housing-2
│  │  │  ├─ 2001:07c0:3101:0a0c::/64 ........... vid(  -1) pool-omi-vms-2
│  │  │  ├─ 2001:07c0:3101:0a30::/64 ........... vid(  30) uni-west-43-iomi
│  │  │  ├─ 2001:07c0:3101:0a31::/64 ........... vid( 114) M26, ZQB
│  │  │  ├─ 2001:07c0:3101:0a32::/64 ........... vid( 209) SGI-Pool O27
│  │  │  ├─ 2001:07c0:3101:0a33::/64 ........... vid(  55) N25/1 mixed
│  │  │  ├─ 2001:07c0:3101:0a34::/64 ........... vid(  74) 74-pm
│  │  │  ├─ 2001:07c0:3101:0a35::/64 ........... vid(  26) AEM & IFM (PLANNED)
│  │  │  ├─ 2001:07c0:3101:0a36::/64 ........... vid( 335) Psychologie-Institute in der Schaffnerstr.
│  │  │  └─ 2001:07c0:3101:0a37::/64 ........... vid(  83) N25/4
│  │  ├─ 2001:07c0:3101:1~~/52 .............. CONTAINER: OPEN: dualstack VLANs, Slaac, AAa=011 BBB=number
│  │  │  ├─ 2001:07c0:3101:1a00::/64 ........... vid( 114) M26, ZQB
│  │  │  ├─ 2001:07c0:3101:1a01::/64 ........... vid(  77) Verteilte Systeme
│  │  │  ├─ 2001:07c0:3101:1a02::/64 ........... vid( 340) housing-vs
│  │  │  ├─ 2001:07c0:3101:1a08::/64 ........... vid(  89) parkstrasse-mixed
│  │  │  └─ 2001:07c0:3101:1a09::/64 ........... vid(  -1) Azubi Playground
│  │  ├─ 2001:07c0:3101:2~~/52 .............. CONTAINER: OPEN: dualstack VLANs, DHCP, AAa=012 BBB=number
│  │  │  ├─ 2001:07c0:3101:2000::/64 ........... vid( 128) kiz-staff-ost
│  │  │  ├─ 2001:07c0:3101:2001::/64 ........... vid( 130) kiz-staff-west
│  │  │  ├─ 2001:07c0:3101:2002::/64 ........... vid( 132) kiz-pool-ost
│  │  │  └─ 2001:07c0:3101:2003::/64 ........... vid( 133) kiz-pool-west
│  │  ├─ 2001:07c0:3101:3~~/52 .............. CONTAINER: FIREWALLED: dualstack VLANs, Static, AAa=013 BBB=number
│  │  │  ├─ 2001:07c0:3101:3000::/64 ........... vid(  64) IOMI-Cloud-Testbed
│  │  │  ├─ 2001:07c0:3101:3001::/64 ........... vid(  65) IOMI-Lehre
│  │  │  ├─ 2001:07c0:3101:3002::/64 ........... vid(  84) M25/1/2 (PLANNED)
│  │  │  ├─ 2001:07c0:3101:3003::/64 ........... vid( 205) ip_205 kiz zbib (PLANNED)
│  │  │  ├─ 2001:07c0:3101:3005::/64 ........... vid(  68) SGI-Server
│  │  │  ├─ 2001:07c0:3101:3006::/64 ........... vid( 208) SGI-Pool O28
│  │  │  └─ 2001:07c0:3101:3007::/64 ........... vid( 209) SGI-Pool O27
│  │  └─ 2001:07c0:3101:4~~/52 .............. CONTAINER: FIREWALLED: dualstack VLANs, Slaac, AAa=014 BBB=number
│  │     ├─ 2001:07c0:3101:4000::/64 ........... vid( 320) welcome (PLANNED)
│  │     ├─ 2001:07c0:3101:4001::/64 ........... vid( 321) eduroam (PLANNED)
│  │     ├─ 2001:07c0:3101:4002::/64 ........... vid(  84) M25/1/2 (PLANNED)
│  │     ├─ 2001:07c0:3101:4003::/64 ........... vid( 205) ip_205 kiz zbib (PLANNED)
│  │     ├─ 2001:07c0:3101:4004::/64 ........... vid(  51) Institute im Pavillon-1, AEA-5
│  │     ├─ 2001:07c0:3101:4005::/64 ........... vid(   2) kiz-staff-ost-old
│  │     ├─ 2001:07c0:3101:4006::/64 ........... vid(  55) N25/1 mixed
│  │     └─ 2001:07c0:3101:4008::/64 ........... vid(  26) AEM & IFM
```

### Zusammenfassung:
Insgesamt sind folgende Meileinsteine erreicht


| Meileinstein | Bemerkung | Status |
| -------- | -------- | -------- |
| M1       |  | erfüllt     |
| M2       |  | erfüllt     |
| M3       |  | erfüllt     |
| M4       |  | erfüllt     |
| M5       | Bereits vor Projektbeginn.   | erfüllt     |
| M6       | Bereits vor Projektbeginn.  | erfüllt     |
| M7       |                           | erfüllt     |
| M8       | Bei Wlan: Klärung der Client-Isolation  | teilw. erfüllt     |
| M9       | Rollout am Laufen, >5 Institute realisiert.   | erfüllt     |
