# Next.js (v14.2.3)

## Routing

### App Directory

Next v13부터 app 디렉토리 구조에 따라 라우팅 라우팅되며 각 파일의 역할(기능)과 컨벤션이 지정되어 있습니다.

또한 기본적으로 모든 컴포넌트들은 RSC(React Server Component)로 동작합니다. 만약 RCC(React Client Component)로 설정하기 위해서는 파일 최상단에 "use client";를 작성해야 합니다.

```javascript
// app/page.tsx
'use client'; // RCC 컴포넌트 선언

export default function Page() {
  return <main>Content,,,</main>;
}
```

### Dynamic Routes

디렉토리명을 "[folderName]"처럼 대괄호로 깜싼 경우 기존 동적 라우팅으로 동작합니다.

동적 라우트 세그먼트는 page.tsx, layout.tsx, route.ts, generateMetadata 함수에 params prop으로 전달됩니다.

```javascript
// app/blog/[slug]/page.tsx
export default function Page({ params }: { params: { slug: string } }) {
  /**
   * route: "/blog/a" -> { params: { slug: 'a' } }
   * route: "/blog/b" -> { params: { slug: 'b' } }
   **/

  return <main>Content,,,</main>;
}
```

> 동적 라우팅이란 URL 경로값으로 페이지 컴포넌트 내 렌더링될 정보를 동적으로 결정하여 페이지를 렌더링합니다. 이는 하나의 페이지를 재사용하여 여러 페이지로 동작하기 위해서 사용합니다.

#### Catch-All Routes

디렉토리 명을 "[...forderName]"으로 생성하게 되면 매칭되는 라우트 세그먼트가 없는 경우 Catch-All Segments가 활성화 됩니다.

- Route: "/app/shop/[...slug]/page.tsx" -> URL: "/shop/a" -> params: { slug: ['a'] }

- Route: "/app/shop/[...slug]/page.tsx" -> URL: "/shop/a/b" -> params: { slug: ['a', 'b'] }

- Route: "/app/shop/[...slug]/page.tsx" -> URL: "/shop/a/b/c" -> params: { slug: ['a', 'b', 'c'] }

#### Optional Catch-All Routes

Catch-All Segments는 params가 1개 이상인 경우에만 활성화 되지만 "[[...forderName]]"으로 디렉토리명을 생성하면 해당 Catch-All Segments는 params가 0개인 경우에도 활성화 됩니다.

- Route: "/app/shop/[...slug]/page.tsx" -> URL: "/shop" -> params: {}

위 예제처럼 params가 0개인 경우 params는 빈 객체가 전달됩니다.

### Route Groups

일반적으로 app디렉토리 내 디렉토리명들은 라우트 세그먼트로 URL 경로와 매핑됩니다. 하지만 디렉토리명을 "(folderName)"처럼 소괄호로 감싼 경우 라우트 세그먼트로 설정되지 않으며 실제 라우팅에서 해당 경로는 무시됩니다.

예를 들어, "app/(marketing)/about" 경로에 생성된 페이지는 "/(marketing)/about"에서 "(marketing)"이 무시된 "/about"으로 라우팅됩니다.

즉, 소괄호로 디렉토리를 만들어 경로들의 그룹을 생성할 수 있으며, layout.tsx나 template.tsx를 통해 URL 경로에 영향을 주지 않고 경로 그룹별 layout을 설정할 때 유용하게 사용할 수 있습니다.

### Parallel Routes

디렉토링명을 "@folderName"로 작성한 경우 라우트 세그먼트가 아닌 슬롯으로 설정되며 실제로 해당 경로는 무시됩니다.

이는 하나의 레이아웃에 여러 page를 표시해주는 병렬 라우팅으로 동작합니다. 만약 "/app/@team/settings/page.tsx"가 존재할 때 "/"에서 "/settigns"로 이동하게 되면 "/app/page.tsx"가 렌더링된 상테에서 "/app/@team/settgins/page.tsx"도 병렬로 렌더링됩니다.

이때 "@forderName/page.tsx"에서 export한 컴포넌트의 경우 @forderName와 동일한 레벨에 있는 layout 컴포넌트 prop으로 전달됩니다. 만약 동일한 레벨에 없다면 가장 가까운 layout.tsx의 Layout 컴포넌트에게 전달됩니다. 이때 전달되는 prop 네이밍은 디렉토리명(forderName)으로 전달됩니다.

예를 들어, "/app/@modal/page.tsx"가 export default한 컴포넌트는 "/app/layout.tsx"가 export default한 컴포넌트에 modal prop으로 전달됩니다.

