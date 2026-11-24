# Fase A - Linha de base sem validação

Nenhum roteador aplica ROV. Corresponde ao estado atual do IFRN.
Compare com a [Fase 0](fase0.md), registrada antes do ataque.

## Preparação
- RNP: sem `route-map ROV-IN` (configuração base).
- OBS: sem `route-map ROV-IN`; enlace direto com o ATK ativo:
  ```
  router bgp 65100
   no neighbor 10.4.0.1 shutdown
  ```

## A1 - Sequestro do prefixo exato (/24)
No ATK:
```
router bgp 65666
 address-family ipv4 unicast
  network 160.19.252.0/24
```

Verificação:
```
show bgp ipv4 unicast 160.19.252.0/24   # no OBS e na RNP
show bgp ipv4 unicast                   # no DEST
traceroute 160.19.252.1                 # no DEST
```

- **OBS:** a rota do ATK (`65666`) é eleita por AS_PATH mais curto que
  `1916 266174`.
  Evidência: [`faseA1_obs.png`](../evidencias/faseA1_obs.png)
- **RNP:** recebe três caminhos para o /24 (`266174` do IFRN, `65666` do ATK
  e `65100 65666` via OBS) e mantém a legítima como melhor pelo critério de
  rota mais antiga (*Older Path*).
  Evidência: [`faseA1_rnp.png`](../evidencias/faseA1_rnp.png)
- **DEST:** aprende o /24 com AS_PATH `65100 65666`, herdado do OBS. O
  `traceroute` para `160.19.252.1` passa pelo OBS (`10.5.0.1`) e para em
  seguida (`* * *`), comportamento compatível com o descarte no ATK (rota
  `Null0`). Na [Fase 0](../evidencias/fase0_dest.png), o mesmo traceroute
  alcançava o IFRN.
  Evidência: [`faseA1_dest.png`](../evidencias/faseA1_dest.png)

## A2 - Sequestro de sub-prefixo (/25)
No ATK:
```
router bgp 65666
 address-family ipv4 unicast
  network 160.19.252.0/25
```
Verificação na RNP:
```
show bgp ipv4 unicast
show ip route 160.19.252.1
traceroute 160.19.252.1     # coberto pelo /25
traceroute 160.19.252.200   # fora do /25
```
- A RNP aprende o /25 pelo ATK (`65666`) e também via OBS (`65100 65666`).
- `show ip route 160.19.252.1` aponta para o /25 via `10.2.0.2` (ATK).
- O traceroute para `.1` fica sem resposta; o traceroute para `.200` alcança
  o IFRN.

Evidência: [`faseA2_rnp.png`](../evidencias/faseA2_rnp.png)
