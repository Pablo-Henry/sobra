# Sobra — Especificação do Projeto

> **Saiba quanto sobra. Guarde o que importa.**
>
> Documento de especificação para o agente que vai construir a aplicação. Leia tudo antes de começar. Se uma regra estiver ambígua ou faltando, **pare e pergunte**; não invente.

**Arquivos de apoio:**
- Contrato da API (OpenAPI 3.1): [`docs/swagger/openapi.yaml`](docs/swagger/openapi.yaml)
- Plano de implementação em fases: [`docs/plano-implementacao.md`](docs/plano-implementacao.md)

---

## 1. Visão geral do produto

O **Sobra** é uma aplicação web para uma pessoa controlar as próprias finanças. Ela registra entradas e saídas de dinheiro, vê os totais do mês, acompanha quanto sobra e recebe a sugestão de guardar 30% do salário. Também pode definir uma meta de economia, se quiser, e ver gráficos que comparam meses e anos.

- **Público:** pessoa física que quer organizar o próprio dinheiro.
- **Ideia central:** "quanto entrou, quanto saiu e quanto sobrou neste mês, e se estou guardando o suficiente".
- **Isolamento:** cada usuário vê e altera **somente** os próprios dados.
- **Fora do escopo (não implementar):** lançamentos recorrentes, importação de extrato, várias moedas, recuperação de senha por e-mail, compartilhamento entre usuários.

### 1.1 Funcionalidades
1. Cadastro e login com autenticação JWT.
2. CRUD de **entradas** e **saídas**, exibidas em duas tabelas diferentes. Um **toggle** escolhe qual delas aparece.
3. Confirmação em pop-up antes de toda ação que altera dados.
4. Total de saídas do mês somado automaticamente a cada lançamento.
5. Três badges separadas: **Entradas** (verde), **Saídas** (vermelha) e **Sobra** (cor conforme a situação, ver RN04).
6. Sugestão fixa de guardar **30% do salário**. O usuário pode editar o salário quando quiser.
7. **Meta de economia** opcional, com barra ou anel de progresso.
8. Visão **mensal** (o que entrou e saiu no mês) e **anual** (como cada mês fechou).
9. **Gráficos de barras** comparando entradas e saídas entre meses e entre anos.
10. **Tema claro/escuro**. Começa igual ao do sistema operacional, e o usuário pode trocar por um botão.
11. Layout **mobile first**, responsivo para celular, tablet e desktop.

---

## 2. Decisões de projeto

| Tema | Decisão |
|---|---|
| Categorias | O sistema vem com categorias padrão (seed) e o usuário pode criar as suas |
| Meta | Separada dos 30%. Tem nome, valor-alvo e prazo opcional. Só uma meta ativa por usuário. O progresso é calculado automaticamente pelas sobras positivas (RN05) |
| Modelo de dados | Uma única tabela `Lancamento` com `tipo` (`ENTRADA`/`SAIDA`). Na interface aparecem duas tabelas separadas pelo toggle |
| Dinheiro | Guardado e trafegado em **centavos** (`Int`), para evitar erros de arredondamento de ponto flutuante |
| Idade | Guarda a **data de nascimento**. A idade é calculada na hora, então nunca fica desatualizada |
| Senha | 8 a 16 caracteres, com pelo menos 1 letra, 1 número e 1 caractere especial. Hash com bcrypt |
| Token JWT | Expira em 1h, fica no `localStorage` e é enviado por um `HttpInterceptor`. Uma resposta 401 faz logout automático |
| Gráficos | Chart.js com o tema do design system |
| Máscaras | ngx-mask |
| Modais e toasts | SweetAlert2 com o visual do design system |
| Confirmações | Toda ação que altera dados pede confirmação (seção 11) |

---

## 3. Stack (usar somente estas tecnologias)

| Camada | Tecnologia |
|---|---|
| Front-end | **Angular 22** (standalone components, signals, control flow `@if`/`@for`, Reactive Forms), TypeScript, HTML, CSS, **Bootstrap 5.3** personalizado |
| Animação | **GSAP** (core; plugins só se for realmente necessário) |
| Ícones | **Lucide** (`lucide-angular`) |
| Máscaras | `ngx-mask` |
| Gráficos | `chart.js`, usado direto em um componente Angular próprio, sem wrapper |
| Modais e alertas | `sweetalert2` |
| Back-end | **NestJS** + TypeScript |
| ORM / Banco | **Prisma ORM 7** + **PostgreSQL** |
| Autenticação | JWT com `@nestjs/jwt`, `@nestjs/passport`, `passport` e `passport-jwt` (Guards) |
| Hash de senha | `bcrypt` |
| Documentação | `@nestjs/swagger` (Swagger UI em `/api/docs`) |

**Dependências que vêm junto com a stack (permitidas):** `class-validator`, `class-transformer`, `@nestjs/config`, `@nestjs/throttler`, `@prisma/adapter-pg` e `pg` (o Prisma 7 exige um driver adapter).

**Proibido:** qualquer biblioteca fora desta lista. Se precisar de outra, pare e peça autorização.

---

## 4. Estrutura de pastas

```
financas/
├─ ESPECIFICACAO.md
├─ docs/
│  ├─ swagger/openapi.yaml
│  └─ plano-implementacao.md
├─ backend/                         # NestJS
│  ├─ .env.exemplo
│  ├─ prisma/
│  │  ├─ schema.prisma
│  │  └─ seed.ts
│  └─ src/
│     ├─ main.ts                    # prefixo /api, ValidationPipe global, Swagger, CORS
│     ├─ app.module.ts
│     ├─ prisma/                    # PrismaService
│     ├─ comum/
│     │  ├─ mensagens.ts            # todas as mensagens de erro e sucesso
│     │  ├─ decorators/             # @UsuarioAtual(), @Publica(), @SemHtml()
│     │  ├─ validadores/            # cpf.validador.ts
│     │  └─ filtros/                # filtro-excecoes.filter.ts (formato 8.2)
│     ├─ autenticacao/              # controller, service, jwt.strategy, jwt-auth.guard, DTOs
│     ├─ usuarios/
│     ├─ categorias/
│     ├─ lancamentos/
│     ├─ resumos/
│     └─ metas/
└─ frontend/                        # Angular
   └─ src/
      ├─ index.html                 # script inline do tema (9.4)
      ├─ styles/
      │  ├─ tokens.css
      │  ├─ tipografia.css
      │  └─ bootstrap-personalizado.css
      └─ app/
         ├─ nucleo/
         │  ├─ mensagens.ts
         │  ├─ guards/              # autenticado.guard.ts, visitante.guard.ts
         │  ├─ interceptors/        # token.interceptor.ts, erro.interceptor.ts
         │  └─ servicos/            # autenticacao, tema, confirmacao, notificacao, api
         ├─ compartilhado/
         │  ├─ componentes/         # grafico-barras, badge-valor, cartao-resumo, anel-progresso, alternador-tema, seletor-mes, estado-vazio
         │  ├─ validadores/         # cpf, nome, senha-forte, telefone, idade, senhas-iguais
         │  └─ pipes/               # moeda-brl.pipe.ts, idade.pipe.ts
         ├─ layout/                 # casca, barra-lateral, navegacao-inferior, cabecalho
         └─ paginas/
            ├─ entrar/
            ├─ cadastro/
            ├─ painel/
            ├─ relatorios/
            └─ perfil/
```

