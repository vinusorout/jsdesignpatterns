# JS Design Patterns

## SOLID Design Principles

### Single Responsibility Principle

* A class should have single primary responsibility.

### Open Closed Principle

* Objects are open for extension, closed for modification
* For example you modify an existing class by making changes and it was alredy tested, this is not consider as good practise 
* Instead use inheritance 

```js
let Color = Object.freeze({
  red: 'red',
  green: 'green',
  blue: 'blue'
});

let Size = Object.freeze({
  small: 'small',
  medium: 'medium',
  large: 'large',
  yuge: 'yuge'
});

class Product
{
  constructor(name, color, size)
  {
    this.name = name;
    this.color = color;
    this.size = size;
  }
}

// Keep adding new filter to this ProductFilter class is violation of OCP
class ProductFilter
{
  filterByColor(products, color)
  {
    return products.filter(p => p.color === color);
  }

  filterBySize(products, size)
  {
    return products.filter(p => p.size === size);
  }
}

let apple = new Product('Apple', Color.green, Size.small);
let tree  = new Product('Tree', Color.green, Size.large);
let house = new Product('House', Color.blue, Size.large);

let products = [apple, tree, house];

// let pf = new ProductFilter();
// console.log(`Green products (old):`);
// for (let p of pf.filterByColor(products, Color.green))
//   console.log(` * ${p.name} is green`);

// ↑↑↑ BEFORE

// ↓↓↓ AFTER

// general interface for a specification
class ColorSpecification
{
  constructor(color)
  {
    this.color = color;
  }

  isSatisfied(item)
  {
    return item.color === this.color;
  }
}

class SizeSpecification {
  constructor(size)
  {
    this.size = size;
  }

  isSatisfied(item)
  {
    return item.size === this.size;
  }
}

class BetterFilter
{
  filter(items, spec)
  {
    return items.filter(x => spec.isSatisfied(x));
  }
}

// specification combinator
class AndSpecification
{
  constructor(...specs)
  {
    this.specs = specs;
  }

  isSatisfied(item)
  {
    return this.specs.every(x => x.isSatisfied(item));
  }
}

let bf = new BetterFilter();
console.log(`Green products (new):`);
for (let p of bf.filter(products,
  new ColorSpecification(Color.green)))
{
  console.log(` * ${p.name} is green`);
}

console.log(`Large products:`);
for (let p of bf.filter(products,
  new SizeSpecification(Size.large)))
{
  console.log(` * ${p.name} is large`);
}

console.log(`Large and green products:`);
let spec = new AndSpecification(
  new ColorSpecification(Color.green),
  new SizeSpecification(Size.large)
);
for (let p of bf.filter(products, spec))
  console.log(` * ${p.name} is large and green`);
```

### Liskov Substitution Principle

* IF a function takes a base class then it should be able to take the derived class as argument, without breaking the functionality in whatsoever. Means if you are creating a drived class then it should not break the earlier function.

### Interface Segregation Principle

* Use interfaces

### Dependency Inversion Principle

* Define a relationship between low level module and high level module.
* High level classes should not use low level classes, instead they should use abstract classes or interfaces
* DIP is about decoupling.
* Coupling indicates how dependent modules are on the inner workings of each other.


## Design Patterns:

* Design principles are typically split into three categories:
* *Creational Patterns* Deals with creation
* *Structural Patterns* Concerned with the structure eg class members
* *Behavioral Patterns* They are all different; no central theme

### Builder Pattern (Categorised as Creational)

* Some objects are simple and can be created in a single initializer call
* Other objects require a lot of ceremony to create
* Having an object with 10 initializer arguments is not productive
* Instead, opt for piecewise construction
* Builder provides an API for constructing an object step by step.

