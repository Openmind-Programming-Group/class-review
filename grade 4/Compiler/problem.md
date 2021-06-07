# 컴파일러 설계 예상문제 풀이

## Context Free Grammar (CFG)

#### 1. CFG는 몇 가지 요소를 갖고 있는가?

- `G = {T, N, S, P}`이다.
  - T: A set of terminals (터미널 기호 집합, `tokens`라고도 불림)
  - N: A set of Nonterminals (논터미널 기호 집합)
  - S: Start Symbol (시작 기호, S는 P의 부분집합)
  - P: A set of rules called `Productions` (생성 규칙의 집합)

#### 2. E ⟶ (E) | a 일 때, ((a))는 문법에 맞는가?

- `G = {T, N, S, P}`

  - N = {E}
  - T = {(, ), a}
  - S = E

- E ⟶ (E) ⟶ ((E)) ⟶ ((a)) (**Correct**)

#### 3. E ⟶ (E) | a 문법이 정의하는 언어는?

- 문법이 정의하는 언어 `L(G)`에 대해
  L(G) = {a, (a), ((a)), (((a))), ...} = {<img src="https://latex.codecogs.com/svg.image?(^{n}a)^{n}" title="(^{n}a)^{n}" /> | n ≥ 0}

#### 4. E ⟶ (E) 문법이 정의하는 언어는?

- 논터미널을 풀 수 있는 터미널 기호가 존재하지 않기 때문에 L(G) = { }

#### 5. E ⟶ E + a | a 문법이 정의하는 언어는?

- L(G) = {a, a+a, a+a+a, ...} = {strings consisting of **a**'s separated by **+**'s}

#### 6. 다음 문법이 생성하는 언어는?

```
statement ⟶ if-stmt | other
if-stmt ⟶ if (exp) | statement
        |  if (exp) statement else statement
exp ⟶ 0 | 1
```

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

#### 7. A ⟶ (A)A | ε 문법이 생성하는 언어는?

- L(G) = {the strings of all "balanced parentheses"}

- (())()의 유도 과정
- A ⟶ (A)A ⟶ (A)(A)A ⟶(A)(A)ε ⟶ (A)(ε) ⟶ ((A)A)() ⟶ ((ε)A)() ⟶ (()ε)() ⟶ (())()

#### 8. 다음 문법이 생성하는 언어는?

```
stmt-sequence ⟶ stmt; stmt-sequence | stmt
stmt ⟶ s
```

- 해당 문법은 **우순환**이기 때문에 A ⟶ ⍺A|β ⟶ <img src="https://latex.codecogs.com/svg.image?\alpha&space;^{*}\beta&space;" title="\alpha ^{*}\beta " />로 치환 가능하다.
- 즉 <img src="https://latex.codecogs.com/svg.image?(stmt;)^{*}stmt&space;\rightarrow&space;(s;)^{*}s" title="(stmt;)^{*}stmt \rightarrow (s;)^{*}s" /> 이다.
- L(G) = {s, s;s, s;s;s, ...} ⟶ ; is a separator

#### 9. 해당 문법에서 `(34-3)*42`를 좌단유도(leftmost derivations), 우단유도(rightmost derivations)하여라.

```
exp ⟶ exp op exp | (exp) | number
op ⟶ + | - | *
```

- 좌단유도

  - exp ⇒ exp op exp
    ⇒ (exp) op exp
    ⇒ (exp op exp) op exp
    ⇒ (number op exp) op exp
    ⇒ (number - exp) op exp
    ⇒ (number - number) op exp
    ⇒ (number - number) \* exp
    ⇒ (number - number) \* number

- 우단유도
  - exp ⇒ exp op exp
    ⇒ exp op number
    ⇒ exp \* number
    ⇒ (exp) \* number
    ⇒ (exp op exp) \* number
    ⇒ (exp op number) \* number
    ⇒ (exp - number) \* number
    ⇒ (number - number) \* number

## 전처리

#### 1. 하향식(Top-Down) 구문 분석을 위해서는 전처리가 필요한가?

- **Yes**. 공통부분이 있는 생성규칙들은 공통부분을 묶은 생성규칙으로 변환해야하며 좌순환 되어있는 생성규칙을 우순환 생성규칙으로 변환하여야 한다.
- 하지만 상향식(Bottom-Up) 구문 분석은 전처리가 필요하지 않다.

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

1. stmt-sequence ⟶ stmt stmt-seq'
   stmt-seq' ⟶ ; stmt-sequence | ε

2. exp ⟶ term exp'
   exp' ⟶ + exp | ε

3. statement ⟶ identifier statement' | other
   statement' ⟶ := exp | (exp - list)

#### 3. 아래 문법을 사용해 `id + id * id`를 좌단 유도로 생성해보자

```
E ⟶ E + T | T
T ⟶ T * F | F
F ⟶ (E) | id
```

- 절대 못한다. 좌단 유도를 통해 E ⟶ E + T 로 파싱했을 경우 이미 오른쪽에 + 기호가 들어가므로 좌단을 통해 해당 식을 만들 수는 없다.

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

정답

```
// 1
A ⟶ bA'
A' ⟶ aA' | ε

// 2
expr ⟶ term R
R ⟶ + term R | ε

// 3
E ⟶ id E'
E' ⟶ + E E' | ε

// 4
exp ⟶ term exp'
exp' ⟶ addop term exp' | ε

// 5
exp ⟶ term exp'
exp' ⟶ + term exp' | - term exp' | ε

// 6
A에 있는 Sd에서 S의 생성규칙을 대입하였을 때
S ⟶ Aa | b
A ⟶ Ac | Aad | bd | e 에서

S ⟶ Aa | b
A ⟶ bdA' | eA'
A'⟶ cA' | adA' | ε
```

## 하향식 구문분석 (Top-Down)

#### 1. First를 구하시오.

```
type ⟶ simple
      |  ^ id
      |  array [simple] of type
simple ⟶ integer
        |  char
        |  num dotdot num

```

- First(type) = {simple, ^, array}
- First(simple) = {integer, char, num}

#### 2. First를 구하시오.

```
// 1
A ⟶ aB | B
B ⟶ bC | C
C ⟶ c

// 2
S ⟶ ABc
A ⟶ bA | ε
B ⟶ c
```

```
// solved 1
First(A) = First(aB) ⋃ First(B) = First(a) ⨁ First(B) ⋃ First(B) = {a, b, c}
First(B) = First(bC) ⋃ First(C) = First(b) ⨁ First(C) ⋃ First(C) = {b, c}
First(C) = {c}

// solved 2
First(S) = First(A) ⨁ First(B) ⨁ First(c) = {b, c}
First(A) = First(bA) ⋃ First(ε)  = First(b) ⨁ First(A) ⋃ First(ε) = {b, ε} = {b}
first(B) = {c}
```
