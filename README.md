
# 🌐 Rede Corporativa — Alfa Engenharia

### Projeto de Infraestrutura e Redes de Computadores

![Status](https://img.shields.io/badge/Status-Em%20evolu%C3%A7%C3%A3o-blue)
![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-049FD9)
![Redes](https://img.shields.io/badge/Redes-TCP%2FIP-green)
![NAT](https://img.shields.io/badge/NAT%2FPAT-Validado-success)

## 📌 Sobre o projeto

O projeto **Rede Alfa Engenharia** consiste no planejamento, desenvolvimento e simulação de uma infraestrutura de rede corporativa utilizando o Cisco Packet Tracer.

O cenário representa uma empresa fictícia com diferentes departamentos que necessitam de comunicação interna, acesso controlado aos recursos de rede e conectividade com uma rede externa simulada.

A solução utiliza segmentação por VLANs, endereçamento IPv4 com VLSM, roteamento entre redes, distribuição dinâmica de endereços por DHCP, listas de controle de acesso (ACLs) e tradução de endereços NAT/PAT.

O objetivo é aplicar conceitos de infraestrutura e redes em um cenário próximo aos desafios encontrados na administração de ambientes corporativos.

---

## 🎯 Objetivos

- Planejar uma rede corporativa segmentada.
- Aplicar VLSM para otimização de endereços IPv4.
- Configurar VLANs para diferentes departamentos.
- Implementar comunicação entre VLANs.
- Centralizar a distribuição de endereços via DHCP.
- Controlar o tráfego entre setores utilizando ACLs.
- Configurar conectividade com um provedor externo simulado.
- Implementar NAT/PAT para comunicação externa.
- Realizar testes de conectividade e troubleshooting.
- Documentar configurações e resultados.

---

## 🛠️ Tecnologias e conceitos

| Tecnologia | Aplicação |
|---|---|
| Cisco Packet Tracer | Simulação da infraestrutura |
| Cisco IOS | Configuração dos dispositivos |
| IPv4 e VLSM | Planejamento de sub-redes |
| VLANs | Segmentação lógica |
| IEEE 802.1Q | Comunicação por trunks |
| Router-on-a-Stick | Roteamento entre VLANs |
| DHCP | Atribuição automática de IP |
| DHCP Relay | Encaminhamento de solicitações DHCP |
| ACLs | Controle de acesso |
| Roteamento estático | Comunicação entre redes |
| NAT/PAT | Tradução de endereços |
| ICMP | Testes de conectividade |

---

## 🏢 Cenário corporativo

A infraestrutura representa quatro setores da Alfa Engenharia:

1. Administrativo
2. Financeiro
3. Tecnologia da Informação
4. Recursos compartilhados

Cada setor utiliza uma VLAN específica, permitindo organização lógica, redução dos domínios de broadcast e aplicação de regras de comunicação entre departamentos.

### Arquitetura lógica simplificada

```text
                REDE EXTERNA
                     |
               SERVIDOR EXTERNO
                 203.0.113.10
                     |
                ROTEADOR ISP
                     |
                LINK WAN /30
                     |
              ROTEADOR ALFA
              NAT/PAT + ACLs
                     |
            ROUTER-ON-A-STICK
                     |
                SW-CENTRAL
                     |
        +------------+------------+
        |            |            |
     VLAN 10      VLAN 20      VLAN 30
  Administrativo  Financeiro       TI
        |
     VLAN 40
 Recursos compartilhados
   DHCP e impressora
```

> Esta representação é simplificada. A topologia detalhada deve ser consultada no arquivo do Cisco Packet Tracer.

---

## 📡 Planejamento de endereçamento IPv4

Foi utilizada a rede base:

**192.168.10.0/24**

A divisão em sub-redes utiliza VLSM (Variable Length Subnet Mask), permitindo alocar blocos de endereços de acordo com as necessidades dos setores.

### Tabela de endereçamento

| VLAN | Departamento | Rede | Máscara | Gateway |
|---|---|---|---|---|
| 10 | Administrativo | 192.168.10.0/27 | 255.255.255.224 | 192.168.10.1 |
| 20 | Financeiro | 192.168.10.32/28 | 255.255.255.240 | 192.168.10.33 |
| 30 | TI | 192.168.10.48/28 | 255.255.255.240 | 192.168.10.49 |
| 40 | Recursos | 192.168.10.64/29 | 255.255.255.248 | 192.168.10.65 |

### Vantagens da utilização de VLSM

- Melhor aproveitamento dos endereços disponíveis.
- Dimensionamento de redes conforme a demanda.
- Separação lógica dos departamentos.
- Organização e facilidade de administração.
- Redução do desperdício de endereços IPv4.

---

## 🔀 VLANs e comunicação entre redes

A segmentação da infraestrutura foi realizada por meio de VLANs.

Cada VLAN representa um domínio de broadcast independente.

Para permitir comunicação entre diferentes VLANs, foi utilizada a arquitetura **Router-on-a-Stick**.

Nessa abordagem, um enlace trunk 802.1Q transporta o tráfego das VLANs até o roteador, que realiza o encaminhamento entre as sub-redes por meio de subinterfaces.

### Interfaces de roteamento

| Interface | VLAN | Gateway |
|---|---|---|
| GigabitEthernet0/0/0.10 | 10 | 192.168.10.1 |
| GigabitEthernet0/0/0.20 | 20 | 192.168.10.33 |
| GigabitEthernet0/0/0.30 | 30 | 192.168.10.49 |
| GigabitEthernet0/0/0.40 | 40 | 192.168.10.65 |

As subinterfaces podem ser verificadas utilizando:

```cisco
show ip interface brief
show running-config
show ip route
```

Esses comandos permitem identificar o estado das interfaces, configurações de encapsulamento e redes conhecidas pelo roteador.

---

## 🖥️ DHCP centralizado

O projeto utiliza distribuição centralizada de endereços IP por meio do protocolo DHCP.

O servidor DHCP está localizado na rede de recursos compartilhados.

**Endereço utilizado no projeto:**

```text
Servidor DHCP: 192.168.10.66
```

Como os clientes estão distribuídos entre diferentes VLANs, foi utilizado DHCP Relay para encaminhamento das solicitações até o servidor.

O mecanismo utiliza o comando Cisco:

```cisco
ip helper-address 192.168.10.66
```

### Funcionamento

1. O cliente solicita uma configuração IP.
2. A solicitação DHCP é transmitida na VLAN local.
3. O roteador recebe a solicitação na interface correspondente.
4. O DHCP Relay encaminha a solicitação ao servidor.
5. O servidor fornece os parâmetros de rede.
6. O cliente recebe IP, máscara e gateway conforme sua configuração.

A implementação permite centralizar a administração dos endereços utilizados pelos departamentos.

---

## 🔐 Controle de acesso entre VLANs

Foram trabalhadas listas de controle de acesso, conhecidas como ACLs (*Access Control Lists*).

As ACLs permitem definir regras de tráfego entre diferentes segmentos da rede.

Sua aplicação é importante para restringir comunicações não autorizadas e estabelecer políticas de acesso entre departamentos.

### Comandos de diagnóstico

```cisco
show access-lists
show ip interface
show running-config
```

### Verificações recomendadas

- Identificar as ACLs existentes.
- Conferir em quais interfaces estão aplicadas.
- Verificar o sentido de aplicação: entrada ou saída.
- Executar testes de tráfego permitido.
- Executar testes de tráfego bloqueado.
- Conferir a ordem das regras.

> A política detalhada e seus resultados devem ser consultados nas configurações e evidências específicas das ACLs. Os testes apresentados abaixo validam a conectividade externa, mas não substituem a validação das regras de acesso entre setores.

---

# 🌍 Conectividade externa — ISP

Foi configurada uma conexão entre o roteador da empresa e um roteador que representa um provedor de serviços de internet (ISP).

O objetivo é permitir que os dispositivos internos acessem recursos localizados fora da rede corporativa simulada.

### Endereçamento WAN

| Dispositivo | Endereço |
|---|---|
| Roteador Alfa | 200.0.0.1/30 |
| Roteador ISP | 200.0.0.2/30 |
| Servidor externo | 203.0.113.10 |

A comunicação externa utiliza uma rota padrão no roteador Alfa.

### Rota padrão

```cisco
ip route 0.0.0.0 0.0.0.0 200.0.0.2
```

A rota padrão é utilizada quando não existe uma rota mais específica para o destino na tabela de roteamento.

Dessa forma, os pacotes destinados a redes externas podem ser encaminhados ao roteador ISP.

---

## 🔁 NAT/PAT — Tradução de endereços

Para permitir a comunicação dos dispositivos internos com a rede externa simulada, foi configurado NAT com sobrecarga (*PAT — Port Address Translation*).

O PAT permite que diferentes dispositivos privados compartilhem um endereço externo.

Na simulação, a tradução utiliza o endereço **200.0.0.1** da interface de saída do roteador Alfa.

### Funcionamento

```text
PC interno
192.168.10.7
      |
      v
Roteador Alfa
NAT/PAT
      |
      v
Endereço traduzido
200.0.0.1
      |
      v
Roteador ISP
      |
      v
Servidor externo
203.0.113.10
```

### Comandos de verificação

```cisco
show ip nat translations
show ip nat statistics
show ip route
```

O comando `show ip nat translations` permite visualizar traduções ativas.

O comando `show ip nat statistics` permite consultar estatísticas e identificar as interfaces configuradas para NAT.

---

# ✅ Validação e evidências dos testes

Os testes apresentados nesta seção foram executados no Cisco Packet Tracer.

O objetivo é comprovar o funcionamento da conectividade externa, do encaminhamento por rota padrão e da tradução NAT/PAT.

## Teste 01 — Conectividade com servidor externo

**Objetivo:** verificar se um dispositivo da rede interna consegue alcançar o servidor externo simulado.

**Dispositivo:** PC0(4)

**Comando executado:**

```cmd
ping 203.0.113.10
```

### Resultado observado

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

O teste apresentou quatro respostas do servidor externo, sem perda de pacotes.

### Evidência

![Ping para o servidor externo](evidencias/01_ping_externo.png)

**Conclusão:** a comunicação ICMP entre o dispositivo interno e o servidor externo simulado foi validada.

---

## Teste 02 — Verificação da rota padrão

**Objetivo:** verificar a existência de uma rota padrão para encaminhamento de pacotes destinados a redes externas.

**Dispositivo:** Roteador Alfa

**Comando executado:**

```cisco
show ip route
```

### Resultado observado

```text
Gateway of last resort is 200.0.0.2 to network 0.0.0.0
```

O roteador apresenta o endereço 200.0.0.2 como gateway de último recurso.

A tabela também apresenta as sub-redes internas diretamente conectadas às subinterfaces do roteador.

### Evidência

![Tabela de roteamento](evidencias/02_rota_padrao.png)

**Conclusão:** a presença do gateway de último recurso foi confirmada na tabela de roteamento.

---

## Teste 03 — Verificação das traduções NAT/PAT

**Objetivo:** verificar se o roteador realiza a tradução dos endereços privados durante a comunicação com o servidor externo.

**Dispositivo:** Roteador Alfa

**Comando executado:**

```cisco
show ip nat translations
```

### Resultado observado

Foram registradas traduções ICMP envolvendo:

| Campo | Valor |
|---|---|
| Protocolo | ICMP |
| Inside Local | 192.168.10.7 |
| Inside Global | 200.0.0.1 |
| Outside Global | 203.0.113.10 |

As entradas apresentam identificadores ICMP distintos, demonstrando o funcionamento da tradução por sobrecarga.

### Evidência

![Traduções NAT e PAT](evidencias/03_nat_translations.png)

**Conclusão:** a captura comprova a existência de traduções ICMP do endereço privado 192.168.10.7 para o endereço externo 200.0.0.1.

---

## Teste 04 — Estatísticas NAT

**Objetivo:** consultar os contadores e as interfaces utilizadas pelo mecanismo de tradução.

**Dispositivo:** Roteador Alfa

**Comando executado:**

```cisco
show ip nat statistics
```

### Resultado observado

```text
Total translations: 0
Hits: 29
Misses: 32
Expired translations: 32
```

A captura registra a interface de saída GigabitEthernet0/0/1 e as subinterfaces internas configuradas para NAT.

O valor de zero traduções indica que não havia entradas ativas no momento dessa consulta.

As estatísticas históricas e a evidência anterior de traduções permitem verificar que o mecanismo havia processado tráfego.

### Evidência

![Estatísticas NAT](evidencias/04_nat_statistics.png)

**Conclusão:** as interfaces NAT e seus contadores foram verificados. As traduções observadas no teste anterior já haviam expirado no momento da última captura.

---

## 📊 Resumo da validação

| Teste | Resultado | Situação |
|---|---|---|
| Ping ao servidor externo | 4 respostas, 0% de perda | ✅ Validado |
| Gateway de último recurso | 200.0.0.2 | ✅ Validado |
| Tradução NAT/PAT | 192.168.10.7 → 200.0.0.1 | ✅ Validado |
| Estatísticas NAT | Interfaces e contadores registrados | ✅ Verificado |
| DHCP por VLAN | Configuração trabalhada no projeto | 📝 Evidência a anexar |
| ACLs | Regras trabalhadas no projeto | 📝 Evidência a anexar |
| Roteamento entre VLANs | Interfaces conectadas na tabela | 📝 Testes entre clientes a anexar |

As evidências permitem comprovar a conectividade externa do dispositivo testado e o funcionamento das traduções NAT/PAT.

Elas não comprovam isoladamente que todos os clientes, serviços e políticas de acesso da infraestrutura estejam funcionando.

---

## 🔍 Troubleshooting e diagnóstico

Durante o desenvolvimento de uma infraestrutura de redes, é importante utilizar ferramentas de diagnóstico para identificar possíveis falhas.

### Comandos utilizados

| Comando | Finalidade |
|---|---|
| `ping` | Testar conectividade IP |
| `ipconfig` | Verificar configuração de rede do cliente |
| `show ip interface brief` | Consultar o estado das interfaces |
| `show ip route` | Verificar rotas disponíveis |
| `show vlan brief` | Consultar VLANs no switch |
| `show interfaces trunk` | Verificar links trunk |
| `show access-lists` | Consultar regras de controle de acesso |
| `show ip nat translations` | Visualizar traduções NAT |
| `show ip nat statistics` | Consultar estatísticas NAT |
| `show running-config` | Consultar configurações dos dispositivos |

### Metodologia de diagnóstico

1. Identificar o problema relatado.
2. Verificar as interfaces envolvidas.
3. Conferir endereçamento IP e gateway.
4. Consultar a tabela de roteamento.
5. Testar conectividade local e remota.
6. Avaliar ACLs e traduções NAT, quando aplicável.
7. Aplicar a correção necessária.
8. Executar novamente os testes.
9. Registrar os resultados e evidências.

---

## 📁 Organização do repositório

Estrutura recomendada para documentação e manutenção:

```text
rede_Alfa_eng/
│
├── README.md
│
├── packet-tracer/
│   └── projeto_rede_alfa.pkt
│
├── evidencias/
│   ├── 01_ping_externo.png
│   ├── 02_rota_padrao.png
│   ├── 03_nat_translations.png
│   └── 04_nat_statistics.png
│
└── docs/
    └── relatorio_testes.md
```

> A organização apresentada é uma sugestão. Os caminhos do arquivo `.pkt` e das evidências devem ser ajustados conforme os arquivos efetivamente enviados ao repositório.

---

## ▶️ Como explorar o projeto

### Pré-requisitos

- Cisco Packet Tracer instalado.
- Conhecimentos introdutórios de IPv4.
- Noções de VLANs, roteamento e redes locais.

### Procedimento

1. Clone ou baixe o repositório.
2. Abra o arquivo `.pkt` no Cisco Packet Tracer.
3. Examine a topologia e as conexões.
4. Consulte as configurações dos roteadores e switches.
5. Verifique o endereçamento dos clientes.
6. Execute os testes de conectividade documentados.
7. Compare os resultados com as capturas da pasta `evidencias`.

### Exemplo de teste

Em um computador interno:

```cmd
ping 203.0.113.10
```

No roteador Alfa:

```cisco
show ip route
show ip nat translations
show ip nat statistics
```

Os resultados podem variar conforme o estado da simulação e o momento em que os comandos são executados.

---

## 🚀 Próximas melhorias

O projeto continuará evoluindo com foco em conhecimentos de infraestrutura, administração de sistemas e monitoramento.

### Etapa 1 — Documentação de redes

- [x] Planejamento com VLSM.
- [x] Configuração das VLANs.
- [x] Roteamento entre VLANs.
- [x] DHCP centralizado.
- [x] Configuração de ACLs.
- [x] Conectividade com ISP simulado.
- [x] Rota padrão.
- [x] NAT/PAT.
- [x] Evidências de conectividade externa.
- [ ] Adicionar evidências completas dos testes de DHCP e ACLs.
- [ ] Finalizar a documentação da topologia.

### Etapa 2 — Administração Linux

- [ ] Preparar ambiente Ubuntu.
- [ ] Praticar comandos básicos de Linux.
- [ ] Administrar processos e serviços.
- [ ] Configurar servidor Nginx.
- [ ] Consultar logs e diagnosticar falhas.
- [ ] Simular indisponibilidade e recuperação de serviço.

### Etapa 3 — Monitoramento com Zabbix

- [ ] Preparar ambiente de monitoramento.
- [ ] Configurar Zabbix Server e Agent.
- [ ] Monitorar CPU, memória e disco.
- [ ] Criar verificações de disponibilidade.
- [ ] Configurar triggers.
- [ ] Simular e investigar um incidente.
- [ ] Documentar métricas, alertas e recuperação.

### Etapa 4 — Dashboards e observabilidade

- [ ] Criar dashboard de infraestrutura.
- [ ] Visualizar métricas de desempenho.
- [ ] Investigar alertas e incidentes.
- [ ] Avaliar integração com Grafana.

> O laboratório Linux/Zabbix é uma evolução complementar, separada da topologia simulada no Packet Tracer. Uma integração direta entre os ambientes depende de recursos de conectividade e monitoramento específicos.

---

## 📚 Aprendizados

O desenvolvimento da Rede Alfa Engenharia permite aplicar conhecimentos relacionados a:

- Planejamento e organização de infraestrutura.
- Endereçamento e subnetting IPv4.
- Configuração de equipamentos de rede.
- Segmentação lógica e roteamento.
- Distribuição dinâmica de endereços.
- Controle de comunicação entre departamentos.
- Tradução de endereços privados.
- Diagnóstico de conectividade.
- Validação técnica por evidências.
- Documentação de projetos de infraestrutura.

Além das configurações, o projeto busca desenvolver a capacidade de compreender o funcionamento da rede, interpretar resultados e investigar problemas de forma estruturada.

---

## 📖 Referências

- [Cisco Networking Academy](https://www.netacad.com/)
- [Documentação oficial Cisco](https://www.cisco.com/c/en/us/support/index.html)
- [Documentação Ubuntu Server](https://ubuntu.com/server/docs)
- [Documentação Zabbix](https://www.zabbix.com/documentation/current/pt/)
- [Documentação Grafana](https://grafana.com/docs/)

---

## 👩‍💻 Autoria

**Hellen Neves Barbosa**

Graduanda em Engenharia da Computação — Universidade Federal do Maranhão (UFMA).

Projeto acadêmico e pessoal voltado ao desenvolvimento de conhecimentos práticos em infraestrutura, redes de computadores e administração de sistemas.

**GitHub:** [hellenbarbosa08](https://github.com/hellenbarbosa08)

---

**Rede Alfa Engenharia — Projeto de Infraestrutura e Redes**
