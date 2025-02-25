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
<blockquote>
<p><strong>구현 전략</strong>
<code>Docs</code>, <code>Feat</code>, <code>Test</code> 모두 한번에 <code>Commit</code> 하는 방식으로 진행하였다.
(= 기능단위로 커밋)</p>
</blockquote>
<hr />
<h2 id="unit-test">Unit Test</h2>
<blockquote>
<p><strong>도메인의 비즈니스로직을 단위테스트하는 것을 중심으로 작성되었다.</strong></p>
</blockquote>
<p>우선 <code>AssertJ</code>를 <code>import</code>한다.</p>
<pre><code class="language-java">import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.assertj.core.api.Assertions.assertThat;</code></pre>
<hr />
<h3 id="선택한-test-code-format">선택한 Test Code Format</h3>
<pre><code class="language-java">@Test
@DisplayName(&quot;1에서 9까지의 숫자가 아니라면 예외가 발생한다.&quot;)
void validateRange() { // 사용되어 지는 메서드 명
        assertThatThrownBy(() -&gt; new GameNumber(List.of(0, 2, 3)))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessageContaining(&quot;숫자는 1에서 9까지의 수로 이루어져야 합니다.&quot;);

}</code></pre>
<hr />
<h3 id="테스트-커버리지-확인하는-방법"><a href="https://velog.io/@pgmjun/Java-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80-%ED%99%95%EC%9D%B8%EB%B2%95">테스트 커버리지 확인하는 방법</a></h3>
<hr />
<h3 id="테스트-라이브러리-활용">테스트 라이브러리 활용</h3>
<p><a href="https://jaehoney.tistory.com/210"><code>@ParameterizedTest</code></a> 을 <code>@Test</code> 애노테이션 대신 사용하면 여러 개의 파라미터 값에 대해 각각 테스트를 수행하는 코드를 간편하게 작성할 수 있다.</p>
<ul>
<li><p><code>@ValueSource(strings = {&quot;&quot;, &quot;  &quot;, &quot;  &quot;})</code></p>
</li>
<li><p><code>@CsvSource(value = {&quot;1:2&quot;, &quot;2:4&quot;, &quot;3:6&quot;}, delimiter = ':')</code></p>
</li>
<li><p><code>@NullAndEmptySource</code></p>
</li>
<li><p><code>@EnumSource(Week.class)</code></p>
</li>
<li><p><code>@MethodSource(&quot;paramsForIsBlank&quot;)</code></p>
<pre><code class="language-java">  @ParameterizedTest
  @MethodSource(&quot;matchData&quot;)
  @DisplayName(&quot;다른 숫자와 비교해 같은 자리에 같은 수가 몇개 있는지 알 수 있다.&quot;)
  void matchCount(BaseballNumber computerNumber, BaseballNumber userNumber, long expected){
      assertThat(computerNumber.matchCount(userNumber)).isEqualTo(expected);
  }

  static Stream&lt;Arguments&gt; matchData() {
      BaseballNumber computerNumber = new BaseballNumber(List.of(4, 2, 3));
      return Stream.of(
              Arguments.of(computerNumber, new BaseballNumber(List.of(4, 2, 3)), 3L),
              Arguments.of(computerNumber, new BaseballNumber(List.of(1, 2, 3)), 2L),
              Arguments.of(computerNumber, new BaseballNumber(List.of(4, 3, 2)), 1L),
              Arguments.of(computerNumber, new BaseballNumber(List.of(3, 4, 5)), 0L)
      );
  }</code></pre>
</li>
</ul>
<p><a href="https://www.baeldung.com/assertj-exception-assertion"><code>AssertJ Exception Assertions</code></a></p>
<ul>
<li><code>assertThatIllegalArgumentException()</code>와 같이 예외를 특정해서 테스트할 수 있는 장점이 있지만, 아래와 같이 예외 처리하는 것이 더 간편한 것 같다.<pre><code class="language-java">assertThatThrownBy(() -&gt; new OrderSheets(orderDuplicate))
              .isInstanceOf(IllegalArgumentException.class)
              .hasMessageContaining(&quot;[ERROR] 유효하지 않은 주문입니다. 다시 입력해 주세요.&quot;);</code></pre>
