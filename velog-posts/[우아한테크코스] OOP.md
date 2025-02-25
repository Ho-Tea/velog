<blockquote>
<p><strong>우아한테크코스 6기 최종 코딩테스트</strong>를 준비하면서 작성된 글의 일부로,
해당 글은 <strong>오로지 최종 코딩테스트</strong>에만 대비하기 위해 작성된 글입니다.</p>
</blockquote>
<p>아래의 우아한테크코스의 프리코스 과제를 수행해 오면서 정리한 내용들로 이루어져 있습니다.</p>
<ul>
<li><code>oncall</code> - 최종 코딩 테스트!</li>
<li><code>subway-path</code></li>
<li><code>pairmatching-precourse</code></li>
<li><code>bridge</code></li>
<li><code>baseball</code></li>
<li><code>menu</code></li>
<li><code>christmas</code></li>
<li><code>lotto</code></li>
<li><code>racingcar</code></li>
<li><code>vendingmachine</code></li>
<li><code>onboarding</code></li>
</ul>
<hr />
<h1 id="oop">OOP</h1>
<h2 id="현실에-객체지향-대입">현실에 객체지향 대입</h2>
<ul>
<li><p><strong>객체는 항상 생성부터 소멸 시점까지 유효한 데이터를 가져야 한다.</strong>
키가 <code>99999</code> 인 사람은 존재하지 않기 때문에 비현실적이다.
이를 막기 위해서 생성 시부터 제약을 가해야 한다.</p>
<blockquote>
<p><strong>객체 지향에서의 객체는 생명체던 비생명체던 모두 자아를 가지고 스스로 동작할 수 있는 존재라고 생각하자.</strong></p>
</blockquote>
</li>
<li><p><strong>추상화</strong></p>
<ul>
<li><strong>현실에 존재하는 사물이 가진 속성 중에서 필요한 것만 추출하여 코드로 옮기는 것</strong></li>
<li><strong>객체를 의인화 하고 동작(메소드)을 추상화하여 가독성을 크게 높일 수 있다.</strong></li>
</ul>
</li>
<li><p><strong>캡슐화</strong></p>
<ul>
<li><strong>속성과 동작을 클래스로 묶어서 만드는 것이 바로 캡슐화다.</strong></li>
<li><strong>추상화</strong>를 통해 현실에 존재하는 사물의 속성과 동작 중에서 <strong>필요한 것만 추출했고,</strong>
이를 <strong>클래스로 옮기면 추상화와 캡슐화를 한 것이다.</strong></li>
</ul>
</li>
<li><p><strong>상속, 다형성</strong></p>
</li>
</ul>
<blockquote>
<p><strong>처음부터 객체지향을 준수하여 프로그래밍하기는 어려운것이 사실이다.</strong>
따라서 <code>소트웍스 앤솔러지</code>라는 책에서 제시하는 객체지향 프로그래밍을 잘 하기 위한 9가지 원칙을 먼저 정리하므로써 객체지향에 대한 기본 틀과 구성 방식에 대해 알아보자.</p>
</blockquote>
<hr />
<h2 id="객체지향-생활-체조-원칙">객체지향 생활 체조 원칙</h2>
<h3 id="1-한-함수메서드에-최소한의-들여쓰기indent만-허용했는가-br-최대-depth--2까지만-허용">1. 한 함수(메서드)에 최소한의 들여쓰기(indent)만 허용했는가? <br /> (최대 depth : 2까지만 허용)</h3>
<ul>
<li>들여쓰기를 줄이는 가장 좋은 방법은 함수를 분리하는 것이다.</li>
</ul>
<h3 id="2-else-예약어를-사용하지-않았는가">2. else 예약어를 사용하지 않았는가?</h3>
<ul>
<li><code>else</code>와 같이 조건없이 모든 경우를 열어주는 코드는 큰 버그를 초래할 수 있기 때문에 지양하는 것이 좋다.</li>
</ul>
<h3 id="3-모든-원시값과-문자열을-포장했는가">3. 모든 원시값과 문자열을 포장했는가?</h3>
<ul>
<li><p>변수를 선언하는 방법에는 두가지가 존재하는데, <strong>객체로 포장해라</strong>
(<code>Collection</code>으로 선언한 변수도 포장한다.) -&gt; <code>일급 컬렉션</code></p>
<pre><code class="language-java">int age = 20; // 원시타입의 변수
Age age = new Age(20) // 원시 타입의 변수를 객체로 포장한 변수</code></pre>
</li>
<li><p>아래의 경우를 살펴보면, <code>User</code> 클래스의 필드는 단 2개만 존재하지만,
해당 <code>User</code>클래스가 해야할 일은 굉장히 많다</p>
<pre><code class="language-java">public class User {
  private String name;
  private int age;

  public User(String nameValue, String ageValue) {
      int age = Integer.parseInt(ageValue);
      validateAge(age);
      validateName(nameValue);
      this.name = nameValue;
      this.age = age;
  }

  private void validateName(String name) {
      if (name.length() &lt; 2) {
          throw new RuntimeException(&quot;이름은 두 글자 이상이어야 합니다.&quot;);
      }
  }

  private void validateAge(int age) {
      if (age &lt; 0) {
          throw new RuntimeException(&quot;나이는 0살부터 시작합니다.&quot;);
      }
  }
}</code></pre>
</li>
<li><p>아래와 같이 원시타입 객체를 포장하여 <code>User</code>가 해야할 일을 덜어 줄 수있다</p>
<pre><code class="language-java">public class User {
  private Name name;
  private Age age;

  public User(String name, String age) {
      this.name = new Name(name);
      this.age = new Age(age);
  }
}
</code></pre>
</li>
</ul>
<p>public class Name {
    private String name;</p>
<pre><code>public Name(String name) {
    if (name.length() &lt; 2) {
        throw new RuntimeException(&quot;이름은 두 글자 이상이어야 합니다.&quot;);
    }
    this.name = name;
}</code></pre><p>}</p>
<p>public class Age() {
    private int age;</p>
<pre><code>public Age(String input) {
    int age = Integer.parseInt(input);
    if(age &lt; 0) {
        throw new RuntimeException(&quot;나이는 0살부터 시작합니다.&quot;);
    }
}</code></pre><p>}</p>
<pre><code>이 말은 박싱된 기본타입을 쓰라는 말이랑은 다르다!
`(EffectiveJava: 박싱된 기본 타입보다는 기본 타입을 사용하라)`

