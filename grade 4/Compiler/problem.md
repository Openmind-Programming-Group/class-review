# 컴파일러 설계 예상문제 풀이

## Context Free Grammar (CFG)

#### 1. CFG는 몇 가지 요소를 갖고 있는가?

<details>
 <summary>정답</summary>

- `G = {T, N, S, P}`이다.
  - T: A set of terminals (터미널 기호 집합, `tokens`라고도 불림)
  - N: A set of Nonterminals (논터미널 기호 집합)
  - S: Start Symbol (시작 기호, S는 P의 부분집합)
  - P: A set of rules called `Productions` (생성 규칙의 집합)

</details>

#### 2. E ⟶ (E) | a 일 때, ((a))는 문법에 맞는가?

<details>
 <summary>정답</summary>

- `G = {T, N, S, P}`

  - N = {E}
  - T = {(, ), a}
  - S = E

- E ⟶ (E) ⟶ ((E)) ⟶ ((a)) (**Correct**)

</details>

#### 3. E ⟶ (E) | a 문법이 정의하는 언어는?

<details>
 <summary>정답</summary>

- 문법이 정의하는 언어 `L(G)`에 대해
  L(G) = {a, (a), ((a)), (((a))), ...} = {<img src="https://latex.codecogs.com/svg.image?(^{n}a)^{n}" title="(^{n}a)^{n}" /> | n ≥ 0}

</details>

#### 4. E ⟶ (E) 문법이 정의하는 언어는?

<details>
 <summary>정답</summary>

- 논터미널을 풀 수 있는 터미널 기호가 존재하지 않기 때문에 L(G) = { }

</details>

#### 5. E ⟶ E + a | a 문법이 정의하는 언어는?

<details>
 <summary>정답</summary>

- L(G) = {a, a+a, a+a+a, ...} = {strings consisting of **a**'s separated by **+**'s}

</details>

#### 6. 다음 문법이 생성하는 언어는?

```
statement ⟶ if-stmt | other
if-stmt ⟶ if (exp) | statement
        |  if (exp) statement else statement
exp ⟶ 0 | 1
```

<details>
 <summary>정답</summary>

```
other
if (0) other
if (1) other
if (0) other else other
if (1) other else other
if (0) if (0) other
if (0) if (1) other else other
if (1) other else if (0) other else other
.....
```

</details>

#### 7. A ⟶ (A)A | ε 문법이 생성하는 언어는?

<details>
 <summary>정답</summary>

- L(G) = {the strings of all "balanced parentheses"}

- (())()의 유도 과정
  - A ⟶ (A)A ⟶ (A)(A)A ⟶(A)(A)ε ⟶ (A)(ε) ⟶ ((A)A)() ⟶ ((ε)A)() ⟶ (()ε)() ⟶ (())()

</details>

#### 8. 다음 문법이 생성하는 언어는?

```
stmt-sequence ⟶ stmt; stmt-sequence | stmt
stmt ⟶ s
```

<details>
 <summary>정답</summary>

- 해당 문법은 **우순환**이기 때문에 A ⟶ ⍺A|β ⟶ <img src="https://latex.codecogs.com/svg.image?\alpha&space;^{*}\beta&space;" title="\alpha ^{*}\beta " />로 치환 가능하다.
- 즉 <img src="https://latex.codecogs.com/svg.image?(stmt;)^{*}stmt&space;\rightarrow&space;(s;)^{*}s" title="(stmt;)^{*}stmt \rightarrow (s;)^{*}s" /> 이다.
- L(G) = {s, s;s, s;s;s, ...} ⟶ ; is a separator

</details>

#### 9. 해당 문법에서 `(34-3)*42`를 좌단유도(leftmost derivations), 우단유도(rightmost derivations)하여라

```
exp ⟶ exp op exp | (exp) | number
op ⟶ + | - | *
```

<details>
 <summary>정답</summary>

