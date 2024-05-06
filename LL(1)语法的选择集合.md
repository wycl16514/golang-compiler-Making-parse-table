在前面我们构造的解析跳转表中，最顶部一行对应所有终结符，最左边一列对应非终结符，然后表中的格子对应表达式编号，我们先从解析堆栈拿到当前要解析的非终结符，然后从输入中读入终结符，接着从跳转表中查询要使用的推导表达式。我们看一个具体例子，假设有如下表达式：

1, terminal -> PERKIN_ELEMER pk

2, terminal -> ADM3 adm

3, terminal -> dec_term

4, dec_term -> VT_52

5, dec_term -> VT_100

然后它对应如下解析跳转表：

![LL(1) selection](https://github.com/wycl16514/golang-compiler-Making-parse-table/assets/7506958/f1dc2d77-7920-4f43-a235-dc1a8cc7ffd4)

根据以上信息我们得出每个表达式对应的 Select 集合如下：

Select(1) = {PERKIN_ELMER}

Select(2) = {ADM3}

Select(3) = {VT_52, VT_100}

Select(4) = {VT52}

Select(5) = {VT_100}

其中 Select(3)表示当当前输入的终结符是 VT_52, VT_100 ，解析堆栈顶部的非终结符是表达式 3 箭头左边的非终结符 terminal 时，选择表达式 3. 这里跟我们前面一节不同的是，集合针对的是表达式的编号，而不是表达式的非终结符。对于 LL(1)语法来说，
如果多个表达式箭头左边的非终结符一样，那么表达式对应的 Select 集合必须不同，要不然语法解析就无法进行，因为给定同一个非终结符，那么对应当前输入的终结符，它就可以有多个表达式可以选择，于是语法解析就不知道该选哪一个好。

我们看看生成 Select 集合的基本步骤：
1，如果一个表达式它右边所有的非终结符都可以推导出 EPSILON 或者它右边就是 EPSILON，那么我们称该表达式为 Nullable。

2，对非 nullable 的表达式，假设它有如下形式 s -> a1 a2 ...an b ... ,其中 s 是非终结符，a1...an 是一系列可以推导到 EPSILON 的非终结符集，b 是一个终结符或者是不能推导到 EPSILON 的非终结符，假设该表达式的编号为n,
那么 Select(n) = {First(a1), ..., First(an), First(b)}, 如果 a1,...an,中包含不能推导到 EPSILON 的非终结符，那么 Select(n)={First(b)}

3, 对于 nullable 的表达式 s -> a1, a2, ... an，也就是 a1, a2...an 能推导到 EPSILON，假设其编号为 n,那么 Select(n)={First(a1), First(a2),...,First(an), Follow(s)}

下面我们给出自动化创建解析跳转表的步骤：

1， 将跳转表 parse_table的每个格子设置为 error,

2, 遍历每个表达式，假设当前表达式编号为 n, lhs 为表达式箭头左边的非终结符, token 为 Select(n)中的终结符，那么有parse_table[lhs][token] = n


