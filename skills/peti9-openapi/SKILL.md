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
  "qual endpoint da open api faz Y", ou quando
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

## Configuração inicial (uma vez)

A skill precisa de três variáveis de ambiente. Elas vêm de **duas origens
diferentes**, e é isso que confunde: a API Key é sua, do parceiro; o usuário e a
senha são do **cliente**, criados por ele na plataforma dele.

### 1. Conta no Developer Portal → a API Key

1. Acesse `https://devportal.peti9.com` e clique em **Register**. O acesso é
   restrito a parceiros certificados.
2. Confirme o e-mail e faça **Sign In**.
3. Abra **My Dashboard** no menu do topo. O bloco **API Key** mostra a chave em
   texto puro — é o valor de `PETI9_OPENAPI_API_KEY`.
4. A chave nasce **sem assinatura em nenhum grupo**. Para liberar, envie e-mail a
   `comercial@peti9.com` com o nome da empresa, o e-mail do cadastro e a descrição
   da integração. A liberação é por grupo, depois de NDA e contrato de parceria.

Enquanto a liberação não sai, toda chamada devolve `403 {"message":"Forbidden"}`.

### 2. Usuário de integração → na plataforma do cliente

A Peti9 **não** cria essa credencial: por LGPD o cliente é o controlador do dado e
o parceiro é operador. Quem executa o passo abaixo é o cliente, na plataforma
dele, e depois repassa a você.

Na plataforma Peti9, em **Configurações → Usuários → novo usuário**:

- **Nome** — identifique como integração, ex. `Integração <parceiro>`, para
  ninguém desativar achando que é gente.
- **Usuário** — o login. Aceita **apenas letras e números**: sem espaço, acento,
  ponto ou `@`. A restrição é do sistema, não só da tela.
- **Senha** e **Confirmar a senha** — mínimo 8 caracteres, com letras e números.
- **Grupo Usuário** — define a que telas/rotas o usuário tem acesso.
- **Ativo** — marcado.
- **Empresas** — marque **todas as empresas** cujos dados a integração vai ler ou
  escrever. É esse vínculo que decide quais valores de `empresa_id` funcionam
  depois; sem ele a API responde vazio, sem erro.

Não existe "tipo integração" nem perfil especial de API: é um usuário comum da
plataforma. Quem limita o alcance dele é o Grupo Usuário e as empresas marcadas.

### 3. O username da API não é o login

Ao salvar, a plataforma propaga o usuário para o Keycloak como
**`<login>@<tenant>`**. É esse valor composto que vai na variável — não o login
sozinho. Um usuário `integracao` no tenant `clinicaexemplo` autentica como
`integracao@clinicaexemplo`.

Se você não souber o tenant, ele está no token de qualquer sessão da plataforma
(claim `tenant_id`), ou basta perguntar ao cliente.

### 4. Exportar as variáveis

| Variável | Origem |
|---|---|
| `PETI9_OPENAPI_API_KEY` | *My Dashboard* do Developer Portal (passo 1) |
| `PETI9_OPENAPI_USERNAME` | `<login>@<tenant>` do usuário criado (passos 2 e 3) |
| `PETI9_OPENAPI_PASSWORD` | a senha definida no passo 2 |

Guarde num arquivo fora de qualquer repositório, só seu:

```bash
cat > ~/.peti9-openapi.env <<'EOF'
export PETI9_OPENAPI_API_KEY='...'
export PETI9_OPENAPI_USERNAME='...'
export PETI9_OPENAPI_PASSWORD='...'
EOF
chmod 600 ~/.peti9-openapi.env
```

E carregue no shell, acrescentando ao `~/.zshrc` (ou `~/.bashrc`):

```bash
[ -f ~/.peti9-openapi.env ] && . ~/.peti9-openapi.env
```

Abra um terminal novo e confirme sem imprimir nada:

```bash
for v in PETI9_OPENAPI_API_KEY PETI9_OPENAPI_USERNAME PETI9_OPENAPI_PASSWORD; do
  eval "val=\$$v"; [ -n "$val" ] && echo "$v: ok" || echo "$v: FALTANDO"
done
```

Três regras que não se negociam: **não** commite o arquivo, **não** ponha os
valores num `.env` de projeto que vá para o git, e **não** cole a senha no chat.

## Se alguma variável faltar

Pare e peça ao usuário que configure, apontando o passo acima que resolve. Nunca
invente valor, nunca peça para colar no chat.

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
liberado àquele grupo — é liberação comercial pendente, não erro de código. Logo
depois de uma liberação nova, a resposta ainda pode vir `Forbidden` por cerca de
um minuto, até a mudança propagar.

## Passo 2 — chamar um endpoint

Quatro headers, e os dois de empresa são tão obrigatórios quanto o Bearer:

```bash
curl -s -H "x-api-key: $PETI9_OPENAPI_API_KEY" \
     -H "Authorization: Bearer $TOKEN" \
     -H "empresa_id: 1" \
     -H "empresa_cadastro: 1" \
     "https://openapi.peti9.com/items/item/paginar/lista?pagina=0&limit=2&orderBy=nome"
```

- **`empresa_id`** — a empresa do contexto da chamada (a unidade que está operando).
- **`empresa_cadastro`** — a empresa em que o dado está cadastrado. Costuma ser o
  mesmo valor, mas em rede com matriz e filiais **não é**, e trocar um pelo outro
  devolve lista vazia sem erro nenhum.

