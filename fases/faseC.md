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
Verificação no OBS:
```
show bgp ipv4 unicast 160.19.252.0/24
clear bgp ipv4 unicast * soft in
show bgp ipv4 unicast neighbors 10.4.0.1 received-routes
show bgp ipv4 unicast neighbors 10.4.0.1 filtered-routes
show bgp ipv4 unicast
traceroute 160.19.252.1
```
- **Antes da reavaliação:** a rota `65666` já aparece com
  `rpki validation-state: invalid`, mas ainda é eleita (`best (AS Path)`).
  A política só passa a valer após o `clear ... soft in`.
- **Depois:** o /24 e o /25 do ATK são filtrados; a tabela fica só com
  `1916 266174` (*valid*), e o traceroute alcança o IFRN
  (`10.3.0.1` → `160.19.252.1`).

Evidência: [`faseC1_obs.png`](../evidencias/faseC1_obs.png)

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

Antes do anúncio do atacante, o OBS tem uma única rota, `65300 1916 266174`,
recebida do TRANSIT (router ID `3.3.3.3`) e classificada como *valid*.
Evidência: [`faseC2_obs_pre.png`](../evidencias/faseC2_obs_pre.png)

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

Com o ATK anunciando o /24 e o /25, ambos com a origem forjada:

- As duas rotas para o /24 são *valid*: `65666 266174` (ATK) e
  `65300 1916 266174` (TRANSIT). A do ATK é eleita por AS_PATH mais curto,
  com próximo salto `10.4.0.1`.
- O /25 forjado é filtrado: a origem confere com o ROA, mas o comprimento
  excede o maxLength 24. Como sem o VRP do IFRN ele seria *not found* e
  passaria, isso também mostra que o OBS tinha o VRP carregado durante o C2.
- O traceroute do OBS para `160.19.252.1` fica sem resposta já no primeiro
  salto, comportamento compatível com o descarte no ATK (rota `Null0`). A
  prova do desvio é a tabela, que elege a rota do ATK.
- A propagação até o DEST neste cenário não foi registrada.

Evidências:
[`faseC2_obs.png`](../evidencias/faseC2_obs.png),
[`faseC2_obs_parte2.png`](../evidencias/faseC2_obs_parte2.png)
