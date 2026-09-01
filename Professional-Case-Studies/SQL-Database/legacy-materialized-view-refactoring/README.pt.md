# Case Study — Evolução e Refatoração de Materialized View Oracle

> **Nota de confidencialidade:** este case é uma versão anonimizada e abstraída de uma especificação elaborada em contexto profissional real. Nomes de sistemas, órgãos, entidades, identificadores, perfis, integrações, contratos de API, endpoints, rotas, estruturas e objetos de banco de dados, payloads, documentos, valores, códigos de situação, caminhos, dados pessoais e demais detalhes técnicos ou operacionais foram removidos, alterados ou generalizados para preservar a confidencialidade. Nenhum código, dado, contrato, estrutura física ou detalhe de infraestrutura do ambiente original é reproduzido. A lógica de análise de requisitos, os padrões de especificação, as decisões funcionais e os conceitos necessários para demonstrar a abordagem profissional foram preservados.

## 1. Contexto

Este case representa a manutenção evolutiva de uma **Materialized View Oracle legada**, utilizada para consolidar informações de contas, saldos, entidades e referências de negócio provenientes de diferentes estruturas relacionais.

A Materialized View original já existia e **não foi criada por mim**. Minha atuação concentrou-se em alterações evolutivas realizadas sobre o objeto existente, incluindo recuperação de informação removida em refatoração anterior, adequação da consulta para dois modelos históricos de relacionamento e correção de duplicidades produzidas pela cardinalidade dos dados legados.

## 2. Minha contribuição

As alterações sob minha responsabilidade envolveram quatro frentes principais:

- reinclusão de uma informação de agrupamento que havia deixado de ser retornada após uma alteração anterior;
- recuperação de referências de programa e processo para registros pertencentes a períodos históricos distintos;
- refatoração da consulta utilizada para resolver esses relacionamentos;
- correção de duplicidade em registros legados nos quais múltiplas referências de negócio podiam utilizar a mesma conta.

O desafio principal surgiu porque o modelo de relacionamento mudou ao longo do tempo. Registros mais recentes possuíam vínculo direto entre conta e referência de negócio, enquanto registros legados exigiam reconstruir o relacionamento por uma cadeia intermediária de processamento.

## 3. Problema técnico

Uma única estratégia de JOIN não representava corretamente todo o histórico.

Para registros do modelo atual, a referência podia ser obtida diretamente pela conta. Para registros do modelo legado, a associação precisava atravessar entidades intermediárias até alcançar a referência original.

Além disso, no modelo legado, **uma mesma conta podia estar associada a várias referências**. Um JOIN direto sobre essa relação 1:N multiplicava as linhas da consulta final, produzindo duplicidade de saldos.

O problema, portanto, não era remover linhas duplicadas ao final da consulta, mas corrigir a cardinalidade antes do JOIN com o conjunto principal.

## 4. Estratégia adotada

A consulta foi dividida conceitualmente em dois caminhos por meio de CTEs:

```text
                    ┌─ Modelo atual ──────── vínculo direto pela conta
Conta + Saldo ──────┤
                    └─ Modelo legado ─────── reconstrução por relações intermediárias
                                              ↓
                                     consolidação por conta
                                              ↓
                                  resultado final sem multiplicação
```

### Caminho atual

Para dados pertencentes ao modelo mais recente, o relacionamento direto é preservado porque existe uma chave capaz de associar a conta à referência correspondente sem reconstrução histórica.

### Caminho legado

Para dados anteriores à mudança do modelo, as referências são localizadas por relacionamentos intermediários. Como várias referências podem chegar à mesma conta, elas são primeiro deduplicadas e consolidadas por dados normalizados da conta.

Somente depois dessa consolidação o resultado é associado à consulta principal.

## 5. Decisão central — consolidar antes do JOIN

Uma abordagem simplificada como esta é problemática quando a relação é 1:N:

```sql
SELECT ...
FROM account a
JOIN legacy_reference r
  ON r.account_id = a.account_id;
```

Se uma conta possuir três referências históricas, o saldo da conta também poderá aparecer três vezes.

A solução pública equivalente utiliza uma etapa intermediária:

```sql
WITH legacy_reference AS (
    SELECT
        LISTAGG(x.reference_number, ' | ')
            WITHIN GROUP (ORDER BY x.reference_number) AS reference_numbers,
        LISTAGG(x.process_number, ' | ')
            WITHIN GROUP (ORDER BY x.reference_number) AS process_numbers,
        x.bank_code,
        x.branch_number,
        x.account_number
    FROM (
        SELECT DISTINCT
            r.reference_number,
            r.process_number,
            p.bank_code,
            LTRIM(p.branch_number, '0 ')  AS branch_number,
            LTRIM(p.account_number, '0 ') AS account_number
        FROM legacy_process p
        JOIN legacy_process_item i
          ON i.process_id = p.process_id
        JOIN business_reference r
          ON r.reference_id = i.reference_id
        WHERE p.reference_year <= :legacy_year
    ) x
    GROUP BY
        x.bank_code,
        x.branch_number,
        x.account_number
)
SELECT ...
FROM account_balance ab
LEFT JOIN legacy_reference lr
       ON lr.bank_code = ab.bank_code
      AND lr.branch_number = LTRIM(ab.branch_number, '0 ')
      AND lr.account_number = LTRIM(ab.account_number, '0 ');
```

