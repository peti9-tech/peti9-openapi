---
name: peti9-openapi
description: >
  Autentica e consome as Open APIs públicas da Peti9 (openapi.peti9.com): gera o
  token JWT a partir de credenciais em variáveis de ambiente, monta os headers
  obrigatórios (Bearer + empresa_id + empresa_cadastro) e escolhe o grupo de API
  certo entre os publicados no Developer Portal. Use quando o usuário disser
  "chamar a open api da peti9", "gerar o token da openapi", "pegar o JWT da api
  pública", "consultar item/estoque/preço/tutor/pedido/prontuário pela API pública",
  "integração de laboratório, plano de saúde, CRM ou MeuPet",
  "qual endpoint da open api faz Y", "configure as credenciais da skill
  peti9-openapi", "guardar minha api key / usuário / senha da Peti9", "como
  consigo a chave da api", "posso usar meu login da Peti9", "criar usuário
  para um parceiro", "revogar o acesso do parceiro", "em qual loja/unidade",
  "não tem api para isso", "pedir uma api nova", ou quando
  reclamar de 401/403/500 na openapi, "Missing Authentication Token",
  "HEADER CUSTOMER IS A MANDATORY INFO", "o token não funciona na api pública".
  Vale apenas para as chamadas que passam por openapi.peti9.com.
---

# Open APIs da Peti9 — token e catálogo

