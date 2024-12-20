# Next.js (v15.0.2)

Next.js v15부터는 react 및 react-dom 최소 버전이 v19입니다.

```json
{
  "name": "nextjs",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "15.0.2",
    "react": "19.0.0-rc-0bc30748-20241028",
    "react-dom": "19.0.0-rc-0bc30748-20241028"
  },
  "devDependencies": {
    "@types/node": "^20",
    "@types/react": "^18",
    "@types/react-dom": "^18",
    "typescript": "^5"
  },
  "packageManager": "yarn@4.2.2"
}
```

## Routing

### App Directory

Next v13부터 app 디렉토리 구조에 따라 라우팅되며 각 파일의 역할(기능)과 컨벤션이 지정되어 있습니다.

기본적으로 모든 컴포넌트들은 RSC(React Server Component)로 동작합니다. 만약 RCC(React Client Component)로 설정하기 위해서는 파일 최상단에 `"use client";`를 작성해야 합니다.

```javascript
// app/page.tsx
'use client' // RCC 컴포넌트 선언

export default function Page() {
  return <main>Content,,,</main>
}
```

### Dynamic Routes

디렉토리명을 "[folderName]"처럼 대괄호로 깜싼 경우 기존 다이나믹 라우팅으로 동작합니다.

다이나믹 라우트 세그먼트는 page.tsx, layout.tsx, route.ts, generateMetadata 함수에 params prop으로 전달됩니다.

```javascript
// app/blog/[slug]/page.tsx
type Params = Promise<{ slug: string }>

interface PageProps {
  params?: Params // 라우트 세그먼트 값 params[folderName]으로 접근
}

export default function Page({ params }: PageProps) {
  /**
   * URL: "/blog/b" , params: { slug: 'b' }
   * URL: "/blog/a" , params: { slug: 'a' }
   **/

  return <main>Content,,,</main>
}
```

> 다이나믹 라우팅이란 URL 경로값으로 페이지 컴포넌트 내 렌더링될 정보를 동적으로 결정하여 페이지를 렌더링합니다. 일반적으로 이는 하나의 페이지를 재사용하여 여러 콘텐츠를 동작하기 위해서 사용합니다.

### Catch-All Routes

디렉토리 명을 "[...forderName]"으로 생성하게 되면 매칭되는 라우트 세그먼트가 없는 경우 Catch-All Segments가 활성화 됩니다.

- Route: "/app/shop/[...slug]/page.tsx" -> URL: "/shop/a" -> params: { slug: ['a'] }

- Route: "/app/shop/[...slug]/page.tsx" -> URL: "/shop/a/b" -> params: { slug: ['a', 'b'] }

- Route: "/app/shop/[...slug]/page.tsx" -> URL: "/shop/a/b/c" -> params: { slug: ['a', 'b', 'c'] }

위 예제처럼 Catch-All Routes는 params로 여러 라우트 세그먼트 값들을 요소로 갖는 배열로 전달받습니다.

#### Optional Catch-All Routes

Catch-All Segments는 params가 1개 이상인 경우에만 활성화 되지만 "[[...forderName]]"으로 디렉토리명을 생성하면 해당 Catch-All Segments는 params가 0개인 경우에도 활성화 됩니다.

- Route: "/app/shop/[[...slug]]/page.tsx" -> URL: "/shop" -> params: { }

위 예제처럼 params가 0개인 경우 params는 빈 객체가 전달됩니다.

### Route Groups

일반적으로 app디렉토리 내 디렉토리명들은 라우트 세그먼트로 URL 경로와 매핑됩니다. 하지만 디렉토리명을 "(folderName)"처럼 소괄호로 감싼 경우 라우트 세그먼트로 설정되지 않으며 실제 라우팅에서 해당 경로는 무시됩니다.

예를 들어, "app/(marketing)/about" 경로에 생성된 페이지는 "/(marketing)/about"에서 "(marketing)"이 무시된 "/about"으로 라우팅됩니다.

즉, 소괄호로 디렉토리를 만들어 경로들의 그룹을 생성할 수 있으며, layout.tsx나 template.tsx를 통해 URL 경로에 영향을 주지 않고 경로 그룹별 layout을 설정할 때 유용하게 사용할 수 있습니다.

### Parallel Routes

디렉토링명을 "@folderName"로 작성한 경우 라우트 세그먼트가 아닌 슬롯으로 설정되며 실제로 해당 경로는 무시됩니다.

패러렐 라우팅은 하나의 레이아웃에 여러 page를 표시하기 위해서 사용합니다.

```markdown
app
├── page.tsx
│
├── layout.tsx
│
└── @team
    └── settings
        └── page.tsx
```

위 구조처럼 "/app/@team/settings/page.tsx"가 존재할 때 "/"에서 "/settigns"로 이동하게 되면 "/app/page.tsx"가 렌더링된 상태에서 "/app/@team/settgins/page.tsx"도 병렬적으로 렌더링할 수 있습니다. 실제 URL은 "/settings"이지만 "/app/page.tsx"가 렌더링된 상태를 유지하면서 "/app/@team/settings/page.tsx"도 병렬로 렌더링 시킬 수 있습니다.

"@forderName/page.tsx"에서 export한 컴포넌트의 경우 @forderName와 동일한 레벨에 있는 layout 컴포넌트 prop으로 전달됩니다. 만약 동일한 레벨에 없다면 가장 가까운 상위 layout.tsx의 Layout 컴포넌트에게 전달됩니다. 이때 전달되는 prop 네이밍은 디렉토리명(forderName)으로 전달됩니다.

<hr />

예를 들어, "/app/@modal/page.tsx"가 export default한 컴포넌트는 "/app/layout.tsx"가 export default한 Layout 컴포넌트에 modal prop으로 전달됩니다.

```javascript
// app/layout.tsx
import { ReactNode } from 'react'

interface LayoutProps {
  children: ReactNode // "app/page.tsx"에서 export default한 컴포넌트
  modal?: ReactNode // "app/@modal/page.tsx"에서 export default한 컴포넌트
}

export default function Layout({ children, modal }: LayoutProps) {
  <html lang="ko">
    <body>
      {children}
      {modal}
    </body>
  </html>
}
```

<hr />

페러렐 라우팅의 주의할 점으로는 아래와 같습니다.

- Soft Navigation 사용하는 경우 Next는 현재 일치하는 슬롯이 없더라면 이전 활성화된 슬롯 활성을 유지합니다. 또한 요청한 경로에 대한 페이지가 없더라도 404 에러를 표시하지 않고 이전에 활성화된 페이지를 계속 표시해줍니다.

```markdown
app
├── page.tsx
│
├── layout.tsx
│
├── @team
│   └── default.tsx
│   └── settings
│       └── page.tsx
│
└── @analytics
    └── page.tsx
```

app 디렉토리 구조가 위와 같고 초기 경로가 "/"일 때 "/app/@analytics/page.tsx"가 활성화됩니다. "/app/@team/page.tsx"는 존재하지 않으므로 "/app/@team/default.tsx"가 활성화됩니다.

