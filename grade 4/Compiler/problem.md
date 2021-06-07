# 컴파일러 설계 에상문제 풀이

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
  L(G) = {a, (a), ((a)), (((a))), ...} = {$(^n$a$)^n$ | n ≥ 0}

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

- 해당 문법은 **우순환**이기 때문에 A ⟶ ⍺A|β ⟶ $⍺^*β$로 치환 가능하다.
- 즉 $(stmt;)^*stmt ⟶ (s;)^*s$ 이다.
