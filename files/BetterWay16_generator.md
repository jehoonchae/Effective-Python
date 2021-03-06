## Better Way 16. 리스트를 반환하는 대신 제너레이터를 고려하자

#### 63쪽

* Created : 2017/02/04
* Modified: 2019/05/13

<br>

## 1. 함수에서 list 반환의 문제점


어떤 일련의 결과를 생성하는 함수에서 반환 Sequence의 타입을 정하는 가장 일반적이고 간단한 방법은 _list_ 를 반환하는 것이다. 예를 들어, 어떤 문자열에 있는 모든 단어의 인덱스를 출력하고 싶다고 하자. 여기서 단어란 `' '`를 기준으로 나뉘는 어구를 단위로 한다.  

나같은 사람이 일반적으로 택할 수 있는 방법은 말한대로 인덱스의 _list_ 를 반환하는 방법일 것이다.


```python
def index_words(text):
    result = []
    text = text.strip()

    if text:
        result.append(0)

    for index, word in enumerate(text):
        if word == ' ':
            result.append(index + 1)
    return result


>> print(index_words('Do you wanna build a snowman?'))

[0, 3, 7, 13, 19, 21]
```

반환된 리스트는 문장의 각 단어가 시작되는 인덱스를 담고 있다. 원 문자열에 _str.strip_ 메소드를 적용함으로써 빈 문자열 등에도 잘 작동하고 있다.

<br>

이 코드에 문제가 있을까? 사실 내가 일반적으로 사용하는 방식도 이런 식이기 때문에 여기서 문제점을 찾자는 저자의 문장에 의아해했다. 어떤 문제점이 있을까? 어떻게 코드를 좀더 바람직하게 만들 수 있을까?  

저자가 말하는 저 코드의 개선할 점은 다음과 같다.  

* **코드가 약간 복잡하고 깔끔하지 않다.**  
  - 이 함수의 핵심은 문자열의 인덱스를 뽑아내는 것이다. 그런데 **함수의 핵심기능과 별 관련이 없는 _result_ 리스트 선언, 원소 추가, 반환에 많은 바이트를 소모하고 있어 정작 인덱스를 뽑아내는 작업에 주의가 덜 집중된다.**  
* **결과를 일괄적으로 반환해서 메모리 에러가 발생할 수 있다.**  
  - 이는 리스트 같은 자료구조를 쓸 때 흔히 발생할 수 있는 에러로 가령 크기가 10억에 달하는 리스트를 선언하고 쓰려고 한다면 시스템에 따라 Segmentation Fault가 발생해서 파이썬 자체가 종료될 수도 있다. 이에 대한 바람직한 해결책이 바로 Iterator, Generator를 사용하는 것이다. 이 둘은 _next_ 내장함수로 평가 시에만 한 번에 한 값씩 반환하기 때문에 아까처럼 10억 개의 개수를 메모리에 모두 담지 않아도 된다.  

<br>

단순 list 사용 시의 단점에 대해서는 잘 알고 있었다. 그것이 Iterator, Generator를 쓰는 핵심적인 이유이기도 하다. 그런데 첫 번째 단점에 대해서는 생각해보지 못했다. 나도 그동안 무감각하게 단순 리스트를 선언하고, 원소 추가하고, 결과를 반환해왔다. 하지만 그게 정말 바람직한 것은 아닐 수 있구나.  

정말 일리 있는 생각이다. **확실히 저 위의 함수를 조금만 민감하게 생각하면 _result_ 와 관련된 코드 때문에 인덱스를 찾아내는 과정이 명료하게 눈에 들어오지 않는다.** 지금 저 함수는 워낙 짧고 간단해서 그렇지 각 함수가 수백, 수천 줄에 이르게 되면 이런 복잡도는 함수의 가독성을 낮추는 치명적인 역할을 할 수 있다.  

이에 대한 해결책으로 저자는 Generator를 사용하자고 말하고 있다.

<br>


## 2. Generator 사용하기


같은 함수를 제너레이터를 사용해 다시 만들어보자. 그리고 결과에 대해 리스트를 만들어야 한다면 그때 제너레이터를 리스트로 변환하자.

```python
def index_words_iter(text):
    text = text.strip()

    if text:
        yield 0

    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1


result = list(index_words('Do you wanna build a snowman?'))
>>> print(result)

[0, 3, 7, 13, 19, 21]
```

같은 기능의 함수를 이번엔 generator를 사용해 한 값씩 반환하는 기능으로 변환했다. 이 함수를 이전의 _index\_words_ 함수와 비교하면서 내가 느낀 확연한 개선은 다음과 같다.

* 가독성이 증대됐다.
  - 확실히, 단어의 시작 인덱스를 출력한다는 원 함수의 취지가 눈에 확 들어온다. 인정?
* 결과 제너레이터에 대한 사용자의 사용 자유도가 증가한다.
  - 이건 반환된 제너레이터를 _list_ 로 변환하는 코드를 통해 느꼈다. 위 예에서는 리스트로 변환했지만 사용자의 수요에 따라 때로는 set, tuple 등으로 충분히 변환할 수 있을 것이다. 이것도 참 괜찮은 것 같다.
  - 그러면 '첫 번째 반환된 리스트에 set, tuple 생성자 등을 써도 되지 않느냐'라는 질문도 충분히 할 수 있다. 하지만 이는 적절하지 않은데, 첫 째로 이미 특정 자료구조로 변환한 값을 다시 다른 자료구조로 변환하는 데 자원이 소모되고, 가령 set 등의 순서없는 자료구조로 변환한 값을 다시 _list_ 등으로 변환하면 '순서'가 사라지는 문제가 발생한다.

<br>

다음 예제를 살펴보자. 이번에는 파일에서 한 줄마다 한 번에 한 단어씩 출력을 내어주는 제너레이터를 만들어보자. **이 함수가 동작할 때 사용하는 메모리는 전체 파일의 크기가 아니라, 입력 한 줄의 최대 길이까지다.**


```python
def index_file(handle):
    offset = 0
    for line in handle :
        if line:
            yield offset
        for letter in line:
            offset += 1
            if letter == ' ':
                yield offset


with open('somebody.txt', 'r') as f:
    it = index_file(f)
    result = itertools.islice(it, 0, 3)

>>> print(list(result))

[0, 5, 11]
```

제너레이터를 사용할 때 한 가지 주의할 점은 **반환되는 이터레이터는 상태가 있고, 한 번 순회하면 다시 재사용할 수 없다는 것이다.** 다시 사용하고 싶다면 다시 할당해야 한다. 이 점만 염두에 두자.

<br>


## 3. 핵심 정리

* 제너레이터를 사용하는 방법이 누적된 결과의 리스트를 반환하는 방법보다 이해하기 명확하다.  
* 제너레이터에서 반환한 이터레이터는 제너레이터 함수의 본문에 있는 _yield_ 표현식에 전달된 값들의 순서 있는 집합이다.
* **제너레이터는 모든 입력과 출력을 메모리에 저장하지 않으므로, 입력값의 양을 알기 어려울 때도 안전하게 각 단위씩 연속된 출력을 만들 수 있다.**
