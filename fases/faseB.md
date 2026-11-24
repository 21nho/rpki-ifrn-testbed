# Fase B - Mitigação com RPKI/ROV na RNP

ROA hipotético do IFRN carregado via SLURM e ROV habilitado na RNP.
O enlace direto ATK-OBS é desativado para que o OBS receba o prefixo
exclusivamente pela RNP.

## Preparação
No OBS:
```
router bgp 65100
 neighbor 10.4.0.1 shutdown
```
Na RNP:
```
router bgp 1916
 address-family ipv4 unicast
  neighbor 10.2.0.2 route-map ROV-IN in
end
clear bgp ipv4 unicast 10.2.0.2 soft in
```
> A política só passa a valer depois da reavaliação (`clear ... soft in`).
> O comportamento anterior (rota *invalid* ainda eleita) foi registrado no
> cenário C1, ver [Fase C](faseC.md).

> Nesta fase a política foi aplicada à sessão por onde o atacante anuncia.
> Como o enlace ATK-OBS está desligado, essa é a única entrada da rota
> maliciosa na RNP. Em operação, um provedor aplicaria a política a todas as
> sessões eBGP.

## B1 / B2
Com o ATK anunciando o /24 e o /25 (mesmos `network` da Fase A).

### Na RNP
```
show rpki cache-connection
show rpki prefix 160.19.252.0/24 266174
show rpki prefix 160.19.252.0/24 65666
show bgp ipv4 unicast neighbors 10.2.0.2 received-routes
show bgp ipv4 unicast neighbors 10.2.0.2 filtered-routes
clear bgp ipv4 unicast 10.2.0.2 soft in
show bgp ipv4 unicast
traceroute 160.19.252.1
```
- Sessão RTR com o validador (`10.1.0.1:3323`) **conectada**.
- O VRP de `160.19.252.0/24` autoriza apenas o AS266174 (maxLength 24); não
  há VRP para o AS65666.
- O /24 e o /25 do ATK são recebidos e ambos aparecem como filtrados: o /24
  por origem divergente do ROA (B1); o /25 por origem divergente e,
  adicionalmente, por exceder o maxLength (B2).
- Após a reavaliação, a tabela contém só a rota legítima (*valid*), e o
  traceroute para `160.19.252.1` alcança o IFRN.

Evidência: [`faseB_rnp.png`](../evidencias/faseB_rnp.png)

### Propagação a jusante (plano de controle)
```
show bgp ipv4 unicast    # no OBS e no DEST
```
- OBS: apenas `1916 266174`.
- DEST: apenas `65100 1916 266174`.

Evidências:
[`faseB_obs.png`](../evidencias/faseB_obs.png),
[`faseB_dest.png`](../evidencias/faseB_dest.png)
