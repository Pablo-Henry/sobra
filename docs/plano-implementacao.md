# Sobra — Plano de Implementação

> **Para agentes:** SUB-SKILL OBRIGATÓRIA: use `/subagent-driven-development` (recomendado) ou `/executing-plans` para executar este plano tarefa por tarefa. Os passos usam checkbox (`- [ ]`) para acompanhamento.

**Objetivo:** construir o Sobra, uma aplicação web de controle de entradas e saídas com autenticação JWT, resumos mensais e anuais, meta de economia, gráficos e tema claro/escuro.

**Arquitetura:** monorepo com duas aplicações independentes. `backend/` é uma API REST em NestJS, organizada em módulos por domínio (Controller → Service → Prisma), com PostgreSQL via Prisma 7. `frontend/` é um SPA Angular 22 com standalone components, signals e Bootstrap personalizado, que consome a API por HTTP com um token Bearer.

**Stack:** Angular 22, TypeScript, Bootstrap 5.3, GSAP, lucide-angular, ngx-mask, chart.js, sweetalert2 · NestJS, Prisma ORM 7, PostgreSQL, @nestjs/jwt, @nestjs/passport, passport-jwt, bcrypt, @nestjs/swagger, @nestjs/throttler, class-validator, class-transformer.

**Spec:** [`ESPECIFICACAO.md`](../ESPECIFICACAO.md) e o contrato [`docs/swagger/openapi.yaml`](swagger/openapi.yaml). Leia os dois antes de começar. As referências "§N" apontam para seções da especificação.

## Restrições globais

Toda tarefa segue estas regras, mesmo quando não aparecem repetidas nela.

- Identificadores em **português**. Inglês só no que pertence à linguagem ou ao framework.
- **Nunca** usar o operador ternário (`a ? b : c`). Usar sempre `if/else`.
- Comentários de no máximo 2 linhas, em linguagem simples, só em trechos importantes.
- Nenhuma biblioteca fora da lista da §3. Se precisar de outra, pare e pergunte.
- Dinheiro sempre em **centavos inteiros** (`Int`). Datas no formato `YYYY-MM-DD`.
- Todos os textos para o usuário vêm de `mensagens.ts` (§10). Nada de string solta.
- Toda consulta ao banco filtra por `usuarioId` vindo do JWT. Recurso de outro usuário responde 404 (§6 RN07).
- Proibido: `$queryRawUnsafe`, `$executeRawUnsafe`, `[innerHTML]` com dados do usuário, `bypassSecurityTrust*`, opção `html` do SweetAlert2 com dados do usuário.
- TypeScript em modo `strict`, sem `any`.
- Commits pequenos em Conventional Commits (`feat:`, `test:`, `chore:`, `docs:`, `style:`).

## Sobre o nível de detalhe

- **Back-end, regras puras e serviços do núcleo do front** (tarefas 1 a 13): o código está completo e é guiado por TDD.
- **Telas** (tarefas 14 a 18): o plano traz a estrutura, os IDs e classes, os comportamentos obrigatórios e o código das partes não óbvias. O visual completo vem das §12 e §13 da especificação, aplicando as skills `/frontend-design`, `/web-design-guidelines`, `/impecable-frontend` e as do GSAP.

---

## Mapa de arquivos

```
financas/
├─ .gitignore
├─ backend/
│  ├─ .env.exemplo
│  ├─ prisma.config.ts
│  ├─ prisma/schema.prisma
│  ├─ prisma/seed.ts
│  ├─ src/main.ts
│  ├─ src/app.module.ts
│  ├─ src/configurar-app.ts                 # pipes, filtro, CORS, prefixo (usado pelo main e pelos testes e2e)
│  ├─ src/prisma/prisma.service.ts
│  ├─ src/prisma/prisma.module.ts
│  ├─ src/comum/mensagens.ts
│  ├─ src/comum/excecao-api.ts
│  ├─ src/comum/filtros/filtro-excecoes.filter.ts
│  ├─ src/comum/fabrica-erros-validacao.ts
│  ├─ src/comum/validadores/cpf.ts            # função pura validarCpf
│  ├─ src/comum/validadores/regras.ts         # regex de nome, senha, e-mail, telefone
│  ├─ src/comum/decorators/cpf-valido.decorator.ts
│  ├─ src/comum/decorators/sem-html.decorator.ts
│  ├─ src/comum/decorators/publica.decorator.ts
│  ├─ src/comum/decorators/usuario-atual.decorator.ts
│  ├─ src/comum/utilitarios/datas.ts          # calcularIdade, intervaloDoMes, intervaloDoAno
│  ├─ src/autenticacao/{autenticacao.module,autenticacao.controller,autenticacao.service,jwt.strategy,jwt-auth.guard}.ts
│  ├─ src/autenticacao/dto/{cadastro.dto,entrar.dto}.ts
│  ├─ src/usuarios/{usuarios.module,usuarios.controller,usuarios.service}.ts
│  ├─ src/usuarios/dto/atualizar-usuario.dto.ts
│  ├─ src/usuarios/usuario.mapeador.ts        # converte o model para a resposta (com idade, sem senha)
│  ├─ src/categorias/{categorias.module,categorias.controller,categorias.service}.ts
│  ├─ src/categorias/dto/criar-categoria.dto.ts
│  ├─ src/lancamentos/{lancamentos.module,lancamentos.controller,lancamentos.service}.ts
│  ├─ src/lancamentos/dto/{criar-lancamento.dto,atualizar-lancamento.dto,filtro-lancamentos.dto}.ts
│  ├─ src/resumos/{resumos.module,resumos.controller,resumos.service}.ts
│  ├─ src/resumos/calculos-resumo.ts          # funções puras
│  ├─ src/resumos/dto/{filtro-mes.dto,filtro-ano.dto,filtro-comparativo.dto}.ts
│  ├─ src/metas/{metas.module,metas.controller,metas.service}.ts
│  ├─ src/metas/dto/salvar-meta.dto.ts
│  └─ test/ (*.e2e-spec.ts, utilitarios-e2e.ts)
└─ frontend/
   ├─ src/index.html
   ├─ src/styles.css                          # importa os 3 arquivos de styles/
   ├─ src/styles/{tokens,tipografia,bootstrap-personalizado}.css
   ├─ src/environments/{environment,environment.development}.ts
   └─ src/app/
      ├─ app.config.ts, app.routes.ts, app.ts
      ├─ nucleo/mensagens.ts
      ├─ nucleo/modelos.ts                    # interfaces iguais aos schemas do openapi.yaml
      ├─ nucleo/servicos/{tema,autenticacao,confirmacao,notificacao,lancamentos,categorias,resumos,metas,usuarios}.service.ts
      ├─ nucleo/interceptors/{token,erro}.interceptor.ts
      ├─ nucleo/guards/{autenticado,visitante}.guard.ts
      ├─ compartilhado/validadores/{cpf,senha-forte,nome,telefone,idade,senhas-iguais}.validador.ts
      ├─ compartilhado/pipes/{moeda-brl,idade}.pipe.ts
      ├─ compartilhado/utilitarios/{dinheiro,situacao-sobra}.ts
      ├─ compartilhado/componentes/{alternador-tema,cartao-resumo,anel-progresso,seletor-mes,estado-vazio,grafico-barras,campo-erro,requisitos-senha,toggle-segmentado}/
      ├─ layout/casca/
      └─ paginas/{entrar,cadastro,painel,relatorios,perfil}/
```

---

## Fase 0 — Repositório

### Tarefa 0: Iniciar o repositório

**Arquivos:**
- Criar: `.gitignore`

- [ ] **Passo 1: Iniciar o git e o `.gitignore`**

```bash
cd financas
git init
```

`.gitignore`:
```
node_modules/
dist/
.angular/
coverage/
.env
backend/src/generated/
```

- [ ] **Passo 2: Commit**

```bash
git add .gitignore ESPECIFICACAO.md docs/
git commit -m "chore: inicia repositório com especificação e contrato da API"
```

---

## Fase 1 — Banco de dados (`databaser`)

### Tarefa 1: Projeto Nest, Prisma 7, schema e seed

**Arquivos:**
- Criar: `backend/` (scaffold do Nest), `backend/prisma.config.ts`, `backend/prisma/schema.prisma`, `backend/prisma/seed.ts`, `backend/.env.exemplo`, `backend/src/prisma/prisma.service.ts`, `backend/src/prisma/prisma.module.ts`

**Interfaces:**
- Produz: `PrismaService` (estende `PrismaClient`), injetável em qualquer módulo via `PrismaModule` global. Os tipos `TipoLancamento`, `Usuario`, `Categoria`, `Lancamento` e `Meta` são importados de `../generated/prisma/client`.

- [ ] **Passo 1: Criar o projeto e instalar as dependências**

```bash
npx @nestjs/cli new backend --package-manager npm --strict --skip-git
cd backend
npm i @prisma/client @prisma/adapter-pg pg @nestjs/config @nestjs/jwt @nestjs/passport passport passport-jwt bcrypt @nestjs/swagger @nestjs/throttler class-validator class-transformer
npm i -D prisma @types/pg @types/passport-jwt @types/bcrypt tsx
```

> Prisma 7: o gerador agora é `prisma-client`, com `output` obrigatório. A URL do banco fica em `prisma.config.ts`, e o `PrismaClient` exige um driver adapter. Se algum detalhe divergir da versão instalada, confira em https://www.prisma.io/docs e siga a documentação oficial.

- [ ] **Passo 2: `.env.exemplo`** (copie para `.env` e ajuste)

```
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/sobra?schema=public"
JWT_SEGREDO="troque-por-um-segredo-longo-e-aleatorio"
JWT_EXPIRACAO_SEGUNDOS=3600
ORIGEM_FRONTEND="http://localhost:4200"
PORTA=3000
```

- [ ] **Passo 3: `prisma.config.ts`**

```ts
import 'dotenv/config';
import { defineConfig, env } from 'prisma/config';

export default defineConfig({
  schema: 'prisma/schema.prisma',
  migrations: {
    path: 'prisma/migrations',
    seed: 'tsx prisma/seed.ts',
  },
  datasource: {
    url: env('DATABASE_URL'),
  },
});
```

> O `dotenv` já vem como dependência do `@nestjs/config`. Se o import não resolver, instale `dotenv`: é uma dependência implícita do Prisma 7, permitida.

- [ ] **Passo 4: `prisma/schema.prisma`**

```prisma
generator client {
  provider     = "prisma-client"
  output       = "../src/generated/prisma"
  moduleFormat = "cjs"
}

datasource db {
  provider = "postgresql"
}

enum TipoLancamento {
  ENTRADA
  SAIDA
}

model Usuario {
  id                String       @id @default(uuid())
  nome              String       @db.VarChar(100)
  dataNascimento    DateTime     @db.Date
  email             String       @unique @db.VarChar(150)
  telefone          String       @db.VarChar(11)
  cpf               String       @unique @db.Char(11)
  salarioEmCentavos Int
  senhaHash         String
  criadoEm          DateTime     @default(now())
  atualizadoEm      DateTime     @updatedAt
  lancamentos       Lancamento[]
  categorias        Categoria[]
  meta              Meta?
}

model Categoria {
  id          String         @id @default(uuid())
  nome        String         @db.VarChar(40)
  tipo        TipoLancamento
  icone       String         @db.VarChar(40)
  usuarioId   String?
  usuario     Usuario?       @relation(fields: [usuarioId], references: [id], onDelete: Cascade)
  lancamentos Lancamento[]

  @@unique([usuarioId, nome, tipo])
}

model Lancamento {
  id              String         @id @default(uuid())
  tipo            TipoLancamento
  descricao       String         @db.VarChar(100)
  valorEmCentavos Int
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

- [ ] **Passo 5: `prisma/seed.ts`** (idempotente: o `@@unique` não protege quando `usuarioId` é nulo)

```ts
import 'dotenv/config';
import { PrismaPg } from '@prisma/adapter-pg';
import { PrismaClient, TipoLancamento } from '../src/generated/prisma/client';

const adaptador = new PrismaPg({ connectionString: process.env.DATABASE_URL });
const prisma = new PrismaClient({ adapter: adaptador });

const categoriasPadrao = [
  { nome: 'Salário', tipo: TipoLancamento.ENTRADA, icone: 'wallet' },
  { nome: 'Freelance', tipo: TipoLancamento.ENTRADA, icone: 'briefcase' },
  { nome: 'Investimentos', tipo: TipoLancamento.ENTRADA, icone: 'trending-up' },
  { nome: 'Outros', tipo: TipoLancamento.ENTRADA, icone: 'plus-circle' },
  { nome: 'Moradia', tipo: TipoLancamento.SAIDA, icone: 'house' },
  { nome: 'Alimentação', tipo: TipoLancamento.SAIDA, icone: 'utensils' },
  { nome: 'Transporte', tipo: TipoLancamento.SAIDA, icone: 'bus' },
  { nome: 'Saúde', tipo: TipoLancamento.SAIDA, icone: 'heart-pulse' },
  { nome: 'Lazer', tipo: TipoLancamento.SAIDA, icone: 'party-popper' },
  { nome: 'Educação', tipo: TipoLancamento.SAIDA, icone: 'graduation-cap' },
  { nome: 'Contas', tipo: TipoLancamento.SAIDA, icone: 'receipt' },
  { nome: 'Outros', tipo: TipoLancamento.SAIDA, icone: 'circle-ellipsis' },
];

async function semear() {
  for (const categoria of categoriasPadrao) {
    // Só cria se ainda não existir (rodar o seed duas vezes não duplica)
    const existente = await prisma.categoria.findFirst({
      where: { nome: categoria.nome, tipo: categoria.tipo, usuarioId: null },
    });
    if (!existente) {
      await prisma.categoria.create({ data: categoria });
    }
  }
}

semear()
  .then(() => prisma.$disconnect())
  .catch(async (erro) => {
    console.error(erro);
    await prisma.$disconnect();
    process.exit(1);
  });
