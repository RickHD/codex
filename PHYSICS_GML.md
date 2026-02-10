# Player (roguelite/roguelike) em GML com variáveis simples (`spd`)

Abaixo está uma versão direta no seu estilo, usando poucas variáveis e pronta para colar.

## Onde colocar
- **Create (obj_player):** inicialização das variáveis.
- **Step (obj_player):** leitura de input, movimento, colisão e flip do sprite.

---

## Create (obj_player)
```gml
spd = 2.5;
```

---

## Step (obj_player) — base simples (igual ao seu estilo)
```gml
// || = or

// Inputs
var key_right = keyboard_check(ord("D")) || keyboard_check(vk_right);
var key_left  = keyboard_check(ord("A")) || keyboard_check(vk_left);
var key_up    = keyboard_check(ord("W")) || keyboard_check(vk_up);
var key_down  = keyboard_check(ord("S")) || keyboard_check(vk_down);

// Vetor de direção
var hspd = key_right - key_left;
var vspd = key_down - key_up;

// Normaliza (evita diagonal mais rápida)
if (hspd != 0 || vspd != 0) {
    var len = point_distance(0, 0, hspd, vspd);
    hspd /= len;
    vspd /= len;
}

// Aplica velocidade
x += hspd * spd;
y += vspd * spd;

// Flip do sprite
if (key_left)  image_xscale = -1.5;
if (key_right) image_xscale =  1.5;
```

---

## Step (obj_player) — com colisão em parede (`obj_wall`)
Se você já tem parede sólida como objeto, use esta versão para não atravessar:

```gml
// || = or

var key_right = keyboard_check(ord("D")) || keyboard_check(vk_right);
var key_left  = keyboard_check(ord("A")) || keyboard_check(vk_left);
var key_up    = keyboard_check(ord("W")) || keyboard_check(vk_up);
var key_down  = keyboard_check(ord("S")) || keyboard_check(vk_down);

var hspd = key_right - key_left;
var vspd = key_down - key_up;

if (hspd != 0 || vspd != 0) {
    var len = point_distance(0, 0, hspd, vspd);
    hspd /= len;
    vspd /= len;
}

var mov_x = hspd * spd;
var mov_y = vspd * spd;

// Eixo X
x += mov_x;
if (place_meeting(x, y, obj_wall)) {
    x -= mov_x;
}

// Eixo Y
y += mov_y;
if (place_meeting(x, y, obj_wall)) {
    y -= mov_y;
}

if (key_left)  image_xscale = -1.5;
if (key_right) image_xscale =  1.5;
```

---

## Extra opcional (ainda simples): dash usando só `spd`
**Onde aplicar:** Step do player.

```gml
// Dash curto no Shift
if (keyboard_check_pressed(vk_shift)) {
    var key_right = keyboard_check(ord("D")) || keyboard_check(vk_right);
    var key_left  = keyboard_check(ord("A")) || keyboard_check(vk_left);
    var key_up    = keyboard_check(ord("W")) || keyboard_check(vk_up);
    var key_down  = keyboard_check(ord("S")) || keyboard_check(vk_down);

    var hspd = key_right - key_left;
    var vspd = key_down - key_up;

    if (hspd != 0 || vspd != 0) {
        var len = point_distance(0, 0, hspd, vspd);
        hspd /= len;
        vspd /= len;

        // multiplica sua velocidade base por 4 no impulso
        x += hspd * (spd * 4);
        y += vspd * (spd * 4);
    }
}
```

---

## Onde aplicar cada parte (resumo)
- `spd = 2.5;` -> **Create** do `obj_player`.
- Input + normalização + `x/y` -> **Step** do `obj_player`.
- `place_meeting(..., obj_wall)` -> **Step**, logo após calcular movimento.
- `image_xscale` -> **Step**, após ler input horizontal.

Se quiser, no próximo passo eu monto **inimigo + tiro** no mesmo estilo minimalista (só `spd`, `dir`, `hp` etc.).
# Física simples em GML para roguelite/roguelike

Este guia traz trechos curtos de GML (GameMaker) com física simples para jogos roguelite/roguelike, além de **onde aplicar** cada trecho (eventos/objetos típicos).

## 1) Movimento com aceleração, atrito e limite de velocidade
**Onde aplicar:** Evento **Create** e **Step** do objeto do jogador (`obj_player`).

