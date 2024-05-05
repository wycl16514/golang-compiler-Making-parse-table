 在上一节内容，我们手动设计了解析跳转表，表的行对应当前解析堆栈上的非终结符，列对应当前读取的非终结符，于是对应的表格数字表示当前应该采取哪个推导表达式。本节我们看看如何自动化构建解析跳转表。首先我们引入一个概念叫 First 集合，我们先看一组表达式：

 statement -> LEFT_BRACKET expression RIGHT_BRACKET | expression SEMICOLON

 expression -> LEFT_PARENT expression RIGHT_PARENT | term MINUS expression | EPSILON

 term -> NUMBER | IDENTIFIER

 我们看看如果从非终结符expression 开始，它可以通过推导“直达”的终结符有哪些？ 首先根据表达式 expression -> LEFT_PARENT expression RIGHT_PARENT 可以看到它能直达非终结符 LEFT_PARENT, 然后通过表达式：
 expression -> term -> NUMBER|IDENTIFIER, 可以看到它也能“直达” NUMBER, IDENTIFIER。 此外通过 expression -> EPSILON 得出它也能直达终结符 EPSILON。
 
 注意 expression 不能“直达” RIGHT_PARENT, 因为要抵达 RIGHT_PARENT ，它需要先经过非终结符 expression, 于是这个路径必须先经过 LEFT_PARENT,
 NUMBER, INDENTIFIER 中的一个，因此 RIGHT_PARENT 无法“直达”。

 由此我们将一个给定的非终结符能直达的终结符的集合称作它的 First 集合，也就是 First(expression)={LEFT_PARENT, NUMBER, IDENTIFIER, EPSILON}. 我们看看计算 First 集合的步骤

 1， 如果 A 是一个终结符，那么 Fisrt(A) = {A}

 2， 如果存在表达式 s -> A a , 其中 s 是非终结符， a 可能是一个或多个终结符和非终结符，例如表达式  expression -> LEFT_PARENT expression RIGHT_PARENT, 那么 expression 就对应 s, LEFT_PARENT 就对应 A, expression RIGHT_PARENT 就
 对应 a, 在 s -> A a 这种情况下， A 属于集合 First(s)。

 3， 对于表达式 s -> b a，其中 s, b 对应一个非终结符， a 可以是一个或多个终结符和非终结符的集合，那么 First(b)是 First(a)的一个子集。

 4，对于表达式 s -> a b c 其中 s 是一个非终结符，a 是一个非终结符，并且 a 可以推导出 EPSILON，b 可以是一个终结符或者非终结符，那么 First(a) 并上 First(b)是 First(a)的子集。
 例如表达式 statement -> expression SEMICOLON，statement 对应 s, expression 对应a, SEMICOLON 对应 c,于是 First(expression)并上 First(SEMICOLON) 是 First(statement)的子集。 因为 First(expression)={LEFT_PARENT, NUMBER,
 IDENTIFIER, EPSILON}, First(SEMICOLON)={SEMICOLON}, 同时 expression->EPSILON，也就是 expression 能推导到EPSILON，所以两个集合并起来也就是{LEFT_PARENT, NUMBER, IDENTIFIER, EPSILON， SEMICOLON}是 First(statement)的一个
 子集
 
 