```js
class Tag
{
  static get indentSize() { return 2; }

  constructor(name='', text='')
  {
    this.name = name;
    this.text = text;
    this.children = [];
  }

  toStringImpl(indent)
  {
    let html = [];
    let i = ' '.repeat(indent * Tag.indentSize);
    html.push(`${i}<${this.name}>\n`);
    if (this.text.length > 0)
    {
      html.push(' '.repeat(Tag.indentSize * (indent+1)));
      html.push(this.text);
      html.push('\n');
    }

    for (let child of this.children)
      html.push(child.toStringImpl(indent+1));

    html.push(`${i}</${this.name}>\n`);
    return html.join();
  }

  toString()
  {
    return this.toStringImpl(0);
  }

  static create(name)
  {
    return new HtmlBuilder(name);
  }
}

class HtmlBuilder
{
  constructor(rootName)
  {
    this.root = new Tag(rootName);
    this.rootName = rootName;
  }

  // non-fluent
  addChild(childName, childText)
  {
    let child = new Tag(childName, childText);
    this.root.children.push(child);
  }

  // fluent
  // In software engineering, a fluent interface is an object-oriented API whose design relies extensively on method chaining
  addChildFluent(childName, childText)
  {
    let child = new Tag(childName, childText);
    this.root.children.push(child);
    return this;
  }

  toString()
  {
    return this.root.toString();
  }

  clear()
  {
    this.root = new Tag(this.rootName);
  }

  build()
  {
    return this.root;
  }
}

// just a single paragraph using string concatenation
const hello = 'hello';
let html = [];
html.push('<p>');
html.push(hello);
html.push('</p>');
console.log(html.join());

// a list with 2 words in it
const words = ['hello', 'world'];
html = [];
html.push('<ul>\n');
for (let word of words)
  html.push(`  <li>${word}</li>\n`);
html.push('</ul>');
console.log(html.join());

// ordinary non-fluent builder
//let builder = new HtmlBuilder('ul');
let builder = Tag.create('ul');
for (let word of words)
  builder.addChild('li', word);
//console.log(builder.toString());
let tag = builder.build();
console.log(tag.toString());

// fluent builder
builder.clear();
builder
  .addChildFluent('li', 'foo')
  .addChildFluent('li', 'bar')
  .addChildFluent('li', 'baz');
console.log(builder.toString());

```


### Factory Pattern (Categorised as Creational)

* Object creation logic becomes too convoluted(extremely complex and difficult to follow)
* Initializer is not descriptive
  * Name is always __init__
  * Cannot overload with same sets of arguments with different names
  * Can turn into 'optional parameter hell'
* Wholesale object creation(non-piecewise, unlike Builder) can be outsourced to
  * A seprate method(Factory Method)
  * That may exists in a seperate class (Factory)
  * Can create hierarchy of factories with Abstract Factory

```js
CoordinateSystem = {
  CARTESIAN : 0,
  POLAR : 1
};

class Point
{
  constructor(x, y)
  {
    this.x = x;
    this.y = y;
  }

  // constructor(a, b, cs=CoordinateSystem.CARTESIAN)
  // {
  //   switch (cs)
  //   {
  //     case CoordinateSystem.CARTESIAN:
  //       this.x = a;
  //       this.y = b;
  //       break;
  //     case CoordinateSystem.POLAR:
  //       this.x = a * Math.cos(b);
  //       this.y = a * Math.sin(b);
  //       break;
  //   }
  //
  //   // steps to add a new system
  //   // 1. augment CoordinateSystem
  //   // 2. change ctor
  // }

  static newCartesianPoint(x, y)
  {
    return new Point(x, y);
  }

  static newPolarPoint(rho, theta)
  {
    return new Point(rho*Math.cos(theta), rho*Math.sin(theta));
  }

  static get factory()
  {
    return new PointFactory();
  }
}

class PointFactory
{
  // not necessarily static
  newCartesianPoint(x, y)
  {
    return new Point(x, y);
  }

  static newPolarPoint(rho, theta)
  {
    return new Point(rho*Math.cos(theta), rho*Math.sin(theta));
  }
}

let p1 = new Point(2, 3, CoordinateSystem.CARTESIAN);
console.log(p1);
// Point → PointFactory
let p2 = PointFactory.newPolarPoint(5, Math.PI/2);
console.log(p2);

// this line will not work if newCartesianPoint is static!
let p3 = Point.factory.newCartesianPoint(2, 3);
console.log(p3);
```



