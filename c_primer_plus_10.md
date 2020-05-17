# C Primer Plus 10 - 数组和指针

## Arrays

> P 383
>
> An *array declaration* tells the compiler how many elements the array contains and what the type is for these elements.

```c
int days[];  // error: definition of variable with array type needs an explicit size or an initializer
```

>P 384
>
>As you can see, you initialize an array by using a comma-separated list of values enclosed in braces.

```c
int powers[3] = {1, 2, 4};
```

> P 386
>
> Just as with regular variables, you should use the declaration to initialize const data because once it's declared const, you can't assign values later.

```c
const int powers[3];
powers[0] = 1 // error: read-only variable is not assignable
```

> P 386
>
> The array members are like ordinary variables - if you don't initalize them, they might has any value

不要使用未初始化的数组.

> P 387
>
> but if you partially initialize an array, the remaining elements are set to 0

```c
int powers[3] = {1};  // 等价于 int powers[3] = {1, 0, 0};
```

> P 387
>
> The compiler is not so forgiving if you have too many list values.

```c
int powers[3] = {1, 2, 4, 8};  // warning: excess elements in array initializer
```

> P 388
>
> When you use empty brackets to initialize an array, the compiler counts the number of items in the list and makes the array that large.
>
> So **sizeof days** is the size, in bytes, of the whole array, and **sizeof days[0]** is the size, in bytes, of one element. Dividing the size of the entire array by the size of one element tells us what many elements are in the array.

在初始化数组时, 如果没有指定大小, 那么编译器会帮你计算. 同时, 可以使用 sizeof 来算出数组的大小.

```c
int powers[] = {1, 2, 4};
printf("%zd\n", sizeof powers / sizeof powers[0]);  // 3
```

> P 389
>
> With C99, you can use an index in brackets in the initialization list to specify a particular element
>
> First, if the code follows a designated initializer with further values, these further values are used to initialize the subsequent elements.
>
> Second, if the code initializes a particular element to a value more than once, the last initialization is the one that takes effect.

```c
int powers[3] = {[2] = 4};  // 等价于 int powers[3] = {0, 0, 4};
int powers[4] = {1, [2] = 4, 8};  // 等价于 int powers[4] = {1, 0, 4, 8};
int powers[SIZE] = {1, [2] = 4, 8, [0] = 16};  // 等价于 int powers[4] = {16, 0, 4, 8};  warning: initializer overrides prior initialization of ths subobject
```

> P 390
>
> Note that the code uses a loop to assign values element by element.

通常如果不初始化数组, 那么就用循环赋值.

```c
int powers[3];
for (int i = 0; i < 3; i++) {
    powers[i] = pow(2, i);
}
```

> P 390
>
> C dosen't let you assign one array to another as a unit.

```c
int powers[3] = {0};
int other[3];
other = powers  // error array type 'int [3]' is not assignable
```

> P 390
>
> For instance suppose you make the following declaration: **int doofi[20];**. Then it'is your responsibility to make sure the program uses indices only in the range 0 through 19, becasue the compiler isn't required to check for you.

你必须自己确保索引范围正确, 编译器没有被要求去检查索引是否合法 (当然有的编译器会抛出 warning). 

```c
int powers[3];
powers[3] = 2;  // warning: array index 3 is past the end of the array (which contains 3 elements)
```

> P 392
>
> Until the C99 standard, the answer has been that you have to use a constant integer expression between the brackets when declaring an array.

数组大小需要是一个常量表达式. 否则创建的是一个 vla, 而并不是一个普通的数组.

```c
const int n = 5;
int m = 5;
int a1[n];  // n 是常量不是一个, 常量表达式, 因此按照书中所说这里创建的是一个 VLA, 但在我的 gcc 中, a1 好像仍是一个普通的数组. 但还是优先参考书中的结果, 不使用这种方式创建数组.
int a2[m];  // gcc -Wvla 如果使用这种方式编译, 那么会抛出 warning: variable length array used
int a3[sizeof(int)];  // sizeof 在编译时就会确定结果, 因而 sizeof(int) 的值是 const
int a4[n + 3]  // n + 3 的值是 const
```

