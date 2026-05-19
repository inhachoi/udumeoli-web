# FSD 구조 설계 — 최소 구성 (app / pages / shared)

## 목표

TanStack Start 프로젝트에 Feature-Sliced Design(FSD) 레이어 구조를 적용한다.
초기 세팅 잔재 파일을 제거하고, 향후 기능 확장에 맞는 기반을 최소로 구성한다.

## 최종 디렉토리 구조

```
src/
├── app/
│   ├── routes/
│   │   ├── __root.tsx
│   │   └── index.tsx       ← 라우트 정의만, UI는 pages/에서 import
│   ├── router.tsx
│   └── styles.css
│
├── pages/
│   └── home/
│       └── ui/
│           └── HomePage.tsx
│
└── shared/
    ├── ui/                 ← shadcn/ui 컴포넌트 생성 위치
    │   └── button.tsx
    └── lib/
        └── utils.ts
```

## 레이어 책임

| 레이어 | 역할 |
|--------|------|
| `app/` | 라우팅, 글로벌 스타일, 앱 진입 설정 |
| `pages/` | 페이지별 UI 컴포넌트 (라우트 컴포넌트 아님) |
| `shared/` | 재사용 UI(shadcn), 유틸리티 등 도메인 무관 공통 코드 |

## shadcn/ui 컴포넌트 경로

`components.json`의 `aliases.components`를 `@/shared/ui`로 설정.
`pnpm dlx shadcn add [component]` 실행 시 `src/shared/ui/`에 자동 생성.

## 변경 사항 목록

### 파일 이동
- `src/routes/` → `src/app/routes/`
- `src/router.tsx` → `src/app/router.tsx`
- `src/styles.css` → `src/app/styles.css`
- `src/components/ui/button.tsx` → `src/shared/ui/button.tsx`
- `src/lib/utils.ts` → `src/shared/lib/utils.ts`

### 파일 삭제
- `src/logo.svg`
- `src/components/` (이전 후 빈 디렉토리)
- `src/lib/` (이전 후 빈 디렉토리)

### 설정 변경

**vite.config.ts** — 라우트 디렉토리 지정:
```ts
tanstackStart({
  routesDirectory: './src/app/routes',
  generatedRouteTree: './src/routeTree.gen.ts',
})
```

**tsconfig.json** — paths 추가:
```json
"@/app/*": ["./src/app/*"],
"@/pages/*": ["./src/pages/*"],
"@/shared/*": ["./src/shared/*"]
```

**components.json** — shadcn alias 수정:
```json
{
  "aliases": {
    "components": "@/shared/ui",
    "utils": "@/shared/lib/utils"
  }
}
```

### import 수정
- `app/routes/__root.tsx`: `../styles.css` → `../styles.css` (상대경로 유지)
- `app/routes/index.tsx`: `@/components/ui/button` → `@/shared/ui/button` + HomePage import 추가
- `shared/ui/button.tsx`: `@/lib/utils` → `@/shared/lib/utils`

## 완료 기준

- `pnpm dev` 실행 시 정상 동작
- `pnpm typecheck` 통과
- shadcn add 명령 시 `src/shared/ui/`에 생성
