 在上一节内容，我们手动设计了解析跳转表，表的行对应当前解析堆栈上的非终结符，列对应当前读取的终结符，于是对应的表格数字表示当前应该采取哪个推导表达式。本节我们看看如何自动化构建解析跳转表。首先我们引入一个概念叫 First 集合，我们先看一组表达式：

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
 子集。需要注意的是，如果a对应三个非终结符的集合x,y,z，并且他们都能推导到 EPSILON, 那么 First(s)就会包含 First(x), First(y), First(z)。 

 我们看个具体例子：
 
 stmt -> expr SEMICOLON
 
 expr -> term expr_prime | EPSILON
 
 expr_prime -> PLUS term expr_prime | EPSILON
 
 term -> factor term_prime
 
 term_prime -> STAR factor term_prime | EPSILON
 
 factor -> LEFT_PAREN expr RIGHT_PAREN | NUMBER

 首先有 First(factor)={LEFT_PAREN, NUMBER}, 由于 factor 出现在 term->factor term_prime, 因此 First(factor)是 First(term)的子集， 同理 First(term)也是 First(expr)的子集，同理 First(expr)是 First(stmt)的子集。

 从表达式中可以观察到 First(factor)={LEFT_PAREN, NUMBER}, term的推导中直接跟着 factor，所以 First(term)=First(factor) = {LEFT_PAREN, NUMBER}. expr 的推导中直接跟着 term，同时它又可以直接推导到 EPSILON,
 
 因此First(expr) = {LEFT_PAREN, NUMBER, EPSILON}, expr_prime 在推导中后面只能直接跟着 PLUS, 所以 First(expr_prime)={PLUS}, stmt 在推导中跟着 expr,注意到 expr能推导到 EPSILON，因此 expr 后面的 SEMICOLON 也属于First(stmt),
 因此有First(stmt)={LEFT_PAREN,NUMBER,EPSILON,SEMICOLON}, 对应 term_prime 来说，它在推导中直接抵达 STAR，因此有 First(term_prime)={STAR}。

除了 First 集合，我们还需要了解另一种集合叫 Follow 集合。 所谓 Follow 集合就是给定某个非终结符，我们把所以在推导表达式中能直接跟着该符号的终结符找出来形成一个集合。我们看具体例子：

 1，compound_stmt -> LEFT_BRACKET stmt_list RIGHT_BRACKET,
 
 2，stmt_list -> stmt_list stmt
 
 3，stmt -> expr sEMICOLON

从第一个表达式看到 RIGHT_BRACKET 跟在 stmt_list 后面，因此它属于集合 Follow(stmt_list)。 下面我们看一个推导过程：

compund_stmt -> LEFT_BRACKET stmt_list RIGHT_BRACKET, 使用表达式 2 替换其中的 stmt_list 就有：compund_stmt -> LEFT_BRACKET stmt_list stmt RIGHT_BRACKET

于是乎 RIGHT_BRACKET 也能在表达式中跟在 stmt 后面因此它也属于集合 Follow(stmt)。 如果使用表达式 3 去替换此时的 stmt 就有：

compound_stmt -> LEFT_BRACKET stmt_list expr SEMICOLON RIGHT_BRACKET，这意味着 SEIMICOLON, RIGHT_BRACKET 属于 Follow(expr)。

我们看看如何计算前面表达式中非终结符的 Follow 集合。 首先从表达式 stmt -> expr SEMICOLON， factor -> LEFT_PAREN expr RIGHT_PAREN 可以看到 SEMICOLON, RIGHT_PAREN 都直接跟在 expr 后面，
因此有 Follow(expr)={RIGHT_PAREN,SEMICOLON} , 从表达式 expr_prime -> PLUS term expr_prime 可以看出，所有出现在 expr_prime 能直达的终结符也必然跟在 term 的后面，因此 First(expr_prime)属于 Follow(term)。
根据表达式 term_prime -> STAR factor term_prime ，任何能出现在 term_prime 能直达终结符也必然跟在 factor 后面，因此 STAR 也属于 Follow(term_prime),于是经过一轮分析我们就有：

Follow(stmt) ={}, 
Follow(expr) = {SEMI, RIGHT_PAREN}
Follow(expr_prime) = {}
Follow(term) = {PLUS}
Follow(term_prime)={}
Follow(factor)={STAR}

其实一轮分析还不够，我们需要返回运用前面的分析过程直到没有任何非终结符对应的 Follow 集合发生变化为止。 综合来说寻找 Follow 集合的步骤如下：

1，如果 s 是推导表达式中的起始符号，也就是第一个表达式箭头左边的符号，那么 EOF(end of input)这个符号先加入 Follow(s)。

2，对于表达式 s -> ... a b ... ，其中 a 是非终结符，b 是终结符或非终结符，那么 First(b)属于 Follow(a)。

3，对于表达式 s -> ... a b c ... ,其中 a 是非终结符，b 是可以推导为 EPSILON 的非终结符，那么 Follow(a)就包含 First(b)和 First(c)。

4，对于表达式 s -> ... a，其中 a 是最右边的非终结符，那么 Follow(s)是 Follow(a)的子集。

5，对于表达式 s -> ... a b1 b2 .. bn，其中 b1, b2...bn 对应可以推导到 EPSILON 的非终结符，那么 Follow(s)是 Follow(a)的子集。

 
 
 
 
