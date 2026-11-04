<div align="center">

# javalang-py

**Implementação em Python das classes `String`, `Integer` e `Float` da especificação Java SE 8**

[![CI](https://github.com/rukaaah/javalang-py/actions/workflows/ci.yml/badge.svg)](https://github.com/rukaaah/javalang-py/actions/workflows/ci.yml)
[![Python](https://img.shields.io/badge/python-3.10%2B-3776ab)](https://www.python.org/)
[![Release](https://img.shields.io/badge/release-v1.0.0-2ea44f)](https://github.com/rukaaah/javalang-py/releases)
[![Ruff](https://img.shields.io/badge/lint-ruff-261230)](https://docs.astral.sh/ruff/)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[Visão geral](#visão-geral) · [Instalação](#instalação) · [Uso](#uso) · [Testes](#testes-e-qualidade) · [Adaptações](#adaptações-em-relação-ao-java) · [Documentação](#documentação)

</div>

`javalang-py` reproduz em Python o contrato público de três classes do `java.lang`: construtores, métodos de instância e métodos estáticos, com os mesmos nomes em `camelCase` e o mesmo comportamento de borda do Java SE 8 — incluindo aritmética de 32 bits com sinal, precisão IEEE 754 de precisão simples e imutabilidade de strings.

> [!NOTE]
> Este repositório é o Trabalho Prático da disciplina de **Gerência de Configuração de Software (GCS 2026.1)** da Universidade Federal de Mato Grosso do Sul (UFMS). O princípio orientador é: *"o código é o pretexto; a prática de GCS é o objeto da avaliação"*. Todo o processo — issues, ADRs, baselines, relatórios e auditoria — está registrado em [`docs/`](docs).

## Visão geral

| Classe | Módulo | Escopo coberto |
| :--- | :--- | :--- |
| `JString` | [`javalang/jstring.py`](javalang/jstring.py) | ~60 métodos: construtores a partir de `char[]`/`byte[]`, acesso e busca, extração, transformações, regex, Unicode e utilitários estáticos |
| `JInteger` | [`javalang/jinteger.py`](javalang/jinteger.py) | ~40 métodos: constantes de 32 bits, parsing em qualquer base, formatação `signed`/`unsigned`, operações bitwise e conversões primitivas |
| `JFloat` | [`javalang/jfloat.py`](javalang/jfloat.py) | Constantes e predicados IEEE 754, parsing e formatação, conversões de bits (`floatToIntBits`) e comparações com tratamento de `NaN` |

As três classes são interoperáveis: `JString` aceita `JInteger` e `JFloat` em `concat`, `contains`, `join` e `format`, e os parsers aceitam `JString` como entrada.

## Instalação

**Requisitos:** Python 3.10 ou superior.

```bash
git clone https://github.com/rukaaah/javalang-py.git
cd javalang-py
```

Crie e ative um ambiente virtual isolado:

```bash
python -m venv venv
```

| Sistema | Comando de ativação |
| :--- | :--- |
| Linux / macOS | `source venv/bin/activate` |
| Windows | `venv\Scripts\activate` |

Instale o projeto em modo editável, junto com as ferramentas de desenvolvimento (`pytest`, `pytest-cov`, `coverage`, `ruff`, `mypy`):

```bash
pip install -e ".[dev]"
```

> [!TIP]
> O prefixo `(venv)` no prompt confirma que o ambiente está ativo. Para validar a instalação, rode `pytest --version`.

## Uso

### JString

Toda operação de transformação devolve uma **nova** `JString`, preservando a imutabilidade do Java. A comparação com `str` nativa funciona diretamente.

```python
from javalang.jstring import JString

s = JString("hello world")

s.length()                      # 11
s.charAt(4)                     # 'o'
s.indexOf("o")                  # 4
s.substring(6) == "world"       # True
s.toUpperCase() == "HELLO WORLD"  # True
s.replace("world", "java") == "hello java"  # True
s.split(" ")[1] == "world"      # True

JString.format("%s tem %d anos", JString("Ana"), 30) == "Ana tem 30 anos"  # True
```

### JInteger

```python
from javalang.jinteger import JInteger

JInteger.MAX_VALUE              # 2147483647
JInteger.parseInt("7f", 16)     # 127
JInteger(255).toHexString()     # 'ff'
JInteger(10).toBinaryString()   # '1010'
JInteger.bitCount(7)            # 3
JInteger.reverseBytes(1)        # 16777216
JInteger.toUnsignedString(-1)   # '4294967295'
```

### JFloat

```python
from javalang.jfloat import JFloat

JFloat.parseFloat("3.14")       # 3.140000104904175 — coagido para 32 bits
JFloat(3.14).toString()         # '3.14'
JFloat.floatToIntBits(1.0)      # 1065353216
JFloat.isFinite(float('inf'))   # False
JFloat.MAX_VALUE                # 3.4028235e+38
```

### Interoperabilidade

```python
from javalang.jfloat import JFloat
from javalang.jinteger import JInteger
from javalang.jstring import JString

JInteger.parseInt(JString("ff"), 16)                    # 255
JString("valor: ").concat(JInteger(42)) == "valor: 42"  # True
JString.join(JString("-"), JString("a"), JInteger(1), JFloat(2.5)) == "a-1-2.5"  # True
```

## Testes e qualidade

A suíte reúne 170 casos distribuídos em quatro arquivos: um por classe, mais [`tests/test_interop.py`](tests/test_interop.py) para a integração entre elas.

```bash
python -m pytest
```

O `pyproject.toml` já injeta o relatório de cobertura, então o comando acima mede `javalang` automaticamente. Para ver as linhas descobertas:

```bash
pytest --cov=javalang --cov-report=term-missing
```

Verificação de estilo e de tipos:

```bash
ruff check .
```

```bash
mypy javalang
```

> [!IMPORTANT]
> O `ruff` está configurado para ignorar `N802`, `N803` e `N815`. O `camelCase` dos métodos é uma exigência do contrato Java e tem precedência sobre a PEP 8 neste projeto — a decisão está registrada na [ADR 0001](docs/adr/0001-nomenclatura-classes.md).

A CI ([`.github/workflows/ci.yml`](.github/workflows/ci.yml)) executa lint e testes com cobertura a cada push e a cada Pull Request contra a `main`. Nenhum PR é mesclado com checks em vermelho.

## Adaptações em relação ao Java

Python e Java divergem em tipagem, sobrecarga, gerenciamento de memória, Unicode e locale. O projeto não contorna essas diferenças à força: documenta cada uma delas.

| Diferença | Tratamento adotado |
| :--- | :--- |
| Sobrecarga de métodos | Unificada em uma única assinatura com argumentos opcionais (ex.: `parseInt(s, radix=10)`, `substring(begin, end=None)`) |
| `StringBuilder` / `StringBuffer` | Sem equivalente no CPython; o construtor de `JString` lança `NotImplementedError` indicando o uso de `''.join()` |
| `intern()` | O CPython não expõe pool de strings para objetos customizados; o retorno é documentado como aproximação |
| Dependências de locale | Métodos sensíveis a `Locale` usam fallback documentado (ver [ADR 0003](docs/adr/0003-tratamento-locale.md)) |
| `int` de precisão arbitrária | Overflow de 32 bits simulado com máscaras e complemento de dois |
| `float` de 64 bits | Precisão simples emulada com o módulo `struct` (`>f`), reproduzindo arredondamento e `NaN` do Java |

O catálogo completo, método a método, está em [`docs/adaptacoes.md`](docs/adaptacoes.md).

## Estrutura do projeto

```
javalang/            # Código fonte das três classes
├── jstring.py
├── jinteger.py
└── jfloat.py
tests/               # Suíte pytest, um arquivo por classe + interoperabilidade
docs/
├── adaptacoes.md            # Catálogo de adaptações e não-implementações
├── adr/                     # Registros de Decisão Arquitetural
├── auditoria.md             # Auditoria interna da release v1.0.0
├── itens-de-configuracao.md # Catálogo de itens de configuração
├── registro-conflitos.md    # Atas de resolução de conflitos de merge
├── relatorios/              # Relatório de status por baseline
└── uso-de-ia.md             # Registro de uso de IA por método
.github/workflows/ci.yml     # Lint e testes automatizados
```

## Equipe e papéis

| Papel | Responsável | Atribuições |
| :--- | :--- | :--- |
| **Mantenedor** | [Pedro Cremonini](https://github.com/rukaaah) | Cria o repositório, configura proteção de branch, aprova merges em `main`, cria releases e tags. Garante a integridade das baselines. |
| **Gerente de Configuração** | [Angelo Antônio](https://github.com/angelo-acds) | Mantém atualizados os documentos de itens de configuração, ADRs e adaptações. Conduz a auditoria interna ao final. |
| **Engenheiro de Qualidade** | [Yan Victor Gomes](https://github.com/GomesYV) | Configura e mantém a CI, garante cobertura mínima dos testes, revisa PRs sob a ótica de qualidade e regressão. |
| **Desenvolvedores** | [Gabriel Mattos](https://github.com/GabrielMattosA), [Jonathan do Amaral](https://github.com/JhonnPA) | Implementam as classes. Cada desenvolvedor é dono de pelo menos um conjunto de métodos atribuído formalmente via issue. |
| **Relator** | [Cleiton Pinheiro](https://github.com/Ton-07) | Produz os relatórios de status a cada baseline e o relatório final. Sintetiza o trabalho para a apresentação. |

Os papéis definem responsabilidade, não exclusividade: todos os membros revisam PRs uns dos outros.

## Fluxo de trabalho

A equipe segue o GitHub Flow com TDD estrito ([ADR 0002](docs/adr/0002-modelo-ramificacao.md) e [ADR 0004](docs/adr/0004-fluxo-de-integração.md)):

1. A branch `main` é protegida — nenhum commit direto é permitido após o primeiro sprint.
2. Toda alteração não-trivial nasce em uma **Issue**.
3. O desenvolvimento ocorre em branch própria (`feat/`, `fix/`, `docs/`, `chore/`, `test/`).
4. O Engenheiro de Qualidade abre o ciclo escrevendo os testes a partir da documentação oficial do Java SE 8.
5. Os desenvolvedores usam os testes falhos como guia de implementação e escrevem apenas o código necessário para torná-los verdes.
6. A integração é feita por **Pull Request**, com aprovação documentada de ao menos um membro além do autor e CI verde.

> [!WARNING]
> Regras anti-atalho vigentes no projeto: um Pull Request não pode conter mais de 7 casos de teste implementados e um commit não pode conter mais de 3 métodos de teste.

## Documentação

| Documento | Conteúdo |
| :--- | :--- |
| [`docs/adaptacoes.md`](docs/adaptacoes.md) | Catálogo de métodos adaptados ou não implementados, com justificativa técnica |
| [`docs/itens-de-configuracao.md`](docs/itens-de-configuracao.md) | Itens de configuração sob controle e seus responsáveis |
| [`docs/adr/`](docs/adr) | Decisões arquiteturais: nomenclatura, ramificação, locale e fluxo de integração |
| [`docs/relatorios/`](docs/relatorios) | Relatório de status de cada baseline, de `v0.1` a `v1.0` |
| [`docs/auditoria.md`](docs/auditoria.md) | Auditoria interna da release final: aderência ao processo, rastreabilidade e lições aprendidas |
| [`docs/registro-conflitos.md`](docs/registro-conflitos.md) | Atas de resolução dos conflitos de merge |
| [`docs/uso-de-ia.md`](docs/uso-de-ia.md) | Registro de uso de IA, módulo por módulo |

As baselines estão marcadas como tags: `v0.1-functional`, `v0.2-jinteger`, `v0.3-jfloat`, `v0.4-jstring` e `v1.0.0`.

## Apresentação

[Vídeo de apresentação do trabalho](https://drive.google.com/file/d/1qOtc9kvFMtVMf1OmZ0luE4oNyb5pBj4Q/view?usp=sharing)