- 좌단유도

  ```
  exp ⇒ exp op exp
    ⇒ (exp) op exp
    ⇒ (exp op exp) op exp
    ⇒ (number op exp) op exp
    ⇒ (number - exp) op exp
    ⇒ (number - number) op exp
    ⇒ (number - number) \* exp
    ⇒ (number - number) \* number
  ```

- 우단유도

  ```
  exp ⇒ exp op exp
      ⇒ exp op number
      ⇒ exp * number
      ⇒ (exp) * number
      ⇒ (exp op exp) * number
      ⇒ (exp op number) * number
      ⇒ (exp - number) * number
      ⇒ (number - number) * number
  ```

</details>

## 전처리

#### 1. 하향식(Top-Down) 구문 분석을 위해서는 전처리가 필요한가?

<details>
 <summary>정답</summary>

- **Yes**. 공통부분이 있는 생성규칙들은 공통부분을 묶은 생성규칙으로 변환해야하며 좌순환 되어있는 생성규칙을 우순환 생성규칙으로 변환하여야 한다.
- 하지만 상향식(Bottom-Up) 구문 분석은 전처리가 필요하지 않다.
  > LR-Parsing-Algorithm에서는 Single Production Rule `S'`를 추가한 확장문법으로 파싱되기에 어느정도의 전처리는 필요하다.

</details>

#### 2. 다음 문법에 대해 좌인수분해 하여라

```
// 1
stmt-sequence ⟶ stmt; stmt-sequence | stmt
stmt ⟶ s

// 2
exp ⟶ term + exp | term

// 3
statement ⟶ assign-stmt | call-stmt | other
assign-stmt ⟶ identifier := exp
call-stmt ⟶ identifier (exp - list)
```

<details>
 <summary>정답</summary>

```
// solved 1
stmt-sequence ⟶ stmt stmt-seq'
stmt-seq' ⟶ ; stmt-sequence | ε

// solved 2
exp ⟶ term exp'
exp' ⟶ + exp | ε

// solved 3
statement ⟶ identifier statement' | other
statement' ⟶ := exp | (exp - list)
```

</details>

#### 3. 아래 문법을 사용해 `id + id * id`를 좌단 유도로 생성해보자

```
E ⟶ E + T | T
T ⟶ T * F | F
F ⟶ (E) | id
```

<details>
 <summary>정답</summary>

- ~~절대 못한다. 좌단 유도를 통해 E ⟶ E + T 로 파싱했을 경우 E의 오른쪽에 + 기호가 들어가므로 좌단을 통해 해당 식을 만들 수는 없다.~~
- 되는 거 같아서 질문에 올림

</details>

#### 4. 해당 문법에 대해 우순환으로 바꾸어라

```
// 1
A ⟶ Aa
A ⟶ b

// 2
expr ⟶ expr + term | term

// 3
E ⟶ E + E
E ⟶ id

// 4
exp ⟶ exp addop term | term

// 5
exp ⟶ exp + term | exp - term | term

// 6
S ⟶ Aa | b
A ⟶ Ac | Sd | e
```

<details>
 <summary>정답</summary>

```
// solved 1
A ⟶ bA'
A' ⟶ aA' | ε

// solved 2
expr ⟶ term R
R ⟶ + term R | ε

// solved 3
E ⟶ id E'
E' ⟶ + E E' | ε

// solved 4
exp ⟶ term exp'
exp' ⟶ addop term exp' | ε

// solved 5
exp ⟶ term exp'
exp' ⟶ + term exp' | - term exp' | ε

// solved 6
A에 있는 Sd에서 S의 생성규칙을 대입하였을 때
S ⟶ Aa | b
A ⟶ Ac | Aad | bd | e 에서

S ⟶ Aa | b
A ⟶ bdA' | eA'
A'⟶ cA' | adA' | ε
```

</details>

## 하향식 구문분석 (Top-Down)

#### 1. First를 구하시오

```
type ⟶ simple
      |  ^ id
      |  array [simple] of type
simple ⟶ integer
        |  char
        |  num dotdot num

```

