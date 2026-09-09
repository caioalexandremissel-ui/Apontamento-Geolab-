# Sistema de Apontamento de Manutenção (GeoLab)

Sistema web para registro de ordens de serviço de manutenção, substituindo o apontamento manual no SAP. Técnicos de manutenção registram tempo de execução diretamente no sistema, e administradores exportam os dados para integração com o SAP.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — API server (port 8080, path /api)
- `pnpm --filter @workspace/manutencao run dev` — Frontend (porta dinâmica, path /)
- `pnpm run typecheck` — typecheck completo
- `pnpm run build` — typecheck + build
- `pnpm --filter @workspace/api-spec run codegen` — regenerar hooks e schemas do OpenAPI
- `pnpm --filter @workspace/db run push` — aplicar mudanças no schema DB

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite, Wouter (roteamento), TanStack Query, Tailwind CSS
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Validação: Zod (zod/v4), drizzle-zod
- API codegen: Orval (OpenAPI spec → hooks + schemas)
- Excel: xlsx (importação e exportação)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — spec OpenAPI (fonte da verdade para contratos)
- `lib/db/src/schema/` — schema do banco (users, service_orders, service_order_lines, confirmations, edit_requests, root_causes, maintenance_codes)
- `artifacts/api-server/src/routes/` — rotas Express (auth, users, service-orders, confirmations, etc.)
- `artifacts/manutencao/src/` — frontend React
  - `src/contexts/AuthContext.tsx` — autenticação baseada em token (localStorage)
  - `src/pages/` — páginas admin e user

## Architecture decisions

- Auth simples com token Base64 (matricula:id:role) via Authorization header — suficiente para uso interno
- Senha inicial = matrícula do colaborador (admin pode alterar depois)
- Cálculo de trabalho real feito no backend no endpoint `/confirmations/:id/finish`
- Exportação para SAP feita via JSON → xlsx no browser (SheetJS)
- Status das confirmações: `started` → `overtime_pending` → `finished` → `edited`

## Product

- Login split: lado esquerdo para Administração, lado direito para Operação (técnicos)
- Técnicos recebem ordens de serviço, clicam em Iniciar (captura data/hora) e Finalizar (registra duração, observação, causa raiz)
- Se ultrapassar o tempo previsto da operação: sistema notifica e exige justificativa
- Administradores importam OS via Excel, gerenciam colaboradores, catálogo de causas raiz e códigos de manutenção
- Exportação para SAP em formato Excel com colunas padrão do sistema

## User preferences

- Interface inteiramente em Português (BR)
- Senhas iniciais = matrícula do colaborador
- Login admin padrão: matrícula `admin`, senha `admin`

## Gotchas

- Sempre rodar codegen após mudar o openapi.yaml: `pnpm --filter @workspace/api-spec run codegen`
- Push do schema antes de testar rotas novas: `pnpm --filter @workspace/db run push`
- O xlsx deve estar no package.json de @workspace/manutencao (devDependencies)

## Pointers

- Ver skill `pnpm-workspace` para estrutura e TypeScript
- Ver skill `react-vite` para o workflow do frontend
