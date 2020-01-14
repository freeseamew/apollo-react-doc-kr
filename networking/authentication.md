# Authentication\(인증\)

로드하는 모든 데이터가 완전히 공개되지 않으면 앱에는 일종의 사용자, 계정 및 권한 시스템이 있습니다. 응용 프로그램에서 사용자마다 권한이 다른 경우 각 요청과 관련된 사용자를 서버에 알리는 방법이 필요합니다.

Apollo Client는 인증을위한 여러 옵션이 포함 된 매우 유연한 Apollo Link를 사용합니다.

### Cookie <a id="cookie"></a>

앱이 브라우저 기반이고 백엔드로 로그인 및 세션 관리를 위해 쿠키를 사용하는 경우 모든 요청과 함께 쿠키를 보내도록 네트워크 인터페이스에 지시하는 것이 매우 쉽습니다. 자격 증명 옵션 만 전달하면됩니다. 예 : 자격 증명 : 백엔드 서버가 동일한 도메인 인 경우 아래와 같이 '동일 출처'또는 자격 증명 : 백엔드가 다른 도메인 인 경우 '포함'.

```text
// enable cors
var corsOptions = {
  origin: '<insert uri of front-end domain>',
  credentials: true // <-- REQUIRED backend setting
};
app.use(cors(corsOptions));
```

### Header <a id="header"></a>

HTTP를 사용할 때 자신을 식별하는 또 다른 일반적인 방법은 권한 부여 헤더를 따라 보내는 것입니다. Apollo Links를 함께 연결하여 모든 HTTP 요청에 인증 헤더를 쉽게 추가 할 수 있습니다. 이 예제에서는 요청이 전송 될 때마다 localStorage에서 로그인 토큰을 가져옵니다.

```text
import { ApolloClient } from 'apollo-client';
import { createHttpLink } from 'apollo-link-http';
import { setContext } from 'apollo-link-context';
import { InMemoryCache } from 'apollo-cache-inmemory';

const httpLink = createHttpLink({
  uri: '/graphql',
});

const authLink = setContext((_, { headers }) => {
  // get the authentication token from local storage if it exists
  const token = localStorage.getItem('token');
  // return the headers to the context so httpLink can read them
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : "",
    }
  }
});

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache()
});
```

위의 예는 apollo-client 패키지에서 ApolloClient를 사용하고 있습니다. 헤더는 여전히 apollo-boost 패키지에서 ApolloClient를 사용하여 수정할 수 있지만, apollo-boost는 사용하는 HttpLink 인스턴스를 수정할 수 없으므로 헤더를 구성 매개 변수로 전달해야합니다.

```text
import ApolloClient from 'apollo-boost'

const client = new ApolloClient({
  request: (operation) => {
    const token = localStorage.getItem('token')
    operation.setContext({
      headers: {
        authorization: token ? `Bearer ${token}` : ''
      }
    })
  }
})
```

이 특정 예제에서 헤더는 모든 요청에 대해 개별적으로 설정되지만 헤더 옵션을 사용할 수도 있습니다. 자세한 내용은 Apollo Boost 구성 옵션 섹션을 참조하십시오.

서버는 해당 헤더를 사용하여 사용자를 인증하고 GraphQL 실행 컨텍스트에 첨부 할 수 있으므로 리졸버는 사용자의 역할 및 권한에 따라 동작을 수정할 수 있습니다.

### Reset store on logout <a id="reset-store-on-logout"></a>

Apollo는 모든 쿼리 결과를 캐시하므로 로그인 상태가 변경되면이를 제거하는 것이 중요합니다.

UI 및 저장소 상태가 현재 사용자의 권한을 반영하도록하는 가장 쉬운 방법은 로그인 또는 로그 아웃 프로세스가 완료된 후 client.resetStore \(\)를 호출하는 것입니다. 그러면 상점이 지워지고 모든 활성 조회가 다시 페치됩니다. 저장소를 지우고 활성 쿼리를 다시 가져 오지 않으려면 client.clearStore \(\)를 대신 사용하십시오. 다른 옵션은 페이지를 새로 고침하는 것인데, 비슷한 효과가 있습니다.

```text
const PROFILE_QUERY = gql`
  query CurrentUserForLayout {
    currentUser {
      login
      avatar_url
    }
  }
`;

function Profile() {
  const { client, loading, data: { currentUser } } = useQuery(
    PROFILE_QUERY,
    { fetchPolicy: "network-only" }
  );

  if (loading) {
    return <p className="navbar-text navbar-right">Loading...</p>;
  }

  if (currentUser) {
    return (
      <span>
        <p className="navbar-text navbar-right">
          {currentUser.login}
          &nbsp;
          <button
            onClick={() => {
              // call your auth logout code then reset store
              App.logout().then(() => client.resetStore());
            }}
          >
            Log out
          </button>
        </p>
      </span>
    );
  }

  return (
    <p className="navbar-text navbar-right">
      <a href="/login/github">Log in with GitHub</a>
    </p>
  );
}
```