이후 "/"에서 "/settings"로 Soft Navigation 사용했다면 "/app/@team/settings/page.tsx"이 활성화 됩니다. 이때 "/app/@analytics/settings/page.tsx"가 존재하지 않기 때문에 기존에 활성화된 "/app/@analytics/page.tsx" 활성을 유지합니다.
"/app/settings/page.tsx" 또한 존재하지 않지만 404 에러를 표시하지 않고 기존에 활성화된 페이지("/app/page.tsx") 렌더링을 유지합니다.
즉, 실제 경로는 "/settings"인데 렌더링된 페이지는 "/app/page.tsx"이며 "/app/@analytics/page.tsx"와 "/app/@team/settings/page.tsx"가 활성된 상태가 됩니다.

> 슬롯과 세그먼트는 서로 독립적입니다. 다시 말해, 실제 경로와 매칭되는 세그먼트와 슬롯 이름이 동일하더라도 서로 충돌하지 않고 독립적으로 동작하게 됩니다. 예를 들어, "/app/settings/page.tsx"와 "/app/@team/settings/page.tsx" 둘 다 존재하더라도 서로 충돌하지 않으며 실제 경로가 "/settings"일 때 둘 다 렌더링됩니다.

- Hard Navigation 사용하는 경우 Next는 먼저 매칭되는 슬롯을 확인하고 만약 없는 경우에는 상위 디렉토리로 올라가면서 가장 가까운 default.tsx가 export default한 컴포넌트 렌더링을 시도합니다. 이때 default.tsx 파일조차 없다면 404에러가 발생하게 됩니다.

```markdown
app
├── page.tsx
│
├── layout.tsx
│
├── @team
│   └── default.tsx
│   └── settings
│       └── page.tsx
│
└── @analytics
    └── page.tsx
```

app 디렉토리 구조가 위와 같을 때 "/settings"로 Hard Navigation하는 경우 "/app/settings/page.tsx"가 존재하지 않아 404에러를 표시하게 됩니다. 만약 "/app/settings/page.tsx"가 있다 하더라도 "/app/@analytics/settings/page.tsx"이 없기 때문에 "/app/@analytics/default.tsx" 렌더링을 시도하는데, 해당 파일조차 존재하지 않아 404에러가 발생시키게 됩니다.

즉, Hard Navigation하는 경우에는 이전에 활성화된 슬롯이나 렌더링된 페이지에 대한 정보를 갖고 있지 않기 때문에 Soft Navigation과 다르게 동작하게 됩니다.

### Intercepting Routes

디렉토리명을 "(...)fortuneName", "(..)fortuneName" 혹은 "(.)fortuneName"로 작성한 경우 인터셉팅 라우트로 동작합니다.

- (.): 동일한 레벨 라우트 세그먼트를 인터셉팅

- (..): 하나의 상위 레벨 라우트 세그먼트를 인터셉팅

- (..)(..): 두 개의 상위 레벨 라우트 세그먼트를 인터셉팅

- (...): app 디렉토리 레벨의 라우트 세그먼트를 인터셉팅

<hr />

주의할 점으로 Route Groups, Slot(Parallel Routes), Private Routes 등 URL 경로에 영향을 주지 않는 것들은 무시되어 인터셉팅됩니다. 즉, 파일 시스템이 아닌 라투트 세그먼트만을 고려하여 인터셉팅합니다.

```markdown
app
├── page.tsx
│
├── layout.tsx
│
├── dashborad
│   └── (..i)
│       └── page.tsx
│
└── i
    └── page.tsx
```

app 디렉토리 구조가 위와 같을 때 "/i"로 이동하는 경우 "app/i/page.tsx"가 아닌 "app/dashboard/(..i)/page.tsx"가 렌더링됩니다.

인터셉팅 라우트는 Soft Navigation의 경우에만 인터셉팅되며 Soft Navigation이 아닌 경우 기존 페이지 컴포넌트가 렌더링됩니다. 즉, "/i"로 Hard Navigation을 사용하여 이동하는 경우에는 "app/i/page.tsx"가 렌더링됩니다.

#### Parallel Routes & Intercepting Routes

Parallel Routes내 Intercepting Routes를 함께 사용하여 독립된 URL 경로를 갖는 모달을 구현할 수 있습니다. 독립된 URL 경로를 갖는 모달의 경우 URL을 통해 모달 내용을 공유할 수 있으며, Hard Navigation을 사용하더라도 그 상태가 유지되고, 뒤로 가기 혹은 앞으로 가기를 통해 모달을 열거나 닫을 수도 있습니다.

```markdown
app
├── page.tsx
│
├── layout.tsx
│
└── @modal
    └── login
        └── page.tsx
```

만약 위와 같은 app 디렉토리 구조를 가진 경우, "/login"으로 Soft Navigation 한다면 "/app/@modal/login/page.tsx"가 활성화 되지만, Hard Navigation을 사용하는 경우에는 404 에러가 발생하게 됩니다.
Hard Navigation 한다면 실제 경로와 매칭되는 페이지를 렌더링하게 되는데 위 구조의 경우 "/app/login/page.tsx" 파일이 존재하지 않아 404 에러가 발생하게 됩니다.

```markdown
app
├── page.tsx
│
├── layout.tsx
│
├── @modal
|   └── login
|       └── page.tsx
|
└── login
    └── page.tsx
```

그렇다면 위와 같은 구조를 가질 때 Hard Navigation을 사용하면 404에러는 발생하지 않지만, Soft Navigation과 Hard Navigation 둘 다 "/app/login/page.tsx"가 렌더링되고 "/app/@modal/login/page.tsx"가 병렬 라우팅됩니다.

일반적으로 모달의 경우 기존 페이지 렌더링을 유지하면서 모달을 표시해주어야 하지만 위의 경우 기존 페이지 렌더링을 유지하지 못하고 "/app/login/page.tsx"가 렌더링됩니다.

<hr />

구현하려던 동작은 Soft Navigation으로 "/login"으로 이동하게 되면 기존 렌더링된 페이지에 병렬로 "/app/@modal/login/page.tsx"가 렌더링되고, Hard Navigation하는 경우에는 404 에러가 발생하지 않으면서 "/app/login/page.tsx"만 렌더링되도록 하려면 Parallel Routes와 Intercepting Routes를 같이 사용하여 구현할 수 있습니다.

```javascript
// app/@modal/(.)login/page.tsx

import Modal from '@/shard/components/Modal'

export default function Page() {
  return (
    <Modal>
      <form>Login Form,,,</form>
    </Modal>
  )
}
```

```javascript
// app/login/page.tsx

export default function Page() {
  return <form>Login Form,,,</form>
}
```

"app/login/page.tsx"와 "app/@modal/(.)login/page.tsx"를 위와 같이 생성해줍니다.

```markdown
app
├── page.tsx
│
├── layout.tsx
│
├── @modal
|   └── (.)login
|       └── page.tsx
|
└── login
    └── page.tsx
```

디렉토리 구조가 위와 같을 때 Soft Navigation을 사용하여 "/login"으로 이동하는 경우 "/app/@modal/(.)login/page.tsx"가 렌더링됩니다. 추가적으로 Hard Navigation하여 이동한 경우에는 "/app/login/page.tsx"만이 렌더링됩니다.

### layout.tsx