* Abstract Factory, create a number of objects

```js
const readline = require('readline');
let rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});
const async = require('async');

class HotDrink
{
  consume() {}
}

class Tea extends HotDrink
{
  consume() {
    console.log('This tea is nice with lemon!');
  }
}

class Coffee extends HotDrink
{
  consume()
  {
    console.log(`This coffee is delicious!`);
  }
}

class HotDrinkFactory
{
  prepare(amount) { /* abstract */ }
}

class TeaFactory extends HotDrinkFactory
{
  prepare(amount) {
    console.log(`Grind some beans, boil water, pour ${amount}ml`);
    return new Coffee();
  }
}

class CoffeeFactory extends HotDrinkFactory
{
  prepare(amount) {
    console.log(`Put in tea bag, boil water, pour ${amount}ml`);
    return new Tea();
  }
}

let AvailableDrink = Object.freeze({
  coffee: CoffeeFactory,
  tea: TeaFactory
});

class HotDrinkMachine
{
  constructor()
  {
    this.factories = {};
    for (let drink in AvailableDrink)
    {
      this.factories[drink] = new AvailableDrink[drink]();
    }
  }

  makeDrink(type)
  {
    switch (type)
    {
      case 'tea':
        return new TeaFactory().prepare(200);
      case 'coffee':
        return new CoffeeFactory().prepare(50);
      default:
        throw new Error(`Don't know how to make ${type}`);
    }
  }

  interact(consumer)
  {
    rl.question('Please specify drink and amount ' +
      '(e.g., tea 50): ', answer => {
      let parts = answer.split(' ');
      let name = parts[0];
      let amount = parseInt(parts[1]);
      let d = this.factories[name].prepare(amount);
      rl.close();
      consumer(d);
    });
  }
}

let machine = new HotDrinkMachine();
// rl.question('which drink? ', function(answer)
// {
//   let drink = machine.makeDrink(answer);
//   drink.consume();
//
//   rl.close();
// });
machine.interact(
  function (drink) {
    drink.consume();
  }
);
```



### Prototype (Creational)

* Complicated objects (eg cars) arent designed from scratch
  * They reiterate existing designs
* An existing (partially or fully constructed) design is a Prototype
* We make a copy(clone) the prototype and customize it
  * Requires 'deep copy' support
* We make the cloning convenient (eg, via a Factory)


### Singleton (Creational)

```js
class Singleton {
  constructor()
  {
    const instance = this.constructor.instance;
    if (instance) {
      return instance;
    }

    this.constructor.instance = this;
  }

  foo() {
    console.log('Doing something...')
  }
}

let s1 = new Singleton();
let s2 = new Singleton();
console.log('Are they identical? ' + (s1 === s2));
s1.foo();
```

### Adapter

* Aconstruct which adapts an existing interface X tp conform to the required interface Y.

*With no caching*

```js
class Point
{
  constructor(x, y)
  {
    this.x = x;
    this.y = y;
  }

  toString()
  {
    return `(${this.x}, ${this.y})`;
  }
}

class Line
{
  constructor(start, end)
  {
    this.start = start;
    this.end = end;
  }

  toString()
  {
    return `${this.start.toString()}→${this.end.toString()}`;
  }
}

class VectorObject extends Array {}

class VectorRectangle extends VectorObject
{
  constructor(x, y, width, height)
  {
    super();
    this.push(new Line(new Point(x,y), new Point(x+width, y) ));
    this.push(new Line(new Point(x+width,y), new Point(x+width, y+height) ));
    this.push(new Line(new Point(x,y), new Point(x, y+height) ));
    this.push(new Line(new Point(x,y+height), new Point(x+width, y+height) ));this.push
  }
}

// ↑↑↑ this is your API ↑↑↑

