# 📋 Requisitos do Projeto — Rede Alfa Engenharia

Este documento reúne os requisitos funcionais, técnicos e de segurança da rede corporativa simulada da **Alfa Engenharia** no Cisco Packet Tracer.

> **Fonte de verdade desta versão:** este documento está alinhado com a implementação atualmente descrita no `README.md` do projeto.

---

## 1. Contexto

A Alfa Engenharia é uma empresa fictícia composta por:

- Administrativo;
- Financeiro;
- Tecnologia da Informação (TI).

A infraestrutura também possui uma VLAN dedicada a recursos e serviços internos.

---

## 2. Dimensionamento

| Setor | Hosts atuais | Planejamento futuro |
|---|---:|---:|
| Administrativo | 8 | 15 |
| Financeiro | 4 | 8 |
| TI | 3 | 6 |
| Recursos | 4 | Expansível |

---

## 3. Endereçamento IPv4

### 3.1 Bloco interno

```text
192.168.10.0/24
```

### 3.2 Sub-redes

| VLAN | Setor | Rede | Máscara | Gateway |
|---:|---|---|---|---|
| 10 | Administrativo | `192.168.10.0/27` | `255.255.255.224` | `192.168.10.1` |
| 20 | Financeiro | `192.168.10.32/28` | `255.255.255.240` | `192.168.10.33` |
| 30 | TI | `192.168.10.48/28` | `255.255.255.240` | `192.168.10.49` |
| 40 | Recursos | `192.168.10.64/29` | `255.255.255.248` | `192.168.10.65` |

### 3.3 Recursos com IP estático

| Recurso | IP | Função |
|---|---|---|
| DHCP | `192.168.10.66/29` | Distribuição de endereços |
| Impressora | `192.168.10.67/29` | Impressão em rede |
| `SRV-GERAIS` | `192.168.10.68/29` | Servidor FTP de arquivos gerais |
| `SRV-FINANCEIRO` | `192.168.10.69/29` | Servidor FTP do Financeiro |

---

## 4. Segmentação

A rede deve ser segmentada em VLANs independentes.

As portas de usuários devem operar em modo **Access**.

Os enlaces entre switches e o enlace com o roteador devem operar em modo **Trunk IEEE 802.1Q**.

---

## 5. Roteamento inter-VLAN

O roteamento entre VLANs deve ser realizado pelo `R-ALFA` utilizando **Router-on-a-Stick**.

Cada subinterface deve funcionar como gateway da respectiva VLAN.

---

## 6. DHCP

O servidor DHCP deve:

- possuir IP estático `192.168.10.66`;
- permanecer na VLAN 40;
- atender clientes das VLANs 10, 20 e 30;
- entregar IP, máscara, gateway e DNS.

As subinterfaces das VLANs de usuários devem utilizar **DHCP Relay** com `ip helper-address`.

---

## 7. Comunicação entre setores

| Origem | ADM | Financeiro | TI | Recursos |
|---|---:|---:|---:|---:|
| **ADM** | ✅ | ❌ | ✅ | ⚠️ Controlado |
| **Financeiro** | ❌ | ✅ | ✅ | ⚠️ Controlado |
| **TI** | ✅ | ✅ | ✅ | ⚠️ Controlado |

### Regras principais

- ADM não deve iniciar comunicação com a rede Financeira.
- Financeiro não deve iniciar comunicação com a rede Administrativa.
- TI deve conseguir alcançar ADM e Financeiro para suporte.
- O acesso à VLAN 40 deve ser refinado por destino/serviço através de ACLs.

---

## 8. Política de acesso aos servidores FTP

Nesta versão existem **dois servidores FTP separados**, e não duas pastas hospedadas no mesmo servidor.

| Origem | `SRV-GERAIS` `.68` | `SRV-FINANCEIRO` `.69` |
|---|---|---|
| Administrativo | ✅ FTP autenticado | ❌ Tráfego IP bloqueado |
| Financeiro | ❌ Tráfego IP bloqueado | ✅ FTP autenticado |
| TI | ✅ FTP autenticado | ⚠️ FTP TCP/21 bloqueado; ICMP permitido |

