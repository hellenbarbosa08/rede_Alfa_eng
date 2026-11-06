# 🏢 Rede Corporativa — Alfa Engenharia

Projeto prático de infraestrutura de redes desenvolvido no **Cisco Packet Tracer**, simulando a rede de uma pequena empresa com segmentação por departamentos, endereçamento IPv4 com VLSM, VLANs, roteamento inter-VLAN, DHCP centralizado e controle de acesso com ACLs.

> **Status:** 🚧 Em desenvolvimento

---

## 📌 Sobre o projeto

A **Alfa Engenharia** é uma empresa fictícia composta por três setores:

- Administrativo;
- Financeiro;
- Tecnologia da Informação (TI).

A empresa também possui recursos compartilhados:

- servidor;
- impressora de rede.

O objetivo do projeto é desenvolver uma infraestrutura capaz de separar logicamente os departamentos, permitir crescimento futuro, centralizar serviços e controlar a comunicação entre as diferentes redes.

O laboratório está sendo desenvolvido de forma incremental, partindo dos requisitos da empresa até a implementação, validação e troubleshooting da infraestrutura.

---

## 🎯 Objetivos

Este projeto busca aplicar, na prática, conceitos fundamentais de redes e infraestrutura:

- planejamento de endereçamento IPv4;
- subnetting;
- VLSM;
- VLANs;
- portas Access;
- trunks IEEE 802.1Q;
- Router-on-a-Stick;
- roteamento inter-VLAN;
- DHCP centralizado;
- DHCP Relay;
- ACLs;
- testes de conectividade;
- troubleshooting;
- documentação técnica.

---

## 🏗️ Cenário da empresa

| Setor | Hosts atuais | Planejamento futuro |
|---|---:|---:|
| Administrativo | 8 | 15 |
| Financeiro | 4 | 8 |
| TI | 3 | 6 |
| Recursos | 2 | Expansível |

A rede de Recursos possui inicialmente:

- 1 servidor;
- 1 impressora de rede.

---

## 🌐 Topologia

A rede utiliza uma topologia hierárquica em estrela.

```text
                         Router
                            |
                     Trunk 802.1Q
                            |
                       SW-Central
             _______________|_______________
            /               |               |               \
        SW-ADM           SW-FIN           SW-TI        SW-RECURSOS
          |                 |               |                |
         PCs               PCs             PCs       Servidor + Impressora
```

O switch central concentra as conexões dos switches dos departamentos e do roteador.

Os enlaces entre switches e entre o switch central e o roteador utilizam **trunks 802.1Q** para transportar múltiplas VLANs.

---

## 🧩 VLANs

Cada departamento foi colocado em uma VLAN própria.

| VLAN | Nome | Função |
|---:|---|---|
| 10 | ADM | Administrativo |
| 20 | FIN | Financeiro |
| 30 | TI | Tecnologia da Informação |
| 40 | RECURSOS | Servidor e impressora |

A segmentação reduz os domínios de broadcast e permite controlar a comunicação entre os diferentes setores.

### Tipos de porta utilizados

**Access**

Utilizada nas interfaces conectadas aos dispositivos finais.

```text
PC → Porta Access → VLAN específica
```

**Trunk**

Utilizada entre equipamentos de rede.

```text
Switch → Trunk 802.1Q → Switch/Router
```

O trunk permite transportar tráfego de múltiplas VLANs através do mesmo enlace físico.

---

## 📐 Plano de endereçamento IPv4

Foi utilizado o bloco privado:

```text
192.168.10.0/24
```

O bloco foi subdividido utilizando **VLSM — Variable Length Subnet Mask**, considerando a quantidade de dispositivos atual e o crescimento futuro.

| Rede | Sub-rede | Máscara | Hosts válidos | Broadcast | Gateway |
|---|---|---|---|---|---|
| ADM | `192.168.10.0/27` | `255.255.255.224` | `.1 - .30` | `.31` | `192.168.10.1` |
| Financeiro | `192.168.10.32/28` | `255.255.255.240` | `.33 - .46` | `.47` | `192.168.10.33` |
| TI | `192.168.10.48/28` | `255.255.255.240` | `.49 - .62` | `.63` | `192.168.10.49` |
| Recursos | `192.168.10.64/29` | `255.255.255.248` | `.65 - .70` | `.71` | `192.168.10.65` |

### Endereços estáticos

| Dispositivo | Endereço IP | Gateway |
|---|---|---|
| Servidor | `192.168.10.66/29` | `192.168.10.65` |
| Impressora | `192.168.10.67/29` | `192.168.10.65` |

---