위의 예시를 들어 설명하자면, `Age`부분에서 원시타입 객체를 쓴것을 볼 수 있다.
거의 모든 경우에 박싱된 기본타입(`Integer`)과 같은 경우 보다는 기본 타입(`int`)을 사용해야 하고, 무조건 박싱된 기본타입을 써야하는 경우는 아래와 같다.
&gt;**1. 제네릭**
**2. 리플렉션**
**3. DTO (null을 허용하기 때문에)**

정리하자면,
하나의 객체 내부에 필드가 존재하는데, **필드 하나가 아닌 필드가 2개이상인 경우에**
해당 필드에 **유효성검증과 같은 검증 로직**이 들어가야 한다면,
해당 필드를 원시타입으로 유지하지 말고, **포장하자!**

### 4. 컬렉션에 대해 일급 컬렉션을 적용했는가?
- **일반적인 컬렉션**
``` java
Map&lt;String, String&gt; map = new HashMap&lt;&gt;();
map.put(&quot;1&quot;, &quot;A&quot;);
map.put(&quot;2&quot;, &quot;B&quot;);
map.put(&quot;3&quot;, &quot;C&quot;);</code></pre><ul>
<li><p><strong>일급 컬렉션</strong> (아래와 같이 <code>Wrapping</code>하는 것) → 하나의 자료구조가 된다.</p>
</li>
<li><p><code>Collection</code>을 <code>Wrapping</code>하면서, 그 외 다른 멤버변수가 없는 상태를 <strong>일급컬렉션</strong>이라 한다.</p>
<pre><code class="language-java">public class GameRanking {
  private Map&lt;String, String&gt; ranks;

  public GameRanking(Map&lt;String, String&gt; ranks) {
      this.ranks = ranks;
  }
}</code></pre>
</li>
</ul>
<h3 id="5-3개-이상의-인스턴스-변수를-가진-클래스를-구현하지-않았는가">5. 3개 이상의 인스턴스 변수를 가진 클래스를 구현하지 않았는가?</h3>
<ul>
<li>인스턴스 변수가 많아질수록 클래스의 응집도는 낮아진다</li>
<li><em>(응집도는 높을수록 좋다)*</em></li>
<li>마틴 파울러는 <strong>대부분의 클래스가 인스턴스 변수 하나만으로 일</strong>을 하는 것이 적합하다고 말했다</li>
<li><strong>따라서, 최대한 클래스를 많이 분리하게끔 강제하여 높은 응집도를 유지하자!</strong></li>
</ul>
<pre><code class="language-java">public class Car {
    private String brand;
    private String model;  // 인스턴스 변수 2개

    public Car(String brand, String model) {
        this.brand = brand;
        this.model = model;
    }

    public String getBrand() {
        return brand;
    }

    public String getModel() {
        return model;
    }
}

public class Engine {};
public class Light {};
public class Wiper {};
public class Brake {};
// ..</code></pre>
<ul>
<li><code>CleanCode</code>에서는 인스턴스 변수 뿐 아니라 메서드의 가장 이상적인 파라미터 개수는 0개라고 한다.</li>
<li><em>즉, 인스턴스 필드나 메서드 파라미터를 최대한 적게 유지해서 응집도를 높여야 한다!*</em></li>
</ul>
<h3 id="6-핵심-로직을-구현하는-도메인-객체에-gettersetter를-사용하지-않고-구현했는가-단-dto는-허용한다">6. 핵심 로직을 구현하는 도메인 객체에 getter/setter를 사용하지 않고 구현했는가? (단, DTO는 허용한다!)</h3>
<ul>
<li><p><code>getter</code>로 객체 내부의 상태를 꺼내와 외부에서 상태를 바꾸는 것은, <strong>객체의 상태값을 바꾼다는 판단</strong>을 외부에 위임한 것이다!</p>
</li>
<li><p><em>객체의 상태값을 바꾼다는 판단을 외부에 맡기지 말자! 
이는 곧 <code>독립적인 객체 설계</code>에 위배되는 행위이다.*</em></p>
</li>
<li><p><strong>즉, 객체의 상태가 변경되는 것은 객체 스스로의 행동에 의해야 한다.</strong>
자율적인 객체가 되고 외부의 영향을 받지 않음으로써 느슨한 결합과 유연한 협력을 이룰 수 있는 것이다.</p>
</li>
<li><p><code>getter</code>와 <code>setter</code>는 자신의 상태 정보를 외부에 노출하는 격이 되고 이것은 외부의 영향으로 상태 정보가 변할 수 있는 가능성을 열어두게된다.</p>
</li>
<li><p><strong>따라서, <code>getter</code>/<code>setter</code>의 사용은 지양하자</strong></p>
<ul>
<li><p>데이터의 이동은 <strong>DTO</strong>를 이용하자!</p>
</li>
<li><p><code>View</code>에서 단지 출력용도로 사용하기 위해서는 <strong><code>getter</code></strong>를 쓸 수도 있다.</p>
<ul>
<li>이때 <strong>방어적 복사</strong>를 통해 외부에서의 데이터 변경이 내부까지 영향이 가지 않도록 구성하자!</li>
</ul>
</li>
</ul>
</li>
</ul>
<pre><code class="language-java">public class DefensiveCopyingExample {
    private List&lt;String&gt; internalList;

    public DefensiveCopyingExample(List&lt;String&gt; originalList) {
        // 방어적 복사를 통해 원본 리스트를 보호
        this.internalList = new ArrayList&lt;&gt;(originalList);
    }

    public List&lt;String&gt; getInternalList() {
        // 방어적 복사를 통해 내부 리스트를 외부로 노출하지 않음
        return new ArrayList&lt;&gt;(internalList);
    }

    public static void main(String[] args) {
        List&lt;String&gt; originalList = new ArrayList&lt;&gt;();
        originalList.add(&quot;Item 1&quot;);
        originalList.add(&quot;Item 2&quot;);

        DefensiveCopyingExample example = new DefensiveCopyingExample(originalList);

        // 외부에서 원본 리스트에 접근
        List&lt;String&gt; externalList = example.getInternalList();

        // 외부에서 리스트에 아이템 추가
        externalList.add(&quot;Item 3&quot;);

        // 원본 리스트에 변화 없음을 확인
        System.out.println(&quot;Original List: &quot; + originalList);  // [Item 1, Item 2]
        System.out.println(&quot;External List: &quot; + externalList);  // [Item 1, Item 2, Item 3]
    }
}    </code></pre>
<h3 id="7-코드-한줄에-점을-하나만-허용했는가">7. 코드 한줄에 점(.)을 하나만 허용했는가?</h3>
<ul>
<li><p><code>단순히 라인에 존재하는 점의 개수를 수치적으로 줄이라는 의미</code>보다는
<code>필드</code>나 <code>메서드</code>를 통해 <code>인스턴스에 접근하는 방식 자체</code>를 재고해보라는 뜻이다.</p>
</li>
<li><p><strong>점의 개수가 많다는 것은 대상 객체의 내부에 깊이 접근하겠다는 의도를 나타낸다.</strong></p>
</li>
<li><p>이는 일반적으로 <strong>호출자</strong>와 <strong>피호출자</strong> 사이의 <strong><code>강한 결합도</code></strong>를 바탕으로 메서드의 응집력을 떨어뜨리고 있을 확률이 높기 때문이다.</p>
<blockquote>
<p>🧨 <strong>디미터의 법칙(&quot;친구하고만 대화하라&quot;)</strong>
자기 소유의 장난감, 자기가 만든 장난감, 그리고 누군가 자기에게 준 장난감하고만 놀 수 있다.</p>
</blockquote>
</li>
<li><p><em>절대 장난감의 장난감과 놀면 안된다.*</em> <br /></p>
</li>
<li><p><em>즉, 객체 간의 관계에서 이웃하지 않는 낯선 객체와 메세지를 보내는 설계는 피하라는 것이다.*</em></p>
</li>
<li><p>물론 가독성 측면에서도 문제가 있다.</p>
<pre><code class="language-java">String result = someObject.getA().getB().getC().calculate();</code></pre>
<pre><code class="language-java">A a = someObject.getA();
B b = a.getB();
C c = b.getC();
String result = c.calculate();</code></pre>
</li>
</ul>
<p><code>점의 개수가 많다면</code>
<strong>1. 대상 객체의 내부에 깊이 접근한것은 아닌지,</strong>
<strong>2. 디미터의 법칙을 위배한 것은 아닌지 경계하고,</strong>
<strong>3. 가독성 측면에서도 문제가 있으니 수정을 요한다!</strong></p>
<h3 id="8-메서드의-인자-수를-3개-이하로-제한했는가">8. 메서드의 인자 수를 3개 이하로 제한했는가?</h3>
<p><strong>(3개를 초과하는 인자는 허용하지 않으며, 3개도 가능하면 줄이기 위해 노력해 본다)</strong></p>
<ul>
<li>메서드에 많은 인자가 있다는 것은 해당 메서드의 <strong>역할</strong>이 많다는 것으로도 해석된다.</li>
</ul>
<h3 id="9-메서드가-한가지-일만-담당하도록-구현했는가">9. 메서드가 한가지 일만 담당하도록 구현했는가?</h3>
<p>하나의 메서드가 <strong>여러 책임을 담당</strong>한다면 코드의 길이가 늘어나게 되며,
다른 개발자가 <strong>해당 메서드의 역할을 이해하기 어렵게 된다.</strong> 
(메서드 이름도 애매해진다)</p>
<h3 id="10-클래스를-작게-유지하기-위해-노력했는가">10. 클래스를 작게 유지하기 위해 노력했는가?</h3>
<p><strong>(메서드당 line을 10까지만 허용하며 길이가 길어지면 <code>메서드로 분리</code>시킨다.)</strong></p>
<ul>
<li><strong>하나의 목적을 염두하고 설계하라는 의미이다.</strong>
50줄 이상의 객체는 한 가지 이상의 일을 하고 있을 확률이 높다.</li>
<li><em>그러니 작게 유지해야 한다.*</em></li>
</ul>
<h3 id="11-매직-리터럴매직-넘버-사용을-자제하고-상수를-사용하자">11. 매직 리터럴/매직 넘버 사용을 자제하고 상수를 사용하자</h3>
<ul>
<li>프로그래밍에서 <code>상수 (static final)</code>로 선언하지 않은 <code>숫자를 
매직 넘버, 문자열을 매직 리터럴</code>이라 한다.</li>
<li><em>이를 정적(static)이고 변경 불가능(final)한 상수로 선언하여 사용하자.*</em></li>
<li>코드에서 상수로 선언되어 있지 않은 숫자, 문자열은 무엇을 의미하는지 확신할 수 없다.
이를 상수로 선언하게 됨으로써 불분명한 값들은 이름을 가지게 된다.
이름을 가지게 된 값은 <strong>그 이름만으로도 어떠한 역할을 하는지 알 수 있게 된다.</strong></li>
<li><strong>하지만 숫자 1을 <code>ONE</code>으로 이름을 짓는 것과 같은 의미없은 상수 변환은 피하도록 하자.</strong></li>
</ul>