---
description: fragments을 사용하여 쿼리에서 필드를 공유하는 방법 알아보기
---

# Using fragments

GraphQL 조각은 클라이언트가 여러 쿼리와 돌연변이간에 공유 할 수있는 논리입니다.

여기서는 Person 객체와 함께 사용할 수있는 NameParts 조각을 선언합니다.

```text
fragment NameParts on Person {
  firstName
  lastName
}
```

조각에는 관련 형식에 대해 선언 된 필드의 하위 집합이 포함됩니다. 위의 예에서 Person 유형은 NameParts 조각이 유효한 firstName 및 lastName 필드를 선언해야합니다.

이제 Person 객체를 참조하는 여러 쿼리 및 돌연변이에 NameParts 조각을 포함시킬 수 있습니다.

```text
query GetPerson {
  people(id: "7") {
    ...NameParts
    avatar(size: LARGE)
  }
}
```

NameParts 정의를 기반으로 위의 쿼리는 다음과 같습니다. NameParts jeon

```text
query GetPerson {
  people(id: "7") {
    firstName
    lastName
    avatar(size: LARGE)
  }
}
```

그러나 나중에 NameParts 조각에 포함 된 필드를 변경하면 NameParts 조각을 사용하는 모든 작업에 포함되는 필드가 자동으로 변경됩니다. 이를 통해 일련의 작업에서 필드의 일관성을 유지하는 데 필요한 노력이 줄어 듭니다.

### Reusing fragments\(조각 재사용\) <a id="reusing-fragments"></a>

단편은 여러 GraphQL 연산에서 동일한 필드 세트를 포함하는 데 유용합니다. 예를 들어 블로그는 주석과 관련된 여러 작업을 정의 할 수 있으며 각 작업에는 댓글 유형의 동일한 기준 필드 집합이 포함되어야합니다.

이 기본 필드 세트를 지정하기 위해 모든 주석 관련 조작에 포함되어야하는 주석 필드를 나열하는 단편을 정의합니다.

```text
import { gql } from '@apollo/client';

CommentsPage.fragments = {
  comment: gql`
    fragment CommentsPageComment on Comment {
      id
      postedBy {
        login
        html_url
      }
      createdAt
      content
    }
  `,
};
```

우리는 조각을 CommentsPage.fragments.comment에 규칙으로 할당합니다.

GraphQL 오퍼레이션 내에 프래그먼트를 임베드하려면 다음과 같이 이름 앞에 3 개의 마침표 \(...\)를 붙입니다.

```text
const SUBMIT_COMMENT_MUTATION = gql`
  mutation SubmitComment($postFullName: String!, $commentContent: String!) {
    submitComment(postFullName: $postFullName, commentContent: $commentContent) {
      ...CommentsPageComment    }
  }
  ${CommentsPage.fragments.comment}
`;

export const COMMENT_QUERY = gql`
  query Comment($postName: String!) {
    entry(postFullName: $postName) {
      comments {
        ...CommentsPageComment      }
    }
  }
  ${CommentsPage.fragments.comment}
`;
```

### Colocating fragments\(조각 배치\) <a id="colocating-fragments"></a>

GraphQL 응답의 트리와 같은 구조는 프론트 엔드의 렌더링 된 컴포넌트의 계층 구조와 유사합니다. 이러한 유사성 때문에 조각을 사용하여 구성 요소간에 쿼리 논리를 분할하여 각 구성 요소가 사용하는 필드를 정확하게 요청하도록 할 수 있습니다. 이렇게하면 구성 요소 논리를보다 간결하게 만들 수 있습니다.

앱의 다음과 같은 뷰 계층 구조를 고려하십시오.

```text
FeedPage
└── Feed
    └── FeedEntry
        ├── EntryInfo
        └── VoteButtons
```

이 응용 프로그램에서 FeedPage 구성 요소는 FeedEntry 개체 목록을 가져 오는 쿼리를 실행합니다. EntryInfo 및 VoteButtons 하위 구성 요소에는 묶는 FeedEntry 오브젝트의 특정 필드가 필요합니다.

#### Creating colocated fragments <a id="creating-colocated-fragments"></a>

공존 된 프래그먼트는 프래그먼트의 필드를 사용하는 특정 구성 요소에 첨부되는 것을 제외하고는 다른 프래그먼트와 같습니다. 예를 들어 FeedPage의 VoteButtons 자식 구성 요소는 FeedEntry 객체의 score 및 vote {choice} 필드를 사용할 수 있습니다.

```text
VoteButtons.fragments = {
  entry: gql`
    fragment VoteButtonsFragment on FeedEntry {
      score
      vote {
        choice
      }
    }
  `,
};
```

자식 구성 요소에서 조각을 정의한 후 부모 구성 요소는 다음과 같이 자체 조각 정의에서 자식 구성 요소 조각을 참조 할 수 있습니다.

