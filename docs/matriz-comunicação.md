# 🔐 Matriz de Comunicação — Alfa Engenharia

Este documento define **quais setores podem se comunicar entre si** dentro da rede da Alfa Engenharia e quais acessos devem ser bloqueados.

A política foi definida considerando:

- separação entre Administrativo e Financeiro;
- acesso do setor de TI aos demais setores para suporte;
- acesso dos setores aos recursos compartilhados;
- controle de acesso aos arquivos no servidor por permissões específicas.

---

## 🌐 Matriz de Comunicação entre Redes

| Origem | ADM | Financeiro | TI | Recursos |
|---|---:|---:|---:|---:|
| **ADM** | ✅ | ❌ | ✅ | ✅ |
| **Financeiro** | ❌ | ✅ | ✅ | ✅ |
| **TI** | ✅ | ✅ | ✅ | ✅ |
| **Recursos** | — | — | — | — |

### Legenda

- ✅ **Permitido**
- ❌ **Bloqueado**
- — **Não se aplica / não definido como origem de usuários**

---

## 🧩 Redes envolvidas

| Setor | VLAN | Rede |
|---|---:|---|
| Administrativo | 10 | `192.168.10.0/27` |
| Financeiro | 20 | `192.168.10.32/28` |
| TI | 30 | `192.168.10.48/28` |
| Recursos | 40 | `192.168.10.64/29` |

---

## 🔒 Políticas implementadas com ACL

### Administrativo → Financeiro

O setor Administrativo **não pode iniciar comunicação com a rede Financeira**.

```text
ADM → Financeiro = BLOQUEADO
```

ACL utilizada:

```cisco
ip access-list extended CL_ADM
 deny ip 192.168.10.0 0.0.0.31 192.168.10.32 0.0.0.15
 permit ip any any
```

Aplicada na entrada da subinterface da VLAN 10:

```cisco
interface GigabitEthernet0/0/0.10
 ip access-group CL_ADM in
```

---

### Financeiro → Administrativo

O setor Financeiro **não pode iniciar comunicação com a rede Administrativa**.

```text
Financeiro → ADM = BLOQUEADO
```

ACL utilizada:

```cisco
ip access-list extended CL_FIN
 deny ip 192.168.10.32 0.0.0.15 192.168.10.0 0.0.0.31
 permit ip any any
```

Aplicada na entrada da subinterface da VLAN 20:

```cisco
interface GigabitEthernet0/0/0.20
 ip access-group CL_FIN in
```

---

## 🛠️ Acesso do setor de TI

O setor de TI permanece com acesso aos demais setores para permitir atividades de:

- suporte;
- manutenção;
- diagnóstico;
- administração da infraestrutura.

Assim:

```text
TI → ADM = PERMITIDO
TI → Financeiro = PERMITIDO
TI → Recursos = PERMITIDO
```

---

## 🗄️ Acesso aos Recursos Compartilhados

Os setores podem acessar a rede de Recursos:

```text
ADM → Recursos = PERMITIDO
Financeiro → Recursos = PERMITIDO
TI → Recursos = PERMITIDO
```

A rede de Recursos utiliza:

```text
192.168.10.64/29
```

Principais dispositivos:

| Recurso | Endereço |
|---|---|
| Servidor | `192.168.10.66` |
| Impressora | `192.168.10.67` |

---

# 📂 Matriz de Permissão de Arquivos

O controle abaixo **não é feito apenas por ACL de rede**.

As ACLs controlam se um setor consegue chegar ao servidor.  
As permissões do servidor determinam **quais arquivos ou pastas cada usuário pode acessar**.

| Setor | ARQUIVOS_GERAIS | ARQUIVOS_FINANCEIRO |
|---|---:|---:|
| Administrativo | ✅ | ❌ |
| Financeiro | ✅ | ✅ |
| TI | ✅ | ✅ |

---

## 🧠 Diferença entre ACL e Permissão no Servidor

### ACL de rede

Responde:

```text
"Este dispositivo pode alcançar essa rede ou esse servidor?"
```

Exemplo:

```text
ADM → Financeiro = BLOQUEADO
```

### Permissão no servidor

Responde:

```text
"Depois que o usuário chegou ao servidor, quais arquivos ele pode acessar?"
```

Exemplo:

```text
ADM → ARQUIVOS_GERAIS = PERMITIDO
ADM → ARQUIVOS_FINANCEIRO = BLOQUEADO
```

---

## ✅ Resultado da Política

A política final da rede é:

```text
ADM
├── Financeiro ❌
├── TI ✅
└── Recursos ✅

Financeiro
├── ADM ❌
├── TI ✅
└── Recursos ✅

TI
├── ADM ✅
├── Financeiro ✅
└── Recursos ✅
```

Essa separação permite proteger informações sensíveis do setor Financeiro sem impedir o acesso da equipe de TI aos demais segmentos da rede.
