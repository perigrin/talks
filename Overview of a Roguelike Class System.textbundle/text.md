# Overview of a Roguelike Class System
^ a few notes up front, code in this talk may use syntax to make the slides more readable none of it should be _bad_ but it may not be the best solution for all audience
^ also all my code uses 5.40.0 which has the latest version of Corinna

---

# Object oriented programming is about **_modeling_** your domain

---

# Most tutorials cover the tooling: classes, methods, encapsulation, inheritance, polymorphism...

---

# ...but invent a crappy domain

---

## Modern Perl 4e by chromatic

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

```perl
class Bank::Account {

    # Of *course* this is too simplistic. The first pedant
    # who points this out loses 500 internet points.

    use My::Exceptions qw(InsufficientFunds);
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
<sub>From "Understanding Class Inheritance" by Curtis Poe, 2022</sub>

---

## They Introduce Inheritance as Taxonomy

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
<sub>From "The Moose is Flying" by Randal Schwartz, 2007</sub>

---

[.build-lists: true]

# The Echidna says Fuck You to Taxonomy

* It's a Monotreme — meaning it lays eggs
* It's ALSO a Marsupial — it lays them in a pouch
* But the pouch is ad-hoc — it only makes the pouch to hold eggs
* And they have cloaca — like birds

---

# We need a domain that is designed to be modeled…

---

> [A] high degree of specificity or accuracy of real-world effects or elements is not always advantageous to a well-themed game.
-- Jeff Warrender and Ben Maddox, You Said This Would Be Fun

---

# Heroes and Monsters

```perl
class Entity {
    field $location :param;
    field $sprite :param;
}

my $hero    = Entity->new( location => [ 0,  0 ],  sprite => '@' );
my $monster = Entity->new( location => [ 80, 60 ], sprite => '👾' );
```

^
* A `class` describes a kind of thing.
* Classes have `field`s that describe their properties
* An object is a specific instance of a class, like $hero or $monster
* :param means that we can take that value in the constructor

---


# Methods of Action

```perl
class Entity {
    field $location :param;
    field $sprite :param;

    method location() { $location }
    method sprite () { $sprite }

    method move( $dx, $dy ) {
        $location[0] += $dx;
        $location[1] += $dy;
    }
}
```
^* A `method` is a subroutine that see and affect the fields
* Encapsulation is the fields of an class are only visible to the methods of the class
* Encapsulation means we (hopefully) know what state the fields of an object are in
* `:reader` says to automatically create a reader for each field

---

# The Game Class

> A game loop runs continuously during gameplay. Each turn of the loop, it processes user input without blocking, updates the game state, and renders the game. It tracks the passage of time to control the rate of gameplay.
-- Bob Nystrom, Game Programming Patterns

---

```perl
class Game {
    use Term::ScreenColor ();

    field $width :param  = 80;
    field $height :param = 50;

    field $hero = Entity->new(
        location => [ $width / 2, $height / 2 ],
        sprite   => '@',
    );

    field @monsters = map Entity->new(
        location => [ rand($width), rand($height) ],
        sprite   => '👾',
    ) => 0 .. 5;

    field $term = Term::ScreenColor->new()->colorizable(1)->curinvis;

    method process_input() { }

    method update_game() { }

    method draw_game() {
        $term->clrscr();
        $term->at( $_->location->@* )->puts( $_->sprite ) for @monsters;
        $term->at( $hero->location->@* )->green()->puts( $hero->sprite );
    }

    method run() {
        while (1) {
            $self->process_input() // next;
            $self->update_game();
            $self->draw_game();
        }
    }
}
Game->new()->run;
```
^This isn't any more complicated than the Entity class in principle. You have some simple state, the height and width, you have a hero and a bunch of monsters.
^Notice that the monsters, hero, and term don't have :param, they're all internal to the class

---

```perl
field $height :param = 50;

