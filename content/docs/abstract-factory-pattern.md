---
title: "Abstract Factory Pattern"
date: 2021-12-16T22:02:43+09:00
draft: true
---
## Abstract Factory Pattern

The abstract factory is one of the most widely used pattern, almost certain to be found in any large object-oriented projects. As the name suggests, it gives us a "factory" that lets us create a system of related objects in their abstract form. Here's an illustration on how it works:

[insert illustration-1]

In the above illustration, the factory produces notebook computers that look exactly the same on the outside, but the components varies depending on the price. The low-end model uses cheaper and slower processor, less memory, etc. The point of this abstract factory pattern is that:

- We use the output of the factory in exactly the same way.
- The product of the factory may do things differently, and produce different result, even that we operate them in exactly the same way. 
- The reason why the product behaves differently is that internally they are different set of components ("implemented" differently to follow different requirements, in this case material cost).
- The user of the product doesn't need to learn or do things differently in order to take use different models.

The advantage of using this pattern are mostly extensibility: 

- It is easier to upgrade or replace components being produced by the factory without having to update the client of the factory as well.
- It is easier to distribute the implementation effort to multiple independent teams. 

This really is a very powerful abstraction that gets used and, unfortunately, abused a lot. In many instances, this pattern causes a bloated overdone system structure. It can be good when used in moderation for some appropriate problems, but this approach tend to encourage us to have "fun" constructing excessive class structure.

### Example Problem: Ecosystem With Animals

To see how the abstract factory pattern being applied to model a relatively simple problem, let's devise an example involving animals:

[insert illustration-2]

- In this system we have three types of animal: `Dog`, `Bird`, and `Fish`. Each of them have different way to move around.
- These three animals move in different mediums: `Dog` runs on `Earth`, `Bird` flies in the `Air`, and `Fish` swims in the `Water`. The mediums might have some effect on animal's movement.
- Let's say that these animals live in an `Ecosystem` where a natural disaster may occur.
- When there is a natural disaster, let's say that the animals would move away from the place where the disaster happens, until they are at a safe distance. The meaning of "safe" is of course relative to each animal. A distance that is safe for a `Bird` might not be safe for a `Dog`.

Knowing about this kind of problem, a seasoned object-oriented programmer would jump in and eagerly imagine a structure of tens of classes to model the system, all in the name of future extensibility (which most of the time, would never have happened because the system got obsolete). 

Before going deeper into how usually we approach this problem in object-oriented imperative way, let's specify some more details to simplify the meaning of animals' "movement":

- Ecosystem where the animals live is represented as a 2-dimensional cartesian plane.
- Animals move when their position (`x` and `y` coordinate) changes. 
- Animals move in units of movement. Different animal move in different unit (that represent speed).
- The unit of movement of each animal is influenced by the medium in which the animal moves.
- The position of the disaster is constant.  

### Ecosystem With Animals: Object-oriented Imperative Approach

Knowing that there are 3 types of animals involved, the most reasonable object-oriented approach would be to make a hierarchy of animal classes like such:

[insert illustration 3]

The three animal classes are conforming to a contract (for example: `interface` in `Java`) such that:

1. All animals have current position.
2. All animals are able to run away from a certain position (useful when there is a disaster).
3. All animals know when they are safe.

Coded into a `Java` `interface`, it would look like this:

```java
/**
 * Representing an animal in an ecosystem.
 */
public interface Animal {

    /**
     * Given a position, this animal is able to move themselves away from it.
     *
     * @param position Position to run away from.
     */
    void runAwayFrom(Position position);

    /**
     * Given a position, this animal is able to decide whether they are within a safe distance from it.
     *
     * @param position Position to evaluate.
     * @return whether this animal is safe from the given position.
     */
    boolean isSafeFrom(Position position);

    /**
     * Getting the current position of this animal.
     */
    Position getPosition();

    /**
     * Setting the position of this animal.
     *
     * @param position New position for the animal
     */
    void setPosition(Position position);
}
```
Let's define `Position` as follows:

```java
public class Position {

    private int x;
    private int y;

    public Position(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public void setX(int x) {
        this.x = x;
    }

    public int getY() {
        return y;
    }

    public void setY(int y) {
        this.y = y;
    }

    /**
     * "Direction" of this position relative to another position.
     *
     * @param anotherPosition The other position for reference.
     * @return
     */
    public Position relativeTo(Position anotherPosition) {
        return new Position(x - anotherPosition.x, y - anotherPosition.y);
    }

    /**
     * Only meaningful when called for the result of {@link #relativeTo(Position)}, to create a unit vector.
     *
     * @return
     */
    public Position direction() {
        return new Position(x/Math.abs(x), y/Math.abs(y));
    }

    /**
     * Utility function to find the distance between to position.
     */
    public static int distance(Position position1, Position position2) {
        int deltax = position1.getX() - position2.getX();
        int deltay = position1.getY() - position2.getY();
        return (deltax * deltax) + (deltay * deltay);
    }
}
```

