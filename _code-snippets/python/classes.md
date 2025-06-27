# Classes

- OOP
  - Write classes that represent real-world things and situations
  - Create objects based on these classes
  - When you write a class, you define the general behavior that a whole category of objects can have
- Instantiation
  - Making an object from a class
  - You'll work with instances of a class, specify the kind of info that can be stored in instances, define actions that can be taken with these instances, and write classes that extend the functionality of existing classes so similar classes can share code efficiently.
- Method
  - Function that is part of a class
  - `__init__()` method is a special method that Python runs automatically when a new instance is created
  - Any variable prefixed with self is available to every method in the class and is also accessible through any instance created from the class. Variables that are accessible through instances like this are called attributes.
- Inheritance
  - When one class inherits from another, it takes on the attributes and methods of the first class. The original class is called the parent class (superclass), and the new class is the child class (subclass).
  - Child class can inherit any or all of the attributes of its parent class, but it's also free to define new attributes and methods of its own.

## Example 1

```python
# classes/Dog.py
class Dog:
  """A simple attempt to model a dog."""

  def __init__(self, name, age):
    """Initialize name and age attributes."""
    self.name = name
    self.age = age

  def sit(self):
    """Simulate a dog sitting in response to a command."""
    print(f"{self.name} is now sitting.")

  def roll_over(self):
    """Simulate rolling over in response to a command."""
    print(f"{self.name} rolled over!")

# -----
from classes.Dog import Dog

my_dog = Dog('Peon', 5)

print(f"Dog name: {my_dog.name}")
print(f"Dog age: {my_dog.age}")

my_dog.sit()
my_dog.roll_over()
```

## Example 2

```python
# classes/Car.py
class Car:
  """A simple attempt to represent a car."""

  def __init__(self, make, model, year):
    """Initialize attributes to describe a car."""
    self.make = make
    self.model = model
    self.year = year
    self.odometer_reading = 0

  def get_descriptive_name(self):
    """Return a neatly formatted descriptive name."""
    long_name = f"{self.year} {self.make} {self.model}"
    return long_name.title()

  def read_odometer(self)
    """Print a statement showing the car's mileage."""
    print(f"This care has {self.odometer_reading} miles on it.")

  def update_odometer(self, mileage):
    """
    Set the odometer reading to the given value.
    Reject the change if it attempts to roll the odometer back.
    """
    if mileage >= self.odometer_reading:
      self.odometer_reading = mileage
    else:
      print("You can't roll back an odometer!")

  def increment_odometer(self, miles):
    """Add the given amount to the odometer reading."""
    self.odometer_reading += miles

class Battery:
  """A simple attempt to model a battery for an electric car."""

  def __init__(self, battery_size=75):
    """Initialize the battery's attributes."""
    self.battery_size = battery_size

  def describe_battery(self):
    """Print a statement describing the battery size."""
    print(f"This car has a {self.battery_size}-kWh battery.")

  def get_range(self):
    """Print a statment about the range this battery provides."""
    if self.battery_size == 75:
      range = 260
    elif self.battery_size == 100:
      range = 315

    print(f"This car can go about {range} miles on a full charge.")

class ElectricCar(Car):
  """Represent aspects of a car, specific to electric vehicles."""

  def __init__(self, make, model, year):
    """
    Initialize attributes of the parent class.
    Then initialize attributes specific to an electric car.
    """
    super().__init__(make, model, year)
    self.battery = Battery()

  def fill_gas_tank(self):
    """Electric cars don't have gas tanks."""
    print("This car doesn't need a gas tank!")

# -----
# a module is a file ending in .py that contains code you wnat to import
from classes.Car import Car, ElectricCar

my_new_car = Car('audi', 'a4', 2019)
my_tesla = ElectricCar('tesla', 'model s', 2019)

print(my_new_car.get_descriptive_name())

my_new_car.read_odometer()
my_new_car.update_odometer(23_500)
my_new_car.update_odometer(15)
my_new_car.read_odometer()
my_new_car.increment_odometer(1500)
my_new_car.read_odometer()

print(my_tesla.get_descriptive_name())

my_tesla.battery.describe_battery()
my_tesla.battery.get_range()
my_tesla.update_odometer(89)
my_tesla.read_odometer()
my_tesla.fill_gas_tank()
```
