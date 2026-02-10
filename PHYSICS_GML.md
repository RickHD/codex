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