</li>
</ul>
<p><a href="https://www.baeldung.com/junit5-assertall-vs-multiple-assertions"><code>assertAll</code></a></p>
<pre><code class="language-java">@DisplayName(&quot;특정 메뉴 그룹에 속하는 메뉴가 몇 개 있는지 알 수 있다.&quot;)
    @Test
    void getNumberOf() {
        OrderSheets orderSheets = new OrderSheets(List.of(&quot;초코케이크-2&quot;, &quot;제로콜라-1&quot;, &quot;시저샐러드-1&quot;));
        Assertions.assertAll(
                () -&gt; assertThat(orderSheets.getNumberOf(MenuGroup.DESSERT)).isEqualTo(2),
                () -&gt; assertThat(orderSheets.getNumberOf(MenuGroup.APPETIZER)).isEqualTo(1),
                () -&gt; assertThat(orderSheets.getNumberOf(MenuGroup.BEVERAGE)).isEqualTo(1)
        );
    }
</code></pre>
<hr />
<h3 id="테스트-코드-명-작성법">테스트 코드 명 작성법</h3>
<ul>
<li>테스트 코드 명을 작성할 때 <code>&quot;스트라이크 테스트&quot;</code> 보다는 <code>&quot;같은 자리에 같은 숫자가 존재하면, 스트라이크이다.&quot;</code>처럼 명사 나열보다는 <strong>문장 형식</strong>이 낫다.<ul>
<li><del>메서드 명을 한글로 쓰고, 상단에 <code>**@DisplayNameGeneration**</code> 라는 애너테이션을 쓰면, 매번 메서드마다 <code>@DisplayName</code> 을 사용하지 않아도 된다</del>  </li>
<li><em>→ 가독성을 더 헤치는 느낌을 받아 <code>@DisplayNameGeneration</code>을 사용하지 않기로 한다.*</em></li>
</ul>
</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/cd312cee-d72d-4a6b-8fdb-486285a78e0e/image.png" /></p>
<hr />
<h3 id="메서드-시그니처를-수정하여-테스트하기-좋은-메서드로-만들기"><a href="https://tecoble.techcourse.co.kr/post/2020-05-07-appropriate_method_for_test_by_parameter/">메서드 시그니처를 수정하여 테스트하기 좋은 메서드로 만들기</a></h3>
<pre><code class="language-java">// 테스트하기 어려운 메서드
public void move() {
        final int number = random.nextInt(RANDOM_NUMBER_UPPER_BOUND);

        if (number &gt;= MOVABLE_LOWER_BOUND) {
            position++;
        }
 }

// 테스트하기 좋은 메서드
public void move(int number) {
    if (number &gt;= MOVABLE_LOWER_BOUND) {
        position++;
    }
}</code></pre>
<hr />
<h3 id="getter-와-같은-단순한-읽기-기능의-목적인-메서드들은-단위테스트에-의미가-없다">Getter 와 같은 단순한 읽기 기능의 목적인 메서드들은 단위테스트에 의미가 없다.</h3>
<pre><code class="language-java">public class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

@Test
public void testGetName() {
    // Given
    Person person = new Person(&quot;John&quot;);

    // When
    String name = person.getName();

    // Then
    assertEquals(&quot;John&quot;, name);
}

