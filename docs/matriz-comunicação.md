# 🔐 Matriz de Comunicação — Alfa Engenharia

Este documento define os **requisitos de comunicação e acesso** da rede da Alfa Engenharia.

A matriz serve como referência para a implementação de:

- segmentação por VLAN;
- roteamento inter-VLAN;
- ACLs;
- acesso aos recursos compartilhados;
- permissões de arquivos no servidor.

---

## 🎯 Objetivo

A rede deve permitir somente as comunicações necessárias entre os setores, mantendo o **TI com acesso para suporte** e protegendo o setor Financeiro contra acessos indevidos.

Os setores considerados são:

- Administrativo (ADM);
- Financeiro (FIN);
- Tecnologia da Informação (TI);
- Recursos Compartilhados.

---

## 🌐 Redes do projeto

| Setor | VLAN | Rede | Gateway |
|---|---:|---|---|
| Administrativo | 10 | `192.168.10.0/27` | `192.168.10.1` |
| Financeiro | 20 | `192.168.10.32/28` | `192.168.10.33` |
| TI | 30 | `192.168.10.48/28` | `192.168.10.49` |
| Recursos | 40 | `192.168.10.64/29` | `192.168.10.65` |

---

# 📋 Requisitos de Comunicação

## Administrativo

O setor Administrativo deve:

- acessar a Internet;
- acessar a impressora;
- acessar o servidor de arquivos;
- acessar `ARQUIVOS_GERAIS`;
- **não acessar `ARQUIVOS_FINANCEIRO`**;
- **não iniciar comunicação com a rede Financeira**.

---

## Financeiro

O setor Financeiro deve:

- acessar a Internet;
- acessar a impressora;
- acessar o servidor de arquivos;
- acessar `ARQUIVOS_GERAIS`;
- acessar `ARQUIVOS_FINANCEIRO`;
- **não iniciar comunicação com a rede Administrativa**.

---

## Tecnologia da Informação

O setor de TI deve:

- acessar a Internet;
- acessar a impressora;
- acessar todos os recursos do servidor;
- acessar `ARQUIVOS_GERAIS`;
- acessar `ARQUIVOS_FINANCEIRO`;
- acessar computadores do Administrativo;
- acessar computadores do Financeiro;
- possuir conectividade com os demais setores para suporte e administração.

---

## Recursos Compartilhados

A rede de Recursos deve concentrar os serviços compartilhados da empresa, incluindo:

- servidor;
- impressora de rede;
- serviços internos futuros.

Endereços planejados:

| Recurso | Endereço |
|---|---|
| Servidor | `192.168.10.66` |
| Impressora | `192.168.10.67` |

---

# 🔀 Matriz de Comunicação entre Redes

| Origem | ADM | Financeiro | TI | Recursos | Internet |
|---|:---:|:---:|:---:|:---:|:---:|
| **ADM** | ✅ | ❌ | ✅ | ✅ | ✅ |
| **Financeiro** | ❌ | ✅ | ✅ | ✅ | ✅ |
| **TI** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Recursos** | — | — | — | — | Conforme necessidade |

### Legenda

- ✅ Permitido
- ❌ Bloqueado
- — Não aplicável / depende do serviço

---

# 📂 Matriz de Acesso aos Arquivos

| Setor | `ARQUIVOS_GERAIS` | `ARQUIVOS_FINANCEIRO` |
|---|:---:|:---:|
| Administrativo | ✅ | ❌ |
| Financeiro | ✅ | ✅ |
| TI | ✅ | ✅ |

---

## ⚠️ ACL x Permissão de Arquivos

As ACLs configuradas no roteador controlam **comunicação entre redes**.

Exemplo:

```text
ADM → Financeiro = BLOQUEADO
Financeiro → ADM = BLOQUEADO
```

Porém, `ARQUIVOS_GERAIS` e `ARQUIVOS_FINANCEIRO` estão previstos no mesmo servidor:

```text
Servidor: 192.168.10.66
```

Uma ACL IP consegue identificar:

```text
origem
destino
protocolo
porta
```

mas não diferencia diretamente duas pastas armazenadas no mesmo servidor.

Por isso, a política de arquivos deve ser implementada na camada de servidor utilizando:

- usuários;
- grupos;
- autenticação;
- permissões de diretório.

Exemplo conceitual:

```text
Grupo ADM
├── ARQUIVOS_GERAIS       → PERMITIDO
└── ARQUIVOS_FINANCEIRO   → NEGADO

Grupo FIN
├── ARQUIVOS_GERAIS       → PERMITIDO
└── ARQUIVOS_FINANCEIRO   → PERMITIDO

Grupo TI
├── ARQUIVOS_GERAIS       → PERMITIDO
└── ARQUIVOS_FINANCEIRO   → PERMITIDO
```

---

# 🔒 Políticas implementadas com ACL

Atualmente foram implementadas as seguintes políticas de rede:

```text
ADM → Financeiro        ❌
Financeiro → ADM        ❌

ADM → Recursos          ✅
Financeiro → Recursos   ✅

TI → ADM                ✅
TI → Financeiro         ✅
TI → Recursos           ✅
```

Essas regras mantêm o isolamento entre Administrativo e Financeiro, sem impedir o setor de TI de realizar suporte.

---

# ✅ Critérios de Validação

A política será considerada corretamente implementada quando:

- [x] ADM não conseguir alcançar PCs do Financeiro;
- [x] Financeiro não conseguir alcançar PCs do ADM;
- [x] TI conseguir alcançar PCs do ADM;
- [x] TI conseguir alcançar PCs do Financeiro;
- [x] ADM conseguir alcançar o servidor;
- [x] Financeiro conseguir alcançar o servidor;
- [x] TI conseguir alcançar o servidor;
- [x] ADM conseguir acessar a rede externa;
- [x] Financeiro conseguir acessar a rede externa;
- [x] TI conseguir acessar a rede externa;
- [ ] ADM acessar somente `ARQUIVOS_GERAIS`;
- [ ] Financeiro acessar `ARQUIVOS_GERAIS` e `ARQUIVOS_FINANCEIRO`;
- [ ] TI possuir acesso completo aos recursos do servidor.

---

## 📌 Resumo

A política de segurança do projeto utiliza duas camadas:

```text
ACLs
↓
controlam quem pode se comunicar entre redes

Permissões do servidor
↓
controlam o que cada usuário pode acessar após chegar ao servidor
```

Essa separação permite manter a rede segmentada e, ao mesmo tempo, aplicar permissões específicas aos recursos corporativos.