```

- [ ] **Passo 6: `src/prisma/prisma.service.ts` e `prisma.module.ts`**

```ts
import { Injectable, OnModuleDestroy } from '@nestjs/common';
import { PrismaPg } from '@prisma/adapter-pg';
import { PrismaClient } from '../generated/prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleDestroy {
  constructor() {
    super({ adapter: new PrismaPg({ connectionString: process.env.DATABASE_URL }) });
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

```ts
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({ providers: [PrismaService], exports: [PrismaService] })
export class PrismaModule {}
```

- [ ] **Passo 7: Criar a migration, gerar o client e rodar o seed**

```bash
npx prisma migrate dev --name inicial
npx prisma generate
npx prisma db seed
npx prisma db seed
```
Esperado: a migration é aplicada sem erro e o seed roda duas vezes. Conferir a quantidade de categorias:

```bash
npx prisma studio
```
Esperado: a tabela `Categoria` tem **12** linhas, não 24.

- [ ] **Passo 8: Commit**

```bash
git add backend
git commit -m "feat(banco): schema Prisma 7, migration inicial e seed de categorias"
```

---

## Fase 2 — Back-end base (`dev-backend`)

### Tarefa 2: Mensagens, formato de erro e configuração do app

**Arquivos:**
- Criar: `src/comum/mensagens.ts`, `src/comum/excecao-api.ts`, `src/comum/fabrica-erros-validacao.ts`, `src/comum/filtros/filtro-excecoes.filter.ts`, `src/configurar-app.ts`
- Modificar: `src/main.ts`, `src/app.module.ts`
- Teste: `src/comum/filtros/filtro-excecoes.filter.spec.ts`

**Interfaces:**
- Produz:
  - `MENSAGENS_ERRO: Record<CodigoErro, string>`, `MENSAGENS_CAMPO` (objeto com chave por campo)
  - `type CodigoErro`
  - `class ExcecaoApi extends HttpException { constructor(status: number, codigo: CodigoErro) }`
  - `fabricaErrosValidacao(erros: ValidationError[]): ExcecaoApi`
  - `FiltroExcecoes`
  - `configurarApp(app: INestApplication): void`

- [ ] **Passo 1: `mensagens.ts`** (os textos são exatamente os da §10)

```ts
export type CodigoErro =
  | 'VALIDACAO_FALHOU'
  | 'CONTEUDO_NAO_PERMITIDO'
  | 'CREDENCIAIS_INVALIDAS'
  | 'SESSAO_EXPIRADA'
  | 'NAO_ENCONTRADO'
  | 'EMAIL_JA_CADASTRADO'
  | 'CPF_JA_CADASTRADO'
  | 'CATEGORIA_JA_EXISTE'
  | 'CATEGORIA_EM_USO'
  | 'MUITAS_TENTATIVAS'
  | 'ERRO_INTERNO';

export const MENSAGENS_ERRO: Record<CodigoErro, string> = {
  VALIDACAO_FALHOU: 'Alguns campos estão inválidos. Confira e tente de novo.',
  CONTEUDO_NAO_PERMITIDO: 'O texto contém caracteres não permitidos.',
  CREDENCIAIS_INVALIDAS: 'E-mail ou senha inválidos.',
  SESSAO_EXPIRADA: 'Sua sessão expirou. Entre novamente.',
  NAO_ENCONTRADO: 'Não encontramos este registro. Ele pode ter sido excluído.',
  EMAIL_JA_CADASTRADO: 'Este e-mail já está cadastrado. Tente entrar na sua conta.',
  CPF_JA_CADASTRADO: 'Este CPF já está cadastrado.',
  CATEGORIA_JA_EXISTE: 'Você já tem uma categoria com este nome.',
  CATEGORIA_EM_USO: 'Esta categoria tem lançamentos. Mova ou exclua os lançamentos antes.',
  MUITAS_TENTATIVAS: 'Muitas tentativas. Aguarde 1 minuto e tente novamente.',
  ERRO_INTERNO: 'Algo deu errado do nosso lado. Tente novamente em instantes.',
};

export const MENSAGENS_CAMPO = {
  obrigatorio: 'Preencha este campo.',
  nome: 'Use apenas letras e espaços (de 3 a 100 caracteres).',
  dataNascimento: 'Informe uma data válida. Você precisa ter entre 16 e 120 anos.',
  email: 'Informe um e-mail válido, como nome@exemplo.com.',
  senha: 'A senha deve ter de 8 a 16 caracteres, com pelo menos 1 letra, 1 número e 1 caractere especial.',
  telefone: 'Informe um telefone com DDD (10 ou 11 dígitos).',
  cpf: 'CPF inválido. Confira os números digitados.',
  valorPositivo: 'Informe um valor maior que zero.',
  descricao: 'Use de 2 a 100 caracteres, sem símbolos como < > { }.',
  dataLancamento: 'Escolha uma data entre 01/01/2000 e 31/12 do próximo ano.',
  categoria: 'Escolha uma categoria.',
  categoriaOutroTipo: 'Esta categoria não pertence a este tipo de lançamento.',
  nomeCategoria: 'Use de 2 a 40 caracteres, sem símbolos como < > { }.',
  nomeMeta: 'Dê um nome à meta (de 2 a 60 caracteres).',
  prazoMeta: 'O prazo precisa ser uma data futura.',
  tipo: 'Escolha entrada ou saída.',
};
```

- [ ] **Passo 2: `excecao-api.ts`**

```ts
import { HttpException } from '@nestjs/common';
import { CodigoErro, MENSAGENS_ERRO } from './mensagens';

export interface ErroDeCampo {
  campo: string;
  mensagem: string;
}

// Erro padrão da API: sempre carrega um código e a mensagem amigável
export class ExcecaoApi extends HttpException {
  constructor(
    status: number,
    public readonly codigo: CodigoErro,
    public readonly erros?: ErroDeCampo[],
  ) {
    super(MENSAGENS_ERRO[codigo], status);
  }
}
```

- [ ] **Passo 3: Escrever o teste do filtro (que deve falhar)**

`src/comum/filtros/filtro-excecoes.filter.spec.ts`:
```ts
import { ArgumentsHost, NotFoundException } from '@nestjs/common';
import { ThrottlerException } from '@nestjs/throttler';
import { FiltroExcecoes } from './filtro-excecoes.filter';
import { ExcecaoApi } from '../excecao-api';

function criarHostFalso() {
  const resposta = { status: jest.fn().mockReturnThis(), json: jest.fn() };
  const host = {
    switchToHttp: () => ({
      getResponse: () => resposta,
      getRequest: () => ({ url: '/api/teste' }),
    }),
  } as unknown as ArgumentsHost;
  return { host, resposta };
}

describe('FiltroExcecoes', () => {
  const filtro = new FiltroExcecoes();

  it('formata ExcecaoApi com código e erros de campo', () => {
    const { host, resposta } = criarHostFalso();
    filtro.catch(new ExcecaoApi(400, 'VALIDACAO_FALHOU', [{ campo: 'cpf', mensagem: 'x' }]), host);
    expect(resposta.status).toHaveBeenCalledWith(400);
    const corpo = resposta.json.mock.calls[0][0];
    expect(corpo.codigo).toBe('VALIDACAO_FALHOU');
    expect(corpo.erros).toEqual([{ campo: 'cpf', mensagem: 'x' }]);
    expect(corpo.caminho).toBe('/api/teste');
    expect(typeof corpo.dataHora).toBe('string');
  });

  it('converte 404 genérico em NAO_ENCONTRADO', () => {
    const { host, resposta } = criarHostFalso();
    filtro.catch(new NotFoundException(), host);
    expect(resposta.json.mock.calls[0][0].codigo).toBe('NAO_ENCONTRADO');
  });

  it('converte throttler em MUITAS_TENTATIVAS', () => {
    const { host, resposta } = criarHostFalso();
    filtro.catch(new ThrottlerException(), host);
    expect(resposta.status).toHaveBeenCalledWith(429);
    expect(resposta.json.mock.calls[0][0].codigo).toBe('MUITAS_TENTATIVAS');
  });

  it('esconde detalhes de erros desconhecidos', () => {
    const { host, resposta } = criarHostFalso();
    filtro.catch(new Error('senha do banco: 123'), host);
    expect(resposta.status).toHaveBeenCalledWith(500);
    const corpo = resposta.json.mock.calls[0][0];
    expect(corpo.codigo).toBe('ERRO_INTERNO');
    expect(JSON.stringify(corpo)).not.toContain('senha do banco');
  });
});
```

- [ ] **Passo 4: Rodar e ver falhar**

Rodar: `npx jest src/comum/filtros`
Esperado: FALHA com "Cannot find module './filtro-excecoes.filter'".

- [ ] **Passo 5: Implementar o filtro**

```ts
import { ArgumentsHost, Catch, ExceptionFilter, HttpException, Logger } from '@nestjs/common';
import { CodigoErro, MENSAGENS_ERRO } from '../mensagens';
import { ErroDeCampo, ExcecaoApi } from '../excecao-api';

// Código usado quando o erro não veio da nossa ExcecaoApi
function codigoPadraoDoStatus(status: number): CodigoErro {
  if (status === 400) { return 'VALIDACAO_FALHOU'; }
  if (status === 401) { return 'SESSAO_EXPIRADA'; }
  if (status === 404) { return 'NAO_ENCONTRADO'; }
  if (status === 429) { return 'MUITAS_TENTATIVAS'; }
  return 'ERRO_INTERNO';
}

@Catch()
export class FiltroExcecoes implements ExceptionFilter {
  private readonly registro = new Logger('FiltroExcecoes');

  catch(excecao: unknown, host: ArgumentsHost) {
    const contexto = host.switchToHttp();
    const resposta = contexto.getResponse();
    const requisicao = contexto.getRequest();

    let status = 500;
    let codigo: CodigoErro = 'ERRO_INTERNO';
    let erros: ErroDeCampo[] | undefined;

    if (excecao instanceof ExcecaoApi) {
      status = excecao.getStatus();
      codigo = excecao.codigo;
      erros = excecao.erros;
    } else if (excecao instanceof HttpException) {
      status = excecao.getStatus();
      codigo = codigoPadraoDoStatus(status);
    } else {
      this.registro.error(excecao);
    }

    const corpo: Record<string, unknown> = {
      statusCode: status,
      codigo: codigo,
      mensagem: MENSAGENS_ERRO[codigo],
      caminho: requisicao.url,
      dataHora: new Date().toISOString(),
    };
    if (erros && erros.length > 0) {
      corpo.erros = erros;
    }
    resposta.status(status).json(corpo);
  }
}
```

- [ ] **Passo 6: Rodar e ver passar**

Rodar: `npx jest src/comum/filtros`
Esperado: PASSA (4 testes).

- [ ] **Passo 7: `fabrica-erros-validacao.ts`**

```ts
import { ValidationError } from 'class-validator';
import { ErroDeCampo, ExcecaoApi } from './excecao-api';

// Transforma os erros do class-validator no formato padrão (§9.2)
export function fabricaErrosValidacao(errosValidacao: ValidationError[]): ExcecaoApi {
  const erros: ErroDeCampo[] = [];
  let temHtml = false;

  for (const erro of errosValidacao) {
    const restricoes = erro.constraints || {};
    if (restricoes.semHtml) {
      temHtml = true;
    }
    const mensagens = Object.values(restricoes);
    if (mensagens.length > 0) {
      erros.push({ campo: erro.property, mensagem: mensagens[0] });
    }
  }

  if (temHtml) {
    return new ExcecaoApi(400, 'CONTEUDO_NAO_PERMITIDO', erros);
  }
  return new ExcecaoApi(400, 'VALIDACAO_FALHOU', erros);
}
```

> Em cada DTO, o decorator de mensagem mais importante deve vir **primeiro** na propriedade.

- [ ] **Passo 8: `configurar-app.ts`, `main.ts` e `app.module.ts`**

```ts
// src/configurar-app.ts
import { INestApplication, ValidationPipe } from '@nestjs/common';
import { FiltroExcecoes } from './comum/filtros/filtro-excecoes.filter';
import { fabricaErrosValidacao } from './comum/fabrica-erros-validacao';

export function configurarApp(app: INestApplication) {
  app.setGlobalPrefix('api');
  app.enableCors({ origin: process.env.ORIGEM_FRONTEND });
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
      exceptionFactory: fabricaErrosValidacao,
    }),
  );
  app.useGlobalFilters(new FiltroExcecoes());
}
```

```ts
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';
import { AppModule } from './app.module';
import { configurarApp } from './configurar-app';

async function iniciar() {
  const app = await NestFactory.create(AppModule);
  configurarApp(app);

  const configuracaoSwagger = new DocumentBuilder()
    .setTitle('Sobra API')
    .setDescription('API de controle de entradas e saídas. Valores em centavos inteiros.')
    .setVersion('1.0.0')
    .addBearerAuth()
    .build();
  const documento = SwaggerModule.createDocument(app, configuracaoSwagger);
  SwaggerModule.setup('api/docs', app, documento, { jsonDocumentUrl: 'api/docs-json' });

  await app.listen(Number(process.env.PORTA) || 3000);
}
iniciar();
```

```ts
// src/app.module.ts (os módulos de domínio entram nas próximas tarefas)
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { PrismaModule } from './prisma/prisma.module';

@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true }), PrismaModule],
})
export class AppModule {}
```

Remova os arquivos de exemplo do scaffold: `app.controller.ts`, `app.service.ts` e o spec deles.

- [ ] **Passo 9: Conferir que sobe**

Rodar: `npm run start:dev`, depois abrir `http://localhost:3000/api/docs`.
Esperado: o Swagger UI abre com o título "Sobra API".

- [ ] **Passo 10: Commit**

```bash
git add backend/src
git commit -m "feat(api): formato padrão de erros, mensagens centralizadas e Swagger"
```

### Tarefa 3: Validadores (CPF, senha, nome, telefone, sem HTML) e utilitários de data

**Arquivos:**
- Criar: `src/comum/validadores/cpf.ts`, `src/comum/validadores/regras.ts`, `src/comum/decorators/cpf-valido.decorator.ts`, `src/comum/decorators/sem-html.decorator.ts`, `src/comum/utilitarios/datas.ts`
- Teste: `src/comum/validadores/cpf.spec.ts`, `src/comum/validadores/regras.spec.ts`, `src/comum/utilitarios/datas.spec.ts`

**Interfaces:**
- Produz:
  - `validarCpf(cpf: string): boolean`
  - `REGEX_NOME`, `REGEX_SENHA`, `REGEX_EMAIL`, `REGEX_CARACTERES_PROIBIDOS`
  - `telefoneValido(digitos: string): boolean`
  - `@CpfValido()`, `@SemHtml()`
  - `calcularIdade(dataNascimento: Date, hoje: Date): number`
  - `intervaloDoMes(mes: number, ano: number): { inicio: Date; fim: Date }` (fim exclusivo)
  - `intervaloDoAno(ano: number): { inicio: Date; fim: Date }`
  - `paraDataIso(data: Date): string` (`YYYY-MM-DD`)

- [ ] **Passo 1: Escrever os testes (que devem falhar)**

```ts
// cpf.spec.ts
import { validarCpf } from './cpf';

describe('validarCpf', () => {
  it('aceita CPFs válidos com e sem máscara', () => {
    expect(validarCpf('52998224725')).toBe(true);
    expect(validarCpf('529.982.247-25')).toBe(true);
    expect(validarCpf('11144477735')).toBe(true);
  });
  it('rejeita dígito verificador errado', () => {
    expect(validarCpf('52998224724')).toBe(false);
    expect(validarCpf('11144477736')).toBe(false);
  });
  it('rejeita sequências repetidas', () => {
    expect(validarCpf('11111111111')).toBe(false);
    expect(validarCpf('00000000000')).toBe(false);
  });
  it('rejeita tamanho errado', () => {
    expect(validarCpf('5299822472')).toBe(false);
    expect(validarCpf('')).toBe(false);
  });
});
```

```ts
// regras.spec.ts
import { REGEX_CARACTERES_PROIBIDOS, REGEX_NOME, REGEX_SENHA, telefoneValido } from './regras';

describe('regras', () => {
  it('senha forte', () => {
    expect(REGEX_SENHA.test('Abc123!@')).toBe(true);
    expect(REGEX_SENHA.test('Sobra@2026')).toBe(true);
    expect(REGEX_SENHA.test('abcdefgh')).toBe(false);
    expect(REGEX_SENHA.test('12345678')).toBe(false);
    expect(REGEX_SENHA.test('Abc12345')).toBe(false);
    expect(REGEX_SENHA.test('Ab1!')).toBe(false);
    expect(REGEX_SENHA.test('Abc123!@Abc123!@x')).toBe(false);
    expect(REGEX_SENHA.test('Abc 123!')).toBe(false);
  });
  it('nome só com letras e espaços', () => {
    expect(REGEX_NOME.test('Maria Souza')).toBe(true);
    expect(REGEX_NOME.test('João Conceição')).toBe(true);
    expect(REGEX_NOME.test('Ma')).toBe(false);
    expect(REGEX_NOME.test('Maria2')).toBe(false);
    expect(REGEX_NOME.test('<b>Maria</b>')).toBe(false);
  });
  it('telefone fixo e celular', () => {
    expect(telefoneValido('1133334444')).toBe(true);
    expect(telefoneValido('11912345678')).toBe(true);
    expect(telefoneValido('11812345678')).toBe(false);
    expect(telefoneValido('0933334444')).toBe(false);
    expect(telefoneValido('113333444')).toBe(false);
  });
  it('detecta caracteres proibidos', () => {
    expect(REGEX_CARACTERES_PROIBIDOS.test('<script>')).toBe(true);
    expect(REGEX_CARACTERES_PROIBIDOS.test('JavaScript:alert(1)')).toBe(true);
    expect(REGEX_CARACTERES_PROIBIDOS.test('Mercado do mês')).toBe(false);
  });
});
```

```ts
// datas.spec.ts
import { calcularIdade, intervaloDoMes, paraDataIso } from './datas';

describe('datas', () => {
  it('calcula idade considerando se já fez aniversário', () => {
    const hoje = new Date(Date.UTC(2026, 9, 21));
    expect(calcularIdade(new Date(Date.UTC(1998, 3, 15)), hoje)).toBe(28);
    expect(calcularIdade(new Date(Date.UTC(1998, 11, 15)), hoje)).toBe(27);
    expect(calcularIdade(new Date(Date.UTC(1998, 9, 21)), hoje)).toBe(28);
  });
  it('intervalo do mês com fim exclusivo', () => {
    const intervalo = intervaloDoMes(12, 2026);
    expect(paraDataIso(intervalo.inicio)).toBe('2026-12-01');
    expect(paraDataIso(intervalo.fim)).toBe('2027-01-01');
  });
});
```

- [ ] **Passo 2: Rodar e ver falhar**

Rodar: `npx jest src/comum`
Esperado: FALHA com os módulos não encontrados.

- [ ] **Passo 3: Implementar**

```ts
// cpf.ts
function calcularDigitoVerificador(base: string, pesoInicial: number): number {
  let soma = 0;
  for (let indice = 0; indice < base.length; indice++) {
    soma = soma + Number(base[indice]) * (pesoInicial - indice);
  }
  const resto = (soma * 10) % 11;
  if (resto === 10) {
    return 0;
  }
  return resto;
}

// Confere os 2 dígitos finais do CPF usando a conta oficial (módulo 11)
export function validarCpf(cpf: string): boolean {
  const digitos = cpf.replace(/\D/g, '');
  if (digitos.length !== 11) {
    return false;
  }
  if (/^(\d)\1{10}$/.test(digitos)) {
    return false;
  }
  const primeiroDigito = calcularDigitoVerificador(digitos.substring(0, 9), 10);
  if (primeiroDigito !== Number(digitos[9])) {
    return false;
  }
  const segundoDigito = calcularDigitoVerificador(digitos.substring(0, 10), 11);
  return segundoDigito === Number(digitos[10]);
}
```

```ts
// regras.ts
export const REGEX_NOME = /^[A-Za-zÀ-ÖØ-öø-ÿ ]{3,100}$/;
export const REGEX_EMAIL = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
export const REGEX_SENHA = /^(?=.*[A-Za-z])(?=.*\d)(?=.*[^A-Za-z\d\s])\S{8,16}$/;
export const REGEX_CARACTERES_PROIBIDOS = /[<>{}\\]|javascript:/i;

// Fixo: 10 dígitos. Celular: 11 dígitos com o 3º igual a 9. DDD de 11 a 99.
export function telefoneValido(digitos: string): boolean {
  if (!/^\d{10,11}$/.test(digitos)) {
    return false;
  }
  const ddd = Number(digitos.substring(0, 2));
  if (ddd < 11) {
    return false;
  }
  if (digitos.length === 11 && digitos[2] !== '9') {
    return false;
  }
  return true;
}
```

```ts
// datas.ts
export function calcularIdade(dataNascimento: Date, hoje: Date): number {
  let idade = hoje.getUTCFullYear() - dataNascimento.getUTCFullYear();
  const mesAtual = hoje.getUTCMonth();
  const mesNascimento = dataNascimento.getUTCMonth();
  const aindaNaoFezAniversario =
    mesAtual < mesNascimento ||
    (mesAtual === mesNascimento && hoje.getUTCDate() < dataNascimento.getUTCDate());
  if (aindaNaoFezAniversario) {
    idade = idade - 1;
  }
  return idade;
}

export function intervaloDoMes(mes: number, ano: number) {
  return {
    inicio: new Date(Date.UTC(ano, mes - 1, 1)),
    fim: new Date(Date.UTC(ano, mes, 1)),
  };
}

export function intervaloDoAno(ano: number) {
  return {
    inicio: new Date(Date.UTC(ano, 0, 1)),
    fim: new Date(Date.UTC(ano + 1, 0, 1)),
  };
}

export function paraDataIso(data: Date): string {
  return data.toISOString().substring(0, 10);
}
```

```ts
// cpf-valido.decorator.ts
import { registerDecorator, ValidationOptions } from 'class-validator';
import { validarCpf } from '../validadores/cpf';

export function CpfValido(opcoes?: ValidationOptions) {
  return function (objeto: object, nomePropriedade: string) {
    registerDecorator({
      name: 'cpfValido',
      target: objeto.constructor,
      propertyName: nomePropriedade,
      options: opcoes,
      validator: {
        validate(valor: unknown) {
          return typeof valor === 'string' && validarCpf(valor);
        },
      },
    });
  };
}
```

```ts
// sem-html.decorator.ts
import { registerDecorator, ValidationOptions } from 'class-validator';
import { REGEX_CARACTERES_PROIBIDOS } from '../validadores/regras';

// Recusa textos com < > { } \ ou "javascript:" (proteção contra HTML injection)
export function SemHtml(opcoes?: ValidationOptions) {
  return function (objeto: object, nomePropriedade: string) {
    registerDecorator({
      name: 'semHtml',
      target: objeto.constructor,
      propertyName: nomePropriedade,
      options: opcoes,
      validator: {
        validate(valor: unknown) {
          if (typeof valor !== 'string') {
            return true;
          }
          return !REGEX_CARACTERES_PROIBIDOS.test(valor);
        },
      },
    });
  };
}
```

- [ ] **Passo 4: Rodar e ver passar**

Rodar: `npx jest src/comum`
Esperado: todos PASSAM.

- [ ] **Passo 5: Commit**

```bash
git add backend/src/comum
git commit -m "feat(api): validadores de CPF, senha forte, telefone e proteção contra HTML"
```

### Tarefa 4: Autenticação (cadastro, login, JWT global, throttler)

**Arquivos:**
- Criar: `src/comum/decorators/publica.decorator.ts`, `src/comum/decorators/usuario-atual.decorator.ts`, `src/autenticacao/*`, `src/usuarios/usuario.mapeador.ts`, `test/utilitarios-e2e.ts`
- Modificar: `src/app.module.ts`
- Teste: `test/autenticacao.e2e-spec.ts`

**Interfaces:**
- Consome: `ExcecaoApi`, `MENSAGENS_CAMPO`, `REGEX_*`, `@CpfValido`, `@SemHtml`, `calcularIdade`, `paraDataIso`, `configurarApp`
- Produz:
  - `@Publica()`
  - `@UsuarioAtual(): string` (o `usuarioId`)
  - `JwtAuthGuard` registrado como `APP_GUARD`
  - `mapearUsuario(usuario: Usuario): RespostaUsuario`
  - `interface RespostaUsuario { id, nome, dataNascimento: string, idade: number, email, telefone, cpf, salarioEmCentavos, criadoEm: string }`
  - `criarAppTeste(): Promise<INestApplication>` e `cadastrarUsuarioTeste(app, sufixo, cpf): Promise<{ token: string; id: string }>`

- [ ] **Passo 1: Decorators**

```ts
// publica.decorator.ts
import { SetMetadata } from '@nestjs/common';
export const CHAVE_ROTA_PUBLICA = 'rotaPublica';
export const Publica = () => SetMetadata(CHAVE_ROTA_PUBLICA, true);
```

```ts
// usuario-atual.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

// Entrega o id do usuário logado, que vem do token (nunca do body)
export const UsuarioAtual = createParamDecorator((_dado: unknown, contexto: ExecutionContext) => {
  const requisicao = contexto.switchToHttp().getRequest();
  return requisicao.user.usuarioId as string;
});
```

- [ ] **Passo 2: DTOs**

```ts
// dto/cadastro.dto.ts
import { ApiProperty } from '@nestjs/swagger';
import { Transform } from 'class-transformer';
import { IsDateString, IsInt, IsNotEmpty, Matches, Max, MaxLength, Min } from 'class-validator';
import { MENSAGENS_CAMPO } from '../../comum/mensagens';
import { REGEX_EMAIL, REGEX_NOME, REGEX_SENHA } from '../../comum/validadores/regras';
import { CpfValido } from '../../comum/decorators/cpf-valido.decorator';

function limparEspacos(valor: unknown) {
  if (typeof valor !== 'string') { return valor; }
  return valor.trim().replace(/\s+/g, ' ');
}

function somenteDigitos(valor: unknown) {
  if (typeof valor !== 'string') { return valor; }
  return valor.replace(/\D/g, '');
}

export class CadastroDto {
  @ApiProperty({ example: 'Maria Souza', description: 'Só letras e espaços' })
  @Transform(({ value }) => limparEspacos(value))
  @Matches(REGEX_NOME, { message: MENSAGENS_CAMPO.nome })
  nome: string;

  @ApiProperty({ example: '1998-04-15', description: 'Idade entre 16 e 120 anos' })
  @IsDateString({ strict: true }, { message: MENSAGENS_CAMPO.dataNascimento })
  dataNascimento: string;

  @ApiProperty({ example: 'maria@exemplo.com' })
  @Transform(({ value }) => {
    if (typeof value !== 'string') { return value; }
    return value.trim().toLowerCase();
  })
  @Matches(REGEX_EMAIL, { message: MENSAGENS_CAMPO.email })
  @MaxLength(150, { message: MENSAGENS_CAMPO.email })
  email: string;

  @ApiProperty({ example: 'Sobra@2026', description: '8 a 16 caracteres, letra, número e caractere especial' })
  @Matches(REGEX_SENHA, { message: MENSAGENS_CAMPO.senha })
  senha: string;

  @ApiProperty({ example: '11912345678', description: 'Só dígitos, com DDD' })
  @Transform(({ value }) => somenteDigitos(value))
  @IsNotEmpty({ message: MENSAGENS_CAMPO.telefone })
  telefone: string;

  @ApiProperty({ example: '52998224725', description: 'Só dígitos' })
  @Transform(({ value }) => somenteDigitos(value))
  @CpfValido({ message: MENSAGENS_CAMPO.cpf })
  cpf: string;

  @ApiProperty({ example: 500000, description: 'Salário em centavos' })
  @IsInt({ message: MENSAGENS_CAMPO.valorPositivo })
  @Min(1, { message: MENSAGENS_CAMPO.valorPositivo })
  @Max(100000000, { message: MENSAGENS_CAMPO.valorPositivo })
  salarioEmCentavos: number;
}
```

A idade (16 a 120) e o telefone (`telefoneValido`) são conferidos no service, porque dependem de regras com data atual e DDD. As falhas viram `ExcecaoApi(400, 'VALIDACAO_FALHOU', [{ campo, mensagem }])`. Exporte `limparEspacos` e `somenteDigitos` em `src/comum/utilitarios/transformacoes.ts` para reutilizar nos outros DTOs.

```ts
// dto/entrar.dto.ts
import { ApiProperty } from '@nestjs/swagger';
import { Transform } from 'class-transformer';
import { IsNotEmpty, IsString } from 'class-validator';
import { MENSAGENS_CAMPO } from '../../comum/mensagens';

export class EntrarDto {
  @ApiProperty({ example: 'maria@exemplo.com' })
  @Transform(({ value }) => {
    if (typeof value !== 'string') { return value; }
    return value.trim().toLowerCase();
  })
  @IsNotEmpty({ message: MENSAGENS_CAMPO.obrigatorio })
  email: string;

  @ApiProperty({ example: 'Sobra@2026' })
  @IsString({ message: MENSAGENS_CAMPO.obrigatorio })
  @IsNotEmpty({ message: MENSAGENS_CAMPO.obrigatorio })
  senha: string;
}
```

- [ ] **Passo 3: Mapeador, strategy e guard**

```ts
// src/usuarios/usuario.mapeador.ts
import { Usuario } from '../generated/prisma/client';
import { calcularIdade, paraDataIso } from '../comum/utilitarios/datas';

export interface RespostaUsuario {
  id: string;
  nome: string;
  dataNascimento: string;
  idade: number;
  email: string;
  telefone: string;
  cpf: string;
  salarioEmCentavos: number;
  criadoEm: string;
}

// Monta a resposta do usuário sem nunca incluir o hash da senha
export function mapearUsuario(usuario: Usuario): RespostaUsuario {
  return {
    id: usuario.id,
    nome: usuario.nome,
    dataNascimento: paraDataIso(usuario.dataNascimento),
    idade: calcularIdade(usuario.dataNascimento, new Date()),
    email: usuario.email,
    telefone: usuario.telefone,
    cpf: usuario.cpf,
    salarioEmCentavos: usuario.salarioEmCentavos,
    criadoEm: usuario.criadoEm.toISOString(),
  };
}
```

```ts
// jwt.strategy.ts
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

interface ConteudoToken {
  sub: string;
  email: string;
}

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SEGREDO as string,
    });
  }

  validate(conteudo: ConteudoToken) {
    return { usuarioId: conteudo.sub, email: conteudo.email };
  }
}
```

```ts
// jwt-auth.guard.ts
import { ExecutionContext, Injectable } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { AuthGuard } from '@nestjs/passport';
import { CHAVE_ROTA_PUBLICA } from '../comum/decorators/publica.decorator';
import { ExcecaoApi } from '../comum/excecao-api';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private readonly refletor: Reflector) {
    super();
  }

  canActivate(contexto: ExecutionContext) {
    const ehPublica = this.refletor.getAllAndOverride<boolean>(CHAVE_ROTA_PUBLICA, [
      contexto.getHandler(),
      contexto.getClass(),
    ]);
    if (ehPublica) {
      return true;
    }
    return super.canActivate(contexto);
  }

  handleRequest<TUsuario>(erro: unknown, usuario: TUsuario): TUsuario {
    if (erro || !usuario) {
      throw new ExcecaoApi(401, 'SESSAO_EXPIRADA');
    }
    return usuario;
  }
}
```

- [ ] **Passo 4: Service e controller**

```ts
// autenticacao.service.ts
import { Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import * as bcrypt from 'bcrypt';
import { PrismaService } from '../prisma/prisma.service';
import { ExcecaoApi } from '../comum/excecao-api';
import { MENSAGENS_CAMPO } from '../comum/mensagens';
import { calcularIdade } from '../comum/utilitarios/datas';
import { telefoneValido } from '../comum/validadores/regras';
import { mapearUsuario } from '../usuarios/usuario.mapeador';
import { Usuario } from '../generated/prisma/client';
import { CadastroDto } from './dto/cadastro.dto';
import { EntrarDto } from './dto/entrar.dto';

const CUSTO_BCRYPT = 10;

@Injectable()
export class AutenticacaoService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly jwt: JwtService,
  ) {}

  async cadastrar(dados: CadastroDto) {
    this.validarIdadeETelefone(dados.dataNascimento, dados.telefone);
    await this.garantirEmailECpfLivres(dados.email, dados.cpf);

    const senhaHash = await bcrypt.hash(dados.senha, CUSTO_BCRYPT);
    const usuario = await this.prisma.usuario.create({
      data: {
        nome: dados.nome,
        dataNascimento: new Date(dados.dataNascimento),
        email: dados.email,
        telefone: dados.telefone,
        cpf: dados.cpf,
        salarioEmCentavos: dados.salarioEmCentavos,
        senhaHash: senhaHash,
      },
    });
    return this.montarRespostaComToken(usuario);
  }

  async entrar(dados: EntrarDto) {
    const usuario = await this.prisma.usuario.findUnique({ where: { email: dados.email } });
    if (!usuario) {
      throw new ExcecaoApi(401, 'CREDENCIAIS_INVALIDAS');
    }
    const senhaConfere = await bcrypt.compare(dados.senha, usuario.senhaHash);
    if (!senhaConfere) {
      throw new ExcecaoApi(401, 'CREDENCIAIS_INVALIDAS');
    }
    return this.montarRespostaComToken(usuario);
  }

  // Também usado pelo UsuariosService ao editar o perfil
  validarIdadeETelefone(dataNascimento: string | undefined, telefone: string | undefined) {
    if (dataNascimento !== undefined) {
      const nascimento = new Date(dataNascimento);
      const idade = calcularIdade(nascimento, new Date());
      if (nascimento > new Date() || idade < 16 || idade > 120) {
        throw new ExcecaoApi(400, 'VALIDACAO_FALHOU', [
          { campo: 'dataNascimento', mensagem: MENSAGENS_CAMPO.dataNascimento },
        ]);
      }
    }
    if (telefone !== undefined && !telefoneValido(telefone)) {
      throw new ExcecaoApi(400, 'VALIDACAO_FALHOU', [
        { campo: 'telefone', mensagem: MENSAGENS_CAMPO.telefone },
      ]);
    }
  }

  private async garantirEmailECpfLivres(email: string, cpf: string) {
    const comEmail = await this.prisma.usuario.findUnique({ where: { email } });
    if (comEmail) {
      throw new ExcecaoApi(409, 'EMAIL_JA_CADASTRADO');
    }
    const comCpf = await this.prisma.usuario.findUnique({ where: { cpf } });
    if (comCpf) {
      throw new ExcecaoApi(409, 'CPF_JA_CADASTRADO');
    }
  }

  private async montarRespostaComToken(usuario: Usuario) {
    const expiraEm = Number(process.env.JWT_EXPIRACAO_SEGUNDOS) || 3600;
    const tokenAcesso = await this.jwt.signAsync(
      { sub: usuario.id, email: usuario.email },
      { expiresIn: expiraEm },
    );
    return { tokenAcesso, expiraEm, usuario: mapearUsuario(usuario) };
  }
}
```

```ts
// autenticacao.controller.ts
import { Body, Controller, HttpCode, Post, UseGuards } from '@nestjs/common';
import { ApiOperation, ApiResponse, ApiTags } from '@nestjs/swagger';
import { ThrottlerGuard } from '@nestjs/throttler';
import { Publica } from '../comum/decorators/publica.decorator';
import { AutenticacaoService } from './autenticacao.service';
import { CadastroDto } from './dto/cadastro.dto';
import { EntrarDto } from './dto/entrar.dto';

@ApiTags('Autenticação')
@Publica()
@UseGuards(ThrottlerGuard)
@Controller('auth')
export class AutenticacaoController {
  constructor(private readonly autenticacaoService: AutenticacaoService) {}

  @Post('cadastro')
  @ApiOperation({ summary: 'Cadastrar novo usuário' })
  @ApiResponse({ status: 201, description: 'Conta criada' })
  @ApiResponse({ status: 400, description: 'Dados inválidos' })
  @ApiResponse({ status: 409, description: 'E-mail ou CPF já cadastrado' })
  @ApiResponse({ status: 429, description: 'Muitas tentativas' })
  cadastrar(@Body() dados: CadastroDto) {
    return this.autenticacaoService.cadastrar(dados);
  }

  @Post('entrar')
  @HttpCode(200)
  @ApiOperation({ summary: 'Entrar (login)' })
  @ApiResponse({ status: 200, description: 'Login realizado' })
  @ApiResponse({ status: 401, description: 'E-mail ou senha inválidos' })
  @ApiResponse({ status: 429, description: 'Muitas tentativas' })
  entrar(@Body() dados: EntrarDto) {
    return this.autenticacaoService.entrar(dados);
  }
}
```

> **Swagger em todos os controllers:** além de `@ApiResponse` com `description`, cada resposta deve apontar `type` para uma classe de resposta (ex.: `RespostaAutenticacaoDto`, `ErroDto`) com `@ApiProperty`, espelhando os schemas do `openapi.yaml`. Crie essas classes em `src/comum/swagger/` (as de erro) e na pasta `dto/` de cada módulo (as de resposta).

```ts
// autenticacao.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { AutenticacaoController } from './autenticacao.controller';
import { AutenticacaoService } from './autenticacao.service';
import { JwtStrategy } from './jwt.strategy';

@Module({
  imports: [PassportModule, JwtModule.register({ secret: process.env.JWT_SEGREDO })],
  controllers: [AutenticacaoController],
  providers: [AutenticacaoService, JwtStrategy],
  exports: [AutenticacaoService],
})
export class AutenticacaoModule {}
```

> `JwtModule.register` lê `process.env` na hora do import. Garanta que o `ConfigModule.forRoot()` carregue o `.env` antes, ou use `JwtModule.registerAsync` com o `ConfigService`.

Em `app.module.ts`, adicionar:
```ts
imports: [
  ConfigModule.forRoot({ isGlobal: true }),
  ThrottlerModule.forRoot([{ ttl: 60000, limit: 5 }]),
  PrismaModule,
  AutenticacaoModule,
],
providers: [{ provide: APP_GUARD, useClass: JwtAuthGuard }],
```

- [ ] **Passo 5: Utilitários dos testes e2e**

```ts
// test/utilitarios-e2e.ts
import { INestApplication } from '@nestjs/common';
import { Test } from '@nestjs/testing';
import request from 'supertest';
import { AppModule } from '../src/app.module';
import { configurarApp } from '../src/configurar-app';

export async function criarAppTeste(): Promise<INestApplication> {
  const modulo = await Test.createTestingModule({ imports: [AppModule] }).compile();
  const app = modulo.createNestApplication();
  configurarApp(app);
  await app.init();
  return app;
}

export async function cadastrarUsuarioTeste(app: INestApplication, sufixo: string, cpf: string) {
  const resposta = await request(app.getHttpServer())
    .post('/api/auth/cadastro')
    .send({
      nome: 'Pessoa Teste',
      dataNascimento: '1995-05-10',
      email: `teste.${sufixo}@exemplo.com`,
      senha: 'Sobra@2026',
      telefone: '11912345678',
      cpf: cpf,
      salarioEmCentavos: 500000,
    });
  return { token: resposta.body.tokenAcesso as string, id: resposta.body.usuario.id as string };
}
```

> Os testes e2e usam um banco separado. Crie `backend/.env.test` com `DATABASE_URL` apontando para `sobra_teste` e rode `npx prisma migrate reset --force` nele antes de cada execução da suíte. Adicione no `package.json` o script `"test:e2e": "dotenv -e .env.test -- jest --config ./test/jest-e2e.json --runInBand"`. Se `dotenv-cli` não estiver disponível, carregue o `.env.test` no topo do `utilitarios-e2e.ts` com `require('dotenv').config({ path: '.env.test', override: true })`, sem instalar nada novo.

- [ ] **Passo 6: Escrever o teste e2e (que deve falhar)**

```ts
// test/autenticacao.e2e-spec.ts
import { INestApplication } from '@nestjs/common';
import request from 'supertest';
import { criarAppTeste } from './utilitarios-e2e';

describe('Autenticação (e2e)', () => {
  let app: INestApplication;
  const corpoValido = {
    nome: 'Maria Souza',
    dataNascimento: '1998-04-15',
    email: 'Maria@Exemplo.com',
    senha: 'Sobra@2026',
    telefone: '(11) 91234-5678',
    cpf: '529.982.247-25',
    salarioEmCentavos: 500000,
  };

  beforeAll(async () => { app = await criarAppTeste(); });
  afterAll(async () => { await app.close(); });

  it('cadastra, normaliza os dados e não devolve a senha', async () => {
    const resposta = await request(app.getHttpServer()).post('/api/auth/cadastro').send(corpoValido);
    expect(resposta.status).toBe(201);
    expect(resposta.body.tokenAcesso).toBeDefined();
    expect(resposta.body.usuario.email).toBe('maria@exemplo.com');
    expect(resposta.body.usuario.cpf).toBe('52998224725');
    expect(resposta.body.usuario.telefone).toBe('11912345678');
    expect(resposta.body.usuario.senhaHash).toBeUndefined();
  });

  it('recusa e-mail duplicado com 409', async () => {
    const resposta = await request(app.getHttpServer())
      .post('/api/auth/cadastro')
      .send({ ...corpoValido, cpf: '11144477735' });
    expect(resposta.status).toBe(409);
    expect(resposta.body.codigo).toBe('EMAIL_JA_CADASTRADO');
  });

  it('recusa CPF inválido e senha fraca listando os campos', async () => {
    const resposta = await request(app.getHttpServer())
      .post('/api/auth/cadastro')
      .send({ ...corpoValido, email: 'outra@exemplo.com', cpf: '52998224724', senha: 'Abc12345' });
    expect(resposta.status).toBe(400);
    expect(resposta.body.codigo).toBe('VALIDACAO_FALHOU');
    const campos = resposta.body.erros.map((erro: { campo: string }) => erro.campo);
    expect(campos).toEqual(expect.arrayContaining(['cpf', 'senha']));
  });

  it('recusa campos extras', async () => {
    const resposta = await request(app.getHttpServer())
      .post('/api/auth/entrar')
      .send({ email: 'maria@exemplo.com', senha: 'Sobra@2026', admin: true });
    expect(resposta.status).toBe(400);
  });

  it('entra com credenciais certas e recusa as erradas com mensagem genérica', async () => {
    const certo = await request(app.getHttpServer())
      .post('/api/auth/entrar')
      .send({ email: 'maria@exemplo.com', senha: 'Sobra@2026' });
    expect(certo.status).toBe(200);
    const errado = await request(app.getHttpServer())
      .post('/api/auth/entrar')
      .send({ email: 'maria@exemplo.com', senha: 'Errada@123' });
    expect(errado.status).toBe(401);
    expect(errado.body.mensagem).toBe('E-mail ou senha inválidos.');
  });

  it('bloqueia a 6ª tentativa em 1 minuto com 429', async () => {
    let ultimoStatus = 0;
    for (let tentativa = 0; tentativa < 6; tentativa++) {
      const resposta = await request(app.getHttpServer())
        .post('/api/auth/entrar')
        .send({ email: 'ninguem@exemplo.com', senha: 'Errada@123' });
      ultimoStatus = resposta.status;
    }
    expect(ultimoStatus).toBe(429);
  });
});
```

> O teste de throttler compartilha o limite com os testes anteriores do mesmo arquivo. Se ele interferir, mova-o para um arquivo e2e próprio.

- [ ] **Passo 7: Rodar e ver falhar, implementar o que faltar e rodar até passar**

Rodar: `npm run test:e2e -- autenticacao`
Esperado no fim: PASSA (6 testes).

- [ ] **Passo 8: Commit**

```bash
git add backend
git commit -m "feat(auth): cadastro e login com bcrypt, Passport JWT global e limite de tentativas"
```

### Tarefa 5: Usuários (perfil)

**Arquivos:**
- Criar: `src/usuarios/usuarios.module.ts`, `usuarios.controller.ts`, `usuarios.service.ts`, `dto/atualizar-usuario.dto.ts`
- Teste: `test/usuarios.e2e-spec.ts`

**Interfaces:**
- Consome: `mapearUsuario`, `AutenticacaoService.validarIdadeETelefone`, `@UsuarioAtual`
- Produz: `GET /usuarios/eu` e `PATCH /usuarios/eu`, que devolvem `RespostaUsuario`

- [ ] **Passo 1: Teste e2e (que deve falhar)**

```ts
import { INestApplication } from '@nestjs/common';
import request from 'supertest';
import { cadastrarUsuarioTeste, criarAppTeste } from './utilitarios-e2e';

describe('Usuários (e2e)', () => {
  let app: INestApplication;
  let token: string;

  beforeAll(async () => {
    app = await criarAppTeste();
    token = (await cadastrarUsuarioTeste(app, 'perfil', '52998224725')).token;
  });
  afterAll(async () => { await app.close(); });

  it('exige token', async () => {
    const resposta = await request(app.getHttpServer()).get('/api/usuarios/eu');
    expect(resposta.status).toBe(401);
    expect(resposta.body.codigo).toBe('SESSAO_EXPIRADA');
  });

  it('devolve o perfil com idade calculada', async () => {
    const resposta = await request(app.getHttpServer())
      .get('/api/usuarios/eu')
      .set('Authorization', `Bearer ${token}`);
    expect(resposta.status).toBe(200);
    expect(typeof resposta.body.idade).toBe('number');
    expect(resposta.body.senhaHash).toBeUndefined();
  });

  it('atualiza o salário e recusa alteração de e-mail', async () => {
    const atualizado = await request(app.getHttpServer())
      .patch('/api/usuarios/eu')
      .set('Authorization', `Bearer ${token}`)
      .send({ salarioEmCentavos: 650000 });
    expect(atualizado.status).toBe(200);
    expect(atualizado.body.salarioEmCentavos).toBe(650000);

    const comEmail = await request(app.getHttpServer())
      .patch('/api/usuarios/eu')
      .set('Authorization', `Bearer ${token}`)
      .send({ email: 'novo@exemplo.com' });
    expect(comEmail.status).toBe(400);
  });
});
```

- [ ] **Passo 2: Implementar**

`AtualizarUsuarioDto` tem `nome`, `dataNascimento`, `telefone` e `salarioEmCentavos`, todos com `@IsOptional()` e as mesmas regras e transformações do `CadastroDto`. Não declarar `email` nem `cpf`, para que o `forbidNonWhitelisted` os recuse.

```ts
// usuarios.service.ts
@Injectable()
export class UsuariosService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly autenticacaoService: AutenticacaoService,
  ) {}

  async buscarPerfil(usuarioId: string) {
    const usuario = await this.prisma.usuario.findUnique({ where: { id: usuarioId } });
    if (!usuario) {
      throw new ExcecaoApi(404, 'NAO_ENCONTRADO');
    }
    return mapearUsuario(usuario);
  }

  async atualizarPerfil(usuarioId: string, dados: AtualizarUsuarioDto) {
    this.autenticacaoService.validarIdadeETelefone(dados.dataNascimento, dados.telefone);
    const novosDados: Prisma.UsuarioUpdateInput = {};
    if (dados.nome !== undefined) { novosDados.nome = dados.nome; }
    if (dados.dataNascimento !== undefined) { novosDados.dataNascimento = new Date(dados.dataNascimento); }
    if (dados.telefone !== undefined) { novosDados.telefone = dados.telefone; }
    if (dados.salarioEmCentavos !== undefined) { novosDados.salarioEmCentavos = dados.salarioEmCentavos; }
    const usuario = await this.prisma.usuario.update({ where: { id: usuarioId }, data: novosDados });
    return mapearUsuario(usuario);
  }
}
```

O controller tem `@ApiTags('Usuários')` e `@ApiBearerAuth()`, com `@Get('eu')` e `@Patch('eu')` usando `@UsuarioAtual() usuarioId: string`. Registrar `UsuariosModule` (que importa `AutenticacaoModule`) no `AppModule`.

- [ ] **Passo 3: Rodar até passar e fazer commit**

Rodar: `npm run test:e2e -- usuarios`, esperado PASSA.

```bash
git add backend
git commit -m "feat(usuarios): ver e atualizar perfil com idade calculada"
```

### Tarefa 6: Categorias

**Arquivos:**
- Criar: `src/categorias/*`
- Teste: `test/categorias.e2e-spec.ts`

**Interfaces:**
- Produz:
  - `CategoriasService.buscarDisponivel(usuarioId: string, categoriaId: string, tipo: TipoLancamento): Promise<Categoria>`, que lança 400 `VALIDACAO_FALHOU` com `{ campo: 'categoriaId', mensagem: MENSAGENS_CAMPO.categoriaOutroTipo }` se o tipo não bater, ou com `MENSAGENS_CAMPO.categoria` se a categoria não existir ou não for acessível
  - `mapearCategoria(categoria): { id, nome, tipo, icone, padrao: boolean }`

- [ ] **Passo 1: Teste e2e (que deve falhar)**

```ts
describe('Categorias (e2e)', () => {
  let app: INestApplication;
  let tokenA: string;
  let tokenB: string;

  beforeAll(async () => {
    app = await criarAppTeste();
    tokenA = (await cadastrarUsuarioTeste(app, 'cat-a', '52998224725')).token;
    tokenB = (await cadastrarUsuarioTeste(app, 'cat-b', '11144477735')).token;
  });
  afterAll(async () => { await app.close(); });

  it('lista as 8 categorias padrão de saída', async () => {
    const resposta = await request(app.getHttpServer())
      .get('/api/categorias?tipo=SAIDA')
      .set('Authorization', `Bearer ${tokenA}`);
    expect(resposta.status).toBe(200);
    const padroes = resposta.body.filter((categoria: { padrao: boolean }) => categoria.padrao);
    expect(padroes.length).toBe(8);
  });

  it('cria categoria própria, recusa duplicada e esconde de outro usuário', async () => {
    const criada = await request(app.getHttpServer())
      .post('/api/categorias')
      .set('Authorization', `Bearer ${tokenA}`)
      .send({ nome: 'Pets', tipo: 'SAIDA', icone: 'paw-print' });
    expect(criada.status).toBe(201);

    const duplicada = await request(app.getHttpServer())
      .post('/api/categorias')
      .set('Authorization', `Bearer ${tokenA}`)
      .send({ nome: 'Pets', tipo: 'SAIDA', icone: 'paw-print' });
    expect(duplicada.status).toBe(409);
    expect(duplicada.body.codigo).toBe('CATEGORIA_JA_EXISTE');

    const exclusaoPorOutro = await request(app.getHttpServer())
      .delete(`/api/categorias/${criada.body.id}`)
      .set('Authorization', `Bearer ${tokenB}`);
    expect(exclusaoPorOutro.status).toBe(404);
  });

  it('não exclui categoria padrão', async () => {
    const lista = await request(app.getHttpServer())
      .get('/api/categorias?tipo=ENTRADA')
      .set('Authorization', `Bearer ${tokenA}`);
    const resposta = await request(app.getHttpServer())
      .delete(`/api/categorias/${lista.body[0].id}`)
      .set('Authorization', `Bearer ${tokenA}`);
    expect(resposta.status).toBe(404);
  });
});
```

- [ ] **Passo 2: Implementar**

```ts
// categorias.service.ts (trechos centrais)
async listar(usuarioId: string, tipo?: TipoLancamento) {
  const filtro: Prisma.CategoriaWhereInput = { OR: [{ usuarioId: null }, { usuarioId }] };
  if (tipo) {
    filtro.tipo = tipo;
  }
  const categorias = await this.prisma.categoria.findMany({
    where: filtro,
    orderBy: [{ usuarioId: 'asc' }, { nome: 'asc' }],
  });
  return categorias.map(mapearCategoria);
}

async criar(usuarioId: string, dados: CriarCategoriaDto) {
  // Checa também as padrão, para não haver duas "Alimentação" na lista
  const existente = await this.prisma.categoria.findFirst({
    where: {
      nome: { equals: dados.nome, mode: 'insensitive' },
      tipo: dados.tipo,
      OR: [{ usuarioId: null }, { usuarioId }],
    },
  });
  if (existente) {
    throw new ExcecaoApi(409, 'CATEGORIA_JA_EXISTE');
  }
  const categoria = await this.prisma.categoria.create({ data: { ...dados, usuarioId } });
  return mapearCategoria(categoria);
}

async excluir(usuarioId: string, id: string) {
  const categoria = await this.prisma.categoria.findFirst({ where: { id, usuarioId } });
  if (!categoria) {
    throw new ExcecaoApi(404, 'NAO_ENCONTRADO');
  }
  const emUso = await this.prisma.lancamento.count({ where: { categoriaId: id } });
  if (emUso > 0) {
    throw new ExcecaoApi(409, 'CATEGORIA_EM_USO');
  }
  await this.prisma.categoria.delete({ where: { id } });
}

async buscarDisponivel(usuarioId: string, categoriaId: string, tipo: TipoLancamento) {
  const categoria = await this.prisma.categoria.findFirst({
    where: { id: categoriaId, OR: [{ usuarioId: null }, { usuarioId }] },
  });
  if (!categoria) {
    throw new ExcecaoApi(400, 'VALIDACAO_FALHOU', [{ campo: 'categoriaId', mensagem: MENSAGENS_CAMPO.categoria }]);
  }
  if (categoria.tipo !== tipo) {
    throw new ExcecaoApi(400, 'VALIDACAO_FALHOU', [{ campo: 'categoriaId', mensagem: MENSAGENS_CAMPO.categoriaOutroTipo }]);
  }
  return categoria;
}
```

```ts
export function mapearCategoria(categoria: Categoria) {
  let padrao = false;
  if (categoria.usuarioId === null) {
    padrao = true;
  }
  return { id: categoria.id, nome: categoria.nome, tipo: categoria.tipo, icone: categoria.icone, padrao };
}
```

`CriarCategoriaDto`: `nome` com `@Transform(limparEspacos)`, `@SemHtml({ message: MENSAGENS_CAMPO.nomeCategoria })` e `@Length(2, 40, { message: MENSAGENS_CAMPO.nomeCategoria })`. `tipo` com `@IsEnum(TipoLancamento, { message: MENSAGENS_CAMPO.tipo })`. `icone` com `@Matches(/^[a-z0-9-]{1,40}$/)`.

O `DELETE` usa `@HttpCode(204)` e `@Param('id', new ParseUUIDPipe())`. Um id inválido vira 400 pelo filtro. Exportar `CategoriasService` no módulo.

- [ ] **Passo 3: Rodar até passar e fazer commit**

```bash
git add backend
git commit -m "feat(categorias): categorias padrão e personalizadas por usuário"
```

### Tarefa 7: Lançamentos

**Arquivos:**
- Criar: `src/lancamentos/*`
- Teste: `test/lancamentos.e2e-spec.ts`

**Interfaces:**
- Consome: `CategoriasService.buscarDisponivel`, `mapearCategoria`, `intervaloDoMes`, `paraDataIso`
- Produz: as rotas `/lancamentos` com o formato `Lancamento` e `PaginaLancamentos` do `openapi.yaml`

- [ ] **Passo 1: Teste e2e (que deve falhar)**

```ts
describe('Lançamentos (e2e)', () => {
  let app: INestApplication;
  let tokenA: string;
  let tokenB: string;
  let categoriaSaidaId: string;
  let categoriaEntradaId: string;

  beforeAll(async () => {
    app = await criarAppTeste();
    tokenA = (await cadastrarUsuarioTeste(app, 'lanc-a', '52998224725')).token;
    tokenB = (await cadastrarUsuarioTeste(app, 'lanc-b', '11144477735')).token;
    const saidas = await request(app.getHttpServer()).get('/api/categorias?tipo=SAIDA').set('Authorization', `Bearer ${tokenA}`);
    const entradas = await request(app.getHttpServer()).get('/api/categorias?tipo=ENTRADA').set('Authorization', `Bearer ${tokenA}`);
    categoriaSaidaId = saidas.body[0].id;
    categoriaEntradaId = entradas.body[0].id;
  });
  afterAll(async () => { await app.close(); });

  function criarSaida(token: string, corpo: Record<string, unknown>) {
    return request(app.getHttpServer())
      .post('/api/lancamentos')
      .set('Authorization', `Bearer ${token}`)
      .send({ tipo: 'SAIDA', descricao: 'Mercado', valorEmCentavos: 12000, data: '2026-10-05', categoriaId: categoriaSaidaId, ...corpo });
  }

  it('cria e lista no mês', async () => {
    const criada = await criarSaida(tokenA, {});
    expect(criada.status).toBe(201);
    expect(criada.body.categoria.id).toBe(categoriaSaidaId);

    const lista = await request(app.getHttpServer())
      .get('/api/lancamentos?tipo=SAIDA&mes=10&ano=2026')
      .set('Authorization', `Bearer ${tokenA}`);
    expect(lista.body.totalItens).toBe(1);
    expect(lista.body.itens[0].data).toBe('2026-10-05');
  });

  it('recusa HTML na descrição com CONTEUDO_NAO_PERMITIDO', async () => {
    const resposta = await criarSaida(tokenA, { descricao: '<script>alert(1)</script>' });
    expect(resposta.status).toBe(400);
    expect(resposta.body.codigo).toBe('CONTEUDO_NAO_PERMITIDO');
  });

  it('recusa categoria de outro tipo', async () => {
    const resposta = await criarSaida(tokenA, { categoriaId: categoriaEntradaId });
    expect(resposta.status).toBe(400);
  });

  it('recusa usuarioId no corpo', async () => {
    const resposta = await criarSaida(tokenA, { usuarioId: 'qualquer' });
    expect(resposta.status).toBe(400);
  });

  it('isola usuários: B não lê, edita nem exclui o lançamento de A', async () => {
    const criada = await criarSaida(tokenA, { descricao: 'Privado' });
    const id = criada.body.id;
    const edicao = await request(app.getHttpServer())
      .patch(`/api/lancamentos/${id}`)
      .set('Authorization', `Bearer ${tokenB}`)
      .send({ valorEmCentavos: 1 });
    expect(edicao.status).toBe(404);
    const exclusao = await request(app.getHttpServer())
      .delete(`/api/lancamentos/${id}`)
      .set('Authorization', `Bearer ${tokenB}`);
    expect(exclusao.status).toBe(404);
    const listaB = await request(app.getHttpServer())
      .get('/api/lancamentos?tipo=SAIDA&mes=10&ano=2026')
      .set('Authorization', `Bearer ${tokenB}`);
    expect(listaB.body.totalItens).toBe(0);
  });

  it('edita e exclui o próprio lançamento', async () => {
    const criada = await criarSaida(tokenA, { descricao: 'Padaria' });
    const edicao = await request(app.getHttpServer())
      .patch(`/api/lancamentos/${criada.body.id}`)
      .set('Authorization', `Bearer ${tokenA}`)
      .send({ valorEmCentavos: 1500 });
    expect(edicao.status).toBe(200);
    expect(edicao.body.valorEmCentavos).toBe(1500);
    const exclusao = await request(app.getHttpServer())
      .delete(`/api/lancamentos/${criada.body.id}`)
      .set('Authorization', `Bearer ${tokenA}`);
    expect(exclusao.status).toBe(204);
  });

  it('recusa data fora do intervalo', async () => {
    const resposta = await criarSaida(tokenA, { data: '1999-12-31' });
    expect(resposta.status).toBe(400);
  });
});
```

- [ ] **Passo 2: Implementar**

DTOs:
- `CriarLancamentoDto`:
  - `tipo` com `@IsEnum`.
  - `descricao` com `@Transform(limparEspacos)`, `@SemHtml({ message: MENSAGENS_CAMPO.descricao })` e `@Length(2, 100, { message: MENSAGENS_CAMPO.descricao })`.
  - `valorEmCentavos` com `@IsInt`, `@Min(1)` e `@Max(1000000000)`, todos com a mensagem `valorPositivo`.
  - `data` com `@IsDateString({ strict: true })` e mensagem `dataLancamento`.
  - `categoriaId` com `@IsUUID('4', { message: MENSAGENS_CAMPO.categoria })`.
- `AtualizarLancamentoDto`: `PartialType(CriarLancamentoDto)`, importado de `@nestjs/swagger`.
- `FiltroLancamentosDto`:
  - `tipo` com `@IsEnum`.
  - `mes` com `@Type(() => Number)`, `@IsInt`, `@Min(1)` e `@Max(12)`.
  - `ano` com `@Type(() => Number)`, `@IsInt` e `@Min(2000)`.
  - `pagina` com `@IsOptional`, `@Type(() => Number)`, `@IsInt` e `@Min(1)`.

```ts
// lancamentos.service.ts (trechos centrais)
const TAMANHO_PAGINA = 20;

function validarDataLancamento(data: string) {
  const ano = Number(data.substring(0, 4));
  const anoMaximo = new Date().getUTCFullYear() + 1;
  if (ano < 2000 || ano > anoMaximo) {
    throw new ExcecaoApi(400, 'VALIDACAO_FALHOU', [{ campo: 'data', mensagem: MENSAGENS_CAMPO.dataLancamento }]);
  }
}

export function mapearLancamento(lancamento: Lancamento & { categoria: Categoria }) {
  return {
    id: lancamento.id,
    tipo: lancamento.tipo,
    descricao: lancamento.descricao,
    valorEmCentavos: lancamento.valorEmCentavos,
    data: paraDataIso(lancamento.data),
    categoria: mapearCategoria(lancamento.categoria),
    criadoEm: lancamento.criadoEm.toISOString(),
    atualizadoEm: lancamento.atualizadoEm.toISOString(),
  };
}

async listar(usuarioId: string, filtro: FiltroLancamentosDto) {
  const intervalo = intervaloDoMes(filtro.mes, filtro.ano);
  let pagina = 1;
  if (filtro.pagina) {
    pagina = filtro.pagina;
  }
  const condicao = { usuarioId, tipo: filtro.tipo, data: { gte: intervalo.inicio, lt: intervalo.fim } };
  const totalItens = await this.prisma.lancamento.count({ where: condicao });
  const itens = await this.prisma.lancamento.findMany({
    where: condicao,
    include: { categoria: true },
    orderBy: [{ data: 'desc' }, { criadoEm: 'desc' }],
    skip: (pagina - 1) * TAMANHO_PAGINA,
    take: TAMANHO_PAGINA,
  });
  return {
    itens: itens.map(mapearLancamento),
    pagina,
    tamanhoPagina: TAMANHO_PAGINA,
    totalItens,
    totalPaginas: Math.ceil(totalItens / TAMANHO_PAGINA),
  };
}

async criar(usuarioId: string, dados: CriarLancamentoDto) {
  validarDataLancamento(dados.data);
  await this.categoriasService.buscarDisponivel(usuarioId, dados.categoriaId, dados.tipo);
  const lancamento = await this.prisma.lancamento.create({
    data: { ...dados, data: new Date(dados.data), usuarioId },
    include: { categoria: true },
  });
  return mapearLancamento(lancamento);
}

// Busca garantindo que é do usuário. Se não for, 404 (RN07)
private async buscarDoUsuario(usuarioId: string, id: string) {
  const lancamento = await this.prisma.lancamento.findFirst({ where: { id, usuarioId } });
  if (!lancamento) {
    throw new ExcecaoApi(404, 'NAO_ENCONTRADO');
  }
  return lancamento;
}

async atualizar(usuarioId: string, id: string, dados: AtualizarLancamentoDto) {
  const atual = await this.buscarDoUsuario(usuarioId, id);
  let tipoFinal = atual.tipo;
  if (dados.tipo) { tipoFinal = dados.tipo; }
  let categoriaFinal = atual.categoriaId;
  if (dados.categoriaId) { categoriaFinal = dados.categoriaId; }
  await this.categoriasService.buscarDisponivel(usuarioId, categoriaFinal, tipoFinal);

  const novosDados: Prisma.LancamentoUncheckedUpdateInput = {};
  if (dados.tipo !== undefined) { novosDados.tipo = dados.tipo; }
  if (dados.descricao !== undefined) { novosDados.descricao = dados.descricao; }
  if (dados.valorEmCentavos !== undefined) { novosDados.valorEmCentavos = dados.valorEmCentavos; }
  if (dados.categoriaId !== undefined) { novosDados.categoriaId = dados.categoriaId; }
  if (dados.data !== undefined) {
    validarDataLancamento(dados.data);
    novosDados.data = new Date(dados.data);
  }
  const lancamento = await this.prisma.lancamento.update({
    where: { id }, data: novosDados, include: { categoria: true },
  });
  return mapearLancamento(lancamento);
}

async excluir(usuarioId: string, id: string) {
  await this.buscarDoUsuario(usuarioId, id);
  await this.prisma.lancamento.delete({ where: { id } });
}
```

- [ ] **Passo 3: Rodar até passar e fazer commit**

```bash
git add backend
git commit -m "feat(lancamentos): CRUD de entradas e saídas com isolamento por usuário"
```

### Tarefa 8: Cálculos de resumo (funções puras)

**Arquivos:**
- Criar: `src/resumos/calculos-resumo.ts`
- Teste: `src/resumos/calculos-resumo.spec.ts`

**Interfaces:**
- Produz:
  - `type SituacaoSobra = 'NEGATIVA' | 'ABAIXO_DA_META' | 'META_ATINGIDA'`
  - `calcularValorSugerido(salarioEmCentavos: number): number`
  - `definirSituacaoSobra(sobra: number, valorSugerido: number): SituacaoSobra`
  - `calcularPercentualGuardado(sobra: number, salario: number): number`
  - `interface LancamentoResumido { tipo: TipoLancamento; valorEmCentavos: number; data: Date }`
  - `interface TotalMes { mes: number; entradas: number; saidas: number; sobra: number }`
  - `agruparPorMes(lancamentos: LancamentoResumido[]): TotalMes[]` (sempre 12 itens)
  - `calcularProgressoMeta(sobrasMensais: number[], valorAlvo: number): { valorAcumulado: number; progresso: number; atingida: boolean }`

- [ ] **Passo 1: Testes (que devem falhar)**

```ts
import {
  agruparPorMes, calcularPercentualGuardado, calcularProgressoMeta,
  calcularValorSugerido, definirSituacaoSobra,
} from './calculos-resumo';

describe('calculos-resumo', () => {
  it('30% do salário arredondado para o centavo', () => {
    expect(calcularValorSugerido(500000)).toBe(150000);
    expect(calcularValorSugerido(333333)).toBe(100000);
  });

  it('situação da sobra nos limites', () => {
    expect(definirSituacaoSobra(-1, 150000)).toBe('NEGATIVA');
    expect(definirSituacaoSobra(0, 150000)).toBe('ABAIXO_DA_META');
    expect(definirSituacaoSobra(149999, 150000)).toBe('ABAIXO_DA_META');
    expect(definirSituacaoSobra(150000, 150000)).toBe('META_ATINGIDA');
  });

  it('percentual guardado com 1 casa decimal', () => {
    expect(calcularPercentualGuardado(139000, 500000)).toBe(27.8);
    expect(calcularPercentualGuardado(-50000, 500000)).toBe(-10);
  });

  it('agrupa por mês e preenche meses vazios com zero', () => {
    const meses = agruparPorMes([
      { tipo: 'ENTRADA', valorEmCentavos: 500000, data: new Date(Date.UTC(2026, 0, 5)) },
      { tipo: 'SAIDA', valorEmCentavos: 120000, data: new Date(Date.UTC(2026, 0, 10)) },
      { tipo: 'SAIDA', valorEmCentavos: 30000, data: new Date(Date.UTC(2026, 2, 1)) },
    ]);
    expect(meses.length).toBe(12);
    expect(meses[0]).toEqual({ mes: 1, entradas: 500000, saidas: 120000, sobra: 380000 });
    expect(meses[1]).toEqual({ mes: 2, entradas: 0, saidas: 0, sobra: 0 });
    expect(meses[2].sobra).toBe(-30000);
  });

  it('progresso da meta ignora sobras negativas e limita a 100%', () => {
    expect(calcularProgressoMeta([100000, -50000, 260000], 500000)).toEqual({
      valorAcumulado: 360000, progresso: 72, atingida: false,
    });
    expect(calcularProgressoMeta([600000], 500000)).toEqual({
      valorAcumulado: 600000, progresso: 100, atingida: true,
    });
  });
});
```

- [ ] **Passo 2: Rodar e ver falhar**

Rodar: `npx jest src/resumos`, esperado FALHA.

- [ ] **Passo 3: Implementar**

```ts
import { TipoLancamento } from '../generated/prisma/client';

export type SituacaoSobra = 'NEGATIVA' | 'ABAIXO_DA_META' | 'META_ATINGIDA';

export interface LancamentoResumido {
  tipo: TipoLancamento;
  valorEmCentavos: number;
  data: Date;
}

export interface TotalMes {
  mes: number;
  entradas: number;
  saidas: number;
  sobra: number;
}

const PERCENTUAL_SUGERIDO = 0.3;

export function calcularValorSugerido(salarioEmCentavos: number): number {
  return Math.round(salarioEmCentavos * PERCENTUAL_SUGERIDO);
}

// Decide a cor e o texto do badge de sobra (RN04)
export function definirSituacaoSobra(sobra: number, valorSugerido: number): SituacaoSobra {
  if (sobra < 0) {
    return 'NEGATIVA';
  }
  if (sobra < valorSugerido) {
    return 'ABAIXO_DA_META';
  }
  return 'META_ATINGIDA';
}

export function calcularPercentualGuardado(sobra: number, salario: number): number {
  if (salario <= 0) {
    return 0;
  }
  return Math.round((sobra / salario) * 1000) / 10;
}

export function agruparPorMes(lancamentos: LancamentoResumido[]): TotalMes[] {
  const meses: TotalMes[] = [];
  for (let numero = 1; numero <= 12; numero++) {
    meses.push({ mes: numero, entradas: 0, saidas: 0, sobra: 0 });
  }
  for (const lancamento of lancamentos) {
    const mes = meses[lancamento.data.getUTCMonth()];
    if (lancamento.tipo === 'ENTRADA') {
      mes.entradas = mes.entradas + lancamento.valorEmCentavos;
    } else {
      mes.saidas = mes.saidas + lancamento.valorEmCentavos;
    }
  }
  for (const mes of meses) {
    mes.sobra = mes.entradas - mes.saidas;
  }
  return meses;
}

// Soma só os meses em que sobrou dinheiro (RN05)
export function calcularProgressoMeta(sobrasMensais: number[], valorAlvo: number) {
  let valorAcumulado = 0;
  for (const sobra of sobrasMensais) {
    if (sobra > 0) {
      valorAcumulado = valorAcumulado + sobra;
    }
  }
  let progresso = Math.round((valorAcumulado / valorAlvo) * 1000) / 10;
  if (progresso > 100) {
    progresso = 100;
  }
  return { valorAcumulado, progresso, atingida: valorAcumulado >= valorAlvo };
}
```

- [ ] **Passo 4: Rodar e ver passar, depois fazer commit**

```bash
git add backend/src/resumos
git commit -m "feat(resumos): cálculos de sobra, sugestão de 30% e progresso de meta"
```

### Tarefa 9: Rotas de resumo

**Arquivos:**
- Criar: `src/resumos/resumos.module.ts`, `resumos.controller.ts`, `resumos.service.ts`, `dto/filtro-mes.dto.ts`, `dto/filtro-ano.dto.ts`, `dto/filtro-comparativo.dto.ts`
- Teste: `test/resumos.e2e-spec.ts`

**Interfaces:**
- Consome: as funções da Tarefa 8, `intervaloDoMes`, `intervaloDoAno`, `mapearCategoria`
- Produz: `ResumosService.sobrasMensaisDesde(usuarioId: string, inicio: Date, hoje: Date): Promise<number[]>`, usado pela Tarefa 10

- [ ] **Passo 1: Teste e2e (que deve falhar)**

```ts
describe('Resumos (e2e)', () => {
  let app: INestApplication;
  let token: string;

  beforeAll(async () => {
    app = await criarAppTeste();
    token = (await cadastrarUsuarioTeste(app, 'resumo', '52998224725')).token; // salário 500000
    const servidor = app.getHttpServer();
    const cabecalho = { Authorization: `Bearer ${token}` };
    const entradas = await request(servidor).get('/api/categorias?tipo=ENTRADA').set(cabecalho);
    const saidas = await request(servidor).get('/api/categorias?tipo=SAIDA').set(cabecalho);
    await request(servidor).post('/api/lancamentos').set(cabecalho).send({ tipo: 'ENTRADA', descricao: 'Salário', valorEmCentavos: 500000, data: '2026-10-01', categoriaId: entradas.body[0].id });
    await request(servidor).post('/api/lancamentos').set(cabecalho).send({ tipo: 'SAIDA', descricao: 'Aluguel', valorEmCentavos: 200000, data: '2026-10-02', categoriaId: saidas.body[0].id });
    await request(servidor).post('/api/lancamentos').set(cabecalho).send({ tipo: 'SAIDA', descricao: 'Mercado', valorEmCentavos: 181000, data: '2026-10-03', categoriaId: saidas.body[1].id });
  });
  afterAll(async () => { await app.close(); });

  it('resumo mensal soma as saídas automaticamente', async () => {
    const resposta = await request(app.getHttpServer())
      .get('/api/resumos/mensal?mes=10&ano=2026')
      .set('Authorization', `Bearer ${token}`);
    expect(resposta.body).toEqual({
      mes: 10, ano: 2026, totalEntradas: 500000, totalSaidas: 381000, sobra: 119000,
      salario: 500000, valorSugerido: 150000, percentualGuardado: 23.8, situacao: 'ABAIXO_DA_META',
    });
  });

  it('resumo anual tem 12 meses e total', async () => {
    const resposta = await request(app.getHttpServer())
      .get('/api/resumos/anual?ano=2026')
      .set('Authorization', `Bearer ${token}`);
    expect(resposta.body.meses.length).toBe(12);
    expect(resposta.body.meses[9].sobra).toBe(119000);
    expect(resposta.body.total).toEqual({ entradas: 500000, saidas: 381000, sobra: 119000 });
  });

  it('totais por categoria em ordem decrescente', async () => {
    const resposta = await request(app.getHttpServer())
      .get('/api/resumos/mensal/categorias?mes=10&ano=2026&tipo=SAIDA')
      .set('Authorization', `Bearer ${token}`);
    expect(resposta.body[0].totalEmCentavos).toBe(200000);
  });

  it('comparativo de 3 anos em ordem crescente', async () => {
    const resposta = await request(app.getHttpServer())
      .get('/api/resumos/comparativo-anos?quantidade=3')
      .set('Authorization', `Bearer ${token}`);
    expect(resposta.body.length).toBe(3);
    expect(resposta.body[0].ano).toBeLessThan(resposta.body[2].ano);
  });

  it('recusa mês 13', async () => {
    const resposta = await request(app.getHttpServer())
      .get('/api/resumos/mensal?mes=13&ano=2026')
      .set('Authorization', `Bearer ${token}`);
    expect(resposta.status).toBe(400);
  });
});
```

- [ ] **Passo 2: Implementar**

```ts
// resumos.service.ts (trechos centrais)
async mensal(usuarioId: string, mes: number, ano: number) {
  const intervalo = intervaloDoMes(mes, ano);
  const usuario = await this.prisma.usuario.findUniqueOrThrow({ where: { id: usuarioId } });
  const somas = await this.prisma.lancamento.groupBy({
    by: ['tipo'],
    where: { usuarioId, data: { gte: intervalo.inicio, lt: intervalo.fim } },
    _sum: { valorEmCentavos: true },
  });
  let totalEntradas = 0;
  let totalSaidas = 0;
  for (const soma of somas) {
    const valor = soma._sum.valorEmCentavos || 0;
    if (soma.tipo === 'ENTRADA') {
      totalEntradas = valor;
    } else {
      totalSaidas = valor;
    }
  }
  const sobra = totalEntradas - totalSaidas;
  const valorSugerido = calcularValorSugerido(usuario.salarioEmCentavos);
  return {
    mes, ano, totalEntradas, totalSaidas, sobra,
    salario: usuario.salarioEmCentavos,
    valorSugerido,
    percentualGuardado: calcularPercentualGuardado(sobra, usuario.salarioEmCentavos),
    situacao: definirSituacaoSobra(sobra, valorSugerido),
  };
}

async anual(usuarioId: string, ano: number) {
  const intervalo = intervaloDoAno(ano);
  const lancamentos = await this.prisma.lancamento.findMany({
    where: { usuarioId, data: { gte: intervalo.inicio, lt: intervalo.fim } },
    select: { tipo: true, valorEmCentavos: true, data: true },
  });
  const meses = agruparPorMes(lancamentos);
  const total = { entradas: 0, saidas: 0, sobra: 0 };
  for (const mes of meses) {
    total.entradas = total.entradas + mes.entradas;
    total.saidas = total.saidas + mes.saidas;
  }
  total.sobra = total.entradas - total.saidas;
  return { ano, meses, total };
}

async comparativoAnos(usuarioId: string, quantidade: number) {
  const anoAtual = new Date().getUTCFullYear();
  const resultado = [];
  for (let ano = anoAtual - quantidade + 1; ano <= anoAtual; ano++) {
    const resumo = await this.anual(usuarioId, ano);
    resultado.push({ ano, ...resumo.total });
  }
  return resultado;
}

async porCategoria(usuarioId: string, mes: number, ano: number, tipo: TipoLancamento) {
  const intervalo = intervaloDoMes(mes, ano);
  const somas = await this.prisma.lancamento.groupBy({
    by: ['categoriaId'],
    where: { usuarioId, tipo, data: { gte: intervalo.inicio, lt: intervalo.fim } },
    _sum: { valorEmCentavos: true },
    orderBy: { _sum: { valorEmCentavos: 'desc' } },
  });
  const categorias = await this.prisma.categoria.findMany({
    where: { id: { in: somas.map((soma) => soma.categoriaId) } },
  });
  const resultado = [];
  for (const soma of somas) {
    const categoria = categorias.find((item) => item.id === soma.categoriaId);
    if (categoria) {
      resultado.push({ categoria: mapearCategoria(categoria), totalEmCentavos: soma._sum.valorEmCentavos || 0 });
    }
  }
  return resultado;
}

// Sobra de cada mês desde "inicio" até o mês de "hoje", incluindo os dois
async sobrasMensaisDesde(usuarioId: string, inicio: Date, hoje: Date): Promise<number[]> {
  const primeiroDia = new Date(Date.UTC(inicio.getUTCFullYear(), inicio.getUTCMonth(), 1));
  const fim = new Date(Date.UTC(hoje.getUTCFullYear(), hoje.getUTCMonth() + 1, 1));
  const lancamentos = await this.prisma.lancamento.findMany({
    where: { usuarioId, data: { gte: primeiroDia, lt: fim } },
    select: { tipo: true, valorEmCentavos: true, data: true },
  });
  const sobrasPorChave = new Map<string, number>();
  for (const lancamento of lancamentos) {
    const chave = `${lancamento.data.getUTCFullYear()}-${lancamento.data.getUTCMonth()}`;
    let atual = sobrasPorChave.get(chave) || 0;
    if (lancamento.tipo === 'ENTRADA') {
      atual = atual + lancamento.valorEmCentavos;
    } else {
      atual = atual - lancamento.valorEmCentavos;
    }
    sobrasPorChave.set(chave, atual);
  }
  return Array.from(sobrasPorChave.values());
}
```

DTOs de filtro:
- `mes`: `@Type(() => Number)`, `@IsInt`, `@Min(1)` e `@Max(12)`.
- `ano`: `@Type(() => Number)`, `@IsInt` e `@Min(2000)`. No service, validar também `ano <= anoAtual + 1`; se passar, lançar 400.
- `quantidade`: `@IsOptional`, `@Type(() => Number)`, `@IsInt`, `@Min(1)` e `@Max(5)`, com padrão 3.
- `tipo`: `@IsEnum`.

Exportar `ResumosService` no módulo.

- [ ] **Passo 3: Rodar até passar e fazer commit**

```bash
git add backend
git commit -m "feat(resumos): rotas de resumo mensal, anual, por categoria e comparativo"
```

### Tarefa 10: Metas

**Arquivos:**
- Criar: `src/metas/*`
- Teste: `test/metas.e2e-spec.ts`

**Interfaces:**
- Consome: `ResumosService.sobrasMensaisDesde`, `calcularProgressoMeta`, `paraDataIso`
- Produz: `GET`, `PUT` e `DELETE /metas` no formato `Meta` do `openapi.yaml`

- [ ] **Passo 1: Teste e2e (que deve falhar)**

```ts
describe('Metas (e2e)', () => {
  let app: INestApplication;
  let tokenA: string;
  let tokenB: string;

  beforeAll(async () => {
    app = await criarAppTeste();
    tokenA = (await cadastrarUsuarioTeste(app, 'meta-a', '52998224725')).token;
    tokenB = (await cadastrarUsuarioTeste(app, 'meta-b', '11144477735')).token;
  });
  afterAll(async () => { await app.close(); });

  it('começa sem meta', async () => {
    const resposta = await request(app.getHttpServer()).get('/api/metas').set('Authorization', `Bearer ${tokenA}`);
    expect(resposta.status).toBe(200);
    expect(resposta.body).toBeNull();
  });

  it('salva meta com progresso e não aparece para outro usuário', async () => {
    const salva = await request(app.getHttpServer())
      .put('/api/metas')
      .set('Authorization', `Bearer ${tokenA}`)
      .send({ nome: 'Viagem', valorAlvoEmCentavos: 500000 });
    expect(salva.status).toBe(200);
    expect(salva.body.progresso).toBe(0);
    const deB = await request(app.getHttpServer()).get('/api/metas').set('Authorization', `Bearer ${tokenB}`);
    expect(deB.body).toBeNull();
  });

  it('recusa prazo no passado', async () => {
    const resposta = await request(app.getHttpServer())
      .put('/api/metas')
      .set('Authorization', `Bearer ${tokenA}`)
      .send({ nome: 'Viagem', valorAlvoEmCentavos: 500000, prazo: '2020-01-01' });
    expect(resposta.status).toBe(400);
  });

  it('remove a meta e devolve 404 se não houver', async () => {
    const primeira = await request(app.getHttpServer()).delete('/api/metas').set('Authorization', `Bearer ${tokenA}`);
    expect(primeira.status).toBe(204);
    const segunda = await request(app.getHttpServer()).delete('/api/metas').set('Authorization', `Bearer ${tokenA}`);
    expect(segunda.status).toBe(404);
  });
});
```

- [ ] **Passo 2: Implementar**

```ts
// metas.service.ts (trechos centrais)
async buscar(usuarioId: string) {
  const meta = await this.prisma.meta.findUnique({ where: { usuarioId } });
  if (!meta) {
    return null;
  }
  return this.montarResposta(meta);
}

async salvar(usuarioId: string, dados: SalvarMetaDto) {
  let prazo: Date | null = null;
  if (dados.prazo) {
    prazo = new Date(dados.prazo);
    if (prazo <= new Date()) {
      throw new ExcecaoApi(400, 'VALIDACAO_FALHOU', [{ campo: 'prazo', mensagem: MENSAGENS_CAMPO.prazoMeta }]);
    }
  }
  // Substituir a meta recomeça o progresso: o criadoEm volta a ser agora
  const meta = await this.prisma.meta.upsert({
    where: { usuarioId },
    create: { usuarioId, nome: dados.nome, valorAlvoEmCentavos: dados.valorAlvoEmCentavos, prazo },
    update: { nome: dados.nome, valorAlvoEmCentavos: dados.valorAlvoEmCentavos, prazo, criadoEm: new Date() },
  });
  return this.montarResposta(meta);
}

async remover(usuarioId: string) {
  const meta = await this.prisma.meta.findUnique({ where: { usuarioId } });
  if (!meta) {
    throw new ExcecaoApi(404, 'NAO_ENCONTRADO');
  }
  await this.prisma.meta.delete({ where: { usuarioId } });
}

private async montarResposta(meta: Meta) {
  const sobras = await this.resumosService.sobrasMensaisDesde(meta.usuarioId, meta.criadoEm, new Date());
  const calculo = calcularProgressoMeta(sobras, meta.valorAlvoEmCentavos);
  let prazo: string | null = null;
  if (meta.prazo) {
    prazo = paraDataIso(meta.prazo);
  }
  return {
    id: meta.id,
    nome: meta.nome,
    valorAlvoEmCentavos: meta.valorAlvoEmCentavos,
    prazo,
    valorAcumuladoEmCentavos: calculo.valorAcumulado,
    progresso: calculo.progresso,
    atingida: calculo.atingida,
    criadoEm: meta.criadoEm.toISOString(),
  };
}
```

`SalvarMetaDto`:
- `nome` com `limparEspacos`, `@SemHtml` e `@Length(2, 60)`, mensagem `nomeMeta`.
- `valorAlvoEmCentavos` com `@IsInt`, `@Min(1)` e `@Max(1000000000)`.
- `prazo` com `@IsOptional()` e `@IsDateString({ strict: true })`.

O `DELETE` usa `@HttpCode(204)`.

> `GET /metas` sem meta devolve `null`. Em alguns setups o Nest responde corpo vazio para `null`. Se o teste falhar por isso, faça o controller usar `@Res({ passthrough: true })` e chamar `res.json(null)`.

- [ ] **Passo 3: Rodar até passar e fazer commit**

```bash
git add backend
git commit -m "feat(metas): meta de economia opcional com progresso automático"
```

### Tarefa 11: QA da API (`qa`)

- [ ] **Passo 1:** Rodar `npm run test` e `npm run test:e2e`. Tudo precisa passar.
- [ ] **Passo 2:** Comparar `http://localhost:3000/api/docs-json` com `docs/swagger/openapi.yaml`: rotas, status, nomes de campos, exemplos e schemas de erro. Listar cada diferença.
- [ ] **Passo 3:** Tentar quebrar a API:
  - token de outro usuário em todas as rotas com `:id`;
  - `"; DROP TABLE` e `<img onerror>` em todos os campos de texto;
  - números negativos, decimais e strings em campos de valor;
  - `mes=0`, `ano=1999` e `pagina=-1`;
  - token adulterado e token expirado (com `JWT_EXPIRACAO_SEGUNDOS=1`).
- [ ] **Passo 4:** Entregar um relatório com Críticos, Importantes e Menores. Os Críticos e Importantes voltam para o `dev-backend` antes da Fase 3.

---

## Fase 3 — Front-end (`dev-frontend`)

### Tarefa 12: Projeto Angular, design system e tema

**Arquivos:**
- Criar: `frontend/` (scaffold), `src/styles/tokens.css`, `src/styles/tipografia.css`, `src/styles/bootstrap-personalizado.css`, `src/environments/*`, `src/app/nucleo/servicos/tema.service.ts`, `src/app/compartilhado/componentes/alternador-tema/*`
- Modificar: `src/index.html`, `src/styles.css`, `angular.json`, `src/app/app.config.ts`
- Teste: `src/app/nucleo/servicos/tema.service.spec.ts`

**Interfaces:**
- Produz:
  - `TemaService { temaAtual: Signal<'light' | 'dark'>; alternar(): void }`
  - componente `<app-alternador-tema />`
  - `environment.urlApi`

- [ ] **Passo 1: Criar o projeto e instalar as dependências**

```bash
npx @angular/cli@22 new frontend --style=css --routing --ssr=false --skip-git
cd frontend
npm i bootstrap gsap lucide-angular ngx-mask chart.js sweetalert2
```

No `angular.json`, adicionar `"node_modules/bootstrap/dist/css/bootstrap.min.css"` em `styles`, **antes** de `src/styles.css`.

- [ ] **Passo 2: `index.html`** (fontes e tema aplicado antes do Angular, sem flash)

```html
<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8">
  <title>Sobra</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
  <meta name="color-scheme" content="light dark">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Afacad:wght@400;500;700&family=Poppins:wght@400;500;600&display=swap" rel="stylesheet">
  <script>
    (function () {
      var tema = null;
      try { tema = localStorage.getItem('sobra-tema'); } catch (e) {}
      if (tema !== 'light' && tema !== 'dark') {
        tema = 'light';
        if (window.matchMedia('(prefers-color-scheme: dark)').matches) { tema = 'dark'; }
      }
      document.documentElement.setAttribute('data-bs-theme', tema);
    })();
  </script>
</head>
<body>
  <app-root></app-root>
</body>
</html>
```

- [ ] **Passo 3: `tokens.css`** (valores exatos da §12.1)

```css
:root,
[data-bs-theme="light"] {
  --cor-marinho: #041119;
  --cor-primaria: #2cac5c;
  --cor-primaria-hover: #249a51;
  --cor-primaria-suave: #d7efe0;
  --cor-fundo: #fbfcff;
  --cor-superficie: #ffffff;
  --cor-superficie-suave: #f5fbf8;
  --cor-texto: #041119;
  --cor-texto-suave: #5f6b72;
  --cor-borda: #e5ece8;
  --cor-perigo: #e8452c;
  --cor-alerta: #eba51b;
  --sombra-suave: 0 8px 24px rgb(4 17 25 / .06);
  --sombra-elevada: 0 16px 40px rgb(4 17 25 / .12);

  --raio-card: 16px;
  --raio-input: 12px;
  --raio-pilula: 999px;
  --espaco-1: 4px;  --espaco-2: 8px;  --espaco-3: 12px; --espaco-4: 16px;
  --espaco-5: 24px; --espaco-6: 32px; --espaco-7: 48px; --espaco-8: 64px;
  --fonte-titulo: 'Afacad', system-ui, sans-serif;
  --fonte-texto: 'Poppins', system-ui, sans-serif;
}

[data-bs-theme="dark"] {
  --cor-primaria: #34c46b;
  --cor-primaria-hover: #2cac5c;
  --cor-primaria-suave: #123826;
  --cor-fundo: #041119;
  --cor-superficie: #0b1d27;
  --cor-superficie-suave: #102631;
  --cor-texto: #f5fbf8;
  --cor-texto-suave: #9fb3ad;
  --cor-borda: #1c3440;
  --cor-perigo: #ff6b55;
  --cor-alerta: #f5b93a;
  --sombra-suave: 0 8px 24px rgb(0 0 0 / .35);
  --sombra-elevada: 0 16px 40px rgb(0 0 0 / .5);
}
```

- [ ] **Passo 4: `tipografia.css` e `bootstrap-personalizado.css`**

```css
/* tipografia.css */
body { font-family: var(--fonte-texto); font-size: 15px; color: var(--cor-texto); background: var(--cor-fundo); }
@media (min-width: 992px) { body { font-size: 16px; } }
h1, .h1 { font-family: var(--fonte-titulo); font-weight: 500; font-size: clamp(2.5rem, 6vw, 4rem); line-height: 1.05; }
h2, .h2 { font-family: var(--fonte-titulo); font-weight: 500; font-size: clamp(2rem, 4.5vw, 3rem); line-height: 1.1; }
h3, .h3 { font-family: var(--fonte-titulo); font-weight: 700; font-size: clamp(1.375rem, 2vw, 1.5rem); }
h4, .h4 { font-family: var(--fonte-titulo); font-weight: 400; font-size: clamp(1.125rem, 1.6vw, 1.25rem); }
h5, .h5 { font-family: var(--fonte-titulo); font-weight: 400; font-size: 1rem; }
.rotulo { font-size: 12px; font-weight: 500; text-transform: uppercase; letter-spacing: .06em; color: var(--cor-texto-suave); }
.valor-monetario { font-family: var(--fonte-titulo); font-weight: 500; font-variant-numeric: tabular-nums; }
```

```css
/* bootstrap-personalizado.css: liga o Bootstrap aos tokens */
:root, [data-bs-theme="light"], [data-bs-theme="dark"] {
  --bs-body-bg: var(--cor-fundo);
  --bs-body-color: var(--cor-texto);
  --bs-border-color: var(--cor-borda);
  --bs-border-radius: var(--raio-input);
  --bs-primary: var(--cor-primaria);
  --bs-danger: var(--cor-perigo);
  --bs-warning: var(--cor-alerta);
  --bs-font-sans-serif: var(--fonte-texto);
}

body { transition: background-color .3s ease, color .3s ease; }

.btn { border-radius: var(--raio-pilula); min-height: 44px; padding-inline: var(--espaco-5); font-weight: 500;
  transition: transform .2s ease, box-shadow .2s ease, background-color .2s ease; }
.btn-primario { background: var(--cor-primaria); color: #fff; border: 0; }
.btn-primario:hover { background: var(--cor-primaria-hover); color: #fff; transform: translateY(-1px); box-shadow: var(--sombra-suave); }
.btn-secundario { background: transparent; color: var(--cor-texto); border: 1px solid var(--cor-texto); }
.btn-secundario:hover { background: var(--cor-superficie-suave); color: var(--cor-texto); }
.btn-perigo { background: var(--cor-perigo); color: #fff; border: 0; }

.cartao { background: var(--cor-superficie); border: 1px solid var(--cor-borda); border-radius: var(--raio-card);
  box-shadow: var(--sombra-suave); padding: var(--espaco-5); }

.form-control, .form-select { min-height: 48px; background: var(--cor-superficie); border-color: var(--cor-borda);
  border-radius: var(--raio-input); color: var(--cor-texto); }
.form-control:focus, .form-select:focus { border-color: var(--cor-primaria); box-shadow: 0 0 0 3px var(--cor-primaria-suave); }
.form-control.is-invalid { border-color: var(--cor-perigo); }

:focus-visible { outline: 2px solid var(--cor-primaria); outline-offset: 2px; }

.badge-status { display: inline-flex; align-items: center; gap: var(--espaco-2); padding: 4px 12px;
  border-radius: var(--raio-pilula); border: 1px solid currentColor; font-size: 12px; font-weight: 500; }
.badge-status::before { content: ''; width: 8px; height: 8px; border-radius: 50%; background: currentColor; }
.badge-entrada { color: var(--cor-primaria); }
.badge-saida { color: var(--cor-perigo); }
.badge-sobra-META_ATINGIDA { color: var(--cor-texto); }
.badge-sobra-ABAIXO_DA_META { color: var(--cor-alerta); }
.badge-sobra-NEGATIVA { color: var(--cor-perigo); }

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { transition-duration: .01ms !important; animation-duration: .01ms !important; }
}
```

`src/styles.css`:
```css
@import './styles/tokens.css';
@import './styles/tipografia.css';
@import './styles/bootstrap-personalizado.css';
```

- [ ] **Passo 5: Teste do `TemaService` (que deve falhar)**

```ts
import { TestBed } from '@angular/core/testing';
import { TemaService } from './tema.service';

describe('TemaService', () => {
  beforeEach(() => {
    localStorage.clear();
    document.documentElement.setAttribute('data-bs-theme', 'light');
  });

  it('lê o tema já aplicado no html', () => {
    const servico = TestBed.inject(TemaService);
    expect(servico.temaAtual()).toBe('light');
  });

  it('alterna, aplica no html e salva a escolha', () => {
    const servico = TestBed.inject(TemaService);
    servico.alternar();
    expect(servico.temaAtual()).toBe('dark');
    expect(document.documentElement.getAttribute('data-bs-theme')).toBe('dark');
    expect(localStorage.getItem('sobra-tema')).toBe('dark');
  });
});
```

- [ ] **Passo 6: Implementar o `TemaService` e o `AlternadorTema`**

```ts
import { Injectable, signal } from '@angular/core';

export type Tema = 'light' | 'dark';
const CHAVE_TEMA = 'sobra-tema';

@Injectable({ providedIn: 'root' })
export class TemaService {
  // O script do index.html já escolheu o tema inicial. Aqui só lemos o que ele aplicou.
  readonly temaAtual = signal<Tema>(this.lerTemaAplicado());

  alternar() {
    let novoTema: Tema = 'dark';
    if (this.temaAtual() === 'dark') {
      novoTema = 'light';
    }
    this.aplicar(novoTema);
  }

  private aplicar(tema: Tema) {
    document.documentElement.setAttribute('data-bs-theme', tema);
    try {
      localStorage.setItem(CHAVE_TEMA, tema);
    } catch {
      // Sem localStorage o tema só não fica salvo
    }
    this.temaAtual.set(tema);
  }

  private lerTemaAplicado(): Tema {
    if (document.documentElement.getAttribute('data-bs-theme') === 'dark') {
      return 'dark';
    }
    return 'light';
  }
}
```

```ts
// alternador-tema.ts
import { Component, computed, inject } from '@angular/core';
import { LucideAngularModule, Moon, Sun } from 'lucide-angular';
import { TemaService } from '../../../nucleo/servicos/tema.service';

@Component({
  selector: 'app-alternador-tema',
  imports: [LucideAngularModule],
  template: `
    <button type="button" class="botao-tema" id="botao-alternar-tema"
            [attr.aria-label]="rotulo()" (click)="tema.alternar()">
      @if (tema.temaAtual() === 'dark') {
        <lucide-icon [img]="iconeSol" [size]="20" aria-hidden="true" />
      } @else {
        <lucide-icon [img]="iconeLua" [size]="20" aria-hidden="true" />
      }
    </button>
  `,
  styles: [`
    .botao-tema { width: 44px; height: 44px; border-radius: var(--raio-pilula); border: 1px solid var(--cor-borda);
      background: var(--cor-superficie); color: var(--cor-texto); display: grid; place-items: center;
      transition: transform .2s ease; }
    .botao-tema:hover { transform: rotate(-15deg); }
  `],
})
export class AlternadorTema {
  protected readonly tema = inject(TemaService);
  protected readonly iconeSol = Sun;
  protected readonly iconeLua = Moon;
  protected readonly rotulo = computed(() => {
    if (this.tema.temaAtual() === 'dark') {
      return 'Ativar modo claro';
    }
    return 'Ativar modo escuro';
  });
}
```

- [ ] **Passo 7: Rodar os testes e conferir manualmente**

Rodar: `ng test`, esperado PASSA. Depois, `ng serve`: com o sistema operacional no modo escuro e o `localStorage` limpo, a página abre escura e sem piscar.

- [ ] **Passo 8: Commit**

```bash
git add frontend
git commit -m "feat(front): design system, Bootstrap personalizado e tema claro/escuro"
```

### Tarefa 13: Núcleo do front (modelos, mensagens, autenticação, interceptors, guards, confirmação, notificação, validadores e pipes)

**Arquivos:**
- Criar: `nucleo/modelos.ts`, `nucleo/mensagens.ts`, `nucleo/servicos/*.service.ts`, `nucleo/interceptors/*`, `nucleo/guards/*`, `compartilhado/validadores/*`, `compartilhado/pipes/*`, `compartilhado/utilitarios/*`
- Modificar: `app.config.ts`
- Teste: `compartilhado/validadores/validadores.spec.ts`, `compartilhado/pipes/moeda-brl.pipe.spec.ts`, `compartilhado/utilitarios/dinheiro.spec.ts`

**Interfaces:**
- Produz:
  - `modelos.ts`: interfaces `Usuario`, `Categoria`, `Lancamento`, `PaginaLancamentos`, `ResumoMensal`, `ResumoAnual`, `TotalMes`, `TotalAno`, `TotalCategoria`, `Meta`, `RespostaAutenticacao`, `ErroApi`, além de `TipoLancamento` e `SituacaoSobra`. Os nomes e campos são **idênticos** aos schemas do `openapi.yaml`.
  - `MENSAGENS` (§10, incluindo sucesso e alertas) e `formatarMensagem(modelo: string, valores: Record<string, string>): string`
  - `AutenticacaoService { usuario: Signal<Usuario | null>; estaLogado(): boolean; cadastrar(dados): Observable<RespostaAutenticacao>; entrar(email, senha): Observable<RespostaAutenticacao>; sair(): void; obterToken(): string | null; atualizarUsuario(usuario: Usuario): void }`
  - `ConfirmacaoService.confirmar(opcoes: { titulo: string; mensagem: string; textoBotao: string; destrutiva?: boolean }): Promise<boolean>`
  - `NotificacaoService { sucesso(mensagem: string): void; erro(mensagem: string): void; aviso(mensagem: string): void }`
  - `tokenInterceptor`, `erroInterceptor` (`HttpInterceptorFn`)
  - `autenticadoGuard`, `visitanteGuard` (`CanActivateFn`)
  - Serviços HTTP: `LancamentosService`, `CategoriasService`, `ResumosService`, `MetasService` e `UsuariosService`, com um método por rota do `openapi.yaml`, todos retornando `Observable`
  - Validadores: `cpfValidador`, `senhaForteValidador`, `nomeValidador`, `telefoneValidador`, `idadeValidador(min, max)`, `senhasIguaisValidador(campoSenha, campoConfirmacao)`. Cada um devolve `{ [chave]: true }` ou `null`
  - `centavosParaReais(centavos: number): number` e `reaisParaCentavos(reais: number): number`
  - Pipes: `moedaBrl` (centavos → "R$ 1.234,56") e `idade` (data ISO → número)
  - `rotuloSituacao(situacao: SituacaoSobra): string`

- [ ] **Passo 1: Testes (que devem falhar)**

```ts
// validadores.spec.ts
import { FormControl, FormGroup } from '@angular/forms';
import { cpfValidador } from './cpf.validador';
import { senhaForteValidador } from './senha-forte.validador';
import { telefoneValidador } from './telefone.validador';
import { idadeValidador } from './idade.validador';
import { senhasIguaisValidador } from './senhas-iguais.validador';

describe('validadores', () => {
  it('cpf', () => {
    expect(cpfValidador(new FormControl('52998224725'))).toBeNull();
    expect(cpfValidador(new FormControl('52998224724'))).toEqual({ cpf: true });
  });
  it('senha forte', () => {
    expect(senhaForteValidador(new FormControl('Abc123!@'))).toBeNull();
    expect(senhaForteValidador(new FormControl('Abc12345'))).toEqual({ senhaForte: true });
  });
  it('telefone', () => {
    expect(telefoneValidador(new FormControl('11912345678'))).toBeNull();
    expect(telefoneValidador(new FormControl('11812345678'))).toEqual({ telefone: true });
  });
  it('idade a partir da data de nascimento', () => {
    expect(idadeValidador(16, 120)(new FormControl('1998-04-15'))).toBeNull();
    const ontem = new Date(Date.now() - 86400000).toISOString().substring(0, 10);
    expect(idadeValidador(16, 120)(new FormControl(ontem))).toEqual({ idade: true });
  });
  it('senhas iguais no grupo', () => {
    const grupo = new FormGroup({ senha: new FormControl('Abc123!@'), confirmacao: new FormControl('Abc123!#') });
    expect(senhasIguaisValidador('senha', 'confirmacao')(grupo)).toEqual({ senhasDiferentes: true });
  });
});
```

```ts
// moeda-brl.pipe.spec.ts
import { MoedaBrlPipe } from './moeda-brl.pipe';

describe('MoedaBrlPipe', () => {
  it('formata centavos em reais', () => {
    const pipe = new MoedaBrlPipe();
    expect(pipe.transform(123456).replace(/\s/g, ' ')).toBe('R$ 1.234,56');
    expect(pipe.transform(-5000).replace(/\s/g, ' ')).toBe('-R$ 50,00');
  });
});
```

```ts
// dinheiro.spec.ts
import { reaisParaCentavos } from './dinheiro';

describe('dinheiro', () => {
  it('converte reais em centavos sem erro de ponto flutuante', () => {
    expect(reaisParaCentavos(19.99)).toBe(1999);
    expect(reaisParaCentavos(0.1 + 0.2)).toBe(30);
  });
});
```

- [ ] **Passo 2: Implementar os validadores, os pipes e os utilitários**

O front tem a **sua própria cópia** de `validarCpf`, `REGEX_SENHA`, `REGEX_NOME` e `telefoneValido`, idênticas às da Tarefa 3, em `compartilhado/utilitarios/regras.ts`. Os dois projetos são separados, então a cópia é intencional.

```ts
// cpf.validador.ts
import { AbstractControl, ValidationErrors } from '@angular/forms';
import { validarCpf } from '../utilitarios/regras';

export function cpfValidador(controle: AbstractControl): ValidationErrors | null {
  const valor = controle.value;
  if (!valor) {
    return null; // o "obrigatório" é responsabilidade do Validators.required
  }
  if (validarCpf(String(valor))) {
    return null;
  }
  return { cpf: true };
}
```

```ts
// idade.validador.ts
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';
import { calcularIdade } from '../utilitarios/datas';

export function idadeValidador(minima: number, maxima: number): ValidatorFn {
  return (controle: AbstractControl): ValidationErrors | null => {
    if (!controle.value) {
      return null;
    }
    const nascimento = new Date(controle.value);
    if (isNaN(nascimento.getTime()) || nascimento > new Date()) {
      return { idade: true };
    }
    const idade = calcularIdade(nascimento, new Date());
    if (idade < minima || idade > maxima) {
      return { idade: true };
    }
    return null;
  };
}
```

```ts
// senhas-iguais.validador.ts
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

export function senhasIguaisValidador(campoSenha: string, campoConfirmacao: string): ValidatorFn {
  return (grupo: AbstractControl): ValidationErrors | null => {
    const senha = grupo.get(campoSenha)?.value;
    const confirmacao = grupo.get(campoConfirmacao)?.value;
    if (!confirmacao || senha === confirmacao) {
      return null;
    }
    return { senhasDiferentes: true };
  };
}
```

`senhaForteValidador`, `nomeValidador` e `telefoneValidador` seguem o mesmo molde do `cpfValidador`, usando `REGEX_SENHA`, `REGEX_NOME` e `telefoneValido`, com as chaves `senhaForte`, `nome` e `telefone`. `calcularIdade` é copiado da Tarefa 3 para `compartilhado/utilitarios/datas.ts`.

```ts
// dinheiro.ts
export function reaisParaCentavos(reais: number): number {
  return Math.round(reais * 100);
}

export function centavosParaReais(centavos: number): number {
  return centavos / 100;
}
```

```ts
// moeda-brl.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

const formatador = new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' });

@Pipe({ name: 'moedaBrl' })
export class MoedaBrlPipe implements PipeTransform {
  transform(centavos: number | null | undefined): string {
    if (centavos === null || centavos === undefined) {
      return formatador.format(0);
    }
    return formatador.format(centavos / 100);
  }
}
```

- [ ] **Passo 3: Autenticação, interceptors e guards**

```ts
// autenticacao.service.ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable, signal } from '@angular/core';
import { Router } from '@angular/router';
import { Observable, tap } from 'rxjs';
import { environment } from '../../../environments/environment';
import { RespostaAutenticacao, Usuario } from '../modelos';

const CHAVE_TOKEN = 'sobra-token';
const CHAVE_USUARIO = 'sobra-usuario';
const CHAVE_EXPIRACAO = 'sobra-expira-em';

@Injectable({ providedIn: 'root' })
export class AutenticacaoService {
  private readonly http = inject(HttpClient);
  private readonly roteador = inject(Router);
  readonly usuario = signal<Usuario | null>(this.lerUsuarioSalvo());

  cadastrar(dados: Record<string, unknown>): Observable<RespostaAutenticacao> {
    return this.http
      .post<RespostaAutenticacao>(`${environment.urlApi}/auth/cadastro`, dados)
      .pipe(tap((resposta) => this.salvarSessao(resposta)));
  }

  entrar(email: string, senha: string): Observable<RespostaAutenticacao> {
    return this.http
      .post<RespostaAutenticacao>(`${environment.urlApi}/auth/entrar`, { email, senha })
      .pipe(tap((resposta) => this.salvarSessao(resposta)));
  }

  sair() {
    localStorage.removeItem(CHAVE_TOKEN);
    localStorage.removeItem(CHAVE_USUARIO);
    localStorage.removeItem(CHAVE_EXPIRACAO);
    this.usuario.set(null);
    this.roteador.navigateByUrl('/entrar');
  }

  obterToken(): string | null {
    const expiraEm = Number(localStorage.getItem(CHAVE_EXPIRACAO));
    if (!expiraEm || Date.now() > expiraEm) {
      return null;
    }
    return localStorage.getItem(CHAVE_TOKEN);
  }

  estaLogado(): boolean {
    return this.obterToken() !== null;
  }

  atualizarUsuario(usuario: Usuario) {
    localStorage.setItem(CHAVE_USUARIO, JSON.stringify(usuario));
    this.usuario.set(usuario);
  }

  private salvarSessao(resposta: RespostaAutenticacao) {
    localStorage.setItem(CHAVE_TOKEN, resposta.tokenAcesso);
    localStorage.setItem(CHAVE_EXPIRACAO, String(Date.now() + resposta.expiraEm * 1000));
    this.atualizarUsuario(resposta.usuario);
  }

  private lerUsuarioSalvo(): Usuario | null {
    const texto = localStorage.getItem(CHAVE_USUARIO);
    if (!texto) {
      return null;
    }
    return JSON.parse(texto) as Usuario;
  }
}
```

```ts
// token.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AutenticacaoService } from '../servicos/autenticacao.service';

export const tokenInterceptor: HttpInterceptorFn = (requisicao, proximo) => {
  const token = inject(AutenticacaoService).obterToken();
  if (!token) {
    return proximo(requisicao);
  }
  return proximo(requisicao.clone({ setHeaders: { Authorization: `Bearer ${token}` } }));
};
```

```ts
// erro.interceptor.ts
import { HttpErrorResponse, HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { catchError, throwError } from 'rxjs';
import { AutenticacaoService } from '../servicos/autenticacao.service';
import { NotificacaoService } from '../servicos/notificacao.service';
import { MENSAGENS } from '../mensagens';

// Mostra o toast de erro. Os erros por campo ficam para o formulário tratar.
export const erroInterceptor: HttpInterceptorFn = (requisicao, proximo) => {
  const notificacao = inject(NotificacaoService);
  const autenticacao = inject(AutenticacaoService);
  return proximo(requisicao).pipe(
    catchError((erro: HttpErrorResponse) => {
      if (erro.status === 0) {
        notificacao.erro(MENSAGENS.erros.SEM_CONEXAO);
      } else if (erro.status === 401 && !requisicao.url.includes('/auth/')) {
        notificacao.erro(MENSAGENS.erros.SESSAO_EXPIRADA);
        autenticacao.sair();
      } else if (erro.error && erro.error.mensagem) {
        notificacao.erro(erro.error.mensagem);
      } else {
        notificacao.erro(MENSAGENS.erros.ERRO_INTERNO);
      }
      return throwError(() => erro);
    }),
  );
};
```

```ts
// autenticado.guard.ts e visitante.guard.ts
export const autenticadoGuard: CanActivateFn = () => {
  if (inject(AutenticacaoService).estaLogado()) {
    return true;
  }
  return inject(Router).parseUrl('/entrar');
};

export const visitanteGuard: CanActivateFn = () => {
  if (inject(AutenticacaoService).estaLogado()) {
    return inject(Router).parseUrl('/painel');
  }
  return true;
};
```

- [ ] **Passo 4: Confirmação e notificação (Facade do SweetAlert2)**

```ts
// confirmacao.service.ts
import { Injectable } from '@angular/core';
import Swal from 'sweetalert2';

export interface OpcoesConfirmacao {
  titulo: string;
  mensagem: string;
  textoBotao: string;
  destrutiva?: boolean;
}

@Injectable({ providedIn: 'root' })
export class ConfirmacaoService {
  // Abre o pop-up e responde true só se a pessoa clicar no botão de ação
  async confirmar(opcoes: OpcoesConfirmacao): Promise<boolean> {
    let classeBotao = 'btn btn-primario';
    let focarCancelar = false;
    if (opcoes.destrutiva) {
      classeBotao = 'btn btn-perigo';
      focarCancelar = true;
    }
    const resultado = await Swal.fire({
      title: opcoes.titulo,
      text: opcoes.mensagem, // "text" escapa HTML; nunca usar "html" com dados do usuário
      showCancelButton: true,
      confirmButtonText: opcoes.textoBotao,
      cancelButtonText: 'Cancelar',
      reverseButtons: true,
      focusCancel: focarCancelar,
      buttonsStyling: false,
      customClass: {
        popup: 'sobra-modal',
        title: 'sobra-modal-titulo',
        htmlContainer: 'sobra-modal-texto',
        confirmButton: classeBotao,
        cancelButton: 'btn btn-secundario',
        actions: 'sobra-modal-acoes',
      },
    });
    return resultado.isConfirmed;
  }
}
```

```ts
// notificacao.service.ts
import { Injectable } from '@angular/core';
import Swal, { SweetAlertIcon } from 'sweetalert2';

@Injectable({ providedIn: 'root' })
export class NotificacaoService {
  sucesso(mensagem: string) { this.mostrar('success', mensagem); }
  erro(mensagem: string) { this.mostrar('error', mensagem); }
  aviso(mensagem: string) { this.mostrar('warning', mensagem); }

  private mostrar(icone: SweetAlertIcon, mensagem: string) {
    let posicao: 'top' | 'top-end' = 'top';
    if (window.matchMedia('(min-width: 992px)').matches) {
      posicao = 'top-end';
    }
    Swal.fire({
      toast: true,
      position: posicao,
      icon: icone,
      text: mensagem,
      showConfirmButton: false,
      timer: 4000,
      timerProgressBar: true,
      customClass: { popup: 'sobra-toast' },
      didOpen: (toast) => {
        toast.addEventListener('mouseenter', Swal.stopTimer);
        toast.addEventListener('mouseleave', Swal.resumeTimer);
      },
    });
  }
}
```

Adicionar ao `bootstrap-personalizado.css`:
```css
.sobra-modal { background: var(--cor-superficie); color: var(--cor-texto); border-radius: var(--raio-card);
  box-shadow: var(--sombra-elevada); font-family: var(--fonte-texto); }
.sobra-modal-titulo { font-family: var(--fonte-titulo); font-weight: 700; font-size: 1.5rem; color: var(--cor-texto); }
.sobra-modal-texto { color: var(--cor-texto-suave); }
.sobra-modal-acoes { gap: var(--espaco-3); }
.sobra-toast { background: var(--cor-superficie) !important; color: var(--cor-texto) !important;
  border-radius: var(--raio-card) !important; box-shadow: var(--sombra-elevada) !important; }
```

- [ ] **Passo 5: `app.config.ts`**

```ts
import { ApplicationConfig, LOCALE_ID } from '@angular/core';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideRouter, withComponentInputBinding } from '@angular/router';
import { registerLocaleData } from '@angular/common';
import localePt from '@angular/common/locales/pt';
import { provideEnvironmentNgxMask } from 'ngx-mask';
import { rotas } from './app.routes';
import { tokenInterceptor } from './nucleo/interceptors/token.interceptor';
import { erroInterceptor } from './nucleo/interceptors/erro.interceptor';

registerLocaleData(localePt);

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(rotas, withComponentInputBinding()),
    provideHttpClient(withInterceptors([tokenInterceptor, erroInterceptor])),
    provideEnvironmentNgxMask({ dropSpecialCharacters: true, thousandSeparator: '.', decimalMarker: ',' }),
    { provide: LOCALE_ID, useValue: 'pt-BR' },
  ],
};
```

`app.routes.ts` segue a tabela da §13.7, com `loadComponent` e `title: 'Sobra · Painel'` etc. As rotas autenticadas ficam como filhas do componente `Casca` (Tarefa 14).

- [ ] **Passo 6: Rodar até passar e fazer commit**

Rodar: `ng test`, esperado PASSA.

```bash
git add frontend
git commit -m "feat(front): núcleo de autenticação, interceptors, confirmação, toasts, validadores e pipes"
```

### Tarefa 14: Casca de layout e componentes compartilhados

**Arquivos:**
- Criar: `layout/casca/*`, `compartilhado/componentes/{cartao-resumo,anel-progresso,seletor-mes,estado-vazio,campo-erro,requisitos-senha,toggle-segmentado}/*`

**Interfaces:**
- Produz:
  - `<app-cartao-resumo rotulo icone [valorEmCentavos] tipo="entrada|saida|sobra" [situacao] />`: anima a contagem do número com GSAP.
  - `<app-anel-progresso [percentual] rotulo />`: SVG com `stroke-dashoffset` animado por GSAP.
  - `<app-seletor-mes [mes] [ano] (mudou)="..." />`: emite `{ mes, ano }`. Rótulo "Outubro de 2026", botões com `aria-label` "Mês anterior" e "Próximo mês".
  - `<app-estado-vazio icone titulo texto textoBotao (acao)="..." />`
  - `<app-campo-erro [controle] [mensagens] idCampo />`: mostra a primeira mensagem quando o campo é `touched && invalid`, com `id="{idCampo}-erro"`, para ser usado no `aria-describedby`.
  - `<app-requisitos-senha [senha] />`: lista de 4 regras com ✓/✗ e `aria-live="polite"`.
  - `<app-toggle-segmentado [opcoes]="[{valor, rotulo}]" [valor] (mudou)="..." />`: `role="tablist"`. O indicador desliza com GSAP.

- [ ] **Passo 1: `Casca`**
  - Mobile: `header.casca-cabecalho` com o logo e `<app-alternador-tema/>`, `main#conteudo-principal` com o `<router-outlet/>`, e `nav.navegacao-inferior` fixa com 3 links (`routerLinkActive="ativo"`, ícones `layout-dashboard`, `chart-column`, `user`). A navegação inferior usa `padding-bottom: env(safe-area-inset-bottom)`.
  - ≥ lg: `aside.barra-lateral` (fundo `--cor-marinho`, texto `#f5fbf8`, 248px, fixa) com logo, links, `<app-alternador-tema/>` e o botão `#botao-sair` no rodapé. A navegação inferior fica oculta. No modo escuro, a barra lateral usa `--cor-superficie-suave` com borda `--cor-borda`.
  - Um link "Pular para o conteúdo" no topo, visível só no foco.
  - `#botao-sair` chama `confirmacao.confirmar({ titulo: 'Sair da conta?', mensagem: 'Você precisará entrar de novo para ver suas finanças.', textoBotao: 'Sair' })`. Se confirmar, chama `autenticacao.sair()` e mostra o toast "Você saiu da sua conta."

- [ ] **Passo 2: Animação de contagem** (padrão para todos os números animados)

```ts
// dentro de CartaoResumo
private readonly elemento = viewChild.required<ElementRef<HTMLElement>>('valor');
private contexto?: gsap.Context;

constructor() {
  effect(() => {
    const centavosFinal = this.valorEmCentavos();
    const alvo = this.elemento().nativeElement;
    this.contexto?.revert();
    this.contexto = gsap.context(() => {
      const contador = { valor: 0 };
      const midia = gsap.matchMedia();
      midia.add('(prefers-reduced-motion: no-preference)', () => {
        gsap.to(contador, {
          valor: centavosFinal,
          duration: 0.8,
          ease: 'power2.out',
          onUpdate: () => { alvo.textContent = formatarBrl(Math.round(contador.valor)); },
        });
      });
      midia.add('(prefers-reduced-motion: reduce)', () => {
        alvo.textContent = formatarBrl(centavosFinal);
      });
    });
  });
}

ngOnDestroy() {
  this.contexto?.revert();
}
```

- [ ] **Passo 3:** Conferir a casca nas larguras 375, 768 e 1280px, nos dois temas, e fazer commit.

```bash
git add frontend
git commit -m "feat(front): casca responsiva e componentes compartilhados"
```

### Tarefa 15: Telas Entrar e Cadastro

**Arquivos:**
- Criar: `paginas/entrar/*`, `paginas/cadastro/*`

**Interfaces:**
- Consome: `AutenticacaoService`, os validadores, `<app-campo-erro>`, `<app-requisitos-senha>`, `NgxMaskDirective`, `MENSAGENS`, `NotificacaoService`

- [ ] **Passo 1: Entrar** (§13.1)
  - `form#formulario-entrar` com `input#campo-email` (`type="email"`, `autocomplete="email"`) e `input#campo-senha` (`autocomplete="current-password"`).
  - `button#botao-mostrar-senha` com `aria-label` "Mostrar senha" ou "Ocultar senha" e ícones `eye`/`eye-off`.
  - `button#botao-entrar` com o estado "enviando" (signal), que desabilita o botão e mostra um spinner.
  - Link para `/cadastro`.
  - ≥ lg: `section.painel-marca` à esquerda, com o slogan e 7 barras em pílula (divs) animadas com GSAP (`scaleY` de 0 até a altura, `stagger: .08`, `transformOrigin: 'bottom'`).
  - Em caso de sucesso: toast `formatarMensagem(MENSAGENS.sucesso.login, { primeiroNome })` e navegação para `/painel`.

- [ ] **Passo 2: Cadastro** (§13.2), com o formulário em 2 passos e um signal `passoAtual = signal<1 | 2>(1)`

```ts
protected readonly formulario = new FormGroup({
  dadosPessoais: new FormGroup({
    nome: new FormControl('', { nonNullable: true, validators: [Validators.required, nomeValidador] }),
    dataNascimento: new FormControl('', { nonNullable: true, validators: [Validators.required, idadeValidador(16, 120)] }),
    email: new FormControl('', { nonNullable: true, validators: [Validators.required, Validators.pattern(REGEX_EMAIL), Validators.maxLength(150)] }),
    senha: new FormControl('', { nonNullable: true, validators: [Validators.required, senhaForteValidador] }),
    confirmacaoSenha: new FormControl('', { nonNullable: true, validators: [Validators.required] }),
  }, { validators: senhasIguaisValidador('senha', 'confirmacaoSenha') }),
  contatoRenda: new FormGroup({
    telefone: new FormControl('', { nonNullable: true, validators: [Validators.required, telefoneValidador] }),
    cpf: new FormControl('', { nonNullable: true, validators: [Validators.required, cpfValidador] }),
    salario: new FormControl<number | null>(null, { validators: [Validators.required, Validators.min(0.01)] }),
  }),
});

protected avancar() {
  const grupo = this.formulario.controls.dadosPessoais;
  if (grupo.invalid) {
    grupo.markAllAsTouched();
    return;
  }
  this.passoAtual.set(2);
}

protected enviar() {
  if (this.formulario.invalid) {
    this.formulario.markAllAsTouched();
    return;
  }
  const pessoais = this.formulario.controls.dadosPessoais.getRawValue();
  const contato = this.formulario.controls.contatoRenda.getRawValue();
  this.enviando.set(true);
  this.autenticacao.cadastrar({
    nome: pessoais.nome,
    dataNascimento: pessoais.dataNascimento,
    email: pessoais.email,
    senha: pessoais.senha,
    telefone: contato.telefone,
    cpf: contato.cpf,
    salarioEmCentavos: reaisParaCentavos(Number(contato.salario)),
  }).subscribe({
    next: (resposta) => {
      this.notificacao.sucesso(formatarMensagem(MENSAGENS.sucesso.cadastro, { primeiroNome: resposta.usuario.nome.split(' ')[0] }));
      this.roteador.navigateByUrl('/painel');
    },
    error: (erro: HttpErrorResponse) => {
      this.enviando.set(false);
      this.aplicarErrosDaApi(erro); // erro.error.erros[] vira setErrors({ api: mensagem }) em cada campo
    },
  });
}
```

Máscaras:
- `input#campo-cpf` com `mask="000.000.000-00"` e `inputmode="numeric"`.
- `input#campo-telefone` com `mask="(00) 0000-0000||(00) 00000-0000"`, `inputmode="numeric"` e `autocomplete="tel"`.
- `input#campo-salario` com `mask="separator.2"`, `prefix="R$ "` e `inputmode="decimal"`.

O texto de ajuda do salário mostra `formatarMensagem('Sugerimos guardar 30%: {valor}', ...)` com um `computed` sobre o valor. O sucesso do cadastro aparece **depois** do clique: o cadastro não pede confirmação, já que não altera nenhum dado existente.

- [ ] **Passo 3: Teste manual**
  - Cada campo inválido mostra a mensagem exata da §10.1 ao perder o foco.
  - O CPF `529.982.247-25` passa e o `529.982.247-24` falha.
  - O corpo enviado (aba Network) tem só dígitos no CPF e no telefone, e o salário em centavos.
  - Um e-mail duplicado mostra o toast "Este e-mail já está cadastrado...".
  - A 6ª tentativa de login seguida mostra "Muitas tentativas...".

- [ ] **Passo 4: Commit**

```bash
git add frontend
git commit -m "feat(front): telas de entrar e cadastro com máscaras e validações"
```

### Tarefa 16: Painel

**Arquivos:**
- Criar: `paginas/painel/painel.ts|html|css`, `paginas/painel/formulario-lancamento/*`, `paginas/painel/formulario-meta/*`, `paginas/painel/lista-lancamentos/*`

**Interfaces:**
- Consome: `ResumosService.mensal`, `LancamentosService`, `CategoriasService.listar`, `MetasService`, `ConfirmacaoService`, `NotificacaoService` e os componentes da Tarefa 14

- [ ] **Passo 1: Estado da página com signals**

```ts
protected readonly mesSelecionado = signal(new Date().getMonth() + 1);
protected readonly anoSelecionado = signal(new Date().getFullYear());
protected readonly tipoVisivel = signal<TipoLancamento>('SAIDA');
protected readonly resumo = signal<ResumoMensal | null>(null);
protected readonly pagina = signal<PaginaLancamentos | null>(null);
protected readonly meta = signal<Meta | null>(null);
protected readonly estado = signal<'carregando' | 'pronto' | 'erro'>('carregando');

// Recarrega tudo sempre que o mês ou o toggle mudam (padrão Observer com signals)
constructor() {
  effect(() => {
    this.carregar(this.mesSelecionado(), this.anoSelecionado(), this.tipoVisivel());
  });
}
```

- [ ] **Passo 2: Template** (IDs que serão usados no JS e nos testes)
  - `#seletor-mes-painel`
  - `section#resumo-mes` com 3 `<app-cartao-resumo>`: `#cartao-entradas` (badge `badge-entrada`), `#cartao-saidas` (`badge-saida`) e `#cartao-sobra` (`badge-sobra-{situacao}`, com o texto de `rotuloSituacao`)
  - `#cartao-guarde-30`: valor sugerido, sobra e barra `progress` com `aria-valuenow`, mais o alerta da §10.4
  - `#cartao-meta`: `<app-anel-progresso>` ou `<app-estado-vazio>` com `#botao-criar-meta`, e os botões `#botao-editar-meta` e `#botao-remover-meta`
  - `<app-toggle-segmentado id="toggle-tipo">` com as opções Entradas e Saídas
  - `<app-lista-lancamentos>`: tabela `table#tabela-lancamentos` a partir de `md` e lista de cards `ul#lista-lancamentos-mobile` abaixo dela. Cada item tem `[attr.data-id]`, `button.botao-editar` e `button.botao-excluir` com `aria-label` "Editar {descricao}" e "Excluir {descricao}"
  - `button#botao-novo-lancamento`: no topo a partir de `lg` e flutuante (`position: fixed`, acima da navegação inferior) no mobile
  - `button#botao-carregar-mais` quando `pagina < totalPaginas`
  - Estados: skeleton em cada cartão enquanto carrega, `<app-estado-vazio>` na lista vazia ("Nenhuma saída em outubro. Que tal registrar a primeira?") e o bloco de erro com `#botao-tentar-novamente`

- [ ] **Passo 3: Formulário de lançamento**

O formulário abre em um modal do Bootstrap (componente próprio com `dialog`, prendendo o foco). Ele não é um modal do SweetAlert2, porque o SweetAlert2 fica reservado para as confirmações.

Campos:
- `#campo-tipo` (radio segmentado)
- `#campo-descricao`
- `#campo-valor` (máscara de moeda)
- `#campo-data` (padrão: hoje)
- `#campo-categoria` (`select`, filtrado pelo tipo)

Fluxo de salvar:

```ts
protected async salvar() {
  if (this.formulario.invalid) {
    this.formulario.markAllAsTouched();
    return;
  }
  const dados = this.montarDados();
  let confirmado = false;
  if (this.lancamentoEmEdicao()) {
    confirmado = await this.confirmacao.confirmar({
      titulo: 'Salvar alterações?',
      mensagem: `O lançamento "${dados.descricao}" será atualizado.`,
      textoBotao: 'Salvar alterações',
    });
  } else {
    let nomeTipo = 'saída';
    if (dados.tipo === 'ENTRADA') { nomeTipo = 'entrada'; }
    confirmado = await this.confirmacao.confirmar({
      titulo: `Registrar ${nomeTipo}?`,
      mensagem: `${dados.descricao} · ${this.moeda.transform(dados.valorEmCentavos)} em ${this.nomeCategoria(dados.categoriaId)} no dia ${this.formatarData(dados.data)}.`,
      textoBotao: 'Registrar',
    });
  }
  if (!confirmado) {
    return;
  }
  this.enviar(dados);
}
```

Depois de salvar: fechar o modal, recarregar o resumo e a lista e mostrar o toast da §10.3. Para "Criar saída", o toast usa o `totalSaidas` do resumo **recarregado**, porque esse total é a soma automática do mês. Fechar o modal com alterações não salvas pede a confirmação "Descartar alterações?".

- [ ] **Passo 4: Excluir, meta e salário**
  - **Excluir:** confirmação destrutiva da §11, chamada da API, toast e recarga.
  - **Meta:** criar, editar, substituir e remover com as confirmações da §11. Se `meta.atingida` mudar de `false` para `true` depois de uma recarga, mostrar o toast "Parabéns! Você atingiu a meta..." e uma animação curta no anel (escala 1 → 1.06 → 1).
- [ ] **Passo 5: Animações**
  - Na entrada da página, os cartões aparecem com `gsap.from('.cartao', { y: 16, opacity: 0, stagger: .06, ease: 'power2.out' })` dentro de `gsap.context` e `matchMedia`, com `revert()` no `ngOnDestroy`.
  - Na troca do toggle, a lista faz um fade de 0,2s.
- [ ] **Passo 6: Teste manual**
  - Criar 3 saídas e ver o total de saídas e a sobra mudarem sozinhos.
  - Com sobra < 0 a badge fica vermelha, com sobra < 30% fica âmbar e com sobra ≥ 30% fica marinho.
  - Cancelar qualquer confirmação não altera nada.
- [ ] **Passo 7: Commit**

```bash
git add frontend
git commit -m "feat(front): painel com resumo do mês, toggle de tabelas, lançamentos e meta"
```

### Tarefa 17: Gráficos (Chart.js) e Relatórios

**Arquivos:**
- Criar: `compartilhado/componentes/grafico-barras/*`, `paginas/relatorios/*`

**Interfaces:**
- Produz: `<app-grafico-barras [rotulos]="string[]" [series]="SerieGrafico[]" [indiceDestaque]="number | null" [horizontal]="boolean" titulo descricaoAcessivel />`, com `interface SerieGrafico { nome: string; valores: number[]; cor: 'primaria' | 'marinho' | 'perigo' }`. Os valores chegam em centavos e o tooltip formata em BRL.

- [ ] **Passo 1: Componente de gráfico com o tema dos tokens**

```ts
import { AfterViewInit, Component, ElementRef, effect, inject, input, OnDestroy, viewChild } from '@angular/core';
import { BarController, BarElement, CategoryScale, Chart, Legend, LinearScale, Tooltip } from 'chart.js';
import { TemaService } from '../../../nucleo/servicos/tema.service';

Chart.register(BarController, BarElement, CategoryScale, LinearScale, Tooltip, Legend);

function lerToken(nome: string): string {
  return getComputedStyle(document.documentElement).getPropertyValue(nome).trim();
}

@Component({
  selector: 'app-grafico-barras',
  template: `
    <figure class="grafico">
      <figcaption class="h4">{{ titulo() }}</figcaption>
      <div class="grafico-area">
        <canvas #tela role="img" [attr.aria-label]="descricaoAcessivel()"></canvas>
      </div>
      <table class="visually-hidden">
        <caption>{{ titulo() }}</caption>
        <thead><tr><th scope="col">Período</th>@for (serie of series(); track serie.nome) {<th scope="col">{{ serie.nome }}</th>}</tr></thead>
        <tbody>
          @for (rotulo of rotulos(); track rotulo; let indice = $index) {
            <tr><th scope="row">{{ rotulo }}</th>@for (serie of series(); track serie.nome) {<td>{{ serie.valores[indice] / 100 | currency:'BRL' }}</td>}</tr>
          }
        </tbody>
      </table>
    </figure>
  `,
  imports: [CurrencyPipe],
  styles: [`.grafico-area { position: relative; height: 280px; } @media (min-width: 992px) { .grafico-area { height: 340px; } }`],
})
export class GraficoBarras implements AfterViewInit, OnDestroy {
  readonly titulo = input.required<string>();
  readonly descricaoAcessivel = input.required<string>();
  readonly rotulos = input.required<string[]>();
  readonly series = input.required<SerieGrafico[]>();
  readonly indiceDestaque = input<number | null>(null);
  readonly horizontal = input(false);

  private readonly tela = viewChild.required<ElementRef<HTMLCanvasElement>>('tela');
  private readonly tema = inject(TemaService);
  private grafico?: Chart;

  constructor() {
    // Redesenha quando os dados ou o tema mudam
    effect(() => {
      this.tema.temaAtual();
      this.series();
      this.rotulos();
      if (this.grafico) {
        this.desenhar();
      }
    });
  }

  ngAfterViewInit() { this.desenhar(); }
  ngOnDestroy() { this.grafico?.destroy(); }

  private corDaSerie(serie: SerieGrafico, indice: number): string {
    if (this.indiceDestaque() === indice && this.series().length === 1) {
      return lerToken('--cor-primaria');
    }
    if (serie.cor === 'primaria') { return lerToken('--cor-primaria'); }
    if (serie.cor === 'perigo') { return lerToken('--cor-perigo'); }
    if (this.tema.temaAtual() === 'dark') { return lerToken('--cor-texto'); }
    return lerToken('--cor-marinho');
  }

  private desenhar() {
    this.grafico?.destroy();
    const corTexto = lerToken('--cor-texto-suave');
    const corBorda = lerToken('--cor-borda');
    let eixoIndice: 'x' | 'y' = 'x';
    if (this.horizontal()) { eixoIndice = 'y'; }

    this.grafico = new Chart(this.tela().nativeElement, {
      type: 'bar',
      data: {
        labels: this.rotulos(),
        datasets: this.series().map((serie) => ({
          label: serie.nome,
          data: serie.valores.map((valor) => valor / 100),
          backgroundColor: serie.valores.map((_valor, indice) => this.corDaSerie(serie, indice)),
          borderRadius: 999,
          borderSkipped: false,
          maxBarThickness: 28,
        })),
      },
      options: {
        indexAxis: eixoIndice,
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { labels: { color: corTexto, font: { family: 'Poppins', size: 12 }, usePointStyle: true } },
          tooltip: {
            backgroundColor: lerToken('--cor-marinho'),
            cornerRadius: 999,
            padding: { x: 12, y: 6 },
            callbacks: {
              label: (contexto) => `${contexto.dataset.label}: ${Number(contexto.raw).toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' })}`,
            },
          },
        },
        scales: {
          x: { grid: { display: false }, ticks: { color: corTexto, font: { family: 'Poppins', size: 12 } }, border: { display: false } },
          y: { grid: { color: corBorda }, ticks: { color: corTexto, font: { family: 'Poppins', size: 12 } }, border: { display: false } },
        },
      },
    });
  }
}
```

> **Trilho das barras** (o "fundo" verde-claro da referência): nos gráficos de série única, adicione um dataset extra, com valor igual ao máximo da série em todas as posições, cor `--cor-primaria-suave`, `grouped: false` e `order: 2`. O dataset real fica com `order: 1`. Esse dataset não aparece na legenda (`legend.labels.filter`) nem no tooltip (`tooltip.filter`).

- [ ] **Passo 2: Página Relatórios** (§13.4)
  - Seletor de ano `#seletor-ano` e toggle `#toggle-periodo` (Mensal | Anual).
  - **Mensal:**
    - `#grafico-entradas-saidas-mes`: 12 meses do ano, séries Entradas (`primaria`) e Saídas (`marinho`), com o mês atual em destaque.
    - `#seletor-mes-categorias` com `#grafico-saidas-categoria`: barras horizontais, série única, vinda de `/resumos/mensal/categorias`.
  - **Anual:**
    - `table#tabela-fechamento-anual`: 12 linhas com mês, entradas, saídas e sobra (com badge de situação), mais um `tfoot` com o total.
    - `#grafico-comparativo-anos`: últimos 3 anos, entradas × saídas.
    - `#grafico-saidas-ano-atual-anterior`: duas chamadas a `/resumos/anual` (ano e ano − 1), séries "Saídas {ano}" (`marinho`) e "Saídas {ano − 1}" (`primaria`).
  - Para a situação da sobra na tabela anual, o front usa `definirSituacaoSobra` (cópia da Tarefa 8 em `compartilhado/utilitarios/situacao-sobra.ts`) com 30% do salário do usuário logado.
  - Estados de carregando, vazio e erro em cada bloco.