**Create (obj_player):**
```gml
// movimento básico
move_speed = 0;
move_accel = 0.7;
move_friction = 0.4;
move_max = 5;

hsp = 0;
vsp = 0;
```

**Step (obj_player):**
```gml
var input_x = keyboard_check(vk_right) - keyboard_check(vk_left);
var input_y = keyboard_check(vk_down) - keyboard_check(vk_up);

// aceleração
hsp += input_x * move_accel;
vsp += input_y * move_accel;

// atrito quando não há input
if (input_x == 0) hsp = approach(hsp, 0, move_friction);
if (input_y == 0) vsp = approach(vsp, 0, move_friction);

// limite de velocidade
hsp = clamp(hsp, -move_max, move_max);
vsp = clamp(vsp, -move_max, move_max);

x += hsp;
y += vsp;
```

**Script auxiliar (crie em `scripts/approach.gml` ou no editor):**
```gml
function approach(value, target, amount) {
    if (value < target) return min(value + amount, target);
    if (value > target) return max(value - amount, target);
    return value;
}
```

## 2) Colisão com paredes (tile-based simples)
**Onde aplicar:** Evento **Step** do jogador após atualizar `x/y`. Requer um objeto sólido `obj_wall`.

```gml
// movimento no eixo X
x += hsp;
if (place_meeting(x, y, obj_wall)) {
    while (!place_meeting(x + sign(hsp), y, obj_wall)) {
        x += sign(hsp);
    }
    hsp = 0;
}

// movimento no eixo Y
y += vsp;
if (place_meeting(x, y, obj_wall)) {
    while (!place_meeting(x, y + sign(vsp), obj_wall)) {
        y += sign(vsp);
    }
    vsp = 0;
}
```

## 3) Impulso (dash curto)
**Onde aplicar:** Evento **Step** do jogador. Útil para combate rápido.

```gml
if (keyboard_check_pressed(ord("X"))) {
    var dash = 6;
    hsp += dash * sign(hsp != 0 ? hsp : keyboard_check(vk_right) - keyboard_check(vk_left));
    vsp += dash * sign(vsp != 0 ? vsp : keyboard_check(vk_down) - keyboard_check(vk_up));
}
```

## 4) Recuo (knockback) ao tomar dano
**Onde aplicar:** Evento **Collision** com `obj_enemy` ou no script de dano.

```gml
// supondo que enemy_x/enemy_y sejam as coordenadas do inimigo
var dir = point_direction(enemy_x, enemy_y, x, y);
var knock = 4;
hsp += lengthdir_x(knock, dir);
vsp += lengthdir_y(knock, dir);
```

## 5) Projétil com velocidade e alcance
**Onde aplicar:** Evento **Create** e **Step** do projétil (`obj_bullet`).

**Create (obj_bullet):**
```gml
spd = 10;
dir = image_angle;
range = 200;
start_x = x;
start_y = y;
```

**Step (obj_bullet):**
```gml
x += lengthdir_x(spd, dir);
y += lengthdir_y(spd, dir);

if (point_distance(start_x, start_y, x, y) > range) {
    instance_destroy();
}

if (place_meeting(x, y, obj_wall)) {
    instance_destroy();
}
```

## 6) Inércia para inimigos simples (perseguição com suavidade)
**Onde aplicar:** Evento **Step** do inimigo (`obj_enemy`).

```gml
var target = instance_nearest(x, y, obj_player);
if (instance_exists(target)) {
    var dir = point_direction(x, y, target.x, target.y);
    var acc = 0.2;
    var max = 2.5;

    hsp = clamp(hsp + lengthdir_x(acc, dir), -max, max);
    vsp = clamp(vsp + lengthdir_y(acc, dir), -max, max);
}

x += hsp;
y += vsp;
```

## 7) Regras de aplicação (resumo rápido)
- **Create:** inicialize variáveis (`hsp`, `vsp`, `acel`, `max`).
- **Step:** aplique forças, atrito, limite e mova `x/y`.
- **Collision:** ajuste velocidade, aplique recuo e finalize movimentos.
- **Objects/Assets:** crie `obj_player`, `obj_enemy`, `obj_wall`, `obj_bullet`.

## 8) Dica de estrutura de sala (roguelike)
- `obj_wall`: paredes sólidas (tile ou instâncias).
- `obj_player`: controla input/movimento.
- `obj_enemy`: IA simples e colisão com paredes.
- `obj_bullet`: projéteis com range.
- `obj_room_controller`: spawn de inimigos e itens.
