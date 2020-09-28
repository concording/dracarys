# [Is Java “pass-by-reference” or “pass-by-value”?](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)

## ans1

I just noticed you referenced [my article](http://javadude.com/articles/passbyvalue.htm).

The Java Spec says that everything in Java is pass-by-value. There is no such thing as "pass-by-reference" in Java.

The key to understanding this is that something like

```
Dog myDog;
```

is _not_ a Dog; it's actually a _pointer_ to a Dog.

What that means, is when you have

```
Dog myDog = new Dog("Rover");
foo(myDog);
```

you're essentially passing the _address_ of the created `Dog` object to the `foo` method.

(I say essentially because Java pointers aren't direct addresses, but it's easiest to think of them that way)

Suppose the `Dog` object resides at memory address 42\. This means we pass 42 to the method.

if the Method were defined as

```
public void foo(Dog someDog) {
    someDog.setName("Max");     // AAA
    someDog = new Dog("Fifi");  // BBB
    someDog.setName("Rowlf");   // CCC
}
```

let's look at what's happening.

*   the parameter `someDog` is set to the value 42
*   at line "AAA"
    *   `someDog` is followed to the `Dog` it points to (the `Dog` object at address 42)
    *   that `Dog` (the one at address 42) is asked to change his name to Max
*   at line "BBB"
    *   a new `Dog` is created. Let's say he's at address 74
    *   we assign the parameter `someDog` to 74
*   at line "CCC"
    *   someDog is followed to the `Dog` it points to (the `Dog` object at address 74)
    *   that `Dog` (the one at address 74) is asked to change his name to Rowlf
*   then, we return

Now let's think about what happens outside the method:

_Did `myDog` change?_

There's the key.

Keeping in mind that `myDog` is a _pointer_, and not an actual `Dog`, the answer is NO. `myDog` still has the value 42; it's still pointing to the original `Dog` (but note that because of line "AAA", its name is now "Max" - still the same Dog; `myDog`'s value has not changed.)

It's perfectly valid to _follow_ an address and change what's at the end of it; that does not change the variable, however.

Java works exactly like C. You can assign a pointer, pass the pointer to a method, follow the pointer in the method and change the data that was pointed to. However, you cannot change where that pointer points.

In C++, Ada, Pascal and other languages that support pass-by-reference, you can actually change the variable that was passed.

If Java had pass-by-reference semantics, the `foo` method we defined above would have changed where `myDog` was pointing when it assigned `someDog` on line BBB.

Think of reference parameters as being aliases for the variable passed in. When that alias is assigned, so is the variable that was passed in.

## ans2


Java always passes arguments _by value_, NOT by reference.

* * *

Let me explain this through an [example](https://stackoverflow.com/a/9404727/597657):

```
public class Main {

     public static void main(String[] args) {
          Foo f = new Foo("f");
          changeReference(f); // It won't change the reference!
          modifyReference(f); // It will modify the object that the reference variable "f" refers to!
     }

     public static void changeReference(Foo a) {
          Foo b = new Foo("b");
          a = b;
     }

     public static void modifyReference(Foo c) {
          c.setAttribute("c");
     }

}
```

I will explain this in steps:

1.  Declaring a reference named `f` of type `Foo` and assign it a new object of type `Foo` with an attribute `"f"`.

    ```
    Foo f = new Foo("f");
    ```

    ![enter image description here](https://i.stack.imgur.com/arXpP.png)

2.  From the method side, a reference of type `Foo` with a name `a` is declared and it's initially assigned `null`.

    ```
    public static void changeReference(Foo a)
    ```

    ![enter image description here](https://i.stack.imgur.com/k2LBD.png)

3.  As you call the method `changeReference`, the reference `a` will be assigned the object which is passed as an argument.

    ```
    changeReference(f);
    ```

    ![enter image description here](https://i.stack.imgur.com/1Ez74.png)

4.  Declaring a reference named `b` of type `Foo` and assign it a new object of type `Foo` with an attribute `"b"`.

    ```
    Foo b = new Foo("b");
    ```

    ![enter image description here](https://i.stack.imgur.com/Krx4N.png)

5.  `a = b` makes a new assignment to the reference `a`, **not** `f`, of the object whose its attribute is `"b"`.

    ![enter image description here](https://i.stack.imgur.com/rCluu.png)

6.  As you call `modifyReference(Foo c)` method, a reference `c` is created and assigned the object with attribute `"f"`.

    ![enter image description here](https://i.stack.imgur.com/PRZPg.png)

7.  `c.setAttribute("c");` will change the attribute of the object that reference `c` points to it, and it's same object that reference `f` points to it.

    ![enter image description here](https://i.stack.imgur.com/H9Qsf.png)

I hope you understand now how passing objects as arguments works in Java :)