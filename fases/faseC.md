# Fase C - Limites estruturais do RPKI

ROV ativo também no OBS e enlace direto ATK-OBS reativado.

## Preparação (OBS)
```
router bgp 65100
 no neighbor 10.4.0.1 shutdown
 address-family ipv4 unicast
  neighbor 10.3.0.1 route-map ROV-IN in
  neighbor 10.4.0.1 route-map ROV-IN in
end
clear bgp ipv4 unicast * soft in
```

## C1 - Caminho curto, origem do atacante
No ATK:
```
router bgp 65666
 address-family ipv4 unicast
  network 160.19.252.0/24
```
Resultado no OBS: rota `65666` *invalid* e descartada; vence `1916 266174`.
Evidência: [`evidencias/faseC1_obs.png`](../evidencias/faseC1_obs.png)

## C2 - Origem forjada (forged-origin hijack)

### Mudança de topologia
O TRANSIT (AS65300) entra entre RNP e OBS, substituindo o enlace RNP-OBS.
Assim o caminho legítimo passa a ser `65300 1916 266174` (3 ASes) e o do
atacante `65666 266174` (2 ASes).

- Suba `routers/transit/frr.conf` (eth0 para a RNP, eth1 para o OBS).
- Na RNP, a eth3 passa a ligar ao TRANSIT:
  ```
  interface eth3
   ip address 10.6.0.1/30
  router bgp 1916
   no neighbor 10.3.0.2
   neighbor 10.6.0.2 remote-as 65300
   neighbor 10.6.0.2 description TRANSIT
  ```
- No OBS, o vizinho `10.3.0.1` passa a ser o TRANSIT:
  ```
  router bgp 65100
   neighbor 10.3.0.1 remote-as 65300
   neighbor 10.3.0.1 description TRANSIT
  ```

### Anúncio do atacante
No ATK, a origem forjada é aplicada **só na sessão com o OBS**; se for aplicada
também na RNP, ela veria a origem forjada e o contraste do teste se perde.
```
route-map FORGE-ORIGIN permit 10
 set as-path prepend 266174
!
router bgp 65666
 address-family ipv4 unicast
  network 160.19.252.0/24
  neighbor 10.4.0.2 route-map FORGE-ORIGIN out
```

Resultado no OBS: as duas rotas são *valid*; o ATK vence por AS_PATH mais curto
e o `traceroute` confirma o desvio. Repetindo com `network 160.19.252.0/25`, a
rota forjada é *invalid* por violar o maxLength 24 e é descartada.
Evidência: [`evidencias/faseC2_obs.png`](../evidencias/faseC2_obs.png)
