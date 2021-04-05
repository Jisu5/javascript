#  Chapter 11. The JavaScript Standard Library

## 11.3 Pattern Matching with Regular Expression

- Regular Expression(정규 표현식)은 문자 패턴을 표현하는 객체이다.
- JavaScript의 `RegExp` 클래스는 정규 표현식을 표현하고, `String`과 `RegExp` 에는 정규 표현식을 사용해 패턴 매칭을 수행하는 메서드와 특정 텍스트를 찾아 바꾸는 함수가 정의되어 있다.

### 11.3.1 Defining Regular Expressions

- RegExp객체는 `RegExp()`생성자를 통해 만들 수 있지만, 정규 표현식 리터럴 문법이 더 자주 사용된다.
- 정규표현식 리터럴은 한쌍의 `/`문자 사이에 위치한 문자이다.

```javascript
let pattern = /s$/;
// let pattern = RegExp("s$");
```

- 이 코드는 새로운 RegExp 객체를 생성하고 `pattern`변수에 할당한다.
  이 객체는 "s"로 끝나는 모든 문자열과 매치된다.
- 정규 표현식 패턴은 연속된 문자로 구성되어 있다. 대부분의 문자들 (영숫자를 포함한) 을 패턴에 적혀있는 문자 그대로 매치된다.
  정규 표현식 `/java/` 는 "java"를 포함한 모든 문자열과 매칭된다.
- 몇몇 문자는 문자 그대로 매칭되는 것이 아닌 특별한 의미를 가진다. 예를 들어 `s$` 에서 `s` 는 문자 그대로 매치되고, `$` 는 특수 메타 문자열로 문자열의 끝과 매치된다. 

#### Literal characters