## 🔀 Roteamento inter-VLAN

Como cada VLAN representa uma rede IP diferente, foi necessário implementar roteamento entre elas.

A solução utilizada foi **Router-on-a-Stick**.

A interface física do roteador é dividida em subinterfaces, cada uma associada a uma VLAN através do protocolo IEEE 802.1Q.

```text
G0/0/0
│
├── G0/0/0.10 → VLAN 10 → 192.168.10.1/27
├── G0/0/0.20 → VLAN 20 → 192.168.10.33/28
├── G0/0/0.30 → VLAN 30 → 192.168.10.49/28
└── G0/0/0.40 → VLAN 40 → 192.168.10.65/29
```

Exemplo de configuração:

```cisco
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.224
```

Cada subinterface funciona como o **gateway padrão** da respectiva VLAN.

---

## 📡 DHCP centralizado

Foi utilizado um servidor dedicado para distribuir automaticamente configurações IPv4 para os computadores.

Servidor DHCP:

```text
IP:      192.168.10.66
Rede:    192.168.10.64/29
VLAN:    40
Gateway: 192.168.10.65
```

Os computadores recebem automaticamente:

- endereço IPv4;
- máscara de sub-rede;
- gateway padrão;
- DNS.

---

## 🔁 DHCP Relay

O servidor DHCP está localizado em uma rede diferente das estações de trabalho.

Como broadcasts DHCP não são normalmente encaminhados por roteadores, foi utilizado **DHCP Relay** através do comando `ip helper-address`.

```cisco
interface GigabitEthernet0/0/0.10
 ip helper-address 192.168.10.66

interface GigabitEthernet0/0/0.20
 ip helper-address 192.168.10.66

interface GigabitEthernet0/0/0.30
 ip helper-address 192.168.10.66
```

Fluxo simplificado:

```text
Cliente
   |
   | DHCP Discover
   v
Gateway da VLAN
   |
   | ip helper-address
   v
Servidor DHCP
192.168.10.66
```

Assim, um único servidor DHCP consegue atender clientes localizados em diferentes sub-redes.

---

## 🔐 Controle de acesso com ACLs

Após validar que todas as VLANs conseguiam se comunicar corretamente, foram adicionadas **ACLs estendidas** para implementar políticas de segurança.

### Política atual

| Origem | Destino | Acesso |
|---|---|---|
| ADM | Financeiro | ❌ Bloqueado |
| Financeiro | ADM | ❌ Bloqueado |
| ADM | Recursos | ✅ Permitido |
| Financeiro | Recursos | ✅ Permitido |
| TI | ADM | ✅ Permitido |
| TI | Financeiro | ✅ Permitido |
| TI | Recursos | ✅ Permitido |

O setor de TI permanece com acesso às demais redes para possibilitar atividades de administração e suporte.

---

## 🔒 ACL — Administrativo

A rede Administrativa não pode iniciar comunicação com a rede Financeira.

```cisco
ip access-list extended CL_ADM
 deny ip 192.168.10.0 0.0.0.31 192.168.10.32 0.0.0.15
 permit ip any any
```

A ACL é aplicada na entrada da subinterface da VLAN 10:

```cisco
interface GigabitEthernet0/0/0.10
 ip access-group CL_ADM in
```

---

## 🔒 ACL — Financeiro

A rede Financeira também não pode iniciar comunicação com a rede Administrativa.

```cisco
ip access-list extended CL_FIN
 deny ip 192.168.10.32 0.0.0.15 192.168.10.0 0.0.0.31
 permit ip any any
```

Aplicação:

```cisco
interface GigabitEthernet0/0/0.20
 ip access-group CL_FIN in
```

---

## 📂 Recursos compartilhados

A VLAN 40 foi destinada aos recursos corporativos.

```text
Rede: 192.168.10.64/29
VLAN: 40
```

Recursos planejados:

```text
192.168.10.66 → Servidor
192.168.10.67 → Impressora
```

O servidor deverá disponibilizar:

```text
ARQUIVOS_GERAIS
ARQUIVOS_FINANCEIRO
```

Política planejada:

| Departamento | Arquivos Gerais | Arquivos Financeiros |
|---|---:|---:|
| Administrativo | ✅ | ❌ |
| Financeiro | ✅ | ✅ |
| TI | ✅ | ✅ |

> A diferenciação entre `ARQUIVOS_GERAIS` e `ARQUIVOS_FINANCEIRO` deve ser feita através de autenticação e permissões no servidor, e não apenas através de ACLs IP, pois os dois recursos estarão associados ao mesmo endereço de servidor.

---

## ✅ Testes realizados

