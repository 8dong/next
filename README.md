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
    <>
      An Error occurred: {error.message}
      <button onClick={() => reset()}>Retry</button>
    </>
  )
}
```

#### global-error.tsx

error.tsx는 layout.tsx나 template.tsx에서 발생한 에러는 캐치하지 않습니다. 만약 해당 에러를 캐치하려면 global-error.tsx를 추가해주어야 합니다. 전달받는 props는 error.tsx의 Error 컴포넌트와 동일합니다.

### not-found.tsx

not-found.tsx 파일은 매칭되는 라우트 세그먼트가 존재하지 않는 경우에 표시할 컴포넌트 입니다. 추가적으로 "next/navigation"의 notFound 함수를 호출하면 not-found.tsx 파일에서 export default된 컴포넌트가 렌더링됩니다.

### Route Handlers

Route Handler는 요청 하면 응답으로 페이지가 아닌 JSON을 응답으로 전달해주는 API 역할을 합니다. 즉, Route Handler는 서버에서 실행되는 함수로 쿠키나 헤더에 접근할 수 있습니다.

Route Handler는 첫 번째 인수로 요청 객체를 전달받고, 두 번째 인수로는 동적 라우팅의 경우 params라는 프로퍼티를 갖는 객체를 전달받으며 params 프로러티는 동적 경로에 대한 정보를 객체로서 전달받습니다.

Route Handler는 "app/api" 디렉토리 내 추가할 수 있으며, 이후 중첩된 디렉토리명이 path값 일부로 사용됩니다. 또한 파일 네이밍은 "route.ts"로 추가해주어야 합니다.
예를 들어, "app/api/items/[id]/route.ts"라는 파일을 추가하면 요청할 때는 "/api/items/1"로 요청할 수 있게 됩니다.

Route Handlers가 지원하는 HTTP Method로는 GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS을 지원하며 지원합니다. route.ts 파일은 지원하는 메서드명을 갖는 함수를 export해야 하며 async 함수로 정의하여 API를 추가할 수 있습니다.

```javascript
// app/api/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest, context: { params: { [key: string]: string } }) {
  try {
    const requestBody = await request.json(); // 요청 body 값

    const responseBody = { success: true };
    return NextResponse.json(responseBody, { status: 200 });
  } catch (error) {
    return NextResponse.json('Fail to fetch data', { status: 500 });
  }
}
```

참고로 GET 메서드의 Route Handlers를 Response 객체와 함께 사용할 때 Route Handlers의 응답값은 Next 서버의 Data Cache를 이용하여 캐싱됩니다. 하지만 아래와 같은 경우에는 캐싱되지 않습니다.

- Request 객체에 접근하는 경우

- 같은 파일 내 다른 HTTP Method 함수도 존재하는 경우

- cookies 혹은 headers와 같은 Dynamic Functions를 사용하는 경우

- Segment Config Options으로 캐싱 옵션을 명시한 경우

### Middleware

미들웨어란 실제 요청을 거치기 전에 특정 역할을 수행하기 위해 사용합니다. 미들웨어를 추가하기 위해서는 프로젝트 루트 경로에 "middleware.ts"라는 네이밍으로 파일을 추가해주어야 합니다. 주의할 점으로 페이지나 API뿐만 아니라 모든 리소스에 대한 요청들도 미들웨어를 거치게 됩니다.

middleware 함수는 async 함수로 정의할 수 있으며, 인수로 요청 객체를 전달받습니다.

```javascript
import { NextRequest, NextResponse } from 'next/server';

export async function middleware(request: NextReqeust) {
  return NextResponse.redirect(new URL('/home', request.url));
}