```text
FeedEntry.fragments = {
  entry: gql`
    fragment FeedEntryFragment on FeedEntry {
      commentCount
      repository {
        full_name
        html_url
        owner {
          avatar_url
        }
      }
      ...VoteButtonsFragment
      ...RepoInfoFragment
    }
    ${VoteButtons.fragments.entry}
    ${RepoInfo.fragments.entry}
  `,
};
```

VoteButtons.fragments.entry 또는 RepoInfo.fragments.entry의 이름 지정에는 특별한 것이 없습니다. 구성 요소에 지정된 구성 요소의 조각을 쉽고 일관되게 검색 할 수있는 한 모든 명명 규칙이 작동합니다.

Webpack을 사용할 때 조각 가져 오기

graphql-tag / loader로 .graphql 파일을로드 할 때 import 문을 사용하여 조각을 포함 할 수 있습니다. 예를 들면 다음과 같습니다.

```text
#import "./someFragment.graphql"
```

이를 통해 someFragment.graphql의 내용을 현재 파일에서 사용할 수 있습니다. 자세한 내용은 Webpack Fragments 섹션을 참조하십시오.

### Using fragments with unions and interfaces <a id="using-fragments-with-unions-and-interfaces"></a>

공용체 및 인터페이스에서 조각을 정의 할 수 있습니다.

다음은 세 개의 인라인 조각을 포함하는 쿼리의 예입니다.

query { all\_characters {

```text
... on Character {
  name
}

... on Jedi {
  side
}

... on Droid {
  model
}
```

위의 all\_characters 쿼리는 Character 객체 목록을 반환합니다. Character 유형은 Jedi 및 Droid 유형이 모두 구현하는 인터페이스입니다. 목록의 각 항목은 Jedi 유형의 오브젝트 인 경우 측면 필드를 포함하고 Droid 유형 인 경우 모델 필드를 포함합니다.

그러나이 쿼리가 작동하려면 클라이언트가 Character 인터페이스와이를 구현하는 유형 간의 다형성 관계를 이해해야합니다. 이러한 관계에 대해 클라이언트에 알리기 위해 InMemoryCache를 만들 때 possibleTypes 옵션을 전달할 수 있습니다.

#### Defining `possibleTypes` manually <a id="defining-possibletypes-manually"></a>

> possibleTypes 옵션은 Apollo Client 3.0 이상에서 사용할 수 있습니다.

가능한 유형 옵션을 InMemoryCache 생성자에 전달하여 스키마에 수퍼 유형 하위 유형 관계를 지정할 수 있습니다. 이 객체는 인터페이스 또는 공용체 유형 \(상위 유형\)의 이름을 구현하거나 이에 속하는 유형 \(하위 유형\)에 매핑합니다.

다음은 possibleTypes 선언의 예입니다.

```text
const cache = new InMemoryCache({
  possibleTypes: {
    Character: ["Jedi", "Droid"],
    Test: ["PassingTest", "FailingTest", "SkippedTest"],
    Snake: ["Viper", "Python"],
  },
});
```

이 예제는 세 가지 인터페이스 \(Character, Test 및 Snake\)와이를 구현하는 객체 유형을 나열합니다.

스키마에 몇 개의 공용체와 인터페이스 만 포함 된 경우 가능한 유형을 문제없이 수동으로 지정할 수 있습니다. 그러나 스키마의 크기와 복잡성이 커짐에 따라 스키마에서 가능한 유형을 자동으로 생성하는 것을 고려해야합니다.

#### Generating `possibleTypes` automatically <a id="generating-possibletypes-automatically"></a>

다음 예제 스크립트는 GraphQL 내부 검사 쿼리를 possibleTypes 구성 객체로 변환합니다.

```text
const fetch = require('node-fetch');
const fs = require('fs');

fetch(`${YOUR_API_HOST}/graphql`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    variables: {},
    query: `
      {
        __schema {
          types {
            kind
            name
            possibleTypes {
              name
            }
          }
        }
      }
    `,
  }),
}).then(result => result.json())
  .then(result => {
    const possibleTypes = {};

    result.data.__schema.types.forEach(supertype => {
      if (supertype.possibleTypes) {
        possibleTypes[supertype.name] =
          supertype.possibleTypes.map(subtype => subtype.name);
      }
    });

    fs.writeFile('./possibleTypes.json', JSON.stringify(possibleTypes), err => {
      if (err) {
        console.error('Error writing possibleTypes.json', err);
      } else {
        console.log('Fragment types successfully extracted!');
      }
    });
  });
```

그런 다음 생성 된 가능한 유형 JSON 모듈을 InMemoryCache를 작성하는 파일로 가져올 수 있습니다.

```text
import possibleTypes from './path/to/possibleTypes.json';

const cache = new InMemoryCache({
  possibleTypes,
});
```

