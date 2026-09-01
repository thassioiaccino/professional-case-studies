# Case Study — Incidente de Produção: Reconciliação de Transação Distribuída após Migração

> **Nota de confidencialidade:** este case é uma versão anonimizada e abstraída de uma especificação elaborada em contexto profissional real. Nomes de sistemas, órgãos, entidades, identificadores, perfis, integrações, contratos de API, endpoints, rotas, estruturas e objetos de banco de dados, payloads, documentos, valores, códigos de situação, caminhos, dados pessoais e demais detalhes técnicos ou operacionais foram removidos, alterados ou generalizados para preservar a confidencialidade. Nenhum código, dado, contrato, estrutura física ou detalhe de infraestrutura do ambiente original é reproduzido. A lógica de análise de requisitos, os padrões de especificação, as decisões funcionais e os conceitos necessários para demonstrar a abordagem profissional foram preservados.

## 1. Contexto

Este case descreve um incidente real identificado durante a operação de um sistema crítico integrado a uma plataforma financeira externa.

Após uma migração da aplicação, uma operação era enviada ao sistema externo e processada com sucesso, porém o estado correspondente não era persistido corretamente na aplicação local.

O resultado era uma divergência entre duas fontes:

```text
Aplicação local
     │
     │ envia operação
     ▼
Sistema financeiro externo
     │
     ├── operação criada com sucesso
     │
     ▼
retorno à aplicação
     │
     └── persistência local incompleta
```

Do ponto de vista externo, a operação já existia. Do ponto de vista da aplicação local, ela ainda podia aparecer como não concluída.

Esse tipo de situação exige cuidado porque repetir uma operação sem reconciliar previamente os estados pode criar duplicidades ou exigir procedimentos posteriores de correção/cancelamento.

## 2. Como o incidente foi identificado

Além da atuação em produto e análise funcional, também participo da execução operacional do processo suportado pela aplicação.

Foi durante essa execução que identifiquei diretamente o comportamento inconsistente.

A primeira validação não foi assumir que a integração havia falhado. A prioridade foi verificar o estado real no sistema externo.

Com acesso institucional autorizado à plataforma financeira externa, confirmei que o documento havia sido efetivamente criado.

Isso mudou completamente o diagnóstico:

```text
Hipótese inicial possível
"A operação externa falhou"

        ↓ verificação externa

Evidência encontrada
"A operação externa foi concluída"

        ↓

Novo foco da investigação
"Por que a aplicação local não refletiu o sucesso?"
```

## 3. Minha participação

Minha atuação no incidente incluiu:

- identificação do problema durante a execução operacional;
- confirmação independente do estado da operação no sistema externo;
- início da investigação diretamente na base de dados da aplicação;
- análise dos registros que deveriam representar o retorno externo;
- comparação entre o estado externo confirmado e o estado persistido localmente;
- elaboração e execução de uma correção controlada diretamente no banco de dados;
- validação funcional após a reconciliação;
- acionamento posterior da equipe de desenvolvimento para correção definitiva da aplicação migrada.

Ferramentas de IA foram utilizadas como apoio durante parte da análise e documentação. A condução do diagnóstico, o conhecimento do domínio, a interpretação dos relacionamentos, a decisão operacional e a execução da reconciliação permaneceram sob minha responsabilidade.

## 4. Diagnóstico: sucesso externo não significa sucesso ponta a ponta

Um dos principais pontos deste incidente foi separar duas condições que podem parecer equivalentes, mas não são:

```text
Sucesso da chamada externa
            ≠
Sucesso da transação ponta a ponta
```

A operação só está completamente reconciliada quando o sistema local representa corretamente o resultado confirmado externamente.

Conceitualmente:

```text
[1] Solicitação local
        ↓
[2] Chamada externa
        ↓
[3] Processamento externo
        ↓
[4] Resposta recebida
        ↓
[5] Mapeamento do retorno
        ↓
[6] Persistência local
        ↓
[7] Estado funcional apresentado ao usuário
```

Neste incidente havia evidência de sucesso até o sistema externo, mas o estado final local permanecia inconsistente.

## 5. Investigação orientada por evidências

Antes de qualquer alteração de dados, a investigação precisava responder perguntas objetivas:

1. A operação realmente existe no sistema externo?
2. O registro local analisado corresponde inequivocamente à operação externa?
3. Existe algum outro registro local utilizando o mesmo identificador externo?
4. Quais campos deveriam ter sido atualizados após o retorno?
5. Existem registros de controle ou relacionamento associados à requisição?
6. A correção pode ser executada sem modificar valores funcionais que já estavam corretos?

Um valor financeiro isolado, por exemplo, não é evidência suficiente para correlacionar registros quando diferentes operações podem possuir valores iguais.

