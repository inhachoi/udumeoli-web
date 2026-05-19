# FSD 구조 마이그레이션 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** TanStack Start 프로젝트에 FSD(Feature-Sliced Design) 레이어 구조(app / pages / shared)를 적용하고 초기 세팅 잔재 파일을 제거한다.

**Architecture:** `src/app/routes/`에 TanStack 파일 기반 라우팅을 두고, 실제 페이지 UI는 `src/pages/`에서 분리한다. 재사용 컴포넌트·유틸은 `src/shared/`로 이동하며, shadcn/ui 컴포넌트도 `src/shared/ui/`에 생성되도록 설정한다.

**Tech Stack:** TanStack Start 1.x, TanStack Router (file-based), React 19, Tailwind CSS 4, shadcn/ui, TypeScript, pnpm

---

## 파일 맵

| 상태 | 경로 |
|------|------|
| 생성 | `src/app/styles.css` |
| 생성 | `src/app/router.tsx` |
| 생성 | `src/app/routes/__root.tsx` |
| 생성 | `src/app/routes/index.tsx` |
| 생성 | `src/shared/lib/utils.ts` |
| 생성 | `src/shared/ui/button.tsx` |
| 생성 | `src/pages/home/ui/HomePage.tsx` |
| 수정 | `vite.config.ts` |
| 수정 | `tsconfig.json` |
| 수정 | `components.json` |
| 삭제 | `src/routes/` (전체) |
| 삭제 | `src/router.tsx` |
| 삭제 | `src/styles.css` |
| 삭제 | `src/components/` (전체) |
| 삭제 | `src/lib/` (전체) |
| 삭제 | `src/logo.svg` |
| 자동재생성 | `src/routeTree.gen.ts` |

---

### Task 1: shared 레이어 생성

**Files:**
- Create: `src/shared/lib/utils.ts`
- Create: `src/shared/ui/button.tsx`

- [ ] **Step 1: `src/shared/lib/utils.ts` 생성**

```ts
import { clsx } from "clsx"
import { twMerge } from "tailwind-merge"
import type { ClassValue } from "clsx"

export function cn(...inputs: Array<ClassValue>) {
  return twMerge(clsx(inputs))
}
```

- [ ] **Step 2: `src/shared/ui/button.tsx` 생성**

기존 `src/components/ui/button.tsx` 내용을 복사하되 utils import만 수정:

```tsx
import * as React from "react"
import { cva } from "class-variance-authority"
import { Slot } from "radix-ui"
import type { VariantProps } from "class-variance-authority"

import { cn } from "@/shared/lib/utils"

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-all disabled:pointer-events-none disabled:opacity-50 [&_svg]:pointer-events-none [&_svg:not([class*='size-'])]:size-4 shrink-0 [&_svg]:shrink-0 outline-offset-2 focus-visible:outline focus-visible:outline-2 focus-visible:outline-ring/70 aria-invalid:border-destructive aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40",
  {
    variants: {
      variant: {
        default:
          "bg-primary text-primary-foreground shadow-xs hover:bg-primary/90",
        destructive:
          "bg-destructive text-white shadow-xs hover:bg-destructive/90 focus-visible:outline-destructive",
        outline:
          "border border-input bg-background shadow-xs hover:bg-accent hover:text-accent-foreground",
        secondary:
          "bg-secondary text-secondary-foreground shadow-xs hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2 has-[>svg]:px-3",
        sm: "h-8 rounded-md gap-1.5 px-3 has-[>svg]:px-2.5",
        lg: "h-10 rounded-md px-6 has-[>svg]:px-4",
        icon: "size-9",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

function Button({
  className,
  variant,
  size,
  asChild = false,
  ...props
}: React.ComponentProps<"button"> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean
  }) {
  const Comp = asChild ? Slot : "button"

  return (
    <Comp
      data-slot="button"
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
}

export { Button, buttonVariants }
```

> **주의:** 기존 `src/components/ui/button.tsx`의 실제 내용을 확인해서 위 코드와 동일한지 맞춰야 한다. 다르면 기존 파일 기준으로 작성.

- [ ] **Step 3: typecheck 실행**

```bash
pnpm typecheck
```

