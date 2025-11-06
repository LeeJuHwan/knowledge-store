# 학습 테스트

{% hint style="info" %}
#### 잘 모르는 기능, 라이브러리, 프레임워크를 학습하기 위해 작성하는 테스트

여러 테스트 케이스를 스스로 정의하고 검증하는 과정을 통해 보다 구체적인 동작과 기능을 학습할 수 있다.

공식문서는 매우 우수한 정보를 제공 하지만 직접 작성 하며 학습할 수 있다.
{% endhint %}



"이번에 어떤 라이브러리가 좋다더라, 프레임워크가 좋다더라" 주변에서 이런 말 들을 종종 듣곤한다.

나의 첫 기술 비교는 Pandas vs Polars 였고, 실제 Polras 공식문서를 읽으면서 데이터 분석 해커톤에서 적용해서 사용했었다.

하지만 그 땐 공식문서를 읽어가면서 작업 하다보니 실제 사용해야 할 코드에서 주석과 코드 검증이 많아 매우 보기 힘들었던 단점이 분명했다.

**학습 테스트**는 이런 단점을 개선하면서 공식 문서를 읽으면서 학습하는 것 보다는 조금 더 다이나믹하게 학습할 수 있다.

### 학습 테스트 구성하기

학습 테스트를 위해 한 번도 사용해본 적 없는 구글에서 제공하는 [Guava](https://github.com/google/guava) 라는 라이브러리를 학습할 예정이다.

Java 환경에서 사용하는 컬렉션을 훨씬 더 수월하게 데이터를 핸들링 하고 관리할 수 있게하는 라이브러리로 간단한 예제 몇 개만 살펴보며 코드를 작성한다.



{% stepper %}
{% step %}
#### 제공하는 라이브러리의 메서드 살펴보기

이 땐 문서를 살펴보거나, 블로그를 찾아보거나, IDE 에서 제공하는 메서드 목록을 살펴보는 단계이다.

내가 현재 이 라이브러리의 어떤 기능을 이용할 수 있는지 우선 파악을 해야하는 정보 수집 단계이다.


{% endstep %}

{% step %}
#### 메서드의 결과를 예측하며 검증하기

필요한 정보를 수집 했다면 이제 이 메서드가 어떻게 동작하는지 마음것 테스트를 작성한다.

메서드의 결과를 예측하고 그 결과에 대한 정답을 보면서 어떻게 동작하는지 배워나가는 과정이다.
{% endstep %}
{% endstepper %}



학습 테스트는 메서드 명칭, 테스트 디스플레이 네임 등 구조화 되지 않은 날 것의 상태여도 좋다. 우리가 배우기 위해 장난감을 가지고 노는 퍼즐이라고 생각하자.

퍼즐 판을 구매했고, 하나씩 끼워 맞춰보고 다른 퍼즐과 바꿔보기도 하고 우린 이렇게 퍼즐을 완성 시키는 과정에 놓여있는 것이다.



**학습 테스트 코드의 산출물**

{% tabs %}
{% tab title="테스트 코드" %}
```java
class GuavaLearningTest {

    @DisplayName("주어진 개수만큼 List를 파티셔닝한다.")
    @Test
    void partitionLearningTest1() {
        // given
        List<Integer> integers = List.of(1, 2, 3, 4, 5, 6);

        // when
        List<List<Integer>> partition = Lists.partition(integers, 3);

        // then
        assertThat(partition).hasSize(2)
                .isEqualTo(List.of(
                        List.of(1, 2, 3),
                        List.of(4, 5, 6)
                ));
    }

    @DisplayName("주어진 개수만큼 List를 파티셔닝한다")
    @Test
    void partitionLearningTest2() {
        // given
        List<Integer> integers = List.of(1, 2, 3, 4, 5, 6);

        // when
        List<List<Integer>> partition = Lists.partition(integers, 4);

        // then
        assertThat(partition).hasSize(2)
                .isEqualTo(List.of(
                        List.of(1, 2, 3, 4),
                        List.of(5, 6)
                ));
    }

    @DisplayName("멀티맵 기능 확인")
    @Test
    void multimapLearningTest1() {
        // given
        Multimap<String, String> multimap = ArrayListMultimap.create();
        multimap.put("커피", "아메리카노");
        multimap.put("커피", "카페라떼");
        multimap.put("커피", "카푸치노");
        multimap.put("베이커리", "크루아상");
        multimap.put("베이커리", "식빵");

        // when
        Collection<String> strings = multimap.get("커피");

        // then
        assertThat(strings).hasSize(3)
                .isEqualTo(List.of("아메리카노", "카페라떼", "카푸치노"));
    }

    @DisplayName("멀티맵 기능 확인")
    @TestFactory
    Collection<DynamicTest> multimapLearningTest2() {
        // given
        Multimap<String, String> multimap = ArrayListMultimap.create();
        multimap.put("커피", "아메리카노");
        multimap.put("커피", "카페라떼");
        multimap.put("커피", "카푸치노");
        multimap.put("베이커리", "크루아상");
        multimap.put("베이커리", "식빵");

        return List.of(
                DynamicTest.dynamicTest("1개 value 삭제", () -> {
                    // when
                    multimap.remove("커피", "카푸치노");

                    // then
                    Collection<String> results = multimap.get("커피");
                    assertThat(results).hasSize(2)
                            .isEqualTo(List.of("아메리카노", "카페라떼"));
                }),
                DynamicTest.dynamicTest("1개 key 삭제", () -> {
                    // when
                    multimap.removeAll("커피");

                    // then
                    Collection<String> results = multimap.get("커피");
                    assertThat(results).isEmpty();
                })
        );
    }
}
```
{% endtab %}
{% endtabs %}

학습을 위한 테스트 코드는 아무래도 실제 코드와 같이 반영될 수 없는데 만약 팀 내 모두가 동일하게 학습해야 하는 환경이라면 남겨두어서 같이 공유할 수 있다.

하지만, 스스로 학습만을 위한 코드라면 테스트 후 삭제하여 프로덕션에는 반영하지 않도록 유지하는 것이 낫다.