**Padrões de projeto aplicados:**
- **Camadas / MVC** no Nest: Controller (HTTP) → Service (regra de negócio) → Prisma (dados).
- **Singleton**: `PrismaService` e `TemaService` têm uma única instância no app.
- **Observer**: signals e RxJS atualizam os totais e os gráficos sempre que um lançamento muda.
- **Facade**: `ConfirmacaoService` e `NotificacaoService` escondem o SweetAlert2 dos componentes.
- **Strategy (simples)**: uma função `definirSituacaoSobra()` decide a cor e o texto do badge de sobra.

---

## 5. Modelo de dados (Prisma 7)

```prisma
enum TipoLancamento {
  ENTRADA
  SAIDA
}

model Usuario {
  id                String   @id @default(uuid())
  nome              String   @db.VarChar(100)
  dataNascimento    DateTime @db.Date                 // a idade é calculada a partir daqui
  email             String   @unique @db.VarChar(150)  // sempre salvo em minúsculas
  telefone          String   @db.VarChar(11)           // só dígitos
  cpf               String   @unique @db.Char(11)      // só dígitos
  salarioEmCentavos Int
  senhaHash         String
  criadoEm          DateTime @default(now())
  atualizadoEm      DateTime @updatedAt
  lancamentos       Lancamento[]
  categorias        Categoria[]
  meta              Meta?
}

model Categoria {
  id          String         @id @default(uuid())
  nome        String         @db.VarChar(40)
  tipo        TipoLancamento
  icone       String         @db.VarChar(40)        // nome de um ícone Lucide
  usuarioId   String?                               // null = categoria padrão do sistema
  usuario     Usuario?       @relation(fields: [usuarioId], references: [id], onDelete: Cascade)
  lancamentos Lancamento[]

  @@unique([usuarioId, nome, tipo])
}

model Lancamento {
  id              String         @id @default(uuid())
  tipo            TipoLancamento
  descricao       String         @db.VarChar(100)
  valorEmCentavos Int                               // sempre > 0
  data            DateTime       @db.Date
  categoriaId     String
  categoria       Categoria      @relation(fields: [categoriaId], references: [id])
  usuarioId       String
  usuario         Usuario        @relation(fields: [usuarioId], references: [id], onDelete: Cascade)
  criadoEm        DateTime       @default(now())
  atualizadoEm    DateTime       @updatedAt

  @@index([usuarioId, tipo, data])
}

model Meta {
  id                  String    @id @default(uuid())
  nome                String    @db.VarChar(60)
  valorAlvoEmCentavos Int
  prazo               DateTime? @db.Date
  usuarioId           String    @unique
  usuario             Usuario   @relation(fields: [usuarioId], references: [id], onDelete: Cascade)
  criadoEm            DateTime  @default(now())
}
```

> Atenção: no PostgreSQL, `@@unique([usuarioId, nome, tipo])` **não** impede duplicatas quando `usuarioId` é `null`, porque valores nulos não são comparados como iguais. O seed precisa ser idempotente: usar `upsert` ou checar antes de inserir.

### 5.1 Seed de categorias padrão (`usuarioId = null`)
| Tipo | Nome | Ícone Lucide |
|---|---|---|
| ENTRADA | Salário | `wallet` |
| ENTRADA | Freelance | `briefcase` |
| ENTRADA | Investimentos | `trending-up` |
| ENTRADA | Outros | `plus-circle` |
| SAIDA | Moradia | `house` |
| SAIDA | Alimentação | `utensils` |
| SAIDA | Transporte | `bus` |
| SAIDA | Saúde | `heart-pulse` |
| SAIDA | Lazer | `party-popper` |
| SAIDA | Educação | `graduation-cap` |
| SAIDA | Contas | `receipt` |
| SAIDA | Outros | `circle-ellipsis` |

---

## 6. Regras de negócio

- **RN01 – Totais automáticos:** os totais do mês **nunca** são guardados no banco. São calculados com `aggregate`/`groupBy` do Prisma a cada consulta. Assim, toda saída criada, editada ou excluída já entra na soma do mês. O front recarrega o resumo depois de cada alteração.
- **RN02 – Sobra:** `sobra = totalEntradas − totalSaidas` do mês.
- **RN03 – Sugestão de 30%:** `valorSugerido = salarioEmCentavos × 0,30`, arredondado para o centavo inteiro mais próximo. O percentual é sempre 30% e o usuário não pode alterá-lo. `percentualGuardado = sobra ÷ salário × 100`, com 1 casa decimal.
- **RN04 – Situação da sobra (cor do badge):**

  | Condição | `situacao` | Cor | Texto |
  |---|---|---|---|
  | `sobra < 0` | `NEGATIVA` | `--cor-perigo` (vermelho) | "Você gastou mais do que ganhou" |
  | `0 ≤ sobra < valorSugerido` | `ABAIXO_DA_META` | `--cor-alerta` (âmbar) | "Abaixo dos 30%" |
  | `sobra ≥ valorSugerido` | `META_ATINGIDA` | `--cor-marinho` | "Meta de 30% atingida" |

- **RN05 – Meta (opcional):** `valorAcumulado` é a soma das sobras **positivas** dos meses desde o mês de criação da meta até o mês atual, incluindo o mês atual. Meses com sobra negativa contam como 0. `progresso = valorAcumulado ÷ valorAlvo × 100`, limitado entre 0 e 100. Quando chega a 100, o card mostra um estado de comemoração e o toast "Meta atingida". Se a meta for substituída por `PUT /metas`, a data de criação recomeça.
- **RN06 – Visão anual:** a API devolve sempre os 12 meses do ano pedido, com entradas, saídas e sobra de cada um (o "fechamento" do mês). Meses sem lançamento vêm com 0. Também devolve o total do ano.
- **RN07 – Isolamento:** toda consulta filtra por `usuarioId` vindo do **JWT**, nunca do corpo ou da URL. Um recurso de outro usuário responde **404**, não 403, para não revelar que ele existe.
- **RN08 – Categorias:**
  - O usuário só exclui as próprias categorias. Tentar excluir uma padrão ou de outro usuário responde 404.
  - Se a categoria tiver lançamentos, responde **409** `CATEGORIA_EM_USO`.
  - Um lançamento só aceita categoria padrão ou do próprio usuário, e do **mesmo tipo** do lançamento. Caso contrário, responde 400.
