# peti9-openapi

Skill oficial para integrar assistentes de IA com as **Open APIs da Peti9**.

A Peti9 é a maior plataforma para hospitais veterinários do Brasil e líder em
soluções para o segmento pet e veterinário, atendendo petshops, clínicas e
hospitais veterinários.

Esta skill ensina o Claude a autenticar e consumir as APIs públicas da
plataforma: ela cuida do fluxo do token JWT, preenche os cabeçalhos de empresa
que toda chamada exige, escolhe o grupo de API certo para a pergunta que foi
feita, e traduz os erros que se parecem mas significam coisas diferentes.

## Onde funciona

No **Claude Code** — no terminal, na aba Code do aplicativo Claude ou no VS Code.
A skill precisa executar comandos no seu computador e ler as credenciais dele, então
**não funciona na conversa comum do Claude** (claude.ai ou aba Chat) nem no Cowork.

## Instalação

```
/plugin marketplace add peti9-tech/peti9-openapi
/plugin install peti9-openapi@peti9
```

Ou pelo diretório oficial de plugins do Claude, onde a skill também está publicada:

```
/plugin marketplace add anthropics/claude-plugins-community
/plugin install peti9-openapi@claude-community
```

Há ainda o endereço direto da Peti9,
`https://skill.peti9.com/.claude-plugin/marketplace.json`, que funciona apenas no
Claude Code pelo terminal — a interface web só aceita repositórios Git.

Todos instalam a mesma skill. O passo a passo completo, incluindo como obter as
credenciais, está em **[skill.peti9.com](https://skill.peti9.com)**.

## O que ela faz

Depois de instalada, você pergunta em português e o assistente busca a resposta:

- *"Quanto tem de ração no estoque da minha loja?"*
- *"Quem é o dono do telefone 11 99999-0000?"*
- *"Quais clientes fazem aniversário este mês?"*
- *"Me mostra o prontuário do atendimento 1234."*

## Credenciais

A skill usa três variáveis. Depois de instalar, basta pedir ao próprio Claude:

```
Configure as credenciais da skill peti9-openapi
```

Ele pede os três valores e os guarda no bloco `env` do `~/.claude/settings.json`,
que o Claude Code lê em toda sessão — funciona igual no macOS, no Linux e no
Windows, e também quando o Claude é aberto pelo aplicativo de desktop.

| Variável | Origem |
|---|---|
| `PETI9_OPENAPI_API_KEY` | Developer Portal, em *My Dashboard* |
| `PETI9_OPENAPI_USERNAME` | a **clínica** usa o próprio login da Peti9; o **parceiro** usa o usuário que a clínica criou para ele — sempre no formato `login@tenant` |
| `PETI9_OPENAPI_PASSWORD` | a senha desse login |

Nada é enviado para a Peti9 pela skill: as chamadas vão direto do computador do
usuário para `openapi.peti9.com`. As credenciais nunca são impressas.

**Consultas são feitas direto; qualquer alteração exige confirmação explícita.** A
skill só responde com dados que a API devolveu, e pergunta em qual loja operar
quando o usuário tem mais de uma.

**Por ser uma IA, o Claude pode cometer erros.** A responsabilidade por conferir as
informações e confirmar as alterações é sempre do usuário.

**Dando acesso a um parceiro**, a clínica cria um usuário próprio para ele e é
responsável por esse usuário: criação, permissões e revogação quando a parceria
acabar.

Por exigência da LGPD, a Peti9 não fornece credenciais de integração — elas são
criadas pelo cliente, que é o controlador dos dados.

## Acesso

O uso das Open APIs é restrito a parceiros certificados. Para liberar sua chave,
fale com o time comercial em **comercial@peti9.com**.

## Documentação

- Manual de instalação: [skill.peti9.com](https://skill.peti9.com)
- Referência das APIs: [devportal.peti9.com](https://devportal.peti9.com)

## Uso em outras ferramentas

A skill é um único arquivo Markdown
([`skills/peti9-openapi/SKILL.md`](skills/peti9-openapi/SKILL.md)). Em ferramentas
sem sistema de plugin, cole o conteúdo dele nas instruções do assistente.
