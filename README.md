# Security Engineering

Repositório de estudos práticos da disciplina de Engenharia de Segurança, com foco em laboratórios de redes usando Netkit/Linux.

## Objetivo do repositório

Este projeto organiza materiais de aula, pacotes de laboratório e resoluções práticas para treinar conceitos de segurança de redes em ambiente controlado. A proposta principal é reproduzir cenários reais de administração e defesa de infraestrutura, incluindo:

- configuração de rede e roteamento;
- NAT e firewall com iptables;
- VPN site-to-site com OpenVPN/IPsec;
- análise de tráfego e riscos em redes inseguras;
- técnicas de ataque e defesa em laboratório (ex.: Man in the Middle).

## Estrutura

Cada laboratório fica em uma pasta própria, normalmente com:

- `docs/`: apostilas, slides e dicas;
- `projeto/`: pacote `.tar.gz` do Netkit e/ou pasta extraída;
- `RESOLUCAO.md`: quando disponível, descreve os ajustes e respostas.

Estrutura atual:

- `lab1/`
- `lab2/`
- `lab3/`
- `lab4/`
- `lab5/`

## Resumo dos laboratórios

### Lab 1 - NAT e Firewall

- Tema: compartilhamento de internet, firewall e redirecionamento de portas.
- Tecnologias: `iptables`, `tcpdump`, serviços FTP/SSH.
- Materiais em `lab1/docs/`.
- Resolução disponível em `lab1/RESOLUCAO.md`.

### Lab 2 - VPNs

- Tema: autenticação por chaves e túnel VPN entre redes privadas.
- Tecnologias: `OpenVPN`, `SSH`, rotas estáticas.
- Materiais e dicas em `lab2/docs/`.
- Resolução disponível em `lab2/RESOLUCAO.md`.

### Lab 3 - IPsec (pacote lab15)

- Indícios no pacote: hosts `ALICE`, `BOB`, `EVE` e arquivos `racoon`/`ipsec-tools`.
- Tema esperado: comunicação segura com IPsec e análise de cenário com atacante intermediário.
- Estado: pacote presente em `lab3/projeto/`.

### Lab 4 - Man in the Middle

- Materiais teóricos presentes em `lab4/docs/`.
- Estado: sem pacote de projeto extraído no momento.

### Lab 5

- Estrutura criada (`docs/` e `projeto/`), ainda sem conteúdo.

## Como executar os laboratórios Netkit

Os labs foram organizados para execução em VM Linux com Netkit (conforme material da disciplina).

Fluxo padrão:

1. Entre na pasta de projeto do laboratório desejado.
2. Extraia o pacote `.tar.gz` (se necessário).
3. Inicie o lab com `lstart -d <pasta_do_lab>`.
4. Execute os testes pedidos no roteiro.
5. Finalize com `lhalt -d <pasta_do_lab>` e `lclean -d <pasta_do_lab>`.

Exemplo genérico:

```bash
cd labX/projeto
tar -xf pacote-do-lab.tar.gz
lstart -d labX
```

## Requisitos

- Linux (ou VM Linux com suporte ao Netkit);
- ferramentas do Netkit (`lstart`, `lhalt`, `lclean`);
- utilitários de rede (`tcpdump`, `ssh`, `openvpn`, `iptables`);
- Wireshark para análise de capturas.

## Observações

- Este repositório é voltado a estudo e prática em ambiente de laboratório.
- Alguns passos dos roteiros originais podem ter ambiguidades; as pastas com `RESOLUCAO.md` documentam os ajustes usados para execução prática.