As Open APIs da Peti9 dão acesso aos dados dos clientes que a sua empresa e a
Peti9 atendem em conjunto: tutores, pets, itens e estoque, atendimentos, pedidos,
assinaturas de documento. Elas são servidas em `https://openapi.peti9.com/<grupo>`
e documentadas no [Developer Portal](https://devportal.peti9.com), onde fica o
contrato detalhado de cada endpoint.

Três coisas costumam custar tempo de quem está chegando, e é o que esta skill
resolve: a autenticação passa a senha **no caminho da URL**, quase toda chamada
exige **dois cabeçalhos de empresa** além do token, e o mesmo endpoint aparece em
vários grupos sob caminhos diferentes.

> `openapi.peti9.com` **é produção**. Não há sandbox: o que volta é dado de cliente
> real. Ver "Escrita em produção".

## Onde esta skill funciona

Esta skill funciona no **Claude Code** — no terminal, na aba Code do aplicativo
Claude ou no VS Code. Ela depende de duas coisas que só existem ali: executar
comandos na máquina do usuário e ler as credenciais do `~/.claude/settings.json`.

Se você perceber que está numa conversa comum do Claude (claude.ai, aba Chat) ou
no Cowork — sem terminal para executar —, diga isso logo, em linguagem simples:
aqui você consegue explicar como a API funciona, mas não consultar os dados dele;
para isso, ele precisa usar o Claude Code. Não tente contornar.

## Como falar com o usuário

Quem usa esta skill é, na maioria das vezes, alguém da clínica ou do pet shop —
não um desenvolvedor. Estas regras valem para a skill inteira.

- **Linguagem simples por padrão.** Diga "produtos e estoque", não "grupo Items";
  "a sua loja do Centro", não "empresa_id 3". Nome técnico — cabeçalho, endpoint,
  código HTTP, caminho da API — só quando o usuário pedir, ou quando ele mesmo
  estiver escrevendo código. Colar uma mensagem de erro não faz de ninguém
  desenvolvedor.
- **Responda primeiro o que foi perguntado.** Não despeje o processo inteiro de
  configuração em quem fez uma pergunta pontual; ofereça o próximo passo em uma
  frase.
- **Nunca entregue um comando para o usuário rodar.** Checar, testar e configurar
  é trabalho seu. Se você não conseguir executar, diga isso em uma frase e siga
  com a orientação — sem transferir a tarefa técnica.
- **Pergunte sempre que a decisão for do usuário.** Qual loja, qual período, qual
  tutor quando houver dois parecidos, se é para gravar ou só consultar. Nunca
  escolha por ele o que muda o resultado. E não pergunte o que você mesmo consegue
  descobrir — o código da loja, o tenant, o usuário do token.
- **Uma pergunta por vez, com opções.** "Em qual loja: Centro, Norte ou Sul?" é
  melhor que "qual é o empresa_id?".

## Só o que a API devolveu

**Nunca invente dado.** Toda informação que você der sobre a clínica — nome de
tutor, pet, preço, estoque, horário, número de atendimento, valor — tem que ter
vindo de uma resposta da API nesta conversa.

- **Se a API não devolveu, você não sabe.** Diga isso. Não complete com um valor
  "provável", não estime, não arredonde para parecer completo.
- **Resposta vazia é resposta.** Uma lista vazia quer dizer "nada encontrado" —
  não "deve haver algo". Diga que não encontrou e ofereça buscar de outro jeito.
- **Não junte pedaços para fabricar um número** que a API não dá — um total que
  ela não calcula, uma média de itens que você não consultou todos.
- **Se não conseguir consultar**, não responda de memória nem com um exemplo.
- **Não invente identificadores** — código de atendimento, id de item, id de
  tutor. Busque na API ou pergunte.
- **Se alguma parte for dedução sua**, e não dado da API, diga que é dedução.

## Configuração inicial (uma vez)

A skill precisa de três dados: uma **chave de acesso** (API Key), um **usuário** e
uma **senha** da plataforma Peti9.

**Primeiro, descubra com quem você está falando** — sempre que o assunto for
configuração, credenciais ou acesso. Se não estiver claro, pergunte:

> Você é a clínica ou pet shop que usa a Peti9, ou um parceiro de integração que
> está conectando o sistema de vocês à Peti9 de um cliente?

Os dois casos são igualmente comuns. Muda de onde vêm o usuário e a senha; a
chave de acesso vem do mesmo lugar para os dois.

### 1. A chave de acesso → no Developer Portal

Vale para a clínica e para o parceiro. Os botões do portal estão em **inglês** —
use os nomes exatos, para o usuário reconhecer o que vê na tela.

1. Acesse `https://devportal.peti9.com` e clique em **Register**.
2. Confirme o e-mail e faça **Sign In**.
3. Abra **My Dashboard** no menu do topo. O quadro **API Key** mostra a chave.
4. **A chave nasce sem acesso a nada.** Quem libera é o time comercial da Peti9 —
   e-mail `comercial@peti9.com` ou WhatsApp `(48) 98843-5447` —, a partir do nome
   da empresa, do e-mail usado no cadastro do portal e de para que o acesso vai
   ser usado. Para parceiros, a liberação passa também por NDA e contrato de
   parceria.
5. **Quando o acesso for aprovado, o comercial avisa por e-mail.** A partir desse
   aviso a mesma chave já funciona nas áreas liberadas — não é preciso gerar outra.
   A liberação não é imediata; oriente o usuário a esperar o e-mail em vez de
   testar a toda hora.

Enquanto a liberação não sai, toda chamada devolve `403 {"message":"Forbidden"}`.

### 2. O usuário e a senha → depende de quem é

#### A clínica usa o próprio login

A clínica usa **o login dela na plataforma Peti9**, o mesmo com que entra no
sistema todo dia. Não precisa criar nada. A skill enxerga o que esse login
enxerga: as mesmas lojas e as mesmas permissões.

Duas situações pedem atenção — mencione só se forem acontecer:

- **Senha com símbolo.** A senha vai no endereço da autenticação, e `/`, `#`, `?`,
  `%` e `&` quebram a chamada (`/` não tem conserto). Se a senha tiver algum
  desses, a saída é trocá-la por uma só com letras e números, ou criar um usuário
  próprio para a skill.
- **Troca de senha.** Se a senha mudar na plataforma, a skill para de funcionar
  até as credenciais serem atualizadas — basta pedir de novo "Configure as
  credenciais da skill peti9-openapi".

Se a clínica preferir não usar o login pessoal — para limitar as lojas que a skill
enxerga, por exemplo —, pode criar um usuário só para isso, do jeito descrito mais
abaixo. É opcional.

#### A clínica vai dar acesso a um parceiro

A clínica **nunca entrega o próprio login**. Ela cria um usuário novo, só para
aquele parceiro, e **a responsabilidade por esse usuário é dela**. Ao orientar,
diga isso com todas as letras — "a responsabilidade é sua" — e liste os três
deveres; é o ponto que mais precisa ficar claro:

- **Criar** o usuário e a senha, e entregar ao parceiro — de preferência o usuário
  por um canal e a senha por outro.
- **Controlar as permissões.** O Grupo Usuário define o que o parceiro acessa, e a
  caixa Empresas define quais lojas. Marque só o que a parceria precisa.
- **Revogar quando a parceria acabar**, desativando o usuário na plataforma. A
  partir daí o parceiro não consegue gerar acesso novo. Um acesso gerado antes da
  desativação pode continuar valendo por até 24 horas, que é a validade do token —
  se o encerramento for delicado, desative com essa antecedência.

A Peti9 não cria, não gerencia e não revoga esse usuário: por LGPD, a clínica é a
controladora dos dados, e o parceiro é operador. O parceiro tira a **própria**
chave de acesso no Developer Portal (passo 1) — a clínica não repassa chave.

#### O parceiro usa o usuário que a clínica criou

O parceiro recebe da clínica o usuário e a senha. Se o acesso passar a responder
"usuário ou senha inválido", a primeira hipótese é a clínica ter trocado a senha ou
desativado o usuário — isso se resolve com ela, não com a Peti9.

#### Como a clínica cria um usuário novo

Na plataforma Peti9, em **Configurações → Usuários → novo usuário**:

- **Nome** — identifique para que serve, ex. `Integração <parceiro>`, para ninguém
  desativar achando que é gente.
- **Usuário** — o login. Aceita **apenas letras e números**: sem espaço, acento,
  ponto ou `@`. A restrição é do sistema, não só da tela.
- **Senha** e **Confirmar a senha** — mínimo 8 caracteres, com letras e números.
  Use só letras e números, pelo motivo do símbolo explicado acima.
- **Grupo Usuário** — define a que telas e dados o usuário tem acesso.
- **Ativo** — marcado.
- **Empresas** — marque as lojas cujos dados ele vai ler ou gravar. Sem esse
  vínculo, a API responde vazio, sem erro.

Não existe "tipo integração" nem perfil especial de API: é um usuário comum da
plataforma. Quem limita o alcance dele é o Grupo Usuário e as lojas marcadas.

### 3. O username da API não é o login

Ao salvar, a plataforma propaga o usuário para o Keycloak como
**`<login>@<tenant>`**. É esse valor composto que vai na variável — não o login
sozinho — vale para o login pessoal e para um usuário criado para parceiro. Um
usuário `integracao` no tenant `clinicaexemplo` autentica como
`integracao@clinicaexemplo`.

Se você não souber o tenant, ele está no token de qualquer sessão da plataforma
(claim `tenant_id`), ou basta perguntar ao cliente.

### 4. Guardar as credenciais — quem faz é você, Claude

As três variáveis vão no bloco **`env` do `~/.claude/settings.json`**. O Claude Code
lê esse arquivo sozinho e injeta as variáveis em toda sessão e em todo comando que
executa — independente do sistema operacional, do shell, e de o Claude ter sido
aberto pelo terminal ou pelo aplicativo de desktop.

| Variável | Origem |
|---|---|
| `PETI9_OPENAPI_API_KEY` | *My Dashboard* do Developer Portal (passo 1) |
| `PETI9_OPENAPI_USERNAME` | `<login>@<tenant>` do usuário criado (passos 2 e 3) |
| `PETI9_OPENAPI_PASSWORD` | a senha definida no passo 2 |

**Não use `~/.bashrc`, `~/.zshrc` nem arquivo `.env` carregado pelo shell.** Esse
caminho parece funcionar e falha em silêncio: o aplicativo de desktop, aberto pela
interface gráfica, nunca lê os arquivos de inicialização do shell; e no Ubuntu o
`~/.bashrc` sai logo no começo quando o shell não é interativo, então uma linha
acrescentada no fim nunca roda. O usuário executa tudo certo, reinicia, e o Claude
continua sem as variáveis.

#### Quando o usuário pedir para configurar

**Antes de pedir qualquer dado, mostre o aviso de primeiro acesso**, uma vez, com
estas palavras ou equivalentes:

> Antes de começar: por ser uma inteligência artificial, eu posso cometer erros. A
> responsabilidade por conferir as informações que eu trouxer e por confirmar
> qualquer alteração nos seus dados é sempre sua. Eu nunca altero nada sem a sua
> confirmação explícita.

Depois do aviso:

1. **Peça os três valores numa única mensagem**, dizendo de onde vem cada um e
   lembrando que o usuário vai no formato `login@tenant`. Avise uma vez, sem
   dramatizar, que o que ele digitar fica no histórico desta conversa — e que, se
   preferir não digitar a senha aqui, ele pode editar o arquivo à mão (abaixo).
2. **Faça cópia de segurança** do `~/.claude/settings.json`, se ele existir.
3. **Mescle, nunca sobrescreva.** O arquivo quase sempre já tem outras
   configurações do usuário. Leia o JSON, acrescente as três chaves dentro de
   `env` (criando `env` se não existir) e grave de volta. Se o arquivo não existir,
   crie com apenas o bloco `env`.
4. **Valide o JSON depois de gravar.** Um `settings.json` inválido faz o Claude Code
   **ignorar o arquivo inteiro** — o usuário perde todas as configurações dele, em
   silêncio. Se a validação falhar, restaure a cópia de segurança.
5. **Restrinja a permissão** do arquivo ao próprio usuário (`chmod 600` no macOS e
   no Linux), porque agora ele guarda uma senha.
6. **Nunca repita os valores** na resposta — confirme só pelos nomes das variáveis.
7. **Peça para abrir uma sessão nova** do Claude. As variáveis do `settings.json`
   entram em vigor na próxima sessão, não nesta.

Em macOS e Linux, com `python3` disponível, isto faz os passos 2 a 5 de uma vez:

```bash
PK='<api-key>' PU='<login@tenant>' PS='<senha>' python3 - <<'PY'
import json, os, pathlib, shutil
arq = pathlib.Path.home() / ".claude" / "settings.json"
arq.parent.mkdir(parents=True, exist_ok=True)
dados = {}
if arq.exists():
    shutil.copy2(arq, arq.with_suffix(".json.bak"))
    dados = json.loads(arq.read_text(encoding="utf-8") or "{}")
dados.setdefault("env", {}).update({
    "PETI9_OPENAPI_API_KEY": os.environ["PK"],
    "PETI9_OPENAPI_USERNAME": os.environ["PU"],
    "PETI9_OPENAPI_PASSWORD": os.environ["PS"],
})
arq.write_text(json.dumps(dados, indent=2, ensure_ascii=False) + "\n", encoding="utf-8")
json.loads(arq.read_text(encoding="utf-8"))   # valida: se falhar, restaure o .bak
os.chmod(arq, 0o600)
print("credenciais gravadas em", arq)
PY
```

No Windows, ou sem `python3`, use a ferramenta de JSON do sistema (no PowerShell,
`ConvertFrom-Json` / `ConvertTo-Json`) e siga as mesmas regras: cópia de segurança,
mesclar, validar. O arquivo fica em `%USERPROFILE%\.claude\settings.json`.

Se existir um `~/.peti9-openapi.env` de uma configuração antiga, ele não é mais
necessário. O valor do `settings.json` prevalece sobre o que estiver exportado no
shell, então não há conflito — mas vale oferecer a remoção, para a senha não ficar
guardada em dois lugares.

#### Quem prefere editar à mão

Abrir `~/.claude/settings.json` e acrescentar, sem apagar o que já existe:

```json
{
  "env": {
    "PETI9_OPENAPI_API_KEY": "sua-chave-do-portal",
    "PETI9_OPENAPI_USERNAME": "login@tenant",
    "PETI9_OPENAPI_PASSWORD": "sua-senha"
  }
}
```

Se o arquivo já tiver um bloco `env`, as três linhas entram dentro dele.

#### Regras que não se negociam

**Não** grave as credenciais em nenhum arquivo dentro de repositório — nem em
`.claude/settings.json` de projeto, que costuma ir para o git. O lugar é só o
`settings.json` do **usuário**, na pasta pessoal dele.

## Se alguma variável faltar

Não pare só para avisar: **ofereça configurar agora**, seguindo o procedimento
acima. Se o usuário aceitar, peça os valores; se preferir fazer sozinho, aponte a
edição manual. Nunca invente valor nem use placeholder com cara de credencial.

Se as variáveis faltarem **logo depois** de uma configuração, a causa quase sempre
é a sessão: o `settings.json` só vale em sessão nova. Peça para abrir outra antes
de investigar qualquer coisa.

## Passo 1 — gerar o token

```bash
TOKEN=$(curl -s -H "x-api-key: $PETI9_OPENAPI_API_KEY" \
  "https://openapi.peti9.com/auth/public/oauth/token/$PETI9_OPENAPI_USERNAME/$PETI9_OPENAPI_PASSWORD")
```

Sem dependência nenhuma além do `curl`.

Confira sem imprimir o token:

```bash
case "$TOKEN" in *.*.*) echo "token ok";; *) echo "falhou: $TOKEN";; esac
```

Pontos de atenção:

- **A credencial vai no caminho da URL**, então caractere especial na senha é
  problema de roteamento, não de autenticação. O `@` do username passa cru, sem
  encode. Já `/`, `#`, `?`, `%` e `&` na **senha** quebram a URL, e `/` não tem
  conserto: cru, muda a rota; como `%2F`, é rejeitado antes de chegar na
  aplicação. **Use senha alfanumérica no usuário de integração** — é a regra mais
  simples e a plataforma já exige letras e números. Se herdou uma senha com
  especiais, percent-encode antes de montar a URL; se tiver `/`, não há saída
  senão trocá-la com o cliente.
- **A resposta é o JWT cru, em `text/plain`.** Não é JSON: `jq -r .access_token`
  devolve vazio e você vai culpar a credencial.
- **Referencie a variável, nunca o valor.** O comando expandido entra no histórico
  do shell e no transcript. Não ecoe `$TOKEN` nem a senha.
- **O token vale 24h** e já carrega `tenant_id` e `user_id`. Não existe header de
  tenant: o tenant é o do token. Trocar de cliente é gerar outro token, com a
  credencial daquele cliente.

O `x-api-key` é obrigatório em todos os grupos, **inclusive na rota de token**.
Um `403 {"message":"Forbidden"}` quer dizer que a sua chave ainda não tem acesso
liberado àquele grupo — é liberação comercial pendente, não erro de código. (Logo
depois de uma liberação nova, a resposta ainda pode vir `Forbidden` por cerca de
um minuto. Isso é para o **seu** diagnóstico: só diga ao usuário se ele contar que
a liberação acabou de sair — veja "Quando a resposta for 403 Forbidden".)

## Passo 2 — chamar um endpoint

Toda chamada leva o token e **quatro cabeçalhos de empresa**, que dizem em qual
loja a operação acontece:

```bash
curl -s -H "x-api-key: $PETI9_OPENAPI_API_KEY" \
     -H "Authorization: Bearer $TOKEN" \
     -H "empresa_id: $EMP_ID" \
     -H "empresa_cadastro: $EMP_CADASTRO" \
     -H "empresa_financeiro: $EMP_FINANCEIRO" \
     -H "empresa_preco: $EMP_PRECO" \
     "https://openapi.peti9.com/items/item/paginar/lista?pagina=0&limit=2&orderBy=nome"
```

### Escolher a loja

**A loja é decisão do usuário. Os números são trabalho seu.**

1. Gere o token (passo 1) e leia o `user_id` do payload do JWT.
2. Chame `GET /companies/empresa/{userId}/empresas-login`. Ela devolve as lojas que
   esse login pode acessar, cada uma com os quatro valores prontos:

   ```json
   [{"codigo":12,"nome":"Clinica Exemplo LTDA","nomeFantasia":"Clínica Exemplo Centro","empresaCadastro":12,"empresaFinanceiro":12,"empresaPreco":10}]
   ```

3. Decida conforme o que voltou:
   - **Uma loja:** use-a sem perguntar — não há decisão a tomar — e diga na
     resposta de qual loja é o dado. Numa gravação, cite a loja na confirmação.
   - **Várias lojas, e o usuário não disse qual:** pergunte, listando pelo nome.
     Use o `nomeFantasia`, que é como a equipe chama a loja; se vier vazio, o
     `nome`. Numa **consulta**, ofereça também a opção de ver **todas** — e aí
     consulte uma por uma e mostre o resultado separado por loja. Numa
     **gravação**, é sempre **uma loja só**: nunca grave em várias de uma vez.
     Faça a pergunta **neutra**: não sugira uma resposta ("eu assumiria todas"),
     porque a escolha é do usuário, não sua. Exemplo — o usuário diz "tenho três
     lojas, quanto tem de ração no estoque?": **não** consulte as três nem uma delas
     por conta própria; pergunte "Em qual loja: Centro, Norte ou Sul — ou prefere
     ver as três?" e espere a resposta. Mencionar as lojas não é escolher entre
     elas.
   - **Várias lojas, e o usuário já disse qual:** use essa. Se o nome que ele usou
     não bater claramente com nenhuma, pergunte em vez de escolher a mais parecida.
   - **Nenhuma (`[]`):** o login não tem loja vinculada. Resolve-se na plataforma,
     em Configurações → Usuários, na caixa Empresas — pela própria clínica. Se
     quem está falando é o parceiro, oriente-o a pedir isso à clínica.
4. **Mande os quatro cabeçalhos da loja escolhida, sempre juntos e da mesma linha:**

   | Cabeçalho | Vem de |
   |---|---|
   | `empresa_id` | `codigo` |
   | `empresa_cadastro` | `empresaCadastro` |
   | `empresa_financeiro` | `empresaFinanceiro` |
   | `empresa_preco` | `empresaPreco` |

   Nunca misture valores de lojas diferentes, nunca repita o `empresa_id` nos
   outros três, e nunca peça esses números ao usuário. Em rede com matriz e
   filiais os quatro divergem — é para isso que existem — e um valor trocado traz
   dado de outra loja, ou uma lista vazia, sem erro nenhum.
5. **Guarde a escolha durante a conversa.** Escolhida a loja, continue nela nas
   perguntas seguintes, dizendo sempre de qual loja é cada resposta. Se o usuário
   mencionar outra, troque. Se ficar ambíguo, pergunte.

**Um caso do CRM:** a listagem de clientes do CRM (`/crm/cliente/paginar/lista`)
está declarada no gateway com o cabeçalho `empresa_financeira`, no feminino — o
resto da API usa `empresa_financeiro`. Até isso ser corrigido, nessa chamada mande
os dois, com o mesmo valor.

Confira também de onde veio o `user_id`: em shell, `$UID` é variável reservada do
próprio sistema, e usá-la por engano no lugar do id do token devolve `[]` sem erro.

### Diagnóstico

Cada falha tem uma assinatura própria; casar a assinatura com a causa poupa a maior
parte da investigação.

| Resposta | Causa |
|---|---|
| `401` + `[{"user":"Usuário ou senha inválido!"...}]` | Credencial errada mesmo. |
| `401` sem corpo, numa chamada de negócio | Bearer ausente ou token vencido (24h). Refaça o passo 1. |
| `403` + `{"message":"Missing Authentication Token"}` | Apesar do nome, **não é autenticação**: a URL não casou com nenhuma rota. Base path do grupo errado, caminho digitado errado, ou `/` na credencial. |
| `403` + `{"message":"Forbidden"}` | Outra coisa: a sua API Key não tem acesso liberado àquele grupo. É liberação comercial, pedida por grupo — **siga "Quando a resposta for 403 Forbidden", logo abaixo, para falar com o usuário.** |
| `500` + `CustomerNotFoundException: HEADER CUSTOMER IS A MANDATORY INFO` | Faltou algum dos **quatro** cabeçalhos de empresa — `empresa_id`, `empresa_cadastro`, `empresa_financeiro` ou `empresa_preco`. Confira os quatro, não só os dois primeiros. |
| `500` + `MissingServletRequestParameterException: ... 'x' ... is not present` | Falta um parâmetro de query obrigatório — o nome está na própria mensagem. |
| `429` | Quota da API Key estourada. |

Duas consequências práticas: **erro de contrato volta `500`, não `400`** — não
trate 500 como instabilidade antes de ler o campo `tech` do corpo, que traz a
exception. E os dois `403` **nunca** são usuário/senha: leia o corpo para separar
rota errada (`Missing Authentication Token`) de API Key não autorizada
(`Forbidden`).

### Quando a resposta for 403 Forbidden

O diagnóstico técnico está na tabela acima. Esta seção aplica "Como falar com o
usuário" a esse caso: é sobre **como falar disso com o usuário** — que quase nunca é desenvolvedor, e para quem um erro técnico
soa como "estraguei alguma coisa".

**1. Faça a checagem você mesmo.** Gere o token e chame um grupo que costuma
funcionar (o `empresas-login` do Companies serve). Se o token sai e esse outro
grupo responde, a chave está viva e o bloqueio é só naquele grupo. **Nunca
entregue um `curl` para o usuário rodar** — é trabalho seu, não dele. Se você não
conseguir checar (credenciais ainda não configuradas, sem acesso ao terminal),
não transfira a checagem: explique a causa mais provável e siga para o passo 3.

Se o `Forbidden` vier **já na rota de token**, o caso é outro: a chave inteira
ainda não foi liberada, não um grupo específico. Diga isso — nenhuma consulta vai
funcionar até a liberação.

**2. Traduza o grupo para o que o usuário reconhece.** Ele não sabe o que é
"Digital Signature"; ele sabe o que é *documento assinado*.

| Grupo | Diga ao usuário |
|---|---|
| Companies | os dados das suas unidades |
| Tutors | o cadastro de tutores e pets |
| Items | os produtos, serviços e estoque |
| Orders | os pedidos |
| Services | os atendimentos e prontuários |
| Digital Signature | os documentos assinados digitalmente |
| Crm | a agenda e os atendimentos do CRM |
| Health Plan | o plano de saúde |
| Laboratory Integration | os resultados de exames de laboratório |
| App MeuPet | os links de acesso do app MeuPet |

**3. Responda nesta ordem, em linguagem simples:**

- **Primeiro, que não é erro dele.** Nada quebrou, a senha está certa — é uma
  permissão que ainda não foi liberada.
- **O que está bloqueado, e o que funciona.** No acesso parcial, diga as duas
  coisas: *"Consigo ver seus produtos e estoque, mas a consulta de documentos
  assinados ainda não foi liberada para a sua chave."*
- **Quem libera e como falar com eles.** A liberação é feita pelo time comercial
  da Peti9, pelos dois canais:
  - **E-mail:** comercial@peti9.com
  - **WhatsApp:** (48) 98843-5447
- **Ofereça escrever a mensagem.** Monte o pedido pronto para ele copiar.

**4. O modelo da mensagem.** Preencha só o que você sabe de fato, e deixe
marcado o que precisa ser completado:

```
Olá, time comercial da Peti9.

Uso a integração pelas Open APIs e preciso da liberação de acesso para
<o que está bloqueado, em linguagem de negócio> (grupo <nome técnico>).

Empresa: <nome da empresa de vocês — pergunte>
E-mail do cadastro no Developer Portal: <pergunte>
Para que usamos: <finalidade, se o usuário contou>
```

Cuidado com a empresa: **quem pede a liberação é o dono da API Key.** Se quem
está falando é a clínica, a empresa é ela mesma. Se é um parceiro, é a empresa do
parceiro — não a clínica que aparece no `empresas-login`, que é o cliente cujos
dados estão sendo acessados. Na dúvida, pergunte.

**5. O que não fazer:**

- **Não use jargão.** Comece sempre em linguagem simples; detalhe técnico só se o
  usuário pedir, ou se ele mesmo estiver escrevendo código. Colar uma mensagem de
  erro não faz de ninguém desenvolvedor.
- **Não tente contornar** o bloqueio buscando o mesmo dado por outro grupo.
- **Não repita a chamada em laço** esperando que passe — `Forbidden` não é
  instabilidade, e cada tentativa consome a quota.
- **Não mencione a propagação de um minuto** a menos que o usuário diga que a
  liberação acabou de sair. Para quem ainda nem pediu a liberação, a frase dá a
  falsa esperança de que basta esperar, e ele adia o contato com o comercial.

Os parâmetros obrigatórios variam por endpoint e os nomes são em português
(`pagina`, `limit`, `orderBy`). Quando faltar um, a mensagem do `500` diz qual —
adicione e repita, em vez de adivinhar o contrato inteiro.

## Os grupos de API

Há dois tipos. **Grupo de domínio** reúne endpoints por assunto. **Grupo de
parceiro** é um recorte curado dos mesmos endpoints sob um base path próprio.
Existem outros grupos no portal, de uso interno ou de parceiros específicos. **Não
os apresente ao usuário como caminho para um dado** — veja "Quando o dado não está
aqui".

**A regra que evita a maior parte dos 403:** se o contexto menciona um parceiro
nomeado, use o base path **do parceiro**. `/item/paginar/lista` existe tanto em
`/items` quanto em `/crm` — e a assinatura que abre um não abre o outro.

| Grupo | Base path | Serve para |
|---|---|---|
| Auth | `/auth` | Gerar o JWT |
| Companies | `/companies` | Empresas: por id, por CNPJ, ativas, empresas do usuário logado |
| Tutors | `/tutor` | Tutores: cadastro, busca paginada, detalhe, nome e pets por telefone, appcode, aniversariantes |
| Items | `/items` | Produto ou serviço: lista, detalhe, estoque, preço, preço de custo (Bling) |
| Orders | `/orders` | Pedidos do PDV: criar, buscar por id externo, mudar situação |
| Services | `/services` | Atendimento: prontuário e informação médica |
| Digital Signature | `/assinatura` | Assinaturas de documento: criar, listar, atualizar, excluir, metadata, troca de arquivo |
| Crm | `/crm` | Recorte CRM: atendimento (cancelar, reagendar), links de acesso, itens |
| Health Plan | `/healthplan` | Recorte plano de saúde: atendimento, itens, tutores, pets |
| Laboratory Integration | `/laboratory` | Resultado e anexo de exame, CRMV do profissional |
| App MeuPet | `/tutor` | Geração de link de acesso do tutor (`/api/link/...`) |

**App MeuPet divide o base path `/tutor` com o grupo Tutors**, expondo só os
endpoints de link — não é erro de digitação.

**Para o contrato de uma operação** (headers, parâmetros, corpo, schema de
resposta), abra o Swagger do grupo no portal —
`https://devportal.peti9.com/apis/<id>`, grupo no menu lateral, login de parceiro
exigido — em vez de adivinhar. Esta tabela envelhece: quando um grupo novo é
publicado ou um base path muda, ela precisa ser atualizada aqui.

## Do que o usuário fala para onde isso mora

Quem pergunta usa o vocabulário do dia a dia da clínica ou do pet shop, não o
nome técnico do endpoint. Traduza antes de procurar na lista:

| O usuário diz | É isto | Onde procurar |
|---|---|---|
| cliente, dono, tutor, responsável | tutor | Tutors |
| bicho, animal, pet, paciente | pet | Tutors (`/pet/detalhe`), Health Plan |
| loja, unidade, filial, matriz, clínica | empresa | `empresa_id` / Companies |
| produto, mercadoria, ração, medicamento, vacina | item do tipo PRODUTO | Items |
| serviço, banho, tosa, consulta, exame, procedimento | item do tipo SERVICO | Items |
| quanto tem, saldo, sobrou, disponível | estoque | Items (`/item/stock`) |
| quanto custa, valor, tabela, quanto cobro | preço | Items (`/item/price`) |
| custo, quanto paguei, margem | preço de custo | Items (`/item/costPrice`) |
| atendimento, comanda, consulta, ficha | atendimento | Services, Crm |
| prontuário, histórico, receita, laudo | prontuário | Services |
| agenda, horário, marcação, encaixe | agenda | Crm |
| venda, pedido, compra, comanda fechada | pedido | Orders |
| documento assinado, termo, autorização | assinatura | Digital Signature |
| resultado de exame, laboratório | exame | Laboratory Integration |
| aniversariante | tutores do mês | Tutors (`/cliente/aniversariantes`) |

Duas ambiguidades que valem confirmar antes de chamar: **"comanda"** tanto é
atendimento aberto (Services) quanto pedido fechado (Orders); e **"cliente"** é o
tutor para quem opera a clínica, mas é a própria clínica para quem fala do ponto
de vista do parceiro. Na dúvida, pergunte em uma frase — não chute.

## Receitas

Os caminhos completos das perguntas mais comuns. Todos começam pelo passo 1
(token) e pela descoberta da empresa descrita acima.

**"Quanto tem de X no estoque?"** — e também "quanto custa X"

Uma chamada só. A busca por nome já devolve `estoque` e `precoVenda` de cada item:

```bash
curl -s -H "x-api-key: $PETI9_OPENAPI_API_KEY" -H "Authorization: Bearer $TOKEN" \
     -H "empresa_id: $EMP_ID" -H "empresa_cadastro: $EMP_CADASTRO" \
     -H "empresa_financeiro: $EMP_FINANCEIRO" -H "empresa_preco: $EMP_PRECO" \
     "https://openapi.peti9.com/items/item/paginar/lista?pagina=0&limit=20&orderBy=nome&pesquisa=racao"
```

O `pesquisa` casa por parte do nome, então "ração" acha "Ração Premium 15kg". Cada
item volta com `idItem`, `descricao`, `estoque`, `precoVenda` e `valorCusto` — é
quase sempre tudo o que a pergunta precisa.

**Não use `/item/stock` nem `/item/price` para consultar um produto.** Apesar do
nome, eles servem a sincronização em lote: pedem `companyId` e trabalham por
janela de atualização (`updatedSince` / `updatedUntil`). Consultados por `id`
solto devolvem `[]`, o que parece "produto sem estoque" e não é.

**"Quem é o dono do telefone 11 99999-0000?"**
Direto, sem busca paginada:
`GET /tutor/tutor/{tutorPhoneNumber}` para o nome, `/tutor/{tutorPhoneNumber}/pets/name`
para os pets.

**"Quais clientes fazem aniversário este mês?"**
`GET /tutor/cliente/aniversariantes?dataInicial=AAAA-MM-DD&dataFinal=AAAA-MM-DD` — as
duas datas são obrigatórias; para "este mês", do dia 1 ao último dia do mês.

**"Me mostra o prontuário do atendimento 123."**
`GET /services/atendimento/123/prontuarioatendimento`. Para os dados médicos
detalhados, `/prontuarioinfomedica`.

**"Quais são minhas lojas?"**
É a própria chamada de descoberta: `GET /companies/empresa/{userId}/empresas-login`.
Responda com os nomes, não com os códigos.

## Endpoints por grupo

Use esta lista para saber **qual grupo e qual caminho** atendem o que o usuário
pediu. Os parâmetros de cada operação não estão aqui — para o contrato completo
(query, corpo, schema de resposta), abra o Swagger do grupo no Developer Portal.

Se o que o usuário pede não aparece nesta lista, veja "Quando o dado não está aqui"
antes de tentar um caminho parecido.

> Levantado em 21/09/2026 a partir da documentação publicada. A plataforma evolui: se
> um caminho daqui devolver `403 Missing Authentication Token`, ele mudou ou saiu
> do ar — confirme no portal em vez de insistir.

**Companies** — base `/companies`

| Método | Caminho | O que faz |
|---|---|---|
| `GET` | `/empresa/buscar/cnpj/{cnpj}` | Retrieve Companies by CNPJ |
| `GET` | `/empresa/buscar/id/{idEmpresa}` | Retrieve Company by ID |
| `GET` | `/empresa/index/{companyId}` | Company Data |
| `GET` | `/empresa/listaativa` | Retrieve Active Companies |
| `GET` | `/empresa/{userId}/empresas-login` | Retrieve User's Companies |

**Tutors** — base `/tutor`

| Método | Caminho | O que faz |
|---|---|---|
| `GET` | `/cliente/aniversariantes` | List birthday pet owners |
| `GET` | `/cliente/detalhe/{id}` | Returns the tutor's details based on the provided ID |
| `GET` | `/cliente/paginar/lista` | Paginate tutor list |
| `POST` | `/pessoa` | Register a new tutor |
| `GET` | `/searchperson/paginate/list` | Retrieve paginated list of tutors using filter parameters |
| `GET` | `/tutor/{tutorPhoneNumber}` | Tutor Name |
| `GET` | `/tutor/{tutorPhoneNumber}/appcode` | Returns the tutor’s appcode based on the provided phone |
| `GET` | `/tutor/{tutorPhoneNumber}/pets/name` | Tutor Pet's Names |

**Items** — base `/items`

| Método | Caminho | O que faz |
|---|---|---|
| `GET` | `/item/costPrice` | Get item cost price and stock for Bling integration |
| `GET` | `/item/detalhe/{id}` | Get item detail |
| `GET` | `/item/paginar/lista` | List all items |
| `GET` | `/item/price` | Get item current price |
| `GET` | `/item/stock` | Get item current quantity in stock |
| `GET` | `/item/stockAndPrice` | Get item current quantity in stock and his current price |

**Orders** — base `/orders`

| Método | Caminho | O que faz |
|---|---|---|
| `POST` | `/new/pdv/sale/order` | Create a new order |
| `GET` | `/new/pdv/sale/order/byExternalOrderId/{externalOrderId}` | Get an order by his external order ID |
| `PATCH` | `/new/pdv/sale/order/changeSituation` | Change an order situation |
| `PUT` | `/new/pdv/sale/order/changeSituation` | Change an order situation |
| `GET` | `/searchperson/paginate/list` | Retrieve paginated list of tutors using filter parameters |

**Services** — base `/services`

| Método | Caminho | O que faz |
|---|---|---|
| `GET` | `/atendimento/{id}/prontuarioatendimento` | Service medical record |
| `GET` | `/atendimento/{id}/prontuarioinfomedica` | Medical details of the service record |

**Digital Signature** — base `/assinatura`

| Método | Caminho | O que faz |
|---|---|---|
| `GET` | `/assinatura` | List signatures by document IDs |
| `POST` | `/assinatura` | Create signature |
| `PUT` | `/assinatura` | Update signature |
| `DELETE` | `/assinatura/{idAssinatura}` | Delete signature |
| `POST` | `/assinatura/{idAssinatura}/arquivo/{idArquivo}/replace` | Substitute file signature |
| `GET` | `/assinatura/{idAssinatura}/signature-metadata` | Get signature metadata |
| `GET` | `/cliente/detalhe/{id}` | Returns the tutor's details based on the provided ID |

**Crm** — base `/crm`

| Método | Caminho | O que faz |
|---|---|---|
| `PUT` | `/atendimento/cancelamentos` | Cancel multiple appointments |
| `GET` | `/atendimento/listaagenda` | List company appointments |
| `PUT` | `/atendimento/reagendaratendimentos` | Reschedule multiple appointments |
| `PUT` | `/atendimento/reagendarnocheckin` | Reschedule an appointment at check-in time |
| `PUT` | `/atendimento/reagendarnocheckout` | Reschedule an appointment at checkout time |
| `PUT` | `/atendimento/{id}/cancelamento` | Cancel a single appointment |
| `GET` | `/atendimento/{id}/prontuarioatendimento` | Service medical record |
| `PUT` | `/atendimento/{id}/reagendar` | Reschedule a single appointment |
| `GET` | `/cliente/detalhe/{id}` | Returns the tutor's details based on the provided ID |
| `GET` | `/cliente/paginar/lista` | Paginate tutor list |
| `GET` | `/empresa/listaativa` | Retrieve Active Companies |
| `POST` | `/integracao/agenda/agendamento` | Create appointment in schedule |
| `GET` | `/integracao/agenda/horarios` | List available time slots by date |
| `GET` | `/item/paginar/lista` | List all items |
| `GET` | `/itemagenda/listaagenda` | List service agenda items |
| `GET` | `/link/by-company/{tenantId}/{companyId}` | Generates an access link for tenant and company. |
| `GET` | `/link/{tenantId}/{tutorId}/{companyId}/{petId}` | Generates an access link for a tutor and pet at a tenant and company. |
| `POST` | `/pessoa` | Register a new tutor |
| `PUT` | `/pessoa/{id}` | Update tutor data for integration |
| `POST` | `/pet` | Register a new pet |
| `GET` | `/pet/detalhe` | Get detailed list of pets by tutor |

**Laboratory Integration** — base `/laboratory`

| Método | Caminho | O que faz |
|---|---|---|
| `POST` | `/exame/resultado` | Receive exam result from Vetcloud |
| `GET` | `/profissional/infoprofissional/{id}` | Retrieve professional CRMV and map number |

**Health Plan** — base `/healthplan`

| Método | Caminho | O que faz |
|---|---|---|
| `GET` | `/atendimento/{id}/prontuarioatendimento` | Service medical record |
| `GET` | `/cliente/paginar/lista` | Paginate tutor list |
| `GET` | `/item/paginar/lista` | List all items |
| `GET` | `/pet/detalhe` | Get detailed list of pets by tutor |
| `POST` | `/pessoa` | Register a new tutor |
| `POST` | `/pet` | Register a new pet |

**App MeuPet** — base `/tutor`

| Método | Caminho | O que faz |
|---|---|---|
| `GET` | `/auth/customer/{tenantId}/{tutorId}/{customerId}/{petId}/{token}` | Valida um link de acesso já gerado |
| `GET` | `/auth/link/by-company/{tenantId}/{customerId}` | Gera link de acesso no escopo da empresa |
| `GET` | `/auth/link/{tenantId}/{tutorId}/{customerId}/{petId}` | Gera link de acesso do tutor para um pet |
| `GET` | `/auth/link/{tenantId}/{tutorId}/{customerId}/{petId}/{token}` | Gera link de acesso com token existente |


## Quando o dado não está aqui

Nem tudo da plataforma está disponível pelas APIs. Se o que o usuário pede não
existe em nenhum grupo da tabela, **diga isso com clareza** — e não monte uma
resposta aproximada somando pedaços de outros endpoints: um número incompleto leva
a uma decisão errada.

Responda, em linguagem simples e nesta ordem:

1. **Hoje não há acesso a esse dado pelas APIs.** Sem rodeio, e sem dar a
   entender que o usuário errou.
2. **Onde o dado está agora**, se você souber — normalmente nos relatórios da
   própria plataforma Peti9. É o caminho imediato.
3. **Dá para pedir uma API nova.** A clínica ou o parceiro pode solicitar ao time
   comercial da Peti9 — e-mail `comercial@peti9.com` ou WhatsApp
   `(48) 98843-5447` — uma API para esse dado. **O pedido é avaliado pela Peti9.**
4. **Ofereça escrever o pedido.** O que ajuda a avaliação é a justificativa: qual
   dado, para que vai ser usado e com que frequência.
5. **Ofereça o que dá para fazer hoje**, se existir algo próximo.

**Não prometa.** Não diga que o dado "existe em outra API", que "há grupos que não
estão aqui e podem ser liberados", nem dê prazo — você não sabe se aquele dado
existe em alguma API nem se o pedido será aprovado. O certo é: pode-se pedir, e a
Peti9 avalia.

O modelo do pedido — preencha o que souber e marque o que falta:

```
Olá, time comercial da Peti9.

Gostaria de solicitar uma API para acessar <o dado, em linguagem de negócio>.

Empresa: <nome>
Somos: <clínica cliente da Peti9 | parceiro de integração>
Para que vamos usar: <finalidade>
Com que frequência: <ex.: uma vez por dia>
E-mail do cadastro no Developer Portal: <se já tiver>
```

## Consultar é livre; alterar exige confirmação

**Consultas** podem ser feitas direto. Busque, mostre o resultado e converse com o
usuário para refinar — outro período, outra loja, um filtro a mais.

**Qualquer operação que mude dados exige um "sim" explícito do usuário antes de
executar.** Vale para `POST`, `PUT`, `PATCH` e `DELETE` — e para qualquer chamada
que **gere** algo, mesmo sendo uma leitura: gerar um link de acesso do App MeuPet
é um `GET` e ainda assim cria um acesso novo. A regra é pelo efeito, não pelo tipo
da chamada. `openapi.peti9.com` é produção, e a API não desfaz nada: criar pedido,
cancelar ou reagendar atendimento, excluir assinatura, confirmar anexo de exame,
finalizar venda.

Como pedir a confirmação:

1. **Diga em linguagem simples o que vai ser feito e em qual loja** — "Vou cancelar
   o agendamento das 14h de hoje do Rex, tutor Carlos Souza, na loja Centro."
2. **Quando der, confira antes com uma consulta.** Antes de cancelar ou excluir,
   busque o registro e mostre o que ele é. Número de atendimento se troca fácil, e
   excluir o errado não tem volta.
3. **Mostre, logo abaixo e de forma compacta, o detalhe técnico** — método, caminho
   e dados enviados — para quem quiser conferir.
4. **Espere o "sim".** Uma resposta vaga não serve, e nem um pedido genérico:
   "integra aí" ou "cadastra os tutores" não autoriza as alterações que ele implica.

**Cada confirmação vale só para o que foi mostrado.** Uma alteração nova precisa de
um "sim" novo, mesmo que parecida com a anterior. Nunca trate um "sim" antigo como
autorização geral.

**Num lote, uma confirmação cobre exatamente a lista mostrada** — nem um item a
mais. Mostre todos os itens antes. Se um deles falhar no meio, pare, conte o que
foi feito e o que não foi, e pergunte como seguir; não continue por conta própria.

A loja de uma alteração é **sempre uma só**, e sempre confirmada pelo usuário — os
quatro cabeçalhos errados gravam na loja errada, e ninguém percebe na hora.

## Quota

Cada API Key tem quota diária por grupo, na ordem de mil requisições por dia. O
consumo está no *My Dashboard* do Developer Portal, e um `429` é quota estourada,
não erro de credencial.

Consequência prática ao explorar ou testar: **não varra endpoints em laço**.
Poucas dezenas de chamadas queimam o orçamento do dia, e a partir daí tudo
responde `429` — um erro que ninguém liga ao laço que rodou horas antes.