field $hero = Entity->new(
    location => [ $width / 2, $height / 2 ],
    sprite   => '@',
);

field @monsters = map Entity->new(
    location => [ rand($width), rand($height) ],
    sprite   => '👾',
) => 0 .. 5;

field $term = Term::ScreenColor->new()->colorizable(1);
```
---

# The Game Loop
```perl
method run() {
    while (1) {
        $self->process_input() // next;
        $self->update_game();
        $self->draw_game();
    }
}
```

---

# Rendering is the easy part
```perl
method draw_game() {
        $term->clrscr();
        $term->at( $_->location->@* )->puts( $_->sprite ) for @monsters;
        $term->at( $hero->location->@* )->green()->puts( $hero->sprite );
    }
```

---

# Game Updates

> The game world maintains a collection of objects. Each object implements an update method that simulates one frame of the object’s behavior. Each frame, the game updates every object in the collection.
-- Bob Nystrom, Game Programming Patterns

---

# Game Updates

```perl
method update_game() {
    $_->update for @monsters;
}
```
^Yeah it's that simple

---

# In the Entity class:

```perl
class Entity {
    # [...]
    method update() {
        my $dx = int( rand(3) ) - 1;    # random -1,0,1
        my $dy = int( rand(3) ) - 1;    # random -1,0,1
        $self->move( $dx, $dy );
    }
    # [...]
}
```
^This just has the various entities move in a random walk, one square in each direction each frame.
^ Because we only call the `update` method on the `@monsters` array, it only moves the monsters, but the code is present for the hero as well. This suggests we don't have the right level of abstraction between our classes, and something is off in our model.

---

> [T]he real art lies in finding an appropriate set of abstractions that captures and draws out the essential points of the theme and that doesn't clutter up the game with too many extraneous effects.
-- Jeff Warrender and Ben Maddox, You Said This Would Be Fun

---

[.build-lists: true]

# Did you spot the potential bug?

```perl
field $hero = Entity->new(
    location => [ $width / 2, $height / 2 ],
    sprite   => '@',
);
```

* What happens if someone calls `$hero->update()`?

---

# A Reason to use Inheritance

```perl
class Entity {
    field $location :param :reader;

    method move( $dx, $dy ) {
        $location->[0] += $dx;
        $location->[1] += $dy;
    }

    method update() {...}
}

class Player :isa(Entity) {
    field $sprite :reader = '@';
}

class Monster :isa(Entity) {
    field $sprite :reader = '👾';

    method update() {
        my $dx = int( rand(3) ) - 1;    # random -1,0,1
        my $dy = int( rand(3) ) - 1;    # rnadom -1,0,1
        $self->move( $dx, $dy );

    }
}
```
^ This is different from the Echidna because we're splitting the `Entity` class along behavior lines that we encountered in the actual code, not trying to invent a perfect hierarchy.

---

```perl
class Entity {
    field $location :param :reader;

    method move( $dx, $dy ) {
        $location->[0] += $dx;
        $location->[1] += $dy;
    }

    method update() { ... }
}
```

---

```perl
class Player :isa(Entity) {
    field $sprite :reader = '@';
}
```

---

```perl
class Monster :isa(Entity) {
    field $sprite :reader = '👾';

    method update() {
        my $dx = int( rand(3) ) - 1;    # random -1,0,1
        my $dy = int( rand(3) ) - 1;    # random -1,0,1
        $self->move( $dx, $dy );

    }
}
```
---

# Actions and Commands

> Commands are an object-oriented replacement for callbacks.
-- Bob Nystrom, Game Programming Patterns

---

```perl
class Action {}

class MoveAction :isa(Action) {
    field $dx :param = 0;
    field $dy :param = 0;

    method execute($entity) {
       $entity->move( $dx, $dy );
    }
}
```

---

```perl
class Game {
    [...]
    field @actions;
    field $key_bindings : param = {
        h => MoveAction->new( dx => -1 ),
        j => MoveAction->new( dy => -1 ),
        k => MoveAction->new( dy =>  1 ),
        l => MoveAction->new( dx =>  1 ),
    };