### Observação

O bloqueio aplicado ao TI em `SRV-FINANCEIRO` é específico para a porta TCP 21. Portanto, o servidor pode continuar respondendo a outros protocolos, como ICMP, conforme a política implementada.

---

## 9. Administração remota

O roteador `R-ALFA` deve permitir administração remota por **SSH v2**.

Política:

- TI → SSH no roteador: ✅ permitido;
- Administrativo → SSH no roteador: ❌ bloqueado.

A restrição de origem deve ser aplicada às linhas VTY.

---

## 10. Conectividade externa

A arquitetura externa deve representar:

```text
Rede Alfa
   |
R-ALFA
   |
WAN
   |
Router ISP
   |
Servidor externo
```

### WAN

```text
200.0.0.0/30
```

- `R-ALFA`: `200.0.0.1`
- ISP: `200.0.0.2`

### Rede externa simulada

```text
203.0.113.0/24
```

Servidor externo:

```text
203.0.113.10
```

---

## 11. Rota padrão

O `R-ALFA` deve encaminhar destinos desconhecidos para o ISP através de uma rota padrão.

Próximo salto:

```text
200.0.0.2
```

---

## 12. NAT/PAT

Os endereços privados `192.168.10.x` devem ser traduzidos no `R-ALFA` para permitir acesso à rede externa.

O laboratório deve utilizar **PAT / NAT Overload**, permitindo que múltiplos hosts compartilhem o endereço externo da interface WAN.

Fluxo:

```text
192.168.10.x
      ↓
R-ALFA
      ↓ PAT
200.0.0.1
      ↓
ISP
      ↓
203.0.113.10
```

---

## 13. Segurança

A solução deve demonstrar:

- segmentação por VLAN;
- isolamento entre ADM e Financeiro;
- ACLs estendidas;
- filtragem por destino e protocolo quando necessário;
- administração por SSH restrita à TI;
- autenticação no FTP;
- NAT/PAT na borda.

> FTP tradicional é utilizado apenas para fins didáticos no Packet Tracer e não deve ser tratado como solução segura para arquivos confidenciais em produção.

---

## 14. Critérios de validação

### DHCP

- [x] ADM recebe IP por DHCP.
- [x] Financeiro recebe IP por DHCP.
- [x] TI recebe IP por DHCP.

### Inter-VLAN

- [x] Gateways respondem.
- [x] TI alcança ADM.
- [x] TI alcança Financeiro.
- [x] ADM → Financeiro é bloqueado.
- [x] Financeiro → ADM é bloqueado.

### FTP

- [x] ADM acessa `SRV-GERAIS` mediante autenticação.
- [x] ADM é bloqueado em `SRV-FINANCEIRO`.
- [x] Financeiro acessa `SRV-FINANCEIRO` mediante autenticação.
- [x] Financeiro é bloqueado em `SRV-GERAIS`.
- [x] TI acessa `SRV-GERAIS`.
- [x] TI tem FTP TCP/21 bloqueado em `SRV-FINANCEIRO`.

### SSH

- [x] TI acessa o `R-ALFA` por SSH.
- [x] ADM não acessa o `R-ALFA` por SSH.

### Rede externa

- [x] R-ALFA alcança ISP.
- [x] ISP alcança servidor externo.
- [x] Clientes internos alcançam `203.0.113.10`.

### NAT/PAT

- [x] Traduções são exibidas em `show ip nat translations`.
- [x] Hosts internos compartilham o IP externo através de PAT.

---

## 15. Comandos de diagnóstico

```text
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
show access-lists
show ip nat translations
show ip nat statistics
show ip ssh
ping
ftp
ssh
ipconfig
```

---

## 16. Estrutura documental

```text
docs/
├── requisitos.md
├── enderecamento.md
├── matriz-comunicacao.md
└── troubleshooting.md
```

O README deve apresentar apenas a visão geral e apontar para esses documentos detalhados.
