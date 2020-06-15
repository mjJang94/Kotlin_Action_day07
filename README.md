# 모든 클래스가 정의해야 하는 메소드

책에서 예로 들어준 메소드들이 세가지가 있다.   

toString, equals, hashCode 이 세가지로 얘기가 나뉘는데, 그 중 hashCode를 보자면

<pre>
<code>
class Client(val name: String, val postalCode: Int){
  overrride fun equals(other: Any?): Boolean{
  if(other == null || other !is Client)
  return false
  
  return name == other.name && 
  postalCode == other.postalCode
  }
  
  override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
</code>
</pre>

<pre>
<code>
val processed = hashSetOf(Client("장민종", 4222))
println(processed.contains(Client("장민종", 4222)))

</code>
</pre>

결과는 true일 것 같지만, 실제로는 false를 리턴한다. 그 이유는 Client 클래스가 hashCode 메소드를 정의하지 않았기 때문이다.
JVM 언어에서는 hashCode가 지켜야 하는 equal 메소드가 true를 반환하는 두 객체는 반드시 같은 hashCode를 반환해야 한다라는 제약이 있기 때문이다.   

precessed는 HashSet이다. 그러므로 원소를 비교할 때 비용을 줄이기 위해서 객체의 해시 코드를 비교하고 해시 코드가 같은 경우에만 실제 값을 비교한다.   
위의 예제의 두 Client 인스턴스는 해시 코드가 다르기 때문에 두 번째 인스턴스가 같은 집합이 아니라고 판단한다.

해결을 위해서는 Client에서 hashCode를 구현해야 한다.

<pre>
<code>
class Client(val name: String, val postalCode: Int){
...

override fun hashCode(): Int = name.hashCode() * 31 + postalCode **//질문**
}

</code>
</pre>

# 데이터 클래스와 불변셩: copy() 메소드
데이터 클래스의 모든 프로퍼티가 꼭 불변성인 val의 형태일 필요는 없지만, 권장은 한다. 특히 HashMap 등의 컨테이너에 데이터 클래스 객체를 담는 경우엔 불변성이 필수적이다. 데이터 클래스 객체를 키로 하는 값을 컨테이너에 담은 다음에 키로 쓰인 데이터 객체의 프로퍼티를 변경하면 컨테이너 상태가 잘못될 수 있다. 특히 다중 스레드 프로그램의 경우엔 이런 성질이 더 중요해진다. 왜냐하면 스레드가 사용 중인 데이터를 다른 스레드가 변경할 수 없으므로 스레드를 동기화할 필요가 줄어들기 때문이다.   
이러한 불변 객체를 더 쉽게 활용할 수 있게 코틀린 컴파일러가 제공하는 copy 메소드가 있다. 이 메소드는 객체를 복사하면서 일부 프로퍼티를 바꿀수 있게 도와주는 메소드로 객체를 메모리상에서 직접 바꾸는 대신 복사본을 만든다.   
이 복사본은 원본과는 다른 생명주기를 가지고, 복사를 하면서 일부 프로퍼티 값을 바꾸거나 복사본을 제거해도 프로그램에서 원본을 참조하는 다른 부분에는 무해하다.   

복사를 위해 copy메소드를 직접 구현해보자.

<pre>
<code>
class Client(val name: String, val postalCode: Int){
...

fun copy(name: String = this.name,
          postalCode: Int = this.postalCode) = 
          Client(name, postalCode)

</code>
</pre>

<pre>
<code>

val jang = Client("장민종", 4122)
println(jang.copy(postalCode = 4000))

Client(name=장민종, postalCode=4000)
</code>
</pre>

