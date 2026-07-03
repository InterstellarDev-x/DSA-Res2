# Stack for Expressions

> **Topic:** [Stacks & Queues](../README.md) · **Pattern 5 of 5**
> **Problems:** Evaluate Reverse Polish Notation · Decode String · Basic Calculator II · Score of Parentheses · Basic Calculator (I)

---

## Core Concept

Stacks naturally model expression evaluation because they handle **nesting** and **delayed computation** — we push context when entering a nested scope and pop to restore when exiting.

| Problem Type | What the Stack Holds |
|-------------|---------------------|
| RPN Evaluation | Operands; pop two on each operator |
| Infix with parens | Partial sums + signs; push on `(`, restore on `)` |
| Nested strings | Repeat count + accumulated string |
| Precedence (`*`, `/`) | Stack of terms; apply `*`/`/` immediately |

---

## Problem 1: Evaluate Reverse Polish Notation — LC 150

RPN has no precedence ambiguity — operators always apply to the two most recent operands.

```rust
fn eval_rpn(tokens: Vec<String>) -> i32 {
    let mut stk: Vec<i32> = Vec::new();
    for token in &tokens {
        match token.as_str() {
            "+" => { let b = stk.pop().unwrap(); let a = stk.pop().unwrap(); stk.push(a + b); }
            "-" => { let b = stk.pop().unwrap(); let a = stk.pop().unwrap(); stk.push(a - b); }
            "*" => { let b = stk.pop().unwrap(); let a = stk.pop().unwrap(); stk.push(a * b); }
            "/" => { let b = stk.pop().unwrap(); let a = stk.pop().unwrap(); stk.push(a / b); }
            _   => { stk.push(token.parse::<i32>().unwrap()); }
        }
    }
    *stk.last().unwrap()
}
```

**Order matters for `-` and `/`:** Pop `b` first, then `a = stk.pop().unwrap()`. The result is `a op b`. For `+` and `*` order doesn't matter, but for `-` and `/` it does.

**Complexity:** O(n) time, O(n) space

---

## Problem 2: Decode String — LC 394

`"3[a2[c]]"` → `"accaccacc"`. Nested brackets require a stack to save context.

```rust
fn decode_string(s: String) -> String {
    let mut count_stack: Vec<i32> = Vec::new();
    let mut str_stack: Vec<String> = Vec::new();
    let mut current = String::new();
    let mut k = 0i32;

    for c in s.chars() {
        if c.is_ascii_digit() {
            k = k * 10 + (c as i32 - '0' as i32);  // handle multi-digit counts
        } else if c == '[' {
            count_stack.push(k);               // save current repeat count
            str_stack.push(current.clone());   // save accumulated string so far
            k = 0;                             // reset for inner expression
            current = String::new();
        } else if c == ']' {
            let times = count_stack.pop().unwrap();
            let prev = str_stack.pop().unwrap();
            let repeated = current.repeat(times as usize);
            current = prev + &repeated;        // restore + append repeated
        } else {
            current.push(c);
        }
    }
    current
}
```

**Why two stacks?** We need to separately remember the repeat count and the string accumulated before the `[`. They're pushed/popped in sync.

**Trace for `"3[a2[c]]"`:**
```
'3': k=3
'[': countStack=[3], strStack=[""], current="", k=0
'a': current="a"
'2': k=2
'[': countStack=[3,2], strStack=["","a"], current="", k=0
'c': current="c"
']': times=2, prev="a", repeated="cc", current="a"+"cc"="acc"
']': times=3, prev="", repeated="accaccacc", current=""+"accaccacc"="accaccacc"
```

**Complexity:** O(n × max_repeat) time (string building), O(n) space

---

## Problem 3: Basic Calculator II — LC 227

Handles `+`, `-`, `*`, `/` (no parentheses). Division truncates toward zero.

**Strategy:** Use a stack of "pending terms." For `+`/`-`, push the signed operand. For `*`/`/`, pop the top, compute, and push the result. Final answer = sum of stack.

```rust
fn calculate(s: String) -> i32 {
    let mut stk: Vec<i32> = Vec::new();
    let mut num = 0i32;
    let mut sign = '+';   // sign before the current number
    let bytes = s.as_bytes();
    let n = bytes.len();

    for i in 0..n {
        let c = bytes[i] as char;

        if c.is_ascii_digit() {
            num = num * 10 + (c as i32 - '0' as i32);
        }

        // Process when we hit an operator or end of string
        if (!c.is_ascii_digit() && c != ' ') || i == n - 1 {
            match sign {
                '+' => stk.push(num),
                '-' => stk.push(-num),
                '*' => { let top = stk.pop().unwrap(); stk.push(top * num); }
                '/' => { let top = stk.pop().unwrap(); stk.push(top / num); }
                _   => {}
            }
            sign = c;
            num = 0;
        }
    }

    stk.iter().sum()
}
```

**Key insight:** The `sign` variable holds the operator BEFORE the current number. When we encounter a new operator (or end of string), we apply `sign` to `num`. This lookahead approach avoids needing to parse ahead.

