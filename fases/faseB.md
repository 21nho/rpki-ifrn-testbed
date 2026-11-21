# Fase B - Mitigação com RPKI/ROV na RNP

ROA hipotético do IFRN carregado via SLURM e ROV habilitado na RNP.
O enlace direto ATK-OBS é desativado para isolar o efeito da validação da RNP.

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
clear bgp ipv4 unicast * soft in
```
> A política só passa a valer depois da reavaliação (`clear ... soft in`).
> Antes disso as rotas aparecem como *invalid*, mas continuam eleitas.

## B1 / B2
Com o ATK anunciando o /24 e o /25 (mesmos `network` da Fase A).

Verificação na RNP:
```
show rpki cache-connection
show bgp ipv4 unicast neighbors 10.2.0.2 received-routes
show bgp ipv4 unicast neighbors 10.2.0.2 filtered-routes
show bgp ipv4 unicast
traceroute 160.19.252.1
```
Resultado: /24 do ATK descartado (origem diferente do ROA, B1); /25 descartado
(origem diferente e violação do maxLength, B2). Fica só a rota legítima *valid*.
Evidência: [`evidencias/faseB_rnp.png`](../evidencias/faseB_rnp.png)