// ↓↓↓ this is what you have to work with ↓↓↓

let vectorObjects = [
  new VectorRectangle(1, 1, 10, 10),
  new VectorRectangle(3, 3, 6, 6)
];

let drawPoint = function(point)
{
  process.stdout.write('.');
};

// ↓↓↓ to draw our vector objects, we need an adapter ↓↓↓

class LineToPointAdapter extends Array
{
  constructor(line)
  {
    super();
    console.log(`${LineToPointAdapter.count++}: Generating ` +
      `points for line ${line.toString()} (no caching)`);

    let left = Math.min(line.start.x, line.end.x);
    let right = Math.max(line.start.x, line.end.x);
    let top = Math.min(line.start.y, line.end.y);
    let bottom = Math.max(line.start.y, line.end.y);

    if (right - left === 0)
    {
      for (let y = top; y <= bottom; ++y)
      {
        this.push(new Point(left, y));
      }
    }
    else if (line.end.y - line.start.y === 0)
    {
      for (let x = left; x <= right; ++x)
      {
        this.push(new Point(x, top));
      }
    }
  }
}
LineToPointAdapter.count = 0;

let drawPoints = function()
{
  for (let vo of vectorObjects)
    for (let line of vo)
    {
      let adapter = new LineToPointAdapter(line);
      adapter.forEach(drawPoint);
    }
};

drawPoints();
drawPoints();
```



*With Caching*

* Sometimes the Adaptor pattern casues you to genrate temporary repeated objects as in above scenario. drawPoints calling two times genartes line objects in adaptor again.
* So if you have already generated instance of objects you already encounter before, you should save keep them in a cache and then use it from the cache.

```js
class Point
{
  constructor(x, y)
  {
    this.x = x;
    this.y = y;
  }

  toString()
  {
    return `(${this.x}, ${this.y})`;
  }
}

class Line
{
  constructor(start, end)
  {
    this.start = start;
    this.end = end;
  }

  toString()
  {
    return `${this.start.toString()}→${this.end.toString()}`;
  }
}

class VectorObject extends Array {}

class VectorRectangle extends VectorObject
{
  constructor(x, y, width, height)
  {
    super();
    this.push(new Line(new Point(x,y), new Point(x+width, y) ));
    this.push(new Line(new Point(x+width,y), new Point(x+width, y+height) ));
    this.push(new Line(new Point(x,y), new Point(x, y+height) ));
    this.push(new Line(new Point(x,y+height), new Point(x+width, y+height) ));this.push
  }
}

// ↑↑↑ this is your API ↑↑↑

// ↓↓↓ this is what you have to work with ↓↓↓
String.prototype.hashCode = function(){
  if (Array.prototype.reduce){
    return this.split("").reduce(function(a,b){
      a=((a<<5)-a)+b.charCodeAt(0);return a&a},0);
  }
  let hash = 0;
  if (this.length === 0) return hash;
  for (let i = 0; i < this.length; i++) {
    const character = this.charCodeAt(i);
    hash  = ((hash<<5)-hash)+character;
    hash = hash & hash; // Convert to 32-bit integer
  }
  return hash;
};

class LineToPointAdapter extends Array
{
  constructor(line)
  {
    super();
  // this will not create the Adaptor for a already generated line
    this.hash = JSON.stringify(line).hashCode();
    if (LineToPointAdapter.cache[this.hash])
      return; // we already have it

    console.log(`${LineToPointAdapter.count++}: Generating ` +
      `points for line ${line.toString()} (with caching)`);

    let points = [];

    let left = Math.min(line.start.x, line.end.x);
    let right = Math.max(line.start.x, line.end.x);
    let top = Math.min(line.start.y, line.end.y);
    let bottom = Math.max(line.start.y, line.end.y);

    if (right - left === 0)
    {
      for (let y = top; y <= bottom; ++y)
      {
        points.push(new Point(left, y));
      }
    }
    else if (line.end.y - line.start.y === 0)
    {
      for (let x = left; x <= right; ++x)
      {
        points.push(new Point(x, top));
      }
    }

    LineToPointAdapter.cache[this.hash] = points;
  }