layout.tsx 파일에서 export default된 Layout 컴포넌트는 props로 page.tsx가 export default한 컴포넌트를 children prop으로 전달받습니다. 다이나믹 라우트의 경우 라우트 세그먼트 값을 추가적으로 params prop도 전달받으며 이는 동적 라우팅에 대한 경로값을 객체 형태로 전달받습니다. 동일한 레벨에 페러렐 라우트가 존재한다면 추가적으로 해당 슬롯명으로 컴포넌트를 prop으로 전달받습니다.

app 디렉토리의 루트 레벨에는 필수적으로 하나의 layout.tsx("app/layout.tsx") 파일이 필요하며, 해당 컴포넌트는 html과 body 태그를 정의합니다.

app 디렉토리 내 각 하위 디렉토리 마다 하나씩 설정 가능하며 만약 하위 디렉토리에 layout.tsx가 존재하는 경우 상위 layout.tsx가 하위 layout.tsx를 포함하는 형식으로 적용(nested layout)됩니다.

주의할 점으로는 하위 경로로 Soft Navigation을 사용하는 경우 Layout 컴포넌트는 리렌더링 되지 않습니다. 만약 리렌더링이 필요한 Layout 컴포넌트가 필요한 경우 layout.tsx 대신 template.tsx 사용해야 합니다.

```javascript
// app/layout.tsx
import { ReactNode } from 'react'

type Params = Promise<{ slug: string }>

interface LayoutProps {
  children: ReactNode // app/page.tsx에서 export default된 컴포넌트
  params?: Params // 다이나믹 라우팅되는 경우 동적 라우트 세그먼트 값
}

export default function Layout({ children }: LayoutProps) {
  return (
    <html>
      <body>{children}</body>
    </html>
  )
}
```

### template.tsx

template.tsx는 layout.tsx와 유사하게 페이지 레이아웃 역할을 하지만 Layout 컴포넌트와는 다르게 Soft Navigation하는 경우 매번 새롭게 렌더링됩니다.

template.tsx와 layout.tsx를 같이 사용하게 된다면 아래와 같이 Layout 컴포넌트 하위에 Template 컴포넌트가 렌더링되고, Template 컴포넌트 하위에 Page 컴포넌트가 렌더링됩니다. 일반적으로 Layout과 유사한 역할을 하므로 하나만 사용하며, 되도록 특별한 이유가 없다면 Layout 컴포넌트를 사용해야 합니다.

```javascript
<Layout>
  {/* Note that the template is given a unique key. */}
  <Template key={routeParam}>{children}</Template>
</Layout>
```

Template 컴포넌트는 아래와 처럼 매번 렌더링이 필요한 경우에 유용하게 사용할 수 있습니다.

- "use client"를 작성하여 클라이언트 컴포넌트로 설정하고 useEffect나 useState훅에 의존하는 기능이 필요한 경우 사용할 수 있습니다.

- Next 기존 프레임워크 동작을 변경하고 싶을 때 사용할 수 있습니다. 예를 들어, Layout 컴포넌트 하위에 Suspense 컴포넌트를 사용하더라도 fallback 컴포넌트는 처음에만 표시되고 이후 라우팅되더라도 표시되지 않지만 Template 컴폰넌트는 라우팅될 때매다 표시해줍니다.

### page.tsx

page.tsx에서 export default한 컴포넌트는 layout.tsx(or template.tsx)에서 export default한 컴포넌트 children prop으로 전달됩니다. 즉, 실제 페이지 컨텐츠를 나타내는 컴포넌트입니다.

디렉토리마다 page.tsx 파일을 필수이며, 만약 page.tsx파일이 없다면 해당 경로로 라우팅되지 않습니다.

Page 컴포넌트는 동적 라우트의 경우 동적 라우트 세그먼트 값을 params prop으로 전달받으며, 쿼리스트링에 대한 정보는 searchParams prop으로 전달받습니다.

```javascript
// app/page.tsx
type Params = Promise<{ slug: string }>
type SearchParams = Promise<{ [key: string]: string | string[] | undefined }>

interface PageProps {
  params?: Params // 동적 라우트 세그먼트에 대한 정보
  searchParams?: SearchParams // 쿼리스트링 값에 대한 정보
}

export default function Page({ params, searchParams }: PageProps) {
  return <section>Page Content,,,</section>
}
```

### default.tsx

default.tsx 파일이 export default한 컴포넌트는 Slot(Parallel Routes)에 대한 fallback 컴포넌트 입니다.

페러렐 라우트를 사용하지 않는 하위 경로가 존재하는 경우에는 default.tsx 파일을 상위 경로에 추가해주어야 합니다. 그렇지 않으면 일치하는 슬롯이 존재하지 않을 때 Hard Navigation하는 경우 Layout 컴포넌트가 404 에러를 표시하게 됩니다.

예를 들어, 페러렐 라우트가 "/app/@team/settings/page.tsx"에만 존재하는 경우 "/"으로 라우팅되는 경우 "/app/@team/page.tsx"가 expoprt default한 컴포넌트를 찾지 못해 404에러가 나오게 됩니다. 이러한 경우 "/app/@team/default.tsx"를 추가하여 이와 같은 상황을 방지할 수 있습니다.

### loading.tsx

app 디렉토리 내 loading.tsx 파일이 export default한 컴포넌트는 Next가 자동으로 Page 컴포넌트를 Suspense로 래핑하고 fallback prop에 loading.tsx가 export default한 컴포넌트를 전달하게 됩니다.

Loading 컴포넌트는 Page 컴포넌트 내용을 로드하는 동안 Loading 컴포넌트를 렌더링하고, 이후 렌더링이 완료되면 Page 컴포넌트로 교체시켜줍니다.

```javascript
<Layout>
  <Suspense fallback={<Loading />}>
    <Page />
  </Suspense>
</Layout>
```

위와 같이 Page 컴포넌트가 자동으로 Suspense 컴포넌트로 래핑되며 fallback prop으로 Loading 컴포넌트를 전달하고 있습니다. 즉, Loading 컴포넌트는 페이지 전체에 대한 fallback 컴포넌트로서 동작하게 됩니다.

주의할 점으로는 Loading 컴포넌트는 Hard Navigation의 경우에만 표시되며 Soft Navigation의 경우에는 표시되지 않습니다. 즉, loading.tsx는 전체 페이지 로드 시에만 자동으로 동작하며 Soft Navigation을 사용하는 경우에는 수동으로 로딩 상태를 관리해야 합니다.

### error.tsx

error.tsx 파일이 export default한 컴포넌트는 Next가 자동으로 Page 컴포넌트를 ErrorBoundary 컴포넌트로 래핑하고 fallback prop에 error.tsx가 export default한 컴포넌트를 전달하게 됩니다.

```javascript
<Layout>
  <ErrorBoudnary fallback={<Error />}>
    <Page />
  </ErrorBoudnary>
</Layout>
```

Error 컴포넌트는 필수적으로 "use client";를 작성하여 RCC로서 사용해야 합니다. 이는 서버와 클라이언트 에러 모두를 캐치하기 위해서 클라이언트 컴포넌트로 사용해야 합니다. 클라이언트 컴포넌트는 서버와 클라이언트 양쪽에서 실행되지만 서버 컴포넌트는 서버에서만 실행되기 때문입니다.

Error 컴포넌트는 props로 error와 reset을 전달받습니다. error로 발생한 에러 객체를 전달받으며, reset 함수를 실행하면 Next는 리렌더링을 시도하며, 리렌더링 성공시 성공한 렌더링 결과로 교체시켜줍니다.

