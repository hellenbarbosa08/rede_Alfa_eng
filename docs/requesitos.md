# 📋 Requisitos do Projeto — Rede Alfa Engenharia

Este documento reúne os requisitos funcionais, técnicos e de segurança definidos para o projeto de infraestrutura de rede da **Alfa Engenharia**.

O objetivo é servir como referência para o planejamento, implementação, testes e documentação do laboratório desenvolvido no **Cisco Packet Tracer**.

---

# 1. 🏢 Contexto

A Alfa Engenharia é uma empresa fictícia composta pelos seguintes setores:

- Administrativo;
- Financeiro;
- Tecnologia da Informação (TI).

A empresa também possui recursos compartilhados:

- servidor;
- impressora de rede.

A infraestrutura deve permitir comunicação controlada entre os setores, acesso aos recursos internos e conectividade com uma rede externa simulada.

---

# 2. 👥 Dimensionamento

A solução deve considerar a quantidade atual de dispositivos e a expansão futura de cada setor.

| Setor | Hosts atuais | Planejamento futuro |
|---|---:|---:|
| Administrativo | 8 | 15 |
| Financeiro | 4 | 8 |
| TI | 3 | 6 |
| Recursos Compartilhados | 2 | Expansível |

A solução de endereçamento deve suportar esse crescimento sem necessidade de reprojetar toda a rede.

---

# 3. 🌐 Requisitos de Endereçamento

## 3.1 Bloco principal

A rede interna deve utilizar o bloco:

```text
192.168.10.0/24
```

## 3.2 Sub-redes

O bloco deve ser dividido utilizando **VLSM (Variable Length Subnet Mask)**.

| Setor | Rede | Máscara | Gateway |
|---|---|---|---|
| Administrativo | `192.168.10.0/27` | `255.255.255.224` | `192.168.10.1` |
| Financeiro | `192.168.10.32/28` | `255.255.255.240` | `192.168.10.33` |
| TI | `192.168.10.48/28` | `255.255.255.240` | `192.168.10.49` |
| Recursos | `192.168.10.64/29` | `255.255.255.248` | `192.168.10.65` |

## 3.3 Recursos com IP estático

Os recursos compartilhados devem utilizar endereços fixos.

| Dispositivo | IP |
|---|---|
| Servidor | `192.168.10.66/29` |
| Impressora | `192.168.10.67/29` |

---

# 4. 🧩 Requisitos de Segmentação

Cada setor deve operar em uma VLAN própria.

| VLAN | Setor |
|---:|---|
| 10 | Administrativo |
| 20 | Financeiro |
| 30 | TI |
| 40 | Recursos |

As portas conectadas aos dispositivos finais devem operar em modo **Access**.

Os enlaces entre switches e entre o switch central e o roteador devem operar em modo **Trunk**, utilizando **IEEE 802.1Q**.

---

# 5. 🔀 Requisitos de Roteamento

As VLANs devem conseguir se comunicar através de roteamento inter-VLAN.

A solução utilizada no laboratório deve ser **Router-on-a-Stick**, com subinterfaces no roteador.

Cada subinterface deve funcionar como gateway de sua respectiva VLAN.

---

# 6. 📡 Requisitos de DHCP

A atribuição de endereços aos computadores deve ser automática.

O servidor DHCP deve:

- possuir IP estático;
- ficar localizado na VLAN 40;
- atender clientes das VLANs 10, 20 e 30;
- fornecer endereço IP;
- fornecer máscara;
- fornecer gateway;
- fornecer DNS.

Como o servidor DHCP está em outra rede, o roteador deve utilizar **DHCP Relay** através de `ip helper-address`.

---

# 7. 🔐 Requisitos de Comunicação entre Setores

A política de comunicação deve ser:

| Origem | ADM | Financeiro | TI | Recursos |
|---|---:|---:|---:|---:|
| **ADM** | ✅ | ❌ | ✅ | ✅ |
| **Financeiro** | ❌ | ✅ | ✅ | ✅ |
| **TI** | ✅ | ✅ | ✅ | ✅ |
| **Recursos** | — | — | — | — |

Regras principais:

- Administrativo não deve iniciar comunicação com Financeiro;
- Financeiro não deve iniciar comunicação com Administrativo;
- TI deve conseguir acessar Administrativo;
- TI deve conseguir acessar Financeiro;
- TI deve conseguir acessar Recursos;
- Administrativo deve conseguir acessar Recursos;
- Financeiro deve conseguir acessar Recursos.

Essas regras devem ser implementadas utilizando **ACLs estendidas**.

---

# 8. 📂 Requisitos de Acesso aos Arquivos

O servidor deve representar dois grupos de arquivos:

```text
ARQUIVOS_GERAIS
ARQUIVOS_FINANCEIRO
```

A política de acesso deve ser:

| Setor | ARQUIVOS_GERAIS | ARQUIVOS_FINANCEIRO |
|---|---:|---:|
| Administrativo | ✅ | ❌ |
| Financeiro | ✅ | ✅ |
| TI | ✅ | ✅ |

A diferenciação de acesso aos diretórios deve ser tratada como **permissão de servidor**, e não apenas como ACL de rede.

A ACL determina se o setor consegue chegar ao servidor.

A permissão no servidor determina quais arquivos o usuário pode acessar.

---

# 9. 🖨️ Requisitos de Recursos Compartilhados

A rede deve possuir:

- 1 servidor;
- 1 impressora de rede.

