# Player (roguelite/roguelike) em GML com `spd` + colisão integrada

Abaixo está exatamente o estilo que você pediu: variáveis simples e colisão integrada no mesmo fluxo do movimento.

## Onde aplicar
- **Create (obj_player):** velocidade base.
- **Step (obj_player):** input, normalização, movimento, colisão, sprite flip e dash.

---

## Create (obj_player)
```gml
spd = 2.5;
```

---

## Step (obj_player) — movimento + colisão integrada + dash
```gml
// || = or

var key_right = keyboard_check(ord("D")) || keyboard_check(vk_right);
var key_left  = keyboard_check(ord("A")) || keyboard_check(vk_left);
var key_up    = keyboard_check(ord("W")) || keyboard_check(vk_up);
var key_down  = keyboard_check(ord("S")) || keyboard_check(vk_down);

// direção de input
var hspd = key_right - key_left;
var vspd = key_down - key_up;

// normaliza para não correr mais rápido na diagonal
if (hspd != 0 || vspd != 0) {
    var len = point_distance(0, 0, hspd, vspd);
    hspd /= len;
    vspd /= len;
}

// transforma direção em movimento real usando spd
hspd *= spd;
vspd *= spd;

// movimento no eixo X
x += hspd;
if (place_meeting(x, y, obj_wall)) {
    while (!place_meeting(x + sign(hspd), y, obj_wall)) {
        x += sign(hspd);
    }
    hspd = 0;
}

// movimento no eixo Y
y += vspd;
if (place_meeting(x, y, obj_wall)) {
    while (!place_meeting(x, y + sign(vspd), obj_wall)) {
        y += sign(vspd);
    }
    vspd = 0;
}

// flip do sprite
if (key_left)  image_xscale = -1.5;
if (key_right) image_xscale =  1.5;

// dash curto no Shift (usa mesma direção do input)
if (keyboard_check_pressed(vk_shift)) {
    var dash_h = key_right - key_left;
    var dash_v = key_down - key_up;

    if (dash_h != 0 || dash_v != 0) {
        var dash_len = point_distance(0, 0, dash_h, dash_v);
        dash_h /= dash_len;
        dash_v /= dash_len;

        // multiplica velocidade base por 4
        var dash_spd = spd * 4;
        dash_h *= dash_spd;
        dash_v *= dash_spd;

        // dash com colisão integrada também
        x += dash_h;
        if (place_meeting(x, y, obj_wall)) {
            while (!place_meeting(x + sign(dash_h), y, obj_wall)) {
                x += sign(dash_h);
            }
        }

        y += dash_v;
        if (place_meeting(x, y, obj_wall)) {
            while (!place_meeting(x, y + sign(dash_v), obj_wall)) {
                y += sign(dash_v);
            }
        }
    }
}
```

---

## Resumo rápido
- `spd = 2.5` no **Create**.
- Todo o bloco principal no **Step**.
- `obj_wall` deve ser o objeto de parede/sólido usado no `place_meeting`.

Se quiser, no próximo passo eu separo isso em blocos menores (movimento, colisão e dash) mantendo o mesmo resultado.