- **RN09 – Data do lançamento:** de 01/01/2000 até 31/12 do ano seguinte ao atual.
- **RN10 – E-mail e CPF únicos:** se já existirem, o cadastro responde 409 com o código específico (`EMAIL_JA_CADASTRADO` ou `CPF_JA_CADASTRADO`).
- **RN11 – E-mail e CPF não mudam:** o perfil não permite alterar e-mail nem CPF.

---

## 7. Validações (no front **e** no back; nunca confiar só no cliente)

| Campo | Regra | Máscara / armazenamento |
|---|---|---|
| Nome | `^[A-Za-zÀ-ÖØ-öø-ÿ ]{3,100}$`: só letras (com acento) e espaços. `trim` e espaços duplos reduzidos a um | — |
| Data de nascimento | Data válida, não futura. Idade calculada entre **16 e 120** anos | `<input type="date">`. Salva como `Date`. A API devolve também `idade` |
| E-mail | `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`, até 150 caracteres | Salvo em minúsculas, sem espaços |
| Senha | `^(?=.*[A-Za-z])(?=.*\d)(?=.*[^A-Za-z\d\s])\S{8,16}$`: 8 a 16 caracteres, pelo menos 1 letra, 1 número e 1 caractere especial, sem espaços | Hash `bcrypt` (custo 10) |
| Confirmação de senha | Igual à senha (só no front) | Não é enviada para a API |
| Telefone | 10 dígitos (fixo) ou 11 (celular, com o 3º dígito igual a 9). DDD de 11 a 99 | Exibe `(11) 91234-5678` ou `(11) 1234-5678`. Salva só os dígitos |
| CPF | 11 dígitos. Rejeita sequências repetidas (`111.111.111-11`). Confere os 2 dígitos verificadores | Exibe `000.000.000-00`. Salva só os dígitos |
| Salário | Inteiro em centavos, de 1 a 100.000.000 (R$ 1.000.000,00) | Campo de moeda BRL. Salvo em centavos |
| Descrição | 2 a 100 caracteres. Rejeita `< > { } \` e o texto `javascript:` (sem diferenciar maiúsculas) | `trim` |
| Valor | Inteiro em centavos, de 1 a 1.000.000.000 (R$ 10.000.000,00) | Campo de moeda BRL. Salvo em centavos |
| Data do lançamento | RN09 | `<input type="date">` |
| Nome da categoria / meta | 2 a 40 caracteres (categoria) ou 2 a 60 (meta). Mesma regra de caracteres proibidos da descrição | `trim` |
| Valor-alvo da meta | Inteiro em centavos, de 1 a 1.000.000.000 | Moeda BRL |
| Prazo da meta | Opcional. Se informado, precisa ser uma data futura | `<input type="date">` |

### 7.1 Algoritmo do CPF (função pura, com testes)
```
validarCpf(cpf):
  1. Remova tudo que não for dígito. Se não tiver 11 dígitos, é inválido.
  2. Se todos os dígitos forem iguais, é inválido.
  3. Primeiro dígito: some d[0..8] multiplicados por 10, 9, ..., 2.
     resto = (soma × 10) % 11. Se resto for 10, use 0. Precisa ser igual a d[9].
  4. Segundo dígito: some d[0..9] multiplicados por 11, 10, ..., 2.
     resto = (soma × 10) % 11. Se resto for 10, use 0. Precisa ser igual a d[10].
```
A **mesma lógica** deve existir no back (decorator `@CpfValido()` com `class-validator`) e no front (validador de Reactive Forms).

### 7.2 Máscaras (ngx-mask)
- CPF: `mask="000.000.000-00"`.
- Telefone: `mask="(00) 0000-0000||(00) 00000-0000"`. O ngx-mask passa para a segunda máscara quando há 11 dígitos, mostrando `(11) 1234-5678` ou `(11) 91234-5678`.
- Moeda: `mask="separator.2"`, `thousandSeparator="."`, `decimalMarker=","`, `prefix="R$ "`. Antes de enviar, o valor é convertido para centavos.
- `dropSpecialCharacters: true` para o `FormControl` guardar só os dígitos.

### 7.3 Requisitos de senha visíveis
Abaixo do campo de senha, mostrar uma lista que marca ✓ (verde) ou ✗ (cinza) em tempo real para cada regra:
- 8 a 16 caracteres
- Pelo menos 1 letra
- Pelo menos 1 número
- Pelo menos 1 caractere especial (ex.: `! @ # $ %`)

A lista usa `aria-live="polite"`.

---

## 8. Segurança e proteção contra injeção

- **SQL injection:** usar apenas a API tipada do Prisma, que já parametriza as consultas. É **proibido** usar `$queryRawUnsafe` e `$executeRawUnsafe`.
- **Validação global:** `new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true })`.
- **HTML injection / XSS:**
  - Back: os campos de texto livre (nome, descrição, nomes de categoria e meta) usam `@Transform(trim)` e o decorator `@SemHtml()`, que rejeita `< > { } \` e `javascript:` com o código `CONTEUDO_NAO_PERMITIDO`.
  - Front: exibir dados **somente** com interpolação `{{ }}`, que o Angular escapa automaticamente. É **proibido** usar `[innerHTML]` com dados do usuário e `bypassSecurityTrust*`.
  - SweetAlert2: usar `text`, nunca `html`, sempre que houver dado do usuário na mensagem.
- **Mass assignment:** os DTOs nunca aceitam `id`, `usuarioId`, `senhaHash`, `criadoEm` nem `atualizadoEm`.
- **Senhas:** hash com `bcrypt` (custo 10). A senha nunca aparece em logs nem em respostas. O `select` do Prisma nunca devolve `senhaHash`.
- **JWT:**
  - Payload `{ sub: usuarioId, email }`, expira em 1h, assinado com `JWT_SEGREDO`.
  - `JwtStrategy` (passport-jwt) lê o token do header `Authorization: Bearer`.
  - O `JwtAuthGuard` é **global** (`APP_GUARD`). As rotas públicas usam `@Publica()`.
  - O decorator `@UsuarioAtual()` entrega o `usuarioId` para os controllers.
- **Força bruta:** `@nestjs/throttler` com no máximo 5 requisições por minuto por IP em `POST /auth/entrar` e `POST /auth/cadastro`. Ao estourar, responde 429.
- **Mensagens genéricas:** o login inválido responde sempre "E-mail ou senha inválidos", sem dizer qual dos dois está errado. O filtro global de exceções nunca devolve stack trace.
- **Segredos:** `DATABASE_URL`, `JWT_SEGREDO`, `JWT_EXPIRACAO=1h`, `ORIGEM_FRONTEND=http://localhost:4200` e `PORTA=3000` ficam no `.env`. Versionar apenas o `.env.exemplo`.
- **CORS:** liberado apenas para `ORIGEM_FRONTEND`.

