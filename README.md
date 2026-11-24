# rpki-ifrn-testbed

Artefatos do artigo **"Eficácia e Limites do RPKI na Mitigação de Sequestro de
Prefixos BGP: um Estudo de Caso no IFRN"** (Victor, Rezembrim e Ivanilson,
IFRN, 2026).

O testbed reproduz a conectividade do IFRN (AS266174), que tem a RNP (AS1916)
como único provedor de trânsito, e avalia seis cenários de sequestro de
prefixo, com e sem RPKI/ROV.

## Ambiente
| Componente | Versão |
|---|---|
| GNS3 + Docker | emulação da topologia |
| FRRouting | 10.4.5 (um contêiner por roteador) |
| Routinator (NLnet Labs) | 0.16.0 (validador, RTR na porta 3323) |

## Topologia
| Roteador | AS | Papel |
|---|---|---|
| IFRN-AS266174 | 266174 | vítima; origina `160.19.252.0/24` |
| RNP-AS1916 | 1916 | provedor de trânsito; ROV a partir da Fase B |
| OBS-AS65100 | 65100 | observador (rede terceira); ROV na Fase C |
| DEST-AS65200 | 65200 | destino; herda a decisão do OBS |
| ATK-AS65666 | 65666 | atacante |
| TRANSIT-AS65300 | 65300 | trânsito intermediário, só no cenário C2 |

| Enlace | Sub-rede | Vigência |
|---|---|---|
| IFRN - RNP | 10.0.0.0/30 | permanente |
| RNP - ATK | 10.2.0.0/30 | permanente |
| OBS - DEST | 10.5.0.0/30 | permanente |
| RNP - OBS | 10.3.0.0/30 | Fases A, B e C1 |
| RNP - TRANSIT | 10.6.0.0/30 | cenário C2 |
| TRANSIT - OBS | 10.3.0.0/30 | cenário C2 |
| ATK - OBS | 10.4.0.0/30 | Fases A e C |
| Validador (switch) | 10.1.0.0/24 | sessões RTR |

No switch do validador ficam o validador (`.1`), o IFRN (`.2`), a RNP (`.3`)
e o OBS. O OBS foi ligado diretamente ao switch durante todo o experimento;
o endereço dele não foi registrado, e o `.4` usado em
`routers/obs/frr.conf` é sugerido para reprodução.

## Estrutura
```
routers/     configuração base do FRR de cada roteador
validador/   slurm.json (ROA hipotético do IFRN) e notas do Routinator
fases/       passo a passo de cada fase (0, A, B, C) com os comandos de verificação
evidencias/  capturas de terminal de cada cenário
topologia/   diagrama da topologia
```

## Cenários
A [Fase 0](fases/fase0.md) registra o estado inicial, sem ataque, como
referência para as demais.

| ID | Cenário | Sequestro |
|---|---|---|
| A1 | Prefixo exato (/24), sem ROV | bem-sucedido |
| A2 | Sub-prefixo (/25), sem ROV | bem-sucedido |
| B1 | Prefixo exato (/24), com ROV | bloqueado |
| B2 | Sub-prefixo (/25), com ROV | bloqueado |
| C1 | AS_PATH curto, origem do atacante | bloqueado |
| C2 | AS_PATH curto, origem forjada | bem-sucedido |

## Como reproduzir
1. Crie no GNS3 um contêiner FRR por roteador e um contêiner Routinator,
   ligados conforme a tabela de enlaces.
2. Copie cada `routers/<nome>/frr.conf` para `/etc/frr/frr.conf` do contêiner
   correspondente (com o daemon `bgpd` habilitado e o módulo RPKI carregado
   nos roteadores RNP e OBS).
3. Inicie o Routinator com o `validador/slurm.json` como arquivo de exceções.
4. Siga `fases/fase0.md`, `fases/faseA.md`, `fases/faseB.md` e `fases/faseC.md` em ordem.

Os horários das evidências referem-se ao relógio dos contêineres (UTC).

## Aviso
O prefixo `160.19.252.0/24` pertence ao IFRN e foi usado **apenas em ambiente
isolado**: os roteadores não têm rota padrão e nenhum anúncio foi feito para a
Internet real. O ROA usado é hipotético, inserido localmente via SLURM. Não
reproduza estes anúncios fora de um laboratório isolado.