<details>
 <summary>정답</summary>

- FIRST(type) = {simple, ^, array}
- FIRST(simple) = {integer, char, num}

</details>

#### 2. First를 구하시오

```
// 1
A ⟶ aB | B
B ⟶ bC | C
C ⟶ c

// 2
S ⟶ ABc
A ⟶ bA | ε
B ⟶ c

// 3
E ⟶ Prefix (E) | v Tail
Prefix ⟶ f | ε
Tail ⟶ + E | ε
```

<details>
 <summary>정답</summary>

```
// solved 1
FIRST(A) = FIRST(aB) ∪ FIRST(B) = FIRST(a) ⨁ FIRST(B) ∪ FIRST(B) = {a, b, c}
FIRST(B) = FIRST(bC) ∪ FIRST(C) = FIRST(b) ⨁ FIRST(C) ∪ FIRST(C) = {b, c}
FIRST(C) = {c}

// solved 2
FIRST(S) = FIRST(A) ⨁ FIRST(B) ⨁ FIRST(c) = {b, c}
FIRST(A) = FIRST(bA) ∪ FIRST(ε)  = FIRST(b) ⨁ FIRST(A) ∪ FIRST(ε) = {b, ε} = {b}
first(B) = {c}

// solved 3
FIRST(E) = FIRST(Prefix(E)) ∪  FIRST(v Tail) = FIRST(Prefix) ⨁ FIRST(() ∪ FIRST(v) = {f, (} ∪ {v} = {f, (, v}
FIRST(Prefix) = {f}
FIRST(Tail) = {+}
```

</details>

#### 3. First와 Follow를 구하시오

```
// 1
S ⟶ aAb
A ⟶ aS | b

// 2
S ⟶ ABc
A ⟶ bA | ε
B ⟶ c
```

<details>
 <summary>정답</summary>

```
// solved 1
FIRST(S) = {a}
FIRST(A) = {a, b}
FOLLOW(S) = {$} ∪ FOLLOW(A) = {$, b}
FOLLOW(A) = {b}

// solved 2
FIRST(S) = {b, c}
FIRST(A) = {b}
FIRST(B) = {c}
FOLLOW(S) = {$}
FOLLOW(A) = FIRST(B) = {c}
FOLLOW(B) = {c}
```

</details>

#### 4. First와 Follow를 구하시오

```
S ⟶ A B c
A ⟶ a | ε
B ⟶ b | ε
```

<details>
 <summary>정답</summary>

```
FIRST(S) = FIRST(A) ⨁ FIRST(B) ⨁ FIRST(c) = {a, b, c}
FIRST(A) = {a}
FIRST(B) = {b}
FOLLOW(S) = {$}
FOLLOW(A) = FIRST(B) ⨁ FIRST(c) = {b, c}
FOLLOW(B) = FIRST(c) = {c}
```

</details>

#### 5. First와 Follow를 구하시오

```
E ⟶ Prefix ( E ) | v Tail
Prefix ⟶ f | ε
Tail ⟶ + E | ε
```

<details>
 <summary>정답</summary>

```
FIRST(E) = FIRST(Prefix ( E ) ) ∪ FIRST(v Tail) = FIRST(Prefix) ⨁ FIRST(() ∪ FIRST(v) = {f, (, v}
FIRST(Prefix) = {f}
FIRST(Tail) = {+}
FOLLOW(E) = FIRST()) ∪ FOLLOW(Tail) ∪ {$} = {$, )}
FOLLOW(Prefix) = FIRST(() = {(}
FOLLOW(Tail) = FOLLOW(E) = {$, )}
```

</details>

#### 6. First와 Follow를 구하시오

```
// 1
S ⟶ a S A | ε
A ⟶ b

// 2
S ⟶ a R T b | b R R
R ⟶ c R d | ε
T ⟶ R S | T a T
```

<details>
 <summary>정답</summary>