Expected: `src/shared/` 파일에서 에러 없음 (단, 아직 old 파일도 존재하므로 중복 에러 무시)

- [ ] **Step 4: commit**

```bash
git add src/shared/
git commit -m "feat: add shared layer (ui, lib)"
```

---

### Task 2: pages 레이어 생성

**Files:**
- Create: `src/pages/home/ui/HomePage.tsx`

- [ ] **Step 1: `src/pages/home/ui/HomePage.tsx` 생성**

기존 `src/routes/index.tsx`의 `App` 컴포넌트를 이전:

```tsx
import { Button } from "@/shared/ui/button"

export function HomePage() {
  return (
    <div className="flex min-h-svh p-6">
      <div className="flex max-w-md min-w-0 flex-col gap-4 text-sm leading-loose">
        <div>
          <h1 className="font-medium">Project ready!</h1>
          <p>You may now add components and start building.</p>
          <p>We&apos;ve already added the button component for you.</p>
          <Button className="mt-2">Button</Button>
        </div>
      </div>
    </div>
  )
}
```

- [ ] **Step 2: commit**

```bash
git add src/pages/
git commit -m "feat: add pages layer with home page"
```

---

### Task 3: app 레이어 생성 (routes + router + styles)

**Files:**
- Create: `src/app/styles.css`
- Create: `src/app/router.tsx`
- Create: `src/app/routes/__root.tsx`
- Create: `src/app/routes/index.tsx`

- [ ] **Step 1: `src/app/styles.css` 생성**

기존 `src/styles.css` 내용을 그대로 복사.

```bash
cp src/styles.css src/app/styles.css
```

- [ ] **Step 2: `src/app/router.tsx` 생성**

기존 `src/router.tsx` 내용 복사. routeTree import 경로만 수정:

```tsx
import { createRouter as createTanStackRouter } from "@tanstack/react-router"
import { routeTree } from "../routeTree.gen"

export function getRouter() {
  const router = createTanStackRouter({
    routeTree,
    scrollRestoration: true,
    defaultPreload: "intent",
    defaultPreloadStaleTime: 0,
  })

  return router
}

declare module "@tanstack/react-router" {
  interface Register {
    router: ReturnType<typeof getRouter>
  }
}
```

- [ ] **Step 3: `src/app/routes/__root.tsx` 생성**

기존 `src/routes/__root.tsx`에서 styles.css import 경로 수정 (`../styles.css` → `../styles.css`, 상대경로는 동일):

```tsx
import { HeadContent, Scripts, createRootRoute } from "@tanstack/react-router"
import { TanStackRouterDevtoolsPanel } from "@tanstack/react-router-devtools"
import { TanStackDevtools } from "@tanstack/react-devtools"

import appCss from "../styles.css?url"

export const Route = createRootRoute({
  head: () => ({
    meta: [
      { charSet: "utf-8" },
      { name: "viewport", content: "width=device-width, initial-scale=1" },
      { title: "udumeoli" },
    ],
    links: [{ rel: "stylesheet", href: appCss }],
  }),
  notFoundComponent: () => (
    <main className="container mx-auto p-4 pt-16">
      <h1>404</h1>
      <p>The requested page could not be found.</p>
    </main>
  ),
  shellComponent: RootDocument,
})

function RootDocument({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <head>
        <HeadContent />
      </head>
      <body>
        {children}
        <TanStackDevtools
          config={{ position: "bottom-right" }}
          plugins={[{ name: "Tanstack Router", render: <TanStackRouterDevtoolsPanel /> }]}
        />
        <Scripts />
      </body>
    </html>
  )
}
```

- [ ] **Step 4: `src/app/routes/index.tsx` 생성**

라우트 정의만, UI는 HomePage에서 import:

```tsx
import { createFileRoute } from "@tanstack/react-router"
import { HomePage } from "@/pages/home/ui/HomePage"

export const Route = createFileRoute("/")({ component: HomePage })
```

- [ ] **Step 5: commit (아직 old 파일 유지 상태)**

```bash
git add src/app/
git commit -m "feat: add app layer (routes, router, styles)"
```

---

### Task 4: 설정 파일 업데이트

**Files:**
- Modify: `vite.config.ts`
- Modify: `tsconfig.json`
- Modify: `components.json`