> P 392
>
> As of C99, however, C does allow them, but they create a new breed of array, something called a *variable-length array*, or *VLA* for short. (C11 retreats from this bold initiative, making VLAs an optional rather than mandatory language feature.
>
> C99 introduced variable-length arrays primarily to allow C to become a better language for numerical computing. VLAs have some restrictions; for example, you can’t initialize a VLA in its declaration.

从个人的观点上来看, 先不要使用 VLA. 因而使用 gcc 编译时带上 `-Wvla`, 确保自己没有意外地用了 VLA.

```c
const int n = 5;
int m = 5;
int a1[n] = {0};
int a2[m] = {0};  // error: variable-sized object may not be initialized
```

## Multidimensional Arrays

> P 393
>
> Here is how to declare such an array:
>
> float rain[5][12];   // array of 5 arrays of 12 floats
>
> One way to view this declaration is to first look at the inner portion (the part in bold):
>
> float **rain[5]**[12];   // rain is an array of 5 somethings
>
> It tells us that rain is an array with five elements. But what is each of those elements? Now look at the remaining part of the declaration (now in bold):
>
> **float** rain[5] **[12]**;  // an array of 12 floats
>
> This tells us that each element is of type float[12]; that is, each of the five elements of rain is, in itself, an array of 12 float values.

对于 `float rain[5][12]` 来说, 由于两个 `[]` 优先级相同, 因此从左往右计算. 第一步, 是 `rain[5]`, 即声明了 `rain` 是一个有 5 个元素的数组, 第二步是 `float[12]`, 即每个数组都是由 12 个 float 组成.

> P 397
>
> You could omit the interior braces and just retain the two outermost braces. If you are short of entries, however, the array is filled sequentially, row by row, until the data runs out. Then the remaining elements are initialized to 0.

```c
int powers[2][3] = {{1, 2}, {4, 8}}  // 等价于 {{1, 2, 0}, {4, 8, 0}}
int powers[2][3] = {1, 2, 4, 8}  // 等价于 {{1, 2, 4}, {8, 0, 0}
```

## Pointers and Arrays

> P 397
>
> An example of this disguised use is that an array name is also the address of the first element of the array.

```c
int powers[3];
printf("%d\n", powers == &powers[0]);  // 1
```

> P 399
>
> What is happening here is that when you say "add 1 to a pointer", C adds one storage unit. For arrays, that means the address is increased to the address of the next element, not just the next byte.

```c
int powers[3];
for (int i = 0; i < 3; i++) {
    printf("%p\n", powers + i);  // 每次打印的地址都等于前一个地址 + 4 (sizeof(int) == 4).
}
double powers[3];
for (int i = 0; i < 3; i++) {
    printf("%p\n", powers + i);  // 每次打印的地址都等于前一个地址 + 8 (sizeof(double) == 8) 
}
```

> P 400
>
> As a result of C's cleverness, we have the following equalities:
>
> dates + 2 == &dates[2]
>
> \*(dates + 2) == dates[2]

```c
int powers[3] = {1, 2, 4};
printf("%p %p\n", powers + 2, &powers[2]);  // 两个值是一样的
printf("%d %d\n", *(powers + 2), powers[2]);  // 两个值是一样的
```

## Functions, Arrays, and Pointers

> P 403
>
> Because the name of an array is the address of the first element, an actual argument of an array name requires that the matching formal argument be a pointer. In this context, and only in this context, C interprets int ar[] to mean the same as int * ar; that is, ar is type pointer-to-int. Because prototypes allow you to omit a name, all four of the following prototypes are equivalent:
>
> int sum(int*ar, int n); 
>
> int sum(int*, int);
>
> int sum(int ar[], int n); 
>
> int sum(int [], int);

在函数参数中, int ar[] 和 int* ar 是等价的, 他们都意味着 ar 是 pointer-to-int 类型, 只不过 int ar[] 可以让使用者更加清楚, 这里应该传入一个一维 int 数组, 而不是一个普通的 int 指针.

> P 404
>
> In short, in Listing 10.10, marbles is an array, ar is a pointer to the first element of marbles, and the C connection between arrays and pointers lets you use array nota- tion with the pointer ar.

```c
void test(int ar[]);

int main(void) {
    int powers[3] = {1, 2, 4};
    printf("%zd\n", sizeof powers);  // 12 == 3 * sizeof(int)
    test(powers);  // 8, 因为当前系统使用 8 bytes 来表示地址, 而函数中的 ar 实际上就是一个 int 类型的地址, 因此结果是 8.
    return 0;
}

void test(int ar[]) {
    printf("%zd\n", sizeof ar);
}
```

> P 404
>
> A function working on an array needs to know where to start and stop. The sum() function uses a pointer parameter to identify the beginning of the array and an integer parameter to indicate how many elements to process. Another way to describe the array is by passing two pointers, with the first indicating where the array starts and the second where the array ends.

在函数中操作数组通常需要知道开始位置和结束位置. 第一种方式是告诉开始位置和元素个数, 第二种方式是告诉开始位置和结束位置.

```c
int sum(int ar[], int n);
int sump(int* start, int* end);
int main(void) {
    int powers[3] = {1, 2, 4};
    printf("sum of powers is %d \n", sum(powers, 3));  // 7
    printf("sump of powers is %d\n", sump(powers, powers + 3)); // 7
    return 0;
}
int sum(int ar[], int n) {
    int total = 0;
    for (int i = 0; i < n; i++) {
        total += ar[i];
    }
    return total;
}
int sump(int* start, int* end) {
    /* 计算数组在范围 [start...end) 中的总和. */
    int total = 0;
    while (start < end) {
        total += *start;
        start ++;
    }
    return total;
}
```

> P 407
>
> As far as C goes, the two expressions ar[i] and \*(ar+i) are equivalent in meaning. Both work if ar is the name of an array, and both work if ar is pointer variable. However, using an expression such as ar++ only works if ar is a pointer variable.

```c
int powers[3];
int* pti = powers;
pti++;
powers++;  // error: cannot increment value of type 'int [3]'
```

## Pointer Operations

> P 409
>
> You can assign an address to a pointer. The assigned value can be, for example, an array name, a variable preceded by address operator (&), or another second pointer.

```c
int powers[3];
int num = 10;
int* pt1 = &powers[2];
int* pt2;

pt2 = powers;
printf("%d\n", pt2 == powers);
pt2 = &num;
printf("%d\n", pt2 == &num);
pt2 = pt1;
printf("%d\n", pt2 == pt1);
```

> P 409
>
> Note that the address should be compatible with the pointer type.

```c
double d = 10.0;
int* pt = &d;  // warning: incompatible pointer types initializing 'int *' with an expression of type 'double *'
```

> P 410
>
> You can use the + operator to add an integer to a pointer or a pointer to an integer. In either case, the integer is multiplied by the number of bytes in the pointed-to type, and the result is added to the original address.

```c
int powers[3];
double powers2[3];
int* pti = powers;
double* ptd = powers2;
printf("%p %p\n", pti, pti + 1);  // 后一个值比前一个值多 4 (sizeof(int))
printf("%p %p\n", ptd, ptd + 1);  // 后一个值比前一个值多 8 (sizeof(double))
```

> P 410
>
> You can use the - operator to subtract an integer from a pointer; the pointer has to be the first operand and the integer value the second operand. The integer is multiplied by the number of bytes in the pointed-to type, and the result is subtracted from the original address.

```c
int powers[3];
double powers2[3];
int* pti = &powers[1];
double* ptd = &powers2[1];
printf("%p %p\n", pti, pti - 1);  // 前一个值比后一个值多 4 (sizeof(int))
printf("%p %p\n", ptd, ptd - 1);  // 前一个值比后一个值多 8 (sizeof(double))
```

> P 411
>
> You can find the difference between two pointers. Normally, you do this for two pointers to elements that are in the same array to find out how far apart the elements are. The result is in the same units as the type size.

```c
int powers[3];
int* pt1 = &powers[0];
int* pt2 = &powers[2];
printf("%ld\n", pt2 - pt1);  // 2
printf("%ld\n", pt1 - pt2);  // -2
```

> P 411
>
> You can use the relational operators to compare the values of two pointers, provided the pointers are of the same type.

```c
int powers[3];
int* pt1 = &powers[0];
int* pt2 = &powers[2];
printf("%d\n", pt2 > pt1);
```

## Protecting Array Contents

> P 413
>
> If a function's intent is that it not change the contents of the array, use the keyword const when declaring the formal parameter in the prototype and in the function definition.

```c
int sum(const int ar[], int n);
int main(void) {
    int powers[3] = {1, 2, 4};
    sum(powers, 3);
    return 0;
}
int sum(const int ar[], int n) {
    ar[0]++;  // read-only variable is not assignable
}
```

> P 416
>
> First it's valid to assign the address of either constant data or non-constant data to a pointer-to-constant.
>
> However, only the address of non-constant data can be assigned to regular pointers.

```c
int powers[3] = {1, 2, 4};
const int const_powers[3] = {1, 2, 4};
int* pt;
const int* const_pt;

pt = powers;
pt = const_powers;  // warning: assigning to 'int *' from 'const int[3]' discards qualifiers
const_pt = powers;
const_pt = const_powers;
```

> P 417
>
> For example, you can declare and initialize a pointer so that it can't be made to point elsewhere.

```c
int powers[3] = {1, 2, 4};
int* const pt;
pt = powers;  // error: cannot assign to variable 'pt' with const-qualified type 'int *const'
```

> P 417
>
> Finally, you can use const twice to create a pointer that can neither change where it's pointing nor change the value to which it points

```c
int powers[3] = {1, 2, 4};
const int* const pt = powers;
```

## Pointers and Multidimensional Arrays

> P 417
>
> Suppose you have this declaration:
>
> int zippo\[4\]\[2\];
>
> Because zippo is the address of the array's first element, zippo has the same value as &zippo[0]. Next, zipp\[0\] is itself an array of two integers, so zippo\[0\] has the same value as &zippo\[0\]\[0\], the address of its first element, an int. In short, zippo[0] is the address of an int-sized object, and zippo is the address of a two-int-sized object.

数组在大多数情况下会退化成比自己低一维度的指针, 例如二维数组 zippo 变成指向 int\[2\] 的指针, 即 &zippo[0], 而一维数组 zippo[0] 变成指向 int 的指针, 即 &zippo\[0\]\[0\].

```c
int powers[2][3] = {{1, 2, 4}, {8, 16, 32}};
int (* pt1)[3] = powers;
int (* pt2)[3] = &powers[0];
int* pt3 = powers[0];
int* pt4 = &powers[0][0];
```

> P 417
>
> Adding 1 to a pointer or address yields a value larger by the size of the referred-to object. In this respect, zippo and zippo[0] differ, because zippo refers to an object two ints in size, and zippo[0] refers to an object one int in size.

```c
printf("%p %p\n", pt1, pt1 + 1);  // 相差一个 3 * sizeof(int)
printf("%p %p\n", pt2, pt2 + 1);  // 相差一个 3 * sizeof(int)
printf("%p %p\n", pt3, pt3 + 1);  // 相差一个 sizeof(int)
printf("%p %p\n", pt4, pt4 + 1);  // 相差一个 sizeof(int)
```

> P 417
>
> Dereferencing a pointer or an address (applying the * operator or else the [] operator with an index) yields the value represented by the referred-to object. Because zippo[0] is the adddress of its first element, (zippo\[0\]\[0\]), *(zippo[0]) represents the value stored in zippo\[0\]\[0\], an int value. Similarly, *zippo represents the value of its first element, zippo[0], but zippo[0] itself is the address of an int.

zippo 是一个二维数组, 它通常会退化成一维数组的指针, 因而 *zippo 就等于一维数组, 而 *zippo 或 zippo[0] 代表一维数组, 它通常会退化成一维数组中元素的指针 (这里是 int 的指针), 那么 \*\*zippo 或 \*zippo[0] 就等于元素本身 (这里是 int).

```c
int powers[2][3] = {{1, 2, 4}, {8, 16, 32}};
int* pt = *powers;
int val = **powers;
printf("%p %p\n", pt, pt + 1);  // 相差一个 sizeof(int)
printf("%d\n", val);  // 1
```

> P 420
>
> Here is what you can do:
>
> int (* pz)[2];  // pz points to an array of 2 ints
>
> This statement says that pz is a pointer to an array of two ints. Why the parentheses ? Well, \[ \] has a higher precedence than \*. Therefore with a declaration such as
>
> int* pax[2];  // pax is an array of two pointers-to-int

由于 [] 的优先级高于 *, 所以如果你要生成一个指针, 那么就需要先执行 *, 而 () 中的内容是最先执行的. 实际上 int (\* pz)[2] 变成 int[2] * pz 更好理解一些.

```c
int (* pz)[3];
int* pax[3];
printf("%p %p\n", pz, pz + 1);  // 相差 3 * sizeof(int)
printf("%p %p\n", pax, pax + 1);  // 相差 sizeof(pointer)
```

> P 421
>
> The rules for assigning one pointer to another are tighter than the rules for numeric types. For example, you can assign an int value to a double variable without using a type conversion, but you can't do the same for pointers to these two type.

```c
int n = 5;
double x = n;
int* pi = &n;
double* pd = pi; // warning: incompatible pointer types initializing 'double *' with an expression of type 'int *'
```

> P 422
>
> These restrictions extend to more complex types.

```c
int* pt;
int (*pa)[3];
int ar1[2][3];
int ar2[3][2];
int** p2;

pt = &ar1[0][0];  // pointer-to-int
pt = ar1[0];  // pointer-to-int
pt = ar1;  // warning: incompatible pointer types assigning to 'int *' from 'int [2][3]'
pa = ar1;  // pointer-to-int[3]
pa = ar2;  // warning: incompatible pointer types assigning to 'int (*)[3]' from 'int [3][2]'
p2 = &pt;  // pointer-to-int*
*p2 = ar2[0];  // pointer-to-int
p2 = ar2  // warning: incompatible pointer types assigning to 'int **' from 'int [3][2]'
```

> P 423
>
> But such assignments no longer are safe when you go to two levels of indirection. For instance, you could do something like this:
>
> const int \*\* pp2;
>
> int \* p1;
>
> const int n = 13;
>
> pp2 = &p1;  // allowed, but const qualifier disregarded
>
> *pp2 = &n;  // valid, both const, but sets p1 to point at n
>
> \*p1 = 10;  // valid, but tries to change const n

```c
const int ** pp2;
int * p1;
pp2 = &p1;  // warning: assigning to 'const int **' from 'int **' discards qualifiers in nested pointer types
```

## Functions and Multidimensional Arrays

> P 424
>
> You can declare a function parameter of this type like this:
>
> void somefunction(int (* pt)[4] );
>
> Alternatively, if (and only if) pt is a formal parameter to a function, you can declare it as follows:
>
> void somefunction(int pt\[\]\[4\] );

和一维数组一样, `int pt[][4]` 可以理解为是 `int (* pt)[4]` 的语法糖.本质上 pt 还是一个 int[4] 的指针.

```c
int sum(int ar[][3], int rows);

int main(void) {
    int powers[2][3] = {{1, 2, 4}, {8, 16, 32}};
    printf("%d\n", sum(powers, 2));
    return 0;
}

int sum(int ar[][3], int rows) {
    int total = 0;
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < 3; j++)  {
            total +=  ar[i][j];
        }
    }
    return total;
}
```

> P 426
>
> Be aware that the following declaration will not work properly:
>
> int sum2(int ar\[\]\[\], int rows);  // faulty declaration

```c
int sum2(int ar[][], int rows);  // error: array has incomplete element type 'int []'
```

> P 426
>
> You can also include a size in the other bracket pair, as shown here, but the compiler ignores it:
>
> int sum2(int ar\[3\]\[4\], int rows);  // valid declaration, 3 ignored
>
> In general, to declare a pointer corresponding to an *N*-dimensional array, you must supply values for all but the leftmost set of brackets:
>
> int sum4d(int ar\[\]\[12\]\[20][30\], int rows);

N 维数组需要确定 N - 1 维的大小, 最大维度的大小用另一个参数传入, 如果函数参数中定义了这个最大维度的大小, 那么编译器也会忽略它.

## Variable-Length Arrays (VLAs)

> P 428
>
> You have to pass the array as a one-dimensional array and have the function calculate where each row starts.

```c
int sum(int* ar, int rows, int cols);

int main(void) {
    int powers[2][3] = {{1, 2, 4}, {8, 16, 32}};
    printf("%d\n", sum(*powers, 2, 3));
    return 0;
}

int sum(int* ar, int rows, int cols) {
    int total = 0;
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++)  {
            // 每行的起始位置 == 行数 * 总列数 == i * cols
            total +=  ar[i * cols + j];
        }
    }
    return total;
}
```

> P 428
>
> This need was the primary impulse for C99 introducing variable-length arrays, which allow you tu use variables when dimensioning an array.

```c
int rows = 2;
int cols = 3;
int powers[rows][cols];  // warning: variable length array used
```

> P 428
>
> Also, you can't initialize them in a declaration.

```c
int rows = 2;
int cols = 3;
int powers[rows][cols] = {0};  // error: variable-sized object may not be initialized
```

> P 428
>
> First, here's how to declare a function with a two-dimensional VLA argument:
>
> int sum2d(int rows, int cols, int ar\[rows\]\[cols\]);  // ar a VLA
>
> Note that the first two parameters (rows and cols) are used as dimensions for declaring the array parameter ar. Because the ar declaration uses rows and cols, they have to be declared before ar in the parameter list.

```c
int sum2d2(int ar[rows][cols], int rows, int cols);  // error: use of undeclared identifier 'rows'  error: use of undeclared identifier 'cols'
```

> P 431
>
> Can you use a const symbolic constant when declaring an array ?
>
> For C90, the answer is no (probably).
>
> For C99/C11, the answer is yes, if the array could other wise be a VLA.

在我的 gcc 中使用 const 声明数组并没有生成一个 VLA 而是一个普通的数组. 因为编译时并没有 warning 而且没有提示我不能初始化. 虽然如此, 但还是遵循 C 语言的标准, 不使用这种方式创建数组.

```c
const int n = 2;
int array[n] = {0};
for (int i = 0; i < n; i++) {
    printf("%d\n", array[i]);
}
```

## Compound Literals

> P 432
>
> For arrays, a compound literal looks like an array initialization list preceded by a type name that is enclosed in parentheses.
>
> Just as you can leave out the array size if you initialize a named array, you can omit it from a compound literal.
>
> One way is to use a pointer to keep track of the location.
>
> Another thing you could do with a compound literal is pass it as an actual argument to a function with a matching formal parameter.
>
> You can extend the technique to two-dimensional arrays, and beyond.
>
> Keep in mind that a compound literal is a means of providing values that are needed only temporarily. It has block scope, a concept convered in Chapter 12.
>
> TODO 需要找个例子证明 block scope 这件事

数组字面量的格式: \(类型)\{初始值\}, 你可以在任意地方用数组字面量代替数组, 就像使用整数, 浮点数之类的字面量一样.

```c
int * p1 = (int [3]){1, 2, 4};
int * p2 = (int []){1, 2, 4};
```

```c
int sum(int ar[], int n);
...
sum((int [3]){1, 2, 4}, 3);
```

```c
int (*p)[3] = (int [2][3]){{1, 2, 4}, {8, 16, 32}};
```

## Review Questions

1. 8 8

   4 4

   0 0

   2 2

2. 4

3. `ref` is the address of `ref[0]`.  

   `ref + 1` is the address of `ref[1]`.

   `ref` 是 int[3] 类型, 是一个 const, 不能自增.

4. 12 16

   12 14

5. 12 16

   12 14

6. `&grid[22][56]` `*(grid + 22) + 56` `grid[22] + 56`

   `&grid[22][0]` `*(grid + 22)` `grid[22]`
   
   `&grid[0][0]` `*grid` `grid[0]`
   
7. `int digits[10];`

   `float rates[6];`

   `int mat[3][5];`
   
   `char * pas[20];`
   
   `char (* pstr)[20];`

8. `int ar[6] = {1, 2, 4, 8, 16, 32};`
  
   `ar[2];`
   
   `int ar[100] = {[99] = -1};`
   
   `int ar[100] = {[5] = 101, [10] = 101, 101, 101, 101}`
   
9. \[0...9\]

10. valid

   invalid
   
   invalid
   
   invalid
   
   valid
   
   invalid, things\[5\] 是 float\[5\] 类型, 是一个 const, 不能被赋值. 
   
   invalid	
   
   valid
   
11. `int ar[800][600];`

12. `void f(double ar[], int rows);` `void f(int rows, double ar[rows]);`

   `void f(short ar[][30], int rows);` `void f(int rows, int cols, short ar[rows][cols]);`
   
   `void f(long ar[][10][15], int rows);` `void f(int rows, int cols, int heights, long ar[rows][cols][heights]);`
   
13. `show((double [4]){8, 3, 9, 2}, 4);`
   `show2((double [2][3]{{8, 3, 9}, {5, 4, 1}}), 2)`

## Programming Exercises

### 1

将 `rain[year][month]` 改成 `*(*(rain + year) + month)`

### 2

```c
#include <stdio.h>

#define SIZE 5

void print_arr(const double arr[], int n);
void copy_arr(double target[], const double source[], int n);
void copy_ptr(double target[], const double source[], int n);
void copy_ptrs(double target[], const double * start, const double * end);

int main(void) {
    /* Write a program that initializes an array-of-double and
       then copies the contents of the array into three other arrays. */
    double source[SIZE] = {1.1, 2.2, 3.3, 4.4, 5.5};
    double target1[SIZE];
    double target2[SIZE];
    double target3[SIZE];

    copy_arr(target1, source, SIZE);
    copy_ptr(target2, source, SIZE);
    copy_ptrs(target3, source, source + SIZE);

    print_arr(target1, SIZE);
    print_arr(target2, SIZE);
    print_arr(target3, SIZE);

    return 0;
}

void copy_arr(double target[], const double source[], int n) {
    /* use a function with array notation. */
    for (int i = 0; i < n; i++) {
        target[i] = source[i];
    }
}

void copy_ptr(double target[], const double source[], int n) {    /* use a function with pointer notation and pointer incrementing. */
    for (int i = 0; i < n; i++) {g
        // 这里也可以使用移动指针的方式.
        // *target = *source;
        // source++;
        // target++;
        *(target + i) = *(source + i);
    }
}

void copy_ptrs(double target[], const double * start, const double * end) {
    /* take as arguments the name of the target, the name of source,
       and a pointer to the element following the last element of the source. */
    // 这里也可以先计算出 n, 再使用 for 循环.
    // int n = end - start;
    // for (int i = 0; i < n; i++) {
    while (start < end) {
        *target = *start;
        start++;
        target++;
    }
}

void print_arr(const double arr[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%.1f ", arr[i]);
    }
    printf("\n");
}
```

### 3

```c
#include <stdio.h>

int max(const int ar[], int n);

int main(void) {
    // 只有一个元素
    printf("%d\n", max((int [1]){1}, 1));  // 1
    // 两个相同的元素
    printf("%d\n", max((int [2]){2, 2}, 2));  // 2
    // 两个不同的元素
    printf("%d\n", max((int [2]){100, 2}, 2));  // 100
    // 多个不同的元素, 最大元素在第一个位置
    printf("%d\n", max((int [4]){100, 20, 30, 99}, 4));  // 100
    // 多个不同的元素, 最大元素在中间位置
    printf("%d\n", max((int [4]){20, 100, 30, 99}, 4));  // 100
    // 多个不同的元素, 最大元素在最后一个位置
    printf("%d\n", max((int [4]){99, 20, 30, 100}, 4));  // 100
    // 测试下负数
    printf("%d\n", max((int [4]){-99, -20, -30, -100}, 4));  // -20

    return 0;
}

int max(const int ar[], int n) {
    /* 返回数组 ar 中的最大值. n > 0. */
    int max_val = ar[0];
    for (int i = 1; i < n; i++) {
        max_val = ar[i] > max_val ? ar[i] : max_val;
    }
    return max_val;
}
```

### 4

```c
#include <stdio.h>

int get_max_idx(const double ar[], int n);

int main(void) {
    // 只有一个元素
    printf("%d\n", get_max_idx((double [1]){1.1}, 1));  // 0
    // 两个相同的元素
    printf("%d\n", get_max_idx((double [2]){2.2, 2.2}, 2));  // 0
    // 两个不同的元素
    printf("%d\n", get_max_idx((double [2]){100.1, 2.2}, 2));  // 0
    // 多个不同的元素, 最大元素在第一个位置
    printf("%d\n", get_max_idx((double [4]){100, 20, 30, 99}, 4));  // 0
    // 多个不同的元素, 最大元素在中间位置
    printf("%d\n", get_max_idx((double [4]){20, 100, 30, 99}, 4));  // 1
    // 多个不同的元素, 最大元素在最后一个位置
    printf("%d\n", get_max_idx((double [4]){99, 20, 30, 100}, 4));  // 3
    // 测试下负数
    printf("%d\n", get_max_idx((double [4]){-99, -20, -30, -100}, 4));  // 1

    return 0;
}

int get_max_idx(const double ar[], int n) {
    /* 返回数组 ar 中的最大值的索引, 如果有多个最大值, 则返回第一个. n > 0. */
    int max_idx = 0;
    for (int i = 1; i < n; i++) {
        max_idx = ar[i] > ar[max_idx] ? i : max_idx;
    }
    return max_idx;
}
```

### 5

```c
#include <stdio.h>

double get_diff_between_largest_and_smallest(const double ar[], int n);

int main(void) {
    // 只有一个元素
    printf("%.2f\n", get_diff_between_largest_and_smallest((double [1]){1.1}, 1));  // 0.00
    // 两个相同的元素
    printf("%.2f\n", get_diff_between_largest_and_smallest((double [2]){2.2, 2.2}, 2));  // 0.00
    // 两个不同的元素
    printf("%.2f\n", get_diff_between_largest_and_smallest((double [2]){100.1, 2.2}, 2));  // 97.90
    // 多个不同的元素, 最大元素在第一个位置
    printf("%.2f\n", get_diff_between_largest_and_smallest((double [4]){100, 20, 30, 99}, 4));  // 80.00
    // 多个不同的元素, 最大元素在中间位置
    printf("%.2f\n", get_diff_between_largest_and_smallest((double [4]){20, 100, 30, 99}, 4));  // 80.00
    // 多个不同的元素, 最大元素在最后一个位置
    printf("%.2f\n", get_diff_between_largest_and_smallest((double [4]){99, 20, 30, 100}, 4));  // 80.00
    // 测试下负数
    printf("%.2f\n", get_diff_between_largest_and_smallest((double [4]){-99, -20, -30, -100}, 4));  // 80.00

    return 0;
}

double get_diff_between_largest_and_smallest(const double ar[], int n) {
    /* 返回最大值和最小值之间的差值. n > 0. */
    double max = ar[0];
    double min = ar[0];
    for (int i = 1; i < n; i++) {
        max = ar[i] > max ? ar[i] : max;
        min = ar[i] < min ? ar[i] : min;
    }
    return max - min;
}
```

### 6

```c
#include <stdio.h>

void reverse(double ar[], int n);
void print_ar(const double ar[], int n);

int main(void) {
    double * ar;

    // 只有一个元素
    ar = (double [1]){1.1};
    reverse(ar, 1);
    print_ar(ar, 1);  // 1.1
    // 奇数个元素
    ar = (double [3]){20.2, 100.1, 30.3};
    reverse(ar, 3);
    print_ar(ar, 3);  // 30.3 100.1 20.2
    // 偶数个元素
    ar = (double [4]){20.2, 100.1, 30.3, 99.9};
    reverse(ar, 4);
    print_ar(ar, 4);  // 99.9 30.3 100.1 20.2

    return 0;
}

void reverse(double ar[], int n) {
    /* 返回最大值和最小值之间的差值. n > 0. */
    int l = 0;
    int r = n - 1;
    double temp;
    while (l < r) {
        temp = ar[r];
        ar[r] = ar[l];
        ar[l] = temp;
        l++;
        r--;
    }
}

void print_ar(const double ar[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%.1f ", ar[i]);
    }
    printf("\n");
}
```

### 7

```c
#include <stdio.h>

#define ROWS 3
#define COLS 4

void copy_arr(double target[], const double source[], int n);
void print_ar(const double ar[][COLS], int n);

int main(void) {
    double ar[ROWS][COLS] = {
        {1, 2, 3, 4},
        {10, 20, 30, 40},
        {100, 200, 300, 400},
    };
    double copied_ar[ROWS][COLS];

    for (int i = 0; i < ROWS; i++) {
        copy_arr(copied_ar[i], ar[i], COLS);
    }

    print_ar(copied_ar, ROWS);

    return 0;
}

void copy_arr(double target[], const double source[], int n) {
    for (int i = 0; i < n; i++) {
        target[i] = source[i];
    }
}

void print_ar(const double ar[][COLS], int n) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < COLS; j++) {
            printf("%.1f ", ar[i][j]);
        }
        printf("\n");
    }
}
```

### 8

```c
#include <stdio.h>

void copy_arr(double target[], const double source[], int n);
void print_ar(const double ar[], int n);

int main(void) {
    double ar[7] = {1, 2, 3, 4, 5, 6, 7};
    double copied_ar[3];

    copy_arr(copied_ar, ar + 2, 3);
    print_ar(copied_ar, 3);  // 3.0 4.0 5.0

    return 0;
}

void copy_arr(double target[], const double source[], int n) {
    for (int i = 0; i < n; i++) {
        target[i] = source[i];
    }
}

void print_ar(const double ar[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%.1f ", ar[i]);
    }
    printf("\n");
}
```

### 9

```c
#include <stdio.h>

#define ROWS 3
#define COLS 5

void copy_ar(int rows, int cols, double target[rows][cols], 
             const double source[rows][cols]);
void print_ar(int rows, int cols, const double ar[rows][cols]);

int main(void) {
    double ar[ROWS][COLS] = {100, 99};
    double copied_ar[ROWS][COLS];

    copy_ar(ROWS, COLS, copied_ar, ar);
    print_ar(ROWS, COLS, copied_ar);

    return 0;
}

void copy_ar(int rows, int cols, double target[rows][cols],
             const double source[rows][cols]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            target[i][j] = source[i][j];
        }
    }
}

void print_ar(int rows, int cols, const double ar[rows][cols]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%5.1f ", ar[i][j]);
        }
        printf("\n");
    }
}
```

### 10

```c
#include <stdio.h>

#define ROWS 4

void merge(int target[], const int source1[], const int source2[], int n);
void print_ar(const int ar[], int n);

int main(void) {
    int ar1[ROWS] = {2, 4, 5, 8};
    int ar2[ROWS] = {1, 0, 4, 6};
    int ar3[ROWS];

    merge(ar3, ar1, ar2, ROWS);
    print_ar(ar3, ROWS);  // 3 4 9 14

    return 0;
}

void merge(int target[], const int source1[], const int source2[], int n) {
    for (int i = 0; i < n; i++) {
        target[i] = source1[i] + source2[i];
    }
}

void print_ar(const int ar[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d ", ar[i]);
    }
    printf("\n");
}
```

### 11

```c
#include <stdio.h>

#define ROWS 3
#define COLS 5

void double_ar(int ar[][COLS], int n);
void print_ar(const int ar[][COLS], int n);

int main(void) {
    int ar[ROWS][COLS] = {
        {2, 4, 5, 8, 10},
        {[4] = 444},
        {[1] = 111},
    };

    print_ar(ar, ROWS);
    double_ar(ar, ROWS);
    print_ar(ar, ROWS);

    return 0;
}

void double_ar(int ar[][COLS], int n) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < COLS; j++) {
            ar[i][j] *= 2;
        }
    }
}

void print_ar(const int ar[][COLS], int n) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < COLS; j++) {
            printf("%3gcd ", ar[i][j]);
        }
        printf("\n");
    }
}
```

### 12

```c
#include <stdio.h>

#define YEARS 5
#define MONTHS 12

void yearly(const float rain[][MONTHS], int n);
void monthly(const float rain[][MONTHS], int n);

int main(void) {
    const float rain[YEARS][MONTHS] = {
        {4.3, 4.3, 4.3, 3.0, 2.0, 1.2, 0.2, 0.2, 0.4, 2.4, 3.5, 6.6},
        {8.5, 8.2, 1.2, 1.6, 2.4, 0.0, 5.2, 0.9, 0.3, 0.9, 1.4, 7.3},
        {9.1, 8.5, 6.7, 4.3, 2.1, 0.8, 0.2, 0.2, 1.1, 2.3, 6.1, 8.4},
        {7.2, 9.9, 8.4, 3.3, 1.2, 0.8, 0.4, 0.0, 0.6, 1.7, 4.3, 6.2},
        {7.6, 5.6, 3.8, 2.8, 3.8, 0.2, 0.0, 0.0, 0.0, 1.3, 2.6, 5.2}
    };

    yearly(rain, YEARS);
    monthly(rain, YEARS);

    return 0;
}

void yearly(const float rain[][MONTHS], int n) {
    float yearly_total = 0;
    float total = 0;

    printf(" YEAR RAINFALL (inches)\n");

    for (int y = 0; y < n; y++) {
        yearly_total = 0;
        for (int m = 0; m < MONTHS; m++) {
            yearly_total += rain[y][m];
        }
        printf("%5d %15.1f\n", 2010 + y, yearly_total);
        total += yearly_total;
    }

    printf("\nThe yearly average is %.1f inches.\n\n", total / n);
}

void monthly(const float rain[][MONTHS], int n) {
    float monthly_total = 0;

    printf("MONTHLY AVERAGES:\n\n");
	printf(" Jan Feb Mar Apr May Jun Jul Aug Sep Oct ");
	printf("Nov Dec\n");

    for (int m = 0; m < MONTHS; m++) {
        monthly_total = 0;
        for (int y = 0; y < n; y++) {
            monthly_total += rain[y][m];
        }
        printf("%4.1f", monthly_total / n);
    }

    printf("\n");
}
```

### 13

```c
// TODO 完成从键盘读取数据到 ar 中.
// 然后完成第十四题, 将所有的函数变成 VLA

#include <stdio.h>

#define ROWS 3
#define COLS 5

double average_row(const double ar[], int n);
double average(const double ar[][COLS], int n);
double max(const double ar[][COLS], int n);

int main(void) {
    const double ar[ROWS][COLS] = {
        {1.1, 2.2, 3.3, 4.4, 5.5},
        {[4] = 44.4},
        {[1] = 11.1},
    };

    for (int i = 0; i < ROWS; i++) {
        printf("%.2f\n", average_row(ar[i], COLS));  // 3.30 8.88 2.22
    }
    printf("%.2f\n", average(ar, ROWS));  // 72.0 / 15 == 4.80
    printf("%.2f\n", max(ar, ROWS));  // 44.40
    return 0;
}

double average_row(const double ar[], int n) {
    double total = 0.0;
    for (int i = 0; i < n; i++) {
        total += ar[i];
    }
    return total / n;
}

double average(const double ar[][COLS], int n) {
    double total = 0.0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < COLS; j++) {
            total += ar[i][j];
        }
    }
    return total / (n * COLS);
}

double max(const double ar[][COLS], int n) {
    double max = ar[0][0];
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < COLS; j++) {
            max = ar[i][j] > max ? ar[i][j] : max;
        }
    }
    return max;
}
```

