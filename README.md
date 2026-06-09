# 🔥 BotCity Phoenix

> Renasça seus workflows UiPath em código Python limpo, com BotCity.
> *Rise your UiPath workflows from the ashes as clean BotCity Python code.*

Phoenix é uma skill do Claude Code que pega arquivos do UiPath (`.xaml` + `.json`) e transforma em automações BotCity em Python. Sem drag-and-drop, sem amarras de licença, só código de verdade.

*Phoenix is a Claude Code skill that takes UiPath files (`.xaml` + `.json`) and turns them into BotCity Python automations. No drag-and-drop, no license shackles — just real code.*

---

## 🇧🇷 Português

### O que você precisa antes de começar

1. **Claude Code instalado** — se ainda não tem, [pega aqui](https://docs.claude.com/en/docs/claude-code).
2. **Arquivos do processo UiPath**:
   - O arquivo `.xaml` (o workflow em si)
   - O arquivo `.json` (geralmente `project.json` — contém dependências e config)
3. **BotCity SDK** no seu ambiente Python (`pip install botcity-framework-core`).

### Instalando a skill

Coloque a pasta da skill no diretório de skills do Claude Code:

```bash
# Crie o diretório se não existir
mkdir -p ~/.claude/skills

# Copie a skill phoenix-botcity para lá
cp -r ./phoenix-botcity ~/.claude/skills/
```

Pronto. Sério, é isso. Reinicie o Claude Code e a skill estará disponível.

### Como usar

Abra o Claude Code na pasta onde estão seus arquivos `.xaml` e `.json` e digite:

```
/phoenix-botcity
```

A skill vai:

1. 🔍 **Ler o `.xaml`** e mapear cada atividade do UiPath
2. 📦 **Verificar o `.json`** pra entender dependências e variáveis do projeto
3. 🐍 **Gerar o equivalente em Python** usando BotCity
4. 📝 **Documentar** os pontos que precisam de revisão humana (porque mágica não existe — ainda)

Você também pode dar mais contexto na chamada:

```
/phoenix-botcity converter o ProcessoFaturamento.xaml mantendo os nomes de variáveis originais
```

### O que esperar do output

- Um arquivo `.py` com a automação migrada
- Um `requirements.txt` com as libs BotCity necessárias
- Um `MIGRATION_NOTES.md` com tudo que precisa de atenção (seletores, credenciais, etc.)

### FAQ / Troubleshooting

**"A skill não aparece quando digito `/`"**
Confere se a pasta está em `~/.claude/skills/phoenix-botcity/` e se tem um `SKILL.md` dentro dela. Reinicia o Claude Code.

**"Meu workflow tem custom activities, vai funcionar?"**
Phoenix faz o melhor possível, mas custom activities geralmente viram um `# TODO` no Python. O `MIGRATION_NOTES.md` vai listar todas pra você revisar.

**"E os seletores do UiPath? Funcionam no BotCity?"**
Não diretamente. Phoenix converte os seletores UiPath para o padrão BotCity (`find` + `find_text`), mas seletores muito específicos do UiPath Studio podem precisar de ajuste manual. A skill marca esses pontos.

**"Suporta orquestração via UiPath Orchestrator?"**
Não — a saída é código BotCity puro. Pra orquestração, recomendamos o BotCity Maestro.

**"Posso converter um projeto inteiro com várias `.xaml`?"**
Pode. Aponta o Claude Code pra pasta raiz do projeto e roda `/phoenix-botcity converter o projeto inteiro`. Phoenix processa todas, mantendo a estrutura.

**"Variáveis do tipo `DataTable` do UiPath viram o quê?"**
Pandas DataFrames. É a tradução mais idiomática em Python.

**"O código gerado está esquisito / com bugs"**
Acontece. Phoenix é um acelerador, não um substituto pra revisão. Abra uma issue no repo interno do BotCity com o `.xaml` original e o output gerado.

**"Posso usar em projetos comerciais de clientes?"**
Sim, é essa a ideia. Só lembra de validar o output antes de entregar — você é responsável pela automação final.

---

## 🇬🇧 English

### Prerequisites

1. **Claude Code installed** — if you don't have it yet, [grab it here](https://docs.claude.com/en/docs/claude-code).
2. **UiPath process files**:
   - The `.xaml` file (the workflow itself)
   - The `.json` file (usually `project.json` — contains dependencies and config)
3. **BotCity SDK** in your Python environment (`pip install botcity-framework-core`).

### Installing the skill

Drop the skill folder into Claude Code's skills directory:

```bash
# Create the directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy the phoenix-botcity skill there
cp -r ./phoenix-botcity ~/.claude/skills/
```

That's it. Really. Restart Claude Code and the skill will be available.

### How to use it

Open Claude Code in the folder where your `.xaml` and `.json` files live and type:

```
/phoenix-botcity
```

The skill will:

1. 🔍 **Read the `.xaml`** and map every UiPath activity
2. 📦 **Check the `.json`** to understand dependencies and project variables
3. 🐍 **Generate the Python equivalent** using BotCity
4. 📝 **Document** anything that needs human review (because magic isn't real — yet)

You can also pass extra context in the call:

```
/phoenix-botcity convert BillingProcess.xaml keeping original variable names
```

### What to expect as output

- A `.py` file with the migrated automation
- A `requirements.txt` with the required BotCity libs
- A `MIGRATION_NOTES.md` flagging everything that needs attention (selectors, credentials, etc.)

### FAQ / Troubleshooting

**"The skill doesn't show up when I type `/`"**
Check that the folder lives at `~/.claude/skills/phoenix-botcity/` and that it has a `SKILL.md` inside. Restart Claude Code.

**"My workflow has custom activities, will it work?"**
Phoenix does its best, but custom activities usually become a `# TODO` in the Python output. The `MIGRATION_NOTES.md` lists all of them for your review.

**"What about UiPath selectors? Do they work in BotCity?"**
Not directly. Phoenix converts UiPath selectors to BotCity's pattern (`find` + `find_text`), but very Studio-specific selectors may need manual tuning. The skill flags these spots.

**"Does it support UiPath Orchestrator orchestration?"**
No — the output is pure BotCity code. For orchestration, we recommend BotCity Maestro.

**"Can I convert a whole project with multiple `.xaml` files?"**
Yes. Point Claude Code at the project root and run `/phoenix-botcity convert the whole project`. Phoenix processes all of them while keeping the structure.

**"What do UiPath `DataTable` variables become?"**
Pandas DataFrames. It's the most idiomatic Python translation.

**"The generated code looks weird / has bugs"**
It happens. Phoenix is an accelerator, not a substitute for review. File an issue on the internal BotCity repo with the original `.xaml` and the generated output.

**"Can I use this on commercial client projects?"**
Yes, that's the whole idea. Just validate the output before shipping — you're responsible for the final automation.

---

## 🔥 Why "Phoenix"?

Because migrating from UiPath to Python isn't just a port — it's a rebirth. Your automation comes back leaner, version-controllable, license-free, and ready to live in any CI/CD pipeline you throw at it.

*Porque migrar do UiPath pra Python não é só portar — é renascer. Sua automação volta mais enxuta, versionável, sem amarras de licença, e pronta pra rodar em qualquer pipeline de CI/CD.*

---

Made with 🔥 by BotCity