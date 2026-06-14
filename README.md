[README-en.md](https://github.com/user-attachments/files/28933455/README-en.md)[README-pt.md](https://github.com/user-attachments/files/28933450/README-pt.md)
# Projeto CCNA 200-301 — Rede Corporativa Hierárquica no Cisco Packet Tracer

Terceiro projeto acumulativo de preparação para a CCNA 200-301, desenvolvido no Cisco Packet Tracer com abordagem incremental — cada lab constrói sobre o anterior, evoluindo de um planejamento de endereçamento até uma rede corporativa completa com matriz e filial interconectadas.

---

## Sobre o projeto

O objetivo foi construir do zero uma rede corporativa realista seguindo o modelo hierárquico de três camadas (Core, Distribution, Access). Dois sites — matriz e filial — conectados por link serial WAN, com serviços como telefonia IP, wireless, DHCP centralizado, NAT, segurança por ACLs e roteamento dinâmico com OSPF multi-área.

Cada lab foi desenvolvido de forma incremental, preservando todas as configurações anteriores e adicionando novos serviços e funcionalidades.

---

## Topologia

```
Internet (ISP)
      |
   R1 (2911/Matriz) ── Serial ── R3 (2911/Filial)
   |        |                         |
 ISP      R2 (2911)              SW-FILIAL (2960)
           |                     |         |
        SW-CORE (3560 L3)     PCs 8~13   AP-FILIAL
        |          |                        |
     SW-DIST1   SW-DIST2               Laptop-Filial
     (2960)     (2960)
      |    ╚══════╝    |   ← EtherChannel LACP Po1
   SW-ACC1         SW-ACC2
   (2960)          (2960)
      |                |
   PCs 0~3          PCs 4~7
   IP Phone         Server0
   Matriz           Printer0
                    Access Point0
                    Laptop1
```

### Equipamentos

| Equipamento | Modelo | Função |
|---|---|---|
| R1 | Cisco 2911 | Gateway matriz — OSPF, NAT, DHCP |
| R2 | Cisco 2911 | Redundância interna — HSRP |
| R3 | Cisco 2911 | Gateway filial — Router-on-a-Stick |
| ISP | Cisco 2911 | Simulação de internet |
| R-Telephony | Cisco 2811 | Servidor de telefonia IP |
| SW-CORE | Cisco 3560 L3 | Núcleo — roteamento inter-VLAN via SVIs |
| SW-DIST1/2 | Cisco 2960 | Distribuição — EtherChannel LACP |
| SW-ACC1/2 | Cisco 2960 | Acesso — conexão com dispositivos finais |
| SW-FILIAL | Cisco 2960 | Switch da filial |

### VLANs

| VLAN | Nome | Rede | Dispositivos |
|---|---|---|---|
| 10 | TI-MATRIZ | 10.10.10.0/24 | PCs ACC1, IP Phone |
| 20 | RH-MATRIZ | 10.10.20.0/24 | PCs ACC2, Server0, Printer0, AP0, Laptop1 |
| 30 | TI-FILIAL | 10.20.10.0/24 | PCs Filial, IP Phone Filial |
| 40 | RH-FILIAL | 10.20.20.0/24 | AP-FILIAL, Laptop-Filial |
| 99 | VOIP | 10.99.20.0/24 | IP Phones |
| 100 | GERENCIAMENTO | 10.99.10.0/24 | Switches |

### Endereçamento IP — links de infraestrutura

| Segmento | Rede | Dispositivos |
|---|---|---|
| R1 ↔ R2 | 10.99.1.0/30 | R1 Gig0/1 ↔ R2 Gig0/0 |
| R2 ↔ SW-CORE | 10.99.2.0/30 | R2 Gig0/1 ↔ SW-CORE Gig0/2 |
| R1 ↔ R3 | 10.99.3.0/30 | R1 Serial0/1/0 ↔ R3 Serial0/0/0 |
| R1 ↔ ISP | 10.99.4.0/30 | R1 Gig0/0 ↔ ISP Gig0/0 |
| R1 ↔ SW-CORE | 10.99.5.0/30 | R1 Gig0/2 ↔ SW-CORE Gig0/1 |
| Gerenciamento | 10.99.10.0/24 | SVIs dos switches |
| VoIP | 10.99.20.0/24 | IP Phones |

---

## Labs

### Lab 1 — Fundamentos e subnetting
Planejamento completo da topologia antes de qualquer configuração. Definição de todas as redes com VLSM, tabela de endereçamento completa e introdução ao IPv6.

**Conceitos abordados:** VLSM, modelos OSI e TCP/IP, TCP vs UDP, IPv4 privado vs público, topologias de rede.

---

### Lab 2 — Configurações básicas e SSH
Configuração inicial de segurança em todos os dispositivos da rede. SSH versão 2 habilitado com chave RSA de 1024 bits. CDP e LLDP verificados. Serviços desnecessários desativados.

**Comandos principais:** `ip domain-name`, `crypto key generate rsa`, `ip ssh version 2`, `transport input ssh`, `username admin privilege 15 secret`, `lldp run`, `no cdp run`

---

### Lab 3 — VLANs, trunking e EtherChannel
Criação das VLANs em todos os switches com native VLAN 100 nos trunks. EtherChannel LACP entre SW-DIST1 e SW-DIST2. Root Bridge manual no SW-CORE. PortFast e BPDU Guard nas portas de acesso. Port Security com sticky MAC.

**Comandos principais:** `vlan 10`, `switchport trunk encapsulation dot1q`, `switchport trunk native vlan 100`, `channel-group 1 mode active`, `spanning-tree vlan 10 root primary`, `spanning-tree portfast`, `spanning-tree bpduguard enable`, `switchport port-security mac-address sticky`

---

### Lab 4 — Roteamento OSPF e inter-VLAN
IPs configurados em todos os roteadores e SW-CORE. SVIs para roteamento inter-VLAN. OSPF área 0 entre R1, R2 e SW-CORE. OSPF área 1 para a filial via R3. Rota default redistribuída via OSPF. HSRP entre R1 e R2 com IP virtual 10.99.1.3.

**Comandos principais:** `ip routing`, `router ospf 1`, `router-id`, `network x.x.x.x wildcard area`, `default-information originate`, `standby 1 ip`, `standby 1 priority`, `standby 1 preempt`

---

### Lab 5 — IP services — DHCP, NAT, NTP, Syslog
DHCP centralizado no R1 com pools separados por VLAN e relay configurado no SW-CORE e R3. NAT estático para o Server0 e PAT para acesso à internet. NTP com R1 como stratum 1. Syslog centralizado no Server0.

**Comandos principais:** `ip dhcp excluded-address`, `ip dhcp pool`, `option 150 ip`, `ip helper-address`, `ip nat inside source static`, `ip nat inside source list overload`, `ntp master 1`, `ntp server`, `logging`

---

### Lab 6 — Segurança — ACLs e layer 2
ACL estendida controlando acesso ao Server0: VLAN 10 (TI) e filial permitidas, VLAN 20 (RH) bloqueada. DHCP Snooping e Dynamic ARP Inspection configurados nos switches de acesso para proteção contra ataques de camada 2.

**Conceito documentado:** ACLs em SVIs de switches L3 no Packet Tracer apresentam comportamento inconsistente — no IOS real funcionaria perfeitamente. Workaround aplicado no R1.

**Comandos principais:** `ip access-list extended`, `permit ip`, `deny ip`, `ip access-group NOME in`, `ip dhcp snooping`, `ip dhcp snooping vlan`, `ip arp inspection vlan`

---

### Lab 7 — Wireless e VoIP
R-Telephony 2811 configurado como servidor de telefonia. IP Phone da matriz registrado no ramal 1001, IP Phone da filial no ramal 1002. Ligação realizada com sucesso entre os dois sites. AP-FILIAL com SSID WIFI-FILIAL WPA2-PSK no canal 1. AP da matriz com SSID WIFI-MATRIZ WPA2-PSK no canal 11.

**Lição aprendida crítica:** O MAC address real do IP Phone é diferente do MAC exibido no modo de simulação do Packet Tracer. O MAC correto deve ser capturado via `show mac address-table interface Fa0/X` no switch. O comando `button 1:1` no ephone é obrigatório — sem ele o telefone entra em loop.

**Comandos principais:** `telephony-service`, `ip source-address`, `max-ephones`, `ephone-dn`, `number`, `ephone`, `mac-address`, `button 1:1`, `auto assign`

---

### Lab 8 — IPv6
Endereços IPv6 unicast global configurados em R1, R2 e R3 com dual-stack IPv4 + IPv6. OSPFv3 área 0 e área 1 estabelecidos com adjacências FULL. Ping IPv6 entre R1 e R3 com 100% de sucesso.

**Conceitos abordados:** formato 128 bits, unicast global vs link-local, EUI-64, NDP como substituto do ARP, diferenças OSPFv2 vs OSPFv3.

**Comandos principais:** `ipv6 unicast-routing`, `ipv6 address`, `ipv6 enable`, `ipv6 router ospf 1`, `ipv6 ospf 1 area`, `show ipv6 interface brief`, `show ipv6 route`

---

### Lab 9 — Automação e SDN
Estudo de SDN — separação de control plane e data plane, conceitos de underlay, overlay e fabric. REST API com verbos GET, POST, PUT e DELETE. Simulador interativo de API construído sobre os dados reais da rede. YANG, NETCONF e RESTCONF. Ansible e Terraform.

---

### Lab 10 — Revisão final e simulado
Checklist completo da rede com verificação de OSPF, DHCP, NAT, EtherChannel, IPv6 e VoIP. Ligação VoIP entre matriz e filial validada. Simulado de 30 questões no estilo CCNA 200-301 com resultado de 80% de acerto.

---

## Troubleshooting real documentado

**Lab 7 — IP Phone travado em loop entre "Configuring IP" e "Configuring CM List"**
O MAC address exibido no modo de simulação do Packet Tracer (`0002.1618.A602`) era diferente do MAC real da interface de rede do telefone (`00D0.BC2C.5519`). O ephone foi configurado com o MAC errado, impedindo o registro. Solução: capturar o MAC correto via `show mac address-table interface Fa0/2` no switch após o phone se conectar fisicamente.

**Lab 7 — Port Security bloqueando IP Phones e APs**
Após configurar o port security com sticky MAC nas portas de acesso, os IP Phones e Access Points foram bloqueados porque seus MACs não estavam na lista de MACs autorizados. Solução: desativar o port security nas portas de dispositivos VoIP e wireless, ou aumentar o `maximum` para incluir todos os MACs esperados.

**Lab 6 — ACL em SVI não funcional no Packet Tracer**
A ACL estendida aplicada na SVI Vlan20 do SW-CORE não filtrou o tráfego conforme esperado — limitação conhecida do simulador. No IOS real o SW-CORE 3560 suportaria plenamente ACLs em SVIs. Workaround: ACL aplicada na interface Gig0/2 do R1.

**Lab 4 — Interface serial com nome diferente do esperado**
O módulo HWIC inserido no R1 criou interfaces do tipo FastEthernet em modo Layer 2 (HWIC-4ESW) em vez de interfaces seriais roteáveis. Solução: substituir o módulo por HWIC-2T, que cria interfaces Serial0/1/0 e Serial0/1/1 compatíveis com links WAN.

---

## Observações de arquitetura

Em ambiente de produção, recomenda-se link redundante para a filial (segundo link R2 ↔ R3) e dual-homing no ISP (segundo link de R2 para um ISP alternativo). Ambas as simplificações foram feitas intencionalmente para manter o foco no escopo da CCNA 200-301.

---

## Comandos de verificação por serviço

```
show ip ospf neighbor              — adjacências OSPF
show ip route                      — tabela de roteamento
show ip dhcp binding               — IPs concedidos por DHCP
show ip nat translations           — traduções NAT ativas
show etherchannel summary          — estado do EtherChannel
show vlan brief                    — VLANs e portas associadas
show interfaces trunk              — trunks ativos e VLANs permitidas
show spanning-tree vlan X          — estado do STP por VLAN
show port-security interface Fa0/X — estado do port security
show ipv6 ospf neighbor            — adjacências OSPFv3
show ephone                        — registro dos IP Phones
show standby brief                 — estado do HSRP
show ip dhcp snooping              — estado do DHCP Snooping
show ip arp inspection             — estado do DAI
```

---

## Sobre o autor

Supervisor de TIC na Minsait, atuando na Petrobras REFAP. Tecnólogo em Segurança da Informação pela Uniasselvi. Certificações: ITIL V4, HDI, NSE1/NSE2/NSE3, CCST Networking.


[Upload# CCNA 200-301 Project — Hierarchical Corporate Network in Cisco Packet Tracer

Third cumulative project for CCNA 200-301 exam preparation, built in Cisco Packet Tracer using an incremental approach — each lab builds on the previous one, evolving from address planning into a fully operational corporate network with headquarters and branch sites.

---

## About the project

The goal was to build from scratch a realistic corporate network following the three-tier hierarchical model (Core, Distribution, Access). Two sites — headquarters and branch — connected via a serial WAN link, running services such as IP telephony, wireless, centralized DHCP, NAT, ACL-based security, and dynamic routing with multi-area OSPF.

Each lab was developed incrementally, preserving all previous configurations while adding new services and features.

---

## Topology

```
Internet (ISP)
      |
   R1 (2911/HQ) ── Serial ── R3 (2911/Branch)
   |        |                      |
 ISP      R2 (2911)           SW-BRANCH (2960)
           |                   |          |
        SW-CORE (3560 L3)   PCs 8~13   AP-BRANCH
        |          |                       |
     SW-DIST1   SW-DIST2            Laptop-Branch
     (2960)     (2960)
      |    ╚══════╝    |   ← EtherChannel LACP Po1
   SW-ACC1         SW-ACC2
   (2960)          (2960)
      |                |
   PCs 0~3          PCs 4~7
   IP Phone HQ      Server0
                    Printer0
                    Access Point0
                    Laptop1
```

### Equipment

| Device | Model | Role |
|---|---|---|
| R1 | Cisco 2911 | HQ gateway — OSPF, NAT, DHCP |
| R2 | Cisco 2911 | Internal redundancy — HSRP |
| R3 | Cisco 2911 | Branch gateway — Router-on-a-Stick |
| ISP | Cisco 2911 | Internet simulation |
| R-Telephony | Cisco 2811 | IP telephony server |
| SW-CORE | Cisco 3560 L3 | Core — inter-VLAN routing via SVIs |
| SW-DIST1/2 | Cisco 2960 | Distribution — EtherChannel LACP |
| SW-ACC1/2 | Cisco 2960 | Access — end device connectivity |
| SW-BRANCH | Cisco 2960 | Branch switch |

### VLANs

| VLAN | Name | Network | Devices |
|---|---|---|---|
| 10 | TI-MATRIZ | 10.10.10.0/24 | ACC1 PCs, IP Phone HQ |
| 20 | RH-MATRIZ | 10.10.20.0/24 | ACC2 PCs, Server0, Printer0, AP0, Laptop1 |
| 30 | TI-FILIAL | 10.20.10.0/24 | Branch PCs, IP Phone Branch |
| 40 | RH-FILIAL | 10.20.20.0/24 | AP-BRANCH, Laptop-Branch |
| 99 | VOIP | 10.99.20.0/24 | IP Phones |
| 100 | MANAGEMENT | 10.99.10.0/24 | Switches |

### IP Addressing — infrastructure links

| Segment | Network | Devices |
|---|---|---|
| R1 ↔ R2 | 10.99.1.0/30 | R1 Gig0/1 ↔ R2 Gig0/0 |
| R2 ↔ SW-CORE | 10.99.2.0/30 | R2 Gig0/1 ↔ SW-CORE Gig0/2 |
| R1 ↔ R3 | 10.99.3.0/30 | R1 Serial0/1/0 ↔ R3 Serial0/0/0 |
| R1 ↔ ISP | 10.99.4.0/30 | R1 Gig0/0 ↔ ISP Gig0/0 |
| R1 ↔ SW-CORE | 10.99.5.0/30 | R1 Gig0/2 ↔ SW-CORE Gig0/1 |
| Management | 10.99.10.0/24 | Switch SVIs |
| VoIP | 10.99.20.0/24 | IP Phones |

---

## Labs

### Lab 1 — Fundamentals and subnetting
Full topology planning before any device configuration. All networks defined using VLSM, complete addressing table built, and basic IPv6 introduced.

**Concepts covered:** VLSM, OSI and TCP/IP models, TCP vs UDP, private vs public IPv4, network topologies.

---

### Lab 2 — Basic configuration and SSH
Initial security setup on all network devices. SSH version 2 enabled with a 1024-bit RSA key. CDP and LLDP verified. Unnecessary services disabled.

**Key commands:** `ip domain-name`, `crypto key generate rsa`, `ip ssh version 2`, `transport input ssh`, `username admin privilege 15 secret`, `lldp run`, `no cdp run`

---

### Lab 3 — VLANs, trunking and EtherChannel
VLANs created on all switches with native VLAN 100 on all trunks. LACP EtherChannel between SW-DIST1 and SW-DIST2. Manual Root Bridge election on SW-CORE. PortFast and BPDU Guard on all access ports. Port Security with sticky MAC.

**Key commands:** `vlan 10`, `switchport trunk encapsulation dot1q`, `switchport trunk native vlan 100`, `channel-group 1 mode active`, `spanning-tree vlan 10 root primary`, `spanning-tree portfast`, `spanning-tree bpduguard enable`, `switchport port-security mac-address sticky`

---

### Lab 4 — OSPF routing and inter-VLAN
IPs assigned to all routers and SW-CORE. SVIs for inter-VLAN routing. OSPF area 0 between R1, R2, and SW-CORE. OSPF area 1 for the branch via R3. Default route redistributed into OSPF. HSRP between R1 and R2 with virtual IP 10.99.1.3.

**Key commands:** `ip routing`, `router ospf 1`, `router-id`, `network x.x.x.x wildcard area`, `default-information originate`, `standby 1 ip`, `standby 1 priority`, `standby 1 preempt`

---

### Lab 5 — IP services — DHCP, NAT, NTP, Syslog
Centralized DHCP on R1 with per-VLAN pools and relay configured on SW-CORE and R3. Static NAT for Server0 and PAT for internet access. NTP with R1 as stratum 1. Centralized Syslog on Server0.

**Key commands:** `ip dhcp excluded-address`, `ip dhcp pool`, `option 150 ip`, `ip helper-address`, `ip nat inside source static`, `ip nat inside source list overload`, `ntp master 1`, `ntp server`, `logging`

---

### Lab 6 — Security — ACLs and layer 2
Extended ACL controlling access to Server0: VLAN 10 (IT) and branch allowed, VLAN 20 (HR) blocked. DHCP Snooping and Dynamic ARP Inspection configured on access switches to protect against layer 2 attacks.

**Documented limitation:** Extended ACLs applied to SVIs on L3 switches behave inconsistently in Packet Tracer — on real IOS this would work as expected. Workaround applied on R1's physical interface.

**Key commands:** `ip access-list extended`, `permit ip`, `deny ip`, `ip access-group NAME in`, `ip dhcp snooping`, `ip dhcp snooping vlan`, `ip arp inspection vlan`

---

### Lab 7 — Wireless and VoIP
Cisco 2811 R-Telephony configured as telephony server. HQ IP Phone registered on extension 1001, branch IP Phone on extension 1002. Successful call placed between the two sites. Branch AP configured with SSID WIFI-FILIAL WPA2-PSK on channel 1. HQ AP configured with SSID WIFI-MATRIZ WPA2-PSK on channel 11.

**Critical lesson learned:** The real MAC address of the IP Phone differs from the MAC shown in Packet Tracer's simulation mode. The correct MAC must be retrieved via `show mac address-table interface Fa0/X` on the connected switch. The `button 1:1` command inside the ephone block is mandatory — without it the phone loops indefinitely.

**Key commands:** `telephony-service`, `ip source-address`, `max-ephones`, `ephone-dn`, `number`, `ephone`, `mac-address`, `button 1:1`, `auto assign`

---

### Lab 8 — IPv6
Global unicast IPv6 addresses configured on R1, R2, and R3 with dual-stack IPv4 + IPv6. OSPFv3 area 0 and area 1 established with FULL adjacencies. IPv6 ping between R1 and R3 at 100% success.

**Concepts covered:** 128-bit format, global unicast vs link-local, EUI-64, NDP as ARP replacement, OSPFv2 vs OSPFv3 differences.

**Key commands:** `ipv6 unicast-routing`, `ipv6 address`, `ipv6 enable`, `ipv6 router ospf 1`, `ipv6 ospf 1 area`, `show ipv6 interface brief`, `show ipv6 route`

---

### Lab 9 — Automation and SDN
SDN fundamentals — control plane vs data plane separation, underlay, overlay, and fabric concepts. REST API with GET, POST, PUT, and DELETE verbs. Interactive API simulator built on top of the actual network data. YANG, NETCONF, and RESTCONF overview. Ansible and Terraform concepts.

---

### Lab 10 — Final review and mock exam
Full network checklist verifying OSPF, DHCP, NAT, EtherChannel, IPv6, and VoIP. Live VoIP call between HQ and branch validated. 30-question CCNA 200-301 style mock exam completed with 80% score.

---

## Real troubleshooting documented

**Lab 7 — IP Phone stuck in loop between "Configuring IP" and "Configuring CM List"**
The MAC address displayed in Packet Tracer's simulation mode (`0002.1618.A602`) was different from the actual NIC MAC of the phone (`00D0.BC2C.5519`). The ephone was configured with the wrong MAC, preventing registration. Fix: retrieve the correct MAC using `show mac address-table interface Fa0/2` on the switch after the phone physically connects.

**Lab 7 — Port Security blocking IP Phones and APs**
After configuring sticky MAC port security on access ports, IP Phones and Access Points were blocked because their MACs were not in the authorized list. Fix: disable port security on VoIP and wireless device ports, or increase the `maximum` to accommodate all expected MACs.

**Lab 6 — ACL on SVI not working in Packet Tracer**
The extended ACL applied to SVI Vlan20 on SW-CORE did not filter traffic as expected — a known Packet Tracer limitation. On real IOS, the 3560 SW-CORE would fully support ACLs on SVIs. Workaround: ACL applied to R1's Gig0/2 interface instead.

**Lab 4 — Serial interface with unexpected name**
The HWIC module inserted in R1 created Layer 2 FastEthernet interfaces (HWIC-4ESW) instead of routable serial interfaces. Fix: replace the module with HWIC-2T, which creates Serial0/1/0 and Serial0/1/1 interfaces compatible with WAN links.

---

## Architecture notes

In a production environment, a redundant branch link (second R2 ↔ R3 path) and ISP dual-homing (second uplink from R2 to an alternate ISP) would be strongly recommended. Both simplifications were made intentionally to keep the scope aligned with CCNA 200-301 objectives.

---

## Verification commands by service

```
show ip ospf neighbor              — OSPF adjacencies
show ip route                      — routing table
show ip dhcp binding               — active DHCP leases
show ip nat translations           — active NAT entries
show etherchannel summary          — EtherChannel state
show vlan brief                    — VLANs and associated ports
show interfaces trunk              — active trunks and allowed VLANs
show spanning-tree vlan X          — STP state per VLAN
show port-security interface Fa0/X — port security state
show ipv6 ospf neighbor            — OSPFv3 adjacencies
show ephone                        — IP Phone registration state
show standby brief                 — HSRP state
show ip dhcp snooping              — DHCP Snooping state
show ip arp inspection             — DAI state
```

---

## About the author

IT Supervisor at Minsait, working on-site at Petrobras REFAP. Bachelor's in Information Security from Uniasselvi. Certifications: ITIL V4, HDI, NSE1/NSE2/NSE3, CCST Networking.

Preparing for CCNA 200-301 (July 2026) and Fortinet NSE4 (December 2026). Goal: specialization in firewall management and network architecture.

*Studying with professor Gustavo Kalau on YouTube.*
ing README-en.md…]()


Preparação para CCNA 200-301 (julho/2026) e NSE4 Fortinet (dezembro/2026). Objetivo: especialização em firewall e arquitetura de redes.

*Estudando com o professor Gustavo Kalau no YouTube.*