  get items()
  {
    return LineToPointAdapter.cache[this.hash];
  }
}
LineToPointAdapter.count = 0;
LineToPointAdapter.cache = {};

let vectorObjects = [
  new VectorRectangle(1, 1, 10, 10),
  new VectorRectangle(3, 3, 6, 6)
];

let drawPoint = function(point)
{
  process.stdout.write('.');
};

let draw = function()
{
  for (let vo of vectorObjects)
  {
    for (let line of vo)
    {
      let adapter = new LineToPointAdapter(line);
      adapter.items.forEach(drawPoint);
    }
  }
};

draw();
draw();
```

### Bridge

* Decouple abstraction from implementation
* A mechanism that decouples an interface(hierarchy) from an implementation(hierarchy).
* It is just a way to connect two hirerchy together.
* Reminder: JS has duck typing, so definitions of interfaces are not strictly necessary.

```js
class VectorRenderer
{
  renderCircle(radius)
  {
    console.log(`Drawing a circle of radius ${radius}`);
  }
}

class RasterRenderer
{
  renderCircle(radius)
  {
    console.log(`Drawing pixels for circle of radius ${radius}`);
  }
}

class Shape
{
  // Bridge: we can use any renderer here so it connect shape with renderer
  constructor(renderer) // *renderer* consider as a interface
  {
    this.renderer = renderer;
  }
}

class Circle extends Shape
{
  constructor(renderer, radius) {
    super(renderer);
    this.radius = radius;
  }

  draw()
  {
    this.renderer.renderCircle(this.radius);
  }

  resize(factor)
  {
    this.radius *= factor;
  }
}

// imagine Square, Triangle
// different ways of rendering: vector, raster
// we don't want a cartesian product of these

let raster = new RasterRenderer();
let vector = new VectorRenderer();
let circle = new Circle(vector, 5);
circle.draw();
circle.resize(2);
circle.draw();
```

### Decorator

* Want to augment an object with additional functionality
* DO not want to rewrite or alter existing code (OCP)
* Want to keep new functionality seprate (SRP)
* Two options:
  * Inherit from required object (if possible)
  * Build a Decorator, which simply refrences the decorated objects

```js
// We can add color to Shape, but it will violate the SOLID principle
// So create a decorator class that will extends the shape and implement the color property
class Shape {}

class Circle extends Shape
{
  constructor(radius=0)
  {
    super();
    this.radius = radius;
  }

  resize(factor)
  {
    this.radius *= factor;
  }

  toString()
  {
    return `A circle of radius ${this.radius}`;
  }
}

class Square extends Shape
{
  constructor(side=0)
  {
    super();
    this.side = side;
  }

  toString()
  {
    return `A square with side ${this.side}`;
  }
}

// we don't want ColoredSquare, ColoredCircle, etc.
class ColoredShape extends Shape
{
  constructor(shape, color)
  {
    super();
    this.shape = shape;
    this.color = color;
  }

  toString()
  {
    return `${this.shape.toString()} ` +
      `has the color ${this.color}`;
  }
}

class TransparentShape extends Shape
{
  constructor(shape, transparency)
  {
    super();
    this.shape = shape;
    this.transparency = transparency;
  }

  toString()
  {
    return `${this.shape.toString()} has ` +
      `${this.transparency * 100.0}% transparency`;
  }
}

let circle = new Circle(2);
console.log(circle.toString());

// Use Decorator, to color the circle 
let redCircle = new ColoredShape(circle, 'red');
console.log(redCircle.toString());

// IMPORTANT
// impossible: redHalfCircle is not a Circle
// redHalfCircle.resize(2);
// To use the actual shape methods like 'resize'
// redHalfCircle.shape.resize(2);

let redHalfCircle = new TransparentShape(redCircle, 0.5);
console.log(redHalfCircle.toString());