There are some utility functions that are defined here to make the movement calculation easier later. The purpose of these functions will become clear later.

Because animals move in a medium, we can define the medium  as an `interface` too:

```java
/**
 * Medium in/on which animals move.
 */
public interface Medium {

    /**
     * Representing how a medium may influence the way an animal moves through it.
     *
     * @param animal animal that moves
     */
    void letAnimalMovesThrough(Animal animal);

}
```

Now that we have the class for medium, we can associate it with `Animal` by adding following method in the `Animal` interface:

```java
public interface Animal {
    
    // other methods...

    /**
     * Initialize the medium in/on which this animal moves.
     *
     * @param clazz class of the medium to be associated with this animal.
     */
    public <T extends Medium> void initMedium(Class<T> clazz);

    /**
     * 
     * @return the medium associated with this animal.
     */
    Medium getMedium();
}
```

Because there are properties and operations shared by all animals (namely, all have positions, all moves through a medium), we can write an `abstract class` for the parent class of all animals:

```java
public abstract class AbstractAnimal implements Animal {

    /**
     * The current position of the animal.
     */
    protected Position position;

    /**
     * The medium on/in which the animal moves.
     */
    protected Medium medium;

    @Override
    public Position getPosition() {
        return position;
    }

    @Override
    public void setPosition(Position position) {
        this.position = position;
    }

    @Override
    public Medium getMedium() {
        return medium;
    }

    @Override
    public <T extends Medium> void initMedium(Class<T> clazz)
            throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        // NOTES: because of the use of reflection here, exceptions can be thrown.
        // The method definition in the interface will need to be adjusted as well.
        this.medium = clazz.getDeclaredConstructor().newInstance();
    }

    @Override
    public void runAwayFrom(Position position) {
        // Only defines that the animal run away from position by moving through a medium.
        // Some specific animal logic should be specified in the subclass.
        this.medium.letAnimalMovesThrough(this);
    }
    
}
```
Having the `abstract class` as a base class for all animals, we can start to write the details of each animal. Here's the `Bird`:

```java
public class Bird extends AbstractAnimal {

    @Override
    public void runAwayFrom(Position position) {
        // calls super, because that's where we model the "influence" of the medium on animal's movement
        super.runAwayFrom(position);
        flapWingsAway(position);
    }

    private void flapWingsAway(Position fromPosition) {
        Position direction = this.position.relativeTo(fromPosition).direction();
        // basically move several units (as defined by the magic number) in the specified direction.
        this.position.setX(position.getX() + (Constants.MAGIC_NUMBER_UNIT_PER_WINGS_FLAP * direction.getX()));
        this.position.setY(position.getY() + (Constants.MAGIC_NUMBER_UNIT_PER_WINGS_FLAP * direction.getY()));
    }

    @Override
    public boolean isSafeFrom(Position position) {
        // To make it simple, let's just say that a bird feels safe if they are 100 unit away from a position.
        return Position.distance(this.position, position) > 100;
    }

}
```

To make things simpler, let's just make the `Dog` moves in a similar way, except for the constant that multiplies the direction in which the animal moves, and the distance where the animal feels safe:

```java
public class Dog extends AbstractAnimal {

    @Override
    public void runAwayFrom(Position position) {
        super.runAwayFrom(position);
        jumpAway(position);
    }

    private void jumpAway(Position fromPosition) {
        Position direction = this.position.relativeTo(fromPosition).direction();
        this.position.setX(position.getX() + (Constants.MAGIC_NUMBER_UNIT_PER_JUMP * direction.getX()));
        this.position.setY(position.getY() + (Constants.MAGIC_NUMBER_UNIT_PER_JUMP * direction.getY()));
    }

    @Override
    public boolean isSafeFrom(Position position) {
        return Position.distance(this.position, position) > 200;
    }

}
```

Similarly, for `Fish` implementation:

```java
public class Fish extends AbstractAnimal {

    @Override
    public void runAwayFrom(Position position) {
        super.runAwayFrom(position);
        swimAway(position);
    }

    private void swimAway(Position fromPosition) {
        Position direction = this.position.relativeTo(fromPosition).direction();
        this.position.setX(position.getX() + (Constants.MAGIC_NUMBER_UNIT_PER_STROKE * direction.getX()));
        this.position.setY(position.getY() + (Constants.MAGIC_NUMBER_UNIT_PER_STROKE * direction.getY()));
    }

    @Override
    public boolean isSafeFrom(Position position) {
        return Position.distance(this.position, position) > 75;
    }

}
```
Now that we have all objects involved in the ecosystem ready, let's see how these objects can be used to model animals running from a disaster. Following snippet is a test code that shows how we use factories to instantiate animals:

```java
public class EcosystemTest {

    static final Position birdInitialPosition = new Position(12, 12);
    static final Position fishInitialPosition = new Position(12, 8);
    static final Position dogInitialPosition = new Position(8, 12);

    static final Position disasterLocation = new Position(10, 10);

    @Test
    void testImperative() throws InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
        ImperativeEcosystemJava ecosystem = new ImperativeEcosystemJava(List.of(
                // shows that we use a Bird "Factory" to instantiate a bird with an initiali position
                BirdFactory.create(birdInitialPosition),
                // Fish "Factory" for a fish
                FishFactory.create(fishInitialPosition),
                // and Dog "Factory" for a dog
                DogFactory.create(dogInitialPosition)
        ));
        // now let the disaster begin.
        ecosystem.onNaturalDisaster(disasterLocation);
        
        // make sure the animals are safe now
        List<Animal> animals = ecosystem.getAnimals();
        assertPositions(animals.stream().map(Animal::getPosition).toList());
    }

    void assertPositions(List<Position> positions) {
        assertEquals(positions.get(0).getX(), 28);
        assertEquals(positions.get(0).getY(), 26);
        assertEquals(positions.get(1).getX(), 22);
        assertEquals(positions.get(1).getY(), -2);
        assertEquals(positions.get(2).getX(), 24);
        assertEquals(positions.get(2).getY(), -4);
    }
}
```

Before defining the factories to be used like above, let's define what an ecosystem is:

```java
/**
 * Ecosystem is the "client" of Abstract Factory
 */
public class ImperativeEcosystemJava {

    /**
     * All animals in the ecosystem
     */
    List<Animal> animals;

    public ImperativeEcosystemJava(List<Animal> animals) {
        this.animals = animals;
    }

    /**
     * When a natural disaster happens, all animals run away until they're all safe.
     *
     * @param whereItHappens the position where the disaster happens
     */
    public void onNaturalDisaster(Position whereItHappens) {
        // all animals run away from where the disaster happens until they're safe
        boolean allSafe;
        do {
            allSafe = animalsRunFrom(whereItHappens);
        } while (!allSafe);
    }

    private boolean animalsRunFrom(Position whereItHappens) {
        boolean allSafe = true;
        for (Animal animal : animals) {
            if (!animal.isSafeFrom(whereItHappens)) {
                animal.runAwayFrom(whereItHappens);
                allSafe = false;
            }
        }
        return allSafe;
    }

    public List<Animal> getAnimals() {
        return animals;
    }
}
```

Here's how we may implement the factory for `Bird` class:

```java
public class BirdFactory {

    public static Animal create(Position initialPosition)
            throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Animal ret = new Bird();
        ret.initMedium(Air.class);
        ret.setPosition(initialPosition);
        return ret;
    }

}
```

As you can see, the factory matches the created instance of `Bird` with its associated `Air` instance as the medium in which the bird moves. This is one of the most important task of the factory: construct the object with all its related objects. Similarly, the `Dog` factory also associate it with `Earth` as medium, as shown below:

```java
public class DogFactory {

    public static Animal create(Position initialPosition)
            throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Animal ret = new Dog();
        ret.initMedium(Earth.class);
        ret.setPosition(initialPosition);
        return ret;
    }

}
```
Because there are too much similarities between above two factories, it's preferable to generalize them into one factory such as:

```java
public class AnimalFactory {

    public static <T extends Animal, U extends Medium> Animal createAnimal(Class<T> animalClass, Class<U> mediumClass, Position initialPosition)
            throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Animal ret = animalClass.getDeclaredConstructor().newInstance();
        ret.initMedium(mediumClass);
        ret.setPosition(initialPosition);
        return ret;
    }

}
```

With this generic version of the factory, we can define the factory for `Bird` in a simpler way:

```java
public class BirdFactory {

    public static Animal create(Position initialPosition)
            throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        return AnimalFactory.createAnimal(Bird.class, Air.class, initialPosition);
    }
}
```

Similarly, for `Dog` and `Fish`:

```java
public class DogFactory {

    public static Animal create(Position initialPosition)
            throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        return AnimalFactory.createAnimal(Dog.class, Earth.class, initialPosition);
    }

}

public class FishFactory {

    public static Animal create(Position initialPosition)
            throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        return AnimalFactory.createAnimal(Fish.class, Water.class, initialPosition);
    }

}
```

### Ecosystem With Animals: Functional Approach

The idea of constructing "Abstract Factory" in a functional way doesn't make much sense. However, we can still ask: is there a way to approach the same problem as it is solved by "Abstract Factory" pattern but purely as a functional composition.

To answer that question, we first need to identify what is it exactly the problem, and rephrase it in such a way that is easier to approach from functional perspective. Unlike with object-oriented reasoning, we usually identify what "actors" or "objects" are involved in the problem to be solved. With functional reasoning, we start with identifying what is it that the whole system is computing. What kind of output we want from what kind of input we can give?  


### Commentary

<!--stackedit_data:
eyJoaXN0b3J5IjpbOTY3NjA1MDM0XX0=
-->