export const config = {
  matcher: ['/about/:path', '/dashboard/:path']
};
```

미들웨어 설정의 경우에는 config라는 객체를 export하여 설정할 수 있으며, config.matcher를 통해 미들웨어를 실행할 특정 경로를 설정할 수 있습니다. 위 예제의 경우에는 "/about" 이하 모든 경로와 "/dashboard" 이하 모든 경로에 대해서 미들웨어가 실행됩니다. matcher는 정규표현식으로도 작성 가능합니다.

주의할 점으로는 config 객체 자체는 빌드타임에 실행되어 적용되기 때문에 동적인 값은 사용할 수 없습니다.

### NextRequest & NextResponse

"next/server"가 제공하는 NextRequest와 NextResponse는 Web Request/Response API를 확장한 것입니다.

```javascript
import { NextRequest, NextResponse } from 'next/server';

const request = new NextRequest();
const response = NextResponse.next();

// 요청/응답 cookie 값을 set 시켜줍니다.
request.cookies.set('key', 'value');
response.cookies.set('key', 'value');

// 매칭된 cookie 값을 반환합니다.
// 매칭된 cookie가 없는 경우 undefined를 반환하고, 여러 개가 매칭된 경우 첫 번째로 매칭된 cookie를 반환합니다.
request.cookies.get('key');
response.cookies.get('key');

// 매칭된 모든 cookie 값을 배열에 담아 반환합니다.
request.cookies.getAll('key');
response.cookies.getAll('key');

// 매칭된 cookie 값을 제거합니다.
// 반환값은 제거 성공 여부를 불리언 값으로 반환합니다.
request.cookies.delete('key');
response.cookies.delete('key');

// 매칭된 cookie 값 존재 여부를 불리언 값으로 반환합니다.
request.cookies.has('key');

// 요청의 Set-Cookie 헤더를 제거합니다.
request.cookies.clear();

// URL 도메인을 반환합니다.
request.nextUrl.baseUrl;

// URL path값을 반환합니다.
request.nextUrl.pathname;

// URL 쿼리 스트링 값을 객체로 반환합니다.
request.nextUrl.searchParams;

// JSON을 body로 갖는 응답을 생성합니다.
NextResponse.json({ success: true }, { status: 200 });

// 특정 URL로 redirect시키는 응답을 생성합니다.
// 클라이언트측에서 해당 응답을 전닯받게 되면 "/home" 경로로 이동하게 됩니다.
NextResponse.redirect(new URL('/home', reqeust.url));

// rewrite 메서드는 route handler에서 사용할 수 없고, next middleware에서 사용할 수 있습니다.
// redirect와는 다르게 요청한 URL path값은 변경하지 않고, 다른 페이지나 route handler로 요청을 전달할 수 있습니다.
NextResponse.rewrite(new URL('/proxy', reqeust.url));

// next 메서드는 route handler에서 사용할 수 없고, next middleware에서 사용할 수 있습니다.
// next 메서드는 요청을 중단하지 않고 다음 단계로 넘길 수 있습니다. 즉, 미들웨어 이후 실제 요청을 이어서 진행하도록 도와줍니다.
NextResponse.next();
```

### Route Segment Config

layout.tsx, page.tsx, Route Handlers에는 Route Segment 옵션을 설정할 수 있습니다.

Route Segment 옵션을 통해 데이터 Next 서버에 캐싱되어 있는 Data Cache와 Full Route Cache를 다룰 수 있습니다.

#### dynamic

- "auto"(default): Next가 페이지에서 사용되는 테이터 소스와 fetch 요청을 분석하여 자동으로 페이지 생성 방식을 정적 혹은 동적으로 결정하게 됩니다.

- "force-dynamic": Next 서버의 Data Cache와 Full Route Cache를 사용하지 않도록 설정합니다. 즉, 매번 fetch하여 받은 응답값을 사용하고 매번 페이지를 생성하여 클라이언트측에 전달합니다.

- "force-static": Next 서버의 Data Cache와 Full Route Cache를 사용하도록 설정합니다. 즉, fetch에 대한 응답값은 Next 서버에 캐싱된 데이터(Data Cache)를 재사용하고 페이지의 경우 빌드 타임때 생성한 페이지(Full Route Cache)를 클라이언트측에 전달합니다.

#### dynamicParams

- true(default): 동적 라우팅 처리가 런타임에 요청된 경로로 동적으로 해석되어 처리됩니다.

- false: 동적 경로가 런타임이 아닌 빌드 타임때 결정되기 때문에 getStaticPath라는 함수를 export하여 생성될 동적 경로 정보를 작성해주어야 합니다.

```javascript
export const dynamicParams = false;