- [ ] **Passo 3: Teste manual**
  - Trocar o tema com um gráfico na tela: as cores mudam na hora.
  - Leitor de tela (NVDA): cada gráfico anuncia o `aria-label` e a tabela equivalente.
- [ ] **Passo 4: Commit**

```bash
git add frontend
git commit -m "feat(front): relatórios mensal e anual com gráficos Chart.js no tema do design system"
```

### Tarefa 18: Perfil

**Arquivos:**
- Criar: `paginas/perfil/*`

- [ ] **Passo 1: Formulário `#formulario-perfil`**
  - Campos: `#campo-nome`, `#campo-data-nascimento` (com o texto "Você tem {idade} anos", usando o pipe `idade`), `#campo-telefone` (máscara) e `#campo-salario` (máscara de moeda), com a prévia "Nova sugestão de guardar: {valor30}".
  - Ao salvar:
    - Se o salário mudou, usar a confirmação "Atualizar salário?" da §11 e o toast "Salário atualizado. Sua sugestão de guardar agora é {valor30}."
    - Senão, usar a confirmação "Salvar seus dados?" e o toast "Dados atualizados."
    - Em seguida, `autenticacao.atualizarUsuario(resposta)`.
  - Somente leitura: o e-mail e o CPF mascarado, com a função `mascararCpf('52998224725')` retornando `***.982.247-**`.

