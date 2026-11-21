# Fase A - Linha de base sem validação

Nenhum roteador aplica ROV. Corresponde ao estado atual do IFRN.

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
Verificação no OBS: `show bgp ipv4 unicast 160.19.252.0/24`.
A rota `65666` é eleita por AS_PATH mais curto que `1916 266174`.
Evidência: [`evidencias/faseA1_obs.png`](../evidencias/faseA1_obs.png)

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
traceroute 160.19.252.1     # coberto pelo /25 -> desviado para o ATK
traceroute 160.19.252.200   # fora do /25 -> alcança o IFRN
```
Evidência: [`evidencias/faseA2_rnp.png`](../evidencias/faseA2_rnp.png)
