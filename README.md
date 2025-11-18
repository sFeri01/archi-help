# Archi segédlet

## Fejlesztőkörnyezet

- Visual Studio Code
  - MASM/TASM extension (feltöltő: clcxsrolau)

## Regiszterek

### Alapszabály

- ha `X`-re végződik, 16 bit
  - pl `AX`
- Ha `H`-ra vagy `L`-re végződik (**H**igh, **L**ow), 8 bit
  - pl `AH`, az `AX` felső 8 bitje

### Általános célú regiszterek

- `AX` Accumulator (általános regiszter)
  - `AH`, `AL`
- `BX` Base - Memóriacímzésnél használjuk
  - `BH`, `BL`
- `CX` Ciklusnál használjuk
  - A `loop` utasítás minden ciklusa 1-gyel csökkenti
- `DX`

### Index regiszterek

- `SI` String forrás cím
- `DI` String cél cím

### Mutató regiszterek

- `SP` (**S**tack **P**ointer)
  - Verem tetejére mutat
- `BP`
  - A verem egy elemét jelöli

### Státuszregiszter
<!-- https://www.tutorialspoint.com/assembly_programming/assembly_registers.htm -->
- `SR`
- Flagek
  - `CF` Carry flag
    - 1, ha előjel nélküli összeadásnál átvitel (carry), vagy kivonásnál kölcsönvétel (borrow) történik
  - `ZF` Zero flag
    - 1, ha az utasítás eredménye 0
  - `SF` Sign flag
    - 1, ha az utasítás eredménye negatív
  - `OF` Overflow flag
    - 1, ha túlcsordulás történt (előjeles számoknál)
  - Parity, Auxiliary
    - Mi nem használjuk
  
## Hello world

```asm
.model small
.data
    msg db "Hello$" 
.code
.startup
    mov ah, 09h
    mov dx, offset msg
    int 21h 
.exit
end
```

## Alap utasítások

### `mov` Adat mozgatása (másolása)

```asm
mov ax, 5
mov bx, ax
```

### `add` / `sub` Összeadás/kivonás

```asm
add ax, 5    ; Eredmény az ax-ben
sub bx, ax   ; bx-ax. Eredmény a bx-ben
```

### `inc` / `dec` 1-gyel növelés/csökkentés (increment/decrement)

```asm
mov ax, 5
inc ax      ; 6
dec ax      ; 5
```

### `cmp` Összehasonlítás