// 의미가 없다!</code></pre>
<p>하지만, 계산된 값이 있는 경우나 로직이 내장된 경우와 같이 <code>Getter</code>가 내부적으로 어떤 로직을 수행하거나 특별한 경우를 처리하는 등의 추가적인 로직이 있는 경우에는 해당 로직에 대한 테스트를 추가하는 것이 좋다.</p>
<pre><code class="language-java">public class Person {
    private String name;
        private int age;

    public Person(String name, int age) {
        this.name = name;
                this.age = age;
    }

    public String getName() {
        if (age &lt; 18) {
            return &quot;Child &quot; + name;
        } else {
            return name;
        }
    }

@Test
public void testGetName() {
    // Given
    Person childPerson = new Person(&quot;Alice&quot;, 15);
    Person adultPerson = new Person(&quot;Bob&quot;, 25);

    // When
    String childName = childPerson.getName();
    String adultName = adultPerson.getName();

    // Then
    assertEquals(&quot;Child Alice&quot;, childName);
    assertEquals(&quot;Bob&quot;, adultName);
}</code></pre>
<p><strong>이경우에는 솔직히 <code>Getter</code>라는 이름보다는 다른 이름을 쓰는 것이 나은 판단 같다.</strong></p>
<p><strong>Q. 어떻게 캡슐화를 코드에 스며들게 할까?</strong>
<strong>A.</strong> 객체를 만들고 객체 안에서 메서드( 기능 )을 구현할 때 이름을 <strong>추상적이게 명명해야 한다.</strong></p>
<p><strong>Q. 그럼 추상적이게 명명한다는 것이 무엇일까?</strong>
<strong>A1.</strong> 먼저 내가 이 메서드를 왜 사용해야 하는지를 먼저 생각해보자
<code>Ex) 성인 콘텐츠와 미성년자 콘텐츠를 구분하여야 한다.</code>
<strong>A2.</strong> 은닉화 되어있는 <code>age</code>의 <code>getter</code>를 내가 가져와야 하는 이유로 명명하자.
<code>Ex) getAge → checkAdult</code>
<strong>A3.</strong> 위와 같이 작성했을 경우 외부에서 이 메서드를 사용하는 사용자는 <code>'성인 여부를 확인하는 메서드이구나'</code>라고 생각하고 사용하게 될 것이다. </p>
<blockquote>
<p><strong>그렇게 된다면,
이 안에서 어떤 속성을 썼는지 내부 로직을 예측할 수 없게 된다. -&gt; <code>추상적</code></strong> 
<strong>이렇게 사용자가 기능만 알고 사용하게 된다면 캡슐화는 잘 적용된 것일 것이다.</strong></p>
</blockquote>
<blockquote>
<p>메서드의 이름이 추상적이다 라는 것은 약간 오해의 소지가 있지만,
<strong><code>”추상적이다 = 구체적이지 않게 작성하라는 뜻”</code></strong>이 아니다
 애매하게 쓰이는 이름보다는 구체적인게 오히려 더 낫다
 <code>getAge</code>를 통해 나이를 알아오는 메서드이름을 추상화해서 표현하여
 <strong>사용자가 내부의 특정 속성, 특정 로직을 예측하지 못하게 구성하고,</strong>
 사용자가 어떤 기능인지만 알고 수행하게끔 <code>checkAdult</code>와 같이 추상화해서 표현하라는 뜻이다.</p>
