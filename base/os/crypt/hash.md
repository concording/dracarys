通常，要求密码学hash函数（安全hash函数）h满足以下三点**安全属性**：

1.抗原像攻击（Preimage Resistant）：已知y \in Y（数据摘要集合），要找出x \in X（数据报文集合），使得h(x)=y是困难的；也称单向性（One-way）。

2.抗第二原像攻击(Second Preimage Resistant)：已知x \in X， 找出另一个x' \in X，使得h(x')=h(x)是困难的（计算不可行）；也称**弱抗碰撞性**(Weak Collision-Resistant)。

3.抗碰撞性(Collision-Resistant)：找出任意两个不同的x,x' \in X，使得h(x)=h(x')是困难的（计算不可行）；也称**强抗碰撞性**（Strong Collision-Resistant ）。

很多中文翻译教材将第二、第三点分别翻译为抗弱碰撞、抗强碰撞，我认为并不准确，容易导致歧义。“抗”字在前，容易导致以为有“弱碰撞”和“强碰撞”的属性。从英文的表述也可以看出，抗碰撞是一个完整的单词。