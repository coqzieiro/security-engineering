# Security Engineering

Repositório da disciplina de Engenharia de Segurança com laboratórios práticos de redes em ambiente Netkit/Linux.

## Visão geral

O projeto reúne roteiros, materiais de apoio e resoluções para praticar:

- configuração de redes e roteamento;
- NAT e firewall com `iptables`;
- VPN site-to-site (`OpenVPN`) e cenários com IPsec;
- análise de tráfego com capturas (`pcap`) e Wireshark;
- técnicas de ataque/defesa em laboratório (ex.: MITM, footprinting e portscan).

## Estrutura do repositório

Cada laboratório fica em uma pasta própria e normalmente contém:

- `docs/`: PDFs e dicas;
- `projeto/`: pacote `.tar.gz` e/ou cenário já extraído;
- `RESOLUCAO.md`: resolução comentada (quando disponível).

Labs disponíveis:

- `lab1/`
- `lab2/`
- `lab3/`
- `lab4/`
- `lab5/`

## Resumo dos laboratórios

### Lab 1 - NAT e Firewall

- Foco: roteamento, NAT/MASQUERADE, DNAT e regras de filtro.
- Material em `lab1/docs/`.
- Pacotes em `lab1/projeto/`.
- Resolução em `lab1/RESOLUCAO.md`.

### Lab 2 - VPNs

- Foco: autenticação por chave SSH e túnel VPN entre redes privadas.
- Material em `lab2/docs/`.
- Pacotes em `lab2/projeto/`.
- Resolução em `lab2/RESOLUCAO.md`.

### Lab 3 - IPsec

- Foco: cenário de comunicação segura com IPsec (pacote `netkit_lab15-lab3`).
- Pacote em `lab3/projeto/`.
- Atualmente sem `RESOLUCAO.md` no repositório.

### Lab 4 - Man in the Middle

- Foco: ARP poisoning, interceptação de tráfego e alteração de conteúdo em trânsito.
- Material em `lab4/docs/`.
- Capturas em `lab4/projeto/` (`mitm.pcap`, `mitm_filter.pcap`, etc.).
- Resolução em `lab4/RESOLUCAO.md`.

### Lab 5 - Portscan, Footprinting e Honeypot

- Foco: varredura de portas, exposição de serviços via NAT/DNAT e honeypot.
- Material em `lab5/docs/`.
- Pacote em `lab5/projeto/`.
- Resolução em `lab5/RESOLUCAO.md`.

## Como executar um laboratório (Netkit)

Fluxo padrão:

1. Entre na pasta do laboratório desejado.
2. Acesse `projeto/`.
3. Extraia o pacote `.tar.gz` (se necessário).
4. Inicie com `lstart -d <nome_do_cenario>`.
5. Ao terminar, use `lhalt -d <nome_do_cenario>` e `lclean -d <nome_do_cenario>`.

Exemplo:

```bash
cd lab1/projeto
tar -xf netkit_lab1-no-edisciplinas.tar.gz
lstart -d lab1
```

Observação: o nome do cenário pode variar conforme o pacote (`lab1`, `lab2`, `lab4`, `lab5`, `lab12`, etc.).

## Requisitos

- Linux (ou VM Linux);
- Netkit instalado (`lstart`, `lhalt`, `lclean`);
- ferramentas de apoio: `tcpdump`, `iptables`, `ssh`, `openvpn`, `nmap`;
- Wireshark para análise de tráfego.

## Observações

- Conteúdo voltado a estudo e prática em ambiente controlado.
- Em caso de ambiguidades nos roteiros, priorize os ajustes documentados nos arquivos `RESOLUCAO.md`.
