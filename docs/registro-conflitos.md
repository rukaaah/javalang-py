# Registro de Resolução de Conflitos de Merge

## Sobre este documento
Este arquivo reúne as atas de resolução dos conflitos de merge enfrentados pela equipe ao longo do projeto. Cada registro descreve o contexto, os arquivos afetados e a decisão tomada, servindo como evidência de rastreabilidade para a auditoria interna (ver [`auditoria.md`](auditoria.md)).

---

## Conflito de Merge 01

### Contexto do Conflito
Ao tentar mesclar as atualizações, o Git apontou conflito no arquivo `tests/test_jinteger.py`. Isso ocorreu porque uma das branches continha um template genérico de documentação e um teste de rascunho vazio para `parseInt` (`test_jinteger_parse_int`), enquanto a branch atual implementou a estrutura de testes focada no escopo inicial da **Issue #11** (`feat/jinteger-base`), adicionando o fluxo de QA/Dev e os testes específicos das constantes estáticas.

### Arquivos Afetados
- `tests/test_jinteger.py`

### Decisão e Resolução
A equipe analisou as marcações do Git e decidiu manter o novo escopo focado na Issue #11. Descartamos o rascunho do método `parseInt` (que será abordado em PRs futuros) e mantivemos a classe `TestJIntegerConstantes` com as asserções de `MAX_VALUE`, `MIN_VALUE` e `SIZE`. Além disso, a importação correta de `JInteger` foi ativada e a docstring com as regras anti-atalho de GCS e o fluxo de trabalho QA/Dev foi preservada. O código foi limpo e os testes via `pytest` foram executados com sucesso no ambiente local antes do push final, garantindo a integridade do contrato da classe.

## Conflito de Merge 02

### Contexto do Conflito
Ao realizar o merge da PR #31 (implementação dos métodos parseInt e parseUnsignedInt da classe JInteger), o Git identificou conflitos nos arquivos javalang/jinteger.py e docs/adaptacoes.md. O conflito ocorreu devido a alterações concorrentes na lógica de sobrecarga de métodos e nas assinaturas dos parsers, uma vez que a branch atual introduziu o uso de argumentos opcionais (radix=10) e validações de sinal negativo, enquanto a branch main continha registros de documentação e stubs que divergiam da nova implementação estruturada.

### Arquivos Afetados
- `javalang/jinteger.py`
- `docs/adaptacoes.md`

### Decisão e Resolução
A equipe analisou as marcações do Git e decidiu priorizar a implementação da PR #31, mantendo a unificação das assinaturas dos métodos via argumentos opcionais para contornar a limitação de sobrecarga do Python.No arquivo javalang/jinteger.py, resolvemos os conflitos mantendo a lógica de validação de sinal (-) no parseUnsignedInt e o padrão radix=10.No arquivo docs/adaptacoes.md, os registros foram mesclados de forma aditiva, garantindo que a nova justificativa técnica sobre o comportamento de parsing conforme o Java SE 8 esteja integrada aos registros anteriores.A solução foi validada localmente com ruff e pytest, assegurando que o contrato da classe JInteger permanece íntegro e em conformidade com as regras anti-atalho de GCS.

## Conflito de Merge 03

### Contexto do Conflito
Ao realizar o merge da PR #50 (padronização das entradas da classe JInteger para o template oficial), o Git identificou conflitos no arquivo docs/adaptacoes.md. O conflito ocorreu devido à convergência de alterações no mesmo arquivo, onde a branch da PR aplicava uma nova formatação visual enquanto a branch main continha atualizações técnicas e descritivas concorrentes nas mesmas entradas documentais.

### Arquivos Afetados
- `jinteger.py`
- `docs/adaptacoes.md`

### Decisão e Resolução
A equipe analisou as marcações do Git e decidiu priorizar a nova estrutura de documentação proposta na PR #50, garantindo a preservação integral do conteúdo técnico presente na branch main. No arquivo docs/adaptacoes.md, resolvemos os conflitos migrando os textos exatos e atualizados das três entradas da branch destino para dentro do novo formato padronizado. Nenhuma informação técnica ou justificativa foi descartada. A solução foi revisada e validada localmente com ruff e pytest, assegurando que a documentação permanece íntegra e em total conformidade com as regras anti-atalho de GCS.

*Documento gerado para fins de rastreabilidade e auditoria da disciplina GCS 2026.1.*
