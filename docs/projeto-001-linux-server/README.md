# Projeto 001 — Linux Server: Provisioning & Administration

> Status: **em andamento**. Esta documentação registra o estado atual do laboratório; o incidente controlado e a revisão final ainda serão executados.

## Objetivo

Provisionar e administrar uma VM Linux Server desde a instalação, aplicando fundamentos de identidade do host, atualização de pacotes, serviço Web, processos, rede, SSH, usuários, grupos e permissões.

## Ambiente provisionado

- **Distribuição:** Ubuntu Server 26.04 LTS — Resolute Raccoon
- **Família:** Debian
- **Virtualização:** VMware
- **Arquitetura:** x86_64
- **CPU:** 2 vCPUs
- **RAM:** aproximadamente 4 GB (3,3 GiB reconhecidos)
- **Disco:** 10 GB em `/dev/sda`, raiz gerenciada por LVM
- **Interface lógica:** `bond0`
- **Interface física:** `ens33` (slave da `bond0`)
- **IPv4:** `192.168.254.132/24`, via DHCP
- **Gateway:** `192.168.254.2`
- **DNS:** `192.168.254.2`

## Identidade do servidor

O hostname original `linuxserverproj` foi substituído por `cloudjourney-web01`. O arquivo `/etc/hosts` também foi ajustado para manter a resolução local coerente com o novo hostname.

## Atualização e pacotes

O servidor foi atualizado utilizando APT. Foram validados `curl`, `git` e `unzip`.

- `apt update`: atualiza o índice/lista de pacotes disponíveis nos repositórios.
- `apt upgrade`: atualiza os pacotes já instalados para versões disponíveis mais recentes.

## Serviço Web

Foi escolhido o **Nginx**.

Validações já realizadas:

- serviço `active (running)`;
- serviço `enabled` para inicialização automática;
- página Web acessível remotamente via `http://192.168.254.132`;
- página padrão substituída por uma página simples de identificação do Projeto 001;
- processo master e workers identificados;
- porta TCP/80 associada ao Nginx.

O master process foi observado executando por `root`, enquanto os worker processes executam por `www-data`.

## Rede

Foram utilizados `ip`, `resolvectl` e `ss` para validar endereço IPv4, interface principal, gateway padrão, DNS, portas em escuta e processos associados.

- TCP/22 — SSH (`sshd`)
- TCP/80 — HTTP (`nginx`)

## SSH

O acesso administrativo normal está sendo realizado remotamente via SSH/PuTTY.

Configuração efetiva observada com `sshd -T`:

```text
port 22
permitrootlogin prohibit-password
passwordauthentication yes
```

Decisão atual: não utilizar login remoto direto de `root` por senha. A administração é feita com usuário nominal e elevação por `sudo`/`sudo -i`.

## Usuários e grupos

Grupos criados:

```text
cloudjourney-dev
cloudjourney-ops
```

Estratégia aplicada:

- **Administrador:** usuário nominal com `sudo`;
- **Desenvolvimento:** grupo `cloudjourney-dev`;
- **Operação:** grupo `cloudjourney-ops`.

Usuários de laboratório foram criados com home e shell Bash. Foi praticada a expiração imediata da senha inicial com `chage -d 0`, obrigando troca no primeiro login.

Também foram praticados `id`, `groups`, `usermod -G` e `usermod -aG`.

## Estrutura da aplicação e logs

```text
/opt/cloudjourney/app
/var/log/cloudjourney
```

| Caminho | Proprietário | Grupo | Permissão | Objetivo |
|---|---|---|---:|---|
| `/opt/cloudjourney/app` | `root` | `cloudjourney-dev` | `775` | DEV pode criar/alterar |
| `/var/log/cloudjourney` | `root` | `cloudjourney-ops` | `750` | OPS pode consultar; outros sem acesso |

## Validação prática das permissões

Foram executados testes reais com sessões SSH distintas:

- usuário DEV conseguiu criar diretório em `/opt/cloudjourney/app`;
- usuário DEV recebeu `Permission denied` ao tentar acessar `/var/log/cloudjourney`;
- usuário OPS recebeu `Permission denied` ao tentar criar conteúdo em `/opt/cloudjourney/app`;
- usuário OPS conseguiu acessar `/var/log/cloudjourney`;
- usuário administrador, fora do grupo OPS durante o teste, não obteve acesso direto aos logs e precisou usar privilégio administrativo quando necessário.

Isso confirma separação prática de responsabilidades entre desenvolvimento, operação e administração.

## Pontos ainda pendentes

- confirmar no VMware o modo de rede (NAT/Bridge);
- aprofundar localização e acompanhamento em tempo real dos logs;
- validar registros de SSH e logs do sistema;
- provocar e investigar o incidente controlado obrigatório;
- realizar review técnica final;
- fazer curadoria final das evidências e atualizar o roadmap/Issue #1.

## Observações de segurança

Senhas utilizadas no laboratório não são registradas nesta documentação nem no histórico de comandos publicado. O arquivo de comandos usa o marcador `<SENHA_INICIAL>`.
