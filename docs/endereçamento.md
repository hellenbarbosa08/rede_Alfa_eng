# 🌐 Plano de Endereçamento Lógico

Este documento apresenta o plano de endereçamento IPv4 da rede da **Alfa Engenharia**, organizado por setor e dimensionado com **VLSM (Variable Length Subnet Mask)**.

O bloco base utilizado no projeto é:

```text
192.168.10.0/24
```

A rede foi subdividida de acordo com a quantidade de hosts necessária em cada setor, buscando reduzir desperdício de endereços e permitir crescimento da infraestrutura.

---

## 📊 Resumo das Sub-redes

| Setor | Rede / Prefixo | Máscara | Gateway | Faixa de Hosts | Broadcast | Hosts Úteis |
|---|---|---|---|---|---|---:|
| Administrativo | `192.168.10.0/27` | `255.255.255.224` | `192.168.10.1` | `192.168.10.1 - 192.168.10.30` | `192.168.10.31` | 30 |
| Financeiro | `192.168.10.32/28` | `255.255.255.240` | `192.168.10.33` | `192.168.10.33 - 192.168.10.46` | `192.168.10.47` | 14 |
| TI | `192.168.10.48/28` | `255.255.255.240` | `192.168.10.49` | `192.168.10.49 - 192.168.10.62` | `192.168.10.63` | 14 |
| Recursos Compartilhados | `192.168.10.64/29` | `255.255.255.248` | `192.168.10.65` | `192.168.10.65 - 192.168.10.70` | `192.168.10.71` | 6 |

---

## 🧩 Administrativo

```text
Rede:              192.168.10.0/27
Máscara:           255.255.255.224
Endereço de rede:  192.168.10.0
Primeiro host:     192.168.10.1
Último host:       192.168.10.30
Broadcast:         192.168.10.31
Gateway:           192.168.10.1
Hosts disponíveis: 30
```

A máscara `/27` fornece **32 endereços totais**, sendo 30 utilizáveis por hosts.

```text
Bloco = 256 - 224 = 32
```

---

## 💰 Financeiro

```text
Rede:              192.168.10.32/28
Máscara:           255.255.255.240
Endereço de rede:  192.168.10.32
Primeiro host:     192.168.10.33
Último host:       192.168.10.46
Broadcast:         192.168.10.47
Gateway:           192.168.10.33
Hosts disponíveis: 14
```

A máscara `/28` fornece **16 endereços totais**, sendo 14 utilizáveis por hosts.

```text
Bloco = 256 - 240 = 16
```

---

## 🖥️ Tecnologia da Informação

```text
Rede:              192.168.10.48/28
Máscara:           255.255.255.240
Endereço de rede:  192.168.10.48
Primeiro host:     192.168.10.49
Último host:       192.168.10.62
Broadcast:         192.168.10.63
Gateway:           192.168.10.49
Hosts disponíveis: 14
```

Assim como o setor Financeiro, a rede de TI utiliza `/28`, disponibilizando 14 endereços utilizáveis.

---

## 🗄️ Recursos Compartilhados

```text
Rede:              192.168.10.64/29
Máscara:           255.255.255.248
Endereço de rede:  192.168.10.64
Primeiro host:     192.168.10.65
Último host:       192.168.10.70
Broadcast:         192.168.10.71
Gateway:           192.168.10.65
Hosts disponíveis: 6
```

A rede `/29` fornece **8 endereços totais**, sendo 6 utilizáveis por hosts.

Nesta sub-rede ficam os recursos compartilhados da empresa, como:

- servidor;
- impressora de rede;
- outros serviços internos que possam ser adicionados futuramente.

---

## 🧠 Lógica utilizada

Para determinar cada sub-rede foi considerado o número de hosts necessário e escolhido o menor prefixo capaz de comportar essa quantidade.

| Prefixo | Máscara | Tamanho do Bloco | Hosts Úteis |
|---|---|---:|---:|
| `/27` | `255.255.255.224` | 32 | 30 |
| `/28` | `255.255.255.240` | 16 | 14 |
| `/29` | `255.255.255.248` | 8 | 6 |

Para calcular o tamanho do bloco:

```text
Bloco = 256 - último octeto da máscara
```

Exemplo para `/28`:

```text
256 - 240 = 16
```

Logo, as redes avançam em blocos de 16:

```text
192.168.10.0
192.168.10.16
192.168.10.32
192.168.10.48
192.168.10.64
...
```

O broadcast de uma sub-rede corresponde ao endereço imediatamente anterior ao início da próxima rede.

Exemplo:

```text
Rede atual:     192.168.10.32/28
Próxima rede:   192.168.10.48
Broadcast:      192.168.10.47
```

---

## ✅ Resultado

O uso de VLSM permitiu dividir o bloco `192.168.10.0/24` em sub-redes de tamanhos diferentes, de acordo com a necessidade de cada setor.

Isso evita reservar a mesma quantidade de endereços para todos os departamentos e torna o planejamento da rede mais eficiente e escalável.