</blockquote>
<hr />
<h3 id="검증된-메서드를-활용하는-메서드에-대한-테스트">검증된 메서드를 활용하는 메서드에 대한 테스트</h3>
<p> 내부적으로 검증된 메서드들을 사용하더라도 <strong>메서드 시그니처</strong>가 검증된 <strong>메서드들의 시그니처</strong>와 다를 시 테스트 하자.
아래의 경우 <code>finish()</code>에 대한 테스트는 굳이 진행하지 않아도 되겠다.</p>
<pre><code class="language-java">// BridgeGame
public boolean finish() {
        return bridge.end();
}
// Bridge
public boolean end() {
        return unit.size() == index;
}</code></pre>
<pre><code class="language-java">/**
 * @param size 다리의 길이
 * @return 입력받은 길이에 해당하는 다리 모양. 위 칸이면 &quot;U&quot;, 아래 칸이면 &quot;D&quot;로 표현해야 한다.
     */
 public List&lt;String&gt; makeBridge(int size) {
    validateSize(size);
    List&lt;String&gt; bridge = new ArrayList&lt;&gt;();
    for(int i = 0; i &lt; size; i++){
        bridge.add(BridgeUnit.of(bridgeNumberGenerator.generate()).getSignatureLetter());
    }
    return bridge;
}
</code></pre>
<p>위와 같이 이루어져 있는 경우에 <code>makeBridge(int size)</code>로 생성되는 <code>List&lt;String&gt;</code>형의 문자들은 정확히 어떤 문자들이 들어가는지 예측할 수가 없다.
<code>(내부적으로 랜덤한 값을 생성해 문자로 변환하기 때문에)</code></p>
<p>하지만, 랜덤한 값을 생성하는 <code>bridgeNumberGenerator</code>가 정상적으로 동작하는지에 대해 검증했고, <code>BridgeUnit.of()</code> 또한 정상작동하는 것을 검증했다면,
정확히 <code>List&lt;String&gt;</code>에 어떤 순서로 값이 들어가는 지는 별로 중요하지 않고,</p>
<p>특정 문자 <code>U</code>와 <code>D</code>만 정상적으로 들어갔는지 검사하는 것으로 테스트를 할 수 있겠다.</p>
<pre><code class="language-java">@Test
@DisplayName(&quot;무작위 값을 이용해 다리를 생성할 수 있다.&quot;)
void makeBridge() {
    BridgeMaker bridgeMaker = new BridgeMaker(new BridgeRandomNumberGenerator());
    assertAll(
            () -&gt; assertThat(bridgeMaker.makeBridge(3).size()).isEqualTo(3),
            () -&gt; assertThat(bridgeMaker.makeBridge(3)).containsAnyElementsOf(List.of(CrossingDirection.TOP.getSignatureLetter(), CrossingDirection.BOTTOM.getSignatureLetter()))
    );
}</code></pre>
<hr />
<h3 id="입력-예외-검증-테스트">입력 예외 검증 테스트</h3>
<p>아래와 같이 입력에 대한 예외를 검증하는 부분에 대해서는 테스트 코드를 작성하지 않는다.</p>
<pre><code class="language-java">      private void validateNullAndEmpty(String input) {
        if (Objects.isNull(input) || input.isEmpty()) {
            throw new IllegalArgumentException(&quot;null 이거나 길이가 없는 문자열 입니다.&quot;);
        }
    }

    private void validateNumeric(String input) {
        if (!NUMERIC_PATTERN.matcher(input).matches()) {
            throw new IllegalArgumentException(&quot;문자열이 숫자 1부터 9까지로 이루어져 있지 않습니다.&quot;);
        }
    }
    private void validateSingleLetter(String input) {
        if (input.length() != 1) {
            throw new IllegalArgumentException(&quot;문자열의 크기는 한개로 이루어져야 합니다.&quot;);
        }
    }</code></pre>
