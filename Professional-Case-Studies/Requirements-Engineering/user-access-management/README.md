# Case Study — Refinamento de Gestão de Usuários e Perfis

> **Nota de confidencialidade:** este case é uma versão anonimizada e abstraída de uma especificação elaborada em contexto profissional real. Nomes de sistemas, identificadores, perfis, integrações, caminhos e demais detalhes foram alterados ou generalizados para preservar a confidencialidade. A lógica de análise de requisitos e os padrões de especificação foram preservados.

## 1. Contexto

Uma aplicação corporativa possuía uma funcionalidade de administração de usuários responsável por consultar usuários, associar perfis de acesso e apresentar as permissões derivadas desses perfis.

A listagem apresentava dois problemas principais:

- o campo de perfis não refletia corretamente os vínculos existentes para cada usuário;
- uma coluna de permissões exibia uma quantidade excessiva de informações, prejudicando a leitura da tela.

Na edição de um usuário, os perfis podiam ser selecionados, mas não havia uma visualização clara das permissões resultantes dessas escolhas.

## 2. Objetivo da melhoria

Como usuário responsável pela administração de acessos,

**quero** visualizar na listagem somente os perfis efetivamente vinculados a cada usuário e consultar, durante a edição, as permissões correspondentes aos perfis selecionados,

**para** tornar a informação de acesso consistente, reduzir a poluição visual da listagem e facilitar a conferência das permissões antes da gravação.

## 3. Escopo funcional

A solução especificada contempla:

- remoção da coluna detalhada de permissões da listagem;
- correção da apresentação dos perfis associados ao usuário;
- suporte a múltiplos perfis sem duplicidade;
- criação de uma área somente de leitura para exibir permissões derivadas dos perfis;
- atualização da visualização quando a seleção de perfis for alterada;
- preservação da regra existente de persistência somente após a ação explícita de salvar;
- preservação das regras existentes de autenticação e autorização.

## 4. Regras de negócio selecionadas

### RN01 — Remoção somente visual

A remoção da coluna de permissões ocorre apenas na listagem. Perfis, permissões e vínculos existentes continuam fazendo parte do modelo de controle de acesso.

### RN02 — Perfis efetivamente vinculados

A listagem deve apresentar exclusivamente os perfis associados ao usuário consultado, sem utilização de um perfil padrão quando ele não estiver realmente vinculado.

### RN03 — Múltiplos perfis

Quando houver mais de um perfil, seus nomes devem ser apresentados de forma legível, sem duplicidades e com separação padronizada.

Exemplo fictício:

```text
Consulta; Operação; Supervisão
```

### RN04 — Consistência entre consulta e edição

A listagem e a tela de edição devem representar a mesma coleção de perfis vinculados ao usuário.

### RN05 — Permissões derivadas dos perfis

As permissões exibidas na edição devem ser calculadas a partir dos perfis atualmente selecionados.

### RN06 — União sem duplicidade

Quando uma mesma permissão for concedida por dois ou mais perfis selecionados, ela deve aparecer apenas uma vez.

### RN07 — Somente leitura

A área de permissões é informativa. A concessão ou retirada de permissões continua sendo realizada por meio dos perfis, e não por associação direta ao usuário.

### RN08 — Persistência explícita

Alterar a seleção de perfis pode atualizar imediatamente a visualização das permissões, mas a persistência dos novos vínculos ocorre somente após a ação de salvar ser concluída com sucesso.

### RN09 — Controle de acesso preservado

A melhoria não deve modificar as regras existentes de autenticação, autorização ou integração com o serviço corporativo de identidade.

## 5. Critérios de aceitação selecionados

- A listagem não deve mais apresentar a coluna detalhada de permissões.
- As demais informações e ações existentes devem continuar disponíveis.
- Um usuário com um único perfil deve apresentar somente esse perfil.
- Usuários com múltiplos perfis devem ter todos os perfis exibidos sem duplicidade.
- A listagem não deve atribuir um perfil padrão inexistente.
- Os perfis apresentados na listagem devem corresponder aos perfis carregados na edição.
- Ao abrir a edição, as permissões correspondentes aos perfis já associados devem ser apresentadas.
- Ao marcar ou desmarcar um perfil, a visualização das permissões deve ser recalculada.
- Permissões compartilhadas por vários perfis devem aparecer uma única vez.
- A área de permissões não deve permitir alteração direta.
- Alterações de perfis não devem ser persistidas antes da ação explícita de salvar.
- Os fluxos existentes de consulta, inclusão, edição, alteração de status e salvamento devem continuar funcionando após a melhoria.

## 6. Fora do escopo

Não fazem parte desta melhoria:

- alterar o cadastro de perfis;
- criar ou excluir permissões;
- permitir associação manual de permissões diretamente ao usuário;
- modificar as regras de autenticação;
- alterar relatórios ou exportações não relacionados à melhoria.

## 7. Decisões de análise

### Separar visualização de persistência

A atualização imediata das permissões durante a seleção dos perfis melhora a capacidade de conferência do usuário, mas não deve representar uma alteração persistida. Essa separação reduz o risco de alterações acidentais e mantém o padrão transacional já existente.

### Derivar permissões dos perfis

A especificação evita criar uma segunda forma de administrar permissões diretamente por usuário. Os perfis permanecem como fonte de concessão, reduzindo inconsistências entre diferentes mecanismos de autorização.

### Evitar duplicidade

Uma permissão pode pertencer a mais de um perfil. A interface deve apresentar a união dessas permissões, evitando duplicações que dificultariam a leitura e poderiam gerar uma interpretação incorreta do acesso efetivo.

## 8. Competências demonstradas

Este case evidencia práticas de:

- levantamento e refinamento de requisitos;
- análise de comportamento existente;
- definição de escopo e fora de escopo;
- especificação de regras de negócio;
- elaboração de critérios de aceitação testáveis;
- análise de regressão;
- modelagem de controle de acesso baseado em perfis;
- consistência entre diferentes visões da aplicação;
- comunicação entre necessidade de negócio e implementação técnica.

## 9. Sobre este case

Este material não reproduz documentação interna, código, banco de dados, endpoints, dados pessoais ou detalhes de infraestrutura do ambiente profissional de origem. O conteúdo foi recriado especificamente para fins de portfólio, mantendo apenas os conceitos necessários para demonstrar a abordagem utilizada na análise e especificação de requisitos.
