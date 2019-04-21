# GraphQL에 대해 쓰게 된 이유

요즘 여기저기서 GraphQL을 도입했다는 이야기가 들려온다. 물론 이 기술 자체는 꽤나 오랜 시간 전에 등장했지만, 그냥 이런 기술도 있네 하고 넘겼던 사람들이 많았을 것이란 생각이 든다. 나도 그 중 하나였는데, gatsby로 블로그를 만드려고 보니 GraphQL을 사용한다고 해서 어라 이게 뭐지? 하고 사용해보다가 신기하기도 하고 꽤나 쓸모가 많은 기술 같아서 블로그에 어느정도 지식을 정리해두려고 한다.

# 공식 문서에서의 설명
> 원문은 https://graphql.org/learn/ 여기에서 볼 수 있습니다.

GraphQL은 API를 위한 질의 언어이며, 데이터에 대한 우리가 정의한 타입 시스템을 사용함으로써 쿼리를 실행하는 서버사이드 런타임입니다. GraphQL은 어떤 특정한 데이터베이스나 저장소 엔진(store engine)에 매여있지 않습니다. 대신 우리가 작성한 코드와 데이터를 기반으로 합니다. 

GraphQL 서비스는 그 타입들에 대한 필드와 타입을 정의함으로써 만들어집니다. 그 후에 각 타입의 필드에 대한 함수를 제공합니다. 예를 들면, GraphQL 서비스는 우리에게 누가 로그인된 유저인지 말해주고, 유저의 이름을 다음과 같이 보여줍니다.

```
type Query {
  me: User  
}

type User {
  id: ID,
  name: String
}
```

각 타입의 필드에 대한 함수들도 함께 보여줍니다.

```js
function Query_me(request) {
  return request.auth.user;  
}

function User_name(user) {
  return user.getName();  
}
```

GraphQL 서비스가 일단 돌아가기 시작하면(일반적으로는 웹 서비스의 URL에서 돌아갑니다.), 유효화하고 실행하기 위해 GraphQL 쿼리로 보내질 수 있습니다. 받아진 쿼리는 먼저, 정의된 타입과 필드를 참조하는지 체크됩니다. 그 후에 결과를 생성하기 위해 제공된 함수를 실행합니다.

다음의 예제 쿼리는,

```js
{
  me {
    name
  }
}
```

다음과 같은 결과를 내뱉습니다.

```js
{
  "me" : {
    "name": "Luke Skywalker"
  }
}
```

