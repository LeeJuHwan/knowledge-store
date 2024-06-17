<details>

<summary>Properties</summary>

:pencil:2024.06.16

:computer: [source_code](https://github.com/java-playground-hiking/java-baseball/pull/5/commits/aac1aebcf5911e37d45ba1d55b017f7d6fb30a31)

</details>

## 넥스트 스텝: 자바 플레이그라운드 - 숫자 야구게임


### 숫자야구게임 - 학습 테스트 실습

---


<br>

### StringClass 학습 테스트
---

✅ **요구사항 1**

- "1, 2"을 `,`로 `split` 했을 때 1과 2로 잘 분리 되는지 테스트 케이스 작성
- "1"을 `,`로 `split` 했을 때 1만을 포함하는 배열이 반환 되는지 테스트 케이스 작성

> 힌트 키워드
- assertj contains()
- assertj containsExactly()

> 내용

학습 키워드가 어떤 상황에 어떤 테스트 메서드로 검증 해야 하는지 명확하게 나와 있었기 때문에 어려운 내용이 없었다. 본 미션을 들어가기 전 미션을 진행 하는 방법을 알려주는 워밍업 단계 인 것 같다. 그럼에도 불구하고 Java 생태계에서 실무를 하고 있지 않았기 때문에 구글링을 바탕으로 미션을 해결 하고있다.

> 참고 자료
- 검색 키워드: Google > "java assertj contains", "how to write unit test  java"
- [AssertJ/contains](https://bcp0109.tistory.com/317)
- [given-when-then/TestCase](https://medium.com/@gitaeklee/given-when-then-junit-test-ba49564303e7)

<br>

✅ **요구사항 2**

- "(1,2)" 값이 주어졌을 때 ()을 제거하고 "1,2"를 반환하는 테스트 케이스 작성

> 힌트 키워드
- String substring

> 내용

요구사항 1번을 해결 하면서 나름의 눈치를 배웠는지 어떠한 검색도 없이 바로 String 클래스의 `substring`에 접근 해서 원하는 데이터를 뽑은 후 검증을 진행 했다.


<br>

✅ **요구사항 3**

- "abc" 값이 주어졌을 때 특정 위치의 문자를 가져온 후 검증 하는 테스트 케이스 작성
- IndexOutOfBound Exception 케이스 작성
- DisplayName으로 테스트 케이스에 대한 내용 작성

> 힌트 키워드
- [AssertJ Exception Assertions 문서 링크](https://joel-costigliola.github.io/assertj/assertj-core-features-highlight.html#exception-assertion)

> 내용

`DisplayName` 테스트 메서드를 보자마자 다른 테스트 케이스에도 모두 작성 해줬다. 기존에는 파이썬의 Docstring 형태로 주석을 달아서 사용 하려고 했지만 매우 유용한 메서드가 있었기 때문에 안 쓸 이유가 없었다. 

힌트에 제공 해준 문서를 보자마자 바로 해결 할 수 있을 것 같다고 판단 했는데 문서에 명시 된 상황과 내가 작성하는 상황과 유사한 느낌이 들지 않아서 구글링을 한 번 더 하여 정확하게 키워드를 뽑아내었다.  결국 `assertThatThrownBy` 내부에서 실행 한 함수를 `Exception` 으로 잡은 뒤 체이닝 방식으로 검증을 이어나갈 수 있다는 걸 배웠다. 이렇게 되면 Given, When, Then 구조가 정확하게 나온다.
- Given: "abc" 값
- When: `charAt(4)` 을 `assertThatThrownBy`를 이용 하여 `Exception`객체로 변환
- Then: `isInstanceOf`, `hasMessageContaining` 을 이용한 검증

> 참고 자료
- 검색 키워드: Google > "Junit exception test"
- [assertThatThrownBy](https://covenant.tistory.com/256)

<br>

### SetCollection 학습 테스트
---

**Fixture**

```java
    @BeforeEach
    void setUp() {
        numbers = new HashSet<>();
        numbers.add(1);
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
    }
```


✅ **요구사항 1**

- Fixture numbers의 `size` 메서드를 활용해 데이터 크기 검증

> 힌트 키워드

- 없음

> 내용

워낙 간단한 코드이기도 하고, 요구사항이 명확했기 때문에 특별한 검색 없이 바로 검증을 할 수 있었는데 `StringClass` 학습 테스트 파일은 기존에 스켈레톤 코드 형식으로 작성 되어있는 부분이 있었다. 하지만, `SetCollection` 학습 테스트 파일은 Scratch로 다시 작성 해야했기 때문에 `import` 하는 내용이 이렇게 많았었나 싶었다.

> 참고 자료

- 없음

<br>

✅ **요구사항 2**

- Fixture numbers의 `contains` 메서드를 활용해 데이터 존재 여부 검증
    - 단, 반복 되는 코드를 최적화 할 것

> 힌트 키워드

- [Parameterized Test](https://www.baeldung.com/parameterized-tests-junit-5)


> 내용

`Parameterized Test`는 `StringClass` 학습 테스트 당시 사용 해봤기 때문에 내용을 어느정도 알고 있었다. 아는듯 한 느낌이 아닌 안다고 확신 할 수 있었기 때문에 해당 요구사항도 별 다른 검색 없이 진행 할 수 있었다.

> 참고 자료

- 없음

<br>

✅ **요구사항 3**

- Fixture numbers의 `contains` 메서드를 활용해 데이터 유무 검증 및 예외 케이스(존재 하지 않는 데이터 일 때) 검증

> 힌트 키워드

- Parameterized Test의 `@CsvSource`

> 내용

테스트 메서드에 Key, Value로 매핑되는 값을 받을 수 있어 정적인 데이터를 검증 할 때 편리했다. 그 중 처음 `StringClass` 학습 테스트 때 사용 했을 땐 `delimiter` 를 몰랐었던 터라 사용하지 않고 쉼표로 구분 했었는데 콜론으로 구분 할 수 있어서 가독성이 조금 더 나아진 것 같다.

> 참고 자료

- 없음
