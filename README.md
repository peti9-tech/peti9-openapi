# peti9-openapi

Skill oficial para integrar assistentes de IA com as **Open APIs da Peti9**.

A Peti9 é a maior plataforma para hospitais veterinários do Brasil e líder em
soluções para o segmento pet e veterinário, atendendo petshops, clínicas e
hospitais veterinários.

Esta skill ensina o Claude a autenticar e consumir as APIs públicas da
plataforma: ela cuida do fluxo do token JWT, preenche os cabeçalhos de empresa
que toda chamada exige, escolhe o grupo de API certo para a pergunta que foi
feita, e traduz os erros que se parecem mas significam coisas diferentes.

## Instalação

Direto da Peti9:

```
/plugin marketplace add https://skill.peti9.com/.claude-plugin/marketplace.json
/plugin install peti9-openapi@peti9
```

Ou pelo diretório oficial de plugins do Claude, onde a skill também está publicada:

```
/plugin marketplace add anthropics/claude-plugins-community
/plugin install peti9-openapi@claude-community
```

Os dois caminhos instalam a mesma skill.

O passo a passo completo, em linguagem simples, está em
**[skill.peti9.com](https://skill.peti9.com)** — incluindo como obter as
credenciais e como configurá-las no macOS, Linux ou Windows.

## O que ela faz

Depois de instalada, você pergunta em português e o assistente busca a resposta:

- *"Quanto tem de ração no estoque da minha loja?"*
- *"Quem é o dono do telefone 11 99999-0000?"*
- *"Quais clientes fazem aniversário este mês?"*
- *"Me mostra o prontuário do atendimento 1234."*

## Credenciais

A skill usa três variáveis de ambiente, definidas pelo próprio usuário na
máquina dele:

| Variável | Origem |
|---|---|
| `PETI9_OPENAPI_API_KEY` | Developer Portal, em *My Dashboard* |
| `PETI9_OPENAPI_USERNAME` | usuário de integração criado pelo cliente, no formato `login@tenant` |
| `PETI9_OPENAPI_PASSWORD` | senha desse usuário |

Nada é enviado para a Peti9 pela skill: as chamadas vão direto do computador do
usuário para `openapi.peti9.com`. As credenciais nunca são impressas, e toda
operação de escrita em produção exige confirmação explícita.

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