# 어떻게 활용되는가?
> 여기에서는 [이 포스팅](https://medium.com/@JeffLombardJr/when-and-why-to-use-graphql-24f6bce4839d)의 내용을 많이 참조했습니다.

공식 문서의 `Introduction`에 있는 간단한 설명만 읽어보면, GraphQL이란 무엇인지, 왜 GraphQL을 사용해야 하는지 잘 이해되지 않을 수도 있습니다. 얼핏보면 SQL과 같은 질의어와 별다를게 없는데, 그냥 데이터를 가져오는 방식만 바뀌는 것이 아닐까? 라고 생각할 수 있습니다.

도대체 왜 RESTful을 사용하지 않고 굳이 GraphQL로 다음과 같이 데이터를 조회하면 좋은 점은 무엇이고 그 의미는 무엇일까요?
```js
{
  table {
    column
  }
}
```

## 하나의 엔드포인트(endpoint)로 다양한 정보를 얻는데 활용

엔드포인트라는 용어가 생소하신 분도 있을 거라고 생각합니다. 여기서 말하는 엔드포인트란 간단하게 설명하면 우리가 어떤 API에서 데이터를 요청할 때, 우리가 접근하는 URI의 끝지점을 말합니다. 이를테면 우리가 로컬에서 데이터를 제공하는 API 서버를 운영한다고 가정했을 때, **http://localhost:3001/book/1**에 **GET 메소드**를 보냈을 때, 번호가 1번인 책의 정보가 나온다면 **http://localhost:3001/book/1** 주소와 **GET 메소드**가 엔드포인트가 될 수 있습니다. 

그렇다면 엔드포인트가 적은게 왜 좋을까요? 그건 바로 API의 복잡성을 줄일 수 있기 때문입니다. 이를테면 위에서 든 예에서 우리가 책의 정보를 API로 가져온다고 가정했으니 우리가 서점을 운영하고 책 정보를 관리하는 API를 구성한다치면 최소 거의 4개의 엔드포인트는 필요합니다. /book/{id}와 같은 형태의 URL과 조회 시에 get 메소드, 삭제 시에 delete 메소드, 업데이트 시에 put 메소드, 추가 시에 post 메소드. 하지만 GraphQL에서는 /graphql이라는 엔드포인트 하나로 전부 가능합니다.

엔드포인트가 적어지면 백엔드의 복잡성이 줄어들고, 엔드포인트 관리에 큰 힘을 기울일 필요가 없습니다.

다음의 그림이 이해에 큰 도움을 줄 것입니다.


![API img1.png](https://images.velog.io/post-images/jakeseo_me/be47be00-641e-11e9-b41f-c128b9fea3ea/API-img1.png)
> RESTful API 아키텍쳐 [그림 출처(original source)](https://medium.com/@JeffLombardJr/when-and-why-to-use-graphql-24f6bce4839d)

![API img2.png](https://images.velog.io/post-images/jakeseo_me/c3d56750-641e-11e9-b41f-c128b9fea3ea/API-img2.png)
> GraphQL API 아키텍쳐 [그림 출처(original source)](https://medium.com/@JeffLombardJr/when-and-why-to-use-graphql-24f6bce4839d)

## 특정 웹 아키텍쳐 디자인 패턴을 구현하기 용이하다.

우리에겐 이미 안정적인 `RESTful` 서비스가 있는데 왜 graphQL을 써야할까요?

특정 디자인 패턴을 구현하기 용이합니다. 앞으로 소개될 디자인 패턴들은 사실 다른 도구로 구현될 수도 있습니다. 그런데 GraphQL을 이용하는게 가장 좋은 이유는 데이터의 요청과 조작(manipulation) 쿼리가 완전히 분리되어 있다는 것입니다.

OOP에서 웹 서비스에 적용될 수 있는 몇가지 유용한 패턴들에 대해 살펴봅시다.

### 컴포지트 패턴(Composite Pattern)

컴포지트 패턴은 우리가 다양한 장소에서 데이터를 긁어모아 한 API에서 제공하고 싶을 때를 말합니다.

이를테면 우리가 신문사를 운영하는데 각각 다른 소스에서 뉴스에 대한 데이터와 전문가들이 쓴 컬럼에 대한 데이터, 오피니언에 대한 데이터를 한데 모아 보고 싶다면 이 패턴을 이용할 수 있을 것입니다.

![Composite pattern 1.png](https://images.velog.io/post-images/jakeseo_me/1314e110-641f-11e9-a124-2f048c126f03/Composite-pattern-1.png)
> 컴포지트 디자인 패턴 [그림 출처(original source)](https://medium.com/@JeffLombardJr/when-and-why-to-use-graphql-24f6bce4839d)

[위키피디아 설명](https://en.wikipedia.org/wiki/Composite_pattern)

### 프록시 패턴(Proxy Pattern)

프록시 패턴은 오래된 API에 기능을 추가하고 싶을 때 사용합니다.

이 패턴은 우리가 오래된 API를 운영하는데, 인증(authentication)을 추가하고 싶을 때 사용할 수 있을 것입니다.

![Proxy Pattern img 1.png](https://images.velog.io/post-images/jakeseo_me/e9b86390-641f-11e9-b41f-c128b9fea3ea/Proxy-Pattern-img-1.png)
> 프록시 디자인 패턴 [그림 출처(original source)](https://medium.com/@JeffLombardJr/when-and-why-to-use-graphql-24f6bce4839d)

[위키피디아 설명](https://en.wikipedia.org/wiki/Proxy_pattern)

### 퍼사이드 패턴(Facade Pattern)

복잡한(Complex) API를 단순화(Simplify)시키고 싶을 때 사용합니다.

프록시 패턴과 퍼사이드 패턴의 차이는 간단합니다. 프록시는 인증과 같은 기능을 추가하더라도 원본을 보여줍니다. 퍼사이드 패턴은 원본을 단순화(Simplify)합니다.

웹사이트에 새로운 위젯을 추가하고 싶을 때, 3개의 서로 다른 호출 방식에 대해 모두 코드를 작성해야 한다고 가정해봅시다. GraphQL을 이용한 위젯 오브젝트 하나를 노출시키면, GraphQL 서버로의 한번의 호출이 있을 때, 서버가 3개의 서포팅 호출에 대해서 모두 트리거를 해줍니다.

![Facade Pattern img 1.png](https://images.velog.io/post-images/jakeseo_me/a9833470-6420-11e9-bc62-57f0c2aea682/Facade-Pattern-img-1.png)
> 퍼사이드 디자인 패턴 [그림 출처(original source)](https://medium.com/@JeffLombardJr/when-and-why-to-use-graphql-24f6bce4839d)

[위키피디아 설명](https://en.wikipedia.org/wiki/Facade_pattern)

### 멀티 패턴(Multi Pattern): Combination

이 패턴들을 섞는 것도 가능합니다. 예를 들면 컴포지트 패턴에서 오래된 api에 대한 facade 패턴을 추가할 수도 있죠. 

### 안티 패턴(Anti Pattern): Do Nothing Proxy

그냥 GraphQL을 사용하고 싶어서, GraphQL을 존재하는 API의 랩퍼(Wrapper)로 이용하진 말아야 합니다. 위에서 서버에서 서버로의 대부분의 패턴은 성능 고려사항들이 있습니다. 만일 다른 인터페이스로 정확히 같은 엑세스를 제공하고 싶다면, 클라이언트를 위한 웹 sdk를 고려해보세요.

# 내 간단한 생각과 결론

GraphQL의 가장 큰 목적은 아무래도 **유연성의 증대**와 **시스템의 단순화**로 볼 수 있을 것 같다. 간단한 하나의 엔드포인트에서 다양한 방식의 GraphQL 쿼리를 보내어 유연하게 다양한 정보를 받을 수 있다. 

그리고 