```javascript
'use client' // Error 컴포넌트는 필수적으로 RCC로 설정

interface ErrorProps {
  error: Error
  reset: () => void
}

export default function Error({ error, reset }: ErrorProps) {
  return (
    <section>
      An Error occurred: {error.message}<br />
      <button onClick={() => reset()}>Retry</button>
    </section>
  )
}
```

#### global-error.tsx

error.tsx는 layout.tsx나 template.tsx에서 발생한 에러는 캐치하지 않습니다. 만약 해당 에러를 캐치하려면 global-error.tsx를 추가해주어야 합니다. 전달받는 props는 error.tsx의 Error 컴포넌트와 동일합니다.

### not-found.tsx

not-found.tsx 파일은 매칭되는 라우트 세그먼트가 존재하지 않는 경우에 표시할 컴포넌트 입니다. 

추가적으로 "next/navigation"의 notFound 함수를 호출하면 not-found.tsx 파일에서 export default된 컴포넌트가 렌더링됩니다.

### Route Handlers

Route Handlers는 요청 하면 응답으로 페이지가 아닌 JSON을 응답으로 전달해주는 API 역할을 합니다. 즉, Route Handler는 서버에서 실행되는 함수로 쿠키나 헤더에 접근할 수 있습니다.

Route Handlers는 첫 번째 인수로 요청 객체(NextRequest)를 전달받고, 두 번째 인수로는 동적 라우팅의 경우 params라는 프로퍼티를 갖는 객체를 전달받으며 params 프로러티는 동적 경로에 대한 정보를 객체로서 전달받습니다.

Route Handlers는 "app/api" 디렉토리 내 추가할 수 있으며, 이후 중첩된 디렉토리명이 path값 일부로 사용됩니다. 또한 파일 네이밍은 "route.ts"로 추가해주어야 합니다.

예를 들어, "app/api/items/[id]/route.ts"라는 파일을 추가하면 요청할 때는 "/api/items/1"로 요청할 수 있게 됩니다.

Route Handlers가 지원하는 HTTP Method로는 GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS을 지원하며 지원합니다. route.ts 파일은 지원하는 메서드명을 갖는 함수를 export해야 하며 async 함수로 정의하여 API를 추가할 수 있습니다.

```javascript
// app/api/route.ts
import { NextRequest, NextResponse } from 'next/server'

type Params = Promise<{ slug: string }>

export async function GET(request: NextRequest, context: { params?: Params }) {
  try {
    const requestBody = await request.json() // 요청 body 값

    const responseBody = { success: true }

    return NextResponse.json(responseBody, { status: 200 })
  } catch (error) {
    return NextResponse.json('Fail to fetch data', { status: 500 })
  }
}
```

### Middleware

미들웨어란 실제 요청을 거치기 전에 특정 역할을 수행하기 위해 사용합니다. 미들웨어를 추가하기 위해서는 프로젝트 루트 경로에 "middleware.ts"라는 네이밍으로 파일을 추가해주어야 합니다. 주의할 점으로 페이지나 API뿐만 아니라 모든 리소스에 대한 요청들도 미들웨어를 거치게 됩니다.

middleware 함수는 async 함수로 정의할 수 있으며, 인수로 요청 객체(NextRequest)를 전달받습니다.

```javascript
import { NextRequest, NextResponse } from 'next/server'

export async function middleware(request: NextReqeust) {
  return NextResponse.redirect(new URL('/home', request.url))
}

export const config = {
  matcher: ['/about/:path', '/dashboard/:path']
}
```

미들웨어 설정의 경우에는 config라는 객체를 export하여 설정할 수 있으며, config.matcher를 통해 미들웨어를 실행할 특정 경로를 설정할 수 있습니다. 위 예제의 경우에는 "/about" 이하 모든 경로와 "/dashboard" 이하 모든 경로에 대해서 미들웨어가 실행됩니다. matcher는 정규표현식으로도 작성 가능합니다.

주의할 점으로는 config 객체 자체는 빌드타임에 실행되어 적용되기 때문에 동적인 값은 사용할 수 없습니다.

## NextRequest & NextResponse

"next/server"가 제공하는 NextRequest와 NextResponse는 Web Request/Response API를 확장한 것입니다.

```javascript
import { NextRequest, NextResponse } from 'next/server'

const request = new NextRequest()
const response = NextResponse.next()

// 요청/응답 cookie 값을 set 시켜줍니다.
request.cookies.set('key', 'value')
response.cookies.set('key', 'value')

// 매칭된 cookie 값을 반환합니다.
// 매칭된 cookie가 없는 경우 undefined를 반환하고, 여러 개가 매칭된 경우 첫 번째로 매칭된 cookie를 반환합니다.
request.cookies.get('key')
response.cookies.get('key')

// 매칭된 모든 cookie 값을 배열에 담아 반환합니다.
request.cookies.getAll('key')
response.cookies.getAll('key')

// 매칭된 cookie 값을 제거합니다.
// 반환값은 제거 성공 여부를 불리언 값으로 반환합니다.
request.cookies.delete('key')
response.cookies.delete('key')

// 매칭된 cookie 값 존재 여부를 불리언 값으로 반환합니다.
request.cookies.has('key')

// 요청의 Set-Cookie 헤더를 제거합니다.
request.cookies.clear()

// URL 도메인을 반환합니다.
request.nextUrl.baseUrl

// URL path값을 반환합니다.
request.nextUrl.pathname

// URL 쿼리 스트링 값을 객체로 반환합니다.
request.nextUrl.searchParams

// JSON을 body로 갖는 응답을 생성합니다.
NextResponse.json({ success: true }, { status: 200 })

// 특정 URL로 redirect시키는 응답을 생성합니다.
// 클라이언트측에서 해당 응답을 전달받게 되면 "/home" 경로로 이동하게 됩니다.
NextResponse.redirect(new URL('/home', reqeust.url))

// rewrite 메서드는 route handler에서 사용할 수 없고, next middleware에서 사용할 수 있습니다.
// redirect와는 다르게 요청한 URL path값은 변경하지 않고, 다른 페이지나 route handler로 요청을 전달할 수 있습니다.
NextResponse.rewrite(new URL('/proxy', reqeust.url))

// next 메서드는 route handler에서 사용할 수 없고, next middleware에서 사용할 수 있습니다.
// next 메서드는 요청을 중단하지 않고 다음 단계로 넘길 수 있습니다. 즉, 미들웨어 이후 실제 요청을 이어서 진행하도록 도와줍니다.
NextResponse.next()
```

## Route Segment Config

layout.tsx, page.tsx, Route Handlers에는 Route Segment 옵션을 설정할 수 있습니다.

Route Segment 옵션을 통해 데이터 Next 서버에 캐싱되어 있는 Data Cache와 Full Route Cache를 다룰 수 있습니다.

### dynamic

- "auto"(default): Next가 페이지에서 사용되는 테이터 소스와 fetch 요청을 분석하여 자동으로 페이지 생성 방식을 정적 혹은 동적으로 결정하게 됩니다.

