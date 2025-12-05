---
description: 리플렉션 이해하기
---

# 런타임 도중 코드를 조작하는 마법, 리플렉션

### 리플렉션이란?

***

{% hint style="info" %}
#### 리플렉션

런타임에 클래스 로더에 의해 메모리에 로드된 클래스 정보를 통해 클래스의 구조(필드, 메서드, 생성자)를 분석하고 객체를 동적으로 생성하거나 메서드를 호출하는 기술이다.
{% endhint %}

이 리플렉션을 번역하면 반사 라고 직역이 되는데, 쉽게 이해를 돕기 위해 이 직역된 단어 "반사" 를 생각해보고 의미를 해석하면 이렇다.

JVM 이 동작할 때 클래스 로더가 바이트코드를 읽고 클래스의 정보를 메모리 영역에 할당하는 작업을 하게 되는데, 이 메모리 영역에 올라온 클래스 정보가 마치 사용자가 작성한 소스코드가 반사 되어 내부에 감춰진 속성들이 드러나게 되는 것이다.

이 의미를 이해한 뒤 리플렉션을 통해 얻는 이점을 살펴보자.

#### 리플렉션을 통해 얻을 수 있는 이점

1. 클래스의 명칭만 알고 있으면 클래스를 직접 조작할 수 있다.
2. 클래스의 속성, 메서드, 생성자 등에 접근이 가능하고 접근제어자를 통해 감춰둔 요소들을 강제로 접근할 수 있다.



#### 강력한 기능 만큼 강력한 단점도 존재한다

***

1. 리플렉션을 사용하면 JVM 컴파일 단계에서 발생하는 예외를 지원받지 못한 채, 런타임에서 발견하게 된다.
2. JVM 의 컴파일 시점에서 타입 검증을 통해 생성하는 방법과 달리 리플렉션은 런타임 중 클래스에 대한 분석이 필요하기 때문에 성능이 저하된다.
3. 객체지향 프로그래밍의 특성인 캡슐화와 추상화를 깨트려, 유지보수가 힘든 코드를 생성할 수 있다.



### 리플렉션 사용 해보기

***

#### 학습 테스트

{% tabs %}
{% tab title="Member" %}
```java
package jhlee.springrecipes.reflectiontest;

import java.util.Objects;

public class Member {

    private String name;

    public Member(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    private void setName(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object o) {
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Member member = (Member) o;
        return Objects.equals(name, member.name);
    }

    @Override
    public int hashCode() {
        return Objects.hashCode(name);
    }

    @Override
    public String toString() {
        return super.toString();
    }
}
```
{% endtab %}

{% tab title="Role" %}
```java
package jhlee.springrecipes.reflectiontest;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface Role {
    String value();

}
```
{% endtab %}

{% tab title="MemberTest" %}
```java
class MemberTest {

    @DisplayName("Member 객체를 리플렉션을 통해 생성자를 호출한다.")
    @Test
    void memberConstructorReflection()
            throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Class<?> clazz = Class.forName("jhlee.springrecipes.reflectiontest.Member");
        Constructor<?> constructor = clazz.getDeclaredConstructor(String.class);
        Object member = constructor.newInstance("Juhwan");

        assertThat(member).isEqualTo(new Member("Juhwan"));
    }

    @DisplayName("setter 메서드 없이 객체 속성 값을 변경할 수 있다.")
    @Test
    void fieldSet()
            throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException, NoSuchFieldException {
        // Arrange
        Class<?> clazz = Class.forName("jhlee.springrecipes.reflectiontest.Member");
        Constructor<?> constructor = clazz.getDeclaredConstructor(String.class);
        Object member = constructor.newInstance("Juhwan");
        Field name = clazz.getDeclaredField("name");
        name.setAccessible(true);

        // Act
        name.set(member, "Juhwan Lee");

        // Assert
        assertThat(member).isEqualTo(new Member("Juhwan Lee"));
    }

    @DisplayName("Member 의 private 접근 제어 메서드를 리플렉션을 통해 호출한다.")
    @Test
    void memberSetNameReflection()
            throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        // Arrange
        Class<?> clazz = Class.forName("jhlee.springrecipes.reflectiontest.Member");
        Constructor<?> constructor = clazz.getDeclaredConstructor(String.class);
        Object member = constructor.newInstance("Juhwan");
        Method setName = clazz.getDeclaredMethod("setName", String.class);
        setName.setAccessible(true);

        // Act
        setName.invoke(member, "Juhwan Lee");

        // Assert
        assertThat(member).isEqualTo(new Member("Juhwan Lee"));
    }

    @DisplayName("클래스의 어노테이션 정보를 가져올 수 있다.")
    @Test
    void annotation() throws ClassNotFoundException {
        // Arrange
        Class<?> clazz = Class.forName("jhlee.springrecipes.reflectiontest.Member");

        // Act
        boolean hasRoleAnnotation = clazz.isAnnotationPresent(Role.class);
        Role annotation = clazz.getAnnotation(Role.class);

        // Assert
        assertThat(hasRoleAnnotation).isTrue();
        assertThat(annotation.value()).isEqualTo("user");
    }

}
```
{% endtab %}
{% endtabs %}





### 스프링은 리플렉션을 어떻게 쓸까?

> #### 빈 객체 생성

