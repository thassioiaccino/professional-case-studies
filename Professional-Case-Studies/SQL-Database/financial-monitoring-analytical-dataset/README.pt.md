# Case Study — Evolução de Dataset Analítico para Monitoramento Financeiro

> **Nota de confidencialidade:** este case é uma versão anonimizada e abstraída de uma especificação elaborada em contexto profissional real. Nomes de sistemas, órgãos, entidades, identificadores, perfis, integrações, contratos de API, endpoints, rotas, estruturas e objetos de banco de dados, payloads, documentos, valores, códigos de situação, caminhos, dados pessoais e demais detalhes técnicos ou operacionais foram removidos, alterados ou generalizados para preservar a confidencialidade. Nenhum código, dado, contrato, estrutura física ou detalhe de infraestrutura do ambiente original é reproduzido. A lógica de análise de requisitos, os padrões de especificação, as decisões funcionais e os conceitos necessários para demonstrar a abordagem profissional foram preservados.

## 1. Contexto

Este case representa a evolução de uma consulta Oracle SQL utilizada para formar um **dataset analítico de acompanhamento financeiro**, consumido por uma solução corporativa de Business Intelligence.

O objetivo do produto analítico é apoiar departamentos responsáveis pelo acompanhamento da instrução dos processos antes e depois do pagamento, oferecendo uma visão consolidada de informações que originalmente estão distribuídas em diferentes domínios do modelo de dados.

A consulta já existia e **não foi originalmente criada por mim**. Minha atuação ocorreu em evoluções relevantes solicitadas ao longo do uso operacional do painel. A necessidade funcional era apresentada pela gestão, enquanto a investigação do modelo, definição dos relacionamentos necessários e implementação das alterações SQL ficaram sob minha responsabilidade.

## 2. Visão conceitual do dataset

A consulta consolida diferentes etapas do ciclo financeiro em uma visão analítica única:

```text
Programação / Destinação
          ↓
    Regra de negócio
          ↓
       Empenho
          ↓
   Conta vinculada
          ↓
      Pagamento
          ↓
 Dataset analítico / BI
```

Essa consolidação permite que equipes operacionais acompanhem o processo sem precisar reconstruir manualmente os relacionamentos existentes entre diferentes estruturas transacionais.

## 3. Minha contribuição

As evoluções sob minha responsabilidade podem ser agrupadas em três frentes.

### 3.1 Rastreabilidade operacional de contas

Foi solicitada a inclusão das informações de agência e conta vinculadas às programações.

A conta é criada como parte do fluxo operacional associado ao processamento financeiro. Por isso, a equipe responsável precisa conseguir identificar posteriormente **qual contexto de negócio originou determinada abertura de conta**.

Um exemplo de uso ocorre quando um ente beneficiário solicita esclarecimentos sobre uma conta aberta. A equipe pode utilizar o dataset analítico para relacionar a conta ao contexto da programação e do pagamento, reduzindo a necessidade de investigação manual em diferentes fontes.

A demanda funcional partiu da gestão. A identificação do caminho dos dados e a implementação SQL foram realizadas por mim com base no conhecimento acumulado do domínio e do modelo relacional.

### 3.2 Evolução da identificação do responsável pela destinação

Outra alteração ocorreu na identificação do parlamentar associado a determinados tipos de destinação orçamentária.

Em alguns cenários, o titular formal associado à emenda não representa necessariamente o parlamentar que indicou ou apoiou determinado beneficiário. A consulta passou a considerar essa diferença e, quando aplicável, priorizar a identificação do **parlamentar solicitante/apoiador**.

A evolução ocorreu no contexto de novas exigências de transparência e rastreabilidade das emendas parlamentares, fortalecidas por decisões públicas relacionadas à transparência da execução orçamentária.

Conceitualmente, a regra pode ser representada assim:

```sql
CASE
    WHEN transparency_rule_applies = 1
    THEN COALESCE(supporting_member, formal_author)
    ELSE formal_author
END AS responsible_member
```

O exemplo é deliberadamente genérico e não reproduz campos, códigos ou estruturas do ambiente original.

### 3.3 Remoção de atributo sensível

Também implementei uma alteração solicitada pela gestão para remover um atributo considerado sensível do dataset analítico.

A decisão sobre a classificação e retirada da informação partiu da gestão; minha responsabilidade foi implementar tecnicamente a alteração preservando o restante do funcionamento da consulta e do conjunto de dados.

Por razões de confidencialidade, o atributo, sua origem, regra de obtenção e demais características não são descritos neste case.

## 4. Desafio técnico — evolução histórica do modelo de contas