```
// solved 1
FIRST(S) = {a}
FIRST(A) = {b}
FOLLOW(S) = FIRST(A) ∪ {$} = {b, $}
FOLLOW(A) = FOLLOW(S) = {b, $}

// solved 2
좌순환을 우순환으로 전처리
T ⟶ R S T'
T' ⟶ a T T' | ε

FIRST(S) = {a, b}
FIRST(R) = {c}
FIRST(T) = FIRST(R) ⨁ FIRST(S) = {a, b, c}
FOLLOW(S) = FOLLOW(T) ∪ {$} = {a, b, $}
FOLLOW(R) = FIRST(Tb) ∪ FIRST(R) ∪ FOLLOW(S) ∪ {d} ∪ FIRST(S) = {a, b, c, d, $}
FOLLOW(T) = {a} ∪ {b} = {a, b}
```

</details>

#### 7. 해당 생성규칙에서 다음 PREDICT Set을 구하시오

```
S ⟶ A C $
C ⟶ c | ε
A ⟶ a B C d | B Q
B ⟶ b B | ε
Q ⟶ q | ε

PREDICT(S ⟶ A C $) = ?
PREDICT(Q ⟶ q) = ?
PREDICT(Q ⟶ ε) = ?
```

<details>
 <summary>정답</summary>

```
FIRST(S) = {a, b, c, q, $}
FIRST(C) = {c}
FIRST(A) = {a, b, q}
FIRST(B) = {b}
FIRST(Q) = {q}
FOLLOW(S) = {$}
FOLLOW(C) = {$, d}
FOLLOW(A) = {c, $}
FOLLOW(B) = {c, d, q, $}
FOLLOW(Q) = {c, $}

// 1
PREDICT(S ⟶ A C $) = S 는 single rule 이므로  predict set을 구할 필요가 없음

PREDICT(Q ⟶ q) = FIRST(Q) = {q}
PREDICT(Q ⟶ ε) = FOLLOW(Q) = {c, $}
```

</details>

#### 8. 다음 식에 대하여 LL(1) 조건을 만족하는지 살펴보시오

```
// 1
S ⟶ aAb
A ⟶ aS | b

// 2
S ⟶ ABc
A ⟶ bA | ε
B ⟶ c

// 3
A ⟶ iB⬅e
B ⟶ SB | ε
S ⟶ [eC] | .i
C ⟶ eC | ε

// 4
E ⟶ E + id | id
```

<details>
 <summary>정답</summary>

- LL(1) 조건을 만족하려면 A ⟶ ⍺|β 에 대해 다음과 같아야 한다.
  > 1.  FIRST( ⍺ ) ∩ FIRST( β ) = ∅
  > 2.  FOLLOW( A ) ∩ FIRST( β )= ∅,if ε ⊂ FIRST( ⍺ )

```
// solved 1
FIRST(aS) ∩ FIRST(b) = {a} ∩ {b} = ∅
>>> It's LL(1)

// solved 2
FIRST(bA) | FOLLOW(A) = {b} ∩ {c} = ∅
>>> It's LL(1)

// solved 3
FIRST(SB) ∩ FOLLOW(B) = {[, .} ∩ {⬅} = ∅
FIRST([eC]) ∩ FIRST(.i) = {[} ∩ {.} = ∅
FIRST(eC) ∩ FOLLOW(C) = {e} ∩ {]} = ∅
>>> It's LL(1)

// solved 4
우순환을 좌순환으로 변경
E ⟶ id E'
E' ⟶ + id E' | ε

FIRST(+ id E') ∩ FOLLOW(E') = {+} ∩ {$} = ∅
```

</details>

#### 9. 다음 문법과 Input값을 가지고 파싱 및 파싱테이블을 구성하시오

```
S ⟶ (S)S | ε

Input : ()
```

<details>
 <summary>정답</summary>

- FIRST(`(S)S`) ∩ FOLLOW(`S`) = {(} ∩ {), $} = ∅ 이므로 LL(1) 조건에 만족한다.