**Descubra esses valores sozinho, não peça ao usuário.** Ele quase nunca sabe o
número da empresa, e perguntar transforma uma pergunta simples num interrogatório
técnico. O procedimento é sempre o mesmo:

1. Gere o token (passo 1) e leia o `user_id` do payload do JWT.
2. Chame `GET /companies/empresa/{userId}/empresas-login`.
3. Se vier **uma** empresa, use os valores dela e siga em frente, sem perguntar
   nada. Apenas diga de qual unidade é o dado na resposta.
4. Se vierem **várias**, aí sim pergunte — mas pergunte pelo **nome** da unidade,
   nunca pelo número, e faça a tradução você mesmo.
5. Se vier `[]`, explique que o usuário de integração não tem nenhuma unidade
   vinculada e que isso se resolve em Configurações → Usuários, na caixa
   "Empresas".

**Não adivinhe esses dois valores: pergunte à API.** O `user_id` está no payload
do JWT, e `GET /companies/empresa/{userId}/empresas-login` devolve as empresas
vinculadas com os dois números já prontos:

```json
[{"codigo":12,"nome":"Clinica Exemplo LTDA","empresaCadastro":12,"empresaFinanceiro":12,"empresaPreco":12}]
```

`empresa_id` é o `codigo`; `empresa_cadastro` é o `empresaCadastro`. Em rede com
matriz e filiais os dois divergem, e é justamente aí que o palpite erra.

Uma resposta `[]` significa que o usuário de integração não tem empresa vinculada
— é a caixa "Empresas" do cadastro, no cliente, não erro de chamada. E confira de
onde veio o `user_id`: em shell, `$UID` é variável reservada do próprio sistema, e
usar ela por engano no lugar do id do token devolve `[]` sem erro nenhum.

### Diagnóstico

Cada falha tem uma assinatura própria; casar a assinatura com a causa poupa a maior
parte da investigação.

| Resposta | Causa |
|---|---|
| `401` + `[{"user":"Usuário ou senha inválido!"...}]` | Credencial errada mesmo. |
| `401` sem corpo, numa chamada de negócio | Bearer ausente ou token vencido (24h). Refaça o passo 1. |
| `403` + `{"message":"Missing Authentication Token"}` | Apesar do nome, **não é autenticação**: a URL não casou com nenhuma rota. Base path do grupo errado, caminho digitado errado, ou `/` na credencial. |
| `403` + `{"message":"Forbidden"}` | Outra coisa: a sua API Key não tem acesso liberado àquele grupo. É liberação comercial, pedida por grupo. |
| `500` + `CustomerNotFoundException: HEADER CUSTOMER IS A MANDATORY INFO` | Faltou `empresa_id` / `empresa_cadastro`. |
| `500` + `MissingServletRequestParameterException: ... 'x' ... is not present` | Falta um parâmetro de query obrigatório — o nome está na própria mensagem. |
| `429` | Quota da API Key estourada. |

Duas consequências práticas: **erro de contrato volta `500`, não `400`** — não
trate 500 como instabilidade antes de ler o campo `tech` do corpo, que traz a
exception. E os dois `403` **nunca** são usuário/senha: leia o corpo para separar
rota errada (`Missing Authentication Token`) de API Key não autorizada
(`Forbidden`).

Os parâmetros obrigatórios variam por endpoint e os nomes são em português
(`pagina`, `limit`, `orderBy`). Quando faltar um, a mensagem do `500` diz qual —
adicione e repita, em vez de adivinhar o contrato inteiro.

## Os grupos de API

Há dois tipos. **Grupo de domínio** reúne endpoints por assunto. **Grupo de
parceiro** é um recorte curado dos mesmos endpoints sob um base path próprio.
Existem outros grupos no portal, de uso interno ou de parceiros específicos, que
não estão aqui: o acesso a eles passa pelo comercial.

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
     -H "empresa_id: $EMP" -H "empresa_cadastro: $ECAD" \
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
`GET /tutor/cliente/aniversariantes`.

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

Nem tudo da plataforma é exposto nas APIs públicas. Se o que o usuário pede não
existe em nenhum grupo da tabela, **diga isso** — não force um endpoint parecido
de outro grupo, que só devolve 404 e faz o usuário achar que errou a chamada. Há
grupos internos e recortes de parceiro fora desta documentação, e o caminho para
eles é comercial, não técnico. A resposta certa é: esse dado não está nas APIs
públicas, fale com o comercial da Peti9.

## Escrita em produção

`GET` é livre: leia à vontade.

`POST`, `PUT`, `PATCH` e `DELETE` batem em dado de cliente real e a API não desfaz.
Criar pedido, cancelar ou reagendar atendimento, excluir assinatura, confirmar
anexo de exame, finalizar venda — tudo irreversível por ela.

Antes de qualquer um deles: **mostre método, URL completa, headers de empresa e
payload ao usuário e espere um "sim" explícito**. Um pedido genérico ("integra aí",
"cadastra os tutores") não autoriza as escritas que ele implica. Confirme também
**qual empresa** vai receber a escrita — `empresa_id` errado grava na unidade
errada e ninguém percebe na hora.

## Quota

Cada API Key tem quota diária por grupo, na ordem de mil requisições por dia. O
consumo está no *My Dashboard* do Developer Portal, e um `429` é quota estourada,
não erro de credencial.

Consequência prática ao explorar ou testar: **não varra endpoints em laço**.
Poucas dezenas de chamadas queimam o orçamento do dia, e a partir daí tudo
responde `429` — um erro que ninguém liga ao laço que rodou horas antes.
