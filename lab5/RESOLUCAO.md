# Resolução do Laboratório 5 - Portscan, Footprinting e Honeypot

## Objetivo técnico

Mapear exposição de superfície de ataque a partir da Internet, validar publicação de serviços internos via DNAT e instrumentar honeypot para observação de tentativas de acesso.

## Base utilizada

- `docs/Aula Prática 5 - Parte1 - Portscan.pdf` (Lab XII)
- `docs/Portscan_Honeypots.pdf`
- `docs/Dicas.txt`
- `projeto/netkit_lab5-no-edisciplinas.tar.gz` (cenário `lab12`)

## Arquivos ajustados

Automação aplicada em `projeto/lab12`:

- `SERVIDOR.startup`
- `EMPRESA1.startup`
- `EMPRESA2.startup`
- `EMPRESA3.startup`

## O que foi automatizado

### EMPRESA1, EMPRESA2 e EMPRESA3

Serviços inicializados automaticamente:

- Apache (`apache2`)
- Proxy (`squid`)
- DNS (`bind`)
- FTP (`proftpd`)
- SSH (`ssh`)

### SERVIDOR

- `ip_forward=1`;
- limpeza das tabelas `filter`, `nat`, `mangle`;
- NAT de saída (`MASQUERADE`);
- DNAT e encaminhamento para serviços internos;
- SSH local ativo;
- captura em `/hosthome/lab12.pcap`.

### Honeypot

- criação de `/etc/honeypot/mel.conf`;
- script `/usr/local/bin/start_honeypot.sh`;
- script `/usr/local/bin/stop_honeypot.sh`.

Nota: o comando correto de inicialização é com `honeyd`.

## Tabela de redirecionamento aplicada

| Serviço | Destino | Porta pública | Porta interna |
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

```bash
cd /home/seu_usuario/nklabs
lstart -d lab12
```

Encerrar:

```bash
lhalt -d lab12
lclean -d lab12
rm -f /tmp/*.disk
```

## Exploração dos resultados

### Validação 1: isolamento da rede privada

Da `INTERNET`:

```bash
ping 10.1.1.12
```

Esperado:

- sem conectividade direta para IP privado.

Interpretação:

- confirma que acesso externo depende de publicação explícita no gateway.

### Validação 2: enumeração de portas expostas

```bash
nmap -sS -O 202.135.187.131
nmap -p 1-1500 -sS -O 202.135.187.131
```

Esperado:

- portas redirecionadas aparecem abertas;
- portas nao publicadas permanecem fechadas/filtradas.

Interpretação:

- compara configuração planejada com superfície realmente visível para atacante externo.

### Validação 3: comprovação de redirecionamento por serviço

```bash
ssh joaquim@202.135.187.131
ssh joaquim@202.135.187.131 -p 1122
telnet 202.135.187.131 1180
```

No HTTP via telnet:

```text
HEAD / HTTP/1.0

```

Esperado:

- acesso sem porta customizada atende serviço local do gateway (quando aplicável);
- acesso na porta publicada cai no host interno correspondente;
- banner/resposta HTTP condiz com servidor de destino.

### Validação 4: captura e evidências

- arquivo esperado: `/hosthome/lab12.pcap`;
- filtros úteis no Wireshark:
	- `tcp.port == 1122`
	- `tcp.port == 1180`
	- `ip.addr == 10.1.1.11`

Leitura recomendada:

- identificar mudança de destino após passagem pelo gateway;
- confirmar relação entre tentativa externa e resposta interna.

## Etapa Honeypot

No `SERVIDOR`:

```bash
start_honeypot.sh
```

Parar:

```bash
stop_honeypot.sh
```

Comando manual equivalente:

```bash
honeyd -i eth0 -d -f /etc/honeypot/mel.conf
```

Resultados a observar:

- conexões de reconhecimento chegando ao honeypot;
- possibilidade de identificar padrão de scan por portas e sequência de tentativas.

## Análise técnica

- DNAT permite publicação granular de serviços sem expor toda a rede.
- A mesma técnica pode ampliar risco se houver serviço vulnerável internamente.
- Portscan mostra a "verdade externa" da configuração, útil para auditoria contínua.
- Honeypot agrega visibilidade e inteligência sobre tentativas de intrusão.

## Falhas comuns e troubleshooting

- porta esperada não aparece no nmap: revisar regra DNAT e serviço interno;
- conexão abre e cai: verificar retorno de rota e regras de FORWARD;
- honeypot sem eventos: validar interface correta e processo em execução;
- captura incompleta: confirmar ponto de captura e filtro aplicado.

## Formule as teorias (respostas)

### 1. Funcionamento do redirecionamento de portas

O pacote chega ao IP público, passa por `PREROUTING` com DNAT (troca destino para host interno) e segue via `FORWARD`. A resposta retorna pelo gateway e mantém consistência de sessão para o cliente externo.

### 2. Comandos úteis de iptables e nmap

`iptables`:

- `iptables -L -n -v`
- `iptables -t nat -L -n -v`
- `iptables -D ...`
- `-m conntrack --ctstate ESTABLISHED,RELATED`
- `-m limit --limit 10/second`

`nmap`:

- `-sV`, `-sC`, `-sU`, `-p-`, `-T1..-T5`
- saídas `-oN`, `-oG`, `-oX`, `-oA`

### 3. Outras formas de levantamento de informações

- enumeração DNS;
- WHOIS/ASN;
- fingerprint web;
- enumeração SNMP exposta;
- análise passiva de tráfego;
- banner grabbing e metadados públicos.

### 4. Bloqueio de portscan e limitações

Rate limit, listas dinâmicas e IDS/IPS ajudam, mas scans lentos/distribuídos e técnicas evasivas reduzem eficácia. O mais robusto é defesa em camadas com mínima exposição e monitoramento contínuo.

### 5. Uso de scanners no laboratório

Scanners como Nmap, Masscan, Unicornscan e RustScan servem para validar exposição real, comparar antes/depois de regras e medir aderência da configuração ao desenho de segurança.

## Conclusão

O lab foi consolidado no cenário `lab12` com foco em:

- footprinting e portscan externo;
- publicação controlada de serviços por DNAT;
- verificação de resultados em captura de tráfego;
- instrumentação de honeypot para observabilidade.