Lényegében egy kivonás, ami nem tárolja el az eredményt, csak a a status flageket tárolja [feltételes ugrásokhoz](#feltételes-ugrások) használjuk

```asm
cmp ax, bx
```

Flagek (ZF: Zero Flag, CF: Carry Flag):

Megjegyzés: Csak akkor működik így, ha előjel nélküli számként használjuk a regisztert (16 bit esetén 0-65535 között, 8 bit esetén 0-255)

- ZF = 1, ha ax = bx
  - mert ax-bx = 0
- ZF = 0, CF = 0, ha ax > bx
  - mert ax-bx > 0
- ZF = 0, CF = 1, ha ax < bx
  - mert ax-bx < 0

### `mul` Szorzás

#### 8 bites szorzás

A `mul` utasítás az `AL`-t szorozza be a kapott értékkel és az `AX`-be (16 bit!) teszi az eredményt

```asm
mov al, 5
mov bl, 3

mul bl    ; ax = al*bl = 15
```

#### 16 bites szorzás

A `mul` utasítás az `AX`-et szorozza be a kapott értékkel és `DX:AX`-be (`DX` felső 16 bit, `AX` alsó 16 bit => 32 bit) teszi az eredményt

```asm
mov ax, 5000
mov bx, 3000

mul bx   ; DX:AX = 15000000
```

### `div` (Madadékos) Osztás

#### 8 bites osztás

Az `AX`-et (16 bites) osztja el a kapott értékkel. `AL`-be kerül az (egész) hányados, `AH`-ba a maradék

```asm
mov ax, 12345
mov bl, 100

div bl    ; AX / BL

;AL = 123   (Hányados)
;AH = 45    (Haradék)
```

#### 16 bites osztás

A `DX:AX` (32 bites) regiszterpárt osztja el a kapott 16 bites értékkel. `AX`-be kerül a hányados, `DX`-be a maradék.

```asm
mov ax, 18416  ; 18416 dec = 49F0h
mov dx, 2      ; DX:AX = 000249F0h = 150000

mov bx, 4000

div bx    ; DX:AX / BX

; AX = 37   (Hányados)
; DX = 2000 (Maradék)
```

## Ugrások

### `jmp` Feltétel nélküli ugrás

```asm
hello:
  mov ah, 09h
  mov dx, offset msg
  int 21h 
```

```asm
jmp hello
```

### Feltételes ugrások

Ugrás a [státuszregiszterben](#státuszregiszter) található flagek alapján

- `jz`
  - Jump if Zero
  - Ugrás, ha az előző utasítás eredménye 0
- `jnz`
  - Jump if not Zero
  - Ugrás, ha az előző utasítás eredménye nem 0
- `jc`, `jnc`
  - Jump if carry/no carry
- `jo`, `jno`
  - Jump if overfllow/no overflow
- `js`, `jns`
  - Jump if Signed (negative)/Jump if not Signed (Positive or zero)

Példa: [gombnyomás kérése](#gombnyomás-kérése-ah--0)

```asm
xor ax, ax
int 16h
```

A `cmp al, 27` beállítja a Zero flaget, ha al=27

```asm
cmp al, 27 ; ESC

jz Program_vege
```

## Regiszter nullázás

```asm
xor ax, ax
```

Ugyanaz, mint `mov ax, 0`, de gyorsabb

## `int 10h` BIOS interrupt

[https://en.wikipedia.org/wiki/INT_10H#List_of_supported_functions](https://en.wikipedia.org/wiki/INT_10H#List_of_supported_functions)

### Video mode / text mode `AH = 0`

#### 80x25 Text mode ("képernyő törlése") `AL = 03h`

```asm
mov ax, 0003h    ; AH = 0, AL = 03h
int 10h
```

#### 320x200 Grafikus mód `AL = 13h`

```asm
mov ax, 0013h    ; AH = 0, AL = 13h
int 10h
```

### Kurzor pozíció `AH = 02h`

```asm
mov ah, 02h
mov bh, 0   ; Oldal 
mov dh, 5   ; Sor
mov dl, 28  ; Oszlop
int 10h
```

## `int 16h` BIOS interrupt

Billentyűzet funkciók

[https://en.wikipedia.org/wiki/INT_16H#List_of_services_of_the_INT_16_h](https://en.wikipedia.org/wiki/INT_16H#List_of_services_of_the_INT_16_h)

### Gombnyomás kérése `AH = 0`

`AH`-ba menti a [scan code](https://www.millisecond.com/support/docs/current/html/language/scancodes.htm)-ot,

`AL`-be az [ASCII](https://www.ascii-code.com/) kódot

#### Beolvasás

```asm
xor ax, ax
int 16h
```

#### Tesztelés

```asm
cmp al, 27 ; ESC
jz Program_vege
```

## `int 21h` DOS interrupt

[http://bbc.nvg.org/doc/Master%20512%20Technical%20Guide/m512techb_int21.htm](http://bbc.nvg.org/doc/Master%20512%20Technical%20Guide/m512techb_int21.htm)

### String kiírás `AH = 09h`

```asm
.data
    msg db "Hello$" 
```

```asm
mov ah, 09h
mov dx, offset msg
int 21h 
```

## Loop

A `loop` utasítás 1-gyel csökenti `CX`-et és ha ezután ez nem 0, ugrik a megjelölt címkére

```asm
mov cx, 5   ; Hányszor szeretnénk a ciklust lefuttatni
loop_start:
    ; Kiírjuk, hogy "Hello "
    mov ah, 09h
    mov dx, offset msg    ; msg db "Hello $" 
    int 21h 

    ; Csökkenti `CX`-et. Ha nem 0, ugrás loop_start-ra
    loop loop_start 
```