- [ ] **Step 1: `vite.config.ts` 수정**

`tanstackStart()`에 라우트 디렉토리와 router 파일 경로 지정:

```ts
import { defineConfig } from "vite"
import { devtools } from "@tanstack/devtools-vite"
import { tanstackStart } from "@tanstack/react-start/plugin/vite"
import viteReact from "@vitejs/plugin-react"
import viteTsConfigPaths from "vite-tsconfig-paths"
import tailwindcss from "@tailwindcss/vite"
import { nitro } from "nitro/vite"

const config = defineConfig({
  plugins: [
    devtools(),
    nitro(),
    viteTsConfigPaths({
      projects: ["./tsconfig.json"],
    }),
    tailwindcss(),
    tanstackStart({
      routesDirectory: "./src/app/routes",
      generatedRouteTree: "./src/routeTree.gen.ts",
    }),
    viteReact(),
  ],
})

export default config
```

> **주의:** `tanstackStart()`가 `routerSrc` 옵션도 지원하면 `routerSrc: './src/app/router.tsx'`도 추가. 지원하지 않으면 `src/router.tsx`를 `src/app/router.tsx`의 re-export로 유지 (Task 5에서 처리).

- [ ] **Step 2: `tsconfig.json` paths 수정**

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@/app/*": ["./src/app/*"],
      "@/pages/*": ["./src/pages/*"],
      "@/shared/*": ["./src/shared/*"]
    }
  }
}
```

- [ ] **Step 3: `components.json` 수정**

```json
{
  "tailwind": {
    "css": "src/app/styles.css"
  },
  "aliases": {
    "components": "@/shared/ui",
    "utils": "@/shared/lib/utils",
    "ui": "@/shared/ui",
    "lib": "@/shared/lib",
    "hooks": "@/shared/hooks"
  }
}
```

- [ ] **Step 4: commit**

```bash
git add vite.config.ts tsconfig.json components.json
git commit -m "chore: update config for FSD structure"
```

---

### Task 5: router.tsx 처리 및 검증

**Files:**
- Modify or Delete: `src/router.tsx`

- [ ] **Step 1: dev 서버 실행해서 라우터 파일 인식 여부 확인**

```bash
pnpm dev
```

- `src/app/routes/`를 인식하고 `routeTree.gen.ts`가 재생성되면 → 성공
- 에러가 나면 TanStack Start가 `src/router.tsx`를 찾는 경우 → Step 2로

- [ ] **Step 2: (필요한 경우만) `src/router.tsx`를 re-export로 교체**

dev 서버가 `src/router.tsx`를 필수로 요구하면:

```tsx
export { getRouter } from "@/app/router"
```

- [ ] **Step 3: typecheck 실행**

```bash
pnpm typecheck
```

Expected: 에러 없음

- [ ] **Step 4: 브라우저에서 `http://localhost:3000` 확인**

- 홈 페이지 정상 렌더링
- Button 컴포넌트 표시
- Devtools 정상

---

### Task 6: 기존 파일 삭제

**Files:**
- Delete: `src/routes/` (전체)
- Delete: `src/router.tsx` (Task 5에서 re-export로 변경하지 않은 경우)
- Delete: `src/styles.css`
- Delete: `src/components/` (전체)
- Delete: `src/lib/` (전체)
- Delete: `src/logo.svg`

- [ ] **Step 1: 파일 삭제**

```bash
rm -rf src/routes src/components src/lib src/logo.svg src/styles.css
```

> `src/router.tsx`를 re-export로 유지 중이면 삭제하지 않는다.

- [ ] **Step 2: typecheck + dev 재확인**

```bash
pnpm typecheck
pnpm dev
```

Expected: 에러 없음, 페이지 정상 렌더링

- [ ] **Step 3: final commit**

```bash
git add -A
git commit -m "chore: remove legacy files after FSD migration"
```

---

## 완료 기준

- `pnpm typecheck` 통과
- `pnpm dev` 실행 시 홈 페이지 정상 렌더링
- `pnpm dlx shadcn add [component]` 실행 시 `src/shared/ui/`에 생성
- `src/routeTree.gen.ts` 자동 재생성 확인
