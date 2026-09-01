# Case Study — Modernização de Sistema Legado com Governança Funcional

> **Nota de confidencialidade:** este case é uma versão anonimizada e abstraída de uma especificação elaborada em contexto profissional real. Nomes de sistemas, órgãos, entidades, identificadores, perfis, integrações, contratos de API, endpoints, rotas, estruturas e objetos de banco de dados, payloads, documentos, valores, códigos de situação, caminhos, dados pessoais e demais detalhes técnicos ou operacionais foram removidos, alterados ou generalizados para preservar a confidencialidade. Nenhum código, dado, contrato, estrutura física ou detalhe de infraestrutura do ambiente original é reproduzido. A lógica de análise de requisitos, os padrões de especificação, as decisões funcionais e os conceitos necessários para demonstrar a abordagem profissional foram preservados.

## 1. Contexto

Este case descreve minha atuação funcional durante a modernização tecnológica de um sistema crítico utilizado em processos administrativos e financeiros.

A iniciativa foi definida conjuntamente entre a equipe técnica e a área de negócio responsável pelo produto. O objetivo era atualizar uma base tecnológica antiga, reduzir passivo técnico e preparar a aplicação para evolução futura, preservando ao mesmo tempo as regras de negócio existentes.

A modernização envolveu atualização simultânea das principais tecnologias de backend e frontend, atravessando diversas versões de framework e runtime.

O principal risco não era apenas fazer o novo código compilar ou a nova interface abrir.

O desafio era garantir que:

```text
Tecnologia nova
      +
Comportamento funcional preservado
      +
Integrações mantidas
      +
Operação real validada
```

## 2. O problema oculto: modernizar sem baseline funcional

Antes da iniciativa, não existia uma documentação funcional consolidada do sistema.

Havia histórico de histórias de usuário produzidas em momentos diferentes por outros profissionais, mas não existia um repositório único que explicasse de forma organizada:

- o funcionamento dos módulos;
- regras de negócio vigentes;
- filtros e comportamentos condicionais;
- extrações;
- integrações;
- endpoints existentes;
- estruturas técnicas relevantes;
- fluxos operacionais;
- exceções conhecidas;
- critérios funcionais esperados.

Isso criava um problema importante para a modernização:

```text
Código legado existe
        ↓
Comportamento real existe
        ↓
Conhecimento distribuído entre pessoas e código
        ↓
Documentação insuficiente
        ↓
Risco de migrar tecnologia e alterar comportamento sem perceber
```

## 3. Minha contribuição

Minha atuação ficou concentrada na continuidade funcional da aplicação durante a modernização.

Participei da decisão de modernizar em conjunto com a liderança da área e a equipe técnica e, durante o trabalho, atuei principalmente em:

- comparação entre a versão legada em produção e o novo ambiente;
- conferência funcional tela a tela e fluxo a fluxo;
- levantamento de divergências;
- identificação de melhorias;
- explicação de regras de negócio para os desenvolvedores;
- validação de grids, filtros, filtros condicionais, extrações e demais comportamentos;
- execução da regressão funcional;
- homologação funcional;
- priorização das correções antes da entrada em produção;
- decisão funcional sobre o comportamento esperado quando havia divergência;
- aceite funcional para entrada em produção;
- acompanhamento pós-implantação;
- identificação de regressões em produção;
- suporte operacional e abertura de correções.

A implementação técnica da migração — alteração do código, frameworks e componentes — foi realizada pela equipe de desenvolvimento.

Meu papel não foi migrar o código. Meu papel foi garantir que a modernização **não quebrasse o produto que o código precisava continuar representando**.

## 4. Construção de uma baseline funcional

A ausência de documentação consolidada me levou a iniciar, por decisão própria, a construção de uma documentação funcional e técnica organizada do sistema.

Esse trabalho foi iniciado e conduzido por mim.

A equipe de desenvolvimento contribuiu pontualmente com informações técnicas que não existiam nas histórias anteriores, como endpoints e estruturas já implantadas.

A documentação passou a consolidar elementos como:

```text
Visão geral do sistema
        ↓
Módulos e funcionalidades
        ↓
Fluxos operacionais
        ↓
Regras de negócio
        ↓
Histórias de usuário
        ↓
Integrações
        ↓
Estruturas de dados relevantes
        ↓
Testes, homologação e troubleshooting
```

Essa baseline passou a ter duas funções importantes:

1. preservar o conhecimento funcional do produto;
2. fornecer referência para evolução, investigação e validação da aplicação modernizada.

## 5. Comparação legado x versão modernizada

A principal estratégia de validação foi comparar sistematicamente o comportamento da aplicação legada com o novo ambiente.

A lógica era simples:

```text
Produção legada
      │
      ├── comportamento conhecido
      │
      ▼
Homologação modernizada
      │
      ├── comportamento observado
      │
      ▼
Comparação funcional
      │
      ├── equivalente → aprovado
      │
      ├── divergente esperado → melhoria validada
      │
      └── divergente indevido → defeito
```

Essa comparação incluía, entre outros aspectos:

- campos disponíveis;
- valores apresentados;
- grids;
- filtros;
- filtros condicionais;
- ordenações;
- extrações;
- mensagens;
- habilitação/desabilitação de ações;
- regras de negócio;
- transições de estado;
- comportamento de integrações.

## 6. Conhecimento funcional como dependência da migração

Durante a modernização, era frequente a equipe técnica precisar de contexto sobre o funcionamento real da aplicação.

Exemplos comuns envolviam perguntas como:

- quais filtros são obrigatórios em determinado contexto?
- quando um filtro deve aparecer ou desaparecer?
- a grid deve retornar exatamente os mesmos registros da extração?
- quais colunas dependem de determinada situação?
- o que deve acontecer quando um registro já foi processado?
- qual regra deve prevalecer quando dois estados parecem possíveis?

Em muitos desses casos, a resposta não estava em uma documentação anterior completa.

Era necessário combinar:

```text
Conhecimento do processo
        +
Comportamento observado no legado
        +
Histórias existentes
        +
Dados e integrações
        ↓
Definição funcional esperada
```

## 7. Regressão funcional

A regressão funcional foi conduzida por mim diretamente.

O objetivo não era apenas verificar se a tela carregava, mas se o fluxo permanecia semanticamente correto.

Uma validação típica seguia esta lógica:

```text
1. Executar cenário no legado
2. Registrar comportamento esperado
3. Executar cenário equivalente no novo ambiente
4. Comparar resultado
5. Identificar divergência
6. Classificar:
   - comportamento correto;
   - melhoria intencional;
   - regressão;
7. Encaminhar ajuste quando necessário
8. Retestar
```

## 8. Decisão funcional sobre divergências

Durante a migração surgiram situações em que a versão nova apresentava um comportamento diferente da aplicação anterior.

Nesses casos, eu tinha autonomia funcional para determinar se o comportamento representava:

- uma correção válida;
- uma melhoria desejada;
- uma diferença aceitável;
- ou um defeito de regressão.

Isso é importante porque uma divergência visual ou técnica não é automaticamente um bug.

A decisão precisava considerar a regra de negócio.

Conceitualmente:

```text
Comportamento diferente
        ↓
É compatível com a regra de negócio?
   ┌──────────┴──────────┐
  SIM                    NÃO
   │                      │
Mudança válida       Regressão funcional
   │                      │
Documentar            Corrigir e retestar
```

## 9. Priorização antes da entrada em produção

Nem toda divergência encontrada tinha o mesmo impacto.

A priorização das correções funcionais antes da implantação ficou sob minha responsabilidade.

A análise considerava fatores como:

- impacto no fluxo principal;
- risco de impedir operação;
- possibilidade de gerar inconsistência de dados;
- impacto em integrações;
- criticidade financeira/operacional;
- existência ou não de alternativa temporária;
- facilidade de identificação posterior;
- risco de regressão adicional.

Uma matriz conceitual utilizada para raciocínio era:

| Impacto | Existe contorno? | Decisão típica |
|---|---|---|
| Alto | Não | Corrigir antes da produção |
| Alto | Sim, mas arriscado | Prioridade alta |
| Médio | Sim | Avaliar janela e risco |
| Baixo | Sim | Pode entrar em backlog pós-go-live |

## 10. Go/No-Go funcional

Antes da entrada em produção, a decisão funcional dependia da homologação dos principais fluxos.

Minha participação cobria integralmente o aceite funcional da versão.

O raciocínio de go/no-go considerava:

```text
Fluxos críticos homologados?
        +
Regressões bloqueadoras resolvidas?
        +
Integrações principais validadas?
        +
Riscos conhecidos aceitáveis?
        ↓
GO / NO-GO funcional
```

Esse aceite não substituía os testes técnicos da equipe de desenvolvimento. Ele tratava especificamente da pergunta:

> A nova versão está funcionalmente apta a substituir a anterior na operação real?

## 11. Pós-implantação

A responsabilidade funcional continuou depois do go-live.

Minha atuação pós-implantação incluiu:

- monitoramento do comportamento em produção;
- execução de operações reais;
- comparação com o funcionamento esperado;
- identificação de regressões não detectadas anteriormente;
- diagnóstico inicial;
- suporte aos usuários;
- registro e priorização de correções;
- validação dos ajustes entregues pela equipe técnica.

A modernização, portanto, não terminou no deploy.