    method process_input() {
        push @actions, $key_bindings->{ $screen->getch() };
        $screen->flush_input();
    }

    method update_game() {
        $_->execute($hero) for @actions;
        $_->update for @monsters;
    }
    #[..]
}
```

---

```perl
class Monster :isa(Entity) {
    field $sprite :reader = '👾';

    method update() {
        # TODO replace with a better AI
        my $action = MoveAction->new( dx => int( rand(3) ) - 1, dy => int( rand(3) ) - 1 );

        $action->execute($self);
    }
}
```
---

```perl
class RunAction :isa(MoveAction) {
    method execute($entity) {
        return unless $entity isa 'Entity';

        $entity->move( $dx * 5, $dy * 5 );
    }
}

class AttackAction :isa(Action) {
    field $target :param;
    method execute($entity) { ... }
}
```
---

# Dealing with Complexity

---

# How to avoid Echidna Taxonomy

```perl
class Ghost :isa(Entity) {
   field $sprite :reader = '👻';
   method update() {...}
}

class Vampire :isa(Entity) {
   field $sprite :reader = '🧛';
   method update() {...}
} #
```

---

# Use a Factory

```perl
package Beastiary {
    sub monster() { Monster->new( sprite => '👾' ) }
    sub ghost()   { Monster->new( sprite => '👻' ) }
    sub vampire() { Monster->new( sprite => '🧛' ) }
} # end
```
---

# A naive combat implementation

```perl
class AttackAction :isa(Action) {
    field $target :param;
    method execute($entity) {
        $target->update_health( $entity->attack() );
    }
}

class Entity {
    # [...]
    field $hp :param  = rand(5) + 1; # 1d6
    field $str :param = rand(5) + 1; # 1d6
    method take_damage($damage) { $hp -= $damage }
    method attack()             { $str }
    # [...][
}
```
<sub>Based on the combat system from _Cairn_ by Yochai Gal</sub>

---

[.build-lists: true]

# This works but ...

* It won't scale well with more stats (5e anyone?)
* Every entity now has HP and Strength rather than just being a thing with a location
* There's a better choice...

---

# Use Delegation

```perl
class HealthComponent {
    field $hp : param = rand(5) + 1;
    method take_damage($damage) { $hp -= $damage }
}

class CombatComponent {
    field $str : param = rand(5) + 1;
    method attack() { $str }
}
```

---

# Update the Entity class

```perl
class Entity {
    field $components : param : reader = [];

    method find_component($type) {
        grep { $_ isa $type } $components->@*
    }
    method add_component($component) {
        push @components, $component;
    }
}
```

---

# And the attack action

```perl
class AttackAction : isa(Action) {
    field $target : param;

    method execute($entity) {
        my $defender = $target->find_component('HealthComponent');
        my $attacker = $entity->find_component('CombatComponent');
        return unless $attacker && $defender;
        $defender->take_damage( $attacker->attack() );
    }
}
```
---

# The primary goal of a good model is to reduce coupling and thus cognitive load.

---

# Changes have consequences
## Modifications can unbalance systems and lead to death

---

# The map is not the territory.
## The power of modeling is the choice of what is kept and what is left out

---

# Thank You
## Questions?

---

![inline](IMG_1879.png)

^XKCD #1879 by Randal Munroe

---

# BONUS: A Metamodernist Object System

Corinna oscillates between modernist positions regarding Object Oriented Design, and the postmodern deconstructionist principles that were embraced dramatically by Moose.

Corinna rejects the postmodern concept of deconstructing the existing object system (by rebuilding it with a robust framework) and instead returns to the modernist design of extending the core language with the attributes we found most appealing after gazing at the system with our postmodernist mindset, but without the baggage that the postmodernist ironic self reflexive implementation required us to have.