- "force-dynamic": Next가 페이지 생성 방식을 동적 생성으로 강제하며(Full Route Cache), fetch 요청에 대한 데이터를 캐싱하지 않고 언제나 최신 데이터를 사용하게 됩니다(Data Cache). 즉, 페이지 요청할 때마다 fetch로 가져온 최신 데이터 기반으로 생성한 페이지를 클라이언트측에 전달합니다.

- "force-static": Next가 페이지 생성 방식을 정적 생성으로 강제하며(Full Route Cache), fetch 요청에 대한 데이터도 언제나 캐싱하여 사용하게 됩니다(Data Cache). 즉, fetch에 대한 응답값을 Next 서버에 캐싱된 데이터를 재사용하여 빌드 타임때 생성한 페이지를 클라이언트측에 전달합니다.

```javascript
export const dynamic = "force-dynamic"

export const dynamic = "force-static"

// ,,,
```

### dynamicParams

- true(default): 다이나믹 라우팅 처리가 런타임에 클라이언트가 요청된 경로로 동적으로 해석되어 처리됩니다.

- false: 동적 경로가 런타임이 아닌 빌드 타임때 결정되기 때문에 generateStaticParams라는 함수를 export하여 빌드 타임에 생성될 동적 경로 정보를 작성해주어야 합니다.

```javascript
export const dynamicParams = false

type Params = Promise<{ slug: string }>

interface PageProps {
  params?: Params
}

export default function Page({ params }: PageProps) {
  // ,,,
}

export async function generateStaticParams() {
  const paths = [{ params: { slug: 'post-1' } }, { params: { slug: 'post-2' } }]

  return { paths, fallback: false }
}
```

### revalidate

Data Cache와 Full Route Cache의 캐싱 지속시간을 설정할 수 있습니다.

- false(default): fetch의 응답 데이터는 Next 서버의 Data Cache에 캐싱된 데이트를 재사용하고, 페이지는 빌드 타임때 생성한 페이지를 사용하게 됩니다.

- number: Next 서버의 Data Cache에 캐싱된 fetch 응답값과 Full Route Cache의 지속시간을 초 단위로 설정할 수 있습니다.

```javascript
export const revalidate = 3600 // 60 seconds

// ,,,
```

### fetchCache

fetch 요청에 대한 캐싱 요청을 라우트 세그먼트 별로 제어할 수 있습니다. 다시 말해, fetch의 Data Cache 설정을 라우트별로 지정하는 옵션입니다.

- "auto"(default): cache 옵션을 명시하지 않은 fetch에 대해 cache 옵션을 "no-store"으로 설정합니다.

- "default-cache": fetch의 cache 옵션이 명시되어 있지 않은 fetch에 대해 cache 옵션을 "force-cache"로 설정합니다.

- "force-cache": 모든 fetch의 Data Cache 사용을 강제하며 fetch의 cache 옵션을 "force-cache" 설정으로 강제합니다.

- "only-cache": 모든 fetch의 Data Cache 사용을 강제하며 fetch의 cache 옵션을 "force-cache" 설정으로 강제합니다. 추가적으로 "no-store"를 사용하는 fetch에 대해서는 에러를 발생시킵니다,

- "default-no-store": fetch의 cache 옵션이 명시되어 있지 않은 fetch에 대해 cache 옵션을 "no-store"로 설정합니다.

- "force-no-store": 모든 fetch의 Data Cache를 사용하지 않으며 fetch의 cache 옵션을 "no-store" 설정으로 강제합니다.

- "only-no-store": 모든 fetch의 Data Cache를 사용하지 않으며 fetch의 cache 옵션을 "no-store" 설정으로 강제합니다. 추가적으로 "force-cache"를 사용하는 fetch에 대해서는 에러를 발생시킵니다.

## Linking and Navigating

### Link

"next/link"가 제공하는 Link 컴포넌트는 html a 태그를 확장한 컴포넌트로서 prefetching 기능까지 제공합니다. Link 컴포넌트는 RSC와 RCC 둘 다 사용할 수 있습니다.

Link 컴포넌트트는 a 태그를 확장한 컴포넌트로 a 태그에 작성 가능한 어트리뷰트들을 그대로 작성할 수 있습니다.

```javascript
import Link from 'next/link'

export default function Page() {
  return (
    <>
      <Link href='/item' />
    </>
  )
}
```

### useRouter

"next/navigation"가 제공하는 userRouter 훅이 반환하는 객체를 통해서 Soft Navigating을 사용할 수 있습니다. useRouter는 리액트 훅으로 RCC에서만 사용할 수 있습니다.

```javascript
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  // History stack에 하나의 스택을 push 하고 이동합니다.
  router.push('/items')

  // 현재 URL에 대해 refresh를 수행합니다.
  router.refresh()

  // 특정 URL을 prefetching하여 더 빠른 Navigating을 제공합니다.
  router.prefetch('/item')

  // History stack에서 하나의 스택을 pop 하고 이동합니다.
  router.back()

  // History stack에서 하나의 스택 앞으로 이동합니다.
  router.forward()

  // ,,,
}
```

### permanentRedirect

"next/navigation"이 제공하는 permanentRedirect 함수는 RCC, RSC, Route Handler, Server Action 모두 사용 가능합니다.

```javascript
import { permanentRedirect } from 'next/navigation'

export default function Page() {
  permanentRedirect('/login', { type: 'replace' })

  // ,,,
}
```

permanentRedirect 함수 첫 번째 인수로는 URL을 전달하고 두 번째 인수로는 객체를 전달할 수 있으며 type 프로퍼티에 "replace"(default) 혹은 "push"를 전달할 수 있습니다.

주의할 점으로 Server Actions에서 사용할 경우에는 type의 default가 push로 동작하게 됩니다.

### redirect

"next/navigation"이 제공하는 redirect 함수는 RSC, Route Handler, Server Action에서만 사용 가능합니다.

```javascript
import { redirect } from 'next/navigation'

export default function Page() {
  redirect('/login', { type: 'replace' })

  // ,,,
}
```

redirect 함수 첫 번째 인수로는 URL을 전달하고 두 번째 인수로는 객체를 전달할 수 있으며 type 프로퍼티에 "replace"(default) 혹은 "push"를 전달할 수 있습니다.

### notFound

"next/navigation"이 제공하는 notFound 함수 호출 시 not-found.tsx 파일에서 export default된 컴포넌트가 렌더링됩니다.

## Functions

### cookie(Dynamic Function)

"next/headers"가 제공하는 cookies 함수는 RSC, Route Handler, Server Action에서 사용 가능한 함수로 요청 객체의 쿠키 값을 읽을 수 있습니다.

```javascript
import { cookies } from 'next/headers'

export default async function Page() {
  // 요청 객체에 대한 쿠키
  const cookieStore = await cookies()

  // 인수로 전달한 쿠키 이름과 매칭된 쿠키값을 반환합니다. 매칭된 쿠키가 없는 경우 undeinfed를 반환합니다.
  // 매칭된 결과가 여러 개인 경우에도 하나만 반환합니다.
  cookieStore.get('key')

  // 인수로 전달한 쿠키 이름과 매칭된 쿠키값들을 반환합니다.
  // get 메서드와는 다르게 매칭된 모든 쿠키값들을 요소로 갖는 배열로 반환합니다.
  cookieStore.getAll('key')

  // 인수로 전달한 쿠키 이름과 매칭된 쿠키값 존재 여부를 불리언 값으로 반환합니다.
  cookieStore.has('key')

  // ,,,
}
```

