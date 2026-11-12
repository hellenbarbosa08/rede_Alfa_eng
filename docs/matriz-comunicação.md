# 🔐 Matriz de Comunicação — Alfa Engenharia

Este documento registra a política de comunicação efetivamente descrita na versão atual do laboratório.

---

## 1. Redes

| Setor | VLAN | Rede |
|---|---:|---|
| Administrativo | 10 | `192.168.10.0/27` |
| Financeiro | 20 | `192.168.10.32/28` |
| TI | 30 | `192.168.10.48/28` |
| Recursos | 40 | `192.168.10.64/29` |

---

## 2. Comunicação entre setores

| Origem | ADM | Financeiro | TI | Recursos |
|---|---:|---:|---:|---:|
| **ADM** | ✅ | ❌ | ✅ | ⚠️ Controlado |
| **Financeiro** | ❌ | ✅ | ✅ | ⚠️ Controlado |
| **TI** | ✅ | ✅ | ✅ | ⚠️ Controlado |

### Legenda

- ✅ Permitido
- ❌ Bloqueado
- ⚠️ O acesso depende do recurso/protocolo

---

## 3. Recursos da VLAN 40

| Recurso | IP | Serviço |
|---|---|---|
| DHCP | `192.168.10.66` | DHCP |
| Impressora | `192.168.10.67` | Impressão |
| `SRV-GERAIS` | `192.168.10.68` | FTP |
| `SRV-FINANCEIRO` | `192.168.10.69` | FTP |

---

## 4. Matriz de acesso aos servidores FTP

| Origem | `SRV-GERAIS` `.68` | `SRV-FINANCEIRO` `.69` |
|---|---|---|
| Administrativo | ✅ FTP autenticado | ❌ Tráfego IP bloqueado |
| Financeiro | ❌ Tráfego IP bloqueado | ✅ FTP autenticado |
| TI | ✅ FTP autenticado | ⚠️ TCP/21 bloqueado; ICMP permitido |

Essa tabela é mais precisa do que simplesmente marcar toda a VLAN 40 como permitida, porque as ACLs possuem regras específicas para cada servidor e serviço.

---

## 5. Isolamento ADM ↔ Financeiro

### Administrativo → Financeiro

```cisco
ip access-list extended CL_ADM
 deny ip 192.168.10.0 0.0.0.31 192.168.10.32 0.0.0.15
```

Resultado:

```text
ADM → Financeiro = BLOQUEADO
```

### Financeiro → Administrativo

```cisco
ip access-list extended CL_FIN
 deny ip 192.168.10.32 0.0.0.15 192.168.10.0 0.0.0.31
```

Resultado:

```text
Financeiro → ADM = BLOQUEADO
```

> As demais linhas das ACLs devem ser consultadas diretamente na configuração do roteador, pois existem regras adicionais relacionadas aos servidores e serviços.

---

## 6. Acesso administrativo por SSH

| Origem | SSH no R-ALFA |
|---|---:|
| Administrativo | ❌ |
| Financeiro | ❌ / não autorizado |
| TI | ✅ |

A ACL `SSH_TI` restringe a origem permitida para administração remota do roteador.

---

## 7. Diferença entre conectividade e serviço

Ter conectividade IP com um equipamento não significa necessariamente poder utilizar todos os serviços dele.

Exemplo da política do TI para `SRV-FINANCEIRO`:

```text
ping 192.168.10.69 → pode responder
ftp 192.168.10.69 → bloqueado na porta TCP 21
```

Isso ocorre porque a ACL pode filtrar um protocolo ou porta específica sem bloquear todo o tráfego IP.

---

## 8. Resultado resumido

```text
ADM
├── Financeiro ❌
├── TI ✅
├── SRV-GERAIS ✅ FTP autenticado
└── SRV-FINANCEIRO ❌

Financeiro
├── ADM ❌
├── TI ✅
├── SRV-GERAIS ❌
└── SRV-FINANCEIRO ✅ FTP autenticado

TI
├── ADM ✅
├── Financeiro ✅
├── SRV-GERAIS ✅ FTP autenticado
├── SRV-FINANCEIRO ⚠️ FTP bloqueado
└── R-ALFA via SSH ✅
```
