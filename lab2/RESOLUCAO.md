# Resolução do Laboratório 2 — VPNs

## Arquivos ajustados

A resolução foi aplicada diretamente nos scripts de inicialização do laboratório em `projeto/lab2`:

- `EMPRESA1A.startup`
- `SERVIDORA.startup`
- `SERVIDORB.startup`

## O que foi automatizado

### EMPRESA1A

- Configuração de IP e gateway (já existente).
- Criação da chave privada SSH do usuário `joaquim` em `/home/joaquim/.ssh/id_rsa`.
- Geração da chave pública correspondente (`id_rsa.pub`).
- Ajuste de permissões de `.ssh`.
- Configuração de cliente SSH para não travar com prompt interativo de host key.

### SERVIDORA (Matriz)

- IP forwarding ativado.
- NAT para saída na interface pública.
- Regras de `FORWARD` para tráfego entre LAN e túnel VPN.
- Serviço SSH iniciado.
- `authorized_keys` do usuário `joaquim` configurado com a chave pública da `EMPRESA1A`.
- Chave estática OpenVPN criada em `/etc/openvpn/minhavpn.key`.
- Arquivo `/etc/openvpn/matriz.conf` criado.
- OpenVPN iniciado em modo daemon.
- Rota para rede da empresa B: `192.168.2.0/24` via `10.0.0.2`.

### SERVIDORB (Filial)

- IP forwarding ativado.
- NAT para saída na interface pública.
- Regras de `FORWARD` para tráfego entre LAN e túnel VPN.
- Mesma chave estática OpenVPN criada em `/etc/openvpn/minhavpn.key`.
- Arquivo `/etc/openvpn/filial.conf` criado.
- OpenVPN iniciado em modo daemon.
- Rota para rede da empresa A: `192.168.1.0/24` via `10.0.0.1`.

## Como executar

Na VM Netkit:

```bash
cd /home/seu_usuario/nklabs
lstart -d lab2
```

Encerrar:

```bash
lhalt -d lab2
lclean -d lab2
```

## Testes esperados

1. Teste SSH da `EMPRESA1A` para `SERVIDORA`:

```bash
su joaquim
ssh joaquim@192.168.1.1
```

Resultado esperado: acesso sem senha (chave já configurada).

2. Teste VPN entre servidores:

No `SERVIDORA`:

```bash
ping 10.0.0.2
```

No `SERVIDORB`:

```bash
ping 10.0.0.1
```

3. Teste comunicação entre redes após VPN e rotas:

- Da rede A para a rede B (ex.: `EMPRESA1A` para `EMPRESA1B`).
- Da rede B para a rede A (ex.: `EMPRESA1B` para `EMPRESA1A`).

## Ajustes de interpretação das dicas

- O passo 6 (do PDF) foi eliminado de forma interativa ao deixar a chave SSH já pronta.
- O passo 9 (copiar chave via `/hosthome`) não é mais necessário para o cenário resolvido.
- O passo 20 foi considerado corretamente com `10.0.0.1` e `10.0.0.2` (e não `10.0.0.0`).

## Exercícios (respostas)

### 1. Outras ferramentas para montar VPN

- WireGuard
- IPsec (strongSwan)
- SoftEther VPN
- Tinc
- ZeroTier
- OpenConnect

### 2. Modo TLS no OpenVPN (resumo de como adaptar)

No modo TLS, em vez de uma chave estática compartilhada (`secret`), usa-se uma PKI com:

- Autoridade certificadora (CA)
- Certificado/chave do servidor
- Certificado/chave do cliente
- Opcionalmente `tls-auth` ou `tls-crypt`

Exemplo de diretivas no servidor:

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

Exemplo no cliente:

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

### 3. Vantagem do OpenVPN sobre SSH para interligar redes

- OpenVPN é pensado para VPN de rede (camada 3/2), não apenas túnel de sessão.
- Escala melhor para múltiplos clientes e sub-redes.
- Tem mecanismos próprios de gestão de túnel, rotas e reconexão.
- Permite topologias site-to-site de forma mais limpa.
- Em cenários corporativos, costuma ser mais administrável e auditável que túneis SSH improvisados.