- 정규 표현식에서 모든 알파벳과 숫자들은 문자 그대로 매치되고, non-alphabetic 문자들은 `\`로 시작되는 이스케이프 문자열을 통해 지원된다.

| Character              | Matches                                                      |
| ---------------------- | ------------------------------------------------------------ |
| Alphanumeric character | Itself                                                       |
| \0                     | The NUL character (\u0000)                                   |
| \t                     | Tab (\u0009)                                                 |
| \n                     | Newline (\u000A)                                             |
| \v                     | Vertical tab (\u000B)                                        |
| \f                     | Form feed (\u000C)                                           |
| \r                     | Carriage return (\u000D)                                     |
| \xnn                   | The Latin character specified by the hexadecimal number nn; <br />for example, \x0A is the same as \n. (16진수) |
| \uxxxx                 | The Unicode character specified by the hexadecimal number xxxx; for example, \u0009 is the same as \t. |
| \u{n}                  | The Unicode character specified by the codepoint n, where n is one to six hexadecimal digits between 0 and 10FFFF. Note that this syntax is only supported in regular expressions that use the u flag. |
| \cX                    | The control character ^X; for example, ÄcJ is equivalent to the newline character Än. |

- 몇몇 구두점 문자들은 정규표현식에서 특별한 의미를 가진다.

```
^ $ . * + ? = ! : | \ / ( ) [ ] { }
```

- 이 문자 중 몇 개는 특정 정규 표현식 컨텍스트에서만 특별한 의미가 있고, 다른 컨텍스트에선 리터럴하게 다루어 진다. 
- 일반적인 규칙으로, 정규표현식에서 이 구두점 문자들을 리터럴하게 사용하고자 하면 `\`를 문자 앞에 붙여야 한다.
- 따옴표나 @ 같은 구두점 문자는 특별한 의미를 가지지 않고 문자 그대로 사용된다.
- 만약 구두점 문자를 정확히 기억하지 못한다면 모든 구두점 문자앞에 백슬래쉬(`\`)를 사용해 이스케이프할 수 있다. 
  하지만 많은 문자 및 숫자들은 앞에 `\`가 놓이면 특별한 의미를 가지기 때문에, 리터럴하게 매칭시키고 싶을 땐 백슬래쉬를 사용하지 말아야 한다.
- 정규표현식에서 백슬래쉬`\`를 리터럴 하게 사용하고 싶다면 백슬래쉬를 통해 이스케이프 해야한다. : `/\\/`
- `RegExp()`생성자를 사용한다면, 모든 백슬래쉬를 두개씩 사용해야한다는 것을 유의해야한다.  => 문자열들 역시 이스케이프 문자로 백슬래쉬를 사용하기 때문에

#### Character classes

- 개별 리터럴 문자들은 대괄호로 묶어서 문자 클래스(character class)가 될 수 있다. 
- 문자클래스는 그 클래스의 모든 문자에 매치된다. 즉, 정규표현식 `/[abc]/`는 a, b, c중 어느 문자에든 매치된다.
- 부정(Negated) 문자클래스도 정의될 수 있다. 이는 대괄호 안에 포함된 문자들을 제외한 모든 문자와 매치된다. 
  부정 문자 클래스는 `^`를 첫번째 대괄호 뒤에 사용해 표현한다. (`/[^abc]/`)
- 문자 클래스는 하이픈`-`을 사용해 문자범위를 지정할 수 있다. 알파벳 소문자와 매치되게 하려면 `/[a-z]/`를 사용할 수 있고, 모든 라틴 알파벳 혹은 숫자와 매치되게 하려면 `/[a-zA-Z0-9]/`를 사용할 수 있다.
- 어떤 문자 클래스는 자주 사용되기 때문에, 자바스크립트의 정규표현식 문법에는 자주 사용되는 특수 문자와 이스케이프 문자열로 이러한 자주 사용되는 클래스를 표현해 놓았다.
  예를 들어 `\s`는 스페이스 문자, 탭 문자, 다른 유니코드 공백 문자와 매칭된다. `\S`는 유니코드의 공백 문자가 아닌 다른 모든 문자와 매칭된다. 
- 아래 문자 클래스 리스트는 오직 ASCII 문자에 대해서만 유효하고, 유니코드 문자에 대해서는 작동하지 않을 수 있다는 점을 유의하자

| Character | Matches                                                      |
| --------- | ------------------------------------------------------------ |
| [...]     | Any one character between the brackets.                      |
| [^...]    | Any one character not between the brackets.                  |
| .         | Any character except newline or another Unicode line terminator. <br />Or, if the RegExp uses the s flag, then a period matches any character, including line terminators. |
| \w        | Any ASCII word character. Equivalent to [a-zA-Z0-9_].        |
| \W        | Any character that is not an ASCII word character. Equivalent to `[^a-zA-Z0-9_]`. |
| \s        | Any Unicode whitespace character.                            |
| \S        | Any character that is not Unicode whitespace.                |
| \d        | Any ASCII digit. Equivalent to [0-9].                        |
| \D        | Any character other than an ASCII digit. Equivalent to `[^0-9]`. |
| [\b]      | A literal backspace (special case).                          |

- 이 특별한 문자 클래스 이스케이프는 대괄호 안에서 사용할 수 있다.
  `\s`는 공백 문자에 매치되고, `\d`는 모든 숫자에 매치되기 때문에, `/[\s\d]/`는 모든 공백 혹은 숫자에 매치된다.
- 특수 케이스 `\b`는 백스페이스를 나타내며, 정규표현식에서 하나의 요소만 있는 문자 클래스를 사용해야 한다. : `/[\b]/`

##### Unicode Character Classes

- ES2018에서, 만약 정규표현식이 `u` 플래그를 사용할 때, 문자 클래스 `\p{...}`과 부정 문자클래스 `\P{...}`를 지원한다.
- 이 문자 클래스는 유니코드 표준에서 정의한 프로퍼티에 기초하며, 표현하는 문자 셋은 유니코드가 진화됨에 따라 변경된다.
- `\d` 문자 클래스는 ASCII 숫자에 대해서만 매칭된다. 만약 어떤 십진 숫자든 매칭하고자 한다면 `/\p{Decimal Number}/u`를 사용하면 된다.
- 만약 십진수가 아닌 어떤 한 문자를 모든 언어에 대해 매칭하고자 한다면 대문자 P를 사용해 `\P{Decimal_Number}`를 사용할 수 있다.
- 만약 number-like 문자(분수, 로마숫자)를 매칭하고자 한다면 `\p{Number}`를 사용한다.
- `\w`는 ASCII 문자열에 대해 작동하지만 `\p`와 함께 사용할 수 있다. : `/[\p{Alpabetic}\p{Decimal_Number}\p{Mark}]/u`

#### Repetition

- 지금까지 배운 문법을 사용해 두자리 숫자를 `/\d\d/`로 네자리 숫자를 `/\d\d\d\d/`로 표현할 수 있다. 그러나 임의 자리의 수 또는 세개의 문자와 생략가능한 숫자로 이루어진 문자열을 표현하기는 어려울 것이다.
- 복잡한 패턴의 정규표현식을 작성 할 때 요소가 얼마나 반복되는지를 특정해야 한다.
- 반복을 지정할 문자는 반복을 지정할 패턴 뒤에 나온다. 예를 들어 + 기호는 앞의 패턴이 한번 이상 나타나는 경우를 말한다.

| Character | Meaning                                  |
| --------- | ---------------------------------------- |
| {n,m}     | 앞의 패턴이 n번 이상, m번 이하 나타남    |
| {n,}      | 앞의 패턴이 n번 이상 나타남              |
| {n}       | 앞의 패턴이 n번 나타남                   |
| ?         | 앞의 항목이 0 또는 1번 나타남 (== {0,1}) |
| +         | 앞의 항목이 한번 이상 나타남 (== {1,})   |
| *         | 앞의 항목이 0번 이상 나타남 (== {0,})    |

```javascript
let r = /\d{2,4}/; // 두 자리에서 네자리 숫자와 매칭
r = /\w{3}\d?/; // 3개의 문자와 생략 가능한 숫자와 매칭
r = /\s+java\s+/; // "java"와 앞뒤로 하나이상의 공백문자를 포함한 문자열과 매치
r = /[^(]*/; // 열림 괄호가 아닌 0개 이상의 문자와 매치
```

- 하나의 문자 혹은 문자클래스에만 반복이 적용되는 것을 유의해야 한다.
- 더 복잡한 표현식에 반복이 매치되는 것을 원한다면, 다음 섹션에서 설명하는 괄호로 그룹을 정의하는 것이 필요하다.

- *, ? 반복 문자를 사용하는데 주의해야 한다. 이 문자들은 앞의 문자가 무엇이든 0번 매치된다고 간주하기 때문에 아무것도 매치되지 않는 것을 허용하기 때문이다.
- 예를 들어 `/a*/`는 문자 a가 0번 나타나는 문자열 "bbbb"와도 매치될 수 있다.

#### Non-greedy repetition

- 위의 표에 있는 반복 문자들은 주어진 정규 표현식의 부분들이 허락하는 한 가능한 많은 문자와 매칭된다. 이런 반복을 "greedy"라고 부른다.
- 반복을 non-greedy방법으로 수행하게 할 수도 있다. 
  - 반복 문자나 일반 문자 뒤에 물음표(?)를 추가하여  가능하다.
  - ??, +?, *?, {1,5}?
  - 한 개 이상의 a와 매칭되는 `/a+/`는  "aaa"에 적용하면 세 문자 모두에 매칭된다.
    `/a+?/`는 매칭되는 a가 여러번 나타나더라도 최소한의 글자에만 매칭된다. 즉 첫번째 a에만 매치된다.

- Non-greedy 반복이 항상 기대했던 결과값을 주지는 않는다. 
  - `/a+b/` 패턴을 보았을 때, 하나 이상의 a와 그 뒤에 b가 있는 문자열에 매치된다.
    "aaab"문자열에서 전체 문자열과 매치된다.
  - Non-greedy version: `/a+?b/`를 "aaab"와 매치할 때, 맨 마지막 a와 b에 매치되는 것을 기대하지만, greedy 버전처럼 전체 문자열과 매치된다. => 매칭이 가능한 첫번째 문자부터 매치를 시작하기 때문에 더 짧은 매칭은 고려하지 않는다.

#### Alternation, grouping, and references

- 정규 표현식 문법은 alternatives, grouping subexpressions, 이전의 subexpression을 참조하는 특별한 문자를 포함하고 있다.

- `|`문자는 alternative를 구분한다.

  - `/ab|cd|ef/`는 문자열  “ab” 혹은  “cd” 혹은 “ef ”와 매치된다.
  - `/\d{3}|[a-z]{4}/` 는 숫자 3개 혹은 4개의 알파벳 문자열과 매치된다.

  - alternative는 매치되는 것을 찾을 때까지 왼쪽에서 오른쪽으로 수행된다. (왼쪽이 먼저 매칭되면 오른쪽은 무시)

- 정규표현식의 괄호의 목적 중 하나는 여러항목을 하나로 그룹화 해 subexpression으로 만드는 것이다. 그룹화된 표현식은 하나의 유닛으로   |, *, +, ? 등과 사용될 수 있다.
  - ` /java(script)?/`는  “java” 뒤에 옵셔널 하게 붙는 "script"문자열과 매치된다.

- 괄호의 또다른 목적은 전체 패턴 안에서 부분 패턴을 정의하는 것이다.
  정규표현식이 타겟 문자열과 성공적으로 매칭될 때, 괄호로 둘러싸인 특정 패턴과 매치되는 부분 문자열을 추출 할 수 있다.
  - 만약 `/[a-z]+\d+/` 패턴에서 매치 이후에 숫자만을 신경 쓰고 싶다면,  `(/[a-z]+(\d+)/)`처럼 숫자를 부분 패턴으로 정의해 매치되는 숫자만을 찾을 수 있다.

- 괄호로 둘러싸인 부분 표현식(subexpression)을 사용할 때, 해당 표현식을 다시 참조할 수 있다. `\`뒤에 숫자 혹은 숫자들을 두면 된다.
  이 숫자는 정규표현식 내의 괄호처리된 subexpression의 위치를 참조한다.
  - `\1`은 첫번째 부분표현식을 참조한다.
  - 부분 표현식이 nested 될 수 있기 때문에 이 숫자는 왼쪽 괄호가 나타나는 순서임을 주의해야 한다.

| Character | Meaning                                                      |
| --------- | ------------------------------------------------------------ |
| `|`       | Alternation: match either the subexpression to the left or the subexpression to the right. |
| (...)     | Grouping: group items into a single unit that can be used with *, +, ?, \|, and so on. Also **remember** the characters that match this group for use with later references. |
| (?:...)   | Grouping only: group items into a single unit, but do not remember the characters that match this group |
| \n        | Match the same characters that were matched when group number n was first matched.<br />Groups formed with (?: are not numbered. |



#### Specifying match position

- 정규표현식의 매치가 일어날 위치를 지정하는것이 앵커이다. 
- 가장 자주 쓰이는 앵커요소는 문자열의 시작에 정규표현식을 고정시키는 `^`과 문자열의 끝에 정규표현식을 고정시키는 `$`이다.
- 예를 들어 문자열의 시작과 끝 전부가 "JavaScript"인 (한줄로 표현되는) 것과 매치하기 위해서 `/^JavaScript$/`를 사용할 수 있다.
- "Java"라는 단어 자체를 찾으려는 예제에서는 `/\sJava\s/`를 시도해 볼 수 있지만, 여기에는 두가지 문제점이 있다. 
  - 첫번째로, 앞뒤로 공백문자가 있어야만 매칭이 이루어 진다는 것이다. "Java" 문자열은 앞뒤로 공백이 없기 때문에 매칭되지 않는다.
  - 두번째로, 결과값으로 나온 값에 불필요한 공백이 포함된다는 점이다.
  - 따라서 \s 대신 \b로 단어 경계를 매치하면 위의 문제를 피할수 있다.
- \b로 단어  경계를 매치할 수 있고, \B는 단어 경계가 아닌 곳에 정규 표현식을 고정한다. 
  - `/\B[Ss]cript/`는 "postscript", "JavaScript"와 매치되지만 "script" 혹은 "Scripting"과는 매치되지 않는다.
- 임의의 정규표현식을 앵커로 사용할 수 있다.  `(?= 표현식)` 은 해당 표현식이 반드시 매치되어야 한다는 의미지만, 매치된 문자열이 결과에 포함되지는 않는다. (Lookahead)
-   `(?! 표현식)`은 부정 탐색으로 해당 표현식이 매치되면 안된다는 뜻이다. (Lookahead)



| Character | Meaning                                                      |
| --------- | ------------------------------------------------------------ |
| ^         | Match the beginning of the string or, with the m flag, the beginning of a line. |
| $         | Match the end of the string and, with the m flag, the end of a line. |
| \b        | Match a word boundary. That is, match the position between a \w character and a \W character or between a \w character and the beginning or end of a string. (Note, however, that [\b] matches backspace.) |
| \B        | Match a position that is not a word boundary.                |
| (?=p)     | A positive lookahead assertion. Require that the following characters match the pattern p, but do not include those characters in the match. |
| (?!p)     | A negative lookahead assertion. Require that the following characters do not match the pattern p. |

#### Flags

- 모든 정규표현식은 매칭행위를 대체하기 위해 관련된 하나 이상의 플래그를 가질 수 있다.
- 6가지 flag를 정의하고 있고, 각각은 한 문자로 표현된다.
- 플래그는 `//`쌍 바깥에 위치하거나,  RegExp() 생성자의 두번째 인자로 전달된다.

##### g

- 정규표현식이 'global'임을 의미한다. 첫번째 매칭뿐 아니라 모든 매칭 결과를 얻는다

##### i

- 대소문자 구분없이 매칭한다.

##### m

- Multiline mode. 여러 줄의 문자열에 대해 정규표현식을 수행하고, `^`와 `$` 앵커를 통해 문자열의 시작과 끝, 줄의 시작과 끝을 나타내기도 한다.

##### s

- m 플래그처럼 new line이 포함된 텍스트에서 수행하기에 유용하다.
- 보통 정규표현식에서 “.”는  line terminator을 제외한 모든 문자와 매칭되지만, s플래그를 함깨 사용하면 line terminator를 포함한 모든 문자와 매치된다.

##### u

- u 플래그는 유니코드를 위해 사용되고, 정규표현식이 16 비트 값보다 유니코드 코드포인트와 매치되게 한다.
- ES6에서 소개된 플래그로, 특별한 이유없는한 모든 정규표현식에 사용하는 습관을 들이자.
-  이 플래그 없이는 이모지나 한자등 다른 문자들이 포함된 텍스트에서 잘 작동하지 않는다.

##### y

- y 플래그는 정규표현식이 'sticky'함을 가리키고, 문자열의 첫번째 문자와 패턴이 매치되어야 함을 말한다.
- 하나의 매칭을 찾도록 설계된 정규표현식을 사용할때 `^`앵커를 통해 문자열의 시작지점을 지정할 수 있다.
- 이 플래그는 문자열에서 모든 매치를 반복적으로 찾는 정규표현식에서 유용하다.



### 11.3.2 String Methods for Pattern Matching

- 정규 표현식을 사용하는 String 객체의 메서드

#### search()

- 매치되는 문자열의 위치를 반환한다. 만약 매칭되는 문자가 없다면 -1을 반환한다.
- `search()`에 넘겨지는 인자가 정규표현식이 아니라면, 인자는 RegExp() 생성자로 넘겨져 정규 표현식으로 변환된다.
- 전역 검색을 지원하지 않기 때문에, g플래그는 무시된다.

```javascript
"JavaScript".search(/script/ui) // => 4
"Python".search(/script/ui) // => -1
```

#### replace()

- 검색 후 바꾸기를 수행한다.
- 첫 번째 인자로 정규 표현식, 두번째 인자로 교체될 문자열을 받는다. 
- g 플래그로 전역 검색과 바꾸기를 수행할 수 있다. g플래그가 없다면 첫번째로 매치되는 문자열만 찾아 바꾸기를 수행한다.
- 만약 첫번째 인자로 정규표현식이 아닌 문자열을 받는다면, 정규표현식으로 변환하지 않고 문자열 그대로 찾아 바꾼다. (search()와 다르다.)

```javascript
// 대소문자 구분없이 전역으로 검색하여 변환한다.
text.replace(/javascript/gi, "JavaScript");
```

- 괄호로 둘러싸인 부분 표현식은 왼쪽에서 오른쪽으로 번호가 매겨지고 각 부분 표현식이 매치된 텍스트를 참조한다.
- 만약 교체할 문자열의 $다음 숫자가 나오면 replace()는 해당 번호가 가리키는 부분 표현식에 매치된 텍스트로 바꾼다.

```javascript
// 인용부호와 인용부호가 아닌 임의 개수의 문자, 그리고 인용부호가 오는 정규 표현식 ("text")
let quote = /"([^"]*)"/g;
// Replace the straight quotation marks with guillemets
// leaving the quoted text (stored in $1) unchanged.
'He said "stop"'.replace(quote, '«$1»') // => 'He said «stop»'
```

- name captured group을 사용해 번호 대신에 이름으로 참조하여 바꿀 수 있다.

```javascript
let quote = /"(?<quotedText>[^"]*)"/g;
'He said "stop"'.replace(quote, '«$<quotedText>»') // => 'He said «stop»'
```

- replace()의 두번째 인자로는 동적으로 교체될 문자열을 계산하는 함수가 될 수도 있다.
- 교체 함수는 여러 인자를 받는데, 첫번째 인자는 매치되는 전체 문자열이고, 다음으로 만약 정규표현식이 capturing group을 가진다면 그 그룹으로 capturing 되는 substring이 인자로 온다. 그리고 다음 인자로 매치될 문자열의 위치가 전달된다.

```javascript
let s = "15 times 15 is 225";
s.replace(/\d+/gu, n => parseInt(n).toString(16)) // => "f times f is e1"
```



#### match()

- 가장 일반적인 String 정규표현식 메서드이다.
- 정규 표현식 하나만을 인자로 전달받고, 정규 표현식이 아닌 경우에는 RegExp()생성자로 정규표현식으로 변환한다. 
- 매치 결과를 배열로 반환한다. (g 플래그가 있던 없던 배열로 반환한다.)

```javascript
"7 plus 8 equals 15".match(/\d+/g) // => ["7", "8", "15"]
```

- 전역 검색이 아닌 경우에는 첫번째 요소는 매치된 문자열이고, 나머지는 정규 표현식의 부분표현식에 매치된 문자열이다.

  배열 n번째 인덱스에는 $n에 해당하는 내용이 저장되어 있다.

```javascript
// A very simple URL parsing RegExp
let url = /(\w+):\/\/([\w.]+)\/(\S*)/;
let text = "Visit my blog at http://www.example.com/~david";
let match = text.match(url);
let fullurl, protocol, host, path;
if (match !== null) {
  fullurl = match[0]; // fullurl == "http://www.example.com/~david"
  protocol = match[1]; // protocol == "http"
  host = match[2]; // host == "www.example.com"
  path = match[3]; // path == "~david"
}
```

- 전역 검색이 아닌 경우에 match()로 반환되는 결과는 배열 뿐 아니라 객체 프로퍼티도 가지고 있다.
- Match()로 호출된 문자열은 input 프로퍼티를 참조한다. index 프로퍼티는 매치가 시작된 인덱스가 저장되어 있다.
- 만약 정규표현식이  named capture groups를 포함한다면 반환된 배열은 groups 프로퍼티 역시 가지고 있다.

```javascript
let url = /(?<protocol>\w+):\/\/(?<host>[\w.]+)\/(?<path>\S*)/;
let text = "Visit my blog at http://www.example.com/~david";
let match = text.match(url);
match[0] // => "http://www.example.com/~david"
match.input // => text
match.index // => 17
match.groups.protocol // => "http"
match.groups.host // => "www.example.com"
match.groups.path // => "~david"
```

- Match() 메소드는 정규표현식이 g플래그를 가지고 있는지 아닌지에 따라 다르게 동작한다.
- 뿐만 아니라, y플래그 여부에 따라서도 굉장히 다르게 동작한다.
- 만약 g, y 플래그를 둘다 가지고 있을 때, y없이 g 플래그만 있는 것처럼 결과값을 반환한다.
- 하지만 첫 매치는 무조건 문자열의 시작이어야 하며,  모든 연속된 매치는 앞전의 매치 뒤에 연속되어야 한다.

- g 없이 y 플래그만 사용되었을 때, match()는 하나의 매치만 찾으며, 이 매칭은 문자열의 처음부터 찾는 것이 기본 값이다.
- 이  default match의 시작 위치를 바꾸기 위해서는 RegExp 객체의 lastIndex 프로퍼티를 셋팅하면 된다.
- 매칭을 찾게되면 lastIndex는 자동으로 매치 다음 첫 글자로 업데이트 된다. 
- 연속으로 match()를 호출한다면 연속된 매치를 찾을 수 있다. 

```javascript
let vowel = /[aeiou]/y; // Sticky vowel match
"test".match(vowel) // => null: "test" does not begin with a vowel
vowel.lastIndex = 1; // Specify a different match position
"test".match(vowel)[0] // => "e": we found a vowel at position 1
vowel.lastIndex // => 2: lastIndex was automatically updated
"test".match(vowel) // => null: no vowel at position 2
vowel.lastIndex // => 0: lastIndex gets reset after failed match
```

- match() 메소드에 non-global 정규표현식을 전달하는것은 정규표현식의 exec()메소드에 문자열을 전달하는 것과 같은 결과값을 가진다.

#### matchAll()

- ES 2020부터 정의되었다.
- g 플래그와 함꼐 사용되는 것을 기대한다.
- match()처럼 배열을 반환하지 않고, iterator를 반환한다. 
- g 플래그가 없는 것처럼 하나의 match object들이 각각 들어가 있다.

```javascript
// One or more Unicode alphabetic characters between word boundaries
const words = /\b\p{Alphabetic}+\b/gu; // \p is not supported in Firefox yet
const text = "This is a naïve test of the matchAll() method.";
for(let word of text.matchAll(words)) {
console.log(`Found '${word[0]}' at index ${word.index}.`);
}
/*
Found 'This' at index 0.
Found 'is' at index 5.
Found 'a' at index 8.
Found 'naïve' at index 10.
Found 'test' at index 16.
Found 'of' at index 21.
Found 'the' at index 24.
Found 'matchAll' at index 28.
Found 'method' at index 39.
*/
```

-  matchAll()에서 문자열의 어떤 인덱스에서 매칭이 되는지 lastIndex property로 확인할 수 있다.

#### split()

- Split() 메소드는 주어진 인자를 구분자 삼아 문자열을 부분 문자열로 쪼개 배열로 반환한다.

```javascript
"123,456,789".split(",") // => ["123", "456", "789"]
```

- 구분자로 정규표현식을 인자로 받을 수 있다. 

```javascript
"1, 2, 3,\n4, 5".split(/\s*,\s*/) // => ["1", "2", "3", "4", "5"]
```

- RegExp delimiter와 capturing groups가 포함된 정규표현식을 전달받을 때는 capturing groups에 매치되는 텍스트를 배열에 포함해 반환한다.

```javascript
const htmlTag = /<([^>]+)>/; // < followed by one or more non->, followed by >
"Testing<br/>1,2,3".split(htmlTag) // => ["Testing", "br/", "1,2,3"]
```



### 11.3.3 The RegExp Class

- 정규 표현식은 RegExp 객체로 표현된다. 

#### RegExp properties

- source
  - Read-only
  - / 사이의 문자열

- flags
  - Read-only
  -  정규표현식에 설정되어있는 플래그 문자열

- global
  - Read-only, boolean
  - g 플래그가 설정되었다면 true\\\\\\\\\\\\\\\

- ignoreCase
  - Read-only, boolean
  - i 플래그가 설정되었다면 true

- multiline
  - Read-only, boolean
  - m 플래그가 설정되었다면 true

- dotAll
  - Read-only, boolean
  - s 플래그가 설정되었다면 true

- unicode
  - Read-only, boolean
  - u 플래그가 설정되었다면 true

A read-only boolean property that is true if the u flag is set.

- sticky
  - Read-only, boolean
  - y 플래그가 설정되었다면 true

- lastIndex
  - Read/write 정수
  - g / y 플래그가 있는 패턴에서 다음 검색이 시작되는 위치를 특정한다.

#### test()

- 정규 표현식을 사용하는 가장 간단한 메소드이다.
- 매치되는 문자열이 있다면 true, 없다면 false를 반환한다.
- lastIndex에 지정된 위치부터 문자열을 검색
- g, y 플래그와 같이 사용한다면 lastIndex 프로퍼티에 의존적으로 동작한다.

#### exec()

- 정규 표현식을 사용하는 가장 일반적이며 파워풀한 방법이다.
- 하나의 문자열 인자를 가지고 주어진 문자열에 대해 매칭을 수행한다.
- 매치되는것이 없다면 null을 반환한다. 매치되는 것이 있다면 non-global 탐색을 했던 match()메소드 처럼 배열을 반환한다.
- 배열의 0번쨰 요소는 정규표현식과 매치되는 문자열이고, capturing group과 매치되는 substring이 연속된 요소로 들어온다.
- 반환되는 배열은 index, input, groups 프로퍼티를 가진다.

- match()메소드와 다르게 exec()는 g 플래그를 가지던 말던 같은 종류의 배열을 반환한다.
- 항상 하나의 매치 결과 만을 반환하고, 해당 매치에 대한 전체 정보를 제공한다.
- g / y 플래그가 있는 정규표현식을 exec()로 부른다면, lastIndex 프로프티는 매치된 문자열 바로 다음 문자의 위치가 저장된다.
- 새로 생성된 RegExp 객체의 lastIndex 는 0이며 탐색은 문자열의 처음에서 시작한다.
- Exec()로 매치를 성공적으로 찾는다면 lastIndex는 매칭 문자열 다음 문자로 업데이트 되고, 만약 매칭을 찾는데 실패했다면 lastIndex 는 0으로 리셋된다.

```javascript
let pattern = /Java/g;
let text = "JavaScript > Java";
let match;
while((match = pattern.exec(text)) !== null) {
console.log(`Matched ${match[0]} at ${match.index}`);
console.log(`Next search begins at ${pattern.lastIndex}`);
}
```