```ts
export function mascararCpf(cpf: string): string {
  return `***.${cpf.substring(3, 6)}.${cpf.substring(6, 9)}-**`;
}
```

- [ ] **Passo 2: Seção `#minhas-categorias`**
  - Duas listas (Entradas e Saídas) só com as categorias próprias (`padrao === false`).
  - Formulário `#formulario-categoria` com nome, tipo e ícone. O ícone vem de uma lista fixa: `paw-print`, `shopping-bag`, `gift`, `plane`, `dumbbell`, `baby`, `car`, `smartphone`, `coffee`, `piggy-bank`.
  - Criar e excluir passam pelas confirmações da §11. Ao excluir uma categoria em uso, a API devolve 409 e o toast mostra a mensagem dela.
- [ ] **Passo 3: Botão `#botao-sair-perfil`**, com o mesmo fluxo de sair da casca.
- [ ] **Passo 4: Commit**

```bash
git add frontend
git commit -m "feat(front): perfil com edição de dados, salário e categorias próprias"
```

---

## Fase 4 — Qualidade e documentação

### Tarefa 19: QA de ponta a ponta (`qa`)

- [ ] **Passo 1:** Rodar `npm run test` e `npm run test:e2e` no back e `ng test` no front. Tudo precisa passar.
- [ ] **Passo 2:** Executar o roteiro manual inteiro da §17.4.
- [ ] **Passo 3:** Rodar uma auditoria com `/web-design-guidelines` e `/impecable-frontend` em todas as telas. Conferir:
  - contraste AA nos dois temas;
  - foco visível;
  - navegação por teclado;
  - `aria-*` corretos;
  - alvos de toque ≥ 44px;
  - nenhuma rolagem horizontal em 320px.
