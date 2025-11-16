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
- `CX` Cilusnál használjuk
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
  - `CF` Carry flag <!-- TODO Carry leírás -->
  - `ZF` Zero flag
    - 1, ha az utasítás eredménye 0
  - `SF` Sign flag
    - 1, ha az utasítás eredménye negatív
  - `OF` Overflow flag
    - 1, ha túlcsordulás történt
  - Parity, Auxiallary
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
  - Jump if Signed (negative)/Jump is not Signed (Positive or zero)

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
int 10h;
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
mov cx, 5   ; Hányszor szeretnénk a cilust lefuttatni
loop_start:
    ; Kiírjuk, hogy "Hello "
    mov ah, 09h
    mov dx, offset msg    ; msg db "Hello $" 
    int 21h 

    ; Csökkenti `CX`-et. Ha nem 0, ugrás loop_start-ra
    loop loop_start 
```