Assim, múltiplas referências são transformadas em **uma linha por conta antes da associação com o saldo**.

## 6. Técnicas SQL aplicadas

### CTE — Common Table Expressions

As estratégias atual e legada são isoladas em blocos lógicos independentes, reduzindo a complexidade do SELECT principal e facilitando manutenção e diagnóstico.

### `DISTINCT` antes da agregação

A deduplicação ocorre antes do `LISTAGG`, impedindo que a mesma referência seja repetida dentro da lista consolidada.

### `LISTAGG`

Permite representar múltiplas referências legítimas de uma mesma conta sem multiplicar o registro principal.

### `COALESCE`

No resultado consolidado, permite priorizar a referência obtida pelo modelo atual e utilizar a estratégia legada quando o vínculo direto não estiver disponível.

### Normalização com `LTRIM`

Dados bancários legados podem possuir preenchimento com zeros ou espaços. A normalização é aplicada de forma consistente nos dois lados do relacionamento para evitar falhas de correspondência provocadas apenas por representação textual.

### Tratamento de valores nulos

Funções como `NVL` são utilizadas conceitualmente para garantir que componentes nulos de um saldo consolidado não invalidem o cálculo do total.

### Regras condicionais

`CASE` e funções condicionais do Oracle permitem derivar atributos de apresentação conforme características da entidade ou classificação da conta.

## 7. Exemplo anonimizado do padrão final

```sql
WITH current_reference AS (
    SELECT
        r.reference_id,
        TO_CHAR(r.reference_number) AS reference_number,
        TO_CHAR(r.process_number)   AS process_number,
        a.account_id
    FROM account a
    JOIN business_reference r
      ON r.reference_id = a.reference_id
    WHERE r.reference_year >= :current_model_year
),
legacy_reference AS (
    SELECT
        LISTAGG(x.reference_number, ' | ')
            WITHIN GROUP (ORDER BY x.reference_number) AS reference_number,
        LISTAGG(x.process_number, ' | ')
            WITHIN GROUP (ORDER BY x.reference_number) AS process_number,
        x.bank_code,
        x.branch_number,
        x.account_number
    FROM (
        SELECT DISTINCT
            TO_CHAR(r.reference_number) AS reference_number,
            TO_CHAR(r.process_number)   AS process_number,
            p.bank_code,
            LTRIM(p.branch_number, '0 ')  AS branch_number,
            LTRIM(p.account_number, '0 ') AS account_number
        FROM legacy_process p
        JOIN legacy_process_item i
          ON i.process_id = p.process_id
        JOIN business_reference r
          ON r.reference_id = i.reference_id
        WHERE r.reference_year <= :legacy_year
    ) x
    GROUP BY
        x.bank_code,
        x.branch_number,
        x.account_number
)
SELECT
    ab.account_id,
    COALESCE(cr.reference_number, lr.reference_number) AS reference_number,
    COALESCE(cr.process_number, lr.process_number)     AS process_number,
    NVL(ab.checking_balance, 0)
      + NVL(ab.savings_balance, 0)
      + NVL(ab.investment_balance, 0)                 AS total_balance
FROM account_balance ab
LEFT JOIN current_reference cr
       ON cr.account_id = ab.account_id
LEFT JOIN legacy_reference lr
       ON lr.bank_code = ab.bank_code
      AND lr.branch_number = LTRIM(ab.branch_number, '0 ')
      AND lr.account_number = LTRIM(ab.account_number, '0 ')
WHERE COALESCE(cr.reference_number, lr.reference_number) IS NOT NULL;
```

O SQL acima é uma **reconstrução didática e anonimizada**. Os nomes, estruturas e relacionamentos foram alterados e simplificados; ele não representa o modelo físico do ambiente profissional original.

## 8. Resultado da alteração

A evolução permitiu que a consulta:

- retornasse referências de negócio para dados dos dois modelos históricos;
- preservasse múltiplas referências legítimas associadas a uma mesma conta;
- evitasse multiplicação indevida das linhas de saldo;
- mantivesse uma única representação consolidada da conta no resultado;
- recuperasse informação funcional necessária que havia deixado de ser retornada;
- deixasse a separação entre regras atuais e legadas mais explícita para manutenção futura.

## 9. Competências demonstradas

- Oracle SQL;
- manutenção de código legado;
- análise de cardinalidade 1:N;
- diagnóstico de duplicidade causada por JOIN;
- CTEs;
- `INNER JOIN` e `LEFT JOIN`;
- `LISTAGG` e `GROUP BY`;
- `DISTINCT`;
- `COALESCE` e `NVL`;
- normalização de dados para relacionamento;
- tratamento de modelos históricos distintos;
- refatoração orientada à regra de negócio;
- análise de impacto em consultas consolidadas.

## 10. Autoria e transparência

A Materialized View que originou este estudo já fazia parte de uma solução existente e foi originalmente desenvolvida por outros profissionais. Este case **não reivindica autoria sobre o objeto original**.

Minha contribuição corresponde às evoluções, correções e refatorações descritas neste documento. O exemplo SQL publicado foi recriado especificamente para o portfólio e serve apenas para demonstrar as técnicas e o raciocínio empregados nessas alterações.
