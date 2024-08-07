# Readable code

## 인지적 경제성을 추구하자

> 특정 대상을 특징에 따라 분류 하는 범주화를 기준으로 최소한의 정보로 최대한을 기억 해낼 수 있다.

---

<br></br>

### Early Return으로 else의 사용을 지양하자

```java
if (a > 3) {
	code();
} else if (a <= 3 && b > 1) {
	code2();
} else {
	code3();
}

```

위와 같은 코드가 있을 때 코드를 읽는 대상은 조건문의 흐름도에 따라 선수 조건이 만족 했는지에 대한 정보를 지속적으로 기억하고 있어야한다. 그렇게 매우 긴 코드를 거쳐 마지막 `else` 문을 읽을 때 쯤 되면 첫 조건을 잊을 수도 있다. 그렇다면 다시 돌아가야 할까?

**Example use case**:

- :thumbsdown: **BAD!**
	```java
		int selectedColIndex = getSelectedColIndex(cellInput);
		int selectedRowIndex = getSelectedRowIndex(cellInput);
		if (doesUserChooseToPlantFlag(userActionInput)) {
			board[selectedRowIndex][selectedColIndex] = "⚑";
			checkIfGameIsOver();
		} else if (doesUserChooseToOpenCell(userActionInput)) {
			if (isLandMineCell(selectedRowIndex, selectedColIndex)) {
				board[selectedRowIndex][selectedColIndex] = "☼";
				changeGameStatusToLoose();
				continue;
			} else {
				open(selectedRowIndex, selectedColIndex);
			}
			checkIfGameIsOver();
		} else {
			System.out.println("잘못된 번호를 선택하셨습니다.");
		}
	```

- :thumbsup: **GOOD!**
	```java
		actionOnCell(cellInput, userActionInput);

		private static void actionOnCell(String cellInput, String userActionInput) {
			int selectedColIndex = getSelectedColIndex(cellInput);
			int selectedRowIndex = getSelectedRowIndex(cellInput);
			if (doesUserChooseToPlantFlag(userActionInput)) {
				BOARD[selectedRowIndex][selectedColIndex] = FLAG_SIGN;
				checkIfGameIsOver();
				return;
			}
			if (doesUserChooseToOpenCell(userActionInput)) {
				if (isLandMineCell(selectedRowIndex, selectedColIndex)) {
					BOARD[selectedRowIndex][selectedColIndex] = LAND_MINE_SIGN;
					changeGameStatusToLoose();
					return;
				}
				open(selectedRowIndex, selectedColIndex);
				checkIfGameIsOver();
				return;
			}
			System.out.println("잘못된 번호를 선택하셨습니다.");
		}
	```

앞선 조건을 다시 생각 해보자면 첫번째 조건이 만족 해야 한다는 정보가 필요 없어지고 개별 조건에 따른 정보만 얻으면 된다. 이렇게 `else` 는 내가 처리하지 못하는 예외 케이스를 접근하게 하기도 하며 까다로운 상황을 만들 수 있는 요인이 되기 때문에 최대한 지양하는 방법을 사용해보자.


### 사고의 Depth 줄이기


**리팩터링을 위한 메서드 변경 팁 그리고 리팩터링을 해야 하는 이유**

{% hint style="info" %}

- 리팩터링 할 때 수정하는 메서드가 다른 모듈에서 참조 하고 있어 잘못된 수정 시 컴파일 에러를 발생 시키는 상황이라면 개발자는 매우 골치 아픈 리팩터링 과정이 될 것이다. 이럴때는 **동일한 메서드의 이름만 살짝 수정하여 메서드 리팩터링을 진행 하고 정상 동작을 확인한 후 기존 메서드와 변경** 하는 방법을 사용하면 리팩터링에 대한 피로가 덜 할것이다.

- 추상화를 하는 과정은 읽는 사람이 읽는 동안 해당 코드가 언제 쓰이는지를 지속적으로 피로감을 느끼도록 하는 것을 줄이는게 핵심

{% endhint %}

> 중첩 분기문, 중첩 반복문
> - **BAD!**
> 	```java
> 	for (int i = 0; i < 20; i++) {
> 		for (int j = 20; j < 30; j++) {
> 			if (i >= 10 && j < 25) {
> 				code();
> 			}
> 		}
> 	}
> 	```
> 	- 이 코드를 읽어보면 반복문이 순회 하는 `i` 와 중첩 반복문이 순회 되는 `j` 리고 조건분기의 조건이 만족하는 시점을 우린 항시 기억하며 코드를 읽어야 한다. 그 와중에 조건이 부합할 시 코드 내부를 파악해야 한다면? 우린 기억에 남지 못하는 코드를 쭉 읽어왔던 것이다.
> - **BETTER!**
> 	```java
> 	for (int i = 0; i < 20; i++) {
> 		codeWithI(i);
> 	}
> 	
> 	private void codeWithI(int i) {
> 		for (int j = 20; j < 30; j++) {
> 			codeWithIJ(i, j);
> 		}
> 	}
> 	
> 	private void codeWithIJ(int i, int j) {
> 		if (i >= 10 && j < 25) {
> 			code();
> 		}
> 	}
> 	```
> 	- 메서드 추출만으로 사고의 depth를 줄여 읽는 과정에서 흐름을 방해하지 않는 선택이 나을 수 있다.


{% hint style="info" %}

