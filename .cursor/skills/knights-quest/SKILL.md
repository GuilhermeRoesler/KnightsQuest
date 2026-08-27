---
name: knights-quest
description: >-
  Especificações e convenções do Knight's Quest (First Game): mecânicas,
  scripts, cenas, input, áudio e critérios de aceitação. Usar ao desenvolver,
  corrigir ou estender o jogo Godot 4.4 / GDScript deste repositório.
---

# Knight's Quest — Skill do projeto

## Identidade

| Campo | Valor |
| --- | --- |
| Nome no projeto | First Game |
| Nome popular | Knight's Quest |
| Motor | Godot 4.4 (Forward Plus) |
| Linguagem | GDScript |
| Género | Plataforma 2D / pixel art |
| Export | Windows Desktop x86_64 |
| Main scene | `scenes/game.tscn` |

## Visão

Side-scroller: cavaleiro recolhe moedas e evita killzones/slimes. Falha → slow-motion → reload da cena. Sem ecrã de vitória formal.

## Regras de design (versão atual)

1. Single-player, single-level
2. Sem inventário, vidas, power-ups ou menu formal
3. Falha = `reload_current_scene()` (score volta a 0)
4. Inimigos sem HP — contacto letal via `Killzone`
5. Pixel art com filtro nearest (`default_texture_filter = 0`)
6. Input só teclado (WASD + setas + espaço)

## Controlos (`project.godot` Input Map)

| Ação | Teclas |
| --- | --- |
| `move_left` | A, Seta Esquerda |
| `move_right` | D, Seta Direita |
| `jump` | Espaço, Seta Cima |

Deadzone `0.2`. Sem gamepad.

## Mecânicas

### Jogador — `scripts/player.gd` (`CharacterBody2D`)

- `SPEED = 130.0`, `JUMP_VELOCITY = -300.0`
- Salto só com `is_on_floor()` + `jump`
- Animações: `idle` / `run` (chão) · `jump` (ar)
- Flip horizontal conforme direção

### Moeda — `scripts/coin.gd` (`Area2D`)

- `body_entered` → `%GameManager.add_point()` → animação `pickup`

### Slime — `scripts/slime.gd` (`Node2D`)

- Patrulha `SPEED = 60`; vira com `RayCastRight` / `RayCastLeft`
- Tem `Killzone` filha (contacto = morte)

### Killzone — `scripts/killzone.gd` (`Area2D`)

1. `Engine.time_scale = 0.5`
2. Remove `CollisionShape2D` do body
3. `Timer` → no timeout: `time_scale = 1` + `reload_current_scene()`

### Plataformas

- `AnimatableBody2D`; algumas usam `AnimationPlayer` (móveis)

### Câmara

- `Camera2D` filha do `Player`

## Sistemas

### Score — `scripts/game_manager.gd`

- `score` inicia em 0; `add_point()` incrementa e atualiza label: `You collected N coins.`
- Moedas referenciam via `%GameManager`

### Áudio

- Autoload `Music` → `scenes/music.tscn` (autoplay, bus `Music`)
- BGM: `assets/music/time_for_adventure.mp3`
- SFX em `assets/sounds/`: `coin`, `jump`, `hurt`, `explosion`, `power_up`, `tap`
- Pickup usa `PickupSound` na cena da moeda

### Nível (`scenes/game.tscn`)

`GameManager`, `TileMap`, `Player`+`Camera2D`, `Killzone`, `Platforms`, `Coins` (10), `Slime`, `Labels`, `HUD`

## Arquitetura

```
assets/{fonts,music,sounds,sprites}/
scenes/{game,player,coin,slime,platform,killzone,music}.tscn
scripts/{player,game_manager,coin,slime,killzone}.gd
```

| Script | Nó | Papel |
| --- | --- | --- |
| `player.gd` | `CharacterBody2D` | Física, input, animações |
| `game_manager.gd` | `Node` | Score + UI |
| `coin.gd` | `Area2D` | Coleta |
| `slime.gd` | `Node2D` | Patrulha |
| `killzone.gd` | `Area2D` | Morte + reinício |

Convenção: lógica em `scripts/`, composição em `scenes/`. Preferir estender cenas existentes a inventar estrutura paralela.

## Critérios de aceitação

- [ ] Movimento/salto conforme Input Map
- [ ] Animações `idle`/`run`/`jump` alinhadas ao estado
- [ ] Moeda incrementa score e dá feedback de pickup
- [ ] Killzone/slime → slow-mo → reload
- [ ] Música autoplay
- [ ] Sprites sem blur (nearest)
- [ ] Câmara segue o jogador

## Extensões (só se pedidas)

Menu/pausa/vitória · vidas/checkpoints · multi-nível · ataque · high score · gamepad · mais plataformas de export · ligar SFX ainda não usados

## Ao alterar o jogo

1. Respeitar constantes e fluxos acima, salvo pedido contrário
2. Manter unique name `%GameManager` para score
3. Novos perigos letais devem reutilizar `killzone.tscn` / o mesmo fluxo
4. Não mudar filtro de texturas para linear
5. Validar mentalmente contra a checklist de aceitação
