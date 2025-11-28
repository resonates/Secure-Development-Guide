# Python面向对象编程

## 什么是面向对象编程？

面向对象编程（Object-Oriented Programming，简称OOP）是一种编程范式，它将数据（属性）和操作数据的方法（函数）封装在一个称为"对象"的单元中。面向对象编程的核心思想是使用类（Class）来定义对象的蓝图，并通过实例化类来创建具体的对象。

Python是一种面向对象的编程语言，它支持类、继承、多态和封装等面向对象编程的核心概念。

## 类与对象

### 类的定义

类是对象的蓝图或模板，它定义了对象的属性和方法。在Python中，使用`class`关键字来定义类：

```python
class Person:
    # 类属性
    species = "Homo sapiens"
    
    # 初始化方法（构造函数）
    def __init__(self, name, age):
        # 实例属性
        self.name = name
        self.age = age
    
    # 实例方法
    def greet(self):
        return f"Hello, my name is {self.name} and I am {self.age} years old."
    
    # 类方法
    @classmethod
    def from_birth_year(cls, name, birth_year):
        current_year = 2023
        return cls(name, current_year - birth_year)
    
    # 静态方法
    @staticmethod
    def is_adult(age):
        return age >= 18
```

### 对象的创建与使用

对象是类的实例，通过调用类的构造函数来创建：

```python
# 创建对象
person1 = Person("Alice", 30)
person2 = Person("Bob", 25)

# 访问实例属性
print(person1.name)  # 输出: Alice
print(person1.age)   # 输出: 30

# 调用实例方法
print(person1.greet())  # 输出: Hello, my name is Alice and I am 30 years old.

# 访问类属性
print(Person.species)  # 输出: Homo sapiens
print(person1.species)  # 输出: Homo sapiens

# 调用类方法
person3 = Person.from_birth_year("Charlie", 1990)
print(person3.age)  # 输出: 33

# 调用静态方法
print(Person.is_adult(20))  # 输出: True
```

## 继承

继承是面向对象编程中实现代码复用的重要机制，它允许一个类（子类）继承另一个类（父类）的属性和方法。

### 基本继承

```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return "Some generic sound"

# Dog类继承自Animal类
class Dog(Animal):
    def speak(self):
        return f"{self.name} says Woof!"

# Cat类继承自Animal类
class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"

# 创建子类对象
rover = Dog("Rover")
whiskers = Cat("Whiskers")

# 调用继承的方法
print(rover.speak())  # 输出: Rover says Woof!
print(whiskers.speak())  # 输出: Whiskers says Meow!
```

### 方法重写

子类可以重写父类的方法，以提供自己的实现：

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
    
    def calculate_bonus(self):
        return self.salary * 0.1

class Manager(Employee):
    def calculate_bonus(self):
        # 重写父类方法，经理获得更高的奖金
        return self.salary * 0.2

# 创建对象
emp = Employee("Alice", 50000)
manager = Manager("Bob", 80000)

# 调用重写的方法
print(emp.calculate_bonus())  # 输出: 5000.0
print(manager.calculate_bonus())  # 输出: 16000.0
```

### super()函数

`super()`函数用于调用父类的方法，通常用于在子类中扩展父类的功能：

```python
class Parent:
    def __init__(self, name):
        self.name = name
    
    def greet(self):
        return f"Hello, I am {self.name}"

class Child(Parent):
    def __init__(self, name, age):
        # 调用父类的初始化方法
        super().__init__(name)
        self.age = age
    
    def greet(self):
        # 扩展父类的greet方法
        parent_greeting = super().greet()
        return f"{parent_greeting} and I am {self.age} years old"

child = Child("Charlie", 5)
print(child.greet())  # 输出: Hello, I am Charlie and I am 5 years old
```

### 多重继承

Python支持多重继承，一个类可以继承自多个父类：

```python
class Flyable:
    def fly(self):
        return "Flying..."

class Swimmable:
    def swim(self):
        return "Swimming..."

# Duck类继承自Flyable和Swimmable
class Duck(Flyable, Swimmable):
    pass

duck = Duck()
print(duck.fly())  # 输出: Flying...
print(duck.swim())  # 输出: Swimming...
```

**注意**：多重继承可能导致菱形继承问题（钻石问题），Python通过C3线性化算法（方法解析顺序，MRO）来解决这个问题。

## 多态

多态是指不同类型的对象可以响应相同的方法调用，但可能有不同的行为。在Python中，多态是通过动态类型系统实现的。

```python
class Shape:
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        import math
        return math.pi * self.radius ** 2

# 多态示例：不同类型的对象响应相同的方法
shapes = [Rectangle(5, 4), Circle(3)]

for shape in shapes:
    print(f"Area: {shape.area()}")
    # 输出:
    # Area: 20
    # Area: 28.274333882308138