Antes da aplicação das ACLs, toda a comunicação inter-VLAN foi validada.

| Teste | Resultado |
|---|---|
| ADM recebe IP via DHCP | ✅ |
| Financeiro recebe IP via DHCP | ✅ |
| TI recebe IP via DHCP | ✅ |
| ADM → Gateway | ✅ |
| Financeiro → Gateway | ✅ |
| TI → Gateway | ✅ |
| ADM → Servidor | ✅ |
| Financeiro → Servidor | ✅ |
| TI → Servidor | ✅ |
| Comunicação inter-VLAN antes das ACLs | ✅ |
| ADM → Financeiro após ACL | ❌ Bloqueado |
| Financeiro → ADM após ACL | ❌ Bloqueado |
| TI → ADM | ✅ |
| TI → Financeiro | ✅ |

Os principais testes de estação foram realizados com:

```bash
ipconfig
ping <endereço-ip>
```

Nos equipamentos Cisco foram utilizados:

```cisco
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
show access-lists
```

---

## 🛠️ Troubleshooting

Durante a implementação das ACLs ocorreu um problema em que o nome da ACL criada era diferente do nome inicialmente aplicado à interface.

A configuração foi investigada utilizando:

```cisco
show access-lists
```

Após identificar a diferença entre a ACL existente e aquela aplicada à subinterface, a configuração foi corrigida.

Os testes de `ping` confirmaram posteriormente que o tráfego entre Administrativo e Financeiro estava sendo bloqueado conforme planejado.

Esse cenário foi mantido como parte do aprendizado do projeto, demonstrando a importância de:

- validar configurações;
- interpretar resultados de testes;
- investigar o caminho dos pacotes;
- utilizar comandos de diagnóstico.

---

## 🧠 Conceitos aplicados

### Camada 2

- Ethernet;
- VLAN;
- portas Access;
- Trunk;
- IEEE 802.1Q;
- domínio de broadcast.

### Camada 3

- IPv4;
- subnetting;
- VLSM;
- default gateway;
- roteamento;
- Router-on-a-Stick;
- inter-VLAN routing.

### Serviços

- DHCP;
- DHCP Relay.

### Segurança

- ACL estendida;
- wildcard mask;
- filtragem inbound;
- implicit deny;
- first match;
- segmentação de rede.

---

## 📁 Estrutura do repositório

```text
rede_Alfa_eng/
│
├── README.md
│
├── docs/
│
└── packet-tracer/
    └── LAB_01_ALFA.pkt
```

O laboratório do Cisco Packet Tracer está disponível em:

[`packet-tracer/LAB_01_ALFA.pkt`](packet-tracer/LAB_01_ALFA.pkt)

---

## 🚧 Status

### Concluído

- [x] Levantamento dos requisitos
- [x] Planejamento IPv4
- [x] VLSM
- [x] VLANs
- [x] Portas Access
- [x] Trunks 802.1Q
- [x] Router-on-a-Stick
- [x] Roteamento inter-VLAN
- [x] DHCP Server
- [x] DHCP Relay
- [x] Testes de conectividade
- [x] ACL ADM → Financeiro
- [x] ACL Financeiro → ADM

### Próximas etapas

- [ ] Criar conexão WAN
- [ ] Simular ISP
- [ ] Configurar rota padrão
- [ ] Configurar NAT/PAT
- [ ] Testar acesso à rede externa
- [ ] Configurar serviços adicionais
- [ ] Implementar política de permissões do servidor
- [ ] Adicionar screenshots e evidências
- [ ] Exportar configurações dos dispositivos
- [ ] Criar cenários adicionais de troubleshooting
- [ ] Finalizar documentação

---

## 🗺️ Próxima evolução

A próxima etapa será adicionar conectividade externa:

```text
Rede Alfa
   |
   v
Router Alfa
   |
   | WAN
   v
Router ISP
   |
   v
Rede externa
```

Essa etapa será utilizada para estudar:

- rota padrão;
- redes WAN;
- NAT;
- PAT / NAT Overload;
- interfaces NAT Inside e Outside;
- endereços privados e públicos.

---

## 🧰 Tecnologias e ferramentas

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- VLSM
- VLAN
- IEEE 802.1Q
- Router-on-a-Stick
- DHCP
- DHCP Relay
- ACL
- Git
- GitHub

---

## 📚 Objetivo de aprendizado

Este laboratório faz parte de uma trilha prática de estudos 

Além de construir uma rede funcional, o objetivo é desenvolver capacidade de planejamento, implementação, documentação e troubleshooting de ambientes de infraestrutura.