아래 코드는 `SpringFramework` 의 `BeanUtils` 추상 클래스이며, 클래스를 인스턴스화 하는 메서드에서 리플렉션을 잔뜩 사용하고 있는것을 볼 수 있다.

```java
public abstract class BeanUtils {
	
	/**
	 * Convenience method to instantiate a class using the given constructor.
	 * <p>Note that this method tries to set the constructor accessible if given a
	 * non-accessible (that is, non-public) constructor, and supports Kotlin classes
	 * with optional parameters and default values.
	 * @param ctor the constructor to instantiate
	 * @param args the constructor arguments to apply (use {@code null} for an unspecified
	 * parameter, Kotlin optional parameters and Java primitive types are supported)
	 * @return the new instance
	 * @throws BeanInstantiationException if the bean cannot be instantiated
	 * @see Constructor#newInstance
	 */
 	public static <T> T instantiateClass(Constructor<T> ctor, @Nullable Object... args) throws BeanInstantiationException {
		Assert.notNull(ctor, "Constructor must not be null");
		try {
			ReflectionUtils.makeAccessible(ctor);
			if (KOTLIN_REFLECT_PRESENT && KotlinDetector.isKotlinType(ctor.getDeclaringClass())) {
				return KotlinDelegate.instantiateClass(ctor, args);
			}
			else {
				int parameterCount = ctor.getParameterCount();
				Assert.isTrue(args.length <= parameterCount, "Can't specify more arguments than constructor parameters");
				if (parameterCount == 0) {
					return ctor.newInstance();
				}
				Class<?>[] parameterTypes = ctor.getParameterTypes();
				@Nullable Object[] argsWithDefaultValues = new Object[args.length];
				for (int i = 0 ; i < args.length; i++) {
					if (args[i] == null) {
						Class<?> parameterType = parameterTypes[i];
						argsWithDefaultValues[i] = (parameterType.isPrimitive() ? DEFAULT_TYPE_VALUES.get(parameterType) : null);
					}
					else {
						argsWithDefaultValues[i] = args[i];
					}
				}
				return ctor.newInstance(argsWithDefaultValues);
			}
		}
		catch (InstantiationException ex) {
			throw new BeanInstantiationException(ctor, "Is it an abstract class?", ex);
		}
		catch (IllegalAccessException ex) {
			throw new BeanInstantiationException(ctor, "Is the constructor accessible?", ex);
		}
		catch (IllegalArgumentException ex) {
			throw new BeanInstantiationException(ctor, "Illegal arguments for constructor", ex);
		}
		catch (InvocationTargetException ex) {
			throw new BeanInstantiationException(ctor, "Constructor threw exception", ex.getTargetException());
		}
	}
}
```



> #### 의존성 주입

`Autowired` 어노테이션을 사용하고 있는 메타데이터를 찾거나 캐쉬에서 검색하여 반환 받은 `AutowiredElement` 를 상속 받은 객체의 `inject()` 메서드를 호출하는 과정이다.

<pre class="language-java"><code class="lang-java">public class AutowiredAnnotationBeanPostProcessor implements SmartInstantiationAwareBeanPostProcessor,
		MergedBeanDefinitionPostProcessor, BeanRegistrationAotProcessor, PriorityOrdered, BeanFactoryAware {
	
	/**
	 * &#x3C;em>Native&#x3C;/em> processing method for direct calls with an arbitrary target
	 * instance, resolving all of its fields and methods which are annotated with
	 * one of the configured 'autowired' annotation types.
	 * @param bean the target instance to process
	 * @throws BeanCreationException if autowiring failed
	 * @see #setAutowiredAnnotationTypes(Set)
	 **/
 	public void processInjection(Object bean) throws BeanCreationException {
		Class&#x3C;?> clazz = bean.getClass();
		InjectionMetadata metadata = findAutowiringMetadata(clazz.getName(), clazz, null);
		try {
<strong>			metadata.inject(bean, null, null);
</strong>		}
		catch (BeanCreationException ex) {
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					"Injection of autowired dependencies failed for class [" + clazz + "]", ex);
		}
	}
	
	private class AutowiredFieldElement extends AutowiredElement {

		private volatile boolean cached;

		private volatile @Nullable Object cachedFieldValue;

		public AutowiredFieldElement(Field field, boolean required) {
			super(field, null, required);
		}

		@Override
		protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
			Field field = (Field) this.member;
			Object value;
			if (this.cached) {
				try {
					value = resolveCachedArgument(beanName, this.cachedFieldValue);
				}
				catch (BeansException ex) {
					// Unexpected target bean mismatch for cached argument -> re-resolve
					this.cached = false;
					logger.debug("Failed to resolve cached argument", ex);
					value = resolveFieldValue(field, bean, beanName);
				}
			}
			else {
				value = resolveFieldValue(field, bean, beanName);
			}
			if (value != null) {
				ReflectionUtils.makeAccessible(field);
				field.set(bean, value);
			}
		}
}
</code></pre>







<details>

<summary>참고 자료</summary>

* [https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/BeanUtils.java](https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/BeanUtils.java)
* [https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.java#L526](https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.java#L526)
* [https://www.linkedin.com/pulse/java-reflection-api-how-spring-uses-shivangam-soni/](https://www.linkedin.com/pulse/java-reflection-api-how-spring-uses-shivangam-soni/)
* [https://blogs.oracle.com/javamagazine/java-reflection-introduction/](https://blogs.oracle.com/javamagazine/java-reflection-introduction/)

</details>

