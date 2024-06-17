<details>

<summary>Properties</summary>

:pencil:2024.06.17

:computer: [source_code](https://github.com/java-playground-hiking/java-baseball/pull/8/commits/c9ced178a51f2856a80b5f02fb81c79d0811f02d)

</details>

## 넥스트 스텝: 자바 플레이그라운드 - 숫자 야구게임


### 숫자야구게임 - 1. 문자열 계산기 구현 및 단위 테스트 작성

---


<br>

### 문자열 계산기 기능 구현
---

✅ **요구사항**

1. 사용자가 입력한 문자열 값에 따라 사칙연산을 수행할 수 있는 계산기를 구현해야 한다.
2. 연산자 우선 순위는 적용 하지 않으며 왼쪽 부터 사칙연산을 수행한다.

> 문제 정의

1. 사용자가 입력할 수 있는 `Scanner`를 이용 할 `User`가 필요함.
2. 문자열을 입력 받아서 개별 요소에 접근이 쉬운 `Array` 배열로 변환이 필요함.
3. `Array`에서 개별 요소에 접근 할 때 마다 문자열 수식이 피연산자인지, 연산자인지 구분이 필요함.
4. 연산자에 따른 피연산자와의 연산을 수행 하는 핵심 비즈니스 로직이 필요함.

-> 필요한 내용을 코드로 옮길 때, 어떤 아키텍처 구조가 만들어져야 할까?

- 역할과 관심사 분리를 위한 계층 구조 설계
- 사용자의 입, 출력을 다루는 `Presentation Layer`
- 문자열 수식을 연산 할 `Service Layer`
- 어플리케이션을 실행 할 `Main`

☝️ 위 방식으로 가장 러프하게 코드가 작동 할 수 있도록 구현

<br>

> 셀프 리뷰

기능이 정상적으로 작동할 수 있는 가장 작은 단위의 개발을 시작 하고 클래스명을 기준으로 계층을 잠시 나눴다.
`CalculatorService`, `CalculatorUser`, `StringCalculatorMain` 이렇게 파일을 생성 한 뒤 가장 쉽게 접근 할 수 있는 사용자의 입력 값을 받는 과정을 먼저 구현 하고
`split` 메서드를 활용한 `String -> Array` 변환 작업을 마무리 지었다.

문자열 수식이 배열로 변환 된 상태에서 배열의 요소에 접근 하여 숫자와 연산자를 분리 해야 했는데, 파이썬에서는 `type`으로 쉽게 검사 할 수 있었으나 아쉽게 자바에서는 레퍼런스가 대부분 직접 구현 하는 내용이거나, Apache의 패키지를 이용 하는 것이었는데 미션을 해결하는 데 중점을 뒀다면 당연히 외부 모듈을 사용하여 손 쉽게 풀었을 것 같다. 하지만 자바에 익숙해지는 것이 목표이기 때문에 직접 구현했다.

** 직접 구현한 숫자 타입 검사 **

```java
/**
 * 문자로 이루어진 데이터를 받아 숫자 여부 확인
 * @param string 문자열 데이터
 * @return boolean - true: 숫자, false: 문자
 */
public boolean isNumeric(String string) {
    try {
        Integer.parseInt(string);
        return true;
    } catch (NumberFormatException e) {
        return false;
    }
}
```