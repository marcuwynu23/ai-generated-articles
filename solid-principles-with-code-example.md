# Understanding SOLID Principles in Object-Oriented Design

SOLID principles are a set of five design principles that help developers create more maintainable, flexible, and scalable software systems. Each letter in the SOLID acronym represents one of these principles:

1. **Single Responsibility Principle (SRP)**
2. **Open/Closed Principle (OCP)**
3. **Liskov Substitution Principle (LSP)**
4. **Interface Segregation Principle (ISP)**
5. **Dependency Inversion Principle (DIP)**

Let's dive into each principle and understand its significance along with code examples:

## Single Responsibility Principle (SRP)

A class should have only one reason to change, meaning that it should have only one responsibility or job within the software system.

```python
class FileManager:
    def read_file(self, filename):
        # code to read a file

    def write_file(self, filename, data):
        # code to write to a file

class DataProcessor:
    def process_data(self, data):
        # code to process data

    def save_processed_data(self, data):
        # code to save processed data
```

Here, `FileManager` has the responsibility of handling file operations, and `DataProcessor` has the responsibility of processing data.

## Open/Closed Principle (OCP)

Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification. This means that you should be able to extend the behavior of a module without modifying its source code.

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
        return 3.14 * self.radius ** 2
```

## Liskov Substitution Principle (LSP)

Objects of a superclass should be replaceable with objects of its subclasses without affecting the correctness of the program. In other words, derived classes must be substitutable for their base classes without altering the desirable properties of the program.

```python
class Bird:
    def fly(self):
        pass

class Duck(Bird):
    def fly(self):
        print("Duck flying")

class Ostrich(Bird):
    def fly(self):
        raise NotImplementedError("Ostrich cannot fly")

def bird_fly(bird):
    bird.fly()

duck = Duck()
ostrich = Ostrich()

bird_fly(duck)
bird_fly(ostrich)  # This will raise an error because Ostrich cannot fly
```

## Interface Segregation Principle (ISP)

Clients should not be forced to depend on interfaces they do not use. This principle suggests that you should break interfaces into smaller, specific ones so that clients only need to know about the methods that are relevant to them.

```python
class Printer:
    def print(self, document):
        pass

class Scanner:
    def scan(self, document):
        pass

class Fax:
    def fax(self, document):
        pass

class MultiFunctionDevice(Printer, Scanner, Fax):
    pass

class SimplePrinter(Printer):
    def print(self, document):
        print("Printing:", document)
```

## Dependency Inversion Principle (DIP)

High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions. This principle encourages decoupling between modules by introducing abstractions and relying on them rather than concrete implementations.

```python
class Logger:
    def log(self, message):
        pass

class FileManager:
    def __init__(self, logger):
        self.logger = logger

    def read_file(self, filename):
        self.logger.log(f"Reading file: {filename}")

    def write_file(self, filename, data):
        self.logger.log(f"Writing to file: {filename}")

logger = Logger()
file_manager = FileManager(logger)
file_manager.read_file("example.txt")
file_manager.write_file("example.txt", "Hello, World!")
```

These SOLID principles provide guidelines for writing clean, maintainable, and extensible code, enabling developers to build robust and scalable software systems.

````