**Why sum at end?** `+` and `-` terms are already sign-adjusted and just need summing. `*`/`/` collapse into one value immediately on the stack.

**Trace for `"3+2*2"`:**
```
c='3': num=3
c='+': sign was '+', push 3. stack=[3]. sign='+', num=0
c='2': num=2
c='*': sign was '+', push 2. stack=[3,2]. sign='*', num=0
c='2': num=2
end: sign was '*', pop 2, push 2*2=4. stack=[3,4]
result = 3+4 = 7 ✓
```

---

## Problem 4: Score of Parentheses — LC 856

`"(())"` → 2, `"()()"` → 2, `"(()(()))"` → 6

Rules: `()` = 1; `(A)` = 2×score(A); `AB` = score(A) + score(B)

```rust
fn score_of_parentheses(s: String) -> i32 {
    let mut stk: Vec<i32> = Vec::new();
    stk.push(0);   // base score for the outermost level

    for c in s.chars() {
        if c == '(' {
            stk.push(0);   // new level
        } else {
            let inner = stk.pop().unwrap();
            let outer = stk.pop().unwrap();
            // () → 1, (A) where A>0 → 2*A
            stk.push(outer + (2 * inner).max(1));
        }
    }
    *stk.last().unwrap()
}
```

**Alternative O(1) space — count depth of each `()`:**
```rust
fn score_of_parentheses(s: String) -> i32 {
    let mut result = 0i32;
    let mut depth = 0i32;
    let bytes = s.as_bytes();
    for i in 0..s.len() {
        if bytes[i] == b'(' {
            depth += 1;
        } else {
            depth -= 1;
            if bytes[i - 1] == b'(' { result += 1 << depth; }  // 2^depth
        }
    }
    result
}
```

**O(1) space insight:** Every `()` contributes `2^depth` where `depth` is the nesting level of that `()`. `(()())` at depth 0 wrapping two `()` at depth 1 → each contributes 2^1=2 → total=4. Parent formula gives `2*(1+1)=4`. ✓

---

## Problem 5: Basic Calculator — LC 224

Handles `+`, `-`, and parentheses (no `*`, `/`). Numbers can be large.

**Strategy:** Stack saves `(result_so_far, sign_before_paren)` pairs. Inside parentheses, compute from scratch; on `)`, restore context.

```rust
fn calculate(s: String) -> i32 {
    let mut stk: Vec<i32> = Vec::new();
    let mut result = 0i32;
    let mut num = 0i32;
    let mut sign = 1i32;  // sign: +1 or -1

    for c in s.chars() {
        if c.is_ascii_digit() {
            num = num * 10 + (c as i32 - '0' as i32);
        } else if c == '+' {
            result += sign * num;
            num = 0; sign = 1;
        } else if c == '-' {
            result += sign * num;
            num = 0; sign = -1;
        } else if c == '(' {
            // Push current context; start fresh inside parens
            stk.push(result);
            stk.push(sign);
            result = 0; sign = 1;
        } else if c == ')' {
            result += sign * num;
            num = 0;
            result *= stk.pop().unwrap();    // sign before '('
            result += stk.pop().unwrap();    // result before '('
        }
    }
    result += sign * num;  // don't forget last number
    result
}
```

**Trace for `"(1+(4+5+2)-3)+(6+8)"`:**
```
c='(': push [0,1]. result=0,sign=1
c='1': num=1
c='+': result+=1*1=1. num=0,sign=1
c='(': push [1,1]. result=0,sign=1
c='4': num=4
c='+': result=4, num=0,sign=1
c='5': num=5
c='+': result=9, num=0,sign=1
c='2': num=2
c=')': result+=1*2=11. result*=1=11. result+=1=12. num=0
c='-': result+=1*0=12. num=0,sign=-1
c='3': num=3
c=')': result+=(-1)*3=9. result*=1=9. result+=0=9.
c='+': result+=1*0=9. sign=1
c='(': push [9,1]. result=0,sign=1
c='6': num=6
c='+': result=6. sign=1
c='8': num=8
c=')': result+=8=14. result*=1=14. result+=9=23
End: result+=1*0=23 ✓
```

---

## Expression Stack — Decision Framework

| Problem Features | Approach |
|-----------------|----------|
| No parens, no `*`/`/` | Simple `sign × num` accumulation |
| `*` and `/` only, no parens | Stack of terms; apply `*/` immediately |
| `+`/`-` with parens | Push `(result, sign)` on `(`, restore on `)` |
| All operators with parens | Combine: precedence stack + context save/restore |
| Postfix (RPN) | Pure operand stack; apply on each operator |
| Nested encoded structure | Two stacks: count + string context |

---

## Related Files

- [Stack Basics](./Stack%20Basics.md)
- [Monotonic Stack](./Monotonic%20Stack.md)
- [Recursive Descent Parser (Recursion topic)](../../5.Recursion/Design%20Data%20Structure%20Problems/Recursive%20Descent%20Parser.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md)

> **Last Updated:** 2026-06-26