```javascript
// app/layout.tsx
import { ReactNode } from 'react'

interface LayoutProps {
  children: ReactNode // "app/page.tsx"에서 export default한 컴포넌트
  modal: ReactNode // "app/@modal/page.tsx"에서 export default한 컴포넌트
}

export default function Layout({ children, modal }: LayoutProps) {
  <>
    {children}
    {modal}
  </>
}
```

병렬 라우팅의 주의할 점으로는 아래와 같습니다.

- Soft Navigation하는 경우 Next는 현재 일치하는 슬롯이 없더라도 이전 활성화된 슬롯을 표시해줍니다.

- Hard Navigation하는 경우 Next는 먼저 매칭되는 슬롯을 확인하고 만약 없는 경우에는 default.tsx가 export default한 컴포넌트 렌더링을 시도합니다. 이때 default.tsx 파일조차 없다면 404에러가 발생하게 됩니다.

만약 "/app/@team/settings/page.tsx"가 존재하고 "/app/@analytics/page.tsx"만 존재할 때 Soft Navigation으로 "/settings"로 이동하는 경우 "/app/@team/settings/page/tsx"와 "/app/@analytics/page/tsx"가 렌더링되고, Hard Navigation하는 경우에는 "/app/@analytics/settings/page.tsx"를 찾지 못해 404 에러가 표시됩니다.

또한 설정한 슬롯과 동일한 이름의 라우트 세그먼트를 추가해서는 안됩니다. 예를 들어, "/app/@team/settings/page.tsx"가 존재할 때 "/app/settgins/page.tsx"가 존재해서는 안됩니다.

### Intercepting Routes

디렉토리명을 "(...)fortuneName", "(..)fortuneName" 혹은 "(.)fortuneName"로 작성한 경우 intercepting route로 동작합니다.

- (.): 동일한 레벨 라우트 세그먼트를 인터셉팅

- (..): 하나의 상위 레벨 라우트 세그먼트를 인터셉팅

- (..)(..): 두 개의 상위 레벨 라우트 세그먼트를 인터셉팅

- (...): app 디렉토리 레벨의 라우트 세그먼트를 인터셉팅

주의할 점으로 Route Groups, Slot(Parallel Routes), Private Routes 등 URL 경로에 영향을 주지 않는 것들은 무시되어 인터셉팅됩니다. 즉, file system이 아닌 route segment만을 고려하여 인터셉팅합니다.

또한 Intercepting Routes는 Soft Navigation의 경우에만 Intercepting되며 Soft Navigation이 아닌 경우 기존 페이지 컴포넌트가 렌더링됩니다. 예를 들어, "/dashboard/@modal/(.i)"가 인터셉팅되지 않는다면 "/dashboard/i"로 라우팅됩니다.

#### Parallel Routes & Intercepting Routes

Parallel Routes내 Intercepting Routes를 함께 사용하여 독립된 URL 경로를 갖는 모달을 구현할 수 있습니다. 독립된 URL 경로를 갖는 모달의 경우 URL을 통해 모달 내용을 공유할 수 있으며, 새로고침 하더라도 그 상태가 유지되고, 뒤로 가기 혹은 앞으로 가기를 통해 모달을 열거나 닫을 수도 있습니다.

```javascript
// app/@modal/(.)warning/page.tsx

import Modal from '@/shard/components/Modal';

export default function Page() {
  return (
    <Modal>
      <div>Warning,,,</div>
    </Modal>
  );
}
```

```javascript
// app/warning/page.tsx

export default function Page() {
  return <div>Warning,,,</div>;
}
```

"/warning"으로 Soft Navigation하는 경우에 Intercepting Routes로 지정한 "app/@modal/(.)warning/page.tsx"에서 export default한 컴포넌트가 "/app/layout.tsx" 컴포넌트 내 modal prop으로 전달되어 렌더링됩니다.

### layout.tsx

layout.tsx 파일에서 export default된 Layout 컴포넌트는 props로 page.tsx가 export default한 컴포넌트를 children prop으로 전달받습니다. 추가적으로 params prop도 전달받으며 이는 동적 라우팅에 대한 경로값을 객체 형태로 전달받으며, 동일한 레벨에 Parallel Routes가 존재한다면 해당 리렉토리명으로 컴포넌트를 prop으로 전달받습니다.

app 디렉토리의 루트 레벨에는 필수적으로 하나의 layout.tsx("app/layout.tsx") 파일이 필요하며, 해당 컴포넌트는 html과 body 태그를 정의해주저야 합니다.

app 디렉토리 내 각 하위 디렉토리 마다 하나씩 설정 가능하며 만약 하위 디렉토리에 layout.tsx가 존재하는 경우 상위 layout.tsx가 하위 layout.tsx를 포함하는 형식으로 적용(nested layout)됩니다.