```text
Homologação
    ↓
Go-live
    ↓
Observação real
    ↓
Regressões residuais
    ↓
Diagnóstico
    ↓
Correção
    ↓
Nova validação
```

## 12. Quando a migração técnica funciona, mas o negócio quebra

Um dos principais aprendizados foi que uma migração pode estar tecnicamente funcional e ainda assim introduzir defeitos relevantes.

Exemplos conceituais:

```text
Endpoint responde 200
mas o estado local não é atualizado

Tela abre normalmente
mas um filtro condicional deixou de ser aplicado

Extração é gerada
mas os registros não correspondem à grid

Fluxo conclui tecnicamente
mas uma regra histórica deixou de ser executada
```

Esse tipo de problema reforça a necessidade de testes baseados em comportamento de negócio, e não apenas em disponibilidade técnica.

## 13. Relação entre documentação e modernização

A iniciativa de documentação não foi uma entrega formal imposta pela modernização.

Ela nasceu de uma decisão pessoal ao perceber que um sistema crítico, em evolução tecnológica, não possuía uma fonte consolidada de conhecimento funcional.

Isso criou um ciclo positivo:

```text
Modernização revelou ausência documental
            ↓
Documentação começou a ser estruturada
            ↓
Conhecimento ficou explícito
            ↓
Validação ficou mais consistente
            ↓
Incidentes futuros ficaram mais investigáveis
            ↓
Novas evoluções passaram a ter baseline melhor
```

## 14. Papel de ponte entre negócio, legado e nova implementação

Minha atuação durante a iniciativa pode ser resumida como uma ponte entre três perspectivas:

```text
            Regras de negócio
                  ▲
                  │
                  │
Sistema legado ◄──┼──► Nova implementação
                  │
                  ▼
          Validação funcional
```

Eu utilizava o comportamento existente, conhecimento operacional e regras de negócio para orientar a equipe técnica sobre o que precisava ser preservado, corrigido ou melhorado.

Essa função reduzia o risco de a modernização se transformar apenas em uma troca de tecnologia sem garantia de continuidade funcional.

## 15. Responsabilidades técnicas x funcionais

Este case não atribui a mim a execução técnica da migração de frameworks, runtime ou código.

A separação foi:

### Equipe de desenvolvimento

- atualização de código;
- adequação de frameworks e bibliotecas;
- ajustes de compatibilidade;
- correções técnicas;
- implantação técnica;
- tratamento das alterações necessárias na nova base tecnológica.

### Minha atuação

- conhecimento funcional e operacional;
- comparação legado x novo;
- esclarecimento de regras;
- documentação funcional;
- regressão funcional;
- homologação;
- priorização de correções;
- decisão sobre comportamento esperado;
- aceite funcional;
- monitoramento pós-produção;
- investigação e validação de regressões.

## 16. Competências demonstradas

- legacy system modernization;
- Product Ownership;
- Requirements Engineering;
- business analysis;
- functional governance;
- legacy knowledge discovery;
- regression testing;
- functional homologation;
- acceptance testing;
- release readiness;
- go/no-go analysis;
- defect prioritization;
- production validation;
- post-migration support;
- stakeholder communication;
- business-rule preservation;
- system documentation;
- knowledge management;
- integration validation;
- operational troubleshooting;
- collaboration with development teams;
- modernization risk management.

## 17. Aprendizados

1. **Modernização tecnológica não é apenas atualização de framework.** O comportamento de negócio precisa sobreviver à mudança.
2. **Código legado também é documentação — mas é uma documentação cara de interpretar.** Transformar esse conhecimento em artefatos funcionais reduz risco.
3. **Regressão funcional precisa comparar semântica, não apenas interface.**
4. **Uma versão tecnicamente estável pode estar funcionalmente incorreta.**
5. **A decisão de go-live precisa considerar risco operacional, não só conclusão técnica.**
6. **Documentação é parte da sustentabilidade de um sistema, não burocracia posterior.**
7. **A pessoa que conhece o processo real é um componente crítico de uma modernização bem-sucedida.**

## 18. Autoria e transparência

A decisão de modernizar foi conjunta entre a área responsável pelo produto e a equipe técnica.

A implementação técnica da migração foi realizada pelos desenvolvedores.

Minha contribuição esteve na governança funcional da modernização: comparação entre legado e novo ambiente, esclarecimento de regras, documentação, regressão funcional, homologação, priorização de correções, aceite para produção e acompanhamento pós-implantação.

A documentação funcional consolidada do sistema foi uma iniciativa própria conduzida por mim, com colaboração pontual da equipe técnica para registrar informações técnicas existentes que não estavam documentadas anteriormente.

Nenhum código proprietário, endpoint, estrutura física de banco de dados, ambiente, identificador, dado real ou detalhe interno da arquitetura original é reproduzido neste case.
