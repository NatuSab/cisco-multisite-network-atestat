[README.md](https://github.com/user-attachments/files/34681308/README.md)
# Atestat – Configurare rețea Cisco multi-site

Proiect de atestat realizat în **Cisco Packet Tracer**: o rețea de întreprindere cu 6 locații (București, Iași, Cluj-Napoca, Timișoara, Oradea, Brașov), interconectate printr-o infrastructură de 3 ISP-uri.

**Autori:** Lucea Sabin și Burza Adina Marina

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?logo=cisco&logoColor=white)
![Topic](https://img.shields.io/badge/topic-networking-blue)
![License](https://img.shields.io/badge/license-MIT-green)

---

## Cuprins

- [Ce demonstrează proiectul](#ce-demonstrează-proiectul)
- [Structura repository-ului](#structura-repository-ului)
- [Cum se deschide simularea](#cum-se-deschide-simularea)
- [Topologie și adresare](#topologie-și-adresare)
- [Subiectele implementate](#subiectele-implementate)
- [Verificare](#verificare)
- [Licență](#licență)

---

## Ce demonstrează proiectul

| # | Subiect | Tehnologie |
|---|---------|------------|
| I | Configurații de bază | hostname, banner MOTD, `no ip domain lookup`, `logging synchronous` |
| II | Subnetare optimă | VLSM pe spațiul `145.96.0.0/19` |
| III | Adresare statică | IPv4 pe stații și routere, legături punct-la-punct `/30` |
| IV | VLAN-uri | VLAN 10/20/30, trunking, VLAN `BLACK-HOLE` (999) pentru porturi nefolosite |
| V | Router-on-a-Stick | subinterfețe `dot1Q` pe `Router_Oradea-Office` |
| VI | DHCPv4 | un pool per VLAN, adrese gateway excluse |
| VII | EtherChannel | PAgP (proprietar Cisco) și LACP (standard IEEE) |
| VIII | OSPF | single area, process ID 25, router-ID-uri manuale |
| IX | Securitate | SSH v2, RSA 1024, autentificare locală |

---

## Structura repository-ului

```
.
├── Atestat.pkt                                      # simularea Packet Tracer
├── Lucrare_atestat_Configurare_Retea_Cisco_2026.docx # lucrarea scrisă
├── ATESTAT.md                                       # comenzile de configurare, pe subiecte
├── LICENSE
└── README.md
```

---

## Cum se deschide simularea

1. Instalează **Cisco Packet Tracer** (gratuit, necesită cont Cisco Networking Academy).
2. Deschide `Atestat.pkt` din Packet Tracer (*File → Open*).
3. Așteaptă câteva secunde ca OSPF să convergă (legăturile devin verzi).
4. Intră pe orice echipament (tab-ul *CLI*) și rulează comenzile de verificare de mai jos.

> Fișierul a fost creat cu o versiune recentă de Packet Tracer. Dacă îl deschizi cu o versiune mai veche, este posibil să nu se încarce.

---

## Topologie și adresare

### Rețele locale (LAN)

| Locație | Hosturi necesari | Rețea | Mască | Gateway |
|---------|:---:|-------|-------|---------|
| București-HQ | 588 | `145.96.0.0` | `/22` (255.255.252.0) | `145.96.0.1` |
| Iași-Branch | 159 | `145.96.4.0` | `/24` (255.255.255.0) | `145.96.4.1` |
| Cluj-Napoca-Office | 80 | `145.96.5.0` | `/25` (255.255.255.128) | `145.96.5.1` |
| Timișoara-Office | 67 | `145.96.5.128` | `/25` (255.255.255.128) | `145.96.5.129` |
| Oradea – Guest (VLAN 20) | 42 | `145.96.6.0` | `/26` (255.255.255.192) | `145.96.6.1` |
| Oradea – IT (VLAN 30) | 36 | `145.96.6.64` | `/26` (255.255.255.192) | `145.96.6.65` |
| Oradea – HR (VLAN 10) | 20 | `145.96.6.128` | `/27` (255.255.255.224) | `145.96.6.129` |

### Legături între routere (`/30`)

| Rețea | Capăt A | Capăt B |
|-------|---------|---------|
| `145.96.6.160/30` | Router_Oradea-Office `s0/0/0` – .161 | ISP_Oradea `s0/0/0` – .162 |
| `145.96.6.164/30` | ISP_Oradea `s0/0/1` – .165 | ISP_Transilvania `s0/1/1` – .166 |
| `145.96.6.168/30` | ISP_Oradea `s0/3/0` – .169 | ISP_Bucuresti `s0/3/1` – .170 |
| `145.96.6.172/30` | ISP_Transilvania `s0/1/0` – .173 | ISP_Bucuresti `s0/1/1` – .174 |
| `145.96.6.176/30` | Router_Brasov `s0/0/1` – .177 | ISP_Transilvania `s0/2/0` – .178 |
| `145.96.6.180/30` | Router_Brasov `s0/3/0` – .181 | Router_Cluj-Napoca-Office `s0/3/0` – .182 |
| `145.96.6.184/30` | Router_Brasov `s0/2/1` – .185 | Router_Timisoara-Office `s0/2/0` – .186 |
| `145.96.6.188/30` | Router_Brasov `s0/3/1` – .189 | Router_Iasi-Branch `s0/3/0` – .190 |
| `145.96.6.192/30` | Router_Cluj-Napoca-Office `s0/3/1` – .193 | Router_Timisoara-Office `s0/3/0` – .194 |
| `145.96.6.196/30` | Router_Timisoara-Office `s0/3/1` – .197 | Router_Iasi-Branch `s0/3/1` – .198 |
| `145.96.6.200/30` | Router_Bucuresti-HQ `s0/3/0` – .201 | ISP_Bucuresti `s0/3/0` – .202 |

### Adresare stații

- **Cluj, Timișoara, Iași:** `PC_XX-1`, `PC_XX-2`, `PC_XX-3` primesc, în ordine, primele adrese după gateway (ex. `PC_CJ-1` = `145.96.5.2`).
- **București-HQ:** `PC_Bucuresti-1…4` = `145.96.0.2…5`, `DNS_Server-5` = `145.96.0.6`.
- **Oradea:** stațiile primesc adrese prin DHCP.

---

## Subiectele implementate

### I. Configurații de bază

Pe fiecare router și switch:

```
enable
configure terminal
no ip domain lookup
banner motd #Atestat#
hostname <NUME_ECHIPAMENT>
line console 0
 logging synchronous
```

### II–III. Subnetare VLSM și adresare statică

Rețelele sunt alocate în ordinea descrescătoare a numărului de hosturi, pentru a nu risipi adrese. Exemplu pe `Router_Bucuresti-HQ`:

```
interface g0/0
 no shutdown
 ip address 145.96.0.1 255.255.252.0
```

### IV. VLAN-uri (Oradea)

```
vlan 10
 name Departament_HR
vlan 20
 name Departament_Guest
vlan 30
 name Departament_IT
interface range fa0/2-3, g0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

Porturile nefolosite sunt mutate în VLAN 999 (`BLACK-HOLE`) și dezactivate (`shutdown`) ca măsură de securitate.

### V. Router-on-a-Stick

```
interface g0/0
 no shutdown
interface g0/0.20
 encapsulation dot1Q 20
 ip address 145.96.6.1 255.255.255.192
interface g0/0.30
 encapsulation dot1Q 30
 ip address 145.96.6.65 255.255.255.192
interface g0/0.10
 encapsulation dot1Q 10
 ip address 145.96.6.129 255.255.255.224
```

### VI. DHCPv4

```
ip dhcp excluded-address 145.96.6.1
ip dhcp excluded-address 145.96.6.65
ip dhcp excluded-address 145.96.6.129

ip dhcp pool Guest
 network 145.96.6.0 255.255.255.192
 default-router 145.96.6.1
ip dhcp pool IT
 network 145.96.6.64 255.255.255.192
 default-router 145.96.6.65
ip dhcp pool HR
 network 145.96.6.128 255.255.255.224
 default-router 145.96.6.129
```

### VII. EtherChannel (București-HQ)

| Legătură | Protocol | Channel-group | Porturi | Mod |
|----------|----------|:---:|---------|-----|
| SW-Bucuresti-1 ↔ SW-Bucuresti-2 | PAgP | 1 | `fa0/1-3` | `desirable` |
| SW-Bucuresti-1 ↔ SW-Bucuresti-3 | PAgP | 2 | `fa0/8-10` | `desirable` |
| SW-Bucuresti-2 ↔ SW-Bucuresti-3 | LACP | 3 | `fa0/4-6` | `active` |

```
interface range fa0/1-3
 channel-group 1 mode desirable
```

### VIII. OSPF

Single area (area 0), process ID `25`.

| Router | Router-ID |
|--------|-----------|
| Router_Oradea-Office | 1.1.1.1 |
| ISP_Oradea | 2.2.2.2 |
| ISP_Transilvania | 3.3.3.3 |
| Router_Brasov | 4.4.4.4 |
| Router_Cluj-Napoca-Office | 5.5.5.5 |
| Router_Timisoara-Office | 6.6.6.6 |
| Router_Iasi-Branch | 7.7.7.7 |
| ISP_Bucuresti | 8.8.8.8 |
| Router_Bucuresti-HQ | 9.9.9.9 |

```
router ospf 25
 router-id 9.9.9.9
interface g0/0
 ip ospf 25 area 0
interface s0/3/0
 ip ospf 25 area 0
```

### IX. SSH (Router_Bucuresti-HQ)

```
ip domain name ATESTAT
crypto key generate rsa general-keys modulus 1024
username LNI secret ATESTAT
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
```

> ⚠️ Credențialele de mai sus sunt **doar pentru simulare**. Nu folosi niciodată parole de acest tip pe echipamente reale.

### Salvarea configurației

Pe fiecare echipament:

```
write
```

---

## Verificare

| Ce verifici | Comandă |
|-------------|---------|
| Tabela de rutare | `show ip route` |
| Vecini OSPF | `show ip ospf neighbor` |
| Interfețe | `show ip interface brief` |
| VLAN-uri | `show vlan brief` |
| Trunk-uri | `show interfaces trunk` |
| EtherChannel | `show etherchannel summary` |
| Pool-uri DHCP | `show ip dhcp binding` |
| SSH | `show ip ssh` |
| Conectivitate | `ping` și `tracert` între stații din locații diferite |

---

## Licență

Distribuit sub licența **MIT**. Vezi fișierul [LICENSE](LICENSE).

Proiect realizat în scop educațional.