A reconciliação deve utilizar uma combinação de atributos funcionais e técnicos que permita identificar o registro de maneira inequívoca.

## 6. O risco de simplesmente reenviar

Depois de confirmar que a operação existia externamente, reenviar a solicitação deixou de ser uma estratégia segura de recuperação.

O cenário conceitual seria:

```text
Primeira tentativa
Aplicação ───────► Sistema externo
                       │
                       └── Documento A criado

Aplicação não persiste o retorno

Nova tentativa sem reconciliação
Aplicação ───────► Sistema externo
                       │
                       └── risco de nova operação
```

Em integrações que alteram estado externo, um timeout, erro local ou falha de persistência **não prova que a operação externa falhou**.

Por isso, a estratégia adotada foi:

```text
Não reenviar
     ↓
Consultar estado externo
     ↓
Correlacionar com estado local
     ↓
Reconciliar somente o que foi comprovado
```

## 7. Correção controlada no banco

Como o documento externo já estava confirmado, foi necessária uma reconciliação dos registros locais.

A correção foi realizada diretamente no banco de produção utilizando uma abordagem defensiva, com validações antes da alteração e possibilidade de rollback antes da confirmação definitiva.

O exemplo abaixo é totalmente fictício e serve apenas para demonstrar o padrão utilizado:

```sql
SAVEPOINT BEFORE_RECONCILIATION;

DECLARE
    v_expected_rows  NUMBER := :expected_rows;
    v_valid_rows     NUMBER;
    v_updated_rows   NUMBER;
BEGIN
    SELECT COUNT(*)
      INTO v_valid_rows
      FROM local_transaction t
     WHERE t.internal_id = :internal_id
       AND t.reference_year = :reference_year
       AND t.external_document IS NULL
       AND t.external_created_at IS NULL;

    IF v_valid_rows <> v_expected_rows THEN
        RAISE_APPLICATION_ERROR(
            -20001,
            'Reconciliation stopped: unexpected validation result.'
        );
    END IF;

    UPDATE local_transaction
       SET external_document   = :confirmed_external_document,
           external_sent_at    = :confirmed_date,
           external_created_at = :confirmed_date
     WHERE internal_id = :internal_id
       AND external_document IS NULL
       AND external_created_at IS NULL;

    v_updated_rows := SQL%ROWCOUNT;

    IF v_updated_rows <> v_expected_rows THEN
        RAISE_APPLICATION_ERROR(
            -20002,
            'Reconciliation stopped: unexpected number of updated rows.'
        );
    END IF;
END;
/
```

O exemplo não reproduz o script utilizado no ambiente original.

## 8. Por que a correção foi defensiva

A finalidade não era simplesmente "preencher campos que estavam nulos".

Uma intervenção manual em produção precisava evitar que um registro já reconciliado fosse sobrescrito ou que a correção atingisse uma operação diferente da previamente validada.

Por isso, o padrão adotado utilizou:

- validação prévia da quantidade esperada;
- filtros sobre o estado atual do registro;
- correspondência com identificadores previamente conferidos;
- verificação de `SQL%ROWCOUNT`;
- `SAVEPOINT` antes da alteração;
- validação pós-alteração;
- `COMMIT` somente após conferência;
- possibilidade de `ROLLBACK` diante de qualquer divergência.

Conceitualmente:

```text
Evidência externa
      ↓
Validação local
      ↓
Pré-condições atendidas?
   ┌───────┴───────┐
  NÃO             SIM
   │                │
Abortar         Reconciliar
                    ↓
             Validar resultado
               ┌────┴────┐
              ERRO       OK
               │          │
            Rollback    Commit
```

## 9. Validação pós-correção

Após a intervenção, a validação não se limitou ao retorno do comando SQL.

Foram verificados o estado dos registros e o comportamento funcional da aplicação.

Entre os pontos de conferência estavam:

- quantidade de registros reconciliados igual à esperada;
- identificador externo corretamente associado ao registro local;
- ausência de duplicidade do documento externo;
- preservação dos valores funcionais que não faziam parte da correção;
- coerência das datas associadas ao retorno;
- apresentação do documento no estado esperado pela aplicação.

Após a correção manual, a aplicação voltou a reconhecer corretamente os documentos afetados.

## 10. Correção de dados não é correção da causa

A reconciliação restaurou a consistência operacional, mas isso não significava que a aplicação estava corrigida.

O incidente surgiu após uma migração da aplicação. O comportamento equivalente funcionava na implementação anterior, enquanto a versão migrada deixou de persistir corretamente parte do estado retornado pela integração.

A investigação funcional e de dados comprovou:

```text
operação externa concluída
        +
estado local incompleto
        +
comportamento introduzido após migração
```

