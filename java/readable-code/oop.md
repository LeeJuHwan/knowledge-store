# Readable code

<details>

<summary>Properties</summary>

:pencil:2024.08.16

:page_facing_up: [읽기 좋은 코드를 작성하는 사고법](https://www.inflearn.com/course/readable-code-%EC%9D%BD%EA%B8%B0%EC%A2%8B%EC%9D%80%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%EC%82%AC%EA%B3%A0%EB%B2%95/dashboard)

:paperclip: 추상의 관점으로 바라보는 객체 지향, 객체 설계하기 (1)


</details>

## 객체지향 패러다임
- 비공개 필드(데이터) + 비공개 메서드(코드)의 조합을 공개 메서드로 외부 객체와 소통 하는 방식
- 외부 객체와 소통 하는 방식 덕분에 객체의 협력과 책임이 발생


{% hint style="info" %}
 객체가 제공 하는 것
- 객체의 생성은 관심사를 기준으로 하기 때문에 유지 보수의 장점 제공
- 다양한 객체가 협력 할 때 구현하는 코드 레벨에서 소통 하는 것이 아닌 추상화 레벨에서 비즈니스 로직을 다룰 수 있는 방식 제공
{% endhint %}


### 새로운 객체를 만들 때 주의 할 점

1. **관심사가 여러개는 아닌지 확인하기**

	- 해당 객체를 왜 만들어야 하는가?

	- 객체로 인해 외부 객체와 어떤 소통을 하고자 하는가?

2. **`setter` 사용 자제**

	- 가변적인 데이터는 어떠한 사이드 이펙트의 여지를 남길 수 있음

	- 외부 데이터를 이용하지 않고 객체 내부에서 처리가 가능한지 확인하기

	- 외부 데이터를 이용 해야 하는 경우
		- 단순 데이터를 업데이트 하는 네이밍 사용하기 → ex) `updateAction`
	
	**Example use case**:

	```java
	public class Cell {  
		private int nearbyLandMineCount;
		
		private Cell(int nearbyLandMineCount) {  
			this.nearbyLandMineCount = nearbyLandMineCount;  
		}
	
		public void updateNearbyLandMineCount(int count) {  
			this.nearbyLandMineCount = count;  
		}
	}
	```

3. **`getter` 메서드는 요구사항에 의해 변경되지 않는 한 초기 생성 자제, 꼭 필요한 경우만 생성**

	- 아래 예시를 보고 `getter` 는 폭력적인 메서드라는 것을 기억하자

	- 상황: 친구들과 술집에 갔는데 종업원이 신분증을 검사 하고 있다.

		- `getter` 를 사용 하는 경우의 코드 해석
			- 종업원이 다짜고짜 다가와 내 주머니에서 지갑을 꺼내고 지갑에서 신분증을 꺼내고 나이를 확인 한 다음 입장을 시킨다.

		- 객체 메서드를 이용하는 경우
			- 종업원이 손님에게 "19" 살 이상인지 정중하게 묻고 맞다면 입장시킨다.

	- 위 상황을 어울러 봤을 때 `getter` 는 얼마나 폭력적인가? 객체를 존중 하도록하자.

	```java
	Person person = new Person();

	// Getter를 이용하는 경우
	if (person.get지갑().get신분증().findAge() >= 19) {
		pass();
	}

	// 객체의 공개 메서드를 이용 하는 경우
	if (person.isAgeGreaterThanOrEqualTo(19)) {
		pass();
	}
	```

	**Example use case**:

	```java
	public class Cell {  
  
	    private static final String FLAG_SIGN = "⚑";  
	    private static final String LAND_MINE_SIGN = "☼";  
	    private static final String UNCHECKED_SIGN = "□";  
	    private static final String EMPTY_SIGN = "■";  
	  
	    private int nearbyLandMineCount;  
	    private boolean isLandMine;  
	    private boolean isFlagged;  
	    private boolean isOpened;


		public String getSign() {  
		    if (isOpened) {  
		        if (isLandMine) {  
		            return LAND_MINE_SIGN;  
		        } 
		        if (hasLandMineCount()) {  
		            return String.valueOf(nearbyLandMineCount);  
		        }
		        return EMPTY_SIGN;  
		    }  
		    if (isFlagged) {  
		        return FLAG_SIGN;  
		    }
		      
		    return UNCHECKED_SIGN;
		    }
	}
	```

4. **무분별한 `getter`, `setter` 대신 객체에게 메세지를 보낼 것**

5. **객체가 갖고 있는 필드는 적을수록 좋다**

	- 필드가 많을수록 처리해야 할 복잡한 내용들이 증가함

	- 일부 필드를 사용 하는 대신 메서드로 해결 할 수 있는것인지 고민 해볼 것

		- 단, 시간복잡도가 큰 메서드를 제공할 때 매번 호출되는 것이 성능상 문제가 생긴다면 필드로 사전에 최초 생성하는 방식이 더 나을 수 있음


## SOLID 원칙 알아보기

### SRP: Single Responsibility Principle

- 하나의 클래스는 단 한 가지의 변경 이유만을 가져야 한다.
- 관심사의 분리
- 높은 응집도, 낮은 결합도
	- → ‘변경 이유’ = 책임
- 객체가 가진 공개 메서드, 필드, 상수 등은
- 해당 객체의 단일 책임에 의해서만 변경 되는가?

{% hint style="info" %}
쉽게 예를 든다면 아래와 같이 화면에 보여주는 동작을 하는 구현체와 가까운 코드 뭉치를 `OutputHandler` 가 관리를 하여 메서드를 추상화 시키면 메인 코드에서 쉽게 관리할 수 있고 사용자에게 출력 할 내용을 하나의 객체에서 관리할 수 있고 반대로 사용자가 입력해야 하는 구현체를 갖고 있는 객체는 `InputHandler` 에서 관리할 수 있다.

이렇게 입력과 출력을 관리하는 객체는 극히 작은 예시일 뿐 이지만 비즈니스 로직을 담고 있는 코드 객체는 본인의 책임에 따른 관심사를 기준으로 잡는다면 메인 로직이나 바라보는 관점이 달라질 수 있기 때문에 항상 메서드의 시그니처를 생각 할 때 어떤 정보가 필요한지, 해당 객체에서 관리하는게 맞는 것인지 고민을 하는 습관이 필요하다.
{% endhint %}
	

**Example use case**:

```java
public class ConsoleOutputHandler {  

	public void showGameStartComments() {  
		System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");  
		System.out.println("지뢰찾기 게임 시작!");  
		System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");  
	}  
	public void showBoard(GameBoard board) {  
		System.out.println("   a b c d e f g h i j");  
		for (int row = 0; row < board.getRowSize(); row++) {  
			System.out.printf("%d  ", row + 1);  
			for (int col = 0; col < board.getColSize(); col++) {  
				System.out.print(board.getSign(row, col) + " ");  
			}            System.out.println();  
		
		}        System.out.println();  
	}
}

public class ConsoleInputHandler {  
	public static final Scanner SCANNER = new Scanner(System.in);  
	
	
	public String getUserInput() {  
		return SCANNER.nextLine();  
	}
}

public class Minesweeper {
	public void run() {  
		consoleOutputHandler.showGameStartComments();
	}

	private String getCellInputFromUser() {  
		consoleOutputHandler.printCommentForSelectingCell();  
		return consoleInputHandler.getUserInput();  
	}
}
```
