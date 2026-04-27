# Resolução do Laboratório 1 - NAT e Firewall

## Arquivos ajustados

A resolução foi aplicada diretamente nos scripts de inicialização do laboratório Netkit em `projeto/lab05`:

- `SERVIDOR.startup`: habilita FTP, roteamento IP, NAT/MASQUERADE, DNAT para FTP/SSH internos e bloqueio da porta 20.
- `EMPRESA1.startup`: inicia o serviço SSH.
- `EMPRESA3.startup`: inicia o serviço FTP.

Assim, ao iniciar o laboratório, os comandos principais do roteiro já ficam aplicados automaticamente.

## Como executar

Dentro da VM do Netkit, na pasta onde está o laboratório:

```bash
cd /home/seu_usuario/nklabs
lstart -d lab05
```

Para encerrar:

```bash
lhalt -d lab05
lclean -d lab05
```

## Configuração aplicada no SERVIDOR

O `SERVIDOR` fica com duas interfaces:

- `eth0`: rede interna da empresa - `10.1.1.1/8`
- `eth1`: rede pública - `202.135.187.131/26`
- gateway padrão: `202.135.187.129`

Comandos automatizados:

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

## Testes esperados

### Antes do NAT, conforme roteiro original

- `INTERNET -> SERVIDOR` em `202.135.187.131`: funciona.
- `INTERNET -> EMPRESA2` em `10.1.1.12`: não funciona, pois é rede privada/interna.
- `SERVIDOR -> INTERNET` em `143.102.212.100`: funciona.
- `EMPRESA1 -> INTERNET` em `143.102.212.100`: inicialmente não funcionaria sem NAT.

### Com a resolução aplicada automaticamente

- `EMPRESA1`, `EMPRESA2` e `EMPRESA3` conseguem acessar a Internet por NAT através do `SERVIDOR`.
- A porta pública `21` do `SERVIDOR` é redirecionada para o FTP da `EMPRESA3` (`10.1.1.13`).
- A porta pública `22` do `SERVIDOR` é redirecionada para o SSH da `EMPRESA1` (`10.1.1.11`).
- A porta `20` local do `SERVIDOR` é rejeitada por firewall.

## Respostas - Formule as teorias

### 1. Explique o conteúdo obtido pelo `tcpdump` na máquina EMPRESA2, sendo que ela não foi destino nem origem da comunicação. E se fosse uma rede sem fio com proteção fraca?

A `EMPRESA2` consegue capturar tráfego porque a rede interna do laboratório usa um hub. Em um hub, os quadros recebidos em uma porta são repetidos para todas as portas, então máquinas que não são origem nem destino também recebem cópias dos pacotes. Por isso, o `tcpdump` na `EMPRESA2` pode observar comunicações como FTP e SSH feitas por outras máquinas.

No caso do FTP, credenciais e comandos podem aparecer em texto claro, pois FTP não criptografa a sessão. Já no SSH, a captura mostra pacotes criptografados, mas não revela diretamente usuário, senha ou conteúdo.

Se fosse uma rede sem fio com proteção fraca, o risco seria ainda maior: um atacante próximo poderia capturar pacotes pelo ar sem precisar estar conectado fisicamente ao cabo ou ao hub. Com criptografia fraca ou senha ruim, seria possível quebrar ou contornar a proteção e observar tráfego da rede.

### 2. Quais seriam opções para melhorar a rede interna cabeada? E sem fio?

Na rede cabeada:

- Substituir hub por switch, reduzindo o vazamento de tráfego entre portas.
- Separar setores por VLANs.
- Usar firewall entre segmentos internos.
- Usar autenticação de porta, como IEEE 802.1X.
- Evitar protocolos inseguros, como FTP e Telnet.
- Usar SSH, SFTP, HTTPS e VPN quando necessário.
- Aplicar senhas fortes e controle de acesso por usuário.

Na rede sem fio:

- Usar WPA2 ou WPA3.
- Evitar WEP e WPA antigo.
- Usar senha forte ou autenticação corporativa WPA-Enterprise/802.1X.
- Separar rede de visitantes da rede administrativa.
- Desativar WPS.
- Atualizar firmware dos equipamentos.
- Posicionar e configurar o sinal para reduzir exposição externa desnecessária.

### 3. Como os pacotes são modificados em cada regra do `iptables` executada no laboratório?

- `iptables -F`: limpa as regras da tabela `filter`; não modifica pacotes diretamente, apenas remove regras existentes.
- `iptables -F -t nat`: limpa as regras da tabela `nat`; remove traduções anteriores.
- `iptables -F -t mangle`: limpa a tabela `mangle`; remove alterações especiais anteriores em cabeçalhos.
- `iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE`: altera o endereço IP de origem dos pacotes que saem pela interface pública `eth1`. Os pacotes da rede interna passam a sair com o IP público do `SERVIDOR` (`202.135.187.131`). Quando a resposta volta, o kernel desfaz a tradução e entrega ao host interno correto.
- `iptables -t nat -A PREROUTING -p tcp --dport 21 -j DNAT --to 10.1.1.13`: altera o IP de destino de pacotes TCP destinados à porta pública `21`, encaminhando-os para `EMPRESA3` (`10.1.1.13`).
- `iptables -t nat -A PREROUTING -p tcp --dport 22 -j DNAT --to 10.1.1.11`: altera o IP de destino de pacotes TCP destinados à porta pública `22`, encaminhando-os para `EMPRESA1` (`10.1.1.11`).
- `iptables -A INPUT -p tcp --dport 20 -j REJECT`: não reescreve endereço; rejeita pacotes TCP destinados à porta local `20` do `SERVIDOR`, avisando o remetente que a conexão foi recusada.

### 4. O que aconteceu com o FTP do SERVIDOR ao criar uma regra bloqueando a porta 20?

O FTP local do `SERVIDOR`, configurado para escutar na porta `20`, deixou de aceitar conexões externas porque a regra `REJECT` bloqueia pacotes TCP destinados a essa porta. Assim, quando a máquina `INTERNET` tenta acessar `202.135.187.131` na porta `20`, a conexão é recusada pelo firewall.

O FTP publicado na porta `21` continua funcionando porque essa porta é tratada por uma regra de DNAT e redirecionada para a `EMPRESA3`, não para o serviço FTP local do `SERVIDOR`.