```


### Facade

* Provides a simple, easy to understand/user interface over a large and sophisticated body of code.
* Balancing complexity and presentation/usability
* Typical eg Home,
  * Many subsystems (electrical, sanitation)
  * Complex internal structure (eg floor layers)
  * End User is not exposed to internals
* Same with software!
  * Many systems working to provide flexibility, but...
  * API consumers want it to 'just work'

### Flyweight

* A space optimization technique that lets us use less memory by storing externally the data associated with similar objects.
* Avoid redundancy when storing data
* Eg MMORPG
  * Plenty of users with identical first/last names
  * No sense in storing same first/last name over and over again
  * Store a list of names and references to them
* Eg, old or italic text formatting
  * Dont want each char to have a formating character
  * Operate on ranges (eg line number, start/end positions)

```js
class FormattedText
{
  constructor(plainText)
  {
    this.plainText = plainText;
    this.caps = new Array(plainText.length).map(
      function() { return false; }
    );
  }

  capitalize(start, end)
  {
    for (let i = start; i <= end; ++i)
      this.caps[i] = true;
  }

  toString()
  {
    let buffer = [];
    for (let i in this.plainText)
    {
      let c = this.plainText[i];
      buffer.push(this.caps[i] ? c.toUpperCase() : c);
    }
    return buffer.join('');
  }
}

// this would work better as a nested class
class TextRange
{
  constructor(start, end)
  {
    this.start = start;
    this.end = end;
    this.capitalize = false;
    // other formatting options here
  }

  covers(position)
  {
    return position >= this.start &&
      position <= this.end;
  }
}

class BetterFormattedText
{
  constructor(plainText)
  {
    this.plainText = plainText;
    this.formatting = [];
  }

  getRange(start, end)
  {
    let range = new TextRange(start, end);
    this.formatting.push(range);
    return range;
  }

  toString()
  {
    let buffer = [];
    for (let i in this.plainText)
    {
      let c = this.plainText[i];
      for (let range of this.formatting) {
        if (range.covers(i) && range.capitalize)
          c = c.toUpperCase();
      }
      buffer.push(c);
    }
    return buffer.join('');
  }
}

const text = 'This is a brave new world';
let ft = new FormattedText(text);
ft.capitalize(10, 15);
console.log(ft.toString());

let bft = new BetterFormattedText(text);
bft.getRange(16, 19).capitalize = true;
console.log(bft.toString());
```

### Chain of Responsibility

* A chain of components who all get a chance to process a command or a query, optionally having default processing implementation and an ability to terminate the processing chain.

**Method Chain**
```js
class Creature {
  constructor(name, attack, defense) {
    this.name = name;
    this.attack = attack;
    this.defense = defense;
  }

  toString() {
    return `${this.name} (${this.attack}/${this.defense})`;
  }
}

class CreatureModifier
{
  constructor(creature)
  {
    this.creature = creature;
    this.next = null;
  }

  add(modifier)
  {
    if (this.next) this.next.add(modifier);
    else this.next = modifier;
  }

  handle()
  {
    if (this.next) this.next.handle();
  }
}

class NoBonusesModifier extends CreatureModifier
{
  constructor(creature)
  {
    super(creature);
  }

  handle()
  {
    console.log('No bonuses for you!');
  }
}

class DoubleAttackModifier extends CreatureModifier
{
  constructor(creature)
  {
    super(creature);
  }

  handle()
  {
    console.log(`Doubling ${this.creature.name}'s attack`);
    this.creature.attack *= 2;
    super.handle();
  }
}

class IncreaseDefenseModifier extends CreatureModifier
{
  constructor(creature)
  {
    super(creature);
  }

  handle() {
    if (this.creature.attack <= 2)
    {
      console.log(`Increasing ${this.creature.name}'s defense`);
      this.creature.defense++;
    }
    super.handle();
  }
}

let goblin = new Creature('Goblin', 1, 1);
console.log(goblin.toString());