- [ ] **Passo 4:** Procurar violações das regras globais no código:

```bash
grep -rnE "\?[^.?:]*:" backend/src frontend/src/app --include=*.ts
grep -rn "innerHTML\|bypassSecurityTrust\|queryRawUnsafe\|executeRawUnsafe\|html:" backend/src frontend/src/app
grep -rn ": any\|as any" backend/src frontend/src/app
```
Cada resultado precisa ser revisado. Um ternário ou um `any` é defeito. O optional chaining (`?.`) e o operador `??` não contam como ternário.

- [ ] **Passo 5:** Entregar o relatório de Críticos, Importantes e Menores. Os Críticos e Importantes voltam para o agente dono da área.

### Tarefa 20: Documentação (`documentation-owner`)

- [ ] **Passo 1:** Garantir que `/api/docs` tem todas as rotas, com `summary`, exemplos e respostas de erro iguais ao `openapi.yaml`. O botão "Authorize" precisa funcionar.
- [ ] **Passo 2:** Escrever o `README.md` na raiz com:
  - pré-requisitos (Node LTS, PostgreSQL);
  - como criar o banco;
  - como copiar `.env.exemplo` para `.env`;
  - `npx prisma migrate dev`, `npx prisma db seed`, `npm run start:dev`, `ng serve`;
  - como rodar os testes;
  - o link do Swagger;
  - a estrutura de pastas resumida.
- [ ] **Passo 3: Commit**

```bash
git add README.md backend frontend
git commit -m "docs: README com passo a passo e Swagger completo"
```

---

## Cobertura da especificação

| Requisito da especificação | Tarefa |
|---|---|
| CRUD de entradas e saídas, tabelas separadas por toggle | 7, 16 |
| Pop-ups de confirmação | 13, 16, 18 |
| Gráficos de comparação mensal e anual | 9, 17 |
| Meta opcional | 10, 16 |
| Login, cadastro e isolamento por usuário | 4, 5, 7, 15 |
| Regex de e-mail e senha forte (8–16, letra, número, especial) | 3, 13, 15 |
| Nome, data de nascimento, telefone e CPF com máscara e validação | 3, 13, 15 |
| Salário editável e sugestão de 30% | 5, 8, 18 |
| Soma automática das saídas do mês | 9, 16 |
| Badges de entradas, saídas e sobra | 12, 14, 16 |
| Visão mensal e anual | 9, 17 |
| JWT com guards e proteção contra injeção | 3, 4, 11 |
| Mobile first e responsividade | 12, 14, 19 |
| Design system e tema claro/escuro | 12 |
| Swagger e mensagens de erro e sucesso | 2, 11, 20 |
