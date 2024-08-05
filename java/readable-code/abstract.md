# Readable code

> 가독성 좋은 코드를 작성하는 첫번째 원리 추상과 구체를 이해하자


{% hint style="info" %}
**추상**
- 중요한 정보는 가려내어 남기고, 덜 중요한 정보는 생략하여 버린다.

파블로 피카소 - "추상은 항상 구체적인 실재에서 시작해야 한다."
{% endhint %}

<br></br>

**추상화는 최소 정보의 단위를 묶어서 어떻게 부를 것이냐에 관점이며, 컴퓨터 과학은 이러한 추상화 과정이 조화롭게 겹겹이 쌓여서 만들어진다**

- 예를 들어보자면 1bit를 1byte, 1word 등 0과 1로만 이루어진 단위를 우리는 훨씬 더 큰 단위로 묶어서 부를 수 있고 그 단위로 인해 어플리케이션에서 제어할 수 있는 부분의 범위를 넓힌다.

{% hint style="info" %}
> 💬 읽기 좋은 코드를 작성 하기 위해 추상화의 개념을 왜 알아야 할까?

- 👉 <mark style="color:orange;background-color:purple;">적절한 추상화는 복잡한 데이터와 복잡한 로직을 단순화 하여 이해하기 쉽도록 돕는다</mark>
- 👉 즉, 읽기가 좋다.
{% endhint %}


> 추상화 된 내용으로부터 구체적인 정보를 해석하지 못하는 경우

1. **추상화 과정에서 중요한 정보를 부각 시키지는 못했을 때**
	- 상대적으로 덜 중요한 정보를 남기고 중요한 정보는 제거함
2. **해석자가 동일하게 공유하는 문맥이 없을 때**
	- 중요한 정보의 기준이 다를 수 있음
	- 도메인 영역 별 추상화 기준이 다를 수 있다는 걸 인지 해야함

	{% hint style="danger" %}
	- 잘못된 추상화가 야기하는 사이드 이펙트는 생각보다 정말로 크다.

	- 적절한 추상화는 해당 도메인의 문맥 안에서 정말 중요한 핵심 개념만 남겨서 표현 하는 것

	{% endhint %}


## :pencil: 추상화의 가장 대표적인 행위
---

### 🗒️ 이름을 짓는다
👉 가장 단순하면서도 아주 중요한, 고도의 추상적 사고 행위

> 단수와 복수를 구분하기
- 끝에 -(e)s 를 붙여 어떤 데이터가 단수인지, 복수인지 나타내는 것이 읽는이에게 중요한 정보를 전달 할 수 있음

> 이름 줄이지 않기
- 축약 할 때 효율성을 얻을 수 있으나 얻는 것에 비해 잃는 것이 큰 경우임
- 자제하는 것이 좋긴 하나 관용어처럼 많은 사람들이 자주 사용하는 줄임말 정도는 허용함

> 은어/방업 사용하지 않기

- 농담에서 파생된 용어나 일부 팀원, 현재 의 우리팀만 아는 용어 금지
	- 새로운 사람이 팀에 합류 했을 때 이 용어를 단번에 이해 할 수 없는 단어는 사용 하지 말아야 함

- 도메인 용어 사용하기
	- 도메인 용어를 먼저 정의 하는 과정이 필요할 수 있음
		- 예시) 매장 정보 → shop? store?

> 좋은 코드를 보고 습득하기
- 비슷한 상황에서 자주 사용하는 단어, 개념 습득하기
	- 예시) pool, candidate, threshold, etc


### 🙋‍♂️ **한 메서드의 주제는 반드시 하나다**

{% hint style="info" %}
- 잘 쓰여진 코드라면, 한 메서드의 주제는 반드시 하나다.
- 만약, 생략할 정보와 의미를 부여하고 드러낼 정보를 구분하기 어렵다면 메서가 2가지 이상의 일을 하고 있을 가능성이 높다.
{% endhint %}

### 🧰 메서드 선언(추출)