```javascript
import { cookies } from 'next/headers'

export async function GET() {
  // 요청 객체에 대한 쿠키
  const cookieStore = await cookies()

  // 요청 객체의 쿠키값을 set할 수 있습니다.
  // 주의할 점으로 set 메서드는 Server Action와 Route Handler에서만 사용할 수 있습니다.
  cookieStore.set('key', 'value')
  cookieStore.set({ name: 'key', value: 'value' })

  // 요청 객체의 쿠키값을 제거할 수 있습니다.
  // 주의할 점으로 delete 메서드는 Server Action와 Route Handler에서만 사용할 수 있습니다.
  cookieStore.delete('key')

  // ,,,
}
```

### headers(Dynamic Function)

"next/headers"가 제공하는 headers 함수는 RSC, Route Handler, Server Action에서 사용 가능한 함수로 요청 헤더 값을 읽을 수 있습니다.

headers 함수가 반환하는 헤더 값은 읽기 전용으로 set, delete와 같은 동작은 할 수 없습니다.

```javascript
import { headers } from 'next/headers'

export default async function Page() {
  // 요청 객체에 대한 헤더
  const headersList = await headers()

  // headers 값을 key, value로 갖는 이터레이터 객체를 반환합니다.
  headersList.entries()

  // 인수로 전달한 콜백은 key, value로 갖는 객체를 인수로 전달받아 실행됩니다.
  headersList.forEach()

  // 인수로 전달한 key와 매칭된 value를 반환합니다.
  headersList.get();

  // 인수로 전달한 key와 매칭된 value 여부를 불리언 값으로 반환합니다.
  headersList.has();

  // key 값을 갖는 이터레이터 객체를 반환합니다.
  headersList.keys();

  // value 값을 갖는 이터레이터 객체를 반환합니다.
  headersList.values();
}
```

### fetch

Next.js는 Web fetch API를 확장하여 제공합니다. fetch 자체는 Web API로 서버 사이드에서 사용할 수 없지만 Next에서는 RSC, Server Actions, Route Handlers 등 모두 사용할 수 있습니다.

```javascript
export default async function Page() {
  // no-store: 기본 캐싱 동작이며 항상 최신 데이터를 가져옵니다 (Data Cache 비활성)
  await fetch('https://,,,', { cache: 'no-store' })

  // force-cache: 이전에 요청하여 받은 응답 데이터가 Next 서버에 캐싱되어 있다면 이를 재사용합니다 (Data Cache 활성)
  // 이는 최신 데이터를 반영하지 않을 수 있습니다.
  await fetch('https://,,,', { cache: 'force-cache' })

  // revalidate 옵션을 통해 cache life time을 명시할 수 있습니다.
  // 초 단위 숫자값을 작성하여 cache life time을 지정할 수 있습니다. 즉, revalidate 값을 0은 no-store이며 양수값을 작성한 경우 force-cache를 암시합니다.
  await fetch('https://,,,', { next: { revalidate: 10 } })

  // ,,,
}
```

즉, Next는 fetch 요청에 cache 옵션을 명시하지 않은 경우 기본적으로 "no-store"가 기본값으로 적용되어 Data Cache를 사용하지 않습니다.

fetch 요청에 Data Cache 활성을 설정하기 위한 방법으로는 아래와 같습니다.

- fetch의 cache 옵션을 "force-cache"로 설정

- Route Segment Config의 dynamic을 "force-static"으로 설정

- Route Segment Config의 fetchCache를 "force-cache", "only-cache" 혹은 "default-cache"로 설정

> 만약 dynamic 옵션을 "force-dynamic"으로 설정하고 fetchCache를 "force-cache", "only-cache" 혹은 "default-cache"로 설정하면 페이지는 매번 동적으로 생성되지만 fetch 요청을 Next 서버에 캐싱된 데이터를 사용하여 페이지를 만들게 되고, dynamic 옵션을 "force-static" fetchCache옵션을 "force-no-store", "only-no-store" 혹은 "default-no-store"로 설정하면 페이지는 정적으로 생성되지만 fetch 요청 데이터는 최신 데이터를 사용합니다.

### revalidatePath

"next/cache"가 제공하는 revalidatePath 함수는 특정 라우트에 대한 Data Cache와 Full Route Cache 모두 무효화하고 재검증합니다.

revalidatePath를 호출하면 해당 경로에서 Next 서버에 캐싱된 fetch 응답 데이터와 빌드 타임때 생성되어 캐싱된 페이지를 무효화시킵니다.
이후 다음 번에 사용자가 해당 경로를 요청한 경우 Next가 새로운 데이터를 가져와 페이지를 다시 생성하여 응답으로 전달해줍니다. 이 과정에서 새롭게 생성된 정적 파일과 데이터가 캐시에 저장됩니다.

revalidatePath 함수는 Route Handlers나 Server Actions에서 호출 가능합니다.

```javascript
import { NextRequest, NextResponse } from 'next/server'
import { revalidatePath } from 'next/cache'

export async function POST(request: NextRequest) {
  const { path } = request.nextUrl.searchParams('path')

  if (path) {
    revalidatePath(path, 'page')

    return NextResponse.json({ revalidated: true, now: Date.now() })
  } else {
    return NextResponse.json({ revalidated: false, now: Date.now() })
  }
}
```

revalidatePath 함수 첫 번째 인수로는 무효화할 라우트 세그먼트를 작성하고, 두 번째 인수로는 "layout" 혹은 "page"를 전달합니다.

- "layout": "layout" 전달 시 첫 번째 인수로 전달한 라우트뿐만 아니라 하위 라우트까지 모두 재검증하게 됩니다.

- "page"(default): "page" 전달 시 첫 번째 인수로 전달한 라우트만 재검증하게 됩니다,

즉, "layout"이 "page"보다 재검증하는 범위가 더 큽니다.

### useParams

"next/navigation"이 제공하는 useParams 훅은 동적 라우팅하는 경우 동적으로 결정된 path값을 객체 형태로 반환합니다.

```javascript
'use client'

import { useParams } from 'next/navigation'

export default function ClientComponent() {
  const params = useParams()

  // ,,,
}
```

### usePathname

"next/navigation"이 제공하는 usePathname 훅은 현재 URL의 path 값을 string으로 반환합니다.

```javascript
'use client'

import { usePathname } from 'next/navigation'

export default function ClientComponent() {
  const pathanem = usePathname()

  // ,,,
}
```

### useSearchParams

"next/navigation"이 제공하는 useSearchParams 훅은 현재 URL의 쿼리스트링 정보를 읽기 전용인 URLSearchParams 객체를 반환합니다.

```javascript
'use client'

import { useSearchParams } from 'next/navigation'

export default function ClientComponent() {
  const searchParams = useSearchParams()

  // 쿼리스트링의 value 값을 요소로 갖는 배열을 반환합니다.
  searchParams.getAll()

  // 쿼리스트링의 key 값을 요소로 갖는 이터레이터 객체를 반환합니다.
  searchParams.keys()

  // 쿼리스트링의 value 값을 요소로 갖는 이터레이터 객체를 반환합니다.
  searchParams.values()

  // 쿼리스트링의 key, value 값을 요소로 갖는 배열을 요소로 갖는 이터레이터 객체를 반환합니다.
  searchParams.entries()

  // 인수로 전달한 콜백은 key, value 값을 순차적으로 전달받으며 실행됩니다.
  searchParams.forEach((key, value) => {
    // ,,,
  })

  // ,,,
}
```

