# Programming a golden spiral

About 10 years ago or so, my dad asked me to make a little program to chart a spiral. Not just any spiral, a golden spiral. If you are not familiar with what a golden spiral is, you can read a bit more here: https://en.wikipedia.org/wiki/Golden_spiral. Basically, a golden spiral is a logarithmic spiral that grows at a factor of 1.618. I wrote the program in about a weekend and he has been using it ever since.

Recently my dad asked me to make some updates to it so that he could license the use of the program to other users. The program was originally written in C# and ran on Windows. Rather than making changes to the original program, I thought it would be better to make a brand new version of it for the web. I had been tinkering with Typescript, so I thought this would be a good opportunity to further improve my Typescript skills.

For the last 10 years I had been developing in straight C. All software I developed was for safety critical applications (more on that some other time) and that meant that I could not even use third party libraries or any sort of advance or obscure feature of the language. As you can imagine, jumping to Typescript has been interesting. Now, the concepts are not new, after all I did object oriented programming in C# before and I always borrowed OOP principles in my C programming.

I have been working on this project on and off in my spare time. I started by simply programming the spiral code. The idea is simple. I am creating a class that represents a Spiral. When the class is constructed, a series of parameters are passed in to ultimately configure the shape of the spiral. Things like the starting point (i.e. the tip of the spiral) and the end point (i.e. the center of the spiral). There are other parameters such as the direction of the spin that can also be configured. In the end the class will simply calculate a series of points that when put together by drawing a line between each subsequent point the spiral is drawn.

The first challenge that I ran into when creating this code was handling decimal points. Fortunately, in the OO world there is always a library for everything. And so I found [decimal.js](https://github.com/MikeMcl/decimal.js/) which makes decimal point handling fairly easy:

```Typescript
/**
 * Represents a spiral with a given set of coordinates.
 */
export class Spiral implements IterableIterator<Coordinate> {
  /** PHI - 1.61803 */
  readonly #PHI: Decimal = Decimal.sqrt(5).plus(1).dividedBy(2);

  /** 1/PHI - 0.61803 */
  readonly #PHI_INVERSE = Decimal.div(1, this.#PHI);

  /** PI - 3.141592... */
  readonly #PI = new Decimal(3.141592653589793);

  /** CON - 2*LN(1/PHI)/PI */
  readonly #CON = Decimal.ln(this.#PHI_INVERSE).times(2).dividedBy(this.#PI);

```

The next challenge I ran into was trying to overload constructors in Typescript. I needed this to create instances of my Coordinate class in different ways.
I am not an expert in then language, and there is certainly a way, however it is very convoluted. You basically have to create a single constructor implementation and many constructor signatures. Your constructor implementation has to then be able to deal with all combinations of parameters and this results in a lot of code that checks for the presence or absence of these. Ugly! Instead, I used a technique described by somebody online (I wish I had the like to give credit!) which I found to be a more elegant solution. The solution involves creating static methods with different signatures each returning an instance of the object you are trying to create. This solution reminded me a lot of my C days, so I was sold. The code looks like this:

```Typescript
/**
 * Represents a coordinate on two dimensions.
 */
export class Coordinate {
  /** X axis position */
  private _x: Decimal;
  public get x (): Decimal {
    return this._x;
  }

  public set x (value: Decimal) {
    // Round the incoming value since this will be a coordinate in the screen
    this._x = value.round();
  }

  /** Y axis position */
  private _y: Decimal;
  public get y (): Decimal {
    return this._y;
  }

  public set y (value: Decimal) {
    // Round the incoming value since this will be a coordinate in the screen
    this._y = value.round();
  }

  private constructor () {
    this._x = new Decimal(0);
    this._y = new Decimal(0);
  }

  /**
   * Creates a new coordinate from two values.
   *
   * @param x - X coordinate value
   * @param y - Y coordinate value
   */
  static fromNumbers (x: number, y: number): Coordinate {
    const newInstance = new Coordinate();

    newInstance.x = new Decimal(x);
    newInstance.y = new Decimal(y);

    return newInstance;
  }

  /**
   * Creates a new coordinate using the values from the provided coordinate
   *
   * @param value - Coordinate value to use for the new coordinate
   */
  static fromCoordinate (value: Coordinate): Coordinate {
    const newInstance = new Coordinate();
    newInstance.x = new Decimal(value.x);
    newInstance.y = new Decimal(value.y);
    return newInstance;
  }

  /**
   * Creates a new coordinate with default values (zeroes)
   */
  static fromDefault (): Coordinate {
    return new Coordinate();
  }
```
In the end, a new spiral is simply created by creating a new class and the list of points can be retrieved by iterating over it:

```Typescript
    const newSpiral: Spiral = new Spiral(Coordinate.fromNumbers(600, 500), Coordinate.fromNumbers(100, 500));
    for (const coordinate of newSpiral) {
      ctx.lineTo(coordinate.x.toNumber(), coordinate.y.toNumber());
    }

```

After this, I started thinking about setting up the backend of the application. The idea is to have a simple backend which receives the requests to draw a spiral, generates the spiral and then returns the spiral back to the front end.

Fortunately for me, I had been playing with a backend API using [Node.js](https://nodejs.org/) before I started this project. Particularly with [Express.js](http://expressjs.com/). I wanted a simple framework to build the application and did not want to tie myself to a technology that might have not been widely accepted. Express.js looked like a good balance.





Authenticating requests

dealing with libraries in java script that don't have type definitions.

Drawing in the backend.
