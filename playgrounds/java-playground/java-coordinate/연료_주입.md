# 연료 주입

#### 요구사항

{% hint style="info" %}
우리 회사는 렌터카를 운영하고 있다. 현재 보유하고 있는 차량은 Sonata 2대, Avante 1대, K5 2대로 총 5대의 차량을 보유하고 있다.\
우리 회사는 고객이 인터넷으로부터 예약할 때 여행할 목적지의 대략적인 이동거리를 입력 받는다. 이 이동거리를 활용해 차량 별로 필요한 연료를 주입한다.

차량 별로 주입해야 할 연료량을 확인할 수 있는 보고서를 생성해야 한다.



<details>

<summary>차종 별 연비</summary>

```
* Sonata : 10km/리터
* Avante : 15km/리터
* K5 : 13km/리터
```

</details>
{% endhint %}



#### 상속을 이용한 문제 풀이

***

{% tabs %}
{% tab title="Abstract Car" %}
```java
package abstract_rentcar;

public abstract  class AbstractRentCar {
    private final int distance;

    public AbstractRentCar(int distance) {
        this.distance = distance;
    }

    public abstract int getFuelEfficiency();
    public abstract String getCarName();

    public String getRentCarInfo() {
        return getCarName() + " : " + getRequiredRemainingFuel() + "리터";
    }

    public int getRequiredRemainingFuel() {
        return distance / getFuelEfficiency();

    }
}
```
{% endtab %}

{% tab title="K5" %}
```java
package abstract_rentcar;

public class K5 extends AbstractRentCar {

    private final int fuelEfficiency = 13;
    private final String carName = "K5";

    public K5(int distance) {
        super(distance);
    }

    @Override
    public String getCarName() {
        return carName;
    }

    @Override
    public int getFuelEfficiency() {
        return fuelEfficiency;
    }
}
```
{% endtab %}

{% tab title="Sonata" %}
```java
package abstract_rentcar;

public class Sonata extends AbstractRentCar {

    private final int fuelEfficiency = 10;
    private final String carName = "Sonata";

    public Sonata(int distance) {
        super(distance);
    }

    @Override
    public String getCarName() {
        return carName;
    }

    @Override
    public int getFuelEfficiency() {
        return fuelEfficiency;
    }
}
```
{% endtab %}

{% tab title="Avante" %}
```java
package abstract_rentcar;

public class Avante extends AbstractRentCar {

    private final int fuelEfficiency = 15;
    private final String carName = "Avante";

    public Avante(int distance) {
        super(distance);
    }

    @Override
    public String getCarName() {
        return carName;
    }

    @Override
    public int getFuelEfficiency() {
        return fuelEfficiency;
    }
}
```
{% endtab %}
{% endtabs %}



#### 인터페이스를 이용한 문제 풀이

***

{% tabs %}
{% tab title="Implement Car" %}
```java
package implement_rentcar;

public interface ImplementRentCar {

    int getFuelEfficiency();

    String getCarName();

    String getRentCarInfo();

    int getRequiredRemainingFuel();

}
```
{% endtab %}

{% tab title="K5" %}
```java
package implement_rentcar;

public class K5 implements ImplementRentCar {

    private final int fuelEfficiency = 13;
    private final String carName = "K5";
    private final int distance;

    public K5(int distance) {
        this.distance = distance;
    }

    @Override
    public int getFuelEfficiency() {
        return fuelEfficiency;
    }

    @Override
    public String getCarName() {
        return carName;
    }

    @Override
    public String getRentCarInfo() {
        return getCarName() + " : " + getRequiredRemainingFuel() + "리터";
    }

    @Override
    public int getRequiredRemainingFuel() {
        return distance / getFuelEfficiency();
    }


}
```
{% endtab %}

{% tab title="Sonata" %}
```java
package implement_rentcar;

public class Sonata implements ImplementRentCar {

    private final int fuelEfficiency = 10;
    private final String carName = "Sonata";
    private final int distance;

    public Sonata(int distance) {
        this.distance = distance;
    }

    @Override
    public int getFuelEfficiency() {
        return fuelEfficiency;
    }

    @Override
    public String getCarName() {
        return carName;
    }

    @Override
    public String getRentCarInfo() {
        return getCarName() + " : " + getRequiredRemainingFuel() + "리터";
    }

    @Override
    public int getRequiredRemainingFuel() {
        return distance / getFuelEfficiency();
    }
}
```
{% endtab %}

{% tab title="Avante" %}
```java
package implement_rentcar;

public class Avante implements ImplementRentCar {

    private final int fuelEfficiency = 15;
    private final String carName = "Avante";
    private final int distance;

    public Avante(int distance) {
        this.distance = distance;
    }

    @Override
    public int getFuelEfficiency() {
        return fuelEfficiency;
    }

    @Override
    public String getCarName() {
        return carName;
    }

    @Override
    public String getRentCarInfo() {
        return getCarName() + " : " + getRequiredRemainingFuel() + "리터";
    }

    @Override
    public int getRequiredRemainingFuel() {
        return distance / getFuelEfficiency();
    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
위 두 가지 문제 접근법을 살펴보자면, _<mark style="color:green;">**상속**</mark>_ 과 _<mark style="color:green;">**인터페이스**</mark>_ 이다.



상속을 이용한 코드는 비교적 부모의 메서드를 사용하는 자식의 입장에서 별도로 구현 할 필요가 없기 때문에 코드 작성 양이 굉장히 줄어들었다.



반면, 인터페이스를 활용한 코드는 자식이 모든 것을 구현해야 하기 때문에 코드의 양이 상대적으로 많아진 모습이다.

이 둘의 차이는 "메서드 구현 담당을 어디서 하느냐?" 의 관점에서 바라볼 수 있는데 애석하게도 단순한 코드가 올바른 접근 방법은 아니라는 것이다.



<mark style="color:red;">**상속이 갖고 있는 단점**</mark>은 부모가 모든 것을 구현했기 때문에 자식과 강결한 결합 관계를 나타낸다.

* 강결합인 경우 유연하지 못하기 때문에 사소한 요구사항에 의해 변경사항을 반영하기 굉장히 까다롭다는 점이다.

그럼, 이런 단점이 두각된 설계 방식을 왜 사용할까?

* 프레임워크 수준에서 정의된 부모 - 자식 관계의 코드 재사용이 훨씬 용이한 경우, 부모가 갖고 있는 메서드의 변경이 없을 경우

<mark style="color:red;">**인터페이스는 상속과 반대로 코드를 재사용할 수 없어 중복 코드가 많이 발생하는 단점**</mark>이 나타난다.

* 하나의 공통 인터페이스는 여러 곳에서 구현하기 때문에 확장에 용이함이 있다.
{% endhint %}



이번 진행한 미션은 상속과 인터페이스에 대해 다루었고 두 설계 방식을 코드로 구현 하면서 어떤 관점에서 편리한지, 불편한지 등 몸소 경험할 수 있는 미션이었다.



또한 해당 미션은 학습 테스트라는 방식으로 제공 되었는데, 학습 하는 대상이 훨씬 더 접근하기 쉽도록 스켈레톤 코드가 일부 제공 되고 그 코드를 퍼즐 맞추듯 작성한 뒤 모든 테스트를 통과 시키면 된다.

이 방법은 코드 작성자가 직접 테스트를 할 필요 없이 작성된 테스트가 통과 할 수 있는 선에서 많은 리팩터링을 할 수 있도록 만들어내어 편리했다.