---

## 9. API REST

Prefixo global `/api`. Os valores de dinheiro trafegam como **centavos inteiros**. As datas usam o formato `YYYY-MM-DD`. O contrato completo, com corpos, exemplos e erros, está em [`docs/swagger/openapi.yaml`](docs/swagger/openapi.yaml).

| Método | Rota | Auth | Descrição | Status |
|---|---|---|---|---|
| POST | `/auth/cadastro` | Pública | Cria o usuário | 201 / 400 / 409 / 429 |
| POST | `/auth/entrar` | Pública | Devolve `{ tokenAcesso, expiraEm, usuario }` | 200 / 400 / 401 / 429 |
| GET | `/usuarios/eu` | JWT | Perfil (com `idade` calculada, sem senha) | 200 / 401 |
| PATCH | `/usuarios/eu` | JWT | Edita nome, data de nascimento, telefone e salário | 200 / 400 / 401 |
| GET | `/categorias?tipo=` | JWT | Categorias padrão e as do usuário | 200 / 401 |
| POST | `/categorias` | JWT | Cria categoria própria | 201 / 400 / 401 / 409 |
| DELETE | `/categorias/:id` | JWT | Exclui categoria própria | 204 / 401 / 404 / 409 |
| GET | `/lancamentos?tipo=&mes=&ano=&pagina=` | JWT | Lista paginada (20 por página), da data mais recente para a mais antiga | 200 / 400 / 401 |
| POST | `/lancamentos` | JWT | Cria lançamento | 201 / 400 / 401 |
| PATCH | `/lancamentos/:id` | JWT | Edita lançamento | 200 / 400 / 401 / 404 |
| DELETE | `/lancamentos/:id` | JWT | Exclui lançamento | 204 / 401 / 404 |
| GET | `/resumos/mensal?mes=&ano=` | JWT | `{ mes, ano, totalEntradas, totalSaidas, sobra, salario, valorSugerido, percentualGuardado, situacao }` | 200 / 400 / 401 |
| GET | `/resumos/mensal/categorias?mes=&ano=&tipo=` | JWT | Total por categoria no mês (gráfico de saídas por categoria) | 200 / 400 / 401 |
| GET | `/resumos/anual?ano=` | JWT | 12 meses `{ mes, entradas, saidas, sobra }` e `total` | 200 / 400 / 401 |
| GET | `/resumos/comparativo-anos?quantidade=3` | JWT | Totais por ano dos últimos N anos (1 a 5) | 200 / 400 / 401 |
| GET | `/metas` | JWT | Meta atual com `valorAcumulado` e `progresso`, ou `null` | 200 / 401 |
| PUT | `/metas` | JWT | Cria ou substitui a meta | 200 / 400 / 401 |
| DELETE | `/metas` | JWT | Remove a meta | 204 / 401 / 404 |

**Categoria na resposta:** `{ id, nome, tipo, icone, padrao }`. O campo `padrao` é `true` nas categorias do sistema; o front usa isso para esconder o botão de excluir.

**Regras dos parâmetros:** `mes` de 1 a 12. `ano` de 2000 até o ano atual + 1. `pagina` ≥ 1, padrão 1. `tipo` aceita `ENTRADA` ou `SAIDA`.

**Resposta paginada:**
```json
{ "itens": [], "pagina": 1, "tamanhoPagina": 20, "totalItens": 0, "totalPaginas": 0 }
```

### 9.1 Swagger
- **Contrato primeiro:** `docs/swagger/openapi.yaml` é a fonte da verdade. O código precisa bater com ele.
- **Swagger vivo:** `@nestjs/swagger` gera o Swagger UI em `/api/docs` e o JSON em `/api/docs-json`.
  - `DocumentBuilder` com título "Sobra API", a descrição e `addBearerAuth()`.
  - Todo controller usa `@ApiTags` (`Autenticação`, `Usuários`, `Categorias`, `Lançamentos`, `Resumos`, `Metas`), `@ApiBearerAuth()` nas rotas protegidas, `@ApiOperation({ summary })` em português e um `@ApiResponse` para **cada** status possível, usando o DTO de erro da seção 9.2.
  - Todo DTO usa `@ApiProperty` com `description` e `example` em português.
- **Conferência:** o QA compara `/api/docs-json` com o `openapi.yaml`. Qualquer diferença é um defeito.

### 9.2 Formato padrão de erro
Um filtro global (`FiltroExcecoes`) transforma **qualquer** erro neste formato:
```json
{
  "statusCode": 400,
  "codigo": "VALIDACAO_FALHOU",
  "mensagem": "Alguns campos estão inválidos. Confira e tente de novo.",
  "erros": [{ "campo": "cpf", "mensagem": "CPF inválido. Confira os números digitados." }],
  "caminho": "/api/auth/cadastro",
  "dataHora": "2026-10-21T14:00:00.000Z"
}
```
- `erros` só aparece quando o erro é de validação (400 `VALIDACAO_FALHOU`).
- A `exceptionFactory` do `ValidationPipe` monta `erros[]` a partir das mensagens dos decorators, que vêm de `mensagens.ts`.
- Os erros do Prisma são traduzidos: `P2002` (unique) vira 409 com o código certo e `P2025` (não encontrado) vira 404.
- No front, o `ErroInterceptor` mostra `erros[].mensagem` embaixo de cada campo e `mensagem` em um toast de erro. Um 401 faz logout e redireciona para `/entrar`.

---

## 10. Mensagens de erro e de sucesso

Todas as mensagens ficam em constantes: `backend/src/comum/mensagens.ts` e `frontend/src/app/nucleo/mensagens.ts`. É **proibido** escrever texto solto espalhado pelo código. As variáveis entre chaves (`{valor}`) são preenchidas na hora, e os valores são formatados em BRL.

