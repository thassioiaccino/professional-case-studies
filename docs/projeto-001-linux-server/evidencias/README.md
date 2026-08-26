# Evidências — Projeto 001

A curadoria das evidências está em andamento. Os prints brutos permanecem preservados no workspace da execução e serão consolidados ao final do projeto, evitando transformar o repositório em um tutorial com dezenas de capturas redundantes.

## Evidências já selecionadas

1. Levantamento inicial e hostname anterior.
2. Hostname `cloudjourney-web01` confirmado.
3. Nginx `active (running)` e `enabled`, com PID e consumo básico.
4. Página do Projeto 001 acessível via navegador.
5. Processos master/worker do Nginx e respectivos PIDs.
6. Portas TCP/22 e TCP/80 associadas a `sshd` e `nginx`.
7. DNS efetivo da interface `bond0`.
8. IP, relação `ens33`/`bond0` e gateway padrão.
9. Configuração efetiva de SSH (`port`, `PermitRootLogin`, `PasswordAuthentication`).
10. Usuários e associação aos grupos DEV/OPS.
11. Owner/group/permissões de `/opt/cloudjourney/app` e `/var/log/cloudjourney`.
12. Teste DEV: escrita na aplicação e `Permission denied` nos logs.
13. Teste OPS: leitura dos logs e `Permission denied` para escrita na aplicação.
14. Teste ADMIN: elevação com `sudo` após remoção proposital do grupo OPS.

## Critério de seleção

Serão mantidas evidências de **decisões e resultados**, não um print de cada comando. Antes do fechamento da Issue #1, os arquivos finais serão renomeados semanticamente e relacionados à documentação principal.
