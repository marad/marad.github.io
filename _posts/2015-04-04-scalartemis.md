---
title:  "Scalartemis"
date:   2015-04-04 22:06:00
tags: scala scalartemis
categories: programming
---

Recently I've been creating entity component framework which I named _Scalartemis_. Name is direct descendant of two words. First (surprisingly) is _Scala_ - the language it's written in. Second is _Artemis_.

[Artemis][artemis] is entity component framework that I wanted to port to Scala. During the process of porting I started to change things a bit to achieve better performance/api/usability. At some point I decided that this is pointless and I should start from scratch using the knowlodge I've gained. This is exactly what I've done.

I've found [document][fast-entity-system] which was the base for Artemis. Unfortunately its creators had not followed it closely. I think that there is room for improvements so I decided to give it a shot.

### What it is all about?

If you've ever tried to make a game you must have noticed that you should manage your game objects. You should be able to draw the elements that can be drawn but there could be also entities without graphic representation like some special areas. The entities differ from one another. Hero, bullet, and a gun are all entities but are entirely different. Hero does have stats like _health_ or _stamina_ but this is totally not applicable to his gun.

Component Entity Frameworks are one way of dealing with this problem. They try to make entity as abstract as it can be and then add multiple components describing its properties. I've seen many implementations which tried to represent the _entity_ as a class which had list of components. One of the problems with this aproach is performance. Implementing this you would probably say _'RenderComponent is so common that I should make it special and available from the entity class without searching in the component list.'_. At least this is what I ended up with every time I tried to approach component entity systems.

### What is different from Artemis?

While I liked the overall concept I didn't like the code that was written to satisfy it. The more I read the Artemis codebase the more I wanted to rewrite it from scratch. Starting from the beginning I've reimplemented almost completely (no entity tagging yet) Artemis and I believe I've cut number of classes in a half. We all know that the less code we have the less code can break ;)

Also I've implemented everything using TDD so every functionality has it's unit test. This of course gives me very good indication wheather my changes break something somewhere or not :)


### Enough talking, show me the code!

Sure, sure. I wanted to keep the API very similar to Artemis's:

{% highlight scala linenos %}
case class Position(var x: Int, var y: Int) extends Component

class PositionUpdateSystem(val world: World)
      extends SequentialProcessingSystem(Aspect.forAll[Position])
      with ComponentMapping {

  override def process(entity: Entity, delta: Float): Unit = {
    component[Position](entity) match {
      case Some(position) =>
        position.x += 1
        position.y += 1
      case None =>
    }
  }
}

val world = new World
world.registerSystem(new PositionUpdateSystem(world))
world.createEntity(Position(1, 2))
world.update(1f)
{% endhighlight %}

On the first line we decalre new component class `Position`. Next up is declaration of entity processing system. In this case we extend `SequentialProcessingSystem` which just give us all entities one by one. The interesting thing here is that every entity system has to has an `Aspect`. Aspects define which entities can be processed by the system. Here we want all entities with `Position` component.

The `process` method of entity system is the one that does the job. In this case we just get the component of type `Position` for processed entity and increase its `x` and `y` values by 1.

The `component[T](entity: Entity)` method is provided by `ComponentMapping` trait. This trait requires that the class has the `world: World` field so it can fetch the entity components.

Starting from line 17 we create new World, register our entity processing system and create single entity with component `Position(1, 2)`. This four lines causes a lot of events under the hood. First of all when the system is registered _world_ creates a set of entities with matching aspect required by the system. If set for such aspect already exsits it's only passed to the system. Of course in this case we don't have any entities yet, but empty set is created and is updated when entity with required aspect is created.

Then when the entity is created it's automatically added to the right entity sets and is instantly available to all the system that are interested in it (based on component set).

Last line - 20 - is where everything comes to life. We perform one update of all systems. Here we have only one system with one entity and the logic is not very spectacular but hey, it's something! After this call to `update` method the entity position is going to be `Position(2, 3)` of course .

### That's all!

Ok, I think this post is already too long. I just wanted to present the implementation of component entity framework I'm working on. If you'd like to dive into the code you could view it on [GitHub][github].

Cheers!


[artemis]: http://gamadu.com/artemis/
[github]: https://github.com/marad/scalartemis
[fast-entity-system]: http://entity-systems.wikidot.com/fast-entity-component-system
