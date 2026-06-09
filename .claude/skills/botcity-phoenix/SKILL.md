---
name: botcity-phoenix
description: Migra automações UiPath (.xaml) para projetos Python usando o framework BotCity (BeAPro) e o orquestrador BotCity Maestro. Use sempre que o usuário pedir para converter, transcrever, migrar ou portar uma automação UiPath para Python/BotCity, mencionar um arquivo .xaml, falar em "sair do UiPath", "trocar para BotCity", ou pedir para gerar um projeto BotCity a partir de um workflow existente. Cobre o clone do template BeAPro, leitura/resumo do .xaml, transcrição passo a passo com a tabela de equivalências, organização em pastas por responsabilidade e geração do requirements.txt.
---

# UiPath → BotCity (Python) Migration Skill

Esta skill orienta o Claude Code no fluxo completo de migrar uma automação UiPath
para um projeto Python rodando no framework BotCity e orquestrado pelo BotCity
Maestro. O processo tem quatro etapas obrigatórias e nessa ordem:

1. **Clonar o template BeAPro** da BotCity como ponto de partida.
2. **Ler e resumir o `.xaml`** do UiPath de forma clara e direta.
3. **Transcrever o `.xaml` para Python**, ação por ação, usando a tabela de
   equivalências em `references/uipath_to_python.md` e o SDK do Maestro.
4. **Gerar a estrutura de pastas por responsabilidade** e o `requirements.txt`
   sem versões fixadas (o usuário fará o pinning depois dos testes).

Não pule nenhuma etapa, e não comece a escrever código Python antes de ter o
resumo do `.xaml` validado pelo usuário.

---

## Etapa 1 — Clonar o template BeAPro

O BeAPro (BotCity Enterprise Automation Project) é o framework starter oficial
da BotCity para automações corporativas. Use sempre como base — não invente uma
estrutura paralela.

```bash
git clone https://github.com/botcity-dev/beapro-framework.git <nome-do-projeto>
cd <nome-do-projeto>
rm -rf .git
git init
```

Em seguida:

- Renomeie o módulo interno (pasta com o código do bot) para um nome em
  `snake_case` compatível com o domínio da automação (ex.: `relatorio_vendas`,
  `conciliacao_bancaria`).
- Atualize `README.md`, `VERSION` e `setup.py`/`pyproject.toml` com o nome do
  projeto.
- Confirme com o usuário o `bot_id` antes de prosseguir — esse nome aparece no
  Maestro e em logs.

Se o usuário não conseguir clonar (rede corporativa bloqueada, por exemplo),
ofereça como alternativa o template clássico:

```bash
python -m cookiecutter https://github.com/botcity-dev/bot-python-template
```

A estrutura mínima gerada é:

```
<bot_id>/
├── MANIFEST.in
├── README.md
├── VERSION
├── <bot_id>/
│   ├── __init__.py
│   ├── __main__.py     # entrypoint, não mexer
│   ├── bot.py          # ponto de entrada do bot
│   └── resources/      # imagens, templates, arquivos auxiliares
├── requirements.txt
├── setup.py
├── build.sh
└── build.bat
```

A partir daqui, **expanda** essa estrutura (etapa 4); não a substitua.

---

## Etapa 2 — Ler e resumir o `.xaml`

O `.xaml` do UiPath é XML. Antes de transcrever, leia o arquivo e produza um
**resumo curto, em prosa direta**, com:

1. **Objetivo da automação** em uma frase.
2. **Sequência de blocos principais** em uma lista numerada curta (no máximo
   8–12 itens), nomeando cada ação UiPath e o que ela faz no contexto. Ex.:
   *"3. `Open Browser` → abre o portal do fornecedor"*, *"4. `Type Into` →
   preenche login e senha lidos de um asset"*.
3. **Entradas e saídas**: variáveis de entrada (arguments `in`), arquivos
   lidos, dados consumidos do Orchestrator (assets, queue items) e o que a
   automação produz (arquivo, e-mail, registro em banco etc.).