<p><code>InputView</code>에서 아래 부분의 <code>throw~</code> 로 예외처리되는 부분이 검증이 되지않아 테스트를 진행하지 않아 테스트 코드 커버리지가 떨어지는 걸 볼 수 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/9fb488ce-fdd0-4fce-b43d-ea23f135d261/image.png" /></p>
<p><code>util</code>로 따로 생성해 테스트를 해줄까도 생각해 보았지만 그렇게 하지 않았다.
테스트 코드를 꼼꼼히 작성해 테스트 코드 커버리지가 높아 지는 것은 좋은 일이지만,
<strong>코드 커버리지를 높이기 위한 테스트코드 작성은 주객이 전도된 것이다.</strong></p>
<p>테스트는 결함 검출용으로 사용하고, <code>Code Coverage</code>에는 집착하지 마라
결함 검출을 어느 수준까지 할것인지는 본인의 판단이며,
위와 같은 상황에서 아래와 같다면
<code>input 예외처리 테스트 코드 작성의 비용</code> &gt; <code>입력 검증 Test 결함으로 생기는 SideEffect</code>
테스트 코드를 굳이 작성하지 않고, <code>input</code> 검증에 관한 부분은 결함이 없다고 생각하고 진행하는 것이 맞다</p>
<blockquote>
<p>입력 검증 <code>Test</code> 결함으로 생기는 <code>SideEffect</code>는 거의 없다고 생각되며, 비즈니스 로직과 연관되는 예외처리도 아니므로 그렇게 <code>Critical</code>한 예외 상황도 없을 것이다.</p>
</blockquote>
<hr />
<h3 id="주어진-라이브러리에-대한-테스트">주어진 라이브러리에 대한 테스트</h3>
<p>라이브러리로 주어지는 <code>import camp.nextstep.edu.missionutils.Randoms;</code>과 같은 것을
테스트해야 하는지, 또 테스트를 해야한다면 어떻게 구성해야하는지에 대해 알아보자</p>
<p><strong>테스트를 해야 하는가? -&gt; ⭕️</strong>
<code>숫자를 몇개를 만드는지</code>, <code>범위는 어디서부터 어디까지인지</code> 외부에서 모르기 때문에 <code>generate</code>메서드에 대한 테스트를 만들어야 한다.
또한 <code>Util</code>성을 가진 클래스라도 테스트는 해야하는 것이 맞다.</p>
<pre><code class="language-java">public class RandomNumGenerator {
    public static List&lt;Integer&gt; generate(){
        List&lt;Integer&gt; numbers = new ArrayList&lt;&gt;();
        while (numbers.size() &lt; 3) {
            int randomNumber = Randoms.pickNumberInRange(1, 9);
            if (!numbers.contains(randomNumber)) {
                numbers.add(randomNumber);
            }
        }
        return numbers;
    }
}</code></pre>
<p><strong>그렇다면 어떻게 테스트를 해야하는가?</strong>
<code>Randoms.picikNumberInRange</code>의 경우 내부적으로 어떤 구현이 이루어졌는지 메서드 명만으로는 알 수 없으며, 구현내부를 알더라도 관련된 메서드는 <code>private</code>으로 접근제한을 걸어놓아 항상 <code>1부터 9사이의 수</code>를 반환하는지 검증하기 어렵고, 테스트하기 힘들다.</p>
<p>덧붙이자면, 만약 구현 내부(<code>pickNumberInRange() 내부</code>)의 메서드들을 모두 검증할 수 있다면 <code>항상 1부터 9사이의 수를 반환하는 것</code>을 입증할 수 있지만,
현재는 그렇게 하지 못하므로 <code>항상 1부터 9사이의 수를 반환</code>하는지 검증하기 어렵다는 것이다.</p>
<p><strong>따라서, 충분히 큰 수만큼 테스트를 돌렸을 때 정상적으로 동작한다는 것은 메서드가 정상적으로 동작한다는 것으로 생각하자!</strong></p>
<p>(라이브러리가 정상적으로 동작하는 지에 대한 검증이 필요없을 수 있지만,
해당 라이브러리는 내가 직접 구현한 것이 아니므로 추가적인 검증을 진행한 것이다)</p>
<p><strong>테스트를 10000번 돌려도 되고(<code>라이브러리에 대한 추가 검증</code>),
1번만 돌려도(<code>&quot;라이브러리에 대한 검증&quot;이 됐다라고 판단한 후 진행하는 것</code>) 된다.</strong></p>
<pre><code class="language-java">class RandomNumGeneratorTest {

    public static final int ENOUGH_BIG_NUMBER = 10000;

    @Test
    @DisplayName(&quot;1에서 9까지 서로 다른 임의의 수 3개를 생성한다.&quot;)
    void generate() {
        for (int i = 0; i &lt; ENOUGH_BIG_NUMBER; i++) {
            List&lt;Integer&gt; randomNums = RandomNumGenerator.generate();
            assertAll(
                    () -&gt; assertThat(randomNums.stream().allMatch(num -&gt; num &gt;= 1 &amp;&amp; num &lt;= 9)).isTrue(),
                    () -&gt; assertThat(randomNums.stream().distinct().toList().size()).isEqualTo(3),
                    () -&gt; assertThat(randomNums.size()).isEqualTo(3)
            );
        }
    }
}</code></pre>