### useSelectedlayoutSegment && useSelectedlayoutSegments

"next/navigation"이 제공하는 useSelectedLayoutSegment 훅은 현재 활성화된 한 단계 하위 라우트 세그먼트 반환합니다. useSelectedLayoutSegments 훅은 현재 활성화된 모든 라우트 세그먼크 값을 요소로 갖는 배열을 반환합니다.

예를 들어, 현재 URL path 값이 "/blog/hello-world"인 경우 useSelectedLayoutSegment 훅은 "hello-world"를 반환하고, useSelectedLayoutSegments 훅은 ["blog", "hello-world"]를 반환합니다.

## Server Actions

Server Actions은 React에 내장된 기능이며 form 제출 기능을 제공합니다. Server Actions은 함수이며 컴포넌트 내 직접 정의하거나 별도의 파일로 분리하여 import하여 사용할 수 있습니다. Server Actions 함수를 form 태그의 action 어트리뷰트에 전달하여 사용할 수 있습니다.

Server Actions 함수는 인수로 FormData 객체를 전달받습니다. 주의할 점으로 formData 객체로 폼 데이터에 접근하기 위해서는 form 내부 각 input들은 name 어트리뷰트를 갖고 있어야 합니다. 폼 데이터들을 접근할 때 input의 name 어트리뷰트 값으로 접근합니다.

추가적으로 Server Action 함수 async 함수로 정의되어야 하며, 함수 코드 블록 최상단에 "use server" 선언문을 작성해주어야 하며 만약 분리된 파일로 정의된 경우 파일 최상단에 "use server"를 작성할 수 있습니다.

Server Actions 함수는 Next 서버에서 실행되는 함수이므로 클라이언트측 로직은 사용할 수 없으며 Server Action 함수 자체도 클라이언트 컴포넌트 내에서는 정의할 수 없습니다.

```javascript
export default function Page() {
  // Server Action
  async function create(formData: FormData) {
    'use server'

    // ,,,
  }

  return <form action={create}>,,,</form>
}
```

### useFormStatus

useFormStatus 훅은 "react-dom"이 제공하는 클라이언트 훅으로 form 제출에 대한 정보를 제공합니다. useFormStatus 훅을 사용하는 컴포넌트는 form 태그를 갖는 컴포넌트 자식으로 작성되어야 합니다.

```javascript
'use client'

import { useFormState } from 'react-dom'

export default function SubmitButton() {
  const { pending } = useFormState()

  return (
    <button type='submit' disabled={pending}>
      Add
    </button>
  )
}
```

### useActionState

useActionState 훅은 "react"가 제공하는 클라이언트 훅으로 Server Actions가 반환하는 값에 접근할 수 있습니다. useFormStatus 훅과는 다르게 form 태그를 갖는 컴포넌트 내 작성할 수 있습니다.

useActionState 훅 첫 번째 인수로는 Server Actions 함수를 전달하고 두 번째 인수로는 Server Actions가 반환하는 값의 초기값을 전달해주어야 합니다.
첫 번째 인수로 전달하는 Server Actions은 첫 번째 인수로 이전 Server Actions가 반환한 값을 전달받으며, 두 번째 인수로는 FormData 객체를 전달받습니다.

useActionState 훅은 배열을 반환하며 배열의 첫 번째 요소는 Server Actions가 반환하는 값, 두 번째 요소는 React가 제어하는 Server Actions를 반환합니다. 이때 두 번째 요소로 반환한 Server Action을 form 태그의 action 어트리뷰트로 전달하면 React가 Server Actions가 반환하는 값에 접근할 수 있게 됩니다.

```javascript
'use client'

import { useActionState } from 'react'

import { createUser } from '@/app/actions'

const initState = {
  message: ''
}

export default function SignUp() {
  const [state, formAction] = useActionState(createUser, initState);

  return <form action={formAction}>,,,</form>
}
```

## Caching

### Request Memoization

Request Memoization은 동일한 라우트에서 동일한 설정을 갖는 fetch Request를 중복 요청하지 않고 단일 요청으로 전달하게 됩니다. 즉, 동일한 라우트 내 여러 다른 서버 컴포넌트에서 동일한 설정을 같는 fetch를 호출하더라도 단일 요청으로 전달됩니다. 이는 페이지 첫 렌더링 시점에만 이루어집니다.

예를 들어, Layout과 Page 컴포넌트가 서버 컴포넌트로 작성되었고, 두 컴포넌트 모두 동일한 설정을 갖는 fetch 함수를 호출하게 되면 요청이 두 번 전달되지 않고 단일 요청으로 전달됩니다.

> Request Memoization은 GET 메서드인 fetch에만 적용되며 서버 컴포넌트에서의 fetch에만 해당됩니다. 즉, Route Handlers에서의 fetch는 해당되지 않습니다.

### Data Cache

Data Cache란 Next가 fetch 함수를 통해 서버에서 가져온 응답 데이터를 Next 서버측에 캐싱하고 응답 데이터를 재사용합니다. 즉, Next 서버에서 백엔드로 보내는 요청에 대해서 Next 서버가 해당 요청에 대한 응답 데이터를 캐싱하게 됩니다.

명시적으로 Next 서버에 캐싱된 응답 데이터를 무효화하고 재검증하기 위해 아래와 같은 방법을 사용할 수 있습니다.

- Route Segment Config: layout.tsx, page.tsx, Route Handlers에 dynamic, revalidate, fetchCache 옵션을 사용하여 라우트 전체에 대한 Data Cache 설정할 수 있습니다.

- revalidatePath('/,,,', 'page' | 'layout'): Route Handlers나 Server Actions에서 특정 라우트 세그먼트를 전달하여 Data Cache를 무효화하고 재검증할 수 있습니다.

- fetch('https://,,,', { cache: 'no-store' }): cache 옵션에 "no-store"를 설정하여 Data Cache를 비활성할 수 있습니다.

- fetch('https://,,,', { next: { revalidate: number }}): next.revalidate 옵션으로 Data Cache의 life time을 초단위로 작성할 수 있습니다.

> 어떠한 설정도 하지 않은 경우 fetch의 Data Cache는 비활성이 기본값입니다.

### Full Route Cache

기본적으로 Next는 빌드 타임때 페이지를 미리 생성하고 렌더링된 결과를 Next 서버에 캐싱합니다. 이후 클라이언트가 페이지를 요청하면 Next 서버에 캐싱된 렌더링 결과를 전달하게 됩니다.

만약 페이지 생성 방식을 명시적으로 변경하고 싶다면 아래와 같은 방법을 사용할 수 있습니다.

- Route Segments Config: layout.tsx, page.tsx, Route Handlers에 dynamic 혹은 revalidate 옵션을 사용하여 페이지 생성 방식을 설정할 수 있습니다.

- revalidatePath('/,,,', 'page' | 'layout'): Route Handlers나 Server Actions에서 특정 라우트 세그먼트를 전달하여 페이지 생성 시점을 설정할 수 있습니다.