4. **Pontos de atenção**: ações com `RetryScope`, `TryCatch`, esperas
   condicionais, integrações externas e uso de credenciais.

### Como ler o `.xaml` na prática

Use `xml.etree.ElementTree` ou `lxml` para parsear. Os atributos relevantes
ficam principalmente em:

- `<Sequence DisplayName="...">` — agrupamentos lógicos.
- `<ui:*>` — activities do UiPath.UIAutomation (Click, TypeInto, etc.).
- `<ui:OpenBrowser>`, `<ui:UseApplicationCard>` — escopos.
- `Arguments` no nó raiz `<Activity>` — argumentos de entrada/saída.
- `<TryCatch>` — blocos com tratamento de erro.

Não tente reproduzir o XML inteiro no resumo. O usuário precisa entender o
**fluxo**, não a árvore. Se o `.xaml` referenciar outros `.xaml` via `Invoke
Workflow File`, liste-os e pergunte se deve abri-los também.

**Pare aqui e peça validação ao usuário antes da etapa 3.** Um resumo errado
gera um Python errado.

---

## Etapa 3 — Transcrever para Python + BotCity SDK

Com o resumo validado, transcreva ação por ação seguindo a tabela em
`references/uipath_to_python.md`. Essa tabela é **a fonte da verdade** para a
escolha de bibliotecas. Leia-a antes de escrever a primeira linha de código.

### Princípios obrigatórios

Estes princípios não são negociáveis e o código gerado precisa atendê-los:

- **PEP 8** para nomenclatura e formatação. Funções e variáveis em
  `snake_case`, classes em `PascalCase`, constantes em `UPPER_SNAKE_CASE`.
- **PEPs recentes**: use type hints (PEP 484/604), `pathlib` no lugar de
  `os.path`, f-strings, `match/case` quando fizer sentido, e a sintaxe nova de
  union (`int | None` em vez de `Optional[int]`) — exige Python 3.10+.
- **Docstrings obrigatórias** em todas as funções e classes públicas, no estilo
  Google ou NumPy. Comentários inline só onde a intenção não é óbvia pelo
  código.
- **`try/except` específico**, nunca `except:` ou `except Exception:` no nível
  externo de uma ação crítica. Capture a exceção mais específica que a
  biblioteca lança (ex.: `selenium.common.exceptions.TimeoutException`,
  `requests.exceptions.HTTPError`).
- **Nada de over-engineering**. Sem factories, sem strategy pattern, sem
  injeção de dependência elaborada se a automação tem 200 linhas. Código
  robusto > código rebuscado.
- **`logging` em vez de `print`**. Configure um logger por módulo
  (`logging.getLogger(__name__)`).
- **Recursos externos via variáveis de ambiente ou assets do Maestro**, nunca
  hardcoded. Use `os.environ` ou `maestro.get_credential(...)`.
- **Idempotência** sempre que possível: a automação deve poder rodar duas vezes
  sem efeito colateral indesejado (verificar antes de criar, sobrescrever de
  forma controlada).

### Integração com o BotCity Maestro (orquestrador)

O Maestro substitui o UiPath Orchestrator. Toda automação corporativa deve, no
mínimo:

1. Instanciar o SDK e recuperar a execução atual.
2. Reportar status final (`SUCCESS`, `FAILED`, `PARTIALLY_COMPLETED`).
3. Emitir alertas (`AlertType.INFO/WARN/ERROR`) nos pontos críticos.
4. Registrar resultados quantitativos (`new_result_step`) — itens processados,
   itens com erro etc.

Esqueleto mínimo do `bot.py`:

```python
"""Entrypoint do bot. Orquestra o fluxo e reporta ao Maestro."""
from __future__ import annotations

import logging

from botcity.maestro import AlertType, BotExecution, BotMaestroSDK
from botcity.maestro import FinishStatus

# Não derrubar o bot quando rodando localmente sem Maestro
BotMaestroSDK.RAISE_NOT_CONNECTED = False

logger = logging.getLogger(__name__)


def main() -> None:
    """Ponto de entrada chamado pelo BotCity Runner."""
    maestro = BotMaestroSDK.from_sys_args()
    execution: BotExecution = maestro.get_execution()

    try:
        # Importação local evita custo quando o bot só está validando args
        from relatorio_vendas.flow import executar_fluxo

        total, erros = executar_fluxo(maestro, execution)

        maestro.finish_task(
            task_id=execution.task_id,
            status=FinishStatus.SUCCESS,
            message=f"Processados {total} itens, {erros} com erro.",
            total_items=total,
            processed_items=total - erros,
            failed_items=erros,
        )
    except Exception as exc:  # noqa: BLE001 — borda do bot, logar e reportar
        logger.exception("Falha não tratada na execução do bot.")
        maestro.alert(
            task_id=execution.task_id,
            title="Falha crítica",
            message=str(exc),
            alert_type=AlertType.ERROR,
        )
        maestro.finish_task(
            task_id=execution.task_id,
            status=FinishStatus.FAILED,
            message=f"Erro: {exc}",
        )
        raise


if __name__ == "__main__":
    main()
```

Esse padrão — `try/except` único no topo do `main` para garantir que o Maestro
**sempre** recebe um status final — é a única exceção tolerada ao uso de
`Exception` genérico. As demais exceções devem ser tratadas no nível certo,
próximas à ação que pode falhar.

### Mapeamento das ações UiPath

Para cada activity identificada no resumo da Etapa 2, consulte
`references/uipath_to_python.md` e escolha a biblioteca apropriada. As regras
de bolso:

- **Web automation** → `selenium` ou `playwright` (preferir `playwright` em
  projetos novos pela API mais previsível).
- **Desktop com Computer Vision** → `botcity-framework-core`
  (`DesktopBot.find`, `click`, `type_keys`).
- **Excel** → `openpyxl` para arquivos puros, `pandas` para transformações.
- **PDF** → `pypdf` ≥ 6.7.2 (versão anterior tem CVE de DoS) ou `pdfplumber`
  para extração de texto/tabelas.
- **HTTP/API** → `httpx` (preferir sobre `requests` em código novo).
- **Banco** → `sqlalchemy` como camada universal.
- **OCR** → `pytesseract` ou `easyocr` dependendo do volume.

Não misture estilos: se escolher `playwright`, mantenha em todo o projeto. Não
combine `selenium` e `playwright` no mesmo bot.

---

## Etapa 4 — Estrutura de pastas por responsabilidade

A estrutura final, expandindo o template BeAPro, deve separar claramente as
camadas:

```
<bot_id>/
├── README.md
├── VERSION
├── requirements.txt
├── setup.py
├── .env.example                    # template das variáveis de ambiente
├── <bot_id>/
│   ├── __init__.py
│   ├── __main__.py
│   ├── bot.py                      # entrypoint + integração Maestro
│   ├── flow.py                     # orquestração do fluxo (alto nível)
│   ├── config.py                   # constantes, paths, leitura de .env
│   ├── actions/                    # ações atômicas reutilizáveis
│   │   ├── __init__.py
│   │   ├── web.py                  # clicks, fills, navegação
│   │   ├── excel.py                # leitura/escrita de planilhas
│   │   ├── pdf.py                  # parsing/geração de PDF
│   │   └── email_actions.py        # envio/leitura de e-mail
│   ├── services/                   # integrações externas
│   │   ├── __init__.py
│   │   ├── api_client.py           # chamadas HTTP a sistemas externos
│   │   └── database.py             # conexão e queries
│   ├── models/                     # dataclasses / pydantic models
│   │   ├── __init__.py
│   │   └── entities.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── logger.py               # configuração do logging
│   │   └── retry.py                # decorator de retry quando necessário
│   └── resources/                  # imagens, templates HTML, payloads
└── tests/
    ├── __init__.py
    ├── test_actions.py
    └── test_flow.py
```

