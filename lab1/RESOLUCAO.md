# Resolução do Laboratório 1 - NAT e Firewall

## Objetivo técnico

Validar, em ambiente Netkit, três comportamentos de segurança de rede:

- saída de rede interna para Internet usando NAT;
- publicação controlada de serviços internos com DNAT;
- bloqueio explícito de serviço sensível com regra de filtro.

## Arquivos ajustados

As mudanças foram aplicadas em `projeto/lab05`:

- `SERVIDOR.startup`
- `EMPRESA1.startup`
- `EMPRESA3.startup`

Resumo da automação:

- no `SERVIDOR`: forwarding, NAT, DNAT e bloqueio de porta;
- na `EMPRESA1`: inicialização de SSH;
- na `EMPRESA3`: inicialização de FTP.

## Como executar

```bash
cd /home/seu_usuario/nklabs
lstart -d lab05
```

Encerramento:

```bash
lhalt -d lab05
lclean -d lab05
```

## Configuração aplicada no SERVIDOR

Topologia lógica usada no gateway:

- `eth0`: rede interna `10.1.1.0/8` (IP do servidor `10.1.1.1`)
- `eth1`: rede externa `202.135.187.128/26` (IP do servidor `202.135.187.131`)
- gateway padrão: `202.135.187.129`

Regras configuradas:

```bash
/etc/init.d/proftpd start
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -F
iptables -F -t nat
iptables -F -t mangle
iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
iptables -t nat -A PREROUTING -p tcp --dport 21 -j DNAT --to 10.1.1.13
iptables -t nat -A PREROUTING -p tcp --dport 22 -j DNAT --to 10.1.1.11
iptables -A INPUT -p tcp --dport 20 -j REJECT
```

## Exploração dos resultados

### Validação funcional (esperado x observado)

1. Acesso externo ao IP público do `SERVIDOR`
- esperado: conectividade normal
- observado: acesso funcional

2. Acesso externo direto a host interno (`10.1.1.12`)
- esperado: sem acesso por ser endereço privado
- observado: falha de conexão, confirmando isolamento

3. Saída da rede interna para Internet
- esperado: hosts internos navegam/atingem destinos externos via MASQUERADE
- observado: tráfego de saída concluído quando passa pelo `SERVIDOR`

4. Publicação de FTP interno (porta pública 21)
- esperado: conexão externa chega na `EMPRESA3`
- observado: DNAT operacional para `10.1.1.13:21`

5. Publicação de SSH interno (porta pública 22)
- esperado: conexão externa chega na `EMPRESA1`
- observado: DNAT operacional para `10.1.1.11:22`

6. Bloqueio da porta 20 no `SERVIDOR`
- esperado: recusa explícita
- observado: tentativa retorna rejeição, conforme regra de `INPUT`

### Evidências recomendadas para relatório

Comandos úteis para comprovação:

```bash
iptables -t nat -L -n -v
iptables -L -n -v
tcpdump -ni any tcp port 21 or tcp port 22 or tcp port 20
```

Interpretação objetiva:

- aumento de contadores na regra `POSTROUTING/MASQUERADE` confirma tradução de origem;
- aumento de contadores em `PREROUTING/DNAT` confirma publicação dos serviços;
- aumento do contador em `INPUT dpt:20 REJECT` confirma bloqueio efetivo.

## Análise de segurança

- NAT reduz exposição direta da rede privada, mas nao substitui política de firewall.
- DNAT amplia superfície de ataque porque expõe serviços internos para fora.
- FTP transmite dados sensíveis em texto claro; ideal migrar para SFTP/FTPS.
- O bloqueio pontual de porta funciona, mas o ideal é política padrão restritiva com exceções explícitas.

## Falhas comuns e diagnóstico rápido

- Hosts internos sem saída: verificar `ip_forward=1` e gateway padrão dos clientes.
- DNAT sem efeito: conferir interface correta, rota de retorno e regra de `FORWARD` (se política padrão for DROP).
- Teste FTP inconsistente: validar modo ativo/passivo e portas associadas.

## Respostas - Formule as teorias

### 1. Por que a EMPRESA2 captura tráfego mesmo sem ser origem/destino?

Porque o cenário interno se comporta como meio compartilhado (hub). Nesse modelo, quadros são replicados para múltiplas portas e uma estação em modo promíscuo pode observar tráfego alheio. Em FTP, o conteúdo aparece em claro; em SSH, os pacotes são visíveis, porém cifrados.

Em rede sem fio fraca, o risco aumenta: o atacante pode capturar tráfego a distância, sem acesso físico ao cabo, e explorar criptografia fraca/configuração inadequada.

### 2. Melhorias para rede cabeada e sem fio

Cabeada:

- substituir hub por switch gerenciável;
- segmentar por VLAN e aplicar ACL entre segmentos;
- adotar 802.1X/NAC e mínimo privilégio;
- remover protocolos legados inseguros.

Sem fio:

- usar WPA2/WPA3 com autenticação robusta;
- desativar WPS;
- separar rede administrativa e convidados;
- manter firmware e políticas de senha atualizados.

### 3. Como cada regra do iptables afeta os pacotes

- `-F` nas tabelas: remove regras anteriores, sem reescrita direta de pacote.
- `POSTROUTING MASQUERADE`: reescreve IP de origem na saída pública.
- `PREROUTING DNAT` (21 e 22): reescreve IP de destino para host interno.
- `INPUT ... REJECT` (porta 20): nega entrega local e responde com rejeição.

### 4. Efeito do bloqueio da porta 20

O serviço local na porta 20 deixa de aceitar conexões externas. Já o fluxo de FTP publicado na porta 21 continua, pois segue cadeia de DNAT para a `EMPRESA3`.