// ,,,

export async function getStaticPaths() {
  const paths = [{ params: { slug: 'post-1' } }, { params: { slug: 'post-2' } }];

  return { paths, fallback: false };
}
```

#### revalidate

Data Cache와 Full Route Cache의 캐싱 지속시간을 설정할 수 있습니다.

- false(default): fetch의 응답 데이터는 Next 서버의 Data Cache에 캐싱된 데이트를 재사용하고, 페이지는 빌드 타임때 생성한 페이지를 사용하게 됩니다.

- number: Next 서버의 Data Cache에 캐싱된 fetch 응답값과 Full Route Cache의 지속시간을 초 단위로 설정할 수 있습니다.

```javascript
export const revalidate = 3600;

// ,,,
```

### Linking and Navigating

#### Link

"next/link"가 제공하는 Link 컴포넌트는 html a 태그를 확장한 컴포넌트로서 prefetching 기능까지 제공합니다. Link 컴포넌트는 RSC와 RCC 둘 다 사용할 수 있습니다.

Link 컴포넌트트는 a 태그를 확장한 컴포넌트로 a 태그에 작성 가능한 어트리뷰트들을 그대로 작성할 수 있습니다.

```javascript
import Link from 'next/link';

export default function Page() {
  return (
    <>
      <Link href='/item' />
    </>
  );
}
```

#### useRouter

"next/navigation"이 제공하는 userRouter 훅이 반환하는 객체를 통해서 Soft Navigating을 사용할 수 있습니다. useRouter는 리액트 훅으로 RCC에서만 사용할 수 있습니다.

```javascript
'use client';

import { useRouter } from 'next/navigation';

export default function Page() {
  const router = useRouter();

  // History stack에 하나의 스택을 push 하고 이동합니다.
  router.push('/items');

  // 현재 URL에 대해 refresh를 수행합니다.
  router.refresh();

  // 특정 URL을 prefetching하여 더 빠른 Navigating을 제공합니다.
  router.prefetch('/item');

  // History stack에서 하나의 스택을 pop 하고 이동합니다.
  router.back();

  // History stack에서 하나의 스택 앞으로 이동합니다.
  router.forward();

  // ,,,
}
```

#### permanentRedirect

"next/navigation"이 제공하는 permanentRedirect 함수는 RCC, RSC, Route Handlers, Server Actions 모두 사용 가능합니다.

```javascript
import { permanentRedirect } from 'next/navigation';

export default function Page() {
  permanentRedirect('/login', { type: 'replace' });

  // ,,,
}
```

permanentRedirect 함수 첫 번째 인수로는 URL을 전달하고 두 번째 인수로는 객체를 전달할 수 있으며 type 프로퍼티에 "replace"(default) 혹은 "push"를 전달할 수 있습니다.
주의할 점으로 Server Actions에서 사용할 경우에는 type의 default가 push로 동작하게 됩니다.

#### redirect

"next/navigation"이 제공하는 redirect 함수는 RSC, Route Handlers, Server Actions에서만 사용 가능합니다.

```javascript
import { redirect } from 'next/navigation';

export default function Page() {
  redirect('/login', { type: 'replace' });

  // ,,,
}
```

redirect 함수 첫 번째 인수로는 URL을 전달하고 두 번째 인수로는 객체를 전달할 수 있으며 type 프로퍼티에 "replace"(default) 혹은 "push"를 전달할 수 있습니다.

#### notFound

"next/navigation"이 제공하는 notFound 함수는

## Functions

### cookies

"next/headers"가 제공하는 cookies 함수는 RSC, Server Actions, Route Handlers에서 사용 가능한 함수로 요청 객체의 쿠키 값을 읽을 수 있습니다.

```javascript
import { cookies } from 'next/headers';

