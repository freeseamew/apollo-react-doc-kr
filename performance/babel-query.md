# babel을 사용한 query 컴파일

Javascript 파일에서 GraphQL 쿼리를 같은 위치에 배치하려면 일반적으로 graphql-tag 라이브러리를 사용하여 작성하십시오. 쿼리 문자열을 GraphQL AST로 처리해야하며, 특히 쿼리가 많은 경우 응용 프로그램의 시작 시간에 추가됩니다.

이 런타임 오버 헤드를 피하기 위해 Babel을 사용하여 graphql-tag로 작성된 쿼리를 사전 컴파일 할 수 있습니다. 이를 수행 할 수있는 두 가지 방법이 있습니다.

1. Using [babel-plugin-graphql-tag](https://www.apollographql.com/docs/react/performance/babel/#using-babel-plugin-graphql-tag)
2. Using [graphql-tag.macro](https://www.apollographql.com/docs/react/performance/babel/#using-graphql-tagmacro)

GraphQL 코드를 별도의 파일 \(.graphql 또는 .gql\)로 유지하려면 babel-plugin-import-graphql을 사용할 수 있습니다. 이 플러그인은 여전히 후드 아래에서 graphql-tag를 사용하지만 투명합니다. 각 작업이 GraphQL 파일에서 내보내기 인 것처럼 작업 / 조각을 가져 오기만하면됩니다. 이것은 위의 접근 방식과 동일한 사전 컴파일 이점을 제공합니다.

### Using babel-plugin-graphql-tag <a id="using-babel-plugin-graphql-tag"></a>

이 방법을 사용하면 평소와 같이 graphql-tag 라이브러리를 사용할 수 있으며이 babel 플러그인으로 파일을 처리 할 때 해당 라이브러리에 대한 호출이 사전 컴파일 된 결과로 대체됩니다.

개발자의 의존성에 플러그인을 설치하십시오 :

```text
# with npm
npm install --save-dev babel-plugin-graphql-tag

# or with yarn
yarn add --dev babel-plugin-graphql-tag
```

그런 다음 .babelrc 구성 파일에 플러그인을 추가하십시오.

```text
{
  "plugins": [
    "graphql-tag"
  ]
}
```

그리고  'graphql-tag'에서 gql 가져 오기의 모든 사용법이 제거되고 gql에 대한 호출이 컴파일 된 버전으로 대체됩니다.

### Using graphql-tag.macro <a id="using-graphql-tagmacro"></a>

이 접근 방식은 graphql-tag.macro에 대해 graphql-tag의 모든 사용법을 변경하므로 원래와 동일한 방식으로 사용할 수있는 gql 함수를 내보내므로 좀 더 명확합니다. 이 매크로에는 babel-macros 플러그인이 필요합니다.이 플러그인은 이전 접근 방식과 동일하지만 매크로 가져 오기에서 오는 호출에서만 수행되며 graphql-tag 라이브러리에 대한 일반 호출은 그대로 유지됩니다.

이 방법을 선호하는 이유는 무엇입니까? 주로 구성이 덜 필요하기 때문에 \(babel-macros는 모든 종류의 매크로에서 작동하므로 이미 설치 한 경우 다른 작업을 수행 할 필요가 없음\) 명시 적 때문입니다. 이 블로그 게시물에서 babel-macros 사용의 근거에 대해 자세히 읽을 수 있습니다.

이미 babel-macros가 설치 및 구성되어 있다면이를 사용하려면 다음을 변경하면됩니다.

```text
import gql from 'graphql-tag';

const query = gql`
  query {
    hello {
      world
    }
  }
`;
```

이

```text
import gql from 'graphql-tag.macro'; // <-- Use the macro

const query = gql`
  query {
    hello {
      world
    }
  }
`;
```

### Using babel-plugin-import-graphql <a id="using-babel-plugin-import-graphql"></a>

개발자의 의존성에 플러그인을 설치하십시오 :

```text
# with npm
npm install --save-dev babel-plugin-import-graphql

# or with yarn
yarn add --dev babel-plugin-import-graphql
```

런 다음 .babelrc 구성 파일에 플러그인을 추가하십시오.

```text
{
  "plugins": [
    "import-graphql"
  ]
}
```

이제 GraphQL 파일 형식에서 가져 오는 모든 import 문은 바로 사용할 수있는 GraphQL DocumentNode 객체를 반환합니다.

```text
import React, { Component } from 'react';
import { graphql } from '@apollo/react-hoc';
import myImportedQuery from './productsQuery.graphql';
// or for files with multiple operations:
// import { query1, query2 } from './queries.graphql';

class QueryingComponent extends Component {
  render() {
    if (this.props.data.loading) return <h3>Loading...</h3>;
    return <div>{`This is my data: ${this.props.data.queryName}`}</div>;
  }
}

export default graphql(myImportedQuery)(QueryingComponent);
```

### Fragments\(파편\) <a id="fragments"></a>

이러한 접근 방식은 모두 조각 사용을 지원합니다.

처음 두 가지 접근 방식의 경우 gql에 대한 다른 호출 \(동일한 파일 또는 다른 파일\)로 조각을 정의 할 수 있습니다. 그런 다음 보간법을 사용하여 다음과 같이 기본 쿼리에 포함시킬 수 있습니다.

```text
import gql from 'graphql-tag';
// or import gql from 'graphql-tag.macro';

const fragments = {
  hello: gql`
    fragment HelloStuff on Hello {
      universe
      galaxy
    }
  `
};

const query = gql`
  query {
    hello {
      world
      ...HelloStuff
    }
  }

  ${fragments.hello}
`;
```

babel-plugin-import-graphql을 사용하면 GraphQL 파일에 조각을 사용하는 것과 함께 조각을 포함하거나 \#import 구문을 사용하여 별도의 파일에서 가져올 수 있습니다. 자세한 내용은 README를 참조하십시오.

