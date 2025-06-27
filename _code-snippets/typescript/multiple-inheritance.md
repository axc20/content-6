# Multiple Inheritance and Mixins

Interfaces can extend other interfaces and classes, which can be useful when composing complex types.

```ts
interface Chimera extends Dog, Lion, Monsterish {}

class MyChimera implements Chimera {
  bark: () => string;
  roar: () => string;
  terrorize(): void {}
}

MyChimera.prototype.bark = Dog.prototype.bark;
MyChimera.prototype.roar = Lion.prototype.roar;
```

A mixin is a function that takes a constructor and returns a new class (the mixin class) that is an extension of the constructor. This allows you to combine behavior from multiple sources into a single class.

```ts
class Dog extends Animal {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}

type Constructor<T = {}> = new (...args: any[]) => T;

function canRollOver<T extends Constructor>(Animal: T) {
  return class extends Animal {
    rollOver() {
      console.log('rolled over');
    }
  };
}

const TrainedDog = canRollOver(Dog);
// the type of rover will be Dog & mixin class,
// which is effectively Dog with a rollOver method
const rover = new TrainedDog('Rover');\
rover.rollOver();
```
