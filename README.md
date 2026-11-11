# NPS Calculator

API didática de **pesquisas de satisfação e cálculo de Net Promoter Score (NPS)**, construída com Express, TypeScript e SQLite durante a NLW 4. O repositório contém o backend em `backend/`, sem frontend web.

[![TypeScript](https://img.shields.io/badge/TypeScript-project-3178C6)](https://www.typescriptlang.org/docs/)
[![License](https://img.shields.io/badge/License-MIT-blue)](LICENSE)

## 📋 Table of contents

- [✨ Features](#features)
- [🧪 Technologies](#technologies)
- [🛠️ Environment setup](#environment-setup)
- [🚀 Getting started](#getting-started)
- [🔌 API usage](#api-usage)
- [🗂️ Project structure](#project-structure)
- [✅ Testing and quality](#testing-and-quality)
- [🔎 Troubleshooting](#troubleshooting)
- [🤝 Contributing](#contributing)
- [📜 License](#license)

<a id="features"></a>

## ✨ Features

- Cadastro de usuários e pesquisas de satisfação.
- Envio de pesquisas por email de teste com Ethereal.
- Registro de respostas e cálculo de NPS, com contagens de promotores, neutros e detratores.
- Persistência SQLite e migrations com TypeORM.
- Testes de usuários e pesquisas com Jest/Supertest.

Projeto desenvolvido durante a trilha Node.js da quarta edição da Next Level Week, da [Rocketseat](https://github.com/Rocketseat).

<a id="technologies"></a>

## 🧪 Technologies

| Tecnologia | Versão declarada | Finalidade |
| --- | --- | --- |
| [Node.js](https://nodejs.org/) | Não definida | Runtime |
| [TypeScript](https://www.typescriptlang.org/docs/) | `^4.1.5` | Código e migrations |
| [Express](https://expressjs.com/) | `^4.17.1` | API HTTP |
| [TypeORM](https://typeorm.io/) | `^0.2.31` | Repositórios e migrations |
| [sqlite3](https://github.com/TryGhost/node-sqlite3) | `^5.0.2` | Driver nativo SQLite |
| [Nodemailer](https://nodemailer.com/) | `^6.4.18` | Email de teste |
| [Handlebars](https://handlebarsjs.com/) | `^4.7.7` | Template da pesquisa |
| [Jest](https://jestjs.io/) | `^26.6.3` | Testes com ts-jest |

Faixas de [backend/package.json](backend/package.json), sem lockfile. Node **14.16.0** foi escolhido para tentar reproduzir a combinação de TypeScript 4.1/Jest 26 da época do curso; não é uma versão declarada pelo projeto. A compilação de dependências nativas depende da plataforma e da resolução atual.

<a id="environment-setup"></a>

## 🛠️ Environment setup

Use Git, [nvm](https://github.com/nvm-sh/nvm) e Node.js **14.16.0**. Os comandos abaixo usam npm, fornecido com o Node.js. Não há lockfile versionado; use `npm install`, e não `npm ci`. A resolução das faixas de versão pode variar entre instalações.

```bash
nvm install 14.16.0
nvm use 14.16.0
git clone https://github.com/paulodaluz/NPSCalculator.git
cd NPSCalculator/backend
npm install
```

Todos os comandos npm devem ser executados em **`backend/`**. SQLite é acessado pelo pacote `sqlite3`; não há servidor de banco separado. A instalação pode precisar de binário nativo compatível ou ferramentas de compilação já presentes.

Não há script `prepare`, Husky ou `.env.example`. O código não carrega `.env` automaticamente. Exporte a URL usada nos links dos emails:

```bash
export URL_MAIL=http://localhost:3333/answers
```

| Variável | Uso |
| --- | --- |
| `URL_MAIL` | Base dos links de resposta montados em `npsMail.hbs`; necessária para links úteis no email |
| `NODE_ENV` | Quando igual a `test`, seleciona `database.test.sqlite`; o script de teste define a variável |

A configuração TypeORM está em [backend/ormconfig.json](backend/ormconfig.json). A base normal é `src/database/database.sqlite`; a de teste é `src/database/database.test.sqlite`. Ambos os arquivos já estão versionados: utilize uma cópia de trabalho de desenvolvimento e preserve qualquer dado que precise manter.

O serviço de email cria automaticamente uma conta de testes no Ethereal durante a importação. Isso requer acesso externo a `api.nodemailer.com` e ao SMTP fornecido pelo serviço; não exige suas credenciais pessoais. Não há configuração de SMTP de produção implementada.

<a id="getting-started"></a>

## 🚀 Getting started

Em `backend/`, aplique as migrations na base de desenvolvimento e inicie o servidor:

```bash
npm run typeorm -- migration:run
URL_MAIL=http://localhost:3333/answers npm run dev
```

A porta é fixa em **3333** em `src/server.ts`; `PORT` não é lida. URL base: `http://localhost:3333`.

```bash
curl http://localhost:3333/surveys
```

A listagem depende das tabelas criadas pelas migrations. Não há rota `/` ou de saúde. O script `dev` usa `ts-node-dev --transpile-only`; iniciar o servidor não comprova que os tipos compilam. Não há script de build ou execução de produção.

<a id="api-usage"></a>

## 🔌 API usage

As rotas não possuem middleware de autenticação.

| Método | Caminho | Corpo ou parâmetro |
| --- | --- | --- |
| POST | `/users` | `name`, `email` |
| POST | `/surveys` | `title`, `description` |
| GET | `/surveys` | Lista pesquisas |
| POST | `/sendMail` | `email`, `survey_id` existentes |
| GET | `/answers/:value?u=ID` | Nota e identificador da associação pesquisa/usuário; altera dados |
| GET | `/nps/:survey_id` | Estatística da pesquisa |

Exemplo para uma base local de teste:

```bash
curl -X POST http://localhost:3333/users \
  -H 'Content-Type: application/json' \
  -d '{"name":"Pessoa Exemplo","email":"pessoa@example.com"}'

curl -X POST http://localhost:3333/surveys \
  -H 'Content-Type: application/json' \
  -d '{"title":"Atendimento","description":"Como foi sua experiência?"}'
```

Use os IDs retornados nas demais operações. O serviço imprime um link de visualização Ethereal após o envio; os emails de teste não são entregues aos destinatários reais. A [coleção Postman](%23NLW4.postman_collection.json) é uma referência adicional: revise os dados de exemplo antes de importar.

O cálculo considera notas 0–6 detratoras, 7–8 neutras e 9–10 promotoras. Não há tratamento específico de uma pesquisa com zero respostas no cálculo atual.

<a id="project-structure"></a>

## 🗂️ Project structure

| Caminho | Responsabilidade |
| --- | --- |
| `backend/src/server.ts` | Porta HTTP |
| `backend/src/app.ts` | Express, conexão e erros |
| `backend/src/routes.ts` | Endpoints |
| `backend/src/controllers/` | Usuários, pesquisas, email, respostas e NPS |
| `backend/src/models/` / `repositories/` | Entidades e repositórios TypeORM |
| `backend/src/database/migrations/` | Criação das tabelas |
| `backend/src/services/SendMailService.ts` | Conta e transporte Ethereal |
| `backend/src/views/emails/` | Template Handlebars |
| `backend/src/__tests__/` | Suítes de usuários e pesquisas |

<a id="testing-and-quality"></a>

## ✅ Testing and quality

```bash
cd backend # se estiver na raiz do repositório
npm test
```

O script define `NODE_ENV=test`, executa Jest em série e usa o SQLite de teste. As suítes aplicam migrations e removem as tabelas ao terminar; não aponte a configuração de teste para dados que queira manter. A importação da aplicação também inicia a criação de conta Ethereal. Não há lint, formatação ou build configurados em scripts.

### Validation recorded

Em 11/11/2026, a instalação concluiu com Node 14.16.0/npm 6.14.11, incluindo sqlite3 5.1.7. `npm test` falhou ao carregar `jest.config.ts`, com incompatibilidade de `ts.resolveTypeReferenceDirective` entre TypeScript 4.9.5 e ts-node 9.1.1 resolvidos. `npm run typeorm -- migration:run` apresentou erros de compilação dos modelos e foi encerrado após 30 segundos. Uma tentativa de `npm run dev` abriu a porta 3333, mas a requisição inicial a `/surveys` retornou 500; uma nova tentativa não ficou pronta na janela de observação. O funcionamento da API, migrations e email não foi confirmado. Os arquivos SQLite versionados foram preservados, e todos os processos iniciados foram encerrados.

<a id="troubleshooting"></a>

## 🔎 Troubleshooting

- Erro ao carregar `sqlite3`: conferir o binário nativo e a compatibilidade do Node; atualização do driver fica para tarefa separada.
- `no such table`: executar migrations dentro de `backend/` e conferir o arquivo SQLite selecionado.
- Links de email sem destino: exportar `URL_MAIL`; criar um `.env` sozinho não altera o ambiente do processo.
- Erro ou demora no Ethereal: a inicialização depende de rede externa. O código não aguarda a criação da conta antes de aceitar requisições de email.
- Erros TypeScript em testes: o modo `dev` transpila sem verificar tipos; as duas validações não são equivalentes.

<a id="contributing"></a>

## 🤝 Contributing

Crie uma branch de `main`, teste com dados fictícios e uma base de desenvolvimento, execute `npm test` em `backend/` e descreva verificações e limites no PR. Não inclua mudanças incidentais dos arquivos SQLite ou lockfiles em alterações de documentação.

<a id="license"></a>

## 📜 License

Distribuído sob a licença **MIT**, preservada em [LICENSE](LICENSE).

---

<p align="center">Made with 💜 by <a href="https://github.com/paulodaluz">Paulo da Luz</a></p>
