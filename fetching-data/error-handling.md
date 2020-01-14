---
description: Apollo 오류 처리
---

# Error handling

단순한 것에서 복잡한 것까지 모든 응용 프로그램은 상당한 오류를 가질 수 있습니다. 이러한 오류를 처리하는 것이 중요하며 가능하면 이러한 오류를 사용자에게보고하여 정보를 얻으십시오. GraphQL을 사용하면 실제 GraphQL 응답 자체에서 발생 가능한 새로운 오류가 발생합니다. 이를 염두에두고 몇 가지 다른 유형의 오류가 있습니다.

* GraphQL 오류 : 성공적인 데이터와 함께 나타날 수있는 GraphQL 결과의 오류
* 서버 오류 : 성공적인 응답을 형성하지 못하게하는 서버 내부 오류
* 트랜잭션 오류 : 돌연변이에 대한 업데이트와 같은 트랜잭션 작업 내의 오류
* UI 오류 : 구성 요소 코드에서 발생하는 오류
* 아폴로 클라이언트 오류 : 코어 또는 해당 라이브러리 내의 내부 오류

### Error policies\(오류 정책\) <a id="error-policies"></a>

fetchPolicy와 마찬가지로 errorPolicy를 사용하면 서버의 GraphQL 오류가 UI 코드로 전송되는 방식을 제어 할 수 있습니다. 기본적으로 오류 정책은 모든 GraphQL 오류를 네트워크 오류로 처리하고 요청 체인을 종료합니다. 캐시에 데이터를 저장하지 않으며 오류 소품과 함께 UI를 ApolloError로 렌더링합니다. 요청 당이 정책을 변경하면 캐시 및 UI에서 GraphQL 오류가 관리되는 방식을 조정할 수 있습니다. errorPolicy의 가능한 옵션은 다음과 같습니다.

* none : Apollo Client 1.0의 작동 방식과 일치하는 기본 정책입니다. 모든 GraphQL 오류는 네트워크 오류와 동일하게 취급되며 모든 데이터는 응답에서 무시됩니다.

* ignore : ignore를 사용하면 GraphQL 오류와 함께 리턴되는 데이터를 읽을 수 있지만 오류를 저장하거나 UI에보고하지는 않습니다.

* all : 서버에서 가능한 많은 데이터를 표시하면서 잠재적 인 문제를 사용자에게 알리는 가장 좋은 방법은 모든 정책을 사용하는 것입니다. UI가 사용할 수 있도록 데이터와 오류를 모두 Apollo Cache에 저장합니다.

각 요청에 대해 errorPolicy를 다음과 같이 설정할 수 있습니다.

```text
const MY_QUERY = gql`
  query WillFail {
    badField
    goodField
  }
`;

function ShowingSomeErrors() {
  const { loading, error, data } = useQuery(MY_QUERY, { errorPolicy: 'all' });

  if (loading) return <span>loading...</span>
  return (
    <div>
      <h2>Good: {data.goodField}</h2>
      <pre>Bad: {error.graphQLErrors.map(({ message }, i) => (
        <span key={i}>{message}</span>
      ))}
      </pre>
    </div>
  );
}
```

보고 된 모든 오류는 캐시 또는 서버에서 반환 된 데이터와 함께 오류가 발생합니다.

### Network Errors <a id="network-errors"></a>

Apollo Link를 사용할 때 네트워크 오류를 처리하는 기능이 훨씬 강력합니다. 이를 수행하는 가장 좋은 방법은 아폴로 링크 오류를 사용하여 서버 오류, 네트워크 오류 및 GraphQL 오류를 포착하고 처리하는 것입니다. 다른 링크와 결합하려면 링크 구성을 참조하십시오.

기본사용법

```text
import { onError } from "apollo-link-error";

const link = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors)
    graphQLErrors.map(({ message, locations, path }) =>
      console.log(
        `[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`,
      ),
    );

  if (networkError) console.log(`[Network error]: ${networkError}`);
});
```

옵션 

오류 링크는 오류 발생시 호출되는 기능을 수행합니다. 이 함수는 다음 키를 포함하는 객체와 함께 호출됩니다.

* operation\(작업\) : 오류가 발생한 작업 
* response\(응답\) : 서버의 응답
* graphQLErrors : GraphQL 엔드 포인트의 오류 배열
* networkError : 링크 실행 또는 서버 응답 중 오류

**Ignoring errors\(오류 무시\)**

조건부로 오류를 무시하려면 response.errors = null; 오류 처리기 내에서 :

```text
onError(({ response, operation }) => {
  if (operation.operationName === "IgnoreErrorsQuery") {
    response.errors = null;
  }
})
```

  