### 10.1 Validação de campos (front e back usam o mesmo texto)
| Campo | Mensagem |
|---|---|
| Qualquer campo obrigatório | "Preencha este campo." |
| Nome | "Use apenas letras e espaços (de 3 a 100 caracteres)." |
| Data de nascimento | "Informe uma data válida. Você precisa ter entre 16 e 120 anos." |
| E-mail | "Informe um e-mail válido, como nome@exemplo.com." |
| Senha | "A senha deve ter de 8 a 16 caracteres, com pelo menos 1 letra, 1 número e 1 caractere especial." |
| Confirmação de senha | "As senhas não são iguais." |
| Telefone | "Informe um telefone com DDD (10 ou 11 dígitos)." |
| CPF | "CPF inválido. Confira os números digitados." |
| Salário | "Informe um valor maior que zero." |
| Descrição | "Use de 2 a 100 caracteres, sem símbolos como < > { }." |
| Valor | "Informe um valor maior que zero." |
| Data do lançamento | "Escolha uma data entre 01/01/2000 e 31/12 do próximo ano." |
| Categoria | "Escolha uma categoria." |
| Categoria de outro tipo | "Esta categoria não pertence a este tipo de lançamento." |
| Nome da meta | "Dê um nome à meta (de 2 a 60 caracteres)." |
| Prazo da meta | "O prazo precisa ser uma data futura." |

### 10.2 Erros da API
| Status | `codigo` | Mensagem |
|---|---|---|
| 400 | `VALIDACAO_FALHOU` | "Alguns campos estão inválidos. Confira e tente de novo." |
| 400 | `CONTEUDO_NAO_PERMITIDO` | "O texto contém caracteres não permitidos." |
| 401 | `CREDENCIAIS_INVALIDAS` | "E-mail ou senha inválidos." |
| 401 | `SESSAO_EXPIRADA` | "Sua sessão expirou. Entre novamente." |
| 404 | `NAO_ENCONTRADO` | "Não encontramos este registro. Ele pode ter sido excluído." |
| 409 | `EMAIL_JA_CADASTRADO` | "Este e-mail já está cadastrado. Tente entrar na sua conta." |
| 409 | `CPF_JA_CADASTRADO` | "Este CPF já está cadastrado." |
| 409 | `CATEGORIA_JA_EXISTE` | "Você já tem uma categoria com este nome." |
| 409 | `CATEGORIA_EM_USO` | "Esta categoria tem lançamentos. Mova ou exclua os lançamentos antes." |
| 429 | `MUITAS_TENTATIVAS` | "Muitas tentativas. Aguarde 1 minuto e tente novamente." |
| 500 | `ERRO_INTERNO` | "Algo deu errado do nosso lado. Tente novamente em instantes." |
| (sem rede, só no front) | `SEM_CONEXAO` | "Não foi possível conectar ao servidor. Verifique sua internet." |

### 10.3 Sucesso (toast verde, some sozinho depois de 4s)
| Ação | Mensagem |
|---|---|
| Cadastro | "Conta criada! Bem-vindo(a), {primeiroNome}." |
| Login | "Que bom te ver, {primeiroNome}!" |
| Sair | "Você saiu da sua conta." |
| Criar entrada | "Entrada de {valor} registrada." |
| Criar saída | "Saída de {valor} registrada. Total de saídas do mês: {totalSaidas}." |
| Editar lançamento | "Lançamento atualizado." |
| Excluir lançamento | "Lançamento excluído." |
| Atualizar perfil | "Dados atualizados." |
| Alterar salário | "Salário atualizado. Sua sugestão de guardar agora é {valor30}." |
| Salvar meta | "Meta \"{nome}\" salva. Bora guardar!" |
| Meta atingida | "Parabéns! Você atingiu a meta \"{nome}\"." |
| Remover meta | "Meta removida." |
| Criar categoria | "Categoria \"{nome}\" criada." |
| Excluir categoria | "Categoria excluída." |

### 10.4 Alertas informativos (não são erros; aparecem no painel)
- Sobra negativa: "Atenção: neste mês você gastou {valor} a mais do que ganhou."
- Sobra abaixo de 30%: "Faltam {valor} para você guardar os 30% deste mês."

---

## 11. Modais de confirmação

O `ConfirmacaoService` envolve o **SweetAlert2** e expõe `confirmar(opcoes): Promise<boolean>`. Os componentes nunca chamam `Swal` direto.

- O modal tem título, mensagem (`text`), botão "Cancelar" (secundário) e um botão de ação com o verbo da ação.
- Ações destrutivas usam o botão em `--cor-perigo` e mostram "Esta ação não pode ser desfeita."
- O visual vem de `customClass` + `buttonsStyling: false`: botões em pílula, raio de 16px, Afacad no título e Poppins no texto, cores dos tokens. Assim o modal acompanha o tema claro/escuro.
- O foco começa no botão "Cancelar" nas ações destrutivas. Esc fecha o modal.

| Ação | Título | Mensagem | Botão de ação |
|---|---|---|---|
| Criar lançamento | "Registrar {entrada/saída}?" | "{descricao} · {valor} em {categoria} no dia {data}." | "Registrar" |
| Editar lançamento | "Salvar alterações?" | "O lançamento \"{descricao}\" será atualizado." | "Salvar alterações" |
| Excluir lançamento | "Excluir {entrada/saída}?" | "\"{descricao}\" de {valor} será removido. Esta ação não pode ser desfeita." | "Excluir" |
| Alterar salário | "Atualizar salário?" | "Novo salário: {valor}. Sua sugestão de guardar passa a ser {valor30} por mês." | "Atualizar" |
| Atualizar perfil | "Salvar seus dados?" | "Suas informações de perfil serão atualizadas." | "Salvar" |
| Salvar meta | "Salvar meta?" | "{nome}: {valorAlvo}{ até {prazo}}." | "Salvar meta" |
| Substituir meta | "Substituir meta atual?" | "A meta \"{nomeAtual}\" será trocada e o progresso recomeça." | "Substituir" |
| Remover meta | "Remover meta?" | "A meta \"{nome}\" será removida. Esta ação não pode ser desfeita." | "Remover" |
| Criar categoria | "Criar categoria?" | "\"{nome}\" será adicionada às suas {entradas/saídas}." | "Criar" |
| Excluir categoria | "Excluir categoria?" | "\"{nome}\" será removida. Esta ação não pode ser desfeita." | "Excluir" |
| Sair | "Sair da conta?" | "Você precisará entrar de novo para ver suas finanças." | "Sair" |
| Fechar formulário alterado | "Descartar alterações?" | "As informações que você preencheu serão perdidas." | "Descartar" |

