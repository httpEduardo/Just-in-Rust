Legion tem como objetivo ser uma biblioteca ECS de alto desempenho e rica em recursos para projetos de jogos Rust com o mínimo de boilerplate.

Começando
Mundos
Worlds são coleções de entities, onde cada entidade pode ter uma coleção arbitrária de components anexados.

rust

use legion::*;
let world = World::default();
Entidades podem ser inseridas via push (para uma única entidade) ou extend (para uma coleção de entidades com os mesmos tipos de componentes). O mundo criará um ID exclusivo para cada entidade na inserção que você pode usar para se referir a essa entidade mais tarde.

rust

// um componente é qualquer tipo que seja 'estático, sized, send e sync
#[derive(Clone, Copy, Debug, PartialEq)]
struct Position {
    x: f32,
    y: f32,
}
#[derive(Clone, Copy, Debug, PartialEq)]
struct Velocity {
    dx: f32,
    dy: f32,
}

// push um componente tupla no mundo para criar uma entidade
let entity: Entity = world.push((Position { x: 0.0, y: 0.0 }, Velocity { dx: 0.0, dy: 0.0 }));

// ou estenda via um IntoIterator de tuplas para adicionar muitas de uma vez (isso é mais rápido)
let entities: &[Entity] = world.extend(vec![
    (Position { x: 0.0, y: 0.0 }, Velocity { dx: 0.0, dy: 0.0 }),
    (Position { x: 1.0, y: 1.0 }, Velocity { dx: 0.0, dy: 0.0 }),
    (Position { x: 2.0, y: 2.0 }, Velocity { dx: 0.0, dy: 0.0 }),
]);

Você pode acessar as entidades via entries. As entries permitem que você consulte uma entidade para descobrir quais tipos de componentes estão anexados a ela, obter referências de componentes ou adicionar e remover componentes.