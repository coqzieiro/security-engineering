# Resolução do Laboratório 5 - Portscan, Footprinting e Honeypot

## Base utilizada

Esta resolução foi construída com base em:

- `docs/Aula Prática 5 - Parte1 - Portscan.pdf` (Lab XII)
- `docs/Portscan_Honeypots.pdf` (teoria de portscan + honeypots)
- `docs/Dicas.txt` (correção do comando de honeypot)
- pacote `projeto/netkit_lab5-no-edisciplinas.tar.gz` (cenário `lab12`)

## Arquivos ajustados

A automação foi aplicada em `projeto/lab12`:

- `SERVIDOR.startup`
- `EMPRESA1.startup`
- `EMPRESA2.startup`
- `EMPRESA3.startup`

## O que foi automatizado

### EMPRESA1, EMPRESA2 e EMPRESA3

Além da configuração de IP e rota padrão, os serviços do exercício já sobem automaticamente:

- Apache (`apache2`)
- Proxy (`squid`)
- DNS (`bind`)
- FTP (`proftpd`)
- SSH (`ssh`)

### SERVIDOR

Foi automatizado o cenário completo do Lab XII:

- ativação de encaminhamento IP (`ip_forward`)
- limpeza de regras antigas (`filter`, `nat`, `mangle`)
- NAT/MASQUERADE para saída pela interface `eth1`
- redirecionamento de portas (DNAT + FORWARD) para EMPRESA1, EMPRESA2 e EMPRESA3
- inicialização do SSH no próprio servidor
- captura de tráfego em `/hosthome/lab12.pcap`

Também foi preparada a parte de honeypot:

- criação de `/etc/honeypot/mel.conf`
- criação do script `/usr/local/bin/start_honeypot.sh`
- criação do script `/usr/local/bin/stop_honeypot.sh`

Importante: conforme `docs/Dicas.txt`, o comando correto para iniciar o honeypot é com `honeyd`, não com `iptables`.

## Tabela de redirecionamento aplicada

| Serviço | Destino | Porta origem (pública) | Porta destino (interna) |
|---|---|---:|---:|
| SSH | 10.1.1.11 (EMPRESA1) | 1122 | 22 |
| FTP | 10.1.1.11 (EMPRESA1) | 1121 | 21 |
| HTTP | 10.1.1.11 (EMPRESA1) | 1180 | 80 |
| SSH | 10.1.1.12 (EMPRESA2) | 1222 | 22 |
| FTP | 10.1.1.12 (EMPRESA2) | 1221 | 21 |
| HTTP | 10.1.1.12 (EMPRESA2) | 1280 | 80 |
| SSH | 10.1.1.13 (EMPRESA3) | 1322 | 22 |
| FTP | 10.1.1.13 (EMPRESA3) | 1321 | 21 |
| HTTP | 10.1.1.13 (EMPRESA3) | 1380 | 80 |

## Como executar

Na VM Netkit:

```bash
cd /home/seu_usuario/nklabs
# extraia/copiei a pasta lab12 de projeto para cá
lstart -d lab12
```

Encerrar:

```bash
lhalt -d lab12
lclean -d lab12
rm -f /tmp/*.disk
```

## Testes esperados (Portscan e Footprinting)

1. Da `INTERNET`, testar conectividade interna direta:

```bash
ping 10.1.1.12
```

Esperado: não alcança rede privada diretamente.

2. Da `INTERNET`, escanear o IP público do servidor:

```bash
nmap -sS -O 202.135.187.131
nmap -p 1-1500 -sS -O 202.135.187.131
```

Esperado: portas redirecionadas aparecem abertas após automação.

3. Testar SSH no próprio firewall e SSH redirecionado:

```bash
ssh joaquim@202.135.187.131
ssh joaquim@202.135.187.131 -p 1122
```

Esperado: na segunda conexão, acesso ao conteúdo da EMPRESA1.

4. Testes de coleta de banner e serviço:

```bash
telnet 202.135.187.131 22
ssh -vN 202.135.187.131
telnet 202.135.187.131 1180
```

No telnet HTTP, usar:

```text
HEAD / HTTP/1.0

```

5. Validar captura:

- arquivo esperado em `/hosthome/lab12.pcap`
- analisar no Wireshark com filtros por porta (`tcp.port == 1122`, `1180`, etc.)

## Etapa Honeypot (mesmo cenário lab12)

No `SERVIDOR`:

```bash
start_honeypot.sh
```

Parar:

```bash
stop_honeypot.sh
```

Comando equivalente correto (manual):

```bash
honeyd -i eth0 -d -f /etc/honeypot/mel.conf
```

## Formule as teorias (respostas)

### 1. Explique o funcionamento dos comandos de redirecionamento de portas

O redirecionamento combina duas regras:

- `PREROUTING` na tabela `nat` com `DNAT`: altera o IP/porta de destino do pacote ao chegar na interface pública.
- `FORWARD` na tabela `filter`: permite encaminhar o pacote para o host interno alvo.

Exemplo conceitual:

- cliente externo conecta em `202.135.187.131:1122`
- `DNAT` reescreve para `10.1.1.11:22`
- `FORWARD` permite o tráfego
- resposta volta e o NAT mantém a sessão consistente para o cliente externo

### 2. Comandos interessantes de iptables e opções do nmap

`iptables` úteis:

- listar regras com contadores: `iptables -L -n -v`, `iptables -t nat -L -n -v`
- remover regra específica: `iptables -D ...`
- política padrão restritiva: `iptables -P INPUT DROP` (com regras de exceção)
- limitar taxa para mitigar abuso: `-m limit --limit 10/second`
- rastrear estado: `-m conntrack --ctstate ESTABLISHED,RELATED`

`nmap` úteis:

- detecção de serviços/versão: `-sV`
- scripts NSE: `-sC` ou `--script <nome>`
- scan UDP: `-sU`
- scan completo de portas TCP: `-p-`
- ajuste de timing: `-T1` até `-T5`
- saída para arquivo: `-oN`, `-oG`, `-oX`, `-oA`

### 3. Outras formas de levantar informações de uma possível vítima

- DNS enumeração (`dig`, `nslookup`, transferência de zona mal configurada)
- WHOIS e dados de ASN/blocos de IP
- varredura web (headers, robots, diretórios, fingerprint de CMS)
- SNMP quando exposto (`snmpwalk`)
- análise passiva de tráfego (sniffing)
- coleta de metadados de documentos públicos
- banner grabbing em serviços TCP

### 4. Bloqueio de port-scan e por que nem sempre é efetivo

Configurações possíveis:

- rate limit por IP e por destino
- detecção por padrões de SYN/FIN/NULL/XMAS
- bloqueio temporário com `recent`/`hashlimit`
- fail2ban e IDS/IPS com assinaturas de scan

Limitações:

- scanners lentos/distribuídos driblam limites simples
- falsos positivos podem bloquear tráfego legítimo
- técnicas evasivas (decoys, fragmentação, timing) reduzem eficácia

Melhor abordagem:

- defesa em camadas: firewall + IDS/IPS + segmentação + monitoramento contínuo + hardening de serviços
- reduzir superfície de ataque (fechar portas, restringir exposição, patching)

### 5. Sobre scanners de rede e uso em laboratório

Scanners comuns:

- Nmap/Zenmap
- Masscan
- Unicornscan
- RustScan

No laboratório com roteamento ativo, o scanner ajuda a:

- validar superfície exposta depois do NAT/DNAT
- confirmar efeito de mudanças de firewall
- comparar visibilidade externa versus interna da rede

## Conclusão

O lab 5 foi resolvido de forma completa no cenário disponível (`lab12`), cobrindo:

- footprinting e portscan externo
- NAT e redirecionamento de portas para serviços internos
- análise de tráfego capturado
- preparação e execução de honeypot com comando corrigido (`honeyd`)