A inclusão de agência e conta não podia ser resolvida com um único JOIN.

O modelo de relacionamento entre contas e programações havia evoluído ao longo dos anos. Registros pertencentes ao modelo atual possuem uma associação mais direta, enquanto registros legados exigem reconstruir a relação por entidades intermediárias.

A estratégia conceitual ficou semelhante a:

```text
                         ┌─ Modelo atual ─── relacionamento direto
Programação ─────────────┤
                         └─ Modelo legado ── reconstrução histórica
                                                ↓
                                      normalização das contas
                                                ↓
                                         consolidação
```

A solução utiliza dois caminhos e os reúne antes da associação com o dataset principal.

Um exemplo anonimizado:

```sql
WITH account_reference AS (
    SELECT DISTINCT
        current_data.business_reference,
        current_data.category,
        current_data.branch_number,
        current_data.account_number
    FROM current_account_relation current_data
    WHERE current_data.model_version = 'CURRENT'

    UNION ALL

    SELECT DISTINCT
        legacy_data.business_reference,
        legacy_data.category,
        legacy_data.branch_number,
        legacy_data.account_number
    FROM legacy_process legacy_data
    JOIN legacy_account_relation relation
      ON relation.process_id = legacy_data.process_id
    WHERE legacy_data.model_version = 'LEGACY'
)
SELECT ...
FROM analytical_base base
LEFT JOIN account_reference account
       ON account.business_reference = base.business_reference
      AND account.category = base.category;
```

O código acima foi recriado exclusivamente para este portfólio e não representa o modelo físico original.

## 5. Consolidação de relacionamentos 1:N

O dataset precisa representar informações que naturalmente possuem cardinalidade 1:N.

Uma programação pode estar relacionada, por exemplo, a diferentes registros financeiros ao longo de seu ciclo. Um JOIN direto poderia multiplicar linhas do conjunto principal e distorcer indicadores analíticos.

Por isso, determinados conjuntos são preparados antes de serem associados à consulta principal.

Um padrão equivalente é:

```sql
WITH financial_events AS (
    SELECT
        business_reference,
        category,
        SUM(amount) AS total_amount,
        LISTAGG(document_number, ' | ')
            WITHIN GROUP (ORDER BY event_date DESC) AS documents,
        LISTAGG(TO_CHAR(event_date, 'DD/MM/YYYY'), ' | ')
            WITHIN GROUP (ORDER BY event_date DESC) AS event_dates
    FROM normalized_financial_event
    GROUP BY
        business_reference,
        category
)
SELECT ...
FROM analytical_base base
LEFT JOIN financial_events event
       ON event.business_reference = base.business_reference
      AND event.category = base.category;
```

Assim, múltiplos eventos legítimos podem ser apresentados sem transformar cada ocorrência em uma nova linha do registro analítico principal.

## 6. Recuperação do estado mais recente

O histórico de situação é outro relacionamento naturalmente 1:N: uma mesma programação pode possuir diversas mudanças de estado.

Para recuperar apenas o estado mais recente, é utilizado conceitualmente o padrão de função analítica:

```sql
SELECT business_reference,
       status_description,
       status_date
FROM (
    SELECT
        business_reference,
        status_description,
        status_date,
        ROW_NUMBER() OVER (
            PARTITION BY business_reference
            ORDER BY status_date DESC
        ) AS rn
    FROM status_history
)
WHERE rn = 1;
```

Esse padrão evita que todo o histórico multiplique registros na camada analítica quando o objetivo é apresentar o estado corrente.

## 7. Separação entre dado transacional e dado analítico

Um dos aprendizados centrais deste trabalho foi que uma consulta destinada a Analytics/BI não deve simplesmente reproduzir JOINs entre tabelas transacionais.

O dataset precisa transformar diferentes estruturas em uma representação adequada ao consumo analítico:

- históricos precisam ser reduzidos ao estado relevante;
- relações 1:N precisam ser consolidadas quando o grão analítico exige uma linha principal;
- modelos legados e atuais precisam coexistir;
- valores nulos precisam ser tratados de forma previsível;
- atributos precisam respeitar regras de negócio e de exposição da informação;
- mudanças regulatórias ou operacionais podem alterar a interpretação de um campo sem necessariamente alterar sua origem física.

## 8. Exemplo anonimizado da arquitetura SQL