Isso foi suficiente para direcionar a equipe de desenvolvimento, que foi acionada **após o diagnóstico**, para comparar o comportamento da implementação migrada com o fluxo anteriormente funcional e realizar a correção no código.

Não é atribuído neste case um mecanismo técnico específico como causa raiz — por exemplo, erro de transação, repository, constraint ou mapeamento — porque a investigação descrita comprovou o ponto de falha funcional, mas não isolou publicamente uma causa de implementação mais específica.

## 11. Pontos técnicos que uma investigação de código deveria verificar

Depois de comprovar o problema funcional, uma análise da aplicação pode investigar, entre outros pontos:

```text
Resposta externa
      ↓
Desserialização / mapeamento
      ↓
Regra de atualização
      ↓
Repository / persistência
      ↓
Transação
      ↓
Commit local
```

Também devem ser considerados:

- rollback ocorrido após sucesso externo;
- erro de constraint;
- diferenças de migration/configuração;
- falha na criação de relacionamentos auxiliares;
- logs entre o aceite externo e o commit local;
- exceções tratadas sem propagação adequada;
- comportamento diferente entre implementação legada e migrada.

Esses itens representam pontos de investigação e **não são apresentados como causas confirmadas deste incidente**.

## 12. Idempotência e reconciliação

O incidente reforçou um princípio importante de integrações distribuídas:

> A ausência de confirmação local não deve ser interpretada automaticamente como ausência de execução externa.

Uma solução robusta deve considerar mecanismos como:

- identificadores de correlação;
- idempotency keys quando suportadas;
- consulta do estado externo antes de uma nova tentativa;
- prevenção de duplicidade;
- registro auditável de requisição e resposta;
- estratégias explícitas de reconciliação;
- tratamento do cenário de sucesso externo seguido por falha local.

Em termos conceituais:

```text
           ┌── estado conhecido ──► continuar fluxo
Retry? ────┤
           └── estado incerto ────► consultar antes de reenviar
```

## 13. Separação de responsabilidades durante o incidente

O diagnóstico inicial não foi realizado pela equipe de desenvolvimento ou infraestrutura.

Minha atuação cobriu a detecção, confirmação externa, investigação de dados, identificação da inconsistência, reconciliação e validação funcional.

A equipe de desenvolvimento foi acionada posteriormente para tratar a aplicação e impedir a recorrência do comportamento introduzido após a migração.

Essa separação foi importante porque havia duas necessidades diferentes:

```text
Recuperação operacional imediata
           +
Correção definitiva da aplicação
```

A primeira restaurou a consistência dos registros afetados. A segunda tratou a recorrência do defeito.

## 14. Competências demonstradas

- incident investigation;
- production support;
- Oracle SQL;
- análise de integração entre sistemas;
- reconciliação de dados;
- investigação orientada por evidências;
- análise de estado distribuído;
- troubleshooting de aplicação migrada;
- validação de dados em produção;
- SQL defensivo para correções controladas;
- `SAVEPOINT`, `ROLLBACK` e `COMMIT`;
- validação por `SQL%ROWCOUNT`;
- prevenção de duplicidade;
- conceitos de idempotência;
- análise de risco operacional;
- conhecimento de domínio aplicado à investigação técnica;
- comunicação entre operação, produto e desenvolvimento;
- distinção entre recuperação operacional e correção definitiva.

## 15. Aprendizados

Este incidente reforçou alguns princípios que se aplicam muito além do sistema em que ocorreu:

1. **Erro local não significa falha externa.**
2. **Antes de repetir uma operação mutável, descubra o estado real do sistema externo.**
3. **Correções manuais em produção precisam de pré-condições, validação e rollback.**
4. **Conhecimento de negócio reduz drasticamente o espaço de investigação técnica.**
5. **Restaurar os dados resolve o incidente; corrigir a aplicação evita a recorrência.**
6. **Em sistemas distribuídos, estados intermediários e falhas parciais precisam ser tratados como cenários esperados de arquitetura.**

## 16. Autoria e transparência

Este case descreve uma ocorrência profissional real, mas todo o conteúdo técnico publicado foi reconstruído e anonimizado.

A investigação operacional, análise dos dados, reconciliação manual e validação foram conduzidas por mim. A equipe de desenvolvimento foi acionada posteriormente para corrigir o comportamento da aplicação migrada.

Ferramentas de IA contribuíram como apoio à análise e à estruturação da documentação, utilizando contexto e documentação técnica previamente organizados por mim. Nenhum trecho de código proprietário, estrutura física do banco, endpoint, payload, identificador, dado financeiro real ou detalhe de infraestrutura do ambiente original é reproduzido neste portfólio.
