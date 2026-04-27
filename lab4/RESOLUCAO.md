# Resolução do Laboratório 4 - Man in the Middle

## Arquivos ajustados

A resolução foi aplicada diretamente nos scripts de inicialização do laboratório Netkit em `projeto/lab11`:

- `SERVIDOR.startup`
- `VITIMA.startup`
- `MIDDLEMAN.startup`

## O que foi automatizado

### SERVIDOR

- Reinício de rede.
- Inicialização do serviço `proftpd`.
- Captura de tráfego em background para `/hosthome/lab11serv.pcap`.

### VITIMA

- Reinício de rede.
- Captura de tráfego em background para `/hosthome/lab11vitima.pcap`.

### MIDDLEMAN

- Reinício de rede.
- Habilitação de encaminhamento IP (`ip_forward=1`) para manter o tráfego fluindo durante MITM.
- Criação automática do filtro de alteração de banner FTP em `/root/pftp_filter.src`.
- Compilação do filtro para `/root/pftp_filter.fil` com `etterfilter`.
- Criação de scripts utilitários:
  - `/usr/local/bin/start_mitm_capture.sh` (MITM com ARP poisoning sem filtro)
  - `/usr/local/bin/start_mitm_filter_capture.sh` (MITM com ARP poisoning e filtro)
  - `/usr/local/bin/stop_mitm_and_export.sh` (encerra ataques e copia `mitm*.pcap` para `/hosthome`)

## Como executar

Na VM Netkit:

```bash
cd /home/seu_usuario/nklabs
# copie/extrai o conteúdo de projeto/lab11 para esta pasta
lstart -d lab11
```

Encerrar:

```bash
lhalt -d lab11
lclean -d lab11
rm -f /tmp/*.disk
```

## Roteiro de teste objetivo

1. Confirmar estado inicial da ARP na vítima e no servidor:

```bash
arp
```

2. No `MIDDLEMAN`, iniciar MITM sem filtro:

```bash
start_mitm_capture.sh
```

3. Na `VITIMA`, gerar tráfego:

```bash
ping -c 1 servidor
```

4. Conferir ARP em `VITIMA` e `SERVIDOR`:

```bash
arp
```

Esperado: mudança do MAC associado ao IP do outro host (envenenamento ARP).

5. No `MIDDLEMAN`, parar ataque atual:

```bash
stop_mitm_and_export.sh
```

6. No `MIDDLEMAN`, iniciar MITM com filtragem:

```bash
start_mitm_filter_capture.sh
```

7. Na `VITIMA`, acessar FTP:

```bash
ftp servidor
```

Esperado no banner: `220 ProFTP Hacked! ...`.

8. Encerrar e exportar capturas:

```bash
stop_mitm_and_export.sh
```

## Arquivos de captura esperados

- `/hosthome/lab11serv.pcap`
- `/hosthome/lab11vitima.pcap`
- `/hosthome/mitm.pcap`
- `/hosthome/mitm_filter.pcap`

## Formule as teorias (respostas)

### 1. Através da observação do conteúdo do Wireshark, explique o funcionamento do ataque MITM

No ataque MITM por ARP poisoning, o atacante envia respostas ARP falsas para os dois lados:

- para a vítima, dizendo que o IP do servidor possui o MAC do atacante;
- para o servidor, dizendo que o IP da vítima possui o MAC do atacante.

Com isso, os dois hosts passam a enviar quadros Ethernet para o atacante. O atacante então encaminha os pacotes ao destino real (com `ip_forward` habilitado), ficando no meio da comunicação sem interrompê-la. No Wireshark isso aparece como:

- pacotes ARP de atualização/reply suspeitos e repetidos;
- alteração de mapeamentos IP->MAC nas tabelas ARP;
- tráfego de aplicação da vítima/servidor atravessando a interface do middleman.

No caso com filtro do ettercap, além de interceptar, o atacante modifica o payload em trânsito (banner `ProFTPD` para `ProFTP Hacked!`).

### 2. Estratégia para detectar e combater esse ataque em rede interna

Medidas práticas de defesa:

- substituir hubs por switches gerenciáveis com proteção de camada 2;
- habilitar Dynamic ARP Inspection (DAI) e DHCP Snooping quando disponível;
- usar port-security (limite de MAC por porta) e segmentação por VLAN;
- monitorar anomalias ARP (duplicidade de MAC para IPs críticos, volume anormal de ARP replies);
- usar tabelas ARP estáticas para ativos críticos quando viável;
- priorizar protocolos criptografados (SSH, HTTPS, SFTP, FTPS) para reduzir impacto de interceptação;
- manter IDS/IPS e alertas de variações de latência/trajeto em fluxos internos.

### 3. Outros ataques que podem ser conjugados com MITM além da filtragem

1. Sequestro de sessão (session hijacking):
O atacante captura cookies/tokens de sessão em protocolos sem proteção adequada e reutiliza esses artefatos para assumir sessões autenticadas.

2. DNS spoofing (injeção/forja de respostas DNS):
Durante o MITM, o atacante responde consultas DNS com IP falso, redirecionando a vítima para serviços maliciosos (phishing, malware, coleta de credenciais).
