# Sudoku

Posar aquí el tema de article de Albert Molins a LV i que la gent em preguntava una mica com va això. Doncs comencem amb un exemple senzill. 
(posar links a article Albert LV)
Todo el mundo ha visto un Sudoku en la sección de pasatiempos de su revista o periodico. Pero no siempre es fácil, ¿Verdad? Aquí les dejo un programa para que puedan dejar boquiabiertos a sus amigos. 

 
## Estrategia
 
Una estratégia para solucionar el sudoky del domingo de La Vanguardia que la mayoría de mis amigos usa consiste en seguir dos pasos:

* If a box has a value, then all the boxes in the same row, same column, or same 3x3 square cannot have that same value.
* If there is only one allowed value for a given box in a row, column, or 3x3 square, then the box is assigned that value.

## Algoritmo

### Introducción
Se trata de un simble algoritmo en Python que usa dos técnicas básicas de Inteligencia Artificial:
* **Constraint Propagation**: When trying to solve a problem, you'll find that there are some local constraints to each square. These constraints help you narrow the possibilities for the answer, which can be very helpful. We will learn to extract the maximum information out of these constraints in order to get closer to our solution. Additionally, you'll see how we can repeatedly apply simple constraints to iteratively narrow the search space of possible solutions. Constraint propagation can be used to solve a variety of problems such as calendar scheduling, and cryptographic puzzles.
* **Search**: In the process of problem solving, we may get to the point where two or more possibilities are available. What do we do? What if we branch out and consider both of them? Maybe one of them will lead us to a position in which three or more possibilities are available. Then, we can branch out again. At the end, we can create a whole tree of possibilities and find ways to traverse the tree until we find our solution. This is an example of how search can be used.

### Notación y nomenclatura
Siguiendo el [post fantástico de Peter Norvig](http://norvig.com/sudoku.html) en el que me baso para este ejemplo, Since we're writing an agent to solve the Sudoku puzzle, let's start by labelling rows and columns. The rows will be labelled by the letters A, B, C, D, E, F, G, H, I. The columns will be labelled by the numbers 1, 2, 3, 4, 5, 6, 7, 8, 9. Here we can see the labels for the rows and columns (the 3x3 squares won't be labelled):

 ```
 A1 A2 A3| A4 A5 A6| A7 A8 A9     
 B1 B2 B3| B4 B5 B6| B7 B8 B9    
 C1 C2 C3| C4 C5 C6| C7 C8 C9    
--------+---------+---------    
 D1 D2 D3| D4 D5 D6| D7 D8 D9    
 E1 E2 E3| E4 E5 E6| E7 E8 E9    
 F1 F2 F3| F4 F5 F6| F7 F8 F9    
--------+---------+---------    
 G1 G2 G3| G4 G5 G6| G7 G8 G9    
 H1 H2 H3| H4 H5 H6| H7 H8 H9   
 I1 I2 I3| I4 I5 I6| I7 I8 I9    
 
 ```
 En el código vamos a usar la siguiente nomenclatura:
 
* The individual squares at the intersection of rows and columns will be called `boxes` . These boxes will have labels 'A1', 'A2', ..., 'I9'.
* The complete rows, columns, and 3x3 squares, will be called `units`. Thus, each unit is a set of 9 boxes, and there are 27 units in total.
* For a particular box (such as 'A1'), sus pares ( `peers`)  will be all other boxes that belong to a common unit (namely, those that belong to the same row, column, or 3x3 square). Por tanto para cada casilla, hay 20 pares.  Por ejemplo los pares de 'A1' son :  row: A2, A3, A4, A5, A6, A7, A8, A9 column: B1, C1, D1, E1, F1, G1, H1, I1 3x3 square: B2, B3, C2, C3 (since A1, A2, A3, B1, C1 are already counted).



### Implementación del tablero en Python


We'll record the puzzles in two ways — as a `string` and as a `dictionary`.
* The string will consist of a concatenation of all the readings of the digits in the rows, taking the rows from top to bottom. If the puzzle is not solved, we can use a **.** para indicar que la casilla aun no tiene valor asignado.
* We'll implement the dictionary as follows. The *keys* will be strings corresponding to the boxes — namely, `'A1', 'A2', ..., 'I9'`. The values will either be the digit in each box (if there is one) or a '.' (if not).

Vamos a empezar programando una función de ayuda `cross(a, b)` que dados dos strings `a` y `b` la función retorna la lista  (recordemos que se especifica con `[` `]`) formada por todas las posibles concatenaciones de letras `s`en el string `a` con la `t` en el string `b`.  

```python
def cross(a, b):
      return [s+t for s in a for t in b]
```

Por ejemplo `cross('abc', 'def')` retornara la lista `['ad', 'ae', 'af', 'bd', 'be', 'bf', 'cd', 'ce', 'cf']`. Ahora, para crear todas las etiquetas de las casillas podemos hacerlo de la siguiente manera:


```python
rows = 'ABCDEFGHI'
cols = '123456789'

boxes = cross(rows, cols)

boxes =
    ['A1', 'A2', 'A3', 'A4', 'A5', 'A6', 'A7', 'A8', 'A9',
     'B1', 'B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B9',
     'C1', 'C2', 'C3', 'C4', 'C5', 'C6', 'C7', 'C8', 'C9',
     'D1', 'D2', 'D3', 'D4', 'D5', 'D6', 'D7', 'D8', 'D9',
     'E1', 'E2', 'E3', 'E4', 'E5', 'E6', 'E7', 'E8', 'E9',
     'F1', 'F2', 'F3', 'F4', 'F5', 'F6', 'F7', 'F8', 'F9',
     'G1', 'G2', 'G3', 'G4', 'G5', 'G6', 'G7', 'G8', 'G9',
     'H1', 'H2', 'H3', 'H4', 'H5', 'H6', 'H7', 'H8', 'H9',
     'I1', 'I2', 'I3', 'I4', 'I5', 'I6', 'I7', 'I8', 'I9']


```
Podemos comprobarlo con `print boxes` o `print (boxes)` (dependiendo si usamos Pyhon 2.x o 3.x) que nos dará la siguiente salida:
```python

    ['A1', 'A2', 'A3', 'A4', 'A5', 'A6', 'A7', 'A8', 'A9',
     'B1', 'B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B9',
     'C1', 'C2', 'C3', 'C4', 'C5', 'C6', 'C7', 'C8', 'C9',
     'D1', 'D2', 'D3', 'D4', 'D5', 'D6', 'D7', 'D8', 'D9',
     'E1', 'E2', 'E3', 'E4', 'E5', 'E6', 'E7', 'E8', 'E9',
     'F1', 'F2', 'F3', 'F4', 'F5', 'F6', 'F7', 'F8', 'F9',
     'G1', 'G2', 'G3', 'G4', 'G5', 'G6', 'G7', 'G8', 'G9',
     'H1', 'H2', 'H3', 'H4', 'H5', 'H6', 'H7', 'H8', 'H9',
     'I1', 'I2', 'I3', 'I4', 'I5', 'I6', 'I7', 'I8', 'I9']


```