export default function Page() {
  const cookieStore = cookies();

  // 인수로 전달한 쿠키 이름과 매칭된 쿠키값을 반환합니다. 매칭된 쿠키가 없는 경우 undeinfed를 반환합니다.
  // 매칭된 결과가 여러 개인 경우에도 하나만 반환합니다.
  cookieStore.get('key');

  // 인수로 전달한 쿠키 이름과 매칭된 쿠키값들을 반환합니다.
  // get 메서드와는 다르게 매칭된 모든 쿠키값들을 요소로 갖는 배열로 반환합니다.
  cookieStore.getAll('key');

  // 인수로 전달한 쿠키 이름과 매칭된 쿠키값 존재 여부를 불리언 값으로 반환합니다.
  cookieStore.has('key');
}
```

```javascript
import { cookies } from 'next/headers';

export async function GET() {
  const cookieStore = cookies();

  // 요청 객체의 쿠키값을 set할 수 있습니다.
  // 주의할 점으로 set 메서드는 Server Actions와 Route Handlers에서만 사용할 수 있습니다.
  cookieStore.set('key', 'value');
  cookieStore.set({ name: 'key', value: 'value' });

  // 요청 객체의 쿠키값을 제거할 수 있습니다.
  // 주의할 점으로 set 메서드는 Server Actions와 Route Handlers에서만 사용할 수 있습니다.
  cookieStore.delete('key');
}
```

### headers

"next/headers"가 제공하는 headers 함수는 Server Components에서 사용 가능한 함수로 요청 헤더 값을 읽을 수 있습니다.

headers 함수가 반환하는 헤더 값은 읽기 전용으로 set, delete와 같은 동작은 할 수 없습니다.

```javascript
import { headers } from 'next/headers';

