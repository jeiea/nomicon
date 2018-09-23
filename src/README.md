# 러스트노미콘(The Rustonomicon)

#### 언세이프 러스트 흑마법

> 이 지식은 멘탈을 조각내고 정신을 무한한 우주로 날려버릴 형언할 수 없는 공포
등등에 대해 명시적, 암시적인 어떤 보증도 없이 이대로 제공됩니다.

러스트노미콘은 올바른 언세이프 러스트 프로그램을 짜기 위해 필요한 모든 소름끼치는
세부사항을 다룹니다.

길고 행복한 러스트 프로그래밍 경력을 원한다면, 당장 돌아가서 이 책을 봤다는 걸
잊으면 됩니다. 이 책은 필요하지 않습니다. 하지만 언세이프 코드를 짜려 한다면,
혹은 언어의 내부를 보고 싶다면, 이 책은 유용한 정보를 가지고 있습니다.

[*러스트 프로그래밍 언어*][trpl]와 달리 독자가 선행 지식이 상당히 있다고
전제합니다. 특히 기본적인 시스템 프로그래밍과 러스트에 익숙해야 합니다.
만약 그렇지 않다면 [러스트 프로그래밍 언어][trpl]를 먼저 읽어보는 걸 권합니다. 그렇지만
독자가 꼭 그 책을 읽었다고 생각진 않고, 필요하면 기본 지식도 환기할 겁니다.
러스트 프로그래밍 언어를 건너뛰어도 상관없지만, 바닥부터 모두 설명하지는
않을 거란 것만 알아두세요.

This book exists primarily as a high-level companion to [The Reference][ref].
Where The Reference exists to detail the syntax and semantics of every part of
the language, The Rustonomicon exists to describe how to use those pieces together,
and the issues that you will have in doing so.

The Reference will tell you the syntax and semantics of references, destructors, and
unwinding, but it won't tell you how combining them can lead to exception-safety
issues, or how to deal with those issues.

It should be noted that when The Rustonomicon was originally written, The
Reference was in a state of complete disrepair, and so many things that should
have been covered by The Reference were originally only documented here. Since
then, The Reference has been revitalized and is properly maintained, although
it is still far from complete. In general, if the two documents disagree, The
Reference should be assumed to be correct (it isn't yet considered normative,
it's just better maintained).

Topics that are within the scope of this book include: the meaning of (un)safety,
unsafe primitives provided by the language and standard library, techniques for
creating safe abstractions with those unsafe primitives, subtyping and variance,
exception-safety (panic/unwind-safety), working with uninitialized memory,
type punning, concurrency, interoperating with other languages (FFI),
optimization tricks, how constructs lower to compiler/OS/hardware primitives,
how to **not** make the memory model people angry, how you're **going** to make the
memory model people angry, and more.

The Rustonomicon is not a place to exhaustively describe the semantics and guarantees
of every single API in the standard library, nor is it a place to exhaustively describe
every feature of Rust.

[trpl]: ../book/index.html
[ref]: ../reference/index.html