```

## 封装

封装是指将数据（属性）和操作数据的方法（函数）封装在一个单元中，并控制对其的访问。在Python中，可以通过命名约定和属性装饰器来实现封装。

### 私有属性和方法

Python没有真正的私有属性或方法，但有一个命名约定：以下划线开头的属性或方法被认为是私有的，不应该在类外部直接访问：

```python
class Car:
    def __init__(self, brand, model, year):
        self.brand = brand  # 公共属性
        self._model = model  # 保护属性（约定）
        self.__year = year  # 私有属性（名称修饰）
    
    def get_year(self):
        return self.__year
    
    def _start_engine(self):  # 保护方法
        return "Engine started"
    
    def __private_method(self):  # 私有方法
        return "This is a private method"

car = Car("Toyota", "Corolla", 2020)

print(car.brand)  # 可以访问，输出: Toyota
print(car._model)  # 可以访问（但不推荐），输出: Corolla
# print(car.__year)  # 错误：无法直接访问私有属性

print(car.get_year())  # 输出: 2020
print(car._start_engine())  # 可以调用（但不推荐），输出: Engine started
# print(car.__private_method())  # 错误：无法直接调用私有方法

# 注意：Python的名称修饰使得私有属性可以通过特殊名称访问
print(car._Car__year)  # 输出: 2020
print(car._Car__private_method())  # 输出: This is a private method
```

### 属性装饰器

Python提供了`@property`、`@<attribute>.setter`和`@<attribute>.deleter`装饰器来定义属性的getter、setter和deleter方法：

```python
class Person:
    def __init__(self, name, age):
        self._name = name
        self._age = age
    
    @property
    def name(self):
        return self._name
    
    @property
    def age(self):
        return self._age
    
    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("Age cannot be negative")
        self._age = value
    
    @age.deleter
    def age(self):
        del self._age

person = Person("Alice", 30)

# 使用property装饰器的getter
print(person.name)  # 输出: Alice
print(person.age)  # 输出: 30

# 使用setter设置值
person.age = 31
print(person.age)  # 输出: 31

# 尝试设置无效值
# person.age = -5  # 抛出ValueError异常

# 使用deleter删除属性
del person.age
# print(person.age)  # 抛出AttributeError异常
```

## 特殊方法

Python中的特殊方法（也称为魔术方法或双下划线方法）是以双下划线开头和结尾的方法，它们用于实现对象的特殊行为。

### 常用的特殊方法

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    # 字符串表示
    def __str__(self):
        return f"Vector({self.x}, {self.y})"
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"
    
    # 算术运算符重载
    def __add__(self, other):
        if isinstance(other, Vector):
            return Vector(self.x + other.x, self.y + other.y)
        raise TypeError("Can only add Vector to Vector")
    
    def __sub__(self, other):
        if isinstance(other, Vector):
            return Vector(self.x - other.x, self.y - other.y)
        raise TypeError("Can only subtract Vector from Vector")
    
    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)
    
    # 比较运算符重载
    def __eq__(self, other):
        if isinstance(other, Vector):
            return self.x == other.x and self.y == other.y
        return False
    
    # 容器行为
    def __len__(self):
        # 返回向量的长度（欧几里得范数）
        import math
        return math.sqrt(self.x**2 + self.y**2)
    
    def __getitem__(self, key):
        if key == 0:
            return self.x
        elif key == 1:
            return self.y
        raise IndexError("Vector index out of range")

# 使用特殊方法
v1 = Vector(1, 2)
v2 = Vector(3, 4)

# 字符串表示
print(str(v1))  # 输出: Vector(1, 2)
print(repr(v1))  # 输出: Vector(1, 2)

# 算术运算
v3 = v1 + v2
print(v3)  # 输出: Vector(4, 6)

v4 = v2 - v1
print(v4)  # 输出: Vector(2, 2)

v5 = v1 * 2
print(v5)  # 输出: Vector(2, 4)

# 比较
print(v1 == v2)  # 输出: False
print(v1 == Vector(1, 2))  # 输出: True

# 容器行为
print(len(v1))  # 输出: 2.23606797749979
print(v1[0])  # 输出: 1
print(v1[1])  # 输出: 2
```

## 设计模式与OOP

设计模式是在软件开发中解决常见问题的可重用方案。许多设计模式利用面向对象编程的特性来实现。

### 单例模式

确保一个类只有一个实例，并提供一个全局访问点。

```python
class Singleton:
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# 测试单例模式
s1 = Singleton()
s2 = Singleton()

print(s1 is s2)  # 输出: True
```

### 工厂模式

提供一个创建对象的接口，让子类决定实例化哪个类。

```python
class Animal:
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

class AnimalFactory:
    @staticmethod
    def create_animal(animal_type):
        if animal_type == "dog":
            return Dog()
        elif animal_type == "cat":
            return Cat()
        else:
            raise ValueError(f"Unknown animal type: {animal_type}")

# 使用工厂模式
dog = AnimalFactory.create_animal("dog")
cat = AnimalFactory.create_animal("cat")

print(dog.speak())  # 输出: Woof!
print(cat.speak())  # 输出: Meow!
```

## 总结

面向对象编程是Python中一个强大的编程范式，它通过类和对象的概念，帮助开发者更好地组织和管理代码。理解并熟练应用类、继承、多态和封装等核心概念，以及掌握特殊方法和常用设计模式，可以帮助开发者编写更加优雅、高效和可维护的Python代码。