- ❗"무조건 1depth로 만들다"가 아니다.
	- 보이는 depth를 줄이는 데 급급한 것이 아닌 추상화를 통한 사고 과정의 depth를 줄이는 것이 중요함
- 2중 중첩 구조로 표현하는 것이 읽는 데 도움이 된다고 판단 된다면 메서드 분리보다 그대로 놔두는 것이 옳은 선택일 수 있다.
- 무조건 메서드를 분리하는 것은 더 혼선을 줄 수 있다

{% endhint %}

**Example use case**:

- :thumbsdown: **BAD!**
	
	```java
	private static boolean isAllCellOpened() {  
	    boolean isAllOpened = true;  
	    for (int row = 0; row < BOARD_ROW_SIZE; row++) {  
	        for (int col = 0; col < BOARD_COL_SIZE; col++) {  
	            if (BOARD[row][col].equals(CLOSED_CELL_SIGN)) {  
	                isAllOpened = false;  
	            }
			}
		}

		return isAllOpened;  
	}
	```

- :thumbsup: **GOOD!**
	```java
	private static boolean isAllCellOpened() {  
	    return Arrays.stream(BOARD)  
	            .flatMap(Arrays::stream)  
	            .noneMatch(cell -> cell.equals(CLOSED_CELL_SIGN));  
	}
	```


Stream이 일반 for loop 그리고 foreach문 보다 더 낫다는 설명이 아니다. 코드를 읽을 때 지나친 중첩의 반복문이 있다면 함수형 프로그래밍의 도움을 받아 읽는데 효과적으로 만들 수 있다면 사용하는 이유가 충분할 것이다. 만약 중첩 반복문을 깨트리고 함수형 프로그래밍을 시도 했을 때 가독성이 더 나빠진다면 그 방법은 옳지 않은 것이니 다른 방법을 생각 해봐야한다.



> 사용할 변수는 가깝게 선언하기

사용 해야 할 변수와 거리가 먼 경우는 기존 선언된 변수의 위치에서 사용하기까지 언제 쓰이는지를 계속 신경 쓰고 있어야한다. 지금까지 강조 되었던 추상화를 하는 과정은 읽는 사람이 읽는 동안 해당 코드가 언제 쓰이는지를 지속적으로 피로감을 느끼도록 하는 것을 줄이는게 핵심이었다. 그렇다면 이 내용도 그에 맞게 리팩터링을 할 수 있다.


**Example use case**:

  

- :thumbsdown: **BAD!**

	```java
	public static void main(String[] args) {  
	    showGameStartComments();  
	    Scanner scanner = new Scanner(System.in);  // 스캐너 생성
	    initializeGame();  
	    while (true) {  
	        showBoard();  
	        if (doesUserWinTheGame()) {  
	            System.out.println("지뢰를 모두 찾았습니다. GAME CLEAR!");  
	            break;  
	        }        
			if (doesUserLoseTheGame()) {  
	            System.out.println("지뢰를 밟았습니다. GAME OVER!");  
	            break;  
	        }        
			String cellInput = getCellInputFromUser(scanner);  // 스캐너 사용 
	        String userActionInput = getUserActionInputFromUser(scanner);  
	        actionOnCell(cellInput, userActionInput);  
	    }
	}


	private static String getUserActionInputFromUser(Scanner scanner) {  
		System.out.println("선택한 셀에 대한 행위를 선택하세요. (1: 오픈, 2: 깃발 꽂기)");  
	    return scanner.nextLine();  
	}  
  
	private static String getCellInputFromUser(Scanner scanner) {  
	    System.out.println("선택할 좌표를 입력하세요. (예: a1)");  
	    return scanner.nextLine();  
	}
	```

	- Best Practice로 넘어가기 전 위 코드에서 Scanner의 변수 위치와 사용되는 위치는 거리가 있다는 것을 알 수 있지만 만약 Scanner를 사용하는 위치의 바로 위에 자리하게 된다면 `while` 문 안에서 계속 생성 되어 메모리 풀이 날 수 있다. 그렇다면 이 스캐너는 해당 모듈 전역 상수로 만든다면 파라미터를 줄이는 장점과 더불어 스캐너에 대한 생각을 줄일 수 있다.

- :thumbsup: **GOOD!**
	```java
	public static final Scanner SCANNER = new Scanner(System.in);
	
	public static void main(String[] args) {  
	    showGameStartComments();  
	    initializeGame();  
	    while (true) {  
	        showBoard();  
	        if (doesUserWinTheGame()) {  
	            System.out.println("지뢰를 모두 찾았습니다. GAME CLEAR!");  
	            break;  
	        }       
			if (doesUserLoseTheGame()) {  
	            System.out.println("지뢰를 밟았습니다. GAME OVER!");  
	            break;  
	        }        
			String cellInput = getCellInputFromUser();  
	        String userActionInput = getUserActionInputFromUser();  
	        actionOnCell(cellInput, userActionInput);  
	    }
	}

	private static String getUserActionInputFromUser() {  
    System.out.println("선택한 셀에 대한 행위를 선택하세요. (1: 오픈, 2: 깃발 꽂기)");  
    return SCANNER.nextLine();  
	}  
  
	private static String getCellInputFromUser() {  
	    System.out.println("선택할 좌표를 입력하세요. (예: a1)");  
	    return SCANNER.nextLine();  
	}
	```