> Dynamic Functions(cookies, headers), searchParams prop 혹은 Data Cache를 사용하지 않는 경우 해당 페이지는 Full Route Cache가 동작하지 않게 됩니다. 즉, Full Route Cache가 동작하는 경우는 Data Cache가 사용하며 Dynamic Functions나 searchParams porp을 사용하지 않는 경우에만 Full Route Cache가 동작하게 됩니다.

### Route Cache

Next는 클라이언트측 인메모리에 page.tsx가 export default한 Page 컴포넌트들을 캐싱하고 있습니다. 즉, Route Cache는 Next 서버가 아닌 클라이언트측에서 이루어지는 캐싱입니다.

기본적으로 Route Cache는 비활성 상태이며 이를 활성화시키기 위해서는 아래와 같은 설정이 추가적으로 필요합니다.

```javascript
// next.config.js

/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    staleTimes: {
      dynamic: 30,
      static: 180,
    },
  },
}
 
module.exports = nextConfig
```

- dynamic: Link의 prefetch 속성이 지정되지 않았거나 false로 설정된 Page 컴포넌트의 Route Cache 시간을 초단위로 작성합니다.

- static: Link의 prefetch 속성이 true로 설정되었거나 router.prefetch한 Page 컴포넌트의 Route Cache 시간을 초단위로 작성합니다.

## Optimization

### Image Optimization

"next/image"가 제공하는 Image 컴포넌트를 통해 이미지를 최적화할 수 있습니다.

- 사이즈 최적화: Next 서버는 자동으로 이미지 파일의 크기나 포맷을 최적화하여 제공해줍니다.

- CLS 방지: 이미지가 로드될 때 layout shift 현상을 방지시켜줍니다.

- lazy loading: 전반적으로 이미지 로드는 lazy load가 적용되어 페이지 로드 시간을 단축시킵니다. 즉, 이미지가 viewport에 표시될 때만 이미지를 로드하고 렌더링합니다.

#### Remote Images

프로젝트의 public 폴더 내 파일들은 빌드된 이후 루트 경로에 존재하게 됩니다.

예를 들어, "public/images/logo.png"라는 파일은 빌드된 이후에는 "images/logo.png" 경로에 존재하게 됩니다. 그러므로 실제 src props에 작성될 경로 또한 "images/logo.png"로 작성해주어야 합니다.

Remote Images를 사용하는 경우에는 반드시 width와 height를 명시해주어야 합니다. 이는 빌드 타임에 해당 이미지의 width, height 값을 Next는 알지 못하기 때문에 직접 명시해주어야 합니다.

<hr />

외부 이미지 경로를 작성하는 경우에는 width, height 값을 명시해주어야 하며 추가적으로 next.config.js파일에 외부 경로의 도메인을 명시해주어야 합니다.

```javascript
// next.config.js

module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 's3.amazonaws.com',
        port: '',
        pathname: '/my-bucket/**'
      }
    ]
  }
};
```

#### Local Images

이미지 파일을 import하여 src에 작성한 경우 Next가 빌드 타임에 자동으로 width, height 값을 계산하여 적용하기 때문에 따로 width, height 값을 명시해줄 필요가 없습니다.

#### Props

- src: 이미지 경로나 import한 파일을 작성합니다.

- alt: 대체 텍스트를 작성합니다.

- width: 렌더링될 이미지의 가로 사이즈(px)를 작성합니다.

- height: 렌더링될 이미지의 세로 사이즈(px)를 작성합니다.

> fill props가 true 혹은 Local Images를 사용하는 경우에는 width, height를 생략할 수 있습니다.

- fill: 부모 요소 크기에 맞게 자동으로 이미지 크기 조정 여부를 작성합니다. 이때 부모 요소에는 position 프로퍼티 값으로 "relative", "absolute", "fixed" 값 중 하나를 작성해주어야 합니다.

> fill props로 true를 작성한 경우 해당 이미지의 "position" 값이 자동으로 "absolute"로 설정되기 때문에 부모 요소의 "position"값을 "relative", "absolute", "fixed" 값 중 하나 작성해주어야 합니다.

- szies: fill props이나 반응형 이미지를 사용하는 경우에 유용하게 사용되며 Next가 자동으로 생성한 srcset 정보를 기반으로 로드될 이미지 크기를 결정하는데 사용됩니다.

  - 값으로는 미디어 쿼리 값을 작성하며, 디폴트 값으로는 "100vw"가 적용되어 있습니다. 즉, 디바이스의 viewport의 너비가 100vw일 때의 srcset 정보를 확인하여 이미지를 로드합니다.

> srcset에 대한 정보는 Next가 자동으로 설정하게 되며, 이를 수동으로 설정하기 위해서는 next.config.js에서 images.deviceSizes, images.imageSizes로 설정할 수 있습니다. fill props 값이 false인 경우에는 images.imageSizes를 통해 생성하고, true인 경우에는 images.deviceSizes를 통해 생성합니다.

- quality: 이미지의 퀄리티를 1에서 100사이 값으로 작성합니다. 기본값은 75로 적용되어 있습니다.

- priority: 이미지 로드 preload 여부를 작성합니다. 기본값은 false이며, true로 설정한 경우 lazy loading 또한 비활성화 됩니다.

- placeholder: 이미지가 로드되는 동안 표시할 placeholder 설정을 작성합니다.

  - 기본값은 "empty"이며, 렌더링되는 동안 해당 영역을 빈 영역으로 비워둡니다.

  - "blur"를 작성한 경우에는 blurDataURL props도 함께 작성해주어야 하며 blurDataURL에 base64로 인코딩된 이미지 데이터를 작성하면 해당 이미지를 표시해줍니다.

- blurDataURL: placeholder props값이 "blur"인 경우에 사용되며 base64로 인코딩된 이미지 데이터를 작성합니다. 이는 이미지가 로드되는 동안 표시할 이미지를 작성합니다.

### MetaData

Next는 메타데이터를 설정하기 위한 Metadata API를 제공합니다.

#### Static Metadata

layout.tsx 혹은 page.tsx에서 metadata라는 이름의 변수를 export하여 정적 메타데이터를 설정할 수 있습니다.

```javascript
// page.tsx

import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: ',,,',
  description: ',,,'
};

export default function Page() {
  // ,,,
}
```

#### Dynamic Metadata

"next"가 제공하는 generateMetadata 라는 함수를 export함으로써 동적 메타데이터를 설정할 수 있습니다. generateMetadata 함수는 async 함수로 정의해야하며 첫 번째 인수로 객체를 전달받고 두 번째 인수로는 상위 경로에 대한 메타데이터를 전달받습니다.

첫 번째 인수로 전달받는 객체에는 params, searchParams 프로퍼티가 존재하며 이는 동적 라우팅 세그먼트와 쿼리스트링에 대한 정보를 갖고 있습니다.

```javascript
// page.tsx
import type { Metadata, ResolvingMetadata } from 'next'

type Params = Promise<{ slug: string }>
type SearchParams = Promise<{ [key: string]: string | string[] | undefined }>

type Props = {
  params: Params,
  searchParams: SearchParams
};

export async function generateMetadata({ params, searchParams }: Props, parent: ResolvingMetadata) {
  return {
    title: ',,,',
    description: ',,,'
  };
}

export default function Page({ params, searchParams }: Props) {
  // ,,,
}
```