O `NotificacaoService` usa o modo toast do SweetAlert2: canto superior direito no desktop, topo centralizado no mobile, 4s de duração e pausa ao passar o mouse.

---

## 12. Design system

**Personalidade:** produto SaaS limpo e confiante. Superfícies claras e arejadas, um verde vivo como cor de ação e um azul quase preto como âncora. Cards brancos com sombra bem suave, botões em formato pílula e barras verticais arredondadas.

### 12.1 Tokens de cor (`styles/tokens.css`)
Os tokens são definidos em `:root, [data-bs-theme="light"]` e `[data-bs-theme="dark"]`.

| Token | Claro | Escuro | Uso |
|---|---|---|---|
| `--cor-marinho` | `#041119` | `#041119` | Marca, barra lateral, barras dos gráficos, badge de sobra |
| `--cor-primaria` | `#2cac5c` | `#34c46b` | Botões, destaques, badge de entradas |
| `--cor-primaria-hover` | `#249a51` | `#2cac5c` | Hover dos botões primários |
| `--cor-primaria-suave` | `#d7efe0` | `#123826` | Trilho das barras, fundos de ícone |
| `--cor-fundo` | `#fbfcff` | `#041119` | Fundo da página |
| `--cor-superficie` | `#ffffff` | `#0b1d27` | Cards e modais |
| `--cor-superficie-suave` | `#f5fbf8` | `#102631` | Seções alternadas, hover de linhas |
| `--cor-texto` | `#041119` | `#f5fbf8` | Texto principal |
| `--cor-texto-suave` | `#5f6b72` | `#9fb3ad` | Legendas e rótulos |
| `--cor-borda` | `#e5ece8` | `#1c3440` | Divisórias e inputs |
| `--cor-perigo` | `#e8452c` | `#ff6b55` | Badge de saídas, exclusão, sobra negativa |
| `--cor-alerta` | `#eba51b` | `#f5b93a` | Sobra abaixo de 30% |
| `--sombra-suave` | `0 8px 24px rgb(4 17 25 / .06)` | `0 8px 24px rgb(0 0 0 / .35)` | Cards |
| `--sombra-elevada` | `0 16px 40px rgb(4 17 25 / .12)` | `0 16px 40px rgb(0 0 0 / .5)` | Modais, botão flutuante |

No modo escuro, os elementos marinho sobre fundo marinho (barra lateral, badge de sobra) usam `--cor-superficie-suave` com borda `--cor-borda`, para manter o contraste.

**Bootstrap personalizado:** remapear `--bs-primary`, `--bs-body-bg`, `--bs-body-color`, `--bs-border-color`, `--bs-border-radius`, `--bs-font-sans-serif` etc. para os tokens. O visual padrão do Bootstrap **não pode** aparecer: sem botão azul e sem o foco azul padrão.

### 12.2 Tipografia (Google Fonts: Afacad e Poppins)
| Nível | Fonte | Peso | Desktop | Mobile |
|---|---|---|---|---|
| H1 | Afacad | 500 | 64px | `clamp(2.5rem, 6vw, 4rem)` |
| H2 | Afacad | 500 | 48px | `clamp(2rem, 4.5vw, 3rem)` |
| H3 | Afacad | 700 | 24px | 22px |
| H4 | Afacad | 400 | 20px | 18px |
| H5 | Afacad | 400 | 16px | 16px |
| Corpo | Poppins | 400 | 16px | 15px |
| Rótulos / legendas | Poppins | 500 | 12px (maiúsculas, `letter-spacing: .06em`) | 12px |
| Valores monetários | Afacad | 500 | `font-variant-numeric: tabular-nums` | — |

### 12.3 Forma, espaçamento e componentes
- **Raios:** cards 16px, inputs 12px, botões e badges `999px` (pílula).
- **Espaçamento:** base 4px (4, 8, 12, 16, 24, 32, 48, 64).
- **Botão primário:** pílula verde preenchida com texto branco. No hover, leve `translateY(-1px)` e `--sombra-suave`.
- **Botão secundário:** pílula com contorno de 1px em `--cor-marinho` (no escuro, `--cor-texto`) e fundo transparente.
- **Badge:** pílula com contorno fino e um ponto colorido de 8px à esquerda. As cores seguem o status (entrada, saída, sobra).
- **Cartão de resumo:** card com ícone em círculo, rótulo em maiúsculas, valor grande em Afacad e o badge.
- **Toggle segmentado:** trilho em pílula `--cor-superficie-suave`. A opção ativa fica com fundo `--cor-superficie` e `--sombra-suave` e desliza com GSAP. Acessibilidade: `role="tablist"`, `role="tab"` e `aria-selected`.
- **Gráficos de barras (Chart.js):** barras verticais em pílula (`borderRadius: 999`, `borderSkipped: false`), com trilho `--cor-primaria-suave` desenhado como um dataset de fundo de altura máxima e preenchimento marinho. A barra em destaque (mês atual) fica verde. O tooltip é uma pílula marinha com texto branco. Os eixos usam Poppins 12px em `--cor-texto-suave`, sem linhas de grade verticais. As cores são lidas dos tokens com `getComputedStyle`, e o gráfico é atualizado quando o tema muda. Comparações entrada × saída usam verde (entradas) e marinho (saídas).
- **Barras de progresso:** 6px de altura, arredondadas, com a cor do status.
- **Anel de progresso (meta):** SVG próprio com o percentual no centro, trilho `--cor-primaria-suave` e arco verde.
- **Ícone em círculo:** ícone Lucide branco dentro de um círculo verde de 48px.
- **Inputs:** fundo `--cor-superficie`, borda `--cor-borda`, raio de 12px, altura mínima de 48px. No foco, borda verde e anel de 3px em `--cor-primaria-suave`. No erro, borda `--cor-perigo` e mensagem abaixo com ícone `circle-alert`.

### 12.4 Tema claro/escuro
- No primeiro acesso, o tema segue `prefers-color-scheme` do sistema operacional.
- O botão `AlternadorTema` (ícones Lucide `sun` e `moon`, com `aria-label` "Ativar modo escuro" ou "Ativar modo claro") troca o tema, aplica `data-bs-theme` no `<html>` e salva a escolha no `localStorage` (chave `sobra-tema`). Depois de uma escolha manual, ela vale mais que a do sistema.
- Um script inline no `<head>` do `index.html` aplica o tema **antes** do Angular carregar, para não piscar a tela clara.
- O `TemaService` expõe um signal `temaAtual`, que os gráficos observam para se redesenhar.
- A troca de tema anima as cores com `transition: background-color .3s, color .3s`.