### Por que essa divisão

- **`bot.py`** só sabe falar com o Maestro e delegar para `flow.py`. Trocar o
  orquestrador no futuro mexe num arquivo só.
- **`flow.py`** descreve a automação em alto nível (qual o passo a passo),
  chamando funções de `actions/` e `services/`. Lendo `flow.py` o usuário
  entende a automação sem precisar abrir mais nada.
- **`actions/`** são funções puras e testáveis. Cada arquivo agrupa por
  tecnologia (web, excel, pdf...) e não por feature do bot — isso favorece
  reuso entre projetos.
- **`services/`** isola chamadas a sistemas externos (APIs, banco). Permite
  mockar nos testes.
- **`models/`** centraliza as estruturas de dados. Use `dataclasses` para
  estruturas simples e `pydantic` quando precisar de validação.
- **`utils/`** é para infraestrutura: logging, retry, helpers genéricos. Se
  algo aqui virar grande, promova para uma pasta própria.

### Boas práticas de organização

- Nenhum módulo de `actions/` ou `services/` deve importar de `flow.py` ou de
  `bot.py`. A dependência flui sempre do topo para baixo.
- `flow.py` recebe o `maestro` e o `execution` como parâmetros — não os
  instancia. Isso facilita teste e reuso.
- Funções em `actions/` recebem objetos prontos (um `Page` do Playwright, um
  `DataFrame` do pandas) em vez de criar tudo do zero. Composição > herança.

---

## Etapa 5 — `requirements.txt` sem versões fixadas

Liste somente as dependências **diretas** que o projeto realmente importa,
**sem pinning**. O usuário vai rodar os testes e fixar versões depois com
`pip freeze` ou `pip-compile`.

Formato:

```text
# Core BotCity
botcity-framework-core
botcity-framework-web
botcity-maestro-sdk

# Adicionar conforme a automação usar:
# playwright
# selenium
# pandas
# openpyxl
# pypdf
# pdfplumber
# httpx
# sqlalchemy
# python-dotenv
```

Inclua **apenas** o que a transcrição efetivamente usa. Não jogue todas as
libs da tabela de equivalências no requirements — manter `requirements.txt`
enxuto é uma boa prática de segurança e manutenção.

Comente uma linha por seção indicando o motivo da dependência quando não for
óbvio, ex.:

```text
pypdf  # >= 6.7.2 será fixado após testes (CVE-2026-27628 em versões anteriores)
```

---

## Checklist final antes de entregar

Antes de dar o projeto como pronto, verifique:

- [ ] Template BeAPro clonado, `.git` removido, novo `git init` feito.
- [ ] Resumo do `.xaml` validado pelo usuário.
- [ ] Cada activity do `.xaml` tem correspondência no código Python (ou foi
      explicitamente marcada como não aplicável e justificada).
- [ ] `bot.py` reporta status final ao Maestro em todos os caminhos.
- [ ] `flow.py` legível como roteiro da automação.
- [ ] Funções com docstrings; tipos anotados.
- [ ] `try/except` específicos, sem `except:` solto.
- [ ] Sem credenciais hardcoded — tudo via `.env` ou Maestro credentials.
- [ ] Estrutura de pastas conforme o layout acima.
- [ ] `requirements.txt` sem versões fixadas, contendo só o que é usado.
- [ ] `README.md` explica como rodar local (`python -m <bot_id>`) e como
      empacotar para o Maestro (`./build.sh`).

---

## Referências

- `references/uipath_to_python.md` — tabela completa de equivalências
  UiPath → Python/BotCity. **Leia antes de codar.**
- Documentação BotCity Maestro:
  https://documentation.botcity.dev/maestro/maestro-sdk/
- Template BotCity Python:
  https://github.com/botcity-dev/bot-python-template
- BeAPro framework:
  https://github.com/botcity-dev/beapro-framework