{% hint style="info" %}

반환타입 메서드명 (파라미터) {}

- 메서드명
	- 추상화된 구체를 유추할 수 있는 적절한 의미가 담긴 이름
	- 파라미터와 연결지어 더 풍부한 의미를 전달할 수도 있다.

- 파라미터
	- **파라미터의 타입, 개수, 순서를 통해 의미를 전달**
	- 파라미터는 외부 세계와 소통하는 창구
		- 파라미터는 해당 메서드가 어떠한 행위를 하기 위해서 필요한 정보를 알려주는 것
- 반환타입
	- 메서드 시그니처에 납득이 가는 적절한 타입의 반환 값 돌려주기
		- 반환 타입이 boolean인데 이게 이 메서드에서 무엇을 의미할까에 대한 고민이 길어진다면 다른 타입을 고민 해봐야함
	- void 대신 충분히 반환할 만한 값이 있는지 고민해보기
		- 반환값이 있다면 테스트가 용이함
{% endhint %}


### 💫 같은 패키지 내에서 추상화 레벨 맞추기

{% hint style="info" %}
책을 사러 서점에 갔다. 서점에서 책을 고르기 위해 책의 목차를 읽고 마음에 드는 책을 구매 하는데, 어느 날 보니 책들 사이에 목차가 없는 표지만 있는 책을 발견했다.

이 책을 본 고객은 당황할 수 밖에 없다. 서점의 오류인듯 아닌듯 의도된듯 아닌듯 하지만 이런 시점은 일상이 아닌 코드를 작성하는 시점에서도 발생 할 수 있다는 것이다.

코드를 읽다보니 추출된 메서드가 등장하고 메서드는 각자의 역할이 담겨져있다.
어느덧 중반부 쯤 와서 메서드 사이에 뜬금없이 자리 잡은 조건문은 구현체와 가까운 코드가 작성 되어 있는데 이런 코드는 읽는 사람이 추상화 된 메서드를 읽는 도중 만난다면 혼란을 줄 수 있다.

**🎓 코드를 읽는 모듈은 추상화 레벨이 동등해야한다.**

{% endhint %}

**Bad use case**: 

```java
public static void main(String[] args) {
	showGameStartComments();
	initializeGame();
	showBoard();
	...

	if (gameStatus == 1) {  // <- 추상화 레벨이 다른 구문
		System.out.println("게임 클리어");
		break;
	}
}
```


**Good use case**:

```java
public static void main(String[] args) {
	showGameStartComments();
	initializeGame();
	showBoard();
	...

	if (does?UserWinTheGame()) {
		System.out.println("게임 클리어");
		break;
	}
}
```


### 🌇 매직 넘버, 매직 스트링을 상수로 추출하기

{% hint style="info" %}
- 매직 넘버, 매직 스트링
	- 의미를 갖고 있으나 상수로 추출되지 않은 숫자나 문자
	- 상수 추출로 이름을 짓고 의미를 부여함으로 가독성 및 유지보수 향상 가능
{% endhint %}

**Examples:**

```java
public static final int BOARD_ROW_SIZE = 8;  
public static final int BOARD_COL_SIZE = 10;  
private static final String[][] BOARD = new String[BOARD_ROW_SIZE][BOARD_COL_SIZE];  
private static final Integer[][] NEARBY_LAND_MINE_COUNTS = new Integer[BOARD_ROW_SIZE][BOARD_COL_SIZE];  
private static final boolean[][] LAND_MINES = new boolean[BOARD_ROW_SIZE][BOARD_COL_SIZE];  
public static final int LAND_MINE_COUNT = 10;  
public static final String FLAG_SIGN = "⚑";  
public static final String LAND_MINE_SIGN = "☼";  
public static final String CLOSED_CELL_SIGN = "□";  
public static final String OPENED_CELL_SIGN = "■";
```

예시처럼 자주 사용 되거나 의미를 갖고 있는 숫자나 문자를 뜻 함