### 12.5 Movimento (GSAP)
- **Entrada da página:** os cards aparecem em sequência (`from` com `y: 16, opacity: 0`, `stagger: .06`, `ease: "power2.out"`).
- **Números:** os valores dos cartões de resumo contam até o valor final (cerca de 0,8s).
- **Anel da meta:** o traço é desenhado animando `stroke-dashoffset`.
- **Toggle Entradas/Saídas:** o indicador desliza, e a tabela que sai faz um fade para dar lugar à que entra.
- **Gráficos e modais:** usam a animação nativa do Chart.js e do SweetAlert2. O GSAP **não** anima esses dois.
- **Regras:** usar `gsap.matchMedia()` para respeitar `prefers-reduced-motion`. Animar só `transform` e `opacity`. Criar as animações dentro de `gsap.context()` e chamar `revert()` no `ngOnDestroy`.

---

## 13. Telas

Mobile first. Breakpoints do Bootstrap: `md` 768px (tablet) e `lg` 992px (desktop).

**Casca autenticada (`layout/`):**
- **Mobile:** cabeçalho com o logo "Sobra" e o botão de tema. **Navegação inferior fixa** com Painel (`layout-dashboard`), Relatórios (`chart-column`) e Perfil (`user`).
- **≥ lg:** **barra lateral marinho** fixa à esquerda (como a faixa escura das referências), com logo, navegação, botão de tema e "Sair" no rodapé.

**Logo:** um ponto verde de 10px seguido da palavra "Sobra" em Afacad 700, na cor `--cor-texto`.

### 13.1 Entrar (`/entrar`, guard de visitante)
- Card central com e-mail, senha (botão de mostrar/ocultar com `eye`/`eye-off`), botão "Entrar" e o link "Ainda não tem conta? Cadastre-se".
- **≥ lg:** a tela se divide em duas. À esquerda, um painel marinho com o slogan e uma ilustração de barras em pílula animadas com GSAP. À direita, o formulário.
- O botão mostra um spinner e fica desabilitado enquanto envia.

### 13.2 Cadastro (`/cadastro`, guard de visitante)
- Formulário em **2 passos** com indicador de progresso ("Passo 1 de 2"):
  - **Passo 1, "Seus dados":** nome, data de nascimento, e-mail, senha e confirmação de senha, com a lista de requisitos da seção 7.3.
  - **Passo 2, "Contato e renda":** telefone, CPF e salário mensal. Um texto de ajuda mostra "Sugerimos guardar 30%: {valor}", atualizado enquanto o usuário digita.
- O botão "Continuar" valida o passo 1 antes de avançar. "Voltar" mantém os dados preenchidos.
- Os erros aparecem abaixo de cada campo quando ele perde o foco (`blur`) ou ao enviar.
- Ao concluir, o usuário entra automaticamente e vai para `/painel`.

### 13.3 Painel (`/painel`)
1. **Seletor de mês/ano** com setas `chevron-left`/`chevron-right` e o rótulo "Outubro de 2026". Começa no mês atual.
2. **Três cartões de resumo** (1 coluna no mobile, 3 a partir de `md`):
   - **Entradas:** badge verde e total do mês.
   - **Saídas:** badge vermelha e total do mês, somado automaticamente.
   - **Sobra do mês:** badge conforme RN04 e o texto da situação.
3. **Card "Guarde 30%":** valor sugerido, quanto já sobrou e barra de progresso (`sobra ÷ valorSugerido`). Mostra o alerta informativo da seção 10.4 quando for o caso.
4. **Card da meta:** anel de progresso, nome, quanto já acumulou, valor-alvo e prazo. Botões "Editar" e "Remover". Sem meta, mostra o estado vazio "Defina uma meta (opcional)" com o botão "Criar meta", que abre um modal com o formulário.
5. **Toggle segmentado Entradas | Saídas**, que alterna as duas tabelas:
   - **≥ md:** tabela com Data, Descrição, Categoria (ícone + nome), Valor e Ações.
   - **Mobile:** cada lançamento vira um card com descrição, categoria, data e valor, e as ações em um menu.
   - Ações: editar (`pencil`) e excluir (`trash-2`), cada uma com `aria-label`.
   - Paginação "Carregar mais" quando houver mais de 20 itens.
6. **Botão "Novo lançamento"** (`plus`): no topo à direita no desktop e flutuante no canto inferior no mobile. Abre um modal com tipo (já marcado conforme o toggle ativo), descrição, valor, data (padrão: hoje) e categoria (filtrada pelo tipo).

### 13.4 Relatórios (`/relatorios`)
- Seletor de ano e o toggle **Mensal | Anual**.
- **Mensal:**
  - Gráfico de barras agrupadas: **entradas × saídas** de cada mês do ano escolhido. O mês atual fica em destaque.
  - Seletor de mês e gráfico de barras horizontais com as **saídas por categoria** do mês.
- **Anual:**
  - **Tabela de fechamento:** 12 linhas (mês, entradas, saídas, sobra com badge de situação) e uma linha de total do ano.
  - Gráfico **comparativo entre anos:** entradas × saídas dos últimos 3 anos.
  - Gráfico **saídas mês a mês: ano atual × ano anterior**.
- Cada gráfico tem título, legenda e uma tabela equivalente escondida visualmente, para leitores de tela.

### 13.5 Perfil (`/perfil`)
- Formulário: nome, data de nascimento (mostra "Você tem {idade} anos"), telefone e **salário mensal**. Ao mudar o salário, aparece a prévia "Nova sugestão de guardar: {valor30}".
- Dados só para leitura: e-mail e CPF mascarado (`***.456.789-**`).
- Seção **Minhas categorias:** lista das categorias próprias por tipo, um formulário para criar (nome, tipo, ícone de uma lista curta de opções Lucide) e um botão para excluir.
- Botão **"Sair"** (`log-out`).

### 13.6 Estados obrigatórios
Toda tela e todo bloco de dados precisa ter:
- **Carregando:** skeleton com shimmer suave (sem spinner de tela cheia).
- **Vazio:** ícone, texto amigável e botão de ação. Exemplo: "Nenhuma saída em outubro. Que tal registrar a primeira?"
- **Erro:** mensagem e o botão "Tentar novamente".
- **Sucesso:** toast da seção 10.3.

### 13.7 Rotas e guards
| Rota | Guard | Componente |
|---|---|---|
| `/` | — | redireciona para `/painel` |
| `/entrar` | `visitanteGuard` (logado vai para `/painel`) | Entrar |
| `/cadastro` | `visitanteGuard` | Cadastro |
| `/painel` | `autenticadoGuard` | Painel |
| `/relatorios` | `autenticadoGuard` | Relatórios |
| `/perfil` | `autenticadoGuard` | Perfil |
| `**` | — | redireciona para `/painel` |