- 파싱 과정

  | Parsing Stack | Input |  Action  |
  | :------------ | ----: | :------: |
  | $S            |   ()$ | S ⟶ (S)S |
  | $S)S(         |   ()$ |  match   |
  | $S)S          |    )$ |  S ⟶ ε   |
  | $S)           |    )$ |  match   |
  | $S            |     $ |  S ⟶ ε   |
  | $             |     $ |  accept  |

- 파싱 테이블

  | M[N,T] |    (     |   )   |   $   |
  | :----: | :------: | :---: | :---: |
  |   S    | S ⟶ (S)S | S ⟶ ε | S ⟶ ε |

- 파싱 테이블의 크기는 **N · (T + 1)**이다.

</details>

#### 10. 다음 문법을 가지고 파싱 테이블을 구성하여라

```
각 생성규칙에 대해 번호를 지정하면 다음과 같다.
1. S ⟶ A C $
2. C ⟶ c
3.    | ε
4. A ⟶ a B C d
5.    | B Q
6. B ⟶ b B
7.    | ε
8. Q  ⟶ q
9.    | ε
```

<details>
 <summary>정답</summary>

```
FIRST(S) = {a, b, c, q, $}
FIRST(C) ={c}
FIRST(A) = {a, b, q}
FIRST(B) = {b}
FIRST(Q) = {q}
FOLLOW(S) ={$}
FOLLOW(C) = {$, d}
FOLLOW(A) = {c, $}
FOLLOW(B) = {c, d, q, $}
FOLLOW(Q) = {c, $}
```

- 파싱 테이블 (생성규칙에 미리 지정된 번호를 사용)

  | M[N,T] |  a  |  b  |  c  |  d  |  q  |  $  |
  | :----: | :-: | :-: | :-: | :-: | :-: | :-: |
  |   S    |  1  |  1  |  1  |     |  1  |  1  |
  |   C    |     |     |  2  |  3  |     |  3  |
  |   A    |  4  |  5  |  5  |     |  5  |  5  |
  |   B    |     |  6  |  7  |  7  |  7  |  7  |
  |   Q    |     |     |  9  |     |  8  |  9  |

</details>

#### 11. cdbba를 Input값으로 하여 10번의 파싱테이블을 통해 구문분석 하여라

<details>
 <summary>정답</summary>

| Parsing Stack |  Input |   Action    |
| :------------ | -----: | :---------: |
| $S            | abbdc$ |  S ⟶ A C $  |
| $CA           | abbdc$ | A ⟶ a B C d |
| $CdCBa        | abbdc$ |    match    |
| $CdCB         |  bbdc$ |   B ⟶ b B   |
| $CdCBb        |  bbdc$ |    match    |
| $CdCB         |   bdc$ |   B ⟶ b B   |
| $CdCBb        |   bdc$ |    match    |
| $CdCB         |    dc$ |    B ⟶ ε    |
| $CdC          |    dc$ |    C ⟶ ε    |
| $Cd           |    dc$ |    match    |
| $C            |     c$ |    C ⟶ c    |
| $c            |     c$ |    match    |
| $             |      $ |   accept    |

</details>

#### 12. 다음 문법을 가지고 First와 Follow를 구한 다음 LL(1) 조건을 만족하는지 확인하고 파싱테이블을 구성하시오

```
S ⟶ a S | b A
A ⟶ a A b | ε
```

<details>
 <summary>정답</summary>

```
FIRST(S) = {a, b}
FIRST(A) = {a}
FOLLOW(S) = {$}
FOLLOW(A) = {b, $}

FIRST(aS) ∩ FIRST(bA) = {a} ∩ {b} = ∅
FIRST(aAb) ∩ FOLLOW(A) = {a} ∩ {b, $} = ∅
>>> It's LL(1)
```

| M[N,T] |     a     |    b    |   $   |
| :----: | :-------: | :-----: | :---: |
|   S    |  S ⟶ a S  | S ⟶ b A |       |
|   A    | A ⟶ a A b |  A ⟶ ε  | A ⟶ ε |