let root = new CreatureModifier(goblin);

//root.add(new NoBonusesModifier(goblin));

root.add(new DoubleAttackModifier(goblin));
//root.add(new DoubleAttackModifier(goblin));

root.add(new IncreaseDefenseModifier(goblin));

// eventually...
root.handle();
console.log(goblin.toString());
```


**Event Broker Chain**

```js
class Event
{
  constructor()
  {
    this.handlers = new Map();
    this.count = 0;
  }

  subscribe(handler)
  {
    this.handlers.set(++this.count, handler);
    return this.count;
  }

  unsubscribe(idx)
  {
    this.handlers.delete(idx);
  }

  fire(sender, args)
  {
    this.handlers.forEach(function (v, k)
    {
      v(sender, args);
    });
  }
}

let WhatToQuery = Object.freeze({
  'attack': 1,
  'defense': 2
});

class Query
{
  constructor(creatureName, whatToQuery, value)
  {
    this.creatureName = creatureName;
    this.whatToQuery = whatToQuery;
    this.value = value;
  }
}

class Game
{
  constructor()
  {
    this.queries = new Event();
  }

  performQuery(sender, query)
  {
    this.queries.fire(sender, query);
  }
}

class Creature
{
  constructor(game, name, attack, defense)
  {
    this.game = game;
    this.name = name;
    this.initial_attack = attack;
    this.initial_defense = defense;
  }

  get attack()
  {
    let q = new Query(this.name, WhatToQuery.attack,
      this.initial_attack);
    this.game.performQuery(this, q);
    return q.value;
  }

  get defense()
  {
    let q = new Query(this.name, WhatToQuery.defense,
      this.initial_defense);
    this.game.performQuery(this, q);
    return q.value;
  }

  toString()
  {
    return `${this.name}: (${this.attack}/${this.defense})`;
  }
}

class CreatureModifier
{
  constructor(game, creature)
  {
    this.game = game;
    this.creature = creature;
    this.token = game.queries.subscribe(
      this.handle.bind(this)
    );
  }

  handle(sender, query)
  {
    // implement in inheritors
  }

  dispose()
  {
    game.queries.unsubscribe(this.token);
  }
}

class DoubleAttackModifier extends CreatureModifier
{
  constructor(game, creature)
  {
    super(game, creature);
  }

  handle(sender, query) {
    if (query.creatureName === this.creature.name &&
        query.whatToQuery === WhatToQuery.attack)
    {
      query.value *= 2;
    }
  }
}

class IncreaseDefenseModifier extends CreatureModifier
{
  constructor(game, creature)
  {
    super(game, creature);
  }

  handle(sender, query)
  {
    if (query.creatureName === this.creature.name &&
        query.whatToQuery === WhatToQuery.defense)
    {
      query.value += 2;
    }
  }
}

let game = new Game();
let goblin = new Creature(game, 'Strong Goblin', 2, 2);
console.log(goblin.toString());

let dam = new DoubleAttackModifier(game, goblin);
console.log(goblin.toString());

let idm = new IncreaseDefenseModifier(game, goblin);
console.log(goblin.toString());
idm.dispose();

dam.dispose();
console.log(goblin.toString());
```


### Command

* Uses: GUI Commands, multi-level undo/redo, macro recording etc
* An object which represents an instruction to perform a particular action. Contains all the information necessary for the action to be taken.

```js
class BankAccount
{
  constructor(balance=0)
  {
    this.balance = balance;
  }

  deposit(amount)
  {
    this.balance += amount;
    console.log(
      `Deposited ${amount}, balance is now ${this.balance}`
    );
  }

  withdraw(amount)
  {
    if (this.balance - amount >= BankAccount.overdraftLimit)
    {
      this.balance -= amount;
      console.log(
        `Withdrew ${amount}, balance is now ${this.balance}`
      );
      return true;
    }
    return false;
  }

  toString()
  {
    return `Balance: ${this.balance}`;
  }
}
BankAccount.overdraftLimit = -500;