As páginas usam lazy loading (`loadComponent`). O `<title>` de cada página segue o formato "Sobra · {Página}".

---

## 14. Acessibilidade e UX

- Contraste mínimo **AA** nos dois temas.
- Foco visível em tudo: anel de 2px em `--cor-primaria` com 2px de afastamento (`:focus-visible`).
- Navegação completa por teclado. Os modais prendem o foco dentro deles e fecham com Esc.
- Todo input tem `<label>` visível. Os erros são ligados ao campo com `aria-describedby` e `aria-invalid="true"`.
- Usar `inputmode="numeric"` em CPF e telefone, `inputmode="decimal"` em valores e `autocomplete` correto (`email`, `new-password`, `current-password`, `name`, `tel`, `bday`).
- Botões só com ícone recebem `aria-label`. Os ícones decorativos recebem `aria-hidden="true"`.
- Os `<canvas>` dos gráficos recebem `role="img"` e um `aria-label` com o resumo dos dados, além da tabela equivalente da seção 13.4.
- Os valores nunca dependem só da cor: os badges sempre têm texto ("Entradas", "Saídas", a situação da sobra).
- Áreas de toque de no mínimo 44×44px no mobile. A navegação inferior respeita `env(safe-area-inset-bottom)`.
- Sem rolagem horizontal em 320px de largura.

---

## 15. Regras de código

- **Identificadores em português** (variáveis, funções, classes, propriedades, arquivos). Inglês só no que pertence à linguagem ou ao framework (`ngOnInit`, `@Controller`, `findMany`).
- **Nunca usar o operador ternário** (`a ? b : c`). Usar sempre `if/else`.
- Funções pequenas com uma responsabilidade só e nomes descritivos.
- Comentários de **no máximo 2 linhas**, em linguagem simples, só em trechos importantes.
- Código simples e explícito, sem abstrações prematuras.
- Back: Controller só recebe, valida e repassa. Toda regra de negócio fica no Service.
- Front: componentes de página orquestram e componentes compartilhados só exibem (`input()`/`output()`). As chamadas HTTP ficam nos serviços.
- TypeScript em modo `strict`, sem `any`.

---

## 16. Ordem de construção

Executar com a skill `/subagent-driven-development`, usando os agentes padrão, **em série** quando um passo depende do outro. O detalhamento de cada tarefa está em [`docs/plano-implementacao.md`](docs/plano-implementacao.md).

1. **`databaser`:** schema Prisma 7, migration inicial e seed idempotente das categorias.
2. **`dev-backend`:** base do Nest (config, PrismaService, `mensagens.ts`, filtro global de erros, ValidationPipe, Swagger), validador de CPF, `@SemHtml()`, autenticação (bcrypt, Passport JWT, guard global, throttler).
3. **`dev-backend`:** módulos de usuários, categorias, lançamentos, resumos e metas.
4. **`qa`:** testes da API (isolamento, validações, injeção, formato de erro, Swagger × `openapi.yaml`).
5. **`dev-frontend`:** tokens, tipografia, Bootstrap personalizado, tema, casca de layout, serviços do núcleo (autenticação, interceptors, confirmação, notificação), validadores, pipes e configuração do ngx-mask.
6. **`dev-frontend`:** telas Entrar e Cadastro, depois Painel, Relatórios (Chart.js) e Perfil, com as animações GSAP.
7. **`qa`:** testes de ponta a ponta, responsividade (375, 768 e 1280px), tema e acessibilidade.
8. **`documentation-owner`:** revisar o Swagger e escrever o `README.md` (pré-requisitos, `.env`, migrations, seed, como rodar back e front, como rodar os testes).

Os problemas Críticos e Importantes encontrados pelo `qa` voltam para o agente dono da área antes de seguir.

---

## 17. Critérios de aceite

### 17.1 Testes automatizados do back-end (Jest)
- Validador de CPF: CPFs válidos, dígito verificador errado, sequências repetidas, menos de 11 dígitos.
- Validador de senha: aceita `Abc123!@` e rejeita `abcdefgh`, `12345678`, `Abc12345` (sem caractere especial) e senhas com 7 ou 17 caracteres.
- `definirSituacaoSobra()`: os 3 casos e os limites (sobra = 0 e sobra = valorSugerido).
- Cálculo do resumo mensal e anual (12 meses, meses sem lançamento com 0).
- Progresso da meta (sobras negativas contam como 0, limite de 100%).

### 17.2 Testes e2e da API
- O usuário A **não** consegue ler, editar nem excluir um lançamento, uma categoria ou a meta do usuário B (recebe 404).
- Um payload com `<script>alert(1)</script>` na descrição recebe 400 `CONTEUDO_NAO_PERMITIDO`.
- Um campo extra no body (ex.: `usuarioId`) recebe 400.
- A 6ª tentativa de login em 1 minuto recebe 429.
- Uma rota protegida sem token ou com token expirado recebe 401.
- E-mail ou CPF duplicado recebe 409 com o código certo.
- Todo erro segue o formato da seção 9.2.

### 17.3 Testes do front-end
- Validadores (CPF, telefone, nome, senha forte, idade a partir da data de nascimento, senhas iguais).
- Máscaras: o input mostra `123.456.789-09` e o `FormControl` guarda `12345678909`.
- Pipe `moedaBrl`: `123456` vira `R$ 1.234,56`.

### 17.4 Roteiro manual
1. Cadastrar com dados inválidos e ver cada mensagem da seção 10.1. Depois cadastrar com dados válidos e cair no painel.
2. Criar 3 saídas e conferir que o total do mês soma sozinho e que a sobra muda de cor conforme RN04.
3. Alternar Entradas/Saídas no toggle.
4. Editar e excluir lançamentos passando pelos modais. Cancelar um modal não deve alterar nada.
5. Trocar de mês e ver os totais mudarem.
6. Em Relatórios, ver as visões mensal e anual e os gráficos.
7. Alterar o salário no perfil e conferir os novos 30% no painel.
8. Criar uma meta e acompanhar o progresso. Substituir e remover a meta.
9. Trocar o tema e recarregar a página: a escolha continua. Limpar o `localStorage`, colocar o sistema operacional em modo escuro e conferir que o app abre escuro.
10. Testar nas larguras 375, 768 e 1280px, sem rolagem horizontal.
11. Navegar o app inteiro só pelo teclado.
12. Abrir `/api/docs`, autorizar com o token e executar uma rota protegida.