```sql
WITH latest_status AS (
    SELECT business_reference, status_description
    FROM (
        SELECT
            business_reference,
            status_description,
            ROW_NUMBER() OVER (
                PARTITION BY business_reference
                ORDER BY status_date DESC
            ) AS rn
        FROM status_history
    )
    WHERE rn = 1
),
commitment_summary AS (
    SELECT
        business_reference,
        category,
        SUM(amount) AS committed_amount,
        LISTAGG(document_number, ' | ')
            WITHIN GROUP (ORDER BY document_date DESC) AS documents
    FROM commitment
    GROUP BY business_reference, category
),
account_summary AS (
    SELECT
        business_reference,
        category,
        LISTAGG(branch_number, ' | ')
            WITHIN GROUP (ORDER BY branch_number) AS branches,
        LISTAGG(account_number, ' | ')
            WITHIN GROUP (ORDER BY account_number) AS accounts
    FROM normalized_account_relation
    GROUP BY business_reference, category
),
payment_summary AS (
    SELECT
        business_reference,
        category,
        SUM(NVL(amount, 0)) AS paid_amount
    FROM payment
    GROUP BY business_reference, category
)
SELECT
    base.reference_year,
    base.region,
    base.beneficiary,
    base.resource_type,
    base.programmed_amount,
    commitment.committed_amount,
    account.branches,
    account.accounts,
    payment.paid_amount,
    status.status_description
FROM analytical_base base
LEFT JOIN latest_status status
       ON status.business_reference = base.business_reference
LEFT JOIN commitment_summary commitment
       ON commitment.business_reference = base.business_reference
      AND commitment.category = base.category
LEFT JOIN account_summary account
       ON account.business_reference = base.business_reference
      AND account.category = base.category
LEFT JOIN payment_summary payment
       ON payment.business_reference = base.business_reference
      AND payment.category = base.category;
```

Este exemplo demonstra apenas o padrão técnico utilizado. Entidades, nomes, regras, datas, códigos e relacionamentos foram substituídos ou simplificados.

## 9. Impacto operacional

As alterações ampliaram a utilidade do dataset para as equipes responsáveis pelo acompanhamento do processo financeiro.

A inclusão das informações de conta aumentou a rastreabilidade operacional entre a programação e a abertura da conta associada ao fluxo. A evolução da regra de identificação do responsável melhorou a representação da informação conforme novas necessidades de transparência. A remoção do atributo sensível adequou o conjunto de dados à decisão de exposição definida pela gestão.

O resultado foi uma evolução da camada analítica sem exigir que os usuários reconstruíssem manualmente essas relações em diferentes fontes de informação.

## 10. Transparência e contexto público

O conceito de **parlamentar apoiador/solicitante** é utilizado em mecanismos públicos de transparência para identificar parlamentares que indicaram ou apoiaram emendas ou beneficiários.

Como referência pública de contexto, o Portal da Transparência disponibiliza consulta específica relacionada a apoiadores de emendas:

**Portal da Transparência — Apoiadores de Emendas**  
https://portaldatransparencia.gov.br/emendas/apoiadores

Esta referência é apresentada apenas para contextualizar publicamente o conceito. Ela **não documenta nem implica integração técnica** entre o ambiente descrito neste case e o Portal da Transparência.

O painel corporativo que utiliza o dataset deste estudo não é referenciado, pois seu acesso não foi identificado como público e irrestrito.

## 11. Competências demonstradas

- Oracle SQL;
- Analytics / BI data preparation;
- modelagem de datasets analíticos;
- análise de requisitos aplicada a dados;
- tradução de necessidade operacional em regra SQL;
- manutenção e evolução de consultas legadas;
- `ROW_NUMBER()` e funções analíticas;
- `LISTAGG`;
- `SUM` e `GROUP BY`;
- `DISTINCT`;
- `UNION ALL`;
- `INNER JOIN` e `LEFT JOIN`;
- `CASE`, `COALESCE` e `NVL`;
- tratamento de cardinalidade 1:N;
- coexistência entre modelos de dados atuais e legados;
- rastreabilidade de dados;
- regras temporais e evolução de negócio;
- data governance e tratamento de informação sensível;
- conhecimento de domínio aplicado à investigação de dados.

## 12. Autoria e transparência

A consulta que originou este estudo já fazia parte de uma solução existente e não foi originalmente desenvolvida por mim.

Minha contribuição corresponde às evoluções descritas neste documento: investigação dos relacionamentos necessários, implementação da rastreabilidade de contas, adequação da regra de identificação do responsável e implementação da remoção de um atributo sensível por solicitação da gestão.

Todos os exemplos SQL deste case foram recriados especificamente para o portfólio. Nenhum trecho do código original, estrutura física de banco, dado real ou informação classificada como sensível é reproduzido.
