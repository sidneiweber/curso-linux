+++
title = "Modelos OSI e TCP/IP"
weight = 1
description = "Camadas, encapsulamento e onde cada protocolo se encaixa."
+++

Redes são organizadas em **camadas**: cada uma resolve um problema e oferece serviço à de cima.

| OSI | Camada | TCP/IP (DoD) | Exemplos | Unidade |
|---|---|---|---|---|
| 7 | Aplicação | **Aplicação** | HTTP, DNS, SSH, SMTP, DHCP | Mensagem |
| 6 | Apresentação | ↑ | TLS, codificação | |
| 5 | Sessão | ↑ | | |
| 4 | Transporte | **Transporte** | TCP, UDP | Segmento |
| 3 | Rede | **Internet** | IP, ICMP | Pacote |
| 2 | Enlace | **Acesso à rede** | Ethernet, Wi-Fi, ARP | Quadro |
| 1 | Física | ↑ | Cabo, fibra, rádio | Bits |

O **OSI** (7 camadas) é um modelo de referência, ótimo para conversar sobre problemas
("é camada 2 ou camada 3?"). O **TCP/IP** — também chamado de modelo **DoD**, por ter nascido
em um projeto do Departamento de Defesa dos EUA — é o que a Internet de fato usa.

## Encapsulamento

Ao enviar uma requisição web, cada camada acrescenta seu cabeçalho:

```text
[ Ethernet | IP | TCP | HTTP: GET /index.html ]
   MACs     IPs  portas   dados da aplicação
```

O destino desfaz o processo na ordem inversa. Roteadores olham só até a camada 3 (IP);
switches, até a camada 2 (MAC).

## Diagnóstico por camada

| Camada | Pergunta | Ferramenta |
|---|---|---|
| 1–2 | O cabo/link está ativo? | `ip link`, `ethtool eth0` |
| 2 | Encontro os vizinhos? | `ip neigh` |
| 3 | Tenho IP e rota? Alcanço o destino? | `ip addr`, `ip route`, `ping`, `traceroute` |
| 4 | A porta está aberta? | `ss -tulpn`, `nc -zv host 443` |
| 7 | O serviço responde certo? | `curl -v`, `dig` |

Comece por baixo: não adianta depurar o DNS se a interface está sem link.