export default function Page() {
  const headersList = headers();

  // headers 값을 key, value로 갖는 이터레이터 객체를 반환합니다.
  headersList.entries();

  // 인수로 전달한 콜백은 key, value로 갖는 객체를 인수로 전달받아 실행됩니다.
  headersList.forEach();

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
export default function Page() {
  // force-cache: 기본 캐싱 동작이며, 이전에 요청하여 받은 응답 데이터가 Next 서버에 캐싱되어 있다면 이를 재사용합니다.
  // 이는 최신 데이터를 반영하지 않을 수 있습니다.
  fetch('https://,,,', { cache: 'force-cache' });

  // no-soter: 항상 최신 데이터를 가져와야할 경우에 no-soter 옵션을 명시해주어야 합니다.
  // 이는 캐싱 사용을 비활성화 합니다.
  fetch('https://,,,', { cache: 'no-store' });

  // revalidate 옵션을 통해 cache lifetime을 명시할 수 있습니다.
  // 초 단위 숫자값을 작성하여 cache lifetime을 지정할 수 있습니다. 즉, revalidate 값을 0은 no-store이며 양수값을 작성한 경우 force-cache를 암시합니다.
  fetch('https://,,,', { next: { revalidate: 10 } });
}
```

fetch의 Data Cache 주의할 점은 아래와 같습니다.

- fetch는 기본적으로 force-cache를 사용하며, cookies나 headers와 같은 Dynamic Functions를 사용하는 경우에는 no-store를 기본적으로 사용하게 됩니다.

- fetch의 revalidate 값이 route revalidate보다 작은 경우 라우트 재생성 간격이 감소하게 됩니다.

- 같은 URL에 대한 fetch가 존재하고 revalidate 값이 둘 다 존재한다면 더 작은 revalidate 값이 사용됩니다.

- { revalidate: 0, cache: 'cache-force' } 혹은 { revalidate: number, cache: 'no-store' } 와 같은 옵션은 서로 충돌을 일으켜 애러가 발생하게 됩니다.

### revalidatePath

"next/cache"가 제공하는 revalidatePath 함수는 특정 경로에 대한 캐싱을 모두 무효화하여 최신 데이터를 반영되도록 도와줍니다. 즉, Next 서버측에 캐싱되어 있던 Data Cache와 Full Route Cache 모두 무효화하는 역항를 합니다.

revalidatePath를 호출하면 해당 경로에서 Next 서버에 캐싱된 fetch 응답 데이터와 빌드 타임때 생성되어 캐싱된 페이지를 무효화시킵니다.
이후 다음 번에 사용자가 해당 경로를 요청한 경우 Next가 새로운 데이터를 가져와 페이지를 다시 생성하여 응답으로 전달해줍니다. 이 과정에서 새롭게 생성된 정적 파일과 데이터가 캐시에 저장됩니다.

revalidatePath 함수는 Route Handlers나 Server Actions에서 호출 가능합니다.

```javascript
import { NextRequest, NextResponse } from 'next/server';
import { revalidatePath } from 'next/cache';

export async function POST(request: NextRequest) {
  const { path } = request.nextUrl.searchParams('path');

  if (path) {
    revalidatePath(path, 'page');
    return NextResponse.json({ revalidated: true, now: Date.now() });
  } else {
    return NextResponse.json({ revalidated: false, now: Date.now() });
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
'use client';

import { useParams } from 'next/navigation';

export default function ClientComponent() {
  const params = useParams();

  // ,,,
}
```

### usePathname

"next/navigation"이 제공하는 usePathname 훅은 현재 URL의 path 값을 string으로 반환합니다.

```javascript
'use client';

import { usePathname } from 'next/navigation';

export default function ClientComponent() {
  const pathanem = usePathname();

  // ,,,
}
```

### useSearchParams

"next/navigation"이 제공하는 useSearchParams 훅은 현재 URL의 쿼리스트링 정보를 일기 전용인 URLSearchParams 객체를 반환합니다.

```javascript
'use client';

import { useSearchParams } from 'next/navigation';

export default function ClientComponent() {
  const searchParams = useSearchParams();

  // 쿼리스트링의 value 값을 요소로 갖는 배열을 반환합니다.
  searchParams.getAll();

  // 쿼리스트링의 key 값을 요소로 갖는 이터레이터 객체를 반환합니다.
  searchParams.keys();

  // 쿼리스트링의 value 값을 요소로 갖는 이터레이터 객체를 반환합니다.
  searchParams.values();

  // 쿼리스트링의 key, value 값을 요소로 갖는 배열을 요소로 갖는 이터레이터 객체를 반환합니다.
  searchParams.entries();

  // 인수로 전달한 콜백은 key, value 값을 순차적으로 전달받으며 실행됩니다.
  searchParams.forEach((key, value) => {
    // ,,,
  });

  // ,,,
}
```

### useSelectedlayoutSegment && useSelectedlayoutSegments

"next/navigation"이 제공하는 useSelectedLayoutSegment 훅은 현재 활성화된 한 단계 하위 라우트 세그먼트 반환합니다. useSelectedLayoutSegments 훅은 현재 활성화된 모든 라우트 세그먼크 값을 요소로 갖는 배열을 반환합니다.

예를 들어, 현재 URL path 값이 "/blog/hello-world"인 경우 useSelectedLayoutSegment 훅은 "hello-world"를 반환하고, useSelectedLayoutSegments 훅은 ["blog", "hello-world"]를 반환합니다.

## Caching

### Server Actions

Server Action은 React에 내장된 기능이며 form 제출 기능을 제공합니다. Server Action은 함수이며 컴포넌트 내 직접 정의하거나 별도의 파일로 분리하여 import하여 사용할 수 있습니다. Server Action 함수를 form 태그의 action 어트리뷰트에 전달하여 사용할 수 있습니다.

Server Action 함수는 인수로 FormData 객체를 전달받습니다. 주의할 점으로 formData 객체로 폼 데이터에 접근하기 위해서는 form 내부 각 input들은 name 어트리뷰트를 갖고 있어야 합니다. 폼 데이터들을 접근할 때 input의 name 어트리뷰트 값으로 접근합니다.

추가적으로 Server Action 함수 async 함수로 정의되어야 하며, 함수 코드 블록 최상단에 "use server" 선언문을 작성해주어야 하며 만약 분리된 파일로 정의된 경우 파일 최상단에 "use server"를 작성할 수 있습니다.

Server Action 함수는 Next 서버에서 실행되는 함수이므로 클라이언트측 로직은 사용할 수 없으며 Server Action 함수 자체도 클라이언트 컴포넌트 내에서는 정의할 수 없습니다.

```javascript
export default function Page() {
  // Server Action
  async function create(formData: FormData) {
    'use server';

    // ,,,
  }

  return <form action={create}>,,,</form>
}
```

#### useFormStatus

useFormStatus 훅은 "react-dom"이 제공하는 클라이언트 훅으로 form 제출에 대한 정보를 제공합니다. useFormStatus 훅을 사용하는 컴포넌트는 form 태그를 갖는 컴포넌트 자식으로 작성되어야 합니다.

```javascript
'use client'

import { useFormState } from 'react-dom'

export default function SubmitButton() {
  const { pending } = useFormState()

  return (
    <button type="submit" disabled={pending}>
      Add
    </button>
  )
}
```

#### useActionState

useActionState 훅은 "react"가 제공하는 클라이언트 훅으로 Server Action이 반환하는 값에 접근할 수 있습니다. useFormStatus 훅과는 다르게 form 태그를 갖는 컴포넌트 내 작성할 수 있습니다.

useActionState 훅 첫 번째 인수로는 Server Action 함수를 전달하고 두 번째 인수로는 Server Action이 반환하는 값의 초기값을 전달해주어야 합니다.
첫 번째 인수로 전달하는 Server Action은 첫 번째 인수로 이전 Server Action이 반환한 값을 전달받으며, 두 번째 인수로는 FormData 객체를 전달받습니다.

useActionState 훅은 배열을 반환하며 배열의 첫 번째 요소는 Server Action이 반환하는 값, 두 번째 요소는 React가 제어하는 Server Action을 반환합니다. 이때 두 번째 요소로 반환한 Server Action을 form 태그의 action 어트리뷰트로 전달하면 React가 Server Action이 반환하는 값에 접근할 수 있게 됩니다.

```javascript
'use client'

import { useActionState } from 'react'

import { createUser } from '@/app/actions'

const initState = {
  message: ''
}

export default function SignUp() {
  const [state, formAction] = useActionState(createUser, initState)

  return (
    <form action={formAction}>
      ,,,
    </form>
  )
}
```

### Request Memoization

Request Memoization은 동일한 라우트에서 동일한 설정을 갖는 fetch Request를 중복 요청하지 않고 단일 요청으로 전달하게 됩니다. 즉, 동일한 라우트 내 여러 다른 서버 컴포넌트에서 동일한 설정을 같는 fetch를 호출하더라도 단일 요청으로 전달됩니다. 이는 페이지 첫 렌더링 시점에만 이루어집니다.

예를 들어, Layout과 Page 컴포넌트가 서버 컴포넌트로 작성되었고, 두 컴포넌트 모두 동일한 설정을 갖는 fetch 함수를 호출하게 되면 요청이 두 번 전달되지 않고 단일 요청으로 전달됩니다.

> Request Memoization은 GET 메서드인 fetch에만 적용되며 서버 컴포넌트에만 해당됩니다. 즉, Route Handlers의 fetch에는 해당되지 않습니다.

### Data Cache

Next는 fetch 함수를 통해 서버에서 가져온 응답 데이터를 Next 서버측에 캐싱하고 응답 데이터를 재사용합니다. 즉, Next 서버에서 백엔드로 보내는 요청에 대해서 Next 서버가 해당 요청에 대한 응답 데이터를 캐싱하게 됩니다.

명시적으로 Next 서버에 캐싱된 응답 데이터를 무효화하고 재검증하기 위해 아래와 같은 방법을 사용할 수 있습니다.

- revalidatePath('/,,,', 'page' | 'layout'): 특정 라우트 세그먼트를 전달하여 라우트 전체 fetch에 대해서 재검증을 수행할 수 있습니다.

- fetch('https://,,,', { cache: 'force-cache' | 'no-store' }): cache 옵션으로 개별 요청에 대한 응답 데이터 캐싱 사용여부를 설정할 수 있습니다.

- fetch('https://,,,', { next: { revalidate: number }}): next.revalidate 옵션으로 개별 요청에 대한 응답 데이터가 캐싱될 시간을 초단위로 작성할 수 있습니다.

- Route Segment Config: dynamic 혹은 revalidate 옵션을 사용하여 라우트 전체에 대한 캐싱을 설정할 수 있습니다.

### Full Route Cache

기본적으로 Next는 빌드 타임때 페이지를 미리 생성하고 렌더링된 결과를 Next 서버에 캐싱합니다. 이후 클라이언트가 페이지를 요청하면 Next 서버에 캐싱된 렌더링 결과를 전달하게 됩니다.

만약 페이지 생성 방식을 명시적으로 변경하고 싶다면 아래와 같은 방법을 사용할 수 있습니다.

- Route Segments Config: dynamic 혹은 revalidate 옵션을 사용하여 페이지 생성 방식을 설정할 수 있습니다.

- revalidatePath('/,,,', 'page' | 'layout'): 특정 라우트 세그먼트를 전달하여 페이지 생성 시점을 설정할 수 있습니다.

> Dynamic Functions(cookies, headers), searchParams prop 혹은 Data Cache를 사용하지 않는 "no-store" 옵션을 사용하는 경우 해당 페이지는 Full Route Cache가 동작하지 않게 됩니다. 즉, Full Route Cache가 동작하는 경우는 Data Cache를 사용하며 Dynamic Functions나 searchParams porp을 사용하지 않는 경우에만 Full Route Cache가 동작하게 됩니다.

### Route Cache

Next는 클라이언트측 인메모리에 응답으로 전달받은 서버 컴포넌트들을 캐싱하고 있습니다. 즉, Route Cache는 Next 서버가 아닌 클라이언트측에서 이루어지는 캐싱입니다.

Route Cache는 두 종류가 존재하며 Dynamically Rendered 혹은 Statically Rendered인 경우에 따라서 캐싱 지속시간이 달라집니다.

Dynamic Functions(headers, cookies) 함수를 사용하지 않고 searchParams prop도 접근하지 않으며 Data Cache를 사용하는 경우에만 Statically Rendered로 설정되며 이외 경우에는 Dynamically Rendered로 설정됩니다.

Dynamically Rendered의 경우에는 캐싱 지속시간이 30초이고, Statically Rendered의 경우에는 5분으로 설정됩니다.
캐싱 지속시간이 이후 Dynamically Rendered의 경우 서버측에서 다시 실행한 렌더링 결과를 클라이언트측에 전달하고 Statically Rendered의 경우에는 서버측에서 캐싱하고 있는 렌더링 결과를 클라이언트측에 전달하게 됩니다.

Route Cache 기능은 아예 끌 수 는 없으며 무효화하고자 한다면 아래와 같은 방법을 사용할 수 있습니다.

- revalidatePath('/,,,', 'page' | 'layout'): 특정 라우트 세그먼트를 전달하여 Route Cache를 무효화할 수 있습니다.

- cookies: "next/headers"가 제공하는 cookies 함수로 set 혹은 delete하는 경우 Route Cache가 무효화됩니다. 이때 set와 delete의 경우에는 Server Actions, Route Handlers에서만 가능하므로 Route Cache의 경우에는 Server Actions에만 해당됩니다.

- router.refresh: useRouter 훅이 반환하는 객체의 refresh 함수를 호출하여 명시적으로 Route Cache를 무효화할 수 있습니다.
