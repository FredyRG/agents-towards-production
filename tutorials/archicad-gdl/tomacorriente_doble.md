# Objeto GDL de Archicad: Tomacorriente doble (2D + 3D)

Este ejemplo define un **tomacorriente doble de pared** con:

- Placa frontal rectangular redondeada.
- Dos módulos de toma (izquierdo y derecho).
- Geometría simple para buen rendimiento en planta y 3D.

## 1) Parámetros recomendados (Parameter Script)

Pega esto en **Parameter Script** del objeto:

```gdl
! ===== Parámetros base (m) =====
if A <= 0 then A = 0.12      ! ancho total
if B <= 0 then B = 0.07      ! alto total

if placaEsp <= 0 then placaEsp = 0.008
if bordeR <= 0 then bordeR = 0.006
if inset <= 0 then inset = 0.008
if moduloR <= 0 then moduloR = 0.020
if sepX <= 0 then sepX = 0.035
if huecoR <= 0 then huecoR = 0.0035
if huecoSep <= 0 then huecoSep = 0.010
if embutido <= 0 then embutido = 0.0015

! Materiales / plumas
if matPlaca = 0 then matPlaca = 1
if matModulo = 0 then matModulo = 1
if penCont = 0 then penCont = 1
if penDet = 0 then penDet = 2
```

Crea los parámetros en Archicad con estos tipos:

- `placaEsp, bordeR, inset, moduloR, sepX, huecoR, huecoSep, embutido` → **Length**
- `matPlaca, matModulo` → **Material**
- `penCont, penDet` → **Pen Color**

---

## 2) Script 3D

Pega esto en **3D Script**:

```gdl
! ==============================================
! TOMACORRIENTE DOBLE - 3D SCRIPT
! Ejes locales:
! X = ancho, Y = alto, Z = profundidad (saliente)
! Origen al centro de la placa, apoyada en muro
! ==============================================

resol 36

an = A
al = B
esp = placaEsp
rad = min (bordeR, an/2, al/2)

! -------- Placa frontal --------
material matPlaca
block an, al, esp

! -------- Dos módulos circulares --------
material matModulo

zMod = esp - embutido
rMod = min(moduloR, (an/2 - inset), (al/2 - inset))

for s = -1 to 1 step 2
    addx s * sepX
    addz zMod

    ! Disco frontal del módulo
    cylinder 0.001, rMod

    ! Aro/marco leve
    addz -0.001
    cylinder 0.0015, rMod * 0.92

    ! Huecos de clavija (2 cilindros restados visualmente)
    ! Para simplificar librería, modelamos como cavidades cortas
    addz 0.0002
    addx -huecoSep/2
    cylinder -0.0012, huecoR
    del 1

    addx huecoSep
    cylinder -0.0012, huecoR
    del 1

    del 3
next s
```

---

## 3) Script 2D

Pega esto en **2D Script**:

```gdl
! ==============================================
! TOMACORRIENTE DOBLE - 2D SCRIPT
! Símbolo en planta
! ==============================================

pen penCont
line_type 1

an = A
al = B

! Contorno placa
rect2 -an/2, -al/2, an/2, al/2

! Módulos dobles
pen penDet

r2d = moduloR

for s = -1 to 1 step 2
    cx = s * sepX
    cy = 0

    circle2 cx, cy, r2d

    ! Huecos de clavija
    circle2 cx - huecoSep/2, cy, huecoR
    circle2 cx + huecoSep/2, cy, huecoR
next s

! Línea central de referencia (opcional)
pen penCont
line2 0, -al/2, 0, al/2
```

---

## 4) Opcional: Script de interfaz rápida (UI Script)

Si quieres exponer pocos parámetros al usuario final:

```gdl
ui_page 1, "Tomacorriente"
ui_infield "A", 10, 20, 120, 20
ui_infield "B", 10, 50, 120, 20
ui_infield "sepX", 10, 80, 120, 20
ui_infield "moduloR", 10, 110, 120, 20
```

---

## 5) Notas de uso

- Este código está pensado como **plantilla base**; ajusta dimensiones según normativa local.
- Para más realismo, puedes:
  - Sustituir `block` por una placa con esquinas redondeadas usando perfiles/sweeps.
  - Añadir chaflán o redondeo frontal del módulo.
  - Crear variantes (Schuko, tipo A/B, USB) con parámetros booleanos.
