# Validador (Routinator 0.16.0)

O Routinator roda em contêiner conectado ao switch `10.1.0.0/24`, com endereço
`10.1.0.1`, servindo RTR na porta `3323`. Ele é o único nó com acesso externo,
necessário para sincronizar com os repositórios dos RIRs.

Como o IFRN não publica ROAs, o arquivo `slurm.json` insere localmente um ROA
hipotético (`160.19.252.0/24`, origem AS266174, maxLength 24), conforme a
RFC 8416. Ele é carregado com a opção de exceções locais do Routinator
(`--exceptions slurm.json`).

Para conferir se o ROA foi carregado, em qualquer roteador com RPKI:

```
show rpki cache-connection
show rpki prefix 160.19.252.0/24
```