주의할 점으로는 하위 경로로 Soft Navigation을 사용하는 경우 Layout 컴포넌트는 리렌더링 되지 않습니다. 만약 리렌더링이 필요한 Layout 컴포넌트가 필요한 경우 layout.tsx 대신 template.tsx 사용해야 합니다.

```javascript
// app/layout.tsx
import { ReactNode } from 'react'

interface LayoutProps {
  children: ReactNode // app/page.tsx에서 export default된 컴포넌트
  params: string // 다이나믹 라우팅되는 경우 동적 라우트 세그먼트 값
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

template.tsx는 layout.tsx와 유사하게 페이지 레이아웃 역할을 하지만 Layout 컴포넌트와는 다르게 Soft Navigation하는 경우 매번 새로웁게 마운트됩니다.

template.tsx와 layout.tsx를 같이 사용하게 된다면 아래와 같이 Layout 컴포넌트 하위에 Template 컴포넌트가 렌더링되고, Template 컴포넌트 하위에 Page 컴포넌트가 렌더링됩니다. 일반적으로 Layout과 유사한 역할을 하므로 하나만 사용하며, 되도록 특별한 이유가 없다면 Layout 컴포넌트를 사용해야 합니다.

```javascript
<Layout>
  {/* Note that the template is given a unique key. */}
  <Template key={routeParam}>{children}</Template>
</Layout>
```

Template 컴포넌트는 아래와 처럼 매번 마운트가 필요한 경우에 유용하게 사용할 수 있습니다.

- "use client"를 작성하여 클라이언트 컴포넌트로 설정하고 useEffect나 useState훅에 의존하는 기능이 필요한 경우 사용할 수 있습니다.

- next 기존 프레임워크 동작을 변경하고 싶을 때 사용할 수 있습니다. 예를 들어, Layout 컴포넌트 Suspense 컴포넌트를 사용하더라도 fallback 컴포넌트는 처음에만 표시되고 이후 라우팅되더라도 표시되지 않지만 Template 컴폰넌트는 라우팅될 때매다 표시해줍니다.

### page.tsx

page.tsx에서 export default한 컴포넌트는 layout.tsx(or template.tsx)에서 export default한 컴포넌트 children prop으로 전달됩니다. 즉, 실제 페이지 컨텐츠를 나타내는 컴포넌트입니다.

디렉토리마다 page.tsx 파일을 필수이며, 만약 page.tsx파일이 없다면 해당 경로로 라우팅되지 않습니다.

Page 컴포넌트는 동적 라우트의 경우 동적 라우트 세그먼트 값을 params prop으로 전달받으며, 쿼리스트링에 대한 정보는 searchParams prop으로 전달받습니다.

```javascript
// app/page.tsx

interface PageProps {
  params?: { [key: string]: string } // 동적 라우트 세그먼트에 대한 정보
  searchParams?: { [key: string]: string } // 쿼리스트링 값에 대한 정보
}

export default function Page({ params, searchParams }: PageProps) {
  return <section>Page Content,,,</section>
}
```

### default.tsx

default.tsx 파일이 export default한 컴포넌트는 Slot(Parallel Routes)에 대한 fallback 컴포넌트 입니다.

페러렐 라우트를 사용하지 않은 하위 경로가 존재하는 경우에는 default.tsx 파일을 상위 경로에 추가해주어야 합니다. 그렇지 않으면 Hard Navigation하는 경우 Layout 컴포넌트가 404 에러를 표시하게 됩니다.

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

주의할 점으로는 Loading 컴포넌트는 Hard Navigation의 경우에는 표시되지 않으며 Soft Navigation의 경우에만 표시됩니다. 이는 Page 컴포넌트가 RSC로서 서버에서 미리 실행하여 렌더링되기 때문에 비동기 처리를 기다리지 않습니다.

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
    <>
      An Error occurred: {error.message}
      <button onClick={() => reset()}>Retry</button>
    </>
  )
}
```

#### global-error.tsx

주의할 점으로는 layout.tsx나 template.tsx에서 발생한 에러는 캐치하지 않습니다. 만약 해당 에러를 캐치하려면 global-error.tsx를 추가해주어야 합니다. 전달받는 props는 error.tsx의 Error 컴포넌트와 동일합니다.

### not-found.tsx

not-found.tsx 파일은 매칭되는 라우트 세그먼트가 존재하지 않는 경우에 표시할 컴포넌트 입니다. 추가적으로 "next/navigation"의 notFound 함수를 호출하면 not-found.tsx 파일에서 export default된 컴포넌트가 렌더링됩니다. 