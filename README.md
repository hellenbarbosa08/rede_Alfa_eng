# Rede Corporativa — Alfa Engenharia

**Laboratório prático de redes, controle de acesso e administração básica de servidores Linux.**

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet_Tracer-049FD9?style=flat-square)
![Ubuntu](https://img.shields.io/badge/Ubuntu-WSL_2-E95420?style=flat-square)
![Zabbix](https://img.shields.io/badge/Zabbix-Agent_2-D40000?style=flat-square)
![Tipo](https://img.shields.io/badge/Tipo-Laborat%C3%B3rio_de_estudo-64748B?style=flat-square)

## Visão geral

A **Alfa Engenharia** é uma empresa fictícia utilizada como cenário para estudar como uma rede corporativa é organizada, protegida e diagnosticada.

O projeto reúne **dois laboratórios complementares**:

| Laboratório | Ambiente | O que foi praticado |
|---|---|---|
| **[Rede corporativa](packet-tracer/LAB_01_ALFA.pkt)** | Cisco Packet Tracer | VLANs, VLSM, DHCP, roteamento, NAT/PAT, ACLs, FTP e SSH |
| **[Linux e monitoramento](linux-monitoramento/README.md)** | Ubuntu no WSL 2 | Nginx, incidentes e métricas locais com Zabbix Agent 2 (documentação na própria pasta) |

> **Escopo:** os laboratórios são independentes. O Ubuntu no WSL 2 **não está conectado à topologia simulada** no Packet Tracer, e o Zabbix Agent 2 ainda não envia dados a um Zabbix Server com dashboard e alertas.

## O problema que o laboratório simula

Uma empresa precisa separar seus departamentos, distribuir endereços IP, permitir comunicação autorizada, oferecer serviços de rede e possibilitar manutenção segura dos equipamentos. Além disso, é necessário saber identificar e recuperar falhas em serviços hospedados em Linux.

Em vez de apenas configurar dispositivos, este projeto procura responder a três perguntas:

1. **A rede funciona?** — testar endereçamento, roteamento e acesso externo simulado.
2. **Cada setor acessa somente o que precisa?** — implementar e validar restrições com ACLs e autenticação FTP.
3. **Como diagnosticar uma indisponibilidade?** — verificar serviços, portas, respostas HTTP, logs e métricas locais.

---

## 1. Rede corporativa — Cisco Packet Tracer

### Arquitetura simplificada

```text
             Servidor externo (203.0.113.10)
                          |
                 Roteador ISP (.2)
                          |
                  WAN 200.0.0.0/30
                          |
                   R-ALFA (.1)
                 Router-on-a-Stick
                          |
                     Trunk 802.1Q
                          |
                      SW-CENTRAL
          ________________|________________
         |           |           |         |
      VLAN 10     VLAN 20     VLAN 30   VLAN 40
        ADM       Financeiro      TI    Recursos
                                        |
                                    SW-RECURSOS
                                        |
                          DHCP (.66) / Impressora (.67)
                          FTP Gerais (.68) / FTP Financeiro (.69)
```

O roteador `R-ALFA` realiza o roteamento entre as VLANs por meio de subinterfaces e também encaminha o tráfego para um **ISP simulado**, com rota padrão e NAT/PAT.

### Endereçamento IPv4 com VLSM

Foi utilizado o bloco privado `192.168.10.0/24`, dividido em sub-redes de diferentes tamanhos:

| VLAN | Setor | Sub-rede | Gateway |
|---|---|---|---|
| 10 | Administrativo | `192.168.10.0/27` | `192.168.10.1` |
| 20 | Financeiro | `192.168.10.32/28` | `192.168.10.33` |
| 30 | TI | `192.168.10.48/28` | `192.168.10.49` |
| 40 | Recursos e servidores | `192.168.10.64/29` | `192.168.10.65` |

**Por que VLSM?** Para dimensionar cada sub-rede conforme a quantidade de dispositivos, evitando desperdício de endereços IPv4.

### Funcionalidades implementadas

- **VLANs e trunk 802.1Q:** separação lógica dos departamentos e transporte das VLANs pelo enlace com o roteador.
- **Router-on-a-Stick:** comunicação entre sub-redes por meio das subinterfaces do `R-ALFA`.
- **DHCP centralizado e DHCP Relay:** distribuição de endereços usando o servidor `192.168.10.66` e `ip helper-address`.
- **Rota padrão e NAT/PAT:** comunicação entre a rede privada e o servidor externo simulado `203.0.113.10`.
- **ACLs:** restrições de tráfego entre setores e servidores.
- **FTP autenticado:** contas com permissões de leitura e listagem nos servidores de arquivos simulados.
- **SSH:** acesso administrativo remoto ao roteador, com autenticação e restrição de origem para a VLAN da TI.

### Política de acesso aos arquivos

Foram adicionados dois servidores FTP na VLAN 40:

| Servidor | Endereço IP | Finalidade |
|---|---|---|
| `SRV-GERAIS` | `192.168.10.68` | Arquivos gerais para Administrativo e TI |
| `SRV-FINANCEIRO` | `192.168.10.69` | Arquivos do setor Financeiro |

A política implementada no laboratório é:

| Origem | SRV-GERAIS | SRV-FINANCEIRO |
|---|---|---|
| Administrativo | Acesso permitido, mediante autenticação | Tráfego IP bloqueado |
| Financeiro | Tráfego IP bloqueado | Acesso permitido, mediante autenticação |
| TI | Acesso permitido, mediante autenticação | Conexão FTP pela porta TCP 21 bloqueada |

**Como isso funciona?** As ACLs `CL_ADM`, `CL_FIN` e `CL_TI` são avaliadas no roteador. Já o serviço FTP solicita usuário e senha e aplica permissões como **Read** e **List**. A ACL `SSH_TI` restringe a origem das conexões administrativas ao roteador.

> **Limitação importante:** o bloqueio de TI para `SRV-FINANCEIRO` foi configurado especificamente para o controle FTP na porta TCP 21. O ping para esse servidor continua permitido. Portanto, esse teste **não equivale a bloquear todos os possíveis protocolos de acesso a arquivos**. O FTP tradicional também não é indicado para documentos confidenciais em produção, pois não criptografa credenciais e dados.

### Administração remota por SSH

No roteador `R-ALFA`, foi configurado SSH v2, uma conta local de manutenção e uma ACL de acesso às linhas VTY (`SSH_TI`).

**Testes realizados:**

- **PC da TI → SSH no roteador:** autenticação e acesso concluídos.
- **PC Administrativo → SSH no roteador:** conexão bloqueada.

O teste demonstra administração remota **do roteador**. A configuração de SSH nos demais switches não faz parte desta versão.

### Conectividade e NAT/PAT — evidências

**Comunicação com servidor externo simulado:** teste de ping a `203.0.113.10` concluído com 4 respostas e 0% de perda na evidência registrada.

![Ping para servidor externo simulado](evidencias/pingExterno.png)

**Tradução de endereços:** o comando `show ip nat translations` mostrou traduções do tráfego de um host interno, como `192.168.10.7`, para o endereço externo `200.0.0.1`.

![Traduções NAT/PAT](evidencias/traducaoNat.png)

Outras evidências: [tabela de roteamento](evidencias/roteamento.png) · [estatísticas NAT](evidencias/nat.png).

---

## 2. Laboratório Linux e monitoramento — documentação independente

O projeto também inclui um laboratório prático com **Ubuntu no WSL 2, Nginx e Zabbix Agent 2**. Nele foram realizados testes de disponibilidade HTTP, consulta de métricas locais e simulação controlada de uma falha do Nginx, seguida de recuperação.

Para evitar duplicação, os **comandos, explicações, capturas de tela e resultados** estão documentados no README próprio dessa etapa:

### **[Acessar o laboratório Linux e monitoramento →](linux-monitoramento/README.md)**

> **Escopo:** este laboratório funciona localmente no Ubuntu/WSL 2 e não está integrado à topologia do Cisco Packet Tracer. O Zabbix Agent 2 foi utilizado em testes locais; ainda não há dashboard, histórico centralizado ou alertas automáticos.

---

## 3. Como explorar o projeto

### Estrutura do repositório

```text
rede_Alfa_eng/
├── README.md                         # Visão geral
├── packet-tracer/
│   └── LAB_01_ALFA.pkt              # Topologia e configurações simuladas
├── evidencias/                      # Testes de rede e NAT/PAT
│   ├── pingExterno.png
│   ├── roteamento.png
│   ├── traducaoNat.png
│   └── nat.png
├── linux-monitoramento/
│   ├── README.md                     # Documentação detalhada do laboratório Linux
│   └── evidencias/                  # Cinco capturas do Nginx e Zabbix Agent 2
└── docs/                             # Rascunhos de documentação
```

### Abrir o laboratório de redes

1. Baixe ou clone este repositório.
2. Abra [`packet-tracer/LAB_01_ALFA.pkt`](packet-tracer/LAB_01_ALFA.pkt) em uma versão compatível do Cisco Packet Tracer.
3. Explore o roteador, os switches, as VLANs e os servidores FTP.
4. Reproduza os testes de conectividade e controle de acesso descritos acima.

```bash
git clone https://github.com/hellenbarbosa08/rede_Alfa_eng.git
```

### Comandos úteis para verificar a rede

| Comando | O que verificar |
|---|---|
| `show ip interface brief` | Interfaces, endereços e estado operacional |
| `show vlan brief` | VLANs e portas de acesso |
| `show interfaces trunk` | Portas trunk e VLANs transportadas |
| `show ip route` | Rotas e próximo salto para a rede externa |
| `show access-lists` | Regras de ACL e seus contadores |
| `show ip interface GigabitEthernet0/0/0.30` | ACL aplicada à interface da TI |
| `show ip nat translations` | Traduções de endereços observadas |
| `show ip ssh` | Estado e versão do SSH do roteador |
| `ping` / `ftp` / `ssh` | Conectividade, autenticação FTP e acesso remoto |

## 4. Aprendizados e limites

**Competências praticadas:** planejamento IPv4 com VLSM; VLANs e roteamento; DHCP Relay; NAT/PAT; ACLs; autenticação FTP; SSH em roteador; gerenciamento básico de serviços Linux; diagnóstico de HTTP e logs; consulta de métricas com Zabbix Agent 2.

**O que ainda não foi implementado:** integração entre Packet Tracer e WSL 2, monitoramento contínuo pelo Zabbix Server, dashboard, alertas automáticos e administração por SSH de todos os dispositivos da topologia.

O objetivo é manter um laboratório **reproduzível e explicável**, sem apresentá-lo como uma infraestrutura de produção.

## Referências

- [Cisco Networking Academy](https://www.netacad.com/)
- [Documentação Cisco](https://www.cisco.com/c/en/us/support/index.html)
- [Documentação Ubuntu Server](https://ubuntu.com/server/docs)
- [Documentação Zabbix](https://www.zabbix.com/documentation/current/pt/manual)

---

**Autoria:** Hellen Neves Barbosa · Engenharia da Computação — UFMA · [GitHub](https://github.com/hellenbarbosa08)

*Projeto pessoal desenvolvido para estudo e demonstração prática de fundamentos de redes e infraestrutura de TI.*


---

*Projeto desenvolvido para fins acadêmicos e de aprendizado prático em redes e infraestrutura de TI.*

