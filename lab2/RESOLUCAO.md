# Resolução do Laboratório 2 - VPNs

## Objetivo técnico

Implementar e validar um túnel VPN site-to-site entre duas redes privadas, com autenticação operacional de acesso e roteamento fim a fim entre os dois domínios.

## Arquivos ajustados

Automação aplicada em `projeto/lab2`:

- `EMPRESA1A.startup`
- `SERVIDORA.startup`
- `SERVIDORB.startup`

## O que foi automatizado

### EMPRESA1A

- chave privada SSH do usuário `joaquim` em `/home/joaquim/.ssh/id_rsa`;
- geração de `id_rsa.pub`;
- permissões corretas em `.ssh`;
- ajuste de cliente SSH para evitar bloqueio por prompt interativo.

### SERVIDORA (matriz)

- `ip_forward=1`;
- NAT para saída externa;
- regras de `FORWARD` entre LAN e túnel;
- configuração de `authorized_keys` para autenticação por chave;
- criação de `/etc/openvpn/minhavpn.key`;
- criação de `/etc/openvpn/matriz.conf`;
- OpenVPN em daemon;
- rota para `192.168.2.0/24` via `10.0.0.2`.

### SERVIDORB (filial)

- `ip_forward=1`;
- NAT para saída externa;
- regras de `FORWARD` entre LAN e túnel;
- mesma chave estática OpenVPN;
- criação de `/etc/openvpn/filial.conf`;
- OpenVPN em daemon;
- rota para `192.168.1.0/24` via `10.0.0.1`.

## Como executar

```bash
cd /home/seu_usuario/nklabs
lstart -d lab2
```

Encerrar:

```bash
lhalt -d lab2
lclean -d lab2
```

## Exploração dos resultados

### Validação 1: autenticação SSH sem senha

Teste:

```bash
su joaquim
ssh joaquim@192.168.1.1
```

Resultado esperado:

- acesso sem prompt de senha;
- confirma distribuição correta de chave pública/privada.

### Validação 2: túnel OpenVPN ativo

Testes básicos entre peers:

```bash
# em SERVIDORA
ping 10.0.0.2

# em SERVIDORB
ping 10.0.0.1
```

Interpretação:

- ping com resposta confirma conectividade de túnel;
- perda total indica problema de chave, processo OpenVPN, rota ou filtro.

### Validação 3: tráfego entre redes privadas

Após subida do túnel, testar hosts finais:

- `EMPRESA1A` para host da rede `192.168.2.0/24`;
- host da rede `192.168.2.0/24` para `EMPRESA1A`.

Resultado esperado:

- conectividade bidirecional entre sub-redes;
- ausência de assimetria de rota.

### Evidências recomendadas para relatório

```bash
ip a
ip route
ps aux | grep -i openvpn
iptables -L -n -v
iptables -t nat -L -n -v
tcpdump -ni any host 10.0.0.1 or host 10.0.0.2
```

Pontos de leitura:

- presença da interface de túnel e processos OpenVPN;
- rotas para rede remota apontando para o peer correto;
- contadores de `FORWARD` subindo durante tráfego intersite.

## Ajustes de interpretação das dicas

- passo interativo de geração/cópia de chave foi absorvido na automação;
- uso de `10.0.0.1` e `10.0.0.2` como endpoints de túnel foi mantido conforme topologia.

## Análise de segurança

- chave estática simplifica o laboratório, mas TLS com PKI é superior para produção;
- túnel garante confidencialidade em trânsito entre sites;
- sem política de filtro por serviço, o túnel pode transportar tráfego além do necessário.

## Falhas comuns e troubleshooting

- túnel nao sobe: validar arquivos `.conf`, porta/protocolo e permissões da chave;
- rota sem retorno: checar tabela de rotas nos dois lados;
- ping entre LANs falha com túnel ativo: revisar regras de `FORWARD` e possível NAT indevido sobre tráfego interno;
- SSH por chave pedindo senha: conferir `authorized_keys`, owner e permissões.

## Exercícios (respostas)

### 1. Outras ferramentas para VPN

- WireGuard
- strongSwan (IPsec)
- SoftEther VPN
- Tinc
- ZeroTier
- OpenConnect

### 2. Adaptação para modo TLS no OpenVPN

No modo TLS, substitui-se chave estática por PKI com CA, certificados e chaves por endpoint, podendo incluir `tls-auth`/`tls-crypt` para endurecimento adicional.

Exemplo de servidor:

```conf
port 5000
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
server 10.8.0.0 255.255.255.0
tls-auth ta.key 0
keepalive 10 60
persist-key
persist-tun
verb 3
```

Exemplo de cliente:

```conf
client
dev tun
proto udp
remote 143.102.212.10 5000
ca ca.crt
cert client.crt
key client.key
tls-auth ta.key 1
nobind
persist-key
persist-tun
verb 3
```

### 3. Vantagem do OpenVPN sobre túnel SSH para interligar redes

OpenVPN foi desenhado para VPN de rede, com melhor gestão de rotas, escala e estabilidade para topologias site-to-site em comparação com túnel SSH ad hoc.