Todos os setores autorizados devem conseguir alcançar a VLAN de Recursos.

O servidor e a impressora devem permanecer em endereços fixos para facilitar administração e acesso.

---

# 10. 🌍 Requisitos de Conectividade Externa

A rede deve possuir uma saída para uma rede externa simulada.

A arquitetura deve representar:

```text
Rede Alfa
   |
Router Alfa
   |
WAN
   |
Router ISP
   |
Rede Externa
```

A conexão WAN entre o Router Alfa e o ISP deve utilizar uma rede dedicada.

No laboratório foi utilizada:

```text
200.0.0.0/30
```

A rede externa simulada utiliza:

```text
203.0.113.0/24
```

O servidor externo utilizado para testes possui:

```text
203.0.113.10
```

---

# 11. 🛣️ Requisitos de Roteamento Externo

O Router Alfa deve possuir uma **rota padrão** apontando para o Router ISP.

A finalidade da rota padrão é encaminhar qualquer destino desconhecido para o provedor.

Fluxo esperado:

```text
Destino desconhecido
        ↓
Router Alfa
        ↓
Rota padrão
        ↓
Router ISP
```

---

# 12. 🔄 Requisitos de NAT/PAT

Os dispositivos internos utilizam endereços privados `192.168.10.x`.

Para acessar a rede externa, deve ser utilizado **PAT / NAT Overload**.

O Router Alfa deve:

- identificar as interfaces internas como `NAT inside`;
- identificar a interface WAN como `NAT outside`;
- selecionar as redes internas que participarão da tradução;
- utilizar o endereço da interface externa para a tradução;
- permitir que vários hosts compartilhem um único endereço externo.

Fluxo esperado:

```text
192.168.10.x
      ↓
Router Alfa
      ↓ PAT
200.0.0.1
      ↓
ISP
      ↓
Rede externa
```

---

# 13. 🛡️ Requisitos de Segurança

A rede deve seguir princípios básicos de segmentação e menor privilégio.

Devem ser considerados:

- isolamento entre Administrativo e Financeiro;
- acesso administrativo da equipe de TI;
- segmentação por VLAN;
- ACLs para controle entre redes;
- permissões específicas para recursos do servidor;
- não utilização da VLAN 1 para usuários finais.

---

# 14. ✅ Requisitos de Validação

A solução deve ser considerada funcional quando os seguintes testes forem concluídos.

## DHCP

- [x] ADM recebe IP automaticamente;
- [x] Financeiro recebe IP automaticamente;
- [x] TI recebe IP automaticamente.

## Gateways

- [x] ADM alcança seu gateway;
- [x] Financeiro alcança seu gateway;
- [x] TI alcança seu gateway.

## Recursos

- [x] ADM alcança o servidor;
- [x] Financeiro alcança o servidor;
- [x] TI alcança o servidor.

## ACLs

- [x] ADM não alcança Financeiro;
- [x] Financeiro não alcança ADM;
- [x] TI alcança ADM;
- [x] TI alcança Financeiro.

## Rede externa

- [x] Router Alfa alcança o ISP;
- [x] ISP alcança o servidor externo;
- [x] Router Alfa alcança o servidor externo;
- [x] ADM alcança o servidor externo;
- [x] Financeiro alcança o servidor externo;
- [x] TI alcança o servidor externo.

## NAT/PAT

- [x] NAT/PAT realiza tradução dos IPs internos;
- [x] múltiplos hosts conseguem utilizar o mesmo endereço externo.

---

# 15. 🛠️ Requisitos de Troubleshooting

O projeto deve incluir validação por comandos de diagnóstico.

Principais comandos utilizados:

```text
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
show access-lists
show ip nat translations
show ip nat statistics
ping
ipconfig
```

O projeto deve registrar problemas encontrados, causa identificada e solução aplicada.

---

# 16. 📁 Requisitos de Documentação

O repositório deve conter, no mínimo:

```text
rede_Alfa_eng/
│
├── README.md
│
├── docs/
│   ├── requisitos.md
│   ├── ENDERECAMENTO_LOGICO.md
│   └── matriz_comunicacao.md
│
└── packet-tracer/
    └── LAB_01_ALFA.pkt
```

Como evolução, também devem ser adicionadas:

```text
configs/
evidence/
```

para armazenar configurações dos equipamentos e evidências de testes.

---

# 17. 🎯 Critério de Conclusão

O projeto será considerado concluído quando:

- a segmentação estiver funcionando;
- o roteamento inter-VLAN estiver operacional;
- o DHCP estiver atendendo todos os setores;
- as ACLs estiverem aplicando a política de comunicação;
- o ISP estiver simulado;
- a rota padrão estiver funcionando;
- o NAT/PAT estiver operacional;
- os testes internos e externos forem aprovados;
- a documentação estiver atualizada;
- o arquivo `.pkt` final estiver salvo no repositório.

---

# 18. 📚 Objetivo de Aprendizado

O projeto não busca apenas criar uma topologia funcional.

Seu objetivo é desenvolver capacidade de:

```text
requisito
   ↓
planejamento
   ↓
endereçamento
   ↓
segmentação
   ↓
roteamento
   ↓
serviços
   ↓
segurança
   ↓
testes
   ↓
troubleshooting
   ↓
documentação
```

Esse fluxo aproxima o laboratório de um cenário real de infraestrutura de redes e cria uma base para estudos posteriores de Linux, Cloud e DevOps.
