转载自: [用c实现C++类（八股）](https://blog.csdn.net/xiaolan7777777/article/details/145067123)

# 用纯C实现C++类

在 C 语言中，虽然没有内建的面向对象编程（[OOP](https://so.csdn.net/so/search?q=OOP&spm=1001.2101.3001.7020)）特性（如封装、继承、多态），但通过一些编程技巧，我们仍然可以模拟实现这些概念。下面将用通俗易懂的方式，逐步介绍如何在 C 中实现封装、继承和多态。

#### 1\. 封装（Encapsulation）

**封装**是指将数据和操作数据的函数绑定在一起，隐藏内部实现细节，只暴露必要的接口。在 C 中，我们可以通过 `struct` 和相关的函数来实现封装。

假设我们要创建一个“矩形”对象，包含宽度和高度，并提供计算面积的功能。

**步骤：**

1.  **定义结构体**（隐藏内部细节）
2.  **提供创建和操作该结构体的函数**

**实现：**

```c
// Rectangle.h
#ifndef RECTANGLE_H
#define RECTANGLE_H

typedef struct Rectangle Rectangle;

// 创建矩形
Rectangle* Rectangle_create(double width, double height);

// 销毁矩形
void Rectangle_destroy(Rectangle* rect);

// 计算面积
double Rectangle_getArea(const Rectangle* rect);

#endif // RECTANGLE_H








// Rectangle.c
#include <stdlib.h>
#include "Rectangle.h"

// 定义结构体（隐藏在 .c 文件中）
struct Rectangle {
    double width;
    double height;
};

// 创建矩形
Rectangle* Rectangle_create(double width, double height) {
    Rectangle* rect = (Rectangle*)malloc(sizeof(Rectangle));
    if (rect != NULL) {
        rect->width = width;
        rect->height = height;
    }
    return rect;
}

// 销毁矩形
void Rectangle_destroy(Rectangle* rect) {
    free(rect);
}

// 计算面积
double Rectangle_getArea(const Rectangle* rect) {
    if (rect == NULL) return 0.0;
    return rect->width * rect->height;
}







// main.c
#include <stdio.h>
#include "Rectangle.h"

int main() {
    Rectangle* rect = Rectangle_create(5.0, 3.0);
    if (rect != NULL) {
        printf("面积: %.2f\n", Rectangle_getArea(rect));
        Rectangle_destroy(rect);
    }
    return 0;
}
```

**说明：**

*   **隐藏实现**：`Rectangle` 的具体结构体定义在 `Rectangle.c` 中，外部无法直接访问其成员变量。
*   **接口函数**：通过 `Rectangle_create`、`Rectangle_destroy` 和 `Rectangle_getArea` 提供对 `Rectangle` 对象的操作。

#### 2\. 继承（Inheritance）

**继承**允许一个“子类”拥有“父类”的属性和行为。在 C 中，我们可以通过在子[结构体](https://so.csdn.net/so/search?q=%E7%BB%93%E6%9E%84%E4%BD%93&spm=1001.2101.3001.7020)中包含父结构体来模拟继承。

**基类“形状”和子类“矩形”和“圆形”**

**步骤：**

1.  **定义基类结构体**，包含一个指向函数的指针（模拟虚函数表）。
2.  **定义子类结构体**，在其中包含基类结构体。
3.  **实现子类的功能**。

**实现：**

```c
// Shape.h
#ifndef SHAPE_H
#define SHAPE_H

typedef struct Shape Shape;

// 虚函数表
typedef struct {
    double (*getArea)(const Shape* self);
    void (*destroy)(Shape* self);
} ShapeVTable;

// 基类结构体
struct Shape {
    ShapeVTable* vtable;
};

// 基类接口函数
double Shape_getArea(const Shape* self);
void Shape_destroy(Shape* self);

#endif // SHAPE_H









// Shape.c
#include "Shape.h"

double Shape_getArea(const Shape* self) {
    if (self && self->vtable && self->vtable->getArea) {
        return self->vtable->getArea(self);
    }
    return 0.0;
}

void Shape_destroy(Shape* self) {
    if (self && self->vtable && self->vtable->destroy) {
        self->vtable->destroy(self);
    }
}







// Rectangle.h
#ifndef RECTANGLE_H
#define RECTANGLE_H

#include "Shape.h"

typedef struct {
    Shape base; // 继承自 Shape
    double width;
    double height;
} Rectangle;

// 创建矩形
Shape* Rectangle_create(double width, double height);

#endif // RECTANGLE_H






// Rectangle.c
#include <stdlib.h>
#include "Rectangle.h"

// 矩形的虚函数实现
double Rectangle_getArea(const Shape* self) {
    const Rectangle* rect = (const Rectangle*)self;
    return rect->width * rect->height;
}

void Rectangle_destroy_impl(Shape* self) {
    free(self);
}

// 定义矩形的虚函数表
ShapeVTable rectangle_vtable = {
    .getArea = Rectangle_getArea,
    .destroy = Rectangle_destroy_impl
};

// 创建矩形
Shape* Rectangle_create(double width, double height) {
    Rectangle* rect = (Rectangle*)malloc(sizeof(Rectangle));
    if (rect != NULL) {
        rect->base.vtable = &rectangle_vtable;
        rect->width = width;
        rect->height = height;
    }
    return (Shape*)rect;
}








// Circle.h
#ifndef CIRCLE_H
#define CIRCLE_H

#include "Shape.h"

typedef struct {
    Shape base; // 继承自 Shape
    double radius;
} Circle;

// 创建圆形
Shape* Circle_create(double radius);

#endif // CIRCLE_H




// Circle.c
#include <stdlib.h>
#include "Circle.h"
#include <math.h>

// 圆形的虚函数实现
double Circle_getArea(const Shape* self) {
    const Circle* circle = (const Circle*)self;
    return M_PI * circle->radius * circle->radius;
}

void Circle_destroy_impl(Shape* self) {
    free(self);
}

// 定义圆形的虚函数表
ShapeVTable circle_vtable = {
    .getArea = Circle_getArea,
    .destroy = Circle_destroy_impl
};

// 创建圆形
Shape* Circle_create(double radius) {
    Circle* circle = (Circle*)malloc(sizeof(Circle));
    if (circle != NULL) {
        circle->base.vtable = &circle_vtable;
        circle->radius = radius;
    }
    return (Shape*)circle;
}



// main.c
#include <stdio.h>
#include "Shape.h"
#include "Rectangle.h"
#include "Circle.h"

int main() {
    Shape* shapes[2];
    shapes[0] = Rectangle_create(5.0, 3.0); // 创建矩形
    shapes[1] = Circle_create(2.0);         // 创建圆形

    for (int i = 0; i < 2; ++i) {
        printf("图形 %d 的面积: %.2f\n", i + 1, Shape_getArea(shapes[i]));
        Shape_destroy(shapes[i]);
    }

    return 0;
}
```

**说明：**

*   **基类 `Shape`**：包含一个虚函数表 `vtable`，用于指向具体实现的函数。
*   子类 `Rectangle` 和 `Circle`
    *   包含 `Shape` 作为第一个成员，实现“继承”。
    *   定义自己的虚函数（如 `getArea` 和 `destroy_impl`）。
    *   分别创建自己的虚函数表，并在创建时将 `vtable` 指向自己的表。
*   **多态**：在 `main` 中，通过基类指针 `Shape*` 调用 `getArea`，根据实际对象类型（矩形或圆形）执行不同的函数。

#### 3\. 多态（Polymorphism）

**多态**允许不同类型的对象通过相同的接口调用不同的实现。在上面的继承示例中，我们已经部分实现了多态。下面进一步解释多态的实现。

在 `Shape` 基类中定义了虚函数表 `ShapeVTable`，包含 `getArea` 和 `destroy` 函数指针。每个子类（如 `Rectangle` 和 `Circle`）都提供了自己的实现，并在创建时将 `vtable` 指向自己的函数表。

**如何工作：**

1.  **统一接口**：所有形状都通过 `Shape*` 指针进行操作。
2.  **具体实现**：不同的形状（矩形、圆形）有各自的 `getArea` 实现。
3.  **调用时自动选择**：根据对象的实际类型，调用相应的 `getArea` 函数。

**示例解释：**

```c
for (int i = 0; i < 2; ++i) {
    printf("图形 %d 的面积: %.2f\n", i + 1, Shape_getArea(shapes[i]));
    Shape_destroy(shapes[i]);
}
```

*   `Shape_getArea(shapes[i])` 会根据 `shapes[i]` 的 `vtable` 指向不同的 `getArea` 实现，自动计算出矩形或圆形的面积。
