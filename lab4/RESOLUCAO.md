# Resolução do Laboratório 4 - Man in the Middle

## Objetivo técnico

Demonstrar ataque MITM com ARP poisoning, comprovar a interceptação em captura de rede e validar alteração ativa de conteúdo de aplicação durante o tráfego.

## Arquivos ajustados

Automação aplicada em `projeto/lab11`:

- `SERVIDOR.startup`
- `VITIMA.startup`
- `MIDDLEMAN.startup`

## O que foi automatizado

### SERVIDOR

- reinício de rede;
- inicialização do `proftpd`;
- captura para `/hosthome/lab11serv.pcap`.

### VITIMA

- reinício de rede;
- captura para `/hosthome/lab11vitima.pcap`.

### MIDDLEMAN

- `ip_forward=1` para manter disponibilidade durante interceptação;
- criação do filtro FTP em `/root/pftp_filter.src`;
- compilação com `etterfilter` para `/root/pftp_filter.fil`;
- scripts operacionais:
  - `/usr/local/bin/start_mitm_capture.sh`
  - `/usr/local/bin/start_mitm_filter_capture.sh`
  - `/usr/local/bin/stop_mitm_and_export.sh`

## Como executar

```bash
cd /home/seu_usuario/nklabs
lstart -d lab11
```

Encerramento:

```bash
lhalt -d lab11
lclean -d lab11
rm -f /tmp/*.disk
```

## Exploração dos resultados

### Etapa 1: baseline ARP

Antes do ataque, coletar tabela ARP em vítima e servidor:

```bash
arp
```

Esperado:

- mapeamento IP->MAC legítimo para cada par.

### Etapa 2: MITM sem modificação de payload

Iniciar ataque no `MIDDLEMAN`:

```bash
start_mitm_capture.sh
```

Gerar tráfego na `VITIMA`:

```bash
ping -c 1 servidor
```

Coletar ARP novamente em ambos os lados.

Resultado esperado:

- IP do par passa a apontar para MAC do atacante;
- comunicação segue funcional (ataque furtivo) por causa do forwarding.

### Etapa 3: MITM com alteração de conteúdo

```bash
stop_mitm_and_export.sh
start_mitm_filter_capture.sh
```

Na vítima:

```bash
ftp servidor
```

Resultado esperado:

- banner alterado para `220 ProFTP Hacked! ...`;
- demonstra ataque ativo com manipulação em trânsito.

Encerrar e exportar:

```bash
stop_mitm_and_export.sh
```

## Evidências de captura esperadas

- `/hosthome/lab11serv.pcap`
- `/hosthome/lab11vitima.pcap`
- `/hosthome/mitm.pcap`
- `/hosthome/mitm_filter.pcap`

Leituras recomendadas no Wireshark:

- filtro `arp` para observar replies anômalos e repetitivos;
- filtro `ftp` para verificar alteração de banner;
- correlação temporal entre início do `ettercap` e mudança na tabela ARP.

## Interpretação técnica dos resultados

- O ataque é bem-sucedido quando há ao mesmo tempo:
  - alteração da tabela ARP dos alvos,
  - fluxo de aplicação preservado,
  - tráfego passando pelo middleman.
- A etapa com filtro prova capacidade de adulteração, não apenas escuta passiva.
- O cenário evidencia impacto alto quando há protocolos em claro (FTP).

## Detecção e mitigação

Medidas práticas de defesa:

- switches com DAI e DHCP Snooping;
- port-security e segmentação por VLAN;
- monitoramento de inconsistências ARP;
- tabelas ARP estáticas em ativos críticos;
- preferência por protocolos cifrados (SSH, HTTPS, SFTP, FTPS);
- IDS/IPS com regras para ARP poisoning.

## Falhas comuns e troubleshooting

- ataque derruba tráfego: verificar `ip_forward=1` no atacante;
- sem alteração no banner FTP: validar compilação/aplicação do filtro `etterfilter`;
- ARP não muda: checar interfaces e nomes de host corretos no comando de envenenamento;
- pcap vazio: confirmar permissões e caminho de saída (`/hosthome`).

## Formule as teorias (respostas)

### 1. Funcionamento do MITM observado no Wireshark

O atacante injeta ARP replies falsos para cada ponta, associando o IP do outro host ao seu próprio MAC. Com isso, vítima e servidor passam a encaminhar quadros ao atacante, que retransmite ao destino real. O tráfego continua funcional, porém interceptado. Com filtro ativo, ocorre adulteração de payload em tempo real.

### 2. Estratégia para detectar e combater em rede interna

Combinar controles de camada 2 (DAI, DHCP Snooping, port-security), segmentação e criptografia de aplicação reduz fortemente a efetividade do ataque. Monitoramento contínuo de ARP anômalo e resposta rápida completam a defesa.

### 3. Ataques combináveis com MITM

1. Sequestro de sessão: captura e reutilização de tokens/cookies.
2. DNS spoofing: respostas DNS falsas para redirecionamento malicioso.