</details>

## 상향식 구문분석 (Bottom-Up)

#### 1. 입력문장 id + id \* id 에 대해 우단 유도 과정을 보이고 핸들을 찾아보자

```
E ⟶ E + T | T
T ⟶ T * F | F
F ⟶ (E) | id
```

<details>
 <summary>정답</summary>

|   우문장 형태    |  핸들  | 감축에 사용되는 생성 규칙 |
| :--------------: | :----: | :-----------------------: |
| id1 + id2 \* id3 |  id1   |          F ⟶ id           |
|  F + id2 \* id3  |   F    |           T ⟶ F           |
|  T + id2 \* id3  |   T    |           E ⟶ T           |
|  E + id2 \* id3  |  id2   |          F ⟶ id           |
|   E + F \* id3   |   F    |           T ⟶ F           |
|   E + T \* id3   |  id3   |          F ⟶ id           |
|    E + T \* F    | T \* F |        T ⟶ T \* F         |
|      E + T       | E + T  |         E ⟶ E + T         |
|        E         |        |                           |

</details>

#### 2. A ⟶ XYZ 일 때 LR(0) item은 몇 개인가?

<details>
  <summary>정답</summary>

```
.XYZ
X.YZ
XY.Z
XYZ.

총 4개
```

</details>

#### 3. 다음 생성규칙을 갖는 문법의 LR(0) item을 구하시오

```
// 1
S' ⟶ S
S ⟶ (S)S | ε

// 2
E' ⟶ E
E ⟶ E + n | n
```

<details>
  <summary>정답</summary>

```
// solved 1
S' ⟶ .S
S' ⟶ S.
S ⟶ .(S)S
S ⟶ (.S)S
S ⟶ (S.)S
S ⟶ (S).S
S ⟶ (S)S.
S ⟶ .

// solved 2
E' ⟶ .E
E' ⟶ E.
E ⟶ .E+n
E ⟶ E.+n
E ⟶ E+.n
E ⟶ E+n.
E ⟶ .n
E ⟶ n.
```

</details>

#### 4. 다음 생성규칙을 갖는 문법에서 closure를 구하시오

```
E' ⟶ E
E ⟶ E + T | T
T ⟶ T * F | F
F ⟶ (E) | id

CLOSURE([E'⟶.E]) = ?
CLOSURE([E⟶E+.T]) = ?
CLOSURE([E⟶.T]) = ?
```

<details>
  <summary>정답</summary>

```
CLOSURE([E'⟶.E]) = E' ⟶ E.
                    E ⟶ .E + T
                    E ⟶ .T
                    T ⟶ .T * F
                    T ⟶ .F
                    F ⟶ .(E)
                    F ⟶ .id

CLOSURE([E⟶E+.T]) = E ⟶ E+T.
                    T ⟶ .T * F
                    T ⟶ .F
                    F ⟶ .(E)
                    F ⟶ .id

CLOSURE([E⟶.T]) = E ⟶ T.
                    T ⟶ .T * F
                    T ⟶ .F
                    F ⟶ .(E)
                    F ⟶ .id
```

</details>

#### 5. 4번의 생성규칙을 적용하여 I = {[E' ⟶ E.], [E ⟶ E.+T]}일 때, GOTO(I, +)를 구하시오

<details>
  <summary>정답</summary>

```
GOTO(I, +) = CLOSURE([E ⟶ E+.T])
           = {[E ⟶ E+.T], [T ⟶ .T * F], [T ⟶ .F], [F ⟶ .(E)], [F ⟶ .id]}
```

</details>

#### 6. LR(0) 문법을 충족하려면 어떠한 조건이 만족되어야하는가?

<details>
  <summary>정답</summary>

- 다음과 같은 충돌을 해결해야 LR(0) 문법을 따른다고 볼 수 있다.
  1. shift-reduce conflict
  2. reduce-reduce conflict

</details>