let Action = Object.freeze({
  'deposit': 1,
  'withdraw': 2
});

class BankAccountCommand
{
  constructor(account, action, amount)
  {
    this.account = account;
    this.action = action;
    this.amount = amount;
    this.succeeded = false;
  }

  call()
  {
    switch (this.action)
    {
      case Action.deposit:
        this.account.deposit(this.amount);
        this.succeeded = true;
        break;
      case Action.withdraw:
        this.succeeded = this.account.withdraw(this.amount);
        break;
    }
  }

  undo()
  {
    if (!this.succeeded) return;
    switch (this.action)
    {
      case Action.deposit:
        this.account.withdraw(this.amount);
        break;
      case Action.withdraw:
        this.account.deposit(this.amount);
        break;
    }
  }
}

let ba = new BankAccount(100);

let cmd = new BankAccountCommand(ba, Action.deposit, 50);
cmd.call();
console.log(ba.toString());

console.log('Performing undo:');
cmd.undo();
console.log(ba.toString());
```

### Interpreter

* A component that processes structured text data. Does so by turning it into separate lexical tokens(lexing) and then interpreting sequences of said tokens (paring).

```js
class Integer
{
  constructor(value)
  {
    this.value = value;
  }
}

let Operation = Object.freeze({
  addition: 0,
  subtraction: 1
});

class BinaryOperation
{
  constructor()
  {
    this.type = null;
    this.left = null;
    this.right = null;
  }

  get value()
  {
    switch (this.type)
    {
      case Operation.addition:
        return this.left.value + this.right.value;
      case Operation.subtraction:
        return this.left.value - this.right.value;
    }
    return 0;
  }
}

let TokenType = Object.freeze({
  integer: 0,
  plus: 1,
  minus: 2,
  lparen: 3,
  rparen: 4
});

class Token
{
  constructor(type, text)
  {
    this.type = type;
    this.text = text;
  }

  toString()
  {
    return `\`${this.text}\``;
  }
}

function lex(input)
{
  let result = [];

  for (let i = 0; i < input.length; ++i)
  {
    switch (input[i])
    {
      case '+':
        result.push(new Token(TokenType.plus, '+'));
        break;
      case '-':
        result.push(new Token(TokenType.minus, '-'));
        break;
      case '(':
        result.push(new Token(TokenType.lparen, '('));
        break;
      case ')':
        result.push(new Token(TokenType.rparen, ')'));
        break;
      default:
        let buffer = [input[i]];
        for (let j = i+1; j < input.length; ++j)
        {
          if ('0123456789'.includes(input[j]))
          {
            buffer.push(input[j]);
            ++i;
          } else {
            result.push(new Token(TokenType.integer,
              buffer.join('')));
            break;
          }
        }
        break;
    }
  }

  return result;
}

function parse(tokens)
{
  let result = new BinaryOperation();
  let haveLHS = false;

  for (let i = 0; i < tokens.length; ++i) {
    let token = tokens[i];

    switch (token.type) {
      case TokenType.integer:
        let integer = new Integer(parseInt(token.text));
        if (!haveLHS) {
          result.left = integer;
          haveLHS = true;
        } else {
          result.right = integer;
        }
        break;
      case TokenType.plus:
        result.type = Operation.addition;
        break;
      case TokenType.minus:
        result.type = Operation.subtraction;
        break;
      case TokenType.lparen:
        let j = i;
        for (; j < tokens.length; ++j)
          if (tokens[j].type === TokenType.rparen)
            break; // found it!
        // process subexpression
        let subexpression = tokens.slice(i + 1, j);
        let element = parse(subexpression);
        if (!haveLHS) {
          result.left = element;
          haveLHS = true;
        } else result.right = element;
        i = j; // advance
        break;
    }
  }
  return result;
}

let input = "(13+4)-(12+1)";
let tokens = lex(input);
console.log(tokens.join('  '));

let parsed = parse(tokens);
console.log(`${input} = ${parsed.value}`);
```




