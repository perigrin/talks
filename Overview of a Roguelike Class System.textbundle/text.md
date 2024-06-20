# Playing Games
## With Metamodern Perl
---
## Object oriented programming is about designing a model of your problem domain
---
## Most tutorials cover the basics: classes, methods, inheritance, polymorphism...
---
## ...But make up a crappy problem domain based
---
# Modern Perl (4e)

```perl

 package Cat {
        use Moose;

        has 'name', is => 'ro', isa => 'Str';
        has 'age',  is => 'ro', isa => 'Int';
        has 'diet', is => 'rw';
    }

    my $fat = Cat->new( name => 'Fatty',
                        age  => 8,
                        diet => 'Sea Treats' );

    say $fat->name, ' eats ', $fat->diet;

    $fat->diet( 'Low Sodium Kitty Lo Mein' );
    say $fat->name, ' now eats ', $fat->diet;
```

---
### Something More Recent

```perl

class Bank::Account {

    # Of *course* this is too simplistic. The first pedant
    # who points this out loses 500 internet points.

    use My::Exceptions qw(InsufficientFunds)
    field $customer :param :reader;
    field $balance  :reader {0};

    method deposit ($amount) {
        $balance += $amount;
    }

    method withdraw ($amount) {
        if ( ( $balance - $amount ) < 0 ) {
            InsufficientFunds->throw("Naughty, naughty");
        }
        $balance -= $amount;
    }
}

```
---
## Inheritance as Taxonomy

From "The Moose is Flying" by Randal Schwartz, 2007

```perl

package Animal;
  use Moose;
  has 'name' => (is => 'rw');
  has 'color' => (is => 'rw');
  1;

  package Horse;
  use Moose;
  extends 'Animal';
  1;

```
---
[.build-lists: true]

# The Echidna says Fuck You to Taxonomy

* Monotreme – lays eggs
* Marsupial — in a pouch
* The pouch is ad-hoc
* They have cloaca – like birds

---
# We need a world that is designed to be modeled…
---
### Object Oriented Design has its issues, but we will talk about how to mitigate them some at the end
---
# Heroes and Monsters

Let's start with the basic entities of our game world…

```perl

class Entity {
    field $location :param
    field $sprite :param;
    field $fg :param //= '#fff';
    field $bg :param //= '#000';
}

my $hero = Entity->new( location => [0,0], sprite => '@' );

my $monster = Entity->new( location => [80,60], sprite => '👾');
```

^* A `class` describes a kind of thing.
^* Classes have `field`s that describe their properties
^* An object is a specific instance of a class 

---
# Method Acting

```perl
class Entity {
    field $location :param; # [x,y]
    field $sprite :param;
    field $fg :param //= '#fff';
    field $bg :param //= '#000';

    method render() { $sprite }
    method move($dx, $dy) { 
        $location[0] += $dx; 
        $location[1] += $dy; 
    }
}
```
^* A `method` is a subroutine that see and affect the fields
* Encapsulation is the fields of an class are only visible to the methods of the class
* Encapsulation means we (hopefully) know what state the fields of an object are in

---
# The Game Loop

> A game loop runs continuously during gameplay. Each turn of the loop, it processes user input without blocking, updates the game state, and renders the game. It tracks the passage of time to control the rate of gameplay.
-- Bob Nystrom, Game Programming Patterns

---
```perl

class Game {
   use builtin 'true';
   field $width :param = 80;
   field $heigh :param = 50;

   field @monsters;

   ADJUST { 
       for (0..NUM_MONSTERS) {
           my $point = [rand($width), rand($height)];
           push @monsters, Entity->new( 
               location => $point, 
               sprite => '👾',
           );
       }
   } 

   method run() {
      while(true) {
          $self->process_input();
          $self->update_game();
          $self->render();
      }
   }
}

Game->new()->run;
```
---
# Update Game

Our `update_game` method:

```perl
method update_game() {
    for my $m (@monsters) {
        $m->update();
    }
}
```
---
# In the Entity class: 

```perl
class Character {
# [...]
method update() {
     my $dx = int(rand(2)) - 1; # random -1,0,1
     my $dy = int(rand(2)) - 1; # rnadom -1,0,1 
		$self->move($dx, $dy)
   }
}
```
---
# The Game Map is a Level Factory
---
# Bullet in the Blue Sky

---
# Rendering Polymorphism

```perl
method render() { }
```

---
# Hooking it all up
---
## Delegation over Inheritance

```perl
```
---
# Game Components aren't Real
---
# Abstractions get further from Reality
---
# All Abstractions Leak
---
# The Bad Guys Close In (25)
---
# The Bad Guys Close In (26)
---

# Changes have consequences. 
### Modifications can unbalance systems and lead to death
---
# Dark Night of the Soul (28)
---
# Dark Night of the Soul (28)
---
# Dark Night of the Soul (30)
---
## The Primary Goal of a good model is to reduce coupling and cognitive load
---
# Components

---
# Finale (33)
---
# Finale (34)
---
# Finale (35)
---
# Finale (36)
---
# Finale (37)
---
## A model is just a metaphor pretending to have a blue collar job.
---
# The map is not the territory.
### The power of modeling is the choice of what is kept and what is left out
---

![XKCD 312](IMG_1879.png)

---

##### A Metamodernist Object System 
Corinna oscillates between modernist positions regarding Object Oriented Design, and the postmodern deconstructionist principles that were embraced dramatically by Moose.

Corinna rejects the postmodern concept of deconstructing the existing object system (by rebuilding it with a robust framework) and instead returns to the modernist design of extending the core language with the attributes we found most appealing after gazing at the system with our postmodernist mindset, but without the baggage that the postmodernist ironic self reflexive implementation required us to have.

---
- ECS Architecture
- Metamodernity
