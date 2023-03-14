# 반복되는 코드의 작성을 생략하는 방법 - 롬복

롬복(Lombok)은 데이터 클래스를 생성할 때 반복적으로 사용하는 메서드를 애노테이션으로 대체하는 기능을 제공하는 라이브러리이다.  
애노테이션 기반으로 코드를 자동으로 생성해주므로 반복되는 코드를 생략할 수 있다는 장점이 있다.

## 롬복 적용

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Member { //lombok 적용
    private String memberId;
    private int money;
}
```

```java
public class Member { //DeLombok
    private String memberId;
    private int money;

    public Member(String memberId, int money) {
        this.memberId = memberId;
        this.money = money;
    }

    public Member() {
    }

    public String getMemberId() {
        return this.memberId;
    }

    public int getMoney() {
        return this.money;
    }

    public void setMemberId(String memberId) {
        this.memberId = memberId;
    }

    public void setMoney(int money) {
        this.money = money;
    }

    public boolean equals(final Object o) {
        if (o == this) return true;
        if (!(o instanceof Member)) return false;
        final Member other = (Member) o;
        if (!other.canEqual((Object) this)) return false;
        final Object this$memberId = this.getMemberId();
        final Object other$memberId = other.getMemberId();
        if (this$memberId == null ? other$memberId != null : !this$memberId.equals(other$memberId)) return false;
        if (this.getMoney() != other.getMoney()) return false;
        return true;
    }

    protected boolean canEqual(final Object other) {
        return other instanceof Member;
    }

    public int hashCode() {
        final int PRIME = 59;
        int result = 1;
        final Object $memberId = this.getMemberId();
        result = result * PRIME + ($memberId == null ? 43 : $memberId.hashCode());
        result = result * PRIME + this.getMoney();
        return result;
    }

    public String toString() {
        return "Member(memberId=" + this.getMemberId() + ", money=" + this.getMoney() + ")";
    }
}
```

첫 번째 코드는 롬복을 적용한 것이고, 두 번째 코드는 롬복의 애노테이션을 코드로 풀어쓴 것이다. 한 눈에 봐도 코드의 길이가 상당히 차이나는 것을 확인해볼 수 있다.

## 롬복의 주요 애노테이션

롬복은 다양한 애노테이션을 제공하고 있는데, 그 중에서 많이 사용하는 애노테이션은 아래와 같다.


**@Getter, @Setter**

-   클래스에 선언되어 있는 필드에 대한 getter/setter 메서드를 자동 생성

**생성자 자동 생성 애노테이션**

-   `@NoArgsConstructor`: 매개변수가 없는 생성자를 자동 생성
-   `@AllArgsConstructor`: 모든 필드를 매개변수로 갖는 생성자를 자동 생성
-   `@RequiredArgsConstructor`: 필드 중 final이나 @NotNull이 설정된 변수를 매개변수로 갖는 생성자를 자동 생성

**@ToString**

-   `toString()` 메서드를 생성하는 애노테이션으로, 필드의 값을 문자열로 조합해서 리턴
-   숨겨야 할 정보는 `exclude` 속성을 사용해 특정 필드를 자동 생성에서 제외할 수 있음.

**@EqualsAndHashCode**

-   객체의 동등성(Equality)과 동일성(Identity)을 비교하는 연산 메서드 생성
-   `equals`는 두 객체의 내용이 같은지 동등성을 비교, `hashcode`는 두 객체가 같은 객체인지 동일성을 비교
-   부모 클래스를 상속받아 부모 클래스의 필드까지 비교해야 한다면, `callSuper` 속성을 true로 설정(기본 값은 false)

**@Data**

-   `@Getter/@Setter`, `@RequiredArgsConstructor`, `@ToString`, `@EqualsAndHashCode`를 모두 포괄하는 애노테이션
