# Fase 0 - Estado inicial, sem ataque

Registro do testbed antes de qualquer anúncio malicioso. Serve de referência
(controle) para comparar com as fases seguintes. Nenhum roteador aplica ROV.

## Sessões BGP
Todas as sessões estabelecidas (`show bgp summary`):

| Roteador | Vizinhos | Observação |
|---|---|---|
| IFRN | RNP (10.0.0.2) | anuncia 1 prefixo |
| RNP | IFRN, ATK, OBS | recebe 1 prefixo (do IFRN) |
| OBS | RNP, ATK (enlace direto), DEST | recebe 1 prefixo (da RNP) |
| DEST | OBS | recebe 1 prefixo |
| ATK | RNP, OBS | não anuncia nada (PfxSnt 0) |

Evidências:
[`fase0_ifrn.png`](../evidencias/fase0_ifrn.png),
[`fase0_rnp.png`](../evidencias/fase0_rnp.png),
[`fase0_atk.png`](../evidencias/fase0_atk.png)

## Rota e encaminhamento
```
show bgp ipv4 unicast            # no OBS e no DEST
traceroute 160.19.252.1          # no DEST
```
- OBS: `160.19.252.0/24` via `10.3.0.1`, AS_PATH `1916 266174`.
- DEST: `160.19.252.0/24` via `10.5.0.1`, AS_PATH `65100 1916 266174`.
- O `traceroute` do DEST para `160.19.252.1` se completa:
  `10.5.0.1` (OBS) → `10.3.0.1` (RNP) → `160.19.252.1` (IFRN).

Evidências:
[`fase0_obs.png`](../evidencias/fase0_obs.png),
[`fase0_dest.png`](../evidencias/fase0_dest.png)
